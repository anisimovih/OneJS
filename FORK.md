# Fork notes (anisimovih/OneJS)

This fork exists to send fixes upstream to `Singtaa/OneJS` and to give a game a commit to pin while they wait. This file and its `CLAUDE.md` import live only on the fork's `main`; they must never reach an upstream PR.

## Branches

- `main` = upstream `main` + the one commit that adds this file. To sync: fetch `upstream`, rebase that commit onto `upstream/main`, force-push `main`.
- **A fix branch starts from `upstream/main`, never from fork `main`** (that would carry this file into the PR). Before every push: `git diff upstream/main --stat` lists only the fix's files.
- A pin branch `pin/<base>-<prs>` is what a game pins: the base commit plus the PR commits cherry-picked on it, e.g. `pin/028525a-158-159`. Cherry-pick the commit that is on the PR branch now (after any amend), not an older local one.

## Upstream PRs

- Commits and PR text carry the author alone: no `Co-Authored-By` trailer, no "Generated with" footer. Check `git log --format=%B upstream/main..HEAD` before pushing.
- Leave local-machine problems out of PR text (locale, local compile patches, how the run was set up). State the tests added, red on `main` and green with the fix, and the full-suite result.
- Shape that has worked: Problem (with numbers from the real case), Fix (bullets), Tests (each new test: what it checks, fails on `main` / passes with the fix), full suite line.

## Running the tests

OneJS tests run inside a Unity project that uses the package:

1. In a Unity project no editor has open, point `Packages/manifest.json` at the checkout and make the package testable:
   ```json
   "com.singtaa.onejs": "file:<path to this checkout>",
   ...
   "testables": ["com.singtaa.onejs"]
   ```
2. Run PlayMode in batch mode **with graphics** (`-nographics` fails ~85 GPU-dependent tests):
   ```
   Unity.exe -batchmode -runTests -disable-assembly-updater -projectPath <project>
     -testPlatform PlayMode -testFilter OneJS.Tests -testResults <out>/results.xml -logFile <out>/run.log
   ```
   A single test: `-testFilter OneJS.Tests.<Fixture>.<Test>`. Unity exits on its own; watch the log, a hung run never exits.
3. Restore the project's manifest and `packages-lock.json` afterwards.

Red then green: run the new test with the runtime change stashed (it must fail for the reason the PR describes), then with it; then the full suite.

## Known local failures (not the fix's, never in a PR)

- **Decimal-comma locale** (e.g. ru-RU): 14 tests fail on `0,5` vs `0.5` (FilterStyle, ProcNoise, FastPath Color/Quaternion, Time.deltaTime, ModelLight bake). On Windows, for the run only: `reg export "HKCU\Control Panel\International" <backup>.reg`, `Set-Culture en-US`, run, then `reg import <backup>.reg`.
- **Unity 6000.3**: `Tests/Models/Editor/ActorLightEditModeTests.cs` line 46 does not compile (`FindObjectsByType<Light>(FindObjectsInactive.Exclude)` has no one-argument overload there). Add `, FindObjectsSortMode.None` locally; never commit it.

Last full green run: 956/956 on upstream `e638dd5` + #159 (Unity 6000.3.10f1, Windows, 2026-11-22).
