# Hermes Agent 설치 마법사

Claude가 이 파일을 읽으면 아래 단계를 순서대로 실행한다.
사용자에게 먼저 이 배너를 출력하고 시작한다.

**치명적 원칙:**
- `hermes config set` 의 키 이름은 **전부 대문자** (UPPER_SNAKE_CASE). 소문자 쓰면 안 먹힘.
- API 키가 포함된 명령은 실행 전 **맨 앞에 공백 한 칸**을 넣는다 (`HISTCONTROL=ignorespace` 가 기본이면 bash history에 안 남음).
- 키는 화면에 그대로 출력하지 말 것.

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

## STEP 1 — OS 확인

AskUserQuestion으로 묻는다:

```
질문: "어떤 환경에서 설치하시나요?"
선택지:
  - Mac (추천) — 바로 설치 가능
  - Linux / WSL2 — 바로 설치 가능
  - Windows (WSL2 없음) — WSL2 먼저 설치 (5분, 자동)
  - Android (Termux) — Termux 앱 필요
```

### Windows 선택 시 → WSL2 필수

Hermes는 **Windows 네이티브 미지원** (공식 확인). WSL2 설치:

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
│  6. 사용자이름/비밀번호 설정 (뭐든 상관없음)              │
│  7. 그 Ubuntu 터미널 창에서 계속 진행                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "WSL2 Ubuntu 터미널이 열려있나요?"
선택지:
  - 네, Ubuntu 터미널 열려있어요 — 다음 단계
  - 아직 설치 중이에요 — 완료되면 알려달라고 안내
  - 이미 WSL2 있어요 — 다음 단계

### Android 선택 시

```
┌─────────────────────────────────────────────────────────┐
│  Android → Termux 설치                                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. F-Droid에서 Termux 설치                              │
│     (Play Store 버전은 구버전이라 안 됨)                 │
│     https://f-droid.org → "Termux" 검색                 │
│                                                         │
│  2. Termux 열고 아래 실행:                               │
│     pkg update && pkg upgrade                           │
│                                                         │
│  3. 완료되면 다음 단계 진행                               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## STEP 2 — 설치

아래 내용을 출력한 뒤, Bash로 설치 명령을 실행한다.

```
┌─────────────────────────────────────────────────────────┐
│  설치 중...  (보통 2~5분)                                │
│                                                         │
│  Python, Node.js, hermes-agent, 브라우저 도구            │
│  → 설치 스크립트가 알아서 다 해줍니다.                   │
│                                                         │
│  그냥 기다리시면 됩니다.                                  │
└─────────────────────────────────────────────────────────┘
```

실행할 명령 (공식 설치 스크립트, Android는 자동으로 `.[termux]` extra 적용):
```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

설치 완료 후 반드시 shell reload:
```bash
source ~/.bashrc 2>/dev/null || source ~/.zshrc 2>/dev/null || true
```

hermes가 제대로 설치됐는지 확인:
```bash
hermes version
```

실패하면 PATH 문제:
```bash
export PATH="$HOME/.local/bin:$PATH"
hermes version
```

그래도 실패하면 사용자에게 터미널을 완전히 닫고 다시 열어달라고 안내한다.

---

## STEP 3 — AI 모델 연결

이 단계가 가장 중요하다. **2026년 기준으로 가장 쉬운 4가지 방법**을 제시한다.

AskUserQuestion:

```
질문: "AI 모델 어떻게 쓰시겠어요? (1번이 가장 쉬움)"
선택지:
  - 🎯 OpenRouter 무료 (카드 등록 없음, 1분)  — 가장 쉬움 (Recommended)
  - ⭐ Claude 계정으로 로그인 (Anthropic OAuth)  — Claude 유저용
  - 💳 ChatGPT Plus/Pro 계정으로 크레딧 받기 (OpenAI OAuth)  — GPT 유저용
  - 🛠 이미 API 키 있어요 — 바로 입력
```

---

### 선택 1 — OpenRouter 무료

브라우저 자동 오픈:
```bash
open "https://openrouter.ai/keys" 2>/dev/null || \
xdg-open "https://openrouter.ai/keys" 2>/dev/null || \
powershell.exe /c start "https://openrouter.ai/keys" 2>/dev/null || true
```

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
│     ⚠️  이 창 닫으면 키를 다시 볼 수 없음!              │
│                                                         │
│  💡 카드 등록 없이 무료 모델 사용 가능                    │
│  💡 무료 한도: 하루 50회 요청, 분당 20회                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "OpenRouter 키를 복사하셨나요?"
선택지:
  - 네, 복사했어요 — 키 입력 단계로
  - Google 로그인이 안 돼요 — 안내 재출력
  - 다른 방법 원해요 — STEP 3 처음으로

