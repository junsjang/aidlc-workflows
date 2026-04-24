# Integration Test Ordering Rules

## Overview

These integration test ordering rules are cross-cutting constraints that apply to multi-unit projects during the Build and Test phase. They ensure that integration tests are executed in dependency order, preventing cascading failures and enabling systematic root-cause identification when integration issues arise.

Integration test ordering uses the unit dependency graph (from `unit-of-work-dependency.md`) to determine the correct sequence for testing inter-unit connections. By testing lower-dependency units first and progressively adding higher-level integrations, teams can isolate failures to specific unit boundaries.

**Enforcement**: At the Build and Test stage, the model MUST verify compliance with these rules before presenting the stage completion message to the user.

### Blocking ITO Finding Behavior

A **blocking ITO finding** means:
1. The finding MUST be listed in the stage completion message under an "ITO Findings" section with the ITO rule ID and description
2. The stage MUST NOT present the "Continue to Next Stage" option until all blocking findings are resolved
3. The model MUST present only the "Request Changes" option with a clear explanation of what needs to change
4. The finding MUST be logged in `aidlc-docs/audit.md` with the ITO rule ID, description, and stage context

If an ITO rule is not applicable to the current project (e.g., single-unit project), mark it as **N/A** in the compliance summary — this is not a blocking finding.

### Default Enforcement

All rules in this document are **blocking** by default. If any rule's verification criteria are not met, it is a blocking ITO finding — follow the blocking finding behavior defined above.

### Verification Criteria Format

Verification items in this document are plain bullet points describing compliance checks. Each item should be evaluated as compliant or non-compliant during review.

---

## Rule ITO-01: Dependency-Based Test Ordering

**Rule**: Unit 간 연동 테스트는 `unit-of-work-dependency.md`의 의존성 그래프를 위상 정렬(topological sort)하여 하위 의존성부터 순서대로 실행해야 합니다.

의존성 그래프에서 진입 차수(in-degree)가 0인 Unit부터 시작하여, 해당 Unit의 연동 테스트가 통과한 후에야 이를 의존하는 상위 Unit의 연동 테스트를 실행합니다. 순환 의존성이 존재하는 경우, 해당 순환 그룹을 하나의 테스트 단위로 묶어 동시에 테스트합니다.

**Verification**:
- `integration-test-order.md`에 위상 정렬된 테스트 실행 순서가 명시되어 있음
- 순서가 `unit-of-work-dependency.md`의 의존성 방향과 일치함
- 순환 의존성이 있는 경우 해당 그룹이 식별되고 처리 방식이 문서화되어 있음
- 각 테스트 단계의 선행 조건(prerequisite)이 명시되어 있음

---

## Rule ITO-02: Contract Verification First

**Rule**: 각 연동 테스트 전에 `unit-contracts.md`에 정의된 Contract를 먼저 검증합니다. Contract 검증이 실패하면 해당 연동 테스트를 blocking으로 처리하고, Contract 불일치를 먼저 해결해야 합니다.

Contract 검증 항목:
- Schema 일치 여부: Producer가 생성하는 데이터의 스키마가 Contract에 정의된 스키마와 일치하는가
- 데이터 형식 준수 여부: 실제 데이터가 Contract에 명시된 형식(JSON, Parquet, Avro 등)을 따르는가
- 필수 필드 존재 여부: Contract에 정의된 필수 필드가 모두 포함되어 있는가
- 버전 호환성: Producer와 Consumer가 동일한 Contract 버전을 참조하는가

`unit-contracts.md`가 존재하지 않는 경우(Contract 정의 Skill을 실행하지 않은 경우), 이 규칙은 **N/A**로 처리합니다. 단, Contract 없이 연동 테스트를 진행하는 경우 해당 사실을 `integration-test-order.md`에 명시해야 합니다.

**Verification**:
- 각 연동 테스트 시나리오에 Contract 검증 단계가 포함되어 있음 (unit-contracts.md 존재 시)
- Contract 검증 실패 시 해당 연동 테스트를 blocking으로 처리하는 절차가 명시되어 있음
- unit-contracts.md가 없는 경우 N/A로 표기되고 그 사유가 문서화되어 있음

---

## Rule ITO-03: Integration Test Artifacts

**Rule**: Build and Test 단계에서 다음 산출물을 생성해야 합니다:

| 산출물 | 위치 | 설명 |
|---|---|---|
| `integration-test-order.md` | `aidlc-docs/construction/build-and-test/` | 의존성 기반 테스트 실행 순서 (위상 정렬 결과, Layer별 실행 계획) |
| `integration-test-matrix.md` | `aidlc-docs/construction/build-and-test/` | Unit 쌍별 테스트 시나리오 매트릭스 |

### integration-test-order.md 필수 섹션

1. **의존성 그래프 요약**: `unit-of-work-dependency.md`에서 추출한 의존성 관계
2. **위상 정렬 결과**: Layer별 테스트 실행 순서
3. **Layer별 실행 계획**: 각 Layer의 테스트 대상, 선행 조건, 통과 기준
4. **순환 의존성 처리**: 순환 그룹 식별 및 처리 방식 (해당 시)
5. **Contract 검증 상태**: unit-contracts.md 참조 여부 및 검증 계획

### integration-test-matrix.md 필수 섹션

1. **Unit 쌍별 연동 매트릭스**: 모든 의존 관계가 있는 Unit 쌍 목록
2. **테스트 시나리오**: 각 Unit 쌍별 정상/비정상 시나리오
3. **데이터 흐름**: 각 연동 지점의 입출력 데이터 형식
4. **실패 시나리오**: 연동 실패 시 예상 동작 및 에러 처리

**Verification**:
- `integration-test-order.md`가 생성되어 있음
- `integration-test-matrix.md`가 모든 Unit 간 연동 쌍을 커버함
- 각 산출물이 위 필수 섹션을 포함하고 있음
- 산출물의 Unit 목록이 `unit-of-work-dependency.md`의 Unit 목록과 일치함

---

## Rule ITO-04: Staged Integration (Layered Execution)

**Rule**: 연동 테스트는 단계적으로 실행합니다. 각 Layer가 통과해야 다음 Layer로 진행합니다.

| Layer | 설명 | 예시 |
|---|---|---|
| Layer 0 | 외부 의존성 없는 Unit (독립 테스트) | 데이터 수집 Unit, 공통 라이브러리 Unit |
| Layer 1 | Layer 0에만 의존하는 Unit | 데이터 처리/변환 Unit |
| Layer 2 | Layer 0~1에 의존하는 Unit | 비즈니스 로직 Unit, API Unit |
| Layer N | Layer N-1까지 의존하는 Unit | ML/분석 Unit, 통합 UI Unit |

### Layer 통과 기준

각 Layer의 통과 기준은 다음을 포함해야 합니다:
- 해당 Layer의 모든 Unit 간 연동 테스트 통과
- Contract 검증 통과 (unit-contracts.md 존재 시)
- 에러 처리 시나리오 검증 완료
- 성능 기준 충족 (NFR에 정의된 경우)

### Layer 간 게이트

- Layer N의 모든 테스트가 통과해야 Layer N+1로 진행
- Layer N에서 실패 발생 시, 실패 원인을 해당 Layer 내에서 해결한 후 재실행
- 실패가 하위 Layer의 문제로 판명되면, 해당 하위 Layer부터 재실행

**Verification**:
- `integration-test-order.md`에 Layer별 실행 계획이 명시되어 있음
- 각 Layer의 통과 기준이 정의되어 있음
- Layer 간 게이트(통과 조건)가 명확함
- Layer 할당이 `unit-of-work-dependency.md`의 의존성 구조와 일치함

---

## Enforcement Integration

These rules are cross-cutting constraints that apply to the following AI-DLC stages:

| Stage | Applicable Rules | Enforcement |
|---|---|---|
| Build and Test (Planning) | ITO-01, ITO-02, ITO-03, ITO-04 | Integration test plan must include dependency-ordered execution |
| Build and Test (Execution) | ITO-01, ITO-04 | Tests must be executed in the defined Layer order |
| Build and Test (Completion) | ITO-03 | Required artifacts must be generated and complete |

At the Build and Test stage:
- Evaluate all ITO rule verification criteria against the artifacts produced
- Include an "ITO Compliance" section in the stage completion summary listing each rule as compliant, non-compliant, or N/A
- If any rule is non-compliant, this is a blocking ITO finding — follow the blocking finding behavior defined in the Overview
- Include ITO rule references in test planning and execution documentation
