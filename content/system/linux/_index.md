+++
title  = "linux"
toc    = true
weight = 0
+++

Linux 관련 feature에 대한 이해를 위한 정리.
Filesystem, Resource Isolation, Event Trigger, Module 등.

- [namespace](/system/namespace)
- [cgroup](/system/cgroup)
- capabilities
- mounts
- fuse
- inotify
- epoll
- pipe
- systemd
- rootfs
- chroot
- lkm
- device
- environment
- keyring
- ipc
- cmdline
- zone
- id


## STDIO buffering
아래에 이와 관련된 문제에 대해 상세히 기술되어 있다.

http://www.pixelbeat.org/programming/stdio_buffering/

어떻게 이 현상을 fix 할 것인지.

https://www.perkin.org.uk/posts/how-to-fix-stdio-buffering.html

## Redirect Outputs of Running Process
아래와 같은 상황을 가정하자.
```bash
$ hugo server -v
```

잘동작하는걸 확인했으므로 Background로 돌린다.

<kbd>ctrl</kbd>+<kbd>z</kbd>

```bash
$ bg
```
```bash
[1]  + 23106 continued  hugo server -v
```
이 상태에선 hugo server가 message를 만들면 그대로 terminal에 나타나게 된다.

```bash
$ ls -l /proc/23016/fd
```
```
total 0
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 0 -> /dev/pts/1
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 1 -> /dev/pts/1
lr-x------ 1 keyolk keyolk 64 Jan 16 10:59 10 -> pipe:[20424904]
l-wx------ 1 keyolk keyolk 64 Jan 16 10:59 11 -> pipe:[20424904]
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 12 -> socket:[20424905]
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 2 -> /dev/pts/1
lr-x------ 1 keyolk keyolk 64 Jan 16 10:59 3 -> anon_inode:inotify
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 4 -> anon_inode:[eventpoll]
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 5 -> anon_inode:[eventpoll]
lr-x------ 1 keyolk keyolk 64 Jan 16 10:59 6 -> pipe:[20421334]
l-wx------ 1 keyolk keyolk 64 Jan 16 10:59 7 -> pipe:[20421334]
lr-x------ 1 keyolk keyolk 64 Jan 16 10:59 8 -> anon_inode:inotify
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 9 -> anon_inode:[eventpoll]
```

```bash
$ sudo gdb -p 20106
```
```gdb
(gdb) p dup2(open("/dev/null", 0), 1)
$1 = 1
(gdb) p dup2(open("/dev/null", 0), 2)
$2 = 2
(gdb) detach
(gdb) quit
```

확인해 본다.
```bash
$ ls -l /proc/23016/fd
```
```bash
total 0
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 0 -> /dev/pts/1
lr-x------ 1 keyolk keyolk 64 Jan 16 10:59 1 -> /dev/null
lr-x------ 1 keyolk keyolk 64 Jan 16 10:59 10 -> pipe:[20424904]
l-wx------ 1 keyolk keyolk 64 Jan 16 10:59 11 -> pipe:[20424904]
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 12 -> socket:[20424905]
lr-x------ 1 keyolk keyolk 64 Jan 16 11:03 13 -> /dev/null
lr-x------ 1 keyolk keyolk 64 Jan 16 11:03 14 -> /dev/null
lr-x------ 1 keyolk keyolk 64 Jan 16 10:59 2 -> /dev/null
lr-x------ 1 keyolk keyolk 64 Jan 16 10:59 3 -> anon_inode:inotify
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 4 -> anon_inode:[eventpoll]
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 5 -> anon_inode:[eventpoll]
lr-x------ 1 keyolk keyolk 64 Jan 16 10:59 6 -> pipe:[20421334]
l-wx------ 1 keyolk keyolk 64 Jan 16 10:59 7 -> pipe:[20421334]
lr-x------ 1 keyolk keyolk 64 Jan 16 10:59 8 -> anon_inode:inotify
lrwx------ 1 keyolk keyolk 64 Jan 16 10:59 9 -> anon_inode:[eventpoll]

```

반대로 background process의 output을 보이게 할수도 있다.
```bash
$ hugo server &> /dev/null &
```
```bash
[1] 31130
```
```bash
$ sudo gdb -p 31130
```
```gdb
(gdb) p dup2(open("/dev/pts/1", 1), 1)
$1 = 1
(gdb) p dup2(open("/dev/pts/1", 1), 2)
$2 = 2
(gdb) detach
(gdb) quit
```

이 시점에서 output이 terminal로 들어온다.
