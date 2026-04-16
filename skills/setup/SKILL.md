# Hermes Agent 설치 마법사

Claude가 이 파일을 읽으면 아래 단계를 순서대로 실행한다.
사용자에게 먼저 이 배너를 출력하고 시작한다.

---

## 시작 배너 (출력)

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║    █  █  █▀▀  █▀█  █▀▄▀█  █▀▀  █▀▀                     ║
║    █▀▀█  █▀▀  █▀▄  █ ▀ █  █▀▀  ▀▀█                     ║
║    ▀  ▀  ▀▀▀  ▀ ▀  ▀   ▀  ▀▀▀  ▀▀▀                     ║
║                                                          ║
║         A G E N T   설치 마법사  v0.9                   ║
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
  - Windows (WSL2 없음) — WSL2 먼저 설치 필요
  - Android (Termux) — Termux 앱 필요
```

### Windows 선택 시

아래 내용을 출력하고, 완료했는지 다시 AskUserQuestion으로 확인한다.

```
┌─────────────────────────────────────────────────────────┐
│  Windows → WSL2 설치 (5분)                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Windows 키 누르고 "PowerShell" 검색                  │
│  2. "관리자 권한으로 실행" 클릭                           │
│  3. 아래 명령어 복사해서 붙여넣기:                        │
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

질문: "WSL2 Ubuntu 터미널이 열려있나요?"
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
│     (Play Store 버전은 구버전이라 안 됩니다)             │
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
│  ▸ Python 3.11                                          │
│  ▸ Node.js                                              │
│  ▸ hermes-agent                                         │
│  ▸ 브라우저 도구                                         │
│                                                         │
│  그냥 기다리시면 됩니다.                                  │
└─────────────────────────────────────────────────────────┘
```

실행할 명령:
```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

설치 완료 후 반드시 아래를 실행한다:
```bash
source ~/.bashrc 2>/dev/null || source ~/.zshrc 2>/dev/null || true
```

그리고 hermes가 제대로 설치됐는지 확인:
```bash
hermes version
```

실패하면 PATH 문제다. 아래로 해결:
```bash
export PATH="$HOME/.local/bin:$PATH"
hermes version
```

그래도 실패하면 사용자에게 터미널을 완전히 닫고 다시 열어달라고 안내한다.

---

## STEP 3 — API 키 설정

이 단계가 가장 중요하다. 아래 질문을 AskUserQuestion으로 한다.

```
질문: "AI 모델 API 키가 있으신가요?"
선택지:
  - OpenRouter 키 있음 — 바로 입력
  - Anthropic 키 있음 (Claude) — 바로 입력
  - OpenAI 키 있음 (GPT) — 바로 입력
  - 없음, 지금 만들기 — 무료 가이드
```

### 없음 선택 시 → OpenRouter 무료 가이드

```
┌─────────────────────────────────────────────────────────┐
│  OpenRouter — 무료로 시작하기 (5분)                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. https://openrouter.ai 접속                          │
│  2. "Sign In" → Google 계정으로 가입                     │
│  3. 우측 상단 프로필 → "Keys"                            │
│  4. "Create Key" 클릭                                    │
│  5. 이름 아무거나 입력 → "Create"                        │
│  6. sk-or-v1-... 로 시작하는 키 복사                     │
│                                                         │
│  💡 무료 모델: google/gemini-2.0-flash-001              │
│     가입만 하면 바로 사용 (카드 불필요)                   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "OpenRouter 키를 발급받으셨나요?"
선택지:
  - 네, 키 받았어요 — 다음
  - 카드 입력 없이 쓸 수 있나요? — 설명 (무료 모델은 카드 불필요)
  - 다른 방법 원해요 — Anthropic/OpenAI 안내

### 키 입력 단계

키 종류별로 hermes config 명령으로 설정한다:

**OpenRouter:**
```bash
hermes config set openrouter_api_key [사용자가 입력한 키]
```

**Anthropic:**
```bash
hermes config set anthropic_api_key [사용자가 입력한 키]
```

**OpenAI:**
```bash
hermes config set openai_api_key [사용자가 입력한 키]
```

키는 AskUserQuestion의 "Other" (직접 입력)으로 받는다.
단, 키를 화면에 그대로 출력하지 말 것 (보안).

---

## STEP 4 — 모델 선택

API 키 종류에 따라 자동으로 추천 모델을 설정한다.

```
┌─────────────────────────────────────────────────────────┐
│  추천 모델 설정                                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  OpenRouter:                                            │
│    무료  → google/gemini-2.0-flash-001                  │
│    유료  → anthropic/claude-sonnet-4                    │
│                                                         │
│  Anthropic: claude-sonnet-4                             │
│  OpenAI:    gpt-4o                                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "모델을 어떻게 설정할까요?"
선택지:
  - 추천대로 자동 설정 (Recommended) — 자동 실행
  - 직접 고를게요 — hermes model 실행 안내

추천 선택 시 실행:
```bash
hermes config set provider openrouter    # 또는 해당 제공자
hermes config set model google/gemini-2.0-flash-001   # 또는 해당 모델
```

---

## STEP 5 — 메신저 연동 (선택)

