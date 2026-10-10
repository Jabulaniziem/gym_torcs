# Project Architecture Rules (Non-Obvious Only)

- **Communication is UDP only** — `Client.so` is a `SOCK_DGRAM` socket to `localhost:PORT`. All sensor data arrives as a single TORCS protocol string parsed by `ServerState.parse_server_str()` into a flat dict.
- **`DriverAction.d`** is the only output channel. `__repr__` on `DriverAction` serialises it to the TORCS wire format. Do not bypass `respond_to_server()`.
- **`autostart.sh` is a hard dependency for `gym_torcs.py`** — it sends `xte` key events to navigate the TORCS GUI at startup. If TORCS GUI changes, this script must change.
- **TORCS memory leak**: every race reset leaks memory in the TORCS process. The only workaround is `pkill torcs` + relaunch + `autostart.sh`. `gym_torcs.py` does this every N episodes.
- **`torcs_jm_par.py` is designed for the TORCS Race Runner** (external project): the `--port` flag and `--parameters` JSON flag must remain compatible with that runner's calling convention. Changing parameter names or removing CLI flags breaks multi-car race orchestration.
- **Drive functions are stateless per-step** — `drive_modular()` and `drive_bob()` are called once per simulator tick (≈50 Hz). They read from `S` and write to `R` with no buffering beyond what's in `R.d` itself. Exception: `torcs_bob_v1.py` keeps ABS/TC state in the module-level `_bob` dict.
- **Track sensor array** (`S['track']`) has 19 elements at non-uniform angles (`-45 -19 -12 ... 0 ... 12 19 45` degrees). Index `[9]` is directly ahead. This is set in `snakeoil3_gym.py`'s `setup_connection()`.
- **`gym_torcs.py` reward** is `speedX * cos(angle)` (forward progress). Collision or going off-track overrides reward to `-1` and sets `meta=True` to signal episode end.
