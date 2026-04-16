# Hermes Agent 설치 마법사

## 실행 순서 (Claude 가 이 파일을 읽었을 때 할 일)

1. 시작 배너 출력
2. **STEP 0-A**: Shell 환경 감지 (POSIX vs PowerShell)
3. **STEP 0-B**: 감지된 shell 에 맞는 환경 조사
4. STEP 0 결과로 "감지: <요약>" 1줄 리포트
5. STEP 1 → STEP 2 → STEP 3 → STEP 4 → STEP 5 순서 진행
6. **STEP 6 (선택)**: 파워 유저 기능 안내

STEP 6 는 선택. Skip 가능.

---

## 🔒 치명적 원칙 (모든 STEP 에 적용)

### 키 네이밍 — 공식 `hermes_cli/setup.py` 기준

두 종류가 섞이면 작동 안 한다. 정확히 구분:

| 종류 | 명령 포맷 | 저장 위치 | 예시 |
|------|----------|----------|------|
| **API 키 / 토큰 (env var)** | `hermes config set <UPPER_SNAKE_CASE> "<값>"` | `~/.hermes/.env` | `hermes config set OPENROUTER_API_KEY "sk-or-v1-..."` |
| **설정 값 (YAML path)** | `hermes config set <lower.dot.path> "<값>"` | `~/.hermes/config.yaml` | `hermes config set model.provider "openrouter"` |

- `PROVIDER`, `MODEL` 같은 대문자 설정 키는 env var 목록에 없음 → **동작 안 함**.
- 공식 자동화 가이드 (`setup.py` line 185) 예시:
  ```bash
  hermes config set model.provider "custom"
  hermes config set model.base_url "http://localhost:8080/v1"
  hermes config set model.default "your-model-name"
  ```
- **모든 값은 따옴표**. 공백 없는 값도 일관성 위해 따옴표 유지.

### Provider 값 (공식 `chat_parser` choices 전체)

| 값 | 설명 | 필요한 크레덴셜 |
|---|------|----------------|
| `auto` | 있는 키로 자동 선택 (기본) | — |
| `openrouter` | OpenRouter 프록시 | `OPENROUTER_API_KEY` 또는 `OPENAI_API_KEY` |
| `anthropic` | Anthropic 직결 | `ANTHROPIC_API_KEY` / `ANTHROPIC_TOKEN` — **OAuth 금지** |
| `openai-codex` | ChatGPT OAuth | `hermes login --provider openai-codex` |
| `copilot` / `copilot-acp` | GitHub Copilot | `GITHUB_TOKEN` |
| `gemini` | Google AI Studio 직결 | `GOOGLE_API_KEY` 또는 `GEMINI_API_KEY` |
| `huggingface` | HuggingFace | `HF_TOKEN` |
| `zai` | z.ai / GLM | `GLM_API_KEY` (alias: `ZAI_API_KEY`, `Z_AI_API_KEY`) |
| `kimi-coding` / `kimi-coding-cn` | Kimi / Moonshot | `KIMI_API_KEY` / `KIMI_CN_API_KEY` |
| `minimax` / `minimax-cn` | MiniMax | `MINIMAX_API_KEY` / `MINIMAX_CN_API_KEY` |
| `nous` | Nous Portal OAuth | `hermes login --provider nous` (+ `NOUS_API_KEY`) |
| `kilocode` / `xiaomi` / `arcee` | 기타 | `KILOCODE_API_KEY` / `XIAOMI_API_KEY` / `ARCEEAI_API_KEY` |
| `custom` | 로컬 LMStudio/Ollama | `model.base_url` 설정 |

추가 env var (doctor 가 인식): `OPENAI_BASE_URL`, `DEEPSEEK_API_KEY`, `DASHSCOPE_API_KEY`, `AI_GATEWAY_API_KEY`, `OPENCODE_ZEN_API_KEY`, `OPENCODE_GO_API_KEY`.

**`openai` 는 공식 provider 목록에 없음.** OpenAI 쓰려면 OpenRouter 경유 또는 ChatGPT OAuth.

### 모델 포맷 — 슬래시 (`/`)

공식 `.env.example` 예시: `anthropic/claude-opus-4.6`. config 저장·env 에선 슬래시, `/model` 슬래시 커맨드 안에서만 콜론.

### Anthropic OAuth 절대 금지

