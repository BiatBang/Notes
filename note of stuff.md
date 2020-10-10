# Note of all

## SkipList
A linked list with levels.
- Find: find from max level. When find the right slot, find in next level, ... until find the value.
- Add: Decide how many levels needed for this value (if not existed). Flip a coin until tail, each head plus one level. Then add the value from max level to 1 level. 

## Huffman Code
Huffman Tree solves the problem of least Weighted Path Length (WPL). The larger its weight is, the closer the node to the root. 
- Procedure: use a priorityqueue (or just sort the array with weights). Poll the min 2 and form a tree, with the sum as the root. Add the root into the queue, and repeat until the queue is empty.

## B-Tree and B+Tree
B-Tree is a tree with multiple children in one node, and the number of children cannot exceed order M. B-Tree is a self-balancing tree.
B+Tree is an advanced B-Tree. B+Tree saves space by not recording data in non-leaf nodes. 

## JVM
### Class Load
When loading a field from a class, 
1. find local fields
2. if not, find one interface, and its ancestors, and another interface, and its ancestors...
3. if not, find the parent class, and its parent class...
4. if not, no, throws

## Java
- transient: a key word for serialization. A variable defined as transient will be cleared to default value when serialized. It's useful for security.

## Concurrency
### Sleep, Wait and Yield
- Sleep: Thread based function. Let the thread sleep without releasing the lock. Immediately turns Runnable after waking up.
- Wait: Object-level method. Release the lock and wait until notified. Wait and notify are often used in synchronized block.
- Yield: pause the current thread and let one thread with the same priority to run; if not, continue itself. Give a chance to other threads to do some simple tasks. Until the other thread finished, continue this thread.
- Join: join(long millis) waits for the current thread to end and start other thread. Join can make sure next thread only starts after current thread finished its execution.
### Collections
- CopyOnWrite: When multiple thread working with one collection, there may be writing issues. One solution is to copy when writing. Instead of directly writing to the collection, make a new one with len+1 length, then add to the tail and set it the latest collection. Then unlock the lock. 
- Locks:
  1. ReentrantReadWriteLock: lock on write and share with read.
  2. SpinLock: use a loop to wait for the lock
  3. ReentrantLock: inner methods (including recursively invoked methods) can use the lock.
- BlockQueue: 
  block the queue if offer when full / poll when empty.
  |        | normal  | block  | throws   |
  |--------|---------|--------|----------|
  | add    | offer(): boolean | put(): void  | add(): boolean    |
  | remove | poll()  | take() | remove(): boolean |

  use a Condition.await() to wait for signal (Condition.signalAll()).
  Either use a normal or a block way, the queue will be locked.
- Synchronized v. lock
  - sync is a key word, lock is an object
  - sync will release the lock automatically, after the block executed. Lock needs to be release manually to avoid death lock.
- ThreadPool
  - Execution order: corePool -> workQueue -> threadPool -> throws
  - Common types:
    1. Executors.newFixedThreadPool(int): Using LinkedBlockingQueue -> long-term tasks
    2. Executors.newSingleThreadExecutor(): Using LinkedBlockingQueue -> one task at one time
    3. Executors.newCachedThreadPool(): Using SynchronousQueue -> short-term, async, low-loaded apps
  - Number of threads in a pool:
    - For CPU high-loaded: Means CPUs need to work without block, so the number needs to be low, n(CPU) + 1.
    - For IP high-loaded: means lots of IO operations, lots of blocks. Wait time can be long, so multi-thread can relieve the waiting time, increase the performance. n(CPU) * 2, or, n(CPN) / (1 - block coe (0.8~0.9))
  - Mutual Exclusion （互斥）
  - Hold and Wait （占有并等待）
  - No Preemption （不可抢占）
  - Circular Wait （循环等待）

## Spring
### Features
- Decouple
- AOP
- Transactions
- Testing
- Integrate with other frameworks (MyBatis, Hibernate...)
- X Reflection affects the performance
- X Hard to learn  

### Components
- Spring Core: Base part, including IOC, DI
- Spring Beans: BeanFactory to manage all the beans
- Spring Context: A container, containing components of the app.
- Spring JDBC: A JDBC layer supported by Spring
- Spring AOP:
  - Spring uses **Advice** to cut an execution and inject some actions.
  - Spring supports advices "inserted" into 
    - before
    - after returning
    - after throwing
    - after
    - around
    ```
    @Before("execution(* EmployeeManager.getEmployeeById(..))") //point-cut expression
    public void logBeforeV1(JoinPoint joinPoint)
    {
        System.out.println("EmployeeCRUDAspect.logBeforeV1() : " + joinPoint.getSignature().getName());
    }
    ```
