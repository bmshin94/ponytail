# Ponytail 전수조사 & 활용 가이드 (한국어)

> 이 문서는 `bmshin94/ponytail` 레포를 전수조사하고, 설치/활용/수익화까지
> 분석한 내용을 정리한 기록입니다.

## 🔗 GitHub 주소

| 구분 | 주소 |
|---|---|
| **이 레포 (포크)** | https://github.com/bmshin94/ponytail |
| **원본 레포 (upstream)** | https://github.com/DietrichGebert/ponytail |
| 원작자 | https://github.com/DietrichGebert |
| npm 패키지 | https://www.npmjs.com/package/@dietrichgebert/ponytail |
| 이슈 트래커 | https://github.com/DietrichGebert/ponytail/issues |
| 한국어 README (레포 내장) | https://github.com/DietrichGebert/ponytail/blob/main/README.ko.md |
| 에이전트 이식 문서 | https://github.com/DietrichGebert/ponytail/blob/main/docs/agent-portability.md |
| 에이전틱 벤치마크 리포트 | https://github.com/DietrichGebert/ponytail/blob/main/benchmarks/results/2026-06-18-agentic.md |
| 공식 사이트 (대기명단) | https://ponytail.dev/soon |

- 버전: **v4.10.0**
- 라이선스: **MIT**
- 패키지명: `@dietrichgebert/ponytail`

---

## 1. 이게 뭐하는 거야?

### 한 줄 요약
> **AI 에이전트에게 "게으른 시니어 개발자" 인격을 주입해서, 쓸데없이 많은 코드를
> 짜는 것을 막아주는 규칙(ruleset) 모음집.**

### 캐릭터 설정
README 첫 문단이 프로젝트 전체를 설명한다.

> *"You know him. Long ponytail. Oval glasses. Has been at the company longer
> than the version control. You show him fifty lines; he looks at them, says
> nothing, and replaces them with one."*

슬로건:
- *"He says nothing. He writes one line. It works."*
- *"The best code is the code you never wrote."*

### Before / After
"날짜 선택기 만들어줘" 라고 했을 때:

**그냥 AI** → `flatpickr` 설치 + 래퍼 컴포넌트 + 스타일시트 + 타임존 토론 (404줄)

**Ponytail** →
```html
<!-- ponytail: browser has one -->
<input type="date">
```
(23줄, -94%)

---

## 2. 핵심 엔진: The Ladder (사다리)

`skills/ponytail/SKILL.md`의 심장부. 코드 쓰기 전에 **위에서부터 내려오다가
걸리는 첫 칸에서 멈춘다**(stop at the first rung that holds).

```
1. 이게 애초에 존재할 필요가 있나?   → 없음: 스킵 (YAGNI)
2. 이 코드베이스에 이미 있나?        → 있음: 재사용 (다시 짜지 마)
3. 표준 라이브러리가 해주나?         → 해줌: 그거 써
4. 플랫폼 native 기능이 있나?        → 있음: 그거 써
5. 이미 설치된 의존성이 해주나?      → 해줌: 그거 써
6. 한 줄로 되나?                    → 한 줄로
7. 여기까지 왔으면: 작동하는 최소한만
```

중요: 사다리는 **문제를 이해한 "다음"에** 타는 것이다. 이해를 대체하지 않는다.
> *"Lazy about the solution, never about reading."*

### 버그 수정 = 증상이 아니라 근본 원인
티켓은 증상만 말한다. 고치기 전에 그 함수를 호출하는 곳을 전부 grep한다.
공유 함수에 가드 하나 넣는 것이 호출자마다 넣는 것보다 **diff가 더 작고**,
티켓에 적힌 경로만 패치하면 형제 호출자들은 계속 깨진 상태로 남는다.
> *"The lazy fix IS the root-cause fix."*

### `ponytail:` 주석 = 외상 장부
일부러 대충 만든 자리에 메모를 남긴다.
```python
# ponytail: global lock, per-account locks if throughput matters
#           ↑ 한계(ceiling)   ↑ 업그레이드 조건(upgrade path)
```
`/ponytail-debt`가 이걸 전부 긁어모아 장부로 만들고, **업그레이드 조건이 없는
메모는 `no-trigger` 태그**로 찝어준다. ("나중에"가 "절대 안 함"이 되지 않게)

---

## 3. 게으르면 안 되는 것 (When NOT to be lazy)

이 프로젝트를 장난이 아니게 만드는 부분. **절대 생략 안 함:**

- 신뢰 경계(trust boundary)에서의 입력 검증
- 데이터 손실을 막는 에러 핸들링
- 보안
- 접근성(a11y) 기본
- 사용자가 명시적으로 요청한 것 (풀버전 달라고 하면 그냥 만들어줌)
- **문제 이해하기** — 가장 중요
- 하드웨어 캘리브레이션 (실제 시계는 drift하고, 센서는 오차가 있다)
- **비자명한 로직엔 runnable check 1개** (assert 기반 self-check 또는 작은 테스트 파일 하나. 프레임워크 금지, 테스트에도 YAGNI)

> *"Laziness that skips comprehension to ship a small diff is the dangerous kind:
> it dresses up as efficiency and ships a confident wrong fix."*

---

## 4. 측정된 성능

실제 오픈소스 레포(`fastapi/full-stack-fastapi-template`)에 헤드리스 Claude Code
세션으로 기능 티켓 12개를 수행시키고 `git diff`로 채점 (n=4, Haiku 4.5).

| 스킬 없는 베이스라인 대비 | 코드량 | 토큰 | 비용 | 시간 | 안전성 |
|---|--:|--:|--:|--:|--:|
| **ponytail** | **-54%** | **-22%** | **-20%** | **-27%** | **100%** |
| caveman (간결 말투 대조군) | -20% | +7% | +3% | +2% | 100% |
| "YAGNI + 한줄로" 프롬프트 | -33% | -14% | -21% | -30% | ⚠️ 95% |

**포인트 2개**
1. 모든 지표를 다 깎는 **유일한** 방식 (caveman은 토큰/비용/시간이 오히려 증가)
2. 그냥 "짧게 써줘" 프롬프트는 **안전장치를 하나 빠뜨린다(95%)**. ponytail은 100% 유지.

