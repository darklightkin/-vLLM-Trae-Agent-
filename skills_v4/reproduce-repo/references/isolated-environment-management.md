# Isolated Environment Management

## Responsibility

Create a fresh, repo-specific environment by default and protect the user's existing Python, conda, CUDA, and cache state.

## Default Policy

- Always create a new isolated environment for each repo unless the user explicitly asks to reuse one.
- Prefer conda. Use venv only when conda is not available.
- Never install into base, system Python, or an unrelated existing project environment.
- Never automatically fall back to `base`, a previous repo environment, a discovered old environment, or any environment merely because it already has dependencies installed.
- Treat reuse as an explicit user-requested exception, not a low-interaction optimization.
- Never modify system CUDA, system Python, or global site-packages.

## Environment Naming

Use a sanitized repo name plus timestamp:

```text
<repo-name>-YYYYMMDD-HHMMSS
```

Examples:

```text
muq-eval-20260519-151203
gui-kv-20260519-153955
```

Record the environment name in `run_summary.json.environment_name`.

## Python Version Inference

Infer Python version before creating the environment. Check, in order:

1. README or install docs.
2. `environment.yml`, `conda.yaml`, or lockfiles.
3. `pyproject.toml`, `setup.py`, `setup.cfg`, `requirements*.txt`.
4. CI files such as `.github/workflows/*.yml`.
5. Dockerfile only as documentation; do not run Docker.
6. Syntax/import constraints and dependency compatibility.

If no version is stated, choose a conservative modern version compatible with the repo's likely torch/CUDA stack, usually Python 3.10 for current ML repos and Python 3.8 or 3.9 for older torch-era repos.

Do not infer "use the current Python" from an activated shell. The current shell environment is evidence only; it is not permission to reuse it.

## Conda Creation

Use conda first:

```powershell
conda create -n <env-name> python=<version> -y
conda activate <env-name>
python -m pip install --upgrade pip setuptools wheel
```

If `environment.yml` is present, prefer creating the environment from it while preserving the generated environment name:

```powershell
conda env create -n <env-name> -f environment.yml
```

If the file pins an incompatible name, do not reuse that name; override it when supported or document the blocker.

## Venv Fallback

Use venv only if conda is unavailable:

```powershell
python -m venv .venv-<repo-name>-YYYYMMDD-HHMMSS
.\.venv-<repo-name>-YYYYMMDD-HHMMSS\Scripts\Activate.ps1
python -m pip install --upgrade pip setuptools wheel
```

Prefer placing venvs inside the repo workspace or a designated work directory, not in system-level paths.

Venv fallback means creating a fresh venv. It never means selecting an existing `.venv`, old benchmark environment, or system interpreter.

## Reuse Exception

Reuse is allowed only when the user explicitly requests it, for example "reuse env X" or "use the existing conda env named X".

When reuse is requested:

- verify the exact environment name/path;
- record the user request in the history;
- record that isolation was intentionally relaxed;
- never silently switch to a different existing environment after dependency failure.

## CUDA Compatibility

Before installing torch or CUDA packages:

- inspect `nvidia-smi`;
- record driver version and available GPUs;
- infer CUDA target from README, requirements, torch version, install commands, or wheels;
- install torch wheels compatible with the driver and repo requirements;
- treat `flash-attn`, custom CUDA extensions, xformers, vLLM, and apex as high-risk packages requiring pinned compatibility.

Do not modify system CUDA. If a repo requires a different CUDA toolkit, prefer conda packages, compatible wheels, or record a blocker.

## Cache Handling

- Use repo-local or workdir-local caches when practical for HuggingFace, torch, datasets, and pip.
- Record cache paths in the history.
- Do not violently delete global caches.
- Allowed cleanup without asking: failed partial downloads inside the workspace, `__pycache__`, `.pytest_cache`, temporary build folders, and repo-local incomplete files.
- Ask before deleting checkpoints, raw datasets, source files, conversation history, or anything outside the workspace.

## Cleanup Policy

Keep environments until the reproduction is packaged unless the user asks for cleanup. If cleanup is requested, remove only the environment created for the run and leave logs, metrics, and final outputs intact.
