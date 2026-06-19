# firestar5683 Developer Tracker

> **StarPilot 개발자:** firestar5683 (Dom), firestarsdog (BigUI)  
> **GitHub:** https://github.com/firestar5683  
> **Discord:** https://firestar.link/discord  
> **Wiki:** https://wiki.firestar.link/

---

## 개발 활동 패턴

| 항목 | 내용 |
|------|------|
| firestar5683 (Dom) | 1-3 commits/day, 메인 개발자 |
| firestarsdog | BigUI (큰 화면 접근성 UI) |
| 커밋 스타일 | 재치있는 메시지, 빠른 이터레이션 |
| 최근 포커스 | Bolt lateral tuning, UI 개선, Kia 지원 추가 |
| StarPilot 방향 | GM targeted fork → 범용 QoL 개선 포크 |

---

## 최근 커밋 (2026년 6월)

```
2026-06-13 471e944 Remove Slow Logic atm              — Bolt 토크 응답성 개선
2026-06-12 3a3a467 the slow mo guys                    — Lateral tuning 개선
2026-06-12 87f1334 yes                                  — 빌드/릴리스
2026-06-12 140b047 better raylib                        — UI 렌더링 엔진
2026-06-12 21b1bfd THIS DESERVES A BUMP                — 버전 업
2026-06-12 5da99bc build                                — 빌드 시스템
2026-06-12 a30dc40 wumbology                            — (내부 개선)
2026-06-12 96893d9 Dingle's got dragonclaw?!            — (기능 추가)
2026-06-11 9be0def The speed of cyan [firestarsdog]     — BigUI WIP
2026-06-11 afc8c6f Mr. Freeze                           — (내부 개선)
2026-06-11 b559b0f Fridge Cigarette                     — (내부 개선)
2026-06-11 79e31ff TRY AGAIN, SPIDERMAN                 — (버그 수정)
2026-06-11 1e604fd loopback                             — (네트워크)
2026-06-11 56989a1 update                               — (업데이트)
2026-06-10 eaf5d44 forte he                             — (튜닝)
2026-06-10 93cb562 BigUI WIP: Larger for the Dads       — 접근성 UI
2026-06-10 31d3df6 Kia Xceed                            — 새 차량 지원
2026-06-10 c0aa7a7 oops                                 — (버그 수정)
2026-06-10 498b7f1 build                                — 빌드 시스템
2026-06-10 95b6ece Hello Poppet                         — (기능 추가)
```

---

## Changelog (Wiki 기준)

| 날짜 | 변경사항 | Bolt 관련 |
|------|----------|-----------|
| **2026-02-13** | 새 인스톨러 (`install.firestar.link`), Deep Nuke Params | ✅ Bolt 2017 4.5Nm 공식 지원 |
| **2026-01-12** | Golden Safety Rule, Advanced lateral tuning defaults | ✅ Bolt 기본값 문서화 |
| **2025-10-12** | Green Watermelon v8 승격, Curve Speed Controller 도입, Human Like OFF | ✅ CEM/Radar-less 개선 |
| **2025-09-13** | TRX 브랜치 추가 (non-GM용) | |
| **2025-08-27** | Bolt Pedal 펌웨어 업데이트 안내 | ✅ Comma Pedal 중요 |

---

## StarPilot 포크 계보

```
commaai/openpilot
    └── FrogAi/FrogPilot (커뮤니티 포크)
        └── firestar5683/StarPilot (GM/Bolt 최적화)
            ├── StarPilot (메인) ← Bolt 2017 권장
            └── Dom (대체)
```

---

## 프로젝트 특징

- **100% 오픈소스** — 클린 커밋 히스토리
- **월간 업데이트** — 커뮤니티 기여 기반
- **Galaxy 포털** — 폰에서 설정/모델 관리
- **Custom Installer** — `install.firestar.link`
- **Radar-less 특화** — VoACC/GM 차량 튜닝 개선

---

## 확인 방법 (자동화)

### GitHub API로 최신 활동 확인

```bash
# 최근 10개 커밋
curl -s 'https://api.github.com/repos/firestar5683/StarPilot/commits?per_page=10' \
  | jq -r '.[] | "\(.commit.author.date[0:10]) \(.sha[0:7]) \(.commit.message | split("\n")[0]) [\(.author.login)]"'

# 릴리스 태그
curl -s https://api.github.com/repos/firestar5683/StarPilot/releases/latest \
  | jq '{tag: .tag_name, date: .published_at}'

# Star + Fork 수
curl -s https://api.github.com/repos/firestar5683/StarPilot \
  | jq '{stars: .stargazers_count, forks: .forks_count, updated: .pushed_at}'
```

### 주기적 점검

대교는 cronjob으로 주 1회 firestar5683 활동을 확인하여 C3X 업데이트 필요 여부를 판단합니다.

> **Discord 참여:** `firestar.link/discord` — StarPilot 커뮤니티에서 직접 개발 현황 확인 가능
