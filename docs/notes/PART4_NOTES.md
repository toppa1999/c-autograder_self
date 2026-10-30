# Part 4 of 4: web app, second problem, limitations, Docker, CI, scripts

Unzip into the repo root over Parts 1-3. Nothing from Parts 1-3 is changed.
`problems/add/reference.c`, `spec.md` and `tests/` are included: **if you already have your own, keep yours**
(skip those files; `python scripts/make_problem_tests.py` only writes tests that are missing).

    pip install -r requirements.txt            # after merging requirements-part4.txt (adds Django)
    python scripts/make_problem_tests.py       # only if problems/*/tests is missing
    python -m pytest -q tests                  # Parts 1-4
    python manage.py migrate && python manage.py seed_demo
    python manage.py runserver                 # http://127.0.0.1:8000  teacher/teacher123, student1/student123

## New files
| File | Purpose | Suggested owner |
|---|---|---|
| docs/usefulness_limitations.json | 4 usefulness + 4 limitation examples (each limitation names a wrong/unreliable verdict; examples were measured). Loaded by the coordinator into every report | Ayushi |
| problems/add/{reference.c,spec.md,tests/}, problems/sum_array/*, rules/sum_array_full.json, samples/sum_array_*.c | second problem graded with a different rules file and no code change | Nabam |
| scripts/make_problem_tests.py, run_samples.py, repro_check.py | test generation, sample outputs, N-run reproducibility check | Nabam |
| autograder_web/, manage.py | Django project (SQLite, settings, urls) | Aarushi |
| grader/models.py, jobs.py, services.py, api.py, migrations/ | ORM models, background worker, upload/rate-limit logic, JSON API | Aarushi |
| grader/views.py, forms.py, urls.py, templates/ | upload, status polling, report page, class table + CSV, rule editor, bulk ZIP | Ayushi |
| grader/management/commands/ | seed_demo, grade_pending, export_corpus | Aarushi |
| Dockerfile, .dockerignore, .github/workflows/ci.yml | pinned image, CI with tests + 10-run reproducibility | Aarushi |
| tests/test_part4.py | 18 tests: docs, problems, second rules file, Django views/API/permissions | Nabam |

## Web app behaviour
* Student code never runs in the Django process: the worker thread starts `grade.py` as a child process
  (set `GRADER_USE_DOCKER=1` to run it in `docker run --network none` instead; build the image first:
  `docker build -t c-autograder .`).
* Uploads: `.c` only, UTF-8 text, <= 100 KB, stored under a generated name; max 3 queued/running runs per student
  and 10 uploads per minute (settings.py). CSRF and login are on.
* Rule editor validates JSON, weights and analyzer names (all errors at once) and saves a NEW version;
  each submission keeps the rule version it was graded with. Instructor re-run creates a new GradingRun row.
* Copy flags (similarity / llm_copy / online_copy) are shown to instructors only.
* API: `POST /api/submissions/` (problem, file) and `GET /api/runs/<id>/`, HTTP Basic auth.
* After a server restart: `python manage.py grade_pending`. After a deadline: `python manage.py export_corpus add`.
* Set `DJANGO_SECRET_KEY`, `DJANGO_DEBUG=0` and `DJANGO_ALLOWED_HOSTS` outside development. Runs on Linux/macOS/WSL/Docker.

## Final-week commands
    python scripts/run_samples.py                       # reports/final/*.txt|json + SUMMARY.txt
    python scripts/repro_check.py --runs 10             # byte-identical reports, all nine metrics
    python -m agent.counter_tests samples/add_basic.c --rules rules/add_full.json --count 30

## Known gaps (state them honestly in the report)
* Valgrind/cppcheck were not available where Part 4 was built; the complexity metric used its static fallback
  there. Run the full suite on your machine with them installed.
* Phase 1 job runner is a single in-process thread (queue is lost on restart; use grade_pending). Celery/Redis not done.
* No PDF export of the student report (txt/json download only).
