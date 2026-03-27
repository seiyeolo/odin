# OpenClaw 설치 & 시스템 프롬프트 설정 트러블슈팅 가이드

> 작성일: 2026-03-27
> 작성자: seiyeolo
> 대상: OpenClaw 수강생

---

## 목표

텔레그램 봇(@Kkamaing_bot)이 **내가 원하는 정체성(까마잉)**으로 응답하게 만들기
= KB 파일 내용을 AI의 **시스템 프롬프트**로 주입하기

---

## 문제 1. systemPrompt 키를 어디에 넣어야 하는가?

### 시도한 방법들 (모두 실패)

**시도 1: `agents.defaults.systemPrompt`**
```json
"agents": {
  "defaults": {
    "systemPrompt": "나는 까마잉이야..."
  }
}
```
→ 결과: `Unrecognized key: "systemPrompt"` — 게이트웨이 실행 거부

**시도 2: `channels.telegram.dms.307874469.systemPrompt`**
```json
"channels": {
  "telegram": {
    "dms": {
      "307874469": { "systemPrompt": "나는 까마잉이야..." }
    }
  }
}
```
→ 결과: `Unrecognized key: "systemPrompt"` — 텔레그램의 `dms` 스키마에는 `historyLimit`만 허용됨

### 정답: `channels.telegram.direct.{user_id}.systemPrompt`

```json
"channels": {
  "telegram": {
    "direct": {
      "307874469": {
        "systemPrompt": "나는 까마잉이야. seiyeolo의 개인 AI 에이전트..."
      }
    }
  }
}
```

### 핵심 개념

| 키 | 설명 |
|---|---|
| `channels.telegram.dms` | 대화 기록 설정 (historyLimit만 가능) |
| `channels.telegram.direct` | **DM 상세 설정** — systemPrompt 여기에! |
| `307874469` | 텔레그램 사용자 ID (봇 토큰과 다름!) |

### 사용자 ID vs 봇 토큰

- `8134853884:AAF...` → **봇(Kkamaing_bot)의 토큰** (봇 인증용)
- `307874469` → **나(사용자)의 텔레그램 ID** (누구와 대화하는지 구분용)

페어링 화면에서 확인 가능:
```
Your Telegram user id: 307874469
```

### config 수정 방법 (Python 스크립트)

GCP VM 터미널에서 실행:

```python
# /tmp/set_prompt.py 로 저장 후 실행
import json

config_path = '/home/seiyeolo/.openclaw/openclaw.json'
with open(config_path) as f:
    d = json.load(f)

system_prompt = """나는 까마잉(Kkamaing)이야. seiyeolo의 개인 AI 에이전트.
(여기에 원하는 내용 작성)"""

if 'direct' not in d['channels']['telegram']:
    d['channels']['telegram']['direct'] = {}

d['channels']['telegram']['direct']['여기에_본인_텔레그램_ID'] = {
    'systemPrompt': system_prompt
}

with open(config_path, 'w') as f:
    json.dump(d, f, indent=2, ensure_ascii=False)

print('완료!')
```

설정 후 반드시 검증 및 재시작:
```bash
openclaw config validate
systemctl --user restart openclaw-gateway.service
```

---

## 문제 2. 포트 충돌 (Port 18789 Already in Use)

### 증상

```
Gateway failed to start: another gateway instance is already listening on ws://127.0.0.1:18789
```

게이트웨이가 계속 재시작을 반복하며 정상 실행이 안 됨

### 원인

GCP VM을 여러 수강생이 공유하거나, 이전 세션의 프로세스가 살아있는 경우
→ **다른 UID의 프로세스**가 18789 포트를 점유 중

### 확인 방법

```bash
ps aux | grep openclaw | grep -v grep
# Uid 1000 (다른 사용자) 프로세스가 보이면 kill 불가
```

### 해결 방법: 포트 변경

**1단계: config 파일에서 포트 변경**
```bash
openclaw config set gateway.port 18790
```

**2단계: systemd 서비스 파일 수정**
```bash
# 포트 변경
sed -i 's/--port 18789/--port 18790/g' ~/.config/systemd/user/openclaw-gateway.service
sed -i 's/OPENCLAW_GATEWAY_PORT=18789/OPENCLAW_GATEWAY_PORT=18790/g' ~/.config/systemd/user/openclaw-gateway.service

# 적용 및 재시작
systemctl --user daemon-reload
systemctl --user restart openclaw-gateway.service
```

> ⚠️ config 파일만 바꾸면 안 됩니다. **서비스 파일도 함께** 수정해야 합니다.

---

## 문제 3. CLI가 게이트웨이에 연결 안 됨 (Token Mismatch)

### 증상

```
gateway token mismatch (set gateway.remote.token to match gateway.auth.token)
```

### 해결 방법

```bash
# gateway.auth.token 값 확인
cat ~/.openclaw/openclaw.json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['gateway']['auth']['token'])"

# remote token 동기화
openclaw config set gateway.remote.token 여기에_위에서_확인한_토큰값

# 재시작
systemctl --user restart openclaw-gateway.service
```

---

## 문제 4. 텔레그램 페어링 코드 만료

### 증상

```
No pending pairing request found for code: H4UBVNMH
```

### 원인

페어링 코드는 **일정 시간이 지나면 만료**됨

### 해결 방법

1. 텔레그램에서 봇에게 **메시지를 새로 전송** → 새 코드 발급
2. 새 코드를 즉시 승인:

```bash
openclaw pairing approve telegram 새코드
# 예: openclaw pairing approve telegram LP9YY5AR
```

3. 승인 확인:
```bash
openclaw pairing list      # 대기 중인 요청
openclaw devices list      # 승인된 기기 목록
```

---

## 전체 흐름 요약

```
1. KB 파일 작성 (IDENTITY.md, USER.md, SOUL.md 등)
      ↓
2. openclaw.json에 시스템 프롬프트 설정
   channels.telegram.direct.{내 텔레그램 ID}.systemPrompt
      ↓
3. config validate → 오류 없으면 게이트웨이 재시작
      ↓
4. 텔레그램에서 봇에게 메시지 → 페어링 코드 발급
      ↓
5. openclaw pairing approve telegram {코드} 로 승인
      ↓
6. 봇이 내가 설정한 정체성으로 응답 ✅
```

---

## 최종 작동 확인된 openclaw.json 구조

```json
{
  "agents": {
    "defaults": {
      "model": { "primary": "openai/gpt-5.4" },
      "workspace": "/home/seiyeolo/odin"
    }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "dmPolicy": "pairing",
      "botToken": "봇토큰",
      "direct": {
        "307874469": {
          "systemPrompt": "나는 까마잉이야..."
        }
      }
    }
  },
  "gateway": {
    "port": 18790
  }
}
```

---

## 참고: 유용한 명령어 모음

```bash
# 게이트웨이 상태 확인
systemctl --user status openclaw-gateway.service

# 게이트웨이 로그 확인
journalctl --user -u openclaw-gateway.service -n 30 --no-pager

# config 유효성 검사
openclaw config validate

# 특정 값 확인
openclaw config get channels.telegram

# 채널 상태 확인
openclaw channels list

# 페어링 기기 목록
openclaw devices list
```
