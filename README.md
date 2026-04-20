# SGCLI

Databricks Serverless GPU CLI (`sgcli`) wheel packages and test jobs.

## Quick Start

### 1. Install SGCLI

```bash
pip install sgcli_wheel/databricks_serverless_gpu_cli-0.0.7+fix-py3-none-any.whl --force-reinstall
```

### 2. Authenticate

```bash
databricks auth login --host https://your-workspace.cloud.databricks.com --profile YOUR_PROFILE
```

### 3. Update `repo_path`

Each test folder has a `train_workload.yaml` (or `train.yaml`). Update `repo_path` to point to your local clone:

```yaml
code_source:
  type: snapshot
  snapshot:
    repo_path: /path/to/your/clone/of/this/repo
```

### 4. Run a single test

```bash
cd SGC_hello_world
sgcli run -f train_workload.yaml -p YOUR_PROFILE --watch
```

### 5. Run all tests

```bash
./run_tests.sh -p YOUR_PROFILE
```

Add `--watch` to stream logs (output from parallel jobs will interleave).

## Test Jobs

| Folder | GPU | GPUs | Description |
|--------|-----|------|-------------|
| `SGC_hello_world/` | H100 | 8 | torchrun hello world — verifies CUDA, GPU access, multi-process |
| `SGC_hello_world_error/` | H100 | 8 | Intentional init error — tests SGC error handling and run status |
| `SGC_hello_world_a10/` | A10 | 2 | torchrun hello world on A10 |
| `SGC_hello_world_error_a10/` | A10 | 2 | Intentional init error on A10 |

## Install (other versions)

```bash
pip install sgcli_wheel/<wheel_file>.whl --force-reinstall
```

## Releases

| Version | File | Status | Notes |
|---------|------|--------|-------|
| 0.0.7+fix | `databricks_serverless_gpu_cli-0.0.7+fix-py3-none-any.whl` | **latest** | Fix macOS snapshot root path bug |
| 0.0.7 | `databricks_serverless_gpu_cli-0.0.7-py3-none-any.whl` | stable | Performance, bug fixes, glob cancel, priority scheduling |
| 0.0.6+fix | `databricks_serverless_gpu_cli-0.0.6+fix-py3-none-any.whl` | hotfix | Hotfix on top of 0.0.6 (pre-hotfix), same wheel as the stable 0.0.6 |
| 0.0.6 | `databricks_serverless_gpu_cli-0.0.6-py3-none-any.whl` | stable | Agent-friendly JSON, runtime variable interpolation, telemetry |
| 0.0.6 (pre-hotfix) | `databricks_serverless_gpu_cli-0.0.6-py3-none-any-before-hotfix.whl` | archived | Snapshot before hotfix |
| 0.0.5 | `databricks_serverless_gpu_cli-0.0.5-py3-none-any.whl` | stable | Bug fixes, dry run, color output |
| 0.0.4 | `databricks_serverless_gpu_cli-0.0.4-py3-none-any.whl` | stable | Snapshot fixes, permission granting |
| 0.0.3 | `databricks_serverless_gpu_cli-0.0.3-py3-none-any.whl` | stable | Uncommitted changes support, non-git folders |

## Changelog

### 0.0.7+fix

- [Bug-fix] Fix macOS snapshot root path detection. On macOS, `tar` includes AppleDouble metadata files (`._*`) with extended attributes. These sorted before the real top-level directory, causing `tar -tzf ... | head -1` to return `._project` instead of `project`, breaking the `$HOME` symlink and making code unreachable at runtime. Fix: pass `tar_directory_name` deterministically from the client instead of parsing the tarball at runtime, and exclude `._*` files from tarballs with `--exclude=._*`.

### 0.0.7

- Fix override bug which would silently drop second override
- Protect command field from variable interpolation so bash `${VAR}` works
- [Bug-fix] Fix hyperlinks in status table, show MLflow run name, suppress download bar spam, improve pool validation error message
- [Bug-fix] Fix run submission exiting without setting the MLflow run name
- Add glob-style batch cancellation: `sgcli cancel --match 'pattern' [-y]`
- [CNXT-1853] Add priority field for pool workload scheduling
- [CNXT-1939] Create versioned composite snapshot key
- Introduce `CODE_SOURCE_PATH` environment variable so users can locate uploaded code
- Get client telemetry working with sgcli
- [Bug-fix] Fix symlink failure when extracting git archive snapshots
- Add `--retry` flag to `get logs` to view logs from a specific retry attempt
- Update `get runs` to show all runs by default; add `--active`, `--all-users`, and `--user` flags
- Honor `.gitignore` when snapshotting non-git directories, preventing venv and other ignored files from being uploaded
- [CNXT-1638] Fix `--json` mode to suppress human-readable output and add caching to remove redundant auth calls
- Improve CLI startup performance by ~3.5x via lazy imports of heavy dependencies (databricks-sdk, mlflow)
- [Bug-fix] Ensure MLflow sidecar cleanup runs on any exit (including failures) via EXIT trap to prevent GPU hang
- [Bug-fix] Fix stalled jobs caused by MLflow system metrics sidecar not terminating after user code completes
- Decrease timeout for MLflow API call to update the MLflow run name
- Suppress noisy Apple extended-header warnings during tarball extraction
- Support git worktrees and allow `include_paths` for non-git directories

### 0.0.6

- [CNXT-1924] Make sgcli more agent friendly; introduce `--json` and `monitor` command
- [UX] Make `-p` and `-v` flags global flags that work before or after the subcommand
- [CNXT-1891] Support runtime variable interpolation in `env_variables`
- [Bug-fix] Fix sandbox script for DCS to not assume any package installs
- [Bug-fix] Fix use of `remote_head`
- [Deprecation] Removed `no-image-upload` flag
- [CNXT-1638] Support v5 client and deprecate v3
- [CNXT-1887] Add email support through `--email` flag
- [CNXT-1877] Add client telemetry for SGCLI

### 0.0.5

- [Bug-fix] Remove print_error from env_secrets
- [CNXT-1859] Remove python script from sgcli
- [Bug-fix] Fix uncommitted snapshot code path directories
- [Bug-fix] Fix call to experiment creation
- [CNXT-1778] Add color for sgcli
- [CNXT-1844] Remove git clone from cli src
- [CNXT-1727] Fix bug in setting permissions in config
- [CNXT-1832] Improve performance of `sgcli get runs` command
- [CNXT-1784] Provide yaml pointers from sgcli tool
- [CNXT-1638] Add dry run command
- [CNXT-1828] Add budget policy attribution field

### 0.0.4

- [CNXT-1817] Fix uncommitted changes not being captured in snapshot; allow `include_paths` when changes are outside those paths
- [CNXT-1809] Update log syntax and allow downloading to a specified directory
- [CNXT-1806] Add support for automatic permission granting after job submission with `grant_permissions` field
- [CNXT-1638] Make logs from dependency installation unbuffered

### 0.0.3

- [CNXT-1727] Allow uncommitted changes and simplify git UX
- [CNXT-1759] Add support for non-git folders
- [CNXT-1713] Add support for variable interpretation in local and remote
- [Bug-fix] Fix hardcoded email from databricks.com to actual user email

### 0.0.2

- [CNXT-1727] Support for subfolders in snapshot via git archive; major speed up for large repos
- [CNXT-1716] Remove unique suffix from job run names; experiment corresponds exactly to job run name

### 0.0.1

- [CNXT-1727] Add changelog command
- [CNXT-1706] Fix get runs hyperlinks and add get status hyperlinks
- [CNXT-1713] Add experiment_name validation and reduce log level
- [CNXT-1624] Validate fields and gpu num during workload submission
- [CNXT-1538] Add Streaming Logs capability
