# Regular Expressions

> Basic and extended regex plus Bash matching with =~.

## 11. Regular Expressions

### 11.1 Why Regex Matters

Regular expressions help match text patterns.

Useful in shell scripts for:

- validation
- parsing logs
- filtering lines
- extracting values

### 11.2 Basic vs Extended Regex

| Type | Tool Examples | Notes |
| --- | --- | --- |
| Basic Regular Expressions | `grep`, `sed` | Some metacharacters require escaping |
| Extended Regular Expressions | `grep -E`, `awk`, Bash `=~` | More expressive |

### 11.3 Common Regex Tokens

| Token | Meaning |
| --- | --- |
| `.` | Any single character |
| `*` | Zero or more |
| `+` | One or more |
| `?` | Zero or one |
| `^` | Start of line |
| `$` | End of line |
| `[abc]` | Character class |
| `[^abc]` | Negated class |
| `[0-9]` | Range |
| `(a|b)` | Alternation |

### 11.4 Using Bash `=~`

```bash
email="user@example.com"
if [[ $email =~ ^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$ ]]; then
  echo "Valid email"
fi
```

### 11.5 Capturing Groups in Bash

Matches go into `BASH_REMATCH`.

```bash
text="version=1.2.3"
if [[ $text =~ ^version=([0-9]+)\.([0-9]+)\.([0-9]+)$ ]]; then
  echo "Major: ${BASH_REMATCH[1]}"
  echo "Minor: ${BASH_REMATCH[2]}"
  echo "Patch: ${BASH_REMATCH[3]}"
fi
```

### 11.6 `grep -E`

```bash
grep -E 'error|warning' app.log
```

### 11.7 Validate Numbers

```bash
if [[ $value =~ ^[0-9]+$ ]]; then
  echo "Integer"
fi
```

### 11.8 Validate IPv4 Format

```bash
if [[ $ip =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]]; then
  echo "Looks like IPv4"
fi
```

Note: format validity is not the same as numeric range validity.

### 11.9 Extract Date from Log Line

```bash
line='2025-01-15 INFO started'
if [[ $line =~ ^([0-9]{4}-[0-9]{2}-[0-9]{2})[[:space:]] ]]; then
  echo "Date: ${BASH_REMATCH[1]}"
fi
```

### 11.10 Whitespace Classes

Use POSIX character classes:

- `[[:space:]]`
- `[[:digit:]]`
- `[[:alpha:]]`
- `[[:alnum:]]`
- `[[:lower:]]`
- `[[:upper:]]`

### 11.11 Regex with `sed`

```bash
echo 'user=alice' | sed -E 's/^user=(.*)$/\1/'
```

### 11.12 Regex with `awk`

```bash
awk '/ERROR|WARN/ {print $0}' app.log
```

### 11.13 Quoting Rules with `=~`

Do not quote the regex on the right side in Bash if you want regex behavior.

Good:

```bash
[[ $value =~ ^[0-9]+$ ]]
```

Quoted pattern may behave differently.

### 11.14 Common Patterns Table

| Purpose | Pattern |
| --- | --- |
| Integer | `^[0-9]+$` |
| Identifier | `^[A-Za-z_][A-Za-z0-9_]*$` |
| Date `YYYY-MM-DD` | `^[0-9]{4}-[0-9]{2}-[0-9]{2}$` |
| Time `HH:MM:SS` | `^[0-9]{2}:[0-9]{2}:[0-9]{2}$` |
| Log level | `^(INFO|WARN|ERROR|DEBUG)$` |

### 11.15 Section Summary

Regex is powerful, but readability matters.

Keep patterns documented and tested.

---
