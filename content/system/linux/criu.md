+++
title  = "criu"
toc    = true
weight = 1
+++

Checkpoint & Restore in Userspace

## Usage
```
$ criu dump -t ${PID} -vvv -o dump.log && echo OK
$ criu restore -d -vvv -o restore.log && echo OK
$ criu dump -vvvv -o dump.log -t ${PID --shell-job && echo OK
$ criu restore -vvvv -o restore.log --shell-job && echo OK
```

## Details
* ptrace
* mmap
* parasite code injection
* TCP repair mode

## Dump
* Stop the tasks
  - Freez by PTRACE_SEIZE or cgroup freezer
  - Lock network
* Collect process information
  - proc files
  - parasite injection
    * Credential, memory contents, signals
* Dump pages
  * memory pages copied via vmsplice and splice syscalls

## Restore
* tree

## Issues
* External Resources
  - Unix socket
  - TCP
  - Shell Jobs
  - File locks
  - Bind mounts
* Device Accesses
  - /dev/null, /dev/net/tun
  - X applications
* ETC
  - File System
  - SysV IPC
  - Nested namespace or cgroup

## Reference
* https://media.ccc.de/v/896-exploring-criu
