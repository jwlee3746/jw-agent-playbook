# 05 · 통합과 검증

두 구현 branch를 합치고, 통합본에서 전체 검증을 돌린 뒤, 사람이 읽을 증거를 만든다.

## Integration — 통합만 한다

Backend branch 위에서 출발해 Frontend를 병합한다. **구현 파일은 건드리지 않는다.**

```bash
cd <repo>
git worktree add ../wt-integration -b task/003-integration task/003-backend
cd ../wt-integration
git merge --no-ff task/003-frontend -m "task_003: merge frontend into integration"
```

`--no-ff`를 쓰는 이유는 병합 지점을 커밋으로 남겨 어느 역할의 작업이 언제 들어왔는지 그래프에서 보이게 하려는 것이다. fast-forward로 합치면 이 경계가 사라진다.

충돌이 나면 여기서 멈추고 사람을 부른다. 통합 역할이 충돌을 임의로 해소하면 어느 쪽 구현이 살아남았는지 아무도 모르게 된다.

## 전체 검증 — 실패하면 여기서 멈춘다

```bash
python3 scripts/check_api.py; echo "exit=$?"
python3 scripts/check_dom.py; echo "exit=$?"
python3 scripts/audit.py;     echo "exit=$?"    # 역할별 파일 권한 위반 검사
```

세 번째가 [02](02-work-order.md)에서 만든 권한 감사다. 각 역할이 허용된 파일만 건드렸는지 diff로 대조한다. 여기서 위반이 나오면 통과시키지 않는다.

하나라도 실패하면 위험도는 HIGH다. 구현 역할에게 되돌려 보내고, 통합 역할이 대신 고치지 않는다.

## Tester — 코드는 건드리지 않고 증거만 만든다

```bash
cd <repo>
git worktree add ../wt-tester -b task/003-tester task/003-integration
```

Tester가 만드는 것은 report와 approval draft 둘뿐이다.

- `docs/task_003/test/report.md` — 실행한 명령, exit code, 실패 원인
- `docs/task_003/test/approval_draft.json` — **`status`는 항상 `pending`**

에이전트가 승인 상태를 쓰는 것 자체가 금지다. 권고는 적을 수 있지만 판단은 사람이 한다.

## 사람이 승인 전에 확인할 증거

| 질문 | 증거 |
|---|---|
| 어떤 명령을 실행했는가 | 실행 로그 |
| exit code는 무엇인가 | 0 여부 |
| 어떤 파일이 바뀌었는가 | `git diff --stat` |
| 요구사항과 diff가 맞는가 | 사람이 읽는다 |
| 실패 원인을 기록했는가 | report 파일 |
| 사람이 승인했는가 | approval 파일 |

전달 통로는 둘로 고정한다. **구현은 branch와 commit으로, 판단은 report 파일로.** 별도 patch 파일은 만들지 않는다 — diff가 이미 변경 증거다. report 경로를 `docs/<task>/<role>/`로 고정하면 어느 역할의 주장인지 섞이지 않는다.

## 흔한 실패

- **통합 역할이 구현을 고친다** — 충돌이나 검증 실패를 그 자리에서 손보면 역할 분리가 무너진다
- **Tester가 코드를 고친다** — 검증자가 대상을 고치면 통과의 의미가 사라진다
- **approval draft에 승인을 쓴다** — `pending` 외의 값이 들어오면 게이트가 형식만 남는다
