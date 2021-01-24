# Optimizing Java

## Chap 2. Overview of the JVM
### Interpreting and Classloading
JVM is a **stack** based interpreted machine. Always start from the top of a stack. Classloader is also an object.
JVM *first* models **Bootstrap classloaders** as the initial classes. It loads some basic classes like java.lang.
Second is **extension classloader**. Its parent is Bootstrap classloader. It supplies overrides and native code for some operating systems.
Third is **Application classloader**. It loads user classes from paths. Its parent is Extension classloader.
### Just-In-Time Compilation
Compare to Ahead-Of-Time compilation, JIT compiles only the classes it needs. It monitors the programatic trace info of the current execution, and when it passes the threshold, it compile and optimize other part of code. This is good for optimisations. JIT is compiling from interpreted bytecode.
### Threading and JMM(Java Memory Model)
All threads in one Java app share a single GC heap. Objects are sharable among threads with reference

## Hardware
### Memory Caches
Main memory calculates slower then core processor. CPU can calculate very fast, but need data input. CPU uses some memory area, faster than main memory and slower than CPU registers, as cache, copying some often-accessed memory locations. Memory caches on CPU have layers(L1, L2...). 
Status of a thread: 
- Ready to run
- Running (when scheduler calls)
- Sleeping
- Waiting
- Blocked
- Dead
There is cost to switch from User state to Kernel state. Linux provides mechanism called vDSO(virtual dynamically shared object), a memory area to determine if it's needed to switch to kernel state. To optimize a Java app, we aim to use 100% of the CPU.

## Performance Tests

## Understanding of Garbage Collection
### Jargons: 
- Stop-the-world (STW): all app threads pause when garbage is collected
- Concurrent: If multi threads, GC threads can run while others are running. CMS (Concurrent Mark and Sweep).
- Parallel: Multiple threads doing GC.
- Exact: An exact GC knows enough info about the state of heap, and all garbage can be collected in a single cycle. 
- Conservative: Lacks the info of an exact GC. Ignorant of the type system. Sometimes ineffective.
- Moving: In a moving collector, objects can be relocated in memory. So addresses are not stable. 
- Compacting: At the end of collection, allocated memory(surviving objects) is arranged as a single contiguous region. Avoid fragments.
- Evacuating: At the end of collection, surviving objects are moved to a region of memory (seems like compacting?)
### Hotspot Runtime
Hotspot represents objects with a structure: oop(ordinary object pointer). There are instanceOop and klassOop. Modern Java points the klassOop to an external storage, not in the instance itself. 32-bit system can restrict klassOop in 32 bit and 64-bit in 64 bit. That means it's non-necessary. -XX:+UseCompressedOops command can compress the klassOop. Doesn't have much benefits however. 
GC Root:
- stack frames
- JNI
- Registers (hoisted variable case?)
- Code roots (from JVM code cache)
- globals
- class meta from loaded classes
The standard of root is a reference always pointing to an object in the heap (not null).
### Allocation
allocation rate is the amount of memory used by newly created objects in a time period. It's a driver to evaluate the behavior of GC of a Java app. 
GC heaps should be designed to allow short-lived objects easy to collect, long-lived objects easy to be separated  from short-lived objects. Use a generation counter to see how many rounds of GC an object survives.
Areas (from new to old): Eden(newly created objects) -> Survivor(after collection) -> Tenured(possibly long-lived)
Sometimes, old objects have reference on young objects. HotSpot maintains a structure called *card table* to record old objects who possibly point at young objects. This happens by observing if there is modification on old objects. Modified object will be marked *dirty*.
**Thread-local allocation buffers(TLABs)**: For efficiency, JVM partitions Eden into buffers and assign them to different app threads for creating new objects.
**Hemispheric Collection**: Split the space(**what space?**) into 2 halves. To avoid old objects mixing up with young objects, Each time move the survived objects to the other empty half and empty the collected space. Always have an empty half.
### Parallel Collectors
#### Young Parallel Collections
When a thread wants to create new object into Eden but finds there is no space, it will be paused, and all threads are to be paused(If one thread is unable to allocate, soon all will). Then young generation will be collected and survived objects will be moved to survivor area.
#### Old Parallel Collections
Default **old** generation collector in current Java. A compact collector with only a single contiguous memory space. Ole generation is not split into 2 hemispheres, so no evacuation. Sometimes, large objects will directly created into tenured area. 
#### Limitations
Parallel collectors are fully Stop-the world. STW doesn't hurt young collector, for most of objects die while a collection. If the heap grows and old objects increase, the ParallelOld may have problems as the pause time. 
In page 158:
```The System.gc() method exists, but is basically useless for any practical purpose.```
LOL

