# StarPilot patch candidates

이 디렉터리는 현행 immutable pin을 기준으로 검토·테스트한 exact-base patch artifact와 배포 상태를 보관합니다. artifact 존재 자체는 배포 승인이 아니며 아래 manifest의 상태를 확인해야 합니다.

## 적용 규칙

1. 패치의 exact base SHA와 SHA-256을 확인합니다.
2. 격리 checkout에서 `git apply --check`와 포함된 테스트를 먼저 수행합니다.
3. 적용 결과를 새 exact commit/SHA 또는 base SHA + patch digest + result file hashes의 immutable manifest로 고정하고 GM/Panda safety gate를 통과시킵니다.
4. 차량 offroad·P단 또는 off-vehicle 상태에서 별도 owner approval을 받은 뒤에만 배포합니다.
5. 배포 전 기존 pin과 Params를 백업하고 rollback 경로를 확인합니다.
6. 정차 검증 후에도 별도 승인된 저속 canary 전에는 engage하지 않습니다.

## 후보 목록

| 파일 | exact base | SHA-256 | 상태 |
|---|---|---|---|
| `28ec3ccb-forced-fingerprint-cache.patch` | `28ec3ccb80ff46fc88adbdf48e7b4a40c6afeede` | `e82d1f07cf94f842a1f08471a66294ac99ccacdc1a75994770e8ed8bbbe80858` | 2026-08-27 off-vehicle 배포·재부팅 검증 완료; parked-in-vehicle CAN gate 대기 |

이 patch는 `ForceFingerprint=1`일 때만, `CarParamsPersistent`의 fingerprint·brand·VIN·firmware evidence가 현재 강제 모델과 모두 일치하면 이를 startup cache로 재사용합니다. 검증 실패·불일치·손상 시 기존 live query 경로로 fail closed합니다.

현행 result file SHA-256은 `car_params_cache.py=866fc934…`, `card.py=5b9d386e…`, `test_card_cache.py=98a392f8…`이며 2026-08-31 live readback에서 reviewed worktree와 일치했습니다. 이는 차량 연결 후 CAN fault 해소 증거가 아니므로 parked gate 전에는 engage하지 않습니다.