- Spring Web
- Spring Test
### Design Patterns Used
- Factory: BeanFactory
- Singleton: Each bean is a singleton
- Proxy: AOP uses proxy
- Template: templates to avoid duplicated codes, like RestTemplate, JmsTemplate ...
- Observer (Subscriber): when an object changes its status, all objects depends on it will be notified
### Spring IOC
IOC means pass the control of dependencies to a specified container, and let it handle all the dependencies. Then, the object doesn't need to know how to initiate dependencies. 
- Implementation: Factory + Reflection. BeanFactory receives the name of bean, then 

## SQL
- Delete is a DML(data manipulation language) command, it can be rollback. Truncate is a DDL(data definition language) command, it cannot be rollback, but it's fast
- clustered index v. non-clustered index
  - clustered: stored inside the table, maintain the order of the index, like insert 5 3 4, then the index will be 3 4 5. One clustered index at most
  - non clustered: outside the table, order can be maintained if specified. Slower than clustered, but can have multiple nc indexes
- Denormalization: A optimization happens after normalization. Normalization removes all the redundancy but makes queries sometimes slow because of joins. So, if some columns are frequently queried, it can be added to the related table, as a redundancy but quick way to query. 
- ACID -- for transactions
  - Atomicity: Either fails or succeed all
  - Consistency: From one valid state to another valid state. Ensures the transaction obeys the invariants
  - Isolation: Keeps the database operating correct when facing concurrency, just like if the executions are sequentially. 
  - Durability: If one transaction is committed, it'll be permanently recorded into the database.
- Natural Join and Cross Join:
  - natural join selects common rows
  - cross join makes a m*n join, joining everything with everything in the other table
- Having & Where
  - where is used in normal cases
  - having is used in group by, add conditions to groups you want
- Fetch the odd ids: select studentId from student where mod(studentId, 2)=1;
- stuff function: select stuff('learn sql', 7, 3, 'html'), delete the substring from 7 to 3 length, then insert the 4th param
- replace function: select replace('source', 'old', 'new')
- 多事务的并发进行一般会造成以下几个问题:
  - 脏读: A事务读取到了B事务未提交的内容,而B事务后面进行了回滚.
  - 不可重复读: 当设置A事务只能读取B事务已经提交的部分,会造成在A事务内的两次查询,结果竟然不一样,因为在此期间B事务进行了提交操作.
  - 幻读: A事务读取了一个范围的内容,而同时B事务在此期间插入了一条数据.造成"幻觉".
- MySQL的四种隔离级别如下:
  - 未提交读(READ UNCOMMITTED)
    这就是上面所说的例外情况了,这个隔离级别下,其他事务可以看到本事务没有提交的部分修改.因此会造成脏读的问题(读取到了其他事务未提交的部分,而之后该事务进行了回滚).
    这个级别的性能没有足够大的优势,但是又有很多的问题,因此很少使用.
  - 已提交读(READ COMMITTED)
    其他事务只能读取到本事务已经提交的部分.这个隔离级别有 不可重复读的问题,在同一个事务内的两次读取,拿到的结果竟然不一样,因为另外一个事务对数据进行了修改.
  - REPEATABLE READ(可重复读)
  可重复读隔离级别解决了上面不可重复读的问题(看名字也知道),但是仍然有一个新问题,就是 幻读,当你读取id> 10 的数据行时,对涉及到的所有行加上了读锁,此时例外一个事务新插入了一条id=11的数据,因为是新插入的,所以不会触发上面的锁的排斥,那么进行本事务进行下一次的查询时会发现有一条id=11的数据,而上次的查询操作并没有获取到,再进行插入就会有主键冲突的问题.
  - SERIALIZABLE(可串行化)
  这是最高的隔离级别,可以解决上面提到的所有问题,因为他强制将所以的操作串行执行,这会导致并发性能极速下降,因此也不是很常用.

## Consistent Hashing - Ring
About sharding, we want two things:
1. When adding a new node, only old nodes pass data to the new node, no old nodes pass data to old nodes, averagely
2. When deleting a node, pass the data averagely to other nodes
### procedure
1. name a circle with values, from 0 to a number
2. hash the servers (nodes) into the circle
3. to make it more evenly, split the servers into multiple values, and put them into circles
4. when get / put a value, calculate the hash, use binary search or some other quick search to find the least node that larger than the hash(node or a subnode), and it's it
- When deleting a node
  - scan all the subnodes, moving the values between it and its former subnodes to the next subnode
  A0 - C2 - Jack - B1
  if we want to remove server C, C2 needs to be removed. Then, Jack (between C2 and B1) will be assigned to A0
- When adding a node
  - find its hash (hashes if subnodes), insert them and assign the values between the new node and the node before it to it
  A0 - Jack - B1
  A0 - (C2) - Jack - B1
  then values between C2 and B1 Jack will be assigned to C2

## SQL V. NOSQL
- for simple queries, sql and nosql behaves quick as the same. If there is no much complex queries, both work well
- SQL: 
  - doesn't support horizontal scale well. It'll cause large impact if sharding the database into small machines, the indexes hurt.
  - not fault tolerant. Replicate of sql machines cost a lot. NoSQL management like cassandra works way better than this.

##