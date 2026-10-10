# Project Documentation Rules (Non-Obvious Only)

- `torcs_jm_par.py` is the **competition entry point**, not `gym_torcs.py`. The gym wrapper is the original upstream code; the `jm_par` file is this fork's contribution.
- `snakeoil3_jm2.py` and `jmcncarai.py` are upstream IBM-provided files — do not treat them as canonical reference for this fork's patterns.
- `torcs_bob_v1.py` is a **more advanced driver** built on top of `torcs_jm_par.py`'s `Client` class, not a replacement for it. It uses a stopping-distance speed model instead of the simple `TARGET_SPEED` constant.
- The `vtorcs-RL-color/` directory contains the C++ TORCS simulator source — not Python code. Do not look there for Python logic.
- `bob_log_*.csv` files in the root are **runtime telemetry outputs** from `torcs_bob_v1.py`, not committed test fixtures.
- `practice.xml` is a TORCS race configuration file (XML), not Python config.
- There are **no unit tests** — validation is purely by running against the live TORCS simulator.
- The `README.md` documents two separate things: this fork's changes (top) and the original gym-torcs upstream (below the separator). Only the top section applies to `torcs_jm_par.py`.
