# Daegyo C3X Management Playbook

> **대상:** `tizi-the-galaxy` / Bolt EV 2017 non-ACC / Comma Pedal<br>
> **원칙:** read-only는 증거 수집, mutation은 fresh approval + backup + exact target

## 1. Reachability and identity

```bash
C3X=comma@tizi-the-galaxy.tail1546e7.ts.net
KEY=~/.ssh/id_ed25519

ssh -i "$KEY" -o IdentitiesOnly=yes -o BatchMode=yes -o ConnectTimeout=8 \
  "$C3X" 'id -un; hostname; cat /VERSION'
```

현행 endpoint는 [`ssh-access.md`](ssh-access.md)를 따른다. `tizi-the-pond`는 history다.

## 2. Routine read-only health

```bash
ssh -i "$KEY" -o IdentitiesOnly=yes "$C3X" '
  git -C /data/openpilot rev-parse HEAD HEAD^{tree}
  git -C /data/openpilot branch --show-current
  git -C /data/openpilot status --short
  systemctl is-active comma.service ssh.socket tailscaled.service
  df -h /data
'
```

판정:

- exact deployed SHA/tree와 immutable pin branch 일치
- `comma`, SSH, Tailscale active
- approved forced-fingerprint overlay 3 files와 active-theme 6 entries 외 새 source drift 없음
- disk/temperature/crash/tombstone에 blocking 신호 없음
- secret-bearing Params와 route body는 출력하지 않음

## 3. Parked-in-vehicle CAN gate

First ignition의 fingerprint·Pedal·OP longitudinal·delay/PID 확인은 완료됐지만 Panda `interruptRateCan2`가 반복됐습니다. reviewed forced-fingerprint cache patch 배포 후 차량 연결 검증은 아직 수행하지 않았습니다.

owner가 차량을 P단·안전 장소에 두고 별도 승인한 window에서 다음을 read-only로 확인합니다.

1. `CarParamsPersistent`가 forced `CHEVROLET_BOLT_CC_2017`과 일치하며 cache guard가 선택됨
2. Pedal interceptor와 `GMPedalLongitudinal`
3. `openpilotLongitudinalControl=true`, `networkLocation=fwdCamera`, delay `0.6`와 expected PID
4. Panda `faults`/`faultStatus`, CAN bus RX/error-passive/error counters
5. manager/UI/camera health와 blocking alert
6. P단·0 km/h·`controlsAllowed=false`; 검증 종료 뒤 `IsOffroad=1`, `IsOnroad=0`, `IsEngaged=0`

fault가 재현되면 반복 reboot나 engage를 하지 않고 route/log evidence를 보존합니다. 이 gate를 통과한 뒤에도 owner-driven 저속 canary는 별도 승인이 필요합니다. 원격에서 차량 engage/조향/가감속을 시도하지 않습니다.

## 4. Settings audit

`/data/params/d`와 `/cache/params/d`를 함께 읽고 [`../config/current.md`](../config/current.md)의 key/value와 비교한다. symlink를 따르기 위해 `find -L`을 사용한다. 설정 변경은 한 번에 하나의 논리 묶음으로 하고, 변경 전 owner-only backup을 만든다.

## 5. Upstream review

moving `StarPilot` branch는 조회만 한다. 적용 후보는 exact SHA/tree를 고정하고 GM/Bolt/Pedal/longitudinal/Panda safety diff와 focused tests를 검토한다. 승인 없는 fetch-to-active, checkout, Params write, firmware flash, restart, reboot는 금지한다.

상세 절차: [`update-procedures.md`](update-procedures.md).

## 6. Evidence and repository update

실기 변경 또는 중요한 검증 뒤:

1. `config/current.md`를 live evidence로 갱신
2. `updates/YYYY-MM-DD-<topic>.md`에 시점 기록
3. PR-first로 CI/review/merge
4. 미완료 gate는 issue에 fail-closed로 남김

## 7. Hard boundaries

- 원격 주행·engage·조향·가감속 금지
- owner 승인 없는 reboot/service restart 금지
- `/data/openpilot` 삭제·broad reset/clean 금지
- moving branch 직접 적용 금지
- firmware/AGNOS/Params mutation은 별도 승인
- route/video 삭제·전송은 별도 승인
