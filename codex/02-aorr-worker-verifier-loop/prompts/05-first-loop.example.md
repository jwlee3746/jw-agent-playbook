# 5 · 첫 루프 정확히 1회 — 예제

**가상 프로젝트**: [4단계 예제](04-state-file.example.md)에서 이어진다. 1단계에서 고른 첫 루프는 **인덱스 생성**이다.

시작 전에 4단계의 `blocker`를 해소했다고 가정한다 — 인덱스 크기 상한 2MB, 질의 집합 12개로 확정.

## 프롬프트

```
[5 - 첫 개발 루프 정확히 1회 실행]

MEMORY.md, ANALYSIS.md, DESIGN.md와 기존 파일을 읽고
가장 안전한 첫 개발 루프를 정확히 1회 실행해.

역할:
- Verifier(Claude Code CLI): 변경 전·후 테스트
- Worker(Codex): 테스트 결과에 따른 최소 코드 수정
- Verifier 실행 불가 시 Worker가 대신 검증하지 않고 BLOCKED로 보고

실행 순서:
1. MEMORY.md와 Git 상태 확인
2. Verifier와 모델 확인
3. Verifier가 변경 전 테스트 실행
4. 이번 루프의 최소 완료 기준 하나 확정
5. Worker가 필요한 최소 파일만 생성 또는 수정
6. Verifier가 동일 테스트 재실행
7. 결과를 MEMORY.md와 LOOP_LOG.md에 기록
8. 정확히 한 번의 Act 후 중지

이번 루프 범위:
- 빌드 후 인덱스 JSON을 생성하는 스크립트만 추가
- 검색 UI, 조회 로직, 결과 링크는 이번 루프에서 다루지 않음

제한:
- 검증 실패 후 두 번째 수정은 하지 마
- 실패 시 RETRYING 또는 HITL_REQUIRED로 기록
- push와 배포 금지
- MEMORY.md에는 한 줄 요약만, 상세는 LOOP_LOG.md에

최종 보고:
- 실행 모드와 모델
- 선택한 루프
- 변경 파일
- 변경 전·후 테스트 결과
- 현재 상태
- 다음 작업
```

## 기대하는 로그

`LOOP_LOG.md`에 이런 기록이 남는다.

```markdown
## 루프 1 - 인덱스 생성

- 실행 모드: CODEX_WORKER + CLAUDE_VERIFIER
- 상태 전이: READY → ACTING → VERIFYING → PASSED
- 이번 루프 완료 기준: 인덱스에 문서 200개, 스키마 준수, 크기 2MB 이내

### 변경 전 Verifier
- 실행 주체·모델: Claude Code CLI · <모델명>
- 명령: `node scripts/check_index.mjs`
- exit code: 1
- 핵심 오류: dist/search-index.json 부재
- 관련 파일·라인: 없음
- fingerprint: `INDEX-FILE-MISSING-v1`
- 상태: ACTING

### 단일 Act
- 실행 주체: Codex
- 변경 파일: scripts/build-index.mjs, package.json (build 스크립트에 연결)
- 변경 요약: 빌드 산출물을 훑어 제목·본문·경로를 인덱스 JSON으로 출력
- 제외: 검색 UI, 조회 로직, 지연 로드

### 변경 후 Verifier
- 실행 주체·모델: Claude Code CLI · <모델명>
- 명령: `npm run build && node scripts/check_index.mjs`
- exit code: 0
- 문서 수 200/200, 스키마 준수, 크기 1.4MB
- 토큰 패턴 검사: 해당 없음
- fingerprint: 없음
- 최종 상태: PASSED
```

`MEMORY.md`에는 한 줄만 들어간다.

```
| 루프1-인덱스 | PASSED | CODEX+CLAUDE | build-index.mjs, package.json | 200/200, 1.4MB | 0 | 루프2 검색 |
```

## 실패했다면

두 번째 수정을 하지 않고 여기서 멈춘다.

```markdown
### 변경 후 Verifier
- exit code: 1
- 핵심 오류: 문서 수 187/200 — 하위 디렉토리 문서가 누락됨
- 관련 파일·라인: scripts/build-index.mjs:24 (glob 패턴)
- fingerprint: `INDEX-DOC-COUNT-MISMATCH-v1`
- 최종 상태: RETRYING

다음 작업: 원인 하나(glob 패턴)에 대한 최소 수정. 이번 세션에서는 중지.
```

## 이 예제에서 눈여겨볼 것

- **"이번 루프 범위"를 프롬프트에 명시했다.** 인덱스만 만들고 검색 UI는 손대지 않는다고 못박지 않으면, 에이전트는 "검색 기능 추가"라는 큰 목표를 보고 한 번에 다 만들려 한다.
- **변경 전 Verifier가 FAIL로 시작하는 것이 정상이다.** 파일이 없으니 당연하다. 이 baseline이 있어야 변경 후 결과가 의미를 갖는다.
- 실패 예시에서 `관련 파일·라인`이 `build-index.mjs:24`까지 좁혀져 있다. Verifier 보고가 여기까지 가야 Worker가 "원인 하나"를 특정할 수 있다. "인덱스가 잘못됐다" 수준의 보고로는 최소 수정이 성립하지 않는다.
