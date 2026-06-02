# Python Applications

[Back to guide index](README.md)

## 3.1 Python Deployment Landscape

Python applications range from:

- CLI tools
- Background workers
- APIs
- WSGI web apps
- ASGI services
- Packaging libraries

This section emphasizes production service deployment on Linux.

## 3.2 Python Installation on Linux

System package manager example:

```bash
# apt update
# apt install -y python3 python3-venv python3-pip
```

RHEL-compatible example:

```bash
# dnf install -y python3 python3-pip
```

Verify:

```bash
$ python3 --version
$ pip3 --version
```

## 3.3 Python Version Management with pyenv

`pyenv` is useful when different applications require different Python versions.

Install dependencies first, then install pyenv.

Example shell setup:

```bash
$ curl https://pyenv.run | bash
```

Add to shell config:

```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
```

Install and set a version:

```bash
$ pyenv install 3.12.4
$ pyenv global 3.12.4
$ python --version
```

Use cases:

- Developer workstations
- Shared CI runners
- Build agents with multiple supported Python versions

## 3.4 Using Deadsnakes PPA on Ubuntu

When newer Python versions are needed on Ubuntu, the Deadsnakes PPA is commonly used.

Example:

```bash
# add-apt-repository ppa:deadsnakes/ppa
# apt update
# apt install -y python3.12 python3.12-venv python3.12-dev
```

Use caution on production servers.

Prefer:

- Official distro packages when acceptable
- pyenv for controlled app-specific versions
- Containerized deployment for strict isolation

## 3.5 Virtual Environments with `venv`

`venv` is the standard lightweight way to isolate Python dependencies.

Create a virtual environment:

```bash
$ python3 -m venv .venv
```

Activate it:

```bash
$ source .venv/bin/activate
```

Install dependencies:

```bash
(.venv) $ pip install -r requirements.txt
```

Deactivate:

```bash
(.venv) $ deactivate
```

Production note:

- systemd does not need activation scripts if you point directly to the venv binary path

## 3.6 `virtualenv`

`virtualenv` is an older but still useful tool with extra features.

Install:

```bash
$ pip install virtualenv
$ virtualenv .venv
```

It is helpful when:

- Using older Python versions
- Requiring behaviors not covered by standard `venv`

## 3.7 Pipenv

Pipenv combines environment management and dependency locking.

Common commands:

```bash
$ pipenv install flask
$ pipenv install --dev pytest
$ pipenv shell
$ pipenv run python app.py
```

Files:

- `Pipfile`
- `Pipfile.lock`

Trade-offs:

- Cleaner workflow for some teams
- Less universal in production operations than plain `venv` or Poetry

## 3.8 Poetry

Poetry manages dependencies and packaging through `pyproject.toml`.

Install and initialize:

```bash
$ pip install poetry
$ poetry init
```

Install dependencies:

```bash
$ poetry add flask
$ poetry add --group dev pytest
```

Run commands:

```bash
$ poetry run python app.py
$ poetry run gunicorn app:app
```

Advantages:

- Good dependency management
- Lockfile support
- Standardized packaging workflow

## 3.9 `pip` and `requirements.txt`

Classic dependency installation uses pip plus a requirements file.

Example `requirements.txt`:

```text
Flask==3.0.3
gunicorn==22.0.0
psycopg[binary]==3.2.1
```

Install:

```bash
$ pip install -r requirements.txt
```

Freeze current environment:

```bash
$ pip freeze > requirements.txt
```

Best practice:

- Prefer curated, reviewed dependency pins over freezing everything blindly for libraries
- For applications, pinned transitive dependencies are often acceptable

## 3.10 Building Python Packages with `setup.py`

Legacy packaging often uses `setup.py`.

Example:

```python
from setuptools import setup, find_packages

setup(
    name="demoapp",
    version="1.0.0",
    packages=find_packages(),
    install_requires=["Flask==3.0.3"],
)
```

Build:

```bash
$ python -m pip install --upgrade build
$ python -m build
```

Artifacts:

- Source distribution (`sdist`)
- Wheel (`.whl`)

## 3.11 Building with `pyproject.toml`

Modern Python packaging uses `pyproject.toml`.

Example:

```toml
[build-system]
requires = ["setuptools>=68", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "demoapp"
version = "1.0.0"
dependencies = [
  "Flask==3.0.3"
]
```

Build:

```bash
$ python -m build
```

## 3.12 WSGI Servers

