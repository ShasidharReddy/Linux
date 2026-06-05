# Operators

> Arithmetic, comparison, logical, string, and file test operators.

## 4. Operators
### 4.1 Overview
Shell scripts use operators for:

- Arithmetic
- Comparison
- String tests
- File tests
- Logical evaluation

### 4.2 Arithmetic Operators
| Operator | Meaning | Example |
| --- | --- | --- |
| `+` | Addition | `$((a + b))` |
| `-` | Subtraction | `$((a - b))` |
| `*` | Multiplication | `$((a * b))` |
| `/` | Division | `$((a / b))` |
| `%` | Modulus | `$((a % b))` |
| `++` | Increment | `((a++))` |
| `--` | Decrement | `((a--))` |

Example:

```bash
a=10
b=3
echo $((a + b))
echo $((a % b))
```

### 4.3 Numeric Comparison Operators
Used with `test`, `[ ]`, or `[[ ]]`.

| Operator | Meaning |
| --- | --- |
| `-eq` | Equal |
| `-ne` | Not equal |
| `-gt` | Greater than |
| `-lt` | Less than |
| `-ge` | Greater than or equal |
| `-le` | Less than or equal |

Example:

```bash
if [ "$a" -gt "$b" ]; then
  echo "a is larger"
fi
```

### 4.4 String Comparison Operators
| Operator | Meaning |
| --- | --- |
| `=` or `==` | Equal |
| `!=` | Not equal |
| `-z` | String is empty |
| `-n` | String is not empty |

Example:

```bash
if [[ $name == "admin" ]]; then
  echo "Welcome"
fi
```

### 4.5 File Test Operators
| Operator | Meaning |
| --- | --- |
| `-f` | Regular file exists |
| `-d` | Directory exists |
| `-e` | Path exists |
| `-r` | Readable |
| `-w` | Writable |
| `-x` | Executable |
| `-s` | Non-empty file |
| `-L` | Symbolic link |

Examples:

```bash
[[ -f /etc/passwd ]]
[[ -d /var/log ]]
[[ -x ./deploy.sh ]]
```

### 4.6 Additional File Tests
| Operator | Meaning |
| --- | --- |
| `-b` | Block device |
| `-c` | Character device |
| `-p` | Named pipe |
| `-S` | Socket |
| `-O` | Owned by current user |
| `-G` | Owned by current group |
| `-N` | Modified since last read |
| `file1 -nt file2` | Newer than |
| `file1 -ot file2` | Older than |

### 4.7 Logical Operators
In Bash `[[ ]]`:

```bash
[[ $a -gt 0 && $b -gt 0 ]]
[[ $a -gt 0 || $b -gt 0 ]]
[[ ! -f missing.txt ]]
```

### 4.8 Arithmetic Evaluation with `(( ))`
```bash
count=5
if (( count > 3 )); then
  echo "Greater than 3"
fi
```

### 4.9 Operator Precedence
Use parentheses to make logic clear.

```bash
if [[ ($role == admin || $role == ops) && $enabled == yes ]]; then
  echo "Allowed"
fi
```

### 4.10 Common Comparison Examples
#### Number comparison

```bash
if (( disk_usage > 80 )); then
  echo "Warning"
fi
```

#### String empty check

```bash
if [[ -z ${input:-} ]]; then
  echo "No input"
fi
```

#### File exists check

```bash
if [[ -e $config_file ]]; then
  echo "Config found"
fi
```

### 4.11 Old `expr` Tool
Historically used for arithmetic:

```bash
expr 5 + 2
```

Modern Bash prefers:

```bash
echo $((5 + 2))
```

### 4.12 Section Summary
Use:

- `(( ))` for arithmetic
- `[[ ]]` for modern Bash tests
- `[ ]` for portable tests
- file test operators for filesystem conditions

---
