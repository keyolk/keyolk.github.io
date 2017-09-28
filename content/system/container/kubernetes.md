+++
title  = "kubernetes"
toc    = true
weight = 10
+++

## Intro
Production 환경에서,
Kubernetes를 어떻게 배포/구성할지에 대해 정리.

아래 사항에 대해서 Best Practice 찾아 기술.

- deploy
- availability
- security
- performance

## Deployment
Multi Cloud 환경과 같이 infra 제공자가 다양한 경우
반복적으로 Cluster를 배포하게된다.

Process에 따라 Kubernetes의 Cluster Federation도 활용될 수 있으며
이를 고려한 배포가 이뤄져야 한다.

전반적인 배포 제어는 Ansible과 같은 도구를 활용하여 자동화한다.

### Multi Cloud
Cluster를 배포할때,
Multi Cloud를 통한 Affinity를 고려하자면 Hashicorp의 Terraform과 같은
추상화된 Cloud Management Tool을 사용하는게 좋다.

instance가 확보되고 나면 cloud-config를 같은 도구를 활용하여
일관성 있게 각 instance를 초기화하도록 한다.

### Self Hosting
Self Hosted Cluster를 구성하면 배포 이후 Cluster의 변경사항을 관리하는데 용이하다.
kubelet은 독립적으로 Pod을 hosting 할 수 있으므로 이를 이용해
apiserver, control manager, proxy를 구동하도록 한다.

binary는 hyperkube 같은 묶음을 활용하면 일관성있게 binary를 관리할 수 있다.
hyperkube image에 대한 배포도 container image를 활용하되
kubelet을 실행하는 container engine이 Resilency한지 따져봐야한다.
Docker 경우 최근 version에서 이를 지원하며
그외 CoreOS의 RKT나 LXC 등을 활용할 수 있다.

## Network
Self Hosted Cluster를 구성할땐 CNI network을 사용해야한다.
CNI를 활용할 경우 kubelet은
cluster의 KV storage에 직접 접근하여 network fabric을 구성한다.

CNI option으론 Calico + Flannel 조합으로 VxLan을 많이 사용한다.
아래의 Canal을 통해 쉽게 배포 할 수 있다.

https://github.com/projectcalico/canal

Canal은 새로운 Kubelet이 등록되면 DaemonSet으로 필요한 Network Fabric Asset을
전달하도록 되어있다.

Network에 대한 변경이 이뤄지면 일관적으로 전체 cluster node에 이를 전파할 수 있다.

### Endpoints
kubelet에서 kube apiserver 및 KV storage에 일관적으로 접속이 가능해야한다.
내부 load balancer를 구성하여 connection을 보장해 준다.

Master와 KV storage는 HA하게 구성한다.

## Secret
Kuberntes에서 Secret 관리와 관련된 내용을 정리한다.

http://kubernetes.io/docs/user-guide/secrets/

Kubernetes에선 자체적으로 Secret System을 가지고 있으나
data는 KV stroage 내에서 암호화 되지 않은채로 존재한다.

즉, etcd에 대한 접근 권한이 있으면
모든 secret을 가져갈 수 있다.

etcd 자체적으로 제공하는 auth는 client certificates가 있고
이에 대해서 ACL도 제공하므로 이를 이용하면 보다 secure하게 이를 관리할 수 있으나,
근본적으로 data 유출에 대한 위험은 남아있게된다.

앞서 보았듯 self hosting 하게 cluster를 구성하는경우 network를 CNI로 구성하게 되는데
kubelet이 직접 KV storage에 접근하게 된다.

### Secret Encryption
모든 kubelet node에서 etcd에 대해 접근 권한을 갖게되는것이므로 유의해야한다.

secret data를 암호화하기 위한 option으로 vault가 많이 고려된다.
아직 kubernetes에서 secret에 대해 plugin 형태의 interface를 제고하고 있지 않아
사용하기 용이한 implementation은 아직 없으나 kubernetes에 맞춘 vault controller를 제공하는
proeject가 몇가지 존재한다.

아래는 vault와 관련된 link이다.

- Hashicorp Vault : https://github.com/hashicorp/vault
  - dicussions : https://github.com/kubernetes/kubernetes/issues/10439#issuecomment-263954184
  - generate PKI stufss for kubernetes components : https://www.digitalocean.com/company/blog/vault-and-kubernetes/
  - deploy vault in kubernetes : http://www.devoperandi.com/vault-in-kubernetes-take-2/
  - vault integration with kubernetes pod
    - https://github.com/Boostport/kubernetes-vault
    - https://github.com/kelseyhightower/vault-controller

### TLS Assets
위의 Vault를 통해 Kubernetes Component 들이 사용하는 TLS Asset들을 관리할 수 있다.
Consul + Vault + Consul Template 조합으로 pki backend를 사용한다.

다만 이를 kubelet 하나로 Self Hosted 하게 구성하려고 할때,
당장 편리해보이는 방법을 모르겠다.

apiserver를 구성하는 kubelet에 vault와 consul template을 같이 올리는 방법도 있을것 같으나
좀더 고민이 필요하다.

## Auth
Kubernetes Auth와 관련,
대략적인 내용과 link들을 정리한다.

- Authentication : http://kubernetes.io/docs/admin/authentication/
  - Basic Auth
  - Client Certificates
  - Static Token
  - Webhook Token
  - KeyStone Password
  - Authenticating Proxy
  - OIDC Token :
    - Google OIDC
    - Auth0
    - CoreOS Dex :  https://github.com/coreos/dex
      - Google OIDC : https://github.com/coreos/dex/blob/master/Documentation/openid-connect.md
      - Github OAuth : https://github.com/coreos/dex/blob/master/Documentation/github-connector.md
      - LDAP : https://github.com/coreos/dex/blob/master/Documentation/ldap-connector.md
    - Redhat KeyCloak : http://www.devoperandi.com/kubernetes-authentication-openid-connect/
    - CloudFoundry UAA : https://apigee.com/about/tags/kubernetes
- Authorization : http://kubernetes.io/docs/admin/authorization/
  - ABAC
  - RBAC : https://github.com/uruddarraju/kubernetes-rbac-policies
  - Webhook
    - bitesize-authz-webhook : https://github.com/pearsontechnology/bitesize-authz-webhook

#### AuthN
CoreOS Dex의 경우 JBoss의 Keyclak이나 CloudFoundry의 UAA에 비해서 전반적으로 구조가 단순하고 배포하기 용이하다.
다른 Solution에 비해 덜성숙된 느낌을 같지만 사용함에 문제가 없었다.
별도 Client를 구성 해야하는 번잡함이 있긴하다.

#### AuthZ
RBAC이 현재로는 apiserver의 재기동 없이 dynamic하게 policy를 적용할 수 있는 거의 유일한 option 이다.
Webhook을 통해 외부 instance에서 처리하는 방법도 있으나
별도로 자체 구축해야하며, 알려진 opensource 중에선 뚜렷이 눈에 띄는게 없었다.

#### ToDo
Login : kubectl의 경우 아직 login관련된 feature가 없다 Token을 kubeConfig에 따로 전달해주어야한다.
별도 CLI wrapper를 사용할 수 있다.

RBAC : User record가 생성될때 이에따라 필요한 RBAC policy를 생성할 controller가 필요하다.
없을 시 수동으로 처리해줘야한다.
