+++
title  = "inside"
toc    = true
weight = 4
+++

## Linux Features
* [namespace](/system/namespace)
* [cgroups](/system/cgroup)
* chroot
* pivot_root
* union mount
* bind mount
* iptables
* veth
* capabilities
* seccomp
* LSM
  * SELinux
  * AppArmor

Linux에서 Container를 구성하는 주요 Kerenl Feature는 위와 같다.

## Simple Container
아래와 같이 간단한 container를 만들어 볼 수 있다.
유사한 project로 [bocker](https://github.com/p8952/bocker)가 있다.
```c
#define _GNU_SOURCE
#include <sys/types.h>
#include <sys/wait.h>
#include <sys/mount.h>
#include <stdio.h>
#include <stdlib.h>
#include <sched.h>
#include <signal.h>
#include <unistd.h>

#define ChkErr(expr) if(expr!=0) { line=__LINE__; ret=expr; goto ErrorExit; }
#define STACK_SIZE (1024 * 1024)

static char child_stack[STACK_SIZE];
char* const child_args[] = {
  "/bin/sh",
  NULL
};

int checkpoint[2];
int ret;
int line;

void set_map(char* file, int inside_id, int outside_id, int len)
{
  FILE* mapfd = fopen(file, "w");
  if(NULL==mapfd) {
    perror("open file error");
    return;
  }
  fprintf(mapfd, "%d %d %d", inside_id, outside_id, len);
  fclose(mapfd);
}

void set_uid_map(pid_t pid, int inside_id, int outside_id, int len)
{
  char file[256];
  sprintf(file, "/proc/%d/uid_map", pid);
  set_map(file, inside_id, outside_id, len);
}

void set_gid_map(pid_t pid, int inside_id, int outside_id, int len)
{
  char file[256];
  sprintf(file, "/proc/%d/gid_map", pid);
  set_map(file, inside_id, outside_id, len);
}

int child_main(void* arg)
{
  char c;

  close(checkpoint[1]);

  printf(" - World !\n");
  sethostname("In Namespace", 12);

  read(checkpoint[0], &c, 1);

  chroot("/home/keyolk/study/container/root");
  chdir("/");
  setenv("PATH", "/bin", 1);

  mount("proc", "/proc", "proc", 0, NULL);
  system("ip link set veth1 up");
  system("ip addr add 169.254.1.2/30 dev veth1");

  execv(child_args[0], child_args);

  printf("Ooops!\n");
  return 1;
}

int main()
{
  char* cmd;
  const int gid=getgid(), uid=getuid();
  pipe(checkpoint);

  printf("Parent: eUID = %ld;  eGID = %ld, UID=%ld, GID=%ld\n",
      (long) geteuid(),
      (long) getegid(),
      (long) getuid(),
      (long) getgid());

  printf(" - [%d] Hello ?\n", getpid());

  int child_pid = clone(child_main, child_stack+STACK_SIZE,
      CLONE_NEWUTS |
      CLONE_NEWIPC |
      CLONE_NEWPID |
      CLONE_NEWNS  |
      CLONE_NEWNET |
      SIGCHLD, NULL);

  if (child_pid == -1) {
    printf("Failed to clone process\n");
    return 1;
  }

  printf("Parent %d, Container %d\n", getpid(), child_pid);

  set_uid_map(child_pid, 0, 1000, 1);
  set_gid_map(child_pid, 0, 1000, 1);

  system("ip link add veth0 type veth peer name veth1");
  asprintf(&cmd, "ip link set veth1 netns %d", child_pid);
  system(cmd);
  system("ip link set veth0 up");
  system("ip addr add 169.254.1.1/30 dev veth0");
  free(cmd);

  close(checkpoint[1]);
  waitpid(child_pid, NULL, 0);
  system("ip link delete veth0");
  printf("Parent: Container stopped!\n");

  return 0;
}
```
