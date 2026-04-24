# Multi-Model Review Loop — Opt-In

**Extension**: Multi-Model Automated Review Loop

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: Multi-Model Automated Review
산출물의 누락/일관성 검토를 다양한 AI 모델(또는 서로 다른 검토 관점)로 자동 반복 수행하시겠습니까?

A) Full — 모든 MMR 규칙(MMR-01~04) blocking 적용: 다관점 자동 검토 + 자동 수정 + 수렴 확인 (사람은 판단 사항만 개입, 품질 중시 프로젝트에 권장)
B) Review Only — MMR-01, MMR-02, MMR-04만 blocking 적용, MMR-03(자동 수정 루프)는 N/A 처리: 다관점 검토와 이슈 분류는 수행하되 자동 수정은 사람 승인 후 적용 (안전성 중시 프로젝트에 적합)
C) No — 모든 MMR 규칙 건너뛰기 (단순 프로젝트, PoC, 또는 단일 모델 검토로 충분한 프로젝트에 적합)
X) Other (please describe after [Answer]: tag below)

[Answer]:
```

## aidlc-state.md Recording

When the user selects an option, record the extension status in `aidlc-docs/aidlc-state.md` under `## Extension Configuration`:

| Selection | Enabled | Mode | Notes |
|---|---|---|---|
| A) Full | `true` | `full` | All MMR rules (MMR-01~04) enforced as blocking |
| B) Review Only | `true` | `review-only` | MMR-01, MMR-02, MMR-04 blocking; MMR-03 N/A |
| C) No | `false` | — | All MMR rules skipped |

Example entry in aidlc-state.md:
```markdown
### Multi-Model Review
- **Enabled**: true
- **Mode**: full
- **Blocking Rules**: MMR-01, MMR-02, MMR-03, MMR-04
- **N/A Rules**: —
```
