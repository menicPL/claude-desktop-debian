[< Back to README](../README.md)

# Claude-light

User-local tuning overlay for long Claude Desktop sessions on Linux.

```bash
contrib/claude-light/rebuild-user-asar
CLAUDE_DESKTOP_TUNED_LAUNCHER="$PWD/contrib/claude-light/launcher" \
  contrib/claude-light/claude-desktop-memprio
```

`claude-light` is an experimental, opt-in overlay that leaves the packaged app
under `/usr/lib/claude-desktop` untouched and layers extra launcher/runtime
behavior in user space.

## What it adds

- A tuned launcher that reuses upstream `launcher-common.sh`
- Reduced motion via `--force-prefers-reduced-motion`
- Disabled GPU compositing via `--disable-gpu-compositing`
- Precise memory metrics via `--enable-precise-memory-info`
- Exposed V8 GC via `--js-flags=--expose-gc`
- Cleanup for orphaned Claude helpers when no live Claude UI exists
- A rebuilt user-local `app.asar` with a Linux memory guard injected into the
  main process bundle
- A `systemd-run --user` wrapper with memory and OOM biasing
- A lightweight process memory snapshot helper

## Files

- [`contrib/claude-light/README.md`](../contrib/claude-light/README.md)
  Short overlay reference.
- [`contrib/claude-light/launcher`](../contrib/claude-light/launcher)
  Claude-first tuned launcher.
- [`contrib/claude-light/rebuild-user-asar`](../contrib/claude-light/rebuild-user-asar)
  Rebuilds a user-local `app.asar` and injects the memory guard.
- [`contrib/claude-light/claude-desktop-memprio`](../contrib/claude-light/claude-desktop-memprio)
  Starts Claude with `systemd-run --user` resource biasing.
- [`contrib/claude-light/claude-desktop-memory-snapshot`](../contrib/claude-light/claude-desktop-memory-snapshot)
  Prints RSS/PSS totals for the Claude process tree.

## Runtime knobs

- `CLAUDE_TUNED_USE_SYSTEM_ASAR=1`
  Launch the packaged `app.asar` instead of the rebuilt user-local copy.
- `CLAUDE_TUNED_REDUCED_MOTION=0`
  Disable reduced-motion launcher flags.
- `CLAUDE_TUNED_DISABLE_GPU_COMPOSITING=0`
  Keep GPU compositing enabled.
- `CLAUDE_TUNED_PRECISE_MEMORY_INFO=0`
  Disable precise memory metrics.
- `CLAUDE_TUNED_EXPOSE_GC=0`
  Do not add `--js-flags=--expose-gc`.
- `CLAUDE_TUNED_MEMORY_GUARD=0`
  Disable the injected memory guard at launch time.
- `CLAUDE_TUNED_MEMORY_GUARD_INTERVAL_SEC=300`
  Sampling interval.
- `CLAUDE_TUNED_RENDERER_GC_MB=900`
  Renderer working-set threshold.
- `CLAUDE_TUNED_TOTAL_GC_MB=1800`
  Total Electron working-set threshold.

## Notes

This overlay is intentionally user-local and experimental. It is the right place
to validate Claude-specific tuning before deciding what should become a
first-class upstream feature in `claude-desktop-debian`.
