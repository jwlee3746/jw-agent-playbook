# 04 · 평가 기반 개선 루프 (Claude Code)

**한 줄**: [codex/04](../../codex/04-eval-driven-improvement-loop/)와 같은 설계를 Claude Code에서 돌린다. 프롬프트 본문은 그쪽에 있고, 여기에는 하네스 차이만 적는다.
**출처**: 자작
**상태**: 미검증
**포팅**: [codex/04](../../codex/04-eval-driven-improvement-loop/) ← **본체**

## 이 파일이 짧은 이유

프롬프트를 양쪽에 복사하면 한쪽은 반드시 낡는다. **단계별 프롬프트는 [codex/04/prompts/](../../codex/04-eval-driven-improvement-loop/prompts/)에만 둔다.** 여기서는 Claude Code에서 달라지는 것만 다룬다.

절차·트랙 구분·게이트 조건은 [codex/04의 README](../../codex/04-eval-driven-improvement-loop/README.md)를 먼저 읽는다.

## 무엇이 달라지나

| 설계상 요구 | codex | Claude Code |
|---|---|---|
| 골든셋·rubric 동결 | 프롬프트 금지 | **권한으로 차단** — 아래 설정 |
| holdout 접근 통제 | 프롬프트 금지 | **권한으로 차단.** 개봉은 사람이 연다 |
| 1단계에서 코드 수정 금지 | 프롬프트 금지 | **plan mode** |
| Worker / Gatekeeper 분리 | 다른 하네스 | 서브에이전트. 같은 모델이라는 한계가 남는다 |
| 회차 간 컨텍스트 | 세션 재시작 | **회차마다 새 세션.** 아래 참조 |

## 동결

골든셋과 holdout을 함께 막는다.

```json
{
  "permissions": {
    "deny": [
      "Edit(./eval/golden/**)",
      "Write(./eval/golden/**)",
      "Edit(./eval/judge_rubric.md)",
      "Write(./eval/judge_rubric.md)",
      "Edit(./eval/scoring.py)",
      "Write(./eval/scoring.py)",
      "Read(./eval/splits/holdout.txt)"
    ]
  }
}
```

프로젝트 `.claude/settings.json`에 두고 경로는 실제 구조로 바꾼다.

마지막 줄이 이 설계에서 특히 중요하다. **holdout은 쓰기가 아니라 읽기를 막는다.** 골든셋은 고치는 게 문제지만 holdout은 보는 것 자체가 문제다 — 한 번 보면 그 정보가 이후 회차의 선택에 남는다.

### 여는 시점

| 무엇 | 언제 | 누가 |
|---|---|---|
| 골든셋 쓰기 | D 트랙 승인 회차 | 사람이 열고 닫는다 |
| holdout 읽기 | 5단계 게이트 | **평가 명령만 실행.** 케이스 파일은 열지 않는다 |

5단계에서 holdout 점수가 필요하지만 케이스 내용은 필요 없다. 평가 스크립트가 목록 파일을 읽고 총점만 뱉게 하면 `Read` 차단을 유지한 채로 게이트를 돌릴 수 있다. **이 구조를 만들어두는 것이 권한 설정보다 확실하다.**

## plan mode

1단계(에러 분석 정리)와 3단계(실패 라우팅)는 아무것도 수정하지 않는다.

```bash
claude --permission-mode plan
```

4단계는 실제로 파일을 고치므로 기본 모드로 돌아온다.

## 회차마다 새 세션

[02의 결과](../../codex/02-aorr-worker-verifier-loop/README.md#결과)에 기록된 문제가 이 설계에서 더 크게 나타난다. 컨텍스트에 이전 회차의 실패 이력이 쌓이면 에이전트는 **이미 시도한 방향을 계속 변주**한다. 루프의 목적이 매 회차 새 각도에서 원인을 다시 보는 것이라 이게 직접 손해다.

- 회차마다 세션을 새로 연다
- 상태는 컨텍스트가 아니라 [`LOOP-LOG`](../../codex/04-eval-driven-improvement-loop/LOOP-LOG-template.md)에서 읽는다
- `--continue`는 한 회차 안에서만 쓴다

## Worker / Gatekeeper

서브에이전트로 나누면 **같은 모델이 자기 변경을 판정**하는 구성이 된다.

이 설계에서는 손실이 03보다 크다. 5단계 게이트 조건 다섯 중 셋(동결·범위·점수)은 기계가 세지만, `대상 외 유형이 나빠졌는데 예상 목록에 있는가`와 `이 변경이 다음 회차를 방해하지 않는가`는 판단이다. 그리고 그 판단이 채택 여부를 가른다.

가능하면 Gatekeeper를 다른 하네스로 두고, 안 되면 [`LOOP-LOG`의 `Execution` 섹션](../../codex/04-eval-driven-improvement-loop/LOOP-LOG-template.md)에 `Fallback 구간`으로 회차 번호를 남긴다. **그 구간의 게이트 판정은 신뢰도가 낮다는 것이 나중에 보여야 한다.**

## 정지 상태를 세션으로 다루기

이 설계에는 사람이 풀어야 진행되는 상태가 다섯 개 있다 ([본체 참조](../../codex/04-eval-driven-improvement-loop/README.md#전체-절차)). Claude Code에서는 이걸 **세션 경계로 만드는 것**이 가장 확실하다.

| 상태 | 어떻게 |
|---|---|
| `ROUTING_DECISION_REQUIRED` | 1단계는 plan mode. 승인 자체가 판단 지점이 된다 |
| `TRACK_DECISION_REQUIRED` | 3단계 세션을 끝내고, 사람이 트랙을 정한 뒤 4단계를 새 세션으로 |
| `DATA_APPROVAL_REQUIRED` | 골든셋 `deny`를 사람이 직접 풀어야 진행된다. 권한이 곧 게이트 |
| `ROUND_APPROVAL_REQUIRED` | 5단계 세션을 끝낸다. 승인 뒤 커밋은 사람이 하거나 새 세션에서 |
| `AUDIT_DECISION_REQUIRED` | 6단계 세션을 끝낸다 |

**같은 세션에서 "승인해주세요"라고 묻고 답을 기다리지 않는다.** 에이전트가 스스로 승인으로 읽고 진행하는 경로가 열리고, 그러면 정지 상태가 이름만 남는다. 세션을 끝내면 그 경로 자체가 없다.

`DATA_APPROVAL_REQUIRED`가 특히 깔끔하다 — 권한이 잠겨 있으면 에이전트가 승인을 자의로 해석해도 파일을 못 고친다. **프롬프트 금지보다 권한이 확실하다는 원칙이 여기서 가장 잘 맞는다.**

## 결과

아직 돌려보지 않았다.

- **잘 된 점**:
- **실패한 점**:
- **다음에 바꾼다면**:
