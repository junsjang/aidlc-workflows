# Integration Test Ordering — Opt-In

**Extension**: Integration Test Ordering

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: Integration Test Ordering Extension
멀티 Unit 프로젝트에서 연동 테스트 순서화 규칙을 적용하시겠습니까?

A) Yes — Unit 간 의존성 그래프 기반으로 연동 테스트 순서를 강제 (멀티 Unit 프로젝트 권장)
B) No — 연동 테스트 순서를 강제하지 않음 (단일 Unit 또는 단순 프로젝트에 적합)
X) Other (please describe after [Answer]: tag below)

[Answer]:
```
