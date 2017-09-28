+++
title  = "cgroup"
toc    = true
weight = 0
+++

Process group별 Management를 제공하는 I/F이다.

Cgroup자체는 Process Grouping만을 수행하며, \
Resource에 대한 관리는 Subsystem을 통해 이뤄진다. \
Cgroup으로 만들어진 Process Group은 hierachy하게 만들어질 수 있다. 

결과적으로 Cgroup을 통해 다음과 같은 기능을 제공한다.

* limiting : Group에 대한 resoure 사용 제한. \
* prioritization : CPU 및 disk I/O 자원에 대한 우선순위 부여. \
* accounting : System에서 사용하는 resource 측정. \
* control : Group별 contol 제공. 

sysfs나 procfs와 같이 low-level filesystem interface로 구현되며, \
모든 cgroups 관련 action은 filesystem을 통해서 이뤄진다.

create/remove directory, read/write file, mount/umount

* cgroup inode_operation : cgroup mkidr/rmdir
* cgroup file_system_type : cgroup mount/umount
* cgroup file_operation : read/write to control file

| subsystem       | version |
|-----------------|---------|
| cgroups         | 2.6.24  |
| net_prio        | 3.3     |
| net_cls         | 3.3     |
| blkio async I/O | 3.10    |

![alt tag](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/images/RMG-rule1.png)

![alt tag](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/images/RMG-rule2.png)

![alt tag](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/images/RMG-rule3.png)

![alt tag](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/images/RMG-rule4.png)

## User View

일반적으로 /sys/fs/cgroups에 cgroupfs가 mount된다. 
여기서 생성되는 모든 entry는 reboot될때 지워진다. → persistent 하지 않다.

