# 역할별 프롬프트

각 역할 세션에 넘기는 작업 명세다. 역할마다 **템플릿**과 **예제** 두 파일이 있다.

- 템플릿(`<역할>.md`) — `<...>` 자리를 채워 쓰는 골격
- 예제(`<역할>.example.md`) — 같은 골격을 실제로 채운 것

예제는 전부 같은 가상 작업을 쓴다: **재고 임계값 알림 — 판정 API와 대시보드.** 다섯 역할을 순서대로 읽으면 하나의 작업이 병렬 구현에서 승인까지 가는 흐름이 보인다.

| 역할 | 템플릿 | 예제 | 시작 branch |
|---|---|---|---|
| Backend | [backend.md](backend.md) | [예제](backend.example.md) | `master` |
| Frontend | [frontend.md](frontend.md) | [예제](frontend.example.md) | `master` |
| Integration | [integration.md](integration.md) | [예제](integration.example.md) | Backend branch |
| Tester | [tester.md](tester.md) | [예제](tester.example.md) | Integration branch |
| Main | [main.md](main.md) | [예제](main.example.md) | — |

## 공통 골격

다섯 역할이 모두 같은 일곱 칸을 쓴다.

```markdown
# <task-id> · <역할> 작업 명세

## 작업장
branch `<역할 branch>`, 시작 branch `<시작 branch>`.
worktree를 손으로 만들지 않는다(하네스가 관리하는 경우).

## 목표
`<검증 명령>` 을 PASS 시킨다.

## 구현할 것
<무엇을 만드는가 — 규칙을 서술>
정확한 기대값은 `<채점 파일>` 의 <해당 섹션> 을 기준으로 한다.

## 유지할 것
<건드리면 안 되는 기존 동작>
<환경 제약>

## 권한
- allowedFiles: <수정 가능한 파일 목록>
- forbiddenFiles: <금지 파일 목록>

## 중단 조건
<언제 멈추고 사람을 부르는가>

## 마무리
1. `<검증 명령>` 실행 → PASS 확인
2. `<report 경로>` 작성 — 무엇을 왜 바꿨는지, 실행한 명령과 exit code
3. allowedFiles 만 `git add` → `git commit`
4. merge / rebase / push 금지
```

## 두 가지 원칙

**판정 기준을 프롬프트에 복사하지 말고 참조한다.** 규칙을 서술하더라도 "정확한 기대값은 채점 파일을 따른다"를 반드시 넣는다. 프롬프트와 채점표가 어긋났을 때 채점표가 이겨야 한다. 복사해두면 둘 중 하나가 낡고, 에이전트는 자기에게 유리한 쪽을 고른다.

**역할마다 금지 항목이 다르다.** 위 골격은 공통이고, 각 템플릿이 자기 역할에 필요한 제약을 덧붙인다.

| 역할 | 추가로 넣는 것 |
|---|---|
| Backend / Frontend | 다른 역할의 구현 파일에 손대지 않는다 |
| Integration | 구현 파일 수정 금지. 충돌이 나면 임의로 해소하지 말고 멈춘다 |
| Tester | 구현 코드·검증 스크립트·채점 파일 모두 수정 금지. 통과시키려고 결과를 조작하지 않는다. 시작 전 working tree가 clean한지 확인한다. `status`는 항상 `pending` |
| Main | 구현도 commit도 하지 않는다. 증거를 모아 사람에게 제안만 한다 |

Tester 줄이 가장 길다. 검증자에게만 필요한 제약이 따로 있기 때문이고, 이게 빠지면 검증이 형식만 남는다.
