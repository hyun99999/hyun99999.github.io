---
title:  "iOS) prepareForReuse() 사용해서 셀을 초기화해보자"
categories:
- iOS

date:   2021-08-01  15:00:00 +0900
author_profile: false
---
우리는 셀을 재사용하면서 특정 문제점을 경험해봤을 것이다. 바로 셀이 재사용될 때 발생하는 문제점이다.

### 원인

`tableView(_:cellForRowAt:)` delegate 메서드에서 사용하는 `dequeueReusableCell(withIdentifier:for:)` 메서드에서 셀이 resue 된다.

셀에 configure 되는 내용은 다르지만 셀 자체는 재사용되기 때문에 content 와 무관한 것들 예를들어 셀의 alpha , editing, selection sate 등까지 재사용하게 된다. 

### 해결방법

`prepareForeReuse()` 를 통해서 재사용되는 셀의 속성을 초기화해주면 된다.

```swift
// tableViewCell.swift
override func prepareForReuse() {
    super.prepareForReuse()

        // 셀을 초기화 해주는 코드.
}
```

<img src ="https://user-images.githubusercontent.com/69136340/127811633-e6130730-4ea6-468b-9828-64696029c33c.png" width ="500">

다음은 아래 출처에서 가져온 흐름도이다. `prepareForeReuse()` 가 언제 호출되는지 확인해보자.

여기서 놓치지 말아야하는 부분은 모든 재사용되는 셀에 대해서 초기화해버리면 이미 push out  된 셀도 초기화가 된다 그래서 데이터를 확인해서 다시금 보여주도록 해야한다.

### **또 다른 문제점**

오픈 api 를 활용해서 영화 포스터를 가져오는 예제를 진행했을 때 출처의 글쓴이가 겪는 현상과 같은 현상을 겪었다.

> "이미지가 포함되는 경우를 생각해보자. 아마 실제 상용되는 앱에서도 가끔 경험했을 수 있는데, 인터넷 상황이 좋지 않은 상황에서(이미지 로드에 시간이 더 걸리는 상황에서) 너무 빠르게 스크롤을 하다보면 이미지가 있어야할 곳이 아닌 다른 셀에서 보여지는 경우가 종종있다."
출처: [https://sihyungyou.github.io/iOS-dequeueReusableCell/](https://sihyungyou.github.io/iOS-dequeueReusableCell/)

나 역시 너무 빠르게 스크롤하게 되면 이미지가 있어야할 곳이 아닌 다른 셀에 보여지는 경우가 있었고 이 역시 셀이 재사용되기 때문에 텍스트와 이미지가 올바르게 위치하지 않는 경우였다.

<img src ="https://user-images.githubusercontent.com/69136340/127811467-385cc830-9a20-48ed-99b0-16142f686c9b.gif" width ="250">

(중간 중간에 이미 나온 포스터의 이미지가 보인다)

### 해결방법

이 역시 `prepareForeReuse()` 로 해결가능했다.

image 를 nil 로 초기화해주거나 placehold 가 될 이미지로 설정해주었다.

### 의문

여기서 드는 의문은 위의 문제가 있다면 굳이 `dequeueReusableCell(withIdentifier:for:)` 메서드를 사용해야할까 라는 의문이 생길법한대

10개의 셀을 만들어주기위해서 10개의 원소를 가진 cell 인스턴스 배열을 만드는 방법도 있을 수 있겠다. 하지만 그 이상이 되면 메모리 사용량 측면에서 매우 비효율적일 것이다. 그래서 애플에서도 위의 메서드를 사용하는 것을 권장한다.

### 출처

[iOS) dequeueReusableCellWithIdentifier 과정과 사용이유](https://sihyungyou.github.io/iOS-dequeueReusableCell/)
[Apple Developer - prepareForReuse()](https://developer.apple.com/documentation/uikit/uitableviewcell/1623223-prepareforreuse)
