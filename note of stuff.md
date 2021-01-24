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
- wildcard: Collection<?> -> collection of unknown. only used in <>.
  - <?>
  - <? extends parent>
  - <? super child>

## Concurrency
### Sleep, Wait and Yield
- Sleep: Thread based function. Let the thread sleep without releasing the lock. Immediately turns Runnable after waking up.
- Thread.sleep(0): thread sleep(0)? why?
  Each time a thread sleeps, it's declaring it won't use the CPU for some time. In windows, when current thread sleeps, all threads will have different priorities, and the one with the most priority (maybe itself) will occupy the CPU. sleep(time) won't guarantee the thread will gain access right after *time*, but it's guaranteed it won't use the CPU in this period. So, Thread.sleep(0) will let CPU calculates all threads priorities, and decide the one deserves the CPU. It may be itself, or others.
- Wait: Object-level method. Release the lock and wait until notified. Wait and notify are often used in synchronized block.
- Yield: pause the current thread and let one thread with the same priority to run; if not, continue itself. Give a chance to other threads to do some simple tasks. Until the other thread finished, continue this thread.
- Join: join(long millis) waits for the current thread to end and start other thread. Join can make sure next thread only starts after current thread finished its execution. Who calls it, who waits, (until millis)
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
### Semaphore
- Semaphore(int permits, *boolean fair)
  permits declare the initial available permits, can be negative(release first)
- acquire()
  to use the resource, use acquire. if full, block, until one thread release one permit. when acquire() returns, permit -1.
- release()
  after using the resource, release it, then permit +1.
### Synchronized
- synchronized provides a (re-entrant lock) locking on the resource, prevents reordering of the code, acquires and releases a lock before and after the synchronized block. So, read is locked too.
- synchronized a static method, will make the whole class locked; synchronized on a non-static method, will make the current object locked
- synchronized limits only in one JVM
- level:
  - Object level: synchronized on a non-static object or a non-static method, then the current object will be locked. When sync a method, only the method will be locked, not the whole object.
  - Class level: synchronized on a static method or a class, this means all object from this class will be blocked if one is being synced. Singleton can use this to prevent multiple threads create an instance simultaneously.
- Careful when synchronized on a non-final object. When the reference of the object is changed, the synchronized block may be open.
### Volatile
A machine has one main memory, multiple CPUs. Each thread may copy variables from main memory to a CPU cache to relieve the main memory. Therefore, some changes to a variable may be stored in cache rather than directly to main memory. 
Volatile guarantees the change of the variable will be directly written to main memory. Reading of volatile variables will be from main memory.
- Full volatile visibility
  - when writing a volatile variable, all variable in a thread before a volatile variable are written to main memory
  - when reading a volatile variable, 
- Happen-before guarantee
  - when there is a writing operation to a volatile variable, all read or write of other variables that before this write, will happen-before this write. 
    only operations before it won't be after volatile write, operations after it may still happen-before this write.
  - when there is a reading operation to a volatile variable, all read or write of other variables that are after the read will not happen-before this reading operation
- Problems
  - Volatile only guarantees the operations are directly to and from main memory, but not concurrency guarantees. 
  - If one volatile variable change is based on the latest value, and two threads both do this operation, there will be problem --- they both read the same value
  - If the change doesn't depend on the latest value, it's alright
  - If there is only one thread writing a volatile variable, and other threads just read, volatile guarantees every reading thread can read the latest value of this variable
- Atomic utils are using volatile

