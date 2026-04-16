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
  - 없음, 지금 만들기 (무료) — OpenRouter 무료 가이드
  - OpenRouter 키 있음 — 바로 입력
  - Anthropic 키 있음 (Claude) — 바로 입력
  - OpenAI 키 있음 (GPT) — 바로 입력
```

### 없음 선택 시 → OpenRouter 무료 가이드

아래 명령으로 브라우저에서 OpenRouter 키 발급 페이지를 바로 연다:

```bash
open "https://openrouter.ai/keys" 2>/dev/null || \
xdg-open "https://openrouter.ai/keys" 2>/dev/null || \
powershell.exe /c start "https://openrouter.ai/keys" 2>/dev/null || true
```

브라우저가 열리면 아래 안내를 출력한다:

```
┌─────────────────────────────────────────────────────────┐
│  브라우저가 열렸습니다. 아래 순서대로 따라해 주세요.      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. 우측 상단 "Sign In" 클릭                             │
│     → Google 계정으로 로그인 (가장 쉬움)                 │
│                                                         │
│  2. 로그인 후 우측 상단 프로필 아이콘 클릭               │
│     → 드롭다운에서 "Keys" 클릭                           │
│                                                         │
│  3. 파란 "Create Key" 버튼 클릭                          │
│     → Name: 아무거나 입력 (예: hermes)                   │
│     → Limit: 비워두기 (무제한)                           │
│     → 파란 "Create" 버튼 클릭                            │
│                                                         │
│  4. sk-or-v1- 로 시작하는 키가 나타남                    │
│     → 오른쪽 복사 아이콘 클릭해서 복사                   │
│     ⚠️  이 창 닫으면 키를 다시 볼 수 없습니다!           │
│                                                         │
│  💡 카드 등록 없이 무료 모델 바로 사용 가능합니다.        │
│     무료 추천: google/gemini-2.0-flash-001              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "OpenRouter 키를 복사하셨나요?"
선택지:
  - 네, 복사했어요 — 키 입력 단계로
  - 로그인이 안 돼요 — "Google 계정 로그인 버튼을 눌러주세요. 기존 Google 계정 있으면 바로 됩니다." 안내
  - 카드 없이 정말 무료인가요? — "네, 무료 모델은 카드 없이 됩니다. Create Key 후 바로 사용 가능합니다." 안내
  - 다른 방법 원해요 — Anthropic/OpenAI 안내

### Anthropic 선택 시

```bash
open "https://console.anthropic.com/settings/keys" 2>/dev/null || \
xdg-open "https://console.anthropic.com/settings/keys" 2>/dev/null || \
powershell.exe /c start "https://console.anthropic.com/settings/keys" 2>/dev/null || true
```

```
┌─────────────────────────────────────────────────────────┐
│  Anthropic API 키 발급                                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. 브라우저에서 로그인 (없으면 회원가입)                 │
│  2. "Create Key" 클릭                                    │
│  3. sk-ant- 로 시작하는 키 복사                          │
│  ⚠️  유료 (카드 등록 필요, $5부터 충전)                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### OpenAI 선택 시

```bash
open "https://platform.openai.com/api-keys" 2>/dev/null || \
xdg-open "https://platform.openai.com/api-keys" 2>/dev/null || \
powershell.exe /c start "https://platform.openai.com/api-keys" 2>/dev/null || true
```

```
┌─────────────────────────────────────────────────────────┐
│  OpenAI API 키 발급                                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. 브라우저에서 로그인                                   │
│  2. "+ Create new secret key" 클릭                      │
│  3. sk- 로 시작하는 키 복사                              │
│  ⚠️  유료 (카드 등록 필요)                               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

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

먼저 BotFather 링크를 브라우저로 연다 (Telegram 앱이 설치돼 있으면 앱으로 열림):

```bash
open "https://t.me/BotFather" 2>/dev/null || \
xdg-open "https://t.me/BotFather" 2>/dev/null || \
powershell.exe /c start "https://t.me/BotFather" 2>/dev/null || true
```

아래 안내를 출력한다:

```
┌─────────────────────────────────────────────────────────┐
│  [1/2]  봇 만들기 — BotFather 에서 진행                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  BotFather = Telegram 공식 봇 관리자입니다.              │
│  파란 체크(✓)가 있는 계정이 진짜입니다.                  │
│                                                         │
│  채팅창에 아래를 순서대로 입력하세요:                     │
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
│          (이미 있으면 다른 이름으로)                      │
│                                                         │
│  완료되면 아래 형태의 토큰이 나타납니다:                  │
│  1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ                  │
│  → 이 줄 전체를 복사해두세요                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "봇 토큰을 복사하셨나요?"
선택지:
  - 네, 토큰 복사했어요 — 다음 (내 ID 확인 단계)
  - 이미 bot으로 끝나는 아이디가 있대요 — "다른 이름 뒤에 _bot 붙여서 다시 시도해보세요. 예: myhermes2_bot" 안내
  - BotFather를 못 찾겠어요 — "Telegram 검색창에 @BotFather 입력, 파란 체크 있는 것 선택" 재안내

다음으로 userinfobot 링크를 연다:

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
│  방금 열린 @userinfobot 채팅에서:                        │
│                                                         │
│  /start  입력 (또는 Start 버튼 클릭)                     │
│                                                         │
│  그러면 이렇게 나옵니다:                                  │
│  Id: 123456789                                          │
│  First: 홍                                              │
│  Last: 길동                                             │
│                                                         │
│  "Id:" 뒤의 숫자만 복사하세요 (예: 123456789)            │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion으로 두 가지를 순서대로 받는다:
1. "BotFather에서 받은 봇 토큰을 붙여넣어 주세요" (Other로 직접 입력)
2. "@userinfobot에서 확인한 숫자 ID를 입력해주세요" (Other로 직접 입력)

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

Discord 개발자 포털을 바로 연다:

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
│  브라우저에서 Discord 계정으로 로그인 후:                 │
│                                                         │
│  1. 우측 상단 파란 "New Application" 버튼 클릭           │
│  2. 이름 입력 (예: HermEZ)                               │
│  3. 체크박스 체크 → "Create" 클릭                        │
│                                                         │
│  4. 왼쪽 메뉴에서 "Bot" 클릭                             │
│                                                         │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  ⚠️  여기 안 하면 봇이 완전 먹통됩니다!                  │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                         │
│  5. 아래로 스크롤 → "Privileged Gateway Intents" 찾기    │
│     ┌─────────────────────────────────────────────┐    │
│     │  SERVER MEMBERS INTENT        [켜기 ●    ]  │    │
│     │  MESSAGE CONTENT INTENT       [켜기 ●    ]  │    │
│     └─────────────────────────────────────────────┘    │
│     두 개 모두 오른쪽 토글 클릭해서 파란색으로 켜기       │
│                                                         │
│  6. 하단 "Save Changes" 클릭                             │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "Intent를 켰나요?"
선택지:
  - 네, 두 개 다 켰어요 — 다음 (토큰 발급)
  - Privileged Gateway Intents가 안 보여요 — "Bot 메뉴에서 아래로 스크롤하면 나옵니다. TOKEN 섹션 아래에 있어요." 안내

```
┌─────────────────────────────────────────────────────────┐
│  [2/3]  봇 토큰 발급 + 서버 초대                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  토큰 발급:                                              │
│  1. Bot 메뉴 상단 TOKEN 섹션에서 "Reset Token" 클릭      │
│  2. "Yes, do it!" → 토큰 복사 (MTxxx... 형태)           │
│     ⚠️  이 페이지 벗어나면 다시 볼 수 없습니다!          │
│                                                         │
│  서버 초대 URL 만들기:                                   │
│  3. 왼쪽 메뉴 "OAuth2" → "URL Generator" 클릭           │
│  4. SCOPES 에서 "bot" 체크                               │
│  5. BOT PERMISSIONS 에서 아래 4개 체크:                  │
│     ☑ View Channels                                     │
│     ☑ Send Messages                                     │
│     ☑ Read Message History                              │
│     ☑ Attach Files                                      │
│  6. 하단에 생성된 URL 복사                               │
│  7. 새 탭에서 URL 열기 → 내 서버 선택 → "승인" 클릭      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion: "토큰을 복사하고 봇을 서버에 초대했나요?"
선택지:
  - 네, 완료했어요 — 다음 (내 Discord ID)
  - 서버가 없어요 — "Discord에서 + 버튼으로 새 서버 만든 후 초대하세요" 안내

```
┌─────────────────────────────────────────────────────────┐
│  [3/3]  내 Discord ID 확인                               │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Discord 앱(또는 웹) 열기                             │
│  2. 왼쪽 하단 톱니바퀴(⚙) 클릭 → 설정 열기              │
│  3. 왼쪽 메뉴 "고급" 클릭                                │
│  4. "개발자 모드" 토글 ON으로 켜기                       │
│  5. 설정 닫기                                            │
│  6. 좌측 하단 내 아이콘 우클릭 → "사용자 ID 복사"        │
│     숫자 18자리가 복사됩니다 (예: 123456789012345678)    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

AskUserQuestion으로 두 가지를 순서대로 받는다:
1. "Discord 봇 토큰을 붙여넣어 주세요" (Other로 직접 입력)
2. "내 Discord 사용자 ID(숫자)를 입력해주세요" (Other로 직접 입력)

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
║       github.com/Hybirdss/HermEZ                        ║
║                                                          ║
║                           made by hybirdss              ║
╚══════════════════════════════════════════════════════════╝
```

배너 출력 직후, OS에 맞는 명령으로 브라우저를 자동으로 연다:

```bash
open "https://github.com/Hybirdss/HermEZ" 2>/dev/null || \
xdg-open "https://github.com/Hybirdss/HermEZ" 2>/dev/null || \
powershell.exe /c start "https://github.com/Hybirdss/HermEZ" 2>/dev/null || true
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
