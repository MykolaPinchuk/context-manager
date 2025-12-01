# Context Manager

## Why this repo exists
- Central source of truth for reusable instructions and business context.
- Generates per-repo `agents.md` and `business_context.md` from shared templates.
- Keeps work repos lightweight: they only need `.context-profile.yml` plus optional local overrides.
- Avoids drift by letting a single script (`sync_context.py`) fan context into any repo.

## Layout
```
context-manager/
  instructions/                # shared instruction templates
  business/                    # shared business templates
  config/repos.yml             # optional list of work repos
  sync_context.py              # CLI to sync templates into repos
```

`config/repos.yml` lists the repos you want to sync:
```yaml
base_dir: /home/<USER>/repos
repos:
  - ds1              # resolved as /home/<USER>/repos/ds1
  - swe1             # resolved as /home/<USER>/repos/swe1
```
If `base_dir` is omitted, each entry must be an absolute path.

## Configure a work repo
Create `<repo>/.context-profile.yml` listing profiles per namespace.

Data science dev repo:
```yaml
instructions:
  - global
  - ds-dev
business:
  - biz-global
  - biz-sales-forecasting
```

SWE repo:
```yaml
instructions:
  - global
  - swe
business:
  - biz-global
```

Optional overrides live next to project files (`local_instructions.md`, `business_context.local.md`).

## Onboarding a new repo
1. Create `.context-profile.yml` inside the repo using one of the examples above.
2. (Optional) add `local_instructions.md` and/or `business_context.local.md` for repo-specific notes. A starter template for instructions overrides lives in this repo as `local_instructions.template.md`.
3. Add the repo name (or path) to `config/repos.yml` so `python sync_context.py` picks it up, or run the script once with `--repo /path/to/repo`.
4. Run `python sync_context.py` to generate/update `agents.md` and `business_context.md`.

## Running the sync
Install dependencies once:
```bash
pip install pyyaml
```

Sync every repo listed in `config/repos.yml`:
```bash
python sync_context.py
```

Sync a single repo directly:
```bash
python sync_context.py --repo /path/to/repo
```

`sync_context.py` creates/updates `agents.md` and `business_context.md` in each target repo and appends local overrides if present.
