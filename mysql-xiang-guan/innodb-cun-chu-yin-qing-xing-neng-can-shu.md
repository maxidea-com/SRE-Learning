# InnoDB 存储引擎性能参数

innodb\_buffer\_pool\_size：这是InnoDB最重要的设置，对InnoDB性能有决定性的影响。默认的设置只有8M，所以默认的数据库设置下面InnoDB性能很差。在只有InnoDB存储引擎的数据库服务器上面，可以设置60-80%的内存。更精确一点，在内存容量允许的情况下面设置比InnoDB tablespaces大10%的内存大小。

innodb\_data\_file\_path：指定表数据和索引存储的空间，可以是一个或者多个文件。最后一个数据文件必须是自动扩充的，也只有最后一个文件允许自动扩充。这样，当空间用完后，自动扩充数据文件就会自动增长（以8MB为单位）以容纳额外的数据。例如： innodb\_data\_file\_path=/disk1 /ibdata1:900M;/disk2/ibdata2:50M:autoextend两个数据文件放在不同的磁盘上。数据首先放在ibdata1 中，当达到900M以后，数据就放在ibdata2中。一旦达到50MB，ibdata2将以8MB为单位自动增长。如果磁盘满了，需要在另外的磁盘上面增加一个数据文件。

innodb\_data\_home\_dir：放置表空间数据的目录，默认在mysql的数据目录，设置到和MySQL安装文件不同的分区可以提高性能。

innodb\_log\_file\_size：该参数决定了recovery speed。太大的话recovery就会比较慢，太小了影响查询性能，一般取256M可以兼顾性能和recovery的速度 。 innodb\_log\_buffer\_size：磁盘速度是很慢的，直接将log写道磁盘会影响InnoDB的性能，该参数设定了log buffer的大小，一般4M。如果有大的blob操作，可以适当增大。

innodb\_flush\_logs\_at\_trx\_commit=2： 该参数设定了事务提交时内存中log信息的处理。

```text
1) =1时，在每个事务提交时，日志缓冲被写到日志文件，对日志文件做到磁盘操作的刷新。Truly ACID。速度慢。
2) =2时，在每个事务提交时，日志缓冲被写到文件，但不对日志文件做到磁盘操作的刷新。只有操作系统崩溃或掉电才会删除最后一秒的事务，不然不会丢失事务。
3) =0时， 日志缓冲每秒一次地被写到日志文件，并且对日志文件做到磁盘操作的刷新。任何mysqld进程的崩溃会删除崩溃前最后一秒的事务
```

innodb\_file\_per\_table：可以存储每个InnoDB表和它的索引在它自己的文件中。

transaction-isolation=READ-COMITTED: 如果应用程序可以运行在READ-COMMITED隔离级别，做此设定会有一定的性能提升。

innodb\_flush\_method： 设置InnoDB同步IO的方式：

```text
1) Default – 使用fsync（）。
2) O_SYNC 以sync模式打开文件，通常比较慢。
3) O_DIRECT，在Linux上使用Direct IO。可以显著提高速度，特别是在RAID系统上。避免额外的数据复制和double buffering（mysql buffering 和OS buffering）。
```

innodb\_thread\_concurrency： InnoDB kernel最大的线程数。

```text
1) 最少设置为(num_disks+num_cpus)*2。
2) 可以通过设置成1000来禁止这个限制
```

## 一、innodb和myisam的主要区别

**Innodb**   
1. 支持事务   
2.锁粒度==&gt;行级别   
3.支持mvcc多版本并发控制   
4.不支持地理空间   
5.最大支持64TB存储   
6.支持哈希索引   
7.不支持全文索引\(可以结合sphinx\)   
8.支持聚集索引   
9.不支持压缩数据   
10.支持外键   
11.支持缓冲数据 

**myisam**   
1.不支持事务   
2.锁粒度==&gt;表级别   
3.不支持mvcc多版本并发控制   
4.支持地理空间   
5.没有存储限制   
6.不支持哈希索引   
7.支持全文索引   
8.不支持聚集索引   
9.支持压缩数据   
10.不支持外键   
11.不支持缓冲数据

