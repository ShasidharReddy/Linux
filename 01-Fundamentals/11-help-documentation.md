# Help and Documentation

## 11. Help and Documentation
### 11.1 Why Documentation Skills Matter
Linux has enormous built-in documentation.

The best administrators are not the ones who memorize everything.

They are the ones who know how to find accurate information quickly.

### 11.2 `man`
#### Purpose

Display manual pages.

#### Examples

```bash
man ls
man 5 passwd
man 8 useradd
```

#### Manual Sections

| Section | Content |
|---|---|
| 1 | User commands |
| 2 | System calls |
| 3 | Library functions |
| 4 | Devices and special files |
| 5 | File formats and config files |
| 6 | Games |
| 7 | Miscellaneous |
| 8 | Administrative commands |

#### Useful Searches Inside `man`

- `/pattern` to search
- `n` for next match
- `q` to quit

### 11.3 `info`
#### Purpose

Read GNU info documentation.

#### Example

```bash
info coreutils
info bash
```

#### Notes

Some GNU tools provide richer detail in `info` than in `man`.

### 11.4 `whatis`
#### Purpose

Show a one-line manual page description.

#### Examples

```bash
whatis ls
whatis passwd
```

### 11.5 `apropos`
#### Purpose

Search manual page names and descriptions.

#### Examples

```bash
apropos password
apropos archive
apropos permissions
```

#### Notes

Use `apropos` when you know the concept but not the command name.

### 11.6 `--help`
#### Purpose

Display quick command usage.

#### Examples

```bash
ls --help
find --help
tar --help
```

#### Notes

`--help` is often the fastest way to confirm syntax.

### 11.7 `/usr/share/doc`
Many packages install documentation under `/usr/share/doc`.

Examples:

```bash
ls /usr/share/doc
ls /usr/share/doc/bash
ls /usr/share/doc/openssh-client
```

Documentation there may include:

- changelogs
- examples
- README files
- package-specific notes

### 11.8 Other Helpful Documentation Sources on the System
- shell built-ins via `help` in Bash
- config examples under package directories
- service unit details via `systemctl cat`
- logs via `journalctl`

Examples:

```bash
help cd
systemctl cat sshd
journalctl -u sshd
```

### 11.9 Learning Workflow Example
When you encounter a new command:

1. run `command --help`
2. read `man command`
3. search `apropos` if needed
4. inspect examples in `/usr/share/doc`
5. test in a safe directory

### 11.10 Troubleshooting with Documentation
Example process for an unknown option error:

```bash
tar --help | less
man tar
apropos tar
info tar
```

### 11.11 Documentation Best Practices
- read the examples section first
- note whether a command is in section 1, 5, or 8
- prefer official manual pages over random copy-pasted internet snippets
- test commands on harmless files before production use
- keep personal notes of commands you use often

### 11.12 Quick Reference Table
| Tool | Best Use |
|---|---|
| `man` | Full manual pages |
| `info` | GNU detailed documentation |
| `whatis` | One-line summary |
| `apropos` | Search by topic |
| `--help` | Quick syntax reminder |
| `/usr/share/doc` | Package docs and examples |
| `help` | Shell built-ins |

> Tip:
> Documentation lookup is not a sign of weakness.
> It is standard professional Linux practice.

---

## Final Review Checklist

Use this checklist to verify your Linux fundamentals are solid.

- I know what Linux is and how it differs from a distribution.
- I can name major distro families and package managers.
- I understand Linux architecture layers.
- I can describe the boot process from firmware to login.
- I know the purpose of major FHS directories.
- I can identify Linux file types with `ls -l`.
- I can navigate and manage files with core shell commands.
- I understand permissions, ownership, and numeric modes.
- I can manage users, groups, and sudo safely.
- I can redirect stdin, stdout, and stderr.
- I can build useful pipelines.
- I can search and transform text with classic Unix tools.
- I can archive and compress data correctly.
- I know how to use `man`, `apropos`, and `--help`.

---

## Practice Lab Ideas

### Lab 1: Navigation and Files

- Create a test directory tree.
- Move between directories with `cd`.
- Create files with `touch`.
- Copy and rename them with `cp` and `mv`.
- Remove them safely.

### Lab 2: Permissions

- Create a private directory.
- Change modes with `chmod`.
- Observe the difference between `644`, `600`, `755`, and `700`.
- Test shared group access.

### Lab 3: Users and Groups

- Create a test user.
- Add the user to a group.
- Use `id` and `groups` to verify membership.
- Test `sudo -l`.

### Lab 4: Redirection and Pipelines

- Redirect `ls` output to a file.
- Append command results with `>>`.
- Capture errors using `2>`.
- Use `tee` for a command log.
- Build a `grep | sort | uniq` pipeline.

### Lab 5: Text Processing

- Search a config file with `grep`.
- Replace values with `sed`.
- Extract columns with `awk` and `cut`.
- Compare file versions with `diff`.

### Lab 6: Archives

- Create a tar archive of a sample project.
- Compress it with gzip.
- List its contents.
- Extract it into a restore directory.

---

## Closing Notes

Linux fundamentals are not about memorizing every flag.

They are about understanding the model:

- files and paths
- users and permissions
- processes and services
- text streams and automation
- standard locations and documentation

Once these foundations are strong, advanced topics such as process control, networking, storage administration, shell scripting, systemd, containers, security hardening, and performance tuning become much easier.

Keep practicing.

Read the manual pages.

Use a lab environment.

Build confidence one command at a time.