`hermes model` 의 Anthropic provider 에서 "OAuth (Claude Code)" 옵션 뜨면 **선택 금지**. 이유: Hermes ↔ Claude Code credential 공유 시 `out of extra usage` 에러 ([#6475](https://github.com/NousResearch/hermes-agent/issues/6475)). **API 키 경로만 사용.**

### 모델명 하드코딩 금지

이 파일 안의 `deepseek-*`, `gpt-*`, `claude-*`, `glm-*` 등 모든 모델명은 **예시**. 월 단위 deprecated 된다. STEP 3 에서 반드시 `WebSearch` 로 오늘 날짜 기준 최신 모델 조회 후 사용.

### 🎯 2단 실행 전략 (A+B 하이브리드)

각 설정 작업은 **두 경로** 준비:

1. **주 경로 (자동)** — Claude 가 `hermes config set` 로 값 직접 주입. 한국어 가이드 완비.
2. **폴백 (대화형)** — 주 경로 실패 시 `hermes setup [section]` 대화형. Claude 가 각 prompt 한국어 해설.

Claude 는 주 경로 시도 → 실패 감지 시 자동 폴백.

### 🔐 보안 한계 고지 (설치 시작 전 사용자에게 한 번 알릴 것)

```
⚠️  알아두세요
Claude Code 세션에서 'hermes config set KEY value' 실행하면
명령어와 값이 세션 로그 (~/.claude/projects/...) 에 기록됩니다.
민감도 최고 키 (회사 결제 계정 등) 면 이 세션 밖 별도 터미널에서
'hermes setup model' 직접 실행 권장.
일반 개인 키는 이 세션 안에서 진행해도 실용적으로 OK.
```

Shell history 보호:
- **POSIX (bash/zsh):** 명령 맨 앞에 **공백 한 칸** + `HISTCONTROL=ignorespace` 전제
- **PowerShell:** `Set-PSReadLineOption -HistorySaveStyle SaveNothing` 으로 세션 history 끄고 실행 → 완료 후 `SaveIncrementally` 복원

---

## 🌐 브라우저 오픈 규칙

POSIX (bash/zsh) 환경:
```bash
(cmd.exe /c start "" "[URL]" 2>/dev/null || open "[URL]" 2>/dev/null || xdg-open "[URL]" 2>/dev/null) || printf "\n⚠️  브라우저가 자동으로 안 열립니다. 아래 주소를 복사해서 붙여넣어 주세요:\n\n    %s\n\n" "[URL]"
```

PowerShell 환경:
```powershell
try { Start-Process "[URL]" } catch { Write-Host "`n⚠️  브라우저가 자동으로 안 열립니다. 아래 주소를 복사해서 붙여넣어 주세요:`n`n    [URL]`n" }
```

순서 이유 (POSIX): `cmd.exe /c start` → WSL2 에서 가장 안정적 · `open` → macOS · `xdg-open` → Linux 데스크탑.

---

## 🎨 시작 배너

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

## STEP 0 — 환경 조사

### 0-A. Shell 감지 (가장 먼저)

Claude 가 Bash 툴에서 실행:
```bash
uname -s 2>/dev/null || echo "PowerShell"
```

| 결과 | 모드 | 다음 단계 |
|-----|-----|---------|
| `Darwin` / `Linux` / `MINGW*` | POSIX 모드 | 0-B (POSIX) 실행 |
| `PowerShell` 또는 에러 | Windows 네이티브 | 0-B (PowerShell) 실행 |

### 0-B (POSIX). 환경 조사

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
  for cmd in curl git python3 node npm uv hermes gh; do
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
  [ -f ~/.codex/auth.json ] && echo "Codex CLI 인증 존재" || echo "Codex CLI 인증 없음"
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

### 0-B (PowerShell). 환경 조사

```powershell
Write-Host "=== OS ==="
Get-CimInstance Win32_OperatingSystem | Select-Object Caption, OSArchitecture, Version | Format-List

Write-Host "=== 필수 도구 ==="
foreach ($cmd in @('curl','git','python','node','npm','uv','hermes','gh')) {
  $v = & $cmd --version 2>$null | Select-Object -First 1
  if ($v) { Write-Host "${cmd}: $v" } else { Write-Host "${cmd}: (없음)" }
}

Write-Host "=== 기존 설정 ==="
$hermesHome = "$env:LOCALAPPDATA\hermes"
if (Test-Path $hermesHome) { Write-Host "$hermesHome 존재 (기존 설치)" } else { Write-Host "$hermesHome 없음 (신규 설치)" }

Write-Host "=== 네트워크 ==="
try { $r = Invoke-WebRequest -Uri https://raw.githubusercontent.com -TimeoutSec 5 -UseBasicParsing; Write-Host "github.com: $($r.StatusCode)" } catch { Write-Host "github.com: unreachable" }
```

### 감지 결과 해석

| 감지 | 분기 |
|-----|-----|
| `uname -s` = `Darwin` | STEP 1 Mac 경로 자동 |
| `IS_WSL=1` | STEP 1 WSL2 경로 자동 |
| Linux (WSL 아님) | STEP 1 Linux 경로 자동 |
| Windows PowerShell | STEP 1 Windows 네이티브 경로 |
| `~/.hermes` 존재 | "기존 설치 감지 — 새로/재설정/업데이트?" 묻기 |
| `hermes` 이미 설치 + 최신 | STEP 2 건너뛰고 STEP 3 |
| `~/.codex/auth.json` 존재 | STEP 3 선택 2 에서 OAuth 재사용 제안 |
| `~/.openclaw` 존재 | STEP 2 뒤 `hermes claw migrate` 실행 제안 |
| `gh` 없음 | STEP 5 자가치유는 WebSearch 만 사용 |
| Python < 3.10 | `uv python install 3.11 && uv python pin 3.11` |
| `curl` / `uv` 없음 | STEP 2 전에 설치 안내 |
| `Home 여유 < 2G` | 경고 후 계속 |
| `github.com` unreachable | 프록시 여부 질문 |

감지 결과 한 줄로 리포트:
```
감지: macOS 15.2 (arm64) · zsh · Python 3.11 · Node 24 · hermes 미설치 · 네트워크 OK
```

---

## STEP 1 — OS 확인

STEP 0 결과로 **자동 분기**. 감지 불확실할 때만 AskUserQuestion:

```
질문: "어떤 환경에서 설치하시나요?"
선택지:
  - Mac (추천)
  - Linux / WSL2
  - Windows (네이티브 PowerShell)
```

### Windows 네이티브 선택 시

**Hermes 는 `install.ps1` 공식 제공 → Windows 네이티브 완전 지원.** WSL2 는 선택.

AskUserQuestion:
```
질문: "Windows 에서 어느 쪽?"
선택지:
  - 🪟 네이티브 PowerShell (가장 빠름, Recommended)
  - 🐧 WSL2 Ubuntu (Linux 도구 선호 시)
  - 이미 WSL2 있어요
```

#### WSL2 없으면 설치 안내

```
┌─────────────────────────────────────────────────────────┐
│  Windows → WSL2 설치 (5분)                              │
│                                                         │
│  1. Windows 키 → "PowerShell" 검색                       │
│  2. "관리자 권한으로 실행"                                │
│  3. 붙여넣기:      wsl --install                         │
│  4. 재시작 → Ubuntu 창 자동 열림                          │
│  5. 사용자이름/비밀번호 설정 → 완료                       │
└─────────────────────────────────────────────────────────┘
```

---

## STEP 2 — 설치

### Mac / Linux / WSL2

**`--skip-setup` 필수.** 없으면 install 끝에 대화형 wizard 떠서 우리 STEP 3 와 충돌.

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash -s -- --skip-setup
```

> **`sudo` 로 실행 금지.** `~/.local/bin` 사용자 영역 설치. sudo 쓰면 파일 소유권 충돌로 hermes 실행 실패 (공식 FAQ).

설치 후 shell reload:
```bash
source ~/.bashrc 2>/dev/null || source ~/.zshrc 2>/dev/null || true
```

### Windows 네이티브 PowerShell

`irm | iex` 로는 `-SkipSetup` 파라미터 전달 불가. scriptblock 로 실행:

```powershell
$installer = irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1
& ([scriptblock]::Create($installer)) -SkipSetup
```

설치 후 PowerShell 완전히 닫고 새로 열기 (PATH 반영).

### 확인 (공통)

```bash
hermes version
```

실패 시:

| 증상 | 조치 |
|-----|-----|
| `hermes: command not found` (POSIX) | `export PATH="$HOME/.local/bin:$PATH"` |
| `hermes: 명령을 찾을 수 없음` (PowerShell) | PowerShell 완전히 닫고 새로 열기 |
| `uv: command not found` | POSIX: `curl -LsSf https://astral.sh/uv/install.sh \| sh` · Windows: `irm https://astral.sh/uv/install.ps1 \| iex` |
| Python < 3.10 | `uv python install 3.11 && uv python pin 3.11` |

> **A+B 하이브리드.** 각 STEP 에서 막히면 `hermes setup [model\|gateway\|tools\|tts\|terminal\|agent]` 공식 wizard 로 우회. Claude 가 화면 한국어 해설.

---

## STEP 3 — AI 모델 연결

### ⚠️ 모델 추천 원칙

Claude 는 provider 선택 시 **WebSearch 3개 병렬**:
1. `"site:hermes-agent.nousresearch.com recommended model <provider명> <오늘 YYYY-MM>"`
2. `"<provider명> best model <오늘 YYYY-MM> tool calling agent"`
3. `"<후보 모델명> deprecated"` (hit 있으면 제외)

교차해서 남는 1개를 "`<YYYY-MM-DD> 기준 현재 추천: <모델 ID>. 이유: <1줄>`" 으로 제시하고 사용자 승인.

---

AskUserQuestion:
```
질문: "AI 모델 어떻게 쓰시겠어요? (1번이 가장 쉬움)"
선택지:
  - 🎯 OpenRouter 무료 (카드 없음) — Recommended
  - 🤖 ChatGPT 계정 OAuth — ChatGPT Plus/Pro 유저용
  - 🇨🇳 GLM / z.ai — 저렴한 고성능
  - 💳 유료 API 직접 (Claude / Gemini) — 비용 발생
```

---

### 선택 1 — OpenRouter 무료 (카드 없음)

브라우저 오픈: `https://openrouter.ai/keys`

```
┌─────────────────────────────────────────────────────────┐
│  OpenRouter 무료 키 발급 (카드 없음)                      │
│                                                         │
│  1. 우측 상단 "Sign In" → Google 로그인                  │
│  2. 우측 상단 프로필 → "Keys" → "Create Key"             │
│  3. sk-or-v1-... 키 복사 (이 창 닫으면 못 봄)            │
│                                                         │
│  💡 카드 등록 없이 무료 모델 사용                          │
│  💡 한도: 하루 50회, 분당 20회                           │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "키 복사하셨나요?"
선택지: 네 / Google 로그인 안 됨 (재안내) / 다른 방법 (STEP 3 처음)

#### 주 경로 (자동)

WebSearch 로 `:free` 모델 선정 후:
```bash
 hermes config set OPENROUTER_API_KEY "[키]"
 hermes config set model.provider "openrouter"
 hermes config set model.default "openrouter/<선정 모델>"
```

검증:
```bash
hermes config get model.provider
hermes config get model.default
```

#### 폴백 (대화형)

```bash
hermes setup model
```

| Hermes 질문 | Claude 한국어 안내 | 사용자 답 |
|-----------|-----------------|---------|
| "Choose your provider" | "OpenRouter 선택" | `openrouter` |
| "Enter API key" | "방금 복사한 sk-or-v1- 키" | (붙여넣기) |
| "Choose your model" | "오늘 추천: `<모델명>`" | (모델 선택) |

---

### 선택 2 — ChatGPT 계정 OAuth (OpenAI Codex)

**API 키 복사 불필요.** 디바이스 코드 플로우로 ChatGPT 직접 연결.

```
┌─────────────────────────────────────────────────────────┐
│  ChatGPT 계정 OAuth — API 키 발급 불필요                 │
│                                                         │
│  ✓ Plus / Pro 구독 그대로 사용                          │
│  ✓ 토큰 자동 갱신                                        │
│  ✓ Codex CLI (~/.codex/auth.json) 인증 공유 시도         │
│  ⚠ ChatGPT Free 플랜은 안 됨                            │
└─────────────────────────────────────────────────────────┘
```

```bash
hermes login --provider openai-codex
# 또는
hermes auth add openai-codex
```

> **주의 — 알려진 이슈 [#9283](https://github.com/NousResearch/hermes-agent/issues/9283):** `~/.codex/auth.json` 이 있어도 device code 가 강제될 수 있음. 정상 동작. 브라우저에서 한 번 더 인증하면 됨.

사용자 안내:
```
1. 터미널에 뜬 URL 이 브라우저에 자동 오픈 (안 뜨면 수동 복사)
2. 6자리 코드 입력
3. ChatGPT 계정 로그인 → "Authorize"
4. 터미널에 "Authentication successful" 확인
```

AskUserQuestion:
- 네, 완료 — 모델 선택
- 브라우저 안 열려요 — URL 수동 복사 안내
- ChatGPT Free 래요 — 선택 1(OpenRouter) 로

#### 주 경로 — 모델 prefix 불확실 → 대화형 위임 권장

`openai-codex` provider 의 공식 모델 prefix 공개 확인 안 됨. 하드코딩 금지. 안전한 경로:

```bash
hermes setup model
```
→ 메뉴에서 "OpenAI Codex" 선택 → 모델 목록 중 WebSearch 로 찾은 최신 이름과 가장 근접한 것 선택.

provider 만 설정하려면:
```bash
 hermes config set model.provider "openai-codex"
```

---

### 선택 3 — GLM / z.ai

브라우저 오픈: `https://z.ai/manage-apikey/apikey-list`

```
┌─────────────────────────────────────────────────────────┐
│  z.ai API 키 발급                                        │
│                                                         │
│  1. z.ai 로그인 (없으면 회원가입)                         │
│  2. "Create API Key" → 이름 → 키 복사                    │
│                                                         │
│  💡 GLM Coding Plan 구독은 플랜 크레딧                    │
│  💡 개별 종량제는 토큰당 과금                             │
│  ⚠ 엔드포인트 자동 탐지 — GLM_BASE_URL 수동 설정 불필요  │
└─────────────────────────────────────────────────────────┘
```

#### 주 경로

```bash
 hermes config set GLM_API_KEY "[키]"
 hermes config set model.provider "zai"
 hermes config set model.default "zai/<WebSearch 선정 모델>"
```

(env var alias: `ZAI_API_KEY`, `Z_AI_API_KEY` 도 동일하게 인식됨)

#### 폴백

```bash
hermes setup model
```
→ "z.ai / ZhipuAI" → 모델 선택

---

### 선택 4 — 유료 API 직접 (Anthropic / Gemini)

**⚠️ 전부 토큰당 과금.** 무료 원하면 선택 1.

| Provider | provider 값 | 환경변수 | 단가 확인 |
|---------|------------|---------|---------|
| Anthropic Claude | `anthropic` | `ANTHROPIC_API_KEY` | `WebSearch "Anthropic API pricing YYYY-MM"` |
| Google Gemini | `gemini` | `GOOGLE_API_KEY` 또는 `GEMINI_API_KEY` | `WebSearch "Google Gemini API pricing YYYY-MM"` |

> **OpenAI 직접 API 키 보유자:** 선택 1 (OpenRouter) 로 가되 `OPENROUTER_API_KEY` 자리에 `OPENAI_API_KEY` 써도 됨 (OpenRouter 가 OpenAI 키 인식). 또는 선택 2 (ChatGPT OAuth).

가격 단가 조회 후 "오늘 기준: input $X/M, output $Y/M" 1줄 제시.

AskUserQuestion:
- Anthropic Claude
- Google Gemini
- 취소 (STEP 3 처음)

#### Anthropic Claude

브라우저 오픈: `https://console.anthropic.com/settings/keys`

```
1. 로그인 (없으면 회원가입)
2. Billing → 카드 등록 + $5 선충전
3. "Create Key" → sk-ant-... 복사

⚠ 토큰당 과금. OAuth 경로 사용 금지 (API 키만).
```

```bash
 hermes config set ANTHROPIC_API_KEY "[키]"
 hermes config set model.provider "anthropic"
 hermes config set model.default "anthropic/<WebSearch 선정 모델>"
```

폴백: `hermes setup model` → "Anthropic" → **API 키 입력 경로** (OAuth 금지!).

#### Google Gemini

브라우저 오픈: `https://aistudio.google.com/app/apikey`

```
1. Google 로그인
2. "Create API key" → 프로젝트 선택 → AIza... 복사

⚠ 토큰당 과금. Google Cloud 빌링 필요할 수도.
```

```bash
 hermes config set GOOGLE_API_KEY "[키]"
 hermes config set model.provider "gemini"
 hermes config set model.default "gemini/<WebSearch 선정 모델>"
```

폴백: `hermes setup model` → "Gemini" → API 키 입력.

---

## STEP 4 — 메신저 연동 (선택)

```
📱 Telegram 이나 Discord 에 연결하면 폰으로 어디서든 AI 사용.
```

AskUserQuestion:
```
질문: "메신저 연동 설정할까요?"
선택지:
  - Telegram
  - Discord
  - 둘 다 (Telegram 먼저)
  - 나중에 → STEP 5
```

Slack / WhatsApp / Signal / Email 등 다른 플랫폼은 **STEP 6** 에서 다룸.

### Telegram 연동

브라우저 오픈: `https://t.me/BotFather`

```
┌─────────────────────────────────────────────────────────┐
│  [1/2]  봇 만들기                                        │
│                                                         │
│  BotFather = Telegram 공식 봇 관리자 (파란 체크 확인)     │
│                                                         │
│  Step 1. /newbot                                        │
│  Step 2. 봇 이름 (한글 가능, 예: 내 AI 비서)             │
│  Step 3. 봇 아이디 (영어, bot 으로 끝)                   │
│                                                         │
│  완료 시 토큰 표시 → 줄 전체 복사                         │
│  예: 1234567890:ABCdefGHI...                            │
└─────────────────────────────────────────────────────────┘
```

브라우저 오픈: `https://t.me/userinfobot`

```
┌─────────────────────────────────────────────────────────┐
│  [2/2]  내 Telegram ID                                   │
│                                                         │
│  @userinfobot 에서 /start                                │
│  Id: 123456789 ← 숫자만 복사                            │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion 2번:
1. "봇 토큰 붙여넣어 주세요"
2. "Telegram 사용자 ID (숫자)"

```bash
 hermes config set TELEGRAM_BOT_TOKEN "[토큰]"
 hermes config set TELEGRAM_ALLOWED_USERS "[ID]"
```

### Discord 연동

브라우저 오픈: `https://discord.com/developers/applications`

```
┌─────────────────────────────────────────────────────────┐
│  [1/3]  봇 앱 + Intent                                   │
│                                                         │
│  1. "New Application" → 이름 → Create                    │
│  2. 왼쪽 메뉴 "Bot"                                      │
│                                                         │
│  ⚠️  여기 안 하면 봇 먹통:                                │
│  3. 스크롤 → "Privileged Gateway Intents"                │
│       SERVER MEMBERS INTENT  [ON]                        │
│       MESSAGE CONTENT INTENT [ON]                        │
│  4. "Save Changes"                                       │
└─────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────┐
│  [2/3]  토큰 + 서버 초대                                 │
│                                                         │
│  토큰: TOKEN → "Reset Token" → Yes → MTxxx 복사         │
│                                                         │
│  초대 URL: "OAuth2" → "URL Generator"                    │
│    SCOPES: bot                                           │
│    PERMISSIONS: View Channels, Send Messages,           │
│                 Read Message History, Attach Files      │
│  → URL 복사 → 브라우저 → 서버 선택 → 승인                 │
└─────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────┐
│  [3/3]  내 Discord ID                                    │
│                                                         │
│  설정 → 고급 → "개발자 모드" ON                          │
│  내 아이콘 우클릭 → "사용자 ID 복사" (18자리)            │
└─────────────────────────────────────────────────────────┘
```

```bash
 hermes config set DISCORD_BOT_TOKEN "[토큰]"
 hermes config set DISCORD_ALLOWED_USERS "[ID]"
```

### 게이트웨이 시작

#### 주 경로 (자동)

env var 에 이미 저장됐으니 바로:
```bash
hermes gateway start
```

`.env` 의 토큰을 읽어 활성 플랫폼 자동 감지.

#### 폴백 (대화형)

```bash
hermes gateway setup
```

| Hermes 질문 | Claude 안내 |
|-----------|------------|
| "Enable Telegram? (y/n)" | Telegram 설정했으면 `y` |
| "Telegram bot token" | `.env` 값 있으면 Enter (기본값 유지) |
| "Enable Discord? (y/n)" | Discord 설정했으면 `y`, Intent ON 재확인 |

완료 후:
```bash
hermes gateway status
```

안내:
```
Telegram/Discord 에서 방금 만든 봇에게 "안녕" 보내보세요.
Hermes 가 답장합니다.

[팁] 가족·동료에게 봇 권한 주려면:
  hermes pairing list             # 대기 코드 확인
  hermes pairing approve <code>
  hermes pairing revoke <user>
```

---

## STEP 5 — 검증 및 자가 치유 루프

### 5-A. 기본 검증

```bash
hermes version
hermes doctor
hermes config get model.provider
hermes config get model.default
```

**합격 조건:**
- `hermes version` 에 v2026.x.x 출력
- `hermes doctor` 모든 체크 ✓ (warning OK, error 불가)
- `model.provider` · `model.default` 에 STEP 3 설정값 출력

### 5-B. 실제 대화 테스트

POSIX·Windows 공통 (공식 `-q` 단발 비대화식):
```bash
hermes chat -q "안녕. 그냥 OK 라고만 답해줘." 2>&1 | tail -20
```

`-q` 가 없는 구버전이면:
```bash
hermes --help 2>&1 | grep -i "query\|non-interactive" | head -5
```
로 지원 옵션 확인 후 그에 맞춰 실행.

**합격 조건:** `OK` 또는 유사한 짧은 응답 출력.

### 5-C. 게이트웨이 검증 (STEP 4 설정한 경우만)

```bash
hermes gateway status
```
→ `running` / `healthy` / `connected` 계열 문구 확인.

### 5-D. 자가 치유 루프 (최대 3회)

#### 1차 — `hermes doctor --fix`

```bash
hermes doctor --fix
```

공식 doctor 가 자동 수정 가능한 문제 처리. 효과 있으면 5-A 재실행.

#### 2~3차 — 에러 분석 + 공식 레포 검색

1. 에러 텍스트에서 핵심 1~2줄 추출 (stack trace 제외)
2. **공식 레포 검색** — `gh` 있으면 GitHub API, 없으면 WebSearch:
   ```bash
   # gh 설치된 경우
   gh search issues --repo NousResearch/hermes-agent "<에러>" --state all --limit 5

   # gh 없으면 WebSearch
   #   "site:github.com/NousResearch/hermes-agent <에러 키워드>"
   #   "site:hermes-agent.nousresearch.com troubleshooting <에러>"
   ```
3. **자동 조치 적용** — 에러 유형별 표:

   | 에러 유형 | 자동 조치 |
   |---------|---------|
   | PATH (`command not found`) | `export PATH="$HOME/.local/bin:$PATH"` |
   | Python 3.10 미만 | `uv python install 3.11 && uv python pin 3.11` |
   | Node 22 미만 | `nvm install 24 && nvm use 24` (nvm 있을 때) |
   | 의존성 누락 | `hermes update` |
   | `PROVIDER` / `MODEL` 대문자 오답 | 점 표기법 재설정 |
   | provider 값 오답 | 공식 목록 참조 재설정 |
   | 모델 404 / deprecated | STEP 3 WebSearch 재수행 |
   | `context exceeded` 과소 | `model.context_length` 수동 설정 |
   | Gateway 연결 실패 | `hermes gateway setup` + Discord Intent 재확인 |
   | SSL/네트워크 | 프록시 환경변수 확인 |
   | 알 수 없음 | **대화형 폴백** — `hermes setup` 전체 wizard |

4. 5-A / 5-B 해당 항목 재검증

### 5-E. 3회 실패 시 → 사용자 리포트

```
╔══════════════════════════════════════════════════════════╗
║  설치 검증 실패                                          ║
╠══════════════════════════════════════════════════════════╣
║  시도한 것 (3회):                                        ║
║    1. <조치> → <결과>                                    ║
║    2. <조치> → <결과>                                    ║
║    3. <조치> → <결과>                                    ║
║                                                          ║
║  에러 (정규화): <1줄>                                    ║
║                                                          ║
║  관련 공식 이슈:                                         ║
║    - #<번호>: <제목>                                     ║
║                                                          ║
║  다음 단계:                                              ║
║    a) 새 이슈 올리기 (URL 자동 오픈)                     ║
║    b) Discord 커뮤니티 질문                              ║
║    c) 수동 진행                                          ║
╚══════════════════════════════════════════════════════════╝
```

`hermes dump` 출력 자동 포함하여 새 이슈 URL 생성:
```bash
hermes dump                   # 또는 --show-keys
```

브라우저 오픈: `https://github.com/NousResearch/hermes-agent/issues/new?title=<URL-encoded>&body=<URL-encoded>`

---

## ✓ 완료 배너

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║   ✓  설치 완료!                                          ║
║                                                          ║
║   터미널:       hermes                                    ║
║   웹 대시보드:  hermes dashboard                          ║
║   Telegram:    봇에게 메시지                             ║
║   Discord:     채널에서 @봇 멘션                         ║
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

브라우저 오픈: `https://github.com/Hybirdss/HermEZ`

---

## STEP 6 — 파워 유저 기능 (선택)

기본 설치 완료. 아래는 **선택**. 관심 없으면 "됐어요" 로 끝.

### 6-A. 친절한 안내 출력

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   🎁  설치 완료 — 이런 것도 더 할 수 있어요
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   [1] 📱  메신저 더 연결
       Slack · WhatsApp · Signal · Email · iMessage · Matrix 등
       ✓ 16개 플랫폼. 하나의 Hermes 를 여러 채널에서
       ⚠ 비용: 메시지 있을 때만 토큰. 대기 중엔 0원

   [2] 🧩  스킬 자동 설치 (Skills Hub)
       번역·요약·뉴스 수집·일정 관리 등 추천 패키지
       ✓ 말 한마디로 복잡 작업
       ⚠ 비용: 스킬 호출할 때만 토큰

   [3] 🔌  외부 도구 연결 (MCP 서버)
       Playwright(브라우저)·Filesystem·GitHub·Brave 검색 등
       ✓ AI 가 브라우저·파일·검색 직접 조작
       ⚠ 비용: 도구 호출할 때만 토큰

   [4] ⏰  자동화 (Cron)  ★ 비용 경고 필수
       "매일 9시 뉴스 요약해서 텔레그램으로" 같은 예약
       ✓ 자동 알림·요약·일일 리포트
       ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
       ⚠️  비용
       •  실행마다 AI 호출 = 토큰 소모
       •  OAuth (ChatGPT Plus / Nous): 구독 안, 추가 돈 X
       •  API 키 (Anthropic / Gemini): 실제 돈 ↓
       •  :free: 하루 50회 제한 공유
       •  최소 간격 1시간, 시간당 1회 이하 권장
       끄기:  hermes cron pause <id>  /  remove <id>
       ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

   [5] 💓  상태 모니터링 (Heartbeat)
       "30분마다 서버 체크, 이상하면 알림"
       ✓ 다운·에러 자동 감지
       ⚠ Cron 과 동일 (API 키면 돈)
       끄기:  hermes cron remove <id>

   [6] 📊  웹 대시보드 지금 열기
       http://localhost:9119
       ✓ GUI 로 세션·설정·스킬 관리
       ⚠ 비용 0 (로컬만). Foreground 프로세스라 별도 터미널 필요

   [7] 👤  프로필 분리 (업무/개인)
       같은 컴퓨터에서 여러 Hermes 동시 운영
       ✓ 데이터·모델·키 완전 분리
       ⚠ 비용 0

   [0] 🔕  나중에 할래요

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   💡  나중에 "이거 그만 받을래" 라고 말씀만 하시면 끄드립니다.
       비용 걱정되면 언제든 말씀하세요.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

> **아스키 박스 주의** — 한글·이모지 폭이 터미널마다 달라 `║` 세로 정렬이 완벽하지 않을 수 있음. 왼쪽 `━━━` 상하 구분선만 쓰고 오른쪽은 열어둔 이유.

### 6-B. 선택지 질문

**1차** (multiSelect=true, 4개까지):
```
질문: "어떤 기능 켤까요? (여러 개 선택 가능, '됐어요' 선택하면 종료)"
선택지:
  - 📱 메신저 더 추가
  - 🧩 스킬 자동 설치
  - 🔌 MCP 서버 연결
  - ⏰ 자동화 / Cron
```

사용자가 1차에서 4개 다 선택, 또는 "됐어요" 계열 입력 시 **2차 스킵**.

**2차** (1차에서 일부만 선택했고 나머지 관심 있을 때):
```
질문: "이런 것도 있어요. 관심 있으세요?"
선택지:
  - 💓 상태 모니터링
  - 📊 웹 대시보드 지금 열기
  - 👤 프로필 분리
  - 🔕 다 됐어요
```

### 6-C. 각 기능 세부

#### [1] 메신저 더 추가

STEP 4 에서 Telegram/Discord 했으면 그대로 유지. 추가 플랫폼만 활성:

```bash
hermes setup gateway
```

기존 설정은 "already configured" 로 나옴. Enter 로 유지, 새 플랫폼만 `y`.

공식 지원 16개: Telegram · Discord · Slack · WhatsApp · Signal · SMS · Email · Matrix · iMessage(BlueBubbles) · WeChat · Home Assistant · Mattermost · DingTalk · Feishu · WeCom · Webhook

플랫폼별 메모:
- **Slack**: workspace app 설치, Bot Token + App Token 2개
- **WhatsApp**: `hermes whatsapp` QR 페어링
- **Signal**: signal-cli 필요
- **Email**: IMAP/SMTP 자격증명

#### [2] Skills Hub

```bash
hermes skills browse           # 전체 + 검색
hermes skills install <name>
hermes skills list
```

Claude 가 사용자 용도 (업무/개발/일상) 물어본 뒤 `browse` 출력 파싱, 3~5개 일괄 설치 제안.

#### [3] MCP 서버 연결

Claude 가 `WebSearch "top MCP servers <YYYY-MM>"` 로 최신 목록 확인 후 추천:
- Playwright · Filesystem · GitHub · Brave Search · Context7

```bash
hermes mcp add <name>
hermes mcp test <name>
hermes mcp list
```

추가 후 `hermes doctor` 로 검증.

#### [4] Cron 자동화 ★

**1. Provider 판별:**
```bash
hermes config get model.provider
```

| provider | Claude 가 할 말 |
|---------|---------------|
| `openai-codex` / `nous` / `copilot` | "✓ OAuth. 구독 한도 내, 추가 돈 X. rate limit 주의" |
| `anthropic` / `gemini` | "⚠️  API 직과금. 월 예상 비용 계산해드릴게요" |
| `openrouter` + `:free` | "💡 하루 50회 공유. 초과 시 실패" |
| `openrouter` + 유료 | "⚠️  유료. 토큰당 과금" |
| 기타 API | "⚠️  API 과금 가능" |

**2. 비용 예상** (API 기반):
```
현재 모델: <provider>/<model>
단가: input $X/M · output $Y/M (WebSearch 오늘값)
요청당 예상: ~$0.03 (입력 2K + 출력 1K)
하루 1회 → 월 $1
하루 24회 → 월 $22
```

**3. 최소 간격 규칙:**
- 기본 **1시간**
- "매 5분" 요청 시:
  ```
  ⚠️  최소 간격은 1시간입니다.
  이유:
    1. 매 실행마다 토큰 소모 (API 면 돈)
    2. rate limit 위험 (:free 는 하루 50회)
    3. 일상 자동화는 1시간이면 대부분 충분
  15분 간격 꼭 필요한 특별한 이유 있으시면 알려주세요.
  (예: 긴급 서버 모니터링)
  ```
- 예외: 이유 대면 15분까지. 5분 미만은 거부.

**4. 실행 횟수 경고:**
- 시간당 1회 이하 권장 (= 하루 24회 이하)
- "매 30분 + 하루 48회" → 월 예상 비용 명시 후 재확인
- "매 15분 + 하루 96회" 이상 → 강한 경고 + 재확인

**5. 생성 흐름:**
1. 자연어 입력 ("매일 9시 HN 뉴스 요약해서 텔레그램으로")
2. Claude 가 해석:
   ```
   스케줄: 매일 09:00 (0 9 * * *)
   작업: HN top stories 요약
   전달: Telegram
   예상: 하루 1회 × $0.03 = 월 $1 (API)
   ```
3. 사용자 승인
4. 생성:
   ```bash
   hermes cron create "0 9 * * *" "HN top 10 요약 간결히" \
     --name "daily-hn" \
     --deliver telegram
   ```

**6. 비활성화 안내 (반드시):**
```
끄고 싶을 때:
  hermes cron list              # 목록 + ID
  hermes cron pause <id>        # 일시정지
  hermes cron remove <id>       # 영구 삭제

저한테 "daily-hn 끄고 싶어" 라고 말씀만 해도 돼요.
```

#### [5] Heartbeat

Cron 과 동일 구조 + 최소 주기 **30분** (서버 모니터링).

```bash
hermes cron create "every 30m" "홈 서버 헬스체크. 다운이면 알림" \
  --name "heartbeat-home" \
  --deliver telegram
```

[4] 의 비용 경고 + 비활성화 안내 동일 적용.

#### [6] 웹 대시보드 지금 열기

**Foreground 프로세스 주의.** 터미널 하나 점유함.

두 가지 방법:
```bash
# 방법 1: 별도 터미널 열고 실행 (권장)
hermes dashboard

# 방법 2: 백그라운드 (로그 못 봄)
hermes dashboard &
```

기본 포트 9119, 자동 브라우저 오픈. 종료: Ctrl+C (방법 1) 또는 `kill %1` (방법 2).

#### [7] 프로필 분리

```bash
hermes profile list
hermes profile create work       # 새 프로필
hermes profile use work
hermes profile alias work        # 'hermes-work' 래퍼 스크립트 생성
```

이제 `hermes-work` = 업무용, `hermes` = 기본 프로필. 각각 다른 키·모델·스킬.

### 6-D. 완료 후 안내

```
┌──────────────────────────────────────────────────────────┐
│  👂  언제든 저한테 말씀하세요                             │
│                                                          │
│  "그 자동화 그만해"   → cron 삭제                        │
│  "대시보드 열어"      → hermes dashboard                 │
│  "이번 달 얼마 썼어?" → hermes insights                  │
│  "메신저 하나 더"     → hermes setup gateway             │
│                                                          │
│  특히 API 키 쓰실 때:                                    │
│  •  자주 실행되는 작업은 돈 나감                         │
│  •  이상하면 "cron 확인해줘" 라고 하세요                 │
└──────────────────────────────────────────────────────────┘
```

---

## 📚 Hermes 전체 명령 치트시트

`hermes_cli/main.py` 직독 기반. 사용자가 "이런 것도 됩니다" 로 참고.

### 기본
| 명령 | 설명 |
|------|-----|
| `hermes` / `hermes chat` | 인터랙티브 CLI |
| `hermes chat -q "질문"` | 단발 비대화식 |
| `hermes chat --image <경로> -q "설명"` | 이미지 첨부 단발 |
| `hermes chat --resume <id>` | 세션 이어가기 |
| `hermes chat --continue [name]` | 이름 or 최근 세션 |
| `hermes chat --worktree` | 격리 git worktree |
| `hermes chat --checkpoints` | 파괴 작업 전 스냅샷 (`/rollback` 복원) |
| `hermes chat --yolo` | 승인 스킵 (주의) |
| `hermes chat --max-turns 30` | 턴당 툴 호출 제한 |
| `hermes version` | 버전 + Python + 업데이트 확인 |
| `hermes status [--all\|--deep]` | 상태 점검 |
| `hermes doctor [--fix]` | 진단 / 자동 수정 |
| `hermes dump [--show-keys]` | 지원 요청용 환경 요약 |
| `hermes logs` | 로그 뷰어 |
| `hermes insights [--days N]` | 사용량 통계 |
| `hermes update` / `uninstall` | 업데이트 / 제거 |

### 설정
| 명령 | 설명 |
|------|-----|
| `hermes setup [model\|gateway\|tools\|tts\|terminal\|agent]` | 섹션별 대화형 wizard |
| `hermes config show` / `edit` | 현재 설정 / 에디터 |
| `hermes config get <path>` / `set <key> "<값>"` | 조회 / 저장 |
| `hermes config check` / `path` / `env-path` / `migrate` | 검증 / 경로 / 마이그레이션 |

### 인증
| 명령 | 설명 |
|------|-----|
| `hermes login --provider nous\|openai-codex` | OAuth 디바이스 플로우 |
| `hermes logout` | 크레덴셜 삭제 |
| `hermes auth list` / `add <provider>` / `remove <idx>` / `reset <provider>` | 크레덴셜 풀 관리 |
| `hermes model` | provider/모델 변경 대화형 |

### 메신저 게이트웨이
| 명령 | 설명 |
|------|-----|
| `hermes gateway setup` | 플랫폼 설정 대화형 |
| `hermes gateway run` | Foreground (WSL/Docker) |
| `hermes gateway install` / `start` / `stop` / `restart` / `uninstall` | 시스템 서비스 제어 |
| `hermes gateway status` | 상태 확인 |
| `hermes pairing list` / `approve <code>` / `revoke <user>` | 메신저 유저 승인 |

### 웹 / 자동화
| 명령 | 설명 |
|------|-----|
| `hermes dashboard [--port N --no-open]` | 웹 대시보드 (기본 9119) |
| `hermes cron list` / `create <schedule> "<prompt>"` / `pause` / `remove` | 스케줄 작업 |
| `hermes cron edit <id>` / `run <id>` / `tick` | 수정 / 즉시 실행 / 1회 tick |
| `hermes webhook subscribe` / `list` / `test` | 웹훅 |
| `hermes acp` | IDE 통합 서버 (VS Code/Zed/JetBrains) |

### 확장
| 명령 | 설명 |
|------|-----|
| `hermes skills browse` / `install <name>` / `list` / `update` / `uninstall` / `config` | Skills Hub |
| `hermes plugins list` / `install` / `enable` | 플러그인 |
| `hermes mcp list` / `add` / `remove` / `test` / `serve` / `configure` | MCP 서버 |
| `hermes tools list` / `enable` / `disable` | 툴 on/off |
| `hermes memory setup` / `status` / `off` | 메모리 provider |

### 세션 / 프로필
| 명령 | 설명 |
|------|-----|
| `hermes sessions list` / `browse` / `export <id>` / `prune` / `rename` | 세션 관리 (FTS5 검색) |
| `hermes profile list` / `use` / `create` / `delete` / `show` | 멀티 에이전트 |
| `hermes profile alias <name>` | 래퍼 스크립트 생성 (`hermes-work` 등) |
| `hermes profile export` / `import` | 이식 |

### 기타
| 명령 | 설명 |
|------|-----|
| `hermes backup` / `import` | 백업 / 복원 |
| `hermes debug share` / `delete` | 디버그 공유 |
| `hermes completion <shell>` | bash/zsh/fish autocomplete |
| `hermes whatsapp` | WhatsApp QR 페어링 |
| `hermes claw migrate` / `cleanup` | **OpenClaw → Hermes 자동 이주** |

---

## 🔧 에러 처리 통합 표

중복 피하려 여기서만 관리. 5-D 자가치유는 이 표 참조.

| 에러 | 조치 |
|------|------|
| `hermes: command not found` | POSIX: `export PATH="$HOME/.local/bin:$PATH"` · Windows: PowerShell 재시작 |
| `uv: command not found` | POSIX: `curl -LsSf https://astral.sh/uv/install.sh \| sh` · Windows: `irm https://astral.sh/uv/install.ps1 \| iex` |
| `curl: command not found` | `sudo apt install curl` (Linux) / Xcode CLI tools (Mac) |
| install.sh 가 sudo 로 실행됨 | 권한 복구 먼저: `sudo chown -R "$USER":"$(id -gn)" ~/.hermes ~/.local/bin`. 실패 시 백업: `mv ~/.hermes ~/.hermes.bak-$(date +%s)` 후 재설치 |
| API 키 오류 | `hermes doctor` → env var **대문자**, 설정 값 **점 표기법** 확인 |
| `Unknown provider` | `openai`·`google`·`codex` 는 오답. `openrouter`·`gemini`·`openai-codex` 로 교체 |
| `PROVIDER`·`MODEL` 대문자 key 미인식 | `hermes config set model.provider "X"` / `model.default "X"` |
| `out of extra usage` (Anthropic) | Claude OAuth 쓰지 말 것. API 키로 전환 |
| `preparing <tool>...` stall ([#10364](https://github.com/NousResearch/hermes-agent/issues/10364)) | Ctrl+C → `/new`. 반복 시 `hermes update` |
| `/model` 중복 표시 ([#9545](https://github.com/NousResearch/hermes-agent/issues/9545)) | 무시 — cosmetic 버그 |
| Discord `/sethome` 버그 ([#6447](https://github.com/NousResearch/hermes-agent/issues/6447)) | `~/.hermes/config.yaml` 의 channel ID → `~/.hermes/.env` 로 수동 이동 |
| Slack `/model` 스위치 안 됨 ([#10688](https://github.com/NousResearch/hermes-agent/issues/10688)) | 터미널에서 `hermes model` 실행 후 gateway 재시작 |
| Gateway 연결 실패 | `hermes gateway setup` + Discord Intent ON 재확인 |
| OpenRouter `Insufficient credits` | `:free` 모델 WebSearch 재조회 후 `model.default` 교체 |
| 모델 404 / discontinued | WebSearch 로 최신 모델 교체 |
| `context exceeded` 과소 | `hermes config set model.context_length "131072"` 수동 |
| 브라우저 자동 오픈 실패 | URL echo 안내대로 수동 복사 |
| Python 3.10 미만 | `uv python install 3.11 && uv python pin 3.11` |
| `/model` 이 새 provider 전환 안 됨 | 세션 종료 후 `hermes model` 터미널에서 (공식 FAQ) |
| 수동 설정 막힘 | `hermes setup` 전체 wizard 재실행 (Claude 한국어 해설) |

---

## 🔐 보안 체크리스트

- [ ] API 키 명령 전에 shell history 보호 활성 (POSIX: `HISTCONTROL=ignorespace` + 공백 prefix · PowerShell: `Set-PSReadLineOption -HistorySaveStyle SaveNothing`)
- [ ] 화면에 API 키 raw 출력 금지
- [ ] `~/.hermes/.env` 퍼미션 600 확인 (hermes 자동)
- [ ] 메신저 봇 토큰 공유 금지 (`Reset Token` 으로 재발급)
- [ ] Anthropic OAuth 경로 사용 금지 (API 키만)
- [ ] install.sh / install.ps1 **sudo/관리자 없이** 실행
- [ ] Claude Code 세션 밖에서 키 주입 필요한 경우 별도 터미널에서 `hermes setup model` 직접 실행 (세션 transcript 기록 회피)
