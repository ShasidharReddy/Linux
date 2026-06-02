# Build Systems & Application Deployment Guide for Linux

## How to Use This Guide

This guide is a production-focused reference for building, packaging, deploying, operating, and troubleshooting applications on Linux.

It is intentionally practical.

It covers:

- Native compiled applications
- JVM applications
- Python services
- Node.js services
- Go binaries
- .NET services
- Web server integration
- Process management
- Deployment strategies
- CI/CD patterns
- Monitoring and troubleshooting
- Configuration and secrets management

It is written for:

- Linux administrators
- DevOps engineers
- Backend developers
- SRE teams
- Platform engineers
- Full-stack developers responsible for deployments

Assumptions:

- Target systems are Linux servers
- Bash is available
- You have sudo access when installing packages
- You are deploying server-side applications or services
- You want reliable, repeatable, and observable deployments

Conventions used in this guide:

- Commands prefixed with `$` are run as a regular user
- Commands prefixed with `#` are run as root or with sudo
- File paths are shown as Linux absolute paths where appropriate
- systemd is treated as the default init and service manager
- Nginx is the primary reverse proxy example

---

# 1. Build Systems Overview

## 1.1 What a Build System Does

A build system transforms source code into runnable artifacts.

Artifacts may include:

- Executable binaries
- Shared libraries
- Static libraries
- JAR files
- WAR files
- Python wheels
- Docker images
- Bundled JavaScript assets
- Deployment packages

A build system usually handles:

- Source file discovery
- Dependency resolution
- Compilation or transpilation
- Linking
- Asset bundling
- Testing
- Packaging
- Version stamping
- Artifact publication

A mature build system also supports:

- Incremental builds
- Parallel builds
- Reproducibility
- Caching
- Cross-platform targets
- Profiles for development and production

## 1.2 Compilation Basics

Compilation is the process of translating source code into lower-level code.

Depending on the language, the output may be:

- Machine code
- Bytecode
- Intermediate representation
- Bundled source transformed for a runtime

### 1.2.1 Preprocessing

Languages like C and C++ often include a preprocessing step.

Tasks performed by a preprocessor include:

- Expanding macros
- Resolving `#include` directives
- Applying conditional compilation

Example:

```bash
$ gcc -E hello.c -o hello.i
```

This produces preprocessed source.

### 1.2.2 Compilation Proper

The compiler translates the preprocessed code into assembly or object code.

Example:

```bash
$ gcc -S hello.c -o hello.s
$ gcc -c hello.c -o hello.o
```

- `-S` stops after generating assembly
- `-c` stops after generating an object file

### 1.2.3 Linking

Linking combines object files and libraries into an executable.

Example:

```bash
$ gcc hello.o -o hello
```

If the program depends on libraries:

```bash
$ gcc main.o util.o -lm -lpthread -o app
```

### 1.2.4 Static vs Dynamic Linking

Static linking embeds required library code into the final binary.

Benefits:

- Fewer runtime dependencies
- Easier distribution in some cases

Trade-offs:

- Larger binaries
- Potential duplication across services

Dynamic linking references shared libraries available on the target system.

Benefits:

- Smaller binaries
- Shared updates for common libraries

Trade-offs:

- Runtime dependency issues if versions differ
- Library path problems

### 1.2.5 Debug vs Release Builds

Debug builds favor debuggability.

Common properties:

- Symbols included
- Low optimization
- Extra runtime checks

Release builds favor performance.

Common properties:

- Higher optimization
- Stripped symbols
- Smaller binaries

Example:

```bash
$ gcc -g -O0 main.c -o app-debug
$ gcc -O2 -DNDEBUG main.c -o app-release
```

## 1.3 GCC and G++ Basics

GCC is the GNU Compiler Collection.

It supports multiple languages, including C and C++.

Typical commands:

```bash
$ gcc hello.c -o hello
$ g++ main.cpp -o app
```

Useful flags:

| Flag | Meaning |
|---|---|
| `-Wall` | Enable common warnings |
| `-Wextra` | Enable additional warnings |
| `-Werror` | Treat warnings as errors |
| `-g` | Include debug symbols |
| `-O0` | No optimization |
| `-O2` | Balanced optimization |
| `-O3` | Aggressive optimization |
| `-std=c11` | Use C11 standard |
| `-std=c++20` | Use C++20 standard |
| `-I<dir>` | Add header include directory |
| `-L<dir>` | Add library directory |
| `-l<name>` | Link library |
| `-DNAME=value` | Define macro |
| `-pthread` | Enable pthread support |

Example production-style compile:

```bash
$ gcc -Wall -Wextra -Werror -O2 -g -Iinclude src/main.c src/util.c -o bin/app
```

Example C++ compile:

```bash
$ g++ -std=c++20 -Wall -Wextra -O2 -Iinclude src/*.cpp -o bin/app
```

## 1.4 make Basics

`make` is a build automation tool based on targets, dependencies, and recipes.

A simple `Makefile`:

```make
CC=gcc
CFLAGS=-Wall -Wextra -O2
TARGET=app
SRC=main.c util.c

all: $(TARGET)

$(TARGET): $(SRC)
	$(CC) $(CFLAGS) $(SRC) -o $(TARGET)

clean:
	rm -f $(TARGET)
```

Common usage:

```bash
$ make
$ make clean
$ make -j4
```

Key concepts:

- Targets represent files or actions
- Dependencies must be updated first
- Recipes are shell commands to build targets

Benefits of make:

- Simple
- Ubiquitous
- Good for small and medium native projects

Limitations:

- Manual dependency management can be fragile
- Large projects may become difficult to maintain

## 1.5 CMake Basics

CMake is a meta-build system.

It generates native build files for:

- Make
- Ninja
- Visual Studio
- Xcode

Simple `CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.16)
project(MyApp C)

set(CMAKE_C_STANDARD 11)

add_executable(myapp
    src/main.c
    src/util.c
)

target_include_directories(myapp PRIVATE include)
```

Typical workflow:

```bash
$ cmake -S . -B build
$ cmake --build build
$ ./build/myapp
```

Production-style Release build:

```bash
$ cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
$ cmake --build build --parallel
```

Install target example:

```cmake
install(TARGETS myapp DESTINATION bin)
```

Then:

```bash
$ cmake --install build --prefix /opt/myapp
```

Advantages of CMake:

- Portable
- Scales better than handwritten Makefiles
- Handles external dependencies more cleanly
- Works well with IDEs and CI

## 1.6 Build Automation Tools Comparison

| Tool | Ecosystem | Best For | Strengths | Weaknesses |
|---|---|---|---|---|
| `make` | Native | Simple C/C++ builds | Ubiquitous, lightweight | Can become complex quickly |
| `cmake` | Native | Cross-platform native builds | Flexible, modern, scalable | Syntax is verbose |
| `maven` | Java | Convention-driven Java builds | Predictable lifecycle | XML-heavy |
| `gradle` | JVM | Complex JVM builds | Flexible, powerful | More complexity |
| `pip` + `build` | Python | Packaging | Standard ecosystem | Packaging differences across projects |
| `poetry` | Python | App and package management | Nice dependency workflow | Less universal in legacy repos |
| `npm` | Node.js | JS builds and scripts | Massive ecosystem | Dependency sprawl |
| `pnpm` | Node.js | Efficient package installs | Fast, space-efficient | Team adoption may vary |
| `go build` | Go | Native Go binaries | Extremely simple | Fewer extension points than larger systems |
| `dotnet` CLI | .NET | .NET app lifecycle | Integrated tooling | SDK/runtime version alignment matters |

## 1.7 Build Artifacts and Promotion

Build artifacts should be immutable.

That means:

- Build once
- Promote the same artifact through environments
- Avoid rebuilding for staging and production unless configuration differs by design

Examples of immutable artifacts:

- `app-1.4.2.jar`
- `api-2.1.0.war`
- `service-3.0.1-linux-amd64`
- `myapp-0.9.0-py3-none-any.whl`
- `frontend-2025.01.15.tar.gz`
- Container images tagged with commit SHA

Best practices:

- Include version and build metadata
- Store artifacts in an artifact repository
- Generate checksums
- Sign artifacts when required
- Retain a rollback history

## 1.8 Reproducible Builds

A reproducible build gives the same output for the same source and inputs.

Controls that improve reproducibility:

- Pin compiler versions
- Pin dependency versions
- Avoid fetching floating dependencies in CI
- Use lockfiles
- Record build metadata
- Avoid embedding timestamps unless normalized
- Build inside controlled environments

## 1.9 Build Pipeline Stages

Typical stages:

1. Source checkout
2. Dependency restore
3. Static analysis
4. Unit tests
5. Compilation or packaging
6. Artifact signing
7. Artifact publication
8. Deployment
9. Verification

Mermaid diagram:

```mermaid
graph LR
    A["Source Checkout"] --> B["Dependency Restore"]
    B --> C["Static Analysis"]
    C --> D["Unit Tests"]
    D --> E["Build or Package"]
    E --> F["Artifact Publish"]
    F --> G["Deploy"]
    G --> H["Post-Deploy Verify"]
```

## 1.10 Build Environment Checklist

Use this checklist for reliable Linux builds.

| Item | Why It Matters |
|---|---|
| Compiler/SDK version pinned | Prevents inconsistent output |
| Dependencies locked | Avoids drift |
| Dedicated build user | Improves security |
| Clean workspace | Reduces contamination |
| Artifact repository configured | Enables traceability |
| Test results archived | Supports diagnostics |
| Build logs retained | Supports auditing |
| Secrets not baked into artifacts | Avoids leakage |

## 1.11 Common Build Failures

Typical causes:

- Missing headers or libraries
- Wrong compiler version
- Incompatible dependency versions
- Permissions issues in build workspace
- Out-of-disk conditions
- Case-sensitive path issues
- Path length or shell quoting errors

Basic diagnostics:

```bash
$ gcc --version
$ cmake --version
$ make --version
$ env | sort
$ df -h
$ free -m
```

---

# 2. Java/JVM Applications

## 2.1 JVM Application Types

Common JVM deliverables include:

- Runnable JAR files
- Library JAR files
- WAR files for servlet containers
- EAR files for application servers
- Native images in some toolchains

Typical frameworks:

- Spring Boot
- Jakarta EE
- Micronaut
- Quarkus
- Dropwizard

## 2.2 JDK Installation on Linux

Install JDK using the system package manager when consistency with OS packages is important.

