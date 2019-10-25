# kafka8、tcp backlog

cat /proc/sys/net/core/somaxconn
echo 511 > /proc/sys/net/core/somaxconn


syns queue用于保存半连接状态的请求，其大小通过 /proc/sys/net/ipv4/tcp_max_syn_backlog 指定，一般默认值是 512，不过这个设置有效的前提是系统的syncookies功能被禁用。互联网常见的 TCP SYN FLOOD 恶意 DOS 攻击方式就是建立大量的半连接状态的请求，然后丢弃，导致 syns queue 不能保存其它正常的请求。
accept queue用于保存全连接状态的请求，其大小通过 /proc/sys/net/core/somaxconn 指定，在使用 listen函数时，内核会根据传入的 backlog 参数与系统参数 somaxconn，取二者的较小值。
。



对于SYN flood攻击，调整下面三个参数就可以防范绝大部分的攻击了。

增大tcp_max_syn_backlog
减小tcp_synack_retries
启用tcp_syncookie




重要的事情再说一遍： I/O multiplexing 这里面的 multiplexing 指的其实是在单个线程通过记录跟踪每一个Sock(I/O流)的状态(对应空管塔里面的Fight progress strip槽)来同时管理多个I/O流. 发明它的原因，是尽量多的提高服务器的吞吐能力。


select poll epoll的区别及epoll的底层实现

Select  poll每次循环调用时，都需要将描述符和事件拷贝到内核空间；epoll只需要拷贝一次；
这种情况在对于描述符数量不大的情况下还可以，但是当描述符的数量达到十几万甚至上百万的时候，他们的效率就会急速降低，因为每一次轮询都需要将这些所有的socket描述符从用户态拷贝到内核态，会造成大量的浪费和资源开销；

      2.Select poll每次返回后，需要遍历所有描述符才能找到就绪的，因此他两的时间复杂度为o(n),而epoll则只需要O(1);

       3.Select poll内核是通过轮询的方式完成，时间复杂度为O(N);epoll是在每个描述符上设置回调函数，时间复杂度为O(1)；
select，poll实现需要自己不断轮询所有fd集合，直到设备就绪，期间可能要睡眠和唤醒多次交替。而epoll其实也需要调用epoll_wait不断轮询就绪链表，期间也可能多次睡眠和唤醒交替，但是它是设备就绪时，调用回调函数，把就绪fd放入就绪链表中，并唤醒在epoll_wait中进入睡眠的进程。虽然都要睡眠和交替，但是select和poll在“醒着”的时候要遍历整个fd集合，而epoll在“醒着”的时候只要判断一下就绪链表是否为空就行了，这节省了大量的CPU时间。这就是回调机制带来的性能提升
 

与select相比poll没有太多的区别，唯一的改进是使用了链表来保存fd，使得能够监听的数量远远超过了1024，但是对于将用户数据拷贝到系统空间，线性遍历fd这两个并没有太大的改变。


高并发架构系列：数据库主从同步的3种一致性方案实现，及优劣比较 https://youzhixueyuan.com/database-master-slave-synchronization.html