키를 Other 로 입력받아 설정 (맨 앞 공백 + 대문자 키명):
```bash
 hermes config set OPENROUTER_API_KEY [입력받은 키]
 hermes config set MODEL "openrouter/deepseek/deepseek-chat-v3:free"
```

> **모델 선택 이유:** `deepseek-chat-v3:free` 는 2026-04 기준 OpenRouter 무료 티어에서 가장 안정적. 과거 추천이었던 `google/gemini-2.0-flash-exp:free` 는 2026-02 에 deprecated. `google/gemini-2.0-flash-001` 는 유료 ($0.10/M input).

---

### 선택 2 — Claude 계정 OAuth (Anthropic)

Hermes는 **Claude Code의 credential store를 재사용**하므로, Claude Code 이미 설치·로그인해 있으면 키 입력 필요 없음.

```bash
hermes model
```

대화형 선택 화면에서:
1. `Anthropic` 선택
2. 인증 방식 묻는 화면에서 `OAuth (Claude Code)` 선택
3. 브라우저가 열리면 Claude 계정으로 로그인
4. "Authorize" 클릭
5. 모델 선택 화면에서 `claude-sonnet-4` 선택

```
┌─────────────────────────────────────────────────────────┐
│  Anthropic OAuth 장점                                    │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ✓ Claude Pro/Max 구독 그대로 사용                       │
│  ✓ 키 별도 발급·관리 불필요                              │
│  ✓ 토큰 자동 갱신 (refresh)                              │
│  ✓ Claude Code 와 credential 공유                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "hermes model 로 OAuth 연결이 끝났나요?"
선택지:
  - 네, 완료 — STEP 4로
  - Claude 계정 없음 — 선택 1(OpenRouter) 로 우회
  - "out of extra usage" 에러 — `hermes config set ANTHROPIC_API_KEY` 로 유료 키 전환 안내

---

### 선택 3 — ChatGPT Plus/Pro OAuth (OpenAI)

**핵심:** OpenAI는 `platform.openai.com/api-keys` 에서 "Sign in with ChatGPT" 버튼이 2026에 추가됨. ChatGPT Plus ($5)/Pro ($50) 유저에게 API 크레딧 자동 지급.

브라우저 자동 오픈:
```bash
open "https://platform.openai.com/api-keys" 2>/dev/null || \
xdg-open "https://platform.openai.com/api-keys" 2>/dev/null || \
powershell.exe /c start "https://platform.openai.com/api-keys" 2>/dev/null || true
```

```
┌─────────────────────────────────────────────────────────┐
│  ChatGPT 계정으로 API 키 자동 발급 (Plus: $5, Pro: $50)  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. 페이지 상단 "Sign in with ChatGPT" 버튼 클릭         │
│     (일반 로그인 아님, ChatGPT 버튼임)                   │
│                                                         │
│  2. ChatGPT 계정으로 로그인 (Plus/Pro 구독 계정)         │
│                                                         │
│  3. 조직(Organization) 선택 → "Authorize"                │
│     → 자동으로 API 키 생성됨                             │
│                                                         │
│  4. 생성된 sk-... 키 복사                                │
│     ⚠️  한 번만 표시됨                                   │
│                                                         │
│  💡 Plus: $5 크레딧 / Pro: $50 크레딧 즉시 지급          │
│  💡 크레딧 30일 후 만료                                  │
│                                                         │
│  ChatGPT Free 플랜은 크레딧 없음 → 카드 등록 필요        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

키를 Other 로 입력받아 설정:
```bash
 hermes config set OPENAI_API_KEY [입력받은 키]
 hermes config set MODEL "openai/gpt-4o-mini"
```

> **모델:** `gpt-4o-mini` 는 현재 가장 저렴하면서 도구 호출 성능 좋음. `gpt-4o` 는 10배 비쌈.

---

### 선택 4 — 수동 API 키

이미 키가 있는 숙련자용. 어떤 provider 키인지 묻고 해당 명령 실행.

