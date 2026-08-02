# Frontend — 예제

**가상 작업**: [Backend 예제](backend.example.md)와 같은 작업. 재고 임계값 알림의 대시보드 화면을 만든다.

## 프롬프트

```markdown
# task_014 · Frontend 작업 명세

## 작업장
branch `task/014-frontend`, 시작 branch `master`.
worktree를 손으로 만들지 않는다. 하네스의 작업 트리 기능으로 만든다.

## 작업 지시
`index.html`, `src/app.js`, `src/styles.css` 로 재고 알림 대시보드를 만들어
`scripts/check_dom.py` 를 PASS 시킨다.

## 구현할 것
- 상단 요약: 등급별 건수 (CRITICAL / WARNING / NORMAL)와 조치 대기 건수
- 필터 바: 전체, 등급 3종, 카테고리별
- 알림 보드: 품목 카드 목록. CRITICAL 카드는 지정된 강조 클래스를 갖는다
- 오류 패널: API 호출 실패 시 메시지 표시
- 데스크톱과 모바일 폭에서 가로 스크롤이 생기지 않는다

필수 요소 식별자와 필터 목록의 정확한 값은
`tasks/acceptance.json` 의 `requiredDomIds` 와 `requiredFilters` 를 기준으로 한다.

## 유지할 것
- 기존 정적 자산 경로와 파일명
- 프레임워크·빌드 도구·외부 CDN 사용 금지. 브라우저에서 바로 열려야 한다

## 권한
- allowedFiles: `index.html`, `src/app.js`, `src/styles.css`,
  `docs/task_014/frontend/report.md`
- forbiddenFiles: `server.py`, `data/items.json`, `scripts/`, `tasks/acceptance.json`
- 서버 파일에 손대지 않는다

## API 계약
서버 구현은 Backend 역할이 병렬로 진행 중이다.
호출 경로(`/api/items`, `/api/alerts`)와 응답 필드는
`tasks/acceptance.json` 에 정의된 것을 따른다.
`server.py` 를 읽어 응답 형태를 추론하지 않는다.
API가 아직 응답하지 않아도 화면은 오류 패널로 처리되어야 한다.

## 중단 조건
같은 검사 항목이 3회 연속 실패하면 중단하고 실패 로그와 함께 보고한다.
`scripts/` 나 `tasks/acceptance.json` 을 고쳐서 통과시키지 않는다.

## 마무리
1. `python3 scripts/check_dom.py` 실행 → PASS 확인
2. `docs/task_014/frontend/report.md` 작성 — 무엇을 왜 바꿨는지, 실행한 명령과 exit code
3. 위 세 파일과 frontend report 만 `git add` → `git commit`
4. merge / rebase / push 금지
```

## 이 예제에서 눈여겨볼 것

- **"`server.py`를 읽어 응답 형태를 추론하지 않는다"가 명시돼 있다.** 같은 저장소에 있으니 읽을 수는 있지만, 이 시점의 `server.py`는 Backend가 작업 중인 미완성 코드다. 거기 맞추면 Integration에서 어긋난다.
- **"API가 아직 응답하지 않아도 오류 패널로 처리"** — 병렬 작업의 현실을 요구사항으로 바꿨다. Frontend 검증은 서버 없이도 통과할 수 있어야 한다.
- **필수 식별자를 프롬프트에 나열하지 않고 채점 파일을 가리킨다.** 여기에 id 목록을 복사하면 채점 파일과 어긋날 수 있고, 그때 어느 쪽이 맞는지 다툼이 생긴다.
- Backend 예제와 `forbiddenFiles`가 정확히 대칭이다. 두 역할이 서로의 구현 파일을 금지 목록에 넣고 있어, 감사 스크립트가 침범을 기계로 잡는다.
