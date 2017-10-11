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

#### RAP

libc version이 낮은 system에선 prefix가 정상적으로 bootstrap 되지 않는다.
RAP script를 사용하면 최신 libc부터 시작하여 prefix를 bootstrap 해준다.

```bashl
$ wget http://dev.gentoo.org/~heroxbd/bootstrap-rap.sh
$ chmod +x bootstrap-rap.sh
$ ./bootstrap-rap.sh
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
$ emerge --ask --update --newuse sys-apps/portage
```

fish, tmux, neovim 설치

```bash
$ emerge --ask app-shells/fish
$ emerge --ask app-misc/tmux
$ env USE="-clipboard" emerge --ask app-editors/neovim
$ emerge --ask dev-python/pip
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
