# Day 06 – Linux File I/O Practice

## Create File

### Create an empty file

```bash
touch notes.txt
```

**Observation:** Created an empty file named `notes.txt`.

---

## Write Text to File

### Write first line

```bash
echo "Learning Linux file operations." > notes.txt
```

**Observation:** Added the first line to the file.

### Append another line

```bash
echo "Practicing file read and write commands." >> notes.txt
```

**Observation:** Added a new line without overwriting existing content.

### Use tee command

```bash
echo "Using tee command for output and file writing." | tee -a notes.txt
```

**Observation:** Displayed output on the screen and appended it to the file at the same time.

---

## Read File Content

### Display complete file

```bash
cat notes.txt
```

**Observation:** Displayed the entire contents of the file.

### Display first two lines

```bash
head -n 2 notes.txt
```

**Observation:** Showed the first two lines of the file.

### Display last two lines

```bash
tail -n 2 notes.txt
```

**Observation:** Showed the last two lines of the file.

---

## Quick Learnings

* `touch` creates an empty file.
* `>` writes content and overwrites existing data.
* `>>` appends new content to the existing file.
* `tee -a` writes to a file and displays output simultaneously.
* `cat` displays the full file.
* `head` shows the beginning of a file.
* `tail` shows the end of a file.

---

## Screenshots

Screenshots from today's practice are available in the `screenshots/` folder.

* Tee Command.png
* Cat Output.png
* Head Command.png
* Tail Command.png

---

## Summary

Practiced basic Linux file operations by creating a file, writing content, appending new lines, and reading the file using different commands. This exercise helped reinforce the fundamentals of handling text files in Linux.
