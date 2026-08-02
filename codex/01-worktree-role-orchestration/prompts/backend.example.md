# Backend — 예제

**가상 작업**: 재고 임계값 알림. 품목의 재고 수준과 소진 속도를 받아 알림 등급과 담당 팀을 판정하는 API를 만든다.

## 프롬프트

```markdown
# task_014 · Backend 작업 명세

## 작업장
branch `task/014-backend`, 시작 branch `master`.
worktree를 손으로 만들지 않는다. 하네스의 작업 트리 기능으로 만든다.

## 작업 지시
`server.py` 의 `classify_stock` 과 `POST /api/alerts` 를 완성해서
`scripts/check_api.py` 를 PASS 시킨다.

## 구현할 것
품목 하나를 받아 알림 등급, 필요한 조치 목록, 담당 팀을 판정한다.

- level
  - CRITICAL: 잔여일수 3일 이하이고 대체품이 없음
  - CRITICAL: 또는 잔여일수 7일 이하이고 등급이 A
  - WARNING: 잔여일수 14일 이하, 또는 소진속도가 평균의 2배 이상
  - NORMAL: 그 외
- actions
  - 기본 ["Notify"]
  - CRITICAL 또는 WARNING 이면 "Reorder" 추가
  - 대체품이 없으면 "FindAlternative" 추가
  - 등급 A 이면 "EscalateToPlanning" 추가
  - 중복 제거
- owner: 카테고리별 담당 팀
- reasons: 판정 근거 문자열 배열 (참고용)

정확한 기대값은 `tasks/acceptance.json` 의 `apiCases` 와 `levelRules` 를 기준으로 한다.

## 유지할 것
- `GET /api/items` 는 매 요청마다 `data/items.json` 을 다시 읽는다 (이미 그렇게 되어 있음)
- `GET /api/health` 는 그대로 동작해야 한다
- 표준 라이브러리만 사용. 외부 패키지 금지

## 권한
- allowedFiles: `server.py`, `docs/task_014/backend/report.md`
- forbiddenFiles: `index.html`, `src/app.js`, `src/styles.css`,
  `data/items.json`, `scripts/`, `tasks/acceptance.json`
- 프론트엔드 파일에 손대지 않는다

## 중단 조건
같은 케이스가 3회 연속 실패하면 중단하고 실패 로그와 함께 보고한다.
`scripts/` 나 `tasks/acceptance.json` 을 고쳐서 통과시키지 않는다.

## 마무리
1. `python3 scripts/check_api.py` 실행 → PASS 확인
2. `docs/task_014/backend/report.md` 작성 — 무엇을 왜 바꿨는지, 실행한 명령과 exit code
3. `server.py` 와 backend report 만 `git add` → `git commit`
   - 예: `git commit -m "feat(task_014): implement stock alert classification"`
4. merge / rebase / push 금지
```

## 이 예제에서 눈여겨볼 것

- **판정 규칙을 서술했는데도 "정확한 기대값은 `acceptance.json`을 기준으로 한다"가 붙어 있다.** 서술은 사람이 읽고 이해하기 위한 것이고, 판정은 데이터 파일이 한다. 둘이 어긋나면 데이터 파일이 이긴다.
- **`유지할 것`에 "이미 그렇게 되어 있음"이 적혀 있다.** 새로 만들라는 게 아니라 깨뜨리지 말라는 뜻이다. 이 구분이 없으면 에이전트가 멀쩡한 코드를 다시 쓴다.
- **중단 조건에 "기준을 고쳐서 통과시키지 않는다"가 함께 있다.** 3회 실패하면 대부분 기준을 의심하게 되는데, 그 경로를 막아둔 것이다.
- `git add`할 대상을 명시했다. `git add -A`를 쓰면 금지 파일이 딸려 들어간다.
