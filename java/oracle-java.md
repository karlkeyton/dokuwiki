[Source](http://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html "Permalink to java")

# java

These options control how garbage collection (GC) is performed by the Java HotSpot VM.

-XX:+AggressiveHeap
:

Enables Java heap optimization. This sets various parameters to be optimal for long-running jobs with intensive memory allocation, based on the configuration of the computer (RAM and CPU). By default, the option is disabled and the heap is not optimized.

-XX:+AlwaysPreTouch
:

Enables touching of every page on the Java heap during JVM initialization. This gets all pages into the memory before entering the `main()` method. The option can be used in testing to simulate a long-running system with all virtual memory mapped to physical memory. By default, this option is disabled and all pages are committed as JVM heap space fills.

-XX:+CMSClassUnloadingEnabled
:

Enables class unloading when using the concurrent mark-sweep (CMS) garbage collector. This option is enabled by default. To disable class unloading for the CMS garbage collector, specify `-XX:-CMSClassUnloadingEnabled`.

-XX:CMSExpAvgFactor=_percent_
:

Sets the percentage of time (0 to 100) used to weight the current sample when computing exponential averages for the concurrent collection statistics. By default, the exponential averages factor is set to 25%. The following example shows how to set the factor to 15%:

    -XX:CMSExpAvgFactor=15

-XX:CMSInitiatingOccupancyFraction=_percent_
:

Sets the percentage of the old generation occupancy (0 to 100) at which to start a CMS collection cycle. The default value is set to -1. Any negative value (including the default) implies that `-XX:CMSTriggerRatio` is used to define the value of the initiating occupancy fraction.

The following example shows how to set the occupancy fraction to 20%:

    -XX:CMSInitiatingOccupancyFraction=20

-XX:+CMSScavengeBeforeRemark
:

Enables scavenging attempts before the CMS remark step. By default, this option is disabled.

-XX:CMSTriggerRatio=_percent_
:

Sets the percentage (0 to 100) of the value specified by `-XX:MinHeapFreeRatio` that is allocated before a CMS collection cycle commences. The default value is set to 80%.

The following example shows how to set the occupancy fraction to 75%:

    -XX:CMSTriggerRatio=75

-XX:ConcGCThreads=_threads_
:

Sets the number of threads used for concurrent GC. The default value depends on the number of CPUs available to the JVM.

For example, to set the number of threads for concurrent GC to 2, specify the following option:

    -XX:ConcGCThreads=2

-XX:+DisableExplicitGC
:

Enables the option that disables processing of calls to `System.gc()`. This option is disabled by default, meaning that calls to `System.gc()` are processed. If processing of calls to `System.gc()` is disabled, the JVM still performs GC when necessary.

-XX:+ExplicitGCInvokesConcurrent
:

Enables invoking of concurrent GC by using the `System.gc()` request. This option is disabled by default and can be enabled only together with the `-XX:+UseConcMarkSweepGC` option.

-XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses
:

Enables invoking of concurrent GC by using the `System.gc()` request and unloading of classes during the concurrent GC cycle. This option is disabled by default and can be enabled only together with the `-XX:+UseConcMarkSweepGC` option.

-XX:G1HeapRegionSize=_size_
:

Sets the size of the regions into which the Java heap is subdivided when using the garbage-first (G1) collector. The value can be between 1 MB and 32 MB. The default region size is determined ergonomically based on the heap size.

The following example shows how to set the size of the subdivisions to 16 MB:

    -XX:G1HeapRegionSize=16m

-XX:+G1PrintHeapRegions
:

Enables the printing of information about which regions are allocated and which are reclaimed by the G1 collector. By default, this option is disabled.

-XX:G1ReservePercent=_percent_
:

Sets the percentage of the heap (0 to 50) that is reserved as a false ceiling to reduce the possibility of promotion failure for the G1 collector. By default, this option is set to 10%.

The following example shows how to set the reserved heap to 20%:

    -XX:G1ReservePercent=20

-XX:InitialHeapSize=_size_
:

Sets the initial size (in bytes) of the memory allocation pool. This value must be either 0, or a multiple of 1024 and greater than 1 MB. Append the letter `k` or `K` to indicate kilobytes, `m` or `M` to indicate megabytes, `g` or `G` to indicate gigabytes. The default value is chosen at runtime based on system configuration. See the section "Ergonomics" in _Java SE HotSpot Virtual Machine Garbage Collection Tuning Guide_ at `<http: docs.oracle.com="" javase="" 8="" docs="" technotes="" guides="" vm="" gctuning="" index.html="">`.

The following examples show how to set the size of allocated memory to 6 MB using various units:

    -XX:InitialHeapSize=6291456
    -XX:InitialHeapSize=6144k
    -XX:InitialHeapSize=6m

If you set this option to 0, then the initial size will be set as the sum of the sizes allocated for the old generation and the young generation. The size of the heap for the young generation can be set using the `-XX:NewSize` option.

-XX:InitialSurvivorRatio=_ratio_
:

Sets the initial survivor space ratio used by the throughput garbage collector (which is enabled by the `-XX:+UseParallelGC` and/or -`XX:+UseParallelOldGC` options). Adaptive sizing is enabled by default with the throughput garbage collector by using the `-XX:+UseParallelGC` and `-XX:+UseParallelOldGC` options, and survivor space is resized according to the application behavior, starting with the initial value. If adaptive sizing is disabled (using the `-XX:-UseAdaptiveSizePolicy` option), then the `-XX:SurvivorRatio` option should be used to set the size of the survivor space for the entire execution of the application.

The following formula can be used to calculate the initial size of survivor space (S) based on the size of the young generation (Y), and the initial survivor space ratio (R):

    S=Y/(R+2)

The 2 in the equation denotes two survivor spaces. The larger the value specified as the initial survivor space ratio, the smaller the initial survivor space size.

By default, the initial survivor space ratio is set to 8. If the default value for the young generation space size is used (2 MB), the initial size of the survivor space will be 0.2 MB.

The following example shows how to set the initial survivor space ratio to 4:

    -XX:InitialSurvivorRatio=4

-XX:InitiatingHeapOccupancyPercent=_percent_
:

Sets the percentage of the heap occupancy (0 to 100) at which to start a concurrent GC cycle. It is used by garbage collectors that trigger a concurrent GC cycle based on the occupancy of the entire heap, not just one of the generations (for example, the G1 garbage collector).

By default, the initiating value is set to 45%. A value of 0 implies nonstop GC cycles. The following example shows how to set the initiating heap occupancy to 75%:

    -XX:InitiatingHeapOccupancyPercent=75

-XX:MaxGCPauseMillis=_time_
:

Sets a target for the maximum GC pause time (in milliseconds). This is a soft goal, and the JVM will make its best effort to achieve it. By default, there is no maximum pause time value.

The following example shows how to set the maximum target pause time to 500 ms:

    -XX:MaxGCPauseMillis=500

-XX:MaxHeapSize=_size_
:

Sets the maximum size (in byes) of the memory allocation pool. This value must be a multiple of 1024 and greater than 2 MB. Append the letter `k` or `K` to indicate kilobytes, `m` or `M` to indicate megabytes, `g` or `G` to indicate gigabytes. The default value is chosen at runtime based on system configuration. For server deployments, `-XX:InitialHeapSize` and `-XX:MaxHeapSize` are often set to the same value. See the section "Ergonomics" in _Java SE HotSpot Virtual Machine Garbage Collection Tuning Guide_ at `<http: docs.oracle.com="" javase="" 8="" docs="" technotes="" guides="" vm="" gctuning="" index.html="">`.

The following examples show how to set the maximum allowed size of allocated memory to 80 MB using various units:

    -XX:MaxHeapSize=83886080
    -XX:MaxHeapSize=81920k
    -XX:MaxHeapSize=80m

On Oracle Solaris 7 and Oracle Solaris 8 SPARC platforms, the upper limit for this value is approximately 4,000 MB minus overhead amounts. On Oracle Solaris 2.6 and x86 platforms, the upper limit is approximately 2,000 MB minus overhead amounts. On Linux platforms, the upper limit is approximately 2,000 MB minus overhead amounts.

The `-XX:MaxHeapSize` option is equivalent to `-Xmx`.

-XX:MaxHeapFreeRatio=_percent_
:

Sets the maximum allowed percentage of free heap space (0 to 100) after a GC event. If free heap space expands above this value, then the heap will be shrunk. By default, this value is set to 70%.

The following example shows how to set the maximum free heap ratio to 75%:

    -XX:MaxHeapFreeRatio=75

-XX:MaxMetaspaceSize=_size_
:

Sets the maximum amount of native memory that can be allocated for class metadata. By default, the size is not limited. The amount of metadata for an application depends on the application itself, other running applications, and the amount of memory available on the system.

The following example shows how to set the maximum class metadata size to 256 MB:

    -XX:MaxMetaspaceSize=256m

-XX:MaxNewSize=_size_
:

Sets the maximum size (in bytes) of the heap for the young generation (nursery). The default value is set ergonomically.

-XX:MaxTenuringThreshold=_threshold_
:

Sets the maximum tenuring threshold for use in adaptive GC sizing. The largest value is 15. The default value is 15 for the parallel (throughput) collector, and 6 for the CMS collector.

The following example shows how to set the maximum tenuring threshold to 10:

    -XX:MaxTenuringThreshold=10

-XX:MetaspaceSize=_size_
:

Sets the size of the allocated class metadata space that will trigger a garbage collection the first time it is exceeded. This threshold for a garbage collection is increased or decreased depending on the amount of metadata used. The default size depends on the platform.

-XX:MinHeapFreeRatio=_percent_
:

Sets the minimum allowed percentage of free heap space (0 to 100) after a GC event. If free heap space falls below this value, then the heap will be expanded. By default, this value is set to 40%.

The following example shows how to set the minimum free heap ratio to 25%:

    -XX:MinHeapFreeRatio=25

-XX:NewRatio=_ratio_
:

Sets the ratio between young and old generation sizes. By default, this option is set to 2. The following example shows how to set the young/old ratio to 1:

    -XX:NewRatio=1

-XX:NewSize=_size_
:

Sets the initial size (in bytes) of the heap for the young generation (nursery). Append the letter `k` or `K` to indicate kilobytes, `m` or `M` to indicate megabytes, `g` or `G` to indicate gigabytes.

The young generation region of the heap is used for new objects. GC is performed in this region more often than in other regions. If the size for the young generation is too low, then a large number of minor GCs will be performed. If the size is too high, then only full GCs will be performed, which can take a long time to complete. Oracle recommends that you keep the size for the young generation between a half and a quarter of the overall heap size.

The following examples show how to set the initial size of young generation to 256 MB using various units:

    -XX:NewSize=256m
    -XX:NewSize=262144k
    -XX:NewSize=268435456

The `-XX:NewSize` option is equivalent to `-Xmn`.

-XX:ParallelGCThreads=_threads_
:

Sets the number of threads used for parallel garbage collection in the young and old generations. The default value depends on the number of CPUs available to the JVM.

For example, to set the number of threads for parallel GC to 2, specify the following option:

    -XX:ParallelGCThreads=2

-XX:+ParallelRefProcEnabled
:

Enables parallel reference processing. By default, this option is disabled.

-XX:+PrintAdaptiveSizePolicy
:

Enables printing of information about adaptive generation sizing. By default, this option is disabled.

-XX:+PrintGC
:

Enables printing of messages at every GC. By default, this option is disabled.

-XX:+PrintGCApplicationConcurrentTime
:

Enables printing of how much time elapsed since the last pause (for example, a GC pause). By default, this option is disabled.

-XX:+PrintGCApplicationStoppedTime
:

Enables printing of how much time the pause (for example, a GC pause) lasted. By default, this option is disabled.

-XX:+PrintGCDateStamps
:

Enables printing of a date stamp at every GC. By default, this option is disabled.

-XX:+PrintGCDetails
:

Enables printing of detailed messages at every GC. By default, this option is disabled.

-XX:+PrintGCTaskTimeStamps
:

Enables printing of time stamps for every individual GC worker thread task. By default, this option is disabled.

-XX:+PrintGCTimeStamps
:

Enables printing of time stamps at every GC. By default, this option is disabled.

-XX:+PrintStringDeduplicationStatistics
:

Prints detailed deduplication statistics. By default, this option is disabled. See the `-XX:+UseStringDeduplication` option.

-XX:+PrintTenuringDistribution
:

Enables printing of tenuring age information. The following is an example of the output:

    Desired survivor size 48286924 bytes, new threshold 10 (max 10)
    - age 1: 28992024 bytes, 28992024 total
    - age 2: 1366864 bytes, 30358888 total
    - age 3: 1425912 bytes, 31784800 total
    ...

Age 1 objects are the youngest survivors (they were created after the previous scavenge, survived the latest scavenge, and moved from eden to survivor space). Age 2 objects have survived two scavenges (during the second scavenge they were copied from one survivor space to the next). And so on.

In the preceding example, 28 992 024 bytes survived one scavenge and were copied from eden to survivor space, 1 366 864 bytes are occupied by age 2 objects, etc. The third value in each row is the cumulative size of objects of age n or less.

By default, this option is disabled.

-XX:+ScavengeBeforeFullGC
:

Enables GC of the young generation before each full GC. This option is enabled by default. Oracle recommends that you _do not_ disable it, because scavenging the young generation before a full GC can reduce the number of objects reachable from the old generation space into the young generation space. To disable GC of the young generation before each full GC, specify `-XX:-ScavengeBeforeFullGC`.

-XX:SoftRefLRUPolicyMSPerMB=_time_
:

Sets the amount of time (in milliseconds) a softly reachable object is kept active on the heap after the last time it was referenced. The default value is one second of lifetime per free megabyte in the heap. The `-XX:SoftRefLRUPolicyMSPerMB` option accepts integer values representing milliseconds per one megabyte of the current heap size (for Java HotSpot Client VM) or the maximum possible heap size (for Java HotSpot Server VM). This difference means that the Client VM tends to flush soft references rather than grow the heap, whereas the Server VM tends to grow the heap rather than flush soft references. In the latter case, the value of the `-Xmx` option has a significant effect on how quickly soft references are garbage collected.

The following example shows how to set the value to 2.5 seconds:

    -XX:SoftRefLRUPolicyMSPerMB=2500

-XX:StringDeduplicationAgeThreshold=_threshold_
:

`String` objects reaching the specified age are considered candidates for deduplication. An object's age is a measure of how many times it has survived garbage collection. This is sometimes referred to as tenuring; see the `-XX:+PrintTenuringDistribution` option. Note that `String` objects that are promoted to an old heap region before this age has been reached are always considered candidates for deduplication. The default value for this option is `3`. See the `-XX:+UseStringDeduplication` option.

-XX:SurvivorRatio=_ratio_
:

Sets the ratio between eden space size and survivor space size. By default, this option is set to 8. The following example shows how to set the eden/survivor space ratio to 4:

    -XX:SurvivorRatio=4

-XX:TargetSurvivorRatio=_percent_
:

Sets the desired percentage of survivor space (0 to 100) used after young garbage collection. By default, this option is set to 50%.

The following example shows how to set the target survivor space ratio to 30%:

    -XX:TargetSurvivorRatio=30

-XX:TLABSize=_size_
:

Sets the initial size (in bytes) of a thread-local allocation buffer (TLAB). Append the letter `k` or `K` to indicate kilobytes, `m` or `M` to indicate megabytes, `g` or `G` to indicate gigabytes. If this option is set to 0, then the JVM chooses the initial size automatically.

The following example shows how to set the initial TLAB size to 512 KB:

    -XX:TLABSize=512k

-XX:+UseAdaptiveSizePolicy
:

Enables the use of adaptive generation sizing. This option is enabled by default. To disable adaptive generation sizing, specify `-XX:-UseAdaptiveSizePolicy` and set the size of the memory allocation pool explicitly (see the `-XX:SurvivorRatio` option).

-XX:+UseCMSInitiatingOccupancyOnly
:

Enables the use of the occupancy value as the only criterion for initiating the CMS collector. By default, this option is disabled and other criteria may be used.

-XX:+UseConcMarkSweepGC
:

Enables the use of the CMS garbage collector for the old generation. Oracle recommends that you use the CMS garbage collector when application latency requirements cannot be met by the throughput (`-XX:+UseParallelGC`) garbage collector. The G1 garbage collector (`-XX:+UseG1GC`) is another alternative.

By default, this option is disabled and the collector is chosen automatically based on the configuration of the machine and type of the JVM. When this option is enabled, the `-XX:+UseParNewGC` option is automatically set and you should not disable it, because the following combination of options has been deprecated in JDK 8: `-XX:+UseConcMarkSweepGC -XX:-UseParNewGC`.

-XX:+UseG1GC
:

Enables the use of the garbage-first (G1) garbage collector. It is a server-style garbage collector, targeted for multiprocessor machines with a large amount of RAM. It meets GC pause time goals with high probability, while maintaining good throughput. The G1 collector is recommended for applications requiring large heaps (sizes of around 6 GB or larger) with limited GC latency requirements (stable and predictable pause time below 0.5 seconds).

By default, this option is disabled and the collector is chosen automatically based on the configuration of the machine and type of the JVM.

-XX:+UseGCOverheadLimit
:

Enables the use of a policy that limits the proportion of time spent by the JVM on GC before an `OutOfMemoryError` exception is thrown. This option is enabled, by default and the parallel GC will throw an `OutOfMemoryError` if more than 98% of the total time is spent on garbage collection and less than 2% of the heap is recovered. When the heap is small, this feature can be used to prevent applications from running for long periods of time with little or no progress. To disable this option, specify `-XX:-UseGCOverheadLimit`.

-XX:+UseNUMA
:

Enables performance optimization of an application on a machine with nonuniform memory architecture (NUMA) by increasing the application's use of lower latency memory. By default, this option is disabled and no optimization for NUMA is made. The option is only available when the parallel garbage collector is used (`-XX:+UseParallelGC`).

-XX:+UseParallelGC
:

Enables the use of the parallel scavenge garbage collector (also known as the throughput collector) to improve the performance of your application by leveraging multiple processors.

By default, this option is disabled and the collector is chosen automatically based on the configuration of the machine and type of the JVM. If it is enabled, then the `-XX:+UseParallelOldGC` option is automatically enabled, unless you explicitly disable it.

-XX:+UseParallelOldGC
:

Enables the use of the parallel garbage collector for full GCs. By default, this option is disabled. Enabling it automatically enables the `-XX:+UseParallelGC` option.

-XX:+UseParNewGC
:

Enables the use of parallel threads for collection in the young generation. By default, this option is disabled. It is automatically enabled when you set the `-XX:+UseConcMarkSweepGC` option. Using the `-XX:+UseParNewGC` option without the `-XX:+UseConcMarkSweepGC` option was deprecated in JDK 8.

-XX:+UseSerialGC
:

Enables the use of the serial garbage collector. This is generally the best choice for small and simple applications that do not require any special functionality from garbage collection. By default, this option is disabled and the collector is chosen automatically based on the configuration of the machine and type of the JVM.

-XX:+UseSHM
:

On Linux, enables the JVM to use shared memory to setup large pages.

For more information, see "Large Pages".

-XX:+UseStringDeduplication
:

Enables string deduplication. By default, this option is disabled. To use this option, you must enable the garbage-first (G1) garbage collector. See the `-XX:+UseG1GC` option.

_String deduplication_ reduces the memory footprint of `String` objects on the Java heap by taking advantage of the fact that many `String` objects are identical. Instead of each `String` object pointing to its own character array, identical `String` objects can point to and share the same character array.

-XX:+UseTLAB
:

Enables the use of thread-local allocation blocks (TLABs) in the young generation space. This option is enabled by default. To disable the use of TLABs, specify `-XX:-UseTLAB`. 