```bash
$ ls -l /sys/fs/cgroups
total 0
dr-xr-xr-x 2 root root  0 Jan  1  1970 bfqio
dr-xr-xr-x 4 root root  0 Jan  1  1970 blkio
lrwxrwxrwx 1 root root 11 Jan  1  1970 cpu -> cpu,cpuacct
dr-xr-xr-x 4 root root  0 Jan  1  1970 cpu,cpuacct
lrwxrwxrwx 1 root root 11 Jan  1  1970 cpuacct -> cpu,cpuacct
dr-xr-xr-x 3 root root  0 Jan  1  1970 cpuset
dr-xr-xr-x 4 root root  0 Jan  1  1970 devices
dr-xr-xr-x 3 root root  0 Jan  1  1970 freezer
lrwxrwxrwx 1 root root 16 Jan  1  1970 net_cls -> net_cls,net_prio
dr-xr-xr-x 2 root root  0 Jan  1  1970 net_cls,net_prio
lrwxrwxrwx 1 root root 16 Jan  1  1970 net_prio -> net_cls,net_prio
dr-xr-xr-x 2 root root  0 Jan  1  1970 perf_event
dr-xr-xr-x 4 root root  0 Jan  1  1970 systemd`
```

전체 cgroup에 대한 요약정보를 /proc에서 얻을 수 있다.

```bash
$ cat /proc/cgruops
#subsys_name    hierarchy       num_cgroups     enabled
cpuset  7       3       1
cpu     4       41      1
cpuacct 4       41      1
blkio   5       41      1
memory  0       1       0
devices 3       41      1
freezer 9       3       1
net_cls 2       1       1
bfqio   8       1       1
perf_event      6       1       1
net_prio        2       1       1
```

현재 process와 관련, mount된 cgroup을 다음과 같이 알 수 있다.

```bash
$  cat /proc/self/cgroup
9:devices:/user.slice/user-0.slice/session-c3.scope
8:net_cls,net_prio:/
7:perf_event:/
6:cpuset:/
5:freezer:/
4:bfqio:/
3:blkio:/user.slice/user-0.slice/session-c3.scope
2:cpu,cpuacct:/user.slice/user-0.slice/session-c3.scope
1:name=systemd:/user.slice/user-0.slice/session-c3.scope
[root@pi boot]# cat /proc/$$/cgroup
9:devices:/user.slice/user-0.slice/session-c3.scope
8:net_cls,net_prio:/
7:perf_event:/
6:cpuset:/
5:freezer:/
4:bfqio:/
3:blkio:/user.slice/user-0.slice/session-c3.scope
2:cpu,cpuacct:/user.slice/user-0.slice/session-c3.scope
1:name=systemd:/user.slice/user-0.slice/session-c3.scope
```

cgroup tree에 관해선 아래와 같은 rule이 적용된다. 

## Kernel View

kernel 내 아래 path에 cgroup 관련 hook이 들어가 있다.

* in boot phase
  * start_kernel() : http://lxr.free-electrons.com/source/init/main.c?v=4.0#L489
  * cgroup_init_early() : http://lxr.free-electrons.com/source/init/main.c?v=4.0#L508
  * cgroup_init_early() : http://lxr.free-electrons.com/source/include/linux/cgroup.h#L32
  * cgroup_init_early() : http://lxr.free-electrons.com/source/kernel/cgroup.c?v=4.0#L4953
  * cgroup_init() : http://lxr.free-electrons.com/source/init/main.c?v=4.0#L657
  * cgroup_init() : http://lxr.free-electrons.com/source/include/linux/cgroup.h#L33
  * cgroup_init() : http://lxr.free-electrons.com/source/kernel/cgroup.c?v=4.0#L4987
* in process creation/destory method, fork() & exit()
  * copy_procyss() : http://lxr.free-electrons.com/source/kernel/fork.c?v=4.0#L1190
  * cgroup_fork() : http://lxr.free-electrons.com/source/kernel/fork.c?v=4.0#L1324
  * cgroup_fork() : http://lxr.free-electrons.com/source/include/linux/cgroup.h?v=4.0#L34
  * cgroup_fork() : http://lxr.free-electrons.com/source/kernel/cgroup.c?v=4.0#L5182
  * cgroup_post_fork() : http://lxr.free-electrons.com/source/kernel/fork.c?v=4.0#L1549
  * cgroup_post_fork() : http://lxr.free-electrons.com/source/include/linux/cgroup.h?v=4.0#L35
  * cgroup_post_fork() : http://lxr.free-electrons.com/source/kernel/cgroup.c?v=4.0#L5198
* new file system type “cgroup” (VFS)
  * /sys/fs/cgroup : system에서 사용하는 cgroup mount point
* process descriptor additions (struct task_struct)
* procfs entries :
  * /proc/pid/cgroup
  * /proc/cgroups

task_struct내에서 관련 filed를 찾을 수 있다. \
struct task_struct : http://lxr.free-electrons.com/source/include/linux/sched.h?v=4.0#L1278 \
struct css_set __rcu *cgroups : http://lxr.free-electrons.com/source/include/linux/sched.h?v=4.0#L1570 \
struct css_set : http://lxr.free-electrons.com/source/include/linux/cgroup.h?v=4.0#L325 \

아래에서 list가 link된다. \
struct list_head task : http://lxr.free-electrons.com/source/include/linux/cgroup.h?v=4.0#L343 \
지원하는 subsystem에 대해 기술되어 있다. \
cgroup_subsys.h : http://lxr.free-electrons.com/source/include/linux/cgroup_subsys.h?v=4.0 \

## Subsystem

Group별로 Subsystem을 통해 Group을 제어 할 수 있다. \
상위 Group의 Subsystem 정책은 하위로 상속된다. \

4.1 기준으로 현재 11개의 subsystem을 지원해주고 있다. \

* cpuset_subsys
  * 개별 cpu와 memory node를 task에 할당한다.
  * struct cgroup_subsys cpuset_cgrp_subsys :
  * http://lxr.free-electrons.com/source/kernel/cpuset.c?v=4.0#L2036
* freezer_subsys
  * task를 suspend하거나 resume 한다.
  * struct cgroup_subsys freezer_cgrp_subsys :
  * http://lxr.free-electrons.com/source/kernel/cgroup_freezer.c?v=4.0#L476
* mem_cgroup_subsys
  * task의 memory 사용을 제한하고 task의 memory 사용량을 report 한다.
  * struct cgroup_subsys :
  * http://lxr.free-electrons.com/source/mm/memcontrol.c?v=4.0#L5381
* blkio_subsys
  * block device의 input/output을 제한한다.
  * struct cgroup_subsys blkio_cgrp_subsys :
  * http://lxr.free-electrons.com/source/block/blk-cgroup.c?v=4.0#L924
* net_prio_subsys
  * network interface 별로 priority를 동적으로 설정해준다.
  * struct cgroup_subsys net_prio_cgrp_subsys :
  * http://lxr.free-electrons.com/source/net/core/netprio_cgroup.c?v=4.0#L247
  * module로 사용 가능.
* devices_subsys
  * task의 device access를 제한한다.
  * struct cgroup_subsys devices_cgrp_subsys :
  * http://lxr.free-electrons.com/source/security/device_cgroup.c?v=4.0#L794
* perf_subsys
  * struct cgroup_subsys perf_event_cgrp_subsys :
  * http://lxr.free-electrons.com/source/kernel/events/core.c?v=4.0#L8629
* hugetlb_subsys
  * struct cgroup_subsys hugetlb_cgrp_subsys :
  * http://lxr.free-electrons.com/source/mm/hugetlb_cgroup.c?v=4.0#L418
* cpu_cgrp_subsys
  * struct cgroup_subsys cpu_cgrp_subsys :
  * http://lxr.free-electrons.com/source/kernel/sched/core.c?v=4.0#L8363
* cpuacct_subsys
  * struct cgroup_subsys cpuacct_cgrp_subsys :
  * http://lxr.free-electrons.com/source/kernel/sched/cpuacct.c?v=4.0#L278
* net_cls_subsys
  * module로 사용 가능.

위는 다시 아래와 같이 구분해볼 수 있다.

* Isolation and Sepcial Controller
  * cpuset, namespace, freezer, device, checkpoint/restart
* Resource Controller
  * cpu(scheduler), memory, disk I/O, network

subsystem은 각기 따로 사용할수도 있고 한번에 사용할 수도 있다.

```bash
$ mount -t cgroup -o cpu none /cpu

