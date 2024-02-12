---
title: 보안
description: Istio의 인증과 승인 기능에 대한 설명.
weight: 30
keywords: [security,policy,policies,authentication,authorization,rbac,access-control]
aliases:
    - /docs/concepts/network-and-auth/auth.html
    - /docs/concepts/security/authn-policy/
    - /docs/concepts/security/mutual-tls/
    - /docs/concepts/security/rbac/
    - /docs/concepts/security/mutual-tls.html
    - /docs/concepts/policies/
owner: istio/wg-security-maintainers
test: n/a
---

모놀리식(monolithic) 어플리케이션을 여러개의 서비스로 분리하면 더 나은 민첩성과 확장성, 그리고 서비스 재사용성 등 다양한 이점을 얻을 수 있습니다. 그러나 마이크로서비스는 다음과 같은 특별한 보안 요구사항이 발생합니다:

- 중간자 공격(man-in-the-middle attacks)을 방어하기 위해 트래픽 암호화가 필요
- 유연한 서비스 접근 제어를 위해 상호 TLS와 세분화된 접근 정책 설정이 필요
- 누가 언제 무엇을 했는지 확인을 위해 감사 도구가 필요

Istio는 이러한 문제를 해결하기 위한 포괄적인 보안 솔루션을 제공합니다. 이 페이지에서는 Istio 보안 기능을 사용하여 어디서든 서비스를 보호할 수 있는 방법에 대해 간략하게 살펴 볼 예정입니다. 특히 Istio 보안 기능을 통해 데이터, 엔드포인트, 통신 및 플랫폼에 대한 내외부 위협을 모두 완화할 수 있습니다.

{{< image width="75%"
    link="./overview.svg"
    caption="보안 훑어보기"
    >}}

Istio 보안 기능은 강한 신분 확인, 강력한 정책, 투명한 TLS 암호화, 그리고 인증, 승인 및 감사(authentication, authorization and audit, AAA) 도구를 제공하여 서비스와 데이터를 보호합니다. Istio의 보안 목표는 다음과 같습니다:

- 기본값으로서의 보안: 어플리케이션 코드 및 인프라 변경 불필요
- 심층 방어: 기존 보안 시스템과 통합하여 여러 계층의 방어 시스템 제공
- 제로 트러스트 네트워크: 신뢰할 수 없는 네트워크에 보안 솔루션을 구축

배포된 서비스에서 Istio 보안 기능을 사용하려면 [상호 TLS으로의 이관](/docs/tasks/security/authentication/mtls-migration/)를 참고하십시오. 보안 기능 사용에 대한 자세한 지침을 보려면 [보안 작업](/docs/tasks/security/)을 참고하십시오.

## 구조 훑어보기

Istio 내 보안은 다수의 구성요소를 지니고 있습니다:

- 키 및 인증서 관리를 위한 인증 기관(Certificate Authority, CA)
- 설정 API 서버가 프록시에 배포하는 요소들:

    - [인증 정책](/docs/concepts/security/#authentication-policies)
    - [승인 정책](/docs/concepts/security/#authorization-policies)
    - [보안 명명 정보](/docs/concepts/security/#secure-naming)

- 사이드카 및 경계선 프록시는 [정책 적용 지점]((https://csrc.nist.gov/glossary/term/policy_enforcement_point))(Policy Enforcement Points, PEP)으로 작동하여 클라이언트와 서버 간의 통신을 보호
- 원격 측정(telemetry) 및 감사 관리를 위한 Envoy 프록시 확장 세트

제어 영역(Control plane)은 API 서버의 설정을 처리하고 데이터 영역(data plane)내 PEP(정책 적용 지점)를 설정합니다. PEP는 Envoy를 사용하여 구현됩니다. 다음 다이어그램은 관련 아키텍쳐를 보여줍니다.

{{< image width="75%"
    link="./arch-sec.svg"
    caption="보안 구조"
    >}}

앞으로 이어질 섹션에서는 Istio의 보안 기능에 대해 더 자세하게 다룰 예정입니다.

## Istio 신원 정보

신원 정보는 모든 보안 인프라에 적용되는 기본 개념입니다. 워크로드 간 통신이 시작될 때 두 당사자는 상호 인증을 위해 신분 정보가 포함하고 있는 자격 정보를 교환해야 합니다. 클라이언트 측에서는 서버의 신분을 [보안 명명(secure naming)](/docs/concepts/security/#secure-naming) 정보와 비교하여 워크로드 실행자가 승인을 받았는지 확인 합니다. 서버 측에서는 입력된 [승인 정책](/docs/concepts/security/#authorization-policies)에 의거, 클라이언트가 접근할 수 있는 정보를 결정하고, 누가 언제 무엇을 접근했는지 감사하고, 사용한 작업량에 따라 요금을 부과하고, 요금을 지불하지 못한 클라이언트의 접근을 거부합니다. 

Istio의 신원 정보 모델은 최고 수준의 "서비스 신원(`service identity`)"을 사용하여 요청자의 신원을 결정합니다. 이 모델은 서비스 신원이 인간 사용자, 개별 워크로드 혹은 워크로드 그룹 등 다양한 대상을 대표할 수 있어, 뛰어난 유연성과 정밀성을 제공합니다. 서비스 신원이 없는 플랫폼에서는 Istio가 워크로드 인스턴스를 대표하는 서비스 이름 등을 신원 정보로 사용할 수 있습니다.

다음 목록은 다양한 플랫폼에서 사용할 수 있는 서비스 신원의 예를 보여줍니다.

- 쿠버네티스: 쿠버네티스의 서비스 계정(service account)
- GCE: GCP 서비스 계정(service account)
- 온프레미스(비 쿠버네티스): user account, custom service account, 
    Istio service account, 혹은 GCP service account. 커스텀 서비스 계정은 
    고객의 Identity Directory에서 관리하는 신원 정보와 같은 기존 서비스 계정을 의미합니다.

## 신원 및 인증서 관리 {#pki}

Istio는 X.509 인증서를 사용하여 모든 워크로드에 강력한 신원 정보를 안전하게 제공합니다. 각 Envoy 프록시와 함께 실행되는 `istio-agent`는 `istiod`와의 상호작용을 통해 대규모 키 및 인증서 재생성 프로세스를 자동화 합니다. 아래 그림은 신원 정보 생성 흐름을 보여줍니다.

{{< image width="40%"
    link="./id-prov.svg"
    caption="신원 생성 작업흐름"
    >}}

Istio는 아래의 흐름에 따라 키와 인증서를 생성합니다:

1. `istiod`는 [인증서 서명 요청](https://en.wikipedia.org/wiki/Certificate_signing_request)(certificate signing requests, CSR)을 받아들이기 위해 gRPC 서비스를 제공합니다.
1. 프로세스가 시작되면 `istio-agent`는 개인 키와 CSR을 생성한 다음, 서명을 위해 해당 자격 정보과 함께 CSR을 `istiod`로 보냅니다.
1. `istiod`의 CA(인증 기관)는 CSR에 포함된 자격 정보가 유효한지 확인합니다. 검증에 성공하면 CSR에 서명하여 인증서를 생성합니다.
1. 워크로드가 시작되면 Envoy는 [Envoy SDS(Secret Discovery Service)](https://www.envoyproxy.io/docs/envoy/latest/configuration/security/secret#secret-discovery-service-sds) API를 통해 같은 컨테이너에 있는 `istio-agent`의 인증서와 키를 요청합니다.
1. `istio-agent`는 `istiod`로부터 받은 인증서와 개인 키를 Envoy SDS API를 통해 Envoy로 보냅니다.
1. `istio-agent`는 워크로드별 인증서가 만료되었는지 모니터링하고, 만료되었을 경우 위 프로세스를 반복합니다.



## 인증(Authentication)

Istio는 두가지 형태의 인증방법을 제공합니다:

- 상대방 인증(Peer authentication): 연결을 시도하는 클라이언트를 확인하기 위해 서비스 간 인증에 사용됩니다. 
    Istio는 전송 인증을 위한 풀스택 솔루션으로 [상호 TLS](https://en.wikipedia.org/wiki/Mutual_authentication)를 제공하며, 이는 서비스 코드의 변경 없이 이루어 집니다. 이 솔루션은 다음과 같습니다:
    - 클러스터와 클라우드 간 상호 운용성을 지원하고자 각 서비스에 강력한 신원 정보를 제공
    - 서비스간 통신을 보호
    - 키 및 인증서 생성, 배포, 순환을 자동화하는 키 관리 시스템 제공

- 요청 인증: 요청에 첨부된 자격 증명 정보 확인 목적의 최종 사용자(end-user) 인증에 사용됩니다. 
    Istio는 JSON Web Token(JWT) 검증 방식의 요청별 인증을 지원하며, 
    맞춤형 인증 공급자 또는 OpenID Connect 공급자를 사용하여 간소화된 개발자 경험을 지원합니다. 
    인증 공급자의 예는 다음과 같습니다:
    - [ORY Hydra](https://www.ory.sh/)
    - [Keycloak](https://www.keycloak.org/)
    - [AuthO](https://auth0.com/)
    - [Firebase Auth](https://firebase.google.com/docs/auth/)
    - [Google Auth](https://developers.google.com/identity/protocols/OpenIDConnect)

어느 인증 공급자를 쓰건 상관없이 Istio는 맞춤형 쿠버네티스 API를 통해 Istio 설정 저장소(`Istio config store`)에 인증 정책을 저장합니다. Istio는 각 프록시에 대한 키와 인증 정책을 지속적으로 업데이트 합니다. 또한 Istio는 정책 변경이 시행되기 전 새로 변경된 정책이 보안 상태에 어떤 영향을 미칠 수 있는지 이해하는데 도움이 되도록 허용 모드(permissive mode)에서의 인증을 지원합니다.

### 상호 TLS 인증

Istio는 클라이언트 및 서버측 정책 적용 지점(PEP)을 기준으로 서비스간 통신 터널을 만드는데, 
양측 정책 적용 지점은 [Envoy 프록시](https://www.envoyproxy.io/)로 구현 됩니다.
워크로드가 상호 TLS 인증 방식을 통해 반대편 다른 워크로드에 요청을 보낼 때, 
해당 요청은 아래의 프로세스에 따라 처리됩니다:

1. Istio는 클라이언트에서 나가는 트래픽을 클라이언트에 붙어있는 사이드카 Envoy에 보냅니다.
1. 클라이언트 측 Envoy는 서버 측 Envoy와 상호 TLS 핸드셰이크를 시작합니다. 핸드셰이크 중
    클라이언트 측 Envoy는 서버 인증서에 기재된 서비스 계정이 대상 서비를 실행할 권한이 있는지 
    확인을 하기 위해 [보안 명명](/docs/concepts/security/#secure-naming) 확인을 
    수행합니다.
1. 클라이언트 측과 서버 측 Envoy 간 상호 TLS 연결이 생성되고, Istio는 트래픽을 클라이언트 측 
    Envoy에서 서버 측 Envoy로 전달합니다.
1. 서버 측 Envoy는 해당 요청을 승인합니다. 승인 후, 로컬 TCP 연결을 통해 백엔드 서비스로 트래픽을 
    전달합니다.

Istio는 클라이언트와 서버에 대한 최소 TLS 버전으로 `TLSv1_2`를 사용하며,
다음과 같은 암호 알고리즘을 지원합니다:

- `ECDHE-ECDSA-AES256-GCM-SHA384`

- `ECDHE-RSA-AES256-GCM-SHA384`

- `ECDHE-ECDSA-AES128-GCM-SHA256`

- `ECDHE-RSA-AES128-GCM-SHA256`

- `AES256-GCM-SHA384`

- `AES128-GCM-SHA256`

#### 허용 모드(Permissive mode)

Istio 상호 TLS에는 일반 텍스트 트래픽과 상호 TLS 트래픽을 동시에 받아 들이는 허용 모드가 있습니다.
이 기능은 상호 TLS 적용 만족도를 크게 높이는 요소 입니다.

Istio를 적용하지 않은 서버와 통신하는 많은 비 Istio 클라이언트의 존재는 Istio를 서버에  
적용하여 상호 TLS를 쓰고자 하는 운영자에게 고민거리를 안겨 줍니다. 일반적으로 운영자가 모든 클라이언트에 
Istio 사이드카를 동시에 설치하는 것은 불가능하며, 가능하더라도 일부 클라이언트에서는 그렇게 할 수 있는 
권한도 없습니다. 설사 서버에 Istio 사이드카를 설치했다고 하더라도 운영자는 현재 구축된 통신 연결을 건드리지 
않고 상호 TLS를 활성화 하는 것은 불가능 합니다.

허용 모드가 활성화되면 서버는 일반 텍스트와 상호 TLS 트래픽을 모두 받아들일 수 있습니다. 이 모드는 상호 TLS 
적용 프로세스에 더 큰 유연성을 제공합니다. 서버에 설치된 Istio 사이드카는 기존 일반 텍스트 트래픽은 건드리지 
않은 채 상호 TLS 트래픽을 즉시 받아 들입니다. 결과적으로 운영자는 클라이언트에 상호 TLS 트래픽 전송을 위한 
Istio 사이드카 설치 및 설정을 점진적으로 진행할 수 있습니다. 클라이언트에 대한 설정이 끝나면, 운영자는 상호 TLS 
전용 모드로 서버를 전환할 수 있습니다. 더 자세한 정보는 
[상호 TLS 적용 튜토리얼](/docs/tasks/security/authentication/mtls-migration)을 참고하세요.

#### 보안 명명(Secure naming)

서버 신원은 인증서에서 찾을 수 있으며, 서비스 이름은 쿠버네티스의 discovery service나 DNS를 통해 알 수 있습니다. 
보안 명명 정보는 서버 신원을 서비스 이름과 매핑합니다. `A`라는 신원이 서비스 이름 `B`로 매핑되었다는 
의미는 "`A`가 서비스 `B`를 실행할 권한이 있다" 입니다. 제어 영역(control plane)은 `apiserver`를 
관찰하고, 보안 명명 매핑을 생성하며, 정책 적용 지점(PEP)에 안전하게 배포합니다. 다음 예제는 인증 과정에 
보안 명명이 왜 중요한지를 설명합니다.

만약 `datastore`라는 서비스를 운영하고 있는 서버가 `infra-team`이라는 신원에 대해서만 서비스 이용을 허가한다고 가정해 봅시다. 
그리고 한 악의적인 사용자가 `test-team` 신원에 대한 인증서와 키를 가지고 있으며, 해당 서비스를 위장하여 클라이언트에서 
`datastore` 서비스로 보내는 요청을 가로채고자 합니다. 이를 위해 `test-team` 신원에 대한 인증서와 키를 가지고 있는 
위조된 서버를 배포할 것입니다. 그리고 그 사용자는 (DNS 스푸핑, BGP/경로 하이재킹, ARP 스푸핑 등을 통해) 
`datastore`로 전송되는 트래픽을 하이재킹하여 위조 서버로 전달할 것입니다.

클라이언트가 `datastore` 서비스에 요청을 보내면, 클라이언트는 위장 서버로부터 받은 인증서에서 `test-team` 신원 정보를 
불러온 후, 보안 명명 정보를 바탕으로 `test-team`이 `datastore`를 실행할 권한이 있는지 확인 합니다. 
클라이언트는 `test-team`의 `datastore`서비스에 대한 사용이 허가되지 않음을 확인하게 되고, 결국 인증 실패로 
위장 서버와의 통신 연결이 막히게 됩니다.

HTTP/HTTPS가 아닌 트래픽의 경우, 보안 명명은 DNS 스푸핑을 방지하지 못하며, 공격자는 서비스의 대상 IP를 
수정할 수 있습니다. TCP 트래픽은 `Host` 정보가 포함되지 않고 Envoy는 트래픽 전송 시 대상 IP에만 의존하기에 
Envoy는 트래픽을 하이재킹된 IP의 서비스로 전송할 수 있습니다. 이러한 DNS 스푸핑은 클라이언트 측 Envoy가 
트래픽을 수신하기 전에도 발생할 수 있습니다.

### 인증 구조(Authentication architecture)

상대방 및 요청 인증 정책(peer and request authentication policies)을 사용하여 Istio 메쉬 내 요청을 
수신하는 워크로드에 대한 인증 요구 사항을 지정할 수 있습니다. 메쉬 운영자는 `.yaml`파일을 사용하여 정책을 
지정합니다. 정책은 배포와 동시에 Istio 설정 저장소(Istio configuration storage)에 저장되며, Istio 
controller는 설정 저장소를 모니터링 합니다. 

정책 변경은 관련 설정 변경을 통해 정책 적용 지점(PEP)이 어떻게 인증 매커니즘을 수행해야 할지 재정의 합니다. 
제어 영역(control plane)은 공개 키를 가져온 후 이를 JWT 검증을 위한 설정 정보에 첨부 합니다. 또는 
Istiod가 Istio 시스템이 관리하는 키와 인증서에 대한 경로를 어플리케이션 pod에 설치하고 상호 TLS에 활용 
합니다. 자세한 내용은 [신원 및 인증서 관리 섹션](#pki)에서 확인할 수 있습니다.

Istio는 변화된 설정을 대상 엔드포인트에 비동기적으로 보냅니다. 프록시가 이 설정을 수신 후 새로운 인증 요구 
사항을 해당 pod에 즉시 적용합니다.

요청을 보내는 클라이언트 서비스는 인증 매커니즘을 따라야 합니다. 요청 인증(request authentication)의 경우, 
어플리케이션은 JWT를 획득하고 요청에 첨부해야 합니다. 상대방 인증(peer authentication)의 경우, Istio가 
양 PEP간 모든 트래픽을 상호 TLS로 자동 업그레이드 합니다. 만약 인증 정책이 상호 TLS 모드를 비활성화 할 경우, 
Istio는 PEP 간 통신에 일반 텍스트를 계속 사용합니다. 이 동작을 재정의 하려면 
[대상 규칙](/docs/concepts/traffic-management/#destination-rules)(destination rules)을 사용하여 
상호 TLS 모드를 명시적으로 비활성화 해야 합니다. 
상호 TLS 작동 방식에 대한 더 자세한 내용은 [상호 TLS 인증 섹션](/docs/concepts/security/#mutual-tls-authentication)을 
참고 하세요.

{{< image width="50%"
    link="./authn.svg"
    caption="인증 구조"
    >}}

Istio는 두 인증 유형뿐만 아니라 적용 가능한 다른 자격 증명 방식으로부터 생성된 신원도 다음 계층인 
[인증](/docs/concepts/security/#authorization)으로 보냅니다.

### 인증 정책(Authentication policies)

이 섹션에서는 Istio 인증 정책의 작동 방식에 대하여 더 자세히 다룹니다. [구조 섹션](/docs/concepts/security/#authentication-architecture)에서 언급하였듯, 인증 정책은 서비스에 대한 요청에 적용됩니다. 
상호 TLS에서 클라이언트 측 인증 규칙은 `DestinationRule` 내 `TLSSettings`에서 지정합니다. 
[TLS 설정 참고 문헌](/docs/reference/config/networking/destination-rule#ClientTLSSettings)에서 
더 자세한 내용을 확인할 수 있습니다. 

다른 Istio 설정과 마찬가지로 인증 정책은 `.yaml`파일을 통해 설정할 수 있으며, `kubectl` 명령어로 
정책을 배포합니다. 아래 인증 정책은 `app:reviews` 라벨이 붙어 있는 워크로드에 대한 전송 인증은 
상호 TLS를 사용해야 함을 설정한 예시 입니다.

{{< text yaml >}}
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: "example-peer-policy"
  namespace: "foo"
spec:
  selector:
    matchLabels:
      app: reviews
  mtls:
    mode: STRICT
{{< /text >}}

#### 정책 저장소(Policy storage)

Istio는 메쉬 전체에 적용되는 정책 범위를 루트 네임스페이스에 저장합니다. 이러한 정책은 selector를 
비워놓음으로서 메쉬 내 모든 워크로드에 적용됩니다. 네임스페이스 범위가 지정된 정책은 해당 네임스페이스에 
저장되며, 그 네임스페이스 내에 있는 워크로드에만 적용됩니다. 만약 `selector`필드를 설정하면 
인증 정책은 설정한 조건과 일치하는 워크로드에만 적용됩니다.

상대방 및 요청 인증 정책은 각각 `PeerAuthentication` and `RequestAuthentication`을 통해 
저장됩니다.

#### Selector 필드

상대방 및 요청 인증 정책은 `selector`필드를 통해 정책을 적용할 워크로드를 선택합니다. 다음 예시는 
`app:product-page` 라벨이 붙어 있는 워크로드에 적용되는 정책의 `selector`필드를 보여 줍니다:

{{< text yaml >}}
selector:
  matchLabels:
    app: product-page
{{< /text >}}

만약 `selector`필드를 비워둘 경우, Istio는 정책 저장소 적용 범위 안에 있는 모든 워크로드에  
해당 정책을 적용합니다. 따라서 `selector`필드는 정책의 적용 대상 범위 설정을 도와 줍니다:

- 메쉬 전체에 적용되는 정책: `selector`필드가 비었거나 혹은 아예 없이 루트 네임스페이스에 기재된 정책
- 네임스페이스에 적용되는 정책: `selector`필드가 비었거나 혹은 아예 없이 비루트 네임스페이스에 
    기재된 정책
- 워크로드에 적용되는 정책: `selector`필드에 대상이 기재되어 있고, 일반적은 네임스페이스에 정의된 정책

상대방 및 요청 인증 정책은 `selector`필드에 대해 동일한 계층 구조 원칙을 따르긴 하나, Istio는 
이를 약간 다른 방식으로 결합하고 적용합니다.

메쉬 전체에 적용되는 상대방 인증 정책은 하나만 있을 수 있으며, 네임스페이스에 적용되는 상대방 인증 정책 또한 
하나만 가능합니다. 만약 다수의 메쉬 전체에 혹은 특정 네임스페이스에 적용되는 상대방 인증 정책이 존재할 경우, 
Istio는 최신 정책을 무시하고 기존 정책을 우선시 합니다. 마찬가지로 워크로드에 적용되는 정책이 다수일 경우, 
Istio는 가장 오래된 정책을 선택합니다.

Istio는 아래 순서에 의거 각 워크로드에 대해 가장 범위가 좁은 일치 정책을 적용합니다:

1. 특정 워크로드에 대한 정책
1. 특정 네임스페이스에 대한 정책
1. 메쉬 전체에 대한 정책

Istio는 일치하는 모든 요청 인증 정책을 마치 단일 요청 인증 정책인 것처럼 결합합니다. 따라서 메쉬 
전체에 대한 정책 혹은 네임스페이스에 적용되는 다수의 정책을 설정해도 무방합니다. 하지만 다수의 요청 
인증 정책을 설정하는 것은 좋은 습관이 아님을 알려드리며 추천하지 않습니다.

#### 상대방 인증(Peer authentication)

상대방 인증 정책은 목표 워크로드에 Istio가 수행할 상호 TLS 모드를 지정합니다. 지원하는 모드는 
아래와 같습니다:

- PERMISSIVE: 워크로드는 상호 TLS와 일반 텍스트 트래픽 모두를 받아들입니다. 해당 모드는 
    사이드카가 아직 붙어 있지 않아 워크로드에 상호 TLS 적용이 불가능할 시에 가장 유용합니다.
    워크로드에 사이드카 주입이 완료되면 STRICT모드로 전환하면 됩니다.
- STRICT: 워크로드는 상호 TLS 트래픽만 받아들입니다.
- DISABLE: 상호 TLS를 비활성화 합니다. 자체 보안 솔루션을 적용하지 않는다면 보안 관점에서 볼 때 
    해당 모드를 사용하면 안됩니다. 

모드가 설정되지 않으면 상위 범위의 모드를 준수합니다. 메쉬 전체에 적용되는 상대방 인증 정책의 모드가 
설정되지 않았을 경우 기본적으로 `PERMISSIVE` 모드를 준수합니다.

아래 상대방 인증 정책은 `foo` 네임스페이스 내 모든 워크로드에 대해 상호 TLS를 적용합니다: 

{{< text yaml >}}
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: "example-policy"
  namespace: "foo"
spec:
  mtls:
    mode: STRICT
{{< /text >}}

특정 워크로드에 대한 상대방 인증 정책을 통해서는 포트별로 각기 다른 상호 TLS 모드를 설정할 수 있습니다. 
단, 워크로드 내 상호 TLS 설정이 완료된 포드만 사용할 수 있습니다. 아래는 `app:example-app`
워크로드의 `80`포트에 상호 TLS를 비활성화 하였으며, 나머지 다른 포트는 해당 네임스페이스에 적용되는 
상대방 인증 정책으로 설정된 상호 TLS가 적용되는 예시 입니다:

{{< text yaml >}}
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: "example-workload-policy"
  namespace: "foo"
spec:
  selector:
     matchLabels:
       app: example-app
  portLevelMtls:
    80:
      mode: DISABLE
{{< /text >}}

위 상대방 인증 정책은 `example-app` 워크로드에 대한 요청을 `example-service`의 `80`포트에서 
처리하도록 명시한 아래 서비스 설정 때문에 유효 합니다:

{{< text yaml >}}
apiVersion: v1
kind: Service
metadata:
  name: example-service
  namespace: foo
spec:
  ports:
  - name: http
    port: 8000
    protocol: TCP
    targetPort: 80
  selector:
    app: example-app
{{< /text >}}

#### 요청 인증(Request authentication)

요청 인증 정책은 JSON 웹 토큰(JWT)의 유효성을 검사하는데 필요한 값을 지정합니다. 
즉 아래와 같은 값이 지정됩니다:

- 요청 내 토큰의 위치
- 발행자 혹은 요청
- 공ro JSON 웹 키 세트(JWKS)

Istio는 요청 인증 정책의 규칙에 따라 제시된 토큰을 확인하고 유효하지 않은 토큰이 첨부된 
요청을 거부합니다. 요청에 토큰이 없으면 기본적으로 수락됩니다. 토큰 없는 요청을 거부하려면 
경로 혹은 동작 등 특정 작업에 대한 제한을 가하는 권한 규칙을 설정해야 합니다.

요청 인증 정책은 둘 이상의 JWT를 지정할 수 있는데, 각 JWT가 각기 다른 위치에 사용될 
경우에만 가능합니다. 둘 이상의 정책이 하나의 워크로드와 일치하는 경우, Istio는 마치 단일 
정책으로 지정된 것처럼 모든 규칙을 결합합니다. 이러한 행동은 다른 공급자의 JWT를 허용하도록 
워크로드를 프로그래밍하기에 유용합니다. 그러나 유효한 JWT가 둘 이상인 요청은 해당 요청의 
주체가 정의되지 않기 때문에 지원되지 않습니다.

#### Principals

상대방 인증 정책과 상호 TLS 사용 시 Istio는 상대방 인증으로부터 신원 정보를 `source.principal`의  
값으로 부여합니다. 마찬가지로 요청 인증 정책 사용 시 Istio는 JWT의 신원 정보를 
`request.auth.principal`에 할당합니다. 이러한 principal로 권한 정책을 설정은 물론 
telemetry 출력값으로도 사용합니다.

### 인증 정책 업데이트

인증 정책은 언제든 변경할 수 있으며 Istio는 거의 실시간으로 새 정책을 워크로드에 반영합니다. 
그러나 Istio는 모든 워크로드가 동시에 새 정책을 수신한다고 보장할 수 없습니다. 다음 권장 
사항은 인증 정책 업데이트 시 중단을 방지하는데 도움이 될 것입니다:

- `DISABLE`에서 `STRICT`모드로 또는 그 반대로 변경 시 `PERMISSIVE`모드를 임시 
  상대방 인증 정책으로 활용합니다. 모든 워크로드가 원하는 모드로 성공적으로 전환되면 최종 모드로 
  정책을 적용하면 됩니다. Istio telemetry를 통해 워크로드가 성공적으로 전환되었는지 
  확인할 수 있습니다.
- 한 JWT에서 다른 JWT로 요청 인증 정책을 이관할 시, 기존 규칙의 제거 없이 새 JWT에 대한 규칙을 
  정책에 추가하세요. 그러면 워크로드가 두가지 유형의 JWT를 모두 허용하며, 모든 트래픽이 새 
  JWT로 전환된 후 이전 규칙을 제거하면 됩니다. 다만 각 JWT는 서로 다른 위치를 대상으로 
  사용해야 합니다. 

## 승인(Authorization)

Istio의 승인 기능은 메쉬 내 워크로드에 대해 메쉬, 네임스페이스, 그리고 워크로드 레벨에서의 
접근 제어를 제공합니다. 이러한 레벨별 제어는 다음과 같은 장점이 있습니다:

- 워크로드 간 그리고 최종 사용자와 워크로드 간 승인.
- 간소화된 API: 사용 및 유지 관리가 쉬운 단일 [`AuthorizationPolicy` CRD](/docs/reference/config/security/authorization-policy/).
- 유연한 어법: 운영자는 Istio 속성에 대한 사용자 정의 조건을 정의하고, CUSTOM, DENY, 그리고 ALLOW 동작을 사용.
- 고성능: Istio 승인(`ALLOW`와 `DENY`)은 Envoy에 기본적으로 적용.
- 높은 호환성: gRPC, HTTP, HTTPS 및 HTTP/2는 물론 일반 TCP 프로토콜도 기본적으로 지원.

### 승인 구조(Authorization architecture)

승인 정책은 서버 측 Envoy 프록시의 수신 트래픽에 대한 접근 제어를 시행합니다. 각 Envoy 프록시는 런타임에 요청을 
승인하는 인증 엔진을 실행합니다. 요청이 프록시에 도달하면 권한 부여 엔진은 요청 내용을 현재 승인 정책과 비교 후 
승인 결과를 `ALLOW` 혹은 `DENY`로 반환합니다. 운영자는 `.yaml`파일을 사용하여 Istio 승인 정책을 설정합니다.

{{< image width="50%"
    link="./authz.svg"
    caption="승인 구조"
    >}}

### 암묵적 활성화(Implicit enablement)

Istio의 승인 기능을 명시적으로 활성화할 필요는 없습니다. 기능들은 설치 후 바로 사용 가능합니다. 워크로드에 대한  
접근 제어를 시행하려면 승인 정책을 적용하면 됩니다.

승인 정책 적용이 되지 않은 워크로드에 대해 Istio는 모든 요청을 허용합니다.

승인 정책은 `ALLOW`, `DENY`, 그리고 `CUSTOM` 동작을 지원합니다. 워크로드에 대한 접근을 보호하기 위해 
필요에 따라 각각 다른 동작을 포함하는 여러 정책을 적용할 수 있습니다.

Istio는 `CUSTOM`, `DENY`, `ALLOW` 순서로 레이어에서 일치하는 정책을 확인 합니다. 각 동작 유형에 대해 
Istio는 먼저 해당 동작이 적용된 정책이 있는지 확인 후 요청이 정책 사양과 일치하는지 확인합니다. 요청이 레이어 
중 어느 정책과도 일치하지 않으면, 다음 레이어로 이동하여 확인 작업을 계속 진행 합니다.

아래 그래프는 정책 우선순위를 자세히 보여 줍니다:

{{< image width="50%" link="./authz-eval.svg" caption="승인 정책 우선순위">}}

한 워크로드에 다수의 승인 정책을 적용할 경우, Istio는 하나씩 추가하는 방식으로 적용합니다.

### 승인 정책(Authorization policies)

승인 정책을 설정하려면 [`AuthorizationPolicy` 사용자 지정 리소스](/docs/reference/config/security/authorization-policy/)을 생성해야 합니다. 승인 정책은 선택자(selector), 동작(action), 규칙 리스트
(a list of rules)이 포함됩니다:

- `selector` 필드는 정책의 대상을 설정합니다.
- `action` 필드는 요청에 대한 승인 혹은 거절을 설정합니다.
- `rules`은 어떤 조건에서 동작을 결정할지를 설정합니다.
    - `rules` 내 `from` 필드는 요청 출처를 설정합니다.
    - `rules` 내 `to` 필드는 요청 작업을 설정합니다.
    - `when` 필드는 규칙을 적용하기 위해 필요한 조건을 설정합니다.

아래 예제는 유효한 JWT 토큰이 첨부된 요청에 대해, `foo` 네임스페이스 내 `app: httpbin`와 `version: v1`
라벨이 붙어있는 워크로드에 대한 접근을 두 출처(`cluster.local/ns/default/sa/sleep` 서비스 계정과 
`dev` 네임스페이스)로 부터 온 요청을 승인하는 승인 정책을 보여 줍니다.

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
 name: httpbin
 namespace: foo
spec:
 selector:
   matchLabels:
     app: httpbin
     version: v1
 action: ALLOW
 rules:
 - from:
   - source:
       principals: ["cluster.local/ns/default/sa/sleep"]
   - source:
       namespaces: ["dev"]
   to:
   - operation:
       methods: ["GET"]
   when:
   - key: request.auth.claims[iss]
     values: ["https://accounts.google.com"]
{{< /text >}}

아래 예제는 요청 출처가 `foo` 네임스페이스가 아닌 경우 요청을 기각하는 승인 정책을 보여줍니다:

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
 name: httpbin-deny
 namespace: foo
spec:
 selector:
   matchLabels:
     app: httpbin
     version: v1
 action: DENY
 rules:
 - from:
   - source:
       notNamespaces: ["foo"]
{{< /text >}}

기각 정책은 승인(허용) 정책보다 우선권을 가집니다. 요청은 다수의 승인 정책에 부합하더라도 
단 하나의 기각 정책과 일치하기만 하면 최종적으로 기각 됩니다. Istio는 승인 정책이 기각 정책을 
우회하는 것을 방지하고자 기각 정책을 먼저 심사합니다.

#### 정책 대상(Policy Target)

`metadata/namespace` 필드와 `selector` 필드를 선택적으로 사용하여 정책 범위 또는 대상을 
설정할 수 있습니다. 정책은 `metadata/namespace` 필드의 네임스페이스에 적용이 됩니다. 만약 
해당값이 루트 네임스페이스로 설정되어 있다면, 정책은 메쉬 내 모든 네임스페이스에 적용됩니다. 
루트 네임스페이스의 값은 재설정이 가능하며, 기본값은 `istio-system` 입니다. 만약 그외 
네임스페이스로 지정되었을 경우, 정책은 지정된 네임스페이스에만 적용됩니다.

`selector` 필드를 사용하여 특정 워크로드에 적용할 정책을 추가로 제한할 수 있습니다. `selector`는 
라벨을 사용하여 대상 워크로드를 선택합니다. `selector`는 `{key: value}` 쌍을 포함하는데 
`key`는 라벨의 이름으로 설정합니다. 만약 `selector`가 설정되어 있지 않으면, 승인 정책은 동일 
네임스페이스 내 모든 워크로드에 적용됩니다.

예를 들어, 아래 `allow-read` 정책은 `default` 네임스페이스 내 `app: products` 라벨이 첨부된 
워크로드에 대해 `"GET"`과 `"HEAD"` 접근을 승인(허용)합니다.

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: allow-read
  namespace: default
spec:
  selector:
    matchLabels:
      app: products
  action: ALLOW
  rules:
  - to:
    - operation:
         methods: ["GET", "HEAD"]
{{< /text >}}

#### 값 일치(Value matching)

승인 정책 내 대부분의 필드는 아래의 모든 일치 문법을 지원합니다:

- 정확한 일치: 주어진 string과 정확히 일치.
- 접두사 일치: `"*"`로 끝나는 string. 예를 들어, `"test.abc.*"`는 `"test.abc.com"`, 
    `"test.abc.com.cn"`, `"test.abc.org"` 등과 일치.
- 접미사 일치: `"*"`로 시작하는 string. 예를 들어, `"*.abc.com"`은 `"eng.abc.com"`, 
    `"test.eng.abc.com"` 등과 일치.
- 존재 일치: `*`은 존재하는 모든 것을 대표(빈 것은 포함하지 않음). 필드에 무엇인가 존재해야 함을 설정하기 위해 
    `fieldname: ["*"]`로 표현 가능. 이는 필드에 대한 아무런 설정이 없는 것과는 다른데, 
    필드에 대한 비설정은 빈 것 또한 포함.

다만 주의해야 할 예외 사항이 존재합니다. 예를 들어, 아래 필드는 정확한 일치만 지원합니다:

- `when` 섹션 내에 있는 `key` 필드
- `source` 섹션 아래에 있는 `ipBlocks`
- `to` 섹션 아래에 있는 `ports` 필드

아래 예제 정책은 `/test/*` 접두사를 가졌거나 `*/info` 접미사를 가진 경로에 대한 접근을 허용합니다.

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: tester
  namespace: default
spec:
  selector:
    matchLabels:
      app: products
  action: ALLOW
  rules:
  - to:
    - operation:
        paths: ["/test/*", "*/info"]
{{< /text >}}

#### 제외 일치(Exclusion matching)

`when` 필드의 `notValues`, `source` 필드의 `notIpBlocks`, `to` 필드의 
`notPorts` 등 부정 조건과의 일치를 지원하기 위해, Istio는 제외 일치를 지원합니다. 
아래 예제는 요청 경로가 `/healthz`이 아닐 경우 JWT 인증 과정에서 획득한 유효한 요청 
principals을 요구합니다. 다시 말해 이 정책은 `/healthz` 경로에 대한 요청을 JWT 인증 
대상에서 제외합니다:

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: disable-jwt-for-healthz
  namespace: default
spec:
  selector:
    matchLabels:
      app: products
  action: ALLOW
  rules:
  - to:
    - operation:
        notPaths: ["/healthz"]
    from:
    - source:
        requestPrincipals: ["*"]
{{< /text >}}

아래 예제는 `/admin` 경로에 대한 요청 중 요청 principal이 없는 요청은 기각합니다:

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: enable-jwt-for-admin
  namespace: default
spec:
  selector:
    matchLabels:
      app: products
  action: DENY
  rules:
  - to:
    - operation:
        paths: ["/admin"]
    from:
    - source:
        notRequestPrincipals: ["*"]
{{< /text >}}

#### `allow-nothing`, `deny-all` 그리고 `allow-all` 정책

아래 예제는 어느 것과도 일치되지 않는 `ALLOW` 정책을 보여 줍니다. 만약 다른 `ALLOW` 정책이 없다면, 
"기본적으로 기각" 행동에 의거 요청은 항상 기각될 것입니다.

"기본적으로 기각" 행동은 워크로드가 `ALLOW` 동작이 포함된 승인 정책이 최소한 하나라도 있을 때만 
적용됩니다.

{{< tip >}}
`allow-nothing` 정책으로 시작 후 `ALLOW` 정책을 점진적으로 추가하여 워크로드에 대한 접근을 
높여 나가는 것이 보안 관점에서 좋은 관행입니다.
{{< /tip >}}

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: allow-nothing
spec:
  action: ALLOW
  # rules 필드가 설정되지 않았기에 이 정책은 어느 요청에도 적용되지 않습니다.
{{< /text >}}

아래 예제는 명시적으로 모든 접근을 기각하는 `DENY` 정책을 보여줍니다. 이 정책은 다른 `ALLOW` 
정책이 있더라도 모든 요청을 항상 기각하는데 그 이유는 `DENY` 정책이 `ALLOW` 정책보다 
우선순위가 높기 때문입니다. 이 정책은 워크로드에 일시적으로 모든 접근을 비활성화할 때 유용합니다.

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: deny-all
spec:
  action: DENY
  # rules 필드가 비었기 때문에 이 정책은 모든 요청에 적용됩니다.
  rules:
  - {}
{{< /text >}}

아래 예제는 워크로드에 모든 접근을 허용하는 `ALLOW` 정책을 나타냅니다. 이 예제는 모든 요청을 
허용하기 때문에 다른 `ALLOW` 정책 추가는 의미가 없습니다. 이는 워크로드에 일시적으로 모든 접근을 
허용할 때 유용합니다. `CUSTOM`와 `DENY` 정책 추가 시 일부 요청은 기각될 수 있습니다.

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: allow-all
spec:
  action: ALLOW
  # 모든 요청과 일치.
  rules:
  - {}
{{< /text >}}

#### 사용자 지정 조건(Custom conditions)

`when` 섹션을 통해서 추가적인 조건을 설정할 수 있습니다. 예를 들어, 아래 `AuthorizationPolicy` 
정의는 `request.headers[version]`가 `"v1"` 혹은 `"v2"`이어야 한다는 조건을 포함하고 
있습니다. 이 경우 `request.headers[version]`가 핵심 요소이며, 이는 Istio 속성이자 map 구조인 
`request.headers`의 항목에 해당합니다.

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
 name: httpbin
 namespace: foo
spec:
 selector:
   matchLabels:
     app: httpbin
     version: v1
 action: ALLOW
 rules:
 - from:
   - source:
       principals: ["cluster.local/ns/default/sa/sleep"]
   to:
   - operation:
       methods: ["GET"]
   when:
   - key: request.headers[version]
     values: ["v1", "v2"]
{{< /text >}}

지원되는 조건의 `key`값은 [조건 페이지](/docs/reference/config/security/conditions/)에 
나열되어 있습니다.

#### 인증 및 미인증 신원(Authenticated and unauthenticated identity)

만약 워크로드를 공개적으로 접근 가능토록 만들고 싶다면, `source` 섹션을 비워두면 됩니다. 이는 모든 
(인증되었거나 미인증된) 사용자와 워크로드를 허용합니다. 예를 들어:

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
 name: httpbin
 namespace: foo
spec:
 selector:
   matchLabels:
     app: httpbin
     version: v1
 action: ALLOW
 rules:
 - to:
   - operation:
       methods: ["GET", "POST"]
{{< /text >}}

인증된 사용자만 허용하고 싶을 경우, 아래와 같이 `principals`을 `"*"`로 설정하세요:

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
 name: httpbin
 namespace: foo
spec:
 selector:
   matchLabels:
     app: httpbin
     version: v1
 action: ALLOW
 rules:
 - from:
   - source:
       principals: ["*"]
   to:
   - operation:
       methods: ["GET", "POST"]
{{< /text >}}

### 일반 TCP 프로토콜에서 Istio 승인 사용

Istio 승인은 MongoDB 등 일반 TCP 프로토콜을 사용하는 워크로드를 지원합니다. TCP 또한 HTTP 워크로드와 
같은 방식으로 승인 정책을 설정해주면 됩니다. 주의해야 할 점은 HTTP 워크로드에만 적용 가능한 
몇몇 필드와 조건 입니다. 이들은:

- 승인 정책 내 source 섹션의 `request_principals` 필드
- 승인 정책 내 operation 섹션의 `hosts`, `methods`, 그리고 `paths` 필드

지원 중인 조건은 [조건 페이지](/docs/reference/config/security/conditions/)에 나열되어 
있습니다. HTTP 전용 필드를 TCP 워크로드에 적용할 경우, Istio는 승인 정책 내 이들 필드를 무시하게 
됩니다.

`27017`포트에서 MongoDB 서비스가 운영된다고 가정할 때, 다음 예제는 Istio 메쉬 내 
`bookinfo-ratings-v2` 서비스만 MongoDB 워크로드에 접근을 허용하는 승인 정책 설정을 보여줍니다.

{{< text yaml >}}
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: mongodb-policy
  namespace: default
spec:
 selector:
   matchLabels:
     app: mongodb
 action: ALLOW
 rules:
 - from:
   - source:
       principals: ["cluster.local/ns/default/sa/bookinfo-ratings-v2"]
   to:
   - operation:
       ports: ["27017"]
{{< /text >}}

### 상호 TLS에 대한 의존성

Istio는 상호 TLS를 사용하여 데이터를 클라이언트로부터 서버까지 안전하게 전달합니다.  
상호 TLS는 반드시 승인 정책의 아래 필드 사용 이전 활성화 시켜야 합니다:

- `source` 섹션 내 `principals`과 `notPrincipals` 필드
- `source` 섹션 내 `namespaces`와 `notNamespaces` 필드
- `source.principal` 사용자 지정 조건
- `source.namespace` 사용자 지정 조건

허용(permissive) 상호 TLS 모드에서 일반 텍스트 트래픽이 사용될 때 정책 예상치 못한 요청 거부 혹은 정책 우회를 막기 위해 
`PeerAuthentication` 내 위 필드들은 **strict(엄격한)** 상호 TLS 모드로 설정할 것을 강하게 권고합니다.

엄격한 상호 TLS 모드 활성화가 안되는 경우에 대한 대안이나 더 자세한 내용은 
[보안 권고](/news/security/istio-security-2021-004)를 참고하세요.

## 더 알아보기

기본 개념을 익힌 후 검토해 볼 만한 내용은 아래와 같습니다:

- [인증](/docs/tasks/security/authentication/authn-policy)과 
  [승인](/docs/tasks/security/authorization) 내 과제를 따라하며 보안 정책을 시도해 보세요.

- 메쉬 내 보안 향상에 도움이 되는 보안 [정책 예제](/docs/ops/configuration/security/security-policy-examples)를 
  익혀보세요.

- 어떤 것이 잘 작동되지 않을 때 [일반적인 문제](/docs/ops/common-problems/security-issues/)를 읽고 보안 정책 이슈에 
  대한 문제해결력을 기르세요.
