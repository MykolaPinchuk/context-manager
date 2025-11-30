````markdown
# PRD: Context Manager Framework (`context-manager` repo)

## 0. Purpose

This document describes the requirements for a **context manager** framework that centralizes reusable context (instructions, business/domain context, etc.) and syncs it into multiple work repos as generated Markdown files.

Target user flow:

- All **source-of-truth context** lives in this `context-manager` repo.
- Each work repo declares which context it wants via a small YAML profile file.
- A single script (`sync_context.py`) generates/updates context files inside work repos.
- Coding agents working inside individual work repos only read the generated files; they do not need to read or write anything outside their repo.

---

## 1. Goals

1. Provide a **single source of truth** for:
   - Coding instructions (per profile).
   - Business/domain context (per profile).
2. Avoid duplication and drift of context files (e.g. no more ad-hoc `business_context_vXX` scattered across repos).
3. Allow **layered context**:
   - Global baseline (common to all repos using it).
   - Profile-specific variants:
     - Instructions: `ds-dev`, `ds-prod`, `swe`.
     - Business: `biz-global`, `biz-sales-forecasting`.
   - Optional repo-specific overrides.
4. Make it easy to:
   - Add new profiles.
   - Add new context “namespaces” (e.g. instructions, business, domain, data).
   - Onboard new repos by adding a small YAML file.

---

## 2. Non-goals

- No UI, web service, daemon, or background scheduler.
- No automatic scanning/guessing of repo types based on code.
- No enforcement of agent behavior beyond providing context files (this framework is not a policy engine).
- No complex templating engine is required (simple concatenation is sufficient).

---

## 3. High-level design

### 3.1 Core concepts

- **Namespace**: A type of context. Initially:
  - `instructions`
  - `business`
- **Profile**: A reusable named slice of context within a namespace.

  Initial profiles:

  - Instructions:
    - `global`
    - `ds-dev`
    - `ds-prod`
    - `swe`
  - Business:
    - `biz-global`
    - `biz-sales-forecasting`

- **Per-repo context profile** (`.context-profile.yml`): Declares which profiles from which namespaces apply to that repo.
- **Generated artifacts** (per repo):
  - `agents.md` (instructions)
  - `business_context.md` (business context)
- **Repo-local overrides** (optional, per repo):
  - `agents.local.md`
  - `business_context.local.md`

### 3.2 Data flow

1. Source-of-truth context templates live in `context-manager` under `instructions/` and `business/`.
2. Each work repo contains `.context-profile.yml` specifying profiles for each namespace.
3. `sync_context.py`:
   - Reads `.context-profile.yml` for each repo.
   - Loads the relevant templates from `context-manager`.
   - Concatenates them (in order) into the generated file for that namespace.
   - Appends repo-local overrides if present.
   - Writes the final Markdown files into the repo (e.g. `agents.md`, `business_context.md`).

---

## 4. Repository layout (context-manager)

Implement the following file/folder structure in this repo:

```text
context-manager/
  prd.md                  # this document

  instructions/
    00-global.md          # baseline instructions
    10-ds-dev.md          # DS dev profile
    10-ds-prod.md         # DS prod profile
    10-swe.md             # SWE profile (any SWE repo)

  business/
    00-biz-global.md              # baseline business context
    10-biz-sales-forecasting.md   # sales forecasting business context

  config/
    repos.yml             # (optional) list of work repos + paths; see §6.2

  sync_context.py         # main script
  README.md               # short overview and usage instructions
````

Notes:

* `config/repos.yml` is **optional** but should be supported. If used, it describes which repos exist and where they live (see §6.2).
* The script must support **two modes**:

  * Mode A: use `config/repos.yml` to sync multiple repos.
  * Mode B: use `--repo` CLI option to sync a single repo path.

---

## 5. Per-repo layout (work repos)

Each work repo (e.g. `ds1`, `ds2`, `swe1`, `swe2`) will eventually contain:

```text
<repo>/
  .context-profile.yml       # REQUIRED: declares profiles
  agents.md                  # GENERATED: instructions for this repo
  business_context.md        # GENERATED: business context for this repo

  agents.local.md            # OPTIONAL: local instructions override
  business_context.local.md  # OPTIONAL: local business override

  ... other project files ...
