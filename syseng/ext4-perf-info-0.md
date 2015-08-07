[Source](http://monolight.cc/2011/06/barriers-caches-filesystems/ "Permalink to Barriers, Caches, Filesystems | monolight")

# Barriers, Caches, Filesystems | monolight

With the recent proliferation of ext4 as the new "default" Linux filesystem there's been much talk of write barrier support. The flurry of post-2.6.18 barrier related development in most storage subsystems has left some novice users and administrators perplexed. I hope I can clear it up a bit with this primer/refresher.

If you're familiar with the basics of I/O caching, just skip to the "Barriers" section.

Barriers have long been implemented in the kernel, e.g. in ext3, XFS and ReiserFS 3. However, they are disabled by default in ext3. Up until recently, there was no barrier support for anything other than simple devices.  
&nbsp;

Two words: data safety

Let's take a look at the basic path data takes through the storage layers during a write-out in a modern storage setup:

![][1]

Some of these layers/components have their own caches:

![][2]

There may be other caches in the path, but this is the usual setup. The page cache is omitted if data is written in [O_DIRECT][3] mode.

When a userland process writes data to a filesystem, it's paramount (unless explicitly requested otherwise) that the data makes it safely to physical, non-volatile media. This is a part of the "D" in [ACID][4]. Generally, data may be lost if it's in volatile storage during hardware failure (e.g. power loss) or software crash.  
&nbsp;

Caches

The OS [page cache][5] (a subsystem of the VFS cache) and the buffer cache are in the host's RAM, obviously volatile. The risk here is that the page cache is relatively large compared to other caches. It can't survive OS crashes.

The storage controller write cache is present in most mid- and hi-end controllers and/or HBAs working in modes other than initiator-target: RAID HBAs, DAS RAID boxen, SAN controllers, etc. Every vendor seems to have their own name for it:
* BBWC (Battery-Backed Write Cache)
* BBU (Battery-Back-Up [Write Cache])
* Array Accelerator (BBWC – in HPese)
* FBWC (Flash-Backed Write Cache)

As the names suggest, BBWC is simply some memory and a rechargeable battery, usually in one or more proprietary FRU modules. In hi-end storage systems, the battery modules are hot-swappable, in mid-end systems a controller has to be shut down for battery replacement. RAID HBAs require host down time for battery maintenance unless you have hot-swap slots and multiple HBAs serving multiple paths.

FBWC is the relatively new generation of volatile cache where the battery assembly is replaced with NAND flash storage – not unlike today's SSDs – and a replaceable capacitor bank that holds enough charge to allow data write-out from DRAM to flash in case of power failure.

Both types of cache have their drawbacks: BBWC needs constant battery monitoring and re-learning. Re-learning is a recurring process: the controller fully cycles (discharges and recharges) the battery to learn its absolute capacity – which obviously deteriorates with time and usage (cycles). While re-learning, write cache must be disabled (since at some point in the process the battery will be almost completely discharged and unable to power the BBWC memory). This is a periodic severe performance penalty for write-heavy workloads, unless there's a redundant battery and/or controller to take over. Good controllers allow the administrator to customize re-learn schedules. The batteries must be replaced every few months or years.

Flash-based write cache is also subject to deterioration: the dreaded maximum write count for flash memory cells (however, flash is used only on power failure). The backup capacitors degrade over time. The NAND modules and the capacitor bank must be monitored and replaced if necessary.

Write cache on physical media (disk drives) is almost always volatile. Most enterprise SSDs and some consumer SSDs (e.g. the Intel 320 series, but not the extremely popular X25-M series) have backup capacitors.

Modern disks have 16-64 megabytes of cache. The problem with this type of cache is that not all drives will flush it reliably when requested. SCSI and SAS drives do the right thing: the "`SYNCHRONIZE CACHE`" (opcode 35) command is a part of the SCSI standard. PATA drives have usually outright lied to cheat on benchmarks. SATA does have the "`FLUSH CACHE EXT`" command, but whether the drive actually acts on it depends on the vendor. Get SCSI/SAS drives for mission critical data – nothing new here.

One more caveat with disk write cache is that the controller software – to ensure data durability – MUST guarantee that all data flushed out of the controller write cache is committed to non-volatile media. In other words, when the OS requests a flush and the controller returns success, the data MUST have already been committed to non-volatile media. This is why disk write cache MUST be disabled if BBWC or other form of controller cache is enabled – the controller cache must be flushed directly to non-volatile media and not to another layer of volatile cache.

Software RAID with JBOD is a special case: there is no controller cache, only the drive cache, the OS page cache and buffer cache.  
&nbsp;

Barriers

Think of write barriers on Linux as a unified approach to flushing and forced I/O ordering.

Consider the following setup:

![][6]

This is a bit on the extreme side, but ponder for a moment how many layers of I/O (and caches) the data has to pass through to be stored on the physical disks.

If the filesystem is barrier-aware and all I/O layers support barriers/flushes, an fs transaction followed by a barrier is committed (flushed) to persistent storage (disks). **All requests issued prior to the barrier must be satisfied before continuing.** Also, an `fsync()` or a similar call will flush the write caches of the underlying storage (`fsync()` without barriers does NOT guarantee this!). Barrier `bio`s (block I/Os) actually do _two_ flushes: one before the `bio` and one afterwards. It's possible to issue an empty barrier `bio` to flush only once.

Barriers ensure critical transactions are committed to persistent media _and_ committed in the right order, but they incur a – sometimes severe – performance penalty.

Let's get back to our two hardware setups: software RAID on JBOD and hardware RAID with BBWC.

Since barriers force write-outs to persistent storage, disk write cache can be safely enabled for MD RAID if the following conditions are met:

* the filesystem supports barriers and they are enabled
* the underlying I/O layers support barriers/flushes (see below)
* the disks reliably support cache flushes.

However, on hardware RAID with BBWC, the cache itself is (quasi-)persistent. Since RAID controllers do implement the `SYNCHRONIZE CACHE` command, each barrier would flush the entire write cache, negating the performance advantage of BBWC. It's recommended to disable barriers if – and only if – you have _healthy_ BBWC. If you disable barriers, you must monitor and properly maintain your BBWC.

Full support for barriers on various virtual devices has been added only recently. This is a rough matrix of barrier support in vanilla kernel versions, milestones highlighted:

| ----- |
| **Barrier support** |  **Kernel version** |  **Commit** |
| I/O barrier support |  2.6.9 |  [1][7] |
| ext3 |  2.6.9 |  [1][8] |
| reiserfs |  2.6.9 |  [1][9] |
| SATA |  2.6.12 |  - |
| **XFS – barriers enabled by default** |  2.6.16 |  [1][10] |
| **ext4 – barriers enabled by default ** |  2.6.26 |  [1][11] |
| **DM – simple devices** (i.e. a single underlying device) |  2.6.28 |  [1][12] |
| loop |  2.6.30 |  [1][13] |
| **DM – rewrite of the barrier code** |  2.6.30 |  [1][14] |
| DM – crypt |  2.6.31 |  [1][15] |
| **DM – linear** (i.e. standard LVM concatenated volumes) |  2.6.31 |  [1][16] |
| DM – mpath |  2.6.31 |  [1][17] |
| **virtio-blk** (only really safe with O_DIRECT backing devices) |  2.6.32 |  [1][18] |
| DM – dm-raid1 |  2.6.33 |  [1][19] |
| DM – request based devices |  2.6.33 |  [1][19] |
| **MD barrier support on all personalities** * |  2.6.33 |  [1][20] |
| **barriers removed and replaced with FUA / explicit flushes** |  2.6.37 |  [1][21] [2][22] [3][23] |
* _Note: previously barriers were only supported on MD raid1. This patch can be easily applied to 2.6.32._

As of 2.6.37, block layer barriers have been removed from the kernel for performance reasons. They have been completely superseded by explicit flushes and FUA requests.

FUA is [Force Unit Access][24]: an I/O request flag which ensures the transferred data is written directly to (or read from) persistent media, regardless of any cache settings.

Explicit flushes are just that – write cache flushes explicitly requested by a filesystem. In fact, the responsibility for safe request ordering has been completely moved to filesystems. The block layer or [TCQ/NCQ][25] can safely reorder requests if necessary, since the filesystem will issue flush/FUA requests for critical transactions anyway – and wait for their completion before proceeding.

These changes eliminate the barrier-induced request queue drains that significantly affected write performance. Other I/O requests (e.g. without a transaction) can be issued to a device while a transaction is still being processed.

However, as 2.6.32.x is the longterm kernel for several distros, barriers are here to stay (at least for a few years).  
&nbsp;

Filesystems

Barriers/flushes are supported on most modern filesystems: ext3, ext4, XFS, JFS, ReiserFS 3, etc. ext3/4 are unique in that they support three data journaling modes: `data={ordered,journal,writeback}`.

`data=journal` essentially writes data twice: first to the journal and then to the data blocks.

`data=writeback` is similar to journaling on XFS, JFS, or ReiserFS 3 before Linux 2.6.6. Only internal filesystem integrity is preserved and only metadata is journaled; data may be written to the filesystem out of order. Metadata changes are first recorded in the journal and a commit block is written. After the journal has been updated, metadata and data write-outs may proceed. `data=writeback` can be a severe security risk: if the system crashes while appending to a file, after the metadata has been committed (and additional data blocks allocated), but before the data has been written (data blocks overwritten with new data), then after journal recovery that file may contain blocks filled with data from previously deleted files – from any user.

_Note: ReiserFS 3 supports `data=ordered` since 2.6.6 and it's the default mode. XFS does support ordering in specific cases, but it's neither always guaranteed nor enforced via the journaling mechanism. There is some confusion about that, e.g. this Wikipedia [article][26] on ext3 and this [paper [PDF]][27] seem to contradict what a developer from SGI [stated][28] (the paper seems flawed anyway, as an assumption is made that XFS is running in ordered mode, based on the result of one test)._

`data=ordered` only journals metadata, like writeback mode, but groups metadata and data changes together into transactions. Transaction blocks are written together, data first, metadata last.

With barriers enabled, the order looks more or less like this:

1. the transaction is written
2. a barrier request is issued
3. the commit block is written
4. another barrier is issued

There is a special case on ext4 where the first barrier (between the transaction and the commit block) is omitted: the `journal_async_commit` mount option. ext4 supports journal checksumming – if the commit block has been written but the checksum is incorrect, the transaction will be discarded at journal replay. With `journal_async_commit` enabled the commit block may be written without waiting for the transaction write-out. There's a caveat: before this [commit][29] the barrier was missing at step 4 in async commit mode. The patch adds it, so that now there's a single _empty_ barrier (step 4) _after_ the commit block instead of a full barrier (two flushes) _around_ it.

ext3 tends to flush more often than ext4. By default both ext3 and ext4 are mounted with `data=ordered` and `commit=5`. On ext3 this means not only the journal, but effectively all data is committed every 5 seconds. However, ext4 introduces a new feature: [delayed allocation][30].

_Note: delayed allocation is by no means a new concept. It's been used for years e.g. in XFS; in fact ext4 behaves similarly to XFS in this regard._

New data blocks on disk are not immediately allocated, so they are not written out until the respective dirty pages in the page cache expire. The expiration is controlled by two tunables:

    /proc/sys/vm/dirty_expire_centisecs
    /proc/sys/vm/dirty_writeback_centisecs

The first variable determines the expiration age – 30 seconds by default as of 2.6.32. On expiration, dirty pages are queued for eviction. The second variable controls the wakeup frequency of the "flush" kernel threads, which process the queues.

You can check the current cache sizes:

    grep ^Cached: /proc/meminfo # page cache size
    grep ^Dirty: /proc/meminfo # total size of all dirty pages
    grep ^Writeback: /proc/meminfo # total size of actively processed dirty pages

_Note: The VFS cache (e.g. dentry and inode caches) can be further examined by viewing the `/proc/slabinfo` file (or with the `slabtop` util which gives a nice breakdown of the slab count, object count, size, etc)._

_Note: before 2.6.32 there was a well-known subsystem called `pdflush`: global kernel threads for all devices, spawned and terminated on demand (the rule of thumb is: if all `pdflush` threads have been busy for 1 second, spawn another thread. If one of the threads has been idle for 1 second, terminate). It's been replaced with per-BDI (per-backing-device-info) flushers – one flush thread per each _logical_ device (one for each filesystem)._

On top of all that, there was the dreaded pre-2.6.30 ["ext4 delayed allocation data loss"][31] [bug/feature][32]. Workarounds were introduced in 2.6.30, namely the [`auto_da_alloc][33]` mount option, enabled by default.

You should also take into consideration the size of the OS page cache. These days machines have a lot of RAM (32+ or 64+ GB is not uncommon). The more RAM you have, the more dirty pages can be held in RAM before flushing to disk. By default, Linux 2.6.32 will start writing out dirty pages when they reach 10% of RAM. On a 32 GB machine this is 3.2 GB of uncommitted data in write-heavy environments, where you don't hit the time based constraints mentioned above – quite a lot to lose in the event of a system crash or power failure.

This is why it's so important to ensure data integrity in your software by flushing critical data to disks – e.g. by `fsync()`ing (though at the application level you may only hope the filesystem, the OS and the devices will all do the right thing). This is why database systems have been doing it for decades. Also, this is one of the reasons why some database vendors recommend placing transaction commit logs on a separate filesystem. The synchronous load profile of the commit log would otherwise interfere with the asynchronous flushing of the tablespaces: if the logs were kept on a single filesystem along with the tablespaces, every fsync would flush all dirty pages for that filesystem, killing I/O performance.

_Note: `fsync()` is a double-edged sword in this case. `fsync`ing too often will reduce performance (and spin up devices). That's why only critical data should be `fsync`ed._

Dirty page flushing can be tuned – traditionally with these two tunables:

    /proc/sys/vm/dirty_background_ratio
    /proc/sys/vm/dirty_ratio

Both values are expressed as a percentage of RAM. When the amount of dirty pages reaches the first threshold (`dirty_background_ratio`), write-outs begin in the background via the "flush" kernel threads. When the second threshold is reached, processes will block, flushing in the foreground.

The problem with these variables is their minimum value: even 1% can be too much. This is why another two controls were [introduced][34] in 2.6.29:

    /proc/sys/vm/dirty_background_bytes
    /proc/sys/vm/dirty_bytes

They're equivalent to their percentage based counterparts. Both pairs of tunables are exclusive: if either is set, its respective counterpart is reset to 0 and ignored. These variables should be tuned in relation to the BBWC memory size (or disk write cache size on MD RAID). Lower values generate more I/O requests (and more interrupts), significantly decrease sequential I/O bandwidth but also decrease random I/O latency. The idea is to find a sweet spot where BBWC would be used most effectively: the ideal I/O rate should not allow BBWC to overfill or significantly under-fill. Obviously, this is hit/miss and only theoretically achievable under perfect conditions. As usual, you should tune and benchmark for your specific workload.

When benchmarking, remember ext3 has barriers disabled by default. A direct comparison of ext3 to ext4 with default mount options is usually quite pointless. ext4 offers an increased level of data protection at the cost of speed. Likewise, directly comparing ext3 in ordered mode to a filesystem offering only metadata journaling may not yield conclusive results. [Some][35] [people][36] got their benchmarks wrong.

_Note: I did that kind of [benchmark][37] a while ago: the goal was to measure system file operations (deliberately on default settings), not sequential throughput or IOPS – and ext4 was faster anyway._

All in all, it's your data! Test everything yourself with your specific workloads, hardware and configuration. Here's a [simple barrier test workload][38] to get you going.

[1]: http://monolight.cc/uploads/2011/06/storage.png "storage"
[2]: http://monolight.cc/uploads/2011/06/caches.png "caches"
[3]: http://www.kernel.org/doc/man-pages/online/pages/man2/open.2.html
[4]: http://en.wikipedia.org/wiki/ACID
[5]: http://en.wikipedia.org/wiki/Page_cache
[6]: http://monolight.cc/uploads/2011/06/storage-example.png "storage-example"
[7]: http://kernel.org/git/?p=linux/kernel/git/torvalds/old-2.6-bkcvs.git;a=commit;h=74c50b2c1af1b3535cf6c39ce684e60fa9f5dfdb
[8]: http://kernel.org/git/?p=linux/kernel/git/torvalds/old-2.6-bkcvs.git;a=commit;h=52a75614f1f753e01a5a9610c5390b5b7f795912
[9]: http://kernel.org/git/?p=linux/kernel/git/torvalds/old-2.6-bkcvs.git;a=commit;h=b736095823b073209ef38622227c64c593260b73
[10]: http://kernel.org/git/?p=linux/kernel/git/torvalds/linux-2.6.git;a=commit;h=4ef19dddbaf2f24e492c18112fd8a04ce116daca
[11]: http://git.kernel.org/git/?p=linux/kernel/git/torvalds/linux-2.6.git;a=commit;h=571640cad3fda6475da45d91cf86076f1f86bd9b
[12]: http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=commit;h=ab4c1424882be9cd70b89abf2b484add355712fa
[13]: http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=commitdiff;h=68db1961bbf4e16c220ccec4a780e966bc1fece3
[14]: http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=commitdiff;h=af7e466a1acededbc10beaba9eec8531d561c566
[15]: http://git.kernel.org/linus/647c7db14ef9cacc4ccb3683e206b61f0de6dc2b
[16]: http://git.kernel.org/linus/433bcac5645508b71eab2710b6817c3ef937eba8
[17]: http://git.kernel.org/linus/8627921fa2ef6d40fd9b787e163ba3a9ff8f471d
[18]: http://git.kernel.org/linus/f1b0ef062602713c2c7cfa12362d5d90ed01c5f6
[19]: http://git.kernel.org/linus/d0bcb8786532b01206f04258eb6b7d4ac858436a
[20]: http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=commitdiff;h=a2826aa92e2e14db372eda01d333267258944033
[21]: http://git.kernel.org/linus/28e7d1845216538303bb95d679d8fd4de50e2f1a
[22]: http://git.kernel.org/linus/4fed947cb311e5aa51781d316cefca836352f6ce
[23]: https://lkml.org/lkml/2010/10/22/157
[24]: http://ldkelley.com/SCSI2/SCSI2/SCSI2-09.html
[25]: http://en.wikipedia.org/wiki/Tagged_Command_Queuing
[26]: http://en.wikipedia.org/wiki/Ext3#Journaling_levels
[27]: http://pages.cs.wisc.edu/~vshree/xfs.pdf
[28]: http://oss.sgi.com/archives/xfs/2007-08/msg00755.html
[29]: http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=commitdiff;h=0e3d2a6313d03413d93327202a60256d1d726fdc
[30]: https://ext4.wiki.kernel.org/index.php/Ext4_Howto#Delayed_allocation
[31]: https://bugs.edge.launchpad.net/ubuntu/+source/linux/+bug/317781/comments/45
[32]: http://img217.imageshack.us/img217/121/bugfeatureqq6.jpg
[33]: http://www.mjmwired.net/kernel/Documentation/filesystems/ext4.txt
[34]: http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=commit;h=2da02997e08d3efe8174c7a47696e6f7cbe69ba9
[35]: http://blog.2ndquadrant.com/en/greg-smith/2011/02/
[36]: http://www.phoronix.com/scan.php?page=article&amp;item=linux_perf_regressions&amp;num=1
[37]: http://monolight.cc/2011/02/linux-filesystems-small-file-performance-on-hdds/
[38]: http://www.spinics.net/lists/linux-ext4/msg06665.html
