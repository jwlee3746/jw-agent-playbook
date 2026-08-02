# MEMORY

루프 상태 파일 템플릿. 프로젝트 루트에 `MEMORY.md`로 복사해 쓴다.

**길이 규칙**: 항목당 한 줄. 원시 로그·명령 출력·diff·코드 블록을 넣지 않는다. 섹션을 반복 추가하지 말고 현재 상태를 갱신한다. 전체 60줄 안쪽을 목표로 하고, 상세는 실행 로그 파일로 보낸다.

---

## Project Settings

루프 내내 바뀌지 않는 값. 한 번 적고 임의로 고치지 않는다.

- 대상 저장소 / 배포 주소:
- 인증 파일명: <파일명만. 값은 읽지도 저장하지도 않는다>
- 참고 자료:
- 그 밖의 고정 설정:

## Goal

- <한 문장>

## Scope / Out of Scope

- Scope:
- Out of Scope: <명시적 제외. 이게 없으면 루프가 돌수록 범위가 넓어진다>

## Execution

- Mode: `<WORKER 하네스> + <VERIFIER 하네스>` 또는 `<FALLBACK 표기>`
- 모델: Worker `<모델명>` / Verifier `<모델명>` — 추정하지 말고 실제로 확인한 값
- Fallback: <한쪽을 쓸 수 없을 때의 처리와 그 사실을 남길 위치>
- 마지막 정상 commit · URL: <되돌아갈 지점>
- Rollback 기준: <원인별 되돌림. force push · hard reset · 기록 재작성 금지>
- Last test: <PASS/FAIL과 한 줄 요약>

## Current State

- 상태: `READY | ACTING | VERIFYING | RETRYING | PASSED | DEPLOY_APPROVAL_REQUIRED | DEPLOYED | BLOCKED | HITL_REQUIRED`
- 완료 루프:
- 다음 루프:
- 미배포 변경:
- Retry: <횟수와 각 원인 한 줄>
- fingerprint: <현재 실패 지문>
- blocker:

## Acceptance

- <완료로 인정하는 조건. 검증 명령으로 확인 가능한 형태로 쓴다>

## Guardrails

- 확인되지 않은 사실 생성 금지, 기존 콘텐츠 임의 삭제 금지, 대규모 재작성 금지
- 테스트 삭제·완화 금지, 기능 제거로 우회 금지
- 승인 없는 외부 의존성·서비스 추가 금지
- 비밀정보를 출력·로그·코드·문서·Git에 남기지 않음
- <프로젝트별 금지 사항>

## Retry / HITL

- 오류당 최대 <N>회. 동일 fingerprint <N>회면 중지
- 한 Retry에는 원인 하나와 최소 파일만
- HITL로 넘길 조건: <배포 승인, 공개 범위, 보안, 요구 충돌, 권한 문제 등>
- 상세 실행 기록 위치: `<로그 파일명>`

## Recent Loops

최근 3개만 유지한다.

| Loop | 상태 | 실행 모드·모델 | 변경 파일 | 테스트 결과 | Retry | 다음 작업 |
|---|---|---|---|---|---:|---|
| | | | | | | |
