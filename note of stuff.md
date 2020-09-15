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
