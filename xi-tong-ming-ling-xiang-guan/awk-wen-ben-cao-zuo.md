# AWK文本操作

  
当前WEB服务器中**联接次数最多的ip地址**

\#netstat -ntu \|awk '{print $5}' \|sort \| uniq -c\| sort -nr

查看日志中**访问次数最多的前10个IP**

\#cat access\_log \|cut -d ' ' -f 1 \|sort \|uniq -c \| sort -nr \| awk '{print $0 }' \| head -n 10 \|less

查看日志中**出现100次以上的IP**

\#cat access\_log \|cut -d ' ' -f 1 \|sort \|uniq -c \| awk '{if \($1 &gt; 100\) print $0}'｜sort -nr \|less

**查看最近访问量最高的文件**

\#cat access\_log \|tail -10000\|awk '{print $7}'\|sort\|uniq -c\|sort -nr\|less

**查看日志中访问超过100次的页面**

\#cat access\_log \| cut -d ' ' -f 7 \| sort \|uniq -c \| awk '{if \($1 &gt; 100\) print $0}' \| less 

**统计某url，一天的访问次数**

\#cat access\_log\|grep '12/Aug/2009'\|grep '/images/index/e1.gif'\|wc\|awk '{print $1}'

**前五天的访问次数最多的网页**

\#cat access\_log\|awk '{print $7}'\|uniq -c \|sort -n -r\|head -20

**从日志里查看该ip在干嘛**

\#cat access\_log \| grep 218.66.36.119\| awk '{print $1"\t"$7}' \| sort \| uniq -c \| sort -nr \| less

**列出传输时间超过** **30** **秒的文件**

\#cat access\_log\|awk '\($NF &gt; 30\){print $7}' \|sort -n\|uniq -c\|sort -nr\|head -20

**列出最最耗时的页面\(超过60秒的\)**

\#cat access\_log \|awk '\($NF &gt; 60 && $7~/\.php/\){print $7}' \|sort -n\|uniq -c\|sort -nr\|head -100

`uniq -c`：去重

`sort` 、`sort -r`：将数字当做字符进行排序

`sort -n`、`sort -nr`： 按照整个数字来排序

