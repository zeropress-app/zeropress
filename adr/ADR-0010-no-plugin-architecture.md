# ADR-0010: No Plugin Architecture

## Status

Accepted

## Context

ZeroPress는 Cloudflare 기반의 CMS로, 다음과 같은 특성을 지닌다:

* Cloudflare Workers / Pages / R2 중심의 인프라
* Immutable artifact 기반의 사용자 사이트 빌드 구조
* Admin은 Control Plane, User Site는 Output Artifact
* 보안 단순성, 예측 가능한 실행 모델을 핵심 가치로 둠

기존 CMS(예: WordPress)는 플러그인 기반 확장 모델을 채택하고 있으나, 이는 다음과 같은 문제를 동반한다:

* 임의 코드 실행으로 인한 보안 리스크
* 실행 순서 / 훅 / 의존성으로 인한 복잡도 증가
* 런타임 성능 및 안정성 저하
* SaaS / Serverless 환경과의 구조적 충돌

ZeroPress는 메일 전송, 게시물 퍼블리시(R2, GitHub, S3 등)와 같은 확장을 필요로 하나, 이는 "기능 추가"라기보다 "시스템 동작 방식의 선택"에 가깝다.

## Decision

ZeroPress는 **Plugin 시스템을 도입하지 않는다.**

대신, 확장 개념을 다음 두 범주로 명확히 분리한다:

1. **Providers (Settings 기반)**

   * Mail Provider
   * Publish Provider
   * 기타 시스템 통합 요소

2. **Themes (UI/UX 확장)**

   * 사용자 참여형 Theme Market 허용
   * UI/표현 레이어에 한정된 확장

Providers는 Admin의 `Settings` 하위에 위치하며, 시스템의 동작 방식을 설정하는 역할만 수행한다.

## Rationale

### Plugin을 배제한 이유

* Cloudflare Workers 환경에서는 임의 런타임 코드 로딩이 구조적으로 부적합
* 플러그인 훅 모델은 ZeroPress의 단순한 빌드 파이프라인과 상충
* 보안 및 권한 모델이 과도하게 복잡해짐
* 운영자가 시스템 전체 신뢰성을 보장하기 어려움

### Provider 모델의 적합성

* Provider는 전략(Strategy) 선택 문제에 해당
* 활성 Provider는 명시적이며 예측 가능
* Enable/Disable, Hook 체계 불필요
* Infrastructure-as-Config 개념과 일치

### Theme Market만 허용하는 이유

* Theme은 샌드박싱 가능
* 시스템 권한 불필요
* 사용자 경험 확장에 집중 가능
* 마켓 형태로 확장하더라도 Core 안정성 유지

## Consequences

### Positive

* Admin 및 Core 시스템 단순화
* 보안 모델 명확화
* 빌드 및 퍼블리시 파이프라인의 예측 가능성 확보
* ZeroPress의 Cloud-native 정체성 강화

### Negative / Trade-offs

* WordPress식 플러그인 확장 기대를 충족하지 않음
* 일부 고급 커스터마이징은 Core 변경 필요
* 서드파티 생태계 확장 속도가 느릴 수 있음

## Alternatives Considered

### Plugin System 도입

* ❌ 보안, 복잡성, Cloudflare 환경 부적합으로 기각

### Hook 기반 Extension API

* ❌ 런타임 복잡도 증가 및 디버깅 난이도 문제로 기각

### WASM Plugin

* ❌ 현재 ZeroPress 단계에서는 과도한 설계로 판단

## Related Decisions

* ADR-000Y: Provider-based Publishing Architecture
* ADR-000Z: Immutable Build Artifact Model

## Notes

ZeroPress의 확장은 "무엇을 추가할 수 있는가"보다
"무엇을 **허용하지 않는가**"를 명확히 하는 데서 출발한다.

본 ADR은 ZeroPress의 장기적 안정성과 보안, 그리고 Cloudflare 기반 아키텍처 일관성을 보장하기 위한 결정이다.