## 二、innodb相关重要配置参数

innodb中大量使用了AIO（Async IO）来处理写IO请求，这样可以极大提高数据库的性能。而IO Thread的工作主要是负责这些IO请求的回调（call back）

innodb\_read\_io\_threads 设置read thread\(读线程个数，默认是4个\)

innodb\_write\_io\_threads 设置write thread\(写线程个数，默认是4个\)

事务被提交后，其所使用的undolog（撤销日志）可能不再需要，因此需要Purge Thread（清理线程）来回收已经使用并分配的undo页。 innodb1.1版本之前，purge操作仅在innodb引擎中的Master Thread中完成。而从innodb1.1版本开始。purge操作可以独立到单独的线程中进行。以此来减轻Master Thread的工作。提高cpu使用率以及提升存储引擎性能。 innodb\_purge\_thread = 1 注意：在innodb1.1版本中，即使该值设置大于1，innodb启动时也会将其设置为1。并在错误文件中给出警告。 innodb1.2支持多个Pugre Thread线程。

innodb\_buffer\_pool\_instances = 2 设置多个缓冲池实例，每个页根据哈希值平均分配到不同缓冲池实例中。好处在于减少数据库内部资源竞争。增加数据库并发处理能力。

innodb\_old\_blocks\_pct LRU算法，默认值是37，插入到LRU列表端的37%，差不多3/8的位置。innodb把midpoint之后的列表称为old列表，之前的列表称为new列表，可以理解为new列表中的页都是最为活跃的热点数据。

innodb\_old\_blocks\_time 当第一次读取到的新页，innodb会根据innodb\_old\_blocks\_pct设置的值将读取到的页插入LRU列表指定的midpoint（中间点）位置，如果没有修改，默认是37%,3/8的位置。防止将真正的new列表中的热点数据刷出。当再次读取到old列表中的页时也就是midpoint之后的页，innodb\_old\_blocks\_time用于读取到old列表中的页时需要等待多久才会被加入到LRU列表的首部。防止new列中的热点数据被刷出。innodb\_old\_blocks\_time默认值是0，没有等待。

innodb\_log\_buffer\_size 默认是8M，用来缓冲日志数据的缓冲区的大小，当此值快满时, InnoDB将必须刷新数据到磁盘上. 由于基本上每秒都会刷新一次,所以没有必要将此值设置的太大\(甚至对于长事务而言\)

innodb\_max\_dirty\_pages\_pct = 75 在InnoDB缓冲池中最大允许的脏页面的比例，当缓冲池中脏页的数量占据75%时，强制进行checkpoin，刷新一部分的脏页到磁盘。

innodb\_io\_capacity 表示磁盘io的吞吐量，默认值是200.对于刷新到磁盘页的数量，会按照inodb\_io\_capacity的百分比来进行控制。 规则如下： 在合并插入缓冲时，合并插入缓冲数量为innodb\_io\_capacity 值的5% 在从缓冲池刷新脏页时，刷新脏页的数量为innodb\_io\_capacity的值，也就是默认值200. 如果使用了ssd，或者raid磁盘时，磁盘拥有更高的io速度，可以适当增加该参数的值。

innodb\_adaptive\_flushing 自适应刷新脏页。默认已经开启。

innodb\_purge\_batch\_size 在进行full purge时，回收Undo页的个数，默认是20，可以适当加大。

innodb\_fast\_shutdown 在关闭时，该参数影响表innodb存储引擎行为。参数可取值为0,1,2，默认值是1. 0 表示在mysql数据库关闭时，innodb需要完成所有的full purge，merge insert buffer，并且将所有脏页刷新回磁盘。如果在innodb进行升级时，必须将这个参数设为0.然后在关闭数据库. 1 默认值，表示不需要完成full purge和merge insert buffer，但是在缓冲池中的一些数据脏页还是会刷新回磁盘。 2 表示不完成full purge和merge inser buffer操作，也不将缓冲池中的数据脏页刷新回磁盘，而是将日志写入日志文件，这样不会有事务丢失，但是下次数据库启动时，会进行恢复操作（recovery）

