## Bin log Redo log
### 流程
1 prepare阶段 2 写binlog 3 commit
当在2之前崩溃时
重启恢复：后发现没有commit，回滚。备份恢复：没有binlog 。
一致
当在3之前崩溃
重启恢复：虽没有commit，**但满足prepare和binlog完整，所以重启后会自动commit**。备份：有binlog. 一致


但是正常重启innodb 要做崩溃恢复的，它一看，没redolog, 这个事务是没有的。但是binlog里面已经有了，用你的方法事务就存在。


问题0： 什么时候redo log 会写入落档
答：到了提交阶段，就是把logbuffer写入redolog。
   第二阶段，其实就是binlog和redolog最终提交的阶段

**问题1：如果有多条语句，是不是都提交后才写redo log ，数据先在内存，然后回来在统一写binlog会出现部分内容成功部分失败吗，底层fileWrite应该要保证？ **
答：事务开始时候每条语句在引擎中已经执行，写道redo log 的buffer中吗？在commit时候，相当于二阶段的prepare阶段开始，redo log 都写好。binlog在写上去，在commit，让redolog从缓存写入磁盘。

问题2：“即使写进磁盘，也没关系，再次读到内存以后，还是原来的逻辑”，老师能否扩展下这个解答：是不是指，即便写到磁盘，磁盘的数据也有事务版本信息来保障？
 答：的，一个数据行的trx_id是一个数据的一部分，写磁盘的时候也是带着的。

问题3：还有这两种日志可以在哪直观查看
答：都是文件，在数据库的data目录下，默认的，redolog是ib_log_file0这样的，binlog是xxx-bin.000001这样的

问题4：mysql的redo log有没有log buffer？对于高并发事务或者批量事务 如何优化redo文件的写入瓶颈？
答：有buffer的，事务整个完成才学到redolog文件。redolog是顺序写

问题5：默默的问一句，啥时候数据真正的落到磁盘，进去表中？？？是commit之后吗
答：: 不是，“打烊的时候”和“粉板记不下”的时候这类时机

问题6：老师，问一个问题，客户端发起commit的时候，收到commit成功是在把事务写到innodb_log_buffer_size就返回成功，还是要刷到binlog后才返回成功啊。
答：都要。而且不止写到log_buffer,还要写到redolog文件

问题7：两阶段提交那个地方用了反证法，个人认为并不一定用了两阶段提交就一定可靠，写入redolog-->写binlog-->提交事务处于commit状态，这个是3步，可能在写完binlog后直接服务器断电，碰到过从库的binlog比主库还超前的状况（可能是binlog已通过网络传给从库，但是主库断电数据没落盘），大神可以解释下这种场景，谢谢！
答： 这就是为什么MySQL 在崩溃恢复的时候，会在主库提交这个事务



1.首先客户端通过tcp/ip发送一条sql语句到server层的SQL interface
2.SQL interface接到该请求后，先对该条语句进行解析，验证权限是否匹配
3.验证通过以后，分析器会对该语句分析,是否语法有错误等
4.接下来是优化器器生成相应的执行计划，选择最优的执行计划
5.之后会是执行器根据执行计划执行这条语句。调用6
6.进入到引擎层，首先会去innodb_buffer_pool里的data dictionary(元数据信息)得到表信息，
.在这一步会去open table,如果该table上有MDL，则等待。
如果没有，则加在该表上加短暂的MDL(S)
(如果opend_table太大,表明open_table_cache太小。需要不停的去打开frm文件)
7.通过元数据信息,去lock info里查出是否会有相关的锁信息，并把这条update语句需要的
锁信息写入到lock info里(锁这里还有待补充)
8.然后涉及到的老数据通过快照的方式存储到innodb_buffer_pool里的undo page里,并且记录undo log修改的redo
(如果data page里有就直接载入到undo page里，如果没有，则需要去磁盘里取出相应page的数据，载入到undo page里)
9.在innodb_buffer_pool的data page做update操作。并把操作的物理数据页修改记录到redo log buffer里
由于update这个事务会涉及到多个页面的修改，所以redo log buffer里会记录多条页面的修改信息。
因为group commit的原因，这次事务所产生的redo log buffer可能会跟随其它事务一同flush并且sync到磁盘上
10.同时修改的信息，会按照event的格式,记录到binlog_cache中。(这里注意binlog_cache_size是transaction级别的,不是session级别的参数,
一旦commit之后，dump线程会从binlog_cache里把event主动发送给slave的I/O线程)
11.之后把这条sql,需要在二级索引上做的修改，写入到change buffer page，等到下次有其他sql需要读取该二级索引时，再去与二级索引做merge
(随机I/O变为顺序I/O,但是由于现在的磁盘都是SSD,所以对于寻址来说,随机I/O和顺序I/O差距不大)
12.此时update语句已经完成，需要commit或者rollback。这里讨论commit的情况，并且双1
13.commit操作，由于存储引擎层与server层之间采用的是内部**XA(保证两个事务的一致性,这里主要保证redo log和binlog的原子性),**
所以提交分为prepare阶段与commit阶段
14.prepare阶段,将事务的xid写入，将binlog_cache里的进行flush以及sync操作(大事务的话这步非常耗时)
15.commit阶段，由于之前该事务产生的redo log已经sync到磁盘了。所以这步只是在redo log里标记commit
16.当binlog和redo log都已经落盘以后，如果触发了刷新脏页的操作，先把该脏页复制到doublewrite buffer里，把doublewrite buffer里的刷新到共享表空间，然后才是通过page cleaner线程把脏页写入到磁盘中
