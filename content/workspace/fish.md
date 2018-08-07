+++
title  = 'fish'
toc    = true
weight = 1
+++

## slow startup

보통 set -x를 잘못써서 특정 global variable의 길이가 너무 길어지는게 문제가 된다.

"fish_user_paths" 길이를 체크해볼 필요가 있다.

```
$ set -U # 확인 후
$ set -e fish_user_paths # 필요 시 삭제
```
