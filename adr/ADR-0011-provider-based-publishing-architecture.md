# ADR-0011: Provider-based Publishing Architecture

## Status

Accepted

## Context

ZeroPress는 사용자 콘텐츠를 직접 서빙하는 CMS가 아니라, **사용자 사이트를 빌드하여 외부 스토리지 또는 플랫폼에 퍼블리시하는 시스템**이다.

핵심 전제는 다음과 같다:

* User Site는 Admin과 분리된 **빌드 산출물(Artifact)** 이다
* 퍼블리시는 런타임 요청이 아니라 **빌드 파이프라인의 마지막 단계**이다
* 퍼블리시 대상은 단일 인프라에 종속되지 않는다

초기 설계 단계에서 다음과 같은 퍼블리시 대상이 고려되었다:

* Cloudflare R2
* GitHub (GitHub Pages / Repository)
* Amazon S3
* 기타 Object Storage 또는 Static Hosting

이들은 서로 다른 API, 인증 방식, 디렉토리 구조를 가지며, 단일 구현으로 추상화하기 어렵다.

## Decision

ZeroPress는 퍼블리시 기능을 **Provider 기반 아키텍처**로 설계한다.

퍼블리시는 하나의 고정 구현이 아닌, **명시적으로 선택 가능한 Publish Provider**를 통해 수행된다.

* Publish Provider는 `Settings`에서 선택된다
* 한 시점에 하나의 Provider만 활성화된다
* 퍼블리시는 Build Pipeline의 명확한 단계로 정의된다

## Architecture Overview

```
Content → Build → Artifact → Publish (via Provider) → Destination
```

* Build 단계는 Provider와 무관하다
* Provider는 Artifact를 입력으로 받아 외부 시스템에 전송한다

## Provider Responsibilities

Publish Provider는 다음 책임을 가진다:

* 인증 정보 처리 (Secret / Token / Key)
* 퍼블리시 대상 경로 결정
* 파일 업로드 / 동기화
* 퍼블리시 결과 리포트

Provider는 다음을 수행하지 않는다:

* 콘텐츠 변환
* 템플릿 처리
* 런타임 요청 처리

## Provider Interface (Conceptual)

Publish Provider는 공통 인터페이스를 따른다:

* `publish(artifact, options)`
* `validateConfig()`
* `getStatus()`

세부 구현은 Provider별로 상이하나, Admin 및 Build Pipeline은 동일한 인터페이스를 통해 동작한다.

## Rationale

### 왜 Provider 모델인가

* 퍼블리시 대상은 전략 선택 문제
* 인프라 종속을 피하기 위함
* 설정 변경만으로 퍼블리시 대상 교체 가능

### 왜 Plugin이 아닌가

* Provider는 기능 확장이 아니라 **배포 대상 결정**
* 임의 코드 실행 불필요
* ZeroPress의 No Plugin 원칙과 일치

### 왜 Build 이후 단계인가

* 퍼블리시는 실패 가능성이 높은 외부 I/O 작업
* Build와 분리하여 재시도 및 롤백 가능
* Artifact 불변성 유지

## Consequences

### Positive

* 퍼블리시 로직의 명확한 분리
* 새로운 퍼블리시 대상 추가 용이
* Build 결과 재활용 가능
* 멀티 인프라 대응 가능

### Negative / Trade-offs

* Provider 인터페이스 설계 필요
* 일부 Provider는 기능 제한 존재
* 퍼블리시 실패 시 추가 상태 관리 필요

## Alternatives Considered

### 단일 퍼블리시 타겟 고정

* ❌ 인프라 종속 문제로 기각

### Plugin 기반 퍼블리시

* ❌ 보안 및 복잡성 문제로 기각 (ADR-000X 참조)

### 런타임 퍼블리시

* ❌ 성능 및 안정성 문제로 기각

## Related Decisions

* ADR-000X: No Plugin Architecture
* ADR-0010: Immutable Build Artifact Model

## Notes

Provider-based Publishing Architecture는 ZeroPress를
"Static Site Generator"가 아닌
"Build & Publish Control Plane"으로 정의하는 핵심 결정이다.

본 ADR은 ZeroPress의 멀티 인프라 대응 전략의 기반이 된다.
