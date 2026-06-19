# C3X SSH Access Guide

> **관리:** 대교 (Daegyo) on S23 Ultra (Termux)  
> **대상:** Comma 3X (`tizi-the-pond`), Bolt EV 2017 non-ACC

---

## Credentials

| 항목 | 값 |
|------|-----|
| SSH User | `comma` |
| 인증 방식 | SSH Key (ed25519) |
| 키 경로 (Daegyo) | `~/.ssh/id_ed25519` |
| 키 레이블 | `daegyo-s23-for-c3x` |
| GitHub 사용자 | `jinon86` (C3X가 키를 가져옴) |
| Fallback 비밀번호 | `comma` (비밀번호 인증 활성화 시) |

## Daegyo SSH Key

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP33LgAghnFrrs8ZgibJB3NbfaaATcUK4fpKkhx+9ZoO daegyo-s23-for-c3x
```

- GitHub 등록: `jinon86` 계정 → Settings → SSH Keys
- 공개키 확인: https://github.com/jinon86.keys

---

## Connection Commands

### 기본 접속 (Tailscale — 어디서든)
```bash
ssh -i ~/.ssh/id_ed25519 comma@100.71.169.100
```

### 로컬 WiFi (같은 네트워크, 빠름)
```bash
ssh -i ~/.ssh/id_ed25519 comma@192.168.55.203
```

### 대용량 파일 전송 (로컬 WiFi 권장)
```bash
# Tailscale보다 로컬 WiFi가 훨씬 빠름
scp -i ~/.ssh/id_ed25519 <file> comma@192.168.55.203:/data/
```

### MagicDNS (자동 IP 추적)
```bash
ssh -i ~/.ssh/id_ed25519 comma@tizi-the-pond.tail1546e7.ts.net
```

---

## SSH Key Management

C3X는 GitHub 사용자 프로필에서 SSH 키를 가져옵니다.

### 키 동기화 방식
1. C3X Settings → Network → SSH Keys 필드에 `jinon86` 입력
2. C3X가 `https://github.com/jinon86.keys` 에서 공개키를 가져옴
3. 매칭되는 private key로 인증

### 키 갱신이 필요할 때
새 키를 GitHub에 추가한 후:
1. 진원님께서 C3X Settings → Network → SSH Keys 필드에서 `jinon86` **삭제 후 재입력**
2. 재입력 시 C3X가 GitHub에서 키를 다시 가져옴

---

## SSH가 막혔을 때 확인사항

### C3X Web API로 확인
```bash
# SSH 활성화 여부
curl -s http://100.71.169.100:8082/api/params?key=SshEnabled
# → "true" 여야 함

# Tailscale 설치 여부
curl -s http://100.71.169.100:8082/api/tailscale/installed
```

### C3X Settings 화면에서 확인
1. Settings → Network → **Enable SSH** 토글 ON
2. Settings → Network → **SSH Keys** 필드에 `jinon86` 입력됨

---

## Tailscale API로 기기 상태 확인

```python
import json, urllib.request

api_key = "<TAILSCALE_API_KEY>"
TAILSCALE_API = "https://api.tailscale.com/api/v2"
DEVICE_ID = "4700657545486091"

req = urllib.request.Request(f"{TAILSCALE_API}/device/{DEVICE_ID}")
req.add_header("Authorization", f"Bearer {api_key}")
resp = urllib.request.urlopen(req)
device = json.loads(resp.read())

print(f"Hostname: {device['hostname']}")
print(f"IPs: {device['addresses']}")
print(f"Online: {device['lastSeen']}")
print(f"Connected: {device['connectedToControl']}")
print(f"OS: {device['os']}")
print(f"Client: {device['clientVersion']}")
```

> ⚠️ API 키는 `~/.hermes/.env` 또는 환경변수로 관리, 절대 커밋하지 말 것

---

## 참고

- Tailscale SSH (`tailscale set --ssh`)는 C3X에서 root 필요 → 사용 불가
- 일반 SSH (port 22, C3X Settings → Enable SSH)가 실사용 경로
- `comma` 사용자는 제한된 sudo 권한 보유
- Root 접근 필요 시: `/data/tailscale/tailscale --socket=/data/tailscale/tailscaled.sock`
