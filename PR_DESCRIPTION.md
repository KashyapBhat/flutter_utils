# PR: Batch operations and Git Pull (all in folder)

## Summary

This PR adds batch (multi-package) operations for Pub Get, Clean & Pub Get, and Git Pull, plus improved summary output and error reporting. All new commands use a parent-folder path (defaulting to workspace root) and run the operation in each direct child that matches the criteria.

---

## How to raise this PR (from your fork to upstream main)

1. **Create a branch and commit your changes**
   ```bash
   cd /path/to/flutter_utils
   git checkout -b feature/batch-pub-get-and-git-pull
   git add package.json package-lock.json src/extension.ts src/buildTreeView.ts src/utilityRunner.ts DEVELOPMENT.md
   git commit -m "feat: add Pub Get / Clean & Pub Get / Git Pull (all in folder) with summaries and error capture"
   ```

2. **Push the branch to your fork**
   ```bash
   git push -u origin feature/batch-pub-get-and-git-pull
   ```

3. **Open the PR on GitHub**
   - Go to **your fork**: `https://github.com/KashyapBhat/flutter_utils`
   - You should see a prompt to **Compare & pull request** for the new branch.
   - If the PR is **into the original upstream repo** (e.g. `skalex/flutter_utils` or whoever you forked from): change the base repository to that upstream and base branch to `main`, then create the PR.
   - Paste or adapt the content below into the PR description.

---

## List of changes

### New commands

| Command | ID | Description |
|--------|-----|-------------|
| Flutter: Pub Get (all in folder) | `flutter-build-utils.pubGetAllInFolder` | Runs `flutter pub get` in each direct child folder that contains `pubspec.yaml`. |
| Flutter: Clean & Pub Get (all in folder) | `flutter-build-utils.cleanAndPubGetAllInFolder` | Runs clean + pub get in each direct child with `pubspec.yaml`; optional delete of `pubspec.lock` per package (one prompt for all). |
| Git: Pull (all in folder) | `flutter-build-utils.gitPullAllInFolder` | Runs `git pull origin <current_branch>` in each direct child that is a git repo (has `.git`). |

### Behavior and UX

- **Default path**: For all three “all in folder” commands, the folder path input defaults to the first workspace folder, or the active file’s directory if no workspace folder.
- **Summaries**: After each batch run, a summary is appended to the **flutter-toolbox** Output channel and the panel is shown. The notification says “See Output for details.”
- **Git Pull (all in folder)**:
  - Resolves the **current branch** per repo and runs `git pull origin <branch>` for that repo.
  - Repos where the current branch cannot be resolved are **skipped** and listed as “Skipped (no branch): &lt;name&gt;”.
  - **Failed pulls**: The runner now returns an optional error message. For each failed repo, the summary shows the branch and a truncated one-line error (e.g. merge conflict or pull error).

### Code changes

- **extension.ts**: New handlers `handlePubGetAllInFolder`, `handleCleanAndPubGetAllInFolder`, `handleGitPullAllInFolder`; path prompt with default; discovery (direct children by `pubspec.yaml` or `.git`); loop over folders; summary written to output channel; for Git Pull, failed entries include `error` and it is printed in the summary.
- **package.json**: Three new entries in `contributes.commands` for the above commands.
- **buildTreeView.ts**: Three new sidebar items under UTILS (Pub Get / Clean & Pub Get “all in folder”) and Git Actions (Git Pull “all in folder”).
- **utilityRunner.ts**: `executeCommand` and `executeUtilityWithSession` now return `{ success: boolean; errorMessage?: string }`; `executeGitPull` returns that type; other utilities still return `Promise<boolean>` via `.then(r => r.success)`. Failed git pull error text is truncated (500 chars) and passed back for the summary.

---

## Benefits

- **Monorepos / multi-package workspaces**: Run pub get or clean & pub get across all packages under a single folder without opening each project.
- **Multiple git repos in one parent**: Pull all repos in one go (e.g. a folder of cloned projects) with one command.
- **Visibility**: Summary in Output shows exactly which modules were run, success/fail/skip, and (for Git Pull) the branch and a short error line for failures.
- **No ambiguity on path**: Default path is workspace or active file dir; user can confirm or change before running.
- **Failure handling**: Failed git pulls show the reason in the summary; merge conflicts or other errors are visible without scrolling past other repos’ output.

---

## Output structure

### 1. Pub Get (all in folder)

**Output channel (flutter-toolbox):**
```
============================================================
Pub get (all in folder) — Summary
============================================================
Parent folder: /path/to/packages
Packages run: 5

  ✓ package_a
  ✓ package_b
  ✗ package_c

Result: 2 succeeded, 1 failed.
============================================================
```

**Notification:** `Pub get (all in folder): 2 succeeded, 1 failed. See Output for details.`

---

### 2. Clean & Pub get (all in folder)

**Output channel (flutter-toolbox):**
```
============================================================
Clean & Pub get (all in folder) — Summary
============================================================
Parent folder: /path/to/packages
Delete pubspec.lock: Yes
Packages run: 5

  ✓ package_a
  ✓ package_b
  ✗ package_c

Result: 2 succeeded, 1 failed.
============================================================
```

**Notification:** `Clean & Pub get (all in folder): 2 succeeded, 1 failed. See Output for details.`

---

### 3. Git Pull (all in folder)

**Output channel (flutter-toolbox):**
```
============================================================
Git Pull (all in folder) — Summary
============================================================
Parent folder: /path/to/repos
Repos run: 21

  ✓ repo_a (main)
  ✓ repo_b (develop)
  ✗ repo_c (feature/xyz)
      fatal: Not possible to fast-forward, merge conflicts prevent merging...

Result: 20 succeeded, 1 failed, 0 skipped.
Repos with merge conflicts or pull errors show as failed; resolve them manually.
============================================================
```

**Notification:** `Git pull (all in folder): 20 succeeded, 1 failed, 0 skipped. See Output for details.`

- **Succeeded**: repo name and branch pulled.
- **Failed**: repo name, branch, and a truncated one-line error (e.g. merge conflict message).
- **Skipped**: repos where the current branch could not be resolved (e.g. detached HEAD); listed as `Skipped (no branch): <name>`.

---

## Testing suggestions

- Run each new command with a parent path that has multiple child packages/repos and confirm the summary matches expectations.
- For Git Pull (all in folder), trigger a failure (e.g. merge conflict) in one repo and confirm the failed line and error snippet appear in the summary.
