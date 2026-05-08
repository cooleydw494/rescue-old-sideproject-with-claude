# Dev Ergonomics Scripts (`bin/setup`, `bin/dev`) — Design Guide

A rescued project should include zero-friction setup scripts — the equivalent of the "it just works" philosophy of well-set-up frameworks. `/init` creates these as part of Phase 5.5. This guide is the reference for *how* to write them well.

---

## Two scripts, two jobs

**`bin/setup`** — First-time setup. Run once after cloning. Gets a developer from "I have the repo" to "I can run the app."

**`bin/dev`** — Daily boot. Run every session. Starts services, checks state, confirms readiness. Should complete in seconds when everything is already running.

Some projects only need one (a static site needs neither — a framework with a database and a worker queue probably needs both). Use judgment. **A project that genuinely doesn't need a setup script doesn't need one** — don't manufacture work.

---

## Design principles

1. **Never destroy data.** Setup applies migrations incrementally, not via destructive reset. If a developer wants a clean slate, they do it manually.
2. **Never overwrite `.env`.** If it exists, the developer may have customized it. Only create from `.env.example` when missing.
3. **Skip work already done.** Check before acting — is the service running? Is the package installed? Are dependencies current? Re-running setup should be a fast no-op when everything is already good.
4. **Fail fast, fail clear.** Missing a prerequisite? Print exactly what to install and a URL, not a cryptic error.
5. **No `sudo`, no PATH modifications, no shell config writes.** The script checks for tools and tells the user how to install them. It does not reach into `~/.zshrc` or `/etc/`.
6. **Auto-start what you can.** If a tool isn't running but CAN be started programmatically (e.g., `open -a Docker`), start it. If a dependency is missing but CAN be installed without elevated permissions, install it. Only report what the script genuinely can't fix itself.
7. **Distinguish hard vs soft prerequisites.** A missing language runtime is a hard fail (exit). Docker not running is a soft fail (warn, continue if the rest can proceed). The script should go as far as it can, report all issues at the end, and not die on the first non-critical problem.

---

## Critical pitfalls (learned the hard way)

### Pitfall 1: Assuming Homebrew is writable

On multi-user macOS machines, `/opt/homebrew` is often owned by a different user account. `command -v brew` succeeds (brew is on PATH) but `brew install` fails with permission errors. **Always check writability** before attempting brew operations:

```bash
brew_available() {
  command -v brew &> /dev/null || return 1
  local prefix
  prefix="$(brew --prefix 2>/dev/null)" || return 1
  [ -w "$prefix" ] || return 1
}
```

For tools available as direct binaries (Supabase CLI, kubectl, etc.), prefer **direct binary downloads** to a user-writable location (`~/.local/bin`):

```bash
curl -fsSL "https://example.com/cli/latest/cli_darwin_arm64.tar.gz" | tar xz -C ~/.local/bin
```

This works on any machine, multi-user or not.

### Pitfall 2: `set -e` with soft failures

`set -e` kills the script on any non-zero exit. This makes "check and warn" patterns impossible — the check itself aborts the script before your warning code runs. Use `set -uo pipefail` (without `-e`) and handle errors explicitly:

```bash
# BAD: set -e means brew failure kills the script before fail_soft runs
brew install something
fail_soft "couldn't install something"  # unreachable

# GOOD: explicit handling
if brew_available && brew install something 2>/dev/null; then
  ok "installed"
else
  fail_soft "couldn't install" "alternative install instructions"
fi
```

### Pitfall 3: Empty arrays on Bash 3.2

macOS ships Bash 3.2. Empty arrays with `set -u` crash on Bash 3.2 (`${ERRORS[@]}` is "unbound variable" when empty). Use a counter variable instead of array length, or use `${arr[@]+"${arr[@]}"}` syntax:

```bash
# BAD on Bash 3.2:
ERRORS=()
if [ ${#ERRORS[@]} -gt 0 ]; then ...    # unbound variable

# GOOD:
ERRORS=()
ERROR_COUNT=0
ERRORS+=("something failed")
ERROR_COUNT=$((ERROR_COUNT + 1))
if [ $ERROR_COUNT -gt 0 ]; then ...
```

### Pitfall 4: Auto-start what you can

Don't tell the user to "open Docker Desktop and re-run the script" — open it for them. On macOS, `open -a Docker` launches Docker Desktop. Poll `docker info` in a loop until it's ready (with a timeout):

```bash
if ! docker info &> /dev/null; then
  echo "Starting Docker Desktop..."
  open -a Docker
  for i in $(seq 1 30); do
    docker info &> /dev/null && break
    sleep 2
  done
  docker info &> /dev/null || fail_soft "Docker still not running after 60s — try manually"
fi
```

The setup script should solve problems, not report them. Apply this principle everywhere it's safe to.

### Pitfall 5: Hard vs soft fails treated the same

Don't bail on the first issue. Collect issues, report them all at the end, exit non-zero only if there were hard fails:

```bash
HARD_FAILS=0
SOFT_FAILS=0

check_node() {
  if ! command -v node >/dev/null; then
    echo "[HARD] Node not found. Install: https://nodejs.org/"
    HARD_FAILS=$((HARD_FAILS + 1))
  fi
}

check_docker() {
  if ! docker info &>/dev/null; then
    echo "[SOFT] Docker not running. Some features will be unavailable."
    SOFT_FAILS=$((SOFT_FAILS + 1))
  fi
}

check_node
check_docker
# ... more checks

if [ $HARD_FAILS -gt 0 ]; then
  echo "Setup failed: $HARD_FAILS hard prerequisites missing."
  exit 1
fi

if [ $SOFT_FAILS -gt 0 ]; then
  echo "Setup completed with $SOFT_FAILS warnings."
  exit 0
fi

echo "All checks passed."
```

