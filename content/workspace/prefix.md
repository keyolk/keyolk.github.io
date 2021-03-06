+++
title  = "prefix"
toc    = true
weight = 0
tags = [ "terminal" ]
+++

## Intro

Root 권한이 없는 System에서 
개인 workspace를 만들기위해 필요한 package를 설치하기위해 
[Gentoo Prefix](https://wiki.gentoo.org/wiki/Project:Prefix)를 사용.

관련 Step 정리.

```
$ wget https://gitweb.gentoo.org/repo/proj/prefix.git/plain/scripts/bootstrap-prefix.sh
$ chmod +x bootstrap-prefix.sh
$ ./bootstrap-prefix.sh
```

#### RAP

libc version이 낮은 system에선 prefix가 정상적으로 bootstrap 되지 않는다.
RAP script를 사용하면 최신 libc부터 시작하여 prefix를 bootstrap 해준다.

```bashl
$ wget http://dev.gentoo.org/~heroxbd/bootstrap-rap.sh
$ chmod +x bootstrap-rap.sh
$ ./bootstrap-rap.sh
```

bash version이 낮은 경우 bash 부터 bootstrap 해야한다.
https://wiki.gentoo.org/wiki/Project:Prefix/Bootstrap

```
$ wget https://gitweb.gentoo.org/repo/proj/prefix.git/plain/scripts/bootstrap-bash.sh
$ chmod +x bootstrap-bash.sh
$ mkdir -p /tmp/bash
$ ./bootstrap-bash /tmp/bash
```

#### Initialize

bootstrap이 완료되면 startprefix script 가 생성된다.
```bash
$ ./startprefix
```

이때 부터 emerge 등 gentoo의 package system CLI를 사용할 수 있다.

필요한 package를 아래와 같이 설치한다.


```bash
$ emerge --sync
$ etc-update
$ emerge --update --newuse sys-apps/portage
```

fish, tmux, neovim 설치

```bash
$ emerge app-shells/fish
$ emerge app-misc/tmux
$ env USE="-clipboard" emerge app-editors/neovim
$ emerge dev-python/pip
$ emerge ripgrep
```

python은 아래와 같이 사용할 수 있다.

```bash
$ eselect python list
Available Python interpreters, in order of preference:
  [1]   python2.7
  [2]   python3.4
$ esectct python set 1
$ pip installn neovim
$ esectct python set 2
$ pip installn neovim
```

#### Reference
- https://wiki.gentoo.org/wiki/Project:Prefix
- https://wiki.gentoo.org/wiki/Prefix/libc