| Provider | 명령어 |
|----------|-------|
| OpenRouter | ` hermes config set OPENROUTER_API_KEY [키]` |
| Anthropic | ` hermes config set ANTHROPIC_API_KEY [키]` |
| OpenAI | ` hermes config set OPENAI_API_KEY [키]` |

그 후 모델 설정:
```bash
hermes model    # 대화형으로 provider + model 선택
```

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

AskUserQuestion: "메신저 연동을 설정할까요?"
선택지:
  - Telegram 연동 — 봇 만들고 연결
  - Discord 연동 — 봇 만들고 연결
  - 둘 다 — Telegram 먼저, 그 다음 Discord
  - 나중에 — 건너뛰기 → STEP 5

### Telegram 연동

먼저 BotFather 오픈:
```bash
open "https://t.me/BotFather" 2>/dev/null || \
xdg-open "https://t.me/BotFather" 2>/dev/null || \
powershell.exe /c start "https://t.me/BotFather" 2>/dev/null || true
```

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
│  Step 1. /newbot  (그대로 입력)                          │
│                                                         │
│  Step 2. 봇 이름 입력                                    │
│          예시: 내 AI 비서                                │
│          (한글 가능, 뭐든 상관없음)                      │
│                                                         │
│  Step 3. 봇 아이디 입력                                  │
│          규칙: 영어만, 반드시 bot으로 끝나야 함           │
│          예시: myhermes_bot                              │
│                                                         │
│  완료되면 아래 형태 토큰:                                 │
│  1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ                  │
│  → 이 줄 전체 복사                                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "봇 토큰 복사했나요?"
선택지:
  - 네, 토큰 복사했어요 — 다음
  - bot으로 끝나는 아이디가 이미 있대요 — 다른 이름 재시도 안내
  - BotFather 를 못 찾겠어요 — 검색창에 @BotFather 입력, 파란 체크 확인

userinfobot 오픈:
```bash
open "https://t.me/userinfobot" 2>/dev/null || \
xdg-open "https://t.me/userinfobot" 2>/dev/null || \
powershell.exe /c start "https://t.me/userinfobot" 2>/dev/null || true
```

```
┌─────────────────────────────────────────────────────────┐
│  [2/2]  내 Telegram ID 확인                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  방금 열린 @userinfobot 에서 /start                      │
│                                                         │
│  Id: 123456789  ← 이 숫자만 복사                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion 두 번 (Other 로 직접 입력):
1. "BotFather 에서 받은 봇 토큰을 붙여넣어 주세요"
2. "@userinfobot 에서 확인한 숫자 ID 를 입력해주세요"

받은 값으로 config (맨 앞 공백 + 대문자):
```bash
 hermes config set TELEGRAM_BOT_TOKEN [봇 토큰]
 hermes config set TELEGRAM_ALLOWED_USERS [Telegram ID]
```

### Discord 연동

Discord 개발자 포털 오픈:
```bash
open "https://discord.com/developers/applications" 2>/dev/null || \
xdg-open "https://discord.com/developers/applications" 2>/dev/null || \
powershell.exe /c start "https://discord.com/developers/applications" 2>/dev/null || true
```

```
┌─────────────────────────────────────────────────────────┐
│  [1/3]  봇 앱 만들기                                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Discord 계정으로 로그인                              │
│  2. 우측 상단 "New Application" → 이름 입력 → Create    │
│  3. 왼쪽 메뉴 "Bot" 클릭                                 │
│                                                         │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  ⚠️  여기 안 하면 봇이 완전 먹통!                        │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                         │
│  4. 아래로 스크롤 → "Privileged Gateway Intents"         │
│     ┌─────────────────────────────────────────┐        │
│     │ SERVER MEMBERS INTENT    [ON  ●]         │        │
│     │ MESSAGE CONTENT INTENT   [ON  ●]         │        │
│     └─────────────────────────────────────────┘        │
│     두 개 모두 ON                                        │
│                                                         │
│  5. "Save Changes" 클릭                                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────┐
│  [2/3]  봇 토큰 발급 + 서버 초대                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  토큰:                                                   │
│  1. Bot 메뉴 상단 TOKEN → "Reset Token" → Yes, do it!   │
│  2. MTxxx... 형태 토큰 복사 (한 번만 보임)               │
│                                                         │
│  서버 초대 URL:                                          │
│  3. 왼쪽 "OAuth2" → "URL Generator"                      │
│  4. SCOPES: bot 체크                                     │
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
│  [3/3]  내 Discord ID 확인                               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Discord 앱 → 좌하단 톱니바퀴(⚙) → 설정               │
│  2. "고급" → "개발자 모드" ON                            │
│  3. 설정 닫기                                            │
│  4. 좌하단 내 아이콘 우클릭 → "사용자 ID 복사"          │
│     18자리 숫자 복사 (예: 123456789012345678)            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion 두 번:
1. "Discord 봇 토큰을 붙여넣어 주세요"
2. "내 Discord 사용자 ID(숫자)를 입력해주세요"

