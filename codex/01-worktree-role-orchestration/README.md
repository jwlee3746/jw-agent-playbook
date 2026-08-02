# 01 · worktree 역할 분리 오케스트레이션

**한 줄**: 구현 역할 둘을 병렬로 돌리고, 통합·검증 역할이 그 위에 쌓이며, 승인은 사람만 하는 5역할 파이프라인.

## 용도

한 작업을 여러 에이전트에 쪼개되, **자기가 만든 것을 자기가 통과시키는 경로를 막고 싶을 때** 쓴다.

쓰면 안 되는 경우: 작업이 진짜로 병렬이 아닐 때. 아래 구조는 역할 사이 파일 충돌이 실재할 때 값을 하고, 그렇지 않으면 branch 관리 비용만 남는다. 문서 저장소처럼 충돌이 없는 곳에는 과하다.

## 전제

- 하네스: Codex Desktop 또는 git worktree를 쓸 수 있는 CLI 에이전트
- **검증 스크립트가 있어야 한다.** 이 설계의 판정은 전부 스크립트 통과 여부에 걸려 있다. 통과/실패를 기계가 말해주지 못하면 나머지 구조가 무의미해진다
- 외부 의존성을 금지하고 표준 라이브러리만 쓴다. 검증이 어느 환경에서나 같은 결과를 내게 하려는 것이다

---

# 설계

## 역할과 branch 그래프

```
master ─┬─> task/003-backend ──┐
        │                      ├─> task/003-integration ─> task/003-tester
        └─> task/003-frontend ─┘                                  │
                                                                  v
                                              사람이 local master에서 최종 merge
```

| 역할 | 작업 범위 | 권한 |
|---|---|---|
| **Backend** | 서버 로직 | 지정된 구현 파일 + 자기 report |
| **Frontend** | UI | 지정된 구현 파일 + 자기 report |
| **Integration** | 두 branch 통합, 전체 검증 | **구현 파일 수정 금지.** merge와 자기 report만 |
| **Tester** | 통합본 검증 | **코드 수정 금지.** report와 approval draft만 |
| **Main(오케스트레이터)** | 분해·조율·증거 수집 | **구현도 commit도 하지 않는다.** 사람에게 판단을 제안 |

Backend와 Frontend는 master에서 각각 출발해 서로를 기다리지 않는다. 반면 Integration은 양쪽이 끝나야 시작된다 — 순수 fan-out이 아니라 **의존이 있는 fan-out**이다.

worktree는 "같은 프로젝트의 분리된 작업 책상"이다. 에이전트가 실패해도 기준 책상(master)은 더러워지지 않고, 망가진 책상만 지우고 다시 만들면 된다.

## 권한을 프롬프트가 아니라 데이터로 선언

`tasks/acceptance.json` 하나가 채점 기준과 역할 권한을 모두 소유하고, **어떤 역할도 이 파일을 수정할 수 없다.**

| 항목 | 내용 |
|---|---|
| 판정 규칙 | 분류 규칙, 필수 필드, 허용값 목록 |
| 케이스 | 입력과 기대 출력 쌍. API 검증이 이걸로 돈다 |
| UI 계약 | 필수 DOM id, 필수 필터 목록 |
| `allowedFilesByRole` | 역할별로 **수정 가능한** 파일 목록 |
| `forbiddenFilesByRole` | 역할별로 **금지된** 파일 목록 |
| 위험도 정책 | 검증 전부 통과면 LOW, 하나라도 실패면 HIGH |

핵심은 목록 자체가 아니라 **형식**이다. "너는 server.py만 고쳐라"를 프롬프트로 지시하면 지켜졌는지 사람이 읽어서 판단해야 하지만, 데이터로 선언하면 감사 스크립트가 diff와 대조해 기계로 판정한다. 규칙 위반이 의견이 아니라 사실이 된다.

## 작업 지시서의 7요소

작업 지시서에 들어가야 하는 것들이다. 위 `acceptance.json`은 이 중 범위·허용/금지·검증을 데이터로 옮긴 형태다.

1. **목표(Goal)** — 무엇을 달성하는가
2. **범위(Scope)** — 어느 파일·모듈까지인가
3. **수정 허용(Allow)** — 건드려도 되는 것
4. **수정 금지(Deny)** — 건드리면 안 되는 것
5. **검증(Validate)** — 실행할 명령
6. **산출물(Artifacts)** — 무엇을 남기는가
7. **중단 조건(Fallback Barrier)** — 언제 멈추고 사람을 부르는가

7번이 실무에서 가장 자주 빠진다. 중단 조건이 없으면 에이전트가 컨텍스트와 비용이 소진될 때까지 무한 디버깅 루프를 돈다.

## 증거 규약

- 구현 전달은 **branch와 commit**이다. 별도 patch 파일을 만들지 않는다 — diff가 이미 변경 증거다
- 판단 전달은 **역할별 report 파일**이다. `docs/<task>/<role>/report.md`로 경로가 고정돼 있어 어느 역할의 주장인지 섞이지 않는다
- Tester만 `approval_draft.json`을 쓴다

승인 전에 확인할 증거는 여섯 가지다.

| 질문 | 증거 |
|---|---|
| 어떤 명령을 실행했는가 | 실행 로그 |
| exit code는 무엇인가 | 0 여부 |
| 어떤 파일이 바뀌었는가 | `git diff --stat` |
| 요구사항과 diff가 맞는가 | 사람이 읽는다 |
| 실패 원인을 기록했는가 | report 파일 |
| 사람이 승인했는가 | approval 파일 |

