### Notes

Memory tuning for large Aerospike (and similar) applications
(https://www.safaribooksonline.com/library/view/systems-performance-enterprise/9780133390124/ch07.html#ch07lev1sec6)
- Kernel parameters:
vm.dirty_background_bytes
vm.dirty_background_ratio
vm.dirty_bytes
vm.dirty_ratio
vm.dirty_expire_centisecs
vm.dirty_writeback_centisecs
vm.min_free_kbytes
vm.overcommit_memory
vm.swappiness
vm.vfs_cache_pressure

- tools:
    smem
    slabtop
    sysbench
    sysstat
    inxi


    AMC console
http://fopd-kmk9068.lab1.fanops.net:8081

- tuning parameters
-- nice/priority of the aerospike process
-- cgroups ?
-- interrupt / CPU utilization (networking)
