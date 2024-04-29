---
title: 플랫폼 필수 구성요소
description: Prerequisites for platform setup for Istio.
weight: 1
skip_seealso: true
keywords: [platform-setup,prerequisites]
owner: istio/wg-environments-maintainers
test: no
---


## 클러스터 노드에 대한 커널 모듈 요구사항

iptables 가로채기 모드(interception mode) 사용시, Istio 프록시 사이드카 컨테이너가 첨부된 어플리케이션 포드를 실행하기 위해서는 해당 클러스터 노드가 특정 커널 모듈을 불러와야 합니다. Istio는 iptables 가로채기가 필요없는 `whitebox` 모드를 사용할 수도 있는데, 이경우 특정 커널 모듈이 사용되지 않기에 이 섹션은 참고하지 
않아도 됩니다.

모듈은 특히 `istio-init` 컨테이너 혹은 `istio-cni` 데몬에서 쓰여지는데, 포드 내 iptables 규칙을 변경하여 들어오고 나가는 모든 트래픽을 istio-proxy 컨테이너 내 사이드카 프록시로 보내는 역할을 합니다. 대다수 플랫폼에서는 이 모듈들은 자동으로 로드되긴 하나, 아래 나열된 모듈 중 특정 모듈이 사용할 수 없다던지 혹은 iptables에 의해 자동으로 불러 올 수 없다던지 하는 사고들이 보고되고 있습니다. 따라서 호스트에서 필수 구성요소가 모두 갖추어졌는지 항상 확인해야 합니다. 예를 들어, 이 
[`selinux 이슈`](https://www.suse.com/support/kb/doc/?id=000020241)는 RHEL 내 selinux가 아래 언급된 커널 모듈의 자동 로드를 막는 현상에 대해 소개하고 있습니다.

| 모듈 | 비고 |
| --- | --- |
| `br_netfilter` |  |
| `ip6table_mangle` | IPv6 혹은 듀얼스택 클러스터에서만 필요 |
| `ip6table_nat` | IPv6 혹은 듀얼스택 클러스터에서만 필요 |
| `ip6table_raw` | IPv6 혹은 듀얼스택 클러스터에서만 필요 |
| `iptable_mangle` |  |
| `iptable_nat` |  |
| `iptable_raw` | `DNS` 가로채기에만 필요 |
| `xt_REDIRECT` |  |
| `xt_connmark` | `TPROXY` 가로채기 모드에서만 필요 |
| `xt_conntrack` |  |
| `xt_mark` | `TPROXY` 가로채기 모드에서만 필요 |
| `xt_owner` |  |
| `xt_tcpudp` |  |
| `xt_multiport`|  |

아래 모듈들 또한 위에 나열된 모듈들이 의존하고 있기에 클러스터 노드에 로드되어야 합니다:

| 모듈 | 비고 |
| --- | --- |
| `bridge` |  |
| `ip6_tables` | IPv6 혹은 듀얼스택 클러스터에서만 필요 |
| `ip_tables` |  |
| `nf_conntrack` |  |
| `nf_conntrack_ipv4` |  |
| `nf_conntrack_ipv6` | IPv6 혹은 듀얼스택 클러스터에서만 필요 |
| `nf_nat` |  |
| `nf_nat_ipv4` |  |
| `nf_nat_ipv6` | IPv6 혹은 듀얼스택 클러스터에서만 필요 |
| `nf_nat_redirect` |  |
| `x_tables` |  |
