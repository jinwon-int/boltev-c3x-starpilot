# Current C3X Configuration Snapshot

> **촬영일:** 2026-08-24
> **소스:** SSH direct query (공융 → 대교 → C3X) + The Pond API
> **변경 경계:** read-only 감사. 업데이트·브랜치 수정·재부팅 없음
> **다음 확인:** [issue #6](https://github.com/jinwon-int/boltev-c3x-starpilot/issues/6) 게이트 충족 후

---

## 소프트웨어

| 항목 | 값 |
|------|-----|
| Fork | `firestar5683/StarPilot` |
| Branch | detached HEAD (from `StarPilot`) |
| Installed Commit | `d931d300` (2026-06-19 `build`, prebuilt) |
| Upstream HEAD | `28ec3ccb` (2026-08-23) — 미채택 |
| Latest observed prebuilt | `e9f4c631` (2026-08-21) — 미채택 |
| Delta | upstream 764 commits ahead |
| Update decision | **보류** — Bolt 제어 diff·Pedal firmware·exact-head test 게이트 미완료 |

## Git Remote

```
origin  https://github.com/firestar5683/StarPilot (fetch)
origin  git@github.com:firestar5683/StarPilot (push)
```

## 네트워크

| 항목 | 값 |
|------|-----|
| Tailscale IP | `100.71.169.100` |
| Local WiFi IP | `192.168.55.222` (DHCP, 변동 가능) |
| Web UI | `http://100.71.169.100:8082` |
| SSH | Port 22, key auth (jinon86 GitHub) |
| MagicDNS | `tizi-the-pond.tail1546e7.ts.net` |
| Device ID | `4700657545486091` |

## 차량

| 항목 | 값 |
|------|-----|
| 차량 | Chevrolet Bolt EV 2017 |
| ACC | Non-ACC |
| Pedal | Comma Pedal (stop&go 지원) |
| Torque | 4.5Nm (+50%) |
| Runtime fingerprint | `CHEVROLET_BOLT_CC_2017` |
| OP longitudinal | 활성 |
| Pedal interceptor | 활성 |
| Network location | `fwdCamera` |

## 2026-08-24 런타임 상태

| 항목 | 값 |
|---|---|
| 상태 | Online / Parked / Offroad |
| CPU 온도 | 약 41°C |
| 메모리 | 3.5 GiB 중 1.2 GiB 사용, 2.4 GiB 가용 |
| `/data` | 84% 사용, 약 14 GiB 여유 |
| `/` | 89% 사용, 약 517 MiB 여유 |
| 핵심 프로세스 | manager·pandad·hardwared·UI·mapd 동작 |
| 최근 crash/tombstone | 발견되지 않음 |

## 자동 업데이터 상태

- `UpdaterTargetBranch=HEAD` — detached HEAD가 잘못 target branch로 고정됨.
- `UpdaterFetchAvailable=1`, `UpdateAvailable=0`.
- updater는 `git checkout -B HEAD FETCH_HEAD`에서 `HEAD is not a valid branch name`으로 실패한다.
- 후보 SHA·백업·롤백 계획 승인 전에 target branch를 수정하지 않는다. 상세 [2026-08-24 감사](../updates/2026-08-24-audit.md) 및 [issue #6](https://github.com/jinwon-int/boltev-c3x-starpilot/issues/6).

## 주행 통계

| 항목 | 값 |
|------|-----|
| 전체 주행 | ~5,396 km (FrogPilot 기준) |
| 리셋 후 | ~266 km |

> 주의: 통계값은 SSH/web API로 실측 필요
