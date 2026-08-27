# 2026-08-27 first-drive fail-closed — Panda CAN2 interrupt fault

## 범위와 안전 조치

Owner가 C3X를 2017 Bolt에 연결해 첫 ignition/실주행을 수행했습니다. Daegyo는 원격 read-only telemetry를 확인했고, 장애 확인 뒤 engage 금지와 P단 정차를 요청했습니다.

Owner의 별도 승인으로 C3X reboot 1회, 이어 차량 완전 OFF 약 1분 후 ON 전원 사이클 1회를 수행했습니다. Params·source·firmware·branch·Panda firmware는 변경하지 않았습니다.

## 확인된 정상 항목

- StarPilot exact SHA `28ec3ccb80ff46fc88adbdf48e7b4a40c6afeede`
- AGNOS `19.6.2`
- `CHEVROLET_BOLT_CC_2017`, `networkLocation=fwdCamera`
- Pedal/OP longitudinal profile: actuator delay `0.6`, expected PID, `radarUnavailable=true`
- 차량 CAN main path valid; Panda heartbeat와 safety RX checks 정상
- 필수 manager processes, cameras, logging, storage, thermal status 정상

## 반복 재현된 장애

세 번의 독립 startup(최초 ignition, C3X reboot, 차량 완전 OFF/ON)에서 동일 증상이 재현됐습니다.

- `selfdriveState.engageable=false`, `controlsAllowed=false`
- `commIssue`, `locationdTemporaryError`
- Panda `faultTemp: interruptRateCan2`
- logical bus 1(`OBSTACLE`)은 RX 0, `errorPassive=true`
- 전원 사이클 후 약 67초 표본: `totalErrorCnt=208140`, `canCoreResetCnt=9`, 초기 TX 8
- CAN fingerprint는 bus 0/2에 정상 traffic, bus 1은 empty

C3X UI·camera·manager가 정상으로 보이는 것은 이 low-level Panda fault와 모순되지 않습니다. Onroad event와 Panda health는 UI에 항상 별도 원문으로 노출되지 않으며, 현재는 제어가 fail closed 상태입니다.

## 소스 조사

- GM mapping에서 bus 1은 `CanBus.OBSTACLE`입니다.
- 현행 CarParams는 `radarUnavailable=true`라 controller의 지속 ADAS status 전송은 차단됩니다.
- 매 startup의 `CarParamsCache`는 `CLEAR_ON_MANAGER_START`라 live VIN/FW query가 다시 실행됩니다.
- live query 뒤 bus 1은 초기 TX 8과 함께 ACK 없는 error-passive 상태가 됩니다.
- 공개 [commaai/openpilot#38605](https://github.com/commaai/openpilot/issues/38605)에도 Bolt의 동일 `interruptRateCan2` 관측이 있습니다.
- 그 이슈가 참조한 SPI turnaround fix [commaai/openpilot#38464](https://github.com/commaai/openpilot/pull/38464)는 현행 pin의 `selfdrive/pandad/spi.cc`에 이미 포함돼 있습니다.

따라서 **물리 배선 불량은 미확정**입니다. camera-harness에서 비어 있는 obstacle/OBD path에 startup diagnostics가 전송되는 software/topology interaction, Panda CAN transceiver, harness 연결을 순서대로 분리해야 합니다.

## 후보 수정

[`patches/28ec3ccb-forced-fingerprint-cache.patch`](../patches/28ec3ccb-forced-fingerprint-cache.patch)는 강제 fingerprint가 활성화된 검증 완료 차량에서만 persistent CarParams를 startup cache로 승격합니다.

안전 조건:

- 현재 `CarParamsCache`가 있으면 기존 경로 우선
- `ForceFingerprint=1` 필수
- `CarModel`과 persistent fingerprint exact match
- brand가 `mock`이 아님
- firmware evidence 1개 이상
- 유효한 17자 VIN
- parse/mismatch/incomplete는 기존 live query로 fail closed

후보 테스트에는 cache 선택 truth table과, cached fingerprint 경로가 diagnostic CAN frame을 보내지 않는다는 검증이 포함됩니다.

## 현재 판정

- 첫 ignition/첫 주행 gate: **FAIL, fail-closed**
- StarPilot engage: **금지**
- 후보 패치: **실차 미배포**
- 다음 gate: exact-base patch CI → 독립 리뷰 → 새 immutable candidate → owner-approved parked deployment → Panda fault/CAN health 검증 → 별도 저속 canary 승인
