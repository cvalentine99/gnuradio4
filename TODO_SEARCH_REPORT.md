# TODO & Incomplete Feature Report — GNU Radio 4.0

**Generated:** 2026-02-14
**Scope:** Full codebase search for `TODO`, `FIXME`, `HACK`, `WORKAROUND`, `WIP`, and incomplete implementation markers.

---

## Executive Summary

| Marker Type         | Count |
|---------------------|-------|
| TODO                | 200+  |
| FIXME               | 7     |
| WORKAROUND          | 15*   |
| WIP                 | 6     |
| Not Implemented     | 10    |
| **Total**           | **238+** |

_\*Excludes third-party code (magic_enum)._

The largest concentrations of incomplete work are in the **core runtime headers** (`Block.hpp`, `Port.hpp`, `Graph.hpp`), which collectively account for roughly 40% of all TODOs. Several markers indicate **architectural decisions still pending**, particularly around error handling, port naming, SIMD multi-output support, and GPU compute backends.

---

## 1. Critical / Architectural TODOs

These items represent fundamental design decisions or missing capabilities that affect the framework's architecture.

### 1.1 SIMD Multi-Output Port Support (Not Implemented)
- `core/include/gnuradio-4.0/Graph.hpp:1278` — `"TODO: SIMD for multiple output ports not implemented yet"` — A `static_assert` currently blocks SIMD connections for blocks with more than one output port.

### 1.2 GPU Compute Backend (Not Implemented)
- `core/include/gnuradio-4.0/TensorMath.hpp:743` — `"GPU GEMM not yet implemented"`
- `core/include/gnuradio-4.0/TensorMath.hpp:774` — `"GPU GEMV not yet implemented"`
- `core/include/gnuradio-4.0/TensorMath.hpp:76` — `Dynamic = -1 /// runtime decision (not yet implemented)`

### 1.3 Polynomial Interpolation (WIP)
- `algorithm/include/gnuradio-4.0/algorithm/SchmittTrigger.hpp:37,42,203` — Polynomial interpolation for the Schmitt trigger and Trigger blocks requires Tensor and SVD implementations that don't exist yet.
- `blocks/basic/include/gnuradio-4.0/basic/Trigger.hpp:29` — Same dependency.

### 1.4 Error Handling Strategy (Undecided)
- `core/include/gnuradio-4.0/Port.hpp:700` — `"TODO(error handling): Decide how to surface failures."`
- `core/include/gnuradio-4.0/Port.hpp:981` — Same pattern, repeated.

### 1.5 Port API Naming
- `core/include/gnuradio-4.0/Port.hpp:1077` — `portName()` should be renamed to `name()`.
- `core/include/gnuradio-4.0/Port.hpp:1080` — `portInfo()` should be renamed to `type()`.
- `core/include/gnuradio-4.0/PortTraits.hpp:46` — `FIXME: better name "describes_" instead of "is_"?`

### 1.6 Graph Port Collections
- `core/include/gnuradio-4.0/Graph.hpp:169` — `"TODO: Add support for exporting port collections"`
- `core/include/gnuradio-4.0/Graph.hpp:1178` — `FIXME: How do we refuse connection to a vector<Port>?`

### 1.7 Cross-Graph Block Connections
- `core/test/qa_BlockModel.cpp:277` — `"FIXME: discuss how to connect from/to blocks in different (sub-)graphs"`

---

## 2. Core Runtime (`core/`)

### 2.1 Block.hpp (12 TODOs)
| Line | Description |
|------|-------------|
| 492 | Requirements for block API need discussion |
| 761 | C++26: ensure certain members are not reflected |
| 871 | Refactor: library should not assign names to ports on behalf of blocks |
| 953 | Obsolete lines to be removed |
| 1091 | `_mergedInputTag` filling to be removed in future cleanup |
| 1126 | `autoUpdate` does not need full `Tag` |
| 2037 | Dead code: `"finally remove me"` |
| 2055 | Evaluate if merged block limit special cases can be eliminated |
| 2070 | EOS policy missing for "no work to perform" path |
| 2314 | Emscripten deadlock workaround |
| 2636 | FIXME: inconsistent template specialization for block macros |

### 2.2 Port.hpp (11 TODOs)
| Line | Description |
|------|-------------|
| 543 | Default buffer size (4096) — needs review of max initial buffer limit |
| 700, 981 | Error handling strategy undecided |
| 726 | `_cachedTag` only used in output ports — scope unclear |
| 778 | Mixing ports with different buffer types not supported |
| 1077 | Rename `portName()` → `name()` |
| 1080 | Rename `portInfo()` → `type()` |
| 1151 | Return signature of `connect()` needs update |
| 1227 | Dangerous `const_cast` — temporary fix, PR follow-up volunteered |
| 1235 | Port lifetime management is problematic |