## java.util.concurrent
### Hashtable v ConcurrentHashMap
- both are thread-safe
- Hashtable synchronized every method, making both reading and writing blocked
- ConcurrentHashMap makes attributes volatile, and only synchronize the writing operations. Reading is not blocked since volatile. So concurrentHashMap has better performance
### Future
For an async operation, the process is unsure. Future interface can represent a future result of an async task.
- get([time], [timeUnit]) get the final result, block if not finished
- boolean cancel(boolean mayInterrupt) cancel the task, if may be interrupted, return the result of cancellation
- boolean isCancelled(): returns if the task is cancelled
- boolean isDone(): return if the task is done. Cancelled is done too 
### FutureTask
FutureTask is an implementation of Future. Input a callable / (runnable, result). Constructing a FutureTask won't start it immediately. Call the `run()` to let it run. Other things work the same as Future, `cancel`, `done`...
FutureTask is using a thread to handle the task by default
### LinkedBlockingDeque
A deque allows blocking when full / null
- boolean offerFirst / offerLast: offer to head or tail, throws exception when full
- boolean offer (E e, long timeout, TimeUnit unit): offer when not timeout
  ```
  public boolean offerFirst(E e, long timeout, TimeUnit unit) throws InterruptedException {
      if (e == null) {
          throw new NullPointerException();
      } else {
          LinkedBlockingDeque.Node<E> node = new LinkedBlockingDeque.Node(e);
          // use a Condition.awaitNanos to countdown
          long nanos = unit.toNanos(timeout);
          ReentrantLock lock = this.lock;
          lock.lockInterruptibly();

          try {
              boolean var9;
              while(!this.linkFirst(node)) {
                  if (nanos <= 0L) {
                      var9 = false;
                      return var9;
                  }
                  // usage of awaitNanos
                  nanos = this.notFull.awaitNanos(nanos);
              }

              var9 = true;
              return var9;
          } finally {
              lock.unlock();
          }
      }
    }
  ```
- E pollFirst / pollLast: poll from head or tail, throws when null
- void addFirst / addLast: same as offer
- putFirst / putLast(put): offer to head or tail, block when full
### ConcurrentSkipListMap
SkipList is not just alternative of AVL tree, it's also prone to thread safe. A change to AVL tree will change the structure of the tree, we need to left rotate and right rotate. SkipList is resistent to change. A change will make a node point to another node (or null), but it's still the list, and no change to the whole structure. Look into source code of concurrentskiplistmap, no lock, no volatile, no synchronized, elegant.
So, iterators and spliterators are weakly consistent.
- Structure:
  ```
  Index(1) ------------next-----------> Index(3)
    |                                     |
  down                                  down
    v                                     v
  Index(1) --next--> Index(2) --next--> Index(3)
  ```
  The order of the nodes depends on keys.
  Since the keys are ordered, functions normal maps have will have a little difference
- Set entrySet(): return the entrySet. Use the iterator to get increasing order keys
- Entry firstEntry(): return the entry with the least value key
- K firstKey(): ... key ...
- K lastKey(): ... key ... largest value
- K ceilingKey(K key): the smallest key larger than the given key
- K floorKey(K key): the largest key less then the given key (seems a little weird?)
- Entry pollFirst(Last)Entry: poll as in queue
- CSLM subMap(from, to): get a submap from *from* to *to*
### DelayedQueue
DelayedQueue\<E extends Delayed> is a queue storing elements with a delay time. It has a priorityQueue inside. If the element to poll hasn't expired, return null or wait. Each method (offer*, poll*, size, ...) has a reentrantlock to lock the queue.
- boolean offer(E e, long timeout, TimeUnit unit):
  ```
  public boolean offer(E e, long timeout, TimeUnit unit) {
        return this.offer(e);
    }
  ```
  (⊙ˍ⊙)
- E take(): wait forever until there is an available expired element
- E poll(long timeout, TimeUnit unit): wait until timeout or an available expired element
- int drainTo(Collection<? extends E> c, [int maxElements]): drain all the expired elements from *this* to c, expired only 


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
- Denormalization: An optimization happens after normalization. Normalization removes all the redundancy but makes queries sometimes slow because of joins. So, if some columns are frequently queried, it can be added to the related table, as a redundancy but quick way to query. 
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