요지는 **"테스트 통과했습니다"라는 문장이 아니라 로그와 exit code가 신뢰의 근거**라는 것이다.

## 승인 게이트

- approval draft의 `status`는 **항상 `pending`**이다. 에이전트가 승인 상태를 쓰는 것 자체가 금지다
- 최종 merge는 사람이 local master에서만 한다. 에이전트는 `push`하지 않는다
- **검증 스크립트나 채점 기준을 고쳐서 통과시키는 것을 명시적으로 금지**한다. 이게 없으면 "통과"가 무의미해진다

---

# 재현 절차

## A. Codex Desktop을 쓸 때

이때는 **손으로 `git worktree` 명령을 실행하지 않는다.**

1. 작업 트리는 Codex Desktop의 "새 작업 트리(worktree)" 기능으로 만든다. 하네스가 관리하는 작업장이라 손으로 만든 것은 추적되지 않는다 (경로는 대개 `~/.codex/worktrees/` 아래)
2. 역할 branch는 작업 패널의 "브랜치 생성"으로 만든다
3. 각 역할의 시작 branch를 아래 표대로 지정한다

| 역할 | 시작 branch | 역할 branch |
|---|---|---|
| Backend | `master` | `task/003-backend` |
| Frontend | `master` | `task/003-frontend` |
| Integration | `task/003-backend` | `task/003-integration` |
| Tester | `task/003-integration` | `task/003-tester` |

## B. git CLI로 직접 재현할 때

Codex Desktop 없이 같은 구조를 만드는 경로다. 아래 `<repo>`와 task 번호, 스크립트 이름은 프로젝트에 맞게 바꾼다.

```bash
# 0) 기준 책상
cd <repo>
git switch master

# 1) 구현 역할 둘 — master에서 각각 출발, 서로 기다리지 않음
git worktree add ../wt-backend  -b task/003-backend  master
git worktree add ../wt-frontend -b task/003-frontend master

# 2) 각 작업장에서 에이전트 세션을 따로 띄운다 (터미널 분리)
cd ../wt-backend  && codex      # 또는 claude
cd ../wt-frontend && codex

# 3) 각 역할은 자기 branch에만 commit 한다. push 하지 않는다
git add <허용된 파일만>
git commit -m "task_003(backend): ..."

# 4) 통합 역할 — backend 위에서 출발해 frontend를 병합
cd <repo>
git worktree add ../wt-integration -b task/003-integration task/003-backend
cd ../wt-integration
git merge --no-ff task/003-frontend -m "task_003: merge frontend into integration"

# 5) 통합본에서 전체 검증 — 실패하면 여기서 멈춘다
python3 scripts/check_api.py
python3 scripts/check_dom.py
python3 scripts/audit.py          # 역할별 파일 권한 위반 검사

# 6) 검증 역할 — 코드는 건드리지 않고 증거만 생산
cd <repo>
git worktree add ../wt-tester -b task/003-tester task/003-integration
# docs/task_003/test/report.md, approval_draft.json 작성 (status: pending)

# 7) 사람이 기준 책상에서 최종 통합
cd <repo>
git switch master
git diff --stat master..task/003-tester     # 무엇이 바뀌는지 먼저 본다
git log --graph --oneline master..task/003-tester
git merge --no-ff task/003-tester -m "task_003: merge verified work"

# 8) 정리
git worktree remove ../wt-backend
git worktree remove ../wt-frontend
git worktree remove ../wt-integration
git worktree remove ../wt-tester
git worktree list                            # 남은 작업장 확인
```

`--no-ff`를 쓰는 이유는 병합 지점을 커밋으로 남겨 어느 역할의 작업이 언제 들어왔는지 그래프에서 보이게 하려는 것이다. fast-forward로 합치면 이 경계가 사라진다.

7번의 `git diff --stat`과 `git log --graph`가 사람이 승인 전에 보는 것의 전부다. "완료했습니다"라는 보고가 아니라 이 두 출력과 report 파일을 읽고 판단한다.

---

## 이 설계의 요점

1. **판정자와 구현자를 분리한다.** Tester는 코드 수정 권한이 없고 Main은 구현하지 않는다. 자기 작업을 자기가 승인하는 경로가 구조적으로 없다.
2. **제약을 검증 가능한 형태로 둔다.** 권한도, 채점도 데이터 파일에 있고 스크립트가 판정한다.
3. **자동 승인을 금지한다.** 에이전트는 `pending`까지만 만들고 사람이 게이트를 넘긴다.

1번과 3번은 이 레포의 다른 항목과도 통한다. `상태: 미검증`을 스스로 `검증됨`으로 바꾸지 않는 것과 같은 원리다.

## 결과

실제로 돌려봤으나 관찰 내용이 아직 기록되지 않았다. 아래를 채우면 `검증됨`으로 올린다.

- **잘 된 점**:
- **실패한 점**: 어느 역할에서 막혔는지, 권한 선언이 실제로 지켜졌는지(감사 스크립트가 위반을 잡았는지)
- **사람 승인 게이트**: 실제로 걸렸는지, 아니면 형식적으로만 존재했는지
- **병렬의 실익**: 구현 둘의 동시 진행이 실제로 시간을 줄였는지, 아니면 통합 대기에 흡수됐는지
- **재현 절차 B**: CLI로 직접 돌려본 적은 없다. Codex Desktop 없이도 같은 결과가 나오는지 확인 필요
- **다음에 바꾼다면**:
