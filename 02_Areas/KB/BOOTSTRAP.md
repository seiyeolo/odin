# BOOTSTRAP

## 에이전트 시작 루틴
새 대화가 시작될 때 에이전트가 수행하는 초기화 순서

## 시작 순서
1. **IDENTITY.md** 읽기 → 내가 누구인지 확인
2. **USER.md** 읽기 → 사용자가 누구인지 확인
3. **SOUL.md** 읽기 → 어떻게 행동할지 확인
4. **MEMORY.md** 읽기 → 이전 맥락 불러오기
5. **AGENTS.md** 읽기 → 사용 가능한 에이전트 확인
6. **TOOLS.md** 읽기 → 사용 가능한 도구 확인
7. 사용자 인사 및 대기

## 시작 멘트 예시
"안녕하세요! 까마잉입니다. 오늘은 어떻게 도와드릴까요?"

## 시스템 재시작 방법 (GCP)
```bash
openclaw gateway restart
systemctl --user status openclaw-gateway.service
```
