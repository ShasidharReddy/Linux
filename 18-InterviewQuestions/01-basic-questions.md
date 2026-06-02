# Basic Linux Interview Questions

This guide collects foundational Linux interview questions and answers for entry-level preparation.

## Q1: What is Linux?
**Answer:** Linux is an open-source Unix-like operating system kernel created by Linus Torvalds. In common usage, people often say "Linux" to refer to a full operating system distribution built around the Linux kernel, such as Ubuntu, Debian, Red Hat Enterprise Linux, Rocky Linux, Alpine, and SUSE. A Linux distribution includes the kernel, shell, package manager, system libraries, utilities, and services.

Linux is widely used in servers, cloud platforms, embedded systems, development environments, containers, supercomputers, and networking appliances. It is known for stability, security, automation-friendliness, and transparency.

Key characteristics:
- Multiuser
- Multitasking
- Permission-based security model
- Powerful command-line interface
- Rich networking and scripting capabilities
- Extensive open-source ecosystem

Example commands:
```bash
uname -a
cat /etc/os-release
hostnamectl
```

---

## Q2: What is the difference between the Linux kernel and a Linux distribution?
**Answer:** The **kernel** is the core component that interacts with hardware, manages CPU scheduling, memory, storage, devices, and system calls. A **distribution** is a complete operating system built around the kernel and includes user-space tools, package management, init system, shells, and documentation.

For example:
- Kernel: Linux 6.x
- Distribution: Ubuntu 24.04, RHEL 9, Debian 12

Think of it this way:
- Kernel = engine
- Distribution = complete vehicle

Example commands:
```bash
uname -r
cat /etc/os-release
ls /boot
```

---

## Q3: What are the main components of a Linux system?
**Answer:** A Linux system is typically made of the following major components:

1. **Kernel** — Handles hardware interaction, memory, processes, drivers, and system calls.
2. **Shell** — Command interpreter such as Bash, Zsh, or Fish.
3. **File system** — Organizes data in a hierarchical directory structure.
4. **System libraries** — Provide standard functions for applications, such as glibc.
5. **Init system** — Starts services and manages boot, commonly `systemd`.
6. **User-space utilities** — Tools like `ls`, `cp`, `grep`, `awk`, `sed`, and `ps`.
7. **Package manager** — Installs and updates software, such as `apt`, `dnf`, `yum`, `zypper`, or `pacman`.
8. **Applications/services** — Web servers, databases, SSH, cron, and monitoring tools.

Example commands:
```bash
ps -p 1 -o comm=
which bash
ldd --version
systemctl --version
```

---

## Q4: What is the Linux file system hierarchy?
**Answer:** Linux uses a single-rooted hierarchical directory structure starting at `/`. Every file and directory exists under this root.

Important directories:
- `/` — Root of the entire file system
- `/bin` — Essential user binaries
- `/sbin` — Essential system binaries
- `/etc` — Configuration files
- `/home` — User home directories
- `/root` — Root user home directory
- `/var` — Variable data like logs, caches, spool files
- `/usr` — User programs, libraries, documentation
- `/tmp` — Temporary files
- `/dev` — Device files
- `/proc` — Virtual process/kernel information
- `/sys` — Virtual kernel/device information
- `/boot` — Bootloader files and kernel images
- `/lib`, `/lib64` — Shared libraries
- `/mnt`, `/media` — Mount points
- `/opt` — Optional third-party software

Example commands:
```bash
ls /
tree -L 1 /
find /etc -maxdepth 1 -type f | head
```

---

## Q5: What is the difference between an absolute path and a relative path?
**Answer:** An **absolute path** begins from the root directory `/` and always points to the same location regardless of the current working directory. A **relative path** is interpreted from your current directory.

Examples:
- Absolute: `/etc/ssh/sshd_config`
- Relative: `../logs/app.log`

If you are in `/home/user/projects`:
- `cd ../docs` moves to `/home/user/docs`
- `cat /etc/hosts` always reads the same file

Example commands:
```bash
pwd
cd /var/log
cd ../tmp
readlink -f ../log
```

---

## Q6: What do `.` and `..` mean in Linux paths?
**Answer:** These are special directory references:
- `.` means the **current directory**
- `..` means the **parent directory**

