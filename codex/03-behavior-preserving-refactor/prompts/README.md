# 단계별 프롬프트

각 단계에 넘기는 지시문이다. 단계마다 **템플릿**과 **예제** 두 파일이 있다.

- 템플릿(`NN-*.md`) — `<...>` 자리를 채워 쓰는 골격
- 예제(`NN-*.example.md`) — 같은 골격을 실제로 채운 것

예제는 전부 같은 가상 프로젝트를 쓴다: **TypeScript 주문 처리 서비스. `OrderService`가 620줄이고 결제·재고·알림 책임이 한 클래스에 섞여 있다.** 단계를 순서대로 읽으면 진단에서 동등성 검증까지 가는 흐름이 보인다.

| 단계 | 템플릿 | 예제 | 국면 |
|---|---|---|---|
| 1 범위와 진단 | [01-scope-and-safety-net.md](01-scope-and-safety-net.md) | [예제](01-scope-and-safety-net.example.md) | A 안전망 |
| 2 특성화 테스트 | [02-characterization-tests.md](02-characterization-tests.md) | [예제](02-characterization-tests.example.md) | A 안전망 |
| 3 분류와 계획 | [03-change-classification.md](03-change-classification.md) | [예제](03-change-classification.example.md) | B 변경 |
| 4 기계적 일괄 적용 | [04-mechanical-sweep.md](04-mechanical-sweep.md) | [예제](04-mechanical-sweep.example.md) | B 변경 |
| 5 판단 루프 | [05-judgment-loop.md](05-judgment-loop.md) | [예제](05-judgment-loop.example.md) | B 변경 |
| 6 동등성 검증 | [06-equivalence-check.md](06-equivalence-check.md) | [예제](06-equivalence-check.example.md) | C 확인 |

## 공통 골격

[02](../../02-aorr-worker-verifier-loop/prompts/README.md)와 같은 여섯 칸을 쓴다. 이 설계에서 달라지는 것은 **기준선 칸**이 하나 더 붙는다는 점이다.

```
[<단계 번호> - <단계 이름>]

<읽을 것: 계획 문서, 앞 단계 산출물, 현재 코드, Git 상태>

기준선:
- 안전망 커밋: <SHA>
- 테스트 경로: <경로> — 이 단계에서 수정 금지
- 전체 테스트 명령: <명령>

역할:
- Worker(<하네스>): <담당>
- Verifier(<하네스>): <담당>

실행 순서:
1. ...

규칙:
- <금지 사항>

최종 보고:
- <무엇을 보고할지>

<이 단계에서 하지 말 것>
```

**기준선 칸이 이 설계의 장치다.** 어느 커밋이 되돌아갈 지점인지, 어느 경로가 동결됐는지를 매 단계 프롬프트에 다시 적는다. 한 번만 말하면 컨텍스트가 길어졌을 때 사라진다.

## 반복되는 문구와 그 이유

| 문구 | 막는 것 |
|---|---|
| "테스트 파일을 읽기만 하고 수정하지 마" | 기준선을 움직여서 통과시키는 것 |
| "동작을 바꾸지 마. 버그를 발견해도 고치지 말고 보고만 해" | 리팩토링과 기능 변경이 섞이는 것 |
| "커버리지는 측정한 출력만 쓰고 코드를 읽고 추정하지 마" | 안전망이 있다고 착각하는 것 |
| "계획표에 없는 항목은 착수하지 마" | 범위가 조용히 넓어지는 것 |
| "한 번에 항목 하나, 커밋 하나" | 무엇이 깨뜨렸는지 귀속되지 않는 것 |
| "재시도 2회를 넘기면 고치지 말고 되돌려" | 억지로 성사시키는 것 |
| "이름 변경·이동은 직접 편집하지 말고 LSP/AST 도구로" | 참조를 놓치는 것 |
| "확인되지 않은 내용은 `[사람 확인 필요]`" | 지어낸 사실이 이후 단계에 퍼지는 것 |
