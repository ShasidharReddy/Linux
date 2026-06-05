# 7. Basic Commands

This chapter introduces `pwd` (Print Working Directory), `ls` (List), `cd` (Change Directory), `mkdir` (Make Directory), `cp` (Copy), `mv` (Move), `rm` (Remove), `cat` (Concatenate), `vi` (Visual Interface), `vim` (Vi IMproved), `grep` (Global Regular Expression Print), `awk` (Aho, Weinberger, Kernighan), `sed` (Stream Editor), `chmod` (Change Mode), `chown` (Change Owner), `df` (Disk Free), `du` (Disk Usage), `ps` (Process Status), `top` (Table of Processes), `lsof` (List Open Files), and `ss` (Socket Statistics) so you can recognize them when they appear in daily administration work.

## 7.1 Command Usage Principles

Before learning individual commands, remember these patterns:

- most commands support `--help`
- many commands accept short flags like `-l`
- many commands accept long flags like `--human-readable`
- spaces matter
- Linux is case-sensitive
- relative paths and absolute paths behave differently

## 7.2 `pwd`

### Purpose

Print the current working directory.

### Syntax

```bash
pwd
pwd -P
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-P` | Show physical path without symlink resolution shortcuts |

### Examples

```bash
pwd
pwd -P
cd /etc && pwd
```

### Notes

Use `pwd` whenever you are unsure where you are before using `rm`, `cp`, or `mv`.

## 7.3 `ls`

### Purpose

List directory contents.

### Syntax

```bash
ls
ls [options] [path]
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-l` | Long listing format |
| `-a` | Show hidden files |
| `-h` | Human-readable sizes with `-l` |
| `-t` | Sort by modification time |
| `-r` | Reverse sort order |
| `-R` | Recursive listing |
| `-d` | Show directory itself, not contents |

### Examples

```bash
ls
ls -la
ls -lh /var/log
ls -lt
ls -ld /etc
ls -R project
```

### Notes

Hidden files begin with `.`.

Examples include `.bashrc` and `.ssh`.

## 7.4 `cd`

### Purpose

Change the current directory.

### Syntax

```bash
cd [path]
```

### Useful Shortcuts

| Command | Meaning |
|---|---|
| `cd` | Go to home directory |
| `cd ~` | Go to home directory |
| `cd -` | Go to previous directory |
| `cd ..` | Go up one level |
| `cd /` | Go to root directory |

### Examples

```bash
cd /etc
cd ..
cd ~
cd -
```

### Notes

`cd` is a shell built-in in most shells.

## 7.5 `mkdir`

### Purpose

Create directories.

### Syntax

```bash
mkdir name
mkdir [options] path
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-p` | Create parent directories as needed |
| `-v` | Verbose output |
| `-m` | Set permissions on creation |

### Examples

```bash
mkdir project
mkdir -p app/logs/archive
mkdir -m 755 scripts
mkdir -pv /home/user/test/a/b/c
```

### Notes

`mkdir -p` is safe for automation because it will not fail if directories already exist.

## 7.6 `rmdir`

### Purpose

Remove empty directories.

### Syntax

```bash
rmdir [options] dir
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-p` | Remove parent directories if empty |
| `-v` | Verbose output |

### Examples

```bash
rmdir emptydir
rmdir -p a/b/c
```

### Notes

`rmdir` only works for empty directories.

Use `rm -r` only when you intend recursive removal.

## 7.7 `cp`

### Purpose

Copy files and directories.

### Syntax

```bash
cp source destination
cp [options] source destination
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-r` or `-R` | Recursive copy for directories |
| `-a` | Archive mode, preserve attributes |
| `-i` | Prompt before overwrite |
| `-u` | Copy only when source is newer |
| `-v` | Verbose output |
| `-p` | Preserve mode, ownership, timestamps |

### Examples

```bash
cp file1.txt backup.txt
cp -i config.cfg config.cfg.bak
cp -r src/ backup/
cp -a website/ website-copy/
cp -u report.txt archive/
```

### Notes

Use `cp -a` for backups when you want to preserve metadata.

## 7.8 `mv`

### Purpose

