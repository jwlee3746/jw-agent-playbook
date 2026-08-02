# Main (오케스트레이터) — 예제

**가상 작업**: 재고 임계값 알림. [Backend](backend.example.md)·[Frontend](frontend.example.md)·[Integration](integration.example.md)·[Tester](tester.example.md) 예제를 조율하는 쪽이다.

## 프롬프트

```markdown
# task_014 · Main 작업 명세

## 역할
전체 파이프라인을 분해하고 조율한다. 코드는 직접 짜지 않는다.
증거를 모아 사람이 최종 merge 를 판단하도록 돕는다.

## 순서
1. 작업 분해
   - Backend: `server.py` 의 `classify_stock`, `POST /api/alerts`
   - Frontend: `index.html` / `src/app.js` / `src/styles.css` 의 알림 대시보드
   - Integration: 두 branch 통합 + 전체 검증
   - Tester: 통합본 위에서 검증 + 증거 문서
2. 역할별 작업장은 하네스의 작업 트리 기능으로 만든다.
   사람이 `git worktree add` 를 실행하지 않는다.
3. 각 역할의 allowedFiles / forbiddenFiles 를 `tasks/acceptance.json` 에서 확인한다
4. 병렬 실행
   - Backend, Frontend 는 둘 다 `master` 에서 독립적으로 시작해 동시에 구현한다
   - 각 역할은 자기 branch 에서 검증 통과 후 report 작성 + commit 까지 수행한다
5. 통합
   - Integration: 시작 `task/014-backend` → `task/014-integration` 생성
     → `task/014-frontend` 를 `--no-ff` merge
   - `check_api.py`, `check_dom.py`, `audit.py` 를 모두 실행한다
6. 검증
   - Tester: 시작 `task/014-integration` → `task/014-tester` 생성
   - 코드 수정 없이 `docs/task_014/test/` 에 report 와 approval draft 작성
7. 최종 리뷰
   - `master` 에서 branch diff / report / approval draft / 그래프만 읽고 판단을 제안한다
   - approval draft 의 status 는 항상 pending. recommendation 은 참고만
8. 사람 승인 전에는 절대 `master` 로 merge 하지 않는다

## 금지
- 코드·문서 직접 수정
- `git add` / `commit` / `merge` / `push`
- `git worktree add` 등 수동 worktree 조작
- `scripts/` 와 `tasks/acceptance.json` 수정
- 사람 승인 없는 최종 merge
```

## 기대하는 최종 제안

```markdown
# task_014 · 최종 리뷰 제안

## 읽은 것
- `git diff --stat master..task/014-tester`
- `git log --graph --oneline master..task/014-tester`
- 역할별 report 4개
- `docs/task_014/test/approval_draft.json`

## 변경 요약
| 역할 | 파일 | 추가/삭제 |
|---|---|---|
| backend | server.py | +84 / -6 |
| frontend | index.html, src/app.js, src/styles.css | +212 / -3 |
| 문서 | report 4개, approval draft, audit.json | +96 / -0 |

## 그래프
병합 지점 2개가 보인다 — frontend → integration, 그리고 tester 분기.
fast-forward 로 눌린 구간은 없다.

## 검증
| 검증 | exit code | 결과 |
|---|---|---|
| api | 0 | 8/8 |
| dom | 0 | 9/9, 필터 7/7 |
| audit | 0 | 위반 0건 |

## approval draft
- status: pending
- recommendation: approve_candidate

## 남은 위험 (Tester 보고)
- CRITICAL 강조는 클래스 존재만 확인. 시각적 대비는 검증 범위 밖
- 동시 요청 상황은 검증하지 않음

## 제안
`git merge --no-ff task/014-tester` 를 권합니다.
승인 여부는 사람이 판단해 주세요. 저는 merge 하지 않습니다.

확인을 권하는 지점:
- frontend 의 +212 줄 중 스타일이 큰 비중입니다. 요청 범위를 넘는 변경이 있는지
  `git diff master..task/014-tester -- src/styles.css` 로 한 번 보시길 권합니다.
```

## 이 예제에서 눈여겨볼 것

- **마지막 문장이 "저는 merge 하지 않습니다"다.** 제안과 실행을 문장으로 갈라놓았다. 권고까지가 이 역할의 끝이다.
- **"확인을 권하는 지점"이 있다.** 검증이 전부 통과했는데도 사람이 볼 곳을 짚었다. 스크립트가 통과시킨 것과 사람이 봐야 할 것은 다르다 — 감사 스크립트는 허용된 파일인지만 보지, 그 안의 변경이 요청 범위인지는 보지 않는다.
- **그래프를 읽은 결과가 적혀 있다.** 병합 지점이 보이는지 확인하는 것은 `--no-ff`가 지켜졌는지 검사하는 것과 같다.
- 읽은 것 목록에 `git diff`와 `git log`만 있고 저장소를 바꾸는 명령이 없다.
