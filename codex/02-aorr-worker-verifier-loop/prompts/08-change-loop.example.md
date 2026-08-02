# 8 · 항목별 재루프 — 예제

**가상 프로젝트**: [7단계 예제](07-change-analysis.example.md)에서 이어진다. CR-5는 사람 확인으로 상한 20건 + 더보기로 확정되어 편입했다.

실행 순서: `CR-1 → CR-3 → CR-2 → CR-4 → CR-6 → CR-5`

## 프롬프트

```
[8 - 변경 요청 재루프]

MEMORY.md, CHANGES.md, DESIGN.md, 현재 코드와
마지막 정상 배포 상태를 읽고 Change Item을 의존성 순서대로 구현해.

시작 전에 MEMORY.md에 기록:
- 실행 모드와 모델
- 현재 commit
- 마지막 정상 commit·URL
- Git 상태
- Rollback 기준

실행 순서: CR-1 → CR-3 → CR-2 → CR-4 → CR-6 → CR-5

각 Change Item:
1. Verifier가 변경 전 문제 재현 및 테스트
2. Verifier가 오류·관련 파일·fingerprint 보고
3. Worker가 승인 범위 안에서 최소 수정
4. Verifier가 동일 테스트와 관련 회귀 테스트 실행
5. 실패 시 Worker가 원인 하나만 최소 Retry
6. Verifier가 재검증
7. 전체 검증 통과 시 PASSED

규칙:
- 문서에 없는 사실 생성 금지
- CHANGES.md의 대상 파일 밖 수정 금지
- 오류당 최대 4회, 동일 fingerprint 4회면 BLOCKED 또는 HITL_REQUIRED
- 테스트·완료 기준 완화 금지, 기능 제거로 우회 금지
- 인덱스 크기 상한 2MB를 올려서 통과시키지 않기
- force push, hard reset, 기록 재작성 금지
- 상세 로그는 LOOP_LOG.md에

CR-1과 CR-3은 둘 다 인덱스 크기에 영향을 준다.
각각 끝난 뒤 크기를 확인하고, 합쳐서 상한을 넘기면 중지하고 보고해.

모든 Change Item과 전체 회귀 테스트 통과 후:
- DEPLOY_APPROVAL_REQUIRED로 설정
- 변경 항목, 변경 파일, 대상, 예상 URL을 보여주고 중지
- 사용자 승인 전 commit, push, 재배포 금지

사용자 승인 후:
1. deploy_token.txt와 토큰 패턴 확인
2. commit과 push
3. 재배포
4. HTTP 200 확인
5. Verifier가 배포본 회귀 테스트
6. DEPLOYED, BLOCKED 또는 HITL_REQUIRED 기록
```

## 기대하는 로그 — 정상 흐름

```markdown
## CR-1 - 인덱스에 본문 발췌 추가

### 변경 전 Verifier
- 명령: `node scripts/check_index.mjs --field excerpt`
- exit code: 1
- 핵심 오류: excerpt 필드 없음 (200/200 문서)
- fingerprint: `INDEX-NO-EXCERPT-v1`
- 상태: ACTING

### Act
- 변경 파일: scripts/build-index.mjs
- 변경 요약: 문서별 본문 앞 200자를 excerpt로 저장

### 변경 후 Verifier
- exit code: 0
- excerpt 200/200, 인덱스 크기 1.9MB (상한 2MB)
- 회귀: 기존 질의 12/12 PASS, 문서 수 200/200
- 최종 상태: PASSED

주의: 크기가 1.4MB → 1.9MB로 증가. CR-3이 카테고리 필드를 더하면
상한을 넘길 수 있음. CR-3 전에 확인 필요.
```

## 기대하는 로그 — 상한에 걸린 경우

```markdown
## CR-3 - 인덱스에 카테고리 필드 추가

### 변경 후 Verifier
- exit code: 1
- 핵심 오류: 인덱스 크기 2.15MB — 상한 2MB 초과
- 관련 파일·라인: scripts/build-index.mjs:41
- fingerprint: `INDEX-SIZE-OVER-LIMIT-v1`
- 최종 상태: HITL_REQUIRED

보고: CR-1의 excerpt(200자)와 CR-3의 카테고리를 함께 담으면 상한을 넘는다.
상한을 올리는 것은 Guardrails 위반이므로 임의로 진행하지 않는다.

선택지를 제시하고 판단을 기다린다:
  (a) excerpt 길이를 200자에서 120자로 줄인다 — CR-1의 완료 기준 변경
  (b) 인덱스를 본문과 메타로 분리해 지연 로드한다 — 범위 확대, 새 항목 필요
  (c) 상한 2MB를 조정한다 — 사람 결정 사항
```

## 이 예제에서 눈여겨볼 것

- **프롬프트에 프로젝트 고유의 경고를 넣었다.** CR-1과 CR-3의 상호작용은 7단계 분석에서 이미 발견된 것이고, 그 발견이 실행 프롬프트로 이어졌다. 분석 산출물이 다음 단계에서 실제로 쓰인다.
- **상한 초과에서 `HITL_REQUIRED`로 간다.** Worker가 스스로 상한을 올리면 그건 기대값 완화다. 그래서 프롬프트에 "인덱스 크기 상한 2MB를 올려서 통과시키지 않기"를 따로 넣었다.
- **선택지를 제시하되 고르지 않는다.** (a)는 앞 항목의 완료 기준을 바꾸고, (b)는 범위를 넓히고, (c)는 사람 결정이다. 셋 다 에이전트가 판단할 사안이 아니다.
- **CR-1 로그의 "주의"가 다음 항목으로 넘어간다.** 크기가 1.4→1.9MB로 오른 것은 그 자체로 실패가 아니지만, 다음 항목에서 문제가 될 신호다. 이런 관찰을 남기지 않으면 CR-3에서야 갑자기 막힌다.