## ZooKeeper
A distributed coordination service for distributed applications. Servers inside a zookeeper "ensemble" follow the leader-follower mode. Zookeeper helps distributed application sync.
### Services
- Naming service: identify nodes inside a cluster, working like DNS
- Configuration Management: sync the new coming node with the latest configuration
- Cluster Management: manage the joining / leaving of nodes, in real time
- Leader Election: elaborate later
- Locking and Sync Service: help sync cluster without racing issues
- Reliability: handle the case when some servers are down
### Client-Server Architecture
- Client: machines managed and interacted with zookeeper machines (servers). Periodically send a message (heartbeats) to a server to let it (then them) know it's alive. The server will send back an ack, if not, the client will choose another server to contact
- Server: nodes in Zookeeper. A member of a Zookeeper Ensemble. Contact the clients to know who's alive and who's down
- Ensemble: a batch of servers, with an at least 3 machines
- Leader: recover any node when it's down
- Follower: follow leader's instruction
### Types of Nodes
- Persistence znode: only be removed when a delete call comes. Even if the client who creates it disconnect. Default type.
- Ephemeral znode: only alive when the client who creates it is alive. Ephemeral znodes can't have children
- Sequential znode: either persistence or ephemeral. If a node is identified sequential, zk will maintains a sequence of it with a 10 digit number. ZK maintains a sequence of sequential znodes.
### Workflow
A client can connect to any node in an ensemble, either to a leader or a follower. When a client connects to node, the node will send back an ack. If the client doesn't receive it, retry other nodes. Then, the client starts sending heartbeats.
When a client sends data to the ensemble, the node that connects it will pass the data to leader (unless it's the leader), then the leader will send the data to all followers. If a majority of nodes ("Quorum") response it, the request is successful and the node will return the msg to the client.
It's good to have odd number of nodes inside an ensemble (3, 5, 7...)
### Leader Election
Not a complex way. One node issues an election, then it creates a sequential & ephemeral child "/election". Then, each node will create a sequential childe with a sequence number "/election/guid_n**********". After all nodes get a sequence number, the one with the smallest number becomes the leader. 
Each node should monitor the node with the node with the largest number less than its. If the node being watched by it is down, it will check if its the smallest. If so, it becomes the leader; if not, it find the smallest and let it be the leader. This ensure that there is always one leader. 

## Java Escape Analysis
A mechanism JVM provides, to know what objects are only used in its thread and what are not. It is a waste of memory if an object is only used its caller thread and it uses a lock. If an object is identified as only in one thread, it will be put into stack instead of heap, and its life will be ended when the function ends. That's why last person asked you: are all the objects stored in heap? How to specify them?
Scalar replacement: If an object is used pretty frequently but only some scalars are needed, like new Object().x, JVM will stop creating new objects, and only work with the scalar x.

## Git
### Three Trees
1. working directory (editor) 
2. staging index (after add)
3. commit history (after commit)
**git stash** is basically a kind of commit, may including multiple commits (staged, untracked, ignored). 
`git commit --amend` can add changes into last commit
`git stash [push -m message]` only stores staged changes. Unstaged or ignored files won't be stashed
`git stash apply` restore the staged changes
`git stash list` see the stash history
`git stash pop 1` apply the stash@{1} and delete it
`git stash -a | --all` will include ignored files
`git stash --include-untracked` will include untracked files (in working directory)
`stash` can stash only part of the file
`git stash branch` stash the changes into a new branch
`git stash drop stash@{1}` or `git stash clear` to delete the stashes

`git fetch` sees what others do without merging into local project. 
Git stores local branches in ./.git/refs/heads/ and stores ./.git/refs/remotes/
fetch will store the files from remote in /remotes/
`git branch` shows local branches and
`git branch -r` shows remote branches
`git fetch <remote> <branch>`

Git push
`git push <remote> <branch> [--force]` use force option carefully
`git push <remote> --all`

Git pull
git pull combines git fetch and git merge

Git merge
`git merge --squash <branch>` merge with another branch, and squeeze all commits of that branch into one last commit. After this, commit and add some message.

Git rebase
Base, in a feature branch, means the commit it's separated from the master. Rebase, means updating the base. Master branch evolves while feature branch evolves. When the feature wants to see if it conflicts the latest commit, rebase can change the base to the latest commit of the master. 