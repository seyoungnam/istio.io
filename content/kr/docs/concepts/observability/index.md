---
title: 관찰 가능성
description: Istio가 제공하는 원격 측정 정보 및 모니터링 기능을 설명.
weight: 40
keywords: [telemetry,metrics,logs,tracing]
aliases:
    - /docs/concepts/policy-and-control/mixer.html
    - /docs/concepts/policy-and-control/mixer-config.html
    - /docs/concepts/policy-and-control/attributes.html
    - /docs/concepts/policies-and-telemetry/overview/
    - /docs/concepts/policies-and-telemetry/config/
    - /docs/concepts/policies-and-telemetry/
owner: istio/wg-policies-and-telemetry-maintainers
test: n/a
---

Istio는 메쉬 내 일어나는 모든 서비스 통신에 대한 상세한 원격 측정 정보(telemetry)를 제공합니다. 이 원격 측정 정보는 서비스 동작의 *관찰 가능성*을 제공하기 때문에 
서비스 개발자에게 추가적인 부담을 주지 않고 운영자가 어플리케이션의 문제를 해결하고 어플리케이션의 유지 관리와 최적화를 할 수 있도록 지원합니다. Istio를 통해서 운영자는 
모니터링 서비스가 타 서비스 및 Istio 구성요소와 어떻게 상호작용 하는지에 대한 깊은 이해 수준을 배양할 수 있습니다.

Istio는 전반적인 서비스 메쉬 관찰 가능성을 제공할 목적으로 아래와 같은 유형의 원격 측정 정보를 생성합니다:

