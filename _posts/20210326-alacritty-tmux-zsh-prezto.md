---
title: 리눅스 터미널 업그레이드 - Alacritty + zsh + prezto + tmux + (NeoVim)!
date: 2021-03-26 11:41:13
mathjax: true
tags: 
- Terminal
- Bash
- zsh
- Alacritty
- prezto
- vim
- tmux
categories: 
- [SW, Tools & Tricks]
excerpt: Alacritty + zsh + prezto + tmux로 터미널 관리하기
---

[참조한 링크](https://dev.to/hlappa/my-web-development-environment-alacritty-tmux-neovim-4pd2)

## Alacritty

{% asset_img "alacritty.png" "Alacritty" %}

[Alacritty](https://github.com/alacritty/alacritty)는 Rust로 짜여진 OpenGL 가속을 사용하는 터미널 에뮬레이터다.

기존의 bash terminal을 놔두고 alacritty를 선택한 이유는?
- OpenGL 가속으로 인해 로깅이 훨씬 빠르다고 한다.
  - 실제로 `tree`를 찍어보면 조금 더 빠르다.
- 색깔을 바꿀 수 있다!
  - 커스터마이징은 꼭 해야지 :)

설치 가이드를 쫒아서 하다보니 금방 실행할 수 있었다.

설치 과정에서 눈여겨 본 점은:
- rustup 컴파일러를 설치해야한다는 점
- `cc`가 gcc에 연결되어야하는 점.
  - 이게 되어있지 않다면 빌드 과정 중에 `linker not found 'cc'` 같은 에러가 나타난다.
  - 그럴 때는 `cc --version`을 쳐서 확인해보자.
    - 에러가 뜬다면, gcc를 다시 설치해야한다.
    - 간단한게 `sudo apt remove gcc`, 그 후 `sudo apt install gcc`를 수행해서 gcc를 다시 설치한다.
    - Alacritty build는 `cargo clean`으로 클린 빌드를하고, 다시 `cargo build --release`로 빌드하자. 내 경우는 gcc를 다시 깔고 클린빌드 후 다시 빌드하니 해결되었다.

폰트 설정을 기대해서 JetBrains Mono 폰트를 써보려고 했는데 깨진다 :(

&nbsp;

---

## zsh

[zsh](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH)는 Ubuntu Linux에서 기본으로 제공하는 bash 쉘의 기능을 포함하여 편리한 기능이 여러가지 추가되어 있는 Shell 환경이다.

`sudo apt install zsh`로 설치하였다.
그 후, 기본 shell을 zsh로 변경하기 위해 `chsh -s $(which zsh)`를 수행했다.

보통 많이들 ['oh-my-zsh'](https://ohmyz.sh/) 많이 설치하는데, 나는 다른 zsh 프레임워크를 설치했다. 이유는 다음 섹션에서 설명한다.

&nbsp;

---

## prezto

[prezto](https://github.com/sorin-ionescu/prezto)는 zsh 프레임워크이다.

oh-my-zsh를 쓰지않고 prezto를 사용하게 된 것은 순전히 [Missing Semesters 강의]() 때문이다 ㅋㅋ 유명한 프레임워크인만큼 거의 대부분의 기능들은 oh-my-zsh와 비슷하게 제공될 것이라고 생각했고, 대신 디자인이 꽤 맘에 들었기 때문이다.

module은 아직 많이 깔지는 않았다.
아마 아래의 것들을 사용하지않을까 싶다
- autosuggestions
- docker
- environment
- git
- history
- python
- ssh
- tmux


&nbsp;

---

## tmux

[tmux](https://github.com/tmux/tmux/wiki)는 터미널 창을 여러개로 쪼개서 쓸 수 있게 해준다.

사실 아직 많이 익숙하지는 않다. 
튜토리얼을 보고 익숙해지려는 중

&nbsp;

---

## Neovim

도입할지 말지 고민중인 에디터.
vscode를 버려야 할 이유가 있을까?
굳이 버리는건 아닌가? 흐음

&nbsp;

---

## 결과물 

{% asset_img "result.png" "result" %}
