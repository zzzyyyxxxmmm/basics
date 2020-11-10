### [Streams and Monk – How Yelp is Approaching Kafka in 2020](https://engineeringblog.yelp.com/2020/01/streams-and-monk-how-yelp-approaches-kafka-in-2020.html)

将不同的stream分类, 分为紧急非紧急, 从而根据不同情况来部署cluster

另外创建topic也是根据用户指定的信息来在不同的cluser上创建

之后用到了一个叫Monk的unified stream ingestion pipeline, 对于fire-forget, 每次produce信息, 都是先produce到一个1M的memory (disk, disk可以避免信息丢失)中, 由其他的工作线程来读取然后push到topic里, 相当于一个缓冲, 避免overload