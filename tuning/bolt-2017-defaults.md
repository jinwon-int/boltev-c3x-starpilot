# Bolt EV 2017 non-ACC — Approved Profile and Safety Reference

> **live-check:** 2026-08-25 KST<br>
> **차량:** `CHEVROLET_BOLT_CC_2017`, Comma Pedal<br>
> **주의:** 아래는 Params에 투영된 승인 profile입니다. hardware-derived 값은 first ignition 뒤 별도 검증합니다.

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

`/data/params/d`와 `/cache/params/d`의 위 key들은 2026-08-25 live parity를 통과했습니다.

## First ignition expected values

현재 source exact `28ec3ccb…`와 Bolt focused evidence가 기대하는 실차 값:

- fingerprint: `CHEVROLET_BOLT_CC_2017`
- Pedal interceptor: present
- `openpilotLongitudinalControl=true`
- `networkLocation=fwdCamera`
- longitudinal actuator delay: `0.6`
- candidate의 새 PID values

이 값들은 Params 설정만으로 증명되지 않습니다. `CarParams`와 firmware query가 생성된 뒤 decode해 확인합니다.

## Safety limits

| 한계 | 운영 원칙 |
|---|---|
| 저속 조향 | 약 7 mph 이하 제한 가능 |
| 제동 | 회생제동 중심, 최대 제동 한계에서 수동 브레이크 필요 |
| Stop sign | 감지 불안정/rolling-stop 가능성에 대비 |
| Non-ACC + Pedal | pedal 인식 실패 시 OP longitudinal 검증 중단 |
| 첫 주행 | owner-driven 저속, 즉시 수동 제동 준비 |

## Change policy

- 한 번에 하나의 논리 설정 묶음만 변경
- current snapshot/backup 후 변경
- 설정 store parity와 offroad postcondition 확인
- driving model/tuning 추천은 moving upstream이므로 자동 적용 금지
- PID, delay, friction, torque, safety, firmware 변경은 단순 “10% 규칙”으로 승인하지 않고 exact source/hardware evidence와 별도 owner approval을 요구

과거 2026-06 기본 권장값(Sport, ForceStandstill ON, stop 8s)은 history이며 현재 승인 profile과 다릅니다. 현행 판단에는 이 문서와 [`../config/current.md`](../config/current.md)를 우선합니다.
