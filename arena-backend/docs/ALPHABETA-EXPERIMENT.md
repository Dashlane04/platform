# OTTv2 Alpha–Beta example: local experiment

Source: `../examples/alphabeta/bot.cpp`; algorithm notes:
`../examples/alphabeta/README.md`. Run from `arena-backend` in WSL:

```bash
.venv/bin/python tests/alphabeta_experiment.py
```

Uses a separate directory under `test-results/`, never the live `.platform-state`.
Builds and executes using the real Docker worker image. No host execution fallback.

Verified 2026-10-10 with C++17/O2 and the 15,000 ms turn profile:

| Check | Result |
|---|---|
| C++ rules vs canonical Python engine | 104 positions, 4,556 legal transitions match |
| Incremental Zobrist hash vs full recomputation | All transitions match |
| Real worker benchmark | 72/72 legal actions, all received before deadline |
| Benchmark wall time | median 250.329 ms; P95 250.638 ms; max 255.333 ms |
| Immediate base win | h8 → i9 |
| Eliminate last piece of a type | e5 → f5 |
| Defend base | c2 → b2 |
| Save last scissors from rock | e5 → d6 |
| Advanced bot as Red vs original starter | Red wins by type elimination, 63 plies |
| Advanced bot as Blue vs original starter | Blue wins by type elimination, 46 plies |

Neither match had protocol, illegal-action or timeout faults. Benchmark timing
includes pipe/supervisor overhead; it is not algorithm-only CPU time. Cache survives
between observations, as in production, and the public suite repeats its 24 states
three times. These measurements are specific to this local machine and worker image.
Two matches against one simple starter do not establish competitive strength.
Random differential coverage is evidence, not a proof of equivalence for all boards.

Full machine-readable report (ignored test output):
`../test-results/alphabeta-dg1ehgbg/report.json`, including image/engine digests,
build provenance, per-turn timings, CPU and memory metrics.
