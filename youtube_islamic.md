# GitHub Manager - Feature Roadmap & Architecture

## New Project Structure

```
ghm/
├── ghm.sh                    # Bash installer/launcher
├── github_manager.py         # Entry point
├── lib/
│   ├── __init__.py
│   ├── api.py                # GitHub API client (existing)
│   ├── colors.py             # ANSI colors (existing)
│   ├── config.py             # Config management (existing)
│   ├── ui.py                 # UI helpers (existing)
│   ├── actions.py            # Basic actions (existing)
│   │
│   │── backup.py             # NEW: Auto-backup & clone all
│   │── health.py             # NEW: Repo health dashboard
│   │── bulk.py               # NEW: Bulk operations
│   │── gitlog.py             # NEW: Interactive git log / graph
│   │── workflows.py          # NEW: GitHub Actions monitor
│   │── templates.py          # NEW: Repo templating
│   │── deps.py               # NEW: Dependency scanner
│   │── releases.py           # NEW: Release manager
│   │── analytics.py          # NEW: Repo analytics & traffic
│   │── sshkeys.py            # NEW: SSH key manager
│   │── search.py             # NEW: Cross-repo search
│   │── timestamps.py         # NEW: Commit timestamp editor
│   └── ai.py                 # NEW: AI commit summaries
│
├── templates/                # Repo templates
│   ├── python-basic/
│   │   ├── README.md
│   │   ├── .gitignore
│   │   ├── setup.py
│   │   └── src/__init__.py
│   ├── esp32-arduino/
│   │   ├── README.md
│   │   ├── .gitignore
│   │   ├── platformio.ini
│   │   └── src/main.cpp
│   └── web-basic/
│       ├── README.md
│       ├── .gitignore
│       ├── index.html
│       └── style.css
│
├── README.md
└── requirements.txt          # requests (+ optional: rich, openai)
```

---

## Updated Menu Structure

```
MAIN MENU
  1   List repos (sorted)
  2   Search repos
  3   Repo details & stats

REPO OPERATIONS
  4   Create new repo
  5   Delete a repo
  6   Edit repo
  7   Clone a repo

GIT INFO
  8   View commits
  9   View branches
  10  Interactive git log          ← NEW

POWER TOOLS
  11  Backup manager               ← NEW
  12  Bulk operations              ← NEW
  13  Repo health dashboard        ← NEW
  14  Cross-repo search            ← NEW

DEVOPS
  15  GitHub Actions monitor       ← NEW
  16  Release manager              ← NEW
  17  Dependency scanner           ← NEW
  18  Repo templates               ← NEW

INSIGHTS
  19  Quick stats (overview)
  20  Repo analytics & traffic     ← NEW
  21  AI commit summaries          ← NEW
  22  Commit rhythm analyzer       ← NEW

GIT TOOLS
  23  Commit timestamp editor      ← NEW
  24  SSH key manager              ← NEW
  25  Git archaeology              ← NEW
  26  Magic diff storyteller       ← NEW

ORGANIZATION
  27  Smart repo grouper           ← NEW
  28  Repo relationship graph      ← NEW (web only)
  29  Danger zone dashboard        ← NEW

SPECIAL
  30  Multi-account manager        ← NEW
  31  ESP32 firmware OTA           ← NEW (web only)

SYSTEM
  32  Settings
  0   Exit

WEB-ONLY MODES (--web flag)
  /timemachine    Git time machine
  /dna            Repo DNA fingerprint
  /radar          Live collaboration radar
  /ambient        Ambient dashboard (full-screen)
  /voice          Voice control
  /portable       Create USB package
```

---

## Feature Specifications

---

### 1. AUTO-BACKUP (backup.py)

**What it does:**
- Clone ALL repos (or filtered set) in one command
- Pull updates for already-cloned repos
- Schedule via cron (Linux) or Task Scheduler (Windows)
- Compression option (zip/tar.gz per repo)

**Menu:**
```
BACKUP MANAGER
  1  Backup all repos now
  2  Backup only private repos
  3  Backup only public repos
  4  Update existing backups (pull)
  5  Compress backups
  6  Schedule automatic backups
  7  View backup history
  0  Back
```

**API calls:**
- GET /user/repos (paginated, all)
- git clone / git pull (subprocess)

**Config additions:**
```json
{
  "backup_directory": "~/github-backups",
  "backup_compress": false,
  "backup_schedule": "weekly",
  "backup_last_run": "2026-02-28T10:00:00Z"
}
```

**Schedule mechanism:**
- Linux: writes crontab entry
- MSYS2: creates Windows Task Scheduler XML + schtasks command
- Logs to ~/.github-manager/backup.log

---

### 2. REPO HEALTH DASHBOARD (health.py)

**What it does:**
- Scans ALL repos for common issues
- Color-coded report card per repo

**Checks:**
| Check               | Pass              | Fail                    |
|---------------------|-------------------|-------------------------|
| Has README          | README.md exists  | Missing                 |
| Has LICENSE         | Any license file  | Missing                 |
| Has .gitignore      | File exists       | Missing                 |
| Has description     | Non-empty         | Empty                   |
| Recent activity     | Updated < 6 months| Stale                   |
| Default branch      | main              | Still using master      |
| Open issues         | < 10              | Too many unresolved     |
| Has topics/tags     | At least 1        | No discoverability      |
| Vulnerability alerts| None              | Has security alerts     |

**Output:**
```
REPO HEALTH DASHBOARD

  abourdim/rxy
    ✓ README   ✓ LICENSE   x .gitignore   ✓ Description
    ✓ Active   ✓ main      ✓ Issues < 10  x No topics
    Score: 6/8  ██████░░ 75%

  abourdim/bit-bot
    ✓ README   x LICENSE   ✓ .gitignore   x Description
    x Stale    x master    ✓ Issues < 10  x No topics
    Score: 3/8  ███░░░░░ 38%

  SUMMARY: 17 repos | avg score 62% | 5 need attention
```

**API calls:**
- GET /repos/{owner}/{repo}/contents/ (check file existence)
- GET /repos/{owner}/{repo} (description, default_branch, updated_at)
- GET /repos/{owner}/{repo}/topics
- GET /repos/{owner}/{repo}/vulnerability-alerts

---

### 3. BULK OPERATIONS (bulk.py)

**What it does:**
- Apply changes to multiple repos at once
- Interactive selection with checkboxes

**Menu:**
```
BULK OPERATIONS
  1  Change visibility (public ↔ private)
  2  Add/update description
  3  Add topics/tags
  4  Delete archived/stale repos
  5  Archive old repos
  6  Enable/disable features (issues, wiki, projects)
  7  Transfer repos to organization
  0  Back
```

**Selection UI:**
```
Select repos (space to toggle, enter to confirm):
  [x] rxy
  [x] bit-bot
  [ ] bitmoji-lab
  [x] tethkir
  [ ] salat-times

  Selected: 3 repos
  Action: Make private

  > Confirm? [y/N]:
```

**API calls:**
- PATCH /repos/{owner}/{repo} (for each selected)
- DELETE /repos/{owner}/{repo} (bulk delete)
- PUT /repos/{owner}/{repo}/topics

---

### 4. INTERACTIVE GIT LOG (gitlog.py)

**What it does:**
- Visual commit graph in terminal (like `git log --graph`)
- Navigate with arrow keys
- Show diffs inline

**Display:**
```
  abourdim/rxy - commit history

  * 3a7f2c1  fix: sensor calibration offset    (2h ago)
  |
  *   b891dc4  Merge branch 'feature/ble'      (1d ago)
  |\
  | * e4c2a01  feat: BLE advertisement packet   (2d ago)
  | * 1f89b03  feat: BLE scan handler            (3d ago)
  |/
  * a0d4e72  chore: update platformio.ini       (4d ago)
  * 7bc1234  docs: add wiring diagram            (1w ago)

  [↑/↓ navigate]  [Enter: show diff]  [q: quit]
```

**Implementation:**
- Uses git log --graph --oneline --all via subprocess
- Parses and colorizes output
- Optional: curses-based scrolling if terminal supports it
- Fallback: paginated plain output

**Requires:** repo must be cloned locally

---

### 5. GITHUB ACTIONS MONITOR (workflows.py)

**What it does:**
- View workflow runs across all repos
- Re-trigger failed builds
- View logs

**Menu:**
```
GITHUB ACTIONS
  1  List recent workflow runs (all repos)
  2  View runs for specific repo
  3  Re-run failed workflow
  4  View workflow logs
  5  List workflows (templates)
  0  Back
```

**Display:**
```
  RECENT WORKFLOW RUNS

  #   Repo              Workflow         Status     Duration   When
  1   rxy               CI Build         ✓ pass     2m 31s     3h ago
  2   bit-bot           Deploy           x fail     0m 45s     5h ago
  3   tethkir           Test Suite       ✓ pass     1m 12s     1d ago
  4   salat-times       Lint             ~ running  --         now

  > Re-run #2? [y/N]:
```

**API calls:**
- GET /repos/{owner}/{repo}/actions/runs
- POST /repos/{owner}/{repo}/actions/runs/{id}/rerun
- GET /repos/{owner}/{repo}/actions/runs/{id}/logs

---

### 6. REPO TEMPLATING (templates.py)

**What it does:**
- Create repos from predefined templates
- Custom templates stored in templates/ folder
- Variable substitution (project name, author, year)

**Menu:**
```
REPO TEMPLATES
  1  Create repo from template
  2  List available templates
  3  Create new template from existing repo
  4  Edit template
  0  Back
```

**Built-in templates:**
```
  Available templates:

  1  python-basic     Python project with setup.py, tests, CI
  2  esp32-arduino    ESP32 PlatformIO project
  3  web-basic        HTML/CSS/JS starter
  4  node-express     Node.js Express API
  5  docs-only        Documentation repo (mkdocs)

  > Choose template: 2
  > Repo name: smart-garden
  > Description: ESP32 plant monitoring system
  > Private? [Y/n]: y

  Creating smart-garden from esp32-arduino template...
  ✓ Repo created
  ✓ Template files pushed
  ✓ Done: https://github.com/abourdim/smart-garden
```

**Implementation:**
- Creates repo via API
- Pushes template files via git (subprocess)
- Replaces {{PROJECT_NAME}}, {{AUTHOR}}, {{YEAR}} in files

---

### 7. DEPENDENCY SCANNER (deps.py)

**What it does:**
- Scans repos for dependency files
- Checks for outdated packages
- Security vulnerability check via GitHub Advisory API

