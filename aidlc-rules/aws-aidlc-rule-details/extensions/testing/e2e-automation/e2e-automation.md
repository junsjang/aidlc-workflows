# E2E Test Automation Rules

## Overview

These E2E test automation rules provide a structured approach to deriving end-to-end test scenarios from User Story Acceptance Criteria. They ensure that E2E tests are systematically generated, environment-aware, and traceable back to the original requirements.

E2E tests serve as the final integration verification baseline — when all E2E tests pass, the entire system's integration is considered verified. When E2E tests fail, the integration test ordering artifacts (`integration-test-order.md`) provide a systematic path for root-cause identification.

**Enforcement**: At the Build and Test stage, the model MUST verify compliance with these rules before presenting the stage completion message to the user.

### Blocking E2E Finding Behavior

A **blocking E2E finding** means:
1. The finding MUST be listed in the stage completion message under an "E2E Findings" section with the E2E rule ID and description
2. The stage MUST NOT present the "Continue to Next Stage" option until all blocking findings are resolved
3. The model MUST present only the "Request Changes" option with a clear explanation of what needs to change
4. The finding MUST be logged in `aidlc-docs/audit.md` with the E2E rule ID, description, and stage context

If an E2E rule is not applicable to the current project (e.g., no User Stories defined), mark it as **N/A** in the compliance summary — this is not a blocking finding.

### Default Enforcement

All rules in this document are **blocking** by default. If any rule's verification criteria are not met, it is a blocking E2E finding — follow the blocking finding behavior defined above.

### Partial Enforcement (Opt-In B 선택 시)

Opt-In에서 **B) Partial**을 선택한 경우, 규칙별 적용 범위는 다음과 같습니다:

| Rule | Partial 시 적용 | 설명 |
|---|---|---|
| E2E-01 | **Blocking** | 시나리오 도출은 E2E의 핵심이므로 항상 적용 |
| E2E-02 | **N/A** | 환경별 전략은 건너뜀 (환경 컨텍스트 미수집 시 적합) |
| E2E-03 | **Blocking** | 연동 기준선 역할은 환경과 무관하므로 항상 적용 |
| E2E-04 | **Blocking** (환경별 설정 제외) | `e2e-scenarios.md`와 `e2e/` 테스트 코드는 필수, `e2e/config/` 환경별 설정은 N/A |

### Verification Criteria Format

Verification items in this document are plain bullet points describing compliance checks. Each item should be evaluated as compliant or non-compliant during review.

---

## Rule E2E-01: E2E Test Scenario Derivation

**Rule**: User Stories의 Given-When-Then AC를 기반으로 E2E 테스트 시나리오를 자동 도출합니다. 각 Must/Should 우선순위 스토리에 대해 최소 1개 E2E 시나리오가 필수입니다.

시나리오 도출 기준:
- 각 Must/Should 우선순위 스토리에 대해 최소 1개 E2E 시나리오 필수
- 시나리오는 `stories.md`의 AC를 직접 참조
- Could 우선순위 스토리는 E2E 시나리오 선택적
- Won't 우선순위 스토리는 E2E 시나리오 제외
- 복수의 AC를 가진 스토리는 AC별로 별도 시나리오 또는 통합 시나리오 가능

**Verification**:
- `e2e-scenarios.md`에 모든 Must/Should 스토리의 E2E 시나리오가 존재
- 각 시나리오가 `stories.md`의 AC를 참조하고 있음
- 시나리오 수 ≥ Must 스토리 수 + Should 스토리 수
- Story ID ↔ E2E 시나리오 매핑 테이블이 존재하고 빠짐없이 대응됨
- Won't 스토리에 대한 E2E 시나리오가 포함되어 있지 않음

---

## Rule E2E-02: Environment-Aware E2E

**Rule**: `environment-context/environments.md`를 참조하여 환경별 E2E 전략을 정의합니다. 환경 컨텍스트가 존재하지 않는 경우(Environment Context Skill을 실행하지 않은 경우), 이 규칙은 **N/A**로 처리합니다.

| 환경 | E2E 전략 | 목적 |
|---|---|---|
| 로컬 | 클라우드 서비스 에뮬레이터 기반 E2E (예: LocalStack, Azurite, 로컬 컨테이너 등) | 빠른 피드백, 개발 중 검증 |
| 스테이징 | 실제 클라우드 서비스 대상 E2E | 실환경 연동 검증 |
| 프로덕션 | Canary/Synthetic 모니터링 | 배포 후 지속 검증 (Operations 단계) |

