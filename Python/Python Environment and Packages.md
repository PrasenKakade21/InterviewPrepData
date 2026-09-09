# Python Environment & Packages

This section is about **managing Python projects, dependencies, and configuration**. These concepts become especially important when working with Flask, FastAPI, automation scripts, or any real-world Python application.

---

##  `venv`

`venv` is Python's **built-in module for creating virtual environments**.

Create one:

```bash
python -m venv venv
```

This creates a folder named `venv` containing an isolated Python environment.

Activate it:

**Windows:**

```bash
venv\Scripts\activate
```

**macOS/Linux:**

```bash
source venv/bin/activate
```

You will usually see something like:

```text
(venv) C:\projects\kakade>
```

Deactivate:

```bash
deactivate
```

### Why use it?

Without a virtual environment, all projects may share the same installed packages.

For example:

```text
Project A → Django 4
Project B → Django 5
```

A virtual environment allows each project to have its own dependencies.

---

#  Virtual Environments

A **virtual environment** is an isolated Python environment containing:

* Python interpreter
* Installed packages
* Package versions
* Project-specific dependencies

For example:

```text
kakade.dev/
│
├── venv/
├── app.py
└── requirements.txt
```

Packages installed inside `venv` don't affect your global Python installation.

### Typical workflow

```bash
python -m venv venv
```

```bash
venv\Scripts\activate
```

```bash
pip install flask
```

Now Flask is installed **only inside this project's environment**.

### Best practice

Usually don't commit `venv/` to Git.

Add:

```gitignore
venv/
```

---

#  `pip`

`pip` is Python's **package manager**.

It allows you to install, remove, inspect, and manage Python packages.

For example:

```bash
pip install requests
```

You can then use it:

```python
import requests

response = requests.get("https://example.com")
```

### Check pip version

```bash
pip --version
```

A safer way to ensure you're using the pip belonging to your Python installation:

```bash
python -m pip --version
```

---

#  `pip install`

Used to install packages.

```bash
pip install requests
```

Install a specific version:

```bash
pip install requests==2.32.3
```

Install a minimum version:

```bash
pip install "requests>=2.30"
```

Install multiple packages:

```bash
pip install flask fastapi requests
```

### Upgrade a package

```bash
pip install --upgrade requests
```

or:

```bash
pip install -U requests
```

### Install from `requirements.txt`

```bash
pip install -r requirements.txt
```

This is extremely common when setting up an existing project.

---

#  `pip uninstall`

Removes an installed package.

```bash
pip uninstall requests
```

pip will normally ask for confirmation:

```text
Proceed (Y/n)?
```

You can confirm with:

```text
Y
```

### Example

If you installed:

```bash
pip install flask requests
```

and don't need Requests anymore:

```bash
pip uninstall requests
```

Flask remains installed.

---

#  `pip list`

Shows packages currently installed in the environment.

```bash
pip list
```

Example:

```text
Package      Version
------------ -------
Flask        3.1.0
requests     2.32.3
Werkzeug     3.1.3
```

This is useful when you want to quickly see **what is installed right now**.

---

#  `pip freeze`

`pip freeze` outputs installed packages in a format suitable for a requirements file.

```bash
pip freeze
```

Example:

```text
Flask==3.1.0
requests==2.32.3
Werkzeug==3.1.3
```

You can directly save it:

```bash
pip freeze > requirements.txt
```

Now `requirements.txt` contains the installed dependencies.

### `pip list` vs `pip freeze`

| Command      | Purpose                                         |
| ------------ | ----------------------------------------------- |
| `pip list`   | Human-friendly list of packages                 |
| `pip freeze` | Dependency list suitable for `requirements.txt` |

Think:

```text
pip list   → "What's installed?"
pip freeze → "What exact versions should I reproduce?"
```

---

#  `requirements.txt`

A `requirements.txt` file describes the Python packages required by a project.

Example:

```txt
Flask==3.1.0
requests==2.32.3
python-dotenv==1.0.1
```

Another developer can install everything with:

```bash
pip install -r requirements.txt
```

### Typical project

```text
kakade.dev/
│
├── venv/
├── app.py
├── requirements.txt
└── .gitignore
```

### Create it

```bash
pip freeze > requirements.txt
```

### Install from it

```bash
pip install -r requirements.txt
```

### Why is it important?

Imagine your project requires:

```text
Flask 3.1.0
requests 2.32.3
```

Without recording those dependencies, another machine might install newer versions and potentially break the application.

---

#  Environment Variables

Environment variables are **configuration values stored outside your source code**.

For example:

```text
DATABASE_URL=...
API_KEY=...
DEBUG=True
PORT=8000
```

In Python:

```python
import os

api_key = os.getenv("API_KEY")

print(api_key)
```

If the environment variable doesn't exist:

```python
api_key = os.getenv("API_KEY")
```

returns:

```python
None
```

You can provide a default:

```python
port = os.getenv("PORT", "8000")
```

Now if `PORT` isn't defined:

```text
8000
```

is used.

### Why use environment variables?

They allow configuration to change without modifying your code.

For example:

```text
Development → localhost database
Production  → production database
```

Your code can remain the same.

---

#  `.env` Files

A `.env` file is a convenient way to store environment variables during local development.