**Supported files:**
| File                 | Ecosystem |
|----------------------|-----------|
| requirements.txt     | Python    |
| setup.py / pyproject.toml | Python |
| package.json         | Node.js   |
| platformio.ini       | C/C++ IoT |
| Cargo.toml           | Rust      |

**Display:**
```
  DEPENDENCY SCAN: abourdim/rxy

  requirements.txt:
    requests     2.28.0 → 2.31.0    ⚠ update available
    flask        2.3.0  → 3.0.1     ⚠ major update
    numpy        1.24.0              ✓ up to date

  package.json:
    express      4.18.0 → 4.19.2    ⚠ security fix
    lodash       4.17.21             ✓ up to date

  SECURITY ALERTS: 1 critical, 2 moderate
```

**API calls:**
- GET /repos/{owner}/{repo}/contents/requirements.txt (etc.)
- GET /repos/{owner}/{repo}/vulnerability-alerts
- pip index versions {package} (or PyPI JSON API)
- registry.npmjs.org/{package}/latest

---

### 8. RELEASE MANAGER (releases.py)

**What it does:**
- Create GitHub releases with tags
- Upload binary assets (firmware .bin, .hex, .zip)
- Auto-generate changelogs from commits
- Draft and publish releases

**Menu:**
```
RELEASE MANAGER
  1  List releases for repo
  2  Create new release
  3  Upload asset to release
  4  Auto-generate changelog
  5  Delete a release
  0  Back
```

**Create flow:**
```
  > Repo: rxy
  > Tag (e.g. v1.2.0): v1.3.0
  > Title [v1.3.0]: v1.3.0 - BLE Support
  > Auto-generate changelog from last tag? [Y/n]: y

  Changelog (since v1.2.0):
    - feat: BLE advertisement packet
    - feat: BLE scan handler
    - fix: sensor calibration offset

  > Draft or publish? [d/P]: p
  > Upload assets? [y/N]: y
  > File path: ./build/firmware.bin

  ✓ Release v1.3.0 published
  ✓ firmware.bin uploaded (245 KB)
```

**API calls:**
- GET /repos/{owner}/{repo}/releases
- POST /repos/{owner}/{repo}/releases
- POST /repos/{owner}/{repo}/releases/{id}/assets
- GET /repos/{owner}/{repo}/compare/{base}...{head}

---

### 9. REPO ANALYTICS & TRAFFIC (analytics.py)

**What it does:**
- Clone/visitor traffic (last 14 days, API limit)
- Star history over time
- Referral sources
- Popular content (paths)
- Contribution heatmap in terminal

**Display:**
```
  ANALYTICS: abourdim/rxy

  TRAFFIC (last 14 days):
    Views:    342 (87 unique)
    Clones:   28 (12 unique)

    Daily views:
    Mon ██████████████ 48
    Tue ████████████ 41
    Wed █████████ 30
    ...

  TOP REFERRERS:
    google.com        52 views
    github.com        31 views
    reddit.com        18 views

  POPULAR CONTENT:
    /README.md        89 views
    /src/main.cpp     34 views

  CONTRIBUTION HEATMAP (last year):
    Jan ░░█░░█░  Feb ██░███░  Mar █░░░██░ ...
```

**API calls:**
- GET /repos/{owner}/{repo}/traffic/views
- GET /repos/{owner}/{repo}/traffic/clones
- GET /repos/{owner}/{repo}/traffic/popular/referrers
- GET /repos/{owner}/{repo}/traffic/popular/paths
- GET /repos/{owner}/{repo}/stats/commit_activity
- GET /repos/{owner}/{repo}/stargazers (with Accept: application/vnd.github.star+json)

**Note:** Traffic data requires push access and only covers last 14 days.

---

### 10. SSH KEY MANAGER (sshkeys.py)

**What it does:**
- Generate ED25519/RSA SSH keys
- Upload public key to GitHub
- Test SSH connection
- List/remove keys from GitHub

**Menu:**
```
SSH KEY MANAGER
  1  List SSH keys on GitHub
  2  Generate new SSH key pair
  3  Upload public key to GitHub
  4  Test SSH connection
  5  Remove SSH key from GitHub
  6  Show local public key
  0  Back
```

**Generate flow:**
```
  > Key type (ed25519/rsa) [ed25519]: ed25519
  > Email for key: karim@workshop-diy.org
  > Passphrase (empty for none):

  Generating key...
  ✓ Private key: ~/.ssh/id_ed25519_github
  ✓ Public key:  ~/.ssh/id_ed25519_github.pub

  > Upload to GitHub now? [Y/n]: y
  > Key title [abourdim-msys2-2026]: my-workstation

  ✓ Key uploaded to GitHub
  ✓ SSH test: Hi abourdim! You've authenticated.
```

**API calls:**
- GET /user/keys
- POST /user/keys
- DELETE /user/keys/{id}
- ssh -T git@github.com (subprocess for test)
- ssh-keygen (subprocess for generation)

---

### 11. CROSS-REPO SEARCH (search.py)

**What it does:**
- Search code across ALL your repos
- Search filenames, file contents, commit messages
- Regex support

**Menu:**
```
CROSS-REPO SEARCH
  1  Search code (file contents)
  2  Search filenames
  3  Search commit messages
  4  Search issues & PRs
  0  Back
```

**Display:**
```
  > Search code: WiFi.begin

  Results (5 matches in 3 repos):

  abourdim/rxy
    src/wifi_manager.cpp:42    WiFi.begin(ssid, password);
    src/wifi_manager.cpp:78    WiFi.begin(AP_SSID);

  abourdim/smart-garden
    src/main.cpp:15            WiFi.begin(WIFI_SSID, WIFI_PASS);

  abourdim/bit-bot
    lib/network.cpp:23         WiFi.begin(config.ssid);
    lib/network.cpp:89         WiFi.begin();
```

**API calls:**
- GET /search/code?q=user:{username}+{query}
- GET /search/commits?q=author:{username}+{query}
- GET /search/issues?q=user:{username}+{query}

---

### 12. AI COMMIT SUMMARIES (ai.py)

**What it does:**
- Summarize what changed across repos in a time period
- Weekly digest email-style report
- Uses OpenAI API or local LLM (optional)
- Falls back to basic diff summary without AI

**Menu:**
```
AI SUMMARIES
  1  This week's activity summary
  2  Summarize specific repo (last N commits)
  3  Compare two branches (plain English)
  4  Generate PR description from diff
  0  Back
```

**Display (no AI fallback):**
```
  WEEKLY SUMMARY (Feb 22 - Feb 28, 2026)

  Active repos: 3 of 17

  abourdim/rxy (8 commits)
    + 4 files changed, 156 insertions, 23 deletions
    Main changes:
    - BLE support added (3 commits)
    - Sensor calibration fix
    - PlatformIO config update

  abourdim/salat-times (2 commits)
    + 1 file changed, 12 insertions, 5 deletions
    Main changes:
    - Prayer time calculation fix

  abourdim/tethkir (1 commit)
    + 3 files changed, 45 insertions
    Main changes:
    - New dhikr entries added
```

**With AI (optional):**
```
  > Summarize this week

  This week you focused primarily on your rxy project, adding
  Bluetooth Low Energy support with advertisement and scan
  capabilities. You also fixed a sensor calibration offset that
  was causing incorrect readings. Minor updates were made to
  salat-times (prayer calculation accuracy) and tethkir (new
  content). Overall: 11 commits across 3 repos.
```

**Config additions:**
```json
{
  "ai_provider": "none",
  "openai_api_key": "",
  "ai_model": "gpt-4o-mini"
}
```

**API calls:**
- GET /repos/{owner}/{repo}/commits?since={date}
- GET /repos/{owner}/{repo}/compare/{base}...{head}
- OpenAI API (optional, only if configured)

---

### 13. COMMIT TIMESTAMP EDITOR (timestamps.py)

**What it does:**
- View and change author/committer dates on any commit
- Interactive commit picker (list recent, select by hash)
- Bulk shift: move all commits by offset (e.g. +3 hours)
- Spread commits: redistribute timestamps evenly across a time range
- Preset patterns: make commits look like working hours, specific day, etc.
- Works on local cloned repos (subprocess git commands)
- Optional force-push to remote after editing

**Menu:**
```
COMMIT TIMESTAMP EDITOR
  1  Edit single commit date
  2  Edit last N commits
  3  Bulk shift (offset all commits by ±hours/days)
  4  Spread commits evenly across time range
  5  Set all commits to specific time pattern
  6  View commit timestamps
  7  Force push changes to remote
  0  Back
```

**Flow — Edit single commit:**
```
  > Repo path (or clone name): ~/repos/rxy
  
  Recent commits:
   #  Hash     Date                 Message
   1  3a7f2c1  2026-02-28 15:30    fix: sensor calibration
   2  b891dc4  2026-02-27 10:15    Merge branch 'feature/ble'
   3  e4c2a01  2026-02-26 22:41    feat: BLE advertisement
   4  1f89b03  2026-02-25 09:00    feat: BLE scan handler
   5  a0d4e72  2026-02-24 14:20    chore: update platformio

  > Commit to edit [1]: 3
  > New date (YYYY-MM-DD HH:MM): 2026-02-26 14:00
  > Change author date too? [Y/n]: y

  ✓ Commit e4c2a01 updated
    Old: 2026-02-26 22:41
    New: 2026-02-26 14:00
```

**Flow — Bulk shift:**
```
  > Repo path: ~/repos/rxy
  > How many commits from HEAD? [all]: 10
  > Shift by: -3h          (examples: +2h, -1d, +30m, -2d12h)

  Preview:
    3a7f2c1  15:30 → 12:30  fix: sensor calibration
    b891dc4  10:15 → 07:15  Merge branch 'feature/ble'
    e4c2a01  22:41 → 19:41  feat: BLE advertisement
    ...

  > Apply? [y/N]: y
  ✓ 10 commits shifted by -3 hours
```

**Flow — Spread evenly:**
```
  > Repo path: ~/repos/rxy
  > How many commits from HEAD? 5
  > Start date: 2026-02-24 09:00
  > End date:   2026-02-28 17:00
  > Weekdays only? [Y/n]: y
  > Working hours only (9-18)? [Y/n]: y

  Preview:
    a0d4e72  → 2026-02-24 09:00  chore: update platformio
    1f89b03  → 2026-02-24 14:30  feat: BLE scan handler
    e4c2a01  → 2026-02-25 10:00  feat: BLE advertisement
    b891dc4  → 2026-02-26 11:30  Merge branch 'feature/ble'
    3a7f2c1  → 2026-02-27 14:00  fix: sensor calibration

  > Apply? [y/N]: y
  ✓ 5 commits spread across Feb 24-28 (weekdays, 9AM-6PM)
```

