# PTZ Agent — Demo Script

Step-by-step instructions for running a live demo of the PTZ agent on a
freshly powered-on host.

---

## 0. Cold start (host just powered on)

```bash
cd ~/Downloads/MSA-main/ptz-agent

# Load camera credentials into the shell
source ~/.msa.env

# Activate the Python virtualenv
source .venv/bin/activate

# Make sure Ollama is running (skip if already up)
pgrep -x ollama >/dev/null || (ollama serve >/tmp/ollama.log 2>&1 &)

# Clean reset of supervisor + workers + state
pkill -f msa.supervisor 2>/dev/null
pkill -f msa.worker     2>/dev/null
rm -f ~/.msa/state.db ~/.msa/supervisord.sock

# Start the agent (auto-spawns the supervisor)
python -m msa
```

You should see:

```
MSA chat — gemma4:e2b on ollama; 0 worker(s) running. Ctrl+D to exit.
you>
```

The agent is now ready. Each line below is one user turn typed at the
`you>` prompt.

---

## 1. (Optional) Find the camera's IP

Use this if the camera's IP is unknown or DHCP changed it.

```
you> find the ptz camera
```

Expect: `ptz_find_camera` runs an ARP scan, writes the discovered IP
back to `~/.msa.env`, and replies with the address.

---

## 2. Calibrate the camera

```
you> calibrate the camera
```

Expect: `ptz_calibrate` drives both axes into hard-stops (~90 s),
records pan/tilt ranges to `tools/calibration.json`, and replies with
the measured ranges.

---

## 3. Move to a specific pan/tilt

```
you> move to pan 90 tilt 20
```

Expect: `ptz_move({"pan": 90, "tilt": 20})`, then a confirmation like
`Pan is at 91.4°, tilt at 20°.` (drift up to ±2° is normal).

Other phrasings that work:

```
you> point at pan 180
you> turn 30 degrees to the right
you> look up a bit
```

---

## 4. Describe the current frame

```
you> what is in the current frame?
```

Expect: `ptz_observe` (no movement) → snapshot → Gemma 4 caption →
agent replies with a short scene description. The JPEG is saved to
`./snapshots/observe_<timestamp>_p<pan>.jpg`.

---

## 5. Scan the room

```
you> scan the room and identify all objects and people you detect
```

Expect: `ptz_scan({"describe": true})` — sweeps the full pan range
(~8 stops, 60–90 s), captures a snapshot at each stop, captions each
one, and replies with a consolidated summary of objects and people.

---

## 6. Schedule a recurring task

```
you> every 5 minutes, scan the room and report any people you see
```

Expect: agent calls `schedule_task`, replies with something like
`Scheduled "room_scan" every 300s.` Verify with the slash command:

```
you> /tasks
```

To remove it later:

```
you> /tasks
# note the task name, then in another shell:
msa schedule rm room_scan
```

---

## 7. Deleting scheduled tasks

You can ask the agent in plain English:

```
you> list my scheduled tasks
you> delete the room_scan task
you> remove all scheduled tasks
```

Or use the slash command + a shell command in another terminal:

```
you> /tasks                # see name + interval + next run for each task
```

```bash
msa schedule list          # same info, JSON-friendly
msa schedule rm room_scan  # remove by name
```

If a task is currently running when you delete it, the in-flight worker
finishes its current iteration and then exits — no future runs will
fire.

---

## 8. Resetting if the agent gets stuck

### Symptom: a single worker is stuck in a tool-call loop

You'll see the same tool being called over and over in the chat output.
The driver guards usually catch this (`{"skipped": true, "reason": ...}`),
but if it doesn't:

```
you> /workers              # find the running worker's id, e.g. w-cc77
you> /cancel w-cc77        # SIGTERM that worker only
```

The chat keeps running. Your next message starts a fresh worker.

### Symptom: chat is unresponsive or wedged

In a second terminal:

```bash
msa workers                # see what's running
msa cancel <worker-id>     # kill the offender
```

Or kill every worker at once without touching the supervisor:

```bash
pkill -f msa.worker
```

### Symptom: supervisor itself is broken / nothing responds

Soft restart — keeps your scheduled tasks and history:

```bash
pkill -f msa.supervisor
pkill -f msa.worker
python -m msa              # auto-respawns the supervisor
```

### Nuclear reset — when in doubt

Wipes scheduled tasks, transcripts, worker history, and the IPC socket.
Camera credentials in `~/.msa.env` and calibration in
`tools/calibration.json` are kept.

```bash
pkill -f msa.supervisor 2>/dev/null
pkill -f msa.worker     2>/dev/null
rm -f ~/.msa/state.db ~/.msa/supervisord.sock
python -m msa
```

After this you're back to the same state as the cold-start in § 0. If
the demo has any recurring tasks you want, re-add them via § 6.

### Tail the logs while debugging

```bash
tail -f ~/.msa/logs/supervisor.log     # supervisor decisions
msa tail <worker-id>                   # live event stream for one worker
msa logs <worker-id>                   # full transcript after the fact
```

---

## Cleanup at the end of the demo

```
Ctrl+D                  # exit chat
msa schedule list       # show any tasks still scheduled
msa schedule rm <name>  # remove demo tasks so they don't keep firing
pkill -f msa.supervisor # stop the supervisor
```