---

## Skeleton: `bin/setup`

```bash
#!/usr/bin/env bash
# bin/setup — First-time project setup. Idempotent.
set -uo pipefail

PROJECT_ROOT="$(cd "$(dirname "$0")/.." && pwd)"
cd "$PROJECT_ROOT"

# --- helpers ---
ok()       { echo "  ✓ $*"; }
warn()     { echo "  ⚠ $*"; }
fail()     { echo "  ✗ $*"; HARD_FAILS=$((HARD_FAILS + 1)); }
soft()     { echo "  ⚠ $*"; SOFT_FAILS=$((SOFT_FAILS + 1)); }
HARD_FAILS=0
SOFT_FAILS=0

echo "Setting up <project-name>..."

# --- 1. Hard prerequisites (language runtimes, package managers) ---
echo ""
echo "Checking prerequisites..."
# command -v node >/dev/null && ok "node" || fail "node not found — https://nodejs.org/"
# command -v <runtime> ...

# --- 2. Soft prerequisites (services, optional tools) ---
# docker info &>/dev/null && ok "docker running" || { open -a Docker; ...poll loop...; }

# --- 3. .env handling ---
echo ""
if [ ! -f .env ]; then
  cp .env.example .env
  ok ".env created from .env.example"
  warn ".env has placeholder values — edit before running"
else
  ok ".env exists (preserved)"
fi

# --- 4. Dependencies ---
echo ""
echo "Installing dependencies..."
# npm ci --silent && ok "npm packages installed" || fail "npm ci failed"
# composer install --no-interaction && ok "composer packages installed" || fail "composer install failed"

# --- 5. Database (idempotent migrations) ---
# php artisan migrate --force && ok "migrations up to date" || fail "migration failure"

# --- 6. Build (only if missing — `bin/dev` handles ongoing dev mode) ---
# [ ! -d public/build ] && npm run build && ok "build artifacts generated"

# --- summary ---
echo ""
if [ $HARD_FAILS -gt 0 ]; then
  echo "Setup failed: $HARD_FAILS hard prerequisites missing. Fix the [HARD] items above."
  exit 1
fi
if [ $SOFT_FAILS -gt 0 ]; then
  echo "Setup complete with $SOFT_FAILS warnings. You can use the project; some features may be limited."
fi
if [ $HARD_FAILS -eq 0 ] && [ $SOFT_FAILS -eq 0 ]; then
  echo "Setup complete. Next: ./bin/dev"
fi
```

---

## Skeleton: `bin/dev`

```bash
#!/usr/bin/env bash
# bin/dev — Daily boot. Starts services, runs the dev server. Re-run-safe.
set -uo pipefail

PROJECT_ROOT="$(cd "$(dirname "$0")/.." && pwd)"
cd "$PROJECT_ROOT"

ok()   { echo "  ✓ $*"; }
warn() { echo "  ⚠ $*"; }

echo "Starting dev environment..."

# --- 1. Verify setup ran ---
[ -f .env ] || { echo "✗ .env missing. Run ./bin/setup first."; exit 1; }
# [ -d node_modules ] || { echo "✗ node_modules missing. Run ./bin/setup first."; exit 1; }

# --- 2. Start required services if not running ---
# docker info &>/dev/null || { open -a Docker; ...poll loop...; }
# pgrep -f redis-server >/dev/null || { brew services start redis; ok "redis started"; }

# --- 3. Hand off to the actual dev server ---
# Use exec so signals are forwarded correctly and there's no orphan process.
echo ""
echo "Launching dev server. Ctrl+C to stop."
echo ""
# exec npm run dev
# exec composer dev   # for `concurrently`-style multi-process dev
# exec php artisan serve
```

The point of `bin/dev` is that it boots everything needed AND ends with `exec` to the actual dev process so the user sees logs immediately and `Ctrl+C` cleanly stops everything.

---

## Where the scripts live

```
project-root/
├── bin/
│   ├── setup            ← first-time setup (run once)
│   ├── dev              ← daily boot (run every session)
│   └── deploy           ← (optional, prod-time only)
└── ...
```

Make them executable: `chmod +x bin/setup bin/dev`.

Reference them from the project README's `## Setup` and `## Development` sections so future contributors find them without spelunking.

---

## Anti-patterns

- **`bin/setup` that destroys data.** Migrations, never resets. If reset is what's needed, that's a separate `bin/reset-dev-database` script with explicit user confirmation.
- **`bin/dev` that takes more than ~10s on a re-run.** It should detect "everything is already running" and just hand off. Slow re-runs train the user not to use it.
- **Setup that asks questions.** Setup should be 100% non-interactive. If a configuration choice exists, encode it in `.env.example` with a comment explaining the choice.
- **Hardcoded paths.** Scripts derive `PROJECT_ROOT` from their own location, never assume cwd.
- **No `chmod +x`.** Beautiful script that won't run because it's not executable. Always set the bit.
- **Setup script that's "documentation in shell form".** If 80% of the script is `echo "Now do X"`, just write a Markdown checklist instead. The script should DO things.
