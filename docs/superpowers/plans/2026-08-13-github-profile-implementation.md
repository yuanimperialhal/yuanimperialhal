# GitHub Profile Reproduction Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish the approved Chinese introduction and an automatically generated contribution snake on the `yuanimperialhal` GitHub profile.

**Architecture:** The root `README.md` is the only profile presentation file. A single GitHub Actions workflow generates light and dark SVG assets from the repository owner's contribution graph and publishes them to an `output` branch, which the README loads through raw GitHub URLs.

**Tech Stack:** GitHub-flavored Markdown, HTML `<picture>`, GitHub Actions YAML, `Platane/snk`, `crazy-max/ghaction-github-pages`, Git, PowerShell, Python 3.11 with PyYAML 6.0.3

## Global Constraints

- Target repository: `https://github.com/yuanimperialhal/yuanimperialhal.git`.
- Default branch: `main`.
- Generated asset branch: `output`.
- The introduction text must match the user-provided Chinese copy exactly.
- Keep only the centered introduction and contribution snake; do not add badges, counters, GitHub cards, social links, or other profile modules.
- Generate both `github-snake.svg` and `github-snake-dark.svg`.
- Do not force-push. Stop if the remote gains unexpected commits before publication.

---

## File Map

- Modify `README.md`: render the approved introduction and theme-aware snake animation on the profile.
- Modify `.github/workflows/snake.yml`: generate the two SVG files and publish them to `output`.
- No application source files, dependencies, or generated SVG files are stored on `main`.

### Task 1: Profile README

**Files:**

- Modify: `README.md`
- Test: inline PowerShell content assertions

**Interfaces:**

- Consumes: raw SVG URLs under `yuanimperialhal/yuanimperialhal@output`.
- Produces: the root profile README automatically rendered by GitHub.

- [ ] **Step 1: Run the README contract check and verify the old template fails**

Run from `G:\codextest\turbo-main`:

```powershell
$ErrorActionPreference = 'Stop'
$readme = Get-Content -Raw -Encoding UTF8 -LiteralPath 'README.md'
if ($readme -notmatch [regex]::Escape('# 你好，我是 Turbo Yuan 👋')) { throw 'Missing approved title.' }
if ($readme -match 'seiry') { throw 'README still references the template account.' }
```

Expected: the command stops with `Missing approved title.`

- [ ] **Step 2: Replace `README.md` with the approved profile content**

Use this complete content:

```markdown
<div align="center">

# 你好，我是 Turbo Yuan 👋

### 机器人软件开发工程师｜具身智能应用方向

**Python · C++ · ROS 2 · Agent · Isaac Sim · Docker · Jetson**

主要关注机器人软件开发、大模型 Agent 应用、机器人仿真与边缘设备部署。

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/yuanimperialhal/yuanimperialhal/output/github-snake-dark.svg" />
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/yuanimperialhal/yuanimperialhal/output/github-snake.svg" />
  <img alt="GitHub contribution snake animation" src="https://raw.githubusercontent.com/yuanimperialhal/yuanimperialhal/output/github-snake.svg" />
</picture>

</div>
```

- [ ] **Step 3: Run the complete README contract check**

```powershell
$ErrorActionPreference = 'Stop'
$readme = Get-Content -Raw -Encoding UTF8 -LiteralPath 'README.md'
$required = @(
  '# 你好，我是 Turbo Yuan 👋',
  '### 机器人软件开发工程师｜具身智能应用方向',
  '**Python · C++ · ROS 2 · Agent · Isaac Sim · Docker · Jetson**',
  '主要关注机器人软件开发、大模型 Agent 应用、机器人仿真与边缘设备部署。',
  'https://raw.githubusercontent.com/yuanimperialhal/yuanimperialhal/output/github-snake.svg',
  'https://raw.githubusercontent.com/yuanimperialhal/yuanimperialhal/output/github-snake-dark.svg'
)
foreach ($text in $required) {
  if (-not $readme.Contains($text)) { throw "Missing README content: $text" }
}
if ($readme -match 'seiry') { throw 'README still references the template account.' }
if (([regex]::Matches($readme, '<div align="center">')).Count -ne 1) { throw 'Centered wrapper count is not one.' }
if (([regex]::Matches($readme, '</div>')).Count -ne 1) { throw 'Closing wrapper count is not one.' }
'README contract passed.'
```