Ubuntu example:

```bash
# apt update
# apt install -y openjdk-17-jdk
```

RHEL-compatible example:

```bash
# dnf install -y java-17-openjdk-devel
```

Check installation:

```bash
$ java -version
$ javac -version
```

## 2.3 Managing Multiple JDK Versions with SDKMAN

SDKMAN is useful for developer workstations and CI hosts requiring multiple versions.

Install:

```bash
$ curl -s "https://get.sdkman.io" | bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

List Java candidates:

```bash
$ sdk list java
```

Install and select a version:

```bash
$ sdk install java 17.0.12-tem
$ sdk use java 17.0.12-tem
$ sdk default java 17.0.12-tem
```

Benefits:

- Simple per-user version switching
- Good for development
- Good for build agents with multiple stacks

Trade-offs:

- Less ideal for tightly controlled production servers
- Shell initialization required

## 2.4 Managing Alternatives

On Linux servers, `alternatives` or `update-alternatives` is often preferred.

Example:

```bash
# update-alternatives --config java
# update-alternatives --config javac
```

This lets you switch system-default Java versions.

Best practice:

- Pin the target JDK version in deployment automation
- Avoid manual switching on shared production hosts

## 2.5 Maven Overview

Maven is a convention-first build system for Java.

Core concepts:

- `pom.xml` defines project metadata
- Dependencies are resolved from repositories
- Lifecycle phases are standardized
- Plugins extend behavior

Minimal `pom.xml`:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>demo-app</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>
</project>
```

## 2.6 Maven Build Lifecycle

Main default lifecycle phases:

| Phase | Purpose |
|---|---|
| `validate` | Validate project structure |
| `compile` | Compile main source code |
| `test` | Run unit tests |
| `package` | Create JAR/WAR |
| `verify` | Run checks against package |
| `install` | Install artifact to local repo |
| `deploy` | Publish artifact to remote repo |

Common commands:

```bash
$ mvn clean
$ mvn compile
$ mvn test
$ mvn package
$ mvn install
$ mvn deploy
```

### 2.6.1 clean

Removes previous build output.

```bash
$ mvn clean
```

### 2.6.2 compile

Compiles source in `src/main/java`.

```bash
$ mvn compile
```

### 2.6.3 test

Runs tests from `src/test/java`.

```bash
$ mvn test
```

### 2.6.4 package

Creates deployable output.

```bash
$ mvn package
```

Output examples:

- `target/demo-app-1.0.0.jar`
- `target/demo-app-1.0.0.war`

### 2.6.5 install

Installs the artifact to the local Maven repository.

```bash
$ mvn install
```

Local repository path is usually:

```text
~/.m2/repository
```

### 2.6.6 deploy

Uploads artifacts to a remote repository manager such as:

- Nexus
- Artifactory
- GitHub Packages

```bash
$ mvn deploy
```

## 2.7 Maven Profiles

Profiles allow environment-specific variations.

Example:

```xml
<profiles>
  <profile>
    <id>prod</id>
    <properties>
      <skipTests>true</skipTests>
    </properties>
  </profile>
</profiles>
```

Run with:

```bash
$ mvn package -Pprod
```

Use profiles carefully.

Better pattern:

- Keep build output environment-agnostic
- Inject environment configuration at deploy time

## 2.8 Building Executable JAR Files

Many Spring Boot and standalone apps produce runnable JARs.

Example plugin:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

Build command:

```bash
$ mvn clean package
```

Run:

```bash
$ java -jar target/demo-app-1.0.0.jar
```

## 2.9 Building WAR Files

WAR files are used for deployment to servlet containers or Java application servers.

Set packaging:

```xml
<packaging>war</packaging>
```

Build:

```bash
$ mvn clean package
```

Output:

```text
target/demo-app-1.0.0.war
```

## 2.10 Gradle Basics

Gradle is a flexible build automation tool for JVM ecosystems.

Key features:

- Incremental builds
- Dependency management
- Plugin ecosystem
- Groovy or Kotlin DSL

Example `build.gradle`:

```groovy
plugins {
    id 'java'
}

group = 'com.example'
version = '1.0.0'

repositories {
    mavenCentral()
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.2'
}

test {
    useJUnitPlatform()
}
```

Common commands:

```bash
$ ./gradlew clean
$ ./gradlew build
$ ./gradlew test
$ ./gradlew bootJar
```

Advantages over Maven in some cases:

- More scripting flexibility
- Better for complex multi-project builds

Trade-offs:

- More room for build logic sprawl
- Requires discipline for maintainability

## 2.11 Building with Gradle Wrapper

Prefer the wrapper over a system-wide Gradle install.

Example:

```bash
$ ./gradlew clean build
```

Why:

- Project controls Gradle version
- Better CI reproducibility
- Less host drift

## 2.12 Running JVM Apps with `java -jar`

Basic example:

```bash
$ java -jar app.jar
```

Passing JVM options:

```bash
$ java -Xms512m -Xmx1024m -jar app.jar
```

Passing application properties:

```bash
$ java -jar app.jar --server.port=8081 --spring.profiles.active=prod
```

Using environment variables:

```bash
$ export SPRING_PROFILES_ACTIVE=prod
$ java -jar app.jar
```

## 2.13 JVM Heap Sizing

Heap options:

- `-Xms` sets initial heap size
- `-Xmx` sets maximum heap size

Example:

```bash
$ java -Xms1g -Xmx1g -jar app.jar
```

Guidelines:

- Avoid setting `-Xmx` so high that the OS starves
- Consider container memory limits
- Leave headroom for metaspace, thread stacks, native buffers, and the OS page cache

## 2.14 GC Options

Modern JVMs default to G1GC in many cases.

Common options:

```bash
$ java -XX:+UseG1GC -Xms1g -Xmx2g -jar app.jar
```

Other useful flags:

| Flag | Purpose |
|---|---|
| `-XX:+HeapDumpOnOutOfMemoryError` | Write heap dump on OOM |
| `-XX:HeapDumpPath=/var/log/myapp` | Heap dump location |
| `-Xlog:gc*:file=/var/log/myapp/gc.log:time,uptime,level,tags` | GC logging |
| `-XX:MaxRAMPercentage=75` | Heap sizing relative to RAM |

Example production launch:

```bash
$ java \
  -XX:+UseG1GC \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/var/log/myapp \
  -Xlog:gc*:file=/var/log/myapp/gc.log:time,uptime,level,tags \
  -Xms512m \
  -Xmx1024m \
  -jar /opt/myapp/app.jar
```

## 2.15 systemd Service for a JAR Application

Example service file:

