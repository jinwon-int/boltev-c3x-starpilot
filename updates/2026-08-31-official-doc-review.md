# 2026-08-31 StarPilot official-doc review and live reconciliation

## Scope

Daegyo reviewed the public StarPilot documentation at `wiki.firestar.link`, compared it with this tracker and performed a bounded read-only C3X check. No source, Params, firmware, service, branch, model selection, reboot, route, or vehicle state was changed.

## Official guidance reviewed

- [Bolt Hardware](https://wiki.firestar.link/cars/bolt/): steering is unavailable below 7 mph; a non-ACC Bolt with Comma Pedal uses up to about 70 kW of regen, cannot perform emergency stops and may need manual braking at low speed.
- [Configure Settings](https://wiki.firestar.link/usage/settings/): manual GM fingerprint, CEM curve trigger OFF, slower/stopped lead ON for radarless Bolts, and NNFF/NNFFLite OFF align with the current profile. The guide recommends a 7-second CEM stop time; the owner-approved profile remains 7.7 seconds and was not changed.
- [Driving Model](https://wiki.firestar.link/usage/driving-model/): current recommendations are Regret Driving Framework v4, Pop v2 and SC Driving. The device already selects RDF v4; Pop v2 and SC Driving remain downloaded but inactive.
- [Gen1 Bolt Pedal Firmware](https://wiki.firestar.link/software/pedal-firmware/): the warning applies to 2017–2021 pedals purchased before June 2025 that were not updated. The available device record is a June 2025 purchase confirmation, not a direct firmware-version readback. The accepted purchase-date inference is recorded without claiming firmware verification.
- [Install StarPilot](https://wiki.firestar.link/software/starpilot/): `StarPilot` is the stable branch and `Dom` is unstable. This does not override the tracker’s exact-pin, diff, test, rollback and approval gates.

The documentation homepage sends release notes to Discord rather than publishing a release ledger. The public `StarPilot` branch remained `a497c0f83966a73cc07f338a7387ac2c4669cfa6`: 7 commits ahead of the deployed base, consisting of the already-reviewed AGNOS 19.6.10 build/reapply and mirror batch. No new GM/Panda/fingerprint/SPI fix justified a live source or AGNOS update.

## 2026-08-31 live readback

- base HEAD/tree: `28ec3ccb80ff46fc88adbdf48e7b4a40c6afeede` / `cd13b2453cb3f46cf0573e440a4af663a09ae74b`
- branch/origin architecture: `bolt-starpilot-28ec3ccb` / immutable local pin
- AGNOS: `19.6.2`
- services: `comma.service`, `ssh.socket`, `tailscaled.service` active
- state: `IsOffroad=1`, `IsOnroad=0`, `IsEngaged=0`
- persistent evidence: `CarParamsPersistent` present, 1,808 bytes; transient `CarParams` and `FirmwareQueryDone` absent off-vehicle
- selected model: `rdf43` / Regret Driven Framework V4 / `v15`
- stored inactive artifacts: `pop223` and `sc23`
- `/data/params/d` and `/cache/params/d`: all 23 tracked profile keys matched
- overlay result SHA-256:
  - `selfdrive/car/car_params_cache.py`: `866fc9346fa4f4fbfee68310aead513d8ed56a711939dff2c61c659dfa072605`
  - `selfdrive/car/card.py`: `5b9d386ed97fff39d89d1b48102094c71c8a7a215dad57a51f7cc8d5e06112b2`
  - `selfdrive/car/tests/test_card_cache.py`: `98a392f80f50ab3d237eb7597187b801ea21040704c6eb292e8081589012d95e`

These hashes match the reviewed detached worktree. They prove deployment identity, not in-vehicle CAN fault clearance.

## Decision and remaining gate

Keep the current exact base, reviewed overlay, AGNOS 19.6.2 and RDF v4 selection. Do not follow the moving stable branch or switch to `Dom`.

The next vehicle operation remains a separately approved P-only parked CAN validation after the cache patch. It must verify Panda fault state, logical bus health, fingerprint/Pedal/longitudinal values and fail closed on any recurrence. A low-speed owner-driven canary requires another explicit approval after the parked gate passes.
