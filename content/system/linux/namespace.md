+++
title  = "namespace"
toc    = true
weight = 0
+++

 Namespace를 통해 Kernel 내 Global Resource에 대해 Process 별로 Partitioning을 제공한다. \
 기본적으로 Parent Process의 Namespace를 Child 에서 상속받는다.

| Namespace | Constant        | Related Resource                 | Supporting Version |
|-----------|-----------------|----------------------------------|--------------------|
| Mount     | CLONE_NEWNS     | Mount Points                     | 2.4.19             |
| UTS       | CLONE_NEWUTS    | Hostname, NIS domain name        | 2.6.24             |
| IPC       | CLONE_NEWIPC    | SystemV IPC, POSIX Message Queue | 2.6.24             |
| PID       | CLONE_NEWPID    | Process IDs                      | 2.6.24             |
| Net       | CLONE_NEWNET    | Network Stacks                   | 2.6.29             |
| User      | CLONE_NEWUSER   | Network Stacks                   | 3.8                |
| CGroup    | CLONE_NEWCGROUP | Network Stacks                   | 4.6                |

아래 System Call을 통해 Namespace를 다룰 수 있다.

* clone()
* unshare()
* setns()

User를 제외한 5가지 Namespace에 대해선 CAP_SYS_ADMIN capabilities가 필요하다. \
모든 Process는 procfs에서 아래와 같이 namespace 정보를 inode 형태로 가지고 있다. \
같은 namespace에 속한 process는 같은 inode 값을 갖는다.

```bash
$ ls -l /proc/$$/ns
total 0
lrwxrwxrwx. 1 mtk mtk 0 Jan 14 01:20 ipc -> ipc:[4026531839]
lrwxrwxrwx. 1 mtk mtk 0 Jan 14 01:20 mnt -> mnt:[4026531840]
lrwxrwxrwx. 1 mtk mtk 0 Jan 14 01:20 net -> net:[4026531956]
lrwxrwxrwx. 1 mtk mtk 0 Jan 14 01:20 pid -> pid:[4026531836]
lrwxrwxrwx. 1 mtk mtk 0 Jan 14 01:20 user -> user:[4026531837]
lrwxrwxrwx. 1 mtk mtk 0 Jan 14 01:20 uts -> uts:[4026531838]
```

위 file에 대한 bind mount가 이뤄지면, \
namespace 내에 속한 모든 process가 terminated 된다고 해도 namespace가 사라지지 않는다. \
이 때 해당하는 file descriptor를 setns()를 통해 다른 process로 전달하면 \
해당 process를 주어진 namespace에 할당해 줄 수 있다.

Kernel이 아래 configuration을 통해 namespace가 지원된다.

* CONFIG_NAMESPACES
* CONFIG_UTS_NS
* CONFIG_IPC_NS
* CONFIG_PID_NS
* CONFIG_NET_NS
* CONFIG_USER_NS
* CONFIG_CGROUP_NS

task가 속한 namespace에 대한 접근은 \
task_struct의 filed인 nsproxy를 통해서 이뤄진다. 

struct task_struct : http://lxr.free-electrons.com/source/include/linux/sched.h?v=4.0#L1278 \
struct nsproxy *nsproxy : http://lxr.free-electrons.com/source/include/linux/sched.h?v=4.0#L1467 

nsproxy 구조체 내에 usernamespace를 제외한 5가지 namespace에 대한 pointer가 존재한다. \
struct ns_proxy : http://lxr.free-electrons.com/source/include/linux/nsproxy.h?v=4.0#L29 

usernamespace 는 task의 filed 인 cred 내에서 찾아볼 수 있다. 

struct cred : http://lxr.free-electrons.com/source/include/linux/cred.h?v=4.0#L103 struct user_namespace *user_ns : http://lxr.free-electrons.com/source/include/linux/cred.h?v=4.0#L137 \

아래에서 namespace를 만드는 function을 확인할 수 있다. 
create_new_namespaces() : http://lxr.free-electrons.com/source/kernel/nsproxy.c?v=4.0#L59 \

task는 따로 명시되지 않아도 default로 global namespace에 속해있다. 

struct nsproxy init_nsproxy : http://lxr.free-electrons.com/source/kernel/nsproxy.c?v=4.0#L31 \
do_fork() : http://lxr.free-electrons.com/source/kernel/fork.c?v=4.0#L1631 \
p = copy_process() : http://lxr.free-electrons.com/source/kernel/fork.c?v=4.0#L1659 \
copy_process() : http://lxr.free-electrons.com/source/kernel/fork.c?v=4.0#L1190 \
retval = copy_namespaces() : http://lxr.free-electrons.com/source/kernel/fork.c?v=4.0#L1398 \
copy_namesapces() : http://lxr.free-electrons.com/source/include/linux/nsproxy.h?v=4.0#L65 \
copy_namesapces() : http://lxr.free-electrons.com/source/kernel/nsproxy.c?v=4.0#L124 \

