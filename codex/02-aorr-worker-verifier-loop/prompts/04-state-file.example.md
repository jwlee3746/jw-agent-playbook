# 4 · 상태 파일 완성 — 예제

**가상 프로젝트**: [3단계 예제](03-verify-loop.example.md)에서 이어진다.

## 프롬프트

```
[4 - 상태 파일 작성]

앞 단계에서 만든 MEMORY.md와 DESIGN.md를 읽고,
Project Settings를 보존하면서 복구용 MEMORY.md를 완성해.

구성:
- Project Settings (기존 값 보존)
- Goal: 한 줄
- Scope / Out of Scope
- Execution: 실행 모드, 모델, 마지막 정상 commit·URL, Rollback 기준, Last test
- Current State: 상태, 완료 루프, 다음 루프, Retry, fingerprint, blocker
- Acceptance
- Guardrails
- Retry / HITL
- Recent Loops (표)

길이 규칙:
- 항목당 한 줄
- 원시 로그, 전체 명령 출력, diff, 코드 블록 저장 금지
- 같은 섹션을 반복 추가하지 말고 현재 상태를 갱신
- Project Settings를 중복 생성하거나 임의로 바꾸지 않기
- Recent Loops는 최근 3개만
- 전체 60줄 이내
- 상세 실행 기록은 LOOP_LOG.md에 저장

Guardrails:
- 확인되지 않은 사실 생성 금지, 기존 문서 임의 삭제 금지, 대규모 재작성 금지
- 테스트 삭제·완화 금지, 기능 제거로 우회 금지
- 검색 라이브러리를 포함한 외부 의존성 추가는 사전 승인 필요
- 인덱스 크기 상한을 임의로 올리지 않음
- deploy_token.txt 값을 출력·로그·코드·문서·Git에 남기지 않음

아직 코드 수정, 테스트, push, 배포는 하지 마.
```

## 기대하는 산출물 일부

설계만 끝난 시점이므로 상태는 아직 `READY`다.

```markdown
## Execution
- Mode: CODEX_WORKER + CLAUDE_VERIFIER
- 모델: Worker Codex / Verifier <3단계에서 확인한 실제 모델명>
- Fallback: Verifier 실행 불가 시 대신 검증하지 않고 BLOCKED로 보고
- 마지막 정상 commit · URL: 없음 (첫 루프 전)
- Rollback 기준: 원인별 되돌림. force push · hard reset · 기록 재작성 금지
- Last test: 미실행

## Current State
- 상태: READY
- 완료 루프: 없음
- 다음 루프: 루프 1 인덱스 생성
- Retry: 0
- fingerprint: 없음
- blocker: 인덱스 크기 상한과 질의 집합 미확정 [사람 확인 필요]

## Acceptance
- 인덱스에 문서 200개가 모두 포함되고 크기가 상한 이내
- 질의 집합의 각 항목이 기대 문서를 반환
- 결과 항목에서 해당 문서로 이동
- 검색 미사용 시 인덱스가 전송되지 않음
- 배포본에서 위 네 가지가 동일하게 동작
```

## 이 예제에서 눈여겨볼 것

- **`blocker`에 1단계에서 미룬 항목이 그대로 남아 있다.** 인덱스 크기 상한과 질의 집합이 정해지지 않았으므로 `Acceptance`의 두 줄은 아직 판정할 수 없다. 이 상태로 루프를 시작하면 Verifier가 기준 없이 판정하게 된다. **설계 단계의 `[사람 확인 필요]`는 실행 전에 해소한다.**
- **Guardrails에 프로젝트 고유 항목이 둘 있다** — 외부 의존성 사전 승인, 인덱스 크기 상한 임의 상향 금지. 두 번째가 중요하다. 상한이 있으면 통과시키려고 상한을 올리는 경로가 생기고, 그것도 "기대값 완화"의 한 형태다.
- `Rollback 기준`이 첫 루프 전부터 적혀 있다. 되돌릴 일이 생긴 뒤에 정하면 늦다.
