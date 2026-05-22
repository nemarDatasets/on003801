[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on003801-blue)](https://doi.org/10.82901/nemar.on003801)

This mobile EEG auditory attention experiment consists of 20 participants.
In a two-competing speaker paradigm subjects either sat on a chair or walked a route indoors 
Attention was disrupted by environmental salient eventsfrom in front of the participant 

- Lisa Straetmans (Sep, 2021)

## NEMAR curation changes (2026-05-21)

BIDS validator: 21 errors + 944 warnings → 0 errors + 661 warnings. Raw `.set`/`.fdt` binary payloads unchanged.

### `dataset_description.json`
- `BIDSVersion`: `"v2.0"` → `"1.8.0"`. Why: the previous value carried a non-canonical `v` prefix and is below the validator's recognised-version floor; the validator fired `UNKNOWN_BIDS_VERSION` on the dataset root.
- Added `DatasetType: "raw"`. Why: BIDS-validator otherwise infers a derivative-rules cascade when `DatasetType` is missing alongside `GeneratedBy`, producing spurious warnings.
- Added `GeneratedBy: [{Name: "nemar-cli", Version: "0.8.8", CodeURL: "https://github.com/nemar-org/nemar-cli"}]`. Why: records the NEMAR rehost step in the dataset's provenance chain.

### `sub-NNN/eeg/sub-NNN_task-NeuralTrackingToGo_electrodes.tsv` (20 files)
- `git mv` to `sub-NNN/eeg/sub-NNN_electrodes.tsv` (dropped the `task-` entity). Why: electrode positions are subject-level per BIDS, not task-level; the entity-bearing filename fires `EXCESSIVE_ELECTRODE_SPECIFICITY` on every recording.

### `sub-NNN/eeg/sub-NNN_task-NeuralTrackingToGo_coordsystem.json` (20 files)
- `git mv` to `sub-NNN/eeg/sub-NNN_coordsystem.json` (same rationale). Why: coordinate-system files are subject-level per BIDS; closes `EXCESSIVE_COORDSYSTEM_SPECIFICITY`. File contents unchanged.

### `sub-NNN/eeg/sub-NNN_task-NeuralTrackingToGo_channels.tsv` (20 files)
- `units` column: `microV` → `uV` for all 24 EEG rows in every file. Why: `microV` is not a BIDS-recognised unit string for microvolts; the canonical form is `uV` (or `µV`). The actual signal magnitudes are unchanged — only the textual unit label.

### `task-NeuralTrackingToGo_events.json` (new, inheriting root sidecar)
- Created at the dataset root, replacing the 20 byte-identical per-recording `_events.json` files (now removed). One root sidecar inherits to all recordings.
- Declares the non-canonical `value` column as free-form text. Why: the per-recording `value` column contains string event labels (`start_exp`, `play`, `no`, `yes`, `rest_sit`, `rest_stand`, `start_sound{1-6}_{sit,walk}`, `stop_sound{1-6}_{sit,walk}`, …) — 30 distinct labels in total. The old per-recording sidecar declared only 3 of those labels in `value.Levels` (`no` / `play` / `yes`, all button presses), causing the BIDS-validator to reject every other label as the wrong type (20× `TSV_VALUE_INCORRECT_TYPE:value @ msg='start_exp'` errors, one per recording). Dropping the `Levels` enum lets every observed label through. The 27 trial-type labels also remain declared under `trial_type.Levels` (unchanged from the original sidecar).
- Declares `sample` (BIDS column, was undocumented) and re-declares `response_time` with `Units: "s"` (the per-recording sidecar omitted the unit) so the canonical schema applies.
- `onset` / `duration` are intentionally NOT redeclared — the BIDS built-in schema already enforces `Units: "s"` and re-declaring them with a free-form `Description` would fire `TSV_COLUMN_TYPE_REDEFINED`.

### `sub-NNN/eeg/sub-NNN_task-NeuralTrackingToGo_events.json` (20 per-recording sidecars)
- Removed. Why: every per-recording file was byte-identical; the new inheriting root `task-NeuralTrackingToGo_events.json` covers all 20 recordings via BIDS inheritance. Removing the redundant per-recording copies also avoided having to re-edit all 20 to drop the wrong `value.Levels`.