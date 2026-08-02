# 06 · 승인과 정리

사람이 기준 책상에서 diff를 읽고 판단한 뒤 통합한다. 그다음 작업장을 회수한다.

## 승인 전에 읽는 것

```bash
cd <repo>
git switch master

git diff --stat master..task/003-tester          # 무엇이 얼마나 바뀌는지
git log --graph --oneline master..task/003-tester # 어느 역할이 언제 들어왔는지
```

이 두 출력과 report 파일이 판단 재료의 전부다. **"완료했습니다"라는 보고는 증거가 아니다.**

`--graph`에 각 역할의 병합 지점이 보이지 않으면 [05](05-integrate-verify.md)에서 `--no-ff`가 빠진 것이다.

## 최종 merge

Tester branch는 Integration 위에서 만들어졌고 Integration은 Backend·Frontend를 합친 것이므로, **Tester 하나만 merge하면 전부 들어온다.**

```bash
git merge --no-ff task/003-tester -m "task_003: merge verified work"
```

- 최종 merge는 **사람이 local master에서만** 한다
- 에이전트는 `push` 하지 않는다
- 검증이 실패한 상태로 merge하지 않는다. 되돌려 보내는 게 정상 경로다

## 작업장 회수

```bash
git worktree remove ../wt-backend
git worktree remove ../wt-frontend
git worktree remove ../wt-integration
git worktree remove ../wt-tester

git worktree list      # 남은 작업장 확인
```

커밋되지 않은 변경이 남아 있으면 `remove`가 거부된다. 버릴 변경이면 `--force`를 쓰되, 무엇을 버리는지 먼저 확인한다.

역할 branch는 바로 지우지 않는 편이 낫다. 나중에 "어느 역할이 무엇을 했는가"를 되짚을 때 필요하다.

## 실패했을 때

작업장 하나가 망가져도 기준 책상은 멀쩡하다. 그 작업장만 지우고 [03](03-worktrees.md)부터 다시 만든다.

```bash
git worktree remove --force ../wt-backend
git branch -D task/003-backend
```

되돌리는 비용이 낮다는 것이 이 구조를 쓰는 이유 중 하나다.

## 흔한 실패

- **diff를 안 보고 merge 한다** — 승인 게이트가 형식만 남는다
- **fast-forward로 합친다** — 역할 경계가 그래프에서 사라진다
- **작업장을 안 치운다** — 다음 실행에서 `git worktree list`가 지저분해지고, 오래된 작업장에서 실수로 작업하게 된다
