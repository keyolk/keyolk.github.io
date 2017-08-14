+++
title  = "office"
toc    = true
weight = 1
prev   = "/workspace/personal"
next   = "/workspace/blog"
+++

## Intro
사무실용 환경 설정에 필요한 Step 정리.

- Corporation Firewall에 따른 개인용 Transparent Proxy.
- 이종 OS간 Keyboard/Mouse Sharing.
- 내부 가상화 환경.
- SSH.

## OpenSSH
### Remote Host

Configuration File을 통해 Remote Host를 관리하면 편하다.
```bash
$ cat ~/.ssh/config
```
```
Host remote1
  Hostname 1.2.3.4
  User root
  IdentityFile remote1
  ForwardX11 yes
  ProxyCommand ssh proxy nc %h %p
```
위의 예는 SSH를 연결하면서 X11Forward와 Dynamic Proxy 연결을 같이 한다.

IdentityFile을 별도로 만들경우 아래와 같이 사용 가능하다.
```bash
$ ssh -i $IDENTITY_FILE $REMOTE
```

### Proxy
#### Forward Proxy

```bash
$ ssh -N -f -L 0.0.0.0:8080:1.2.3.4:8080 $REMOTE
```

#### Backward Proxy

```bash
$ ssh -R 0.0.0.0:8080:1.2.3.4:8080 $REMOTE
```

0.0.0.0에 port를 binding 하려면 아래 설정이 필요하다.
```bash
$ cat /etc/ssh/sshd_config | grep -i gatewayports
GatewayPorts yes
```
#### Dynamic Proxy

```bash
$ ssh -D 8080 $REMOTE
```

#### X11 Forward

```bash
$ ssh -X $REMOTE
```

XAuth를 Skip한다.
```bash
$ ssh -Y $REMOTE
```

관련 설정
```bash
$ cat /etc/ssh/sshd_config | grep -i x11
X11Forwarding yes
#X11DisplayOffset 10
X11UseLocalHost yes
```

## Redosocks
Corporation Firewall로 인해 internet으로 가는 pakcet은 모두 Proxy를 거쳐가야한다.
HTTP proxy를 일일이 등록하는건 힘드므로,
개인용 Transparent Proxy를 구축한다.

```bash
# pacman -Sy redsocks
```

```bash
# vi /etc/redsocks.conf
```

```bash
base {
	log_debug = off;
	log_info = on;
	log = stderr;
	daemon = on;
	redirector = iptables;
}

redsocks {
	local_ip = 0.0.0.0;
	local_port = 10080;

	ip = proxy;
	port = 8080;

	type = http-relay;
}

redsocks {
	local_ip = 0.0.0.0;
	local_port = 10443;

	ip = proxy;
	port = 8080;

  type = http-connect;
}
```

```bash
# systemctl enable redsocks
# systemctl start redsocks
```

위와 같이 구성하면 redsocks는 10080/10443으로 오는 packet을
SOCKS channel을 통해 모두 proxy:8080으로 보내게된다.

```bash
#!/bin/bash

iptables -t nat -N PROXY
iptables -t nat -F PROXY
iptables -t nat -I OUTPUT 1 -j PROXY
iptables -t nat -I PREROUTING 1 -j PROXY

for network in `ip a | grep 'inet ' | grep -v 'inet6' | awk '{print $2}'`;
do
    sudo iptables -t nat -A PROXY -j RETURN --dest $network -p tcp
done

iptables -t nat -A PROXY -p tcp --dport 80 -j REDIRECT --to 10080
iptables -t nat -A PROXY -p tcp --dport 443 -j REDIRECT --to 10443
```
위와 같이하면 내부 network을 제외,
외부로 나가는 모든 HTTP/HTTPS packet은 각기 10080/10443을 통해 proxy로 가게된다.

## Proxy Auto Config
내부 System과 외부 System이 혼재되어 있을때 이를 모두 iptable를 통해 proxy 처리하려면 힘들다.
간단한 작업은 PAC를 통해할 수 있다.
또한 특정 target과 연동되어있는 System의 경우 Dynamic Proxy를 통해 VPN과같은 효과를 사용해야한다.

아래 10000 port는 target system에 대해 VPN endpoint가 된다.
```bash
ssh -N -f -D :10000 target

```

아래와 같이 URI/IP에 따라 경유되는 channel을 바꿔줄 수 있다.
```pac
GATEWAY_TARGET="SOCKS localhost:10000; DIRECT";
GATEWAY_PROXY="proxy:8080;"

function FindProxyForURL(url, host)
{
    if(host=="127.0.0.1" || host=="localhost") return "DIRECT";

    if(host=="1.2.3.4") return GATEWAY_TARGET;
    if(host=="example-1.org" || dnsDomainIs(host, ".example-1.org")) return GATEWAY_TARGET;

    if(host=="5.6.7.8") return GATEWAY_PROXY;
    if(host=="example-2.org" || dnsDomainIs(host, ".example-2.org")) return GATEWAY_PROXY;

    return GATEWAY_PROXY;
}
```
위 pac를 Browser나 NetworkManager의 Proxy설정에 반영하면
HTTP/HTTPS packetㅇ네 대해서 위에 정의한대로 Routing 시켜준다.

## Vagrant
### Plugin
내 경우 보통 아래 plugin들을 사용한다.

