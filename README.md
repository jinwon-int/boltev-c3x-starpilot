# Bolt EV 2017 (non-ACC) — C3X StarPilot Tracker

[![StarPilot](https://img.shields.io/badge/StarPilot-firestar5683-blue)](https://github.com/firestar5683/StarPilot)
[![C3X](https://img.shields.io/badge/Device-Comma%203X-green)](https://comma.ai/)
[![Maintained by](https://img.shields.io/badge/maintained%20by-대교%20(Daegyo)-purple)](https://github.com/jinwon-int)

> **관리 주체:** 대교(Daegyo) / Seo Jin On (`jinon86`)<br>
> **차량:** Chevrolet Bolt EV 2017 non-ACC + Comma Pedal<br>
> **기기:** Comma 3X `comma-dbba2a27` / Tailscale `tizi-the-galaxy`

## 목적과 경계

이 저장소는 C3X에 직접 설치하는 소스가 아니라 **실기 상태·설정·업데이트 근거·안전 게이트를 관리하는 정본**입니다. 실제 주행 소스는 기기의 `/data/openpilot`에 있는 StarPilot입니다.

- 소프트웨어 exact SHA와 AGNOS 상태
- FireStar/차량 설정 snapshot
- 업데이트·복구·rollback 근거
- 첫 ignition 및 첫 주행 안전 게이트
- SSH/Tailscale/Web UI 운영 절차

과거 시점의 `updates/` 기록은 당시 증거이므로 현행 endpoint로 일괄 치환하지 않습니다.

## Live canonical state — 2026-08-31 KST

| 항목 | 현행값 |
|---|---|
| Tailscale | `tizi-the-galaxy` / `100.90.4.121` |
| MagicDNS | `tizi-the-galaxy.tail1546e7.ts.net` |
| Local Wi-Fi | `192.168.55.222` (DHCP, mutable) |
| SSH | `comma@100.90.4.121:22`, Daegyo `~/.ssh/id_ed25519` |
| Web UI | `http://100.90.4.121:8082` |
| StarPilot base | `28ec3ccb80ff46fc88adbdf48e7b4a40c6afeede` |
| Base tree | `cd13b2453cb3f46cf0573e440a4af663a09ae74b` |
| Active overlay | reviewed forced-fingerprint cache patch, 3-file hashes verified |
| Branch/origin | `bolt-starpilot-28ec3ccb` / immutable local pin |
| AGNOS | `19.6.2` |
| Driving model | `rdf43` / Regret Driven Framework V4 / `v15` |
| Stored alternatives | Pop v2 (`pop223`), SC Driving (`sc23`), inactive |
| 서비스 | `comma`, `ssh.socket`, `tailscaled` active |

복구·설정 상세는 [`config/current.md`](config/current.md)와 [`updates/2026-08-25-recovery.md`](updates/2026-08-25-recovery.md)를 참조합니다.

## 현재 안전 게이트

2026-08-27 첫 ignition에서 Bolt 2017 fingerprint·Pedal·OP longitudinal·delay/PID는 확인됐지만, 세 번의 독립 startup에서 Panda `interruptRateCan2`와 `commIssue`가 반복됐습니다. 검토된 forced-fingerprint cache patch는 같은 날 off-vehicle 배포·재부팅·cache guard 검증을 통과했지만, 차량에 연결한 P단 parked CAN 재검증은 아직 수행하지 않았습니다. 현재 gate는 **FAIL / fail-closed**입니다. [최초 장애 기록](updates/2026-08-27-first-drive-can2-fault.md)과 [현행 후속 상태](updates/2026-08-31-official-doc-review.md)를 확인하고, 별도 승인된 parked 검증과 저속 canary 전에는 engage하지 않습니다.

## 차량 기준

| 항목 | 값 |
|---|---|
| 차량 | Chevrolet Bolt EV 2017 |
| ACC | 없음(non-ACC) |
| Pedal | Comma Pedal, stop-and-go 지원 |
| 수동 fingerprint | `CHEVROLET_BOLT_CC_2017` |
| OP longitudinal | first ignition에서 활성·delay `0.6`·expected PID 확인 |
| 제동 한계 | 최대 약 70 kW 회생제동, emergency stop 불가·저속 감속 취약; 즉시 수동 제동 준비 |

## 문서 구조

- [`config/current.md`](config/current.md): live snapshot
- [`tuning/bolt-2017-defaults.md`](tuning/bolt-2017-defaults.md): 현행 승인 profile과 안전 기준
- [`docs/playbook.md`](docs/playbook.md): 점검·운영 흐름
- [`docs/ssh-access.md`](docs/ssh-access.md): 접속 정본
- [`docs/api-reference.md`](docs/api-reference.md): Web/SSH/Tailscale 조회
- [`docs/update-procedures.md`](docs/update-procedures.md): exact-pin 업데이트 절차
- [`docs/troubleshooting.md`](docs/troubleshooting.md): 장애 진단
- [`patches/`](patches/): exact-base 패치 artifact, 배포 manifest와 검증 규칙
- `updates/`: append-only 시점별 이력

## 변경 정책

read-only 상태 점검과 upstream 비교는 자유롭게 수행합니다. Params/파일/branch/firmware 변경, 서비스 재시작, reboot, 첫 주행은 차량 제어 경계이므로 매 작업마다 오너의 fresh approval이 필요합니다. 모든 변경은 backup → exact target → offroad 검증 → owner-driven 저속 검증 순서로 진행합니다.
