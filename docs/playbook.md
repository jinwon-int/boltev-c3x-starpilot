# 대교 (Daegyo) — C3X 자율 관리 플레이북

> **스킬:** `infrastructure/c3x-management`  
> **실행 주체:** Hermes Agent on S23 Ultra (Termux)  
> **대상:** Comma 3X (`tizi-the-pond`), Bolt EV 2017 non-ACC

---

## 1. 상태 점검 (Health Check)

```bash
# 1. Tailscale API로 온라인 확인
python3 -c "
import json, urllib.request
req = urllib.request.Request('https://api.tailscale.com/api/v2/device/4700657545486091')
req.add_header('Authorization', 'Bearer <API_KEY>')
d = json.loads(urllib.request.urlopen(req).read())
print(f'Online: {d[\"lastSeen\"]}')
print(f'Addresses: {d[\"addresses\"]}')
"

# 2. 웹 API로 소프트웨어 정보
curl -s http://100.71.169.100:8082/api/stats | jq .softwareInfo

# 3. SSH 접속 테스트
ssh -o ConnectTimeout=5 -i ~/.ssh/id_ed25519 comma@100.71.169.100 echo OK
```

---

## 2. 업데이트 확인

```bash
# GitHub 최신 커밋 확인
LATEST=$(curl -s https://api.github.com/repos/firestar5683/StarPilot/commits/StarPilot | jq -r .sha[0:7])

# C3X 현재 커밋 확인
CURRENT=$(ssh comma@100.71.169.100 'cd /data/openpilot && git log -1 --format=%h')

echo "Latest: $LATEST, Current: $CURRENT"
if [ "$LATEST" != "$CURRENT" ]; then
  echo "업데이트 필요!"
fi
```

---

## 3. 업데이트 실행

### 표준 방법 (git fetch가 가능할 때)
```bash
ssh comma@100.71.169.100 '
  cd /data/openpilot && \
  git fetch origin StarPilot && \
  git checkout StarPilot && \
  git reset --hard origin/StarPilot
'
```

### 대체 방법 (tarball 우회, 인터넷 느릴 때)
```bash
# Daegyo에서
curl -fsSL "https://api.github.com/repos/firestar5683/StarPilot/tarball/StarPilot" -o ~/starpilot.tar.gz
scp -i ~/.ssh/id_ed25519 ~/starpilot.tar.gz comma@192.168.55.203:/data/

# C3X에서
ssh comma@192.168.55.203 '
  cd /data && rm -rf starpilot_new && tar xzf starpilot.tar.gz && \
  STAR_DIR=$(ls -d firestar5683-StarPilot-*) && \
  rsync -a --delete $STAR_DIR/ /data/openpilot/ && \
  cd /data/openpilot && rm -rf .git && git init && \
  git remote add origin https://github.com/firestar5683/StarPilot
'
```

---

## 4. 사후 검증

```bash
# 업데이트 후 확인
ssh comma@100.71.169.100 '
  echo "=== Version ==="
  cd /data/openpilot && git log -1 --format="%h %s %ai"
  echo "=== Remote ==="
  git remote -v
  echo "=== Branch ==="
  git branch --show-current
'
```

---

## 5. 재부팅 요청 (진원님께)

```
대교: "C3X StarPilot 업데이트 완료했습니다.
      최신 커밋: <sha>, 날짜: <date>
      변경사항: <summary>
      재부팅만 해주세요! (전원 코드 뽑았다 꽂기)"
```

---

## 6. 트러블슈팅

| 증상 | 확인 | 해결 |
|------|------|------|
| SSH 거부 | `curl http://100.71.169.100:8082/api/params?key=SshEnabled` | C3X 설정에서 Enable SSH 재확인 |
| C3X 오프라인 | Tailscale API `lastSeen` 확인 | 차량 시동 ON 확인 |
| git fetch 타임아웃 | 직접 C3X에서 실행 | tarball 우회 방법 사용 |
| IP 변경 | `nmap -p 8082 --open 192.168.55.0/24` | 새 IP로 업데이트 |
| GitHub 키 인증 실패 | `https://github.com/jinon86.keys` 확인 | 진원님께 C3X SSH Keys 필드에 jinon86 재입력 요청 |

---

## 7. 안전 규칙

- ⛔ `reboot` 명령어 직접 실행 불가 (Hermes hardline blocklist)
- ⛔ `rm -rf /data/openpilot` 절대 금지 (항상 rsync --delete 사용)
- ✅ 모든 변경 전 현재 상태를 이 레포의 `config/` 에 저장
- ✅ 업데이트 로그를 `updates/YYYY-MM-DD.md` 로 기록
- 🔑 Golden Safety Rule: 설정 변경 시 10% 이하만 조정
