---
id: 01JBR7GTJNN684VBW3NVFE86E8
modified: 2024-11-03T01:23:27-04:00
title: Creating & Deleting Files
tags:
  - linux
  - programming
  - roadmaps
  - terminal
---
# Creating Files

Creating files in Linux is about making new blank or filled files on your computer. You can use commands like `touch` to create an empty file, `echo` to make a file with some text inside, or `cat` to type directly into a new file. These commands help you set up and save your documents or data.

Here’s an example of file creation with the `touch` command:

```
touch newfile.txt
```

and with `cat` command:

```
cat > newfile.txt
```

Both these commands create a new “newfile.txt” if it does not already exist.

# Deleting Files

Deleting files in Linux means getting rid of unwanted or unnecessary files from your computer. You use the `rm` command to delete a file, and it’s permanent, so be careful. You can also use `rm -i` (interactive) to ask for confirmation before deleting, which helps prevent accidental loss of important files.

```
# Deletes the file named example.txt
rm example.txt
```

```
# Ask for confirmation
rm -i [filename]
```

```
# Removes an empty directory
rmdir [directory] 
```

Learn more from the following resources:

Free Resources

---

- [ArticleLinux rm Command: File Removing](https://labex.io/tutorials/linux-linux-rm-command-file-removing-209741)