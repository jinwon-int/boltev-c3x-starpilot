# 2026-08-27 — Upstream delta check and pinned-SHA Bolt test-evidence inventory

## Scope

Read-only repository evidence work. No C3X fetch, checkout, Params write, service
restart, reboot, firmware operation, route mutation, or driving action was performed.
The device (`tizi-the-galaxy`, `100.90.4.121`) was offline during this check —
live readback remains gated on first ignition exactly as recorded in issue #6.

## Upstream delta since the deployed pin

Deployed pin: `28ec3ccb80ff46fc88adbdf48e7b4a40c6afeede` (branch `bolt-starpilot-28ec3ccb`,
local immutable origin `file:///data/starpilot-pins/28ec3ccb.git`).

Measured 2026-08-27 via GitHub compare `28ec3ccb...HEAD`:

- upstream ahead by **7 commits**, pin behind by **0** (pin is an ancestor; deployment stays valid)
- commit subjects: `Agnos 19.6.10`, `build`, revert×2, reapply×2, `Add AGNOS download mirrors`
- **zero** changes under `opendbc_repo/opendbc/car/gm/` or any car-port path

Conclusion: no control-relevant drift. AGNOS 19.6.2 on the device remains a valid,
deliberately pinned choice; 19.6.10 is noted for a future separately-approved update.

## Pinned-SHA Bolt 2017 assertion inventory (source-verified at exact head)

Static verification of the assertions the 2026-08-24 audit relied on, extracted from
the exact deployed SHA (paths relative to repo root):

| Evidence | Location at pin |
|---|---|
| CC setpoint deadband ignores small error — matrix includes `CHEVROLET_BOLT_CC_2017` | `opendbc_repo/opendbc/car/gm/tests/test_gm.py::test_bolt_cc_redneck_ignores_small_setpoint_error` (parametrized 2017 / 2018–2021 / 2022–2023) |
| Non-Bolt CC setpoint selector unchanged (comparison guard) | `test_gm.py::test_non_bolt_cc_redneck_keeps_existing_setpoint_selector` (EQUINOX_CC expects selector still fires) |
| Pedal-long shared planning delay without PID retune | `test_gm.py::test_bolt_pedal_long_uses_shared_planning_delay_without_retuning_pid` asserts `longitudinalActuatorDelay == 0.6`, `kpV == [0.095, 0.085, 0.065, 0.050]`, `kiV == [0.07, 0.10, 0.15, 0.24]`, `kf == 0.20` |
| Dedicated lateral tuning block for CC 2017 | `values.py` `STEER_MAX=450, STEER_DELTA_UP=15, STEER_DELTA_DOWN=34, ALLOWANCE=78, MULTIPLIER=6, FACTOR=100` |
| 2017-specific torque curve | `interface.py` `NON_LINEAR_TORQUE_PARAMS["CHEVROLET_BOLT_CC_2017"] left=[2.15,1.00,0.129,0.0], right=[2.15,1.00,0.145,0.0]` |
| Pedal-long membership + gen1-only cancel personality | `interface.py` `BOLT_PEDAL_LONG_CARS` ∋ `CHEVROLET_BOLT_CC_2017`; `BOLT_GEN1_CANCEL_PERSONALITY_CARS` = {2017, 2018–2021} |
| Stock-friction experiment excluded from 2017 | `carcontroller.py::supports_bolt_acc_pedal_friction_experiment` gates solely on `CHEVROLET_BOLT_ACC_2022_2023_PEDAL` |

This satisfies the audit precondition "produce a complete Bolt/GM/pedal/
longitudinal/safety diff summary" at source level for the exact deployed head:
the pin contains the 2017 fingerprint logic with regression tests around it,
and the experimental friction path cannot activate on this vehicle.

### Runtime execution environment — explicitly recorded absence

The two precondition options were "executable targeted test evidence OR record
the absence of a reproducible exact-head test environment". Recording the latter:

- running `test_gm.py` requires the full openpilot/opendbc/cereal/panda Python
  stack (capnp, scons-generated DBC libs, msgq bindings) that this docs tracker
  host does not provide;
- the C3X itself is offline pending owner first ignition, so an on-device
  repro is equally unavailable;
- therefore the assertions above are recorded as statically verified at the
  exact tree, with the authoritative runtime confirmation deferred to the
  first-ignition readback (fingerprint, pedal detection, delay/PID live values).

## Unchanged fail-closed gate

Everything in the issue's remaining checklist stands: pedal Gen1 firmware check,
updater target normalization inside an approved window (now moot post-format —
origin is a local pin), first ignition CarParams/pedal/health verification, and
the separately approved owner-driven low-speed first drive.
