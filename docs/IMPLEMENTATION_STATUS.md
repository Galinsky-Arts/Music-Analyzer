# Implementation and next-agent handoff

Reconciled with GitHub on 2026-10-10. The complete user-approved plan is
[DEVELOPMENT_PLAN.md](DEVELOPMENT_PLAN.md); its original document remains archived
as [DEVELOPMENT_PLAN.docx](DEVELOPMENT_PLAN.docx). This status does not replace or
reduce the approved scope. Read [AGENTS.md](../AGENTS.md) before implementation.

## Review stack

Reconciled with GitHub on 2026-11-15. All 16 planned code PRs (PR01 through PR16)
are merged into `main`. PR16 (native Mac delivery) merged on 2026-10-10 as
`50b0c3cb11d8934c3123932e3bb458c86518c0a4`; later main commits are documentation,
registry, monitoring and CI changes (current main `754044c8f7f61ae3a5ca1d774f079a198df1de46`;
PR #24 moved Mac packaging to the macos-15 runner after GitHub retired macos-14).
The repository now lives at `Galinsky-Arts/Music-Analyzer`; run links to
`neophilism/Music-Analyzer` below are historical evidence from before the move.

Before merge, PR16 head `5c051162e7820354353433da59e3ede7f636410f` passed all four jobs in
[Tests run 38067565724](https://github.com/neophilism/Music-Analyzer/actions/runs/38067565724):
Linux core, macOS 15 core, macOS 14 package, and macOS 15 clean installation.
The current unsigned review package is built from `main` by the same `Tests`
workflow; see the README for the latest run and artifact. The live transition record and next actions are in
[repo-development-progress.md](../repo-development-progress.md). No public
release or signing success is implied by a source merge.

Open, unmerged: PR19 (E2EESA status tracking, documentation/records) and PR20
(CircleCI full suites; on hold because CircleCI macOS executors cost credits).
Neither is required for the merged product scope.

| PR | Implementation | Verification / remaining limits |
| --- | --- | --- |
| 01 | Native architecture, model/license choice, complete plan | Merged; documentation-only audit |
| 02 | Python/Qt shell, entry point and core CI | Merged; Linux and macOS checks passed |
| 03 | Imports, hashes, stable IDs, duplicates and SQLite | Merged; Linux and macOS checks passed |
| 04 | Bounded spawned workers, cache and cancel/resume | Merged; Linux and macOS checks passed |
| 05 | Original-level LUFS, LRA and volume variability | Merged; independent reference checks |
| 06 | Local tempo, variability, ambiguity and tap support | Merged; click and ramp fixtures |
| 07 | Tuned chroma, key/mode candidates and uncertainty | Merged; tonal and unknown fixtures |
| 08 | Verified pinned installation and offline genres | Merged; actual checkpoint inference tested |
| 09 | Instrumental valence, arousal and moods | Merged; held-out listening accuracy remains open |
| 10 | Frozen corpus/reference scales and intensity presets | Merged; actual owner anchor recordings remain needed |
| 11 | Ascending buckets, album DP, endpoints and pins | Merged; exhaustive small-case comparisons |
| 12 | Native listening/review/plots and album editing | Merged; manual Mac playback/accessibility remains open |
| 13 | Personal anchors, positivity and intensity calibration | Merged; independent automatic values retained |
| 14 | Verified numbered copy/move, reports, resume and undo | Merged; platform collision and fault-injection tests |
| 15 | 200-track benchmark and independent evaluation tools | Merged; synthetic fixtures are not listening evidence |
| 16 | Mac bundle, package verification, notices and signing workflow | Merged (`50b0c3c`); all four CI jobs passed. Signing/notarization still needs Apple credentials |
| 17–18 | Cleared labeled data and supervised heads | Conditional; not activated without inadequate-baseline evidence and usable data |

## Earlier CI failures and repairs

PR02–PR13 failed on Linux because Qt could not load `libEGL.so.1`; macOS passed.
The workflows install the Qt/audio runtime libraries before running the same
tests. PR14's Mac tests assumed case/Unicode collisions could only be rejected
during execution, while macOS correctly rejected them at preview. The tests
cover both rejection points without weakening unrelated-byte, source-change or
permission checks. Repairs were carried through dependent branches with ancestry
preserved; no tests were removed or disabled.

Exact earlier branch heads and successful runs remain in
[ci-repair-2026-10-10.json](../validation/ci-repair-2026-10-10.json). Judge current
head checks rather than historical red runs. Package builds depend on both core
matrix jobs succeeding.

### Signed-build Python regression

The first PR16 package run,
[38066288991](https://github.com/neophilism/Music-Analyzer/actions/runs/38066288991),
failed in `actions/setup-python`: Python 3.12.15 was unavailable for the macOS 14
arm64 runner. The unsigned workflow was repaired to use 3.12.10 and successfully
built and verified the native application, but the manual signing workflow
still requested 3.12.15. This repair aligns signing with the tested 3.12.10 pin.

`tests/test_macos_workflow_configuration.py` checks equal explicit Python patch
versions, equal package/signing runners, and the package dependency on core CI.
The regression failed on the old signing pin (one failure, two passes) and passes
all three checks after the repair. Full lint, source tests, benchmarks, actual
model tests and frozen Mac verification must pass on the new PR head. Signing
itself still requires separately configured Apple credentials; the static guard
is not evidence of signing, notarization or Gatekeeper acceptance.

## Evidence and limits

The earlier implementation handoff reports 76 local core tests and two separate
actual-model tests. The pinned checkpoint passed CPU inference and invariance to
opposite filenames and listener overrides. Loudness is compared independently
with DeMan/reference BS.1770 and FFmpeg; tempo uses clicks and ramps, tonality
uses tuned chords, sequencing has exhaustive small cases, and export tests inject
interruptions around publication, source removal, backup, recovery and undo.
New test totals must be read from the current CI run, not inferred from this
historical baseline.

[batch-200-linux.json](../validation/batch-200-linux.json) records 200 unique
12.5-second WAV/FLAC/AIFF/OGG fixtures (50 each), one worker, cancellation after
five saved tracks and restart of the remaining 195. Analysis including restart
took 18.95 seconds; cached rerun 0.97 seconds; scoring/order 0.047 seconds; verified
200-file export 3.28 seconds. Largest parent/individual child RSS was 489.5 MiB,
not aggregate memory. These are short synthetic acoustic fixtures with models
disabled. Their manual labels exercise mechanics, not prediction accuracy.
Both core CI platforms retain fresh benchmark artifacts.

The successful PR16 Mac run also verifies the frozen GUI, spawned workers,
offline model outputs, close/reopen, album order and byte-verified two-file
export. Its macOS 15 clean-install job uses the application without installing
developer Python dependencies, and explicitly installs the model through the
packaged CLI before offline verification. Synthetic listener assignments are
limited to smoke-test ordering/export. They do not establish listening quality.

The following remain release gates, not completed work:

- Recording-disjoint held-out genre/valence/mood evaluation and intensity pairs;
  actual quiet-piano and dense-metal anchor recordings.
- Full-length 100–200-track runtime and aggregate memory on a 16 GB Apple Silicon
  Mac, plus real playback, appearance, keyboard use and accessibility checks.
- Final linked-library notice/source/replacement audit, Apple signing and
  notarization, and clean-device Gatekeeper verification.

No owner recordings, listener annotations or Apple signing credentials have
been supplied in this session. Conditional PR17–18 need evidence that the baseline
is inadequate plus cleared labeled data; neither outcome may be fabricated.

## Where to work

| Area | Modules |
| --- | --- |
| Domain, project and import | `domain.py`, `project.py`, `audio.py` |
| Cache and spawned analysis | `jobs.py`, `engine.py` (algorithm version 6) |
| Acoustic measurements | `loudness.py`, `rhythm.py`, `tonality.py` |
| Learned interpretation | `models.py`, `emotion.py`, `resources/clap_manifest.json` |
| Scores, personal fitting and order | `scoring.py`, `calibration.py`, `sequencing.py` |
| Native desktop | `app.py`, `window.py`, `ui_support.py` |
| Export and recovery | `exporting.py`; `docs/EXPORT_RECOVERY.md` |
| Benchmark and held-out evaluation | `benchmark.py`, `evaluation.py`; `docs/EVALUATION.md` |
| Frozen delivery | `package_check.py`, `packaging/`, `.github/workflows/` |

Use README setup/check commands. Pin and verify the existing checkpoint; do not
fetch arbitrary revisions. Preserve original-level measurements and immutable
automatic results. Never infer mood from names, tags, lyrics, major/minor or a
hand-written tempo rule. Missing model output stays unknown, not fabricated.

Before merging, verify current checks and unresolved reviews, retarget a stacked
PR to main after its predecessors merge, and use merge commits to preserve
ancestry. Never force over concurrent work. Before release complete the
[Mac checklist](MAC_RELEASE_CHECKLIST.md), update the evidence, PR description
and [EVALUATION.md](EVALUATION.md), and retain the entire approved plan.
