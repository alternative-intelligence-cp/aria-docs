# GC Tuning

The Nitpick garbage collector has three knobs you can turn from the
environment without recompiling. They are all read once, at GC
initialization time, by `GCState::init()` in
`src/runtime/gc/gc.cpp`.

| Variable                      | Units | Default              | Effect                                |
| ----------------------------- | ----- | -------------------- | ------------------------------------- |
| `NPK_GC_NURSERY_SIZE`         | bytes | `4194304` (4 MB)     | Capacity of the copying nursery.      |
| `NPK_GC_OLD_GEN_THRESHOLD`    | bytes | `67108864` (64 MB)   | Mark-sweep old generation threshold.  |
| `NPK_GC_MODE`                 | enum  | `stw`                | `concurrent` enables SATB background mark; `stw` is stop-the-world. |

Sizes accept either decimal (`8388608`) or hex (`0x800000`) input via
`strtoull(env, nullptr, 0)`. A value of `0` or any unparseable input
falls back to the default for that knob.

## Resolution order

For each size knob the runtime applies the first non-zero value it
finds in this order:

1. The argument passed to `npk_gc_init(nursery_size, old_gen_threshold)`,
   if non-zero. (Programs that initialize the GC explicitly always win.)
2. The matching `NPK_GC_*` env-var, if set and parseable as a non-zero
   integer.
3. The compiled-in default (4 MB / 64 MB).

For `NPK_GC_MODE` there is no API equivalent at init time;
`npk_gc_enable_concurrent(1)` can flip the flag later, but the env-var
is the only way to start in concurrent mode from the first allocation
onward.

## When to tune

- **Long-running services with bursty allocation**: a larger nursery
  (8–32 MB) reduces minor GC frequency at the cost of longer per-cycle
  pauses. Concurrent mode (`NPK_GC_MODE=concurrent`) keeps the major
  mark phase off the mutator critical path.
- **Memory-constrained embedded targets**: shrink the nursery (256 KB
  – 1 MB) and cap the old generation. Smaller heaps trade throughput
  for footprint.
- **Throughput-bound batch jobs**: increase
  `NPK_GC_OLD_GEN_THRESHOLD` so the major collector runs less often;
  leave the nursery near the default.

The runtime exposes three small queries the test suite uses to verify
that env-var resolution worked, and that you can also call from your
own code:

```c
size_t  npk_gc_nursery_size_bytes(void);
size_t  npk_gc_old_gen_threshold_bytes(void);
uint8_t npk_gc_concurrent_enabled(void);  // 1 if concurrent, else 0
```

These are declared in `include/runtime/gc.h` and lazy-initialize the
GC if it has not started yet.

## Sane minimums

Both size knobs are in raw bytes. The runtime does not enforce a
minimum — picking a value that is too small (e.g. less than the
allocation header overhead) will surface as immediate evacuation
failures or out-of-memory errors. As a rough rule, do not go below
`262144` (256 KB) for the nursery or `1048576` (1 MB) for the old
generation unless you know the workload's working set is genuinely
that small.

## Modes

`NPK_GC_MODE` accepts:

- **`stw`** *(default)* — stop-the-world mark and sweep. Lowest
  implementation cost, longest pauses.
- **`concurrent`** — background mark thread with SATB
  (snapshot-at-the-beginning) write barrier. Sweep remains
  incremental and amortized across allocation calls. Recommended for
  latency-sensitive applications.
- **`off`** — *deferred*. There is no infrastructure to disable the
  collector at runtime; the value is silently ignored today. If you
  need a no-GC build, link against the wasm runtime variant or set
  the heap large enough that collection never triggers in practice.
  Tracked for a future release.

Unrecognized values are ignored and treated as `stw`.

## Verification

The CTest target `bug_tests_v0264` exercises all three env-vars and
the no-env-var default fallback in fresh subprocesses. Each fixture
queries the runtime via the small `npk_gc_*` accessors above and
exits zero only when the resolved value matches the expectation.

```bash
cd build && ctest -R bug_tests_v0264 --output-on-failure
```