Expected: `README contract passed.`

- [ ] **Step 4: Commit the README change**

```powershell
git add -- README.md
git commit -m "feat: add Chinese GitHub profile introduction"
```

Expected: one commit containing only `README.md`.

### Task 2: Contribution Snake Workflow

**Files:**

- Modify: `.github/workflows/snake.yml`
- Test: PyYAML syntax and semantic assertions

**Interfaces:**

- Consumes: `${{ github.repository_owner }}` and `${{ secrets.GITHUB_TOKEN }}` supplied by GitHub Actions.
- Produces: `github-snake.svg` and `github-snake-dark.svg` on the `output` branch.

- [ ] **Step 1: Run the workflow contract check and verify the old workflow fails**

```powershell
$ErrorActionPreference = 'Stop'
$workflow = Get-Content -Raw -Encoding UTF8 -LiteralPath '.github/workflows/snake.yml'
if ($workflow -notmatch 'uses:\s+Platane/snk/svg-only@v3') { throw 'Workflow is not using the SVG-only action.' }
```

Expected: the command stops with `Workflow is not using the SVG-only action.`

- [ ] **Step 2: Replace `.github/workflows/snake.yml` with the approved workflow**

Use this complete content:

```yaml
name: Generate contribution snake

on:
  schedule:
    - cron: "0 2 * * *"
  workflow_dispatch:
  push:
    branches:
      - main

permissions:
  contents: write

jobs:
  generate:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - name: Generate SVG files
        uses: Platane/snk/svg-only@v3
        with:
          github_user_name: ${{ github.repository_owner }}
          outputs: |
            dist/github-snake.svg
            dist/github-snake-dark.svg?palette=github-dark

      - name: Publish SVG files to output branch
        uses: crazy-max/ghaction-github-pages@v4
        with:
          target_branch: output
          build_dir: dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

- [ ] **Step 3: Parse the YAML and verify its behavior contract**

```powershell
$ErrorActionPreference = 'Stop'
python -c "import pathlib,yaml; p=pathlib.Path('.github/workflows/snake.yml'); d=yaml.load(p.read_text(encoding='utf-8'), Loader=yaml.BaseLoader); assert d['on']['push']['branches']==['main']; assert d['permissions']['contents']=='write'; steps=d['jobs']['generate']['steps']; assert steps[0]['uses']=='Platane/snk/svg-only@v3'; assert steps[0]['with']['github_user_name']=='${{ github.repository_owner }}'; assert steps[0]['with']['outputs'].splitlines()==['dist/github-snake.svg','dist/github-snake-dark.svg?palette=github-dark']; assert steps[1]['uses']=='crazy-max/ghaction-github-pages@v4'; assert steps[1]['with']=={'target_branch':'output','build_dir':'dist'}; print('Workflow contract passed.')"
```

Expected: `Workflow contract passed.`

- [ ] **Step 4: Confirm the main branch does not contain generated assets**

```powershell
$generated = Get-ChildItem -Path . -Recurse -File | Where-Object { $_.Name -in @('github-snake.svg', 'github-snake-dark.svg') }
if ($generated) { throw "Generated assets must not be committed on main: $($generated.FullName -join ', ')" }
'No generated snake assets on main.'
```

Expected: `No generated snake assets on main.`

- [ ] **Step 5: Commit the workflow change**

```powershell
git add -- .github/workflows/snake.yml
git commit -m "ci: generate contribution snake animation"
```

Expected: one commit containing only `.github/workflows/snake.yml`.

### Task 3: Pre-Publication Verification and Push

**Files:**

- Verify: `README.md`
- Verify: `.github/workflows/snake.yml`
- Verify: Git history and `origin`

**Interfaces:**

- Consumes: local `main` commits and remote `origin`.
- Produces: published `main` branch at `yuanimperialhal/yuanimperialhal`.

- [ ] **Step 1: Verify the complete local tree**

```powershell
$ErrorActionPreference = 'Stop'
git status --short --branch
git log --oneline --decorate -5
git ls-files
```

Expected: branch is `main`; the worktree is clean; tracked files are `README.md`, `.github/workflows/snake.yml`, the design spec, and this implementation plan.

- [ ] **Step 2: Recheck that the target remote has no branches**

```powershell
$ErrorActionPreference = 'Stop'
$refs = git ls-remote --heads origin
if ($LASTEXITCODE -ne 0) { throw 'Could not read origin.' }
if ($refs) { throw "Origin gained a branch before publication: $refs" }
'Origin is still empty.'
```

Expected: `Origin is still empty.`

- [ ] **Step 3: Push `main` without force**

```powershell
git push -u origin main
```

Expected: a new `main` branch is created and local `main` tracks `origin/main`. If Git Credential Manager opens a browser, sign in as `yuanimperialhal` and authorize the push. If authentication fails, stop without changing the remote and report the exact error.

- [ ] **Step 4: Verify the remote main commit equals local HEAD**

```powershell
$ErrorActionPreference = 'Stop'
$local = git rev-parse HEAD
$remoteLine = git ls-remote --heads origin main
if (-not $remoteLine) { throw 'Remote main was not created.' }
$remote = ($remoteLine -split '\s+')[0]
if ($local -ne $remote) { throw "Remote main $remote does not match local HEAD $local." }
"Remote main matches local HEAD: $local"
```

Expected: `Remote main matches local HEAD:` followed by the commit SHA.

### Task 4: Live GitHub Verification

**Files:**

- Verify remotely: root `README.md`
- Verify remotely: `output/github-snake.svg`
- Verify remotely: `output/github-snake-dark.svg`

**Interfaces:**

- Consumes: GitHub Actions run created by the `main` push.
- Produces: a confirmed live profile and two accessible SVG assets.

- [ ] **Step 1: Poll the public Actions API until the pushed workflow completes**

```powershell
$ErrorActionPreference = 'Stop'
$deadline = (Get-Date).AddMinutes(6)
$run = $null
while ((Get-Date) -lt $deadline) {
  $response = Invoke-RestMethod -Headers @{ 'User-Agent' = 'Codex-profile-verifier' } -Uri 'https://api.github.com/repos/yuanimperialhal/yuanimperialhal/actions/runs?branch=main&per_page=10'
  $run = $response.workflow_runs | Where-Object { $_.name -eq 'Generate contribution snake' } | Select-Object -First 1
  if ($run -and $run.status -eq 'completed') { break }
  Start-Sleep -Seconds 10
}
if (-not $run) { throw 'No contribution snake workflow run appeared within six minutes.' }
if ($run.status -ne 'completed') { throw "Workflow did not finish within six minutes: $($run.html_url)" }
if ($run.conclusion -ne 'success') { throw "Workflow conclusion was $($run.conclusion): $($run.html_url)" }
"Workflow succeeded: $($run.html_url)"
```

Expected: `Workflow succeeded:` followed by the public run URL.

- [ ] **Step 2: Verify the `output` branch and both SVG responses**

```powershell
$ErrorActionPreference = 'Stop'
$outputRef = git ls-remote --heads origin output
if (-not $outputRef) { throw 'The output branch does not exist.' }
$urls = @(
  'https://raw.githubusercontent.com/yuanimperialhal/yuanimperialhal/output/github-snake.svg',
  'https://raw.githubusercontent.com/yuanimperialhal/yuanimperialhal/output/github-snake-dark.svg'
)
foreach ($url in $urls) {
  $response = Invoke-WebRequest -UseBasicParsing -Uri $url
  if ($response.StatusCode -ne 200) { throw "$url returned HTTP $($response.StatusCode)." }
  if ($response.Content -notmatch '<svg') { throw "$url did not return SVG content." }
  "SVG available: $url"
}
```

Expected: the `output` branch exists and both URLs print `SVG available:`.

- [ ] **Step 3: Verify the public profile repository and profile page**

Open and inspect:

```text
https://github.com/yuanimperialhal/yuanimperialhal
https://github.com/yuanimperialhal
```

Expected: the repository root shows the approved Chinese introduction, and the profile page displays the same README with a visible contribution snake in the current GitHub theme.

- [ ] **Step 4: Record the final evidence**

Report the final commit SHA, workflow run URL, repository URL, profile URL, and whether both SVG URLs returned HTTP 200. Do not claim completion if any of these checks failed.
