---
title: "Notes on Redirection in Bash"
subtitle: "What does `exec chroot . sh <dev/console >/dev/console 2>&1` mean?"

summary: "What does `exec chroot . sh <dev/console >/dev/console 2>&1` mean?"


date: 2024-01-25T10:50:00+08:00

lastmod: 2024-01-25T10:50:00+08:00

draft: false

featured: false

authors:
  - admin

categories:
  - Linux
  - Shell
---

## Motivation

What does the following command mean? I don't fully understand the redirection before reading docs.

```shell
exec chroot . sh <dev/console >/dev/console 2>&1
```

## Read the Docs

In `bash(1)`, general introduction:

> Redirection  allows  commands'  file  handles  to be duplicated, opened, closed, made to refer to different files, and can change the files the command reads from and writes to.  Redirection may also be used to modify file handles in the current  shell  execution environment.

### Redirecting Input and Output

The following docs explains `<`  in our example.

> `Redirecting Input Redirection``: of input causes the file whose name results from the expansion of word to be opened for reading on file descriptor n, or the standard input (file descriptor 0) if n is not specified.The general format for redirecting input is: [n]<word

The following docs explains `>`  in our example.

> Redirection  of  output causes the file whose name results from the expansion of word to be opened for writing on file descriptor n, or the standard output (file descriptor 1) if n is not specified.  If the file does not exist it is created; if it does  exist it is truncated to zero size.The general format for redirecting output is:[n]>word

Note that the order of redirections is significant.  For example, the command

```bash
              ls > dirlist 2>&1
```

directs both standard output and standard error to the file dirlist, while the command

```bash
              ls 2>&1 > dirlist
```

directs only the standard output to file dirlist, because the standard error was duplicated from the standard output  before  the standard output was redirected to dirlist.

### Duplicating File Descriptor

The redirection operator

```bash
              [n]<&word
```

is used to duplicate input file descriptors.  If word expands to one or more digits, the file descriptor denoted by n is made  to be  a  copy  of that file descriptor.  If the digits in word do not specify a file descriptor open for input, a redirection error occurs.  If word evaluates to -, file descriptor n is closed.  If n is not specified, the standard input (file descriptor  0)  is used.

The operator

```bash
              [n]>&word
```

is  used similarly to duplicate output file descriptors.  If n is not specified, the standard output (file descriptor 1) is used.If the digits in word do not specify a file descriptor open for output, a redirection error occurs.  If word evaluates to -, file descriptor  n  is  closed.  As a special case, if n is omitted, and word does not expand to one or more digits or -, the standard output and standard error are redirected as described previously.

Some simplication can be made.

This  construct allows both the standard output (file descriptor 1) and the standard error output (file descriptor 2) to be redirected to the file whose name is the expansion of word.There are two formats for redirecting standard output and standard error: `&>word` and `>&word` Of the two forms, the first is preferred.  This is semantically equivalent to `>word 2>&1`

### Opening File Descriptors for Reading and Writing

The redirection operator `[n]<>word` causes  the  file  whose name is the expansion of word to be opened for both reading and writing on file descriptor n, or on file descriptor 0 if n is not specified.  If the file does not exist, it is created.
