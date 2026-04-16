# Hermes Agent 설치 마법사

Claude가 이 파일을 읽으면 아래 단계를 순서대로 실행한다.
사용자에게 먼저 이 배너를 출력하고 시작한다.

## 치명적 원칙

- API 키가 포함된 명령은 실행 전 **맨 앞에 공백 한 칸**을 넣는다 (`HISTCONTROL=ignorespace` 가 기본이면 bash history에 안 남음).
- 키는 화면에 그대로 출력하지 말 것.
- **Anthropic Claude 는 OAuth 금지.** Hermes 의 "OAuth (Claude Code)" 경로 선택 X. API 키만 사용 (credential 공유 시 "out of extra usage" 에러 — GitHub #6475). Claude 는 유료 API.
- **Claude / Gemini 는 유료 API.** 토큰당 과금. 카드 등록 필수.
- **이 파일 안에 적힌 모델명(`deepseek-*`, `gpt-*`, `claude-*`, `glm-*` 등)을 그대로 쓰지 말 것.** STEP 3 에 모델 선정 절차가 있음. 반드시 `WebSearch` 로 오늘 날짜 기준 최신 모델 조회 후 사용. 모델은 월 단위로 deprecated 된다.

### 🔑 키 네이밍 — 공식 `hermes_cli/setup.py` 기준

두 종류가 섞이면 작동 안 한다. 정확히 구분:

| 종류 | 명령 포맷 | 저장 위치 | 예시 |
|------|----------|----------|------|
| **API 키 / 토큰 (env var)** | `hermes config set <UPPER_SNAKE_CASE> <값>` | `~/.hermes/.env` | `hermes config set OPENROUTER_API_KEY sk-or-v1-...` |
| **설정 값 (YAML path)** | `hermes config set <lower.dot.path> <값>` | `~/.hermes/config.yaml` | `hermes config set model.provider openrouter` |

**핵심:**
- `PROVIDER`, `MODEL` 같은 대문자 설정 키는 **env var 목록에 없음 → 동작 안 함.**
- 공식 자동화 가이드가 직접 보여주는 형태:
  ```bash
  hermes config set model.provider custom
  hermes config set model.base_url http://localhost:8080/v1
  hermes config set model.default your-model-name
  ```
- `hermes config set` 은 키 이름 패턴으로 자동 분류 (대문자 snake → .env, 점 표기 → YAML).

### Provider 값 (공식 `cli-config.yaml.example` 기준)

`model.provider` 에 들어갈 수 있는 값:

| 값 | 설명 | 필요한 크레덴셜 |
|---|------|----------------|
| `auto` | 있는 키로 자동 선택 (기본값) | — |
| `openrouter` | OpenRouter (다수 모델 프록시) | `OPENROUTER_API_KEY` 또는 `OPENAI_API_KEY` |
| `anthropic` | Anthropic 직결 | `ANTHROPIC_API_KEY` (OAuth 금지) |
| `openai-codex` | ChatGPT 계정 OAuth | `hermes auth add openai-codex` |
| `gemini` | Google AI Studio 직결 | `GOOGLE_API_KEY` 또는 `GEMINI_API_KEY` |
| `zai` | z.ai / ZhipuAI GLM | `GLM_API_KEY` |
| `nous` | Nous Portal OAuth | `hermes login` |
| `copilot` | GitHub Copilot | `GITHUB_TOKEN` |
| `kimi-coding` | Kimi / Moonshot | `KIMI_API_KEY` |
| `minimax` | MiniMax 글로벌 | `MINIMAX_API_KEY` |
| `custom` | 로컬 LMStudio/Ollama 등 OpenAI-compat | `model.base_url` 설정 |

> **`openai` 는 공식 목록에 없음.** OpenAI 모델 쓰려면 OpenRouter 경유 (provider: `openrouter` + `OPENAI_API_KEY`) 또는 ChatGPT OAuth (provider: `openai-codex`).

### 모델 지정 포맷 — 슬래시 (`/`)

공식 `.env.example` 예시: `anthropic/claude-opus-4.6`. 콜론(`:`) 은 `/model` 슬래시 커맨드용이고, **config 저장·env 에선 슬래시**.

### 🎯 2단 실행 전략 (A+B 하이브리드)

각 STEP 의 설정 작업은 **항상 두 경로 준비**:

1. **주 경로 (자동)** — Claude 가 `hermes config set` 으로 직접 값 주입. 빠르고 한국어 가이드 완비.
2. **폴백 (대화형)** — 주 경로 실패 / 사용자가 선호 시 `hermes setup [section]` 실행. Claude 는 화면 출력을 읽고 각 prompt 를 한국어로 해설하며 다음 행동 안내.

Claude 는 기본적으로 주 경로 시도 → 실패 감지 시 자동으로 폴백으로 전환.

## 브라우저 오픈 규칙

URL 을 열 때 아래 **한 줄 패턴**을 사용한다 (`[URL]` 만 치환):

```bash
(cmd.exe /c start "" "[URL]" 2>/dev/null || open "[URL]" 2>/dev/null || xdg-open "[URL]" 2>/dev/null) || echo "⚠️  브라우저가 자동으로 안 열립니다. 아래 주소를 복사해서 브라우저에 직접 붙여넣어 주세요:\n\n    [URL]\n"
```

순서 이유 (2026 실측 기준):
- `cmd.exe /c start "" "URL"`: WSL2 에서 `explorer.exe` 또는 `powershell.exe` 보다 안정적. `start` 의 첫 인자 `""` 는 창 제목 자리(빈 문자열).
- `open`: macOS 네이티브.
- `xdg-open`: Linux 데스크탑 (GNOME/KDE 등).
- 전부 실패 → URL 을 깔끔하게 출력해서 사용자가 수동 복사.

`2>/dev/null` 로 stderr 억제 (WSL 아닌 Mac 에서 `cmd.exe` 못 찾음 에러 안 보이게).

---

## 시작 배너 (출력)

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║    █  █  █▀▀  █▀█  █▀▄▀█  █▀▀  █▀▀                     ║
║    █▀▀█  █▀▀  █▀▄  █ ▀ █  █▀▀  ▀▀█                     ║
║    ▀  ▀  ▀▀▀  ▀ ▀  ▀   ▀  ▀▀▀  ▀▀▀                     ║
║                                                          ║
║           A G E N T   설치 마법사                         ║
║                                                          ║
║    Claude가 처음부터 끝까지 설치해드립니다.               ║
║    아무것도 몰라도 됩니다. 선택만 하세요.                 ║
║                                                          ║
║                           made by hybirdss              ║
╚══════════════════════════════════════════════════════════╝
```

---

## STEP 0 — 환경 조사 (가장 먼저, 반드시 실행)

사용자에게 묻기 전에 환경을 **자동으로** 수집한다. 사용자의 답변은 틀릴 수 있지만, 아래 명령 출력은 틀리지 않는다.

한 번에 실행 (출력 변수에 수집):
```bash
{
  echo "=== OS ==="
  uname -s -m -r
  [ -f /etc/os-release ] && cat /etc/os-release | head -3
  [ -f /proc/version ] && grep -qi microsoft /proc/version && echo "IS_WSL=1" || echo "IS_WSL=0"

  echo ""
  echo "=== Shell ==="
  echo "SHELL=$SHELL"
  echo "TERM=$TERM"
  [ -n "$TERM_PROGRAM" ] && echo "TERM_PROGRAM=$TERM_PROGRAM"

  echo ""
  echo "=== 필수 도구 ==="
  for cmd in curl git python3 node npm uv hermes; do
    if command -v "$cmd" >/dev/null 2>&1; then
      v=$("$cmd" --version 2>&1 | head -1)
      echo "$cmd: $v"
    else
      echo "$cmd: (없음)"
    fi
  done

  echo ""
  echo "=== 기존 설정 ==="
  [ -d ~/.hermes ] && echo "~/.hermes 존재 (기존 설치)" || echo "~/.hermes 없음 (신규 설치)"
  [ -f ~/.codex/auth.json ] && echo "Codex CLI 인증 존재 (OAuth 재사용 가능)" || echo "Codex CLI 인증 없음"
  [ -d ~/.claude ] && echo "Claude Code 존재" || echo "Claude Code 없음"
  [ -d ~/.openclaw ] && echo "⚕ OpenClaw 감지 — 설치 후 'hermes claw migrate' 로 자동 이주 가능" || echo "OpenClaw 없음"

  echo ""
  echo "=== 네트워크 ==="
  curl -s -o /dev/null -w "github.com: %{http_code}\n" --max-time 5 https://raw.githubusercontent.com 2>/dev/null || echo "github.com: unreachable"

  echo ""
  echo "=== 디스크 ==="
  df -h "$HOME" | tail -1 | awk '{print "Home 여유: " $4 " / 전체: " $2}'
} 2>&1
```

**수집 결과 해석 원칙:**

| 감지 결과 | 분기 |
|----------|-----|
| `IS_WSL=1` | STEP 1 OS 선택 자동으로 "Linux / WSL2" 경로 (사용자에게 묻지 않음) |
| `uname -s` = `Darwin` | STEP 1 "Mac" 경로 자동 |
| Windows PowerShell (이 SKILL.md 는 bash 기반이라 STEP 0 이 처음부터 안 돌 수도) | 사용자에게 "PowerShell 에서 이 명령 실행하세요" 안내: `irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1 \| iex` — 설치 후 `hermes` 로 돌아오기 |
| `~/.hermes 존재` | "기존 설치 감지. 새로 설치 / 재설정 / 업데이트?" 묻기 |
| `Codex CLI 인증 존재` | STEP 3 선택 2(ChatGPT OAuth) 에서 자동 import 제안 |
| `hermes` 이미 설치 + 버전 최신 | STEP 2 설치 스킵, STEP 3 로 점프 |
| `curl 없음` | STEP 2 전에 curl 먼저 설치 안내 |
| `uv 없음` | install.sh 가 자동 설치 시도 — 실패 시 `curl -LsSf https://astral.sh/uv/install.sh \| sh` 수동 |
| Python 버전 < 3.10 | `uv python install 3.11 && uv python pin 3.11` |
| `Home 여유 < 2G` | 경고 출력 후 계속 |
| `github.com: 000` 또는 unreachable | 네트워크 경고, 프록시 사용 여부 질문 |

**사용자에게 요약 한 줄로 리포트:**
```
감지: macOS 15.2 (arm64) · zsh · Python 3.11 · Node 24 · hermes 미설치 · 네트워크 OK
```

사용자가 "다르게 해달라" 하기 전까지 감지 결과로 자동 진행.

---

## STEP 1 — OS 확인

**STEP 0 감지 결과로 자동 분기 우선.** 아래 표 참조:

| STEP 0 감지 | 분기 |
|------------|-----|
| `uname -s` = `Darwin` | Mac 경로로 자동 진행 (묻지 않음) |
| `IS_WSL=1` | Linux/WSL2 경로로 자동 진행 |
| Linux (WSL 아님, `DISPLAY` 있음) | Linux 경로 자동 진행 |
| Linux (`DISPLAY` 없음) + SSH 감지 | 서버 환경 — 메신저 연동 단계에서 원격 URL 출력 모드 |
| `~/.openclaw` 존재 | STEP 2 설치 후 `hermes claw migrate` 실행 자동 제안 (세션·스킬·키 이주) |
| 위 어느 것도 명확하지 않을 때만 | AskUserQuestion 으로 직접 질문 |

감지가 확실하면 다음 블록을 사용자에게 출력하고 STEP 2 로 진행:
```
감지된 환경: <OS> · <arch> · <shell>
이 환경에서 설치 가능합니다. 계속 진행합니다.
```

### 감지 불확실 시 (fallback) — AskUserQuestion

```
질문: "어떤 환경에서 설치하시나요?"
선택지:
  - Mac (추천) — 바로 설치 가능
  - Linux / WSL2 — 바로 설치 가능
  - Windows (WSL2 없음) — WSL2 먼저 설치 (5분, 자동)
```

### Windows 선택 시 → 두 경로 지원

**공식 `install.ps1` 존재 → Windows 네이티브 완전 지원.** WSL2 는 선택사항.

AskUserQuestion:
```
질문: "Windows 에서 어느 쪽으로 갈까요?"
선택지:
  - 🪟 네이티브 PowerShell (가장 빠름, Recommended)  — install.ps1 자동 설치
  - 🐧 WSL2 Ubuntu                                    — Linux 도구 선호 시
  - 이미 WSL2 있어요                                  — WSL2 경로로
```

#### 네이티브 PowerShell 선택 시

설치 환경 조건:
- Python/uv/Git 없어도 OK — install.ps1 이 자동 설치 (uv 로 Python 3.11 자동, 관리자 권한 불필요)
- `$env:LOCALAPPDATA\hermes` 에 설치됨

STEP 2 에서 아래 명령 실행 예정:
```powershell
irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1 | iex
```

#### WSL2 선택 시 (또는 이미 있음)

WSL2 설치 안내 (이미 있으면 스킵):
```
┌─────────────────────────────────────────────────────────┐
│  Windows → WSL2 설치 (5분)                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Windows 키 → "PowerShell" 검색                       │
│  2. "관리자 권한으로 실행" 클릭                           │
│  3. 아래 명령어 붙여넣기:                                 │
│                                                         │
│     wsl --install                                       │
│                                                         │
│  4. 컴퓨터 재시작                                        │
│  5. 재시작 후 Ubuntu 창이 자동으로 열림                   │
│  6. 사용자이름/비밀번호 설정                              │
│  7. 그 Ubuntu 터미널 창에서 계속 진행                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "WSL2 Ubuntu 터미널이 열려있나요?"
선택지:
  - 네, Ubuntu 열려있어요 — 다음
  - 아직 설치 중 — 완료되면 알려달라고 안내
  - 이미 WSL2 있어요 — 다음

---

## STEP 2 — 설치

```
┌─────────────────────────────────────────────────────────┐
│  설치 중... (보통 2~5분)                                 │
│                                                         │
│  Python 3.11, Node 22, uv, hermes-agent                 │
│  → 공식 설치 스크립트가 알아서 다 해줍니다.              │
│  (Python 없으면 uv 가 자동 설치 — 관리자 권한 불필요)    │
└─────────────────────────────────────────────────────────┘
```

**중요 — `--skip-setup` 플래그 필수.** 공식 스크립트는 설치 끝에 대화형 `hermes setup` wizard 를 띄운다. 우리가 STEP 3 에서 직접 처리하므로 반드시 스킵.

### Mac / Linux / WSL2

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash -s -- --skip-setup
```

> **절대 `sudo` 로 실행 금지.** install.sh 는 `~/.local/bin` 사용자 영역에 설치. sudo 쓰면 권한 충돌로 실패 (공식 FAQ).

설치 완료 후 shell reload:
```bash
source ~/.bashrc 2>/dev/null || source ~/.zshrc 2>/dev/null || true
```

### Windows 네이티브 PowerShell

```powershell
irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1 | iex
```

install.ps1 도 대화형 setup 을 띄우지만, 이후 새 PowerShell 창 열면 됨 (wizard 중간에 Ctrl+C 로 빠져나와도 설치 자체는 완료).

### 확인 (공통)

```bash
hermes version
```

실패 시 PATH 문제:

| OS | 명령 |
|----|-----|
| Mac/Linux/WSL2 | `export PATH="$HOME/.local/bin:$PATH"` |
| Windows | PowerShell 완전히 닫고 새로 열기 (install.ps1 이 PATH 자동 업데이트함) |

그래도 `hermes version` 실패 시:
- uv 가 PATH 에 없을 수 있음 → `curl -LsSf https://astral.sh/uv/install.sh | sh` (Mac/Linux) 또는 `irm https://astral.sh/uv/install.ps1 | iex` (Windows)
- 터미널 완전히 닫고 다시 열기
- Python 3.10+ 버전 확인 (`python3 --version`) — Hermes 는 3.10+ 지원, 3.11 권장

> **A+B 하이브리드 전략.** Hermes 는 `hermes setup` 공식 wizard 를 제공. 우리 SKILL.md 는 각 STEP 에서 **자동 주 경로 (`hermes config set model.provider ...`) + 대화형 폴백 (`hermes setup [section]`)** 두 가지를 준비. Claude 가 자동 경로 먼저 시도하고 실패 감지 시 대화형으로 전환. 대화형 화면은 Claude 가 한국어로 실시간 해설한다.
> 공식 setup 서브커맨드: `hermes setup [model|tts|terminal|gateway|tools|agent]`.

---

## STEP 3 — AI 모델 연결

### ⚠️  모델 추천 원칙 (이 STEP 전체에 적용)

**하드코딩된 모델명을 믿지 말 것.** 이 파일에 예시로 남은 모델 이름도 월 단위로 교체·deprecated 된다.

사용자가 provider 를 선택하는 순간, Claude 는 아래를 **반드시 `WebSearch` 로 조회**한다:

1. **Hermes 공식 현재 추천** — 쿼리: `"site:hermes-agent.nousresearch.com recommended model {provider명} {오늘의 YYYY-MM}"`
2. **해당 provider 의 최신 주력 모델** — 쿼리: `"{provider명} best model {오늘의 YYYY-MM} tool calling agent"`
3. **deprecation 확인** — 쿼리: `"{후보 모델명} deprecated"` (hit 있으면 제외)

세 결과 교차해서 남는 모델 1개를 사용자에게 제시. 근거 1줄 + 오늘 날짜 표기. 사용자 승인 후 `hermes config set model.default <모델>` 실행.

> 오늘 날짜는 환경 `currentDate` 참조.

---

AskUserQuestion:

```
질문: "AI 모델 어떻게 쓰시겠어요? (1번이 가장 쉬움)"
선택지:
  - 🎯 OpenRouter 무료 (카드 없음, 1분) — 가장 쉬움 (Recommended)
  - 🤖 ChatGPT 계정 연결 (OpenAI Codex OAuth) — ChatGPT Plus/Pro 유저용
  - 🇨🇳 GLM / z.ai — GLM Coding Plan 또는 저렴한 고성능
  - 💳 유료 API 직접 (Claude / Gemini) — 비용 발생
```

---

### 선택 1 — OpenRouter 무료 (카드 없음)

브라우저 오픈 (규칙 패턴 사용): `https://openrouter.ai/keys`

```
┌─────────────────────────────────────────────────────────┐
│  OpenRouter 무료 키 발급 (카드 없음)                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. 우측 상단 "Sign In" 클릭                             │
│     → Google 계정으로 로그인 (가장 쉬움)                 │
│                                                         │
│  2. 로그인 후 우측 상단 프로필 → "Keys"                  │
│                                                         │
│  3. "Create Key" → 이름 아무거나 (예: hermes) → Create  │
│                                                         │
│  4. sk-or-v1-... 로 시작하는 키 복사                     │
│     ⚠️  이 창 닫으면 키 다시 못 봄!                     │
│                                                         │
│  💡 카드 등록 없이 무료 모델 사용 가능                    │
│  💡 무료 한도: 하루 50회 요청, 분당 20회                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "OpenRouter 키를 복사하셨나요?"
선택지:
  - 네, 복사했어요 — 키 입력
  - Google 로그인이 안 돼요 — 안내 재출력
  - 다른 방법 원해요 — STEP 3 처음으로

**모델은 WebSearch 로 조회.** 병렬 2개:
- `"site:hermes-agent.nousresearch.com OpenRouter free model {오늘의 YYYY-MM}"`
- `"OpenRouter best free model {오늘의 YYYY-MM} agent tool calling :free"`

조건:
- 모델 ID 가 `:free` 로 끝남
- 도구 호출 지원
- deprecated 언급 없음
- 컨텍스트 32k+

선정 결과를 `"YYYY-MM-DD 기준 현재 추천: <모델 ID>. 이유: <1줄>"` 으로 제시하고 사용자 승인 받는다.

#### 주 경로 (자동) — 권장

Other 로 받은 키와 WebSearch 로 고른 모델을 한 번에 주입:
```bash
 hermes config set OPENROUTER_API_KEY [입력받은 키]
 hermes config set model.provider openrouter
 hermes config set model.default "openrouter/<선정된 모델>"
```

검증:
```bash
hermes config get model.provider
hermes config get model.default
```

#### 폴백 (대화형) — 주 경로 실패 시 또는 사용자가 원할 때

```bash
hermes setup model
```

대화형 메뉴가 뜸. Claude 는 각 화면을 읽어 한국어로 안내:

| Hermes 질문 (영문) | Claude 의 한국어 안내 | 사용자가 고를 것 |
|-------------------|---------------------|----------------|
| "Choose your provider" | "Provider 를 선택하세요. OpenRouter 무료 경로면 `openrouter` 선택" | `openrouter` |
| "Enter API key" | "조금 전 복사한 sk-or-v1- 키 붙여넣기" | (키 붙여넣기) |
| "Choose your model" | "WebSearch 로 찾은 모델: `<모델명>`. 목록에 있으면 그대로, 없으면 상단 flagship" | (모델명 입력/선택) |

완료 후 `hermes doctor` 로 검증.

---

### 선택 2 — ChatGPT 계정 OAuth (OpenAI Codex)

**제대로 된 OAuth 방법.** API 키 복사·붙여넣기 없이, `hermes` 가 디바이스 코드 플로우로 ChatGPT 계정 직접 연결. 크레덴셜은 `~/.hermes/auth.json` 저장, 토큰 자동 갱신.

```
┌─────────────────────────────────────────────────────────┐
│  ChatGPT 계정 OAuth — API 키 발급 불필요                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ✓ Plus / Pro 구독 그대로 사용                          │
│  ✓ 토큰 자동 갱신 — 한 번 로그인하면 계속 유지           │
│  ✓ Codex CLI (~/.codex/auth.json) 있으면 자동 가져옴     │
│                                                         │
│  ⚠️  ChatGPT Free 플랜은 안 됨                           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

아래 명령 중 하나 실행 (공식적으로 두 경로 모두 지원 — 동일 OAuth 플로우):

```bash
hermes login --provider openai-codex
# 또는
hermes auth add openai-codex
```

두 명령 다 디바이스 코드 생성 + 브라우저 자동 오픈. `hermes login` 이 더 최신·선호 명령이나 차이는 없음.

사용자 안내:
```
1. 터미널에 뜬 URL 이 자동으로 브라우저에 열림
2. 안 열리면 URL 직접 복사해서 브라우저에 붙여넣기
3. 터미널 표시 6자리 코드를 브라우저 페이지에 입력
4. ChatGPT 계정으로 로그인
5. "Authorize" 클릭
6. 터미널에 "Authentication successful" 확인
```

AskUserQuestion: "OAuth 인증 완료됐나요?"
선택지:
  - 네, successful 떴어요 — 모델 선택으로
  - 브라우저가 안 열려요 — 터미널 URL 직접 복사 안내
  - ChatGPT Free 플랜이래요 — 선택 1(OpenRouter) 또는 선택 4(유료 API)
  - 이미 Codex CLI 있어요 — `~/.codex/auth.json` 자동 import (안 되면 `hermes auth add openai-codex` 강제)

**모델 WebSearch** (선정 원칙 적용):
- `"site:hermes-agent.nousresearch.com OpenAI Codex recommended model {오늘의 YYYY-MM}"`
- `"OpenAI Codex latest model {오늘의 YYYY-MM} agent tool calling"`

#### 주 경로 (자동)

```bash
 hermes config set model.provider openai-codex
 hermes config set model.default "openai/<선정 모델>"
```

(모델 prefix 는 Codex 가 호출하는 실제 OpenAI 모델 네임스페이스. WebSearch 로 최신 이름 확인.)

#### 폴백 (대화형)

```bash
hermes setup model
```

메뉴에서 "OpenAI Codex" 선택 → 모델 선택. Claude 는 WebSearch 로 찾은 모델명 먼저 알려주고, 메뉴에 동일 명칭 있으면 그대로, 없으면 가장 근접한 상단 flagship 선택 안내.

---

### 선택 3 — GLM / z.ai

중국 Z.AI (ZhipuAI) GLM 시리즈. Coding Plan 구독 또는 저렴한 고성능 모델 원할 때.

브라우저 오픈 (규칙 패턴 사용): `https://z.ai/manage-apikey/apikey-list`

```
┌─────────────────────────────────────────────────────────┐
│  z.ai API 키 발급                                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. z.ai 계정 로그인 (없으면 회원가입)                    │
│  2. "Create API Key" → 이름 입력                         │
│  3. 생성된 키 복사 (한 번만 표시)                        │
│                                                         │
│  💡 GLM Coding Plan 구독자는 해당 플랜 크레딧 사용        │
│  💡 개별 종량제는 입력/출력 토큰당 과금                   │
│                                                         │
│  ⚠️  글로벌/중국/Coding variants 엔드포인트는 Hermes 가  │
│     자동 탐지 — GLM_BASE_URL 수동 설정 불필요            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "z.ai API 키 복사했나요?"
선택지:
  - 네, 복사했어요 — 키 입력
  - z.ai 가입 안 돼요 — 선택 1(OpenRouter) 로 우회
  - 다른 방법 원해요 — STEP 3 처음으로

**모델 WebSearch** (선정 원칙 적용):
- `"site:hermes-agent.nousresearch.com GLM zai recommended model {오늘의 YYYY-MM}"`
- `"z.ai GLM latest model {오늘의 YYYY-MM} coding agent"`

#### 주 경로 (자동)

```bash
 hermes config set GLM_API_KEY [입력받은 키]
 hermes config set model.provider zai
 hermes config set model.default "zai/<선정 모델>"
```

#### 폴백 (대화형)

```bash
hermes setup model
```
→ "z.ai / ZhipuAI" 선택 → 모델 선택.

---

### 선택 4 — 유료 API 직접 (Anthropic / Gemini)

**⚠️  전부 토큰당 과금.** 무료 원하면 선택 1(OpenRouter) 로.

| Provider | provider 값 | 환경변수 | 최소 | 단가 |
|---------|-------------|----------|-----|-----|
| **Anthropic Claude** | `anthropic` | `ANTHROPIC_API_KEY` | $5 선충전 | `WebSearch "Anthropic API pricing YYYY-MM"` |
| **Google Gemini** | `gemini` | `GOOGLE_API_KEY` 또는 `GEMINI_API_KEY` | pay-as-you-go | `WebSearch "Google Gemini API pricing YYYY-MM"` |

> **OpenAI 직접 API 는 Hermes 가 native 지원 안 함.** 공식 provider 목록에 `openai` 없음. OpenAI 모델 쓰려면:
> - 선택 2 (ChatGPT OAuth, `openai-codex`) — 구독 있으면 이쪽이 유리
> - 또는 선택 1 (OpenRouter, `OPENAI_API_KEY` 를 OpenRouter 인증 키로도 사용 가능)

> **Anthropic 은 Hermes OAuth 경로 절대 사용 금지.** `hermes model` 의 Anthropic 제공자에서 "OAuth (Claude Code)" 옵션 나와도 선택하지 말 것. Hermes ↔ Claude Code credential store 공유 시 "out of extra usage" 에러, 토큰 동기화 불안정 (GitHub #6475). **API 키 경로만.**

가격 단가를 사용자에게 보여준 뒤 진행. `WebSearch "{provider} API pricing {오늘의 YYYY-MM}"` 로 현재 값 확인해서 "오늘 기준: input $X/M, output $Y/M" 한 줄로 제시.

AskUserQuestion: "어느 provider 직접 쓰시겠어요?"
선택지:
  - Anthropic Claude API (유료, 비쌈)
  - Google Gemini API (유료, 중간)
  - 취소 — STEP 3 처음으로

#### Anthropic Claude (API 키 only)

브라우저 오픈 (규칙 패턴): `https://console.anthropic.com/settings/keys`

```
1. 로그인 (없으면 회원가입)
2. Billing 에서 카드 등록 + 최소 $5 선충전
3. "Create Key" → 이름 → 생성
4. sk-ant-... 키 복사 (한 번만 표시)

⚠️  토큰당 과금. 사용 많으면 크레딧 빨리 소진.
⚠️  Hermes OAuth 경로 쓰지 말고 이 API 키만 사용.
```

모델 WebSearch 후 주입:

```bash
 hermes config set ANTHROPIC_API_KEY [입력받은 키]
 hermes config set model.provider anthropic
 hermes config set model.default "anthropic/<선정 모델>"
```

(공식 예시 포맷 `anthropic/claude-opus-4.6` — 실제 최신은 WebSearch 로.)

폴백: `hermes setup model` → "Anthropic" 선택 → **API 키** 입력 경로 (OAuth 금지!) → 모델 선택.

#### Google Gemini (API 키)

브라우저 오픈 (규칙 패턴): `https://aistudio.google.com/app/apikey`

```
1. Google 계정 로그인
2. "Create API key" → 프로젝트 선택 → 생성
3. 키 복사 (AIza... 형태)

⚠️  토큰당 과금. Google Cloud 빌링 계정 연결 필요할 수 있음.
💡 AI Studio 에 free tier 있지만 제한적 — 업무용은 유료 권장.
```

모델 WebSearch 후 주입 (`GOOGLE_API_KEY` 가 공식 primary, `GEMINI_API_KEY` 는 alias — 아무거나):

```bash
 hermes config set GOOGLE_API_KEY [입력받은 키]
 hermes config set model.provider gemini
 hermes config set model.default "gemini/<선정 모델>"
```

폴백: `hermes setup model` → "Gemini" 선택 → API 키 입력 → 모델 선택.

---

## STEP 4 — 메신저 연동 (선택)

```
┌─────────────────────────────────────────────────────────┐
│  📱 메신저에서 Hermes 사용하기                           │
│                                                         │
│  Telegram 이나 Discord 에 연결하면                       │
│  폰으로 어디서든 AI 쓸 수 있음.                          │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion:
```
질문: "메신저 연동을 설정할까요?"
선택지:
  - Telegram 연동 — 봇 만들고 연결
  - Discord 연동 — 봇 만들고 연결
  - 둘 다 — Telegram 먼저, 그 다음 Discord
  - 나중에 — 건너뛰기 → STEP 5
```

### Telegram 연동

브라우저 오픈 (규칙 패턴): `https://t.me/BotFather`

```
┌─────────────────────────────────────────────────────────┐
│  [1/2]  봇 만들기 — BotFather 에서 진행                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  BotFather = Telegram 공식 봇 관리자.                    │
│  파란 체크(✓) 있는 계정이 진짜.                         │
│                                                         │
│  채팅창에 순서대로:                                       │
│                                                         │
│  Step 1. /newbot                                        │
│                                                         │
│  Step 2. 봇 이름 (한글 가능)                             │
│          예시: 내 AI 비서                                │
│                                                         │
│  Step 3. 봇 아이디 (영어만, bot 으로 끝나야 함)          │
│          예시: myhermes_bot                              │
│                                                         │
│  완료되면 토큰 표시:                                      │
│  1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ                  │
│  → 줄 전체 복사                                          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "봇 토큰 복사했나요?"
선택지:
  - 네, 토큰 복사 — 다음
  - bot 으로 끝나는 아이디 중복 — 다른 이름 재시도
  - BotFather 를 못 찾음 — 검색창에 @BotFather, 파란 체크 확인

브라우저 오픈 (규칙 패턴): `https://t.me/userinfobot`

```
┌─────────────────────────────────────────────────────────┐
│  [2/2]  내 Telegram ID 확인                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  @userinfobot 에서 /start                                │
│                                                         │
│  Id: 123456789  ← 이 숫자만 복사                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion 두 번 (Other 로 직접 입력):
1. "BotFather 에서 받은 봇 토큰 붙여넣어 주세요"
2. "@userinfobot 에서 확인한 숫자 ID 를 입력해주세요"

저장:
```bash
 hermes config set TELEGRAM_BOT_TOKEN [봇 토큰]
 hermes config set TELEGRAM_ALLOWED_USERS [Telegram ID]
```

### Discord 연동

브라우저 오픈 (규칙 패턴): `https://discord.com/developers/applications`

```
┌─────────────────────────────────────────────────────────┐
│  [1/3]  봇 앱 만들기                                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Discord 계정 로그인                                  │
│  2. 우상단 "New Application" → 이름 → Create            │
│  3. 왼쪽 메뉴 "Bot"                                      │
│                                                         │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  ⚠️  여기 안 하면 봇이 완전 먹통!                        │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                         │
│  4. 스크롤 → "Privileged Gateway Intents"                │
│     ┌─────────────────────────────────────────┐        │
│     │ SERVER MEMBERS INTENT    [ON ●]          │        │
│     │ MESSAGE CONTENT INTENT   [ON ●]          │        │
│     └─────────────────────────────────────────┘        │
│     두 개 모두 ON                                        │
│                                                         │
│  5. "Save Changes"                                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────┐
│  [2/3]  봇 토큰 + 서버 초대                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  토큰:                                                   │
│  1. Bot 메뉴 TOKEN → "Reset Token" → Yes, do it!        │
│  2. MTxxx... 토큰 복사 (한 번만 표시)                    │
│                                                         │
│  서버 초대 URL:                                          │
│  3. 왼쪽 "OAuth2" → "URL Generator"                      │
│  4. SCOPES: bot                                          │
│  5. BOT PERMISSIONS:                                     │
│     ☑ View Channels                                     │
│     ☑ Send Messages                                     │
│     ☑ Read Message History                              │
│     ☑ Attach Files                                      │
│  6. 하단 URL 복사 → 새 탭에서 열기 → 서버 선택 → 승인   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────┐
│  [3/3]  내 Discord ID                                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Discord 앱 → 좌하단 톱니바퀴(⚙) → 설정               │
│  2. "고급" → "개발자 모드" ON                            │
│  3. 설정 닫기                                            │
│  4. 좌하단 내 아이콘 우클릭 → "사용자 ID 복사"          │
│     18자리 숫자 (예: 123456789012345678)                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion 두 번:
1. "Discord 봇 토큰을 붙여넣어 주세요"
2. "내 Discord 사용자 ID(숫자) 입력해주세요"

저장:
```bash
 hermes config set DISCORD_BOT_TOKEN [봇 토큰]
 hermes config set DISCORD_ALLOWED_USERS [Discord ID]
```

### 게이트웨이 시작 (공통)

#### 주 경로 (자동) — config 만 저장하고 바로 start

위 Telegram/Discord 섹션에서 이미 `hermes config set TELEGRAM_BOT_TOKEN` 등으로 env var 설정 완료. `hermes gateway setup` 대화형 건너뛰고 바로 start 시도 가능:

```bash
hermes gateway start
```

`hermes gateway start` 는 `.env` 의 토큰을 읽어 자동으로 활성 플랫폼 감지. 성공하면 완료.

#### 폴백 (대화형) — start 가 플랫폼 인식 못 할 때

```bash
hermes gateway setup
```

대화형 화면이 뜨면 Claude 가 한국어 해설:
| Hermes 질문 | Claude 안내 |
|------------|------------|
| "Enable Telegram? (y/n)" | Telegram 봇 만들었으면 `y` |
| "Telegram bot token" | `.env` 에 이미 저장돼 있으면 기본값 그대로 Enter |
| "Enable Discord? (y/n)" | Discord 봇 만들었으면 `y`, Intent ON 확인했는지 재점검 |
| "Discord bot token" | 기본값 Enter |

완료 후:
```bash
hermes gateway start
hermes gateway status
```

안내:
```
Telegram/Discord 에서 방금 만든 봇에게 메시지 보내보세요.
"안녕" 보내면 Hermes 가 답장합니다.

[팁] 나중에 가족/동료에게 봇 권한 주려면:
  hermes pairing list       # 대기 중인 페어링 코드 확인
  hermes pairing approve <code>
  hermes pairing revoke <user>
```

---

## STEP 5 — 검증 및 자가 치유 루프

### 5-A. 기본 검증

3개 명령을 순서대로 실행:
```bash
hermes version
hermes doctor
hermes config get model.default   # 또는 hermes config list
```

**검증 합격 조건:**
1. `hermes version` 에 정상 버전 출력 (v2026.x.x 형태)
2. `hermes doctor` 의 모든 체크 ✓ (또는 warning 만 있고 error 없음)
3. `hermes config get model.default` 에 STEP 3 에서 설정한 모델명 출력 (`hermes config get model.provider` 도 병행 확인)

### 5-B. 실제 대화 테스트 (무과금 최소 호출)

```bash
echo "안녕. 그냥 OK 라고만 답해줘." | hermes --non-interactive 2>&1 | head -20
```

`--non-interactive` 플래그가 없는 버전이면 대안:
```bash
timeout 20 hermes <<< "안녕. OK 라고 답해줘" 2>&1 | head -20
```

**합격 조건:** `OK` 또는 유사한 응답이 출력되면 설정 정상.

### 5-C. 게이트웨이 검증 (STEP 4 에서 설정한 경우만)

```bash
hermes gateway status
```

출력에 `running` / `healthy` / `connected` 계열 문구가 있어야 합격.

---

### 5-D. 실패 시 자가 치유 루프 (최대 3회)

5-A / 5-B / 5-C 중 하나라도 실패하면 **자동으로 해결 시도**. 사용자에게 에러 raw 텍스트만 던지지 말 것.

**루프 1회당 절차:**

1. **에러 텍스트 정규화** — 실제 에러 메시지에서 stack trace 를 제외하고 핵심 1~2 줄만 추출. 예:
   ```
   원본: "Traceback ... ModuleNotFoundError: No module named 'foo'. File line..."
   정규화: "ModuleNotFoundError: No module named 'foo'"
   ```

2. **공식 레포에서 해결책 검색** — 병렬로:
   ```bash
   # hermes-agent Issues 검색
   gh search issues --repo NousResearch/hermes-agent "<정규화된 에러 키워드>" --state all --limit 5

   # hermes-agent PRs 검색
   gh search prs --repo NousResearch/hermes-agent "<정규화된 에러 키워드>" --state all --limit 5
   ```
   동시에 `WebSearch`:
   - `"site:github.com/NousResearch/hermes-agent <에러 키워드>"`
   - `"site:hermes-agent.nousresearch.com troubleshooting <에러 키워드>"`

3. **해결책 후보 선정** — 검색 결과 중:
   - closed + merged PR (fix 확정)
   - 공식 답변 달린 issue
   - 공식 docs 의 troubleshooting 섹션

4. **자동 수정 적용** — 해결책의 요지를 파싱해서 가능한 조치:
   | 에러 유형 | 자동 조치 |
   |----------|---------|
   | PATH 문제 (`command not found`) | `export PATH="$HOME/.local/bin:$PATH"` 후 재시도 |
   | Python 버전 | `uv python install 3.11 && uv python pin 3.11` |
   | Node 버전 | `nvm install 24 && nvm use 24` (nvm 있을 때) |
   | 의존성 누락 | `hermes update` → 재설치 스크립트 재실행 |
   | `PROVIDER` / `MODEL` 대문자 key 오답 | 점 표기법으로 재설정: `hermes config set model.provider <값>` / `hermes config set model.default <값>` |
   | provider 값 오답 (`openai`, `google`, `codex`) | 공식 목록으로 교체 (`openrouter` / `gemini` / `openai-codex`) |
   | Anthropic "out of usage" | OAuth 경로면 API 키로 전환 안내 |
   | Gateway 연결 실패 | `hermes gateway setup` 재실행 + Discord Intent 재확인 |
   | 모델 404/deprecated | STEP 3 모델 WebSearch 재수행 후 `model.default` 재설정 |
   | SSL/네트워크 | 프록시 환경변수 확인, `curl -v` 로 진단 |
   | 자동 경로 알 수 없는 에러 | **대화형 폴백** — `hermes setup` 전체 wizard 실행 후 Claude 가 각 화면 해설 |

5. **재검증** — 5-A/5-B 해당 항목 재실행.

6. **성공하면** → 완료 배너로. **실패하면** 루프 2회차, 다른 검색어/해결책 시도.

### 5-E. 3회 실패 시 → 사용자에게 정제된 리포트

3회 시도 후에도 실패하면 사용자에게 아래 형식으로 보고:

```
╔══════════════════════════════════════════════════════════╗
║  설치 검증 실패                                          ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║  시도한 것 (3회):                                        ║
║  1. <조치 1> → <결과>                                    ║
║  2. <조치 2> → <결과>                                    ║
║  3. <조치 3> → <결과>                                    ║
║                                                          ║
║  실제 에러 (정규화):                                     ║
║    <에러 핵심 1줄>                                       ║
║                                                          ║
║  관련 공식 이슈:                                         ║
║    - #<번호>: <제목> (<상태>)                            ║
║    - #<번호>: <제목>                                     ║
║                                                          ║
║  다음 단계 (사용자 선택):                                 ║
║    a) 새 이슈 올리기 (URL 자동 오픈)                     ║
║    b) Discord 커뮤니티에 질문 (URL)                      ║
║    c) 지금은 건너뛰고 수동 진행                          ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