They are useful in navigation, scripting, and file operations.

Examples:
- `./script.sh` runs a script in the current directory
- `cd ..` moves up one directory
- `cp file.txt ../backup/` copies a file to the parent directory's backup directory

Example commands:
```bash
pwd
ls -la .
ls -la ..
cd ..
```

---

## Q7: How do you list files in Linux?
**Answer:** The `ls` command lists files and directories. Common options provide detailed information, hidden files, sorting, and human-readable sizes.

Common options:
- `ls` — Simple listing
- `ls -l` — Long format
- `ls -a` — Include hidden files
- `ls -lh` — Human-readable sizes
- `ls -lt` — Sort by modification time
- `ls -R` — Recursive listing

Example commands:
```bash
ls
ls -la
ls -lh /var/log
ls -lt /etc | head
```

---

## Q8: How do you create, copy, move, and delete files?
**Answer:** Basic file operations are essential in Linux administration.

- Create empty file: `touch`
- Copy file: `cp`
- Move or rename file: `mv`
- Delete file: `rm`
- Remove directory: `rmdir` or `rm -r`

Be careful with deletion, especially with recursive operations.

Example commands:
```bash
touch notes.txt
cp notes.txt notes.bak
mv notes.bak archive.txt
rm archive.txt
mkdir demo_dir
rmdir demo_dir
```

---

## Q9: What are hidden files in Linux?
**Answer:** Hidden files are files or directories whose names begin with a dot (`.`). They are commonly used for configuration.

Examples:
- `.bashrc`
- `.profile`
- `.ssh/`
- `.gitconfig`

They are not shown in a normal `ls` output unless you use `-a`.

Example commands:
```bash
ls
ls -a
ls -ld ~/.ssh ~/.bashrc
```

---

## Q10: How do you view file contents?
**Answer:** Linux provides many tools for viewing file contents depending on use case.

Common tools:
- `cat` — Print full file content
- `less` — Paginated viewing
- `more` — Simpler pager
- `head` — First lines
- `tail` — Last lines
- `tail -f` — Follow a growing file, useful for logs

Example commands:
```bash
cat /etc/hosts
head -n 20 /etc/passwd
tail -n 50 /var/log/system.log
less /etc/ssh/sshd_config
```

---

## Q11: How do you search for files in Linux?
**Answer:** The main tools are `find`, `locate`, and shell globbing.

- `find` performs real-time searches with filters like name, type, size, and age.
- `locate` uses a prebuilt database and is faster but may be outdated.
- Globbing uses patterns such as `*.log`.

Example commands:
```bash
find /var/log -name "*.log"
find /home -type f -size +100M
locate sshd_config
ls /etc/*.conf
```

---

## Q12: How do you search for text inside files?
**Answer:** The `grep` command searches file content using patterns. It can search recursively, ignore case, show line numbers, or use regular expressions.

Useful options:
- `grep "text" file`
- `grep -i` — Case-insensitive
- `grep -r` — Recursive
- `grep -n` — Line numbers
- `grep -v` — Invert match
- `grep -E` — Extended regex

Example commands:
```bash
grep "PermitRootLogin" /etc/ssh/sshd_config
grep -rin "error" /var/log
grep -E "^(root|admin)" /etc/passwd
```

---

## Q13: What is a regular expression?
**Answer:** A regular expression (regex) is a pattern used to match text. Linux tools such as `grep`, `sed`, `awk`, and many programming languages support regex.

Common patterns:
- `.` — Any character
- `*` — Zero or more of previous item
- `+` — One or more
- `^` — Start of line
- `$` — End of line
- `[0-9]` — Any digit
- `[^a-z]` — Not lowercase letter

Example commands:
```bash
grep -E "^root" /etc/passwd
grep -E "[0-9]{3}" sample.txt
grep -E "error|warning|critical" app.log
```

---

## Q14: What is the purpose of `pwd`?
**Answer:** `pwd` stands for **print working directory**. It shows your current location in the file system. This is especially useful in scripts and while navigating large directory trees.

Example commands:
```bash
pwd
cd /etc/ssh
pwd
```

---

