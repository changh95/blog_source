---
title: Ctrl과 Caps lock 위치 바꾸기 (Windows / Linux)
date: 2021-03-25 10:38:19
mathjax: true
tags: 
- Tricks
categories: 
- [SW, Tools & Tricks]
excerpt: Ctrl과 Caps lock 위치 바꾸기 
---

## 바꾸는 이유

- 주 키보드 중 하나가 해피해킹이라서 (i.e. ctrl이 자체적으로 caps lock위치에 있음)
  - 이걸 쓰다가 보통의 키보드나 노트북을 쓸 때 항상 헷갈림
- Ctrl+C, Ctrl+V, Ctrl+W 모두 캡스락 위치가 손목이 더 편하다.


&nbsp;

---
## Windows

- [링크](https://johnhaller.com/useful-stuff/disable-caps-lock)로 들어간다.
- `Swap Caps Lock and the left Control key`의 파일을 다운로드하고, 레지스트리 등록.

&nbsp;

---
## Linux (Ubuntu)

- `gnome-tweak-tools` 다운로드.
  - i.e. `sudo apt install gnome-tweak-tools`
- 메뉴에서 'Tweaks' 실행
  -  그 후, Keyboard & Mouse > Keyboard > Additional Layout Options > Ctrl position로 탐색
  -  `Swap ctrl and caps lock` 체크.

{% asset_img "swap.png" "swap" %}

> 중대 Update!
> 노트북에 해피해킹을 연결하면, 해피해킹 키보드의 ctrl 버튼이 caps lock으로 맵핑되어버린다 ㄷㄷ... 해피해킹을 노트북에도 연결하는 경우에는 'Caps lock as ctrl' 옵션을 하는게 더 편하다.