### 단일샷 예제 비교 (`examples/`, 실제 벤치마크 원문)
| 예시 | 스킬 없이 | Ponytail |
|---|--:|--:|
| 이메일 검증 | 75줄 | **3줄** |
| Debounce | 116줄 | **10줄** |
| CSV 합계 | 20줄 | **3줄** |
| React 카운트다운 | 267줄 | **9줄** |
| Rate Limiting | 128줄 | **10줄** |

### 정직성(이 프로젝트의 미덕)
처음엔 "80~94% 감소"로 발표했으나, 이슈 #126에서 *"베이스라인이 설명문을
떡칠해서 줄 수가 뻥튀기된 것"*이라는 지적을 받고 **수치를 내려 정정**했다.
README에 옛 수치를 지우지 않고 접어둔 채 "왜 틀렸는지"까지 남겨뒀다.

`skills/ponytail-gain/SKILL.md`에는 **Honesty boundary** 섹션이 있어,
"이 레포에서 N줄 절약했습니다" 같은 **레포별 절약 수치 출력을 금지**한다.
안 쓴 코드는 비교할 원본이 없기 때문. (환각 방지 패턴의 좋은 예)

---

## 5. 강도 3단계

| 레벨 | 명령어 | 성격 |
|---|---|---|
| **lite** | `/ponytail lite` | 요청대로 만들고, 더 게으른 대안을 한 줄로 알려줌. 선택은 사용자. |
| **full** | `/ponytail` | **기본값.** 사다리 강제. 최단 diff, 최단 설명. |
| **ultra** | `/ponytail ultra` | YAGNI 극단주의자. 한 줄 던지고 요구사항 자체에 토를 단다. |

"API 응답 캐시 추가해줘"에 대한 반응:
- **lite**: "했어. 참고로 `functools.lru_cache`면 한 줄인데."
- **full**: "`@lru_cache(maxsize=1000)` 붙였음. 커스텀 캐시 클래스 스킵."
- **ultra**: "프로파일러가 필요하다고 할 때까진 캐시 없음. 손으로 만든 TTL 캐시
  클래스는 히트율 달린 버그 농장이야."

### 기본 모드 설정
우선순위: **환경변수 > 설정파일 > `full`**
```bash
export PONYTAIL_DEFAULT_MODE=ultra
```
```json
// ~/.config/ponytail/config.json (Windows: %APPDATA%\ponytail\config.json)
{ "defaultMode": "lite" }
```

끄기: `"stop ponytail"` 또는 `"normal mode"`를 **메시지 전체로** 보내기 (`/ponytail off`도 가능)

---

## 6. 폴더 전수조사

### 두뇌 (실제 행동 정의)
| 경로 | 내용 |
|---|---|
| `skills/ponytail/SKILL.md` | **본체.** 사다리 + 룰 + 강도 + 금지사항 |
| `skills/ponytail-review/SKILL.md` | diff의 과설계만 전문 리뷰 → 삭제 리스트 |
| `skills/ponytail-audit/SKILL.md` | 레포 **전체** 과설계 감사, 큰 삭제건부터 랭킹 |
| `skills/ponytail-debt/SKILL.md` | `ponytail:` 주석 → 기술부채 장부 |
| `skills/ponytail-gain/SKILL.md` | 벤치마크 성과 ASCII 스코어보드 |
| `skills/ponytail-help/SKILL.md` | 명령어 치트시트 |
| `AGENTS.md` | 스킬 못 읽는 에이전트용 **압축 룰셋 1장** (핵심 이식 수단) |

#### 리뷰 태그 체계
- `delete:` 죽은 코드 → 대체물 없음
- `stdlib:` 표준 라이브러리에 있는 걸 손으로 짬 → 함수명 지적
- `native:` 플랫폼이 이미 하는 걸 라이브러리로 함
- `yagni:` 구현체 1개인 추상화, 아무도 안 쓰는 config
- `shrink:` 같은 로직 더 짧게

출력 예시:
```
L12-38: stdlib: 27줄 validator 클래스. email에 "@" 있나 체크, 1줄.
L4: native: moment.js를 포맷 1번 호출에 import. Intl.DateTimeFormat, 0 deps.
repo.py:L88: yagni: 구현체 1개인 AbstractRepository. 두 번째 생길 때까지 인라인.
net: -94 lines possible.
```

### 손발 (자동 실행 장치)
| 경로 | 내용 |
|---|---|
| `hooks/claude-codex-hooks.json` | 훅 3개 등록: `SessionStart`, `UserPromptSubmit`, `SubagentStart` |
| `hooks/ponytail-activate.js` | 세션 시작 시 모드 플래그 쓰고 룰셋을 숨은 컨텍스트로 주입 |
| `hooks/ponytail-instructions.js` | `SKILL.md`를 읽어 **현재 레벨 해당 줄만 필터링**해 주입 (읽기 실패 시 하드코딩 fallback 보유) |
| `hooks/ponytail-config.js` | 모드 해석 우선순위 + 크로스플랫폼 설정 경로 처리 |
| `hooks/ponytail-mode-tracker.js` | `/ponytail ultra` 같은 레벨 전환 감지 |
| `hooks/ponytail-statusline.sh/.ps1` | 터미널에 `[PONYTAIL:ULTRA]` 배지 |
| `hooks/cursor-hooks.json`, `copilot-hooks.json`, `qoder-hooks.json` | 호스트별 훅 맵 |

**왜 훅이 필요한가**: 스킬은 "AI가 필요하다고 판단할 때" 발동하지만, 훅은
"무조건 매번" 발동한다. 대화가 길어지면 AI가 초반 지시를 흘리는(drift) 문제를
**메커니즘으로 강제 해결**한 것. `SKILL.md`에도 명시: *"ACTIVE EVERY RESPONSE."*

