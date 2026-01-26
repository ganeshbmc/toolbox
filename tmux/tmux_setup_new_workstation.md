# tmux Setup (WSL2/Linux) — Install + Configure + Resurrect Workflow

This guide documents how to set up **tmux + TPM + tmux-resurrect + tmux-continuum** on a fresh workstation, using the exact configuration currently used in this repo.

It includes:
- Prerequisites
- Installation steps
- `~/.tmux.conf` contents (current config)
- Plugin install steps
- Save/restore workflow (especially useful on WSL2 after shutdowns)

---

## 1) Prerequisites

### Install tmux
**Ubuntu / Debian (including WSL2 Ubuntu):**
```bash
sudo apt update
sudo apt install -y tmux git
```

Verify:
```bash
tmux -V
git --version
```

---

## 2) Install TPM (Tmux Plugin Manager)

TPM is installed to:
- `~/.tmux/plugins/tpm`

Run:
```bash
mkdir -p ~/.tmux/plugins
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

---

## 3) Create `~/.tmux.conf` (Current Configuration)

Create/edit:
```bash
nano ~/.tmux.conf
```

Paste **exactly**:

```tmux
# Enable Mouse
set -g mouse on

# TPM plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'

# Plugin configuration (MUST be above TPM run line)
set -g @continuum-restore 'on'
set -g @continuum-save-interval '5'

# --- Hard-bind resurrect (bypass plugin keybinding issues) ---
bind-key -T prefix S run-shell "~/.tmux/plugins/tmux-resurrect/scripts/save.sh" \; display-message "Resurrect: saved"
bind-key -T prefix R run-shell "~/.tmux/plugins/tmux-resurrect/scripts/restore.sh" \; display-message "Resurrect: restored"
bind-key -T prefix K run-shell 'tmux kill-session -t "#{session_name}"'

# Initialize TPM (MUST be last line)
run '~/.tmux/plugins/tpm/tpm'
```

Notes:
- The `run '~/.tmux/plugins/tpm/tpm'` line **must be last**.
- `prefix` is `Ctrl+b` unless changed elsewhere.

---

## 4) Start tmux and install plugins via TPM

Start tmux:
```bash
tmux
```

Inside tmux, press:
- `prefix + I` (capital i) to **install** plugins

Expected plugins installed under:
- `~/.tmux/plugins/tmux-resurrect`
- `~/.tmux/plugins/tmux-continuum`

---

## 5) Confirm that resurrect is saving to the correct directory

tmux-resurrect typically writes snapshots to:
- `~/.local/share/tmux/resurrect/`

Check:
```bash
ls -lah ~/.local/share/tmux/resurrect/
```

You should see files like:
- `last`
- `tmux_resurrect_YYYYMMDDTHHMMSS.txt`

If you don’t see this directory yet, do the first manual save (next section).

---

## 6) Daily Usage / Workflow

### Manual save (recommended)
Inside tmux:
- `prefix + S`

Expected message:
- `Resurrect: saved`

### Manual restore
Inside tmux:
- `prefix + R`

Expected message:
- `Resurrect: restored`

### Automatic save
Continuum auto-saves every 5 minutes:
- `set -g @continuum-save-interval '5'`

### Automatic restore
Continuum attempts auto-restore when tmux starts and there are no existing sessions:
- `set -g @continuum-restore 'on'`

---

## 7) WSL2 restart behavior (important)

On WSL2, when WSL shuts down:
- tmux server dies
- `tmux ls` will show nothing on next start

Typical workflow after WSL restart:
```bash
tmux
```
This can create a **junk numeric session** before restore kicks in.

### Kill junk session quickly
Inside tmux:
- `prefix + K`

This kills the **current session** (`#{session_name}`), so only use it when you are in the junk session.

Optional safer version (only kill numeric sessions) — replace the `K` binding with:
```tmux
bind-key -T prefix K run-shell '
case "#{session_name}" in
  [0-9]*) tmux kill-session -t "#{session_name}" ;;
esac
'
```

---

## 8) Sanity test (recommended once after setup)

1. Create a couple of sessions:
```bash
tmux new -s proto
tmux new -s srma
```

2. Save:
- `prefix + S`

3. Kill tmux server:
```bash
tmux kill-server
```

4. Start tmux again:
```bash
tmux
```

5. Restore (if not auto-restored):
- `prefix + R`

6. Verify:
```bash
tmux ls
```

---

## 9) Troubleshooting

### Plugins not installing
- Ensure TPM exists at `~/.tmux/plugins/tpm`
- Inside tmux run: `prefix + I`

### Resurrect directory not found
Run a manual save first:
- `prefix + S`

Then check:
```bash
ls -lah ~/.local/share/tmux/resurrect/
```

---

## Summary (what matters)

- tmux + TPM + resurrect + continuum are installed
- Save: `prefix + S`
- Restore: `prefix + R`
- WSL2 junk session cleanup: `prefix + K`
