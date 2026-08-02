# 원작 README 한국어판

원작: [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) (저자 Jiayuan / forrestchang, MIT)
원작 커밋 `2c60614` (2026-04-20) 기준. 옮긴이 주는 인용 표시 없이 문단 끝에 적었다.

Claude Code의 행동을 개선하는 `CLAUDE.md` 파일 하나. Andrej Karpathy가 [X에 남긴 관찰](https://x.com/karpathy/status/2015883857489522876)에서 출발했다.

## 문제

Karpathy가 지적한 것을 원작이 인용한 대목이다.

> "모델은 사용자를 대신해 잘못된 가정을 세우고, 확인 없이 그대로 밀고 나간다. 자신의 혼란을 관리하지 않고, 명확히 해달라고 요청하지 않고, 모순을 드러내지 않고, 트레이드오프를 제시하지 않고, 반대해야 할 때 반대하지 않는다."

> "코드와 API를 지나치게 복잡하게 만들고, 추상화를 부풀리고, 죽은 코드를 정리하지 않는다... 100줄이면 될 것을 1000줄짜리 구조로 만든다."

> "작업과 무관한데도, 충분히 이해하지 못한 주석과 코드를 부수 효과로 바꾸거나 지운다."

## 해법

파일 하나에 담은 네 가지 원칙이 각 문제에 직접 대응한다.

| 원칙 | 대응하는 문제 |
|---|---|
| **Think Before Coding** (만들기 전에 생각한다) | 잘못된 가정, 숨긴 혼란, 빠뜨린 트레이드오프 |
| **Simplicity First** (단순한 것부터) | 과도한 복잡화, 부풀린 추상화 |
| **Surgical Changes** (필요한 곳만 건드린다) | 작업과 무관한 수정, 건드리면 안 되는 코드 |
| **Goal-Driven Execution** (목표 기반 실행) | 테스트 우선·검증 가능한 완료 조건으로 얻는 레버리지 |

네 원칙의 상세 내용은 같은 디렉토리의 [`CLAUDE-kr.md`](CLAUDE-kr.md)에 있다. 원문은 [`CLAUDE.md`](CLAUDE.md).

## 설치

**방법 A: Claude Code 플러그인 (원작 권장)**

Claude Code 안에서 마켓플레이스를 먼저 추가한다.

```
/plugin marketplace add forrestchang/andrej-karpathy-skills
```

그다음 플러그인을 설치한다.

```
/plugin install andrej-karpathy-skills@karpathy-skills
```

이렇게 하면 스킬 형태로 설치되어 모든 프로젝트에서 쓸 수 있다.

**방법 B: CLAUDE.md 파일 (프로젝트별)**

새 프로젝트:

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md
```

기존 프로젝트에 덧붙이기:

```bash
echo "" >> CLAUDE.md
curl https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md >> CLAUDE.md
```

한국어판을 쓰려면 이 디렉토리의 `CLAUDE-kr.md` 내용을 프로젝트 `CLAUDE.md`에 붙인다.

## Cursor에서 쓰기

원작 레포에는 Cursor 프로젝트 룰(`.cursor/rules/karpathy-guidelines.mdc`)이 함께 커밋되어 있어, 같은 프로젝트를 Cursor로 열어도 동일한 지침이 적용된다. 설정 방법은 원작의 `CURSOR.md`를 본다.

## 핵심 통찰

> "LLM은 구체적인 목표에 도달할 때까지 반복하는 데 대단히 능하다... 무엇을 하라고 말하지 말고, 완료 조건을 주고 지켜봐라."

네 번째 원칙이 이걸 담고 있다. 명령형 지시를 검증 루프가 딸린 선언형 목표로 바꾸는 것이다.

## 잘 되고 있는지 판단하는 법

- **디프에 불필요한 변경이 줄어든다** — 요청한 변경만 나타난다
- **과설계로 인한 재작성이 줄어든다** — 처음부터 단순하게 나온다
- **질문이 구현 전에 나온다** — 실수한 뒤가 아니라
- **PR이 작고 깔끔하다** — 지나가다 하는 리팩터링이나 "개선"이 없다

## 커스터마이징

프로젝트별 지침과 병합해서 쓰도록 설계됐다. 기존 `CLAUDE.md`에 추가하거나 새로 만든다.

프로젝트 규칙은 이런 식으로 덧붙인다.

```markdown
## 프로젝트 규칙

- TypeScript strict 모드를 쓴다
- 모든 API 엔드포인트에 테스트가 있어야 한다
- 에러 처리는 `src/utils/errors.ts`의 기존 패턴을 따른다
```

## 트레이드오프

이 지침은 **속도보다 신중함** 쪽으로 치우쳐 있다. 사소한 작업(오타 수정, 뻔한 한 줄짜리)에는 판단해서 적용한다 — 모든 변경에 전체 절차가 필요하진 않다.

목적은 사소한 작업을 늦추는 게 아니라, 사소하지 않은 작업에서 비용이 큰 실수를 줄이는 것이다.

## 라이선스

원작은 MIT. 이 번역본도 같은 조건을 따른다.