```
$ vagrant plugin install landrush vagrant-cachier vagrant-triggers vagrant-vbguest  vagrant-proxyconf vagrant-ca-certificates
```

landrush는 vagrant instance들과 host machine 사이에
DNS를 구축하는걸 도와준다.

구체적으론 resolveconf와 dnsmasq를 이용한 구성이다.
Arch 등 일부 OS에서는 자동으로 dnsmasq 설치와 설정을 처리해주지 않으므로,
직접 구성해줘야 한다.

openresolv 경우,
```
# vi /etc/resolv.conf.head
```
```
nameserver 127.0.0.1
```

```
# resolveconf -u
```

위와 같이 persist한 DNS 설정을 추가할 수 있다.

dnsmasq는,
```
# vi /etc/dnsmasq.conf
```

위 설정에 아래 line을 추가하면 .vagrant의 TLD를 +
10053 port에서 service하는 landrush에 물어보도록 할 수 있다.
```
server=/.vagrant/127.0.0.1#10053
```

### Trouble Shooting
```
Failed to open a session for the virtual machine seconion-standalone.

Implementation of the USB 2.0 controller not found!

Because the USB 2.0 controller state is part of the saved VM state, the VM cannot be started. To fix this problem, either install the 'Oracle VM VirtualBox Extension Pack' or disable USB 2.0 support in the VM settings (VERR_NOT_FOUND).

Result Code: NS_ERROR_FAILURE (0x80004005)
Component: Console
Interface: IConsole {8ab7c520-2442-4b66-8d74-4ff1e195d2b6}
```

```
config.vm.provider "virtualbox" do |vb|
  vb.customize ["modifyvm", :id, "--usb", "on"]
  vb.customize ["modifyvm", :id, "--usbehci", "off"]
end
```

```
No usable default provider could be found for your system.

Vagrant relies on interactions with 3rd party systems, known as
"providers", to provide Vagrant with resources to run development
environments. Examples are VirtualBox, VMware, Hyper-V.

The easiest solution to this message is to install VirtualBox, which
is available for free on all major platforms.

If you believe you already have a provider available, make sure it
is properly installed and configured. You can see more details about
why a particular provider isn't working by forcing usage with
`vagrant up --provider=PROVIDER`, which should give you a more specific
error message for that particular provider.

```

여러가지 이유가 있겠지만
지원하지 않는 version의 driver만 설치되어 있을 경우에도 발생할 수 있다.
driver의 version을 낮추거나 vagrant의 version을 높여도 되지만.
다소 naive한 해결 방법이 존재한다.

```bash
$ cd /opt/vagrant/embedded/gems/gems/vagrant-1.8.4/plugins/providers/virtualbox/driver/
$ ls
base.rb  version_4_0.rb  version_4_2.rb  version_5_0.rb
meta.rb  version_4_1.rb  version_4_3.rb
$ cp version_5_0.rb version_5_1.rb

$ cat version_5_1.rb| grep 5_0
      class Version_5_0 < Base
          @logger = Log4r::Logger.new("vagrant::provider::virtualbox_5_0")

$ cat meta.rb| grep -A 7 'driver_map  '
          driver_map   = {
            "4.0" => Version_4_0,
            "4.1" => Version_4_1,
            "4.2" => Version_4_2,
            "4.3" => Version_4_3,
            "5.0" => Version_5_0,
          }

$ cat ../plugin.rb| grep -A 8 'module Driver'
    module Driver
      autoload :Meta, File.expand_path("../driver/meta", __FILE__)
      autoload :Version_4_0, File.expand_path("../driver/version_4_0", __FILE__)
      autoload :Version_4_1, File.expand_path("../driver/version_4_1", __FILE__)
      autoload :Version_4_2, File.expand_path("../driver/version_4_2", __FILE__)
      autoload :Version_4_3, File.expand_path("../driver/version_4_3", __FILE__)
      autoload :Version_5_0, File.expand_path("../driver/version_5_0", __FILE__)
    end
```
위에서 보듯이 3개 file을 수정해주면 정상적으로 실행 가능하다.

## Synergy
이런저런 이유로 Windows와 Linux Machine을 같이 써야한다.
Synergy를 사용하여 이종 OS간 Keyboard 및 Mouse, Clipboard를 공유할 수 있다.

아래 URI에서 가입 및 구매가 필요하다.

https://symless.com/synergy/

### Configuration
```bash
# pacman -Sy synergy
```

기본 Monitor 배치 구성을 먼저 한다.
```bash
$ vi /etc/synergy.conf
```

```
section: screens
  keyolk-arch:
	keyolk-windows:
end

section: links
	keyolk-arch:
		right = keyolk-windows
	keyolk-windows:
		left  = keyolk-arch
end
```

Synergy는 Client/Server 구조를 이뤄져있다.
실제 Keyboard와 Mouse가 연결되어 있는 쪽이 Server가 된다.

Server 구동.
```
$ systemctl enable synergys
$ systemctl start synergys
```

Client에서 synergy를 설치 후 Server로 접속하면 된다.

## Chrome
### HSTS
Private Network에서 Corporation Certificate를 사용하는 경우
HSTS로 인해 정상적으로 Browsing을 못할 수 있다.
Chrome의 경우 borwser에서 해당 domain의 HSTS 정보를 삭제해야한다.

```bash
chrome://net-internals/#hsts
```

