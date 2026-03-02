name: algo-alchemist
version: "1.2.0"
author: Kilocode Team
description: Transforms raw algorithms into elegant, optimal solutions through performance analysis, complexity reduction, and code refinement.
main: bin/algo-alchemist
entrypoint: algo-alchemist
tags:
  - optimization
  - algorithms
  - performance
  - refactoring
  - complexity
language: python
dependencies:
  - python>=3.8
  - mypy
  - pytest
  - timeit
  - astunparse
  - networkx
  - numpy
optional_dependencies:
  - py-spy
  - memory_profiler
  - pyperf
  - tqdm
environment:
  ALGO_CACHE_DIR: "~/.cache/algo-alchemist"
  ALGO_TIMEOUT: "30"
  ALGO_VERBOSE: "0"
  ALGO_MAX_ITERATIONS: "10"
---
# Algo Alchemist

Transforms raw algorithms into elegant, optimal solutions through systematic performance analysis, complexity reduction, and code refinement.

## Purpose

Real use cases:
- Improve time/space complexity of existing code (O(n²) → O(n log n) or better)
- Transform naive implementations into efficient, production-ready solutions
- Optimize critical performance bottlenecks in latency-sensitive code
- Refactor spaghetti code into clean, maintainable algorithms
- Convert between algorithmic paradigms (recursive ↔ iterative, DP ↔ greedy)
- Apply memoization, dynamic programming, and divide-and-conquer techniques
- Optimize data structures for specific access patterns (lookup-heavy vs write-heavy)
- Reduce memory footprint through in-place operations or streaming algorithms

## Scope

Supported languages: Python 3.8+, JavaScript (Node.js 14+), TypeScript 4.0+

### Commands

`algo-alchemist analyze <file> [--function <name>] [--profile]`
Analyzes algorithm complexity, identifies bottlenecks, optionally profiles runtime. Outputs complexity analysis, hotspot report, and improvement suggestions.

`algo-alchemist optimize <file> --target-complexity <O(n)|O(n log n)|O(1)> [--preserve-api] [--max-iterations <int>]`
Applies automated transformations to achieve target complexity. Iteratively tests and validates improvements.

`algo-alchemist transform <file> --paradigm <iterative|recursive|dynamic|greedy> [--function <name>]`
Converts algorithm to specified paradigm while preserving semantics.

`algo-alchemist memoize <file> --function <name> [--strategy <lru|custom|dp>]`
Adds memoization/caching to recursive or repetitive computations.

`algo-alchemist datastruct <file> --optimize <lookup|insertion|iteration> --constraints <memory|speed>`
Recommends and applies data structure optimizations (list → dict, array → set, etc.)

`algo-alchemist profile <file> --input <test-data> [--runs <int>] [--memory]`
Runs microbenchmarks with provided input data. Generates performance profiles.

`algo-alchemist suggest <file> [--function <name>]`
Returns actionable optimization suggestions without applying changes.

`algo-alchemist apply <file> --suggestion-id <id> [--dry-run]`
Applies a specific optimization suggestion from the suggest output.

`algo-alchemist verify <file> [--tests <path>] [--complexity-check]`
Validates correctness against tests and verifies complexity improvements.

`algo-alchemist undo <file> [--steps <int>]`
Reverts the last N transformations (default: 1). Maintains transformation history.

`algo-alchemist history <file>`
Shows complete transformation log with diffs and complexity changes.

`algo-alchemist benchmark <file> --baseline <commit|branch> --input <test-data>`
Compares performance against a baseline. Requires py-spy or pyperf for detailed profiling.

### Real Example Prompts

```bash
# Analyze nested loops in find_duplicates.py
algo-alchemist analyze find_duplicates.py --function find_pairs

# Optimize to O(n) using hash map
algo-alchemist optimize find_duplicates.py --target-complexity O(n) --preserve-api

# Convert recursive DFS to iterative stack-based
algo-alchemist transform maze_solver.py --paradigm iterative --function dfs

# Add memoization to expensive recursive calculation
algo-alchemist memoize fibonacci.py --function fib --strategy lru

# Optimize list-based membership checks to set-based
algo-alchemist datastruct filter_items.py --optimize lookup --constraints speed

# Profile with real dataset
algo-alchemist analyze sort_compare.py --profile --input tests/data/large_array.json

# Get suggestions without applying
algo-alchemist suggest network_routing.py --function shortest_path

# Apply suggestion #3 only
algo-alchemist apply network_routing.py --suggestion-id 3 --dry-run

# Verify correctness after optimization
algo-alchemist verify optimized_sort.py --tests tests/test_sorts.py --complexity-check
```

