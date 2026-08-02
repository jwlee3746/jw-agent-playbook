# 2 · 상태 머신 설계 — 예제

**가상 프로젝트**: [1단계 예제](01-analysis.example.md)에서 이어진다. 정적 문서 사이트에 클라이언트 사이드 전문 검색 추가.

## 프롬프트

```
[2 - 상태 머신 설계]

상태 파일의 Project Settings, ANALYSIS.md, 저장소 구조를 읽고
실행 가능한 AORR 상태 머신을 설계해.

역할:
- Worker(Codex): 코드 분석과 최소 수정
- Verifier(Claude Code CLI): 테스트 실행과 판정
- Verifier를 쓸 수 없을 때만 Worker가 수정과 테스트를 모두 수행하고,
  그 사실과 이유를 CODEX_FALLBACK으로 기록

최종 목표:
- 빌드 시점에 인덱스가 생성되고 문서 200개가 모두 포함된다
- 검색 페이지에서 지정한 질의 집합이 기대 문서를 반환한다
- 결과 항목에서 해당 문서로 이동한다
- 검색을 쓰지 않으면 인덱스를 내려받지 않는다
- 배포본에서 위 네 가지가 동일하게 동작한다

DESIGN.md에 다음만 작성해:
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
INDEX, SEARCH, UI, PERFORMANCE, CONTENT,
TEST, ENVIRONMENT, DEPLOYMENT, UNKNOWN

규칙:
- 한 Retry에서는 원인 하나와 관련 파일만 수정
- Verifier의 전체 검증이 통과해야 PASSED
- Verifier를 쓸 수 없으면 그 사실과 이유를 기록
- 불명확한 내용은 [사람 확인 필요]

아직 코드 수정, 테스트, push, 배포는 하지 마.
```

## 기대하는 산출물 일부

`DESIGN.md`의 4절(실패 원인 분류)이 이렇게 나오면 된다.

| 분류 | 무엇이 실패했을 때 | 누가 처리 |
|---|---|---|
| `INDEX` | 인덱스 생성 스크립트, 문서 수 불일치, 인덱스 스키마 | Worker |
| `SEARCH` | 조회 로직, 토큰화, 순위, 기대 문서 미반환 | Worker |
| `UI` | 검색 페이지 마크업, 결과 렌더링, 링크 연결 | Worker |
| `PERFORMANCE` | 인덱스 크기, 지연 로드, 첫 화면 전송량 | Worker |
| `CONTENT` | 문서 원본의 문제, 공개 범위 | 사람 |
| `TEST` | 질의 집합·기대값 자체가 틀림 | 임의 수정 금지, 보고 |
| `ENVIRONMENT` | 빌드 도구, 의존성, 로컬 서버 | 환경 복구 먼저 |
| `DEPLOYMENT` | 호스팅 설정, 경로, 캐시 | 환경 복구 먼저 |
| `UNKNOWN` | 증거 부족 | `HITL_REQUIRED` |

7절 루프 표는 1단계의 표를 상태 전이까지 붙여 옮긴 것이다.

| 루프 | 입력 | Worker Act | Verifier | 통과 기준 | 다음 상태 |
|---|---|---|---|---|---|
| 1 인덱스 | 빌드 산출물 | 인덱스 생성 스크립트 | 문서 수·스키마·크기 검사 | 200개 일치, 크기 상한 이내 | 다음 루프 `ACTING`, 실패 시 `RETRYING` |
| 2 검색 | 인덱스 | 조회 로직 | 질의 집합 실행, 콘솔 오류 | 기대 문서 반환 | 위와 같음 |
| 3 이동 | 검색 결과 | 결과 링크 | 링크 유효성·앵커 | 해당 문서로 이동 | 위와 같음 |
| 4 성능 | 전체 | 지연 로드 | 첫 화면 전송량 | 미사용 시 인덱스 미전송 | 위와 같음 |
| 5 회귀·배포 | 전체 | 없음 또는 원인 하나 수정 | 전체 세트 | 전부 PASS | `PASSED` → `DEPLOY_APPROVAL_REQUIRED` |

## 이 예제에서 눈여겨볼 것

- **`PERFORMANCE`를 도메인 분류에 넣었다.** 1단계에서 "로드 시간이 나빠지면 안 된다"가 제약으로 나왔기 때문이다. 제약이 있으면 그것도 실패할 수 있는 축이므로 분류에 있어야 한다.
- **`TEST` 분류의 처리가 "임의 수정 금지"다.** 질의 집합이 틀렸을 때 Worker가 그걸 고치기 시작하면, 통과시키기 위해 기대값을 낮추는 것과 구분되지 않는다.
- 루프 5의 다음 상태가 `PASSED`에서 끝나지 않고 `DEPLOY_APPROVAL_REQUIRED`로 이어진다. **검증 통과와 배포 허가는 다른 사건이다.**