**Flow — Time patterns:**
```
  PRESET PATTERNS
   1  Working hours (Mon-Fri, 9:00-18:00, random spread)
   2  Night owl (20:00-03:00)
   3  Specific day (all commits on one date, spaced 10-45min apart)
   4  Custom cron pattern

  > Pattern [1]: 1
  > Week of: 2026-02-24
  > Apply to last N commits: 8

  ✓ 8 commits redistributed across Mon-Fri 9AM-6PM
```

**Implementation details:**
- Uses `git log --format="%H %aI %s"` to read commits
- Uses `git filter-branch --env-filter` or `git rebase -i` with GIT_COMMITTER_DATE / GIT_AUTHOR_DATE
- Prefers `git filter-repo` if installed (faster, safer) with fallback to filter-branch
- Parses offset strings: `+3h`, `-1d`, `+2d6h30m`
- Random jitter option for natural-looking timestamps
- Safety: creates backup branch before any rewrite
- Warning: explains force-push implications

**Safety measures:**
```
  ⚠  WARNING: Rewriting commit history!
  
  - This changes commit hashes
  - Requires force-push to update remote
  - Collaborators will need to re-clone or reset
  - A backup branch 'backup/pre-timestamp-edit' will be created
  
  > Continue? [y/N]:
```

**Subprocess commands used:**
```bash
# Read commits
git log --format="%H %aI %cI %s" HEAD~N..HEAD

# Edit single commit (via rebase)
GIT_COMMITTER_DATE="$DATE" git commit --amend --date="$DATE" --no-edit

# Bulk rewrite (filter-branch)
git filter-branch --env-filter '
if [ "$GIT_COMMIT" = "abc123..." ]; then
    export GIT_AUTHOR_DATE="2026-02-26T14:00:00"
    export GIT_COMMITTER_DATE="2026-02-26T14:00:00"
fi
' HEAD~N..HEAD

# Safer alternative (git filter-repo, if installed)
git filter-repo --commit-callback '
commit.author_date = b"1709042400 +0100"
commit.committer_date = b"1709042400 +0100"
'

# Force push
git push --force-with-lease

# Backup
git branch backup/pre-timestamp-edit
```

**Requires:** repo must be cloned locally (prompts clone if not)

---

## Implementation Priority

| Phase | Features                              | Complexity |
|-------|---------------------------------------|------------|
| 1     | Backup, Health, Bulk ops              | Medium     |
| 2     | Git log, Cross-repo search            | Medium     |
| 3     | Actions monitor, Release manager      | Medium     |
| 4     | Templates, SSH keys, Timestamp editor | Low-Medium |
| 5     | Analytics, Dependency scanner         | Medium     |
| 6     | AI summaries                          | Medium     |
| 7     | Web UI mode                           | Medium-High|

---

## WEB UI MODE

### Overview

A local web server (`localhost:5000`) that reuses all existing `lib/` modules.
Launched with `python github_manager.py --web` or from the bash launcher.
Zero deployment, zero extra dependencies — Python built-in `http.server`.

### Why

- Works on locked-down PCs (only needs Python + browser, zero extra install)
- No admin, no terminal skills required
- Calendar date picker for timestamp editor (instead of typing dates)
- Real commit graph visualization (SVG/canvas, not ASCII)
- Checkboxes for bulk operations (instead of space-to-toggle)
- Accessible from phone/tablet on same LAN if needed
- Can be used alongside CLI — same config, same token
- Enables features impossible in terminal: time machine slider, DNA fingerprints, voice control, ambient display

### Project Structure (additions)

```
ghm/
├── github_manager.py         # Add --web flag entry point
├── lib/
│   └── (all existing modules)
│   │── backup.py
│   │── health.py
│   │── bulk.py
│   │── gitlog.py
│   │── workflows.py
│   │── templates.py
│   │── deps.py
│   │── releases.py
│   │── analytics.py
│   │── sshkeys.py
│   │── search.py
│   │── timestamps.py
│   │── ai.py
│   │── rhythm.py             # NEW: Commit rhythm analyzer
│   │── grouper.py            # NEW: Smart repo grouper
│   │── archaeology.py        # NEW: Git archaeology
│   │── danger.py             # NEW: Danger zone scanner
│   │── storyteller.py        # NEW: Magic diff storyteller
│   └── accounts.py           # NEW: Multi-account support
│
├── web/
│   ├── __init__.py
│   ├── server.py             # HTTPServer + GHMHandler (built-in)
│   ├── router.py             # URL → handler dispatch
│   ├── templating.py         # {{var}} template engine (no Jinja2)
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── dashboard.py      # GET /
│   │   ├── repos.py          # /repos
│   │   ├── backup.py         # /backup
│   │   ├── health.py         # /health
│   │   ├── bulk.py           # /bulk
│   │   ├── workflows.py      # /workflows
│   │   ├── releases.py       # /releases
│   │   ├── templates_rt.py   # /templates
│   │   ├── deps.py           # /deps
│   │   ├── analytics.py      # /analytics
│   │   ├── search.py         # /search
│   │   ├── timestamps.py     # /timestamps
│   │   ├── ssh.py            # /ssh
│   │   ├── ai.py             # /ai
│   │   ├── timemachine.py    # /timemachine        ← NEW
│   │   ├── dna.py            # /dna                ← NEW
│   │   ├── radar.py          # /radar              ← NEW
│   │   ├── archaeology.py    # /archaeology         ← NEW
│   │   ├── danger.py         # /danger              ← NEW
│   │   ├── ambient.py        # /ambient             ← NEW
│   │   ├── ota.py            # /ota                 ← NEW
│   │   ├── graph.py          # /graph               ← NEW
│   │   └── settings.py       # /settings
│   │
│   ├── static/
│   │   ├── css/
│   │   │   └── style.css     # Single stylesheet, dark theme
│   │   ├── js/
│   │   │   ├── app.js        # Fetch helpers, toasts, SPA router
│   │   │   ├── graph.js      # D3.js commit/repo graph
│   │   │   ├── calendar.js   # Flatpickr date picker wrapper
│   │   │   ├── bulk.js       # Checkbox select-all
│   │   │   ├── timemachine.js# Slider + file tree rendering
│   │   │   ├── voice.js      # Speech recognition commands
│   │   │   ├── radar.js      # SSE live event feed
│   │   │   └── dna.js        # SVG fingerprint generator
│   │   └── img/
│   │       └── favicon.ico
│   │
│   └── templates/
│       ├── base.html         # Layout: sidebar + content area
│       ├── dashboard.html
│       ├── repos/
│       │   ├── list.html
│       │   ├── detail.html
│       │   ├── create.html
│       │   └── edit.html
│       ├── backup.html
│       ├── health.html
│       ├── bulk.html
│       ├── workflows.html
│       ├── releases.html
│       ├── templates.html
│       ├── deps.html
│       ├── analytics.html
│       ├── search.html
│       ├── timestamps.html
│       ├── ssh.html
│       ├── ai.html
│       ├── timemachine.html
│       ├── dna.html
│       ├── radar.html
│       ├── archaeology.html
│       ├── danger.html
│       ├── ambient.html      # Full-screen, no sidebar
│       ├── ota.html
│       ├── graph.html
│       └── settings.html
│
├── templates/                # Repo boilerplate templates
│   ├── python-basic/
│   ├── esp32-arduino/
│   └── web-basic/
│
├── README.md
└── portable/                 # Portable USB mode files
    ├── run.bat
    └── run.sh
```

### Architecture

```
Browser (localhost:5000)
   │
   ├── Static files (CSS/JS/images) served directly
   │
   ├── Page routes → HTML templates with {{data}}
   │
   └── /api/ routes → JSON responses
         │
         ▼
   web/router.py dispatches to web/routes/*.py
         │
         ▼
   Shared lib/ modules (api.py, config.py, etc.)
         │
         ▼
   GitHub API + local git (subprocess)
```

Key principle: **routes are thin**. They call the same `lib/` functions as the CLI,
then render HTML templates with string formatting. No business logic in routes.

### Entry Point

```python
# github_manager.py additions:
import sys

if "--web" in sys.argv:
    from web.server import start_server
    port = 5000
    lan = "--lan" in sys.argv
    host = "0.0.0.0" if lan else "127.0.0.1"
    print(f"  + GitHub Manager Web UI: http://localhost:{port}")
    import webbrowser
    webbrowser.open(f"http://localhost:{port}")
    start_server(host=host, port=port)
else:
    # existing CLI main()
    main()
```

### Web Server (web/server.py)

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json
import urllib.parse

class GHMHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        parsed = urllib.parse.urlparse(self.path)
        path = parsed.path

        # Static files
        if path.startswith("/static/"):
            self._serve_static(path)
        # API routes return JSON
        elif path.startswith("/api/"):
            self._handle_api(path, parsed.query)
        # Page routes return HTML
        else:
            self._handle_page(path)

    def do_POST(self):
        # Read body, parse JSON, dispatch to API handler
        ...

    def _serve_static(self, path):
        # Serve CSS/JS/images from web/static/
        ...

    def _handle_page(self, path):
        # Load HTML template, inject data, send response
        ...

    def _handle_api(self, path, query):
        # Call lib/ functions, return JSON
        ...

def start_server(host="127.0.0.1", port=5000):
    server = HTTPServer((host, port), GHMHandler)
    server.serve_forever()
```

### Template Engine

Simple string-based templating (no Jinja2 needed):
```python
def render(template_name: str, **context) -> str:
    """Load HTML template and replace {{variable}} placeholders."""
    path = f"web/templates/{template_name}"
    with open(path) as f:
        html = f.read()
    for key, value in context.items():
        html = html.replace("{{" + key + "}}", str(value))
    return html
```

Supports `{{variable}}` replacement and `<!-- LOOP:items -->...<!-- ENDLOOP -->` blocks.

### ghm.sh Addition

```
MENU
  ...
  3   Launch (CLI)
  4   Launch (Web UI)              ← NEW
  ...
