---
title:  "iOS) NVActivityIndicatorView 라이브러리 사용해보기"
categories:
- iOS

date:   2022-01-01  22:10:00 +0900
author_profile: false
---
- 서버통신하는 동안 유저가 로딩되고 있음을 보여줄 수 있도록 activity indicator 를 제공하는 라이브러리를 사용해보자.

# NVActivityIndicatorView

로딩 애니메이션을 제공하는 오픈 라이브러리

[GitHub - ninjaprox/NVActivityIndicatorView: A collection of awesome loading animations](https://github.com/ninjaprox/NVActivityIndicatorView)

아래와 같은 로딩 애니메이션을 제공한다.
<img src="https://user-images.githubusercontent.com/69136340/147851256-fdd48036-ed2c-445d-b88c-b78145467814.gif" width ="250">

<img src="https://user-images.githubusercontent.com/69136340/147851257-61029a63-6249-401e-9dc6-cfb25b5e50a5.png" width ="600">

### pod install

```swift
pod 'NVActivityIndicatorView'
```

### initializer

<img src="https://user-images.githubusercontent.com/69136340/147851298-7a8ce659-6485-47bc-8b56-933fe1cb4d2b.png" width ="500">

### 시작하기

- 불투명한 배경을 가지는 activity indicator 만들기

```swift
import NVActivityIndicatorView

// lazy: 사용되기 전까지 연산되지 않는다. 로딩이 불필요한 경우에도 메모리를 잡아먹지 않는다.
lazy var loadingBgView: UIView = {
        let bgView = UIView(frame: CGRect(x: 0, y: 0, width: UIScreen.main.bounds.width, height: UIScreen.main.bounds.height))
        bgView.backgroundColor = .bottomDimmedBackground
        
        return bgView
}()
    
lazy var activityIndicator: NVActivityIndicatorView = {
        // ✅ activity indicator 설정
        let activityIndicator = NVActivityIndicatorView(frame: CGRect(x: 0, y: 0, width: 40, height: 40),
                                                        type: .ballBeat,
                                                        color: .mainColorNadaMain,
                                                        padding: .zero)
        activityIndicator.translatesAutoresizingMaskIntoConstraints = false
        
        return activityIndicator
}()
```

- 불투명 뷰 추가. 애니메이션 시작.

```swift
private func setActivityIndicator() {
        // 불투명 뷰 추가
        view.addSubview(loadingBgView)
        // activity indicator 추가
        loadingBgView.addSubview(activityIndicator)
        
        NSLayoutConstraint.activate([
            activityIndicator.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            activityIndicator.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
        
        // 애니메이션 시작
        activityIndicator.startAnimating()
}
```

- dispatch queue 에서 제공하는 main 큐는 serial 큐이기 때문에 직렬적으로 작업을 처리한다. 그래서 `setActivityIndicator()` 가 실행되고 aync 즉, 비동기적으로 작업이 마칠때까지 기다리지 않고 리턴하여 다음 작업을 main 큐에 넣게 되는 구조이다. 그리고 serial 큐이기 때문에 애니메이션이 시작되고 끝나는 순서가 보장된다.

```swift
DispatchQueue.main.async {
        self.setActivityIndicator()
}
        
DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
        // 애니메이션 정지.
        // 서버 통신 완료 후 다음의 메서드를 실행해서 통신의 끝나는 느낌을 줄 수 있다.
        self.avtivityIndicator.stopAnimating()
        self.loadingBgView.removeFromSuperview()
}
```
