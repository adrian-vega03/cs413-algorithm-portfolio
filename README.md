# Pioneer Delivery Algorithm Portfolio - Part I

C++17 implementation of the CS 413 greedy-algorithm assignment. The program reads
delivery intervals from a file, reports Tasks A-C, and demonstrates the Task D
counterexample. It uses only the C++ standard library.

## Compile and run

From the project root, using GCC/MinGW (PowerShell on Windows):

```powershell
New-Item -ItemType Directory -Force build
g++ -std=c++17 -O2 -Wall -Wextra -Wpedantic -Iinclude src/main.cpp src/scheduler.cpp -o build/pioneer.exe
.\build\pioneer.exe data/deliveries.txt
```

Run without an argument to use `data/deliveries.txt`; use `--help` for usage.
Quote paths containing spaces. On Linux/macOS, use `mkdir -p build`, omit `.exe`
from output names, and run `./build/pioneer data/deliveries.txt`.

An optional CMake build is also supplied (requires CMake 3.16+ and a C++17 compiler):

```text
cmake -S . -B build
cmake --build build --config Release
ctest --test-dir build -C Release --output-on-failure
```

CMake places the executable in `build/` for a single-configuration generator or
`build/Release/` for a multi-configuration generator. Run from the project root or
pass an explicit input path.

## Input file

Each nonblank line contains `ID START FINISH`, separated by whitespace. IDs must
be unique and have no whitespace; times are signed 64-bit integers with
`START < FINISH`. Blank lines and `#` comments are supported. There is no initial
count or header row. Negative times are accepted for Tasks A-C. Intervals are
half-open: `[start, finish)`, so a driver may start a delivery when the previous
one finishes. An empty file is valid and produces zero for all three counts.

The supplied `data/deliveries.txt` contains all seven example requests from the
assignment. Malformed rows, duplicate IDs, invalid intervals, out-of-range times,
and unreadable files produce an error and a nonzero exit status.

## Program organization

- `include/scheduler.hpp`: delivery representation and algorithm interfaces.
- `src/scheduler.cpp`: input parsing, greedy algorithms, and Task D demonstration.
- `src/main.cpp`: command-line handling and readable output.
- `tests/test_scheduler.cpp`: required cases, validity checks, and independent
  optimality checks on small random inputs.
- `data/deliveries.txt`: sample input.
- `CMakeLists.txt`: optional portable build and test configuration.

Algorithms receive or create their own copies, leaving the original input intact.
Ties are broken by time and ID (and driver index in the heap) for reproducible
output. Different optimal schedules are allowed by the assignment.

## Task A: maximum deliveries for one driver

**Greedy choice:** Sort by increasing finish time and select each delivery whose
start is at least the finish of the last selected delivery.

**Why optimal:** Let `g` be an earliest-finishing request, and let `o` be the first
request in an optimal schedule. Since `finish(g) <= finish(o)`, replacing `o` with
`g` leaves every subsequent request feasible without reducing the number selected.
There is therefore an optimal schedule starting with `g`. Apply the same exchange
argument to the remaining requests starting at or after `finish(g)`; induction
shows that every greedy selection preserves an optimal solution.

**Complexity:** Sorting costs O(n log n), followed by an O(n) scan. Total time is
O(n log n), with O(n) storage for the input copy and selected deliveries.

The output includes the selected IDs, their start and finish times, and their count.

## Task B: minimum drivers for all deliveries

Sort requests by increasing start time. A priority queue (min-heap) stores one
`(last finish time, driver index)` pair for each existing driver. The top gives
the earliest available driver. If its finish time is at most the next request's
start, remove that pair and reuse the driver; otherwise create a driver. Insert
the driver's updated finish time after assigning the request.

**Why optimal:** Suppose a new driver is required when `d` drivers already exist.
The minimum finish time exceeds the current start, so every existing driver's
last request is still active at that start. Because requests are processed by
start time, all those requests have already started. Including the current
request, there are `d + 1` simultaneous deliveries, which require at least
`d + 1` drivers in any schedule. Thus each driver creation meets a necessary
lower bound. Reuse never causes an overlap, so the final assignment is feasible
and uses the minimum possible number of drivers.

**Complexity:** Sorting and at most one removal and one insertion per request
cost O(n log n) time. The heap has O(d) entries; including input and output
schedules, total storage is O(n). The output lists every driver's deliveries
with times, followed by the minimum driver count.

## Task C: minimum rejections

