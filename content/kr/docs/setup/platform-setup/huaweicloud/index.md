---
title: Huawei Cloud
description: Instructions to set up an Huawei Cloud kubernetes cluster for Istio.
weight: 23
skip_seealso: true
aliases:
    - /docs/setup/kubernetes/prepare/platform-setup/huaweicloud/
    - /docs/setup/kubernetes/platform-setup/huaweicloud/
keywords: [platform-setup,huawei,huaweicloud,cce]
owner: istio/wg-environments-maintainers
test: no
---

[화웨이 클라우드 컨테이너 엔진](https://www.huaweicloud.com/intl/product/cce.html)을 활용하여 Istio가 설치된 
클러스터를 준비하고자 한다면 아래 설명을 따르세요. `클라우드 컨테이너 엔진 콘솔`에서 Istio를 지원하는 쿠버네티스 클러스터를 쉽고 간편하게 화웨이 클라우드에 배포할 수 있습니다.

{{< tip >}}
화웨이는 화웨이 클라우드 컨테이너 엔진 내 {{< gloss >}}managed control plane{{< /gloss >}} 애드온(add-on)을 제공하고 있으며, 이를 활용하면 Istio를 직접 설치할 필요가 없습니다.
더 자세한 내용은 [화웨이 어플리케이션 서비스 메쉬](https://support.huaweicloud.com/asm/index.html)를 참고하세요.
{{< /tip >}}

Istio를 직접 설치하기 전 클러스터 준비 시 [화웨이 클라우드 설명서](https://support.huaweicloud.com/en-us/qs-cce/cce_qs_0008.html)를 따르세요:

1. CCE 콘솔에 로그인 하세요. **Dashboard** > **Buy Cluster**를 선택하여 **Buy Hybrid Cluster** 페이지를 여세요. 해당 페이지를 여는 다른 방법은 네비게이션 창에서 **Resource Management** > **Clusters**을 선택하고 **Hybrid Cluster** 옆에 있는 **Buy**를 클릭하세요.

1.  **Configure Cluster** 페이지에서 클러스터 변수들을 설정하세요. 아래 예시에서는 대부분의 변수가 디폴트값으로 설정되어 있습니다. 클러스터 설정이 
    끝났다면 Next: **Create Node**을 클릭하여 노드 생성 페이지로 이동하세요.

    {{< tip >}}
    Istio는 쿠버네티스 버전별로 상이한 요구사항이 존재합니다. Istio의 [지원 정책](/docs/releases/supported-releases#support-status-of-istio-releases)에 따라 버전을 선택하세요.
    {{< /tip >}}

    아래 이미지는 클러스터를 생성하고 설정하는 GUI를 보여줍니다:

    {{< image link="./create-cluster.png" caption="Configure Cluster" >}}

1.  노드 생성 페이지에서 아래 변수를 설정하세요:

    {{< tip >}}
    Istio는 추가적인 자원을 요구합니다. 경험적으로 최소 4 vCPU와 8 GB 메모리를 확보하시는 것이 좋습니다.
    {{< /tip >}}

    아래 이미지는 노드를 생성하고 설정하는 GUI를 보여줍니다:

    {{< image link="./create-node.png" caption="Configure Node" >}}

1.  [kubectl 설정](https://support.huaweicloud.com/intl/en-us/cce_faq/cce_faq_00041.html)

1.  [설치 가이드](/docs/setup/install)에 따라 CCE 클러스터에 Istio를 설치할 수 있습니다.

1.  필요에 따라 [ELB](https://support.huaweicloud.com/intl/productdesc-elb/en-us_topic_0015479966.html) 설정으로 
    Istio 수신 게이트웨이를 노출하세요.

    - [Elastic Load Balancer 생성](https://console.huaweicloud.com/vpc/?region=ap-southeast-1#/elbs/createEnhanceElb)

    - ELB 인스턴스를 `istio-ingressgateway` 서비스에 묶기

      ELB 인스턴스 ID와 `loadBalancerIP`를 `istio-ingressgateway`로 세팅하세요.

{{< text bash >}}
$ kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubernetes.io/elb.class: union
    kubernetes.io/elb.id: 4ee43d2b-cec5-4100-89eb-2f77837daa63 # ELB ID
    kubernetes.io/elb.lb-algorithm: ROUND_ROBIN
  labels:
    app: istio-ingressgateway
    install.operator.istio.io/owning-resource: unknown
    install.operator.istio.io/owning-resource-namespace: istio-system
    istio: ingressgateway
    istio.io/rev: default
    operator.istio.io/component: IngressGateways
    operator.istio.io/managed: Reconcile
    operator.istio.io/version: 1.9.0
    release: istio
  name: istio-ingressgateway
  namespace: istio-system
spec:
  clusterIP: 10.247.7.192
  externalTrafficPolicy: Cluster
  loadBalancerIP: 119.8.36.132     ## ELB EIP
  ports:
  - name: status-port
    nodePort: 32484
    port: 15021
    protocol: TCP
    targetPort: 15021
  - name: http2
    nodePort: 30294
    port: 80
    protocol: TCP
    targetPort: 8080
  - name: https
    nodePort: 31301
    port: 443
    protocol: TCP
    targetPort: 8443
  - name: tcp
    nodePort: 30229
    port: 31400
    protocol: TCP
    targetPort: 31400
  - name: tls
    nodePort: 32028
    port: 15443
    protocol: TCP
    targetPort: 15443
  selector:
    app: istio-ingressgateway
    istio: ingressgateway
  sessionAffinity: None
  type: LoadBalancer
EOF
{{< /text >}}

다양한 [과제](/docs/tasks)를 시도하면서 Istio를 경험하세요.
