# Waza Skill Resolver

## Shared Output Marker

모든 스킬은 같은 출력 규칙을 따른다. 첫 줄에 `🥷`를 인라인으로 붙이고, 별도 단락으로 떼지 않는다. 이 규칙은 각 `SKILL.md`에 적혀 있고, `verify-skills.sh`도 이 규칙을 검증한다.

트리거 단어에서 스킬로 가는 라우팅 표. Claude Code는 각 SKILL.md의 `description`으로 자동 매칭하고, 이 문서는 사람이 보는 인덱스이자 `verify-skills.sh`의 검증 기준이다. SKILL.md의 적용 범위를 바꿀 때 같이 바꾼다.

> **행동 전에 스킬 파일을 먼저 읽는다.** 두 스킬이 동시에 매칭될 수 있으면 둘 다 읽는다. 스킬은 체이닝되도록 설계됐다(예: `/think` → 구현 → `/check`).

## 워크플로 단계별 라우팅

### Pre-build (작업 시작 전)

| 트리거 | 스킬 |
|------|------|
| 새 기능 / 아키텍처 결정 / "어떻게 설계할까" / "어떤 방안이 좋을까" / "판단해줘" / "필요할까" / "할 가치가 있을까" | `skills/think/SKILL.md` |
| UI / 컴포넌트 / 페이지 / 시각 인터페이스 / 프론트엔드 | `skills/design/SKILL.md` |

### Post-build (인도 전)

| 트리거 | 스킬 |
|------|------|
| 구현 완료 / 머지 전 / "review 해줘" / "이 코드 봐줘" | `skills/check/SKILL.md` |
| review issue / review PR / triage / 일괄 처리 / "issue 좀 봐줘" | `skills/check/SKILL.md` (Triage Mode) |

### Diagnostic (문제 발생)

| 트리거 | 스킬 |
|------|------|
| 에러 / 크래시 / 테스트 실패 / 비정상 동작 / "왜 안 돼" | `skills/hunt/SKILL.md` |
| Claude가 지시 무시 / hook 안 먹음 / MCP 이상 / 설정 점검 | `skills/health/SKILL.md` |

### Content (들어오고 나가는 콘텐츠)

| 트리거 | 스킬 |
|------|------|
| 메시지에 http(s) URL / 웹페이지 링크 / PDF 경로 / "이거 봐줘", "요약해줘" | `skills/read/SKILL.md` |
| 글쓰기 / 원고 수정 / 다듬기 / AI 티 빼기 (한국어/중국어/영어) | `skills/write/SKILL.md` |
| 낯선 도메인 깊게 조사 / 6단계 연구 → 출간 / 자료 묶음을 글로 | `skills/learn/SKILL.md` |

## Disambiguation (모호함 해소)

여러 스킬이 동시에 매칭되면 다음 규칙으로 판단한다.

1. **더 구체적인 게 우선**: `/design`은 `/think`보다 구체적이다(UI 결정 한정). 사용자가 "로그인 페이지 만들어줘"라고 하면 `/design`.
2. **URL은 콘텐츠 타입으로 2차 분기**: 메시지에 URL → 먼저 `/read`로 Markdown 추출 → 긴 연구 자료면 `/learn` 연결, 한 줄 요약이면 `/read`에서 멈춤.
3. **버그 수정 vs review**: 코드가 인도되거나 PR로 갔다 → `/check`. 코드가 안 돌거나 동작이 틀렸다 → `/hunt`. 둘 다 "한번 봐줘"에 매칭될 수 있으니 "구체 에러 현상이 있는가"로 판단.
4. **설정 이상 vs 코드 에러**: Claude 자체가 말 안 듣거나 hook이 안 뜨거나 MCP가 안 됨 → `/health`. 사용자가 짠 코드가 예외를 던짐 → `/hunt`.
5. **장문 제작 vs 다듬기**: 0에서 완성까지 → `/learn`. 이미 있는 원고 다듬기 → `/write`.
6. **판단 vs 디버깅**: "판단해줘" + 에러/예외/안 됨 → `/hunt` (문제 진단). "판단해줘" + 필요한지/유지할지/가치 있는지 → `/think` Evaluation Mode (가치 판단).
7. **마지막 안전망**: 둘 다 모호하면 두 SKILL.md의 "Not for" 절을 읽고 배제법으로. 그래도 모호하면 사용자에게 묻는다.

## Chaining (자주 쓰는 연결)

스킬 간 전환은 사용자가 수동으로 트리거한다. 자동 연결되지 않는다. 각 스킬은 작업 끝나면 멈추고 다음 단계를 사용자가 결정하길 기다린다.

- `/think` 방안 도출 → **사용자 "구현해"** → 구현 → **사용자 "/check"** → `/check` 검수
- `/read` 여러 URL 수집 → **사용자 "/learn"** → `/learn` 종합
- `/learn` 초안 → **사용자 "/write"** → `/write` AI 티 제거
- `/hunt` 근본 원인 → **사용자 "고쳐"** → 수정 → **사용자 "/check"** → `/check` 부작용 확인
- `/health` 스킬 설정 문제 발견 → **사용자 "고쳐"** → 수정 → **사용자 "/health"** → `/health` 재실행

## Latent vs Deterministic

Waza의 스킬은 모두 fat skill (Markdown 판단)이고, 결정론적 제약은 `scripts/verify-skills.sh`와 `rules/*.md`로 간다. 새 능력을 더할 때 먼저 묻는다.

- 판단 / 상황 적응 / 사용자에게 되묻기가 필요? → skill
- 입력 같으면 출력 같음 / 검증과 나열만? → script 또는 rule

lint 검사를 skill로 짜지 말고, "낯선 도메인을 어떻게 조사할까"를 셸 스크립트에 넣지 말라. 자세한 건 루트의 `CLAUDE.md` 결정 표 참고.
