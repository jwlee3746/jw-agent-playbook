# Integration — 예제

**가상 작업**: [Backend](backend.example.md)와 [Frontend](frontend.example.md) 예제에서 이어진다. 두 branch가 각각 commit을 마친 상태다.

## 프롬프트

```markdown
# task_014 · Integration 작업 명세

## 작업장
branch `task/014-integration`, 시작 branch `task/014-backend`.
worktree를 손으로 만들지 않는다. 하네스의 작업 트리 기능으로 만든다.

## 작업 지시
`task/014-frontend` 를 병합하고 전체 검증을 통과시킨다.

## 하는 일
1. `git merge --no-ff task/014-frontend -m "task_014: merge frontend into integration"`
2. 전체 검증 실행
   - `python3 scripts/check_api.py`
   - `python3 scripts/check_dom.py`
   - `python3 scripts/audit.py`
3. 세 결과와 exit code를 report에 기록

## 하지 않는 일
- **`server.py`, `index.html`, `src/` 를 수정하지 않는다.**
  검증이 실패하면 해당 역할에게 되돌려 보낸다
- 병합 충돌을 **임의로 해소하지 않는다**

## 권한
- allowedFiles: `docs/task_014/integration/report.md`
- forbiddenFiles: `server.py`, `index.html`, `src/app.js`, `src/styles.css`,
  `data/items.json`, `scripts/`, `tasks/acceptance.json`

## 중단 조건
- 병합 충돌이 발생하면 즉시 중단하고 충돌 파일 목록과 함께 보고한다
- 검증이 실패하면 원인 분류와 관련 파일을 보고하고 중단한다
- `audit.py` 가 권한 위반을 보고하면 통과시키지 않는다

## 마무리
1. 세 검증의 결과와 exit code를 기록
2. `docs/task_014/integration/report.md` 작성 — 병합 대상, 검증 결과, 남은 위험
3. report 만 `git add` → `git commit`
4. push 금지
```

## 기대하는 report — 통과

```markdown
# task_014 · Integration report

## 병합
- 대상: task/014-frontend → task/014-integration
- 방식: --no-ff
- 충돌: 없음

## 검증
| 검증 | 명령 | exit code | 결과 |
|---|---|---|---|
| 기능 | python3 scripts/check_api.py | 0 | 케이스 8/8 PASS |
| 화면 | python3 scripts/check_dom.py | 0 | 필수 요소 9/9, 필터 7/7 |
| 권한 | python3 scripts/audit.py | 0 | 위반 0건 |

## 남은 위험
- CRITICAL 카드 강조 클래스는 검사되지만 시각적 대비는 검증 범위 밖이다
```

## 기대하는 보고 — 권한 위반

```markdown
## 검증
| 권한 | python3 scripts/audit.py | 1 | 위반 1건 |

위반 내역
- 역할: frontend
- 파일: data/items.json
- 사유: forbiddenFiles 에 포함된 파일이 수정됨

조치: 통과시키지 않는다. frontend 역할에게 되돌려 보낸다.
Integration 에서 해당 변경을 되돌리지 않는다 — 되돌리면 위반 사실이 히스토리에서 흐려진다.
```

## 이 예제에서 눈여겨볼 것

- **권한 위반을 Integration이 되돌리지 않는다.** 되돌리면 그 자리에서 문제는 사라지지만, 어느 역할이 무엇을 침범했는지가 히스토리에서 희미해진다. 위반한 역할이 자기 branch에서 되돌려야 기록이 남는다.
- **`남은 위험` 칸이 있다.** 검증이 전부 통과해도 "검증 범위 밖"인 것이 있다. 이걸 적어두지 않으면 PASS가 "완벽함"으로 읽힌다.
- `allowedFiles`가 report 하나뿐이다. 이 역할이 코드에 손댈 수 없다는 것이 권한 목록만 봐도 드러난다.
- 병합 명령의 `-m` 메시지에 task id가 들어 있다. 나중에 `git log --graph`에서 어느 작업의 병합인지 바로 보인다.
