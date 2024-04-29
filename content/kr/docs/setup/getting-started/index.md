---
title: 시작하기
description: Istio 기능을 빠르고 쉽게 시도.
weight: 5
aliases:
    - /kr/docs/setup/kubernetes/getting-started/
    - /kr/docs/setup/kubernetes/
    - /kr/docs/setup/kubernetes/install/kubernetes/
keywords: [getting-started, install, bookinfo, quick-start, kubernetes]
owner: istio/wg-environments-maintainers
test: yes
---

{{< tip >}}
{{< boilerplate gateway-api-future >}}
게이트웨이 API를 통해 Istio를 시작하고 싶다면,
아래 내용 대신 [시작하기 미래 버전](/kr/docs/setup/additional-setup/getting-started/)을 참고하세요.
{{< /tip >}}

이 가이드는 당신이 Istio를 빠르게 평가해 볼 기회를 제공합니다. 이미 Istio에 대해 익숙하거나 
다른 설정 프로파일 혹은 고급 [배포 모델](/kr/docs/ops/deployment/deployment-models/) 설치에 관심이 있다면,
[어떤 Istio 설치 방법을 선택해야 할까?](/kr/about/faq/#install-method-selection)의 FAQ 페이지를 
참고하세요.

아래 내용을 따라가기 위해서는 [지원 중인 버전](/kr/docs/releases/supported-releases#support-status-of-istio-releases)의 쿠버네티스 ({{< supported_kubernetes_versions >}})가 실행되고 있는 {{< gloss >}}cluster{{< /gloss >}}가 있어야 합니다.
또한 [미니큐브](https://kubernetes.io/docs/tasks/tools/install-minikube/) 혹은 [플랫폼별 설치 방법](/kr/docs/setup/platform-setup/)에 기재되어 있는 플랫폼을 사용해야 합니다.


Istio를 시작하기 위해 아래 단계를 따르십시오:

1. [Istio를 다운로드 및 설치하기](#download)
1. [샘플 어플리케이션 배포하기](#bookinfo)
1. [어플리케이션을 외부 트래픽에 오픈하기](#ip)
1. [대시보드 보기](#dashboard)

## Istio 다운로드 {#download}

1.  [Istio 배포]({{< istio_release_url >}}) 페이지로 가서 
    당신의 OS에 맞는 설치 파일을 다운로드 하거나, 최신 배포 버전을 다운로드 하고
    압축을 푸세요(Linux 혹은 macOS):

    {{< text bash >}}
    $ curl -L https://istio.io/downloadIstio | sh -
    {{< /text >}}

    {{< tip >}}
    위 명령어는 Istio의 최신 배포 버전(숫자순)을 다운로드 합니다.
    명령어에 변수를 전달하여 특정 버전을 다운로드 하거나 프로세서 아키텍처를 재정의할 수 
    있습니다.
    예를 들어 x86_64 아키텍처용 Istio {{< istio_full_version >}}를 다운로드 하려면,
    다음을 실행하세요:

    {{< text bash >}}
    $ curl -L https://istio.io/downloadIstio | ISTIO_VERSION={{< istio_full_version >}} TARGET_ARCH=x86_64 sh -
    {{< /text >}}

    {{< /tip >}}

1.  다운로드 한 Istio 패키지 디렉토리로 이동하세요. 가령 패키지명이 
    `istio-{{< istio_full_version >}}`이라면:

    {{< text syntax=bash snip_id=none >}}
    $ cd istio-{{< istio_full_version >}}
    {{< /text >}}

    해당 디렉터리 내부에는 아래의 요소들이 포함되어 있습니다:

    - `samples/` 내에는 샘플 어플리케이션 존재
    - `bin/` 디렉터리에는 [`istioctl`](/kr/docs/reference/commands/istioctl) 클라이언트 바이너리 존재

1.  `istioctl` 클라이언트를 path에 추가하세요 (Linux or macOS):

    {{< text bash >}}
    $ export PATH=$PWD/bin:$PATH
    {{< /text >}}

## Istio 설치하기 {#install}

1.  이 설치 가이드에서는 `demo` [설정 프로필](/kr/docs/setup/additional-setup/config-profiles/)을 
    사용합니다. 테스팅에 적합한 디폴트 세팅 때문에 `demo`를 선택하는 것이긴 하나,
    실배포 혹은 성능 테스팅을 위한 다른 프로필도 존재합니다.

    {{< warning >}}
    만약 당신의 플랫폼이 Openshift와 같은 벤더 맞춤 설정 프로필을 가지고 있다면,
    아래 명령어에 `demo` 프로필 대신 해당 프로필을 입력하시길 바랍니다. 자세한 내용은 
    [플랫폼 지침서](/kr/docs/setup/platform-setup/)를 참고하세요.
    {{< /warning >}}

    {{< text bash >}}
    $ istioctl install --set profile=demo -y
    ✔ Istio core installed
    ✔ Istiod installed
    ✔ Egress gateways installed
    ✔ Ingress gateways installed
    ✔ Installation complete
    {{< /text >}}

1.  어플리케이션이 배포될 때 stio가 Envoy 사이드카 프록시를 자동으로 주입될 수 있도록 
    네임스페이스 라벨을 추가하십시오:

    {{< text bash >}}
    $ kubectl label namespace default istio-injection=enabled
    namespace/default labeled
    {{< /text >}}

## 샘플 어플리케이션 배포하기 {#bookinfo}

1.  [`Bookinfo` 샘플 어플리케이션](/kr/docs/examples/bookinfo/)을 배포하세요:

    {{< text bash >}}
    $ kubectl apply -f @samples/bookinfo/platform/kube/bookinfo.yaml@
    service/details created
    serviceaccount/bookinfo-details created
    deployment.apps/details-v1 created
    service/ratings created
    serviceaccount/bookinfo-ratings created
    deployment.apps/ratings-v1 created
    service/reviews created
    serviceaccount/bookinfo-reviews created
    deployment.apps/reviews-v1 created
    deployment.apps/reviews-v2 created
    deployment.apps/reviews-v3 created
    service/productpage created
    serviceaccount/bookinfo-productpage created
    deployment.apps/productpage-v1 created
    {{< /text >}}

1.  어플리케이션이 시작되면 각 포드(pod)가 준비가 되면, Istio 사이드카가 각 포드 옆에 배포될 것입니다.

    {{< text bash >}}
    $ kubectl get services
    NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    details       ClusterIP   10.0.0.212      <none>        9080/TCP   29s
    kubernetes    ClusterIP   10.0.0.1        <none>        443/TCP    25m
    productpage   ClusterIP   10.0.0.57       <none>        9080/TCP   28s
    ratings       ClusterIP   10.0.0.33       <none>        9080/TCP   29s
    reviews       ClusterIP   10.0.0.28       <none>        9080/TCP   29s
    {{< /text >}}

    그리고

    {{< text bash >}}
    $ kubectl get pods
    NAME                              READY   STATUS    RESTARTS   AGE
    details-v1-558b8b4b76-2llld       2/2     Running   0          2m41s
    productpage-v1-6987489c74-lpkgl   2/2     Running   0          2m40s
    ratings-v1-7dc98c7588-vzftc       2/2     Running   0          2m41s
    reviews-v1-7f99cc4496-gdxfn       2/2     Running   0          2m41s
    reviews-v2-7d79d5bd5d-8zzqd       2/2     Running   0          2m41s
    reviews-v3-7dbcdcbc56-m8dph       2/2     Running   0          2m41s
    {{< /text >}}

    {{< tip >}}
    위 명령어를 재실행하고 다음 단계로 가기 전에, 
    모든 포드의 READY가 `2/2`로 되고 STATUS가 `Running`이 될 때까지 기다리세요. 
    이 과정은 플랫폼에 따라 수 분이 소요될 수 있습니다. 
    {{< /tip >}}

1.  
    이 시점까지 모든 것이 올바르게 작동하는지 확인하십시오. 앱이 클러스터 내에서 실행 중이고 
    HTML 페이지를 보여주는지 확인하려면 다음 명령어를 실행하세요:

    {{< text bash >}}
    $ kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
    <title>Simple Bookstore App</title>
    {{< /text >}}

## 어플리케이션을 외부 트래픽에 오픈하기 {#ip}

Bookinfo 어플리케이션은 배포되긴 했으나 외부에서의 접근은 불가능합니다. 외부로부터의 접근을 위해
[Istio 수신 게이트웨이](/kr/docs/concepts/traffic-management/#gateways)를 만들어야 합니다. 
Istio 수신 게이트웨이는 메쉬의 경계에서 접근 경로(path)를 라우트(route)와 매핑해주는 역할을 합니다. 

1.  이 어플리케이션과 Istio 게이트웨이를 연결하세요:

    {{< text bash >}}
    $ kubectl apply -f @samples/bookinfo/networking/bookinfo-gateway.yaml@
    gateway.networking.istio.io/bookinfo-gateway created
    virtualservice.networking.istio.io/bookinfo created
    {{< /text >}}

1.  설정에 이상이 없는지 확인하십시오:

    {{< text bash >}}
    $ istioctl analyze
    ✔ No validation issues found when analyzing namespace: default.
    {{< /text >}}

### 수신 IP 및 포트 확인하기

게이트웨이 접근을 위한 `INGRESS_HOST` 및 `INGRESS_PORT` 변수 설정을 위해 
다음 지침을 따르십시오. 탭을 이용하여 당신의 플랫폼에 맞는 지침을 선택하세요:

{{< tabset category-name="gateway-ip" >}}

{{< tab name="Minikube" category-value="external-lb" >}}

 미니큐브 터널을 시작하기 위해 새 터미널 창에서 아래 명령어를 실행하십시오. 
 미니큐브 터널은 트래픽을 당신의 Istio 수신 게이트웨이로 보내는 역할을 합니다.
 즉, 이 명령어는 외부 로드 밸런서인 `service/istio-ingressgateway`의 `EXTERNAL-IP`값을 생성해 줍니다.

{{< text bash >}}
$ minikube tunnel
{{< /text >}}

수신 호스트와 수신 포트 변수를 설정하십시오:

{{< text bash >}}
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
{{< /text >}}

IP 주소와 포트가 각 환경변수에 성공적으로 매핑되었는지 확인하십시오:

{{< text bash >}}
$ echo "$INGRESS_HOST"
127.0.0.1
{{< /text >}}

{{< text bash >}}
$ echo "$INGRESS_PORT"
80
{{< /text >}}

{{< text bash >}}
$ echo "$SECURE_INGRESS_PORT"
443
{{< /text >}}

{{< /tab >}}

{{< tab name="Other platforms" category-value="node-port" >}}

아래 명령어를 수행하여 당신의 쿠버네티스 클러스터가 외부 로드 벨런서를 지원하는 환경에서 실행되고 있는지 확인하십시오:

{{< text bash >}}
$ kubectl get svc istio-ingressgateway -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                      AGE
istio-ingressgateway   LoadBalancer   172.21.109.129   130.211.10.121  80:31380/TCP,443:31390/TCP,31400:31400/TCP   17h
{{< /text >}}

만약 `EXTERNAL-IP` 값이 설정되어 있다면, 당신의 환경은 수신 게이트웨이로 쓸 수 있는 외부 로드 벨런서를 가지고 있습니다.
만약 `EXTERNAL-IP` 값이 `<none>` (혹은 계속 `<pending>`)이라면, 당신의 환경은 수신 게이트웨이로 쓸 수 있는 외부 로드 벨런서를 가지고 있지 않습니다.
이 경우 서비스의 [노드 포트(node port)](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport)를 
사용하여 게이트웨이에 접근할 수 있습니다.

당신의 환경과 관련된 지침을 선택하세요:

**당신의 환경에 외부 로드 벨런서가 존재할 경우에만 아래 지침을 따르십시오.**

수신 IP와 포트를 설정하십시오:

{{< text bash >}}
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].port}')
{{< /text >}}

{{< warning >}}
특정 환경에서는 IP 주소 대신 호스트 네임을 통해 로드 벨런서를 노출 시킵니다.
이 경우, 수신 게이트웨이의 `EXTERNAL-IP` 값은 IP 주소가 아닌 호스트 네임이며, 위 명령어를 통한 `INGRESS_HOST` 환경 변수 
설정은 실패할 것입니다. `INGRESS_HOST` 값 수정을 위해 아래 명령어를 사용하십시오:

{{< text bash >}}
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
{{< /text >}}

{{< /warning >}}

**당신의 환경에 외부 로드 벨런서가 존재하지 않아 노드 포트를 대신 사용할 경우 아래 지침을 따르십시오.**

수신 포트를 설정하십시오:

{{< text bash >}}
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
$ export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
{{< /text >}}

_GKE:_

{{< text bash >}}
$ export INGRESS_HOST=worker-node-address
{{< /text >}}

`ingressgateway` 서비스 포트에 TCP 트래픽을 허용하기 위해 방화벽 규칙을 생성해야 합니다.
아래 명령어를 통해 HTTP 포트, 보안 포트(HTTPS), 혹은 두 포트 모두에 대한 트래픽을 허용하십시오:

{{< text bash >}}
$ gcloud compute firewall-rules create allow-gateway-http --allow "tcp:$INGRESS_PORT"
$ gcloud compute firewall-rules create allow-gateway-https --allow "tcp:$SECURE_INGRESS_PORT"
{{< /text >}}

_IBM Cloud Kubernetes Service:_

{{< text bash >}}
$ ibmcloud ks workers --cluster cluster-name-or-id
$ export INGRESS_HOST=public-IP-of-one-of-the-worker-nodes
{{< /text >}}

_Docker For Desktop:_

{{< text bash >}}
$ export INGRESS_HOST=127.0.0.1
{{< /text >}}

_기타 환경:_

{{< text bash >}}
$ export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
{{< /text >}}

{{< /tab >}}

{{< /tabset >}}

1.  `GATEWAY_URL`을 설정하십시오:

    {{< text bash >}}
    $ export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
    {{< /text >}}

1.  IP 주소와 포트가 각 환경변수에 성공적으로 매핑되었는지 확인하십시오:

    {{< text bash >}}
    $ echo "$GATEWAY_URL"
    127.0.0.1:80
    {{< /text >}}

### 외부 접근 확인하기 {#confirm}

Bookinfo 어플리케이션이 외부로부터 접근이 가능한지 확인하기 위해 
브라우저를 통해 Bookinfo product 페이지를 여십시오.

1.  아래 명령어로 Bookinfo 어플리케이션의 외부 주소를 불러 오십시오.

    {{< text bash >}}
    $ echo "http://$GATEWAY_URL/productpage"
    {{< /text >}}

1.  위 명령어의 결과값을 웹 브라우저에 붙여 넣은 후, Bookinfo product 페이지가 보여지는지 확인하세요.

## 대시보드 보기 {#dashboard}

Istio는 [다양한](/kr/docs/ops/integrations) 원격 측정 어플리케이션과 결합할 수 있습니다. 이들 어플리케이션은 당신의 서비스 메쉬 구조에 대한 이해를 
돕고, 메쉬의 배치를 보여주며, 메쉬의 상태를 분석합니다.

아래 지침을 활용하여 [Prometheus](/kr/docs/ops/integrations/prometheus/), [Grafana](/kr/docs/ops/integrations/grafana), 
[Jaeger](/kr/docs/ops/integrations/jaeger/)와 함께, [Kiali](/kr/docs/ops/integrations/kiali/) 대시보드를 배포하십시오.

1.  [Kiali 및 여타 addons]({{< github_tree >}}/samples/addons)을 설치하고 이들이 배포될 때까지 기다리십시오.

    {{< text bash >}}
    $ kubectl apply -f samples/addons
    $ kubectl rollout status deployment/kiali -n istio-system
    Waiting for deployment "kiali" rollout to finish: 0 of 1 updated replicas are available...
    deployment "kiali" successfully rolled out
    {{< /text >}}

    {{< tip >}}
    addons 설치 과정에 에러가 발생하는 경우, 명령어를 재실행 하십시오. 일부 타이밍 문제들은 명령어 재실행을 통해 
    해결 가능 합니다.
    {{< /tip >}}

1.  Kiali 대시보드에 접속하세요.

    {{< text bash >}}
    $ istioctl dashboard kiali
    {{< /text >}}

1.  왼쪽 네비게이션 메뉴에서 _Graph_ 를 선택하고, _Namespace_ 드랍다운에서 _default_ 를 선택하세요.

    {{< tip >}}
    {{< boilerplate trace-generation >}}
    {{< /tip >}}

    Kiali 대시보드를 통해 `Bookinfo` 샘플 어플리케이션 내 서비스 간 관계를 훑어볼 수 있습니다.
    또한 트래픽 흐름을 시각화 할 수 있도록 필터도 제공합니다.

    {{< image link="./kiali-example2.png" caption="Kiali Dashboard" >}}

## 다음 단계

평가 설치의 완료를 축하합니다!

아래 리스트는 `demo` 설치를 활용하여 초보자들이 Istio의 기능들을 평가해 볼 수 있는 좋은 과제들 
입니다:

- [요청 라우팅(Request routing)](/kr/docs/tasks/traffic-management/request-routing/)
- [결함 주입(Fault injection)](/kr/docs/tasks/traffic-management/fault-injection/)
- [트래픽 전환(Traffic shifting)](/kr/docs/tasks/traffic-management/traffic-shifting/)
- [측정지표 검색(Querying metrics)](/kr/docs/tasks/observability/metrics/querying-metrics/)
- [측정지표 시각화(Visualizing metrics)](/kr/docs/tasks/observability/metrics/using-istio-dashboard/)
- [외부 서비스 접근(Accessing external services)](/kr/docs/tasks/traffic-management/egress/egress-control/)
- [메쉬 시각화(Visualizing your mesh)](/kr/docs/tasks/observability/kiali/)

실배포 사용을 위한 Istio 설정 전, 아래 리소스를 참고하세요: 

- [배포 모델(Deployment models)](/kr/docs/ops/deployment/deployment-models/)
- [배포 모범 사례(Deployment best practices)](/kr/docs/ops/best-practices/deployment/)
- [포드 요구사항(Pod requirements)](/kr/docs/ops/deployment/requirements/)
- [일반 설치 지침(General installation instructions)](/kr/docs/setup/)

## Istio 커뮤니티 참여

[Istio 커뮤니티](/get-involved/)에 참여하여 질문과 피드백을 주시길 바랍니다.

## 제거

`Bookinfo` 샘플 어플리케이션 및 관련 설정을 제거하려면, [`Bookinfo` 지우기]를 
참고하십시오.

Istio를 제거하면 `istio-system` 네임스페이스 아래의 RBAC 권한 및 모든 리소스가 
계층적으로 삭제됩니다. 존재하지 않는 리소스에 대한 오류는 이미 삭제된 리소스에 대한 내용이어서 
무시해도 괜찮습니다.

{{< text bash >}}
$ kubectl delete -f @samples/addons@
$ istioctl uninstall -y --purge
{{< /text >}}

`istio-system` 네임스페이스는 기본적으로 지워지지 않습니다. 
만약 더이상 필요하지 않을 경우, 아래 명령어를 통해 지우시면 됩니다:

{{< text bash >}}
$ kubectl delete namespace istio-system
{{< /text >}}

Istio에 Envoy 사이드카 프록시를 자동으로 주입하도록 명령하는 라벨은 기본적으로 지워지지 않습니다.
만약 더이상 필요하지 않을 경우, 아래 명령어를 통해 제거하십시오:

{{< text bash >}}
$ kubectl label namespace default istio-injection-
{{< /text >}}
