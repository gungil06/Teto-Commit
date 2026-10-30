*Project title: TetoCommit: Parallel Git Commit Analyzer*

*Project description*

TetoCommit is Python application that analyzes Git commit history and demonstrates parallel processing using virtual workers. It is inspired by the following project:
https://github.com/MikolajSzafraniecUPDS/commits-analyzer

Users select a local Git repository, choose an execution mode, and run an analysis. The application extracts commit records, divides them into batches, processes those batches, and combines the results into a dashboard.

The project has two objectives: summarize repository activity and investigate how worker count affects processing performance. We will compare sequential execution with process pools containing 1, 2, and 4 workers.

Each virtual worker is a separate Python process on one computer. The application therefore demonstrates a single-machine simulation of distributed processing, consistent with the assignment’s allowance for multiprocessing.

*Relationship to the reference project*

The reference project provides inspiration for commit extraction, analysis, and dashboard presentation. Our midterm version will use a smaller architecture focused on the assignment requirements.

| Area | Midterm design |
|---|---|
| Repository input | An already-cloned local Git repository |
| Extraction | Python calls Git commands to collect commit metadata |
| Storage | In-memory records and JSON/CSV files |
| Analysis | Author counts, activity by date, and message analysis |
| Execution | Sequential processing and configurable worker processes |
| Interface | A small Dash dashboard |
| Evaluation | Time, throughput, CPU, memory, and output correctness |

PostgreSQL, Docker services, automatic cloning, and PDF generation are outside the initial scope. If source code is adapted, we will identify the original project and preserve applicable attribution and license notices.

*Functional requirements*

| Feature | Expected behavior |
|---|---|
| Repository validation | Report an understandable error for an invalid repository path |
| Commit extraction | Read commit hash, author, timestamp, and message |
| Author analysis | Count commits for each author |
| Activity analysis | Count commits by date |
| Message analysis | Count words and classify `feat:`, `fix:`, `docs:`, and other prefixes |
| Execution controls | Select sequential mode or a worker count and batch size |
| Worker reporting | Show worker PID, completed batches, and processing duration |
| Performance comparison | Compare execution time, throughput, CPU, and memory |
| Export | Download analysis summaries and benchmark results as CSV |

These statistics describe repository activity. Commit counts and message labels will not be presented as direct measures of developer quality.

*Application design*

The dashboard will have three views:

| View | Contents |
|---|---|
| *Analysis* | Repository path, run controls, total commits, author chart, activity chart, and message statistics |
| *Workers* | Configured workers, completed batches, worker PIDs, durations, and reported errors |
| *Benchmarks* | Experiment controls, performance comparison charts, correctness status, and CSV download |

The first version can display worker summaries after execution. Continuous live updates are an optional improvement.

*System architecture*

The system separates input collection, processing, measurement, and presentation.

| Component | Responsibility |
|---|---|
| Dashboard | Collect user settings and display results |
| Collector | Extract commit history and normalize it into plain records |
| Coordinator | Create batches, start execution, and track completed tasks |
| Worker processes | Analyze independent batches and return partial summaries |
| Aggregator | Combine partial counts into the final analysis |
| Metrics collector | Measure execution and sample resource usage |
| Exporter | Write analysis and benchmark results to CSV |

The processing flow is:

1. Validate the repository and extract a fixed set of commit records.
2. Assign each record to exactly one batch.
3. Process batches sequentially or submit them to a process pool.
4. Return a partial summary with each batch ID.
5. Merge successful summaries without counting a batch twice.
6. Display results and save performance measurements.

Workers receive plain Python records rather than live Git objects or dashboard components. The coordinator controls execution; workers do not update the interface directly.

*Shared data structures*

| Structure | Main fields |
|---|---|
| Commit record | `sha`, `author`, `timestamp`, `message` |
| Task | `job_id`, `batch_id`, `records` |
| Worker result | `batch_id`, `worker_pid`, `summary`, `elapsed_seconds`, `error` |
| Analysis summary | `total_commits`, `by_author`, `by_date`, `by_category`, `word_counts` |
| Benchmark result | `mode`, `workers`, `batch_size`, `repetition`, `elapsed_seconds`, `throughput`, resource measurements |

Before development, we will agree on branch selection, merge-commit handling, author identity, and the timezone used for daily activity counts.

*Proposed folder structure*

```text
tetocommit/
    app.py
    requirements.txt
    README.md
    LICENSE
    .gitignore

    core/
        __init__.py
        models.py
        collector.py
        analyzer.py
        workers.py
        metrics.py
        benchmark.py
        exporter.py

    dashboard/
        __init__.py
        layout.py
        callbacks.py
        charts.py

    tests/
        test_collector.py
        test_analyzer.py
        test_workers.py

    data/
        sample_commits.json

    results/
        .gitkeep

    docs/
        project-plan.md
        architecture.md
        experiment-results.md
        report.md
```

`app.py` launches the interface. Business logic stays in `core/`, while dashboard layout and callbacks stay in `dashboard/`. Analyzed repositories remain outside the application folder.

*Group assignments*

| Member | Responsibility | Deliverables |
|---|---|---|
| *Gilbert* | Coordination and integration | Repository setup, shared models, module interfaces, integrated application, architecture explanation |
| *Louis* | Commit extraction | `collector.py`, repository validation, Git extraction, normalized sample data |
| *Kindy* | Analysis and aggregation | `analyzer.py`, author/date counts, message analysis, summary merging |
| *Edbert* | Worker execution | `workers.py`, batching, process pool, task identifiers, worker results and errors |
| *Ethan* | Measurement and experiments | `metrics.py`, `benchmark.py`, repeatable experiments, performance analysis |
| *Jessica* | Dashboard | `app.py`, dashboard layout, callbacks, analysis charts, worker and benchmark views |
| *Jhonson* | Verification, export, and documentation | `exporter.py`, integration checks, setup guide, report assembly, demonstration script |

Each member will test and document their own work. Jhonson coordinates verification; Gilbert coordinates integration. Everyone will review another member’s contribution and present their own component.

*Experiment plan*

Use the same extracted records for every configuration. Compare direct sequential execution with 1-, 2-, and 4-worker process pools, repeating each configuration five times.

Report median execution time and variability, commits processed per second, speedup, and sampled CPU/memory usage. Include process startup, batch transfer, and aggregation in the processing benchmark; measure repository extraction separately.

Every parallel summary must match the sequential summary. Basic counting may be slower in parallel because of overhead, especially for small histories. We will report the observed behavior without adding artificial delays to improve benchmark results.

*Development milestones*

| Milestone | Completion requirement |
|---|---|
| 1. Sequential prototype | Extract a repository and produce correct summaries |
| 2. Parallel execution | Process batches with workers and match sequential output |
| 3. Dashboard | Display analysis, worker summaries, and exports |
| 4. Experiments | Record repeatable performance comparisons |
| 5. Submission | Complete application, report, and presentation |

*Final deliverables*

The submission will include the Python application, GitHub source repository, installation instructions, correctness checks, benchmark CSV files, a short report, and a demonstration explaining workload distribution and measured performance.
