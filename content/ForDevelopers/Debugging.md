---
title: Debugging
---

Compared to interpreted execution engines, compiling engines come with many advantages but also some challenges.
Especially debugging can become a challenge, as one not only needs to debug the engine code, but also the generated code.
When debugging generated code typically two main questions arise:

1. Where exactly is the generated code wrong?
2. Where does this wrong part come from?

Possible solutions to these problems have been discussed before for debugging [Hyper](https://ieeexplore.ieee.org/document/8667737) and [Umbra](https://dl.acm.org/doi/abs/10.1145/3395032.3395321).

## General Approach in LingoDB
To solve these challenges in LingoDB, we use a combination of location tracking, snapshotting, and alternative execution engines.

### Location Tracking in MLIR
In MLIR, every operation is associated with a *Location*, that must be provided during operation creation.
While it is possible to provide a *Unknown Location*, it should be avoided.
When parsing a MLIR file, MLIR automatically annotates the parsed operations with the corresponding file locations.
When new operations are created during a pass they are usually annotated with the location of the current operation that is transformed or lowered.
**All passes in LingoDB ensure that correct locations are set afterwards.**

### Snapshotting
MLIR already comes with a `LocationSnapshotPass` that takes an operation (e.g. a MLIR Module) and writes it to disk, including the annotated locations.
Then, this file is now read back in, now annotating the locations *according to the location inside this newly written file*.

If enabled, LingoDB performs multiple location snapshots on multiple abstraction levels (in the current working directory):
1. `input.mlir`: initial MLIR module that is e.g., produced from an SQL query
2. `snapshot-0.mlir`: location snapshot after query optimization
3. `snapshot-1.mlir`: location snapshot after lowering high-level operators to sub-operators
4. `snapshot-2.mlir`: location snapshot after lowering sub-operators to imperative operations
5. `snapshot-3.mlir`: location snapshot after lowering high-level imperative operations
6. `snapshot-4.mlir`: final location snapshot of low-level IR (e.g., llvm dialect)

Using this snapshot files, we can track the origin of any operation, by recursively following the following steps
1. get the origin location of the current operation by looking in the appropriate snapshot file
2. find the origin operation by going to this location

For example, if the debugger reports a problem (e.g. SEGFAULT) at `snapshot-4.mlir:1234`,
* We first go to line `1234` of `snapshot-4.mlir` for the problematic operation and look at the corresponding location data (e.g., `snapshot-3.mlir:42`)
* Next, we visit line `42` of `snapshot-3.mlir` to find the corresponding higher-level operation and look at the corresponding location data (e.g., `snapshot-2.mlir:13`)
* Next, we visit line `13` of `snapshot-2.mlir` to find the corresponding higher-level operation and look at the corresponding location data (e.g., `snapshot-1.mlir:5`)
* Finally, we visit line `5` of `snapshot-1.mlir` to find the 'problematic' sub-operator.

### Compiler Backends for Debugging
In addition to location tracking and snapshotting, LingoDB implements two special compiler backends for debugging.

#### LLVM-Debug Backend
Instead of using the standard LLVM backend, another LLVM-based backend can be used that adds debug information and performs no optimizations.
This backend is selected by setting the environment variable `LINGODB_EXECUTION_MODE=DEBUGGING`.
During the execution, standard debuggers like `gdb` will then point to the corresponding operation in `snapshot-4.mlir`.
This enables basic tracking of problematic operations, but advanced debugging will remain difficult.

#### C++-Backend
For more advanced debugging, a *C++-Backend* can be used by setting `LINGODB_EXECUTION_MODE=C`.
This backend directly translates a fixed set of low-level generic MLIR operations to C++ statements and functions that are written to a file called `mlir-c-module.cpp`.
Next, LingoDB automatically invokes `clang++` (must be installed!) with `-O0` and `-g` to compile this C++ file into a shared library with debug informations.
This shared library is then loaded with `dlopen` and the main function is called.
Thus, the generated code can be debugged as any usual C++ program.
To help with tracking an error to higher-level MLIR operations, each C++ statement is preceeded with a comment containing the original operation and it's location.

#### When to choose which backend?
In most cases, choosing the C++-Backend is the better option, as it makes debugging much more user-friendly.
However, there are two cases when the LLVM-Debug backend should be used:
1. The C++-Backend may fail if unsupported MLIR operations are used for which no translation to C++ code is defined
2. The behavior of the C++-Backend deviates from the previously expected behavior (e.g., in the case of a bug in the lowering to llvm).

