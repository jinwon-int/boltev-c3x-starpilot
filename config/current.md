# Current C3X Configuration Snapshot

> **live-check:** 2026-08-31 KST, Daegyo → Tailscale SSH<br>
> **대상:** `tizi-the-galaxy` / Comma 3X `comma-dbba2a27`<br>
> **경계:** 이 문서는 read-only snapshot입니다. parked CAN gate 전 차량 준비 완료를 주장하지 않습니다.

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
| Overlay | forced-fingerprint cache patch 활성화 완료 |
| Overlay result hashes | `car_params_cache.py=866fc934…`, `card.py=5b9d386e…`, `test_card_cache.py=98a392f8…` |
| Driving model | `rdf43` / Regret Driven Framework V4 / `v15` |
| Stored inactive models | `pop223` / Pop v2, `sc23` / SC Driving |

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
| `/data` | 약 16 GiB / 89 GiB, 19% 사용 |
| routes | `/data/media/0/realdata` 아래 86 directories 관측 |

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

## First ignition result and current parked gate

2026-08-27 first ignition에서 다음을 확인했습니다.

1. runtime fingerprint `CHEVROLET_BOLT_CC_2017`
2. Comma Pedal 인식과 `GMPedalLongitudinal`
3. `openpilotLongitudinalControl=true`, `networkLocation=fwdCamera`
4. longitudinal actuator delay `0.6`와 exact-source expected PID
5. manager/camera/logging/storage/thermal 정상

동시에 세 번의 startup에서 Panda `interruptRateCan2`, logical bus 1 RX 0/error-passive, `commIssue`가 반복돼 gate는 fail closed가 됐습니다. 검토된 forced-fingerprint cache patch는 2026-08-27 off-vehicle 배포·재부팅 후 reviewed file hash와 일치했고 cache guard 6개 및 standalone Panda no-fault를 통과했습니다.

2026-08-31 off-vehicle readback은 `IsOffroad=1`, `IsOnroad=0`, `IsEngaged=0`, `CarParamsPersistent` present(1,808 bytes)입니다. manager startup에서 clear되는 `CarParams`와 `FirmwareQueryDone`은 현재 absent이므로, 이 두 transient key만으로 first ignition 수행 여부를 되돌려 판정하지 않습니다.

남은 gate는 차량 P단에서 patch 적용 후 Panda fault·bus health를 다시 확인하는 **parked-in-vehicle CAN validation**입니다. 이를 통과하고 별도 승인을 받기 전에는 engage 또는 저속 canary를 수행하지 않습니다.
