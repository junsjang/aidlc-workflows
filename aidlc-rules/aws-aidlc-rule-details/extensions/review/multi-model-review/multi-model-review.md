# Multi-Model Review Loop Rules

## Overview

These multi-model review rules provide a structured approach to automated, multi-perspective artifact review. Instead of relying on a single model's single pass, multiple reviewers (different models or different review prompts) examine artifacts from complementary angles — structural integrity, logical consistency, and omission detection — then automatically fix mechanical errors and escalate only judgment-requiring items to the human.

The core principle is: **검토는 AI가 반복, 사람은 판단만 한다.** (AI iterates on review; humans only make decisions.)

**Enforcement**: At each stage completion, the model MUST verify compliance with these rules before presenting the stage completion message to the user.

### Blocking MMR Finding Behavior

A **blocking MMR finding** means:
1. The finding MUST be listed in the stage completion message under an "MMR Findings" section with the MMR rule ID and description
2. The stage MUST NOT present the "Continue to Next Stage" option until all blocking findings are resolved
3. The model MUST present only the "Request Changes" option with a clear explanation of what needs to change
4. The finding MUST be logged in `aidlc-docs/audit.md` with the MMR rule ID, description, and stage context

If an MMR rule is not applicable to the current context (e.g., no artifacts to review), mark it as **N/A** in the compliance summary — this is not a blocking finding.

### Default Enforcement

All rules in this document are **blocking** by default. If any rule's verification criteria are not met, it is a blocking MMR finding — follow the blocking finding behavior defined above.

### Review Only Enforcement (Opt-In B 선택 시)

Opt-In에서 **B) Review Only**를 선택한 경우, 규칙별 적용 범위는 다음과 같습니다:

| Rule | Review Only 시 적용 | 설명 |
|---|---|---|
| MMR-01 | **Blocking** | 다관점 검토는 핵심이므로 항상 적용 |
| MMR-02 | **Blocking** | 이슈 분류는 검토 결과 정리에 필수 |
| MMR-03 | **N/A** | 자동 수정 루프 건너뜀 — 모든 수정은 사람 승인 후 적용 |
| MMR-04 | **Blocking** | 사람 에스컬레이션 프로토콜은 항상 적용 |

### Verification Criteria Format

Verification items in this document are plain bullet points describing compliance checks. Each item should be evaluated as compliant or non-compliant during review.

---

## Rule MMR-01: Automated Multi-Perspective Review Cycle

**Rule**: 각 단계 산출물 완료 후, 최소 2개 이상의 서로 다른 검토 관점으로 자동 검토를 수행합니다. 서로 다른 모델을 사용하거나, 동일 모델에 서로 다른 검토 프롬프트를 적용하여 다관점 검토를 실현합니다.

### 검토자 역할 정의

| 역할 | 검토 관점 | 적합 모델 특성 | 검토 항목 예시 |
|---|---|---|---|
| **구조 검토자** | 문서 구조, 포맷, 참조 무결성 | 긴 컨텍스트 처리에 강한 모델 | 섹션 누락, 참조 링크 깨짐, ID 불일치, 포맷 오류 |
| **논리 검토자** | 요구사항-설계-구현 간 논리 정합성 | 추론 능력이 강한 모델 | FR↔Story 매핑 누락, 설계-구현 불일치, 우선순위 모순 |
| **누락 검토자** (선택) | 빠진 항목, 엣지 케이스, 암묵적 가정 | 창의적 탐색에 강한 모델 | 미정의 에러 처리, 누락된 비기능 요구사항, 경계 조건 |

### 실행 절차

1. 산출물 생성 완료
2. 검토자 A (구조 검토자): 문서 구조, 포맷, 참조 무결성 검토
3. 검토자 B (논리 검토자): 요구사항-설계-구현 간 논리 정합성 검토
4. (선택) 검토자 C (누락 검토자): 빠진 항목, 엣지 케이스 탐색
5. 이슈 통합 및 중복 제거
6. MMR-02에 따라 이슈 분류
7. MMR-03에 따라 자동 수정 (Full 모드 시)
8. MMR-04에 따라 사람에게 에스컬레이션

### 멀티 모델 구현 메커니즘

서브에이전트별로 서로 다른 모델을 지정하거나, 동일 모델에 서로 다른 검토 프롬프트를 적용합니다:

