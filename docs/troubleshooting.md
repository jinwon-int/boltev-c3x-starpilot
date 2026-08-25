# C3X Troubleshooting Guide

> **현행 identity:** `tizi-the-galaxy` / `100.90.4.121`<br>
> **안전:** 진단은 read-only. 차량 전원·firmware·service·Params 변경은 fresh approval.

## 1. Unreachable

```bash
# Tailnet 관리 노드에서
 tailscale status | grep tizi-the-galaxy
 tailscale ping -c 3 tizi-the-galaxy

# Daegyo에서 bounded SSH
ssh -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes -o BatchMode=yes \
  -o ConnectTimeout=8 comma@100.90.4.121 'echo SSH_OK'
```

| 관측 | 해석/다음 단계 |
|---|---|
| peer offline | 차량/C3X 전원과 Wi-Fi를 물리 확인 |
| ping 성공, port 22 timeout | 부팅 중 또는 `ssh.socket` 비활성 |
| port 22 banner, publickey denied | user `comma`, Daegyo C3X key, GitHub key sync 확인 |
| SSH만 되고 Web 8082 down | `comma.service`와 UI 로그 read-only 확인 |
| old endpoint만 시도 | `tizi-the-pond` / `100.71.169.100` 사용 중단 |

현재 local Wi-Fi는 `192.168.55.222`지만 DHCP 값이다. 같은 LAN에서 사용하기 전에 live address를 다시 확인한다.

## 2. Host-key mismatch

포맷·OS 교체 뒤 SSH host key가 바뀔 수 있다. 무조건 우회하지 말고 실기 identity와 live fingerprint를 교차 확인한다.

현행 ED25519 fingerprint(2026-08-25):

```
SHA256:kxVkcmAn96V4ezKlV2mefv3TQbqHnREFmThp2zvo4ls
```

검증 후에만 해당 host의 known_hosts entry를 갱신한다.

## 3. Permission denied

```bash
ssh -vvv -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes comma@100.90.4.121
```

확인 순서:

1. user가 `comma`인지
2. key가 `~/.ssh/id_ed25519` / label `daegyo-s23-for-c3x`인지
3. C3X Settings → Network → Enable SSH
4. GitHub SSH Keys user `jinon86` 동기화
5. `ssh.socket` active 여부

비밀번호 fallback을 자동화·문서화하지 않는다.

## 4. StarPilot/manager unhealthy

```bash
ssh -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes comma@100.90.4.121 '
  systemctl is-active comma.service
  journalctl -u comma.service -n 100 --no-pager
  git -C /data/openpilot rev-parse HEAD HEAD^{tree}
  git -C /data/openpilot status --short
'
```

로그는 bounded하게 읽고 token/DongleId/route body를 기록하지 않는다. 서비스 restart는 자동 복구 절차가 아니라 별도 승인 mutation이다.

## 5. First ignition Params absent

복구 직후 `CarParams`, `CarParamsPersistent`, `FirmwareQueryDone`가 absent인 것은 첫 ignition 전 상태일 수 있다. 파일을 임의 생성·복사하지 않는다. 차량 전원을 켜고 manager가 hardware query를 완료하도록 한 뒤 fingerprint/Pedal/longitudinal/Panda를 확인한다.

잘못된 fingerprint, Pedal 미인식, blocking alert가 있으면 **주행/engage 금지**하고 증거만 수집한다.

## 6. Git/updater failure

active origin은 immutable local pin이다. moving upstream으로 즉시 repoint하거나 `git reset --hard`, tarball rsync, `.git` 삭제를 하지 않는다. 현재 HEAD/tree/pin/backup을 보존하고 [`update-procedures.md`](update-procedures.md)의 exact-candidate gate로 돌아간다.

## 7. Web UI/API mismatch

```bash
curl -fsS --max-time 5 http://100.90.4.121:8082/api/stats | jq .softwareInfo
curl -fsS --max-time 5 'http://100.90.4.121:8082/api/params?key=SshEnabled'
```

POST가 HTML을 반환하면 mutation 성공으로 판정하지 않는다. reboot/update/Params write는 Web UI 추정이 아니라 승인된 절차와 live postcondition으로만 확인한다.
