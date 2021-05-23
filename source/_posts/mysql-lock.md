---
title: Mysql的锁
date: 2021-05-23 14:28:06
tags:
 - Mysql
 - Lock
categories:
 - Mysql
---

我们都知道事务的ACID性质（参考[Mysql系列 - 事务](https://runnerliu.github.io/2017/08/28/mysqltransaction/)），数据库为了维护这些性质，尤其是一致性(C)和隔离性(I)，一般使用加锁这种方式，同时数据库又是个高并发的应用，同一时间会有大量的并发访问，如果加锁过度，会极大的降低并发处理能力。所以对于加锁的处理，可以说就是数据库对于事务处理的精髓所在。

#### 一次封锁or两段锁

因为有大量的并发访问，为了预防死锁，一般应用中推荐使用一次封锁法，就是在方法的开始阶段，已经预先知道会用到哪些数据，然后全部锁住，在方法运行之后，再全部解锁。这种方式可以有效的避免循环死锁，但在数据库中却不适用，因为在事务开始阶段，数据库并不知道会用到哪些数据。

数据库遵循的是两段锁协议，将事务分成两个阶段，加锁阶段和解锁阶段（所以叫两段锁）：

- 加锁阶段：在该阶段可以进行加锁操作。在对任何数据进行读操作之前要申请并获得S锁（共享锁，其它事务可以继续加共享锁，但不能加排它锁），在进行写操作之前要申请并获得X锁（排它锁，其它事务不能再获得任何锁）。加锁不成功，则事务进入等待状态，直到加锁成功才继续执行。
- 解锁阶段：当事务释放了一个封锁以后，事务进入解锁阶段，在该阶段只能进行解锁操作不能再进行加锁操作。

两段锁虽然无法避免死锁，但是可以保证事务的并发调度是串行化（串行化很重要，尤其是在数据恢复和备份的时候）的。

#### 锁类型

从锁定资源的角度来看，MySQL 中的锁分为：

- 表级锁：对整张表加锁。开销小，加锁快，不会出现死锁，锁定粒度大，发生锁冲突的概率最高，并发度最低。适合以查询为主。
- 行级锁：对行记录加锁。开销大，加锁慢，会出现死锁，锁定粒度最小，发生锁冲突的概率最低，并发度也最高。适合于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用。
- 页面锁：开销和加锁时间界于表锁和行锁之间，会出现死锁，锁定粒度界于表锁和行锁之间，并发度一般

##### 行级锁

InnoDB加行锁时通过给索引上的索引项加锁来实现，这种行锁实现特点意味着，只有通过索引条件检索条件数据，InnoDB才使用行锁，否则InnoDB将使用表锁。

行锁的加锁算法：

- 记录锁 Record Locks：对单个记录的索引项加锁，即使表没有建立索引，InnoDB也会创建一个隐藏的聚簇索引(隐藏的递增主键索引)，并使用此索引进行记录锁定。
- 间隙锁 Gap Locks：锁定一个范围，但不包含记录本身，间隙锁作用在索引记录之间的间隔，又或者作用在第一个索引之前，最后一个索引之后的间隙。
- Next-key锁：Gap Lock+Record Lock，锁定一个范围，并且锁定记录本身。当查询的索引含有唯一属性时，InnoDB存储引擎会对Next-Key Lock 进行优化，将其降级为 Record Lock，即仅锁住索引本身，而不是范围。

##### 意向共享锁

一个事务给一个数据行加共享锁时，必须先获得表的意向共享锁

##### 共享锁

共享锁又叫做读锁。 当用户要进行数据的读取时，对数据加上共享锁，共享锁可以同时加上多个。

加了共享锁后，该事务只能对数据进行读取而不能修改，并且其它事务只能加共享锁，不能加排他锁。

加锁方式：在执行语句后面加上 `lock in share mode` 就代表对某些资源加上共享锁了。

##### 意向排它锁

一个事务给一个数据行加排他锁时，必须先获得表的意向排他锁

##### 排他锁

排他锁又叫做写锁。 当用户要进行数据的写入时，对数据加上排他锁。排他锁只可以加一个，和其他的排他锁、共享锁都相斥。

事务对数据加上排他锁时，只允许此事务读取和修改此数据，并且其它事务不能对该数据加任何锁。

##### 悲观锁

悲观锁的实现，往往依靠数据库提供的锁机制（也只有数据库层提供的锁机制才能真正保证数据访问的排他性，否则，即使在本系统中实现了加锁机制，也无法保证外部系统不会修改数据）。

悲观锁认为数据随时会被修改，因此每次读取数据之前都会上锁，防止其它事务读取或修改数据（共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程），应用于数据更新比较频繁的场景。

悲观锁的实现方式：

- 实现悲观锁时，我们必须先使用`set autocommit=0; `，关闭mysql的AutoCommit属性。因为我们查询出数据之后就要将该数据锁定，关闭自动提交后，我们需要手动开启事务。

悲观锁的优点：

- 悲观锁保证了数据处理时的安全性。

悲观锁的缺点：

- 加锁造成了开销增加，并且增加了死锁的机会。降低了并发性。

##### 乐观锁

乐观锁不是数据库自带的，需要我们自己去实现。乐观锁是指操作数据库时（更新操作），想法很乐观，认为这次的操作不会导致冲突，在操作数据时，并不进行任何其他的特殊处理（也就是不加锁），而在进行更新后，再去判断是否有冲突，适用于读多写少的场景。

乐观锁的实现方式：

- 版本号机制：加一个版本号或者时间戳字段，每次数据更新时同时更新这个字段。一般是在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加一。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若刚才读取到的version值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。
- CAS算法：Compare And Swap（比较与交换），是一种有名的无锁算法。先读取想要更新的字段或者所有字段，更新的时候比较一下，只有字段没有变化才进行更新。一般情况下是一个自旋操作，即不断的重试。

乐观锁的优点：

- 乐观锁机制避免了长事务中的数据库加锁开销，大大提升了大并发量下的系统整体性能表现。

乐观锁的缺点：

- ABA问题：如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个问题被称为CAS操作的 "ABA"问题。
- 循环时间长开销大：自旋CAS（也就是不成功就一直循环执行直到成功）如果长时间不成功，会给CPU带来非常大的执行开销。

#### 死锁

死锁是指两个或两个以上事务在执行过程中因争抢锁资源而造成的互相等待（形成环路）的现象。**表级锁不会产生死锁**，所以解决死锁主要还是针对于最常用的InnoDB。

死锁的解决：

- 查出阻塞的进程，将其kill掉，将资源释放

- 设置锁的超时时间：InnoDB 行锁的等待时间，单位秒。可在会话级别设置，RDS 实例该参数的默认值为 50s。生产环境不推荐使用过大的 `innodb_lock_wait_timeout` 参数值该参数支持在会话级别修改，方便应用在会话级别单独设置某些特殊操作的行锁等待超时时

- 如果不同程序会并发存取多个表，尽量约定以相同的顺序访问表，可以大大降低死锁机会

- 在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁产生概率

- 对于非常容易产生死锁的业务部分，可以尝试使用升级锁定颗粒度，通过表级锁定来减少死锁产生的概率，**不推荐**



Read More:









Read More:

> [Innodb中的事务隔离级别和锁的关系](https://tech.meituan.com/2014/08/20/innodb-lock.html)
>
> [深入理解MySQL数据库各种锁（总结）](https://www.jianshu.com/p/4f2311f38040)
>
> [面试官：小伙子，给我说一下mysql 乐观锁和悲观锁吧](https://segmentfault.com/a/1190000022839728)
>
> [MySQL锁总结](https://zhuanlan.zhihu.com/p/29150809)
>
> [一张图彻底搞懂 MySQL 的锁机制](https://learnku.com/articles/39212?order_by=vote_count&)