- **서로 다른 모델 사용**: 커스텀 에이전트 설정의 `model` 필드를 활용하여 검토자별 모델 지정
- **동일 모델 + 다른 프롬프트**: 모델이 제한적인 환경에서는 검토 관점별 프롬프트 분리로 다관점 효과 확보
- **폴백**: `model` 필드가 지정되지 않거나 해당 모델이 사용 불가한 경우, 기본 모델로 폴백. 프롬프트 관점 분리에 의한 다관점 검토 효과는 유지

**Verification**:
- `review-log.md`에 각 검토자의 검토 결과가 기록되어 있음
- 최소 2개 이상의 서로 다른 검토 관점이 적용되었음
- 각 검토자의 역할(구조/논리/누락)이 명시되어 있음
- 검토 대상 산출물 목록이 기록되어 있음

---

## Rule MMR-02: Issue Classification

**Rule**: 발견된 이슈를 자동 수정 가능 여부에 따라 3단계로 분류합니다.

| 분류 | 설명 | 처리 방식 | 예시 |
|---|---|---|---|
| **Auto-Fix** | 기계적으로 수정 가능한 오류 | AI가 자동 수정 후 재검토 (Full 모드) 또는 수정안 제시 후 사람 승인 (Review Only 모드) | ID 불일치, 참조 깨짐, 포맷 오류, 누락된 매핑 항목 추가 |
| **Suggest-Fix** | 수정안을 제시할 수 있으나 확인 필요 | 수정안과 함께 사람에게 제시 | 모호한 요구사항 해석, 대안이 여러 개인 설계 결정 |
| **Human-Required** | AI가 판단할 수 없는 사항 | 사람에게 에스컬레이션 | 비즈니스 우선순위 결정, 기술 스택 선택, 보안 정책 판단 |

### 분류 기준

- **Auto-Fix 판정 기준**: 수정 방법이 유일하고, 수정 결과가 기존 규칙/규약에 의해 검증 가능한 경우
- **Suggest-Fix 판정 기준**: 수정 방법이 복수이거나, 수정 결과가 비즈니스 맥락에 따라 달라질 수 있는 경우
- **Human-Required 판정 기준**: 기술적 판단이 아닌 비즈니스/정책 판단이 필요한 경우

**Verification**:
- 모든 이슈가 위 3가지 분류 중 하나로 태깅되어 있음
- Auto-Fix 항목은 수정 완료 상태이거나 수정안이 제시되어 있음 (모드에 따라)
- Human-Required 항목만 사람에게 에스컬레이션됨
- 각 이슈에 분류 사유가 기록되어 있음

---

## Rule MMR-03: Convergence Criteria (Auto-Fix Loop)

**Rule**: 자동 검토-수정 루프의 종료 조건을 정의합니다. Full 모드에서만 적용되며, Review Only 모드에서는 N/A입니다.

### 종료 조건

1. **수렴**: 연속 2회 검토에서 새로운 이슈가 발견되지 않음
2. **최대 반복**: 3회 반복 후에도 수렴하지 않으면 잔여 이슈를 사람에게 보고
3. **심각도 기반**: Critical 이슈 발견 시 즉시 루프 중단, 사람에게 에스컬레이션

### Auto-Fix 루프 절차

```
1. 검토 실행 (MMR-01)
2. 이슈 분류 (MMR-02)
3. Auto-Fix 항목 자동 수정
4. 수정된 산출물에 대해 재검토 (MMR-01 재실행)
5. 새로운 이슈 발견 여부 확인
   - 새로운 이슈 없음 → 수렴, 루프 종료
   - 새로운 이슈 있음 → 반복 횟수 확인
     - 3회 미만 → 3단계로 돌아감
     - 3회 도달 → 잔여 이슈를 사람에게 보고, 루프 종료
6. Critical 이슈 발견 시 → 즉시 루프 중단, 사람에게 에스컬레이션
```

### 심각도 정의

| 심각도 | 설명 | 처리 |
|---|---|---|
| **Critical** | 산출물의 핵심 구조가 손상되거나, 요구사항 누락이 발견된 경우 | 즉시 루프 중단, 사람 에스컬레이션 |
| **Major** | 일관성 오류이나 산출물 구조는 유지되는 경우 | Auto-Fix 시도, 실패 시 사람 에스컬레이션 |
| **Minor** | 포맷 오류, 오타, 사소한 참조 불일치 | Auto-Fix 적용 |

**Verification**:
- `review-log.md`에 수렴 상태 또는 최대 반복 도달이 기록되어 있음
- 각 반복의 발견 이슈 수와 수정 이슈 수가 기록되어 있음
- 미해결 이슈 목록이 사람에게 제시되어 있음
- Critical 이슈 발견 시 즉시 중단된 기록이 있음 (해당 시)