```

```bash
action_launch_web() {
    # same checks as CLI launch
    $PYTHON_CMD "$APP_SCRIPT" --web
}
```

### Page Specs

---

#### Dashboard (GET /)

The landing page. Quick glance at everything.

```
+----------------------------------------------------------+
|  GitHub Manager                    abourdim   [Settings]  |
+----------+-----------------------------------------------+
|          |                                                |
| SIDEBAR  |   DASHBOARD                                   |
|          |                                                |
| Dashboard|   +----------+ +----------+ +----------+      |
| Repos    |   | 17 Repos | | 142 ★    | | 3 Active |     |
| Backup   |   +----------+ +----------+ +----------+      |
| Health   |                                                |
| Bulk     |   RECENT ACTIVITY                              |
| Actions  |   ┌─────────────────────────────────────┐      |
| Releases |   │ rxy          fix: sensor cal.  2h   │      |
| Templates|   │ salat-times  prayer fix         1d   │      |
| Deps     |   │ tethkir      new dhikr          3d   │      |
| Analytics|   └─────────────────────────────────────┘      |
| Search   |                                                |
| Timestamp|   HEALTH OVERVIEW                              |
| SSH Keys |   ██████████░░░░ 71% avg score                 |
| AI       |   5 repos need attention                       |
|          |                                                |
+----------+-----------------------------------------------+
```

**Cards:**
- Total repos (public/private split)
- Total stars
- Active repos this week
- Health score average
- Pending workflow failures

**Recent activity:** last 5 pushes across all repos.
**Quick links:** to health issues, failed workflows.

---

#### Repos List (/repos)

```
+----------------------------------------------------------+
| REPOSITORIES                                              |
|                                                           |
| Sort: [Updated ▼]  Type: [All ▼]  Lang: [All ▼]  🔍 ___ |
|                                                           |
| ┌──┬────────────────┬─────────┬──────────┬───┬──────────┐ |
| │  │ Name           │ Vis     │ Updated  │ ★ │ Language  │ |
| ├──┼────────────────┼─────────┼──────────┼───┼──────────┤ |
| │  │ rxy            │ 🟢 pub  │ 2h ago   │ 3 │ C++      │ |
| │  │ bit-bot        │ 🔴 priv │ 5d ago   │ 0 │ Python   │ |
| │  │ salat-times    │ 🟢 pub  │ 1d ago   │ 1 │ JS       │ |
| └──┴────────────────┴─────────┴──────────┴───┴──────────┘ |
|                                                           |
| [+ Create New Repo]                                       |
+----------------------------------------------------------+
```

- Sortable columns (click header)
- Filter dropdowns (type, language)
- Search box (instant filter)
- Click repo name → detail page
- Create button → modal form

---

#### Repo Detail (/repos/<name>)

```
+----------------------------------------------------------+
| ← Back    rxy                          🟢 public         |
|                                                           |
| ESP32 sensor platform with BLE                            |
| https://github.com/abourdim/rxy                           |
|                                                           |
| Created: 2024-03-15    Updated: 2h ago    Size: 1.2 MB   |
| Stars: 3  Forks: 1  Issues: 2  Branch: main              |
|                                                           |
| LANGUAGES                                                 |
| C++      ████████████████████░░░░  78.3%                  |
| Python   ████░░░░░░░░░░░░░░░░░░░  15.2%                  |
| Shell    █░░░░░░░░░░░░░░░░░░░░░░   6.5%                  |
|                                                           |
| RECENT COMMITS                                            |
| 3a7f2c1  fix: sensor calibration         2h ago           |
| b891dc4  Merge branch 'feature/ble'      1d ago           |
| e4c2a01  feat: BLE advertisement          2d ago           |
|                                                           |
| [Edit] [Clone] [Releases] [Analytics] [Delete]            |
+----------------------------------------------------------+
```

---

#### Timestamp Editor (/timestamps)

The killer feature that really benefits from web UI.

```
+----------------------------------------------------------+
| COMMIT TIMESTAMP EDITOR                                   |
|                                                           |
| Repo: [rxy              ▼]   [Load Commits]              |
|                                                           |
| MODE: ○ Single  ○ Bulk shift  ● Spread  ○ Pattern        |
|                                                           |
| ┌──┬─────────┬─────────────────────┬────────────────────┐ |
| │☑ │ Hash    │ Current Date        │ New Date           │ |
| ├──┼─────────┼─────────────────────┼────────────────────┤ |
| │☑ │ 3a7f2c1 │ 2026-02-28 15:30   │ 📅 2026-02-28 14:00│ |
| │☑ │ b891dc4 │ 2026-02-27 10:15   │ 📅 2026-02-27 11:30│ |
| │☑ │ e4c2a01 │ 2026-02-26 22:41   │ 📅 2026-02-26 10:00│ |
| │☐ │ 1f89b03 │ 2026-02-25 09:00   │                    │ |
| │☐ │ a0d4e72 │ 2026-02-24 14:20   │                    │ |
| └──┴─────────┴─────────────────────┴────────────────────┘ |
|                                                           |
| SPREAD SETTINGS                                           |
| Start: 📅 2026-02-24 09:00                                |
| End:   📅 2026-02-28 17:00                                |
| ☑ Weekdays only  ☑ Working hours (9-18)  ☑ Random jitter |
|                                                           |
| [Preview Changes]  [Apply]  [Force Push to Remote]        |
+----------------------------------------------------------+
```

**Why web wins here:**
- Calendar date pickers (📅) instead of typing ISO dates
- Checkboxes to select commits visually
- Side-by-side old vs new dates
- Preview before apply
- Mode tabs (single/shift/spread/pattern) are natural in HTML

---

#### Health Dashboard (/health)

```
+----------------------------------------------------------+
| REPO HEALTH                          Overall: 71% ████▓░ |
|                                                           |
| ┌────────────────┬───┬───┬───┬───┬───┬───┬───┬───┬─────┐ |
| │ Repo           │RM │LIC│GIT│DES│ACT│BRN│ISS│TOP│Score│ |
| ├────────────────┼───┼───┼───┼───┼───┼───┼───┼───┼─────┤ |
| │ rxy            │ ✓ │ ✓ │ ✓ │ ✓ │ ✓ │ ✓ │ ✓ │ ✗ │ 88% │ |
| │ bit-bot        │ ✓ │ ✗ │ ✓ │ ✗ │ ✗ │ ✗ │ ✓ │ ✗ │ 38% │ |
| │ salat-times    │ ✓ │ ✓ │ ✓ │ ✓ │ ✓ │ ✓ │ ✓ │ ✓ │100% │ |
| └────────────────┴───┴───┴───┴───┴───┴───┴───┴───┴─────┘ |
|                                                           |
| RM=README LIC=License GIT=.gitignore DES=Description      |
| ACT=Active BRN=main ISS=Issues<10 TOP=Has topics          |
|                                                           |
| NEEDS ATTENTION (score < 60%):                            |
|   bit-bot, face-quest, ble-logger, cowsay, tesbih         |
|                                                           |
| [Fix All: Add missing .gitignore] [Fix All: Add topics]  |
+----------------------------------------------------------+
```

**Web advantage:** "Fix All" buttons that batch-fix common issues.

---

#### Analytics (/analytics/<name>)

```
+----------------------------------------------------------+
| ANALYTICS: rxy                                            |
|                                                           |
| TRAFFIC (last 14 days)                                    |
| Views: 342 (87 unique)  Clones: 28 (12 unique)           |
|                                                           |
| [========== Chart.js line graph ===========]              |
| |     ·                                                   |
| |    · ·    ·                                             |
| |   ·   ·  · ·   ·                                       |
| |  ·     ··    · · ·                                      |
| | ·              ·   ·                                    |
| +---+---+---+---+---+---+---+                            |
|   Mon Tue Wed Thu Fri Sat Sun                             |
|                                                           |
| — Views  ··· Clones                                       |
|                                                           |
| TOP REFERRERS              POPULAR PATHS                  |
| google.com      52         /README.md        89           |
| github.com      31         /src/main.cpp     34           |
| reddit.com      18         /docs/setup.md    21           |
|                                                           |
| CONTRIBUTION HEATMAP                                      |
| [========== GitHub-style heatmap grid ===========]        |
+----------------------------------------------------------+
```

**Web advantage:** real interactive Chart.js charts, GitHub-style heatmap SVG.

---

#### Bulk Operations (/bulk)

```
+----------------------------------------------------------+
| BULK OPERATIONS                                           |
|                                                           |
| Action: [Make Private  ▼]                                 |
|                                                           |
| ☑ Select All                                              |
| ┌──┬──────────────────┬──────────┬───────────┬──────────┐ |
| │☑ │ rxy              │ 🟢 pub   │ C++       │ 2h ago   │ |
| │☑ │ bit-bot          │ 🟢 pub   │ Python    │ 5d ago   │ |
| │☐ │ salat-times      │ 🔴 priv  │ JS        │ 1d ago   │ |
| │☑ │ face-quest       │ 🟢 pub   │ JS        │ 2w ago   │ |
| │☐ │ tethkir          │ 🔴 priv  │ Dart      │ 3d ago   │ |
| └──┴──────────────────┴──────────┴───────────┴──────────┘ |
|                                                           |
| Selected: 3 repos                                         |
| [Preview] [Apply to Selected]                             |
+----------------------------------------------------------+
```

---

#### GitHub Actions (/workflows)

```
+----------------------------------------------------------+
| GITHUB ACTIONS                                            |
|                                                           |
| Filter: [All repos ▼]  Status: [All ▼]                   |
|                                                           |
| ┌────────────────┬──────────────┬────────┬───────┬──────┐ |
| │ Repo           │ Workflow     │ Status │ Time  │      │ |
| ├────────────────┼──────────────┼────────┼───────┼──────┤ |
| │ rxy            │ CI Build     │ ✓ pass │ 2m31s │      │ |
| │ bit-bot        │ Deploy       │ ✗ fail │ 0m45s │[⟳]  │ |
| │ tethkir        │ Test Suite   │ ✓ pass │ 1m12s │      │ |
| │ salat-times    │ Lint         │ ◷ run  │ --    │      │ |
| └────────────────┴──────────────┴────────┴───────┴──────┘ |
|                                                           |
| [⟳] = Click to re-trigger                                |
+----------------------------------------------------------+
```

---

### UI Design Decisions

**Branding — every page, always:**

```
+----------------------------------------------------------+
|     بسم الله الرحمن الرحيم                                 |
|                                                           |
| GitPulse                       Powered by workshop-diy.org|
+----------------------------------------------------------+
|                                                           |
|  (page content)                                           |
|                                                           |
+----------------------------------------------------------+
```

- **Bismillah** (`بسم الله الرحمن الرحيم`) always displayed centered at the very top of every page, above everything. Arabic text, elegant font (Amiri or Scheherazade from Google Fonts CDN). Never hidden, never scrolled away — fixed or part of the top bar.
- **Header bar** below bismillah: app name left, "Powered by workshop-diy.org" right (linked to http://workshop-diy.org).
- Both present on every page including ambient mode, OTA page, settings.
- In CLI mode: bismillah shown in the banner on launch.

```
  بسم الله الرحمن الرحيم

  +----------------------------------------------------------+
  |              GitPulse v0.1.0                              |
  |              Powered by workshop-diy.org                  |
  +----------------------------------------------------------+