## Q15: How do permissions work in Linux?
**Answer:** Linux file permissions control who can read, write, or execute files and directories. Permissions are divided into three categories:

- **User (u)** — Owner
- **Group (g)** — Group members
- **Others (o)** — Everyone else

Permission bits:
- `r` = read
- `w` = write
- `x` = execute

Example long listing:
```text
-rwxr-x--- 1 alice devops 2048 Jan 10 10:00 deploy.sh
```
This means:
- Owner: read, write, execute
- Group: read, execute
- Others: no access

Example commands:
```bash
ls -l deploy.sh
chmod u+x deploy.sh
chmod 750 deploy.sh
```

---

## Q16: What do numeric permission modes like 755 and 644 mean?
**Answer:** Numeric mode is an octal representation of permissions.

Values:
- `r = 4`
- `w = 2`
- `x = 1`

Examples:
- `755` = owner `7` (rwx), group `5` (r-x), others `5` (r-x)
- `644` = owner `6` (rw-), group `4` (r--), others `4` (r--)
- `600` = owner only read/write

Typical uses:
- Directories/scripts: `755`
- Regular config/data files: `644`
- Private key files: `600`

Example commands:
```bash
chmod 755 script.sh
chmod 644 app.conf
chmod 600 ~/.ssh/id_rsa
```

---

## Q17: What is the difference between file and directory permissions?
**Answer:** File and directory permissions behave differently.

For files:
- `r` = read contents
- `w` = modify file contents
- `x` = execute file as a program/script

For directories:
- `r` = list directory contents
- `w` = create, delete, rename entries within directory
- `x` = enter/traverse directory

Important nuance: to access a file inside a directory, you usually need execute permission on the directory.

Example commands:
```bash
mkdir testdir
chmod 700 testdir
ls -ld testdir
```

---

## Q18: How do you change file ownership?
**Answer:** Use `chown` to change owner and optionally group, and `chgrp` to change only the group.

Examples:
- `chown alice file.txt`
- `chown alice:devops file.txt`
- `chgrp devops file.txt`
- `chown -R nginx:nginx /var/www/html`

Example commands:
```bash
sudo chown alice report.txt
sudo chown alice:developers report.txt
sudo chgrp developers report.txt
```

---

## Q19: How do you create users in Linux?
**Answer:** User accounts can be created with commands such as `useradd` or `adduser` depending on distribution.

Typical steps:
1. Create the user
2. Set a password
3. Optionally assign shell, home directory, and groups

Example commands:
```bash
sudo useradd -m -s /bin/bash appuser
sudo passwd appuser
id appuser
getent passwd appuser
```

---

## Q20: How do you manage groups?
**Answer:** Groups simplify permission management across multiple users.

Common commands:
- `groupadd` — Create group
- `groupdel` — Delete group
- `usermod -aG` — Add user to supplementary group
- `groups` — Show user group membership
- `id` — Show UID, GID, and groups

Example commands:
```bash
sudo groupadd developers
sudo usermod -aG developers alice
id alice
groups alice
```

---

## Q21: What files store user and group account information?
**Answer:** Linux stores account information in several key files:

- `/etc/passwd` — User account information
- `/etc/shadow` — Password hashes and aging data
- `/etc/group` — Group definitions
- `/etc/gshadow` — Secure group information

Examples:
- `/etc/passwd` includes username, UID, GID, comment, home, shell
- `/etc/shadow` is readable only by privileged users

Example commands:
```bash
cat /etc/passwd | head
sudo cat /etc/shadow | head
cat /etc/group | head
getent passwd root
```

---

## Q22: What is the root user?
**Answer:** The `root` user is the superuser with unrestricted administrative privileges. Root can read or modify nearly any file, manage users, change configuration, install software, and control services.

Because root access is powerful and risky:
- Use it carefully
- Prefer `sudo` where possible
- Audit administrative actions
- Disable direct remote root login when appropriate

Example commands:
```bash
whoami
id
sudo -l
sudo systemctl restart sshd
```

---

## Q23: What is `sudo` and why is it preferred over logging in as root?
**Answer:** `sudo` allows an authorized user to run commands with elevated privileges, usually as root. It is preferred because it provides accountability, finer-grained control, and reduced exposure compared to logging in directly as root.