Move or rename files and directories.

### Syntax

```bash
mv source destination
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-i` | Prompt before overwrite |
| `-n` | Do not overwrite existing files |
| `-v` | Verbose output |
| `-u` | Move only when source is newer |

### Examples

```bash
mv old.txt new.txt
mv report.txt /archive/
mv -i app.conf /etc/app.conf
mv -n data.csv existing.csv
```

### Notes

A rename within the same filesystem is usually fast.

## 7.9 `rm`

### Purpose

Remove files or directories.

### Syntax

```bash
rm [options] target
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-r` or `-R` | Recursive removal |
| `-f` | Force removal, no prompt |
| `-i` | Prompt before each removal |
| `-v` | Verbose output |
| `-d` | Remove empty directory |

### Examples

```bash
rm file.txt
rm -i important.txt
rm -r old_project/
rm -rf build/
rm -v *.log
```

### Notes

`rm -rf` is powerful and dangerous.

Always confirm the path with `pwd` and `ls` first.

> Warning:
> There is no recycle bin in standard shell usage.
> A mistaken `rm` can permanently remove data.

## 7.10 `touch`

### Purpose

Create an empty file or update file timestamps.

### Syntax

```bash
touch file
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-a` | Change access time only |
| `-m` | Change modification time only |
| `-t` | Set explicit timestamp |
| `-c` | Do not create file if missing |

### Examples

```bash
touch newfile.txt
touch app.log
touch -c existing.txt
touch -t 202401011200 sample.txt
```

### Notes

`touch` is often used to create placeholder files quickly.

## 7.11 `cat`

### Purpose

Display file contents or concatenate files.

### Syntax

```bash
cat [options] file
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-n` | Number output lines |
| `-b` | Number non-empty lines |
| `-A` | Show special characters |

### Examples

```bash
cat /etc/hosts
cat file1 file2 > merged.txt
cat -n script.sh
cat -A data.txt
```

### Notes

Use `cat` for short files.

Use `less` for longer files.

## 7.12 `less`

### Purpose

View file contents interactively, page by page.

### Syntax

```bash
less file
```

### Useful Keys

| Key | Meaning |
|---|---|
| `Space` | Next page |
| `b` | Previous page |
| `/pattern` | Search forward |
| `n` | Next match |
| `q` | Quit |

### Examples

```bash
less /var/log/syslog
less +G app.log
```

### Notes

`less` is safer and more practical for large text files.

## 7.13 `more`

### Purpose

View text one screen at a time.

### Examples

```bash
more README.txt
```

### Notes

`more` is older and simpler than `less`.

Most administrators prefer `less`.

## 7.14 `head`

### Purpose

Show the first lines of a file.

### Syntax

```bash
head [options] file
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-n` | Number of lines |
| `-c` | Number of bytes |

### Examples

```bash
head /etc/passwd
head -n 20 app.log
head -c 100 file.bin
```

### Notes

Useful for quickly inspecting file structure.

## 7.15 `tail`

### Purpose

Show the last lines of a file.

### Syntax

```bash
tail [options] file
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-n` | Number of lines |
| `-f` | Follow file growth |
| `-F` | Follow across rotations |
| `-c` | Number of bytes |

### Examples

```bash
tail /var/log/syslog
tail -n 50 app.log
tail -f /var/log/nginx/access.log
tail -F /var/log/messages
```

### Notes

`tail -f` is essential for real-time log monitoring.

## 7.16 `wc`

### Purpose

Count lines, words, bytes, or characters.

### Syntax

```bash
wc [options] file
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-l` | Count lines |
| `-w` | Count words |
| `-c` | Count bytes |
| `-m` | Count characters |

### Examples

```bash
wc notes.txt
wc -l /etc/passwd
wc -w article.txt
```

### Notes

Useful in scripts for quick file metrics.

## 7.17 `file`

### Purpose

Identify file type based on content, not only extension.

### Syntax

```bash
file target
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-i` | Show MIME type information |
| `-b` | Brief output |

### Examples

```bash
file script.sh
file /bin/ls
file -i image.png
```

### Notes

Very helpful when an unknown file lacks a useful extension.

