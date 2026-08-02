# 04 · 구현 병렬 실행

Backend와 Frontend를 각자의 작업장에서 동시에 돌린다. 서로 기다리지 않는다.

## 세션 띄우기

작업장마다 터미널을 따로 연다. 한 터미널에서 `cd`로 옮겨 다니면 세션이 섞인다.

```bash
# 터미널 1
cd ../wt-backend && codex        # 또는 claude

# 터미널 2
cd ../wt-frontend && codex
```

각 세션에 [02](02-work-order.md)에서 만든 지시서를 준다. 자기 역할의 허용/금지 파일이 무엇인지 명시한다.

## 각 역할이 하는 일

1. 자기 범위의 파일만 수정한다
2. 검증 스크립트를 돌려 통과시킨다
3. 자기 report를 쓴다
4. **자기 branch에만 commit 한다**

```bash
git add <허용된 파일만>
git commit -m "task_003(backend): ..."
```

## 규칙으로 못박을 것

- **`push` 하지 않는다.** 원격에 올리는 것은 사람이 최종 승인한 뒤의 일이다
- **다른 역할의 branch를 merge 하지 않는다.** 통합은 [05](05-integrate-verify.md)의 Integration 역할이 한다
- **`git add -A`를 쓰지 않는다.** 허용된 파일만 명시해서 스테이징한다. 전체 추가는 금지 파일을 딸려 보내는 가장 흔한 경로다

## 확인

각 작업장에서:

```bash
git status                          # 의도한 파일만 변경됐는지
git diff --stat master..HEAD        # 무엇이 얼마나 바뀌었는지
python3 scripts/check_api.py; echo "exit=$?"
```

report에는 실행한 명령과 exit code를 적는다. "테스트 통과했습니다"라는 문장이 아니라 로그와 exit code가 신뢰의 근거다.

## 흔한 실패

- **범위를 넘어 "개선"한다** — 인접 코드나 포맷을 고친다. 감사 스크립트가 [05](05-integrate-verify.md)에서 잡는다
- **검증을 통과시키려고 기준을 고친다** — 지시서와 검증 스크립트는 금지 파일에 들어 있어야 한다
- **중단 조건 없이 루프를 돈다** — 지시서 7번이 비어 있으면 여기서 비용이 샌다
