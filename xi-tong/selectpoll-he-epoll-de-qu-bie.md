# select、poll和epoll的区别

操作系统在处理io的时候，主要有两个阶段：

等待数据传到io设备 io设备将数据复制到user space 我们一般将上述过程简化理解为：

等到数据传到kernel内核space kernel内核区域将数据复制到user space（理解为进程或者线程的缓冲区） select，poll，epoll都是IO多路复用的机制。I/O多路复用就通过一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。但select，poll，epoll本质上都是同步I/O，因为他们都需要在读写事件就绪后自己负责进行读写，也就是说这个读写过程是阻塞的，而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。

select 单个进程就可以同时处理多个网络连接的io请求（同时阻塞多个io操作）。基本原理就是程序呼叫select，然后整个程序就阻塞状态，这时候，kernel内核就会轮询检查所有select负责的文件描述符fd，当找到其中那个的数据准备好了文件描述符，会返回给select，select通知系统调用，将数据从kernel内核复制到进程缓冲区\(用户空间\)。

下图为select同时从多个客户端接受数据的过程

虽然服务器进程会被select阻塞，但是select会利用内核不断轮询监听其他客户端的io操作是否完成

Poll介绍 poll的原理与select非常相似，差别如下：

描述fd集合的方式不同，poll使用 pollfd 结构而不是select结构fd\_set结构，所以poll是链式的，没有最大连接数的限制 poll有一个特点是水平触发，也就是通知程序fd就绪后，这次没有被处理，那么下次poll的时候会再次通知同个fd已经就绪。 select的几大缺点：

（1）每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大

（2）同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大

（3）select支持的文件描述符数量太小了，默认是1024

细谈事件驱动--&gt;epoll epoll 提供了三个函数：

int epoll\_create\(int size\); 建立一個 epoll 对象，并传回它的id

int epoll\_ctl\(int epfd, int op, int fd, struct epoll\_event \*event\); 事件注册函数，将需要监听的事件和需要监听的fd交给epoll对象

int epoll\_wait\(int epfd, struct epoll\_event \*events, int maxevents, int timeout\); 等待注册的事件被触发或者timeout发生

epoll解决的问题：

epoll没有fd数量限制 epoll没有这个限制，我们知道每个epoll监听一个fd，所以最大数量与能打开的fd数量有关，一个g的内存的机器上，能打开10万个左右

epoll不需要每次都从用户空间将fd\_set复制到内核kernel epoll在用epoll\_ctl函数进行事件注册的时候，已经将fd复制到内核中，所以不需要每次都重新复制一次

select 和 poll 都是主动轮询机制，需要遍历每一个人fd； epoll是被动触发方式，给fd注册了相应事件的时候，我们为每一个fd指定了一个回调函数，当数据准备好之后，就会把就绪的fd加入一个就绪的队列中，epoll\_wait的工作方式实际上就是在这个就绪队列中查看有没有就绪的fd，如果有，就唤醒就绪队列上的等待者，然后调用回调函数。

虽然epoll。poll。epoll都需要查看是否有fd就绪，但是epoll之所以是被动触发，就在于它只要去查找就绪队列中有没有fd，就绪的fd是主动加到队列中，epoll不需要一个个轮询确认。 换一句话讲，就是select和poll只能通知有fd已经就绪了，但不能知道究竟是哪个fd就绪，所以select和poll就要去主动轮询一遍找到就绪的fd。而epoll则是不但可以知道有fd可以就绪，而且还具体可以知道就绪fd的编号，所以直接找到就可以，不用轮询。 

总结 select, poll是为了解決同时大量IO的情況（尤其网络服务器），但是随着连接数越多，性能越差 epoll是select和poll的改进方案，在 linux 上可以取代 select 和 poll，可以处理大量连接的性能问题.

答案2：

select poll每次循环调用时，都需要将描述符和文件拷贝到内核空间；epoll 只需要拷贝一次； 对于这种情况对于描述符数量不大还可以，但是当描述符的数量达到十几万甚至上百万的时候，效率就会急速降低，因为每一次轮询的时候都需要将这些所有的socket文件从用户态拷贝到内核态，会造成大量的浪费和资源开销； select每次返回后，都需要遍历所有的描述文件才能找到就绪的，时间复杂度为O\(n\)，而epoll则需要O\(1\)； select poll内核是通过内核方式来完成的，时间复杂度O\(n\)；epoll在每个描述文件上设置回调函数，时间复杂度为O\(1\)； select相比poll没有太大的改进，唯一的区别就是使用链表来保存fd，使的能监听的数量远远大于1024，但对于将用户数据拷贝到内存空间，线性遍历fd这两个没有太大的改变。



