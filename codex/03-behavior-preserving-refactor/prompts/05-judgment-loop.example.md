# 5 · 판단 루프 — 예제

J3(`settle()`의 재시도 로직을 `RetryPolicy`로 분리)을 돈다. 위험도 상 항목이다.

## 프롬프트

```
[5 - 판단 루프 · 항목 J3]

기준선:
- 안전망 커밋: a3f81c2
- 직전 정상 커밋: 5f0c93d (J2까지 적용)   ← 되돌아갈 지점
- 테스트 경로: test/ — 동결. 읽기만 한다
- 전체 테스트 명령: npx vitest run
- 기준 결과: 61 passed (T1으로 3건 추가됨)

역할:
- Worker(Codex): 항목 하나 적용. 테스트 실행과 통과 판정은 하지 않음
- Verifier(Claude): 동일 테스트 실행, 결과 보고. 코드 수정하지 않음

이번 항목: J3 — settle()의 결제 재시도 로직을 RetryPolicy로 분리한다.
외부 호출 순서(charge → reserve → send)를 반드시 유지한다.
대상 파일: src/order/OrderService.ts, src/order/RetryPolicy.ts (신규)
예상되는 영향: test/characterization/settle.spec.ts의 타임아웃 재시도 3건

실행 순서:
1. Verifier: 변경 전 전체 테스트 실행. 61 passed인지 확인
2. Worker: 이 항목만 적용. 대상 파일 밖으로 나가지 않는다
3. Verifier: 동일 테스트 재실행
4. 판정:
   - 61 passed와 동일 → 커밋하고 종료
   - 다름 → 깨진 테스트가 "예상되는 영향"에 있는지 확인
       있으면: 원인 하나에 최소 수정 (최대 2회)
       없으면: 변경이 계획 범위를 넘었다는 신호. 되돌린다
   - 재시도 2회를 넘기면 고치지 말고 되돌린다:
       git reset --hard 5f0c93d

금지:
- test/ 아래 파일 수정, 기대값 완화, skip·xfail 추가, 타임아웃 늘리기
- 동작 변경. 버그를 발견해도 고치지 말고 보고만 해
- 대상 파일 밖 수정
- 이번 항목 외의 변경. 한 번에 항목 하나
- 공개 인터페이스(export, 시그니처) 변경.
  OrderService.settle()의 시그니처는 그대로 둔다

최종 보고:
- 적용 / 되돌림
- 테스트 결과 (기준 대비)
- 실제로 깨진 테스트와 예상의 차이
- 사람 리뷰가 필요한 판단이 있었으면 무엇인지
```

## 나온 결과

### 1차 시도

```
Verifier: 59 passed | 2 failed

FAIL test/characterization/settle.spec.ts > retries 3 times on gateway timeout
  expected charge to be called 3 times, got 1

FAIL test/characterization/settle.spec.ts > preserves call order
  expected [charge, reserve, send], got [reserve, charge, send]
```

첫 번째는 예상된 영향이다. 두 번째는 **예상에 없었다.**

Worker가 재시도를 추출하면서 `charge` 호출을 `RetryPolicy.run()` 안으로 옮겼고, `RetryPolicy`가 비동기로 감싸면서 `reserve`가 먼저 실행됐다.

### 판정

`preserves call order`는 `예상되는 영향`에 없는 테스트다. 규칙대로면 되돌리기 신호지만, 여기서는 원인이 명확하고 대상 파일 안이라 재시도 1회를 썼다.

> Verifier 판단: 깨진 테스트 2건이 모두 `settle.spec.ts` 안이고 원인이 하나(await 위치)로 좁혀진다. 최소 수정 1회 허용.

### 2차 시도

```
Verifier: 61 passed
```

커밋 `d17a4e8`.

### 사람 리뷰에 올린 것

> `RetryPolicy`를 `settle()` 전용으로 만들었다. `cancel()`의 `refund`도 재시도가 필요해 보이지만 현재 코드에는 재시도가 없다. 같은 정책을 적용하면 **동작이 바뀌므로** 하지 않았다. 별도 항목으로 남길지 판단이 필요하다.

## 이 예제에서 볼 것

**호출 순서 테스트가 잡아낸 것**이 이 설계가 1단계부터 준비한 것이다. 반환값만 고정했다면 61 passed로 통과했을 것이고, 결제 실패 시 재고가 먼저 잡히는 동작 변경이 조용히 들어갔을 것이다. 프로덕션에서 결제사 장애가 났을 때 재고만 빠지는 형태로 드러난다.

**예상 밖 실패에 재시도를 허용한 판단**은 규칙의 예외다. Verifier가 근거(같은 파일, 단일 원인)를 적어서 남긴 것이 중요하다. 근거 없이 예외를 열면 규칙이 사라진다. 근거가 애매했다면 되돌리고 J3을 두 항목으로 쪼개는 것이 맞다.

**`cancel()`에 재시도를 넣지 않은 것**이 동작 보존이다. 넣는 편이 나아 보이지만 그건 리팩토링이 아니라 기능 변경이고, 이 루프의 통과 판정 안에서는 검증되지 않는다. 별도 작업으로 [02](../../02-aorr-worker-verifier-loop/)를 돌릴 일이다.