```

**Languages — full i18n (EN / FR / AR):**

- Three languages: English, French, Arabic
- Language switcher in header (🌐 EN | FR | AR)
- Arabic activates RTL layout automatically (CSS `direction: rtl`)
- All UI labels, menu items, toasts, error messages translated
- Dates formatted per locale (FR: 28/02/2026, AR: ٢٠٢٦/٠٢/٢٨)
- Stored in config: `"language": "en"`
- Translation files: `web/i18n/en.json`, `fr.json`, `ar.json`
- CLI: same 3 languages, loaded from `lib/i18n/`

```json
// web/i18n/en.json
{
  "dashboard": "Dashboard",
  "repos": "Repositories",
  "settings": "Settings",
  "create": "Create",
  "delete": "Delete",
  "search": "Search...",
  "powered_by": "Powered by workshop-diy.org"
}

// web/i18n/fr.json
{
  "dashboard": "Tableau de bord",
  "repos": "Dépôts",
  "settings": "Paramètres",
  "create": "Créer",
  "delete": "Supprimer",
  "search": "Rechercher...",
  "powered_by": "Propulsé par workshop-diy.org"
}

// web/i18n/ar.json
{
  "dashboard": "لوحة التحكم",
  "repos": "المستودعات",
  "settings": "الإعدادات",
  "create": "إنشاء",
  "delete": "حذف",
  "search": "...بحث",
  "powered_by": "بدعم من workshop-diy.org"
}
```

---

**7 Islamic Art Themes:**

Theme selector in settings. Each theme changes: colors, background patterns, borders, sidebar style, font accents. Bismillah always present, styled to match theme.

---

**Theme 1: Al-Andalus (الأندلس)**
Inspired by Moorish Spain — warm golds, deep reds, intricate geometric tilework.

```css
/* Andalusian zellige tile patterns */
--bg:           #1a0f07;           /* dark walnut */
--surface:      #2a1a0e;           /* warm mahogany */
--border:       #8b6914;           /* antique gold */
--text:         #f0e6d3;           /* parchment */
--accent:       #d4a030;           /* Andalusian gold */
--accent2:      #c0392b;           /* Cordoba red */
--green:        #6b8e23;           /* olive grove */
/* Background: subtle repeating geometric star pattern (SVG) */
/* Borders: scalloped arches on cards */
/* Sidebar: dark with gold trim lines */
```

---

**Theme 2: Masjid (المسجد)**
Inspired by mosque interiors — calm, spiritual, turquoise domes, white marble.

```css
/* Serene mosque interior */
--bg:           #0a1628;           /* night sky */
--surface:      #112240;           /* deep blue */
--border:       #1e90ff;           /* dome turquoise */
--text:         #e8f0fe;           /* moonlight white */
--accent:       #00b4d8;           /* turquoise */
--accent2:      #48cae4;           /* light cyan */
--green:        #2dc653;           /* minaret green */
/* Background: subtle dome arch silhouette watermark */
/* Cards: rounded tops like mihrab arches */
/* Sidebar: marble texture gradient */
```

---

**Theme 3: Al-Hamra (الحمراء)**
Inspired by Alhambra palace — terracotta, intricate muqarnas, lush garden greens.

```css
/* Alhambra palace gardens */
--bg:           #1c1208;           /* dark earth */
--surface:      #2d1f10;           /* terracotta dark */
--border:       #cd853f;           /* terracotta */
--text:         #faebd7;           /* antique white */
--accent:       #e07020;           /* burnt orange */
--accent2:      #228b22;           /* garden green */
--green:        #32cd32;           /* fountain green */
/* Background: muqarnas honeycomb pattern (SVG) */
/* Headers: arabesques flourish underlines */
/* Cards: terracotta border with green accent line top */
```

---

**Theme 4: Nūr (نور) — Light Calligraphy I**
Minimalist. White and gold. Inspired by illuminated Quran manuscripts.

```css
/* Illuminated manuscript — light mode */
--bg:           #fdf8f0;           /* aged paper */
--surface:      #ffffff;           /* pure white */
--border:       #d4a030;           /* gold ink */
--text:         #2c1810;           /* dark sepia */
--accent:       #b8860b;           /* dark gold */
--accent2:      #1e3a5f;           /* lapis lazuli */
--green:        #2e7d32;           /* malachite */
/* Background: faint geometric rosette watermark center */
/* Bismillah: ornate gold frame around it */
/* Cards: thin gold border, slight paper texture */
/* Font accent: Amiri for headers */
```

---

**Theme 5: Raqsh (رقش) — Light Calligraphy II**
Flowing, organic. Cream and indigo. Inspired by Ottoman tulip-era calligraphy.

```css
/* Ottoman calligraphic flow — light mode */
--bg:           #f5f0e8;           /* warm cream */
--surface:      #fefcf7;           /* soft white */
--border:       #4a5568;           /* ink grey */
--text:         #1a202c;           /* deep charcoal */
--accent:       #2b4c7e;           /* Ottoman indigo */
--accent2:      #c62828;           /* tulip red */
--green:        #388e3c;           /* ceramic green */
/* Background: flowing calligraphic swirl watermark (opacity 0.03) */
/* Headers: brush-stroke underline effect */
/* Cards: soft shadow, rounded, no hard borders */
/* Font accent: Scheherazade for Arabic elements */
```

---

**Theme 6: Sahra (صحراء) — Desert Night**
Deep desert sky. Stars. Sand dunes silhouette. Contemplative.

```css
/* Desert under stars */
--bg:           #0b0e17;           /* desert night sky */
--surface:      #141b2d;           /* deep indigo */
--border:       #c4a35a;           /* sand gold */
--text:         #d4c5a9;           /* sandstone */
--accent:       #e6c34d;           /* desert gold */
--accent2:      #7c3aed;           /* twilight purple */
--green:        #65a30d;           /* oasis green */
/* Background: subtle star field (CSS radial gradients) */
/* Bottom of page: sand dune silhouette SVG */
/* Bismillah: glowing gold text with subtle text-shadow */
/* Cards: glass morphism with slight blur */
```

---

**Theme 7: Zahra (زهرة) — Geometric Garden**
Vibrant. Inspired by Islamic geometric art — interlocking patterns, jewel tones.

```css
/* Islamic geometric garden */
--bg:           #0f1923;           /* deep ocean */
--surface:      #162232;           /* midnight blue */
--border:       #e57c23;           /* amber */
--text:         #ecf0f1;           /* cloud white */
--accent:       #00a878;           /* emerald */
--accent2:      #e74c3c;           /* ruby */
--green:        #00e676;           /* jade */
/* Background: interlocking 8-point star tessellation (SVG, opacity 0.05) */
/* Sidebar: gradient from emerald to deep blue */
/* Cards: hexagonal clip-path on hover */
/* Borders: alternating jewel color accents */
```

---

**Theme Implementation:**

```
web/static/css/
├── style.css                 # Base layout, components
├── themes/
│   ├── andalus.css           # Al-Andalus
│   ├── masjid.css            # Masjid
│   ├── alhamra.css           # Al-Hamra
│   ├── nur.css               # Nūr (light)
│   ├── raqsh.css             # Raqsh (light)
│   ├── sahra.css             # Sahra (dark)
│   └── zahra.css             # Zahra (dark)
├── patterns/
│   ├── zellige.svg           # Andalusian tile pattern
│   ├── muqarnas.svg          # Alhambra honeycomb
│   ├── rosette.svg           # Illuminated rosette
│   ├── stars.svg             # 8-point star tessellation
│   └── dunes.svg             # Desert dune silhouette
└── fonts/
    └── (loaded from Google Fonts CDN)
        Amiri, Scheherazade New, Noto Naskh Arabic
```

**Theme switcher UI:**
```
+----------------------------------------------------------+
| THEME                                                     |
|                                                           |
| ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐         |
| │ ██████  │ │ ██████  │ │ ██████  │ │ ░░░░░░  │         |
| │ ██████  │ │ ██████  │ │ ██████  │ │ ░░░░░░  │         |
| │Andalus  │ │ Masjid  │ │Al-Hamra │ │  Nūr    │         |
| └─────────┘ └─────────┘ └─────────┘ └─────────┘         |
| ┌─────────┐ ┌─────────┐ ┌─────────┐                      |
| │ ░░░░░░  │ │ ██████  │ │ ██████  │                      |
| │ ░░░░░░  │ │ ██████  │ │ ██████  │                      |
| │ Raqsh   │ │ Sahra   │ │ Zahra   │                      |
| └─────────┘ └─────────┘ └─────────┘                      |
|                                                           |
| Each box shows a mini-preview of the color palette.       |
| Click to apply instantly. Saved in config.                |
+----------------------------------------------------------+
```

**Config:**
```json
{
  "language": "en",
  "web_theme": "masjid",
  "mode": "beginner"
}
```

---

**Two Modes — Beginner & Advanced:**

Switchable anytime from header toggle or settings. Saved in config.

**Header toggle:**
```
| GitPulse    [🐣 Beginner ↔ 🔥 Advanced]    ⚙ Settings |
```

**Beginner mode — clean, guided, safe:**

| Aspect | Behavior |
|--------|----------|
| Menu | Only essential items visible (8 options) |
| Labels | Plain language, no git jargon |
| Tooltips | Explains every button on hover |
| Confirmations | Double-confirm on destructive actions |
| Dangerous features | Hidden (force push, delete, bulk delete, danger zone) |
| Timestamp editor | Only "single commit" mode, calendar picker |
| Terminal jargon | Replaced: "repo" → "project", "clone" → "download", "push" → "upload" |
| Errors | Friendly messages with "what to do next" suggestions |
| Onboarding | First-run wizard: set token, choose theme, pick language |
| Web sidebar | Grouped with icons and labels, no abbreviations |

**Beginner CLI menu:**
```
  GitPulse (Beginner)

  MY PROJECTS
   1  View my projects
   2  Search projects
   3  Project details

  ACTIONS
   4  Create new project
   5  Edit project
   6  Download a project

  INFO
   7  My account overview
   8  Settings

  0  Exit
