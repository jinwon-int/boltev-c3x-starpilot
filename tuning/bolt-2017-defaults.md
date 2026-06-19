# Bolt EV 2017 (non-ACC) — Tuning Reference

> **출처:** [StarPilot Docs — Settings](https://wiki.firestar.link/usage/settings/), [Bolt EV Hardware Guide](https://wiki.firestar.link/cars/bolt/)  
> **최종 업데이트:** 2026-06-19

---

## 🔧 Bolt 2017 Lateral Tuning Defaults

| 파라미터 | 기본값 | 설명 |
|-----------|--------|------|
| Actuator Delay | **0.2** | 변경 시 auto-learn 중단 |
| Friction | **0.05** | 변경 시 auto-learn 중단 |
| Steering Torque | **4.5 Nm** | +50% over stock, 2017 전용 |
| Torque Controller | Custom (D-Term PID) | Bolt-specific |

> ⚠️ **절대 규칙:** delay/friction은 기본값 유지 → auto-learn 활성화  
> ⚠️ **2017 Bolt fingerprint 필수** — 다른 연식 선택 시 4.5Nm 토크 효과 없음

---

## 🎯 Essential Settings Checklist (업데이트마다 확인)

### Openpilot Toggles
| 설정 | 값 | 비고 |
|------|----|------|
| Openpilot | **ON** | |
| Openpilot Longitudinal Control | **ON** | 없으면 무시 |
| Experimental Mode | **OFF** | CEM이 대체 |

### Gas / Brake (FrogPilot)
| 설정 | 값 | 비고 |
|------|----|------|
| Conditional Experimental Mode (CEM) | **ON** | radar-less 차량 필수 |
| Curve Detected Ahead | **OFF** | CSC가 대체 |
| Curve Speed Controller (CSC) | **ON** | 학습형 |
| Slower Lead | **ON** | radar-less |
| Stopped Lead | **ON** | radar-less |
| Navigation Data | **OFF** | |
| openpilot Wants to Stop In | **8s** | 기본값 |
| Status Widget | **ON** | CEM 트리거 표시 |

### Longitudinal Tuning
| 설정 | 값 | 비고 |
|------|----|------|
| Acceleration Profile | **Sport** | |
| Deceleration Profile | **Eco** | |
| Taco Bell Run | **OFF** | |
| Human Like Following | **OFF** | ⚠️ ***필수*** |
| Human Like Acceleration | **OFF** | ⚠️ ***필수*** |

### Quality of Life
| 설정 | 값 | 비고 |
|------|----|------|
| Reverse Cruise Increase | **ON** | short=+5, long=+1 |
| Force Keep Standstill | **ON** | |
| Force Stop Detected | **ON** | |
| Increase Stopped Distance | default | 최종 정지 위치만 조정 |

### Steering
| 설정 | 값 | 비고 |
|------|----|------|
| Advanced Lateral Tuning | **ON** | |
| Force Auto Tune Off | **ON** | |
| Force Auto Tune On | **OFF** | |
| Always on Lateral (AoL) | **ON** | |
| Custom Torque Controller | default | Bolt 기본값 유지 |

### Lane Changes
| 설정 | 값 | 비고 |
|------|----|------|
| Automatic Lane Changes | **OFF** | |
| One Lane Change Per Signal | **ON** | |

### Vehicle Controls
| 설정 | 값 | 비고 |
|------|----|------|
| Auto Fingerprint | **OFF** | ⚠️ ***2017 Bolt 수동 선택*** |

---

## 🚨 Bolt 2017 한계 (Wiki 발췌)

| 한계 | 내용 |
|------|------|
| 저속 스티어링 | 7 mph 이하 조향 불가 |
| 제동 | 회생제동 only, max **70 kW** |
| 긴급 제동 | 70kW 한계 — 수동 브레이크 필요할 수 있음 |
| Stop-and-go | ✅ Comma Pedal로 가능 (ACC 모델은 불가) |
| Stop sign | 감지 불안정, rolling stop 주의 |
| LKAS 버튼 | 작동 안 함 (기능적 문제 없음) |

---

## 🧠 권장 Driving Model

| 우선순위 | 모델 | 비고 |
|----------|------|------|
| 🥇 | **Green Watermelon v8** | 현재 StarPilot 권장 |
| 🥈 | Steam Powered v2 | 대체 모델 |
| 🏷️ | Firehose | FrogPilot upstream 기본 |

> 모델 추천은 변동될 수 있음 — https://wiki.firestar.link/usage/driving-model/ 참조

---

## 🔑 Golden Safety Rule

> **값을 변경할 때는 한 번에 10% 이하만 조정할 것**

모든 튜닝 값은 이 규칙을 따릅니다. 급격한 변경은 위험한 주행 동작을 유발할 수 있습니다.
