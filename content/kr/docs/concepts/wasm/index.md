---
title: 확장성
description: Istio의 웹어셈블리 플러그인 시스템에 대한 설명.
weight: 50
keywords: [wasm,webassembly,emscripten,extension,plugin,filter]
owner: istio/wg-policies-and-telemetry-maintainers
test: n/a
---

웹어셈블리(WebAssembly)는 Istio 프록시(Envoy)를 확장하는 데 사용할 수 있는 샌드박스 기술입니다. Proxy-Wasm 샌드박스 API는 Istio의 
기본 확장 메커니즘으로 Mixer를 대체합니다.

웹어셈블리 샌드박스의 목표는:

- **효율성** - 짧은 대기 시간, 낮은 CPU 및 메모리 오버헤드.
- **기능** - 정책 시행, 원격 측정 데이터 수집, 페이로드(payload) 변경.
- **격리** - 한 플러그인의 프로그래밍 오류나 충돌은 다른 플러그인에 미영향.
- **설정** - 플러그인은 다른 Istio API와 호환되는 API를 사용하여 설정. 확장은 동적으로 설정.
- **운영자** - 확장은 log-only, fail-open, 혹은 fail-close 모드로 카나리아 및 일반 배포 가능.
- **확장 개발자** - 다양한 언어로 플러그인 작성 가능.

이 [담화 영상](https://youtu.be/XdWmm_mtVXI)은 웹어셈블리 통합 구조에 관한 소개 영상 입니다.

## 구조 훑어보기

Istio 확장 기능(Proxy-Wasm 플러그인)은 여러 구성요소를 지닙니다:

- 필터용 Proxy-Wasm 플러그인을 구축하기 위한 **필터 서비스 제공자 인터페이스(SPI)**.
- Envoy에 내장된 V8 Wasm 런타임 **샌드박스**.
- 헤더, 트레일러(trailfers), 메타데이터용 **호스트 APIs**.
- gRPC 및 HTTP 호출을 위한 **콜아웃 APIs**.
- 측정과 모니터링을 위한 **통계 및 로깅 APIs**.

{{< image width="80%" link="./extending.svg" caption="Istio/Envoy 확장" >}}

## 예시

필터용 C++ Proxy-Wasm의 예시는 [여기](https://github.com/istio-ecosystem/wasm-extensions/tree/master/example)에서 
보실 수 있습니다. [이 가이드](https://github.com/istio-ecosystem/wasm-extensions/blob/master/doc/write-a-wasm-extension-with-cpp.md)를 참고하여 C++ Wasm 확장을 구현하세요.

## 생태계

- [Istio 생태계 Wasm 확장](https://github.com/istio-ecosystem/wasm-extensions)
- [Proxy-Wasm ABI 스펙](https://github.com/proxy-wasm/spec)
- [Proxy-Wasm C++ SDK](https://github.com/proxy-wasm/proxy-wasm-cpp-sdk)
- [Proxy-Wasm Rust SDK](https://github.com/proxy-wasm/proxy-wasm-rust-sdk)
- [Proxy-Wasm 어셈블리 스크립트 SDK](https://github.com/solo-io/proxy-runtime)
- [WebAssembly Hub](https://docs.solo.io/web-assembly-hub/latest/tutorial_code/)
- [네트워크 프록시를 위한 WebAssembly 확장 (비디오)](https://www.youtube.com/watch?v=OIUPf8m7CGA)
