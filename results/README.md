# Recorded measurements

The 2026-10-21 workerd and original Basecamp measurements were collected together in [celld-campfire-bench at commit 439b240573074e3171166fa7286dda117f5b595d](https://github.com/hyfdev/celld-campfire-bench/tree/439b240573074e3171166fa7286dda117f5b595d). Their shared cohort identifier is `2026-10-21T05:52:59Z`.

| Directory | Measurements |
| --- | --- |
| [2026-10-21-workerd](2026-10-21-workerd/REPORT.md) | Three workerd modes, 63 load trials, 30 startup trials, and functional validation |
| [2026-10-21-basecamp](2026-10-21-basecamp/REPORT.md) | Original Basecamp Rust application, 12 load trials, 10 startup trials, and functional validation |

The raw JSON and `sources/` snapshots are copied byte for byte. Snapshot paths retain the measured repository layout, including `experiments/` and shared celld benchmark helpers. The active scripts in this repository use the standalone `workerd/`, `basecamp/`, and `bench/` layout. Reports and README tables are generated from the original data with local links.

Every data file records source hashes, a seed hash map, and a source snapshot path. Executables are represented by hashes rather than shipped in the repository. The build manifests record runtime/compiler versions and flags. Report checks verify the captured files against the recorded hashes and recompute table values from individual trials.

The standalone extraction keeps the measured application, Wasm module source, C++ adapter, and workerd patch unchanged. File relocation and measurement-script changes have not produced a new performance cohort. Future runs write separate data under `.runs/`.

```sh
python3 workerd/report.py --check --readme
python3 basecamp/report.py --check --readme
```
