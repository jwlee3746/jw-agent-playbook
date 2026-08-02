# Tester — 예제

**가상 작업**: [Integration 예제](integration.example.md)에서 이어진다. 통합본이 전체 검증을 통과해 commit된 상태다.

## 프롬프트

```markdown
# task_014 · Tester 작업 명세

## 작업장
branch `task/014-tester`, 시작 branch `task/014-integration`.
통합 결과(Backend + Frontend 병합본) 위에서 검증한다.

## 규칙
- **구현 코드를 수정하지 않는다** (`server.py`, `index.html`, `src/`, `data/`)
- **`scripts/` 와 `tasks/acceptance.json` 을 수정하지 않는다**
- **통과시키려고 결과를 조작하지 않는다**
- 증거는 `docs/task_014/test/` 아래에만 생성한다

## 시작 확인 (아직 아무것도 바꾸지 않는다)
- branch 생성 시점의 HEAD 가 `task/014-integration` 최신 commit 과 같은가
- working tree 가 clean 한가

## 실행
- `python3 scripts/check_api.py`
- `python3 scripts/check_dom.py`
- `python3 scripts/audit.py --out docs/task_014/test/audit.json`

## 증거 작성
- `docs/task_014/test/report.md`
  - 실제 PASS/FAIL, 위험도, 남은 리스크를 기록
- `docs/task_014/test/approval_draft.json`
  - `status` 는 항상 `"pending"`
  - 모두 통과하면 recommendation 은 `"approve_candidate"`
  - 하나라도 실패하면 recommendation 은 `"hold"`

## 권한
- allowedFiles: `docs/task_014/test/report.md`,
  `docs/task_014/test/approval_draft.json`,
  `docs/task_014/test/audit.json`
- forbiddenFiles: `server.py`, `index.html`, `src/app.js`, `src/styles.css`,
  `data/items.json`, `scripts/`, `tasks/acceptance.json`

## 마무리
1. 위 파일들만 `git add` → `git commit`
2. merge / rebase / push 금지
```

## 기대하는 approval draft

```json
{
  "task": "task_014",
  "verified_commit": "<통합본 commit sha>",
  "status": "pending",
  "recommendation": "approve_candidate",
  "checks": [
    { "name": "api",   "command": "python3 scripts/check_api.py", "exit": 0, "result": "8/8" },
    { "name": "dom",   "command": "python3 scripts/check_dom.py", "exit": 0, "result": "9/9, filters 7/7" },
    { "name": "audit", "command": "python3 scripts/audit.py",     "exit": 0, "result": "violations 0" }
  ],
  "residual_risks": [
    "CRITICAL 강조는 클래스 존재만 확인. 시각적 대비는 검증 범위 밖",
    "동시 요청 상황은 검증하지 않음"
  ]
}
```

`status`가 `pending`이고 `recommendation`이 `approve_candidate`다. **둘은 다른 칸이다** — 권고는 에이전트가, 승인은 사람이 쓴다.

## 기대하는 report — 실패한 경우

```markdown
# task_014 · Test report

## 시작 확인
- HEAD: <sha> — task/014-integration 최신과 일치
- working tree: clean

## 결과
| 검증 | exit code | 결과 |
|---|---|---|
| api | 0 | 8/8 PASS |
| dom | 1 | 필수 요소 8/9 — `alert-board` 없음 |
| audit | 0 | 위반 0건 |

## 실패 상세
- 항목: requiredDomIds 의 `alert-board`
- 재현: `python3 scripts/check_dom.py` → 해당 id 미검출
- 관련 파일: index.html
- 추정 원인 1개: 보드 컨테이너 id 가 `alerts-board` 로 작성됨 (하이픈 위치 차이)

## 위험도
HIGH — 검증 하나가 FAIL

## 조치
frontend 역할에게 되돌려 보낸다. Tester 는 `index.html` 을 수정하지 않는다.
```

## 이 예제에서 눈여겨볼 것

- **원인을 찾았는데도 고치지 않는다.** id 오타라는 것까지 특정했지만 한 글자도 고치지 않고 되돌려 보낸다. 여기서 고치면 검증자가 구현자가 되고, 자기가 고친 것을 자기가 통과시키게 된다.
- **`approval_draft.json`에 `verified_commit`이 있다.** 무엇을 검증한 결과인지 못박아두면, 나중에 코드가 바뀌었을 때 이 승인 초안이 유효한지 판단할 수 있다.
- **`residual_risks`가 채워져 있다.** 전부 PASS인 경우에도 검증 범위 밖이 무엇인지 남긴다. 사람이 승인할 때 읽어야 할 것은 통과 목록이 아니라 이쪽이다.
- **"시작 확인"이 report 맨 위에 있다.** 검증 대상이 무엇이었는지가 결과보다 먼저 온다.
