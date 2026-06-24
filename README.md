# Run DV Flow

**GitHub Action to run [DV Flow Manager](https://github.com/dv-flow/dv-flow-mgr)
(`dfm`) tasks in CI and publish a diagnostics report.**

This action runs one or more `dfm` tasks, collects the per-task logs and status
into a report bundle (`dfm run --report`), uploads it as a workflow artifact,
and renders a PASS/FAIL summary into the job summary.

It does **not** install DV Flow. Installation and dependency resolution are the
job of [`fvutils/ivpm-setup`](https://github.com/fvutils/ivpm-setup), which
populates `./packages` (including the `packages/python` venv that provides
`dfm`). This action activates that environment via [direnv](https://direnv.net)
— using the project's `.envrc` exactly as a developer's shell would — and runs
`dfm`.

## Usage

```yaml
jobs:
  flow:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Install dfm + dependencies into ./packages
      - uses: fvutils/ivpm-setup@v1
        with:
          python-version: "3.11"
          dep-set: default-dev

      # Run dfm tasks and publish the report
      - uses: dv-flow/run-dvflow@v1
        with:
          tasks: build sim
```

## Prerequisites

- A prior `fvutils/ivpm-setup` step has populated `./packages` so that
  `packages/python/bin/dfm` exists.
- The project provides an `.envrc` that puts `dfm` on `PATH`, e.g. the standard
  DV Flow pattern:

  ```bash
  PATH_add packages/python/bin
  path_add PYTHONPATH src
  ```

  (or one that sources the ivpm-exported `packages/packages.envrc`).

## Inputs

| Input               | Default          | Description                                                        |
|---------------------|------------------|--------------------------------------------------------------------|
| `tasks`             | *(required)*     | Space-separated task name(s) passed to `dfm run`.                  |
| `args`              | `""`             | Extra arguments appended to the `dfm run` command line.           |
| `working-directory` | `.`              | Directory containing the project `.envrc` / flow files.           |
| `report`            | `dvflow-report`  | Directory (relative to `working-directory`) for the report bundle.|
| `ui`                | `log`            | dfm console UI style (`log` recommended for CI).                  |
| `install-direnv`    | `true`           | Install + hook direnv (ivpm-setup does not provide it).           |
| `job-summary`       | `true`           | Append `report.md` to the GitHub job summary.                     |
| `upload-artifact`   | `true`           | Upload the report bundle as a workflow artifact.                  |
| `artifact-name`     | `dvflow-report`  | Name of the uploaded artifact.                                    |
| `fail-on-error`     | `true`           | Fail the step if `dfm` reports one or more failed tasks.          |

## Outputs

| Output       | Description                                          |
|--------------|------------------------------------------------------|
| `report-dir` | Absolute path to the generated report bundle.        |
| `status`     | `dfm` exit status (`0` == all tasks passed).         |

## Report bundle

The `--report` directory produced by `dfm` (schema `dvflow-report/1`) contains:

- `report.json` — structured per-task status and markers.
- `markers.jsonl` — flat stream of error/warning/info markers.
- `report.md` — human-readable PASS/FAIL summary (rendered into the job summary).
- `logs/<task>.log` — per-task log files.

## Examples

See [`examples/`](examples/) for additional usage, including parameter
overrides and multiple report artifacts.

## License

[Apache License 2.0](LICENSE)
