# Java and JVM Applications

[Back to guide index](README.md)

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
