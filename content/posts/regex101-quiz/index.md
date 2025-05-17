---
title: "Regex 101 Quiz"
date: 2023-08-08T10:55:00+08:00
summary: "finish the quiz of Regex 101"
showtoc: false
categories:
- Regex
authors:
- admin
---

## Introduction

+ [Regex 101 Website](https://regex101.com/)
  
## 1. Word Boundary

+ 忽略大小写?

```javascript
/\bword\b/ // \b is not quantifiable
```

## 2. Capitalizing I

+ g:Don't return after first match
+ Use substitution to replace every occurrence of the word i with the word I (uppercase, I as in me). E.g.: i'm replacing it. am i not? -> I'm replacing it. am I not?.

A regex match is replaced with the text in the Substitution field when using substitution.

```javascript
/\bi\b/g
```

## 3 UPPERCASE CONSONANTS

With regex you can count the number of matches. Can you make it return the number of uppercase consonants (B,C,D,F,..,X,Y,Z) in a given string? E.g.: it should return 3 with the text ABcDeFO!. Note: Only ASCII. We consider Y to be a consonant!

Example: the regex /./g will return 3 when run against the string abc.

```javascript
/[B-DF-HJ-NP-TV-Z]/gu
```

## 6 BROKEN KEYBOARD

Oh no! It seems my friends spilled beer all over my keyboard last night and my keys are super sticky now. Some of the time whennn I press a key, I get two duplicates.

Can you ppplease help me fix thhhis?

```javascript
/(.)\1\1/g // Pattern
$1 // Substitution
```

## 7 Validate AN IP

```javascript
^(?:(?:2[0-4]\d|25[0-5]|[01]?\d{1,2})\.){3}(?:2[0-4]\d|25[0-5]|[01]?\d?\d)$
```

## 8 HTML TAGS

Strip all HTML tags from a string. HTML tags are enclosed in < and >.

The regex will be applied on a line-by-line basis, meaning partial tags will need to be handled by the regex. Don't worry about opening or closing tags; we just want to get rid of them all.

Note: This task is meant to be a learning exercise, and not necessarily the best way to parse HTML.

```javascript
/<[^<>\n]*>|<[^>\n]*|[^<\n]*>/mg
<[^<>\n]*>?|<?[^<>\n]*> // optimized version
```

## 10:FOLLOWED BY

For every occurrence of the char #, match the previous character and save it in a group (backreference).

Example: for the text "a#bc# -#", set backreferences with a, c and -.

You are not allowed to consume the hash character.

```javascript
/(.)(?=#)/g
```