## Chap 3. Advanced Garbage Collection
GC components is pluggable for most JVMs, since different apps need different extent of GC. 
Main concerns of choosing GC:
- pause time
- throughput (GC time to app time)
- pause frequency
- reclamation efficiency (how much garbage will be collected in one cycle)
- pause consistency (all pauses have the same length)
*The minor disadvantage of this arrangement is the delay of the computation proper; its
major disadvantage is the unpredictability of these garbage collecting interludes.
—Edsger Dijkstra*
Pause time is acceptable, but not knowing when to GC is annoying. Then, concurrent garbage collector comes. 
OS has the authority to preempt a JVM thread, so JVM maintains a *safepoint* for each app thread. At safepoint, the states of the thread is good to pause. Then, when a GC thread wants to start a GC, it hasn't the authority to demand other threads to stop. So other app threads need to cooperate. JVM cannot force a thread to safepoint, but can prevent it leaving the safepoint. When a GC thread propose a GC, JVM sets a global *time to pause* flag, and threads process to safepoint then pause after seeing this flag. 
**Tri-Color Marking** is an algorithm to find collectable object. 
- Root as gray, others as white
- pick a random gray node, gray all its children and black itself
- do until no gray nodes
- black objects are reachable, white are not
In concurrent situations, marking thread is working, and mutator (app thread) makes a black node points to a white node, then the white node is reachable but not colored, then it'll be deleted. Solution 1: color the black to gray. Solution 2: add a "fixup" round after main phase.
**CMS** (Concurrent Mark and Sweep) is used for Tenured space. So, often work with a young collector. CMS uses half of avail threads to perform GC, and another half still perform app. Problem: app threads still create objects during GC. When Eden fills up, young GC comes up, while CMS is running. If some objects moved to Tenured from Eden, CMS needs to coordinate with this young collector. 
When there is high allocation, many objects in Eden need to move into Tenured. (Premature Promotion) This is called *Concurrent mode failure*. JVM needs to STW in all threads. To avoid this, CMS needs some space to avoid Tenured space full. Usually collects when 75%. CMS doesn't compact Tenured space, unlike ParallelOld. 
To use it: `-XX:+UseConcMarkSweepGC`
**G1** Garbage First collector. A collector that breaks the rules and stands. G1 heap is based on regions. Region has a size of <Heap size\> / 2048 (1MB e.g.). It's like cutting a heap into grids. If an object is more than half a region size, it's a *humongous object*. Then, it'll be allocated in *humongous region*, which is a free, contiguous region belong to Tenured. 
Unlike hemisphere collector, in G1 heap, Tenured, Eden, Survivor and Humongous are scarcely distributed. G1 uses Remembered Sets for each region, to keep track of outside references into other regions.
Procedure:
- Initial Mark (STW)
- Concurrent Root Scan
  - scan survivor regions for references to the old generation. Need to complete before young GC
- Concurrent Mark
- Remark (STW)
- Cleanup (STW)
To use it: `+XX:UseG1GC`
Developers can set a "pause time" for G1, let the app pause on collection cycle for a max pause time, however may not meet the goal. 
To use it: `-XX:MaxGCPauseMillis=200`
Also, G1 heap size can be changed: `-XX:G1HeapRegionSize=<n>`, and n must be power of 2 (1~64).
**Shenandoah** from Red Hat. Not in production as the book published. Shenandoah aims to reduce the pause time. Shenandoah uses *Brooks pointer*. It's additional word for every object to indicate if the object has been relocated during previous phase of GC. Brooks pointer points to next memory unit if not relocated, and points to new memory location if relocated. 
Procedure:
- Initial Mark (STW)
- Concurrent Marking
- Final Marking (STW)
- Concurrent Compaction
**C4 (Zing and Zulu)** Continuously Concurrent Compacting Collector from Azul.
**Balanced (IBM J9)**

