# 03 · 작업장 만들기

역할마다 독립된 worktree와 branch를 만든다. 경로는 두 가지다.

## A. Codex Desktop을 쓸 때

이때는 **손으로 `git worktree` 명령을 실행하지 않는다.** 하네스가 관리하는 작업장이라 손으로 만든 것은 추적되지 않는다.

1. "새 작업 트리(worktree)" 기능으로 작업장을 만든다 (경로는 대개 `~/.codex/worktrees/` 아래)
2. 작업 패널의 "브랜치 생성"으로 역할 branch를 만든다
3. 각 역할의 **시작 branch**를 아래 표대로 지정한다

| 역할 | 시작 branch | 역할 branch |
|---|---|---|
| Backend | `master` | `task/003-backend` |
| Frontend | `master` | `task/003-frontend` |
| Integration | `task/003-backend` | `task/003-integration` |
| Tester | `task/003-integration` | `task/003-tester` |

시작 branch가 이 표와 다르면 [05](05-integrate-verify.md)의 통합이 성립하지 않는다.

## B. git CLI로 직접 만들 때

Codex Desktop 없이 같은 구조를 만드는 경로다. 구현 역할 둘만 먼저 만든다 — 통합·검증 작업장은 앞 단계가 끝나야 만들 수 있다.

```bash
cd <repo>
git switch master

# 구현 역할 둘 — master에서 각각 출발, 서로 기다리지 않음
git worktree add ../wt-backend  -b task/003-backend  master
git worktree add ../wt-frontend -b task/003-frontend master
```

## 확인

```bash
git worktree list      # 작업장 경로와 branch가 짝이 맞는지
git branch -vv         # 역할 branch가 모두 master에서 갈라졌는지
```

## 흔한 실패

- **작업장 하나에서 branch를 바꿔 가며 쓴다** — 그러면 격리가 사라진다. 역할마다 폴더가 따로 있어야 하는 이유가 그것이다
- **시작 branch를 잘못 잡는다** — Integration을 `master`에서 만들면 Backend 구현이 빠진 채로 통합하게 된다
- **worktree를 저장소 안에 만든다** — 저장소 자신의 변경으로 잡힌다
