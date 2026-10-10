# Project Coding Rules (Non-Obvious Only)

- **Never hardcode the port** — use `self.port` set from `--port` CLI flag. The external TORCS Race Runner assigns ports dynamically.
- **`--parameters` is a single JSON string arg** (parsed via `json.loads`), not individual flags. Modifying key names breaks compatibility with Race Runner.
- All drive logic reads/writes state through `S = c.S.d` and `R = c.R.d` dicts. Never add instance attributes to `Client`/`ServerState`/`DriverAction` for sensor data.
- Global tunable constants (`TARGET_SPEED`, `STEER_GAIN`, etc.) are overwritten at runtime by the `--parameters` handler using `global`. Functions that use them must not cache them at import time.
- `clip()` is intentionally duplicated in each file — do not add cross-file imports; these files are meant to be run standalone.
- Sections between `#### REQUIRED FOR RACE MODE` / `#### REQUIRED FOR PRACTICE MODE` comments **must not be removed or restructured** — they are parsed/expected by external tooling.
- `gym_torcs.py` hardcodes port `3101` for the RL env client; `torcs_jm_par.py` defaults to `3001` for standalone use. Keep these separate.
- `torcs_bob_v1.py` uses `if __name__ == "__main__":` guard; `torcs_jm_par.py` also uses it. Do not add top-level execution code outside this guard.
