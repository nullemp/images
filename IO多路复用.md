# I/O多路复用

### 一、传统网络I/O模型



![fun wangluo](E:\images\net.jpg)

### 二、Linux一切皆文件

![linux file](E:\images\block.jpg)

对文件执行 read()，write() 的具体流程如下

1. 当执行 read() 时，会从内核读缓冲区中读取数据，如果缓冲区中没有数据，则会**阻塞等待**，等数据到达后，会通过 DMA 拷贝将数据拷贝到内核读缓冲区中，然后会唤醒用户线程将数据从内核读缓冲区拷贝到应用缓冲区中.
2. 当执行 write() 时，会将数据从应用缓冲区拷贝到内核写缓冲区，然后再通过 DMA 拷贝将数据从写缓冲区发送到设备上传输出去，如果写缓冲区满，则 write 会**阻塞等待**写缓冲区可写.

经过以上分析，我们可以看到传统的 socket 通信会阻塞在 connect，accept，read/write 这几个操作上，这样的话如果 server 是单进程/线程的话，只要 server 阻塞，就不能再接收其他 client 的处理了，由此可知**传统的 socket 无法支持高并发场景**。

### 三、阻塞I/O



![func read](E:\images\read.jpg)

### 四、怎么办？？？

#### 1.非阻塞I/O

![fun nonblock_read](E:\images\nonblock_read.jpg)

```c
int setnonblocking(int fd)
{
	int old_option = fcntl(fd, F_GETFL);
	int new_option = old_option | O_NONBLOCK;
	fcntl(fd, F_SETFL, new_option);
	return old_option;
}
```







### 2.多进程/多线程

```c
// Server Pseudocode
while(1) {
  listenfd = socket(...);
  ......  
  connfd = accept(listenfd);  // block
    
  pthread_create（doWork);  	 
}

void doWork() {
  int n = read(connfd, buf);  // block
  doSomeThing(buf);  
  close(connfd);     // close socket
}
```

```c
while(1) {
  connfd = accept(listenfd);  // block
  
  if (fork() == 0) {
    // accept 后子进程开始工作
    doWork(connfd);
  }
}
void doWork(connfd) {
  int n = read(connfd, buf);  // block
  doSomeThing(buf);  // 利用读到的数据做些什么
  close(connfd);     // 关闭连接，循环等待下一个连接
}
```

通过这种方式确实解决了单进程 server 阻塞无法处理其他 client 请求的问题，但众所周知 fork 创建子进程是非常耗时的，包括页表的复制，进程切换时页表的切换，每来一个请求就创建一个进程也没那么多资源。，

为了节省进程创建的开销，于是有人提出把多进程改成多线程，创建线程（使用 pthread_create）的开销确实小了很多，但同样的，线程与进程一样，都需要占用堆栈等资源，而且碰到阻塞，唤醒等都涉及到用户态，内核态的切换，这些都极大地消耗了性能。

<font color="red">【一个进程最多可以创建多少个进程或者线程？？？】</font>

**NOTE**: 在 Linux 下进程和线程都是用统一的 task_struct 表示，区别不大，所以下文描述不管是进程还是线程区别都不大.