```

**Beginner web sidebar:**
```
+------------------+
| 📁 My Projects   |
| 🔍 Search        |
| ➕ Create New     |
| 📊 Overview      |
| 🎨 Themes        |
| ⚙  Settings      |
+------------------+
```

---

**Advanced mode — everything unlocked, power user:**

| Aspect | Behavior |
|--------|----------|
| Menu | All 30+ options visible, categorized |
| Labels | Technical git terminology |
| Tooltips | Off by default (toggle in settings) |
| Confirmations | Single confirm, can disable in settings |
| All features | Unlocked including danger zone, bulk delete, force push |
| Timestamp editor | All modes (single, shift, spread, pattern, rhythm) |
| Keyboard shortcuts | Enabled (Ctrl+K search, g+r repos, g+h health) |
| Web sidebar | Compact, icon-only option, all sections |
| CLI output | Verbose with hashes, API details |
| Extra panels | Raw JSON viewer, API call log, git command log |

**Advanced CLI menu:**
```
  GitPulse v0.5.0 (Advanced)

  REPOS          GIT            POWER TOOLS
   1  List        8  Commits    11 Backup
   2  Search      9  Branches   12 Bulk ops
   3  Details    10  Git log    13 Health
                               14 Search

  DEVOPS         INSIGHTS       GIT TOOLS
  15 Actions     19 Stats       23 Timestamps
  16 Releases    20 Analytics   24 SSH keys
  17 Deps        21 AI          25 Archaeology
  18 Templates   22 Rhythm      26 Diff story

  ORGANIZATION   SPECIAL        SYSTEM
  27 Groups      30 Accounts    32 Settings
  28 Graph       31 OTA         0  Exit
  29 Danger
```

**Advanced web sidebar:**
```
+------------------+
| 📁 Repos         |
| 🔍 Search        |
| ─────────────── |
| 💾 Backup        |
| ⚡ Bulk          |
| 🏥 Health        |
| 🔎 Code Search   |
| ─────────────── |
| ▶  Actions       |
| 📦 Releases      |
| 📋 Deps          |
| 📐 Templates     |
| ─────────────── |
| 📊 Stats         |
| 📈 Analytics     |
| 🤖 AI            |
| 🎵 Rhythm        |
| ─────────────── |
| 🕐 Timestamps    |
| 🔑 SSH           |
| 🦴 Archaeology   |
| 📖 Diff Story    |
| ─────────────── |
| 📂 Groups        |
| 🕸  Graph         |
| ⚠  Danger        |
| 👥 Accounts      |
| 🔌 OTA           |
| ─────────────── |
| 🕰  Time Machine  |
| 🧬 DNA           |
| 📡 Radar         |
| 🖥  Ambient       |
| 🎤 Voice         |
| ─────────────── |
| ⚙  Settings      |
+------------------+
```

**Mode-aware components:**

```javascript
// Every component checks mode
if (mode === "beginner") {
    // show simplified version
    // hide dangerous buttons
    // add tooltips
    // use friendly labels
} else {
    // show full version
    // all features unlocked
    // technical labels
    // keyboard shortcuts active
}
```

**Transition between modes:**
- Switching from beginner → advanced: instant, all features appear
- Switching from advanced → beginner: confirm dialog "Some features will be hidden"
- Settings made in advanced mode are preserved when switching to beginner
- Beginner mode never deletes data or changes config from advanced mode

**First-run detection:**
```
Welcome to GitPulse!

How do you want to start?

  🐣  Beginner — Simple interface, guided steps
      Best if you're new to git or GitHub

  🔥  Advanced — Full power, all features
      Best if you're comfortable with git

You can switch anytime in Settings.
```

**Styling approach:** Single CSS file, no frameworks. CSS variables for theming:
```css
:root {
    --bg:       #0d1117;
    --surface:  #161b22;
    --border:   #30363d;
    --text:     #c9d1d9;
    --accent:   #58a6ff;
    --green:    #3fb950;
    --red:      #f85149;
    --yellow:   #d29922;
}
```

**No heavy frameworks.** Just:
- Python built-in `http.server` + custom `BaseHTTPRequestHandler` (server)
- Custom `{{variable}}` template engine (no Jinja2)
- Vanilla JS + fetch API (interactivity)
- Chart.js CDN (analytics charts only)
- D3.js CDN (relationship graph, DNA fingerprint)
- Flatpickr CDN (calendar date picker for timestamps only)
- highlight.js CDN (code syntax in search results)

**Responsive:** Works on phone browser over LAN too (CSS grid, collapsible sidebar).

### API Pattern (Frontend ↔ Flask)

Pages load via normal GET routes. Interactive actions use fetch() to JSON endpoints:

```
GET  /repos                    → HTML page
GET  /api/repos                → JSON list
POST /api/repos                → create repo
PATCH /api/repos/<name>        → edit repo
DELETE /api/repos/<name>       → delete repo

POST /api/timestamps/preview   → preview date changes
POST /api/timestamps/apply     → apply date changes
POST /api/timestamps/push      → force push

GET  /api/health               → JSON health scores
POST /api/bulk/apply           → apply bulk action
POST /api/workflows/<id>/rerun → re-trigger workflow
```

**Pattern:** every page has a normal HTML route + an `/api/` JSON counterpart.
Pages work without JS (forms submit normally). JS enhances with async fetch and toasts.

### Security

- Binds to `127.0.0.1` only (not `0.0.0.0`) by default
- Optional `--lan` flag to bind `0.0.0.0` for LAN access
- Token never exposed in HTML/JS — stays server-side in config
- CSRF protection via hidden token in forms (custom middleware)
- No auth needed (it's your local machine)
- Optional basic password if using `--lan` mode
- QR code displayed in terminal for quick phone access in LAN mode

### Config Additions

```json
{
  "web_port": 5000,
  "web_host": "127.0.0.1",
  "web_theme": "dark",
  "web_auto_open": true
}
```

### LAN Mode

```bash
python github_manager.py --web --lan
```

```
  + GitHub Manager Web UI
  + Local:   http://localhost:5000
  + Network: http://192.168.1.42:5000
  
  ⚠ LAN mode: accessible to others on your network
  > Set a password? [y/N]: y
  > Password: ****
  + Password protection enabled
