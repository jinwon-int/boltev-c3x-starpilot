# Bolt EV 2017 non-ACC — Approved Profile and Safety Reference

> **live-check:** 2026-08-31 KST<br>
> **차량:** `CHEVROLET_BOLT_CC_2017`, Comma Pedal<br>
> **주의:** 아래는 Params에 투영된 승인 profile입니다. hardware-derived 값은 2026-08-27 first ignition에서 확인했지만 parked CAN gate는 미완료입니다.

## Current profile

| 영역 | 설정/key | 현행값 |
|---|---|---|
| Openpilot | Experimental mode | OFF |
| Openpilot | OP longitudinal | ON (`DisableOpenpilotLongitudinal=0`) |
| Vehicle | Manual fingerprint | ON / `CHEVROLET_BOLT_CC_2017` |
| Vehicle | GM Pedal longitudinal | ON |
| CEM | Conditional Experimental | ON |
| CEM | Curve trigger | OFF |
| CEM | Lead / slower lead / stopped lead | ON / ON / ON |
| CEM | model stop time | `7.7s` |
| Curves | Curve Speed Controller | ON |
| Longitudinal | Acceleration profile | Standard (`0`) |
| Longitudinal | Deceleration profile | Eco (`1`) |
| Cruise | reverse increment | ON; short `+5`, long `+1` |
| Stop behavior | ForceStops / ForceStandstill | ON / OFF |
| Lateral | Always on Lateral / Advanced tune | ON / ON |
| Models | NNFF / NNFFLite / TurnDesires | OFF / OFF / OFF |
| Models | model randomizer | OFF |

`/data/params/d`와 `/cache/params/d`의 위 key들은 2026-08-31 live parity를 통과했습니다.

## First ignition verified values

2026-08-27 first ignition에서 current source exact `28ec3ccb…`와 대조해 확인한 값:

- fingerprint: `CHEVROLET_BOLT_CC_2017`
- Pedal interceptor: present
- `openpilotLongitudinalControl=true`
- `networkLocation=fwdCamera`
- longitudinal actuator delay: `0.6`
- exact-source expected PID values

이 확인은 Panda `interruptRateCan2`가 동시에 재현된 사실을 상쇄하지 않습니다. 현 gate는 patch 적용 후 parked-in-vehicle CAN validation 대기입니다.

## Safety limits

| 한계 | 운영 원칙 |
|---|---|
| 저속 조향 | 공식 Bolt 지침상 7 mph 이하 조향 불가 |
| 제동 | non-ACC + Pedal은 최대 약 70 kW 회생제동, emergency stop 불가·저속 감속 취약; 수동 브레이크 필요 |
| Stop sign | 감지 불안정/rolling-stop 가능성에 대비 |
| Non-ACC + Pedal | pedal 인식 실패 시 OP longitudinal 검증 중단 |
| 첫 주행 | owner-driven 저속, 즉시 수동 제동 준비 |

## Official guidance comparison — 2026-08-31

- [StarPilot settings guide](https://wiki.firestar.link/usage/settings/)의 GM manual fingerprint, Bolt slower/stopped lead ON, curve trigger OFF, NNFF/NNFFLite OFF 권고와 현 profile은 일치합니다.
- 공식 CEM stop-time 권고는 `7s`, 현 owner-approved 값은 `7.7s`입니다. 차이를 기록하되 자동 변경하지 않습니다.
- [Driving Model guide](https://wiki.firestar.link/usage/driving-model/)의 현행 추천은 RDF v4, Pop v2, SC Driving입니다. 선택된 RDF v4는 추천 범위 안이며 나머지 2종은 inactive 보관 상태입니다.
- [Gen1 Bolt Pedal firmware guide](https://wiki.firestar.link/software/pedal-firmware/)는 2025년 6월 이전 구매·미업데이트 pedal을 경고합니다. 현 근거는 2025년 6월 구매 확인이며 firmware version 직접 readback은 아닙니다. 이를 수용된 구매시점 추론으로 취급하고, 물리 flash/검증은 별도 owner approval 없이는 수행하지 않습니다.

## Change policy

- 한 번에 하나의 논리 설정 묶음만 변경
- current snapshot/backup 후 변경
- 설정 store parity와 offroad postcondition 확인
- driving model/tuning 추천은 moving upstream이므로 자동 적용 금지
- PID, delay, friction, torque, safety, firmware 변경은 단순 “10% 규칙”으로 승인하지 않고 exact source/hardware evidence와 별도 owner approval을 요구

과거 2026-06 기본 권장값(Sport, ForceStandstill ON, stop 8s)은 history이며 현재 승인 profile과 다릅니다. 현행 판단에는 이 문서와 [`../config/current.md`](../config/current.md)를 우선합니다.