## 7.18 `stat`

### Purpose

Display detailed file or filesystem status.

### Syntax

```bash
stat target
```

### Common Flags

| Flag | Meaning |
|---|---|
| `-f` | Show filesystem status |
| `-c` | Custom output format |

### Examples

```bash
stat /etc/passwd
stat -f /
stat -c '%n %a %U %G' myfile
```

### Notes

`stat` is excellent for permissions, timestamps, inode numbers, and ownership details.

## 7.19 `which`

### Purpose

Show the path of a command found in `PATH`.

### Syntax

```bash
which command
```

### Examples

```bash
which bash
which python3
which ls
```

### Notes

Works only for commands found in your executable path.

## 7.20 `whereis`

### Purpose

Locate binary, source, and man page files for a command.

### Syntax

```bash
whereis command
```

### Examples

```bash
whereis bash
whereis ssh
```

### Notes

Unlike `which`, it can return man page and source locations too.

## 7.21 `find`

### Purpose

Search for files and directories recursively.

### Syntax

```bash
find path [tests] [actions]
```

### Common Tests and Actions

| Pattern | Meaning |
|---|---|
| `-name` | Match name |
| `-type` | Match file type |
| `-size` | Match size |
| `-mtime` | Match modified time |
| `-user` | Match owner |
| `-perm` | Match permissions |
| `-exec` | Run command on each result |
| `-delete` | Delete results |

### Examples

```bash
find . -name '*.log'
find /etc -type f -name '*.conf'
find /var/log -type f -mtime -1
find . -type d -name '.git'
find . -type f -size +100M
find . -type f -name '*.tmp' -delete
find . -type f -exec ls -lh {} \;
```

### Notes

`find` is one of the most important Linux commands.

Be cautious with `-delete` and `-exec rm`.

## 7.22 `locate`

### Purpose

Search for files using a prebuilt database.

### Syntax

```bash
locate pattern
```

### Examples

```bash
locate sshd_config
locate '*.service'
```

### Notes

`locate` is fast, but results depend on an index that may not be up to date.

Database refresh often uses:

```bash
sudo updatedb
```

## 7.23 Combining Basic Commands

### Example: find the largest logs

```bash
find /var/log -type f -name '*.log' -exec du -h {} \; | sort -h | tail
```

### Example: copy a config before editing

