# C3X SSH Access Guide

> **관리:** Daegyo / S23 Ultra Termux<br>
> **대상:** `tizi-the-galaxy`, Bolt EV 2017 non-ACC

## Current endpoint

| 항목 | 값 |
|---|---|
| SSH user | `comma` |
| Tailscale IPv4 | `100.90.4.121` |
| MagicDNS | `tizi-the-galaxy.tail1546e7.ts.net` |
| Local Wi-Fi | `192.168.55.222` (DHCP, mutable) |
| Port | `22` |
| Daegyo key | `~/.ssh/id_ed25519` |
| Public-key label | `daegyo-s23-for-c3x` |
| Live host-key | ED25519 `SHA256:kxVkcmAn96V4ezKlV2mefv3TQbqHnREFmThp2zvo4ls` |

## Connection

```bash
ssh -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes \
  comma@tizi-the-galaxy.tail1546e7.ts.net

# Current-IP fallback
ssh -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes comma@100.90.4.121

# Same-LAN transfer; first live-check DHCP address
scp -i ~/.ssh/id_ed25519 <file> comma@192.168.55.222:/data/
```

`StrictHostKeyChecking=no`를 상시 사용하지 않습니다. 포맷·OS 교체 뒤 host key가 바뀌면 실기 fingerprint와 대조한 후 known_hosts를 갱신합니다.

## Key provisioning

C3X Settings → Network의 SSH Keys에 GitHub user `jinon86`을 지정하면 `https://github.com/jinon86.keys`에서 공개키를 가져옵니다. private key는 Daegyo에만 보관하며 레포·Wiki·로그로 복사하지 않습니다.

접속이 거부되면 다음 순서로 확인합니다.

1. `tailscale ping tizi-the-galaxy` 또는 peer online 상태
2. TCP 22와 SSH banner
3. C3X `Enable SSH` 및 `ssh.socket`
4. GitHub SSH Keys의 `jinon86` 재동기화
5. Daegyo key fingerprint가 C3X용 공개키와 일치하는지 확인

## History

`100.71.169.100` / `tizi-the-pond`는 포맷 전 identity입니다. 복구 후 persistent state가 없어서 새 identity `tizi-the-galaxy`가 생성됐으며, 현행 접속에 old endpoint를 사용하지 않습니다. stale old Tailscale node 삭제는 별도 승인 작업입니다.

Tailscale SSH가 아니라 일반 OpenSSH(port 22)를 사용합니다. `comma` 계정 권한을 root로 간주하지 않습니다.
