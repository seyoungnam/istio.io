---
title: Alibaba Cloud
description: Instructions to set up an Alibaba Cloud Kubernetes cluster for Istio.
weight: 5
skip_seealso: true
aliases:
    - /docs/setup/kubernetes/prepare/platform-setup/alicloud/
    - /docs/setup/kubernetes/platform-setup/alicloud/
keywords: [platform-setup,alibaba-cloud,aliyun,alicloud]
owner: istio/wg-environments-maintainers
test: n/a
---

{{< boilerplate untested-document >}}

[알리바바 클라우드 쿠버네티스 컨테이너 서비스](https://www.alibabacloud.com/product/kubernetes) 클러스터 내 Istio 사용을 위해 
아래 설명을 참고하세요. `Container Service console`에서 알리바바 클라우드에 Istio를 완전히 지원하는 쿠버네티스 클러스터를 빠르고 쉽게 배포할 
수 있습니다. 


{{< tip >}}
알리바바 클라우드는 Alibaba Cloud Service Mesh (ASM)라 불리는 완전 관리형 서비스 메시 플랫폼을 제공하며, Istio와 완벽하게 
호환됩니다. 더 자세한 내용과 설명은 [Alibaba Cloud Service Mesh](https://www.alibabacloud.com/help/doc-detail/147513.htm)를 
참고하세요.
{{< /tip >}}

## 필수 구성요소

1. Container Service, Resource Orchestration Service (ROS), 그리고 RAM 서비스를 사용하려면 
[알리바바 클라우드 설명서](https://www.alibabacloud.com/help/doc-detail/95108.htm)를 참고하세요.


## 순서

1. `컨테이너 서비스 콘솔`에 로그인한 후, 왼쪽 네비게이션 섹션 내 **Kubernetes** 아래  **Clusters**를 
클릭하여 **Cluster List** 페이지에 들어가십시오.

1. 오른쪽 상단의 **Create Kubernetes Cluster** 버튼을 클릭하세요.

1. 클러스터 이름을 입력하세요. 클러스터 이름은 1-63 글자수 내에서 설정해야 하며, 숫자, 한자, 영어, 그리고 하이픈 (-)도 입력 가능합니다.

1. 클러스터가 구동되고 있는 **region**과 **zone**을 선택하세요.

1. 클러스터 네트워크 타입을 선택하세요. 현재 쿠버네티스 클러스터는 VPC 네트워크 타입만을 지원합니다.

1. 노드 타입을 설정하세요. Pay-As-You-Go와 구독 타입이 지원됩니다.

1. 마스터 노드를 설정하세요. 마스터 노드의 generation, family, 및 type을 선택하세요.

1. 워커 노드를 설정하세요. 새 워커 노드를 만들거나 기존의 ECS 인스턴스를 워커 노드로 추가하세요.

1. 로그온 모드를 설정하고, Pod Network CIDR과 Service CIDR을 설정하세요.
