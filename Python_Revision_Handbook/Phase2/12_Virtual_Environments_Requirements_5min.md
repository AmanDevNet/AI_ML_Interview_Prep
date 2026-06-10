# Chapter 12: Virtual Environments & Requirements — "Dependency Isolation"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | A **Virtual Environment** is a self-contained directory tree that contains a specific Python installation plus additional packages. **Requirements files** contain dependency declarations. |
| **Why It Exists** | Without isolation, installing package versions globally would break other apps that need different versions of the same package (dependency hell). |
| **Where It Is Used** | Every python project! Local development, staging, production deployment, CI/CD pipelines. |
| **Real-World Analogy** | Having separate isolated laboratory cleanrooms for different experiments, so chemicals from one experiment do not contaminate another. |

---

## Think of It This Way

Imagine you have two Python applications on your computer:
1. **App A:** A legacy script that requires `pandas==0.24`.
2. **App B:** A new machine learning model that requires `pandas==2.2`.

If you install packages globally, running `pip install pandas==2.2` will upgrade the pandas package system-wide. This will break **App A** instantly because of API deprecations in pandas.

**Virtual Environments** solve this. 
You create one virtual environment (`env_a`) for App A, and another (`env_b`) for App B. 
Each has its own private copies of the Python executable and third-party packages. Neither environment knows the other exists.

---

## Step 1: Virtual Environments (`venv`)

Python has a built-in module called `venv` to manage virtual environments.

### 1. Creating a Virtual Environment
Run this in your terminal inside your project directory:
```bash
# Windows
python -m venv myenv

# macOS / Linux
python3 -m venv myenv
```
This creates a folder named `myenv/` containing a copy of Python, `pip`, and a private site-packages directory.

### 2. Activating the Environment
Activating tells your shell to use this environment's Python and pip.
```bash
# Windows (PowerShell)
myenv\Scripts\Activate.ps1

# Windows (Command Prompt)
myenv\Scripts\activate.bat

# macOS / Linux
source myenv/bin/activate
```
**How it works under the hood:** Activation alters your terminal session's `PATH` environment variable. It places `myenv/Scripts` (or `myenv/bin`) at the very front of your `PATH`, so when you type `python` or `pip`, the OS runs the isolated copy instead of the global one.

### 3. Deactivating
To exit the environment:
```bash
deactivate
```

---

## Step 2: Requirements Files (`requirements.txt`)

To share your project with others, you shouldn't share the virtual environment folder (it is large, machine-specific, and contains compiled code). Instead, you share a list of the packages your project needs.

### 1. Generating a requirements file
```bash
pip freeze > requirements.txt
```
This writes all installed packages and their exact versions to `requirements.txt`.
Example `requirements.txt` content:
```text
numpy==1.26.4
pandas==2.2.1
requests==2.31.0
```

### 2. Installing from a requirements file
When another developer downloads your code, they create their own virtual environment and run:
```bash
pip install -r requirements.txt
```
This installs all listed packages in one command.

---

## Dependency Specification Rules

When writing a requirements file manually, you can specify versions in different ways:

| Specifier | Example | Meaning |
|---|---|---|
| **Exact** | `requests==2.31.0` | Installs exactly version `2.31.0`. Highly recommended for production stability. |
| **Minimum** | `requests>=2.30.0` | Installs version `2.30.0` or any newer version. |
| **Compatible** | `requests~=2.31.0` | Installs any `2.31.x` version, but not `2.32.0` or higher (compatible release). |
| **Exclusion** | `requests!=2.30.0` | Any version except `2.30.0`. |

---

## Code/Terminal Examples

Here is a common terminal workflow for setting up a project:

### Creating, Activating, and Installing
```powershell
# 1. Navigate to your project folder
# cd d:\PYTHON\MyProject

# 2. Create environment
python -m venv .venv

# 3. Activate
.venv\Scripts\Activate.ps1

# 4. Upgrade pip first
python -m pip install --upgrade pip

# 5. Install libraries
pip install requests flask

# 6. Check what was installed
pip list

# 7. Generate requirements
pip freeze > requirements.txt
```

---

## Common Mistakes & Edge Cases

### 1. Committing the Virtual Environment folder to Git
Beginners often check their `venv/` or `.venv/` folder into version control. This is a bad practice. The folder contains thousands of files specific to their OS and Python architecture, which will fail or corrupt on other machines.
* **Fix:** Add `.venv/` or `venv/` to your `.gitignore` file.

### 2. "pip install" outside of an active environment
If you forget to activate your virtual environment, running `pip install` will write packages to your global system directory, cluttering your global python installation and potentially requiring administrative permissions.
* **Mnemonic / Tip:** Always check your terminal prompt. An active environment displays its name in parentheses at the start of the line, e.g., `(.venv) PS C:\Users...`.

### 3. Mixing `pip freeze` with transitive dependencies
`pip freeze` lists *everything* currently installed, including packages you didn't install directly but were installed as dependencies of other packages (transitive dependencies). This can make `requirements.txt` cluttered and hard to maintain.
* **Alternative:** Modern Python projects use tools like `pip-tools`, `Poetry`, `Pipenv`, or `uv` to separate direct dependencies (e.g., `flask`) from pinned transitive dependencies.

---

## Interview Questions (Top 5)

**Q1: What does activating a virtual environment actually do under the hood?**
> Activation is just a shell script that prepends the virtual environment's bin/script directory to the shell's `PATH` environment variable. When you type `python` or `pip`, the shell searches `PATH` sequentially and runs the binaries inside the virtual environment directory instead of the system-wide Python. It also sets the `VIRTUAL_ENV` environment variable.

**Q2: Why shouldn't you commit your virtual environment folder (`venv/` or `.venv/`) to git?**
> A virtual environment folder is highly platform-dependent (containing OS-specific paths and compiled C extensions). If created on Windows, it will not run on macOS or Linux. It is also large and contains thousands of external dependency files. The correct approach is to list dependencies in a metadata file (like `requirements.txt` or `pyproject.toml`) and install them on the target machine.

**Q3: What is the difference between `pip freeze` and `pip list`?**
> `pip list` displays a human-readable list of all installed packages and their versions. `pip freeze` outputs the package names and versions formatted in `package==version` format, which is the exact syntax expected by `requirements.txt`. It also excludes utility packages like `pip`, `setuptools`, and `wheel` by default.

**Q4: What does the specification `requests~=2.31.0` mean in a `requirements.txt` file?**
> It is a compatible release specifier. It is equivalent to `>= 2.31.0` and `< 2.32.0`. It allows upgrading to bugfix/patch releases (like `2.31.1`, `2.31.2`) but locks major and minor versions to prevent breaking API changes from being installed.

**Q5: What are Pipenv, Poetry, and `uv`, and why are they used instead of standard `venv` + `requirements.txt`?**
> They are modern package managers. Standard `requirements.txt` mixes direct dependencies with sub-dependencies and does not guarantee deterministic builds. Tools like Poetry, Pipenv, and `uv` use a lockfile (`poetry.lock` or `uv.lock`) which records exact cryptographic hashes and versions of all dependencies, ensuring environment replication across developers and production environments.

---

*← [Previous: Chapter 11 (Logging)](11_Logging.md) | [Next: Chapter 13 (Context Managers) →](13_Context_Managers.md)*