환경별 전략은 프로젝트의 기술 스택과 배포 환경에 맞게 조정합니다. 위 표는 참고 예시이며, 실제 환경 구성에 따라 전략이 달라질 수 있습니다.

**Verification**:
- `e2e-scenarios.md`에 환경별 E2E 전략 섹션이 존재 (environments.md 존재 시)
- 각 환경의 E2E 실행 방법이 문서화되어 있음
- environments.md가 없는 경우 N/A로 표기되고 그 사유가 문서화되어 있음

---

## Rule E2E-03: E2E Test as Integration Baseline

**Rule**: E2E 테스트 결과를 최종 연동 검증의 기준선으로 사용합니다. E2E 통과는 전체 시스템 연동 검증 완료를 의미하며, E2E 실패 시 연동 테스트 순서(`integration-test-order.md`)를 기반으로 원인을 추적합니다.

- E2E 통과 = 전체 시스템 연동 검증 완료
- E2E 실패 = 연동 문제 식별 → `integration-test-order.md` 기반 원인 추적 (존재 시)
- E2E 결과는 `build-and-test-summary.md`에 포함

`integration-test-order.md`가 존재하지 않는 경우(Integration Ordering Skill을 실행하지 않은 경우), E2E 실패 시 원인 추적은 일반적인 디버깅 절차를 따릅니다.

**Verification**:
- `build-and-test-summary.md`에 E2E 테스트 결과 섹션이 존재
- E2E 실패 시 원인 추적 경로가 문서화되어 있음
- E2E 통과/실패 상태가 명확히 기록되어 있음

---

## Rule E2E-04: E2E Artifacts

**Rule**: Build and Test 단계에서 다음 산출물을 생성해야 합니다:

| 산출물 | 위치 | 설명 |
|---|---|---|
| E2E 시나리오 목록 | `aidlc-docs/construction/build-and-test/e2e-scenarios.md` | Story ID ↔ E2E 시나리오 매핑 포함 |
| E2E 테스트 코드 | `e2e/` (워크스페이스 루트) | 실행 가능한 E2E 테스트 코드 |
| 환경별 설정 | `e2e/config/` | local, staging, prod 설정 (environments.md 존재 시) |

### e2e-scenarios.md 필수 섹션

1. **Story-E2E 매핑 테이블**: 모든 Must/Should 스토리와 E2E 시나리오의 매핑
2. **E2E 시나리오 상세**: 각 시나리오의 Given-When-Then, 테스트 데이터, 기대 결과
3. **환경별 E2E 전략**: 환경별 실행 방법 및 설정 (environments.md 존재 시, 없으면 N/A)
4. **E2E 실행 가이드**: 테스트 실행 명령어, 사전 조건, 정리 절차

### e2e/ 디렉토리 구조

```
e2e/
├── config/           # 환경별 설정 (environments.md 존재 시)
│   ├── local.env     # 로컬 환경 설정
│   ├── staging.env   # 스테이징 환경 설정
│   └── prod.env      # 프로덕션 환경 설정
├── scenarios/        # 시나리오별 테스트 코드
└── README.md         # E2E 테스트 실행 가이드
```

**Verification**:
- `e2e-scenarios.md`에 시나리오 목록이 Story ID와 매핑되어 있음
- `e2e/` 디렉토리에 테스트 코드가 존재
- `e2e/config/`에 환경별 설정 파일이 존재 (environments.md 존재 시)
- `e2e/README.md`에 실행 가이드가 포함되어 있음

---

## Enforcement Integration

These rules are cross-cutting constraints that apply to the following AI-DLC stages:

| Stage | Applicable Rules | Enforcement |
|---|---|---|
| Build and Test (Planning) | E2E-01, E2E-02, E2E-04 | E2E test plan must include scenario derivation and environment strategy |
| Build and Test (Execution) | E2E-01, E2E-03 | E2E tests must cover all Must/Should stories, results recorded |
| Build and Test (Completion) | E2E-03, E2E-04 | Required artifacts must be generated and complete |
| Code Generation | E2E-04 | E2E test code must be generated in `e2e/` directory |

At the Build and Test stage:
- Evaluate all E2E rule verification criteria against the artifacts produced
- Include an "E2E Compliance" section in the stage completion summary listing each rule as compliant, non-compliant, or N/A
- If any rule is non-compliant, this is a blocking E2E finding — follow the blocking finding behavior defined in the Overview
- Include E2E rule references in test planning and execution documentation
