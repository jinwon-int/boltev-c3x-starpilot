# C3X API Reference

> **Base URL:** `http://100.71.169.100:8082` (Tailscale)  
> **Fallback:** `http://192.168.55.203:8082` (로컬 WiFi)

---

## Web API Quick Reference

모든 API는 인증 없이 로컬 네트워크에서 접근 가능합니다.

### 소프트웨어 정보

```bash
# 전체 소프트웨어 정보
curl -s http://100.71.169.100:8082/api/stats | jq .softwareInfo
```

응답 예시:
```json
{
  "branch": "StarPilot",
  "remote": "https://github.com/firestar5683/StarPilot",
  "commit": "471e944",
  "buildDate": "2026-06-13"
}
```

### 업데이트 상태

```bash
# 현재 업데이트 상태
curl -s http://100.71.169.100:8082/api/params?key=UpdaterState

# 가능한 값:
# - "" (idle)
# - "downloading... XX%"
# - "downloading... 100%" (다운로드 완료, 재부팅 필요)
# - "installing..."
```

### 시스템 파라미터

```bash
# SSH 활성화 상태
curl -s http://100.71.169.100:8082/api/params?key=SshEnabled

# Tailscale 설치 여부
curl -s http://100.71.169.100:8082/api/tailscale/installed

# 기기 ID
curl -s http://100.71.169.100:8082/api/params?key=DongleId
```

### 라이브 로그 (SSE Stream)

```bash
# tmux 로그 실시간 스트리밍
curl -s http://100.71.169.100:8082/api/tmux_log/live
```

---

## Tailscale API Reference

> **Base URL:** `https://api.tailscale.com/api/v2`  
> **Auth:** Bearer token 헤더  
> **Device ID:** `4700657545486091`

### 기기 정보 조회

```bash
curl -s \
  -H "Authorization: Bearer <API_KEY>" \
  https://api.tailscale.com/api/v2/device/4700657545486091 \
  | jq '{
    hostname,
    addresses,
    lastSeen,
    connectedToControl,
    os,
    clientVersion,
    tailnetName
  }'
```

응답 예시:
```json
{
  "hostname": "tizi-the-pond",
  "addresses": ["100.71.169.100", "fd7a:..."],
  "lastSeen": "2026-06-19T12:00:00Z",
  "connectedToControl": true,
  "os": "linux",
  "clientVersion": "1.80.0",
  "tailnetName": "tail1546e7.ts.net"
}
```

### 간편 스크립트 (Python)

```python
import json, urllib.request
api_key = "<API_KEY>"  # 환경변수에서 로드할 것
TAILSCALE_API = "https://api.tailscale.com/api/v2"
DEVICE_ID = "4700657545486091"

def get_device():
    req = urllib.request.Request(f"{TAILSCALE_API}/device/{DEVICE_ID}")
    req.add_header("Authorization", f"Bearer {api_key}")
    return json.loads(urllib.request.urlopen(req).read())

def is_online():
    d = get_device()
    return d["connectedToControl"]

def get_ips():
    d = get_device()
    return d["addresses"]

if __name__ == "__main__":
    d = get_device()
    print(f"Hostname: {d['hostname']}")
    print(f"Online: {'✅' if d['connectedToControl'] else '❌'}")
    print(f"IPs: {', '.join(d['addresses'])}")
    print(f"Last seen: {d['lastSeen']}")
```

---

## GitHub API Reference

> C3X의 업스트림 StarPilot 레포 정보 조회용

### 최신 커밋 확인

```bash
# StarPilot 브랜치 최신 커밋
curl -s https://api.github.com/repos/firestar5683/StarPilot/commits/StarPilot \
  | jq '{sha: .sha[0:7], date: .commit.author.date, message: .commit.message | split("\n")[0], author: .author.login}'
```

### 최근 커밋 목록 (N개)

```bash
curl -s 'https://api.github.com/repos/firestar5683/StarPilot/commits?per_page=10' \
  | jq -r '.[] | "\(.commit.author.date[0:10]) \(.sha[0:7]) \(.commit.message | split("\n")[0]) [\(.author.login)]"'
```

### 릴리스 정보

```bash
curl -s https://api.github.com/repos/firestar5683/StarPilot/releases/latest \
  | jq '{tag: .tag_name, date: .published_at, name: .name}'
```

---

## SSH 원격 명령어

### Git 상태 확인

```bash
ssh comma@100.71.169.100 'cd /data/openpilot && git log -1 --format="%h %s %ai" && git branch --show-current && git remote -v'
```

### 현재 커밋만

```bash
ssh comma@100.71.169.100 'cd /data/openpilot && git log -1 --format="%h"'
```

### 디스크 사용량

```bash
ssh comma@100.71.169.100 'df -h /data'
```

### 프로세스 확인

```bash
ssh comma@100.71.169.100 'ps aux | grep -E "openpilot|tmux" | grep -v grep'
```