$ mount -t cgroup -o cpuset none /cpuset

$ mount -t cgroup none /cgroups
```

## Cgroup Control
#### Systemd

systemd를 사용하는 system에선 systemd 도구로 cgroup을 제어할 수 있다. 
다음 명령은 system 상에 존재하는 cgroup과 subsystem 구조를 보여준다.

```bash
$ systemd-cgls
```

cgroup별 resource 사용 현황도 살필 수 있다.

```bash
$ systemd-cgtop
```
현재 process에 적용된 정책을 살펴보자
```bash
$ cat /sys/fs/cgroup/cpu/system.slice/smbd.service/cpu.shares
1024

$ systemctl set-property smbd.service CPUShares=200

$ systemctl show -p CPUShares smbd.service
CPUShares=200

$ cat /sys/fs/cgroup/cpu/system.slice/smbd.service/cpu.shares
200
```
  
#### cpu 
```bash
$ ls /sys/fs/cgroup/cpu
cgroup.clone_children  cpu.cfs_period_us  cpu.rt_runtime_us  cpuacct.stat          notify_on_release  tasks
cgroup.procs           cpu.cfs_quota_us   cpu.shares         cpuacct.usage         release_agent      user.slice
cgroup.sane_behavior   cpu.rt_period_us   cpu.stat           cpuacct.usage_percpu  system.slice
```

주요 I/F는 아래와 같다.

| I/F               | Description                                                   |
|-------------------|---------------------------------------------------------------|
| cpu.shares        | CPU core의 점유율을 2~262114의 수치로 지정한다(default 1024). |
| cpu.cfs_period_us | CPU 사용 시간 비율의 상한을 지정(default는 무제한)            |
| cpu.cfs_quota_us  |                                                               |

#### cpuset

cpuset을 이용해 process별 core 사용에 대한 제한을 가한다. 

```bash
$ ls /sys/fs/cgroup/cpuset
cgroup.clone_children  cpuset.effective_cpus  cpuset.memory_pressure          cpuset.sched_load_balance
cgroup.procs           cpuset.effective_mems  cpuset.memory_pressure_enabled  cpuset.sched_relax_domain_level
cgroup.sane_behavior   cpuset.mem_exclusive   cpuset.memory_spread_page       notify_on_release
cpuset.cpu_exclusive   cpuset.mem_hardwall    cpuset.memory_spread_slab       release_agent
cpuset.cpus            cpuset.memory_migrate  cpuset.mems                     tasks
```

주요 I/F는 아래와 같다.

| I/F                  | Description                                                               |
|----------------------|---------------------------------------------------------------------------|
| cpuset.cpus	         | process를 실행하는 CPU core 지정.                                         |
| cpuset.cpu_exclusive | 1로 set되면 이 group이 지정한 cpu cores는 다른 group에서 지정하지 못한다. |
| cpuset.mems	         | NUMA architecture의 process가 이용하는 memory node를 지정.                |

stress tool을 사용해서 비교 해본다.

```bash
$ unshare /bin/sh