받은 값:
```bash
 hermes config set DISCORD_BOT_TOKEN [봇 토큰]
 hermes config set DISCORD_ALLOWED_USERS [Discord ID]
```

### 게이트웨이 시작 (공통)

**반드시 setup 먼저** (공식 순서):
```bash
hermes gateway setup
```

대화형 화면에서 어떤 플랫폼 켤지 선택. 방금 config 한 것들이 자동으로 인식됨.
완료되면 start:
```bash
hermes gateway start
```

완료 후 안내:
```
Telegram/Discord 에서 방금 만든 봇에게 메시지 보내보세요.
"안녕" 보내면 Hermes 가 답장합니다.
```

---

## STEP 5 — 검증 및 완료

```bash
hermes doctor
```

모든 체크가 ✓ 면 완료. 문제 있으면 해당 섹션 출력 보고 조치.

### 완료 배너 (출력)

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║   ✓  설치 완료!                                          ║
║                                                          ║
║   터미널에서:   hermes                                    ║
║   웹 대시보드:  hermes web                                ║
║   Telegram:    봇에게 메시지 보내기                       ║
║   Discord:     채널에서 @봇이름 멘션                      ║
║                                                          ║
║   업데이트:    hermes update                             ║
║   진단:        hermes doctor                             ║
║                                                          ║
║   ⭐  도움이 됐다면 GitHub 별 하나 부탁드려요!            ║
║       github.com/Hybirdss/HermEZ                        ║
║                                                          ║
║                           made by hybirdss              ║
╚══════════════════════════════════════════════════════════╝
```

배너 출력 직후 브라우저 자동 오픈:
```bash
open "https://github.com/Hybirdss/HermEZ" 2>/dev/null || \
xdg-open "https://github.com/Hybirdss/HermEZ" 2>/dev/null || \
powershell.exe /c start "https://github.com/Hybirdss/HermEZ" 2>/dev/null || true
```

안내:
```
브라우저가 열렸습니다. 도움이 됐다면 ⭐ 눌러주세요!
(브라우저 안 열렸으면 주소 직접 복사)
```

---

## 에러 처리 규칙

| 에러 | 조치 |
|------|------|
| `hermes: command not found` | `export PATH="$HOME/.local/bin:$PATH"` 후 재시도 |
| `curl: command not found` | `sudo apt install curl` (Linux) / Xcode tools (Mac) |
| API 키 오류 | `hermes doctor` → 키 재확인. config 키 이름이 **대문자**인지 확인 |
| `out of extra usage` (Anthropic OAuth) | `hermes config set ANTHROPIC_API_KEY` 로 유료 키 전환 |
| 게이트웨이 연결 실패 | `hermes gateway setup` 다시 실행, Discord Intent ON 확인 |
| OpenRouter `Insufficient credits` | 유료 모델 사용 중. `hermes config set MODEL "openrouter/deepseek/deepseek-chat-v3:free"` 로 무료 전환 |
| Python 버전 오류 | `uv python install 3.11` |
| `gemini-2.0-flash-exp:free` 404 | 2026-02 deprecated. 위 무료 모델로 교체 |

에러 발생 시 사용자에게 에러 메시지 보여달라고 하고 위 표 참조.
해결 안 되면 `hermes doctor` 출력 기반 진단.

---

## 보안 체크리스트

- [ ] `hermes config set` 명령 맨 앞에 **공백 한 칸** 넣었는지 (bash history 노출 방지)
- [ ] 화면에 API 키 raw 출력 금지
- [ ] config 저장 후 `~/.hermes/.env` 퍼미션 확인 (hermes가 자동으로 600 설정)
- [ ] 메신저 봇 토큰은 절대 공유 금지 (`Reset Token` 으로 재발급 가능)
