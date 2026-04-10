# SGCLI Wheel Releases

Databricks Serverless GPU CLI (`sgcli`) wheel packages.

## Install

```bash
pip install sgcli_wheel/<wheel_file>.whl --force-reinstall
```

## Releases

| Version | File | Status | Notes |
|---------|------|--------|-------|
| 0.0.7+dev0 | `databricks_serverless_gpu_cli-0.0.7+dev0-py3-none-any.whl` | dev | Development build |
| 0.0.6+fix | `databricks_serverless_gpu_cli-0.0.6+fix-py3-none-any.whl` | hotfix | Hotfix on top of 0.0.6 |
| 0.0.6 | `databricks_serverless_gpu_cli-0.0.6-py3-none-any.whl` | stable | Latest stable release |
| 0.0.6 (pre-hotfix) | `databricks_serverless_gpu_cli-0.0.6-py3-none-any-before-hotfix.whl` | archived | Snapshot before hotfix |
| 0.0.5 | `databricks_serverless_gpu_cli-0.0.5-py3-none-any.whl` | stable | Bug fixes, dry run, color output |
| 0.0.4 | `databricks_serverless_gpu_cli-0.0.4-py3-none-any.whl` | stable | Snapshot fixes, permission granting |
| 0.0.3 | `databricks_serverless_gpu_cli-0.0.3-py3-none-any.whl` | stable | Uncommitted changes support, non-git folders |

## Changelog

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
- [CNXT-1817] Fix uncommitted changes not being captured in snapshot
- [CNXT-1809] Update log syntax and allow for downloading to a specified directory
- [CNXT-1806] Add support for automatic permission granting after job submission
- [CNXT-1638] Make logs from dependency installation unbuffered

### 0.0.3
- [CNXT-1727] Allow uncommitted changes and simplify git UX
- [CNXT-1759] Add support for non-git folders
- [CNXT-1713] Add support for variable interpretation in local and remote
- [Bug-fix] Fix hardcoded email from databricks.com to actual user email