Benefits:
- Commands are logged
- Access can be delegated per user/group
- Reduces accidental full-session root usage
- Supports least privilege

Example commands:
```bash
sudo whoami
sudo visudo
sudo systemctl status sshd
```

---

## Q24: How do you change your password and view password aging settings?
**Answer:** Use `passwd` to change a password. Use `chage` to inspect or update password aging policies.

Example commands:
```bash
passwd
sudo passwd alice
chage -l alice
sudo chage -M 90 -W 7 alice
```

Meaning:
- `-M 90` — Maximum 90 days before expiry
- `-W 7` — Warn 7 days before expiry

---

## Q25: What is a shell?
**Answer:** A shell is a command interpreter that lets users interact with the operating system. It reads commands, executes programs, supports scripting, handles variables, redirection, pipes, and job control.

Common shells:
- `bash`
- `sh`
- `zsh`
- `ksh`
- `fish`

Example commands:
```bash
echo $SHELL
cat /etc/shells
ps -p $$ -o comm=
```

---

## Q26: What is the difference between Bash and Sh?
**Answer:** `sh` traditionally refers to the Bourne shell or a POSIX-compatible shell interface, while `bash` is the GNU Bourne Again Shell with many extended features.

Bash provides:
- Arrays
- Brace expansion
- Advanced conditionals
- Improved history and completion
- More scripting conveniences

Portable scripts often use POSIX `sh` syntax. Scripts requiring Bash features should declare `#!/bin/bash`.

Example commands:
```bash
ls -l /bin/sh /bin/bash
bash --version
sh --version || true
```

---

## Q27: What are environment variables?
**Answer:** Environment variables are key-value pairs inherited by processes. They influence shell behavior, executable lookup, locale, editors, and application runtime settings.

Common variables:
- `PATH`
- `HOME`
- `USER`
- `SHELL`
- `LANG`
- `EDITOR`

Example commands:
```bash
env | sort
echo $PATH
echo $HOME
export APP_ENV=production
echo $APP_ENV
```

---

## Q28: What is the `PATH` variable?
**Answer:** `PATH` is an environment variable containing a colon-separated list of directories that the shell searches to find executable commands.

If `PATH` includes `/usr/local/bin:/usr/bin:/bin`, then running `ls` causes the shell to search those directories in order.

Best practices:
- Avoid adding writable directories for untrusted users
- Use absolute paths in critical scripts
- Verify command location with `which`, `type`, or `command -v`

Example commands:
```bash
echo $PATH
which python3
command -v systemctl
type ls
```

---

## Q29: What are pipes and redirection in Linux?
**Answer:** Pipes and redirection are fundamental shell features.

- `>` redirects stdout to a file, overwriting it
- `>>` appends stdout to a file
- `<` redirects file input to a command
- `2>` redirects stderr
- `|` pipes stdout of one command into another command's stdin

Examples:
- `ls > files.txt`
- `grep error app.log | wc -l`
- `command >out.txt 2>err.txt`

Example commands:
```bash
ls -l > listing.txt
grep root /etc/passwd | cut -d: -f1
find /etc -name "*.conf" 2>/dev/null | head
```

---

## Q30: What is the difference between stdout, stderr, and stdin?
**Answer:** Standard streams are default communication channels for processes.

- **stdin (0)** — Standard input
- **stdout (1)** — Standard output
- **stderr (2)** — Standard error

Separating stdout and stderr is important in scripting and automation.

Example commands:
```bash
echo "hello" > out.txt
ls /does-not-exist 2> err.txt
sort < /etc/passwd | head
command > all.out 2>&1
```

---

## Q31: How do you count words, lines, and characters in a file?
**Answer:** Use `wc` (word count).

Common options:
- `wc -l` — Line count
- `wc -w` — Word count
- `wc -c` — Byte count
- `wc -m` — Character count

Example commands:
```bash
wc /etc/passwd
wc -l /etc/passwd
wc -w notes.txt
wc -c logfile.txt
```

---

## Q32: How do you compare files?
**Answer:** Linux offers several file comparison tools.

- `diff` — Line-by-line differences
- `cmp` — Byte-by-byte comparison
- `comm` — Compare sorted files
- `md5sum`/`sha256sum` — Compare checksums

