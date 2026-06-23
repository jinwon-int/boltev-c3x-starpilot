# Bolt EV 2017 (non-ACC) — C3X StarPilot Tracker

[![StarPilot](https://img.shields.io/badge/StarPilot-firestar5683-blue)](https://github.com/firestar5683/StarPilot)
[![C3X](https://img.shields.io/badge/Device-Comma%203X-green)](https://comma.ai/)
[![Maintained by](https://img.shields.io/badge/maintained%20by-대교%20(Daegyo)-purple)](https://github.com/jinwon-int)
[![Bolt](https://img.shields.io/badge/Car-Bolt%20EV%202017-orange)](https://wiki.firestar.link/cars/bolt/)

> **관리 주체:** 대교 (Daegyo) — Hermes Agent on Samsung S23 Ultra (Termux)  
> **담당자:** 진원님 (Seo Jin On, `jinon86`)  
> **차량:** Chevrolet Bolt EV 2017, non-ACC + Comma Pedal  
> **디바이스:** Comma 3X (comma-dbba2a27), 호스트명 `tizi-the-pond`

---

## 📋 레포 목적

이 저장소는 진원님의 **Bolt EV 2017 non-ACC C3X**에 설치된 StarPilot 소프트웨어의
업데이트 이력, 설정 스냅샷, 튜닝 참조값, 그리고 자율 관리 플레이북을 추적합니다.

- ✅ **소프트웨어 업데이트 로그** — 언제, 어떤 브랜치/커밋으로 업데이트했는지
- ✅ **설정 스냅샷** — StarPilot 튜닝 파라미터 현재값 백업
- ✅ **튜닝 레퍼런스** — Bolt 2017 전용 lateral/longitudinal 기본값
- ✅ **트러블슈팅** — 발생했던 문제와 해결책 기록
- ✅ **플레이북** — 대교가 자율적으로 C3X를 관리할 때 참조하는 절차

---

## 🚗 차량 사양

| 항목 | 값 |
|------|-----|
| 모델 | Chevrolet Bolt EV 2017 |
| ACC | ❌ 없음 (non-ACC) |
| 페달 | ✅ Comma Pedal (회생제동 + stop&go) |
| 스티어링 토크 | 4.5Nm (+50%, 2017 전용) |
| LKAS | Premier + Driver Confidence II |

---

## 📡 C3X 디바이스

| 항목 | 값 |
|------|-----|
| 기기 | Comma 3X (comma-dbba2a27) |
| 소프트웨어 | StarPilot (`firestar5683/StarPilot`, `StarPilot` 브랜치) |
| Tailscale IP | `100.71.169.100` |
| 로컬 WiFi IP | `192.168.55.203` (변동 가능) |
| Web UI | `http://100.71.169.100:8082` |
| SSH | `comma@100.71.169.100` (키 인증) |

---

## 📂 구조

```
boltev-c3x-starpilot/
├── README.md                      # 이 파일 (개요 + 차량 사양 + 링크)
├── .gitignore
├── config/
│   └── current.md                 # 현재 C3X 설정 스냅샷
├── updates/
│   └── YYYY-MM-DD.md              # 업데이트 로그 (날짜별)
├── tuning/
│   └── bolt-2017-defaults.md      # Bolt 2017 튜닝 레퍼런스 + 설정 체크리스트
└── docs/
    ├── playbook.md                # 대교 자율 관리 플레이북
    ├── ssh-access.md              # SSH 접속 가이드 (키, 연결, 관리)
    ├── api-reference.md           # API 레퍼런스 (Web, Tailscale, GitHub)
    ├── troubleshooting.md         # 트러블슈팅 가이드 (6가지 시나리오)
    ├── update-procedures.md       # 업데이트 절차 (Git + Tarball + 검증)
    └── developer-tracker.md       # firestar5683 개발 활동 트래커
```

---

## 🔗 핵심 링크

| 리소스 | URL |
|--------|-----|
| StarPilot GitHub | https://github.com/firestar5683/StarPilot |
| StarPilot Wiki | https://wiki.firestar.link/ |
| Bolt 하드웨어 가이드 | https://wiki.firestar.link/cars/bolt/ |
| 설정 가이드 | https://wiki.firestar.link/usage/settings/ |
| 운영 가이드 | https://wiki.firestar.link/usage/operation/ |
| Galaxy 원격 설정 | https://wiki.firestar.link/usage/galaxy/ |
| Discord 커뮤니티 | https://firestar.link/discord |
| 관리 스킬 (Hermes) | `infrastructure/c3x-management` |

---

## 🤖 자율 관리

대교(Daegyo)는 다음을 자율적으로 수행합니다:

1. **주기적 업데이트 확인** — C3X 웹 API + GitHub 커밋 비교
2. **SSH 접속** — `~/.ssh/id_ed25519` 키로 comma@Tailscale 접속
3. **브랜치 전환/업데이트** — git fetch + checkout + pull
4. **설정 검증** — Essential Settings Checklist 확인
5. **진원님 알림** — 재부팅 필요 시 Telegram으로 통보

> ⚠️ `reboot` 명령어는 Hermes hardline blocklist에 포함되어 직접 실행 불가.
> 재부팅은 진원님이 직접 수행합니다.

---

## ✅ CI 범위

- `.github/workflows/baseline-ci.yml`는 PR/push에서 Markdown/상대링크, JSON/YAML/TOML 스냅샷 구문, 보수적 secret pattern scan만 수행합니다.
- CI는 C3X/차량 SSH, Web UI, StarPilot API 호출, 업데이트, 재부팅을 실행하지 않는 read-only 문서/스냅샷 검증입니다.
- 실제 차량/디바이스 변경은 기존 플레이북의 수동 승인과 별도 evidence 수집 절차로만 진행합니다.

---

## 📝 라이선스

이 저장소의 문서/로그는 자유롭게 참조 가능합니다.
StarPilot 소프트웨어는 [firestar5683/StarPilot](https://github.com/firestar5683/StarPilot) 라이선스를 따릅니다.
