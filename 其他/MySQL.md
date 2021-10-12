### MySQL

#### 1、MySQL体系结构和存储引擎

- MySQL基本上能保证在各平台上的物理体系结构的一致性
- 支持全文索引

##### （1）数据库和实例

- 术语
  - 数据库 (database）：物理操作系统文件或其他形式文件类型的集合
    - 数据库文件可以是 frm、MYD、MYI、ibd结尾的文件
    - 当使用NDB引擎时，数据库的文件可能不是操作系统上的文件，而是存放于内存之中的文件
  - 实例 ( instance) ：MySQL数据库由后台线程以及一个共享内存区组成
    - 共享内存可以被运行的后台线程所共享
    - 数据库实例才是真正用于操作数据库文件的
  - 在MySQL数据库中，实例与数据库的关通常系是一一对应的，即一个实例对应一个数据库，一个数据库对应一个实例
    - 在集群情况下可能存在一个数据库被多个数据实例使用的情况
- MySQL被设计为一个单进程多线程架构的数据库，这也就是说，MySQL数据库实例在系统上的表现就是一个进程。
- Linux操作系统
  - 启动MySQL数据库实例：`# ./mysqld_safe&`
    - 当启动实例时，MySQL数据库会去读取配置文件，根据配置文件的参数来启动数据库实例，可以没有配置文件，在这种情况下，MySQL会按照编译时的默认参数设置启动实例
    - 查看当MySQL数据库实例启动时，会在哪些位置查找配置文件：`# mysql --help | grep my.cnf` 
    - 如果几个配置文件中都有同一个参数，MySQL数据库会以读取到的最后一个配置文件中的参数为准
    - Linux环境下，配置文件一般放在 letc/my.cnf下。Windows平台下，配置文件的后缀名可能是.cnf，也可能是.ini
  - 观察MySQL数据库启动后的进程情况：`# ps -ef | grep mysqld` 

##### （2）体系结构

<img src="C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210706200925476.png" alt="image-20210706200925476" style="zoom: 67%;" />

- 组成
  - 连接池组件
  - 管理服务和工具组件
  - SQL接口组件
  - 查询分析器组件
  - 优化器组件
  - 缓冲(Cache）组件
  - **插件式存储引擎**
    - 基于表，而不是数据库
  - 物理文件

##### （3）InnoDB存储引擎

- 支持事务，面向在线事务处理(OLTP)的应用
- 特点：行锁设计、支持外键、支持非锁定读，即默认读取操作不会产生锁
- 从5.5.8版本开始，InnoDB存储引擎是默认的存储引擎
- 将数据放在一个逻辑的表空间中，像黑盒一样由InnoDB存储引擎自身管理。从 MySQL4.1（包括4.1）版本开始，将每个InnoDB存储引擎的表单独存放到一个独立的ibd文件中。InnoDB存储引擎支持用裸设备（row disk）用来建立其表空间
- 通过使用多版本并发控制(MVCC）来获得高并发性，实现了SQL标准的4种隔离级别，默认为REPEATABLE级别。同时使用next-keylocking 的策略来避免幻读（phantom）现象的产生。除此之外，InnoDB储存引擎还提供了插入缓冲（insert buffer)、二次写(double write)、自适应哈希索引 ( adaptive hash index）、预读（read ahead）等高性能和高可用的功能
- 对于表中数据的存储采用聚集(clustered）的方式，每张表的存储都是按主键的顺序进行存放。如果没有显式地在表定义时指定主键，InnoDB存储引擎会为每一行生成一个6字节的ROWID，并以此作为主键

##### （4）MyISAM存储引擎

- 不支持事务、表锁设计，支持全文索引，主要面向一些OLAP数据库应用。在MySQL 5.5.8版本之前是默认的存储引擎（除Windows版本外)
- 缓冲池只缓存（cache）索引文件，而不缓冲数据文件
- 表由MYD和 MYI组成，MYD用来存放数据文件，MYI用来存放索引文件。可以通过使用myisampack 工具来进一步压缩数据文件，因为myisampack工具使用赫夫曼(Huffman）编码静态算法来压缩数据，压缩后的表是只读的，也可以通过myisampack来解压数据文件
- 在MySQL 5.0版本之前，MyISAM默认支持的表大小为4GB，如果需要支持大于4GB的MyISAM表时，需要制定MAX_ROWS 和 AVG_RoW_LENGTH属性。从MySQL 5.0版本开始，MyISAM默认支持256TB的单表数据
- 对于MyISAM存储引擎表，MySQL数据库只缓存其索引文件，数据文件的缓存交由操作系统本身来完成

##### （5）NDB存储引擎

- 集群存储引擎，结构是share nothing的集群架构，能提供更高的可用性
- 特点：数据全部放在内存中（从MySQL 5.1版本开始，可以将非索引数据放在磁盘上)，因此主键查找( primary key lookups）的速度极快，通过添加NDB数据存储节点(Data Node）可以线性地提高数据库性能，是高可用、高性能的集群系统
- 连接操作(JOIN)在MySQL数据库层完成，而不是在存储引擎层完成的。复杂的连接操作需要巨大的网络开销，因此查询速度很慢

##### （6）Memory存储引擎（HEAP存储引擎）

- 将表中的数据存放在内存中，如果数据库重启或发生崩溃，表中的数据都将消失。它非常适合用于存储临时数据的临时表，以及数据仓库中的纬度表
- 默认使用哈希索引，而不是B+树索引
- 虽然速度非常快，但在使用上还是有一定的限制。如，只支持表锁、并发性能较差、不支持TEXT和BLOB列类型
- 存储变长字段( varchar）时是按照定常字段(char）的方式进行的，浪费内存
- MySQL数据库使用Memory存储引擎作为临时表来存放查询的中间结果集（ intermediate result)。如果中间结果集大于Memory存储引擎表的容量设置，或者中间结果含有TEXT或BLOB列类型字段，则 MySQL数据库会把其转换到MyISAM存储引擎表而存放到磁盘中。MyISAM不缓存数据文件，性能对于查询会有损失

##### （7）Archive存储引擎

- 只支持INSERT 和SELECT操作，从 MySQL 5.1开始支持索引
- 使用zlib算法将数据行（row）进行压缩后存储，压缩比一般可达1∶10，适合存储归档数据，如日志信息
- 使用行锁来实现高并发的插入操作，但是其本身并不是事务安全的存储引擎，主要是提供高速的插入和压缩功能

##### （8）Federated存储引擎

- 不存放数据，它只是指向一台远程MySQL数据库服务器上的表，只支持MySQL数据库表，不支持异构数据库表

##### （9）Maria存储引擎

- 主要是用来取代原有的MyISAM存储引擎，从而成为MySQL的默认存储引擎

- 特点：支持缓存数据和索引文件、应用了行锁设计、提供了MVCC功能、支持事务和非事务安全的选项、及更好的BLOB字符类型的处理性能

  <img src="C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210706205146661.png" alt="image-20210706205146661" style="zoom: 67%;" />

#### 2、连接MySQL

- 连接MySQL操作是一个连接进程和 MySQL数据库实例进行通信，本质上是进程通信

##### （1）TCP/IP

- 在任何平台下都提供的连接方式，也是网络中使用得最多的一种方式。在TCP/IP连接上建立一个基于网络的连接请求，一般情况下客户端(client）在一台服务器上，而MySQL实例( server）在另一台服务器上，这两台机器通过一个TCP/IP网络连接。如在Windows服务器下请求一台远程Linux服务器下的MySQL实例：

  ```mysql
  c:\>mysql -h192.168.0.101 -u david -p
  Enter password:
  ```

- 连接到MySQL实例时，MySQL数据库会先检查一张权限视图，用来判断发起请求的客户端IP是否允许连接到MySQL实例，该视图在mysql架构下，表名为user

##### （2）命名管道和共享内存

- 在 MySQL数据库中须在配置文件中启用--enable-named-pipe选项
- 在 MySQL4.1之后的版本中，提供共享内存的连接方式，通过在配置文件中添加--shared-memory实现，连接时，MySQL客户端还必须使用--protocol=memory选项

##### （3）UNIX域套接字

- 在Linux 和UNIX环境下，还可以使用UNIX域套接字。UNIX域套接字其实不是一个网络协议，所以只能在MySQL客户端和数据库实例在一台服务器上的情况下使用

#### 3、InnoDB存储引擎

##### （1）InnoDB架构

- InnoDB存储引擎有多个内存块，组成了一个大的内存池，负责：

  - 维护所有进程/线程需要访问的多个内部数据结构
  - 缓存磁盘上的数据，方便快速地读取，同时在对磁盘文件的数据修改之前在这里缓存
  - 重做日志（redo log）缓冲
  - ...

  <img src="C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210706211630412.png" alt="image-20210706211630412" style="zoom: 67%;" />

- 后台线程：

  - 作用：
    - 刷新内存池中的数据，保证缓冲池中的内存缓存的是最近的数据
    - 将已修改的数据文件刷新到磁盘文件，同时保证在数据库发生异常的情况下InnoDB能恢复到正常运行状态
  - InnoDB存储引擎是多线程的模型
  - Master Thread：负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，包括脏页的刷新、合并插人缓冲（INSERT BUFFER)、UNDO页的回收等
  - 在InnoDB存储引擎中大量使用了AIO (Async IO）来处理写IO请求，极大提高数据库的性能
    - IO Thread负责这些IO请求的回调(call back）处理
  - 事务被提交后，其所使用的undolog可能不再需要，需要 Purge Thread来回收已经使用并分配的undo页。从InnoDB 1.1版本开始，purge操作可以独立到单独的线程中进行，减轻Master Thread的工作，提高CPU的使用率以及提升存储引擎的性能
  - Page Cleaner Thread：将之前版本中脏页的刷新操作都放入到单独的线程中来完成。减轻原Master Thread 的工作及对于用户查询线程的阻塞，提高InnoDB存储引擎的性能

- 内存

  - 缓冲池

    - 基于磁盘存储的，并将其中的记录按照页的方式进行管理。可将其视为基于磁盘的数据库系统(Disk-base Database)

    - 由于CPU速度与磁盘速度之间的鸿沟，基于磁盘的数据库系统通常使用缓冲池技术来提高数据库的整体性能

    - 一块内存区域，通过内存的速度来弥补磁盘速度较慢对数据库性能的影响。在数据库中进行读取页的操作，首先将从磁盘读到的页存放在缓冲池中，这个过程称为将页“FIX”在缓冲池中。下一次再读相同的页时，首先判断该页是否在缓冲池中。若在缓冲池中，称该页在缓冲池中被命中，直接读取该页。否则，读取磁盘上的页

    - 对于数据库中页的修改操作，则首先修改在缓冲池中的页，然后再以一定的频率刷新到磁盘上，页从缓冲池刷新回磁盘的操作并不是在每次页发生更新时触发，而是通过Checkpoint的机制刷新回磁盘，提高数据库的整体性能

    - 缓冲池缓存的数据页类型：索引页、数据页、undo页、插入缓冲( insert buffer)、自适应哈希索引( adaptive hash index）、InnoDB存储的锁信息（lockinfo)、数据字典信息（data dictionary)等

      <img src="C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210706212635993.png" alt="image-20210706212635993" style="zoom: 67%;" />

    - 从InnoDB 1.0.x版本开始，允许有多个缓冲池实例。每个页根据哈希值平均分配到不同缓冲池实例中。减少数据库内部的资源竞争，增加数据库的并发处理能力。通过参数innodb_buffer_pool_instances配置，大于1即可

  - LRU List、Free List、Flush List

    - 通常来说，数据库中的缓冲池是通过LRU (Latest Recent Used，最近最少使用）算法来进行管理的
    - 在 InnoDB存储引擎中，缓冲池中页的大小默认为16KB，同样使用LRU算法对缓冲池进行管理，LRU列表中还加入了midpoint位置。新读取到的页，虽然是最新访问的页，但并不是直接放入到LRU列表的首部，而是放入到LRU列表的midpoint位置。这个算法称为midpoint insertion strategy。在默认配置下，该位置在LRU列表长度的5/8处。midpoint位置可由参数innodb_old_blocks_pct控制，midpoint之后的列表称为old列表，之前的列表称为new列表
    - 若直接将读取到的页放入到LRU的首部，某些SQL操作可能会使缓冲池中的页被刷新出，影响缓冲池的效率，常见操作为索引或数据的扫描操作，需要访问表中的许多页，甚至是全部的页，而这些页通常来说又仅在这次查询操作中需要，并不是活跃的热点数据
    - 参数innodb_old_blocks_time，用于表示页读取到mid位置后需要等待多久才会被加入到LRU列表的热端

    ---

    - 当数据库刚启动时，LRU列表是空的，这时页都存放在Free列表中。当需要从缓冲池中分页时，首先从Free列表中查找是否有可用的空闲页，若有则将该页从Free列表中删除，放入到LRU列表中。否则，根据LRU算法，淘汰LRU列表末尾的页，将该内存空间分配给新的页。当页从LRU列表的old部分加入到new部分时，称此时发生的操作为page made young，而因为innodb_old_blocks_time的设置而导致页没有从old部分移动到new部分的操作称为page not made young。可以通过命令SHOW ENGINE INNODB STATUS来观察LRU列表及Free列表的使用情况和运行状态（不是当前状态）

      > Free buffers表示当前Free列表中页的数量
      >
      > Database pages表示LRU列表中页的数量
      >
      > pages made young 显示了LRU列表中页移动到前端的次数
      >
      > Buffer pool hit rate，表示缓冲池的命中率，通常该值不应该小于95%。若小于95%，要观察是否是由于全表扫描引起的LRU列表被污染的问题

    - InnoDB存储引擎从1.0.x版本开始支持压缩页的功能，即将原本16KB的页压缩为1KB、2KB、4KB和8KB。对于非16KB的页，是通过unzip_LRU列表进行管理的，每个表的压缩比率可能各不相同，unzip_LRU从缓冲池中分配内存：

      > 在unzip_LRU列表中对不同压缩页大小的页进行分别管理
      >
      > 通过伙伴算法进行内存的分配。如对需要从缓冲池中申请页为4KB的大小:
      > 	1.检查4KB的unzip_LRU列表，检查是否有可用的空闲页
      >
      > ​	2.若有，则直接使用
      >
      > ​	3.否则，检查8KB的unzip_LRU列表
      >
      > ​	4.若能够得到空闲页，将页分成2个4KB页，存放到4KB的unzip_LRU列表
      >
      > ​	5.若不能得到空闲页，从LRU列表中申请一个16KB的页，将页分为1个8KB的页、2个4KB的页，分别存放到对应的 unzip_LRU列表中

    ---

    - 在LRU列表中的页被修改后，称为脏页( dirty page)，即缓冲池中的页和磁盘上的页的数据产生了不一致。这时数据库会通过CHECKPOINT机制将脏页刷新回磁盘，Flush列表中的页即为脏页列表
    - 脏页既存在于LRU列表中，也存在于Flush列表中。LRU列表用来管理缓冲池中页的可用性，Flush列表用来管理将页刷新回磁盘，二者互不影响
    - 同LRU列表一样，Flush列表也可以通过命令SHOW ENGINE INNODB STATUS来查看

- 重做日志缓冲(redo log buffer)

  - InnoDB存储引擎首先将重做日志信息先放入到这个缓冲区，按一定频率将其刷新到重做日志文件
  - 一般不需要设置得很大，一般每一秒钟会将重做日志缓冲刷新到日志文件，只需要保证每秒产生的事务量在这个缓冲大小之内即可。该值可由配置参数innodb_log_buffer_size控制，默认为8MB
  - 重做日志将重做日志缓冲中的内容刷新到外部磁盘的重做日志文件中的情况：
    - Master Thread每一秒将重做日志缓冲刷新到重做日志文件
    - 每个事务提交时会将重做日志缓冲刷新到重做日志文件
    - 当重做日志缓冲池剩余空间小于1/2时，重做日志缓冲刷新到重做日志文件

- 额外的内存池

  - 在InnoDB存储引擎中，对内存的管理通过内存堆（heap)方式进行
  - 在对一些数据结构本身的内存进行分配时，需要从额外的内存池中进行申请，当该区域的内存不够时，会从缓冲池中进行申请。如，分配了缓冲池（innodb_buffer_pool)，但是每个缓冲池中的帧缓冲(frame buffer）还有对应的缓冲控制对象( buffer control block)，这些对象记录了一些如LRU、锁、等待等信息，需要从额外内存池中申请

##### （2）Checkpoint（检查点）技术

- 倘若每次一个页发生变化，就将新页的版本刷新到磁盘，那么这个开销是非常大的。若热点数据集中在某几个页中，那么数据库的性能将变得非常差。同时，如果在从缓冲池将页的新版本刷新到磁盘时发生了宕机，那么数据就不能恢复了。为了避免发生数据丢失的问题，当前事务数据库系统普遍都采用了Write Ahead Log策略，即当事务提交时，先写重做日志，再修改页。当由于发生宕机而导致数据丢失时，通过重做日志来完成数据的恢复。这也是事务ACID中D (Durability持久性）的要求
- Checkpoint解决：
  - 缩短数据库的恢复时间
  - 缓冲池不够用时，将脏页刷新到磁盘
  - 重做日志不可用时，刷新脏页
- 当数据库发生宕机时，数据库不需要重做所有的日志，因为Checkpoint之前的页都已经刷新回磁盘。故数据库只需对Checkpoint后的重做日志进行恢复。这样就大大缩短了恢复的时间
- 当缓冲池不够用时，根据LRU算法会溢出最近最少使用的页，若此页为脏页，需要强制执行Checkpoint，将脏页也就是页的新版本刷回磁盘
- 重做日志出现不可用的情况是因为当前事务数据库系统对重做日志的设计都是循环使用的，重做日志可以被重用的部分是指这些重做日志已经不再需要，即当数据库发生宕机时，数据库恢复操作不需要这部分的重做日志，可以被覆盖重用。若此时重做日志还需要使用，必须强制产生Checkpoint，将缓冲池中的页至少刷新到当前重做日志的位置
- 对于InnoDB存储引擎，通过LSN (Log Sequence Number）来标记版本。LSN是8字节的数字，单位是字节。每个页有LSN，重做日志中也有LSN，Checkpoint也有LSN。通过命令SHOW ENGINE INNODB STATUS观察
- 分类
  - Sharp Checkpoint：数据库关闭时将所有的脏页都刷新回磁盘，默认，即参数innodb_fast_shutdown=1
  - Fuzzy Checkpoint：InnoDB存储引擎内部使用，即只刷新一部分脏页，而不是刷新所有的脏页回磁盘
    - Master Thread Checkpoint：差不多以每秒或每十秒的速度从缓冲池的脏页列表中刷新一定比例的页回磁盘。异步的，用户查询线程不会阻塞
    - FLUSH_LRU_LIST Checkpoint：InnoDB存储引擎需要保证LRU列表中需要有差不多100个空闲页可供使用。若没有InnoDB存储引擎会将LRU列表尾端的页移除。如果有脏页，进行Checkpoint，从MySQL 5.6版本开始，这个检查被放在了一个单独的Page Cleaner线程中进行，可以通过参数innodb_lru_scan_depth控制LRU列表中可用页的数量，默认为1024
    - Async/Sync Flush Checkpoint：指重做日志文件不可用情况，需要强制将一些页刷新回磁盘，此时脏页是从脏页列表中选取的，保证重做日志循环使用的可用性
    - Dirty Page too much Checkpoint：脏页的数量太多，导致InnoDB存储引擎强制进行Checkpoint，保证缓冲池中有足够可用的页。可由参数innodb_max_dirty_pages_pct控制:，默认75，表示当缓冲池中脏页的数量占据75%时，强制进行Checkpoint，刷新一部分的脏页到磁盘

##### （3）Master Thread 工作方式

**1.0版本前**

- 每秒一次的操作：
  - 日志缓冲刷新到磁盘，即使这个事务还没有提交（总是）
  - 合并插入缓冲，InnoDB存储引擎会判断当前一秒内发生的IO次数是否小于5次，小于就认为当前的IO压力很小，可以执行合并插入缓冲的操作（可能）
  - 至多刷新100个InnoDB的缓冲池中的脏页到磁盘，InnoDB存储引擎通过判断当前缓冲池中脏页的比例(buf_get_modified_ratio_pct）是否超过配置文件中innodb_max_dirty_pages_pct参数（默认90)，超过这个阈值认为需要做磁盘同步的操作，将100个脏页写入磁盘中（可能)
  - 如果当前没有用户活动，则切换到background loop （可能)
- 每十秒一次的操作：
  - 刷新100个脏页到磁盘，InnoDB存储引擎会先判断过去10秒之内磁盘的IO操作是否小于200次，如果是，认为当前有足够的磁盘IO操作能力，将100个脏页刷新到磁盘（可能)
  - 合并至多5个插入缓冲（总是)
  - 将日志缓冲刷新到磁盘（总是)
  - 删除无用的Undo页，即执行full purge操作，最多尝试回收20个undo页（总是)
  - 刷新100个或者10个脏页到磁盘，InnoDB存储引擎会判断缓冲池中脏页的比例(buf_get_modified_ratio_pct)，超过70%，则刷新100个脏页到磁盘，小于，则只需刷新10%的脏页到磁盘（总是)
- background loop：若当前没有用户活动（数据库空闲时）或者数据库关闭( shutdown)，就会切换到这个循环
  - 删除无用的Undo页〔总是)
  - 合并20个插入缓冲（总是)
  - 跳回到主循环（总是)
  - 不断刷新100个页直到符合条件（可能，跳转到flush loop中完成)
    - 若flush loop中也没有什么事情可以做了，InnoDB存储引擎会换到suspend_loop，将Master Thread挂起，等待事件的发生。若启用(enable）了InnoDB存储引擎，却没有使用任何InnoDB存储引擎的表，那么Master Thread总是处于挂起的状态

---

**1.0版本后**

- 参数innodb_io_capacity，表示磁盘IO的吞吐量，默认75
  - 在合并插入缓冲时，合并插入缓冲的数量为innodb_io_capacity值的5%
  - 在从缓冲区刷新脏页时，刷新脏页的数量为innodb_io_capacity
- 参数innodb_adaptive_flushing(自适应地刷新)：影响每秒刷新脏页的数量。原来的规则是：脏页在缓冲池所占的比例小于innodb_max_dirty_pages_pct时不刷新脏页；大于时刷新100个脏页。现在InnoDB存储引擎会通过buf_flush_get_desired_flush_rate函数判断需要刷新脏页最合适的数量。其通过判断产生重做日志（redo log）的速度来决定最合适的刷新脏页数量。当脏页的比例小于innodb_max_dirty_pagespct时也会刷新一定量的脏页
- 之前每full purge操作最多回收20个Undo页，现在引入参数innodb_purge_batch_size，可以控制每次 fullpurge回收的Undo页的数量。默认20

---

**1.2版本**

- 对于刷新脏页的操作，从Master Thread线程分离到一个单独的Page Cleaner Thread，减轻Master Thread工作，提高系统的并发性

##### （4）特性

- 插入缓冲( Insert Buffer)：和数据页一样，物理页的组成部分

  - 在InnoDB存储引擎中，主键是行唯一的标识符。通常应用程序中行记录的插入顺序是按照主键递增的顺序进行插入的。插入聚集索引(Primary Key)一般是顺序的，不需要磁盘的随机读取，这类插入很快
  - 插入操作时，数据页的存放按主键顺序存放，对于非聚集索引叶子节点的插入不再是顺序的了，需要离散地访问非聚集索引页，由于随机读取导致了插入操作性能下降。因为B+树的特性决定了非聚集索引插入的离散性。某些情况下，辅助索引的插入依然是顺序的，如用户购买表中的时间字段
  - Insert Buffer：对于非聚集索引的插入或更新操作，不是每一次直接插入到索引页中，而是先判断插入的非聚集索引页是否在缓冲池中，若在则直接插人；若不在则先放入到一个Insert Buffer对象中，然后再以一定的频率和情况进行Insert Buffer 和辅助索引页子节点的merge（合并）操作，通常能将多个插入合并到一个操作中（因为在一个索引页中)，大大提高了对于非聚集索引插入的性能
  - 条件：
    - 索引是辅助索引(secondary index)
    - 索引不是唯一 ( unique）的
  - 若MySQL数据库发生宕机，有的Insert Buffer并没有合并到实际的非聚集索引中去。恢复可能需要很长的时间，辅助索引不能是唯一的，因为在插入缓冲时，数据库并不去查找索引页来判断插入的记录的唯一性
  - 可以通过命令SHOW ENGINE INNODB STATUS来查看插入缓冲的信息
  - 修改IBUF_POOL_SIZE_PER_MAX_SIZE可以对插入缓冲的大小进行控制。如改为3，则最大只能使用1/3的缓冲池内存

  ---

  - 1.0引入Change Buffer，Insert Buffer的升级，InnoDB存储引擎可以对DML操作 -- INSERT、DELETE、UPDATE都进行缓冲，分别是:Insert Buffer、Delete Buffer、Purge buffer
  - 适用对象是非唯一的辅助索引。对一条记录进行UPDATE操作可能分为：
    - 将记录标记为已删除
    - 真正将记录删除
  - Delete Buffer对应UPDATE操作第一个过程，即将记录标记为删除。Purge Buffer对应UPDATE操作的第二个过程，即将记录真正的删除
  - 参数innodb_change_buffering：开启各种 Buffer 的选项。可选值：inserts、deletes、purges、changes、all、none
    - inserts、deletes、purges是前面讨论三种情况
    - changes表示启用inserts 和 deletes
    - all 表示启用所有。默认
    - none表示都不启用
  - InnoDB 1.2.x版本开始，通过参数innodb_change_buffer_max_size控制Change Buffer最大使用内存的数量，默认25，即最多使用1/4的缓冲池内存空间，最大可设50

  ---

  - Insert Buffer的数据结构是一棵B+树。在MySQL4.1之前的版本中每张表有一棵 Insert Buffer B+树。现在全局只有一棵 Insert Buffer B+树，负责对所有的表的辅助索引进行Insert Buffer。存放在共享表空间中，默认ibdatal中

  - 试图通过独立表空间ibd文件恢复表中数据时，往往导致CHECK TABLE失败。因为表的辅助索引中的数据可能还在Insert Buffer，即共享表空间中，通过ibd文件进行恢复后，还需要进行REPAIR TABLE操作来重建表上所有的辅助索引

  - Insert Buffer是一棵B+树，非叶节点存放查询的search key（键值)

    - scarch key一共占用9个字节
    - space表示待插入记录所在表的表空间id，在InnoDB存储引擎中，每个表有一个唯一的 space id，通过space id查询得知是哪张表。占用4字节
    - marker占用1字节，用来兼容老版本的 Insert Buffcr
    - offset专示页所在的偏移量，占用4字节

    ![image-20210707162347361](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210707162347361.png)

  - 当一个辅助索引要插入到页( space，offset）时，如果这个页不在缓冲池中，InnoDB存储引擎首先根据上述规则构造一个search key，查询Insert Buffer这棵B+树，将这条记录插入到Insert Buffer B+树的叶子节点中，不是直接将待插入的记录插入

    - metadata占用4字节，存储表中数据

    ![image-20210707162615432](C:\Users\13085\AppData\Roaming\Typora\typora-user-images\image-20210707162615432.png)

    |                             名称                             | 字节 |
    | :----------------------------------------------------------: | :--: |
    | IBUF_REC_OFFSET_COUNT（排序每个记录进入Insert Buffer 的顺序） |  2   |
    |                     IBUF_REC_OFFSET_TYPE                     |  1   |
    |                    IBUF_REC_OFFSET_FLAGS                     |  1   |

  - 从Insert Buffer叶子节点的第5列开始，就是实际插入记录的各个字段了。较之原插入记录，Insert Buffer B+树的叶子节点记录需要额外13字节的开销。因为辅助索引页(space，page_no）中的记录可能被插入到Insert Buffer B+树中，为了保证每次Merge Insert Buffer页成功，还需要有一个特殊的页用来标记每个辅助索引页(space，page_no）的可用空间。其类型为Insert Buffer Bitmap

  P66

- 两次写( Double Write)

- 自适应哈希索引(Adaptive Hash Index）

- 异步IO(Async IO)

- 刷新邻接页(Flush Neighbor Page）