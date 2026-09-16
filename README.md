# Python Project Setup with uv and Jupyter

This guide explains how to create a Python project using [uv](https://docs.astral.sh/uv/) and work with Jupyter notebooks.

The commands below use PowerShell on Windows. The same `uv` commands work in macOS and Linux terminals.

## 1. Install uv

Install uv with the official PowerShell installer:

```powershell
irm https://astral.sh/uv/install.ps1 | iex
```

Restart your terminal, then verify the installation:

```powershell
uv --version
```

You should see the installed uv version.

## Check for the latest version of uv

To see which version of uv is currently installed, run:

```powershell
uv --version
```

The simplest way to update uv to the latest available version is to use uv's built-in self-update command:

```powershell
uv self update
```

After the update completes, verify the installed version again:

```powershell
uv --version
```

If the version is newer than the one shown before the update, uv was successfully updated. If it is unchanged, you were already using the latest version or the update could not be completed. Restart your terminal if the command is not recognized after an update.

## 2. Create a project directory

Create a new directory and move into it:

```powershell
mkdir my-python-project
cd my-python-project
```

## 3. Check and install the latest Python version

List the Python versions available through uv:

```powershell
uv python list
```

The list shows the Python versions that uv can install. Choose the newest stable version shown in the list. Preview versions may also appear, so avoid versions marked as alpha, beta, release candidate, or otherwise pre-release unless you specifically need one.

Install the latest stable Python version known to uv:

```powershell
uv python install
```

You can verify the installed Python versions with:

```powershell
uv python list --only-installed
```

The version you choose must be written explicitly when initializing the project. For example, if the newest stable version from `uv python list` is Python 3.13, install and select it with:

```powershell
uv python install 3.13
```

Replace `3.13` with the newest stable version you found in the list. Version numbers change over time, so do not copy this example version blindly.

## 4. Initialize the project with the selected Python version

Initialize the project and tell uv which Python version to use:

```powershell
uv init --python 3.13
```

Replace `3.13` with the newest stable version you selected. This creates project files including:

- `pyproject.toml`: project metadata and dependencies
- `README.md`: project documentation
- `.python-version`: the selected Python version
- `src/my_python_standards/`: the importable application package

The selected version is recorded in `.python-version` so the project uses the same Python version consistently. You can confirm it with:

```powershell
uv python pin
```

### Change an existing project to Python 3.14.7

If the project has already been initialized, do not run `uv init` again. Install Python 3.14.7, pin it for the project, recreate the virtual environment, and synchronize the dependencies:

```powershell
uv python install 3.14.7
uv python pin 3.14.7
uv venv --python 3.14.7 --clear
uv sync
```

Check that the project now uses Python 3.14.7:

```powershell
uv run python --version
```

The `.python-version` file should contain `3.14.7`. The `requires-python` value in `pyproject.toml` describes the range of Python versions supported by the project; it does not select the interpreter used by the current virtual environment.

## 5. Create the project environment

Create the virtual environment managed by uv:

```powershell
uv venv
```

The environment is created in the `.venv` directory.

You can activate it in PowerShell with:

```powershell
.\.venv\Scripts\Activate.ps1
```

To deactivate it later:

```powershell
deactivate
```

Activation is optional when using `uv run`, because uv can run commands inside the project environment automatically.

## 6. Add the required packages

Add Jupyter and the IPython kernel as project dependencies:

```powershell
uv add jupyter ipykernel
```

Add common data-science packages when needed:

```powershell
uv add numpy pandas matplotlib
```

Development-only tools, such as pytest and Ruff, can be added with:

```powershell
uv add --dev pytest ruff
```

uv updates `pyproject.toml` and creates or updates `uv.lock`.

## 7. Create a notebook directory

Keep notebooks in a dedicated directory:

```powershell
mkdir notebooks
```

Start JupyterLab from the project root:

```powershell
uv run jupyter lab
```

In JupyterLab, open the `notebooks` directory, select **File > New > Notebook**, choose the `Python (my-python-standards)` kernel, and save the notebook as:

```text
data_structures.ipynb
```

`data_structures.ipynb` uses the Python convention of separating words with underscores. A filename such as `data-structures.ipynb` also works, but do not use dashes in Python module, package, class, or variable names because they are interpreted as minus signs in Python code.

A typical project layout is:

```text
my-python-project/
├── .venv/
├── notebooks/
│   └── exploration.ipynb
├── src/
│   └── my_python_project/
│       ├── __init__.py
│       └── ...application code...
├── .python-version
├── pyproject.toml
├── README.md
└── uv.lock
```

Keep application code inside `src/<package-name>/`. Keep `notebooks/` at the project root so notebooks are separate from the installable Python package. A root-level `main.py` is appropriate for a simple script-style project, but this project uses the `src` package layout instead.

Do not commit `.venv`, because it is a local generated environment. Add it to `.gitignore`:

```gitignore
.venv/
__pycache__/
.ipynb_checkpoints/
```

## 8. Register the project as a Jupyter kernel

Register the project environment so Jupyter can use it:

```powershell
uv run python -m ipykernel install --user --name my-python-standards --display-name "Python (my-python-standards)"
```

Run this command from the project root. In the notebook kernel list, select the entry displayed as `Python (my-python-standards)`.

The `--name` value is the internal kernel name. The `--display-name` value is what appears in Jupyter and VS Code.

## 9. Start JupyterLab

Launch JupyterLab from the project directory:

```powershell
uv run jupyter lab
```

JupyterLab opens in your browser. Select the `Python (my-python-standards)` kernel when creating or opening a notebook.

To use the classic Jupyter Notebook interface instead:

```powershell
uv run jupyter notebook
```

## 10. Create and run a notebook

The first notebook in this project is `notebooks/data_structures.ipynb`. In JupyterLab:

1. Open the `notebooks` directory.
2. Select **File > New > Notebook**.
3. Select the `Python (my-python-standards)` kernel.
4. Save the notebook as `data_structures.ipynb`.
5. Run a test cell:

```python
import sys

print(sys.executable)
print("The project environment is working.")
```

The printed executable should point to the project's `.venv` directory.

## 11. Run Python files with uv

Run the project using its configured console script without manually activating the environment:

```powershell
uv run my-python-project
```

Replace `my-python-project` with the script name configured under `[project.scripts]` in `pyproject.toml`. In this project, that name is `my-python-standards`.

Run a Python command in the project environment:

```powershell
uv run python -c "import pandas; print(pandas.__version__)"
```

## 12. Install dependencies from the lockfile

When setting up the project on another machine, sync the environment from `uv.lock`:

```powershell
uv sync
```

For a reproducible setup that refuses to change the lockfile:

```powershell
uv sync --locked
```

Use `--locked` in continuous integration when the lockfile should already be up to date.

## 13. Update dependencies

See available outdated packages:

```powershell
uv outdated
```

Update dependencies and the lockfile:

```powershell
uv lock --upgrade
uv sync
```

Update one package:

```powershell
uv add "pandas>=2.2"
```

## 14. Run tests and quality checks

Run tests:

```powershell
uv run pytest
```

Run Ruff checks:

```powershell
uv run ruff check .
```

Format the project with Ruff:

```powershell
uv run ruff format .
```

## 15. Use the project in VS Code

Install these VS Code extensions:

- Python (`ms-python.python`)
- Jupyter (`ms-toolsai.jupyter`)

Then:

1. Open the project folder in VS Code.
2. Open a notebook in `notebooks`.
3. Select the kernel from the top-right kernel picker.
4. Choose the environment in `.venv` or the registered `Python (my-python-standards)` kernel.

If the environment does not appear, run:

```powershell
uv sync
uv run python -m ipykernel install --user --name my-python-standards --display-name "Python (my-python-standards)"
```

## 16. Recommended Git workflow

Initialize Git and make the first commit:

```powershell
git init
git add .
git commit -m "Initialize Python project with uv and Jupyter"
```

Commit these files:

- `pyproject.toml`
- `uv.lock`
- `.python-version`
- Source code
- Useful notebooks
- Documentation

Do not commit these generated or local files:

- `.venv/`
- `__pycache__/`
- `.ipynb_checkpoints/`
- Secrets or local configuration files

## Common commands

| Purpose | Command |
| --- | --- |
| Install dependencies | `uv sync` |
| Add a package | `uv add package-name` |
| Add a development package | `uv add --dev package-name` |
| Remove a package | `uv remove package-name` |
| Run a command in the environment | `uv run command` |
| Run JupyterLab | `uv run jupyter lab` |
| Run tests | `uv run pytest` |
| List installed packages | `uv pip list` |
| Show project help | `uv --help` |

## Troubleshooting

### `uv` is not recognized

Restart the terminal after installing uv. If it still is not found, verify that uv's installation directory is on your `PATH`.

### PowerShell blocks environment activation

If PowerShell does not allow activation scripts, either use `uv run` without activation or update the execution policy for your user account:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### The notebook uses the wrong Python environment

Select the registered project kernel from the notebook kernel picker, or reinstall it:

```powershell
uv run python -m ipykernel install --user --name my-python-standards --display-name "Python (my-python-standards)"
```

### A package is missing from the notebook

Add it to the project and synchronize the environment:

```powershell
uv add package-name
uv sync
```

Restart the notebook kernel after installing the package.
