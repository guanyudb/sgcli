# SGCLI Wheel Releases

Databricks Serverless GPU CLI (`sgcli`) wheel packages.

## Install

Recommended — `uv tool install` (isolated env, adds `sgcli` to PATH):

```bash
# Install uv if needed
curl -LsSf https://astral.sh/uv/install.sh | sh

uv tool install --python 3.12 sgcli_wheel/databricks_serverless_gpu_cli-0.1.0-py3-none-any.whl
sgcli --version
```

`uv tool install` works cleanly because 0.1.0 declares `pydantic` and `packaging` as explicit wheel dependencies — earlier versions relied on transitive resolution and could break in isolated envs.

Alternative (`pip` into an active venv):

```bash
pip install --force-reinstall sgcli_wheel/databricks_serverless_gpu_cli-0.1.0-py3-none-any.whl
```

If you previously installed an older `sgcli` in a venv and want to switch to the `uv tool install` copy, run `pip uninstall sgcli` first — venv copies take precedence when the venv is active.

## Releases

| Version | File | Status | Notes |
|---------|------|--------|-------|
| 0.1.0 | `databricks_serverless_gpu_cli-0.1.0-py3-none-any.whl` | **latest** | DCS auto-credential discovery, ai-training service routing, CODE_SOURCE_PATH semantics change, MLflow REST client, `uv tool install` support |
| 0.0.7+fix | `databricks_serverless_gpu_cli-0.0.7+fix-py3-none-any.whl` | stable | Fix macOS snapshot root path bug |
| 0.0.7 | `databricks_serverless_gpu_cli-0.0.7-py3-none-any.whl` | stable | Performance, bug fixes, glob cancel, priority scheduling |
| 0.0.6+fix | `databricks_serverless_gpu_cli-0.0.6+fix-py3-none-any.whl` | hotfix | Hotfix on top of 0.0.6 (pre-hotfix), same wheel as the stable 0.0.6 |
| 0.0.6 | `databricks_serverless_gpu_cli-0.0.6-py3-none-any.whl` | stable | Agent-friendly JSON, runtime variable interpolation, telemetry |
| 0.0.6 (pre-hotfix) | `databricks_serverless_gpu_cli-0.0.6-py3-none-any-before-hotfix.whl` | archived | Snapshot before hotfix |
| 0.0.5 | `databricks_serverless_gpu_cli-0.0.5-py3-none-any.whl` | stable | Bug fixes, dry run, color output |
| 0.0.4 | `databricks_serverless_gpu_cli-0.0.4-py3-none-any.whl` | stable | Snapshot fixes, permission granting |
| 0.0.3 | `databricks_serverless_gpu_cli-0.0.3-py3-none-any.whl` | stable | Uncommitted changes support, non-git folders |

## Changelog

### 0.1.0

**Major changes / new behavior:**

- **`CODE_SOURCE_PATH` semantics changed.** Now points to the *extracted* `code_source` directory (e.g. `/databricks/code_source/<dir_name>`). Use `cd $CODE_SOURCE_PATH` instead of the old `cd $CODE_SOURCE/<dir>`.
- **Relative `repo_path` resolution** is now relative to the YAML file's directory, not the current working directory (`sgcli run -f` invocation cwd).
- **`set -euo pipefail`** is now wrapped around user scripts by default for fail-fast behavior.
- **DABs-compatible `permissions` field** added; `grant_permissions` deprecated (with backward compatibility).
- **OmegaConf variable interpolation is disabled unconditionally.** Removed `no_interpolation` field and `--no-interpolation` flag. Literal `${VAR}` strings in YAML are preserved as-is; bash `$VAR` shell expansion at runtime is unaffected.
- **Embedded `agent_skills` removed.** Use the standalone "sgcli" Claude Code plugin instead.
- **On-demand workloads now route through the `ai-training` service** via `HardwareAcceleratorType` enum names (`GPU_1xA10`, `GPU_8xH100`, etc.). Pool workloads continue to use the legacy `gpu_type` strings on the MAPI path.

**DCS (`register image`):**

- Auto-discover Docker credentials from `~/.docker/config.json` (including `credHelpers` / `credsStore`) when `--scope/--key` and `--interactive-authenticate` are absent. A single `docker login` makes subsequent registrations need no flags.
- `--interactive-authenticate` now validates Databricks workspace auth *before* prompting for Docker username/PAT, so unauthenticated users get a clear error immediately instead of typing creds and hitting 401.
- **Security fix:** default Docker credential scope is now per-Databricks-user (`docker-credentials-<user>`) and creator-only, preventing cross-user PAT reads.
- Fix `register image --tag-policy latest` ignoring the policy when no `--scope/--key` were passed.
- [CNXT-2030] Normalize short-form Docker Hub image names (e.g. `ubuntu:latest` → `docker.io/library/ubuntu:latest`).

**MLflow:**

- [CNXT-2084] Always emit MLflow system-metrics sidecar + run-name update in the launch script, including for custom Docker images, so multi-node jobs report per-node metrics.
- Replace MLflow Python SDK with a direct Databricks MLflow REST client; drop the `mlflow` wheel dependency.
- Forward `credentials-for-read` response headers to the pre-signed URL fetch in `mlflow_rest_client.download_artifact`. Fixes log/artifact downloads on Azure-backed workspaces where SAS URIs require headers like `x-ms-version`.

**Bug fixes:**

- [ES-1857723] Fix broken `$HOME/<repo>` symlink for macOS users whose snapshot repos have extended attributes. Pass tar directory name from Python instead of parsing tarball contents; exclude AppleDouble (`._*`) files from snapshot tarballs.
- Fix `sgcli --json run --watch` returning immediately with `status=PENDING` instead of blocking until terminal. The JSON path now emits a `SUBMITTED` event with `run_id`, streams `STATUS` / `LOG` / `ALERT` JSONL events through the watch, then emits a final envelope reflecting the actual terminal state (`SUCCESS` / `FAILED` / `TIMEDOUT` / `CANCELED`).
- Fix `--watch` and `get logs` hanging on quick timeout/error; show logs when a job ends before reaching `RUNNING`.
- Fix `sgcli monitor`: surface every loss key and match the real per-GPU keys for auto-detected `gpu_utilization` / `gpu_memory`.
- [CNXT-2024] Create default secret scope with open permissions; surface a clear error when the workspace scope quota is exceeded.

**Internal / packaging:**

- Declare `pydantic` and `packaging` as wheel dependencies (previously relied on transitive resolution; required for `uv tool install` to produce a working sgcli).
- Drop unused wheel deps (`cloudpickle`, `psutil`, `pynvml`); delete vestigial `cli/requirements.txt`.
- Pass user `env_variables` through `gen_ai_compute_task.env_vars`.
- Pass `tar_workspace_path` and `requirements_yaml_path` through `gen_ai_compute_task` so the server-side entry script can extract the workspace tarball and install pip dependencies.
- [internal] Add hidden `--via` flag and git-aware `code_source` telemetry signals (`via_flag`, `code_source_uses_git`).

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