```

The coding agent must not assume specific repo contents beyond `.context-profile.yml` and the generated artifacts.

---

## 6. Configuration formats

### 6.1 `.context-profile.yml` (per repo)

Location: `REPO_ROOT/.context-profile.yml`

Purpose: For each namespace, list the profiles (by name) to apply in order.

Example for a DS dev repo:

```yaml
instructions:
  - global
  - ds-dev

business:
  - biz-global
  - biz-sales-forecasting
```

Example for a SWE repo:

```yaml
instructions:
  - global
  - swe

business:
  - biz-global
```

Rules:

* Each top-level key corresponds to a namespace (e.g. `instructions`, `business`).
* Each value is an ordered list of profile names (strings).
* The script must validate:

  * The namespace exists in its mapping (`PROFILE_MAP`).
  * Each profile name is known for that namespace.
* On invalid namespace/profile name, the script should:

  * Print a clear error.
  * Skip generation for that repo (or at least for that namespace), rather than writing partial/invalid output.

### 6.2 `config/repos.yml` (optional, context-manager)

Location: `context-manager/config/repos.yml`

Purpose: Mechanism to declare known repos and assist with scripted multi-repo sync.

Example schema:

```yaml
base_dir: /home/<USER>/repos    # base directory for repos (optional)

repos:
  - name: ds1
    path: ds1
  - name: ds2
    path: ds2
  - name: swe1
    path: swe1
  - name: swe2
    path: swe2
```

Behavior:

* If `base_dir` is present, the script resolves each repo path as `base_dir/path`.
* If `base_dir` is omitted, each `path` must be absolute.
* If a repo path does not exist:

  * Log a warning.
  * Skip that repo.

---

## 7. Template files (initial minimal content)

Templates are **intentionally short** and will be overwritten later. Implement exactly these minimal scaffolds.

### 7.1 `instructions/00-global.md`

```markdown
# Global Instructions (Baseline)

- Do not read or modify files outside the repository root.
- Prefer small, focused changes and clear explanations.
```

### 7.2 `instructions/10-ds-dev.md`

```markdown
# DS Dev Profile

- This repo is primarily for data science development and experiments.
- Prefer adding reusable logic to `src/` and keeping notebooks simple.
```

### 7.3 `instructions/10-ds-prod.md`

```markdown
# DS Prod Profile

- This repo is closer to production; be conservative with changes.
- Ensure tests exist or are added when modifying core code paths.
```

### 7.4 `instructions/10-swe.md`

```markdown
# SWE Profile

- This repo is a software service or library.
- Preserve existing patterns for logging, error handling, and testing.
```

### 7.5 `business/00-biz-global.md`

```markdown
# Business Context – Global

- We operate in a business environment where financial and operational outcomes matter.
- Key objectives: good long-term economics, compliance, and stakeholder trust.
```

### 7.6 `business/10-biz-sales-forecasting.md`

```markdown
# Business Context – Sales Forecasting

- Focus on forecasting sales or revenue to support planning and decision-making.
- Metrics of interest include forecast accuracy and impact on business decisions.
```

---

## 8. `sync_context.py` requirements

### 8.1 Language and dependencies

* Implement in Python 3.
* Use standard library where possible:

  * `pathlib`, `argparse`, `logging`, `os`, `sys`.
* For YAML:

  * Prefer `pyyaml` if available.
  * If `pyyaml` is used, document the dependency in `README.md`.
  * If `pyyaml` is not installed, print a clear error and exit.

(For this PRD, it is acceptable to depend on `pyyaml` and require it.)

### 8.2 CLI interface

Implement at least:

```bash
# Mode A: use config/repos.yml to sync multiple repos
python sync_context.py

