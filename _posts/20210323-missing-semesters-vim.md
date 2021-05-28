---
title: Missing Semesters (2020) 3강 Vim
date: 2021-03-23 22:38:19
mathjax: true
tags: 
- Missing Semesters
- MIT
- Vim
- Bash
categories: 
- [  SW, MIT Missing Semesters 정리]
excerpt: Missing Semesters (2020) - Vim 정리
---

{% youtube a6Q8Na575qc %}

&nbsp;

## Vim modes

- **Normal mode**
  - 목적: Navigation
  - 다른 mode에서 `esc`를 눌러서 이동
- **Insert mode**
  - Normal mode에서 `i`를 눌러서 이동
  - 목적: Typing text into the text buffer
- **Replace mode**
  - Normal mode에서 `r`을 눌러서 이동
- **Visual mode**
  - Normal mode에서 `v`를 눌러서 이동
  - Visual line
    - Normal mode에서 `shift + v`를 눌러서 이동
  - Visual block
    - Normal mode에서 `ctrl + v`를 눌러서 이동
- **Command-line mode**
  - Normal mode에서 `:`을 눌러서 이동

이것을 보면 `esc`를 많이 누르게 될 것을 알고 있다. `esc`키를 누르기 위해 왼손 새끼손가락을 쭉 올리는거보다, 종종 캡스락 키에 `esc` 키를 바인딩 하기도 한다.

## Open file

- `vim`을 눌러서 vim을 키고 끌 수 있다.
- `vim file.md`를 써서 파일을 열 수 있다.
- vim에서 파일을 읽거나 저장하거나, vim을 종료하려고 할 때는 command-line mode를 사용한다.
  - Normal mode에서 `:`를 눌러서 command-line mode로 와야한다.
  - `:quit`을 누르면 끌 수 있다.
  - `:q`는 짧은 버전이다.
  - `:w`를 통해 write, 즉 저장할 수 있다.
  - `:help xyz`를 눌러서 xyz에 대한 document를 볼 수 있다.

## Buffers, tabs, windows

- VSCode는 동시에 여러개의 파일을 열 수 있다.
  - Vim 도 가능하다. 여러개의 buffer를 열 수 있다.
- 기존의 에디터 프로그램들에서 사용하는 개념들은 vim에서는 다음과 같이 맵핑된다.
  - File -> buffer
  - Tab -> Tab
  - Window -> Window
- Vim도 여러개의 buffer (i.e.file)을 열 수 있고, 여러개의 탭을 열 수 있고, 여러개의 윈도우를 사용할 수 있다.
- Buffer는 윈도우 없이 여러개를 열 수 있다.
- Window를 열기 위해서는 `:sp` 커맨드를 사용한다.
  - `:q` 또는 `:quit`을 사용해서 윈도우를 닫는다.
  - `:qa` (i.e. quit all)을 사용해서 모든 윈도우를 한번에 닫을 수 있다.
- `tabnew`를 사용하여 새 tab을 열 수 있다.
  - 하나의 tab에는 여러 윈도우를 띄울 수 있다. 

## Buffer

- Navigate around buffer in normal modeS?
  - `hjkl` buttons to move cursors
    - Instead of `hhhhh`, we can also do `5h`.
  - `w` and `b` to move cursor by words
    - `e` to move cursor to the end of the word
  - `0` to move by line
    - `$` to move to the end of a line
    - `^` to move to first non-empty character of the line
  - `ctrl + U` moves up the page, `ctrl + D` moves down the page.
  - `G` moves to the bottom of the page, `gg` moves to the top of the page
  - `L` moves your cursor to the bottom of the screen, `M` to the middle of the screen, `H` to the top of the screen
- Find words
  - `fo` to find the first 'o'.
  - `Fo` to find the first 'o' backwards.
  - `to` to find the first 'o', but place the cursor right before it.
  - `To` to find the fisrt 'o', but place the cursor right before it.
  - `/word` to search for a word
- Editing
  - `o` in normal mode will automatically activate insert mode after making a new line
  - `O` in normal mode will automatically activate insert mode after making a new line above the cursor.
  - `dw` to delete a word
    - We can do `7dw` to delete 7 words.
  - `de` to delete to end of a word
  - `dd` to delete a line. Repeating the command affects the entire line.
  - `ce` to change to end of a word. It deletes the content to end of a word and automatically activate insert mode.
  - `cc` to change a line. Repeating the command affects the entire line.
  - `x` deletes a character
  - `ra` replace a character by 'a'
  - `u` Undo action
  - `R` redo action
  - `y` to copy (yank), and `p` to paste.
    - `yy` to copy a line
    - `yw` to copy a word
    - To select a block of text, we need to enter the visual mode. Use `v` to enter the visual mode. Use `V` to enter the visual line mode. Use `ctrl + V` to enter the visual block mode.
      - Use `hjkl` to navigate cursor and select the block of text
      - then, use `y` to copy the block of text and return to normal mode.
  - `~` to change the case
- Modifer commands 
  - Use `ci(` or `ci[` to change the contents inside the brackets (i for inside)
  - Use `%` to jump back and forth the parentheses.
  - Use `da(` to delete the entire content + the parentheses. (a for around)
- Repeat command
  - `.` to repeat the previous command.
