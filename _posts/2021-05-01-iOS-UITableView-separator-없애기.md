---
title:  "iOS) UITalbeView Separator 없애기"
categories:
- iOS

date:   2021-05-01 23:40:00 +0900
author_profile: false
---
UITalbeView Separator 없애기

두가지 방법이 있다.

### 1. storyboard
separator 를 None 으로 해주었다.

<img src = "https://user-images.githubusercontent.com/69136340/116786206-56bfcb80-aad8-11eb-86d5-385be08af177.png" width="300">


### 2. code
`separatorStyle` 속성을 설정해주었다.

```swift
tableView.separatorStyle = .none
```
