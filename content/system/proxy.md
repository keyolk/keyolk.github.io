+++
title  = "proxy"
toc    = true
weight = 0
+++

## Proxy Auto Config
대부분의 Browser에서 지원한다.
javascript syntax를 사용한다.

CLI에서 지원을 잘 안하는것 같다.

* libproxy가 이를 지원하기위해 개발되고 있다.
https://libproxy.github.io/libproxy

* pacparser를 사용하면 pac으로 부터 URL에 해당되는 Proxy 정보를 가져올 수 있다.
https://github.com/pacparser/pacparser

## SSH Proxy
* OpenSSH를 활용하여 간단한 SOCKS proxy를 만들 수 있다.
https://keyolk.github.io/2016/06/30/Open-S-S-H.html

* SOCKS proxy를 사용하지 못하는 application에서는 tsocks나 proxychains, sshuttle을 사용한다.

## Utilities
* [tsocks](http://tsocks.sourceforge.net)를 사용하면 SOCKS proxy를 지원하지 않는 application에서도 이를 사용할 수 있게 할 수 있다.
* [proxychains](https://github.com/haad/proxychains)를 사용하면 application별로 proxy를 할당해 줄 수 있다.
* [sshuttle](https://github.com/apenwarr/sshuttle)은 동적인 iptable을 구성하여 ssh proxy를 사용할 수 있도록 도와준다.
* [redsocks](https://github.com/darkk/redsocks)를 사용하면 좀 더 편하게 SOCKS proxy를 구축할 수 있다.
* [squid](http://www.squid-cache.org/) 다소 사용이 복잡하지만 여러 상황에서 활용할 수 있다.