$ pstree -p
systemd(1)-+-agetty(194)
           |-agetty(195)
           |-avahi-daemon(262)---avahi-daemon(263)
           |-dbus-daemon(190)
           |-docker(198)-+-{docker}(200)
           |             |-{docker}(201)
           |             |-{docker}(202)
           |             |-{docker}(204)
           |             `-{docker}(206)
           |-haveged(193)
           |-nmbd(207)
           |-smbd(256)---smbd(260)
           |-sshd(199)-+-sshd(266)---bash(273)
           |           `-sshd(285)---bash(287)---sh(332)---pstree(344)
           |-systemd(269)---(sd-pam)(270)
           |-systemd-journal(115)
           |-systemd-logind(189)
           |-systemd-resolve(197)
           |-systemd-timesyn(182)---{sd-resolve}(184)
           `-systemd-udevd(141)
  
$ stress -m 3 --vm-bytes 128m -t 60s

$ top
```

4개 core를 모두 100% 사용함을 확인할 수 있다.
이제 cgroup으로 제약을 가한다.

```bash
$ cd /sys/fs/cgroup/cpuset/

$ mkdir test/

$ cd test/

$ echo 0 > cpuset.cpus

$ echo 0 > cpuset.mems

$ echo 332 > tasks
```

같은 test를 반복한다.

```bash
$ stress -m 3 --vmbytes 128m -t 60s
$ top
```

0번 core만 사용됨을 알 수 있다.

#### memory

process 별 memory 사용에 대한 제약을 가한다. \
memory가 부족할 경우 적용할 사항을 oom_control을 통해 조정해 줄 수 있다. \

```bash
$ ls /sys/fs/cgroup/memory
cgroup.clone_children           memory.kmem.tcp.max_usage_in_bytes  memory.pressure_level
cgroup.event_control            memory.kmem.tcp.usage_in_bytes      memory.soft_limit_in_bytes
cgroup.procs                    memory.kmem.usage_in_bytes          memory.stat
cgroup.sane_behavior            memory.limit_in_bytes               memory.swappiness
memory.failcnt                  memory.max_usage_in_bytes           memory.usage_in_bytes
memory.force_empty              memory.memsw.failcnt                memory.use_hierarchy
memory.kmem.failcnt             memory.memsw.limit_in_bytes         notify_on_release
memory.kmem.limit_in_bytes      memory.memsw.max_usage_in_bytes     release_agent
memory.kmem.max_usage_in_bytes  memory.memsw.usage_in_bytes         system.slice
memory.kmem.slabinfo            memory.move_charge_at_immigrate     tasks
memory.kmem.tcp.failcnt         memory.numa_stat                    user.slice
memory.kmem.tcp.limit_in_bytes  memory.oom_control
```

주요 I/F는 아래와 같다.

