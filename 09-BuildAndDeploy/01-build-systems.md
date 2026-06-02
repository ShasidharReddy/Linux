# Build Systems

[Back to guide index](README.md)

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
