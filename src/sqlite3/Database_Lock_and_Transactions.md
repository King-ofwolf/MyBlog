# sqlite3的锁及事务类型

sqlite3总共有三种事务类型：
```SQL
BEGIN [DEFERRED / IMMEDIATE / EXCLUSIVE] TRANSCATION
```
五种锁，按锁的级别依次是：
```SQL
UNLOCKED /SHARED /RESERVERD /PENDING /EXCLUSIVE
```
当执行select即读操作时，需要获取到SHARED锁（共享锁），当执行insert/update/delete操作(即内存写操作时)，需要进一步获取到RESERVERD锁（保留锁），当进行commit操作(即磁盘写操作时)，需要进一步获取到EXCLUSIVE锁（排它锁）。

**对于RESERVERD锁**，sqlite3保证同一时间只有一个连接可以获取到保留锁，也就是同一时间只有一个连接可以写数据库(内存)，但是其它连接仍然可以获取SHARED锁，也就是其它连接仍然可以进行读操作（这里可以认为写操作只是对磁盘数据的一份内存拷贝进行修改，并不影响读操作）。

**对于EXCLUSIVE锁**，是比保留锁更为严格的一种锁，在需要把修改写入磁盘即commit时需要在保留锁/未决锁的基础上进一步获取到排他锁，顾名思义，排他锁排斥任何其它类型的锁，即使是SHARED锁也不行，所以，在一个连接进行commit时，其它连接是不能做任何操作的（包括读）。

**PENDING锁（即未决锁）**，则是比较特殊的一种锁，它可以允许已获取到SHARED锁的事务继续进行，但不允许其它连接再获取SHARED锁，当已存在的SHARED锁都被释放后（事务执行完成），持有未决锁的事务就可以获得commit的机会了。sqlite3使用这种锁来防止writer starvation（写饿死）。
 

## 死锁的情况

死锁的情况：当两个连接使用begin transaction开始事务时，第一个连接执行了一次select操作（已经获取到SHARED锁），第二个连接执行了一次insert操作（已经获取到了RESERVERD锁），此时第一个连接需要进行一次insert/update/delete（需要获取到RESERVERD锁），第二个连接则希望执行commit（需要获取到EXCLUSIVE锁），由于第二个连接已经获取到了RESERVERD锁，根据RESERVERD锁同一时间只有一个连接可以获取的特性，第一个连接获取RESERVERD锁的操作必定失败，而由于第一个连接已经获取到SHARED锁，第二个连接希望进一步获取到EXCLUSIVE锁的操作也必定失败。就导致了事务死锁。

## 事务类型的使用原则

在用”begin transaction”显式开启一个事务时，默认的事务类型为DEFERRED，锁的状态为UNLOCKED，即不获取任何锁，如果在使用的数据库没有其它的连接，用begin就可以了。如果有多个连接都需要对数据库进行写操作，那就得使用BEGIN IMMEDIATE/EXCLUSIVE开始事务了。
使用事务的好处是：
1. 一个事务的所有操作相当于一次原子操作，如果其中某一步失败，可以通过回滚来撤销之前所有的操作，只有当所有操作都成功时，才进行commit，保证了操作的原子特性；
2. 对于多次的数据库操作，如果我们希望提高数据查询或更新的速度，可以在开始操作前显式开启一个事务，在执行完所有操作后，再通过一次commit来提交所有的修改或结束事务。

## 对SQLITE_BUSY的处理

当有多个连接同时对数据库进行写操作时，根据事务类型的使用原则，我们在每个连接中用BEGIN IMMEDIATE开始事务，即多个连接都尝试取得保留锁的情况，根据保留锁同一时间只有一个连接可以获取到的特性，其它连接都将获取失败，即事务开始失败，这种情况下，sqlite3将返回一个SQLITE_BUSY的错误，如果我们不希望操作就此失败而返回，就必须处理SQLITE_BUSY的情况，sqlite3提供了sqlite3_busy_handler或sqlite3_busy_timeout来处理SQLITE_BUSY，对于sqlite3_busy_handler，我们可以指定一个busy_handler来处理，并可以指定失败重试的次数。而sqlite3_busy_timeout则是由sqlite3自动进行sleep并重试，当sleep的累积时间超过指定的超时时间时，最终返回SQLITE_BUSY。需要注意的是，这两个函数同时只能使用一个，后面的调用会覆盖掉前次调用。从使用上来说，sqlite3_busy_timeout更易用一些，只需要指定一个总的超时时间，然后sqlite自己会决定多久进行重试以及重试的次数，直到达到总的超时时间最终返回SQLITE_BUSY。并且，这两个函数一经调用，对其后的所有数据库操作都有效，非常方便。

参考：

<http://www.sqlite.org/lockingv3.html>
<http://www.cnblogs.com/hustcat/archive/2009/03/01/1400757.html>