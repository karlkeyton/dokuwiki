
[Source](http://highscalability.com/blog/2015/4/8/the-black-magic-of-systematically-reducing-linux-os-jitter.html "Permalink to The Black Magic of Systematically Reducing Linux OS&nbsp;Jitter")

# The Black Magic of Systematically Reducing Linux OS&nbsp;Jitter

![][1]

How do you systematically identify the sources of jitter and remove them one by one in a low-latency trading system? That was the question asked on the [mechanical-sympathy][2] email list.&nbsp;

[Gil Tene][3], Vice President of Technology and CTO, Co-Founder, Azul Systems gave the sort of answer that comes from the accumulated wisdom born from lots of real-life experience. It's an answer that needed sharing. And [here it is][4]:

Finding the cause of hiccups/jitters in a a Linux system is **black magic**. You often look at the spikes and imagine "what could be causing this".&nbsp;

Based on empirical evidence (across many tens of sites thus far) and note-comparing with others, I use a list of "usual suspects" that I blame whenever they are not set to my liking and system-level hiccups are detected. **Getting these settings right from the start often saves a bunch of playing around** (and no, there is no "priority" to this - you should set them all right before looking for more advice...).

My current starting point for Linux systems that are interested in avoiding many-msec hiccup levels is:

* Turn THP (Transparent Huge Pages) OFF.
* Set vm.min_free_kbytes to AT LEAST 1GB (8GB on larger systems).
* Set Swappiness to 0.
* Set zone_reclaim_mode to 0.

The defaults for items 1-4 are "wrong" on linux, and each can (independently) cause many-msec hiccups to occur. I know because I've personally run into each of them actually doing that (i.e. caught them red-handed, e.g. with a kernel stack trace showing THP's hands in the cookie jar).

In addition, I usually recommend:

* 5\. Turn HT (Hyper-threading) ON. (double the vcore run queues --&gt; umpteen times lower likelihood of waiting for a cpu).

While **HT=Off** is often recommended, it is a **pre-mature speed optimization** that comes with a significantly increased likelihood of "jitter" (if you consider a multi-msec stall "jitter"). Turning hyper threading off will never help reduce your system-caused hiccups (at least not in the &gt;20usec level). Turning HT off *may* (or may not) help improve the linear speed of execution of a thread. But it comes at the cost of halving the number of run queues available to the OS, which dramatically increases the likelihood of a scheduler-quantum level hiccups occurring if more runnable threads than cores exist even for a few msec. That's why I usually like to **keep HT on until you get the hiccups out of your system**. Then you can experiment with turning it off to see if (a) the hiccups don't come back, and (b) some other metrics got better.

Using **numactl, taskset, and isolcpus can all help individual threads with the jitter or hiccups they may experience** (in addition to cache behavior, etc.). Same goes for irqbalance. They are all nice for advanced stuff, but I like to **clean the system up first, and only assign cores to things after that**. In addition, once you &nbsp;do assign cores and such, you should **start measuring hiccups/jitter separately on the cores** (or set of cores) you are interested in. E.g. If you lock your process to node 1 with numactl, makes sure you run jHiccup (or whatever other tool you use) such that it is limited to that same node.

A last note about assigning cores: for hiccup/jitter control, **keeping things away from the cores you use is (much) more important than assigning your workload to specific cores**. It is critically more important to assign the cores for the rest of the system (e.g. init and everything that comes from it after boot) to stay away from your favorite workload than it is to carefully plot out which core will execute which thread of your workload.

I often see systems in which people carefully place threads on cores, but those cores are shared with pretty much any not-specifically-assigned process in the system (Yes, this seems silly, but it is also very common to find in the wild). My starting recommendation is often to** begin by keeping your "whole system" running on one socket** (e.g. node 0), and your latency-sensitive process on "the other socket" (e.g. node 1). Once you get that running nice and clean, and show that you don't have large hiccups you don't like and still need to get rid of, **you can start choosing cores within the socket if you want**. But you'll often find that you don't need to go that far...

Isolcpus are somewhat (but not entirely) different. With isolcpus you get the benefit of knowing that nobody will "accidentally" share your core. But **you also lose the scheduler's core-balancing nature for the threads that you assign to isolcpus cores**. So while it is sometimes useful to choose specific threads to use on isolcpus cores, the rest of the process (e.g. "the rest of the JVM", along with any background workers you may have in your process) will still benefit from a low-hiccup system or node, and that benefit will in turn show up even on the critical isocpus-assigned thread. E.g. in JVMs, even an isolcpus-assigned thread's worst hiccups will be dominated by things like time-to-safepoint across all JVM threads, so you are still susceptible to hiccups outside of isolcpu-assigned threads and cores.

## Related Articles

[1]: https://farm9.staticflickr.com/8718/17052576726_47212284bf_m.jpg
[2]: https://groups.google.com/forum/#!topic/mechanical-sympathy/DWlziVmyW-w
[3]: https://twitter.com/giltene
[4]: https://groups.google.com/forum/m/#!msg/mechanical-sympathy/DWlziVmyW-w/at-54WECjL4J
  
