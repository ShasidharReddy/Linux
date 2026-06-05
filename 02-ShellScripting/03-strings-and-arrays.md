# Strings and Arrays

> String operations plus indexed and associative arrays.

## 3. Data Types & Strings

### 3.1 Data Types in Shell

Shell scripting does not have strict data types like many languages.

Most values are treated as strings.

Arithmetic contexts interpret values numerically.

Conceptually you work with:

- Strings
- Integers
- Arrays
- Command output
- Exit statuses
- File paths

### 3.2 String Declaration

```bash
first_name="Ada"
last_name="Lovelace"
```

### 3.3 String Concatenation

Concatenate by placing values next to each other.

```bash
full_name="$first_name $last_name"
echo "$full_name"
```

No special operator is required.

### 3.4 String Length

```bash
text="shell"
echo "${#text}"
```

### 3.5 Substring Extraction

Bash substring syntax:

```bash
text="shellscripting"
echo "${text:0:5}"
echo "${text:5}"
```

### 3.6 Remove Prefix Pattern

```bash
path="/var/log/syslog"
echo "${path#/var/}"
```

Longest prefix removal:

```bash
echo "${path##*/}"
```

### 3.7 Remove Suffix Pattern

```bash
file="archive.tar.gz"
echo "${file%.gz}"
echo "${file%%.*}"
```

### 3.8 String Replacement

Replace first match:

```bash
msg="hello world"
echo "${msg/world/universe}"
```

Replace all matches:

```bash
echo "${msg//o/0}"
```

### 3.9 Case Conversion

Bash supports case modification.

```bash
word="shell"
echo "${word^^}"
echo "${word,,}"
```

Capitalize first character:

```bash
echo "${word^}"
```

### 3.10 Check Prefix and Suffix

Using pattern matching in `[[ ]]`:

```bash
file="report.csv"
[[ $file == report* ]]
[[ $file == *.csv ]]
```

### 3.11 Pattern Matching

```bash
name="script.sh"
if [[ $name == *.sh ]]; then
  echo "Shell script"
fi
```

### 3.12 String Comparison

```bash
[[ "$a" == "$b" ]]
[[ "$a" != "$b" ]]
[[ -z "$a" ]]
[[ -n "$a" ]]
```

### 3.13 Empty vs Unset

Empty string:

```bash
var=""
```

Unset variable:

```bash
unset var
```

Check carefully when using `set -u`.

### 3.14 Splitting Strings

Use `IFS` and `read`.

```bash
record="alice,30,admin"
IFS=',' read -r name age role <<< "$record"
```

### 3.15 Joining Strings

```bash
parts=(one two three)
joined=$(IFS=','; echo "${parts[*]}")
```

### 3.16 Escaping Special Characters

Common characters needing attention:

- `$`
- `"`
- `'`
- `\`
- `` ` ``
- `*`
- `?`
- `[` and `]`

### 3.17 Single Quotes vs Double Quotes

Single quotes preserve literal text:

```bash
echo 'Value: $HOME'
```

Double quotes allow expansion:

```bash
echo "Value: $HOME"
```

### 3.18 ANSI-C Quoting

```bash
printf '%s\n' $'line1\nline2'
```

Useful for embedded escapes.

### 3.19 Multi-Line Strings

```bash
text="line1
line2
line3"
printf '%s\n' "$text"
```

### 3.20 Trimming Whitespace

Use external tools when needed.

```bash
trimmed=$(echo "$input" | sed 's/^ *//;s/ *$//')
```

### 3.21 Search Inside a String

```bash
if [[ $text == *error* ]]; then
  echo "Contains error"
fi
```

### 3.22 Convert Delimited Data to Array

```bash
IFS=':' read -r -a path_parts <<< "$PATH"
```

### 3.23 String Formatting with `printf`

```bash
name="Ana"
score=95
printf 'User: %-10s Score: %03d\n' "$name" "$score"
```

### 3.24 Numeric Strings

Even numeric-looking values are still strings until used in arithmetic.

```bash
x="42"
echo $((x + 8))
```