Example commands:
```bash
diff file1.txt file2.txt
cmp file1.bin file2.bin
sha256sum image1.iso image2.iso
```

---

## Q33: What is the difference between a hard link and a symbolic link?
**Answer:** A **hard link** points to the same inode as the original file. A **symbolic link** is a separate file that stores a path to another file.

Hard link characteristics:
- Same inode as target
- Usually cannot span file systems
- Typically cannot link directories

Symbolic link characteristics:
- Different inode
- Can span file systems
- Can point to directories
- Can become broken if target is removed

Example commands:
```bash
ln original.txt hardlink.txt
ln -s original.txt symlink.txt
ls -li original.txt hardlink.txt symlink.txt
```

---

## Q34: What is an inode?
**Answer:** An inode is a metadata structure describing a file or directory. It stores information such as ownership, permissions, timestamps, size, and disk block pointers, but not the file name itself. File names are stored in directory entries mapping names to inode numbers.

Useful when troubleshooting:
- Many hard links point to same inode
- File systems can run out of inodes even with free space

Example commands:
```bash
ls -i /etc/hosts
stat /etc/hosts
df -i
```

---

## Q35: How do you check disk usage and free space?
**Answer:** Use `df` for file system free space and `du` for per-file or per-directory usage.

- `df -h` — Human-readable free space
- `df -i` — Inode usage
- `du -sh dir` — Size of directory
- `du -ah dir | sort -h` — Detailed usage

Example commands:
```bash
df -h
df -i
du -sh /var/log
du -ah /var | sort -h | tail
```

---

## Q36: How do you create and extract archives?
**Answer:** Archiving and compression are common admin tasks.

Tools:
- `tar` — Archive files/directories
- `gzip`, `gunzip` — Compress/decompress
- `bzip2`, `xz` — Alternative compressors
- `zip`, `unzip` — ZIP archives

Example commands:
```bash
tar -cvf backup.tar /etc
tar -czvf backup.tar.gz /etc
tar -xzvf backup.tar.gz
zip -r project.zip project/
unzip project.zip
```

---

## Q37: What is package management in Linux?
**Answer:** Package management is the process of installing, updating, removing, and verifying software using distribution-specific tools. Packages include binaries, dependencies, metadata, and scripts.

Major package families:
- Debian/Ubuntu: `.deb` with `apt`, `dpkg`
- RHEL/CentOS/Rocky/Alma: `.rpm` with `dnf`, `yum`, `rpm`
- SUSE: `zypper`, `rpm`
- Arch: `pacman`

Example commands:
```bash
apt list --installed | head
dnf list installed | head
rpm -qa | head
dpkg -l | head
```

---

## Q38: How do you install software on Debian/Ubuntu systems?
**Answer:** Use `apt` for package installation and repository-based management. Use `dpkg` for direct `.deb` file handling.

Common workflow:
1. Refresh package metadata
2. Install package
3. Verify installation

Example commands:
```bash
sudo apt update
sudo apt install -y nginx
apt show nginx
sudo dpkg -i package.deb
sudo apt -f install
```

---

## Q39: How do you install software on RHEL-based systems?
**Answer:** Modern RHEL-based systems commonly use `dnf`; older versions use `yum`. Direct RPM handling uses `rpm`.

Example commands:
```bash
sudo dnf makecache
sudo dnf install -y httpd
rpm -q httpd
sudo rpm -ivh package.rpm
sudo dnf remove -y httpd
```

---

## Q40: How do you update and remove packages?
**Answer:** Updating and removing packages depends on the package manager.

Debian/Ubuntu:
```bash
sudo apt update
sudo apt upgrade -y
sudo apt remove -y nginx
sudo apt purge -y nginx
```

RHEL-based:
```bash
sudo dnf upgrade -y
sudo dnf remove -y httpd
```

Difference between remove and purge:
- `remove` typically keeps some config files
- `purge` removes config files too

---

## Q41: How do you verify whether a command exists?
**Answer:** Use `which`, `type`, or `command -v`.

Recommended:
- `command -v` is POSIX-friendly and script-safe
- `type` shows whether a command is an alias, builtin, function, or file