Example:

```env
API_KEY=abc123
DATABASE_URL=postgresql://localhost/kakade
DEBUG=True
```

You generally **should not commit `.env` to Git**, especially when it contains secrets.

Add this to `.gitignore`:

```gitignore
.env
```

### Important distinction

`.env` is **not automatically loaded by Python**.

You need something such as `python-dotenv` to load it.

---

#  `python-dotenv`

`python-dotenv` loads variables from a `.env` file into Python's environment.

Install:

```bash
pip install python-dotenv
```

`.env`:

```env
API_KEY=abc123
APP_NAME=kakade.dev
DEBUG=True
```

Python:

```python
from dotenv import load_dotenv
import os

load_dotenv()

api_key = os.getenv("API_KEY")
app_name = os.getenv("APP_NAME")

print(app_name)
```

Output:

```text
kakade.dev
```

### Without `load_dotenv()`

This:

```python
os.getenv("API_KEY")
```

will generally **not read `.env` automatically**.

You need:

```python
load_dotenv()
```

first.

### Typical structure

```text
kakade.dev/
│
├── app.py
├── .env
├── .gitignore
└── requirements.txt
```

`.gitignore`:

```gitignore
.env
venv/
```

---

#  Poetry Basics

**Poetry** is a tool for managing Python projects, dependencies, virtual environments, and packaging.

Instead of manually maintaining:

```text
requirements.txt
```

Poetry uses:

```text
pyproject.toml
```

Create a project:

```bash
poetry new kakade.dev
```

Add a dependency:

```bash
poetry add requests
```

Install dependencies:

```bash
poetry install
```

Run Python inside the Poetry environment:

```bash
poetry run python app.py
```

### `pyproject.toml`

Poetry stores project information and dependencies here.

Conceptually:

```toml
[tool.poetry.dependencies]
python = "^3.12"
requests = "^2.32"
flask = "^3.1"
```

### Useful commands

```bash
poetry add flask
```

```bash
poetry remove flask
```

```bash
poetry install
```

```bash
poetry update
```

```bash
poetry show
```

### Poetry vs pip

| pip                                       | Poetry                                        |
| ----------------------------------------- | --------------------------------------------- |
| Package installer                         | Project/dependency manager                    |
| Common and simple                         | More opinionated                              |
| Often uses `requirements.txt`             | Uses `pyproject.toml`                         |
| Doesn't manage the whole project workflow | Handles dependencies, environments, packaging |

You don't necessarily need Poetry for small scripts, but it's useful for larger projects.

---

#  `uv` Basics

**uv** is a very fast Python package and project management tool.

It can handle tasks traditionally done with:

```text
pip
venv
pip-tools
```

and can also manage Python versions and project dependencies.

Install a package:

```bash
uv add requests
```

Create a project:

```bash
uv init kakade.dev
```

Install dependencies:

```bash
uv sync
```

Run a Python program:

```bash
uv run python app.py
```

Create a virtual environment:

```bash
uv venv
```

Install a package directly:

```bash
uv pip install requests
```

### `uv` project structure

A modern uv project might look like:

```text
kakade.dev/
│
├── pyproject.toml
├── uv.lock
└── src/
    └── kakade_dev/
        └── main.py
```

`uv.lock` locks dependency versions so that environments can be reproduced consistently.

### Why is `uv` popular?

It's designed to be **extremely fast** and provides a unified workflow for many Python environment/package-management tasks.

---

# Quick Revision

### Virtual environment

```bash
python -m venv venv
```

### Activate

```bash
venv\Scripts\activate
```

### Install package

```bash
pip install flask
```

### Remove package

```bash
pip uninstall flask
```

### See installed packages

```bash
pip list
```

### Generate dependencies

```bash
pip freeze > requirements.txt
```

### Install dependencies

```bash
pip install -r requirements.txt
```

### Environment variable

```python
import os

value = os.getenv("API_KEY")
```

### `.env`

```env
API_KEY=abc123
```

### Load `.env`

```python
from dotenv import load_dotenv

load_dotenv()
```

### Poetry

```bash
poetry add flask
poetry install
poetry run python app.py
```

### uv

```bash
uv init
uv add flask
uv sync
uv run python app.py
```

---

## The Mental Model

Think of the whole ecosystem like this:

```text
Python
  │
  ├── venv
  │     └── isolated environment
  │
  ├── pip
  │     └── install/manage packages
  │
  ├── requirements.txt
  │     └── record dependencies
  │
  ├── environment variables
  │     └── external configuration
  │
  ├── .env
  │     └── local environment configuration
  │
  ├── python-dotenv
  │     └── load .env into Python
  │
  ├── Poetry
  │     └── project + dependency management
  │
  └── uv
        └── fast modern Python package/project management
```

### What you should remember for interviews

The most important distinctions are:

**`venv`** → isolates your Python environment.

**`pip`** → installs/manages packages.

**`requirements.txt`** → records dependencies.

**Environment variables** → provide configuration outside the code.

**`.env`** → convenient local file for environment variables.

**`python-dotenv`** → loads `.env` variables into Python.

**Poetry** → dependency + project management using `pyproject.toml`.

**uv** → fast modern alternative/toolchain for Python environments and dependencies.
