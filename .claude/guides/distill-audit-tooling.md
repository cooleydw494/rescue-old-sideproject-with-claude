# Distill Audit Tooling — Setup Guide

Companion guide for `/distill`. Helps a new project wire up the per-stack tooling that `/distill` consumes. Read this once when adopting `/distill` in a new project; revisit when you add a new tech layer.

---

## What `/distill` actually needs from the project

Two things, both project-specific:

1. **A static-analysis orchestrator** (typically `scripts/static-analysis.sh`) that the orchestrator runs in Phase 1 as a sanity check after the cold-read pass. Output is human-scannable structural heat (oversized files, duplication, unused symbols). The orchestrator reads it to *strengthen* hypotheses generated from reading — not to seed them.

2. **A green-signal matrix in the verifier** that names, per source area, the test commands and static-analysis subcommands whose findings must not regress for a Tier A change to land. Lives in `.claude/agents/distill-verifier.md` under the PROJECT CUSTOMIZATION block.

The dropbox `commands/distill.md` and `agents/distill-verifier.md` carry PROJECT CUSTOMIZATION blocks marking exactly where these go. Fill them in once per project.

---

## Stack cookbooks

Each cookbook gives a starting `scripts/static-analysis.sh` shape, the tools to install, and a first cut at the verifier's Green Signal Matrix. Tune from there.

The shape is consistent across stacks: **one orchestrator script that routes subcommands**, each subcommand running a single tool, all subcommands non-fatal (`|| true`) because static analysis informs, it doesn't gate. The output is human-scannable; the command-line argument pattern lets individual tools run standalone for Tier A verification.

### Laravel (PHP + Vue/Inertia/Livewire)

**Tools:**
- **Larastan** — PHPStan for Laravel
- **Rector** — automated refactors (dry-run for detection)
- **jscpd** — cross-language duplication (PHP + Vue)
- **knip** — unused JS/Vue exports and orphans
- **madge** — circular dependencies in JS modules
- **Custom `dead-code-audit.sh`** — Eloquent relationship/scope/accessor liveness (grep-based heuristic — see Custom Audit Script Pattern below)
- **Custom `vue-audit.sh`** — Vue component size/depth/import-count/class-string duplication
- **Custom `design-audit.sh`** — design-token compliance if you have a token system

**Install:**
```bash
composer require --dev larastan/larastan rector/rector
npm install --save-dev knip madge jscpd
```

**`scripts/static-analysis.sh` shape:**
```bash
#!/bin/bash
set -uo pipefail
target="${1:-all}"
run_all=true; [ "$target" != "all" ] && run_all=false

if $run_all || [ "$target" = "larastan" ]; then
    vendor/bin/phpstan analyse --memory-limit=512M 2>&1 || true
fi
if $run_all || [ "$target" = "rector" ]; then
    vendor/bin/rector process --dry-run 2>&1 || true
fi
if $run_all || [ "$target" = "jscpd" ]; then
    npx jscpd app/ resources/js/ 2>&1 || true
fi
if $run_all || [ "$target" = "knip" ]; then
    npx knip --no-progress --reporter compact 2>&1 || true
fi
# ... add deadcode / vue-audit / design subcommands following the same pattern
```

**Verifier Green Signal Matrix:**
| Area touched | Required signals |
|---|---|
| `app/**` | `php artisan test --parallel` green + `static-analysis.sh larastan` no new errors |
| `resources/js/**` | `npm run build` clean + `npm test` green + `knip` no new findings + `vue-audit` no regression |
| `resources/css/**` | `npm run build` clean + `static-analysis.sh design` no new violations |
| `config/**`, `routes/**` | `php artisan config:clear && php artisan route:list` clean + full `php artisan test --parallel` |

### Rails (Ruby + Hotwire/Stimulus or ViewComponent)

**Tools:** RuboCop, Brakeman (security/bloat-adjacent), debride (dead methods), reek (code smells), rails_best_practices, rubycritic (complexity × churn).

**Install (Gemfile :development):**
```ruby
gem 'rubocop'; gem 'reek'; gem 'rails_best_practices'
gem 'brakeman'; gem 'debride'; gem 'rubycritic'
```

**Verifier Green Signal Matrix:**
| Area touched | Required signals |
|---|---|
| `app/**` | `rails test` green + `rubocop` no new offenses + `brakeman` no new warnings |
| `config/**` | `rails test` + `rails routes` sanity check |

