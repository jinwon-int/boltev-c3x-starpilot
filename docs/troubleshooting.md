# C3X Troubleshooting Guide

> **관리:** 대교 (Daegyo) on S23 Ultra (Termux)

---

## 1. C3X 접속 불가 (Unreachable)

### 증상
- SSH 연결 타임아웃
- 웹 UI 로딩 안 됨
- ping 응답 없음

### 진단 순서

```bash
# Step 1: Tailscale API로 기기 상태 확인
python3 -c "
import json, urllib.request
req = urllib.request.Request('https://api.tailscale.com/api/v2/device/4700657545486091')
req.add_header('Authorization', 'Bearer <API_KEY>')
d = json.loads(urllib.request.urlopen(req).read())
print(f\"lastSeen: {d['lastSeen']}\")
print(f\"connected: {d['connectedToControl']}\")
"
```

### 원인별 해결

| 원인 | 확인 방법 | 해결 |
|------|-----------|------|
| **C3X 전원 OFF** | Tailscale API `lastSeen`이 오래됨 | 차량 시동 ON |
| **OBD-II 전원 차단** | C3X 화면이 꺼져 있음 | 차량 시동 ON (OBD-II 포트가 전원 공급) |
| **IP 변경** | `lastSeen`은 최신이지만 접속 안 됨 | `nmap -p 8082 --open 192.168.55.0/24` 스캔 |
| **WiFi 변경** | C3X가 새 네트워크에 연결됨 | C3X 터치스크린에서 WiFi 재설정 |
| **Tailscale 연결 끊김** | API `connectedToControl: false` | C3X 화면에서 Tailscale 재연결 |

### IP 스캔 명령어

```bash
# 로컬 네트워크에서 C3X 찾기 (8082 포트)
nmap -p 8082 --open 192.168.55.0/24

# 더 빠른 스캔 (상위 20개 호스트만)
nmap -p 8082 --open 192.168.55.1-254 --host-timeout 2s
```

---

## 2. SSH Permission Denied

### 증상
```
Permission denied (publickey).
```
또는
```
ssh: connect to host 100.71.169.100 port 22: Connection refused
```

### 진단 순서

```bash
# Step 1: SSH 서비스 활성화 여부
curl -s http://100.71.169.100:8082/api/params?key=SshEnabled
# "true" 여야 함

# Step 2: GitHub 키 확인
curl -s https://github.com/jinon86.keys
# daegyo 키가 목록에 있어야 함

# Step 3: 디버그 모드로 접속 시도
ssh -vvv -i ~/.ssh/id_ed25519 comma@100.71.169.100
```

### 원인별 해결

| 원인 | 해결 |
|------|------|
| **SSH 비활성화** | C3X Settings → Network → Enable SSH 토글 ON |
| **GitHub 키 없음** | https://github.com/settings/keys 에 daegyo 키 등록 |
| **C3X 키 캐시 오래됨** | 진원님께 C3X SSH Keys 필드 `jinon86` 삭제 후 재입력 요청 |
| **포트 22 차단** | C3X가 신뢰할 수 있는 네트워크인지 확인 |
| **C3X 부팅 중** | 1~2분 대기 후 재시도 (SSH 데몬 시작 시간) |

---

## 3. Git Fetch Timeout

### 증상
```
fatal: unable to access 'https://github.com/firestar5683/StarPilot/':
  Failed to connect to github.com port 443: Connection timed out
```

### 원인
C3X의 차량 셀룰러/WiFi 인터넷이 느리거나 불안정함

### 해결: Tarball 우회 방법

```bash
# 1. Daegyo에서 GitHub tarball 다운로드 (빠른 인터넷)
curl -fsSL "https://api.github.com/repos/firestar5683/StarPilot/tarball/StarPilot" \
  -o ~/starpilot.tar.gz

# 2. 로컬 WiFi로 C3X에 전송 (Tailscale보다 빠름)
scp -i ~/.ssh/id_ed25519 ~/starpilot.tar.gz comma@192.168.55.203:/data/

# 3. C3X에서 압축 해제 및 적용
ssh comma@192.168.55.203 '
  cd /data && \
  rm -rf starpilot_new && \
  tar xzf starpilot.tar.gz && \
  STAR_DIR=$(ls -d firestar5683-StarPilot-*) && \
  rsync -a --delete $STAR_DIR/ /data/openpilot/ && \
  cd /data/openpilot && rm -rf .git && git init && \
  git remote add origin https://github.com/firestar5683/StarPilot && \
  rm /data/starpilot.tar.gz
'
```

> 💡 **로컬 WiFi IP (`192.168.55.203`)를 사용하세요.** Tailscale IP보다 직접 연결이 훨씬 빠릅니다.

---

## 4. 웹 UI 접속 가능하지만 SSH 안 됨

### 증상
- `http://100.71.169.100:8082` 는 열리는데
- `ssh comma@100.71.169.100` 는 Connection refused

### 해결
1. SSH 데몬 기다리기 — 부팅 후 1~2분 소요
2. C3X Settings → Network → Enable SSH 껐다 켜기
3. 방화벽 확인 — C3X가 신뢰하는 네트워크인지

---

## 5. Webhook / API 오작동

### 증상
- `curl http://100.71.169.100:8082/api/stats` 응답 없음
- API 엔드포인트 404

### 확인
```bash
# 기본 연결 확인
curl -s -o /dev/null -w "%{http_code}" http://100.71.169.100:8082/

# API 목록 확인 (가능한 경우)
curl -s http://100.71.169.100:8082/api/
```

---

## 6. C3X Web UI SPA가 POST를 가로챔

### 증상
- `curl -X POST http://.../api/reboot` → HTML 응답
- API를 통한 reboot 불가

### 설명
C3X의 Single Page Application이 모든 POST 요청을 가로채서 HTML로 응답합니다.
`reboot` 명령어는 SSH나 API를 통한 실행이 불가능합니다.

### 해결
- 진원님께 직접 재부팅 요청 (전원 코드 뽑았다 꽂기)
- 또는 차량 시동 OFF → ON
