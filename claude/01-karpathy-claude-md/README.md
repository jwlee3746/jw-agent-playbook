# 01 · Karpathy CLAUDE.md

**한 줄**: LLM 코딩의 대표적 실패 4가지(성급한 가정, 과설계, 범위 밖 수정, 모호한 완료 기준)를 막는 행동 규칙 파일.
**출처**: Forrest Chang (multica-ai) / https://github.com/multica-ai/andrej-karpathy-skills
**상태**: 미검증
**스냅샷**: 2026-08-02 확인, 커밋 `2c60614` (2026-04-20), 198.6k stars
**라이선스**: 명시 없음 — 원문을 담지 않고 링크와 발췌만 둔다

Karpathy 본인이 쓴 파일이 아닙니다. 2026년 1월 그가 LLM 코딩 실패 유형에 대해 남긴 관찰을 커뮤니티가 설정 파일로 정리한 것입니다. 인용할 때 저자를 혼동하지 않도록 주의합니다.

## 용도

프로젝트 루트나 `~/.claude/CLAUDE.md`에 넣어 에이전트의 기본 행동을 교정할 때. 하네스 오케스트레이션 설계가 아니라 **단일 에이전트의 행동 규칙**이므로, 이 레포의 다른 항목들과 층위가 다릅니다.

원문 스스로 밝히는 트레이드오프: 속도보다 신중함에 치우쳐 있어 사소한 작업에는 과합니다.

## 전제

- 하네스 / 모델: 무관 (Claude Code는 `CLAUDE.md`, Codex는 `AGENTS.md`로 같은 내용을 읽음)
- 필요한 툴·권한: 없음
- 레포·환경 조건: 프로젝트별 지침과 병합해서 씀

## 구성

원문 파일은 담지 않습니다. 라이선스가 명시되지 않아 이 레포(MIT)로 재배포할 수 없습니다.

| 원문 파일 | 역할 |
|---|---|
| `CLAUDE.md` | 아래 4원칙 본문 |
| `CURSOR.md` | Cursor용 같은 내용 |
| `EXAMPLES.md` | 규칙 적용 전후 비교 예시 |
| `skills/karpathy-guidelines/` | Agent Skills 형식 패키징 |

## 발췌

각 원칙의 표제 문장만 옮깁니다. 전문은 위 URL에서 확인합니다.

1. **Think Before Coding** — "Don't assume. Don't hide confusion. Surface tradeoffs."
2. **Simplicity First** — "Minimum code that solves the problem. Nothing speculative."
3. **Surgical Changes** — "Touch only what you must. Clean up only your own mess."
4. **Goal-Driven Execution** — "Define success criteria. Loop until verified."

3번의 판정 기준이 가장 구체적입니다: *"Every changed line should trace directly to the user's request."*

4번은 작업을 검증 가능한 목표로 바꾸라고 요구합니다. "Add validation" → "Write tests for invalid inputs, then make them pass".

## 결과

아직 적용해보지 않았습니다. 미검증 상태이므로 비워둡니다.

검증할 때 볼 것: 원문이 제시하는 성공 신호는 (1) 디프에 불필요한 변경이 줄어드는가, (2) 과설계로 인한 재작성이 줄어드는가, (3) 실수 후가 아니라 구현 전에 질문이 나오는가입니다.
