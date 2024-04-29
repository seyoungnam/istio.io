---
---
추적(trace) 데이터를 보기 위해선, 당신의 서비스에 요청을 보내야 합니다. 요청 횟수는 Istio의 샘플링 비율에 따라 다르며 
[Telemetry API](/kr/docs/tasks/observability/telemetry/)를 사용하여 조정할 수 있습니다. 디폴트 샘플링 비율인 1%의 경우, 
첫번째 추적 데이터를 보기 위해서는 최소 100번의 요청을 보내야 합니다. `productpage` 서비스에 최소 100개의 요청을 보내고자 한다면, 
아래 명령어를 사용하십시오:

{{< text bash >}}
$ for i in $(seq 1 100); do curl -s -o /dev/null "http://$GATEWAY_URL/productpage"; done
{{< /text >}}
