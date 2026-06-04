# claude-light

Experimental user-local tuning overlay for long Claude Desktop sessions on Linux.

This overlay keeps the packaged app under `/usr/lib/claude-desktop` untouched.
It adds:

- a tuned launcher with reduced motion, disabled GPU compositing, precise memory
  info, and exposed V8 GC
- cleanup for orphaned Claude helper processes when no live Claude UI exists
- a user-local rebuilt `app.asar` with a Linux memory guard injected into the
  main process bundle
- a `systemd-run --user` wrapper with memory and OOM biasing
- a simple process memory snapshot helper

## Files

- `launcher`
  Claude-first launcher that reuses upstream `launcher-common.sh` and prefers a
  tuned user-local `app.asar` when present.
- `rebuild-user-asar`
  Extracts the system `app.asar`, injects a Linux memory guard, and writes a
  rebuilt copy under `~/.local/share/claude-desktop-tuned/resources/`.
- `claude-desktop-memprio`
  Runs the tuned launcher through `systemd-run --user` with resource weights and
  OOM protection.
- `claude-desktop-memory-snapshot`
  Prints RSS and PSS totals for Claude Desktop, Claude Code children, and the
  cowork daemon.

## Example flow

```bash
contrib/claude-light/rebuild-user-asar
CLAUDE_DESKTOP_TUNED_LAUNCHER="$PWD/contrib/claude-light/launcher" \
  contrib/claude-light/claude-desktop-memprio
```

## Runtime knobs

- `CLAUDE_TUNED_USE_SYSTEM_ASAR=1`
  Skip the rebuilt user-local `app.asar` and launch the packaged one.
- `CLAUDE_TUNED_REDUCED_MOTION=0`
  Disable `--force-prefers-reduced-motion`.
- `CLAUDE_TUNED_DISABLE_GPU_COMPOSITING=0`
  Keep GPU compositing enabled.
- `CLAUDE_TUNED_PRECISE_MEMORY_INFO=0`
  Disable `--enable-precise-memory-info`.
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

This overlay is intentionally user-local and experimental. It is a good place to
validate Claude-specific tuning before deciding what belongs in upstream
`claude-desktop-debian` proper.
