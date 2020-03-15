​1. FastDFS适用的场景以及不适用的场景
    FastDFS是为互联网应用量身定做的一套分布式文件存储系统，非常适合用来存储图片、音频、视频、文档等文件。对于互联网应用，简洁高效的FastDFS和其他分布式文件系统相比，优势非常明显。具体情况大家可以查阅相关介绍文档，如：FastDFS架构设计文档等等。
    出于简洁考虑，FastDFS没有对文件做分块存储，因此不太适合分布式计算场景。

2. 服务器时间必须保持一致
    因为FastDFS的精巧设计不需要存储文件索引，FastDFS通过比较时间戳来判断文件是否同步完成。因此集群内的服务器时间要保持一致，各台服务器的时间差值不要超过1秒。建议采用NTP对时服务。

3. too many open files错误解决方法
   日志中报打开文件过多的错误，是因为系统允许一个进程打开的文件数设置太小了。Linux环境下的解决办法，修改文件/etc/security/limits.conf，在文件尾部添加如下代码（如果已经存在则修改相应数值）：
root soft nofile 65535
root hard nofile 65535
* soft nofile 65535
* hard nofile 65535
    注：只配置最后两行不就可以了吗，为啥还要单独为root用户配置呢？查了网上资料，说是*这样的通配符对root用户无效，所以root需要单独配置（嗯，阿里云ECS就配置了上面这4行）。

4. FastDFS服务启停
    FastDFS server程序自带start、stop和restart指令，命令行示例如下：
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf  [start | stop | restart]
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf  [start | stop | restart]

    可以使用kill或者killall正常杀掉 fdfs_trackerd 和 fdfs_storaged 进程，但
千万不要加上-9参数强杀，否则可能会导致binlog数据丢失等问题。

5. FastDFS支持断点续传吗？
    上传和下载文件均可支持。
    对于文件上传，需要先上传appender类型的文件，然后使用apend方法。
    如果要上传超过1GB的大文件，建议采用append方式分多次上传，比如每次上传64MB。需要先创建appender类型的文件，可以创建空的appender文件。

     对于超大文件，如果想支持多线程上传以加快上传速度，可以采用如下3个步骤实现：
      1）上传appender类型的文件；
      2)  调用truncate方法将该appender文件设置为最终文件大小；
      3）调用modify方法并发上传文件分片。

    对于文件下载，FastDFS可以指定文件偏移量和获取的文件内容大小。利用这个特性，文件下载可以实现断点续传以及多线程下载。

6. Java SDK非线程安全
   FastDFS提供的Java SDK是非线程安全的，有人已经踩过这个坑了。包括负责与tracker server交互的TrackerClient、与storage server直接通信的StorageClient 和 StorageClient1 这三个类均是非线程安全的。

   为啥会出现两个StorageClient字样的类名呢？二者实现功能完全一样，StorageClient是group和filename分离的用法，StorageClient1是group和filename合体用法（文件ID）。通常使用StorageClient1就好。

    大家在部署和使用FastDFS的过程中有任何疑问，欢迎在FastDFS QQ群或微信公众号交流。我可以根据大家的反馈和交流结果，继续整理FastDFS常见问题。