### 2.3 Graph.hpp (5 TODOs, 1 FIXME)
| Line | Description |
|------|-------------|
| 169 | Port collection export support missing |
| 1178 | FIXME: vector<Port> connection refusal |
| 1200 | Unique ID rationale needs documentation |
| 1278 | SIMD multi-output not implemented |
| 1331 | `work::Result` — check if still needed |
| 1412 | Enum formatter improvement needed |

### 2.4 Scheduler.hpp
| Line | Description |
|------|-------------|
| 181 | `max_work_items` — check whether this is the right default |
| 512 | Workaround: `sleep_for` for incomplete `std::atomic` in nodejs/Emscripten |

### 2.5 Thread Pool (`thread/thread_pool.hpp`) (6 TODOs)
| Line | Description |
|------|-------------|
| 28 | Remove manual implementations and use `std` when moved to modules |
| 122, 132 | Replace with `std::ranges::equal` / `std::lexicographical_compare_three_way` |
| 148 | C++23 STL replacement pending |
| 493 | Incomplete TODO (no description) |
| 702 | Windows/Apple/other platform support not implemented |

### 2.6 Other Core Files
| File | Line | Description |
|------|------|-------------|
| `Buffer.hpp` | 53, 80, 90, 91 | Concepts need better placement; return type docs incomplete |
| `BufferSkeleton.hpp` | 5 | Unclear include guard dependency |
| `Tag.hpp` | 54, 181 | Convenience method relevance; backward compat alias |
| `Settings.hpp` | 76 | Should throw if types mismatch? |
| `annotated.hpp` | 68, 461 | Replace `bool` with enum/tag; brief/meta print mode |
| `BlockModel.hpp` | 668, 894 | Mangled type names; `meta_information` to be read-only |
| `Profiler.hpp` | 85 | Empty `default:` case |
| `PluginLoader.hpp` | 59, 67 | RTLD_LOCAL rationale; dlsym void* → function pointer UB |
| `Sequence.hpp` | 85, 88 | libc++ `std::atomic<shared_ptr>` support; GCC deprecation workaround |
| `Graph_yaml_importer.hpp` | 270, 276 | Missing unit test; scheduler parameters not serialized |
| `CircularBuffer.hpp` | 78 | Allocation retry workaround |
| `YamlPmt.hpp` | 1135 | `std::list` workaround for ASAN + Emscripten |
| `BlockTraits.hpp` | 133, 320, 346 | `requires` usage; Clang bug workaround; redundant check |
| `src/Graph.cpp` | 7 | Hardcoded `reserve(100)` to be removed |
| `src/PluginLoader.cpp` | 10, 18 | Proper path resolution; Windows path separator |

---

## 3. Block Libraries (`blocks/`)

### 3.1 basic/
| File | Line | Description |
|------|------|-------------|
| `FunctionGenerator.hpp` | 17, 138 | Enum values need CamelCase; type alias examples incomplete |
| `DataSink.hpp` | 75, 617, 636 | Port reuse consideration; mutex vs lock-free; **"TODO Important!"** (unspecified) |
| `Selector.hpp` | 84 | PMT exception handling for async input ports |
| `StreamToDataSet.hpp` | 201, 288 | Overlapping dataset workaround; empty DataSet publishing question |
| `PythonBlock.hpp` | 84, 183, 188 | `poc_property_map` needs replacement; settings API migration |
| `Trigger.hpp` | 29 | WIP: needs Tensor + SVD |
| `sources (test)` | 67, 76 | Decimator/stride breaking limits; threading issue |

### 3.2 fourier/
| File | Line | Description |
|------|------|-------------|
| `fft.hpp` | 118, 240 | Caching vectors should move to FFT implementations; timing event propagation missing |

### 3.3 electrical/
| File | Line | Description |
|------|------|-------------|
| `PowerEstimators.hpp` | 103, 104 | Uncertainty propagation needs checking in HP filter |

### 3.4 testing/
| File | Line | Description |
|------|------|-------------|
| `ImChartMonitor.hpp` | 109 | No proper API for clearing tags |
| `SettingsChangeRecorder.hpp` | 60 | `BlockingIO` re-enablement |

---

## 4. Algorithm Library (`algorithm/`)

| File | Line | Description |
|------|------|-------------|
| `SchmittTrigger.hpp` | 37, 42, 203 | Polynomial interpolation needs Tensor + SVD |
| `ImChart.hpp` | 263, 285 | Clang/Emscripten CI workaround to be removed |
| `FilterTool.hpp` | 663 | `std::runtime_format` (C++26) workaround |
| `DataSetEstimators (test)` | 565 | WIP: more DataSet math functionality planned |
| `FilterTool (test)` | 269 | Emscripten test failure needs investigation |

---

## 5. Build & Configuration