### 3.25 Common String Pitfalls

#### Word splitting

```bash
value="a b c"
printf '%s\n' $value
```

This splits into multiple words.

Safer:

```bash
printf '%s\n' "$value"
```

#### Globbing side effects

If a variable contains `*`, unquoted expansion may match filenames.

### 3.26 Section Summary

Key points:

- Strings are central to shell scripting
- Parameter expansion is powerful
- Quote strings safely
- Use `[[ ]]` for cleaner pattern matching in Bash

---

## 3. Arrays
### 3.1 Overview
Arrays are Bash features.

POSIX `sh` does not support arrays the same way.

There are two main Bash array types:

- Indexed arrays
- Associative arrays

### 3.2 Indexed Arrays
```bash
fruits=(apple banana cherry)
```

Access an item:

```bash
echo "${fruits[0]}"
```

### 3.3 Explicit Index Assignment
```bash
fruits[0]="apple"
fruits[1]="banana"
fruits[2]="cherry"
```

### 3.4 All Elements
```bash
echo "${fruits[@]}"
```

### 3.5 Array Length
Number of elements:

```bash
echo "${#fruits[@]}"
```

Length of a single element:

```bash
echo "${#fruits[0]}"
```

### 3.6 Append to Array
```bash
fruits+=(orange)
```

### 3.7 Iterate Over Array
```bash
for fruit in "${fruits[@]}"; do
  echo "$fruit"
done
```

### 3.8 Iterate Over Indexes
```bash
for i in "${!fruits[@]}"; do
  echo "$i -> ${fruits[i]}"
done
```

### 3.9 Delete an Element
```bash
unset 'fruits[1]'
```

Note that this leaves a gap in indexes.

### 3.10 Rebuild Dense Array
```bash
fruits=("${fruits[@]}")
```

### 3.11 Slice an Array
```bash
numbers=(10 20 30 40 50)
echo "${numbers[@]:1:3}"
```

### 3.12 Read Command Output into Array
Preferred with `mapfile` or `readarray`:

```bash
mapfile -t lines < file.txt
```

Or from command output:

```bash
mapfile -t services < <(systemctl list-units --type=service --no-legend | awk '{print $1}')
```

### 3.13 Associative Arrays
Declare first:

```bash
declare -A ages
```

Assign values:

```bash
ages[alice]=30
ages[bob]=28
```

Access:

```bash
echo "${ages[alice]}"
```

### 3.14 Iterate Associative Array Keys
```bash
for key in "${!ages[@]}"; do
  echo "$key -> ${ages[$key]}"
done
```

### 3.15 Test If Key Exists
```bash
if [[ -v 'ages[alice]' ]]; then
  echo "Key exists"
fi
```

### 3.16 Array from Positional Parameters
```bash
args=("$@")
```

### 3.17 Difference Between `${array[*]}` and `${array[@]}`
Quoted:

- `"${array[@]}"` preserves elements separately
- `"${array[*]}"` joins elements into one string using `IFS`

### 3.18 Array Example Script
```bash
#!/usr/bin/env bash
set -euo pipefail

servers=(web1 web2 db1)
for server in "${servers[@]}"; do
  printf 'Checking %s\n' "$server"
done
```

### 3.19 Common Array Operations Table
| Operation | Example |
| --- | --- |
| Create indexed array | `items=(a b c)` |
| Append element | `items+=(d)` |
| Access element | `${items[1]}` |
| Array length | `${#items[@]}` |
| Indexes | `${!items[@]}` |
| Remove element | `unset 'items[2]'` |
| Declare associative | `declare -A map` |
| Access map value | `${map[key]}` |

### 3.20 Reading File into Array Safely
```bash
mapfile -t users < users.txt
for user in "${users[@]}"; do
  printf 'User: %s\n' "$user"
done
```

### 3.21 Pitfalls
- Arrays are Bash-specific
- Unquoted array expansions can split unexpectedly
- Sparse indexes may surprise you

### 3.22 Section Summary
Arrays help manage grouped data cleanly in Bash.

---
