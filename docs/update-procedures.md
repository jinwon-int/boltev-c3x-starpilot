# C3X Exact-Pin Update Procedure

> **대상:** `tizi-the-galaxy`, StarPilot, Bolt EV 2017 non-ACC + Comma Pedal<br>
> **현행 baseline:** `28ec3ccb80ff46fc88adbdf48e7b4a40c6afeede` + reviewed cache overlay, AGNOS 19.6.2<br>
> **경계:** update, Params write, firmware, restart, reboot는 각각 fresh approval 범위 안에서만 실행

## Current architecture

Active checkout `/data/openpilot`은 branch `bolt-starpilot-28ec3ccb`에서 self-contained local bare pin `file:///data/starpilot-pins/28ec3ccb.git`을 origin으로 사용합니다. GitHub upstream은 후보 조사용입니다. 현행 3-file forced-fingerprint cache overlay는 base HEAD를 바꾸지 않으므로 patch digest와 result file hashes를 함께 검증해야 합니다.

과거 detached `d931d300` / `UpdaterTargetBranch=HEAD` blocker와 moving tarball 방식은 [`../updates/2026-08-24-audit.md`](../updates/2026-08-24-audit.md)의 history입니다. 현행 절차로 재사용하지 않습니다.

## 1. Read-only preflight

```bash
C3X=comma@tizi-the-galaxy.tail1546e7.ts.net
KEY=~/.ssh/id_ed25519

ssh -i "$KEY" -o IdentitiesOnly=yes "$C3X" '
  cd /data/openpilot
  git rev-parse HEAD HEAD^{tree}
  git branch --show-current
  git remote -v
  git status --short
  sha256sum selfdrive/car/car_params_cache.py selfdrive/car/card.py selfdrive/car/tests/test_card_cache.py
  cat /VERSION
  systemctl is-active comma.service
  df -h /data
'
```

차량 상태 Params가 absent면 offroad라고 추정하지 않습니다. owner가 물리 상태를 확인하고 first ignition gate를 먼저 마칩니다.

## 2. Candidate gate

1. moving branch가 아닌 exact SHA와 tree를 고정하고, 현 overlay의 이식·제거 방침을 명시
2. GM/Bolt/Pedal/longitudinal/Panda safety 전체 diff 요약
3. exact candidate focused tests와 알려진 결함 확인
4. Pedal/AGNOS 요구 버전과 현재 hardware 호환 확인
5. rollback target, artifact, 예상 reboot 횟수 기록
6. owner에게 source update/OS firmware/reboot 범위를 분리해 승인 요청

## 3. Backup gate

승인된 deployment window에서만 owner-only backup을 만듭니다.

- current HEAD/tree/branch/remotes
- tracked diff 및 runtime active-theme archive
- `/data/params`와 필요한 cache settings snapshot
- updater state와 AGNOS slot/version
- manifest SHA-256

backup 경로·mode·manifest를 검증하기 전 active checkout을 변경하지 않습니다. secret-bearing 값은 레포나 PR에 기록하지 않습니다.

## 4. Immutable candidate pin

후보는 C3X 외부 clean checkout에서 fetch·fsck·test한 뒤, C3X에 self-contained bare pin으로 보존합니다. active origin을 moving GitHub branch로 직접 두지 않습니다.

개념적 postcondition:

```text
/data/starpilot-pins/<short-sha>.git  self-contained + fsck clean
/data/openpilot HEAD                 exact approved SHA
/data/openpilot HEAD^{tree}          expected tree
branch                               candidate-specific stable name
origin                               file:///data/starpilot-pins/<short-sha>.git
upstream                             GitHub read-only comparison
```

실행 명령은 후보마다 source layout/updater/AGNOS 요구가 달라지므로 PR/issue에 exact plan을 먼저 기록합니다.

## 5. Activation and reboot

- 차량 parked/offroad/not engaged를 live evidence로 확인
- updater finalization과 AGNOS inactive-slot flash를 별도 postcondition으로 확인
- reboot는 owner의 명시 승인 뒤 한 번씩 수행
- 재접속이 끊기면 임의 추가 reboot 대신 Tailscale/local Wi-Fi/화면 상태로 fail closed

## 6. Post-activation offroad verification

```bash
ssh -i "$KEY" -o IdentitiesOnly=yes "$C3X" '
  git -C /data/openpilot rev-parse HEAD HEAD^{tree}
  git -C /data/openpilot branch --show-current
  cat /VERSION
  systemctl is-active comma.service ssh.socket tailscaled.service
  git -C /data/openpilot status --short
'
```

추가 필수:

- expected fingerprint/Pedal/OP longitudinal/delay/PID
- Panda/manager/UI/camera health
- blocking alerts 없음
- settings stores parity
- backup/rollback intact

## 7. First-drive gate

owner가 저속·안전 장소에서 직접 운전하며 즉시 수동 제동할 준비를 합니다. 원격 agent는 engage, 조향, 가속, 제동을 수행하지 않습니다. 이상 동작 시 즉시 disengage하고 rollback 판단 전 evidence를 보존합니다.

## Prohibited shortcuts

- `git reset --hard` 또는 broad `git clean`으로 runtime assets 제거
- moving `StarPilot` HEAD 즉시 적용
- tarball rsync 후 `.git` 삭제
- approval 없는 updater target 변경·service restart·reboot
- firmware version을 추정해 gate 통과 처리
