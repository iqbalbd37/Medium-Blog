### *Stream Editor (sed) — A Complete Guide for Beginners*

---

> *Ever needed to change one word across thousands of lines in a file — instantly? That's exactly what **sed** was built for.*

---

### What is Stream Editor?

**Stream Editor**, commonly known as **sed**, is a powerful **Unix/Linux text-processing tool** that reads text from an input stream (a file or a pipe), modifies it according to rules you define, and sends the result to an output stream.

Unlike traditional editors, sed works **line by line** — it reads one line, processes it, outputs it, then moves to the next.

> Think of it as a **factory assembly line for text** — each line passes through, gets modified, and moves on.

---

### What Does "Stream" Actually Mean

A **stream** is simply a **continuous flow of data**.

```
File  ──▶  [sed processes]  ──▶  Output
 (input)      (edit)             (output)
```

Here's the key insight:

- sed does **NOT** load the entire file into memory
- It reads **one line at a time**
- Processes it
- Outputs it
- Moves to the next line

That's the "stream" — continuous, efficient, and memory-friendly.

---

### The Core Syntax

```bash
sed [options] 'script' input_file
```

Breaking down a typical command:

```bash
sed 's/find/replace/flag' filename
     │  │     │       │
     │  │     │       └── flag (g = global, i = case-insensitive)
     │  │     └────────── replacement text
     │  └──────────────── what to search for
     └─────────────────── substitute command
```

---

### A Simple Example

> Input file: `fruits.txt`

```
apple
banana
apple
cherry
```

#### Basic replacement:

```bash
sed 's/apple/mango/' fruits.txt
```

**Output:**

```
mango
banana
apple        ← only first match per line
cherry
```

#### Global replacement (g flag):

```bash
sed 's/apple/mango/g' fruits.txt
```

**Output:**

```
mango
banana
mango
cherry
```

#### In-place edit (-i):

```bash
sed -i 's/apple/mango/g' fruits.txt
```

The **original file itself is modified**.

---

### How sed Works Internally

```
┌─────────────┐
│  Input Line │
└──────┬──────┘
       ▼
┌─────────────────┐
│  Pattern Space  │  ← line arrives here, gets processed
│  (working area) │
└────────┬────────┘
         ▼
┌─────────────────┐
│  Output Line    │
└─────────────────┘
```

1. **Pattern Space** — every line arrives here first
2. **Commands** — replacement, deletion, or insertion happens here
3. **Output** — the processed line is sent out

**Hold Space** is an auxiliary buffer used for advanced operations.

---

### Common sed Use Cases

#### 1. Text Substitution

```bash
sed 's/old/new/' file.txt
sed 's/old/new/g' file.txt              # all occurrences
sed 's/old/new/2' file.txt              # only 2nd occurrence
sed 's/old/new/gi' file.txt             # case-insensitive
```

#### 2. Deleting Lines

```bash
sed '3d' file.txt                        # delete 3rd line
sed '2,5d' file.txt                      # delete lines 2–5
sed '/pattern/d' file.txt                # delete lines matching a pattern
```

#### 3. Printing Lines

```bash
sed -n '5p' file.txt                     # print only line 5
sed -n '1,10p' file.txt                  # print lines 1–10
sed -n '/error/p' file.txt               # print lines containing "error"
```

#### 4. Insert & Append

```bash
sed '3i\New line' file.txt               # insert before line 3
sed '3a\New line' file.txt               # append after line 3
```

#### 5. Removing Comments

```bash
sed '/^#/d' config.conf                  # delete lines starting with #
sed 's/#.*//' config.conf                # remove inline comments
```

---

### Real-World Examples

#### Example 1: Change an IP Address

```bash
sed -i 's/192.168.1.100/192.168.1.200/g' config.ini
```

#### Example 2: Update Multiple Files at Once

```bash
sed -i 's/DEBUG/INFO/g' *.log
```

#### Example 3: Print a Block Between Patterns

```bash
sed -n '/ERROR/,/END/p' server.log       # from ERROR to END
```

#### Example 4: Use with Pipes

```bash
cat access.log | sed 's/GET/POST/g' | grep "api"
```

#### Example 5: Strip Windows Line Endings

```bash
sed -i 's/\r$//' file.txt
```

---

### A Brief History

- **sed** = **S**tream **ED**itor
- Created in **1973–74** at Bell Labs by **Lee E. McMahon**
- One of the oldest and most powerful Unix tools
- Included in the **POSIX** standard
- **GNU sed** (the Linux default) adds many extra features

---

### Safety Tips When Using sed

#### 1. Always Be Careful with -i

```bash
# Risky — no backup
sed -i 's/old/new/g' important.conf

# Safe — with backup
sed -i.bak 's/old/new/g' important.conf
```

The `.bak` creates an automatic backup as `important.conf.bak`.

#### 2. Escape Special Characters

```bash
sed 's/\/path\/old/\/path\/new/' file.txt    # escape slashes
# Or change the delimiter
sed 's|/path/old|/path/new|' file.txt
```

#### 3. Do a Dry Run First

```bash
sed 's/old/new/g' file.txt | diff file.txt -   # see what changes
```

---

### Quick Recap

| Feature | Command |
|---------|---------|
| Basic replace | `sed 's/old/new/' file` |
| Global replace | `sed 's/old/new/g' file` |
| In-place edit | `sed -i 's/old/new/g' file` |
| Delete line | `sed '3d' file` |
| Print line | `sed -n '5p' file` |
| Case-insensitive | `sed 's/old/new/gi' file` |
| With backup | `sed -i.bak 's/old/new/g' file` |

---

### Final Thought

> ***sed*** *lets you modify thousands of lines — without ever opening a file, without touching a mouse, and without any manual effort.*

Whether you're a **DevOps engineer** tweaking server configs, a **developer** cleaning up logs, or a **sysadmin** automating file changes — **sed is a superpower worth mastering**.

---

**Tags:** #Linux #Sed #ShellScripting #DevOps #TextProcessing #Unix #CommandLine

**Read the original article on Medium:** [Stream Editor (sed) — A Complete Guide for Beginners](https://medium.com/@iqbalbd37/stream-editor-sed-a-complete-guide-for-beginners-8e9d3ce603a0)