**코드 품질 디테일**
- `isDeactivationCommand()`: "normal mode"를 메시지 어디서든 매칭하다가
  "normal mode 토글 추가해줘" 같은 평범한 요청에서 꺼져버리는 버그가 있었고,
  **메시지 전체가 정확히 그 명령어일 때만** 끄도록 고쳤다. (자기 룰인 '근본 원인
  고치기'를 자기 코드에도 적용)
- `isShellSafe()`: 설치 경로에 쉘 메타문자가 있으면 명령 삽입 대신 수동 안내로
  폴백한다. (명령 삽입 방어)

### 어댑터 (20개 에이전트 지원)
```
.claude-plugin/    → Claude Code (플러그인 + 마켓플레이스)
.codex-plugin/     → OpenAI Codex
.cursor/rules/     → Cursor (룰 파일)
.windsurf/rules/   → Windsurf
.clinerules/       → Cline
.kiro/steering/    → Kiro
.qoder/, .qoder-plugin/ → Qoder
.grok-plugin/      → Grok Build
.devin-plugin/     → Devin CLI
.openclaw/skills/  → OpenClaw (ClawHub 배포)
.opencode/         → OpenCode (플러그인 .mjs + 커맨드)
.agents/rules/     → Antigravity 등 범용
.github/copilot-instructions.md, .github/plugin/ → GitHub Copilot (에디터 + CLI)
pi-extension/      → pi 에이전트 하네스
plugin.yaml + __init__.py → Hermes Agent (Python 플러그인)
gemini-extension.json → Gemini CLI / Antigravity CLI
ponytail-mcp/      → MCP 서버
commands/*.toml    → 슬래시 커맨드 6개
```

**Adapter Rule** (`docs/agent-portability.md`):
> *"어댑터는 얇게 유지한다. 호스트가 skill/hook을 지원하면 기존 `skills/`,
> `hooks/`를 가리키게만 한다. 프로젝트 instruction만 지원하면 `AGENTS.md`와
> 텍스트를 동기화한다."*

내용은 `skills/` + `AGENTS.md` 단 두 곳(단일 진실 공급원)에만 있고,
`scripts/check-rule-copies.js`가 복사본 동기화를 검사하며 `npm test`가 이를 강제한다.

### 검증 & 기타
| 경로 | 내용 |
|---|---|
| `benchmarks/` | promptfoo 설정 4종(Claude/GPT/GPT-newest/Gemini), 에이전틱 러너, 로컬 벤치, 안전성 감사, 결과 리포트 10개 |
| `benchmarks/arms/` | 대조군 (baseline / caveman / ponytail) |
| `benchmarks/robustness-audit.js` | 안전성 회귀 테스트 (룰이 보안을 깎는지 검증) |
| `benchmarks/correctness.js`, `behavior.yaml` | "짧아졌는데 작동은 하나" 게이트 |
| `tests/behavior.test.js` | 동작 테스트 (`npm test`가 pi-extension, ponytail-mcp까지 함께 실행) |
| `scripts/uninstall.js` | 플러그인 밖에 남는 흔적 청소 |
| `.github/workflows/` | `test.yml` (CI), `publish.yml` (npm 배포) |
| `.env.example` | `ANTHROPIC_API_KEY` — **벤치마크 재현 전용** |
| `README.ko.md`, `README.es.md` | 한국어 / 스페인어 README (이미 내장) |

---

## 7. 설치 및 사용법

### Claude Code (추천)
```
/plugin marketplace add DietrichGebert/ponytail
```
```
/plugin install ponytail@ponytail
```
> ⚠️ **두 개를 별도 메시지로** 보내야 설치된다.

포크를 쓰려면 `bmshin94/ponytail`로 대체.

### 업데이트
```
/plugin marketplace update ponytail
/reload-plugins
```
또는 `/plugin` → Marketplaces → ponytail → Enable auto-update.

### 삭제 (순서 중요)
```bash
node scripts/uninstall.js   # 먼저! (플러그인 밖 흔적 청소)
```
```
/plugin remove ponytail     # 그 다음
```
> 순서를 반대로 하면 `uninstall.js` 자체가 플러그인 파일이라 같이 지워진다.
> 청소 대상: `~/.claude/.ponytail-active`, `~/.config/ponytail/config.json`,
> `~/.cursor/hooks.json`의 ponytail 항목, `~/.claude/settings.json`의 statusLine
> (ponytail 자기 스크립트를 가리킬 때만 제거).

### 호스트별 설치
| 호스트 | 명령 |
|---|---|
| Claude Code | `/plugin marketplace add ...` + `/plugin install ponytail@ponytail` |
| Codex | `codex plugin marketplace add ...` → `codex plugin add ponytail@ponytail` → `/hooks` 승인 |
| Copilot CLI | `copilot plugin marketplace add ...` + `copilot plugin install ponytail@ponytail` |
| Gemini CLI | `gemini extensions install https://github.com/DietrichGebert/ponytail` |
| Antigravity CLI | `agy plugin install https://github.com/DietrichGebert/ponytail` |
| Cursor (훅) | `git clone ...` → `node ponytail/scripts/cursor-hooks.js install` |
| OpenCode | `opencode.json`에 `{ "plugin": ["@dietrichgebert/ponytail"] }` |
| pi | `pi install git:github.com/DietrichGebert/ponytail` |
| Devin CLI | `devin plugins install DietrichGebert/ponytail` |
| Grok Build | `grok plugin install DietrichGebert/ponytail --trust` → 활성화 |
| Hermes | `hermes plugins install DietrichGebert/ponytail --enable` |
| OpenClaw | `clawhub install ponytail` |
| Swival | `swival skills add --global <url>` → `swival skills add ponytail` |
| Windsurf/Cline/Kiro/Qoder/Copilot Chat | 해당 룰 파일 복사 |
| Zed / Amp / Jules / Junie / CodeWhale / VS Code+Codex | `AGENTS.md`만 복사 — **제로 설정** |

### 가장 간단한 설치 (설치 없는 설치)
`AGENTS.md` 한 장을 프로젝트 루트에 복사하면 Codex, Zed, Amp, Jules, CodeWhale,
Antigravity, Qoder, Swival이 전부 자동으로 읽는다. 슬래시 명령은 안 되지만
**룰은 완전히 적용된다.**

### 명령어
| 명령 | 하는 일 |
|---|---|
| `/ponytail` | 현재 레벨 확인 |
| `/ponytail lite\|full\|ultra\|off` | 강도 변경 |
| `/ponytail-review` | 현재 diff 과설계 리뷰 → 삭제 리스트 |
| `/ponytail-audit` | 레포 전체 과설계 감사 |
| `/ponytail-debt` | `ponytail:` 주석 → 기술부채 장부 |
| `/ponytail-gain` | 벤치마크 성과 스코어보드 |
| `/ponytail-help` | 치트시트 |

### 추천 실전 워크플로
```
1. 기능 만들 때       → 기본 full 모드로 작업 (자동 적용)
2. 코딩 끝나고        → /ponytail-review
3. 레거시 리팩토링 전 → /ponytail-audit
4. 월 1회            → /ponytail-debt
5. 열받는 코드 만나면 → /ponytail ultra
```

### 설치 전 체크
- **Node.js가 PATH에 있어야 함** (훅 2개가 Node로 실행. nvm/Nix는 비대화형 쉘
  PATH에도 있어야 함). 없으면 훅은 조용히 꺼지고 스킬만 작동.
- Claude Code 최신 버전 (`/plugin` 미인식 시 `npm i -g @anthropic-ai/claude-code@latest`)
- API 키 **불필요**

---

## 8. 플러그인? 스킬? MCP? → "전부 다"

```
        본질 (진짜 내용)
   ┌─────────────────────────────┐
   │  skills/ponytail/SKILL.md   │  ← "스킬"
   │  AGENTS.md (압축판)          │  ← "룰 파일"
   └──────────────┬──────────────┘
                  │ 하나를 여러 방식으로 포장
   ┌──────────────┼──────────────┬──────────────┐
   ▼              ▼              ▼              ▼
🔌 플러그인    🪝 훅         🔗 MCP서버    📄 룰파일
```

### 배포 티어
| 티어 | 가능한 것 | 호스트 |
|---|---|---|
| **플러그인 티어 (Full)** | 룰 자동 주입 + 슬래시 명령 + 모드 전환 + 서브에이전트 전파 + 스테이터스라인 | Claude Code, Codex, Grok, Devin, OpenCode, pi, Hermes, Qoder, Copilot CLI |
| **스킬 티어** | 룰 + 스킬/슬래시 명령 | Gemini, Swival, OpenClaw, Antigravity |
| **인스트럭션 티어** | 룰만 항상 적용 | Cursor 룰, Windsurf, Cline, Copilot Chat, Kiro, Zed, Amp, Jules, Junie, CodeWhale |
| **MCP 티어** (보조) | 프롬프트/툴로 룰 제공 | MCP 호스트 |

### MCP에 대한 솔직한 한계 (`ponytail-mcp/README.md`가 직접 인정)
> *"It is **not a replacement** for the always-on adapters. MCP prompts are
> user-invoked, and there is **no portable MCP primitive for 'inject this into
> every turn'** across hosts."*

→ MCP는 **프롬프트 메뉴밖에 없는 호스트를 위한 차선책**이다 (이슈 #70).

제공하는 것 2개:
- Prompt `ponytail` — 룰셋을 유저 메시지로 반환 (`mode` 인자 선택)
- Tool `ponytail_instructions` — 같은 텍스트 + `structuredContent`, read-only

```bash
cd ponytail-mcp && npm install && node index.js
```
```json
{ "mcpServers": { "ponytail": { "command": "node", "args": ["ponytail-mcp/index.js"] } } }
```

**결론**: Claude Code 사용자는 플러그인으로 설치하면 스킬+훅까지 전부 따라오므로
MCP는 신경 쓸 필요 없다.

---

## 9. API 토큰 필요한가? → 아니오

Ponytail은 **텍스트 규칙 모음집**이고, 스스로 AI를 호출하지 않는다.

하는 일 전부:
1. `SKILL.md` 파일 읽기 (`fs.readFileSync`)
2. 현재 레벨에 맞는 줄만 필터링
3. stdout으로 출력 → 호스트가 컨텍스트에 주입
4. 모드 플래그 파일 쓰기

훅/스킬 코드에 `fetch`, `axios`, `http` 요청이 **하나도 없다.**

### 오히려 비용을 아껴준다
토큰 -22%, 비용 -20%, 시간 -27%. 룰 주입으로 입력 토큰이 약간 늘지만
AI가 생성하는 출력 코드가 훨씬 많이 줄어서 순이익이다.

### 유일한 예외: 벤치마크 재현
```bash
cp .env.example .env    # ANTHROPIC_API_KEY 입력
npx promptfoo eval -c benchmarks/promptfooconfig.yaml
```
`promptfoo`가 모델을 직접 호출해 비교 실험하기 때문. **Ponytail을 쓰는 데는 무관.**
(GPT/Gemini 설정도 있고, `benchmarks/benchmark-local.py`로 로컬 llama3.2 결과도 있음)

### 보안 체크
| 항목 | 결과 |
|---|---|
| 외부 네트워크 통신 | 없음 |
| 텔레메트리/추적 | 없음 |
| 사용자 코드 외부 전송 | 없음 |
| 파일 쓰기 범위 | `~/.claude/.ponytail-active`, `~/.config/ponytail/config.json`, (동의 시) settings.json의 statusLine |
| 쉘 명령 삽입 방어 | `isShellSafe()` 화이트리스트 검증 + 수동 안내 폴백 |
| 의존성 | 메인 패키지 **외부 의존성 0개** (MCP만 `@modelcontextprotocol/sdk`, `zod`) |
| 라이선스 | MIT |

---

## 10. AI 에이전트 구축에 도움이 될까? → 매우

### 방향 A: 에이전트 설계 교본 (★★★★★)

에이전트/스킬을 만들 때 부딪히는 문제의 답이 전부 들어있다.

| 문제 | Ponytail의 해답 |
|---|---|
| AI가 지시를 잊어버림(drift) | ① `SKILL.md`에 "ACTIVE EVERY RESPONSE" 명시 ② `UserPromptSubmit` 훅으로 매 턴 재주입 ③ `SubagentStart`로 하위 에이전트 전파 |
| 어느 플랫폼에 배포? | 전부. 대신 **내용은 한 곳에만** + 어댑터는 얇게 + 동기화 검사 자동화 |
| 내 프롬프트가 효과 있나? | promptfoo A/B 테스트 + 에이전틱 벤치 + 안전성 감사 + correctness 게이트 |
| 모드/설정 관리 | 환경변수 > 설정파일 > 기본값 / XDG·APPDATA 분기 / BOM 제거 / try-catch 폴백 |
| 스킬 description 작성법 | 무엇 + 언제 + 트리거 키워드 + **"Do NOT use for..."** (오발동 방지 핵심 테크닉) |
| AI가 듣기 좋은 거짓말을 함 | `Honesty boundary` — "할 수 없는 말"을 명시적으로 금지 |
| 안전성 보장 | `When NOT to be lazy` 섹션 + 적대적 안전성 티어 벤치마크로 **"효율 최적화가 안전을 깎을 수 있다"를 측정으로 증명** |

### 방향 B: 에이전트에 탑재하는 부품 (★★★★)
```python
system_prompt = f"""
{open('AGENTS.md').read()}      # Ponytail 룰 통째로

추가 규칙:
- 우리 회사는 TypeScript만 씀
"""
```
→ 토큰 22% 덜 쓰는 효율적 코드 생성기가 된다. `AGENTS.md`가 1장이라 컨텍스트 부담도 적다.

#### 멀티 에이전트 파이프라인 구성도 가능
```
[Builder] ponytail → [Reviewer] ponytail-review
        → [Auditor] ponytail-audit → [Debt] ponytail-debt
```

### 읽어볼 파일 추천 순서
1. `AGENTS.md` — 룰셋 압축의 정석 (1장)
2. `skills/ponytail/SKILL.md` — 스킬 작성 완전체
3. `hooks/ponytail-activate.js` — 훅으로 컨텍스트 주입하는 실전 코드
4. `hooks/ponytail-instructions.js` — 모드별 룰 필터링 로직
5. `hooks/ponytail-config.js` — 설정 우선순위 + 크로스플랫폼
6. `docs/agent-portability.md` — 멀티 플랫폼 배포 전략
7. `benchmarks/results/2026-06-18-agentic.md` — 에이전트 평가 방법론
8. `ponytail-mcp/index.js` — 최소 MCP 서버 구현 (약 40줄)
9. `benchmarks/robustness-audit.js` — 안전성 회귀 테스트
10. `pi-extension/`, `plugin.yaml` + `__init__.py` — JS/Python 양쪽 플러그인 예시

### 한계
- 에이전트 **프레임워크는 아니다**. 오케스트레이션/메모리/툴 라우팅을 제공하지 않는 **행동 규범 레이어**다.
- 추론 모델(GPT-5.5 등)에선 사다리 칸을 "고민"하는 데 thinking 토큰을 써서 역효과 가능 (README 인정).
- 이미 미니멀한 코드엔 효과 거의 0. 과설계 함정이 있을 때 극적.
- "게으름"의 기준이 주관적 → 팀 상황에 맞게 포크해서 튜닝하는 게 좋다.

---

## 11. React / PHP로 만들 수 있나?

### 해석 A: Ponytail 자체를 React/PHP로 재구현
**의미 없다.** 구성의 약 85%가 마크다운 텍스트이고, Node.js 글루 코드는 100줄 수준이다.
- React로? UI가 없다 (터미널 훅이다)
- PHP로? 가능하지만 CLI 에이전트 훅은 Node가 어디나 깔려 있어 Node가 정답

> 참고: "다시 만들 필요가 있나?"가 바로 사다리 1번 칸이다. `AGENTS.md` 마지막 줄:
> *"Yes, this file also applies to agents working on the ponytail repo itself.
> Especially to them."*

### 해석 B: React/PHP 프로젝트에 Ponytail 적용
**완벽하게 가능하고, React에서 효과가 가장 극적이다.**

#### React (★★★★★)
벤치마크 자체가 React 포함 레포에서 측정됐고, 가장 극적인 사례가 전부 프론트엔드다.

| 사례 | Before | After | 감소 |
|---|--:|--:|--:|
| 날짜 선택기 | 404줄 | 23줄 | **-94%** |
| 컬러 선택기 | 287줄 | 23줄 | **-92%** |
| 카운트다운 타이머 | 267줄 | 9줄 | **-97%** |

| 과설계 | 대안 |
|---|---|
| `useMemo`/`useCallback` 남발 | 측정 후에 넣기 |
| 상태관리 라이브러리 즉시 도입 | `useState` → `useReducer` → Context 순 |
| 컴포넌트 과분할 | "Fewest files possible" |
| `styled-components` + 테마 시스템 | CSS / CSS Modules |
| `react-datepicker`, `moment.js` | `<input type="date">`, `Intl.DateTimeFormat` |
| `react-modal` | `<dialog>` 네이티브 |
| 무한스크롤 라이브러리 | `IntersectionObserver` |
| `axios` + 인터셉터 레이어 | `fetch` |
| 폼 라이브러리 즉시 도입 | HTML5 `required`, `pattern`, `type="email"` |

`examples/`에 `modal-dialog.md`, `infinite-scroll.md`, `react-countdown.md`,
`url-params.md`, `number-formatting.md` 등 프론트엔드 예시가 가장 많다.

#### PHP / Laravel (★★★★)
```php
// 과설계: Interface(구현체 1개) + Repository + Factory + Service + DTO + Mapper
// → 6개 파일, 150줄, 모델 1개 조회

// Ponytail
User::find($id);
// ponytail: Eloquent 직접 사용. 두 번째 데이터소스 생기면 리포지토리 추출.
```

| 과설계 | 대안 |
|---|---|
| 구현체 1개 Interface + Repository | Eloquent/PDO 직접 (`yagni`) |
| Factory 패턴 (제품 1개) | `new` 직접 |
| 손으로 짠 이메일 정규식 | `filter_var($e, FILTER_VALIDATE_EMAIL)` (`stdlib`) |
| 배열 헬퍼 직접 구현 | `array_column`, `array_map`, `array_filter` |
| 커스텀 캐시 클래스 | `Cache::remember()` |
| 애플리케이션 레벨 유니크 체크 | **DB UNIQUE 제약** (`native`) |
| Composer 패키지 남발 | `json_encode`, `password_hash`, `hash_hmac` |

PHP에서 추가로 `AGENTS.md`에 넣을 줄:
```markdown
Never lazy about: prepared statements (PDO bindParam), output escaping
(htmlspecialchars / Blade {{ }}), CSRF tokens, password_hash.
```

#### 팀 적용 팁
```bash
cp ponytail/AGENTS.md ./AGENTS.md
cp ponytail/.cursor/rules/ponytail.mdc ./.cursor/rules/   # Cursor 사용 시
git add AGENTS.md && git commit -m "chore: add ponytail rules"
```
레포에 커밋해두면 **팀원 전체의 AI가 같은 규칙**을 따른다.
룰 파일을 여러 개 쓰면 `node scripts/check-rule-copies.js`로 동기화를 검사한다.

---

## 12. 유튜브 강의 영상 제작 가능?

**가능하고, 소재가 매우 좋다.** MIT 라이선스로 영상/교육/수익화 모두 자유
(설명란에 원본 레포 링크 + 원작자 크레딧은 매너).

### 소재가 좋은 이유
1. **숫자가 있다** — "404줄 → 23줄"은 썸네일 하나로 설명된다
2. **캐릭터가 있다** — 포니테일 아저씨, 스토리텔링 쉬움
3. **즉시 재현 가능** — 시청자가 2줄로 따라함 → 이탈 적음
4. **한국어 콘텐츠가 거의 없다** — 선점 기회
5. **트렌드 정중앙** — AI 코딩 / 바이브 코딩 후처리 / 토큰 비용 절감

### 6편 시리즈 기획안
| 편 | 제목 | 길이 | 목표 |
|---|---|---|---|
| EP1 | AI가 404줄 짠 걸 23줄로 줄인 플러그인 | 8~10분 | 조회수 + 구독 |
| EP2 | 사다리 7칸: 시니어는 이렇게 생각한다 | 12~15분 | 체류시간 + 전문성 |
| EP3 | React 프로젝트에 적용했더니 (번들 사이즈 비교) | 15~20분 | 프론트엔드 타겟 |
| EP4 | AI 에이전트 만드는 법: 이 레포가 교과서다 | 20~25분 | 고급 + 유료 전환 |
| EP5 | 내 코드베이스 과설계 감사해보기 | 10~12분 | 댓글 참여 |
| EP6 | 나만의 코딩 룰 플러그인 만들기 | 25~30분 | 강의/전자책 연결 |

**EP1 구성**
```
0:00  훅 — 화면 반 갈라 동시 실행. "날짜 선택기 만들어줘"
      좌: AI가 404줄 쏟아냄 / 우: <input type="date"> 한 줄
      "둘 다 같은 Claude입니다. 차이는 플러그인 하나."
0:40  인트로 + 포니테일 아저씨 캐릭터 소개
2:00  벤치마크 수치 + "과장 아니고, 전에 더 크게 발표했다가 정정한 프로젝트"
3:30  라이브 설치 (2줄)
5:00  라이브 데모 3개 (이메일검증 / debounce / 모달)
8:00  ultra 모드 깜짝쇼 (AI가 요구사항에 토 다는 장면)
9:00  정리 + 다음화 예고
```
썸네일: 좌 `404줄` / 우 `23줄` + 큰 글씨 **-94%**

### 제작 팁
**해야 할 것**
- 좌우 분할 화면 비교 (이 콘텐츠의 생명)
- 줄 수 카운터를 화면에 실시간 표시
- 번들 사이즈 / `node_modules` 용량 Before-After (프론트 개발자 반응 큼)
- ultra 모드 개그 활용 (짤 소재)
- 실제 터미널 녹화 (asciinema 등, 복붙 가능하게)

**주의할 것**
- **"무조건 짧은 게 좋다"로 오해 유발 금지** (가장 중요) — 안전벨트는 못 뺀다
- -94%를 평균처럼 말하지 말기 (평균은 -54%)
- GPT-5.5 역효과 언급 (솔직하면 신뢰 ↑)
- 원작자 크레딧 + 레포 링크
- "공식"이라는 표현 피하기 (상표/오인 리스크)

### 플랫폼별 전망
| 플랫폼 | 전망 |
|---|---|
| 유튜브 한국어 | ★★★★★ 경쟁 거의 없음, 선점 가능 |
| 유튜브 영어 | ★★★ 차별화 필요 |
| 인프런/클래스101 | ★★★★★ "AI 코딩 효율화" 패키징 최적 |
| 쇼츠/릴스 | ★★★★ "404→23줄" 15초 클립 바이럴 가능 |
| 블로그(벨로그/티스토리) | ★★★★ SEO 선점 + 영상 유입 |

**전략**: 쇼츠 후킹 → EP1 롱폼 유입 → EP2~5 신뢰 → EP6 유료 전환 → 기업 교육 문의

---

## 13. 수익화 아이디어

### 라이선스 체크
MIT이므로 상업적 이용 / 수정 / 재배포 / 유료 판매 모두 **가능**.
조건은 저작권 표시 + 라이선스 사본 포함.

단, 매너와 리스크 관리:
- "원작자 Dietrich Gebert" 크레딧 + 원본 레포 링크
- "Ponytail 공식"처럼 오해할 이름 피하기 (상표 리스크)
- `.github/FUNDING.yml`(후원)과 `ponytail.dev/soon`(대기명단)이 있어 **원작자의
  상업화 계획 가능성**이 있음 → 정면 경쟁보다 **보완 / 현지화 / 교육**이 안전

### 아이디어 요약
| # | 아이디어 | 난이도 | 예상 수익 | 추천도 |
|---|---|:-:|---|:-:|
| 1 | 한국어 콘텐츠 선점 (유튜브/애드센스) | ★★ | 월 30~300만원 | ★★★★★ |
| 2 | 한국어 생태계 구축 → 영향력 자산화 | ★★ | 간접 (전환율 ↑↑) | ★★★★★ |
| 3 | 전자책 / 유료 뉴스레터 | ★★★ | 150~1,500만원 | ★★★ |
| 4 | 온라인 강의 (인프런/클래스101) | ★★★ | 1,650만~3억원 | ★★★★★ |
| 5 | **기업 교육 / 사내 워크숍** | ★★★ | **회당 100~500만원** | ★★★★★ |
| 6 | 과설계 리뷰 SaaS (GitHub App) | ★★★★ | 월 100만~5,000만원 | ★★★★ |
| 7 | AI 코딩 비용 최적화 대시보드 | ★★★★★ | 월 300만~1.5억원 | ★★★★ |
| 8 | 에이전트 룰셋 마켓플레이스 | ★★★★★ | 플랫폼 성장 비례 | ★★★ |

### #1 한국어 콘텐츠 선점
수익원: 애드센스 + 스폰서 + AI 툴 어필리에이트. 초기 비용 0원.
```
Week 1  : 쇼츠 1개 ("404줄 vs 23줄" 15초) → 반응 테스트
Week 2-3: EP1 롱폼
Month 2 : EP2~3 + 블로그 (SEO)
Month 3 : 시리즈 완성 + 애드센스
Month 4+: 스폰서 문의 (AI 툴 회사들이 좋아할 주제)
```
팁: "AI 코딩 비용 절감"을 강조 → 기업 담당자 검색 키워드 → B2B 문의로 연결.

### #2 한국어 생태계 구축
> ⚠️ `README.ko.md`가 **이미 레포에 있다.** 단순 번역은 가치가 없으므로
> **한국 스택 특화 + 실전 예제**로 차별화해야 한다.

```
"Ponytail Korea" 패키지
├── 한국어 룰셋 (AGENTS.ko.md)
├── 한국 스택 특화 룰 (React+Next.js / Spring Boot / Laravel / 네이버·카카오 API)
├── 설치 가이드 (스크린샷 풀버전)
├── 한국 개발자 실전 예제 20개
└── 원작자 크레딧 + 원본 링크
```
직접 수익은 없지만 **"AI 코딩 효율화 전문가" 포지션**을 만들어 TIER 2를 열어준다.

### #3 전자책
```
"게으른 시니어처럼 코딩하기 — AI 시대, 안 쓴 코드가 최고의 코드다"

Part 1. 왜 AI는 과하게 만드는가 (AI 슬롭, 코드는 부채, 404줄의 비극)
Part 2. 사다리 7칸 — 시니어의 사고법 (칸마다 실전 예제 5개)
Part 3. 게으르면 안 되는 것들 (안전벨트, 증상 vs 근본원인, 보안/검증/접근성)
Part 4. 스택별 적용 (React/Next.js, PHP/Laravel, Python/Django, Spring Boot)
Part 5. AI 에이전트에 룰 심기 (스킬/훅/MCP, 효과 측정, 팀 룰셋)  ← 차별화
```
Part 5가 "AI 잘 쓰는 법"을 넘어 "AI를 설계하는 법"으로 가서 가격을 올려준다.

### #4 온라인 강의 (8~10시간)
```
섹션 1. AI 코딩의 현실 (40분) — 슬롭 진단, 토큰 비용 계산, 과설계 사례
섹션 2. 사다리 7칸 완전정복 (2시간) — 칸별 실습 + 리팩토링 10개
섹션 3. 설치 & 운영 (1시간) — 호스트별, 팀 적용, 모드 운영
섹션 4. 스택별 실전 (2.5시간) — React 번들 50% 감소, Laravel Repository 탈출
섹션 5. 안전하게 게으르기 (1시간) — 절대 못 깎는 것, 체크리스트
섹션 6. 나만의 AI 룰셋 만들기 (2시간) ★ — SKILL.md, 훅, promptfoo, 배포
부록: 룰셋 템플릿 / 체크리스트 PDF / 과설계 패턴 사전 50개
```
인프런은 실무 적용 가능성을 중시 → 섹션 4를 두껍게. 섹션 6이 가격을 올려준다.

### #5 기업 교육 — **가장 추천**
기업 입장의 ROI 계산:
```
개발자 50명, 1인당 월 AI 코딩 비용 10만원 → 연 6,000만원
20% 절감 → 연 1,200만원 절감
교육비 300만원 → 첫해 4배 ROI (+ 유지보수 비용 절감은 별도)
```
**교육비가 즉시 회수되는 보기 드문 교육 주제** → 영업이 쉽다.

1일 커리큘럼:
```
09:00 진단 세션 — 우리 회사 AI 코딩 비용 실측 + 최근 PR 5개에서 과설계 찾기
10:30 사다리 7칸 워크숍 — 팀별 리팩토링 배틀
13:00 설치 & 세팅 — 전원 설치 + 우리 스택 룰 추가
15:00 커스텀 룰셋 제작 ★ — 팀 규칙을 AGENTS.md로 만들어 레포에 커밋
16:30 안전 교육 — 절대 깎으면 안 되는 것, 보안/검증 체크리스트
17:30 측정 체계 구축 — 도입 전후 비교, 월간 리포트 템플릿
```
애프터케어 추가 매출: 1개월 팔로업 +50만원 / 분기 효과 리포트 월 30만원 /
룰셋 유지보수 계약 월 50~100만원.

영업 한 줄:
> *"AI 코딩 비용 20% 줄여드립니다. 교육비는 첫 달에 회수됩니다."*

진단 세션에서 **그 회사 실제 PR의 과설계를 찾아내 보여주는 것**이 킬러.

### #6 과설계 리뷰 SaaS
```
GitHub App (PR 웹훅)
   → 백엔드 (Node.js or PHP/Laravel): diff → ponytail-review 룰 + LLM → 파싱 → PR 인라인 코멘트
   → 프론트엔드 대시보드 (React): 과설계 점수 추이 / 월간 리포트 / 팀 랭킹 / 부채 장부 / 룰셋 커스터마이징
```
PR 코멘트 예시:
```markdown
## Ponytail Review
**L12-38** `stdlib:` 27줄 이메일 validator 클래스
→ filter_var($e, FILTER_VALIDATE_EMAIL) 1줄.
**L4** `native:` moment.js를 포맷 1번 호출에 import → Intl.DateTimeFormat, 0 deps.
**repo.php:L88** `yagni:` 구현체 1개인 AbstractRepository → 인라인.
net: -94 lines possible
```

기존 도구와의 차별화:
| 기존 (SonarQube, CodeClimate) | 이 제품 |
|---|---|
| 버그/보안/복잡도 측정 | **과설계만 전문** |
| "복잡도 15입니다" | **"이 줄을 이걸로 바꿔라"** |
| 정적 분석 룰 기반 | LLM 기반 (문맥 이해) |
| 추가하라고 말함 | **삭제하라고 말함** |

리스크 대응: LLM 비용 → 캐싱 + diff 크기 제한 + 티어 쿼터 /
원작자 상업화 → 한국 시장·스택 특화 또는 독자 브랜드 /
무료 대안 → 대시보드·추이·리포트·팀 관리가 유료 가치.

> 제품 이름에 "Ponytail"을 쓰지 말 것. 독자 브랜드 + "ponytail 룰셋 기반 (MIT)" 크레딧.

### #7 AI 코딩 비용 최적화 대시보드
*"우리 회사 AI 코딩 비용, 얼마 쓰고 얼마 아꼈나?"* — CTO/EM이 경영진에 AI 코딩
ROI를 보고해야 하는 수요가 늘고 있으나 그 숫자를 만들어주는 제품이 없다.

> ⚠️ **정직성 주의** (Honesty boundary 원칙):
> - 보여줘도 되는 것: 실제 토큰/비용 추이(API 로그), 실제 생성 코드량(git diff),
>   제거된 의존성/번들 사이즈(실측), `ponytail:` 부채 장부 카운트(실집계)
> - 금지: "안 썼으면 몇 줄이었을까" 추정치

### #8 룰셋 마켓플레이스
"AI 에이전트 성격/룰셋의 앱스토어" — Lazy Senior(무료), Security Paranoid,
Performance Hawk, A11y Guardian, TDD Enforcer, K-Startup Stack, 페르소나 스킬 등.
페르소나 스킬 수요는 실재하지만 **플랫폼 사업이라 난이도 최상** → TIER 1~2로
자본/영향력을 쌓은 뒤 도전.

### 추천 로드맵
```
Month 1-2  씨앗 심기 (0원)   : 쇼츠 1개 → EP1 → 한국 스택 룰셋 공개 → 블로그 3편
Month 3-5  신뢰 쌓기         : EP2~6 시리즈 → 커뮤니티 → 전자책 집필
Month 6-8  1차 수익화        : 인프런 강의 → 전자책 출간 → 기업 교육 영업  ← 핵심
Month 9-12 B2B 확장          : 교육 반복 판매 → 유지보수 구독 → SaaS MVP
Year 2     제품화            : 리뷰 SaaS 출시 → 비용 대시보드
```
순서 이유: 콘텐츠는 자본 0원·리스크 0 → 강의 재료로 재활용 → 영향력 없으면
기업이 안 산다 → SaaS는 앞 단계에서 고객 니즈를 이미 파악한 뒤에.

### 단 하나만 고른다면: **기업 교육 + 커스텀 룰셋 컨설팅**
- 단가 최고 (회당 100~500만원)
- ROI를 숫자로 증명 가능 → 영업이 쉽다
- 커리큘럼 한 번 만들어 반복 판매
- 유지보수 계약으로 구독 수익 전환
- 한국에 경쟁자가 거의 없다
- 개발 리스크 0 (SaaS처럼 몇 달 개발이 필요 없다)

### 리스크 관리 체크리스트
| 항목 | 반드시 |
|---|---|
| MIT 라이선스 | 저작권 표시 + 라이선스 사본 포함 |
| 원작자 크레딧 | "Dietrich Gebert의 ponytail 기반" + 레포 링크 |
| 상표 주의 | "Ponytail 공식" 금지, 제품은 독자 브랜드 |
| 숫자 정직하게 | -54%가 평균, -94%는 최대치 |
| 안전 메시지 필수 | "무조건 짧게"로 오해 유발 금지 |
| 원작자 동향 체크 | `ponytail.dev/soon` 대기명단 → 상업화 가능성 |
| 협업 가능성 | 원작자에게 한국 파트너십을 제안해보는 것도 방법 |

---

## 14. 요약 치트시트

| 질문 | 답 |
|---|---|
| 뭐하는 거야? | AI에게 "게으른 시니어" 룰을 주입해 과설계를 막는 규칙 모음집 |
| 핵심은? | 사다리 7칸 — 걸리는 첫 칸에서 멈춤 |
| 효과? | 코드 -54%, 토큰 -22%, 비용 -20%, 시간 -27%, 안전성 100% |
| 플러그인/스킬/MCP? | 전부. 본질은 스킬, Claude Code는 플러그인으로 설치, MCP는 보조 |
| API 키 필요? | **아니오.** 벤치마크 재현할 때만 |
| 설치? | `/plugin marketplace add` + `/plugin install` (별도 메시지 2개) |
| 가장 간단한 설치? | `AGENTS.md` 한 장을 프로젝트 루트에 복사 |
| 끄는 법? | `"stop ponytail"` / `"normal mode"` / `/ponytail off` |
| React에 쓸 수 있어? | **최적.** 효과 가장 극적 (-94%) |
| PHP에 쓸 수 있어? | 좋음. Repository/Factory 과설계 잡기에 유효 |
| 에이전트 구축에 도움? | 프레임워크는 아니지만 **설계 교본으로 최상급** |
| 유튜브 가능? | 가능하고 소재가 좋다. 한국어 선점 기회 |
| 돈 되는 건? | **기업 교육 + 커스텀 룰셋 컨설팅** |
| 시작하기 쉬운 건? | 쇼츠 1개 ("404줄 vs 23줄") |

---

> *"The shortest path to done is the right path."* — Ponytail
>
> 원본: https://github.com/DietrichGebert/ponytail
> 이 포크: https://github.com/bmshin94/ponytail
