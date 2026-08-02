# 1 · 범위와 진단 — 예제

TypeScript 주문 처리 서비스. `OrderService`가 620줄이고 결제·재고·알림이 한 클래스에 섞여 있다.

## 프롬프트

```
[1 - 범위와 진단]

다음을 계획 문서의 Project Settings에 기록해.
계획 문서가 없으면 해당 섹션만 생성해.
- 저장소: 현재 워크스페이스
- 런타임: Node 22 / TypeScript 5.6 (strict)
- 테스트: vitest
- 사용 가능한 도구: tsserver LSP, ts-morph, vitest --coverage (v8)

리팩토링 대상 후보: src/order/OrderService.ts
리팩토링 동기: 결제 재시도 정책을 바꿀 때마다 알림 관련 테스트가
같이 깨진다. 결제·재고·알림이 한 클래스의 필드와 private 메서드를
공유하고 있어서, 한 책임만 떼어 바꿀 수 없다.

1. 대상을 파일과 심볼 단위로 나열해. 모듈 이름이 아니라
   함수·클래스·메서드 단위로 적어.

2. 다음 명령을 실행해서 현재 안전망을 측정해:
   - 전체 테스트: npx vitest run
   - 커버리지: npx vitest run --coverage --coverage.reporter=text
   측정 출력을 그대로 인용해. 코드를 읽고 추정하지 마.

3. 대상별로 다음 표를 만들어:
   심볼 | 라인 수 | 분기 수 | 덮인 분기 | 덮이지 않은 경로 | 외부 호출·부수효과

4. 덮이지 않은 경로를 공백 목록으로 따로 모아. 이게 2단계 입력이 된다.

5. 대상 중 이번 범위에서 빼야 할 것이 있으면 근거와 함께 말해.

확인되지 않은 내용은 [사람 확인 필요]로 표시하고 추정하지 마.
결과를 docs/refactor-plan.md에 저장하고 핵심만 보고해.

코드 수정, 테스트 추가, 커밋은 하지 마.
```

## 나온 결과

측정 출력 (인용):

```
Test Files  4 passed (4)
     Tests  37 passed (37)

File                  | % Stmts | % Branch | Uncovered Line #s
----------------------|---------|----------|------------------
 order/OrderService.ts|   71.24 |    48.61 | 88-104,211-233,401-419,553-570
```

대상 표 (발췌):

| 심볼 | 라인 | 분기 | 덮인 분기 | 덮이지 않은 경로 | 외부 호출·부수효과 |
|---|---|---|---|---|---|
| `settle()` | 96 | 14 | 9 | 부분 환불, 결제사 타임아웃 재시도 | `paymentGateway.charge`, `inventory.reserve`, `mailer.send` — **이 순서** |
| `reserveStock()` | 71 | 11 | 7 | 재고 부족 시 부분 예약 | `inventory.reserve`, `eventBus.publish('stock.reserved')` |
| `notifyCustomer()` | 44 | 6 | 6 | — | `mailer.send`, `sms.send` |
| `#recalcTotals()` | 82 | 19 | 5 | 쿠폰 중복 적용, 세율 0 지역 | 없음 (순수) |
| `cancel()` | 58 | 9 | 4 | 결제 취소 실패 후 롤백 | `paymentGateway.refund`, `inventory.release`, `mailer.send` |

공백 목록 (2단계 입력):

1. `settle()` — 부분 환불 경로
2. `settle()` — 결제사 타임아웃 시 재시도 3회 소진
3. `reserveStock()` — 재고 부족으로 부분 예약되는 경로
4. `#recalcTotals()` — 쿠폰 2장 이상 적용
5. `#recalcTotals()` — 세율 0인 지역
6. `cancel()` — `refund` 실패 후 재고 롤백 경로

범위에서 뺄 것:

- `exportOrderCsv()` — 대상 파일에 있지만 다른 세 책임과 결합이 없다. 지금 동기와 무관하므로 이번 범위에서 뺀다.
- `#legacyPriceHook()` — [사람 확인 필요] 호출 지점을 정적 분석으로 찾지 못했다. 문자열 기반 동적 호출로 보인다. 죽은 코드인지 확인 전에는 건드리지 않는다.

## 이 예제에서 볼 것

**분기 커버리지가 48.61%**다. 라인은 71%라 얼핏 괜찮아 보이지만 절반 이상의 분기가 안 타본 경로다. 이 상태로 리팩토링을 시작하면 안 타본 쪽이 깨져도 알 수 없다.

`settle()`의 부수효과 칸에 **"이 순서"**라고 적힌 것이 중요하다. 반환값만 고정하면 에이전트가 `charge`와 `reserve`를 뒤바꿔도 테스트가 통과한다. 실제로는 결제 먼저인지 재고 먼저인지가 장애 시 동작을 완전히 바꾼다.

`#legacyPriceHook()`을 `[사람 확인 필요]`로 남긴 것도 이 단계가 하는 일이다. 죽은 코드처럼 보이는 것을 지우는 것은 리팩토링에서 가장 흔한 사고다.
