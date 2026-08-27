# StarPilot patch candidates

이 디렉터리는 C3X에 즉시 적용하는 hotfix 모음이 아니라, 현행 immutable pin을 기준으로 검토·테스트할 **후보 패치**를 보관합니다.

## 적용 규칙

1. 패치의 exact base SHA와 SHA-256을 확인합니다.
2. 격리 checkout에서 `git apply --check`와 포함된 테스트를 먼저 수행합니다.
3. 적용 결과를 새 exact commit/SHA로 고정하고 GM/Panda safety gate를 통과시킵니다.
4. 차량 offroad·P단에서 별도 owner approval을 받은 뒤에만 배포합니다.
5. 배포 전 기존 pin과 Params를 백업하고 rollback 경로를 확인합니다.
6. 정차 검증 후에도 별도 승인된 저속 canary 전에는 engage하지 않습니다.

## 후보 목록

| 파일 | exact base | SHA-256 | 상태 |
|---|---|---|---|
| `28ec3ccb-forced-fingerprint-cache.patch` | `28ec3ccb80ff46fc88adbdf48e7b4a40c6afeede` | `517a9cb835d61f45b7efd1460c936454a14a6d733fb19ff89f88f270ec711e79` | 검토 후보 · 실차 미배포 |

이 후보는 `ForceFingerprint=1`일 때만, `CarParamsPersistent`의 fingerprint·brand·VIN·firmware evidence가 현재 강제 모델과 모두 일치하면 이를 startup cache로 재사용합니다. 검증 실패·불일치·손상 시 기존 live query 경로로 fail closed합니다.