Task C is the complement of Task A. If the maximum compatible subset has `k`
requests, reject exactly `n - k`. Rejecting fewer would retain more than `k`
compatible requests, contradicting maximal cardinality. Conversely, retaining
Task A's subset achieves `n - k` rejections. The implementation calls Task A and
subtracts its result size: O(n log n) time and O(n) storage.

## Task D: counterexample to decreasing-profit greedy

These are flexible, unit-time jobs, separate from the fixed intervals in the
input file. Their ready times are nonnegative and their profits are positive
and distinct:

| Delivery | Ready time R | Profit P |
| --- | ---: | ---: |
| A | 0 | 6 |
| B | 0 | 5 |
| C | 1 | 4 |

Greedy G considers A, B, C in that order and places each in the first available
integer slot at or after its ready time:

| Delivery | Greedy slot | Greedy delay | Greedy value | Better slot | Better delay | Better value |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| A | 0 | 0 | 6 | 0 | 0 | 6 |
| B | 1 | 1 | 5/2 | 2 | 2 | 5/3 |
| C | 2 | 1 | 4/2 | 1 | 0 | 4 |

`V(G) = 6 + 5/2 + 2 = 21/2 = 10.5`.

The better schedule S places A at 0, C at 1, and B at 2:
`V(S) = 6 + 4 + 5/3 = 35/3 = 11.666666...`.

Both schedules use distinct integer slots and respect every ready time.
`V(S) - V(G) = 7/6 > 0`, so G is not always optimal. Delaying B one more slot
loses `5/6`, while moving C to its ready time gains `2`. One feasible schedule
with a strictly larger value is enough to disprove the strategy; no claim that
S is globally optimal is needed. The executable also prints this example.

## Tests and required results

Build and run the automated suite directly with GCC/MinGW:

```powershell
g++ -std=c++17 -O2 -Wall -Wextra -Wpedantic -Iinclude tests/test_scheduler.cpp src/scheduler.cpp -o build/scheduler_tests.exe
.\build\scheduler_tests.exe
```

Validation with GCC/MinGW 15.2.0 passed all 18 required cases, all 500 random
inputs, and the additional validation cases, with no compiler warnings under
the flags above. The tables below show the verified results; the suite checks
these counts and the feasibility of the resulting schedules. The sample-file
run and command-line help and missing-file handling were also checked.
CMake configuration is supplied but was not executed because CMake was not
available in the environment.

| Task A case | Intervals in input order | Maximum selected |
| --- | --- | ---: |
| 1 | none | 0 |
| 2 | (1,3) | 1 |
| 3 | (1,3), (3,5), (5,7) | 3 |
| 4 | (1,5), (2,6), (3,7) | 1 |
| 5 | (5,7), (1,3), (3,5), (2,4) | 3 |
| 6 | (1,4), (2,4), (4,6) | 2 |

| Task B case | Intervals in input order | Minimum drivers |
| --- | --- | ---: |
| 1 | none | 0 |
| 2 | (1,3) | 1 |
| 3 | (1,3), (3,5), (5,7) | 1 |
| 4 | (1,5), (2,6), (3,7) | 3 |
| 5 | (1,4), (2,5), (4,7), (5,8) | 2 |
| 6 | (5,8), (1,4), (2,5), (4,6), (6,9) | 2 |

| Task C case | Intervals in input order | Minimum rejected |
| --- | --- | ---: |
| 1 | none | 0 |
| 2 | (1,3) | 0 |
| 3 | (1,3), (3,5), (5,7) | 0 |
| 4 | (1,5), (2,6), (3,7) | 2 |
| 5 | (5,7), (1,3), (3,5), (2,4) | 1 |
| 6 | (1,2), (2,3), (3,4), (1,3) | 1 |

The suite also verifies 500 deterministic random inputs (0-9 requests each).
For each, exhaustive enumeration of every subset independently checks Task A
and therefore Task C; a separate event sweep computes peak simultaneous demand
to check Task B. It verifies each output delivery matches the input, every
request is assigned exactly once in Task B, and no driver has overlapping jobs.
Additional cases cover identical intervals with different IDs, negative times,
64-bit integer boundaries, empty input, comments, malformed input, and Task D's
displayed values. Test failures return a nonzero exit status, even in Release builds.

For the supplied sample file, the expected output counts are **3 selected,
3 drivers, and 4 rejected**. The program selects D02 (2,5), D05 (8,11),
D06 (11,14).