```bash
cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

### Example: inspect a directory safely

```bash
pwd
ls -lah
```

### Example: watch a log after deployment

```bash
tail -f /var/log/app/app.log
```

## 7.24 Path Concepts

### Absolute path

Starts from `/`.

Example:

```bash
/etc/hosts
```

### Relative path

Starts from the current directory.

Example:

```bash
./script.sh
../logs/app.log
```

## 7.25 Globbing Basics

The shell expands wildcards before the command runs.

| Pattern | Meaning |
|---|---|
| `*` | Any characters |
| `?` | One character |
| `[abc]` | One character from set |
| `[0-9]` | One digit |

Examples:

```bash
ls *.txt
rm file?.log
cp report[0-9].txt archive/
```

## 7.26 Safe Command Habits

- run `pwd` before destructive commands
- use `ls` to confirm target paths
- quote filenames with spaces
- use `cp -a` before editing critical configs
- prefer `rm -i` when learning
- avoid running as `root` unless needed

## 7.27 Practice Examples

### Create and inspect a workspace

```bash
mkdir -p ~/linux-lab/docs
cd ~/linux-lab
pwd
ls -la
```

### Create files and copy them

```bash
touch a.txt b.txt
cp a.txt a-copy.txt
ls -l
```

### Rename and move files

```bash
mkdir archive
mv a-copy.txt archive/renamed.txt
ls archive
```

### Search for shell scripts

```bash
find ~ -type f -name '*.sh'
```

### Count users listed in `/etc/passwd`

```bash
wc -l /etc/passwd
```


### Sample command outputs

```bash
$ pwd
# Expected output:
# /home/user/linux-lab
```

```bash
$ ls -la
# Expected output:
# total 16
# drwxr-xr-x  3 user user 4096 Jan 10 09:00 .
# drwxr-xr-x 18 user user 4096 Jan 10 08:55 ..
# -rw-r--r--  1 user user    0 Jan 10 09:00 a.txt
# -rw-r--r--  1 user user  128 Jan 10 09:00 notes.txt
```

```bash
$ mkdir -p app/logs/archive
# Expected output:
# (no output on success)
```

```bash
$ cp -av /etc/hosts backup/
# Expected output:
# '/etc/hosts' -> 'backup/hosts'
```

```bash
$ mv -v a.txt archive/a.txt
# Expected output:
# renamed 'a.txt' -> 'archive/a.txt'
```

```bash
$ cat /etc/hosts
# Sample output:
# 127.0.0.1   localhost
# 192.168.1.20 web01.example.com web01
```

```bash
$ find ~ -type f -name '*.sh'
# Sample output:
# /home/user/bin/backup.sh
# /home/user/projects/deploy.sh
```

```bash
$ du -sh /var/log
# Expected output:
# 1.2G    /var/log
```

```bash
$ df -h
# Expected output:
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        50G   12G   35G  26% /
# /dev/sdb1       100G   45G   50G  47% /data
# tmpfs           3.9G     0  3.9G   0% /dev/shm
```

```bash
$ lsblk
# Expected output:
# NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
# sda      8:0    0   50G  0 disk
# ├─sda1   8:1    0   49G  0 part /
# └─sda2   8:2    0    1G  0 part [SWAP]
# sdb      8:16   0  100G  0 disk
# └─sdb1   8:17   0  100G  0 part /data
```

```bash
$ ps aux | head -5
# Sample output:
# USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# root         1  0.0  0.1 169200 11344 ?        Ss   08:00   0:02 /sbin/init
# root       732  0.0  0.3 235120 28640 ?        Ssl  08:01   0:01 /usr/lib/systemd/systemd-journald
# user      2841  0.2  0.5 512344 44212 pts/0    Ss   09:00   0:00 -bash
```

```bash
$ top
# Sample output:
# top - 09:15:40 up 10 days,  2 users,  load average: 0.08, 0.10, 0.07
# Tasks: 182 total,   1 running, 181 sleeping,   0 stopped,   0 zombie
# %Cpu(s):  2.1 us,  1.0 sy, 96.4 id,  0.4 wa,  0.0 hi,  0.1 si,  0.0 st
# MiB Mem :   7850.0 total,   2140.3 free,   1988.1 used,   3721.6 buff/cache
```

```bash
$ ss -tulpn
# Expected output:
# Netid  State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
# tcp    LISTEN  0       128     0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=1234,fd=3))
# tcp    LISTEN  0       128     0.0.0.0:80          0.0.0.0:*          users:(("nginx",pid=5678,fd=6))
```

## 7.28 Command Summary Table

| Command | Primary Use | Common Example |
|---|---|---|
| `pwd` | Show current directory | `pwd` |
| `ls` | List files | `ls -lah` |
| `cd` | Change directory | `cd /etc` |
| `mkdir` | Create directory | `mkdir -p app/logs` |
| `rmdir` | Remove empty directory | `rmdir olddir` |
| `cp` | Copy files | `cp -a src/ backup/` |
| `mv` | Move or rename | `mv old new` |
| `rm` | Remove files | `rm -i file` |
| `touch` | Create file or update time | `touch notes.txt` |
| `cat` | Print files | `cat /etc/hosts` |
| `less` | Page through text | `less app.log` |
| `more` | Simple pager | `more file.txt` |
| `head` | First lines | `head -n 20 file` |
| `tail` | Last lines | `tail -f file` |
| `wc` | Count lines/words/bytes | `wc -l file` |
| `file` | Detect type | `file /bin/ls` |
| `stat` | Detailed metadata | `stat file` |
| `which` | Path of command | `which bash` |
| `whereis` | Binary and docs | `whereis ssh` |
| `find` | Recursive search | `find . -name '*.conf'` |
| `locate` | Indexed search | `locate passwd` |

> Tip:
> If you can master `ls`, `cd`, `cp`, `mv`, `rm`, `find`, and `tail`, you can already do useful Linux work every day.

---
