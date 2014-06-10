---
layout: post
title: "Bash history expansion cheatsheet"
date: 2013-12-02 12:11
categories: general
---

Often I forget the precise letter incantation for a particular 
bash history expansion shortcut and the bash manual page is crying out
for a tabular form of these history expansions. So I decided to make 
one here for quick reference.

## Event Designators (Line entry in history list)

| Shortcut | Purpose | Example |
| ---------|---------|---------|
| ! | Start a history substitution | ! |
| !n | Refer to command *n* in history | !5 |
| !-n | Refer to current command minus *n* | !-5 |
| !! | Refer to previous command | !! |
| !string | Refer to the most recent command *starting with* "string" | !ls |
| !?string? | Refer to the most recent command *containing* "string" | !?ls? |
| !# | The entire command line type so far | echo !# back at ya |

## Word Designators - Word Inside Particular History Entry

*Start with a ":" to select word*

| Designator | Purpose | Example | Comments | 
| -----------|---------|---------|----------|
| 0 | Zeroth word; the command word | !:0 | |
| n | The nth word | !:5 | For the 5th word | 
| ^ | The first argument i.e., word 1 | !:^ | Or just "!^" seems to work |
| $ | Last argument | !:$ | !$ also seems to work |
| % | Last match from ?string? search | !% | Need to have performed ?bla? search first |
| x-y | A range of words | !:1-3 | Abbr. to "!:-3" will start search from 0 |
| \* | All words but zeroth | !\* | I.e., all arguments |
| x\* | All words from x to end | | 
| x- | All words to end except last word | |

## Word Modifiers

*After the optional word designator, you may use one more more modifiers prefixed with ":"*

_Note: this list is not exhaustive, see manual page for full list_

| Modifier | Purpose | Example |
|----------|---------|---------|
| h | Remove file name leaving only directory (head) | !^:h |
| t | Remove directory leaving only file (tail) | !^:t |
| r | Remove trailling suffix of form .xxx, leaving only base name | !^:r |
| e | Remove all but trailing suffix | !^:e |
| s/old/new/ | Replace old for new in event line. Any deliminator can be used in place of /. "&" means previous word in new substitution | !^:s/old.txt/new.txt/ |
| & | Repeat the previous substitution | !^:& |
| g | Apply changes over whole line in conjunction with :s | !^:sg/old/new/ |