## Mount
Mount Point에 대한 isolation을 제공한다. 

새로 mount namspace에선 생성 시점 이전에 생성된 mount들이 모두 visible하다. 
새로 생성된 mount namspace 내에 mount와 unmount는 다른 system에서 invisible하다. 
rootfs 를 다시 mount 하여 chroot와 비슷한 효과를 가져올 수 있다.

![alt tag](https://uploads.toptal.io/blog/image/677/toptal-blog-image-1416545619045.png)

#### User View
```bash
root$ mkdir test

root$ mount -t tmpfs none test

root$ mount | grep test
none /root/test/test tmpfs rw,relatime 0 0
  
root$ unshare -pmrf /bin/sh
unshared$ mkdir test2
unshared$ mount -t tmpfs none test2
unshared$ cat /proc/mounts | grep test
none /root/test/test tmpfs rw,relatime 0 0
none /root/test/test2 tmpfs rw,nodev,relatime 0 0
  
root$ cat /proc/mounts | grep test
none /root/test/test tmpfs rw,relatime 0 0
```

기본적으로 parent에서 가지고 있는 mount table은 child로 상속되지만, \
아래와 같은 mount flag를 통해서 이를 제어할 수 있다.

* --make-rprivate
* --make-rshared

기존 mount entry에 대한 shared flag 여부는 아래와 같이 알아 볼 수 있다.

```bash
$ cat /proc/self/mountinfo | grep shared
...
33 1 8:3 / / rw,relatime shared:1 - ext4 /dev/sda3 rw,data=ordered
...

```

shared subtree는 2005년에 Ram Pai를 통해 patch가 만들어졌다. \
mount에 아래와 같은 flag가 추가되었으며, 

* –make-slave
* –make-rslave
* –make-unbindable
* –make-runbindable
* ...

마찬가지로 kernel에 아래와 같은 이름으로 flag가 추가되었다. 

* MS_UNBINDABLE
* MS_PRIVATE
* MS_SLAVE
* MS_SHARED

#### Kernel View
관련 자료구조는 아래와 같다. 

struct mnt_namespace : http://lxr.free-electrons.com/source/fs/mount.h?v=4.0#L7 \
copy_mnt_ns() : http://lxr.free-electrons.com/source/fs/namespace.c?v=4.0#L2681 \
new = copy_tree() : http://lxr.free-electrons.com/source/fs/namespace.c?v=4.0#L2709 \
copy_tree() : http://lxr.free-electrons.com/source/fs/namespace.c?v=4.0#L1590 

## UTS
Hostname과 NIS domain name 에 대한 isolation을 제공한다. \
sethostname(), setdomainname() 으로 해당 값을 설정할 수 있으며, \
uname(), gethostname(), getdomainname() 으로 현재 할당된 값을 확인할 수 있다.

#### User View
```bash
root$ uname -n
old

root$ unshare -u /bin/bash

unshared$ hostname new

unshared$ uname -n
new

unshared$ exit

root$ uname -n
old
```

#### Kernel View
관련 자료구조는 아래와 같다. 

http://lxr.free-electrons.com/source/include/linux/utsname.h?v=4.0#L23 \
http://lxr.free-electrons.com/source/include/uapi/linux/utsname.h?v=4.0#L24 \

기존 gethostname 구현, \
http://lxr.free-electrons.com/source/kernel/sys.c?v=2.4.37#L1056 \
여기서 system_utsname 이 global value 이다.

utsname 위한 새 function 추가, \
http://lxr.free-electrons.com/source/include/linux/utsname.h?v=4.0#L72 \

새로운 gethostname() 구현, \
http://lxr.free-electrons.com/source/kernel/sys.c?v=4.0#L1239 \

비슷한 구현이 uname(), sethostname()에 적용되었다.
uname : http://lxr.free-electrons.com/source/kernel/sys.c?v=4.0#L1139 \
sethostname : http://lxr.free-electrons.com/source/kernel/sys.c?v=4.0#L1213 

## IPC
IPC resource(System V IPC와 POSIX message queue)에 대한 isolation을 제공한다. \
IPC namespace는 SYSTEM V IPC identifier와 POSIX message queue filesystem을 들을 갖고 있다. \
위 IPC는 kernel내 아래 2개 configuration으로 정의된다. \

* CONFIG_POSIX_MQUEUE
* CONFIG_SYSVIPC

IPC namespace가 destroyed 되면, \
속한 모든 IPC object는 자동적으로 destroyed된다.

격리되는 IPC 자원들은 procfs에서 확인할 수 있다. \
아래 procfs의 interface들은 IPC namespace별로 구별된다.

* /proc/sys/fs/mqueue/ : POSIX message queue
  * msg_default
  * msg_max
  * msgsize_default
  * queues_max
* /proc/sys/kernel/ : System V IPC
  * msgmax
  * msgmnb
  * msgmni
  * sem
  * shmall
  * shmmax
  * shmmni
  * shm_rmid_forced
* /proc/sysvipc/ : System V IPC
  * msb
  * sem
  * shm

```bash
# ipcmk -Q
Message queue id: 0
  
# ipcs -q
  
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
0xd2489e5b 65536      root       644        0            0
  
# unshare -if /bin/sh
# ipcs -q
  
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages
```

#### Kernel View
관련 자료구조는 아래와 같다. 

http://lxr.free-electrons.com/source/include/linux/ipc_namespace.h?v=4.0#L21

## PID
PID에 대한 isolation을 제공한다. 

* 새로운 namespace에서 생성된 최초의 task는 pid가 1이 된다.
* parent namespace에서 자식의 모든 pid를 볼 수 있다.
* Init process와 유사한 동작을 보인다.
  * child reaping : 한 process가 죽으면 모든 children 은 부모가 PID 1번이 된다.
    * 동일 namespace의 다른 process가 prctl()을 통해 PR_SET_CHILD_SUBREAPER 설정이 되어있지 않을 때에 한한다.
  * SIGKILL signal은 PID 1번 process를 죽일 수 없다.
* PID namespace 내의 init process가 terminated 되면 해당 namespace 내의 모든 process가 종료된다.
* 32회까지 nested 될 수 있다. 

![alt tag](https://uploads.toptal.io/blog/image/674/toptal-blog-image-1416487554032.png)

#### User View
```bash
$ pstree -p
systemd(1)-+-agetty(200)
           |-agetty(201)
           |-avahi-daemon(274)---avahi-daemon(276)
           |-dbus-daemon(197)
           |-docker(202)-+-{docker}(207)
           |             |-{docker}(208)
           |             |-{docker}(209)
           |             |-{docker}(210)
           |             |-{docker}(216)
           |             |-{docker}(277)
           |             |-{docker}(278)
           |             |-{docker}(279)
           |             |-{docker}(280)
           |             |-{docker}(281)
           |             |-{docker}(282)
           |             `-{docker}(283)
           |-haveged(194)
           |-nmbd(212)
           |-smbd(236)-+-smbd(272)
           |           `-smbd(15603)
           |-sshd(204)---sshd(5079)---bash(5085)---pstree(5088)
           |-systemd(5081)---(sd-pam)(5082)
           |-systemd-journal(119)
           |-systemd-logind(196)
           |-systemd-resolve(206)
           |-systemd-timesyn(189)---{sd-resolve}(191)
           `-systemd-udevd(140)
  
# unshare -pf --mount-proc /bin/sh
$ pstree -p
sh(1)---pstree(2)
```

#### Kernel View
관련 자료구조는 아래와 같다. 

struct pid_namespace : http://lxr.free-electrons.com/source/include/linux/pid_namespace.h?v=4.0#L24 \
MAX_PID_NS_LEVEL : http://lxr.free-electrons.com/source/kernel/pid_namespace.c?v=4.0#L80 \
struct upid : http://lxr.free-electrons.com/source/include/linux/pid.h?v=4.1#L50 \
struct pid : http://lxr.free-electrons.com/source/include/linux/pid.h?v=4.1#L57 \

특정 process를 kill하는 scenario를 생각해보자, \ 
signal을 통해서 결과적으로 kernel 내에 아래 function을 부르게 될것이다. \

kill_something_info() : http://lxr.free-electrons.com/source/kernel/signal.c?v=4.1#L1425  \
이 function 내에는 아래와 같은 subroutine이 있다. \
kill_pid_info() : http://lxr.free-electrons.com/source/kernel/signal.c?v=4.1#L1431 \
kill_pid_info() : http://lxr.free-electrons.com/source/kernel/signal.c?v=4.1#L1339 \
find_vpid() : http://lxr.free-electrons.com/source/kernel/pid.c#L380?v=4.1#L380 \
find_pid_ns() : find_vpid() : http://lxr.free-electrons.com/source/kernel/pid.c#L380?v=4.1#L382 \
param인 *pid로 find_vpid()를 통해 struct pid를 가져온다.

## Net
Network Stack에 대한 Isolation을 제공한다.

* 새로 생성된 namespace 엔 loopback 만 존재한다.
* 다른 namespace의 device를 가져올 수 있다.
* socket과 device는 하나의 namespace에만 속할 수 있다.
* namespace간의 통신은 veth를 통해 이뤄질 수 있다.

![alt tag](https://uploads.toptal.io/blog/image/675/toptal-blog-image-1416487605202.png)

#### User View
```bash
# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether b8:27:eb:43:56:26 brd ff:ff:ff:ff:ff:ff
    inet 192.168.137.2/24 brd 192.168.137.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::ba27:ebff:fe43:5626/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.42.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::fc45:45ff:fec7:1756/64 scope link
       valid_lft forever preferred_lft forever
  
# unshare -nf /bin/sh
$ ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

2개의 namespace를 만든다. 
```bash
$ ip netns add myns1
$ ip netns add myns2
```
myns1 namespace를 지운다.

```bash
$ ip netns del myns1
```
namespace를 지우면 모든 migratable한 network device 들은 default network namespace로 할당된다. \
아래 명령을 사용하여 network namespace를 monitoring 할 수 있다.

```bash
$ ip netns monitor
```
아래 명령을 통해 현재 존재하는 network namespace를 list up 할 수 있다.

```bash
$ ip netns list
```
위 명령은 /var/run/netns/ 를 읽게끔 되어있다.
특정 namespace에 속한 process의 id를 아래 명령으로 list 할 수 있다.

```bash
$ ip netns pid [namespace_name]
```
마찬가지로 특정 pid가 속한 namespace도 알 수 있다.
```bash
$ ip netns identify [pid]
```
특정 network interface를 namespace로 할당해 줄 수 있다.

```bash
$ ip link set eth0 netns myns1
```
dev_change_net_namespace() : http://lxr.free-electrons.com/source/include/linux/netdevice.h?v=4.0#L2971 \
shell을 실행하면서 새로운 namespace에 할당해 줄 수 있다.

```bash
$ ip netns exec myns1 bash
```
namespace 내에서 ifconfig -a로 검사하면 eth0과 loopback만 존재하는걸 확인할 수 있다.
network device에 대해서 inital namespace 설정을 persist 하게 수정할 수 있다.
```bash
$ ip link set eth0 netns1
```

#### Kernel View
관련 자료 구조는 아래와 같다. 

struct net : http://lxr.free-electrons.com/source/include/net/net_namespace.h?v=4.0#L44 \
network stack 전체가 담겨있다.

* looback device
* SNMP stat (netns_mib)
* all network tables : routing, neighboring, etc …
* all sockets
* /procfs entries
* /sysfs entries

하나의 network device는 반드시 하나의 network namspace에 속한다. \
struct net_device : http://lxr.free-electrons.com/source/include/linux/netdevice.h?v=4.0#L1499 \
http://lxr.free-electrons.com/source/include/linux/netdevice.h?v=4.0#L1706 \

related function :

 * dev_net() : http://lxr.free-electrons.com/source/include/linux/netdevice.h?v=4.0#L1839

하나의 socket은 반드시 하나의 network namspace에 속한다. \
struct sock : http://lxr.free-electrons.com/source/include/net/sock.h?v=4.0#L301 \
http://lxr.free-electrons.com/source/include/net/sock.h?v=4.0#L306 \
struct sock_common : http://lxr.free-electrons.com/source/include/net/sock.h?v=4.0#L158 \
http://lxr.free-electrons.com/source/include/net/sock.h?v=4.0#L194 \

related function : 

* sock_net() : http://lxr.free-electrons.com/source/include/net/sock.h?v=4.0#L2162
* sock_net_set() : http://lxr.free-electrons.com/source/include/net/sock.h?v=4.0#L2168

system에 존재하는 모든 network namspaces를 담는 list가 추가되었다. \
http://lxr.free-electrons.com/source/include/net/net_namespace.h#L160 \
struct list_head net_namespace_list : http://lxr.free-electrons.com/source/net/core/net_namespace.c?v=4.0#L33 \
http://lxr.free-electrons.com/source/net/core/net_namespace.c?v=4.0#L34 

해당 list를 traverse 하는 macro도 추가되었다. \
for_each_net() : http://lxr.free-electrons.com/source/include/net/net_namespace.h?v=4.0#L276 \
initial network namesapce인 init_net은 loopback device와 모든 physical device, networking table 등을 갖고 있다. \
http://lxr.free-electrons.com/source/include/net/net_namespace.h?v=4.0#L140 \
struct net init_net : http://lxr.free-electrons.com/source/net/core/net_namespace.c?v=4.0#L36 \
init_net 또한 struct net의 instance이다. \
새로 생성되는 각각의 network namespace는 오직 loopback device 만을 가지고 있다. \
socket은 가지고 있지 않다.

## CGroup

## Reference

http://www.toptal.com/linux/separation-anxiety-isolating-your-system-with-linux-namespaces \
https://blog.jtlebi.fr/2013/12/22/introduction-to-linux-namespaces-part-1-uts/ \
http://crosbymichael.com/creating-containers-part-1.html \
http://www.lizhaozhong.info/archives/tag/namespace \
http://www.cnblogs.com/blueswu/p/3566307.html 
