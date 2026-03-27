# HEARTBEAT

## 상태 점검 루틴
에이전트가 정상 작동 중인지 확인하는 주기적 체크리스트

## 점검 항목
- [ ] Telegram 채널 응답 가능
- [ ] OpenAI API 연결 정상
- [ ] GCP Gateway 실행 중 (포트 18789)
- [ ] GitHub odin 레포 동기화 상태
- [ ] KB 파일 최신 상태

## 점검 명령어
```bash
# GCP에서 실행
openclaw channels list
openclaw status
cd ~/odin && git status
```

## 마지막 점검
- **날짜**: 2026-03-27
- **결과**: 모든 항목 정상
- **이슈**: 없음
