# Reproduce the workerd chat benchmark

Run all commands from the repository root. Python 3.11+ can verify the published results on any operating system:

```sh
python3 workerd/report.py --check --readme
python3 basecamp/report.py --check --readme
```

## Build

Building and resource sampling use macOS. Install Git, Xcode Command Line Tools, Python 3.11+, Rustup, and [Bazelisk](https://github.com/bazelbuild/bazelisk), with `bazelisk` on PATH. Workerd selects its Bazel version; the recorded build uses Bazel 9.3.0. The first build compiles V8 and needs substantial disk space and memory.

```sh
python3 workerd/build_workerd.py
```

The builder fetches the exact revision in [workerd.json](workerd/workerd.json), applies [workerd.patch](patches/workerd.patch), copies the C++ Wasm adapter into workerd, and builds with `--config=release --jobs=8`. It also builds the Rust Wasm application and load clients with locked dependencies. The executable is `.build/workerd-host/workerd`; `.build/workerd-host/build.json` records source and executable hashes.

Stop the builder's Bazel server before measuring:

```sh
(cd .build/workerd-source && bazelisk --output_user_root="$PWD/../workerd-bazel" shutdown)
```

## Run all three workerd modes

Port **19880** must be free. Run on an otherwise idle machine:

```sh
python3 workerd/run.py all
python3 workerd/report.py .runs/workerd > .runs/workerd/REPORT.md
python3 workerd/report.py .runs/workerd --check
```

`all` prepares a common seed, validates the three modes, runs three 5-second trials per workload with 32 HTTP clients, and measures ten fresh processes per mode. Mode order rotates between rounds. Workloads cover server-rendered HTML with page caching off/on, JSON reads, saved posts, mixed reads/posts, and live WebSocket broadcasts with HTTP read confirmations. Every trial restores the seed and ends with an audit of all 50 SQLite databases.

For a shorter full validation and measurement run:

```sh
python3 workerd/run.py all --seconds 1 --rounds 1 --cold-rounds 1
```

Individual phases:

```sh
python3 workerd/run.py prepare
python3 workerd/run.py validate
python3 workerd/run.py bench --seconds 5 --rounds 3
python3 workerd/run.py cold --cold-rounds 10
```

Measurements, logs, generated configurations, seed hashes, and measured source snapshots live under `.runs/workerd/`. The runner verifies the build manifest and stops its own server processes. Fresh-process measurements include Wasm compilation; no compiled-module cache is configured. Filesystem caches stay warm.

To measure Basecamp's original Rust application in the same session, follow [the original application comparison](basecamp/README.md#run-the-comparison).

## Open the chat app

After building, prepare the seed and select a mode:

```sh
python3 workerd/run.py prepare
PYTHONPATH=workerd python3 -c '
from run import server
with server("js", "interactive"):
    print("Open http://127.0.0.1:19880; press Enter to stop.")
    input()
'
```

Replace `js` with `wasm-js` or `wasm-v8` to use another mode. Users 1–7 have passwords `password-1` through `password-7`. The browser supports login, room history, posting, and live broadcasts.