---

## Rule MMR-04: Human Escalation Protocol

**Rule**: 사람에게 에스컬레이션할 때 구조화된 형식을 따릅니다. 사람이 빠르게 판단할 수 있도록 이슈 설명, AI 분석, 선택지를 함께 제시합니다.

### 에스컬레이션 형식

```markdown
## 🔴 판단 필요 사항 (N건)

### 1. [이슈 제목]
- **발견 위치**: [파일명:섹션]
- **발견 검토자**: [구조 검토자 / 논리 검토자 / 누락 검토자]
- **이슈 설명**: [구체적 설명]
- **AI 분석**: [왜 자동 수정이 불가능한지]
- **선택지**:
  - A) [옵션 A 설명]
  - B) [옵션 B 설명]
  - C) 직접 지시

### 2. [다음 이슈]
...

## ⚠️ 수정안 확인 필요 (M건) — Suggest-Fix 항목

### 1. [이슈 제목]
- **발견 위치**: [파일명:섹션]
- **현재 상태**: [문제 설명]
- **수정안**: [제안된 수정 내용]
- **수정 사유**: [왜 이 수정이 적절한지]
- **선택지**:
  - A) 수정안 적용
  - B) 다른 방식으로 수정 (직접 지시)
  - C) 현재 상태 유지

## ✅ 자동 수정 완료 (K건) — 참고용
[자동 수정된 항목 요약 목록]
```

### 에스컬레이션 원칙

- **최소 에스컬레이션**: 사람에게는 AI가 판단할 수 없는 항목만 제시
- **선택지 제공**: 가능한 한 선택지를 제시하여 사람의 판단 부담 최소화
- **맥락 제공**: 이슈의 배경과 AI 분석을 함께 제시하여 빠른 판단 지원
- **참고 정보**: 자동 수정된 항목도 참고용으로 함께 제시하여 투명성 확보

**Verification**:
- 사람에게 제시되는 항목이 Human-Required 및 Suggest-Fix 분류만 포함
- 각 항목에 선택지가 제공되어 있음
- 자동 수정 완료 항목이 참고용으로 함께 제시됨
- 에스컬레이션 형식이 위 템플릿을 따르고 있음

---

## Artifacts

Multi-model review produces the following artifacts:

```
aidlc-docs/construction/reviews/
├── review-log.md              # 전체 검토 이력 (검토자별 결과, 반복 횟수, 수렴 상태)
├── auto-fixes.md              # 자동 수정된 항목 목록 및 사유
└── human-decisions.md         # 사람의 판단이 필요한 항목 및 결정 기록
```

### review-log.md 필수 섹션

1. **검토 실행 요약**: 검토 대상 단계, 산출물 목록, 검토 일시
2. **검토자별 결과**: 각 검토자(구조/논리/누락)의 발견 이슈 목록
3. **이슈 통합**: 중복 제거 후 통합된 이슈 목록
4. **반복 이력**: 각 반복의 발견/수정/잔여 이슈 수 (Full 모드 시)
5. **수렴 상태**: 수렴 여부, 최종 반복 횟수

### auto-fixes.md 필수 섹션

1. **자동 수정 요약**: 총 수정 건수, 수정 유형별 분류
2. **수정 상세**: 각 수정의 위치, 수정 전/후, 수정 사유
3. **검증 결과**: 수정 후 재검토 결과

### human-decisions.md 필수 섹션

1. **판단 필요 항목**: MMR-04 형식에 따른 에스컬레이션 항목
2. **결정 기록**: 사람의 결정 내용 및 일시 (결정 후 업데이트)
3. **결정 반영 상태**: 결정이 산출물에 반영되었는지 여부

---

## Enforcement Integration

These rules are cross-cutting constraints that apply across all AI-DLC stages:

| Stage | Applicable Rules | Enforcement |
|---|---|---|
| Any stage completion | MMR-01, MMR-02 | Multi-perspective review must be executed with issue classification |
| Any stage completion (Full mode) | MMR-03 | Auto-fix loop must run until convergence or max iterations |
| Any stage completion | MMR-04 | Human escalation must follow the defined protocol |

At each stage completion:
- Evaluate all MMR rule verification criteria against the review artifacts produced
- Include an "MMR Compliance" section in the stage completion summary listing each rule as compliant, non-compliant, or N/A
- If any rule is non-compliant, this is a blocking MMR finding — follow the blocking finding behavior defined in the Overview
- Include MMR rule references in review documentation
