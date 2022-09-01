# I/O多路复用

### 传统网络I/O模型



![fun wangluo](E:\images\net.jpg)

### Linux一切皆文件

![linux file](E:\images\block.jpg)

对文件执行 read()，write() 的具体流程如下

1. 当执行 read() 时，会从内核读缓冲区中读取数据，如果缓冲区中没有数据，则会**阻塞等待**，等数据到达后，会通过 DMA 拷贝将数据拷贝到内核读缓冲区中，然后会唤醒用户线程将数据从内核读缓冲区拷贝到应用缓冲区中.
2. 当执行 write() 时，会将数据从应用缓冲区拷贝到内核写缓冲区，然后再通过 DMA 拷贝将数据从写缓冲区发送到设备上传输出去，如果写缓冲区满，则 write 会**阻塞等待**写缓冲区可写.

经过以上分析，我们可以看到传统的 socket 通信会阻塞在 connect，accept，read/write 这几个操作上，这样的话如果 server 是单进程/线程的话，只要 server 阻塞，就不能再接收其他 client 的处理了，由此可知**传统的 socket 无法支持高并发场景**。

### 阻塞I/O



![func read](E:\images\read.jpg)

### 非阻塞I/O

![fun nonblock_read](E:\images\nonblock_read.jpg)