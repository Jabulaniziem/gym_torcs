# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Project Overview

Python-only TORCS racing simulator client. No package manager, no build system, no tests. Requires Linux + `vtorcs-RL-color` installed and `xautomation` for GUI automation via `autostart.sh`.

## Running

```bash
# Standalone car client (primary file - IBM competition entry)
python torcs_jm_par.py [--port PORT] [--parameters '{"target_speed":100,"steer_gain":30,"centering_gain":0.20,"brake_threshold":0.9,"gear_thresholds":[0,20,40,80,100,180],"traction_control":true}']

# RL environment experiment
python example_experiment.py

# Bob v1 driver (logs telemetry to bob_log_TIMESTAMP.csv)
python torcs_bob_v1.py [--port PORT]
```

## Critical Architecture

- **`torcs_jm_par.py`** — main competition file. Three-layer architecture: `Client` (UDP socket + CLI), `ServerState` (parses TORCS telemetry string into `S.d` dict), `DriverAction` (builds response string from `R.d` dict). Drive logic is in `drive_modular()` which calls the modular helper functions.
- **`snakeoil3_gym.py`** — the UDP client used by `gym_torcs.py` (RL env). Nearly identical structure to `torcs_jm_par.py` but without the `--parameters` flag.
- **`gym_torcs.py`** — OpenAI Gym-like wrapper. TORCS is launched via `os.system('torcs ...')` at `__init__` time; `autostart.sh` uses `xte` keypresses to navigate the GUI. UDP port hardcoded to `3101`.
- **`torcs_bob_v1.py`** — extended version of `torcs_jm_par.py` with stopping-distance speed model, ABS, live wheel-radius calibration, and CSV telemetry logging.

## Non-Obvious Constraints

- **Port must not be hardcoded** — comments explicitly say `#### DO NOT HARDCODE THE PORT`. Port is set via `--port` / `-p` CLI flag (default 3001). External `run-cars.py` (separate repo) manages port assignment for multi-car races.
- **`--parameters` flag accepts a JSON string**, not individual args. Keys: `target_speed`, `steer_gain`, `centering_gain`, `brake_threshold`, `gear_thresholds` (list of 6 ints), `traction_control` (bool).
- **Sections marked `REQUIRED FOR RACE MODE - DO NOT CHANGE OR REMOVE`** in `torcs_jm_par.py` must be preserved exactly — they are read by the TORCS Race Runner external project.
- **Sections marked `REQUIRED FOR PRACTICE MODE - CAN CHANGE BUT DO NOT REMOVE`** (the parameter block and `--parameters` CLI handler) must also remain intact.
- **`gym_torcs.py` uses port 3101**, not 3001 (the default in all standalone clients).
- **TORCS has a memory leak on race reset** — `example_experiment.py` works around this by calling `env.reset(relaunch=True)` every 3rd episode.
- **`autostart.sh`** sends `xte` keypresses to automate the TORCS GUI; it must be run after TORCS starts and assumes the default Practice mode menu navigation.
- **Sensor values in `gym_torcs.py`** are normalised: `focus`/`opponents`/`track` divided by 200, speeds divided by `default_speed` (50). Raw `rpm` and `wheelSpinVel` are not normalised.
- **Vision input** is a flat vector of length 12288 (64×64×3), iterated in steps of 3 in `obs_vision_to_image_rgb()`. Screen resolution must be set to exactly 64×64 in TORCS Display options.
- **`torcs_bob_v1.py`** writes CSV telemetry log named `bob_log_YYYYMMDD_HHMMSS.csv` to the working directory when `LOG_ENABLED = True` (default).

## Code Style

- No type annotations anywhere in the codebase.
- State accessed as plain dicts: `S = c.S.d`, `R = c.R.d` — never through object attributes.
- Global tunable constants in ALL_CAPS at module top; `drive_modular`/`drive_bob` read them as module globals.
- `clip(v, lo, hi)` utility is duplicated across files — do not import between files.
- No external dependencies beyond `gym`, `numpy`, `socket`, `csv`, `json`, `math` (stdlib).