```

### Dependencies

```
# No extra dependencies for web UI!
# Uses Python built-in http.server
```

Zero install. Works everywhere Python works.

```bash
pip install --user requests    # only dependency for the whole project
```

---

## GENIUS FEATURES

---

### 14. GIT TIME MACHINE (web/timemachine.py)

**What it does:**
- Visual slider scrubs through entire repo history like a video player
- File tree updates in real-time as you drag
- "Snapshot" button — clone the repo at that exact point in time
- Play button — watch the project grow from first commit to now

**UI:**
```
+----------------------------------------------------------+
| GIT TIME MACHINE: rxy                                     |
|                                                           |
| [◀] [▶ Play]  ════════════●══════════  commit 47/203     |
|               2024-03                  2026-02             |
|                                                           |
| Commit: e4c2a01 — feat: BLE advertisement                |
| Author: abourdim — 2026-02-26 14:00                      |
|                                                           |
| FILE TREE                    FILE PREVIEW                 |
| ├── src/                     ┌─────────────────────────┐  |
| │   ├── main.cpp ●+         │ #include <BLE.h>        │  |
| │   ├── ble.cpp  ●NEW       │                         │  |
| │   ├── ble.h    ●NEW       │ void initBLE() {        │  |
| │   └── wifi.cpp             │   BLEDevice::init("");  │  |
| ├── lib/                     │   ...                   │  |
| ├── README.md ●+             └─────────────────────────┘  |
| └── platformio.ini                                        |
|                                                           |
| ● NEW = added this commit   ●+ = modified                |
|                                                           |
| [Snapshot this state]  [Open on GitHub]  [Download .zip]  |
+----------------------------------------------------------+
```

**Implementation:**
- `git log --all --format=...` for commit list
- `git ls-tree -r <hash>` for file tree at any commit
- `git show <hash>:<path>` for file content at any commit
- `git diff <hash1>..<hash2> --stat` for changed files
- Slider is an HTML range input mapped to commit index
- Debounced fetch on slider move

---

### 15. REPO DNA FINGERPRINT (web/dna.py)

**What it does:**
- Generates a unique visual identity per repo
- Based on: commit frequency, languages, file count, age, contributors, size
- Rendered as SVG — shareable, printable, can be used as project badge

**UI:**
```
+----------------------------------------------------------+
| REPO DNA                                                  |
|                                                           |
|   rxy                    bit-bot          salat-times     |
|                                                           |
|   ╭─────╮               ╭─────╮          ╭─────╮         |
|   │▓▓▒░░│               │░░▒▓▓│          │▒▓▒▓▒│         |
|   │▓▓▓▒░│               │░░░▒▓│          │▒▓▓▓▒│         |
|   │▓▒▒░░│               │░░▒▒▓│          │▒▒▓▒▒│         |
|   │▓▓▒▒░│               │░░▒▓▓│          │▒▓▓▓▒│         |
|   ╰─────╯               ╰─────╯          ╰─────╯         |
|   C++ | 203 commits      Python | 45      JS | 128       |
|   Active | 1.2MB          Stale | 80KB    Active | 340KB  |
|                                                           |
| [Download SVG]  [Copy badge URL]  [Generate all]          |
+----------------------------------------------------------+
```

**Encoding scheme:**
- Rows = time periods (quarters)
- Columns = dimensions (commits, files, languages, issues, stars)
- Color intensity = activity level
- Shape variations = language mix
- Each repo gets a deterministic, unique pattern

---

### 16. COMMIT RHYTHM ANALYZER (lib/rhythm.py)

**What it does:**
- Analyzes your real commit patterns across all repos
- Detects: preferred hours, burst vs steady, weekday distribution
- Generates a "coding profile"
- Timestamp editor can use YOUR rhythm to make fake timestamps look natural

**UI:**
```
+----------------------------------------------------------+
| YOUR CODING RHYTHM                                        |
|                                                           |
| HOURLY DISTRIBUTION                                       |
|         ·                                                 |
|        ···                                    ··          |
|       ·····                                 ·····         |
|      ·······                               ·······        |
|  ____·········_____________________________·········___   |
|  00  03  06  09  12  15  18  21  24                       |
|                                                           |
| Peak hours: 09:00-12:00 and 20:00-23:00                  |
| Style: Bimodal (morning + evening coder)                  |
|                                                           |
| WEEKLY PATTERN                                            |
| Mon ████████████ 18%                                      |
| Tue ██████████████ 21%                                    |
| Wed ████████████ 17%                                      |
| Thu ██████████████ 20%                                    |
| Fri ████████ 13%                                          |
| Sat ████ 7%                                               |
| Sun ███ 4%                                                |
|                                                           |
| BURST ANALYSIS                                            |
| Avg commits per session: 4.2                              |
| Avg session duration: 2h 15m                              |
| Avg gap between commits: 32 min                           |
|                                                           |
| [Use this rhythm in timestamp editor]                     |
+----------------------------------------------------------+
```

**Integration with timestamp editor:**
- "Spread using my rhythm" button
- Generates timestamps that match your natural patterns
- Adds realistic jitter based on your avg gap between commits

---

### 17. LIVE COLLABORATION RADAR (web/radar.py)

**What it does:**
- Polls GitHub Events API for real-time activity
- Desktop notification (browser Notification API) on push, PR, issue, star
- Pulse animation on dashboard
- Sound option (subtle ping)

**UI:**
```
+----------------------------------------------------------+
| LIVE RADAR                          ● Connected           |
|                                                           |
| ┌─────────────────────────────────────────────────────┐   |
| │  ★  someone starred rxy                    just now │   |
| │  ↑  you pushed to rxy (3a7f2c1)           2m ago   │   |
| │  !  new issue on salat-times #12           15m ago  │   |
| │  ↑  Azedwise pushed to fork/rxy            1h ago   │   |
| │  ⑂  PR #5 merged on bit-bot                3h ago   │   |
| └─────────────────────────────────────────────────────┘   |
|                                                           |
| PULSE                                                     |
| rxy          ●●●○○  3 events today                       |
| salat-times  ●○○○○  1 event today                        |
| bit-bot      ●●○○○  2 events today                       |
|                                                           |
| [🔔 Notifications: ON]  [🔊 Sound: OFF]  [Poll: 30s]     |
+----------------------------------------------------------+
```

**Implementation:**
- `GET /users/{username}/received_events` polled every 30s
- Server-Sent Events (SSE) from Python to browser (no WebSocket needed)
- Browser Notification API for desktop alerts
- Configurable poll interval in settings

---

### 18. SMART REPO GROUPER (lib/grouper.py + web)

**What it does:**
- Auto-categorizes repos by analyzing language, README content, topics, filenames
- Detects project types: IoT, web, mobile, library, docs, learning, tool
- Custom groups with drag-and-drop in web UI
- Saved in config, used across CLI and web

**Detection rules:**
```
platformio.ini OR .ino files     → IoT / Embedded
index.html + style.css           → Web
package.json + node_modules      → Node.js
setup.py OR pyproject.toml       → Python Library
Dockerfile                       → Containerized
.github/workflows/               → Has CI/CD
README.md only, < 5 files        → Documentation
fork = true                      → Forked
commits < 5, age > 6 months      → Abandoned experiment
```

**UI:**
```
+----------------------------------------------------------+
| REPO GROUPS                                               |
|                                                           |
| 🔌 IoT / Embedded (5)                                    |
|    rxy, bit-bot, esp32-c3-kids-lab, talking-robot,        |
|    ble-logger                                             |
|                                                           |
| 🌐 Web Projects (4)                                      |
|    face-quest, face-tracking, magic-hands, bitPlayground  |
|                                                           |
| 🛠 Tools & Libraries (3)                                  |
|    termlite, loggy, cowsay                                |
|                                                           |
| 🕌 Islamic Apps (2)                                       |
|    salat-times, tesbih                                    |
|                                                           |
| 📦 Ungrouped (3)                                          |
|    prompt-example, workshop-diy, apps                     |
|                                                           |
| [Auto-detect groups]  [Edit groups]  [Drag to reorder]   |
+----------------------------------------------------------+
```

**Config:**
```json
{
  "repo_groups": {
    "IoT / Embedded": ["rxy", "bit-bot", "esp32-c3-kids-lab"],
    "Web Projects": ["face-quest", "face-tracking"],
    "Islamic Apps": ["salat-times", "tesbih"]
  }
}
```

---

### 19. GIT ARCHAEOLOGY MODE (lib/archaeology.py)

**What it does:**
- Scans repos for forgotten/dead/risky stuff
- Finds: dead branches, TODO comments, large files, unreferenced code

**Scans:**

| Scan | What | How |
|------|------|-----|
| Dead branches | Not merged, older than 6 months | `git branch -r --no-merged` + commit date |
| TODO/FIXME/HACK | Comments with age | `git grep` + `git log -S` for when added |
| Large files | Files > 1MB in history | `git rev-list --objects --all` + sort by size |
| Forgotten forks | Forked repos with zero commits ahead | Compare with parent |
| Empty repos | No commits or only initial commit | Commit count check |
| Stale PRs | Open PRs older than 30 days | API: pulls?state=open |

**UI:**
```
+----------------------------------------------------------+
| GIT ARCHAEOLOGY                                           |
|                                                           |
| DEAD BRANCHES (7 across 4 repos)                          |
|   rxy:        feature/old-sensor (8 months, not merged)   |
|   rxy:        test/experimental (1 year, not merged)      |
|   bit-bot:    dev-v2 (6 months, 3 commits ahead)          |
|                                                           |
| TODOs & FIXMEs (12 across 6 repos)                        |
|   rxy/src/main.cpp:42       // TODO: add timeout          |
|     Added: 2025-03-15 (11 months ago!)                    |
|   bit-bot/app.py:89         # FIXME: memory leak          |
|     Added: 2025-01-20 (13 months ago!)                    |
|                                                           |
| LARGE FILES (3 files > 1MB)                               |
|   rxy/docs/schematic.pdf          4.2 MB                  |
|   bit-bot/assets/demo.mp4        12.8 MB                  |
|                                                           |
| [Delete dead branches]  [Export TODO list]                 |
+----------------------------------------------------------+
```

---

### 20. VOICE CONTROL (web — browser Speech API)

**What it does:**
- Browser-native speech recognition (no extra deps)
- Natural language commands mapped to actions
- Works hands-free while soldering, 3D printing, etc.

**Commands:**
```
"list repos"              → navigate to repos page
"create repo smart garden private"  → opens create form pre-filled
"show health"             → navigate to health dashboard
"search WiFi begin"       → cross-repo search
"show stale repos"        → health filtered to score < 50%
"backup all"              → trigger backup
"show commits for rxy"    → commit list for rxy
"what did I do this week" → AI weekly summary
```

**UI:**
```
+----------------------------------------------------------+
| 🎤 Voice Control                                          |
|                                                           |
|  [🎤 Listening...]                                        |
|                                                           |
|  You said: "show stale repos"                             |
|  Action:   Navigating to Health → filtered < 50%          |
|                                                           |
|  Recent commands:                                         |
|   "list repos"              ✓                             |
|   "search ble handler"      ✓                             |
|   "create repo test"        ✓                             |
|                                                           |
|  [🎤 ON]  [History]  [Help — show all commands]           |
+----------------------------------------------------------+
```

**Implementation:**
- `window.SpeechRecognition` (Chrome, Edge — built-in)
- Command parser: regex patterns mapped to URL navigations or API calls
- Fallback: text command bar for unsupported browsers
- No server-side processing — all in browser JS

---

### 21. REPO RELATIONSHIP GRAPH (web/graph.py)

**What it does:**
- Visual network diagram showing connections between your repos
- Links based on: shared dependencies, shared code files, forked from, similar languages
- Detect duplicate files across repos
- Suggest merging related repos

**UI:**
```
+----------------------------------------------------------+
| REPO NETWORK                                              |
|                                                           |
|              [rxy]──────[bit-bot]                         |
|              / |  shared: wifi.h  \                       |
|             /  |                   \                      |
|    [esp32-c3] [ble-logger]     [talking-robot]            |
|        \       |                                          |
|         \      | shared: ble_utils.cpp                    |
|          \     |                                          |
|       [face-tracking]───[face-quest]───[magic-hands]      |
|                    shared: face-api.js                    |
|                                                           |
|                                                           |
|   [salat-times]    [tesbih]    [tethkir]                  |
|        └──────── similar: Islamic apps ────────┘          |
|                                                           |
| Isolated: cowsay, loggy, prompt-example                   |
|                                                           |
| [Force-directed layout]  [Group by language]  [Zoom]      |
+----------------------------------------------------------+
```

**Implementation:**
- D3.js force-directed graph (loaded from CDN)
- Compare file lists across repos via API
- Hash file contents to find duplicates
- Language overlap scoring

---

### 22. DANGER ZONE DASHBOARD (lib/danger.py + web)

**What it does:**
- One-page security and risk overview across ALL repos
- Scans for common mistakes and risks
- One-click fixes where possible

**Scans:**

| Risk | Detection | Fix |
|------|-----------|-----|
| Leaked secrets | Pattern scan: API keys, tokens, passwords in code | Add to .gitignore + purge from history |
| No .gitignore | Missing file | Auto-generate based on language |
| Force pushes | Check reflog via events API | Alert only |
| No backup | Not in backup list | Trigger backup |
| Large binaries | Files > 5MB in git history | Suggest Git LFS |
| Public with secrets | Public repo + detected secrets | Make private immediately |
| Stale tokens | GitHub tokens in code older than 90 days | Rotate reminder |
| Missing branch protection | Default branch unprotected | Enable via API |

**Secret patterns detected:**
```
ghp_[A-Za-z0-9]{36}           GitHub token
AKIA[A-Z0-9]{16}              AWS access key
sk-[A-Za-z0-9]{48}            OpenAI key
-----BEGIN RSA PRIVATE KEY---- Private key
password\s*=\s*['"][^'"]+     Hardcoded password
```

**UI:**
```
+----------------------------------------------------------+
| DANGER ZONE                                               |
|                                                           |
| 🔴 CRITICAL (2)                                           |
|   rxy: possible API key in src/config.h line 12           |
|     [View] [Make private] [Purge from history]            |
|   bit-bot: GitHub token in .env (committed!)              |
|     [View] [Purge from history]                           |
|                                                           |
| 🟡 WARNING (4)                                            |
|   3 repos missing .gitignore    [Fix all]                 |
|   1 repo with 15MB binary       [Suggest LFS]            |
|                                                           |
| 🟢 GOOD (11)                                              |
|   No issues detected                                      |
|                                                           |
| Last scan: 2 minutes ago  [Rescan all]                    |
+----------------------------------------------------------+
```

---

### 23. AMBIENT DASHBOARD (web/ambient.py)

**What it does:**
- Full-screen, always-on display mode
- For second monitor, old laptop, or wall-mounted tablet
- Beautiful minimal design — like an airport departure board
- Auto-refreshes, auto-dims at night

**UI:**
```
+----------------------------------------------------------+
|                                                           |
|              G I T H U B   M A N A G E R                  |
|              abourdim — 17 repositories                   |
|                                                    22:41  |
|                                                           |
|  LATEST                                                   |
|  ───────────────────────────────────────────────────────  |
|  rxy           fix: sensor calibration          2h ago    |
|  salat-times   prayer time calculation fix      1d ago    |
|  tethkir       new dhikr entries                3d ago    |
|                                                           |
|  BUILDS                                                   |
|  ───────────────────────────────────────────────────────  |
|  rxy           CI Build       ✓ pass     2m 31s          |
|  bit-bot       Deploy         ✗ fail     0m 45s          |
|                                                           |
|  STATS                                                    |
|  ───────────────────────────────────────────────────────  |
|  ★ 142 stars    👁 342 views this week    11 commits      |
|                                                           |
+----------------------------------------------------------+
```

**Features:**
- CSS `prefers-color-scheme` + time-based auto-dim (after 22:00)
- `<meta http-equiv="refresh" content="60">` auto-reload
- Full-screen API (`document.requestFullscreen()`)
- Large monospace font, high contrast
- Configurable sections (show/hide builds, stats, activity)
- Clock in corner
- Optional: cycle between pages every 30s

---

### 24. ESP32 FIRMWARE OTA PAGE (web/ota.py)

**What it does:**
- Designed for your ESP32 projects specifically
- Upload `.bin` firmware to GitHub Releases from web UI
- Generates a stable download URL for OTA
- ESP32 can poll that URL to check for updates
- Basically turns GitHub into a free firmware update server

**UI:**
```
+----------------------------------------------------------+
| FIRMWARE OTA MANAGER                                      |
|                                                           |
| Repo: [rxy ▼]                                             |
|                                                           |
| CURRENT FIRMWARE                                          |
| Version:  v1.3.0                                          |
| File:     firmware.bin (245 KB)                           |
| Uploaded: 2026-02-28 14:00                                |
| SHA256:   a3f2c1...                                       |
|                                                           |
| OTA ENDPOINT (give this to your ESP32):                   |
| ┌─────────────────────────────────────────────────────┐   |
| │ https://github.com/abourdim/rxy/releases/           │   |
| │ download/latest/firmware.bin                         │   |
| └─────────────────────────────────────────────────────┘   |
| [Copy URL]                                                |
|                                                           |
| ESP32 CODE SNIPPET:                                       |
| ┌─────────────────────────────────────────────────────┐   |
| │ #include <HTTPUpdate.h>                              │   |
| │ WiFiClient client;                                   │   |
| │ t_httpUpdate_return ret =                            │   |
| │   httpUpdate.update(client, OTA_URL);                │   |
| └─────────────────────────────────────────────────────┘   |
| [Copy snippet]                                            |
|                                                           |
| UPLOAD NEW FIRMWARE                                       |
| [Choose .bin file]  Tag: [v___]  [Upload & Release]       |
|                                                           |
| HISTORY                                                   |
| v1.3.0  firmware.bin  245KB  2026-02-28  [Download]       |
| v1.2.0  firmware.bin  240KB  2026-02-15  [Download]       |
| v1.1.0  firmware.bin  232KB  2026-01-30  [Download]       |
+----------------------------------------------------------+
```

**Implementation:**
- Upload via `POST /repos/{owner}/{repo}/releases` + upload asset
- "Latest" URL uses GitHub's `/releases/latest/download/{asset}` redirect
- Version manifest JSON file in releases for ESP32 to check
- Auto-generates ESP32 Arduino/PlatformIO OTA code snippet

---

### 25. PORTABLE MODE (ghm.sh addition)

**What it does:**
- Entire app packaged to run from USB stick on any Windows PC
- No installation, no admin, no PATH changes needed
- Includes embedded Python (WinPython or portable Python)
- Config stored on the USB stick itself

**Structure on USB:**
```
USB_DRIVE/
├── ghm/
│   ├── python/                # Embedded Python (WinPython)
│   │   ├── python.exe
│   │   └── Lib/
│   ├── app/
│   │   ├── github_manager.py
│   │   ├── lib/
│   │   └── web/
│   ├── config/                # Portable config (not ~/.github-manager)
│   │   └── config.json
│   ├── run.bat                # Windows: double-click to launch
│   ├── run.sh                 # MSYS2/Linux: bash run.sh
│   └── README.txt
```

**run.bat:**
```batch
@echo off
cd /d %~dp0
set GHM_PORTABLE=1
set GHM_CONFIG=%~dp0config
python\python.exe app\github_manager.py --web
pause
```

**run.sh:**
```bash
#!/bin/bash
DIR="$(cd "$(dirname "$0")" && pwd)"
export GHM_PORTABLE=1
export GHM_CONFIG="$DIR/config"
"$DIR/python/python" "$DIR/app/github_manager.py" --web 2>/dev/null \
    || python3 "$DIR/app/github_manager.py" --web
```

**Config detection in config.py:**
```python
if os.environ.get("GHM_PORTABLE"):
    CONFIG_DIR = Path(os.environ["GHM_CONFIG"])
else:
    CONFIG_DIR = Path.home() / ".github-manager"
```

**ghm.sh addition:**
```
MENU
  ...
  8   Create portable USB package    ← NEW
```

Downloads WinPython portable, bundles everything, creates run.bat/run.sh.

---

### 26. MAGIC DIFF STORYTELLER (lib/storyteller.py)

**What it does:**
- Translates raw git diffs into human-readable summaries
- No AI needed — pure pattern matching on diff structure
- Works in both CLI and web

**Pattern rules:**
```
+function           → "Added new function: {name}"
-function           → "Removed function: {name}"
renamed file        → "Renamed {old} to {new}"
+import             → "Added dependency: {package}"
changed number      → "Changed {var} from {old} to {new}"
added file          → "Created new file: {name}"
+TODO/FIXME         → "Added TODO: {text}"
-TODO/FIXME         → "Completed TODO: {text}"
moved code block    → "Moved {function} from {file1} to {file2}"
changed string      → "Updated text: '{old}' → '{new}'"
```

**Display:**
```
  DIFF STORY: e4c2a01 → 3a7f2c1

  What changed:
  • Added new file: src/ble.cpp (BLE handler)
  • Added new file: src/ble.h (BLE header)
  • In src/main.cpp:
    - Added dependency: BLE.h
    - Added function: initBLE()
    - Changed SENSOR_OFFSET from 0.5 to 0.73
    - Completed TODO: "add BLE support"
  • Updated README.md:
    - Added "BLE" to features list

  Summary: Added BLE support with new handler,
  calibrated sensor offset, updated docs.
```

---

### 27. MULTI-ACCOUNT SUPPORT (lib/accounts.py)

**What it does:**
- Switch between personal / work / client GitHub accounts
- Each account has its own token, default settings
- Unified view: see all repos from all accounts
- Or filtered view: one account at a time
- CLI: `ghm --account work` to switch
- Web: dropdown in header to switch

**Config:**
```json
{
  "accounts": {
    "personal": {
      "token": "ghp_...",
      "username": "abourdim",
      "default_sort": "updated"
    },
    "work": {
      "token": "ghp_...",
      "username": "karim-company",
      "default_sort": "created"
    }
  },
  "active_account": "personal"
}
```

**CLI menu addition:**
```
ACCOUNTS
  > Switch account: personal ✓ | work | [+ Add new]
```

**Web header:**
```
+----------------------------------------------------------+
| GitHub Manager    [personal ▼]    🔔 Radar    ⚙ Settings |
|                   ├─ personal (abourdim)                  |
|                   ├─ work (karim-company)                 |
|                   ├─ All accounts (unified)               |
|                   └─ + Add account                        |
+----------------------------------------------------------+
```

---

## Updated Implementation Priority

| Phase | Features                                         | Complexity |
|-------|--------------------------------------------------|------------|
| 1     | Backup, Health, Bulk ops                         | Medium     |
| 2     | Git log, Cross-repo search                       | Medium     |
| 3     | Actions monitor, Release manager                 | Medium     |
| 4     | Templates, SSH keys, Timestamp editor            | Low-Medium |
| 5     | Analytics, Dependency scanner                    | Medium     |
| 6     | AI summaries                                     | Medium     |
| 7     | Web UI core (http.server, dashboard, repos)      | Medium     |
| 8     | Web UI remaining pages                           | Medium     |
| 9     | Danger zone, Archaeology, Repo grouper           | Medium     |
| 10    | Rhythm analyzer, Diff storyteller, Multi-account | Medium     |
| 11    | Time machine, DNA fingerprint, Relationship graph| High       |
| 12    | Live radar, Voice control, Ambient dashboard     | Medium-High|
| 13    | ESP32 OTA, Portable mode                         | Medium     |

---

## Dependencies

**Required:**
- requests

**Optional:**
- openai (AI summaries)

**Zero-install (built-in Python):**
- http.server (web UI)
- json, pathlib, subprocess, hashlib, re, os, sys

**CDN (loaded in browser, no install):**
- Chart.js (analytics charts)
- D3.js (relationship graph, DNA fingerprint)
- Flatpickr (date pickers)
- highlight.js (code syntax highlighting)
- xterm.js (optional terminal emulator)

---

## Notes

- All features work without extra deps (requests only)
- Web UI uses Python built-in http.server — zero extra install
- AI features gracefully degrade: no API key = pattern-based summary
- All modules follow same pattern: function per action, takes GitHubAPI instance
- Menu restructured into categories for clarity
- ghm.sh launcher unchanged (installs same way)
- Web mode reuses 100% of lib/ — routes are thin wrappers
- CLI and Web share same config file and token
- Portable mode enables running from USB on locked-down PCs
- Multi-account allows personal + work GitHub side by side
- ESP32 OTA turns GitHub into firmware update server
