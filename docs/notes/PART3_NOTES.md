# Part 3 of 4: remaining analyzers, counter-tests, profiling

Unzip into the repo root (over Parts 1-2). Nothing from Parts 1-2 is changed.
It does NOT contain problems/add/reference.c: keep your own (any file named
reference*.c in problems/add works).

    sudo apt install valgrind            # needed for the complexity metric (cppcheck from Part 2 too)
    pip install -r requirements-part3.txt
    python -m pytest -q tests            # 86 pass (valgrind tests skip if valgrind is missing)
    python grade.py samples/add_basic.c --rules rules/add_full.json    # all nine metrics now run

## New files
| File | Purpose | Suggested owner |
|---|---|---|
| analyzers/edge_case_runner.py, agent/counter_tests.py, problems/add/edge_cases.json | logical metric: corner cases + seeded random cases, differential test vs reference, input minimiser, explanations; also a CLI | Nabam |
| analyzers/complexity_estimator.py | complexity: callgrind instruction counts, fit a + b*n^k, class + score; static loop-depth fallback | Aarushi |
| agent/profiling.py | callgrind / valgrind memcheck / perf helpers (+ CLI) | Nabam |
| analyzers/llm_review.py | readability: deterministic style features + Ollama review (temp 0, seed, cache) | Ayushi |
| analyzers/similarity_flag.py, agent/similarity.py, corpus/ | originality: normalised winnowing, flags for peers / LLM / online copies | Ayushi |
| tests/test_part3.py | 45 tests incl. fake Ollama server and a 3-run nine-metric determinism test | Nabam |

## Handy commands
    python -m agent.counter_tests samples/add_basic.c --rules rules/add_full.json --count 30 --save problems/add/counter
    python -m agent.profiling samples/add_leak.c --input "7 5"          # add --perf if perf works on your machine

## Optional rule params (existing rules files keep working)
* logical: `cases` may also list `crlf`, `no_trailing_newline`, `leading_zeros`, `plus_sign`, `random`; `random_count`; `minimise_drop_tokens`.
* complexity: `generator` = scalar | single_n | array (how stdin is built from the size n), `method` = auto | callgrind | static, `profile`: true adds a valgrind profile to the evidence.
  Use sizes large enough that the work dominates start-up (about 150k instructions): e.g. [1000, 2000, 4000, 8000] for O(n) problems. For O(1) the default [1, 10, 100] is fine.
* readability: `backend: "none"` skips the LLM, `require_llm: true` makes a missing Ollama an error, `host`.
* originality: `penalize: false` reports flags without lowering the score; `min_tokens`.

## Behaviour you should know (good material for Part 4's limitations section)
* Without Ollama, readability uses the deterministic part only, so a grade can differ between a machine with and without Ollama. The evidence records which was used. Use `backend: "none"` for fully machine-independent grades.
* Tiny programs look alike: the add solutions are ~45 tokens, so independent students can exceed 0.85 similarity. Raise `min_tokens` (or use harder problems) to avoid false flags; a flag is only ever a request for human review.
* Complexity by static loop depth cannot tell O(log n) from O(n) (binary search) or a constant-bound loop from a real one; the report marks it `confidence: low`.
* Callgrind counts the program as compiled at -O2: an optimiser may turn a loop into a formula, and the estimate then describes what actually runs.

## For Part 4
* The upload job runner should keep student files OUT of `corpus/` until grading finishes, or rely on the path check (the submission's own path is ignored).
* `docs/usefulness_limitations.json` (loaded by the coordinator) is still to come in Part 4.
