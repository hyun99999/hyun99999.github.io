---
title:  "iOS) Moya custom Plugin 을 통해서 네트워크 연결 실패를 대응해보자"
categories:
- iOS

date:   2022-04-15  15:45:00 +0900
author_profile: false
---
### 내용

- 서버 통신 시에 네트워크가 유실된 경우 사용자들은 무작정 기다릴 수 밖에 없다. 이때 alert 창과 함께 네트워크 연결 실패에 대해 알려주자!
- Moya 의 Plugin 을 커스텀해서 대응해보자.

### Moya Plugin 에 대해서 알아보자

[[Moya/Plugins.md at master · Moya/Moya](https://github.com/Moya/Moya/blob/master/docs/Plugins.md)](https://github.com/Moya/Moya/blob/master/docs/Plugins.md)

Moya plugin 은 다음의 메서드를 호출해서 request 와 response 를 수정하거나 side-effect 에 대해서 수행할 수 있습니다.

- (`prepare`) after Moya has resolved the `TargetType` to a `URLRequest`. This is an opportunity to modify the request before it is sent (e.g. add headers).
- (`willSend`) before a request is about to be sent. This is an opportunity to inspect the request and perform any side-effects (e.g. logging).
- (`didReceive`) after a response has been received. This is an opportunity to inspect the response and perform side-effects.
- (`process`) before `completion` is called with the `Result`. This is an opportunity to make any modifications to the `Result` of the `request`.

### 기존 코드

- Custom Moya Plugin

```swift
import Foundation

import Moya

// 🔥 custom Moya Plugin
final class MoyaLoggerPlugin: PluginType {
    
  func willSend(_ request: RequestType, target: TargetType) {
    // ...
  }
    
  func didReceive(_ result: Result<Response, MoyaError>, target: TargetType) {
    switch result {
    case let .success(response):
      onSuceed(response)
    // 🔥 응답을 받는데 실패했다면 네트워크 연결 실패로 alert 창을 띄어보자.
    case let .failure(error):
      onFail(error)
    }
  }
 
  func onSuceed(_ response: Response) {
    // ...
  }
  
  func onFail(_ error: MoyaError) {
      var log = "------------------- 네트워크 오류"
      log.append("(에러코드: \(error.errorCode)) -------------------\n")
      log.append("3️⃣ \(error.failureReason ?? error.errorDescription ?? "unknown error")\n")
      log.append("------------------- END HTTP -------------------")
      print(log)
  }
}
```

- API

```swift
import Foundation

import Moya

public class NoticeAPI {
    // 🔥 싱글톤 패턴을 사용.
    static let shared = NoticeAPI()
    
    // ✅ Moya plugin 적용.
    var noticeProvider = MoyaProvider<NoticeService>(plugins: [MoyaLoggerPlugin()])
    
    // ✅ 객체화할 수 없게 만들어서 싱글톤 패턴으로만 사용하도록 접근 제어자 설정.
    private init() { }
    
    func newNoticeFetch(completion: @escaping(NetworkResult<Any>) -> Void) {
        noticeProvider.request(.newNoticeFetch) { result in
            switch result {
            case .success(let response):
                // ...
            case .failure(let err):
                // ...
            }
        }
    }
}
```

- view controller

```swift
private func newNoticeFetchWithAPI(completion: @escaping () -> Void) {
    // 🔥 싱글톤 패턴 적용.
    NoticeAPI.shared.newNoticeFetch { response in
        // ...
    }
}
```

### 해결

- custom Moya Plugin

```swift
import Foundation
import UIKit

import Moya

// 🔥 custom Moya Plugin
final class MoyaLoggerPlugin: PluginType {
    
    // 🔥 alert 창을 띄워줄 view controller 객체를 이니셜라이저를 통해서 초기화.
    private let viewController: UIViewController

    init(viewController: UIViewController) {
        self.viewController = viewController
    }
    
    func willSend(_ request: RequestType, target: TargetType) {
        // ...
    }
    
    func didReceive(_ result: Result<Response, MoyaError>, target: TargetType) {
        switch result {
        case let .success(response):
            onSucceed(response)
        // 🔥 응답을 받는데 실패했다면 네트워크 연결 실패로 alert 창을 띄어보자.
        case let .failure(error):
            onFail(error)
        }
    }
    
    func onSucceed(_ response: Response) {
        // ...
    }
   
    func onFail(_ error: MoyaError) {
        var log = "------------------- 네트워크 오류"
        log.append("(에러코드: \(error.errorCode)) -------------------\n")
        log.append("3️⃣ \(error.failureReason ?? error.errorDescription ?? "unknown error")\n")
        log.append("------------------- END HTTP -------------------")
        print(log)

        // 🔥 present alert view controller.
        let alertViewController = UIAlertController(title: "네트워크 연결 실패", message: "네트워크 환경을 한번 더 확인해주세요.", preferredStyle: .alert)
        alertViewController.addAction(UIAlertAction(title: "확인", style: .default, handler: nil))

        viewController.present(alertViewController, animated: true)
    }
}
```

- API

```swift
public class NoticeAPI {
    // 🔥 이니셜라이저로 전달 받은 view controller 객체를 가지고 MoyaLoggerPlugin 을 초기화. 
    var noticeProvider: MoyaProvider<NoticeService>
    
    // 🔥 싱글톤 패턴에서 변경.
    public init(viewController: UIViewController) {
        noticeProvider = MoyaProvider<NoticeService>(plugins: [MoyaLoggerPlugin(viewController: viewController)])
    }

    func newNoticeFetch(completion: @escaping(NetworkResult<Any>) -> Void) {
        noticeProvider.request(.newNoticeFetch) { result in
            switch result {
            case .success(let response):
                // ...
            case .failure(let err):
                // ...
            }
        }
    }
}
```

- view controller

```swift
private func newNoticeFetchWithAPI(completion: @escaping () -> Void) {
    // 🔥 alert 창을 띄울 view controller 객체를 파라미터로 전달.
    NoticeAPI(viewController: self).newNoticeFetch { response in
        // ...
    }
}
```

### 구현
<img src="https://user-images.githubusercontent.com/69136340/163532193-76fa103a-c4a1-4af3-8a7a-1fd14aa687f7.png" width = "250">

---

**출처:**

[[Moya/CustomPlugin.md at master · Moya/Moya](https://github.com/Moya/Moya/blob/master/docs/Examples/CustomPlugin.md)](https://github.com/Moya/Moya/blob/master/docs/Examples/CustomPlugin.md)