```ini
[Unit]
Description=Demo Java Application
After=network.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
Environment="JAVA_OPTS=-Xms512m -Xmx1024m -XX:+UseG1GC"
ExecStart=/usr/bin/bash -lc '/usr/bin/java $JAVA_OPTS -jar /opt/myapp/app.jar'
SuccessExitStatus=143
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

Install and run:

```bash
# cp myapp.service /etc/systemd/system/
# systemctl daemon-reload
# systemctl enable --now myapp
# systemctl status myapp
```

## 2.16 Deploying to Tomcat

Tomcat is a servlet container commonly used for WAR deployments.

Install Tomcat via package manager or official distribution.

WAR deployment pattern:

1. Build WAR
2. Copy WAR to Tomcat `webapps/`
3. Restart Tomcat or let auto-deploy unpack it

Example:

```bash
$ mvn clean package
# cp target/demo-app.war /opt/tomcat/webapps/demo-app.war
# systemctl restart tomcat
```

Common Tomcat directories:

| Path | Purpose |
|---|---|
| `/opt/tomcat/bin` | Startup scripts |
| `/opt/tomcat/conf` | Main configuration |
| `/opt/tomcat/logs` | Logs |
| `/opt/tomcat/webapps` | Deployed apps |
| `/opt/tomcat/temp` | Temp files |
| `/opt/tomcat/work` | Compiled JSPs and work dirs |

## 2.17 Tomcat Reverse Proxy with Nginx

Example Nginx config:

```nginx
server {
    listen 80;
    server_name java.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 2.18 Deploying to WildFly

WildFly is a full Java application server.

Deployment options:

- Standalone mode
- Domain mode
- CLI-based deployment
- File copy deployment

CLI example:

```bash
$ /opt/wildfly/bin/jboss-cli.sh --connect --command="deploy /path/to/app.war --force"
```

Service management is typically done with systemd.

## 2.19 JVM Deployment Directory Layout

Recommended layout:

```text
/opt/myapp/
├── app.jar
├── config/
│   └── application-prod.yml
├── logs/
└── releases/
```

Alternate release-based layout:

```text
/opt/myapp/
├── current -> /opt/myapp/releases/2025-01-15_120000/
├── releases/
│   ├── 2025-01-10_090000/
│   └── 2025-01-15_120000/
└── shared/
    ├── config/
    └── logs/
```

## 2.20 Java Deployment Checklist

- Correct JDK version installed
- Artifact checksum verified
- Service user exists
- Log directory writable
- Heap settings validated
- Health endpoint available
- Reverse proxy headers configured
- systemd restart policy set
- Old releases retained for rollback

## 2.21 Maven CI Build Example

```bash
$ mvn -B -ntp clean verify
```

Flags:

- `-B` batch mode
- `-ntp` no transfer progress

## 2.22 Gradle CI Build Example

```bash
$ ./gradlew --no-daemon clean test build
```

Use `--no-daemon` in ephemeral CI environments.

## 2.23 Common Java Deployment Mistakes

- Running with the wrong JDK major version
- Shipping a thin JAR without dependencies when a fat JAR was expected
- Not externalizing config
- Oversizing heap and causing host pressure
- Missing `X-Forwarded-*` handling behind a reverse proxy
- Assuming localhost-only health checks are enough

---

# 3. Python Applications

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

# 4. Node.js Applications

## 4.1 Node.js Deployment Model

Node.js apps commonly include:

- APIs with Express, Fastify, NestJS
- SSR apps like Next.js
- Static site builds
- Worker processes
- WebSocket servers

## 4.2 Node Version Management

### 4.2.1 nvm

`nvm` is widely used for per-user Node.js version management.

Install:

```bash
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
$ source ~/.nvm/nvm.sh
```

Use:

```bash
$ nvm install 20
$ nvm use 20
$ node -v
```

### 4.2.2 n

`n` is a simple Node version manager.

```bash
# npm install -g n
# n 20
```

### 4.2.3 fnm

`fnm` is a fast Node manager written in Rust.

Benefits:

- Very fast startup
- Good shell integration

## 4.3 Package Managers

Common choices:

- npm
- yarn
- pnpm

Comparison:

| Tool | Strengths | Notes |
|---|---|---|
| npm | Default, universal | Good default choice |
| yarn | Mature workspace support | Yarn classic vs berry differs |
| pnpm | Fast, disk-efficient | Excellent for monorepos |

## 4.4 `package.json` Scripts

Scripts standardize lifecycle commands.

Example:

```json
{
  "scripts": {
    "dev": "node server.js",
    "build": "vite build",
    "start": "node dist/server.js",
    "test": "vitest run"
  }
}
```

Use:

```bash
$ npm run build
$ npm run test
$ npm start
```

## 4.5 Dependency Installation

Use clean install in CI.

npm:

```bash
$ npm ci
```

Yarn:

```bash
$ yarn install --frozen-lockfile
```

pnpm:

```bash
$ pnpm install --frozen-lockfile
```

Why:

- Uses lockfile strictly
- More reproducible than floating installs

## 4.6 Build and Bundling Tools

Modern Node and frontend-oriented projects commonly use:

- webpack
- Vite
- esbuild
- Rollup

### 4.6.1 webpack

Powerful and highly configurable.

Use for:

- Complex frontend bundling
- Legacy asset pipelines
- Advanced plugin ecosystems

### 4.6.2 Vite

Fast developer experience and excellent modern frontend build support.

Use for:

- React
- Vue
- Svelte
- Static SPA builds

### 4.6.3 esbuild

Extremely fast bundler and transformer.

Use for:

- Tooling
- Fast builds
- Simple production bundles

## 4.7 Express Application Example

Basic app:

```javascript
const express = require('express');
const app = express();

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.listen(3000, '127.0.0.1', () => {
  console.log('Listening on 3000');
});
```

Install and run:

```bash
$ npm install express
$ node server.js
```

## 4.8 Building Node Apps

Typical production sequence:

```bash
$ npm ci
$ npm test
$ npm run build
```

For TypeScript:

```bash
$ npm ci
$ npm run lint
$ npm run test
$ npm run build
$ node dist/server.js
```

## 4.9 PM2 Process Manager

PM2 is widely used for long-running Node processes.

Install:

```bash
# npm install -g pm2
```

Start an app:

```bash
$ pm2 start dist/server.js --name mynodeapp
```

Useful commands:

```bash
$ pm2 status
$ pm2 logs mynodeapp
$ pm2 restart mynodeapp
$ pm2 stop mynodeapp
$ pm2 delete mynodeapp
$ pm2 save
```

Cluster mode example:

```bash
$ pm2 start dist/server.js --name mynodeapp -i max
```

## 4.10 PM2 Ecosystem File

Example `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [
    {
      name: 'mynodeapp',
      script: './dist/server.js',
      instances: 2,
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3000
      }
    }
  ]
};
```

Run:

```bash
$ pm2 start ecosystem.config.js
```

## 4.11 systemd vs PM2

| Aspect | systemd | PM2 |
|---|---|---|
| OS-native | Yes | No |
| Good for non-Node services too | Yes | No |
| User-level Node convenience | Moderate | High |
| Centralized Linux operations | Excellent | Moderate |
| Cluster mode built-in | No | Yes |

For standardized Linux ops, systemd is usually preferred.

PM2 is convenient when teams are Node-centric.

## 4.12 Deploying Express Behind Nginx

Nginx config:

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 4.13 Deploying Next.js

There are two common patterns:

- `next start` behind Nginx
- Static export when the app is fully static

Build and run:

```bash
$ npm ci
$ npm run build
$ npm run start
```

If using standalone output, package the `.next/standalone` directory.

Example systemd `ExecStart`:

```ini
ExecStart=/usr/bin/node /opt/nextapp/current/server.js
```

## 4.14 Node.js systemd Service

Example service file:

```ini
[Unit]
Description=Node.js API Service
After=network.target

[Service]
Type=simple
User=nodeapp
Group=nodeapp
WorkingDirectory=/opt/nodeapp/current
EnvironmentFile=/etc/nodeapp/nodeapp.env
ExecStart=/usr/bin/node /opt/nodeapp/current/dist/server.js
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

Example env file:

```bash
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://nodeapp:secret@db.internal/nodeapp
```

## 4.15 Node Build Artifacts

Common build outputs:

- `dist/`
- `.next/`
- `build/`
- `public/` assets
- Transpiled server bundle

Keep production deployments minimal.

Do not copy:

- Tests
- Local caches
- Dev-only assets
- Unnecessary source maps unless needed for debugging

## 4.16 Lockfiles Matter

Use exactly one lockfile for a project.

Examples:

- `package-lock.json`
- `yarn.lock`
- `pnpm-lock.yaml`

Avoid mixing package managers.

## 4.17 Node Deployment Layout

```text
/opt/nodeapp/
├── current -> /opt/nodeapp/releases/2025-01-15_120000/
├── releases/
└── shared/
    ├── logs/
    └── uploads/
```

## 4.18 WebSocket Proxying for Node Apps

Nginx config example:

```nginx
location /socket.io/ {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
}
```

## 4.19 Common Node Deployment Mistakes

- Running `npm install` instead of `npm ci` in CI
- Forgetting to build TypeScript before starting
- Missing reverse proxy WebSocket headers
- Using dev dependencies at runtime unintentionally
- Running as root
- Not setting `NODE_ENV=production`

## 4.20 Production Node Checklist

- Correct Node version installed
- Lockfile present and used
- Build output generated
- Reverse proxy configured
- Health endpoint exposed
- Memory limits monitored
- Logs centralized
- Restart policy enabled

---

# 5. Go Applications

## 5.1 Why Go Deployments Are Straightforward

Go compiles to native binaries.

This greatly simplifies deployment.

Benefits:

- Single binary distribution in many cases
- Simple process model
- Fast startup
- Strong cross-compilation support

## 5.2 Installing Go on Linux

Install from package manager or official tarball.

Package manager example:

```bash
# apt install -y golang-go
```

Official tarball pattern:

```bash
# tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz
```

Set PATH:

```bash
export PATH=/usr/local/go/bin:$PATH
```

Verify:

```bash
$ go version
```

## 5.3 Workspace Basics

Modern Go uses modules.

Initialize a module:

```bash
$ go mod init example.com/myapp
```

Download dependencies:

```bash
$ go mod tidy
```

Important files:

- `go.mod`
- `go.sum`

## 5.4 Building a Go Binary

Simple build:

```bash
$ go build -o myapp ./cmd/myapp
```

Run:

```bash
$ ./myapp
```

## 5.5 Static Binaries

Many Go apps can be built statically.

Example:

```bash
$ CGO_ENABLED=0 go build -ldflags="-s -w" -o myapp ./cmd/myapp
```

Notes:

- `CGO_ENABLED=0` disables CGO
- `-s -w` reduces binary size by stripping symbol/debug information

Use static builds for:

- Simple host deployments
- Scratch or distroless containers
- Minimal runtime dependencies

## 5.6 Embedding Build Metadata

Example:

```bash
$ go build -ldflags="-X main.version=1.0.0 -X main.commit=$(git rev-parse --short HEAD)" -o myapp ./cmd/myapp
```

This is useful for:

- `/version` endpoints
- Incident diagnostics
- Deployment verification

## 5.7 Cross-Compilation

Go makes cross-compilation easy.

Examples:

```bash
$ GOOS=linux GOARCH=amd64 go build -o myapp-linux-amd64 ./cmd/myapp
$ GOOS=linux GOARCH=arm64 go build -o myapp-linux-arm64 ./cmd/myapp
$ GOOS=darwin GOARCH=arm64 go build -o myapp-darwin-arm64 ./cmd/myapp
```

Common pairs:

| GOOS | GOARCH | Target |
|---|---|---|
| `linux` | `amd64` | x86_64 Linux |
| `linux` | `arm64` | ARM64 Linux |
| `darwin` | `arm64` | Apple Silicon |
| `windows` | `amd64` | Windows x86_64 |

## 5.8 Go Test and Build Workflow

```bash
$ go fmt ./...
$ go test ./...
$ go build ./...
```

For CI:

```bash
$ go test -race ./...
$ go build -trimpath -o myapp ./cmd/myapp
```

## 5.9 Deploying a Go Binary with systemd

Example service:

```ini
[Unit]
Description=Go API Service
After=network.target

[Service]
Type=simple
User=goapp
Group=goapp
WorkingDirectory=/opt/goapp/current
EnvironmentFile=/etc/goapp/goapp.env
ExecStart=/opt/goapp/current/myapp
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

## 5.10 Nginx Reverse Proxy for Go HTTP Services

```nginx
server {
    listen 80;
    server_name go.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 5.11 Minimal Docker Images for Go

Multi-stage Dockerfile example:

```dockerfile
FROM golang:1.22 AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o /out/myapp ./cmd/myapp

FROM gcr.io/distroless/static-debian12
COPY --from=build /out/myapp /myapp
USER nonroot:nonroot
ENTRYPOINT ["/myapp"]
```

Benefits:

- Small final image
- Reduced attack surface
- Fast startup

## 5.12 Release Layout for Go Apps

```text
/opt/goapp/
├── current -> /opt/goapp/releases/2025-01-15_120000/
├── releases/
└── shared/
    ├── logs/
    └── config/
```

## 5.13 Common Go Deployment Mistakes

- Forgetting CGO dependencies when moving binaries between hosts
- Not embedding version metadata
- Running without resource limits or visibility
- Binding publicly when a reverse proxy should front the service
- Shipping debug builds unintentionally

---

# 6. .NET Applications

## 6.1 .NET on Linux

Modern .NET runs well on Linux.

Common app types:

- Console apps
- ASP.NET Core APIs
- Background workers
- Minimal APIs
- MVC applications

## 6.2 Installing the .NET SDK

Install from Microsoft package repositories.

Ubuntu example outline:

```bash
# apt update
# apt install -y dotnet-sdk-8.0
```

Verify:

```bash
$ dotnet --info
$ dotnet --list-sdks
$ dotnet --list-runtimes
```

## 6.3 Basic Build Workflow

Restore dependencies:

```bash
$ dotnet restore
```

Build:

```bash
$ dotnet build -c Release
```

Test:

```bash
$ dotnet test -c Release --no-build
```

Publish:

```bash
$ dotnet publish -c Release -o publish
```

## 6.4 Publish Modes

Common publish modes:

- Framework-dependent deployment
- Self-contained deployment

Framework-dependent means the target host has the required runtime installed.

Self-contained bundles the runtime.

Framework-dependent example:

```bash
$ dotnet publish -c Release -o publish
```

Self-contained example:

```bash
$ dotnet publish -c Release -r linux-x64 --self-contained true -o publish
```

Trade-offs:

| Mode | Pros | Cons |
|---|---|---|
| Framework-dependent | Smaller artifacts | Runtime must exist on host |
| Self-contained | Simpler host runtime requirements | Larger artifacts |

## 6.5 ASP.NET Core and Kestrel

Kestrel is the built-in web server for ASP.NET Core.

Typical production pattern:

- Kestrel listens on localhost
- Nginx reverse proxies external traffic to Kestrel

Example environment:

```bash
ASPNETCORE_URLS=http://127.0.0.1:5000
ASPNETCORE_ENVIRONMENT=Production
```

## 6.6 systemd Service for ASP.NET Core

Example service file:

```ini
[Unit]
Description=ASP.NET Core Application
After=network.target

[Service]
WorkingDirectory=/opt/dotnetapp/current
ExecStart=/usr/bin/dotnet /opt/dotnetapp/current/MyApp.dll
Restart=always
RestartSec=5
KillSignal=SIGINT
SyslogIdentifier=dotnetapp
User=dotnetapp
Group=dotnetapp
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=ASPNETCORE_URLS=http://127.0.0.1:5000

[Install]
WantedBy=multi-user.target
```

## 6.7 Nginx Reverse Proxy for ASP.NET Core

```nginx
server {
    listen 80;
    server_name dotnet.example.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Add helper mapping in the `http` block if needed:

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}
```

## 6.8 Publish Directory Contents

Typical output:

- Application DLLs
- Dependency DLLs
- Config files
- Static assets
- Runtime files in self-contained publishes

## 6.9 Deploying ASP.NET Core

Typical process:

1. `dotnet publish -c Release`
2. Copy `publish/` contents to the target release directory
3. Update symlink or deployment path
4. Restart or reload systemd service
5. Validate health endpoint

## 6.10 Single-File Publish

Example:

```bash
$ dotnet publish -c Release -r linux-x64 -p:PublishSingleFile=true -o publish
```

Use when operational simplicity is more important than extraction/startup considerations.

## 6.11 Common .NET Deployment Mistakes

- Publishing framework-dependent apps without installing the runtime
- Forgetting `ASPNETCORE_ENVIRONMENT`
- Binding Kestrel directly to public interfaces unintentionally
- Not forwarding proxy headers correctly
- Rebuilding on the production host instead of deploying publish output

---

# 7. Web Server Configuration

## 7.1 Why Use a Reverse Proxy

Reverse proxies provide:

- TLS termination
- Request routing
- Compression
- Static file serving
- Connection buffering
- Security controls
- Load balancing
- WebSocket support

Common choices:

- Nginx
- Apache HTTP Server
- HAProxy

This section emphasizes Nginx and Apache.

## 7.2 Nginx Reverse Proxy Essentials

Typical proxy headers:

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

These headers are critical for:

- Accurate client IP logging
- Redirect generation
- Scheme-aware apps
- Audit trails

## 7.3 Complete Nginx Config for a Java App

```nginx
server {
    listen 80;
    server_name java.example.com;

    access_log /var/log/nginx/java-access.log;
    error_log /var/log/nginx/java-error.log warn;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 60s;
    }
}
```

## 7.4 Complete Nginx Config for Python Gunicorn

```nginx
server {
    listen 80;
    server_name python.example.com;

    client_max_body_size 20m;

    location /static/ {
        alias /opt/pythonapp/shared/static/;
        expires 7d;
        add_header Cache-Control "public";
    }

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 60s;
    }
}
```

## 7.5 Complete Nginx Config for Node.js

```nginx
server {
    listen 80;
    server_name node.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 7.6 Complete Nginx Config for Go

```nginx
server {
    listen 80;
    server_name go.example.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 7.7 Complete Nginx Config for ASP.NET Core

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
    listen 80;
    server_name dotnet.example.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 7.8 Static File Serving with Nginx

Serving static assets directly is efficient.

Example:

```nginx
location /assets/ {
    alias /opt/webapp/current/assets/;
    expires 30d;
    add_header Cache-Control "public, immutable";
}
```

Benefits:

- Lower application CPU usage
- Better caching behavior
- Reduced latency

## 7.9 Upload Size Controls

Example:

```nginx
client_max_body_size 50m;
```

Set appropriately for:

- File uploads
- API payloads
- Administrative imports

## 7.10 WebSocket Proxying

WebSockets require HTTP/1.1 and upgrade headers.

Example:

```nginx
location /ws/ {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
}
```

## 7.11 Apache with mod_proxy

Enable modules:

```bash
# a2enmod proxy proxy_http headers ssl rewrite
```

Basic virtual host:

```apache
<VirtualHost *:80>
    ServerName app.example.com

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/

    RequestHeader set X-Forwarded-Proto "http"
    RequestHeader set X-Forwarded-Port "80"
</VirtualHost>
```

## 7.12 Apache WebSocket Proxy Example

```apache
<VirtualHost *:80>
    ServerName socket.example.com

    ProxyPreserveHost On
    ProxyPass /ws/ ws://127.0.0.1:3000/ws/
    ProxyPassReverse /ws/ ws://127.0.0.1:3000/ws/
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

## 7.13 SSL/TLS with Let's Encrypt

Using Certbot with Nginx on Ubuntu:

```bash
# apt install -y certbot python3-certbot-nginx
# certbot --nginx -d app.example.com
```

Using Certbot with Apache:

```bash
# apt install -y certbot python3-certbot-apache
# certbot --apache -d app.example.com
```

Key benefits:

- Free certificates
- Automated renewal
- Easy integration

## 7.14 HTTPS Redirect

Nginx example:

```nginx
server {
    listen 80;
    server_name app.example.com;
    return 301 https://$host$request_uri;
}
```

## 7.15 Hardened TLS Basics

Minimum recommendations:

- Disable obsolete protocols
- Prefer TLS 1.2 and TLS 1.3
- Use strong ciphers where relevant
- Enable HSTS after validation

Nginx snippet:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

## 7.16 Testing Web Server Configurations

Nginx:

```bash
# nginx -t
# systemctl reload nginx
```

Apache:

```bash
# apachectl configtest
# systemctl reload apache2
```

## 7.17 Common Reverse Proxy Mistakes

- Missing `Host` and forwarding headers
- Not enabling upgrade headers for WebSockets
- Serving large uploads without adjusting body size limits
- Proxying to public interfaces instead of localhost
- Forgetting to validate config before reload

---

# 8. Process Management

## 8.1 Why Process Management Matters

Production processes need:

- Automatic restarts
- Dependency ordering
- Startup on boot
- Log collection
- Resource controls
- Environment injection
- Health and readiness integration

## 8.2 systemd Overview

systemd is the standard service manager on most Linux distributions.

Capabilities:

- Unit-based service definitions
- Restart policies
- Resource limits
- Service dependencies
- Journald integration
- Environment files
- Timers and sockets

## 8.3 Common systemd Commands

```bash
# systemctl daemon-reload
# systemctl enable myapp
# systemctl start myapp
# systemctl restart myapp
# systemctl status myapp
# journalctl -u myapp -f
```

## 8.4 Generic systemd Unit Anatomy

```ini
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp/current
EnvironmentFile=/etc/myapp/myapp.env
ExecStart=/opt/myapp/current/bin/start.sh
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

## 8.5 systemd Service for Java

```ini
[Unit]
Description=Java Service
After=network.target

[Service]
Type=simple
User=javaapp
WorkingDirectory=/opt/javaapp/current
EnvironmentFile=/etc/javaapp/javaapp.env
ExecStart=/usr/bin/java -Xms512m -Xmx1024m -jar /opt/javaapp/current/app.jar
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 8.6 systemd Service for Python

```ini
[Unit]
Description=Python Gunicorn Service
After=network.target

[Service]
User=pythonapp
WorkingDirectory=/opt/pythonapp/current
EnvironmentFile=/etc/pythonapp/pythonapp.env
ExecStart=/opt/pythonapp/venv/bin/gunicorn --bind 127.0.0.1:8000 app:app
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 8.7 systemd Service for Node.js

```ini
[Unit]
Description=Node.js Service
After=network.target

[Service]
User=nodeapp
WorkingDirectory=/opt/nodeapp/current
EnvironmentFile=/etc/nodeapp/nodeapp.env
ExecStart=/usr/bin/node /opt/nodeapp/current/dist/server.js
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 8.8 systemd Service for Go

```ini
[Unit]
Description=Go Binary Service
After=network.target

[Service]
User=goapp
WorkingDirectory=/opt/goapp/current
EnvironmentFile=/etc/goapp/goapp.env
ExecStart=/opt/goapp/current/myapp
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 8.9 systemd Service for .NET

```ini
[Unit]
Description=.NET Service
After=network.target

[Service]
User=dotnetapp
WorkingDirectory=/opt/dotnetapp/current
EnvironmentFile=/etc/dotnetapp/dotnetapp.env
ExecStart=/usr/bin/dotnet /opt/dotnetapp/current/MyApp.dll
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## 8.10 Environment Variables Management

Options:

- Inline `Environment=` directives
- `EnvironmentFile=`
- Wrapper scripts
- Secret managers injecting env vars

Prefer `EnvironmentFile=` for maintainability.

Example:

```ini
EnvironmentFile=/etc/myapp/myapp.env
```

Example file:

```bash
APP_ENV=production
APP_PORT=8080
DATABASE_URL=postgresql://user:pass@db.internal/app
```

## 8.11 `.env` Files

`.env` files are common in development.

Production cautions:

- Restrict file permissions
- Avoid committing to source control
- Prefer centrally managed environment files or secret injection

Recommended permissions:

```bash
# chown root:myapp /etc/myapp/myapp.env
# chmod 640 /etc/myapp/myapp.env
```

## 8.12 Supervisor Overview

Supervisor is another process manager.

It is sometimes used when:

- systemd is unavailable
- Teams already standardized on it
- Multiple simple processes need orchestration in older environments

Install example:

```bash
# apt install -y supervisor
```

Program config example:

```ini
[program:pythonapp]
command=/opt/pythonapp/venv/bin/gunicorn --bind 127.0.0.1:8000 app:app
directory=/opt/pythonapp/current
autostart=true
autorestart=true
user=pythonapp
stdout_logfile=/var/log/pythonapp/stdout.log
stderr_logfile=/var/log/pythonapp/stderr.log
```

## 8.13 PM2 for Node.js

PM2 remains useful for:

- Clustered Node apps
- Developer-friendly rolling restarts
- Quick app dashboards

Example startup integration:

```bash
$ pm2 startup systemd
$ pm2 save
```

## 8.14 Resource Controls with systemd

Useful directives:

| Directive | Purpose |
|---|---|
| `MemoryMax=` | Limit memory usage |
| `CPUQuota=` | Limit CPU usage |
| `TasksMax=` | Cap total tasks |
| `LimitNOFILE=` | Max open files |
| `Restart=` | Restart behavior |

Example:

```ini
MemoryMax=1G
CPUQuota=200%
LimitNOFILE=65535
```

## 8.15 Journald Logging

View logs:

```bash
# journalctl -u myapp -f
```

Show last 100 lines:

```bash
# journalctl -u myapp -n 100 --no-pager
```

## 8.16 Graceful Shutdown Considerations

Applications should handle signals properly.

Important behaviors:

- Stop accepting new requests
- Drain active requests
- Close DB connections cleanly
- Flush logs/metrics if possible

systemd options that help:

```ini
TimeoutStopSec=30
KillSignal=SIGTERM
```

## 8.17 Common Process Management Mistakes

- Running services as root
- Missing restart policy
- Using relative paths in `ExecStart`
- Relying on shell-specific environment assumptions
- Not reloading systemd after unit changes
- Writing secrets directly into unit files with broad permissions

---

# 9. Deployment Strategies

## 9.1 Deployment Goals

A deployment strategy should optimize for:

- Reliability
- Repeatability
- Speed
- Rollback simplicity
- Minimal downtime
- Observability
- Controlled blast radius

## 9.2 Manual Deployment Checklist

Use this even when automation exists.

### Pre-deployment

- Confirm target version and artifact checksum
- Confirm maintenance window or traffic conditions if required
- Confirm database migration plan
- Confirm rollback plan
- Confirm secrets/config available
- Confirm disk space and memory headroom
- Confirm monitoring dashboards are ready

### Deployment

- Upload artifact
- Extract into new release directory
- Apply config symlink or environment file
- Run migrations if safe and required
- Switch symlink or update service target
- Restart or reload process manager
- Validate health checks

### Post-deployment

- Check application logs
- Check error rate
- Check latency and throughput
- Confirm background jobs are running
- Verify key business transactions
- Retain previous release for rollback

## 9.3 Release-Based Directory Strategy

A common Linux deployment layout:

```text
/opt/myapp/
├── current -> /opt/myapp/releases/2025-01-15_120000/
├── releases/
│   ├── 2025-01-10_090000/
│   └── 2025-01-15_120000/
└── shared/
    ├── config/
    ├── logs/
    └── uploads/
```

Benefits:

- Easy rollback
- Immutable release directories
- Clean separation of code and persistent data

## 9.4 Automated Deployment Script Example

```bash
#!/usr/bin/env bash
set -euo pipefail

APP_DIR=/opt/myapp
RELEASE_ID=$(date +%Y-%m-%d_%H%M%S)
RELEASE_DIR="$APP_DIR/releases/$RELEASE_ID"
ARTIFACT=/srv/deployments/myapp.tar.gz

mkdir -p "$RELEASE_DIR"
tar -xzf "$ARTIFACT" -C "$RELEASE_DIR"
ln -sfn "$APP_DIR/shared/config/.env" "$RELEASE_DIR/.env"
ln -sfn "$RELEASE_DIR" "$APP_DIR/current"
systemctl restart myapp
curl -fsS http://127.0.0.1:8080/health
```

## 9.5 Blue-Green Deployments

Blue-green deployment keeps two production environments.

- Blue is active
- Green is idle or staging the new version
- Traffic switches only after validation

Benefits:

- Near-instant rollback
- Minimal downtime
- Clear separation of old and new versions

Trade-offs:

- Requires double infrastructure capacity
- Stateful dependencies still need careful handling

## 9.6 Mermaid Diagram: Blue-Green Deployment Flow

```mermaid
graph TD
    A["Users"] --> B["Load Balancer"]
    B --> C["Blue Environment"]
    B --> D["Green Environment"]
    C --> E["Version N Live"]
    D --> F["Version N+1 Warmed"]
    F --> G["Validation Checks"]
    G --> H["Traffic Switch"]
    H --> D
```

## 9.7 Canary Deployments

Canary deployments expose a new version to a small subset of traffic first.

Example rollout steps:

- 1% traffic to canary
- 5% traffic
- 25% traffic
- 50% traffic
- 100% traffic

Benefits:

- Reduced blast radius
- Real production feedback before full rollout

Requirements:

- Traffic routing controls
- Good metrics and alerting
- Fast rollback process

## 9.8 Rolling Deployments

Rolling deployment updates instances gradually.

Example:

- Remove one instance from rotation
- Deploy new version
- Health check
- Return to rotation
- Repeat for remaining instances

Benefits:

- No need for full duplicate environment
- Lower resource overhead than blue-green

Trade-offs:

- Mixed-version fleet during rollout
- Harder rollbacks if incompatible changes exist

## 9.9 Zero-Downtime Deployments

Zero-downtime goals require:

- Multiple instances
- Load balancer health checks
- Graceful shutdown support
- Backward-compatible DB changes
- Session management considerations

Typical requirements:

- No in-flight request drops
- No forced cache invalidation surprises
- No schema changes that break old or new versions mid-rollout

## 9.10 Strategy Comparison Table

| Strategy | Downtime | Rollback Speed | Cost | Complexity | Best Use |
|---|---|---|---|---|---|
| Manual in-place | Often some | Moderate | Low | Low | Small internal apps |
| Rolling | Low | Moderate | Moderate | Moderate | Multi-instance stateless services |
| Blue-green | Very low | Very fast | High | Moderate | Critical services |
| Canary | Very low | Fast | Moderate to high | High | Large user-facing apps |

## 9.11 Mermaid Diagram: Deployment Strategies Comparison

```mermaid
graph LR
    A["Deployment Strategies"] --> B["Manual In-Place"]
    A --> C["Rolling"]
    A --> D["Blue-Green"]
    A --> E["Canary"]
    B --> F["Low Complexity"]
    C --> G["Low Downtime"]
    D --> H["Fast Rollback"]
    E --> I["Small Blast Radius"]
```

## 9.12 Handling Database Migrations Safely

Migration rules:

- Prefer backward-compatible schema changes first
- Deploy code that works with both old and new schema during transition
- Avoid destructive changes during live cutover
- Separate expand and contract migrations

Pattern:

1. Add new column/table/index
2. Deploy code that writes both paths if needed
3. Backfill data
4. Switch reads
5. Remove old structures later

## 9.13 Deployment Verification Checks

Verify:

- Health endpoint returns success
- Key pages or endpoints respond correctly
- Error rate is normal
- CPU and memory are stable
- Logs show expected startup sequence
- Background jobs still process
- Queue consumers reconnected

## 9.14 Rollback Strategy

Rollback should be documented before each release.

Rollback options:

- Repoint symlink to prior release
- Restart previous container image
- Switch traffic back in blue-green
- Reduce canary to 0%

Rollback verification:

- Health returns normal
- Errors stop increasing
- DB compatibility preserved

## 9.15 Deployment Script Hardening Tips

- Use `set -euo pipefail`
- Validate artifact checksum
- Abort on failed health checks
- Keep previous releases
- Log each step
- Avoid editing live files in place
- Use absolute paths

## 9.16 Sample Rolling Deployment Logic

```bash
for host in app1 app2 app3; do
  echo "Deploying to $host"
  ssh "$host" "sudo systemctl stop myapp && sudo tar -xzf /srv/myapp.tar.gz -C /opt/myapp/current && sudo systemctl start myapp"
  curl -fsS "http://$host:8080/health"
done
```

Real deployments should drain instances from the load balancer first.

## 9.17 Common Deployment Anti-Patterns

- Building directly on production hosts
- Manual edits inside release directories
- No rollback path
- Environment-specific code branches
- Changing config and code without tracking both
- Releasing without health checks or observability

---

# 10. CI/CD Pipeline Integration

## 10.1 CI/CD Goals

CI/CD should enable:

- Fast feedback
- Reproducible builds
- Automated testing
- Consistent artifact creation
- Controlled deployment
- Auditability

## 10.2 Typical Pipeline Stages

1. Checkout
2. Restore dependencies
3. Lint and static analysis
4. Unit/integration tests
5. Build/package
6. Publish artifacts
7. Deploy to staging
8. Verify
9. Promote to production

## 10.3 Mermaid Diagram: CI/CD Pipeline

```mermaid
graph LR
    A["Commit Push"] --> B["CI Checkout"]
    B --> C["Dependency Restore"]
    C --> D["Lint and Test"]
    D --> E["Build Artifact"]
    E --> F["Publish Artifact"]
    F --> G["Deploy Staging"]
    G --> H["Verify"]
    H --> I["Deploy Production"]
```

## 10.4 GitHub Actions Basics

GitHub Actions uses YAML workflows stored in:

```text
.github/workflows/
```

Triggers include:

- `push`
- `pull_request`
- `workflow_dispatch`
- `schedule`

## 10.5 GitHub Actions Workflow for Maven

```yaml
name: Java CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'
          cache: maven
      - name: Build
        run: mvn -B -ntp clean verify
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: java-app
          path: target/*.jar
```

## 10.6 GitHub Actions Workflow for Python

```yaml
name: Python CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - name: Install
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      - name: Test
        run: pytest
      - name: Build package
        run: python -m build
```

## 10.7 GitHub Actions Workflow for Node.js

```yaml
name: Node CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - name: Install
        run: npm ci
      - name: Test
        run: npm test
      - name: Build
        run: npm run build
```

## 10.8 GitHub Actions Workflow for Go

```yaml
name: Go CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.22'
      - name: Test
        run: go test ./...
      - name: Build
        run: go build -o myapp ./cmd/myapp
```

## 10.9 GitHub Actions Workflow for .NET

```yaml
name: Dotnet CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      - name: Restore
        run: dotnet restore
      - name: Build
        run: dotnet build -c Release --no-restore
      - name: Test
        run: dotnet test -c Release --no-build
      - name: Publish
        run: dotnet publish -c Release -o publish --no-build
```

## 10.10 Deployment Job with SSH Example

```yaml
name: Deploy

on:
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Copy artifact
        run: |
          scp build/myapp.tar.gz deploy@app.example.com:/srv/deployments/
      - name: Deploy remotely
        run: |
          ssh deploy@app.example.com 'sudo /usr/local/bin/deploy-myapp.sh'
```

Use secrets for:

- SSH keys
- Repository credentials
- Cloud credentials
- Artifact repository tokens

## 10.11 Jenkins Pipeline Basics

A `Jenkinsfile` defines the pipeline.

Declarative example for Java:

```groovy
pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'mvn -B -ntp clean verify'
      }
    }

    stage('Archive') {
      steps {
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }
  }
}
```

## 10.12 Jenkins Pipeline for Node.js

```groovy
pipeline {
  agent any

  stages {
    stage('Install') {
      steps {
        sh 'npm ci'
      }
    }

    stage('Test') {
      steps {
        sh 'npm test'
      }
    }

    stage('Build') {
      steps {
        sh 'npm run build'
      }
    }
  }
}
```

## 10.13 GitLab CI Example

```yaml
stages:
  - test
  - build
  - deploy

python-test:
  stage: test
  image: python:3.12
  script:
    - pip install -r requirements.txt
    - pytest

python-build:
  stage: build
  image: python:3.12
  script:
    - pip install build
    - python -m build
  artifacts:
    paths:
      - dist/

python-deploy:
  stage: deploy
  script:
    - scp dist/*.whl deploy@app.example.com:/srv/deployments/
```

## 10.14 Artifact Management

Artifact repositories include:

- Nexus
- Artifactory
- GitHub Packages
- GitLab Package Registry
- S3-compatible storage

Store:

- JAR/WAR files
- Python wheels
- npm packages
- Binary tarballs
- Docker images

Best practices:

- Tag artifacts with commit SHA and version
- Store metadata and checksums
- Separate snapshots from releases
- Retain rollback candidates

## 10.15 Promotion vs Rebuild

Prefer promotion.

That means:

- Build once in CI
- Test the built artifact
- Promote the same artifact to higher environments

Do not rebuild from the same commit for each environment unless policy demands it.

## 10.16 Secrets in CI/CD

Never store plaintext secrets in:

- Git repositories
- Build scripts
- Dockerfiles
- Public logs

Use:

- GitHub Actions secrets
- Jenkins credentials store
- GitLab protected variables
- Vault or cloud secret stores

## 10.17 CI/CD Pipeline Hardening

- Use ephemeral runners where possible
- Pin action versions
- Restrict deployment environments
- Require approvals for production
- Sign or verify artifacts
- Limit secret scope
- Audit who deployed what and when

## 10.18 Deployment Gates

Useful production gates:

- Manual approval
- Change window validation
- Automated smoke tests
- Error budget checks
- Database migration completion checks

## 10.19 Common CI/CD Mistakes

- Deploying from a developer workstation instead of CI
- Not caching dependencies wisely
- Leaking secrets in logs
- Building different artifacts per environment unintentionally
- Missing rollback automation
- No post-deploy verification step

---

# 11. Application Monitoring

## 11.1 Monitoring Objectives

Monitoring should tell you:

- Is the service up?
- Is it healthy?
- Is it fast enough?
- Is it erroring?
- Is it resource-constrained?
- Are dependencies failing?

## 11.2 Health Check Endpoints

Expose simple health endpoints.

Typical paths:

- `/health`
- `/ready`
- `/live`

Design guidance:

- Liveness should be cheap and minimal
- Readiness may validate critical dependencies
- Avoid overly expensive health logic

Example response:

```json
{
  "status": "ok",
  "version": "1.4.2",
  "commit": "abc1234"
}
```

## 11.3 Metrics to Track

Core metrics:

- Request rate
- Error rate
- Response latency
- CPU usage
- Memory usage
- File descriptor usage
- Queue depth
- Database connection pool usage

## 11.4 Prometheus

Prometheus scrapes metrics endpoints.

Typical endpoint:

```text
/metrics
```

Best for:

- Time-series metrics
- Alerting rules
- Service dashboards

## 11.5 Grafana

Grafana visualizes metrics from Prometheus and other sources.

Useful dashboard panels:

- HTTP status rates
- P95 latency
- JVM heap usage
- Node event loop lag
- Go GC pause duration
- Python worker restarts
- .NET request duration

## 11.6 APM Tools

Commercial APM tools include:

- New Relic
- Datadog
- Dynatrace
- AppDynamics

They often provide:

- Distributed tracing
- Service maps
- Error monitoring
- Deployment markers
- Runtime-specific insights

## 11.7 Prometheus vs APM

| Capability | Prometheus | APM Platforms |
|---|---|---|
| Metrics | Excellent | Excellent |
| Tracing | Limited unless paired | Strong |
| Logs | External integration | Often integrated |
| Cost | Often lower self-hosted | Usually commercial |
| Depth by runtime | Depends on exporters | Often richer |

## 11.8 Log Management

Logs should be:

- Structured where possible
- Centralized
- Searchable
- Retained according to policy
- Correlated with requests and traces

Popular stacks:

- ELK/Elastic Stack
- OpenSearch
- Loki + Grafana
- Splunk

## 11.9 Logging Best Practices

Include fields such as:

- Timestamp
- Level
- Service name
- Hostname
- Environment
- Request ID
- Trace ID
- User ID where appropriate and compliant

## 11.10 Error Tracking with Sentry

Sentry is useful for:

- Capturing exceptions
- Grouping errors
- Tracking releases
- Alerting on regressions

Typical usage:

- Include release version
- Include environment name
- Avoid sending sensitive data

## 11.11 JVM Monitoring

Key JVM metrics:

- Heap used/max
- GC pause time
- Thread count
- Class loading
- CPU usage
- Request latency

Useful tools:

- JMX exporters
- Micrometer
- Java Flight Recorder

## 11.12 Python Monitoring

Useful metrics:

- Gunicorn worker count
- Worker restarts
- Response time
- Queue time
- Memory growth over time
- Dependency latency

## 11.13 Node.js Monitoring

Useful metrics:

- Event loop lag
- Heap usage
- Garbage collection pauses
- Open handles
- Request latency
- WebSocket connection count

## 11.14 Go Monitoring

Useful metrics:

- Goroutine count
- GC duration
- Heap allocation
- Request latency
- Error rate
- Dependency latency

## 11.15 .NET Monitoring

Useful metrics:

- Request duration
- GC collections
- Heap size
- Thread pool saturation
- Exceptions per minute

## 11.16 Alerting Recommendations

Alert on symptoms and saturation, not only uptime.

Examples:

- 5xx rate above threshold
- P95 latency above SLO threshold
- Memory near limit with upward trend
- Connection pool exhaustion approaching critical levels

## 11.17 Deployment Markers

Record deployments in monitoring tools.

This helps correlate:

- Error spikes
- Latency regressions
- Resource behavior changes

## 11.18 Common Monitoring Gaps

- No readiness checks
- No application-level metrics
- No version metadata in health endpoints
- Logs without correlation IDs
- Alerts that are too noisy or too weak

---

# 12. Troubleshooting Applications

## 12.1 Troubleshooting Workflow

A structured workflow prevents guesswork.

Steps:

1. Identify impact and symptoms
2. Confirm recent changes
3. Check process state
4. Check logs
5. Check system resources
6. Check dependencies
7. Reproduce or isolate
8. Mitigate safely
9. Document findings

## 12.2 Application Won't Start

Common causes:

- Port conflict
- Missing runtime or library
- Bad environment variable
- Broken config syntax
- Permission issue
- Missing secret file
- Failed migration or dependency startup

## 12.3 Check Service Status

```bash
# systemctl status myapp
# journalctl -u myapp -n 100 --no-pager
```

## 12.4 Detect Port Conflicts

Use `ss` or `lsof`.

```bash
# ss -ltnp | grep :8080
# lsof -i :8080
```

If another process owns the port, decide whether:

- The wrong service is running
- The config has the wrong port
- A stale process needs controlled shutdown

## 12.5 Missing Dependencies

Examples:

- Shared libraries missing for native apps
- Wrong JDK runtime for Java apps
- Missing Python module in venv
- Node module not installed or wrong build output
- Missing .NET runtime for framework-dependent deployment

Diagnostics:

```bash
$ ldd ./mybinary
$ java -version
$ /opt/pythonapp/venv/bin/python -c "import flask"
$ node -v
$ dotnet --info
```

## 12.6 Permission Issues

Check:

- File ownership
- Execute bits
- SELinux/AppArmor context where applicable
- Directory traversal permissions

Commands:

```bash
$ ls -l /opt/myapp/current
$ namei -l /opt/myapp/current/app.jar
```

## 12.7 Memory Leaks

Symptoms:

- Gradual RSS growth
- OOM kills
- Increased GC pressure
- Restart loops after sustained load

By runtime:

- JVM: heap dump, GC logs, JFR
- Python: object retention, worker growth, native extension leaks
- Node.js: heap snapshots, event loop lag, retaining closures
- Go: pprof heap profiles
- .NET: memory dumps, counters, tracing

## 12.8 OOM Diagnostics on Linux

Check kernel logs:

```bash
# dmesg | grep -i -E 'killed process|out of memory'
```

Check service logs around the time of failure.

## 12.9 Connection Pool Exhaustion

Symptoms:

- Timeouts under load
- Errors acquiring DB connections
- Increased latency on dependent operations

Common causes:

- Leaked connections
- Pool too small
- DB too slow
- Long transactions
- Traffic spike

What to inspect:

- Pool metrics
- DB slow queries
- Application code paths not returning connections
- Thread or worker counts relative to pool size

## 12.10 Slow Response Times

Check:

- Application logs
- Reverse proxy timing fields
- DB latency
- External API latency
- CPU saturation
- Disk I/O wait
- GC pauses

Commands:

```bash
$ top
$ vmstat 1 5
$ iostat -xz 1 5
$ free -m
```

## 12.11 `strace`

`strace` traces system calls.

Example:

```bash
# strace -p 12345
```

Use cases:

- Hanging process
- Unexpected file lookups
- Permission errors
- Network call diagnosis

Start a program under strace:

```bash
# strace -f -o strace.log ./myapp
```

## 12.12 `lsof`

`lsof` lists open files and sockets.

Examples:

```bash
# lsof -p 12345
# lsof -iTCP -sTCP:LISTEN -P -n
```

Use cases:

- Identify open ports
- Inspect leaked file descriptors
- Confirm log files or sockets in use

## 12.13 `tcpdump`

`tcpdump` captures packets.

Example:

```bash
# tcpdump -i any host 10.0.0.20 and port 5432
```

Use with care in production.

Use cases:

- Verify traffic reaches a dependency
- Inspect TLS handshake attempts
- Validate load balancer routing

## 12.14 `curl` for Local Health Testing

```bash
$ curl -v http://127.0.0.1:8080/health
$ curl -vk https://app.example.com/health
```

## 12.15 DNS and Connectivity Checks

Commands:

```bash
$ getent hosts db.internal
$ dig app.example.com
$ ping -c 3 db.internal
$ nc -vz db.internal 5432
```

## 12.16 Reverse Proxy Troubleshooting

Check:

- Nginx or Apache config syntax
- Upstream port availability
- Proxy headers
- TLS cert validity
- Body size limits
- WebSocket upgrade config

Commands:

```bash
# nginx -t
# journalctl -u nginx -n 100 --no-pager
```

## 12.17 Java Troubleshooting Tips

- Inspect GC logs
- Verify heap settings
- Confirm JDK version
- Use `jcmd`, `jmap`, `jstack` if available
- Validate truststore and certificate config for outbound TLS

## 12.18 Python Troubleshooting Tips

- Validate venv interpreter path
- Check Gunicorn worker exits
- Confirm package installation in the correct environment
- Verify `DJANGO_SETTINGS_MODULE` or Flask config vars

## 12.19 Node.js Troubleshooting Tips

- Confirm compiled output exists
- Check `NODE_ENV`
- Review uncaught exception handling
- Inspect heap with the inspector when needed
- Verify WebSocket reverse proxy behavior

## 12.20 Go Troubleshooting Tips

- Expose pprof in secured environments
- Inspect goroutine leaks
- Validate config and env parsing at startup
- Confirm static binary expectations vs CGO use

## 12.21 .NET Troubleshooting Tips

- Check runtime version availability
- Inspect Kestrel bind settings
- Validate forwarded headers middleware configuration
- Use `dotnet-counters` and related diagnostics tools where available

## 12.22 Incident Triage Table

| Symptom | Likely Areas | First Checks |
|---|---|---|
| Service won't start | Config, deps, permissions | `systemctl status`, logs |
| 502 from proxy | Upstream down, bad port | `curl localhost`, Nginx logs |
| High latency | DB, CPU, GC, external APIs | metrics, `top`, app logs |
| OOM restarts | Leak, too-small limits | kernel logs, memory graphs |
| Connection errors | DNS, firewall, pool, creds | `nc`, `dig`, config |

## 12.23 Post-Incident Review Topics

Capture:

- Root cause
- Trigger
- Detection method
- Time to mitigate
- Time to resolve
- Gaps in automation or alerting
- Action items to prevent recurrence

---

# 13. Configuration Management

## 13.1 Why Configuration Management Matters

Configuration separates deployable code from environment-specific behavior.

Examples of environment-specific settings:

- Database endpoints
- API credentials
- Feature flags
- Log levels
- Cache hosts
- Queue names
- Port bindings

## 13.2 Environment-Based Configuration

Typical environments:

- Development
- Test
- Staging
- Production

Best practice:

- Same code artifact across environments
- Different configuration injected per environment

## 13.3 Configuration Approaches

Common options:

- Environment variables
- YAML/JSON/TOML config files
- `.properties` files
- systemd `EnvironmentFile`
- Secret management systems
- Service discovery/config platforms

## 13.4 Configuration File Management

Recommended pattern:

```text
/opt/myapp/
├── current
├── releases/
└── shared/
    └── config/
        ├── app.env
        ├── application-prod.yml
        └── logging.yml
```

Keep mutable config in `shared/config/`.

Release directories should remain immutable.

## 13.5 Environment Variables vs Config Files

| Approach | Best For | Pros | Cons |
|---|---|---|---|
| Environment variables | Simple app settings, secrets injection | Standard, easy to automate | Can become unwieldy at scale |
| Config files | Larger structured configuration | Easier to read, version, template | Need secure distribution |
| Mixed model | Most real systems | Balanced flexibility | Needs discipline |

## 13.6 Secrets Management

Secrets include:

- Database passwords
- API tokens
- Private keys
- OAuth client secrets
- Encryption keys

Avoid:

- Hardcoding secrets in source
- Baking secrets into immutable artifacts
- Storing secrets in world-readable files

Prefer:

- Vault
- AWS Secrets Manager
- Azure Key Vault
- GCP Secret Manager
- Kubernetes secrets with appropriate safeguards
- systemd EnvironmentFile with restricted permissions for simpler setups

## 13.7 Secret Injection Patterns

Patterns include:

- Env vars populated at service start
- Mounted files with strict permissions
- Sidecar or agent injection
- App runtime fetch with caching

## 13.8 Feature Flags

Feature flags enable controlled release behavior.

Use cases:

- Gradual rollout
- Emergency disablement
- A/B testing
- Decoupling deployment from release

Best practices:

- Name flags clearly
- Remove stale flags
- Monitor flag impact
- Separate operational kill switches from product experiments

## 13.9 Templating Configuration

Tools may include:

- `envsubst`
- Ansible templates
- Helm charts
- Jinja2 templates
- CI/CD variable substitution

Example with `envsubst`:

```bash
$ export APP_PORT=8080
$ envsubst < app.env.template > app.env
```

## 13.10 Config Validation

Validate before deployment.

Examples:

- Syntax validation for YAML/JSON
- Application startup dry-runs where possible
- Required variable checks in entrypoint scripts

Simple Bash pattern:

```bash
: "${DATABASE_URL:?DATABASE_URL is required}"
: "${APP_ENV:?APP_ENV is required}"
```

## 13.11 Multi-Environment Config Example

Development:

```bash
APP_ENV=development
LOG_LEVEL=debug
DATABASE_URL=postgresql://dev:dev@localhost/devdb
```

Staging:

```bash
APP_ENV=staging
LOG_LEVEL=info
DATABASE_URL=postgresql://stage:secret@staging-db.internal/stagedb
```

Production:

```bash
APP_ENV=production
LOG_LEVEL=warn
DATABASE_URL=postgresql://prod:secret@prod-db.internal/proddb
```

## 13.12 Configuration Drift Prevention

Prevent drift by:

- Storing config templates in version control
- Managing actual secret values securely outside git
- Automating host configuration
- Auditing live config regularly
- Avoiding manual hotfix changes on servers

## 13.13 Configuration Reloading

Some apps can reload config without full restart.

Patterns:

- `SIGHUP`-triggered reloads
- Reverse proxy reloads
- Dynamic config services

If live reload is unsupported, prefer controlled restart plus health checks.

## 13.14 Common Configuration Mistakes

- Mixing dev and prod credentials
- Using relative file paths
- Putting mutable config inside release directories
- Not documenting required variables
- No schema or validation for config structure
- Leaving stale feature flags enabled forever

## 13.15 Production Configuration Checklist

- Required variables documented
- Secrets stored securely
- Config separated from artifact
- File permissions restricted
- Environment naming consistent
- Validation performed before rollout
- Rollback config available

---

# Appendix A. Common Linux Commands for Build and Deployment

## A.1 Package Management

Ubuntu/Debian:

```bash
# apt update
# apt install -y nginx git curl unzip
```

RHEL-compatible:

```bash
# dnf install -y nginx git curl unzip
```

## A.2 User and Directory Setup

Create service user:

```bash
# useradd --system --home /opt/myapp --shell /usr/sbin/nologin myapp
```

Create directories:

```bash
# mkdir -p /opt/myapp/releases /opt/myapp/shared/logs /etc/myapp
# chown -R myapp:myapp /opt/myapp
```

## A.3 Permission Examples

```bash
# chown root:myapp /etc/myapp/myapp.env
# chmod 640 /etc/myapp/myapp.env
# chmod 755 /opt/myapp/current/myapp
```

## A.4 Tar Packaging

Create archive:

```bash
$ tar -czf myapp.tar.gz dist/ config/
```

Extract archive:

```bash
# tar -xzf myapp.tar.gz -C /opt/myapp/releases/2025-01-15_120000
```

## A.5 Checksum Verification

Generate checksum:

```bash
$ sha256sum myapp.tar.gz
```

Verify checksum:

```bash
$ sha256sum -c myapp.tar.gz.sha256
```

---

# Appendix B. Example End-to-End Deployment Flows

## B.1 Java Spring Boot Flow

1. Build JAR with Maven.
2. Upload JAR or tarball artifact.
3. Place release under `/opt/springapp/releases/`.
4. Link shared config.
5. Restart systemd service.
6. Validate `/actuator/health`.
7. Monitor GC, memory, and error rate.

## B.2 Python Django Flow

1. Build or package app source.
2. Create or update venv.
3. Install pinned dependencies.
4. Run migrations.
5. Run `collectstatic`.
6. Restart Gunicorn service.
7. Reload Nginx if config changed.
8. Validate health and admin login.

## B.3 Node.js API Flow

1. `npm ci`
2. `npm test`
3. `npm run build`
4. Package `dist/` plus production dependencies.
5. Deploy new release.
6. Restart systemd or PM2.
7. Validate `/health` and WebSocket if used.

## B.4 Go API Flow

1. `go test ./...`
2. `CGO_ENABLED=0 go build -o myapp ./cmd/myapp`
3. Package binary and config template.
4. Deploy to release directory.
5. Restart systemd.
6. Validate `/health`.

## B.5 ASP.NET Core Flow

1. `dotnet restore`
2. `dotnet test`
3. `dotnet publish -c Release -o publish`
4. Package publish output.
5. Deploy new release.
6. Restart systemd.
7. Validate readiness endpoint.

---

# Appendix C. Reference Tables

## C.1 Default Local Bind Ports in Examples

| Stack | Example Port |
|---|---|
| Java | `8080` |
| Python Gunicorn | `8000` |
| Node.js | `3000` |
| Go | `8080` |
| ASP.NET Core | `5000` |

## C.2 Common Artifact Types

| Ecosystem | Artifact |
|---|---|
| C/C++ | Binary, `.so`, `.a` |
| Java | `.jar`, `.war`, `.ear` |
| Python | `.whl`, `.tar.gz` |
| Node.js | Bundled `dist/`, package tarball |
| Go | Native binary |
| .NET | Published DLL set or self-contained output |

## C.3 Common Health Endpoint Names

| Name | Typical Purpose |
|---|---|
| `/health` | Basic health |
| `/live` | Liveness |
| `/ready` | Readiness |
| `/metrics` | Metrics scraping |

---

# Appendix D. Operational Principles

## D.1 Build Once, Deploy Many

This principle reduces drift.

It improves traceability.

It simplifies rollback.

## D.2 Prefer Immutable Releases

Do not edit live code in place.

Create a new release directory.

Switch traffic or symlink.

Retain the previous release.

## D.3 Separate Code, Config, and Data

Keep code immutable.

Keep config externalized.

Keep persistent data outside releases.

## D.4 Design for Restartability

Every service should be restart-safe.

That means:

- Idempotent startup
- Graceful shutdown
- Fast readiness reporting
- Dependency retry logic

## D.5 Observe Before You Need It

Monitoring and logs should exist before incidents.

Add:

- Health endpoints
- Metrics
- Structured logs
- Deployment version visibility

---

# Appendix E. Detailed Command Reference by Ecosystem

## E.1 Native Build Tools

```bash
$ gcc --version
$ g++ --version
$ make -j$(nproc)
$ cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
$ cmake --build build --parallel
```

## E.2 Java Commands

```bash
$ java -version
$ javac -version
$ mvn -B -ntp clean verify
$ ./gradlew clean build
$ java -jar app.jar
```

## E.3 Python Commands

```bash
$ python3 -m venv .venv
$ source .venv/bin/activate
$ pip install -r requirements.txt
$ python -m build
$ gunicorn --bind 127.0.0.1:8000 app:app
```

## E.4 Node.js Commands

```bash
$ node -v
$ npm ci
$ npm test
$ npm run build
$ pm2 start dist/server.js --name mynodeapp
```

## E.5 Go Commands

```bash
$ go version
$ go mod tidy
$ go test ./...
$ CGO_ENABLED=0 go build -o myapp ./cmd/myapp
```

## E.6 .NET Commands

```bash
$ dotnet --info
$ dotnet restore
$ dotnet test -c Release
$ dotnet publish -c Release -o publish
```

---

# Appendix F. Production Readiness Checklists

## F.1 Universal Build Checklist

- Source from trusted repository
- Pinned dependencies
- CI reproducibility
- Tests green
- Artifact versioned
- Artifact checksum recorded
- SBOM generated if required

## F.2 Universal Deployment Checklist

- Target host prepared
- Service user exists
- Config present
- Secret access working
- Health checks defined
- Monitoring connected
- Rollback plan ready
- Last known good release retained

## F.3 Universal Runtime Checklist

- Runs as non-root
- Reverse proxy secured
- TLS enabled
- Logs centralized
- Metrics exposed
- Restart policy set
- Resource limits reviewed
- Alerting configured

---

# Appendix G. End-to-End Sample Layouts

## G.1 Java Layout

```text
/opt/javaapp/
├── current
├── releases/
└── shared/
    ├── config/
    └── logs/
```

## G.2 Python Layout

```text
/opt/pythonapp/
├── current
├── releases/
├── shared/
│   ├── static/
│   └── logs/
└── venv/
```

## G.3 Node Layout

```text
/opt/nodeapp/
├── current
├── releases/
└── shared/
    └── logs/
```

## G.4 Go Layout

```text
/opt/goapp/
├── current
├── releases/
└── shared/
    └── config/
```

## G.5 .NET Layout

```text
/opt/dotnetapp/
├── current
├── releases/
└── shared/
    ├── config/
    └── logs/
```

---

# Appendix H. Practical Smoke Tests

## H.1 HTTP Health Check

```bash
$ curl -fsS http://127.0.0.1:8080/health
```

## H.2 TLS Check

```bash
$ curl -vk https://app.example.com/health
```

## H.3 Port Listening Check

```bash
# ss -ltnp | grep -E ':3000|:5000|:8000|:8080'
```

## H.4 Process Check

```bash
# systemctl is-active myapp
```

## H.5 Nginx Config Check

```bash
# nginx -t
```

---

# Appendix I. Sample Deployment Runbooks

## I.1 Minimal Runbook Template

### Objective

Deploy version `X.Y.Z` of the application safely to production.

### Preconditions

- Artifact available
- Monitoring dashboards open
- Rollback artifact available
- Database migration approved

### Steps

1. Announce deployment start.
2. Confirm traffic and alert state.
3. Deploy to target.
4. Validate health.
5. Validate critical transactions.
6. Watch metrics for 15 minutes.
7. Mark deployment complete.

### Rollback

1. Revert to previous release.
2. Restart service or switch traffic.
3. Verify health and error rates.
4. Document incident.

---

# Appendix J. Security Considerations

## J.1 Principle of Least Privilege

Applications should:

- Run as dedicated users
- Have only required file permissions
- Bind to localhost unless public access is required
- Access only required secrets

## J.2 Artifact Integrity

Protect against tampering by:

- Using checksums
- Using signed packages where applicable
- Restricting artifact repository access

## J.3 Secret Hygiene

- Rotate secrets periodically
- Scope credentials narrowly
- Audit secret access
- Never log secret values

## J.4 Network Exposure

Only expose:

- Reverse proxy public ports
- Required internal service ports
- No unnecessary debug ports

## J.5 Log Sanitization

Ensure logs do not leak:

- Passwords
- Tokens
- Session IDs
- PII beyond approved policies

---

# Appendix K. Final Recommendations

## K.1 If You Need the Simplest Linux Deployment Path

A practical default stack is:

- Build in CI
- Package artifact
- Deploy to `/opt/<app>/releases/`
- Use `current` symlink
- Run under systemd
- Front with Nginx
- Expose `/health`
- Centralize logs and metrics

## K.2 If You Need Higher Safety

Add:

- Blue-green or rolling deployment
- Automated smoke tests
- Artifact signing
- Stronger secret management
- SLO-based alerting
- Deployment approval gates

## K.3 If You Need Better Operability

Standardize:

- Directory layouts
- systemd unit conventions
- Reverse proxy templates
- Health endpoint names
- Logging fields
- CI/CD stages
- Rollback procedures

## K.4 If You Need Better Scale

Adopt:

- Load balancing
- Horizontal scaling
- Stateless service design
- External session stores
- Queue-driven async processing
- Gradual rollout strategies

## K.5 Closing Summary

Reliable Linux application delivery is not only about building code.

It is about combining:

- Deterministic builds
- Clean artifact packaging
- Secure configuration handling
- Predictable service management
- Safe deployment strategies
- Effective monitoring
- Fast troubleshooting

When these practices are combined, teams get:

- Faster releases
- Safer changes
- Easier rollbacks
- Better uptime
- More confidence in production

---

# Supplemental Line Expansion Section

The following compact reminders are included intentionally to make this document a practical long-form reference.

## Quick Reminder 1

Use absolute paths in automation.

## Quick Reminder 2

Do not run production services as root.

## Quick Reminder 3

Prefer immutable artifacts.

## Quick Reminder 4

Pin versions in CI.

## Quick Reminder 5

Expose health endpoints.

## Quick Reminder 6

Monitor error rate after each deployment.

## Quick Reminder 7

Keep rollback artifacts ready.

## Quick Reminder 8

Validate Nginx config before reload.

## Quick Reminder 9

Use `npm ci` in CI.

## Quick Reminder 10

Use virtual environments for Python.

## Quick Reminder 11

Use the Gradle wrapper when provided.

## Quick Reminder 12

Use `go mod tidy` carefully and review changes.

## Quick Reminder 13

Use `dotnet publish` for deployment output.

## Quick Reminder 14

Set JVM heap intentionally.

## Quick Reminder 15

Verify proxy headers end to end.

## Quick Reminder 16

Keep config outside release directories.

## Quick Reminder 17

Restrict permissions on environment files.

## Quick Reminder 18

Capture deployment version in logs.

## Quick Reminder 19

Store checksums with artifacts.

## Quick Reminder 20

Prefer systemd for standard Linux operations.

## Quick Reminder 21

Use readiness checks for production rollouts.

## Quick Reminder 22

Keep static assets on the proxy when possible.

## Quick Reminder 23

Record recent changes during incident triage.

## Quick Reminder 24

Measure before tuning.

## Quick Reminder 25

Favor promotion over rebuild.

## Quick Reminder 26

Do not bake secrets into images or tarballs.

## Quick Reminder 27

Use service users with nologin shells.

## Quick Reminder 28

Retain a few previous releases.

## Quick Reminder 29

Prefer structured logs.

## Quick Reminder 30

Test rollback procedures before emergencies.

## Quick Reminder 31

Use `journalctl -u <service> -f` for live troubleshooting.

## Quick Reminder 32

Check `ss -ltnp` for port ownership.

## Quick Reminder 33

Check `dmesg` for OOM kills.

## Quick Reminder 34

Use `curl -fsS` for smoke tests.

## Quick Reminder 35

Validate TLS renewal automation.

## Quick Reminder 36

Avoid mutable app state on local disk when scaling.

## Quick Reminder 37

Prefer backward-compatible DB changes.

## Quick Reminder 38

Use canaries when blast radius matters.

## Quick Reminder 39

Use blue-green when rollback speed matters most.

## Quick Reminder 40

Use rolling updates for stateless multi-instance services.

## Quick Reminder 41

Keep deployment scripts idempotent.

## Quick Reminder 42

Avoid manual server edits outside emergencies.

## Quick Reminder 43

Document required environment variables.

## Quick Reminder 44

Validate runtime versions on the target host.

## Quick Reminder 45

Track dependency caches in CI wisely.

## Quick Reminder 46

Size worker counts using load testing, not guesswork.

## Quick Reminder 47

Avoid overcommitting CPU with too many workers.

## Quick Reminder 48

Check file descriptor limits for high-traffic services.

## Quick Reminder 49

Use non-interactive CI-friendly commands.

## Quick Reminder 50

Treat deployment logs as operational evidence.