- [**측정지표(Metrics)**](#metrics). Istio는 모니터링의 네가지 "황금 시그널"(지연시간, 트래픽, 에러, 포화수준)에 기반한 서비스 측정지표를 생성합니다. 
  또한 [메쉬 제어 영역](/docs/ops/deployment/architecture/)에 대한 자세한 메트릭스도 제공합니다. 이러한 메트릭스를 볼 수 있는 기본적인 메쉬 모니터링 대시보드도 
  제공됩니다.  
- [**분산 추적(Distributed Traces)**](#distributed-traces). Istio는 각 서비스에 대해 분산 추적 범위를 생성하여, 운영자에게 메쉬 내의 호출 흐름 및 
  서비스 종속성에 대한 자세한 이해를 돕습니다.
- [**접근 로그(Access Logs)**](#access-logs). 트래픽이 메쉬 내 서비스로 유입되면 Istio는 원천 및 대상 메타데이터 등 각 요청에 대한 전체 기록을 생성할 수 
  있습니다. 이 정보를 통해 운영자는 개별 [워크로드 인스턴스](/docs/reference/glossary/#workload-instance) 수준까지 서비스 동작을 감사할 수 있습니다. 

## 측정지표(Metrics)

측정지표는 동작을 전체적으로 모니터링하고 이해하는 방법을 제공합니다.

서비스 동작을 모니터링하기 위해 Istio는 Istio 서비스 메쉬 내외부로 흐르는 모든 서비스 트래픽에 대한 지표를 생성합니다. 이러한 지표는 전체 트래픽 볼륨, 트래픽 내 
오류율, 요청에 대한 응답 시간 등의 동작에 대한 정보를 제공합니다. 

메쉬 내 서비스 동작을 모니터링하는 것 외에도, 메쉬 자체의 동작을 모니터링하는 것도 중요합니다. Istio 구성요소는 자체 내부 동작에 대한 측정지표를 내보냄으로서 
메쉬 제어 영역의 상태와 기능에 대한 통찰력을 제공합니다.

### 프록시 수준 측정지표(Proxy-level metrics)

Istio 측정지표 수집은 사이드카 프록시(Envoy)로 시작됩니다. 각 프록시는 프록시를 통과하는 모든 트래픽(인바운드 및 아웃바운드 모두)에 대한 풍부한 측정지표 세트를 
생성합니다. 또한 프록시는 설정 및 상태 정보 등 프록시 자체의 관리 기능에 대한 자세한 통계도 제공합니다.

Envoy가 생성한 측정지표는 listeners나 clusters와 같은 Envoy 자원에 대한 기초 구성단위 수준에서의 모니터링 결과입니다. 따라서 Envoy 측정지표를 모니터링 
하려면 메쉬 서비스와 Envoy 자원간 연결에 대한 이해가 필요합니다.

Istio를 사용하면 운영자는 각 워크로드 인스턴스에서 생성 및 수집되는 Envoy 측정지표를 선택할 수 있습니다. 기본적으로 Istio는 과도한 측정지표 백엔드를 방지하고 
측정지표 수집과 관련된 CPU 오버헤드를 줄이기 위해 Envoy가 생성하는 통계 중 일부만 활성화 합니다. 그러나 운영자는 필요할 경우 수집된 프록시 측정지표 세트를 쉽게 
확장할 수 있습니다. 이는 메쉬 전반 모니터링의 전체 비용을 줄이면서도 네트워킹 동작에 대한 표적 디버깅을 가능케 합니다.

[Envoy 문서 사이트](https://www.envoyproxy.io/docs/envoy/latest/)는 [Envoy 통계 수집](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/observability/statistics.html?highlight=statistics)에 대한 전반적인 내용을 자세히 소개하고 있습니다. 
[Envoy 통계](/docs/ops/configuration/telemetry/envoy-stats/)에 대한 운영 가이드는 프록시 수준 측정지표 생성에 대한 제어와 관련하여 더 많은 정보를 
제공합니다.

프록시 수준 측정지표의 예시:

{{< text json >}}
envoy_cluster_internal_upstream_rq{response_code_class="2xx",cluster_name="xds-grpc"} 7163

envoy_cluster_upstream_rq_completed{cluster_name="xds-grpc"} 7164

envoy_cluster_ssl_connection_error{cluster_name="xds-grpc"} 0

envoy_cluster_lb_subsets_removed{cluster_name="xds-grpc"} 0

envoy_cluster_internal_upstream_rq{response_code="503",cluster_name="xds-grpc"} 1
{{< /text >}}

### 서비스 수준 측정지표(Service-level metrics)

프록시 수준 측정지표 외에도, Istio는 서비스간 통신 모니터링을 위한 서비스 기반 측정지표를 제공합니다. 이 측정지표들은 지연시간, 트래픽, 에러, 그리고 포화수준 등 
네가지 기본 서비스 모니터링 요건을 충족 합니다. Istio는 이들 측정지표 기반의 서비스 동작 모니터링을 위한 기본 [대시보드](/docs/tasks/observability/metrics/using-istio-dashboard/)를 제공합니다.
 
기본적으로 [표준 Istio 측정지표](/docs/reference/config/metrics/)가 [프로메테우스](/docs/ops/integrations/prometheus/)로 제공됩니다.

서비스 수준 측정지표의 사용은 전적으로 선택사항일 뿐입니다. 운영자는 개별 요구사항에 따라 이 측정지표의 생성과 수집을 해제하도록 선택할 수 있습니다. 

서비스 수준 측정지표 예시:

{{< text json >}}
istio_requests_total{
  connection_security_policy="mutual_tls",
  destination_app="details",
  destination_canonical_service="details",
  destination_canonical_revision="v1",
  destination_principal="cluster.local/ns/default/sa/default",
  destination_service="details.default.svc.cluster.local",
  destination_service_name="details",
  destination_service_namespace="default",
  destination_version="v1",
  destination_workload="details-v1",
  destination_workload_namespace="default",
  reporter="destination",
  request_protocol="http",
  response_code="200",
  response_flags="-",
  source_app="productpage",
  source_canonical_service="productpage",
  source_canonical_revision="v1",
  source_principal="cluster.local/ns/default/sa/default",
  source_version="v1",
  source_workload="productpage-v1",
  source_workload_namespace="default"
} 214
{{< /text >}}

### 제어 영역 측정지표(Control plane metrics)

Istio 제어 영역 또한 자체 모니터링 측정지표의 수집이 가능합니다. 이러한 측정지표를 사용하면 Istio 자체의 동작을 모니터링할 수 있습니다(메쉬 내 서비스 동작과는 
별개). 

어떤 측정지표를 수집할 수 있는지에 대한 자세한 내용은 [참조 문헌](/docs/reference/commands/pilot-discovery/#metrics)을 참고하시기 바랍니다.

## 분산 추적(Distributed traces)

분산 추적은 메쉬 내에서 개별 요청이 어디서 어떻게 처리되는지 모니터링 합니다. 이로써 서비스의 동작을 모니터링 하고 이해할 수 있도록 도와 줍니다. 
메쉬 운영자는 추적 기능을 활용하여 서비스 종속성과 지연시간의 원인을 이해할 수 있습니다. 

Istio는 Envoy 프록시를 통해 분산 추적 기능을 지원합니다. 어플리케이션이 적절한 요청 조건(request context)을 프록시에 전달하기만 하면, 
프록시는 자신이 담당하고 있는 어플리케이션을 대신하여 자동으로 추적 범위(trace spans)를 생성합니다.

Istio는 [Zipkin](/docs/tasks/observability/distributed-tracing/zipkin/),
[Jaeger](/docs/tasks/observability/distributed-tracing/jaeger/), [Lightstep](/docs/tasks/observability/distributed-tracing/lightstep/), 그리고 [Datadog](https://www.datadoghq.com/blog/monitor-istio-with-datadog/) 등 다양한 추적 벡엔드 서비스를 
지원합니다. 운영자는 추적 생성을 위한 샘플링 속도(즉, 요청별 추적 데이터가 생성되는 속도)를 제어할 수 있는데, 이를 통해 메쉬 내 생성되는 추적 데이터의 
양과 속도를 제어할 수 있습니다.

Istio의 분산 추적에 대한 더 많은 정보는 [분산 추적에 관한 FAQ](/about/faq/#distributed-tracing)에서 찾아보실 수 있습니다.

단일 요청에 대해 Istio가 생성한 분산추적의 예는 다음과 같습니다:

{{< image link="/docs/tasks/observability/distributed-tracing/zipkin/istio-tracing-details-zipkin.png" caption="단일 요청에 대한 분산 추적" >}}

## 엑세스 로그(Access logs)

엑세스 로그는 개별 워크로드 인스턴스의 관점에서 동작을 모니터링하고 이해하는 방법을 제공합니다.

Istio는 서비스 트래픽에 대한 엑세스 로그를 설정 가능한 형태로 생성하며, 이를 통해 운영자는 로깅의 방법, 내용, 시기, 그리고 위치를 완벽하게 
제어할 수 있습니다. 더 자세한 내용은 [Envoy 엑세스 로그 얻기](/docs/tasks/observability/logs/access-log/)를 참고하세요.

Istio 엑세스 로그 예시:

{{< text plain >}}
[2019-03-06T09:31:27.360Z] "GET /status/418 HTTP/1.1" 418 - "-" 0 135 5 2 "-" "curl/7.60.0" "d209e46f-9ed5-9b61-bbdd-43e22662702a" "httpbin:8000" "127.0.0.1:80" inbound|8000|http|httpbin.default.svc.cluster.local - 172.30.146.73:80 172.30.146.82:38618 outbound_.8000_._.httpbin.default.svc.cluster.local
{{< /text >}}