### Python (Django/FastAPI/Flask/service)

**Tools:** ruff (lint+format), vulture (dead code), mypy/pyright (types), radon (complexity), pylint (deeper structural), jscpd (duplication).

**Install:**
```bash
pip install ruff vulture mypy radon pylint
```

**Verifier Green Signal Matrix:**
| Area touched | Required signals |
|---|---|
| source | `pytest` green + `ruff check` no new issues + `mypy` no new errors |
| `tests/**` | `pytest` green + coverage unchanged |

### Node / TypeScript / React / Vue / Next.js

**Tools:** knip (unused exports/files/deps — best-in-class), madge (circular deps; JS/TS only, not JSX/SFC), ts-prune (alt to knip for TS-heavy), jscpd, custom `component-audit.sh` for size/depth/class-string dup.

**Install:**
```bash
npm install --save-dev knip madge jscpd
```

**Verifier Green Signal Matrix:**
| Area touched | Required signals |
|---|---|
| `src/**` | `npm run build` clean + `npm test` green + `knip` no new findings + `component-audit` no regression |
| `styles/**` | `npm run build` + design-audit no new violations |

### Go

**Tools:** staticcheck (lint+bugs+simplifications, includes `unused`), gocyclo, gocognit, dupl, errcheck.

**Install:**
```bash
go install honnef.co/go/tools/cmd/staticcheck@latest
go install github.com/fzipp/gocyclo/cmd/gocyclo@latest
go install github.com/uudashr/gocognit/cmd/gocognit@latest
go install github.com/mibk/dupl@latest
```

**Verifier Green Signal Matrix:**
| Area touched | Required signals |
|---|---|
| all | `go test ./...` green + `staticcheck ./...` no new issues |

---

## Custom audit script pattern

For bloat signals no off-the-shelf tool produces well (component size thresholds, template nesting depth, class-string duplication, prop-drilling shapes), write a grep+awk script in this shape. Proven portable across Laravel/Vue, Rails/ERB, Python/Jinja, React/JSX.

```bash
#!/usr/bin/env bash
# <Stack> Structural Audit — flags structural heat for /distill.
# Grep/awk-based heuristics, no AST. Structured file:line output.
# Exit 0 always — reporting tool, not a CI gate.

set -uo pipefail
TARGET_DIR="<source dir>"

# Thresholds — tune as you learn the project's normal range.
PAGE_LOC_THRESHOLD=400
COMPONENT_LOC_THRESHOLD=300
ABSOLUTE_LOC_THRESHOLD=500
NESTING_DEPTH_THRESHOLD=8
IMPORT_COUNT_THRESHOLD=20
DUPLICATE_LITERAL_MIN_LEN=50
DUPLICATE_LITERAL_MIN_FILES=3

target="${1:-all}"
run_all=true; [ "$target" != "all" ] && run_all=false

# Subcommands: size / depth / imports / classes — each $run_all || [ "$target" = "<name>" ] gated.
# Each prints file:line findings; final summary line aggregates counts.

exit 0
```

**Principles:**
1. **Non-fatal.** Exit 0 always. Failing CI on this defeats the purpose.
2. **Structured output.** File:line on every finding so the orchestrator can cite them.
3. **Standalone subcommands.** Each check is its own target so the verifier can run just the relevant subset.
4. **No AST.** Grep + awk is the ceiling. AST is off-the-shelf tooling's job (knip, staticcheck).

The shape transfers across stacks mostly unchanged — pick the language-specific tools from the cookbook above, write the orchestrator script in the shape shown, and tune thresholds to your project's normal range over the first few `/distill` runs.

---

## Anti-patterns to avoid

1. **Stack-specific rules in the command file.** The PROJECT CUSTOMIZATION blocks exist precisely so the command stays portable. If you find yourself rewriting prose in `commands/distill.md`, you've lost the portability battle.

2. **Audit scripts that fail CI.** Static analysis informs `/distill`; it doesn't gate the build. Want a CI gate? Run a separate linter tier with fail-on-new-violations.

3. **Treating knip/madge/etc. output as gospel.** False positives are common (dynamically-referenced exports, framework-magic-called symbols). Configure `ignore*` arrays thoughtfully.

4. **Skipping the static-analysis sanity step because "we don't have audits yet."** Install one tool from the cookbook and start; the cold-read pass is the primary signal anyway, but the audit catches things the eye misses.
