# Current C3X Configuration Snapshot

> **live-check:** 2026-08-25 KST, Daegyo → Tailscale SSH<br>
> **대상:** `tizi-the-galaxy` / Comma 3X `comma-dbba2a27`<br>
> **경계:** 이 문서는 read-only snapshot입니다. first ignition 실증 전 차량 준비 완료를 주장하지 않습니다.

## 소프트웨어

| 항목 | 현행값 |
|---|---|
| Fork | `firestar5683/StarPilot` |
| HEAD | `28ec3ccb80ff46fc88adbdf48e7b4a40c6afeede` |
| Tree | `cd13b2453cb3f46cf0573e440a4af663a09ae74b` |
| Branch | `bolt-starpilot-28ec3ccb` |
| Origin | `file:///data/starpilot-pins/28ec3ccb.git` (immutable local pin) |
| Upstream | GitHub `firestar5683/StarPilot` (조회용) |
| AGNOS | `19.6.2` |
| Overlay | exact candidate 활성화 완료 |

`starpilot/assets/active_theme/` 아래 5개 tracked directory 수정과 `steering_wheel/` untracked directory는 runtime theme materialization입니다. broad reset/clean 대상으로 취급하지 않습니다.

## 네트워크와 서비스

| 항목 | 현행값 |
|---|---|
| Tailscale identity | `tizi-the-galaxy` |
| Tailscale IPv4 | `100.90.4.121` |
| MagicDNS | `tizi-the-galaxy.tail1546e7.ts.net` |
| Local Wi-Fi | `wlan0=192.168.55.222/24` (DHCP, mutable) |
| SSH | port 22, user `comma`, Daegyo C3X key |
| Web UI | `http://100.90.4.121:8082` |
| Services | `comma.service`, `ssh.socket`, `tailscaled.service` active |
| `/data` | 약 2.9 GiB / 89 GiB, 4% 사용 |
| routes | 복구 후 1개 directory 관측 |

SSH ED25519 host-key fingerprint(live): `SHA256:kxVkcmAn96V4ezKlV2mefv3TQbqHnREFmThp2zvo4ls`.

## 현행 승인 profile

아래 값은 `/data/params/d`와 `/cache/params/d`의 live parity를 확인했습니다.

| 기능 | key | 값 |
|---|---|---|
| Experimental mode | `ExperimentalMode` | `0` |
| Conditional Experimental | `ConditionalExperimental` | `1` |
| Curve trigger | `CECurves` | `0` |
| Lead/slower/stopped | `CELead` / `CESlowerLead` / `CEStoppedLead` | `1 / 1 / 1` |
| Stop prediction | `CEModelStopTime` | `7.7` |
| Curve Speed Controller | `CurveSpeedController` | `1` |
| Accel / Decel | `AccelerationProfile` / `DecelerationProfile` | `0(Standard) / 1(Eco)` |
| Reverse cruise | `ReverseCruise` | `1` |
| Force stop / standstill | `ForceStops` / `ForceStandstill` | `1 / 0` |
| Manual fingerprint | `ForceFingerprint` / `CarModel` | `1 / CHEVROLET_BOLT_CC_2017` |
| AoL / advanced lateral | `AlwaysOnLateral` / `AdvancedLateralTune` | `1 / 1` |
| OP longitudinal | `DisableOpenpilotLongitudinal` | `0` |
| GM Pedal longitudinal | `GMPedalLongitudinal` | `1` |
| NNFF / NNFFLite | `NNFF` / `NNFFLite` | `0 / 0` |
| Turn desires / randomizer | `TurnDesires` / `ModelRandomizer` | `0 / 0` |

복구 당시 owner-only backup: `/data/boltev-recovery/28ec3ccb-firestar-settings-20260824T175925Z`.

## First ignition gate — pending

2026-08-25 live readback에서 다음 키는 모두 absent였습니다.

- `CarParams`
- `CarParamsPersistent`
- `CarParamsPrevRoute`
- `FirmwareQueryDone`
- `IsOffroad` / `IsOnroad` / `IsEngaged`

차량 전원을 켠 뒤 아래를 확인해야 정합성이 완성됩니다.

1. runtime fingerprint `CHEVROLET_BOLT_CC_2017`
2. Comma Pedal 인식과 `GMPedalLongitudinal`
3. `openpilotLongitudinalControl=true`, `networkLocation=fwdCamera`
4. exact source 기대값인 longitudinal actuator delay `0.6`와 새 PID
5. Panda/manager/UI/camera health 및 blocking alert 없음
6. offroad 확인 후 owner-driven 저속 first-drive, 즉시 수동 제동 준비

이 gate 전에는 **소프트웨어 설치 완료**와 **실차 검증 완료**를 구분합니다.
