# Part 2 of 4: the five analyzers + samples

New: analyzers/{io_tests,compiler_warnings,static_analysis,function_split,sanitizer_run}.py,
agent/testset.py, samples/*.c (six submissions), tests/{helpers.py,test_analyzers.py,__init__.py},
requirements-part2.txt.
CHANGED (overwrite Part 1's copy): agent/sandbox.py - adds an optional `nproc` argument to run().

System tools: gcc with sanitizer runtimes (libasan/libubsan), cppcheck. lizard is a pip package.
Without cppcheck, structure still works (weights become 0.5/0.5).

| Analyzer | Rules metric | Suggested owner |
|---|---|---|
| compiler_warnings, static_analysis, function_split | syntax, structure, modularity | Ayushi |
| io_tests, sanitizer_run, testset, samples, tests | correctness, memory_safety | Nabam |

Test expectations: expected outputs are the exact mathematical sum (long long),
so 2147483647 1 expects 2147483648. tests/helpers.py builds its own 10-case
problem in a temp folder, so the tests do not depend on your problems/add folder.