| File | Line | Description |
|------|------|-------------|
| `CMakeLists.txt` | 364, 484 | WIP compiler flags; pkg-config granularity |
| `CompilerWarnings.cmake` | 60 | `-Wno-dangling-reference` workaround for fmt bug |
| `core/CMakeLists.txt` | 66 | Configure file installation |
| `GnuRadioBlockLibMacros.cmake.in` | 201, 249 | pkg-config file installation (x2) |

---

## 6. Test Files

| File | Line | Description |
|------|------|-------------|
| `qa_UncertainValue.cpp` | 737, 761, 793, 801 | Uncertainty propagation checks incomplete |
| `qa_SchedulerMessages.cpp` | 274, 301, 552 | `sendAndWaitMessage` port; FIXME edge connection |
| `qa_Scheduler.cpp` | 76, 1253 | Destructor exception; nested graph flatten test |
| `qa_Settings.cpp` | 535, 807, 916 | Warning → PMT error; incomplete assertion; Emscripten |
| `qa_Tags.cpp` | 39 | InputSpan/OutputSpan reference semantics |
| `qa_Block.cpp` | 747 | Incomplete chunk processing policy |
| `qa_DynamicBlock.cpp` | 44, 45 | Port count change after connection; Emscripten |
| `qa_buffer.cpp` | 752 | libc++ ranges replacement |
| `qa_Messages.cpp` | 580 | Workaround (unspecified) |
| `qa_PmtTypeHelpers.cpp` | 261, 280 | Type conversion tests for unimplemented paths |
| `CollectionTestBlocks.hpp` | 72 | `std::array<T,N>` collection vs tuple handling |
| `bm_Scheduler.cpp` | 17, 295 | Scheduler re-init issue; broken connect workaround |

---

## 7. Platform-Specific Issues

### 7.1 Emscripten
Multiple workarounds exist for Emscripten/WebAssembly targets:
- Deadlock workaround in `Block.hpp:2314`
- Incomplete `std::atomic` in `Scheduler.hpp:512` and `annotated.hpp:512`
- ASAN problem with vector reallocation in `YamlPmt.hpp:1135`
- Test failures in `qa_FilterTool.cpp:269`, `qa_DynamicBlock.cpp:45`, `qa_Settings.cpp:916`

### 7.2 Clang/libc++
- `ImChart.hpp:263,285` — CI workaround for missing feature
- `Sequence.hpp:85` — libc++ `atomic<shared_ptr>` support
- `BlockTraits.hpp:320` — Clang bug workaround for `std::span`

### 7.3 Windows
- `thread_pool.hpp:702` — Thread affinity not implemented for Windows/Apple
- `PluginLoader.cpp:18` — Path separator (`;` vs `:`) not handled

---

## 8. Priority Recommendations

### High Priority (Architectural Gaps)
1. **Error handling strategy** for port connections (`Port.hpp:700, 981`) — blocks downstream design decisions
2. **SIMD multi-output** (`Graph.hpp:1278`) — limits performance optimization paths
3. **Port lifetime management** (`Port.hpp:1235`) — potential memory safety issue
4. **Dangerous const_cast** (`Port.hpp:1227`) — safety concern flagged for follow-up PR
5. **DataSink "TODO Important!"** (`DataSink.hpp:636`) — unspecified but marked urgent

### Medium Priority (Missing Features)
6. GPU GEMM/GEMV (`TensorMath.hpp`) — needed for GPU compute path
7. Polynomial interpolation (`SchmittTrigger.hpp`) — blocks advanced trigger modes
8. Port collection export (`Graph.hpp:169`) — limits graph composition
9. Cross-graph connections (`qa_BlockModel.cpp:277`) — limits modular graph design
10. Windows/Apple thread affinity (`thread_pool.hpp:702`) — platform support gap

### Low Priority (Cleanup & Modernization)
11. Port API renames (`portName` → `name`, `portInfo` → `type`)
12. Dead code removal (`Block.hpp:953, 2037`, `Graph.cpp:7`)
13. C++23/26 STL migration items in `thread_pool.hpp`
14. pkg-config granularity (`CMakeLists.txt:484`)
15. `PythonBlock.hpp` settings API migration

---

## 9. Statistics by Module

| Module | TODOs | FIXMEs | Workarounds | WIP | Not Impl. |
|--------|-------|--------|-------------|-----|-----------|
| `core/include/` | ~85 | 4 | 8 | 2 | 5 |
| `core/test/` | ~25 | 1 | 1 | 0 | 0 |
| `core/src/` | 3 | 0 | 0 | 0 | 0 |
| `core/benchmarks/` | 3 | 0 | 1 | 0 | 2 |
| `blocks/` | ~30 | 0 | 2 | 1 | 0 |
| `algorithm/` | ~10 | 0 | 2 | 2 | 3 |
| `meta/` | ~5 | 1 | 0 | 0 | 0 |
| `build/cmake` | ~8 | 0 | 1 | 1 | 0 |

---

*This report is a point-in-time snapshot. Counts are approximate where grep results were truncated at 200 lines.*