## Detailed Work Process

### 1. Initial Analysis
- Parse source file using AST
- Identify target function(s) or whole-file analysis
- Compute current Big O time and space complexity
- Detect anti-patterns: nested loops, repeated computations, linear searches in loops, unnecessary copies
- Generate baseline metrics if profiling enabled

### 2. Profiling (Optional)
- Execute with provided input data or generate synthetic test cases
- Measure wall time, CPU usage, memory allocations
- Identify hotspots (functions consuming >50% of runtime)
- Profile memory allocations with memory_profiler if available

### 3. Optimization Planning
- Enumerate applicable transformations based on detected patterns
- Estimate complexity reduction for each transformation
- Prioritize by impact and risk (correctness preservation)
- Generate transformation plan (up to `ALGO_MAX_ITERATIONS` steps)

### 4. Transformation Application
- Apply transformations sequentially with validation between steps
- Each transformation is recorded in history file `.algo-history.json`
- Code is reformatted to match existing style (black for Python, prettier for JS/TS)
- Type checking with mypy/tsc to ensure no type regressions

### 5. Verification
- Run existing test suite (pytest/jest) to ensure semantic equivalence
- Re-analyze complexity to confirm improvement
- Run benchmark comparison if baseline available
- Check for readability regressions (line count, cyclomatic complexity)

### 6. Iteration
- If target complexity not met, repeat from step 3 with remaining transformations
- If tests fail, automatically undo last transformation and try alternative
- Abort after `ALGO_MAX_ITERATIONS` if no successful transformations

### 7. Final Report
- Summary of transformations applied
- Before/after complexity comparison
- Performance improvement metrics (if profiled)
- Diff of code changes
- List of any manual review items

## Golden Rules

1. **Correctness First**: Never sacrifice functional correctness for performance. All tests must pass.
2. **API Stability**: Public function signatures and return types must remain unchanged unless `--preserve-api` is false.
3. **Readability Preserved**: Optimized code must not increase cyclomatic complexity. Favor clear over clever.
4. **Profile-Driven**: Do not apply transformations without profiling data when `--profile` is available. Avoid premature optimization.
5. **Transformations Atomic**: Each change should be isolated and reversible. Never mix multiple optimizations in one commit.
6. **Dependencies Explicit**: Track added dependencies (e.g., functools.lru_cache is stdlib, but external libs must be noted).
7. **Memory Awareness**: Consider space-time tradeoffs explicitly. Document if optimization increases memory usage.
8. **Edge Cases Validated**: Ensure transformed algorithm handles empty inputs, large inputs, boundary conditions identically.

## Examples

### Example 1: O(n²) to O(n) with Hash Map

**Input file `find_duplicates.py` (before):**
```python
def find_duplicates(arr):
  duplicates = []
  for i in range(len(arr)):
    for j in range(i + 1, len(arr)):
      if arr[i] == arr[j] and arr[i] not in duplicates:
        duplicates.append(arr[i])
  return duplicates
```

**Command:**
`algo-alchemist optimize find_duplicates.py --target-complexity O(n)`

**Output (after):**
```python
def find_duplicates(arr):
  seen = set()
  duplicates = set()
  for item in arr:
    if item in seen:
      duplicates.add(item)
    else:
      seen.add(item)
  return list(duplicates)
```

**Verification:**
```
$ algo-alchemist verify find_duplicates.py
✓ All tests passed (5/5)
✓ Complexity: O(n²) → O(n)
✓ Memory: +O(n) for hash set (acceptable tradeoff)
```

### Example 2: Recursive Fibonacci → Memoized

**Input `fibonacci.py`:**
```python
def fib(n):
  if n <= 1:
    return n
  return fib(n-1) + fib(n-2)
```

**Command:**
`algo-alchemist memoize fibonacci.py --function fib --strategy lru`

**Output:**
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
  if n <= 1:
    return n
  return fib(n-1) + fib(n-2)
```

**Profile comparison:**
```
Before: fib(35) = 9227465 (15.2s)
After:  fib(35) = 9227465 (0.004s)
Speedup: 3800×
```

### Example 3: Recursive DFS → Iterative

**Input `maze_solver.py` (recursive DFS):**
```python
def solve(maze, start, end):
  visited = set()
  def dfs(pos):
    if pos == end:
      return True
    visited.add(pos)
    for neighbor in maze.neighbors(pos):
      if neighbor not in visited and dfs(neighbor):
        return True
    return False
  return dfs(start)
