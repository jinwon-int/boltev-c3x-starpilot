# C3X Read-only API Reference

> **MagicDNS base:** `http://tizi-the-galaxy.tail1546e7.ts.net:8082`<br>
> **Current IPv4:** `http://100.90.4.121:8082`<br>
> **Local fallback:** `http://192.168.55.222:8082` (DHCP, live-check first)

## Web UI / API

```bash
BASE=http://100.90.4.121:8082

# software metadata
curl -fsS "$BASE/api/stats" | jq .softwareInfo

# individual Params
curl -fsS "$BASE/api/params?key=SshEnabled"
curl -fsS "$BASE/api/params?key=UpdaterState"
curl -fsS "$BASE/api/params?key=DongleId"

# live log stream; bound output when using from an agent
curl -fsS --max-time 10 "$BASE/api/tmux_log/live" | head -100
```

Web endpoints are read-only unless a separately reviewed endpoint explicitly mutates state. Do not infer that an HTML response to POST means a reboot/update succeeded.

## SSH readback

```bash
C3X=comma@tizi-the-galaxy.tail1546e7.ts.net
KEY=~/.ssh/id_ed25519

ssh -i "$KEY" -o IdentitiesOnly=yes "$C3X" \
  'git -C /data/openpilot rev-parse HEAD HEAD^{tree}; git -C /data/openpilot branch --show-current'

ssh -i "$KEY" -o IdentitiesOnly=yes "$C3X" \
  'systemctl is-active comma.service ssh.socket tailscaled.service; df -h /data'
```

Params store는 atomic symlink 구조이므로 `find`에는 `-L`을 사용합니다.

```bash
ssh -i "$KEY" -o IdentitiesOnly=yes "$C3X" \
  'find -L /data/params/d -maxdepth 1 -type f -printf "%f\n" | sort'
```

secret-bearing Params, tokens, DongleId 값, route contents를 운영 로그에 원문으로 남기지 않습니다.

## Tailscale status

현행 identity는 포맷 전 API device ID와 다릅니다. old numeric device ID를 재사용하지 말고 tailnet의 live peer를 이름/IP로 찾습니다.

```bash
# Tailnet에 로그인된 관리 노드에서
 tailscale status | grep 'tizi-the-galaxy'
 tailscale ping -c 3 tizi-the-galaxy
```

Tailscale Admin API가 필요하면 credential 위치에서 토큰을 읽되 출력·커밋하지 않고, live device list에서 `tizi-the-galaxy`의 current device ID를 먼저 확인합니다.

## GitHub upstream comparison

```bash
# moving upstream은 조회만; 적용 target은 별도 exact SHA로 고정
curl -fsS https://api.github.com/repos/firestar5683/StarPilot/commits/StarPilot \
  | jq '{sha:.sha,date:.commit.author.date,message:(.commit.message|split("\n")[0])}'

ssh -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes "$C3X" \
  'git -C /data/openpilot rev-parse HEAD'
```

업데이트·Params write·service restart·reboot는 이 read-only reference의 범위 밖이며 fresh approval이 필요합니다.