| I/F                         | Description                                                                                         |
|-----------------------------|-----------------------------------------------------------------------------------------------------|
| memory.limit_in_bytes	      | 해당 group을 이용할 수 있는 물리 memory의 상한(byte)을 지정                                         |
| memory.memsw.limit_in_bytes	| 해당 group을 이용할 수 있는 '물리 memory + swap영역'의 상한(byte)을 지정                            |
| memory.use_hierachy	        | 1이 set되면 해당 group의 memory 사용량에 sub group의 process memory 사용량도 추가된다(default는 0). |

```bash
$ cd /sys/fs/cgroup/memory $ mkdir group1 $ echo 128M > group1/memory.limit_in_bytes $ echo $$ > group1/tasks $ stress --vm 1 --vm-bytes 127M --timeout 60s $ stress --vm 1 --vm-bytes 128M --timeout 60s $ echo 1 > group1/memory.oom_control $ echo 2G > group1/memory.limit_in_bytes $ stress --vm 1 --vm-bytes 512M --timeout 60s $ stress --vm 1 --vm-bytes 1G --timeout 60s $ echo 0 > group1/memory.oom_control $ stress --vm 1 --vm-bytes 512M --timeout 60s $ stress --vm 1 --vm-bytes 1G --timeout 60s
```

#### device
device file의 사용에 대한 제한을 가한다. 
3가지 file이 있다.
* devices.allow : whitelist
* devices.deny : blacklist
* devices.list : 사용가능한 device

각각에 대해서 4가지 filed가 있다.

* Type :
  * a : all
  * c : char device
  * b : block device
* Major number
* Minor number
* Access :
  * r : read
  * w : write
  * m : mknode

/dev/null 의 경우 major는 1 minor는 3이다.

```bash
$ mkdir /sys/fs/cgroup/devices/0

$ cat /sys/fs/cgroup/devices/0/devices.list
a ":" rwm

$ echo "a *:* rmw" > /sys/fs/cgroup/devices/0/devices.deny

$ echo $$ > /sys/fs/cgroup/devices/0/tasks

$ echo "test > /dev/null
bash: /dev/null: Operation not permitted

$ echo "a *.* rwm" > /sys/fs/cgroup/devices/0/devices.allow

$ echo "test" > /dev/null
blkio

$ ls /sys/fs/cgroup
blkio.io_merged            blkio.io_service_bytes_recursive  blkio.io_wait_time            blkio.sectors                    blkio.throttle.read_iops_device   blkio.weight           notify_on_release
blkio.io_merged_recursive  blkio.io_service_time             blkio.io_wait_time_recursive  blkio.sectors_recursive          blkio.throttle.write_bps_device   blkio.weight_device    release_agent
blkio.io_queued            blkio.io_service_time_recursive   blkio.leaf_weight             blkio.throttle.io_service_bytes  blkio.throttle.write_iops_device  cgroup.clone_children  system.slice
blkio.io_queued_recursive  blkio.io_serviced                 blkio.leaf_weight_device      blkio.throttle.io_serviced       blkio.time                        cgroup.procs           tasks
blkio.io_service_bytes     blkio.io_serviced_recursive       blkio.reset_stats             blkio.throttle.read_bps_device   blkio.time_recursive              cgroup.sane_behavior   user.slice
```

주요 I/F는 아래와 같다.

| I/F                              |	Description                                                                    |
|----------------------------------|---------------------------------------------------------------------------------|
| blkio.weight                     |	모든 block device에 공통적인 우선순위를 100~1000의 값으로 지정(default는 500). |
| blkio.weight_device	             | 특정 device에 대한 우선순위를 지정.                                             |
| blkio.throttle.read_bps_device	 | 특정 device에 대한 접근 속도의 상한을 Bytes/Sec 단위로 지정.                    |
| blkio.throttle.write_bps_device  |                                                                                 |
| blkio.throttle.read_iops_device	 | 특정 dsevice에 대한 접근 속도의 상한을 IOPS 단위로 지정.                        |
| blkio.throttle.write_iops_device |                                                                                 |

## Reference
https://www.linuxfoundation.jp/jp_uploads/seminar20081119/CgroupMemcgMaster.pdf \
https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/sec-memory.html \