```

**Command:**
`algo-alchemist transform maze_solver.py --paradigm iterative --function solve`

**Output (iterative with explicit stack):**
```python
def solve(maze, start, end):
  visited = set()
  stack = [start]
  while stack:
    pos = stack.pop()
    if pos == end:
      return True
    if pos not in visited:
      visited.add(pos)
      for neighbor in maze.neighbors(pos):
        if neighbor not in visited:
          stack.append(neighbor)
  return False
```

## Rollback Commands

- `algo-alchemist undo <file> [--steps N]` — Revert last N transformations. Example: `algo-alchemist undo optimized.py --steps 2`
- `algo-alchemist restore <file> --to <commit-hash>` — Restore to specific Git commit or history snapshot. Example: `algo-alchemist restore algo.py --to abc1234`
- `algo-alchemist history <file>` — Show transformation log with timestamps, diffs, and complexity changes.
- `algo-alchemist diff <file> --version <N>` — Compare current version to version N in history. Example: `algo-alchemist diff optimized.py --version 3`

**Important**: History is stored in `.algo-history.json` in the same directory. Backup this file if you need persistent rollback across sessions.

## Verification Steps

After any optimization run:

1. **Correctness**: `algo-alchemist verify <file> --tests <path>` — Run full test suite (required).
2. **Complexity**: `algo-alchemist verify <file> --complexity-check` — Ensure Big O improved as expected.
3. **Benchmark**: `algo-alchemist benchmark <file> --baseline <original-commit> --input <dataset>` — Compare runtime performance.
4. **Lint**: `algo-alchemist lint <file>` — Ensure code style compliance (black/flake8 or eslint/prettier).
5. **Type Check**: `mypy <file>` or `tsc <file>` — No type errors introduced.
6. **Manual Review**: Inspect generated diff for readability and maintainability.

## Troubleshooting

**Problem**: Optimization breaks existing tests.
- **Solution**: `algo-alchemist undo <file>` to revert. Manually adjust or use `--preserve-api` flag to enforce interface stability.

**Problem**: Complexity not improved as expected.
- **Solution**: The algorithm may have inherent lower bound. Use `algo-alchemist suggest` to see alternative paradigms. Consider if problem constraints allow better complexity.

**Problem**: Memory usage increased dramatically.
- **Solution**: Review data structure choices. `algo-alchemist datastruct <file> --optimize memory` to explore space-efficient alternatives. Trade time for space if needed.

**Problem**: Stack overflow after iterative conversion.
- **Solution**: Manual review of stack management. Ensure proper termination condition and that all paths are covered.

**Problem**: `algo-alchemist` command not found.
- **Solution**: Ensure `~/.local/bin` in PATH or use `python -m algo_alchemist`. Install dependencies: `pip install -r requirements.txt`.

**Problem**: Profiling hangs or times out.
- **Solution**: Set `ALGO_TIMEOUT` to higher value (e.g., `export ALGO_TIMEOUT=120`). Check input data size. Use `--runs 1` for single run.

**Problem**: Type checking fails after optimization.
- **Solution**: Some transformations (like adding lru_cache) need import. `algo-alchemist` should add imports automatically; if not, add `from functools import lru_cache` manually.

## Dependencies

**Required:**
- Python 3.8+ with ast, astunparse, mypy, pytest, timeit, networkx
- For JavaScript/TypeScript: Node.js 14+, eslint, jest, typescript

**Optional (for profiling):**
- py-spy (sampling profiler, works on production code)
- memory_profiler (line-by-line memory usage)
- pyperf (rigorous benchmarking)

## Configuration

Create `~/.config/algo-alchemist/config.yaml`:
```yaml
cache_dir: "~/.cache/algo-alchemist"
default_timeout: 30
max_iterations: 10
auto_undo_on_failure: true
languages:
  python:
    style: black
    typecheck: mypy
    test_framework: pytest
  javascript:
    style: prettier
    typecheck: eslint
    test_framework: jest
```

## Notes

- Transformation history is stored in `.algo-history.json` (gitignore this file if sharing).
- Always commit before running `algo-alchemist optimize` to have a clean baseline.
- Use `--dry-run` with `apply` to preview changes.
- For large codebases, optimize one function at a time (`--function <name>`) to reduce risk.
- The tool respects existing imports and dependencies; it does not add external libraries without explicit user approval.

---