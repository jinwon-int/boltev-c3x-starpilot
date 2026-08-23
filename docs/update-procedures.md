# C3X Update Procedures

> **관리:** 대교 (Daegyo) on S23 Ultra (Termux)  
> **대상:** Comma 3X (`tizi-the-pond`), StarPilot (`firestar5683/StarPilot`, `StarPilot` 브랜치)
>
> **2026-08-24 BLOCK:** 기기가 detached HEAD이고 `UpdaterTargetBranch=HEAD`라 자동 updater가 실패한다. [issue #6](https://github.com/jinwon-int/boltev-c3x-starpilot/issues/6)의 Pedal firmware·exact SHA·테스트·백업 게이트를 충족하기 전에는 아래 적용 명령을 실행하지 않는다.

---

## 업데이트 전 점검

```bash
# 1. C3X 온라인 상태 확인
ssh -o ConnectTimeout=5 -i ~/.ssh/id_ed25519 comma@100.71.169.100 echo OK

# 2. 현재 상태 확인
ssh comma@100.71.169.100 'cd /data/openpilot && \
  echo "=== Commit ===" && git log -1 --format="%h %s %ai" && \
  echo "=== Branch ===" && git branch --show-current && \
  echo "=== Remote ===" && git remote -v'

# 3. GitHub 최신 커밋과 비교
LATEST=$(curl -s https://api.github.com/repos/firestar5683/StarPilot/commits/StarPilot | jq -r .sha[0:7])
CURRENT=$(ssh comma@100.71.169.100 'cd /data/openpilot && git log -1 --format=%h')
echo "Latest: $LATEST, Current: $CURRENT"

# 4. updater target/exception 확인 — 읽기 전용
ssh comma@100.71.169.100 '
  for k in UpdaterTargetBranch UpdaterFetchAvailable UpdateAvailable LastUpdateException; do
    f=/data/params/d/$k
    [ -r "$f" ] && printf "%s=%s\n" "$k" "$(tr -d "\000\r\n" < "$f")"
  done
'
```

## 필수 승인 게이트

1. 차량이 Parked/Offroad인지 확인한다.
2. Gen1 Bolt Pedal firmware가 현행 StarPilot과 호환되는지 확인한다.
3. moving branch가 아니라 **exact candidate SHA**를 선정한다.
4. Bolt/GM/pedal/longitudinal/safety diff와 focused test 증거를 남긴다.
5. 현재 HEAD·Params·설정·dirty assets·updater 상태를 백업한다.
6. target branch 수정·checkout·reboot는 각각 오너 승인 범위 안에서만 수행한다.
7. offroad 검증 뒤 오너가 저속 첫 주행을 하고 즉시 수동 제동할 준비를 한다.

---

## Method 1: Git 표준 업데이트 (인터넷 양호할 때)

> **현재 실행 금지:** 아래 명령은 moving `StarPilot` HEAD를 즉시 적용하고 dirty 파일을 파괴할 수 있다. [issue #6](https://github.com/jinwon-int/boltev-c3x-starpilot/issues/6)에서 exact SHA·백업 절차를 확정할 때까지 역사적 참고로만 둔다.

```bash
ssh comma@100.71.169.100 '
  cd /data/openpilot && \
  git fetch origin StarPilot && \
  git checkout StarPilot && \
  git reset --hard origin/StarPilot
'

# 상태 재확인
ssh comma@100.71.169.100 'cd /data/openpilot && git log -1 --format="%h %s %ai"'
```

---

## Method 2: Tarball 우회 업데이트 (인터넷 느릴 때)

C3X의 git fetch가 타임아웃될 때 사용했던 역사적 방법입니다.

> **현재 실행 금지:** 이 절차는 moving branch tarball을 받고 기존 `.git`을 삭제하며, tarball에는 커밋 메타데이터가 없어 마지막 `git log` 검증도 성립하지 않는다. pin·무결성·원자 교체·롤백 절차를 다시 설계하기 전에는 사용하지 않는다.

### 전체 절차

```bash
# ═══════════════════════════════════════════
# Phase 1: Daegyo에서 다운로드
# ═══════════════════════════════════════════
cd ~
echo "Downloading StarPilot tarball..."
curl -fsSL "https://api.github.com/repos/firestar5683/StarPilot/tarball/StarPilot" \
  -o ~/starpilot.tar.gz

echo "Archive size: $(du -h ~/starpilot.tar.gz | cut -f1)"

# ═══════════════════════════════════════════
# Phase 2: C3X로 전송 (로컬 WiFi)
# ═══════════════════════════════════════════
echo "Transferring to C3X via local WiFi..."
scp -i ~/.ssh/id_ed25519 ~/starpilot.tar.gz comma@192.168.55.203:/data/

# ═══════════════════════════════════════════
# Phase 3: C3X에서 압축 해제 및 적용
# ═══════════════════════════════════════════
ssh comma@192.168.55.203 '
  set -e
  cd /data

  echo "Extracting..."
  rm -rf starpilot_new
  tar xzf starpilot.tar.gz

  STAR_DIR=$(ls -d firestar5683-StarPilot-*)
  echo "Source dir: $STAR_DIR"

  echo "Syncing to /data/openpilot..."
  rsync -a --delete $STAR_DIR/ /data/openpilot/

  echo "Reinitializing git..."
  cd /data/openpilot
  rm -rf .git
  git init
  git remote add origin https://github.com/firestar5683/StarPilot

  echo "Cleaning up..."
  rm /data/starpilot.tar.gz
  rm -rf /data/$STAR_DIR

  echo "Done! New commit:"
  git log -1 --format="%h %s %ai"
'

# ═══════════════════════════════════════════
# Phase 4: Daegyo 정리
# ═══════════════════════════════════════════
rm ~/starpilot.tar.gz
```

---

## Method 3: Fork/Branch 전환

다른 포크나 브랜치로 전환할 때 사용합니다. 현재 C3X에서는 target branch 수정이 updater의 즉시 fetch/stage를 유발할 수 있으므로 [issue #6](https://github.com/jinwon-int/boltev-c3x-starpilot/issues/6) 배포 창 전에는 실행하지 않습니다.

```bash
# 리모트 변경
ssh comma@100.71.169.100 '
  cd /data/openpilot && \
  git remote set-url origin https://github.com/<new-owner>/<new-repo>.git
'

# 새 브랜치 fetch 및 전환
ssh comma@100.71.169.100 '
  cd /data/openpilot && \
  git fetch origin <new-branch> && \
  git checkout <new-branch>
'
```

### Bolt 2017 관련 Fork/브랜치 목록

| Fork | 브랜치 | 상태 | 비고 |
|------|--------|------|------|
| `firestar5683/StarPilot` | `StarPilot` | ✅ 활성 | 메인 브랜치 (권장) |
| `firestar5683/StarPilot` | `Dom` | 🟡 대체 | Bolt 튜닝 적음 |
| `jc01rho/StarPilot` | `paddleuncompiled` | ❌ 276커밋 뒤 | 구형 |
| `firestar5683/StarPilot-2017` | `Kaofui` | ❌ 삭제됨 | 2026-02-17 중단 |

---

## 업데이트 후 검증

### 필수 확인사항

```bash
ssh comma@100.71.169.100 '
  echo "=== Version ==="
  cd /data/openpilot
  git log -1 --format="%h %s %ai"
  git branch --show-current

  echo "=== Remote ==="
  git remote -v

  echo "=== Disk ==="
  df -h /data | grep /data

  echo "=== Process ==="
  ps aux | grep openpilot | grep -v grep | head -3
'
```

### 설정 검증 체크리스트

업데이트 후 C3X Web UI에서 확인할 필수 설정들:

- [ ] Vehicle Controls → **Auto Fingerprint: OFF** (2017 수동 선택)
- [ ] Conditional Experimental Mode (CEM): **ON**
- [ ] Human Like Following: **OFF**
- [ ] Human Like Acceleration: **OFF**
- [ ] Curve Speed Controller: **ON**
- [ ] Always on Lateral (AoL): **ON**

전체 설정 체크리스트는 `tuning/bolt-2017-defaults.md` 참조

---

## 재부팅 (업데이트 적용)

⚠️ **재부팅은 차량 제어 소프트웨어 적용 경계입니다. 매 작업마다 진원님의 명시 승인을 새로 받아야 합니다.**

승인 후 다음 중 하나를 수행:
- **C3X 전원 코드 뽑았다 꽂기**
- **차량 시동 껐다 켜기**
- **C3X USB-C 전원 어댑터로 재부팅**

재부팅 후 1~2분 후 SSH/웹 UI 접속 재확인.

---

## 업데이트 로그 기록

업데이트 완료 후 이 레포의 `updates/YYYY-MM-DD.md` 에 기록:

```markdown
# YYYY-MM-DD — 업데이트

| 항목 | 변경 전 | 변경 후 |
|------|---------|---------|
| 브랜치 | ... | ... |
| 커밋 | ... | ... |
| 날짜 | ... | ... |
| 방법 | git / tarball |
```
