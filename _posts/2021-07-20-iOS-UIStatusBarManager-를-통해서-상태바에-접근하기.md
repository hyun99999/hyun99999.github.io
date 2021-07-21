---
title:  "iOS) UIStatusBarManager 를 통해서 상태바에 접근하기"
categories:
- iOS

date:   2021-07-20  02:08:00 +0900
author_profile: false
---
UIStatusBarManager

- iOS 13 으로 변경됨에 따라 KVC 를 통한 상태바에 대한 접근을 금지하고 있다. 대신 UIStatusBarManager 로 접근 가능합니다.

### 구현

```swift
if #available(iOS 13.0, *) {
            let margin = view.layoutMarginsGuide
            let window = UIApplication.shared.windows.first { $0.isKeyWindow}
            let statusBarManager = window?.windowScene?.statusBarManager

            let statusBarView = UIView(frame: statusBarManager?.statusBarFrame ?? CGRect.zero)
            statusBarView.backgroundColor = color

            view.addSubview(statusBarView)
            statusBarView.translatesAutoresizingMaskIntoConstraints = false

            NSLayoutConstraint.activate([
                statusBarView.topAnchor.constraint(equalTo: view.topAnchor),
                statusBarView.widthAnchor.constraint(equalTo: view.widthAnchor, multiplier: 1.0),
                statusBarView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
                statusBarView.bottomAnchor.constraint(equalTo: margin.topAnchor)
            ])

        }
```

### 개발자문서

`UIStatusBarManager` object 를 사용하여 연결된 scene 에 대한 status bar 의 현재 구성을 가져온다. `UIStatusBarManager` object 를 직접 만들지 않는다. 대신 `UIWindowScene` 개체의 statusBarManager 속성에서 존재하는 개체를 검색한다. 이 개체를 통해서 status bar 의 구성을 수정하지 않는다. 대신 각 UIViewController 개체에 대해 status bar 구성을 개별적으로 설정한다. 예를들면 status bar 의 기본 visibility 를 수정하려면 `preferStatusBarHidden` 속성을 view controller 에서 재정의 한다.

### 출처

[Apple Developer - UIStatusBarManager](https://developer.apple.com/documentation/uikit/uistatusbarmanager)