# Mode B: sync a single repo
python sync_context.py --repo /path/to/repo
```

Requirements:

* `--repo`:

  * Takes precedence over `config/repos.yml` when provided.
  * Expects an existing directory containing `.context-profile.yml`.
* When run **without** `--repo`:

  * Attempt to load `config/repos.yml` from the same directory as `sync_context.py`.
  * If `config/repos.yml` is missing, print a clear message and exit.

### 8.3 Namespace mapping and target files

Inside the script, define mappings like:

```python
PROFILE_MAP = {
    "instructions": {
        "global": "instructions/00-global.md",
        "ds-dev": "instructions/10-ds-dev.md",
        "ds-prod": "instructions/10-ds-prod.md",
        "swe": "instructions/10-swe.md",
    },
    "business": {
        "biz-global": "business/00-biz-global.md",
        "biz-sales-forecasting": "business/10-biz-sales-forecasting.md",
    },
}

TARGET_FILES = {
    "instructions": "agents.md",
    "business": "business_context.md",
}

LOCAL_OVERRIDES = {
    "instructions": "agents.local.md",
    "business": "business_context.local.md",
}
```

The code should be structured so that adding a new namespace is a matter of extending these mappings.

### 8.4 Generation algorithm (per repo, per namespace)

For a given repo and namespace:

1. Determine the generated filename from `TARGET_FILES[namespace]`.

2. Read `.context-profile.yml` from the repo root.

3. Fetch the list of profiles for this namespace (e.g. `config["instructions"]`).

4. For each profile:

   * Look up the template path via `PROFILE_MAP[namespace][profile_name]`.
   * Read the template content from the `context-manager` repo.

5. Check for repo-local override:

   * Use `LOCAL_OVERRIDES[namespace]` to find the local override filename (if defined).
   * If the override file exists in the repo, read and append it as the last section.
   * If it does not exist, append a short stub:

     ```markdown
     # Repo-Specific Notes

     There are no repo-specific notes yet.
     ```

6. Prepend a header such as:

   ```markdown
   <!-- AUTO-GENERATED ({namespace}). Do not edit this file directly.
   To change shared context, edit templates in this context-manager repo.
   To change repo-specific notes, edit {override_filename} in this repo.
   -->

   ```

7. Join sections with `\n\n---\n\n` between them.

8. Write the resulting Markdown to the target file in the repo (overwriting any existing file).

### 8.5 Safety and path constraints

The script must:

* Only read templates from within the `context-manager` repo (instructions, business, config).
* Only write files under:

  * `<REPO_ROOT>/agents.md`
  * `<REPO_ROOT>/business_context.md`
* It must **not** modify:

  * Source code, tests, or other project files (`src/`, `notebooks/`, `tests/`, etc.).
  * Files outside the specified repo roots.

The coding agent implementing this PRD must not directly modify any work repo beyond what is needed to:

* Add sample `.context-profile.yml` files (if requested).
* Generate and test `agents.md` / `business_context.md`.

---

## 9. README.md requirements (context-manager)

Create a `README.md` in the `context-manager` repo that:

1. Explains the purpose of this repo in 3–5 short bullet points, e.g.:

   * Central source of truth for reusable context (instructions, business).
   * Generates per-repo `agents.md` and `business_context.md`.
   * Avoids duplicated, out-of-sync context files across repos.
2. Documents:

   * Directory structure (briefly).
   * How to configure `.context-profile.yml` in a repo, with:

     * Example for a DS dev repo (`global` + `ds-dev`, `biz-global` + `biz-sales-forecasting`).
     * Example for a SWE repo (`global` + `swe`, `biz-global`).
   * How to run `sync_context.py`:

     * `python sync_context.py` (multi-repo mode).
     * `python sync_context.py --repo /path/to/repo` (single-repo mode).
3. Mentions any dependencies (`pyyaml`) and how to install them.

Keep the README concise.

---

## 10. Implementation tasks (for coding agent)

1. Scaffold the directory structure and files as per §4 and §7.

2. Implement `sync_context.py` as per §8.

3. Implement `README.md` as per §9.

4. Optionally add a simple wrapper, e.g.:

   ```bash
   # sync_all.sh
   python sync_context.py
   ```

   or a `Makefile` target:

   ```make
   sync-all:
   	python sync_context.py
   ```

5. Do not introduce any external dependencies beyond what is documented (e.g. `pyyaml`).

6. Do not modify any work repos beyond what is necessary to:

   * Add or adjust `.context-profile.yml`.
   * Generate `agents.md` and `business_context.md`.
   * Optionally create `agents.local.md` / `business_context.local.md` as examples or tests.

```
```