innodb\_force\_recovery 该参数影响了整个innodb存储引擎的恢复状况。该参数默认值为0，代表发生需要恢复时，进行所有的恢复操作，当不能进行有效恢复时，如数据页发生了损坏，把错误写入错误日志。 还可以设置为6个非零值：1-6，大的数字包含了前面所有小数字的影响。具体如下 1 忽略检查到的corrupt页 2 阻止master thread线程运行。如master thred需要进行full purge操作，会导致crash。 3 不进行事务的回滚。 4 不进行插入缓冲的合并操作 5 不查看撤销日志（undo log），innodb存储引擎会将未提交的事务视为已提交。 6 不进行回滚的操作 注意： 当该参数设置大于0以后，用户可以对表进行select，create，drop，但是innsert，update，delete这类DML操作却是不允许的。

以下参数影响着二进制日志记录信息的行为： max\_binlog\_size binlog\_cache\_szie sync\_binlog binlog-do-db binlog-ignore-db log-slave-update master==&gt;slave==&gt;slave 架构必须开启此参数（slave端开启） binlog\_format

其中比较重要的是sync\_binlog参数，如果想让replication得到最大的可用性。最好将该值设置为1.表示采用同步写磁盘的方式来写二进制日志。不过有一点是会给IO带来一定压力。 当sysnc\_binlog设置为1以后，还有一个参数需要设置。innodb\_support\_xa.也需要将该值设置为1.避免事务不能被回滚的问题。 当进行replication时，最好将二进制格式改为row，避免master使用了rand，uuid等函数，或者触发器，导致主从服务器数据不一致。通常将binlog\_format设置为row格式。

innodb\_data\_file\_path 表空间的设置，默认配置下有一个初始大小为10MB，名为ibdata1的文件。自动增长。 用户可以通过多个文件组成一个表空间，同时定制文件属性，设置如下： innodb\_data\_file\_path= /db/ibdata1:2000M;/db2/ibdata2:2000M:autoextend 若这两个文件位于不同磁盘上，磁盘的负载可能被平均。因此可以提高数据库整体性能。

innodb\_file\_per\_table 默认没有开启 将每个基于innodb引擎的表使用独立的表空间。

以下参数影响重做日志文件属性： innodb\_log\_file\_size 重做日志文件的大小。innodb1.2以前，大小不得超过4G。1.2x以后可以最大到512G innodb\_log\_files\_in\_group 指定了日志文件组中重做日志文件的数量。默认为2 innodb\_mirrored\_log\_groups 指定了日志镜像文件组的数量，默认为1.表示只有一个文件组，没有使用镜像。若磁盘使用了类似磁盘阵列，可以不开启重做日志镜像功能。 innodb\_log\_group\_home\_dir 指定日志文件组所在路径。默认为./，表示在mysql数据库目录下。 注意： 重做日志文件不能设置的太大，否则在恢复时可能需要很长的时间；如果设置的太小，可能导致一个事务日志需要多次切换重做日志文件。此外，重做日志文件太小会导致频繁的发生async checkpoint，导致性能抖动。

innodb\_flush\_log\_at\_trx\_commit 该参数表示在提交操作时（commit），处理重做日志的方式。有效值有0,1,2。 0 代表当提交事务时，并不将事务的重做日志写入磁盘上的日志文件，而是等待主线程（master thread）每秒刷新。 1,2 不同的地方在于： 1 表示在执行commit时将重做日志缓冲同步到写到磁盘。既有fsync的调度。 2 表示将重做日志异步写到磁盘，既写到文件系统的缓冲中。因此不能完全保证在执行commit时肯定会写入重做日志文件，只是有这个动作发生。 所以当为了保证事务的ACID中的持久性，必须将该值设置为1 。也就是每当有事务提交时，就必须确保事务都已经写入重做日志文件。 

