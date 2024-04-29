---
title: Docker Desktop
description: Instructions to set up Docker Desktop for Istio.
weight: 15
skip_seealso: true
aliases:
    - /docs/setup/kubernetes/prepare/platform-setup/docker-for-desktop/
    - /docs/setup/kubernetes/prepare/platform-setup/docker/
    - /docs/setup/kubernetes/platform-setup/docker/
keywords: [platform-setup,kubernetes,docker-desktop]
owner: istio/wg-environments-maintainers
test: no
---

1. Docker Desktop에서 Istio를 실행하기 위해, [지원 중인 쿠버네티스 버전](/docs/releases/supported-releases#support-status-of-istio-releases)({{< supported_kubernetes_versions >}})중 한 버전을 설치하세요.

1. Docker Desktop에 내장된 쿠버네티스에서 Istio를 실행하려면 Docker의 메모리 제한을 늘려야 합니다. Docker Desktop의 *Settings...*에서 *Resources->Advanced* 섹션에서 메모리는 최소 8.0 `GB`, `CPUs`는 최소 4로 설정하세요.

    {{< image width="60%" link="./dockerprefs.png"  caption="Docker Preferences"  >}}

    {{< warning >}}
    최소 메모리 요구조건은 정해져 있지 않으나, Istio와 Bookinfo를 구동하는데 8 `GB`면 충분합니다.
    Docker Desktop에 충분한 메모리가 할당되지 않으면 아래 에러들이 발생할 수 있습니다:

    - 이미지 다운로드 실패 (image pull failures)
    - 헬스체크 시간초과 실패 (healthcheck timeout failures)
    - 호스트에서의 kubectl 실패 (kubectl failures on the host)
    - 하이퍼바이저의 일반적인 네트워크 불안정성 (general network instability of the hypervisor)

    
    아래 명령어를 통해 Docker Desktop 내에서 추가 자원을 확보할 수 있습니다:

    {{< text bash >}}
    $ docker system prune
    {{< /text >}}

    {{< /warning >}}
