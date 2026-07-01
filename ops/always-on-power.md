# Always-on power config (never-sleep) — required for reachability

**Symptom (2026-07-02):** MOMO unreachable via Discord when the mini's screen is **locked**; works again
once logged back in. **Cause:** the mini was set to **system-sleep after 1 minute** idle — lock + walk away
→ sleep → the bridge process suspends + network drops → unreachable. Unlocking wakes it.

## The rule
An always-on host (mini, or any MacBook we later add as a second work-loop host) must **never system-sleep**.
Display sleep is fine (screen off saves power; doesn't touch the bridge) — it's **system** sleep that kills it.

## The fix (requires sudo — Eli runs it; the guard hard-blocks power/system settings for MOMO by design)
```
sudo pmset -a sleep 0        # never system-sleep  ← the fix
sudo pmset -a disksleep 0    # keep the disk awake
```
Verify with `pmset -g` → `sleep` should read **0**.

## Good baseline for a server Mac (confirmed on the mini after fix)
- `sleep 0` · `disksleep 0` — never sleep. **(the fix)**
- `womp 1` — wake on network access.
- `tcpkeepalive 1` — keep TCP alive.
- `autorestart 1` — auto-reboot after a power failure (bonus; good for always-on).
- `displaysleep 10` — fine to leave; screen can go dark.
- `powernap 0`, `standby 0`, `lowpowermode 0`.

## Debug checklist if reachability drops again
1. `pmset -g` → confirm `sleep 0` (first + most likely).
2. If sleep is already 0 but it still drops on lock → suspect **App Nap** on the bridge process, or **Wi-Fi
   disconnecting on lock** (Ethernet + never-sleep is bulletproof for a server).
3. Confirm the launchd agent + tmux `momo` session + the `claude` bridge process are alive (`pgrep`).

Related: [[discord-bridge]] (the bridge itself), [[multi-instance]].