## GC Logging, Monitoring, Tuning, and Tools
### Allocation
Rather than focusing on how to analyze JVM and GC, it's important to know how to make app have a good performance on allocation. 
- Trivial, avoidable object allocation
- Boxing costs (autoboxing and unboxing procedure can bring wasteful objects)
- Domain objects (objects to describe a problem domain and solve it)
  - itself may not be a major contributor. common types causing problems can be:
  - char[]
  - byte[]
  - double[]
  - Map entries
  - object[]
  - internal data structure
- Large number of non-JDK framework objects
Object allocation: first try TLAB; if full or larger than TLAB size, move to Eden; if Eden doesn't have space, GC; if still not, move to Tenured. So, large arrays are easy to create directly to Tenured. 
`-XX:MaxTenuringThreshold=<n>`, n controls how many rounds of GC an object must survive before promoted to Tenured. 
GC time can be affected by:
- number of app threads
- amount of compiled code in the *code cache*
- size of heap
To tune GC, tradeoffs are:
- fully STW
- High GC throughput / computationally cheap
- no possibility of a partial collection
- linearly growing pause time in the size of the heap
Use some flags, we can set ratio of young gen / survivor to heap, set min / max size of young gen, min / max of heap free after GC

## Code Execution on the JVM
JVM uses JIT compilation to what code to be compiled to executable code. JVM is a **stack** based machine for computation, compared to registers in OS. 
JVM provides 3 primary areas to hold data: 
- Evaluation stack: a method's local stack
- Local variables: a method's local and temporarily stored results
- Object heap: shared between methods and methods
### AOT and JIT
* compiling source code ahead of time means there's only 1 single opportunity to take advantage of any potential optimizations. 
### Code Cache 
JIT-compiled code is stored in a memory region called the *code cache*. Code cache comes from unallocated space. It's a linked free space. In memory space, it find free space -> allocate -> compact fragment (if possible)

## Java Language Performance Techniques
If something already exists in java.lang, don't write your own, unless you are a huge master, with a master team. 
Java heap contains:
- the most commonly allocated data structure, like strings, char arrays, instances of Java collections...
- data that corresponds to a **leak** will show up as an anomalously large dataset in jmap
If application-specific domain objects appear in jmap top 30, maybe a leak exists.

`finalize()` can be overrode by app classes. It's a method to let the object be collected. However, error can be thrown when finalizing. In JVM, there is a separated thread handling finalization. This method doesn't work very as stably as C++, careful.
**Try-with-resources** Java provides a `AutoCloseable` interface, witch provides a `close()` method. Add the object into try(), and the close will be invoked after the try-block. That works just the same as close() in the finally block.
```
public void read(File file) throws IOException {
    // BufferReader implements AutoCloseable
    try(BufferReader reader = new BufferReader(new FileReader(file))) {
        String firstLine = reader.readLine();
    }
    // we don't need to close it in the finally block
}
```

## Concurrent
JVM can't afford to provide Strong Memory (promise to let every thread get the same shared data), because JVM means to work on any platform. JVM only promises:
*Happens-Before*: one event happens before another
*Synchronizes-With*: the value of an object synched with main memory
*As-If-Serial*: instructions appear to execute in order outside of the executing thread ? 
*Release-Before-Acquire*: for locks will be released before being acquired by another thread

Unsafe is a class contains API that coupled with JVM. Unsafe provides possibilities: 
- Allocate an object without invoking constructor
- Allow raw memory and the equivalent of pointer arithmetic
- Use processor-specific hardware features
- Fast (de)serialization
- Thread-safe native memory access
- Atomic memory operations
- Efficient object / memory layouts
- Custom memory fences
- Fast interaction with native code
- A multi-operating system replacement for JNI
- Access to array element with volatile
For usage of Atomic classes, `set()` usually uses `Unsafe`'s `getAndSet()`. Inside, it's a loop of CAS's, lock free.
**Latch** Like a dam. Once destroyed, water floods in. CountdownLatch has an upper limit, when countdown to 0, all threads in `await()` can proceed. Latch is a one-time used object and can't be reset. 

**Executors** `submit()` consumes both `Runnable` and `Callable`.
- newFixedTreadPool(int nThreads): a thread pool with a fixed size. Reusable. Use a queue to store tasks when the pool is full.
- newCachedThreadPool(): pool stores threads for 60s. For small and async tasks.
- newSingleThreadExecutor(): create one single thread
- newScheduledThreadPool(int corePoolSize): take Callable, have some methods that allow a thread to be executed in future
Look into `ForkJoinPool` and `Actor`