# CPU高排查

结合linux基本命令和jmap,jstack等工具。

根据top命令，发现PID为28555的Java进程占用CPU高达200%，出现故障。

通过ps aux \| grep PID命令，可以进一步确定是哪个进程出现了问题。怎么定位到具体线程或者代码呢？ 显示当前java进程的线程列表 ps -mp pid -o THREAD,tid,time

从中可以找到了耗时最高的线程28802。

其次将需要的线程ID转换为16进制格式： printf "%x\n" tid

本地打印线程的堆栈信息_（注意主工程由于{}JVM Xms{}设置非常大，不可线上运行时执行！_） jstack pid \|grep tid -A 30

这样，对应异常找到出现问题的代码。

二，linux常用确认服务状态命令。

统计tcp连接状态： netstat -n \| awk '/^tcp/ {++S\[$NF\]} END{for\(a in S\) print a, S\[a\]}'

统计用户进程当前操作句柄数：lsof -n\|awk '{print $2}｜'\|sort\|uniq -c \|sort -nr\|more

统计用户进程内部执行的线程数：ps -eLf\|grep java\|wc --l

查看运行时进程参数设置：cat /proc/pid/limits,cpuset及coredump\_filter

查看当前某端口连接数：netstat -nat\|grep -i "80" \|wc -l

对连接的IP按连接数量进行排序：netstat -ntu \| awk '{print $5}' \| cut -d: -f1 \| sort \| uniq -c \| sort -n （运维操作已放弃netstat，改为ss命令入侵更轻量）

Tcpdump访嗅访问数据包最高请求：tcpdump -i eth0 -tnn dst port 80 -c 1000 

