---
title: Missing Semesters (2020) 4강 Data Wrangling
date: 2021-03-25 15:38:19
mathjax: true
tags: 
- Missing Semesters
- MIT
- Data Wrangling
- Bash
categories: 
- [  SW, MIT Missing Semesters 정리]
excerpt: Missing Semesters (2020) - Data Wrangling 정리
---

{% youtube sz_dsktIjt4 %}

&nbsp;

## Data wrangling?

- Changing the format of data
  - Image? Graph? YAML? JSON?
  - Using `|` operator in bash is already a form of data wrangling!

## Get log data and do something with it

- To do data wrangling, we need some data.
  - In this example, the instructor pulled some log data from remote server and did some data wrangling on it.
  - `ssh` to control servers remotely
  - `grep` to grab content
  - `less` to fit the loooong data into a single page, instead of flooding lines.
  - `ssh '...... | grep "Disconneced from"' > ssh.log` to grab anything that contains 'Disconnected from" in the line and save it to 'ssh.log' file.
  - `cat ssh.log | less` to view the log in a pager.
- `sed` is a stream editor that enables you make changes to the contents of the stream.
  - `cat ssh.log | sed 's/.*Disconnected from//'` will grab the contents from the 'ssh.log', and apply `sed`'s regular expression (regex).
    - Here, we have the 'user name' data that we want to get, after the 'Disconnected from...' string.
    - `s` is an expression to substitute the string. It takes 2 arguments, the first one being search, and the second one being the replacement string.
    - `s/ aaa / bbb /` will replace the 'aaa' string into 'bbb' string.
    - `.` means 'any single character'
    - `*` means 'zero or more of this character'
    - So `.*` means 'zero or more of any of the following characters'... of 'Disconnected from'
  - `echo 'aba' | sed 's/[ab]//'` will return 'ba'.
    - This means 'replace either a or b that appears first time with an empty space'.
    - `[ab]` means either a or b.
    - So `echo 'bba' | sed 's/[ab]//'` will return 'ba' as well.
    - `echo 'aba' | sed 's/[ab]//g'` will return nothing (i.e. removed all a and b). The `g` will do the action as many times it matches.
  - `echo 'abcaba' | sed -E 's/(ab)*//g'` will return 'ca'.
    - means 'I want zero or more of the string 'ab' and replace it with empty space'.
      - 'ab'c'ab'a.
    - So it's not 'either a or b'. It will be looking for 'ab'
    - `-E` is to allow usage of old-fashioned regex. 
  - `echo 'abcababc' | sed -E 's/(ab|bc)*//g'` will return 'cc'.
    - Although we stated to remove 'ab' or 'bc', the c-characters still remain.
    - This is because in the 'abc', the 'ab' is already removed, so when we try to look for 'bc', there is only 'c' left in the string so we could not find 'bc'.
  - 