Example commands:
```bash
command -v ssh
which ssh
type cd
type ll || true
```

---

## Q42: What is the purpose of `man` pages?
**Answer:** `man` pages are built-in documentation for commands, system calls, configuration formats, and library functions. They are invaluable during troubleshooting and interviews.

Man page sections include:
1. User commands
5. File formats/configs
8. System administration commands

Example commands:
```bash
man ls
man 5 passwd
man 8 systemctl
```

---

## Q43: How do you see command history?
**Answer:** Bash stores command history and allows reuse of previous commands.

Useful methods:
- `history` — Show history list
- `!n` — Re-run command number `n`
- `!!` — Re-run previous command
- `Ctrl+r` — Reverse search interactively

Example commands:
```bash
history | tail
history | grep ssh
!!
```

---

## Q44: What is the difference between `echo` and `printf`?
**Answer:** Both print text, but `printf` is more predictable and script-friendly.

- `echo` is simple but behavior may vary with escape sequences or `-n`
- `printf` supports formatting and avoids portability issues

Example commands:
```bash
echo "Hello"
printf "User: %s\n" "$USER"
printf "CPU: %.2f\n" 87.25
```

---

## Q45: How do you sort and extract columns from text output?
**Answer:** Common tools include `sort`, `cut`, `awk`, `column`, and `uniq`.

Examples:
- `sort file.txt`
- `cut -d: -f1 /etc/passwd`
- `awk '{print $1}'`
- `sort | uniq -c`

Example commands:
```bash
cut -d: -f1 /etc/passwd | sort | head
awk -F: '{print $1, $7}' /etc/passwd
sort names.txt | uniq -c
```

---

## Q46: What is the purpose of `touch`?
**Answer:** `touch` creates an empty file if it does not exist, or updates access and modification timestamps if it does exist.

Use cases:
- Create placeholder files
- Update timestamps for build or deployment workflows
- Test scripts and file monitoring

Example commands:
```bash
touch app.log
stat app.log
touch -t 202501011200 sample.txt
```

---

## Q47: How do you determine file type?
**Answer:** Use `file` to inspect file content type and `stat` for metadata. File extensions are not authoritative in Linux.

Example commands:
```bash
file /bin/ls
file archive.tar.gz
stat /bin/ls
```

Typical output can show ELF binary, ASCII text, shell script, gzip compressed data, or symbolic link.

---

## Q48: How do you monitor log files in real time?
**Answer:** The most common tool is `tail -f`, which follows new log entries as they are written. This is extremely useful during deployments, service restarts, and troubleshooting.

Alternative tools:
- `less +F`
- `journalctl -f` for systemd journals
- `multitail` if installed

Example commands:
```bash
tail -f /var/log/syslog
journalctl -u nginx -f
```

---

## Q49: What is the difference between soft reboot and shutdown commands?
**Answer:** Reboot and shutdown commands are used for system power state management.

Common commands:
- `reboot` — Restart the system
- `shutdown -r now` — Reboot now
- `shutdown -h now` — Halt/power off now
- `poweroff` — Shut down system

In production, always ensure users, applications, storage, and maintenance windows are handled properly before rebooting.

Example commands:
```bash
sudo shutdown -r +5 "Kernel maintenance"
sudo shutdown -h now
sudo systemctl reboot
```

---

## Q50: What are the most important beginner Linux commands to know?
**Answer:** A strong beginner should be comfortable with navigation, file management, viewing content, searching, permissions, process observation, package management, and basic networking commands.

Essential commands include:
- `pwd`, `cd`, `ls`
- `cp`, `mv`, `rm`, `mkdir`, `touch`
- `cat`, `less`, `head`, `tail`
- `grep`, `find`, `sort`, `cut`, `awk`
- `chmod`, `chown`, `chgrp`
- `ps`, `top`, `kill`
- `df`, `du`, `free`
- `ip`, `ping`, `ss`
- `apt`, `dnf`, `yum`
- `systemctl`, `journalctl`

Example commands:
```bash
pwd
ls -la
find /etc -name "*.conf" | head
grep -rin "listen" /etc/nginx
ps aux | head
df -h
free -h
ip a
```

---