```
┌─────────────────────────────────────────────────────────┐
│  📱 메신저에서 Hermes 사용하기                           │
│                                                         │
│  Telegram이나 Discord로 연동하면                         │
│  폰으로 어디서든 AI를 쓸 수 있습니다.                    │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "메신저 연동을 설정할까요?"
선택지:
  - Telegram 연동 — 봇 만들고 연결
  - Discord 연동 — 봇 만들고 연결
  - 둘 다 — Telegram 먼저, 그 다음 Discord
  - 나중에 — 건너뛰기

### Telegram 연동 흐름

```
┌─────────────────────────────────────────────────────────┐
│  Telegram 봇 만들기 (3분)                                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Telegram 앱 열기                                     │
│  2. 검색창에 @BotFather 검색 → 파란 체크 있는 거 선택     │
│  3. /newbot 입력 (슬래시 포함)                           │
│  4. 봇 이름 입력 (아무거나, 예: "내 AI 비서")            │
│  5. 봇 아이디 입력 (영어+bot으로 끝나야 함, 예: myai_bot)│
│  6. 토큰 받음: 1234567890:ABCdef... 형태                  │
│     → 이 토큰을 복사해두기                               │
│                                                         │
│  내 Telegram ID 확인:                                    │
│  7. @userinfobot 검색 → /start 입력                     │
│  8. 숫자로 된 ID 확인 (예: 123456789)                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion으로 두 가지를 받는다:
1. "BotFather에서 받은 봇 토큰을 입력해주세요" (Other로 직접 입력)
2. "@userinfobot에서 확인한 내 Telegram ID를 입력해주세요" (Other로 직접 입력)

받은 값으로 실행:
```bash
hermes config set telegram_bot_token [봇 토큰]
hermes config set telegram_allowed_users [Telegram ID]
```

그 다음:
```bash
hermes gateway start
```

검증:
```bash
hermes gateway status
```

완료 후 사용자에게 안내:
```
Telegram에서 자신이 만든 봇에게 메시지를 보내보세요.
"안녕하세요" 보내면 Hermes가 답장합니다.
```

### Discord 연동 흐름

```
┌─────────────────────────────────────────────────────────┐
│  Discord 봇 만들기 (5분)                                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. https://discord.com/developers/applications 접속     │
│  2. "New Application" → 이름 입력 → "Create"            │
│  3. 왼쪽 메뉴 "Bot" 클릭                                 │
│  4. "Add Bot" → "Yes, do it!"                           │
│                                                         │
│  ⚠️  여기가 핵심! 안 하면 봇이 먹통됩니다:               │
│  5. "Privileged Gateway Intents" 아래에서               │
│     → MESSAGE CONTENT INTENT 를 ON으로 켜기             │
│     → SERVER MEMBERS INTENT 도 ON으로 켜기              │
│  6. "Save Changes" 클릭                                  │
│                                                         │
│  봇 토큰 얻기:                                           │
│  7. "Reset Token" → "Yes, do it!" → 토큰 복사           │
│                                                         │
│  봇 서버에 초대:                                         │
│  8. 왼쪽 메뉴 "OAuth2" → "URL Generator"                │
│  9. SCOPES: bot 체크                                     │
│  10. BOT PERMISSIONS: Send Messages, Read Message       │
│      History, Attach Files, View Channels 체크          │
│  11. 생성된 URL 복사 → 브라우저에서 열기 → 서버 선택      │
│                                                         │
│  내 Discord ID 얻기:                                     │
│  12. Discord 설정(톱니바퀴) → 고급 → 개발자 모드 ON      │
│  13. 내 프로필 우클릭 → "사용자 ID 복사"                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion으로 두 가지를 받는다:
1. "Discord 봇 토큰을 입력해주세요"
2. "내 Discord 사용자 ID를 입력해주세요"

받은 값으로 실행:
```bash
hermes config set discord_bot_token [봇 토큰]
hermes config set discord_allowed_users [Discord ID]
```

그 다음:
```bash
hermes gateway start
```

완료 후 사용자에게 안내:
```
Discord 서버 채널에서 봇을 @멘션하거나 DM을 보내보세요.
```

---

## STEP 6 — 검증 및 완료

```bash
hermes doctor
```

출력 결과를 확인하고 문제가 있으면 해결한다.

### 완료 배너 (출력)

```
╔══════════════════════════════════════════════════════════╗
║                                                          ║
║   ✓  설치 완료!                                          ║
║                                                          ║
║   터미널에서:   hermes                                    ║
║   Telegram:    봇에게 메시지 보내기                       ║
║   Discord:     채널에서 @봇이름 멘션                      ║
║                                                          ║
║   업데이트:    hermes update                             ║
║   문제 생기면: hermes doctor                             ║
║                                                          ║
║   ⭐  도움이 됐다면 GitHub 별 하나 부탁드려요!            ║
║       github.com/hybirdss/hermez                        ║
║                                                          ║
║                           made by hybirdss              ║
╚══════════════════════════════════════════════════════════╝
```

배너 출력 직후, OS에 맞는 명령으로 브라우저를 자동으로 연다:

```bash
# Mac
open "https://github.com/hybirdss/hermez" 2>/dev/null ||
# Linux
xdg-open "https://github.com/hybirdss/hermez" 2>/dev/null ||
# WSL
powershell.exe /c start "https://github.com/hybirdss/hermez" 2>/dev/null ||
true
```

명령 실행 후 사용자에게 이렇게 안내한다:
```
브라우저가 열렸습니다. 도움이 됐다면 ⭐ 눌러주세요!
(브라우저가 안 열렸다면 직접 주소를 복사하세요)
```

---

## 에러 처리 규칙

| 에러 | 조치 |
|------|------|
| `hermes: command not found` | `export PATH="$HOME/.local/bin:$PATH"` 실행 후 재시도 |
| `curl: command not found` | `sudo apt install curl` (Linux) 또는 Xcode tools (Mac) |
| API 키 오류 | `hermes doctor` 실행 후 키 재확인 |
| 게이트웨이 연결 실패 | 토큰 재확인 → Intent 설정 확인 (Discord) |
| Python 버전 오류 | `uv python install 3.11` 실행 |

에러가 발생하면 사용자에게 에러 메시지를 보여달라고 하고, 위 표를 참조해서 해결한다.
해결 안 되면 `hermes doctor` 출력을 바탕으로 진단한다.