새 이슈 템플릿은 STEP 0 에서 수집한 환경 정보 + 에러 텍스트 + 시도한 조치를 자동으로 포함. 브라우저 오픈 (규칙 패턴): `https://github.com/NousResearch/hermes-agent/issues/new?title=<URL-encoded>&body=<URL-encoded>`.

---

### 완료 배너 (출력)

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║   ✓  설치 완료!                                          ║
║                                                          ║
║   터미널:       hermes                                    ║
║   웹 대시보드:  hermes dashboard  (http://localhost:9119) ║
║   Telegram:    봇에게 메시지                             ║
║   Discord:     채널에서 @봇이름 멘션                     ║
║                                                          ║
║   업데이트:     hermes update                            ║
║   진단:         hermes doctor                            ║
║   로그:         hermes logs                              ║
║                                                          ║
║   ⭐  도움이 됐다면 GitHub 별 하나 부탁드려요!            ║
║       github.com/Hybirdss/HermEZ                        ║
║                                                          ║
║                           made by hybirdss              ║
╚══════════════════════════════════════════════════════════╝
```

배너 출력 직후 브라우저 오픈 (규칙 패턴): `https://github.com/Hybirdss/HermEZ`

안내:
```
브라우저가 열렸습니다. 도움이 됐다면 ⭐ 눌러주세요!
(브라우저 안 열리면 위 주소 직접 복사)
```

---

## Hermes 전체 명령 치트시트 (설치 후 참고용)

Claude 가 사용자에게 "더 이런 것들도 있어요" 로 제시할 수 있는 공식 명령들. `hermes_cli/main.py` 직독 기반.

### 기본
| 명령 | 설명 |
|------|-----|
| `hermes` | 인터랙티브 CLI 시작 |
| `hermes chat` | `hermes` 와 동일 |
| `hermes version` | 버전 표시 |
| `hermes status` | 상태 점검 (간단) |
| `hermes doctor` | 상세 진단 |
| `hermes logs` | 로그 뷰어 |
| `hermes update` | 업데이트 |
| `hermes uninstall` | 제거 |

### 설정
| 명령 | 설명 |
|------|-----|
| `hermes setup [model\|gateway\|tools\|tts\|terminal\|agent]` | 섹션별 대화형 wizard |
| `hermes config show` | 현재 설정 전체 출력 |
| `hermes config get <path>` | 특정 값 (`model.provider` 등) |
| `hermes config set <key> <값>` | 값 저장 |
| `hermes config check` | 설정 유효성 검사 |
| `hermes config path` / `env-path` | config 파일 경로 출력 |
| `hermes config migrate` | 설정 스키마 업데이트 |
| `hermes config edit` | 기본 에디터로 config 열기 |

### 인증 / 프로바이더
| 명령 | 설명 |
|------|-----|
| `hermes login --provider nous` | Nous Portal OAuth |
| `hermes login --provider openai-codex` | ChatGPT OAuth (= `auth add openai-codex`) |
| `hermes logout` | 크레덴셜 삭제 |
| `hermes auth list` | 저장된 크레덴셜 풀 보기 |
| `hermes auth add <provider>` | 크레덴셜 추가 |
| `hermes auth remove <idx\|id>` | 제거 |
| `hermes auth reset <provider>` | 소진 상태 리셋 |
| `hermes model` | provider/모델 변경 대화형 |

### 메신저 게이트웨이
| 명령 | 설명 |
|------|-----|
| `hermes gateway setup` | 플랫폼 설정 대화형 |
| `hermes gateway run` | 포그라운드 실행 (WSL/Docker/디버그) |
| `hermes gateway install` | systemd/launchd 서비스 설치 |
| `hermes gateway start/stop/restart` | 서비스 제어 |
| `hermes gateway status` | 상태 확인 |
| `hermes gateway uninstall` | 서비스 제거 |
| `hermes pairing list` | 페어링 대기·승인 유저 |
| `hermes pairing approve <code>` | 페어링 승인 |
| `hermes pairing revoke <user>` | 접근 취소 |

### 웹 UI
| 명령 | 설명 |
|------|-----|
| `hermes dashboard` | 웹 대시보드 (기본 포트 9119, 자동 브라우저) |
| `hermes dashboard --port 9000 --no-open` | 포트 변경, 브라우저 안 열기 |

### 스킬 / 플러그인 / MCP
| 명령 | 설명 |
|------|-----|
| `hermes skills browse` | 공식 Skills Hub 탐색 |
| `hermes skills search <쿼리>` | 검색 |
| `hermes skills install <skill>` | 설치 |
| `hermes skills list` / `update` / `uninstall` | 관리 |
| `hermes skills config` | 스킬별 활성화 설정 |
| `hermes plugins list` / `install` / `enable` | 플러그인 관리 |
| `hermes mcp list` / `add` / `remove` / `test` | MCP 서버 관리 |
| `hermes mcp serve` | Hermes 를 MCP 서버로 노출 |
| `hermes tools list` / `enable` / `disable` | 툴 on/off (플랫폼별) |

### 세션 / 메모리 / 프로필
| 명령 | 설명 |
|------|-----|
| `hermes sessions list` | 최근 세션 |
| `hermes sessions browse` | 대화형 브라우저 (FTS5 검색) |
| `hermes sessions export <id> <file>` | JSONL 내보내기 |
| `hermes sessions prune` | 오래된 세션 삭제 |
| `hermes memory setup` | 메모리 provider 설정 |
| `hermes memory status` | 현재 메모리 상태 |
| `hermes memory off` | 외부 provider 끄기 (내장만) |
| `hermes profile list` / `use` / `create` | 멀티 에이전트 프로필 (같은 호스트에서 여러 Hermes 동시 운영) |
| `hermes profile export/import` | 프로필 이식 |

### 자동화 / 통합
| 명령 | 설명 |
|------|-----|
| `hermes cron list` / `create` / `edit` / `pause` | 스케줄 작업 (자연어로 "매일 오전 9시에 X") |
| `hermes cron tick` | 대기 중 작업 즉시 실행 |
| `hermes webhook subscribe` / `list` / `test` | 웹훅 구독 |
| `hermes acp` | ACP 서버 (VS Code/Zed/JetBrains 통합) |
| `hermes dump` | 디버그 덤프 |
| `hermes backup` / `import` | 백업·복원 |
| `hermes debug share` / `delete` | 디버그 정보 공유 |
| `hermes completion <shell>` | bash/zsh/fish autocomplete 생성 |

### OpenClaw 이주
| 명령 | 설명 |
|------|-----|
| `hermes claw migrate` | **OpenClaw 설정·세션·스킬·키를 자동으로 Hermes 로 이주** (OpenClaw 유저가 전환할 때 한 번에) |
| `hermes claw cleanup` | 이주 후 OpenClaw 잔재 제거 |

---

## 에러 처리 규칙

| 에러 | 조치 |
|------|------|
| `hermes: command not found` | Mac/Linux: `export PATH="$HOME/.local/bin:$PATH"`. Windows: PowerShell 재시작 |
| `uv: command not found` | Mac/Linux: `curl -LsSf https://astral.sh/uv/install.sh \| sh` · Windows: `irm https://astral.sh/uv/install.ps1 \| iex` |
| `curl: command not found` | `sudo apt install curl` (Linux) / Xcode tools (Mac) |
| install.sh 가 `sudo` 로 실행됐음 | 중단. `rm -rf ~/.hermes` 후 **sudo 없이** 재실행 (공식 FAQ). `~/.local/bin` 은 사용자 영역 |
| API 키 오류 | `hermes doctor` → 키 재확인. env var 이름 **대문자**, 설정 값은 **점 표기법** |
| `Unknown provider` / `Provider not found` | provider 값이 공식 목록에 있는지 확인 (`openai`, `google`, `codex` 전부 오답 — `openrouter` / `gemini` / `openai-codex`) |
| `PROVIDER` / `MODEL` 대문자 key 안 먹힘 | 점 표기법으로 재설정: `hermes config set model.provider X` / `model.default X` |
| `out of extra usage` (Anthropic) | Claude OAuth 쓰지 말 것. `hermes config set ANTHROPIC_API_KEY` 로 API 키 전환 |
| `preparing <tool>...` 에서 stall (#10364) | Ctrl+C 로 세션 종료 후 `/new` 로 재시작. 반복 시 `hermes update` |
| `/model` picker 중복 (#9545) | 무시. 동일 provider 가 Built-in/User-defined/Compat 으로 3중 표시되는 알려진 버그 — 아무거나 선택 OK |
| Discord `/sethome` 이 동작 이상 (#6447) | config.yaml 에 channel ID 가 잘못 저장될 수 있음. `~/.hermes/config.yaml` 의 해당 엔트리를 `~/.hermes/.env` 로 수동 이동 (`DISCORD_HOME_CHANNEL=...`) |
| 게이트웨이 연결 실패 | `hermes gateway setup` 다시, Discord Intent ON 확인 |
| OpenRouter `Insufficient credits` | 유료 모델 사용 중. STEP 3 선택 1 로 돌아가 `:free` 모델 재조회 후 `model.default` 재설정 |
| 특정 모델 404 / discontinued | `WebSearch` 로 최신 모델 재조회 후 교체 |
| 컨텍스트 감지 오류 (`context exceeded` 너무 빨리) | `~/.hermes/config.yaml` 에 `model.context_length: <숫자>` 수동 설정 |
| 브라우저 자동 오픈 실패 | 규칙 패턴의 echo 안내대로 URL 직접 복사 |
| Python 버전 오류 | Hermes 는 Python **3.10+** 지원, **3.11 권장**. `uv python install 3.11 && uv python pin 3.11` |
| `/model` 이 새 provider 전환 안 됨 | 세션 종료 후 터미널에서 `hermes model` 재실행 (공식 FAQ — `/model` 은 이미 설정된 provider 만 보임) |
| 수동 설정 막혔을 때 | `hermes setup` 공식 wizard 로 완전 재설정 (모든 단계 인터랙티브, Claude 가 한국어 해설) |

에러 발생 시 사용자에게 에러 메시지 보여달라고 하고 위 표 참조.
해결 안 되면 `hermes doctor` 출력 기반 진단.

---

## 보안 체크리스트

- [ ] `hermes config set` 명령 맨 앞에 **공백 한 칸** (bash history 노출 방지)
- [ ] 화면에 API 키 raw 출력 금지
- [ ] config 저장 후 `~/.hermes/.env` 퍼미션 확인 (hermes 가 자동 600)
- [ ] 메신저 봇 토큰은 절대 공유 금지 (`Reset Token` 으로 재발급 가능)
- [ ] Anthropic OAuth 경로 사용 금지 (API 키만)
- [ ] install.sh 를 **sudo 로 실행하지 말 것** — `~/.local/bin` 에 설치되므로 사용자 권한으로 충분. sudo 쓰면 파일 소유권 충돌로 hermes 실행 실패

## 알려진 버그 & 주의사항 (공식 이슈 트래커)

| 이슈 | 증상 | 회피 |
|-----|-----|-----|
| [#6447](https://github.com/NousResearch/hermes-agent/issues/6447) | Discord `/sethome` 이 channel ID 를 `config.yaml` 에 잘못 저장 | 수동으로 `~/.hermes/.env` 에 `DISCORD_HOME_CHANNEL=<id>` 로 이동 |
| [#9545](https://github.com/NousResearch/hermes-agent/issues/9545) | `/model` picker 에 같은 provider 3중 표시 (Built-in, User-defined, Compat) | 아무거나 선택 OK. cosmetic |
| [#10364](https://github.com/NousResearch/hermes-agent/issues/10364) | Agent 가 `preparing <tool>...` 에서 stall | Ctrl+C → `/new` 로 재시작. 반복되면 `hermes update` |
| [#10688](https://github.com/NousResearch/hermes-agent/issues/10688) | Slack gateway 에서 `/model` 이 provider 스위치 못함 | 터미널에서 `hermes model` 로 직접 변경 후 gateway 재시작 |

STEP 5 자가 치유 루프가 위 이슈들을 자동으로 인식해서 대응한다.