Production Python web apps should not use the development server.

Use a production WSGI or ASGI server instead.

Common WSGI servers:

- Gunicorn
- uWSGI
- mod_wsgi

## 3.13 Gunicorn Basics

Gunicorn is a simple and popular WSGI server.

Install:

```bash
$ pip install gunicorn
```

Run a Flask app:

```bash
$ gunicorn --bind 127.0.0.1:8000 app:app
```

Useful options:

| Option | Purpose |
|---|---|
| `--workers` | Number of worker processes |
| `--threads` | Threads per worker |
| `--bind` | Listen address |
| `--timeout` | Worker timeout |
| `--access-logfile -` | Access log to stdout |
| `--error-logfile -` | Error log to stdout |

Example:

```bash
$ gunicorn \
  --bind 127.0.0.1:8000 \
  --workers 4 \
  --threads 2 \
  --timeout 30 \
  app:app
```

## 3.14 uWSGI Basics

uWSGI is feature-rich and battle-tested.

Example:

```bash
$ uwsgi --http 127.0.0.1:8000 --wsgi-file app.py --callable app --processes 4 --threads 2
```

Trade-offs:

- Very powerful
- More configuration complexity than Gunicorn

## 3.15 Deploying Flask with Nginx and Gunicorn

Application file:

```python
from flask import Flask
app = Flask(__name__)

@app.get("/health")
def health():
    return {"status": "ok"}
```

Run via Gunicorn:

```bash
$ /opt/flaskapp/venv/bin/gunicorn --bind 127.0.0.1:8000 app:app
```

Nginx config:

```nginx
server {
    listen 80;
    server_name flask.example.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 3.16 Deploying Django with Nginx and Gunicorn

Install dependencies:

```bash
$ pip install django gunicorn
```

Collect static files:

```bash
$ python manage.py collectstatic --noinput
```

Run Gunicorn:

```bash
$ gunicorn --bind 127.0.0.1:8000 myproject.wsgi:application
```

Nginx with static file serving:

```nginx
server {
    listen 80;
    server_name django.example.com;

    location /static/ {
        alias /opt/djangoapp/static/;
        expires 30d;
        access_log off;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 3.17 systemd Service for Python Apps

Example service file:

```ini
[Unit]
Description=Gunicorn for Flask App
After=network.target

[Service]
User=flaskapp
Group=flaskapp
WorkingDirectory=/opt/flaskapp/current
EnvironmentFile=/etc/flaskapp/flaskapp.env
ExecStart=/opt/flaskapp/venv/bin/gunicorn \
  --workers 4 \
  --bind 127.0.0.1:8000 \
  app:app
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Example environment file:

```bash
FLASK_ENV=production
DATABASE_URL=postgresql://flaskapp:secret@db.internal/flaskdb
```

## 3.18 Python Deployment Layout

Recommended layout:

```text
/opt/flaskapp/
├── current -> /opt/flaskapp/releases/2025-01-15_120000/
├── releases/
├── shared/
│   ├── logs/
│   ├── static/
│   └── uploads/
└── venv/
```

Notes:

- Keep the virtual environment outside each release if dependencies are stable between releases
- Or make each release self-contained if dependency immutability is required

## 3.19 Pip Caching and CI

In CI, cache:

- `~/.cache/pip`
- Poetry cache directories
- Virtualenvs only when safe and deterministic

## 3.20 Python Build and Test Workflow

Typical sequence:

```bash
$ python -m venv .venv
$ source .venv/bin/activate
$ pip install -U pip
$ pip install -r requirements.txt
$ pytest
$ python -m build
```

## 3.21 ASGI Note

Modern frameworks like FastAPI use ASGI.

Production servers include:

- Uvicorn
- Hypercorn
- Gunicorn with Uvicorn workers

Example:

```bash
$ gunicorn -k uvicorn.workers.UvicornWorker --bind 127.0.0.1:8000 app:app
```

## 3.22 Common Python Deployment Mistakes

- Running the dev server in production
- Mixing system packages and app packages in the same interpreter environment
- Forgetting `collectstatic` for Django
- Not pinning dependencies
- Missing OS libraries required for Python wheels or builds
- Pointing systemd to `python` instead of the venv path

## 3.23 Python Logging Recommendations

Prefer application logs to stdout/stderr when managed by systemd and journal.

Or log to files if your organization requires it.

Recommended fields:

- Timestamp
- Level
- Service name
- Correlation/request ID
- Exception details

---
