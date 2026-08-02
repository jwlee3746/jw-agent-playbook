# 2 · 상태 머신 설계 — 템플릿

1단계에서 쪼갠 루프를 상태 머신으로 고정한다. 여전히 코드는 건드리지 않는다.

## 프롬프트

```
[2 - 상태 머신 설계]

상태 파일의 Project Settings, <분석 문서>, 저장소 구조를 읽고
실행 가능한 AORR 상태 머신을 설계해.

역할:
- Worker(<하네스>): 코드 분석과 최소 수정
- Verifier(<하네스>): 테스트 실행과 판정
- <한쪽을 쓸 수 없을 때의 fallback 조건>

최종 목표:
- <완료 상태를 항목으로>

<설계 문서>에 다음만 작성해:
1. Target과 완료 기준
2. Act: Worker가 수행할 최소 수정
3. Observe: Verifier가 실행할 테스트와 수집할 결과
4. Reason: 실패 원인 분류
5. Repeat: 최소 수정 → 동일 테스트 재실행
6. Stop과 HITL 조건
7. 개발 루프 표: 루프 | 입력 | Act | Verify | 통과 기준 | 다음 상태

상태:
READY, ACTING, VERIFYING, RETRYING, PASSED,
DEPLOY_APPROVAL_REQUIRED, DEPLOYED, BLOCKED, HITL_REQUIRED

실패 분류:
<도메인 계열 분류들>, TEST, ENVIRONMENT, DEPLOYMENT, UNKNOWN

규칙:
- 한 Retry에서는 원인 하나와 관련 파일만 수정
- Verifier의 전체 검증이 통과해야 PASSED
- Verifier를 쓸 수 없으면 그 사실과 이유를 기록
- 불명확한 내용은 [사람 확인 필요]

아직 코드 수정, 테스트, push, 배포는 하지 마.
```

## 채울 것

| 자리 | 무엇을 넣나 |
|---|---|
| `<분석 문서>` | 1단계 산출물 |
| `<하네스>` | Worker와 Verifier에 각각 다른 것을 넣는다 |
| `<완료 상태를 항목으로>` | 무엇이 되면 끝인지. 판정 가능한 형태로 |
| `<설계 문서>` | 상태 머신을 저장할 파일명 |
| `<도메인 계열 분류들>` | 작업 대상의 계층별 분류 |

## 주의

**실패 분류에서 도메인 계열과 비도메인 계열을 나눈다.** 뒤의 넷(`TEST`, `ENVIRONMENT`, `DEPLOYMENT`, `UNKNOWN`)은 고정으로 두는 편이 낫다. 이 넷이 없으면 환경 문제나 검증 자체의 결함이 전부 도메인 분류로 흘러들어가고, Worker가 코드로 우회하려 든다.

상태 아홉 개 중 `BLOCKED`와 `HITL_REQUIRED`가 빠지기 쉽다. 빠지면 막혔을 때 갈 곳이 없어 무한 재시도로 간다.
