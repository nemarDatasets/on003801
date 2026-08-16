[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on003801-blue)](https://doi.org/10.82901/nemar.on003801)

This mobile EEG auditory attention experiment consists of 20 participants.
In a two-competing speaker paradigm subjects either sat on a chair or walked a route indoors 
Attention was disrupted by environmental salient eventsfrom in front of the participant 

- Lisa Straetmans (Sep, 2021)

## NEMAR curation changes (2026-05-21, revised 2026-05-27)

The BIDS validator went from 21 errors + 944 warnings to 0 errors + 662 warnings. None of the raw `.set` or `.fdt` files were modified; every change is to a text sidecar.

**Events sidecar consolidated at the dataset root (`task-NeuralTrackingToGo_events.json`)**
- A single root-level events sidecar was created and the 20 per-recording `sub-NNN/eeg/sub-NNN_task-NeuralTrackingToGo_events.json` files were removed. The per-recording sidecars were byte-identical to each other, so one root sidecar covers all 20 recordings via BIDS inheritance and avoids having to re-edit twenty identical files.
- The `value` column is declared as free-form text rather than as an enumerated set. The previous per-recording sidecar listed only three allowed labels (`no`, `play`, `yes`, all button presses) under `value.Levels`, but the actual event tables contain about thirty distinct labels (`start_exp`, `rest_sit`, `rest_stand`, `start_sound1_sit`, `stop_sound1_walk`, and so on). Every label outside the three-item enum was being rejected as the wrong type. Dropping the closed enum lets every observed label through. The trial-type labels remain declared under `trial_type.Levels`, unchanged from the original sidecar.
- The `sample` and `response_time` columns are declared with their BIDS-canonical types (`sample` as an integer sample index, `response_time` with `Units: "s"`) so the validator treats them as the documented standard columns rather than flagging them as undefined.
- `onset` and `duration` were left undeclared on purpose. The BIDS built-in schema already fixes their units to seconds, and redeclaring them in a sidecar would trip a column-type-redefined warning.

**Electrode files renamed (`sub-NNN/eeg/sub-NNN_task-NeuralTrackingToGo_electrodes.tsv`, 20 files)**
- Each file was renamed to `sub-NNN/eeg/sub-NNN_electrodes.tsv`, dropping the `task-` entity from the filename. Electrode positions are a property of the subject, not of a particular task, and the entity-bearing filename was firing an excessive-specificity warning on every recording. File contents are unchanged; only the filename changed.

**Coordinate-system files renamed (`sub-NNN/eeg/sub-NNN_task-NeuralTrackingToGo_coordsystem.json`, 20 files)**
- Same rename pattern: each file became `sub-NNN/eeg/sub-NNN_coordsystem.json`. Coordinate-system metadata is subject-level per BIDS, so the `task-` entity does not belong in the filename. Contents are unchanged.

**Channel tables (`sub-NNN/eeg/sub-NNN_task-NeuralTrackingToGo_channels.tsv`, 20 files)**
- The `units` column for the 24 EEG rows in each file was changed from `microV` to `uV`. `microV` is not a BIDS-recognised spelling for microvolts; the canonical forms are `uV` or `µV`. This is a textual unit label only; the recorded signal magnitudes are unchanged.

**Dataset description (`dataset_description.json`)**
- Added `DatasetType: "raw"` so the dataset is validated against the raw-data rules rather than the derivative rules.
- Updated `BIDSVersion` from `"v2.0"` to `"1.11.1"` (the version the current validator checks against). The original value carried a non-canonical `v` prefix and sat below the validator's recognised-version floor.
- `GeneratedBy` was left absent, exactly as the source published it; nothing was added there.

**Remaining warnings (662), left on purpose**
- These are all "recommended but missing" fields that need information from the study, lab, or equipment that isn't in the dataset (for example: `Manufacturer`, `CapManufacturer`, `CapManufacturersModelName`, `EEGGround`, `HeadCircumference`, `HardwareFilters`, `SoftwareVersions`, `DeviceSerialNumber`, `Instructions`, `SubjectArtefactDescription`, `CogAtlasID`, `CogPOID`, the fiducial- and anatomical-landmark coordinate fields in the coordinate-system sidecars, and `GeneratedBy` at the dataset root). They were left blank rather than filled with guesses.