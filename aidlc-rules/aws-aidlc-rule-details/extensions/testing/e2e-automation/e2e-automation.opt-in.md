# E2E Test Automation — Opt-In

**Extension**: E2E Test Automation

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: E2E Test Automation Extension
E2E 테스트 자동화 규칙을 적용하시겠습니까?

A) Yes — 모든 E2E 규칙(E2E-01~04) blocking 적용: User Story AC 기반 시나리오 도출 + 환경별 전략 + 연동 기준선 + 산출물 강제 (멀티 환경 배포, 체계적 E2E 검증이 필요한 프로젝트에 권장)
B) Partial — E2E-01, E2E-03, E2E-04만 blocking 적용, E2E-02(환경별 전략)는 N/A 처리: 시나리오 도출과 산출물은 강제하되 환경별 전략은 선택적 (단일 환경 또는 환경 컨텍스트 미수집 프로젝트에 적합)
C) No — 모든 E2E 규칙 건너뛰기 (단순 프로젝트, PoC, 또는 별도 E2E 프레임워크를 이미 운용 중인 프로젝트에 적합)
X) Other (please describe after [Answer]: tag below)

[Answer]:
```

## aidlc-state.md Recording

When the user selects an option, record the extension status in `aidlc-docs/aidlc-state.md` under `## Extension Configuration`:

| Selection | Enabled | Mode | Notes |
|---|---|---|---|
| A) Yes | `true` | `full` | All E2E rules (E2E-01~04) enforced as blocking |
| B) Partial | `true` | `partial` | E2E-01, E2E-03, E2E-04 blocking; E2E-02 N/A |
| C) No | `false` | — | All E2E rules skipped |

Example entry in aidlc-state.md:
```markdown
### E2E Test Automation
- **Enabled**: true
- **Mode**: partial
- **Blocking Rules**: E2E-01, E2E-03, E2E-04
- **N/A Rules**: E2E-02
```
