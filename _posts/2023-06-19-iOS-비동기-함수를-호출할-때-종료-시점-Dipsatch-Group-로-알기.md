---
title:  "iOS) 비동기 함수를 호출할 때 종료 시점 Dipsatch Group 로 알기"
categories:
- iOS

date:   2023-06-19  15:30:00 +0900
author_profile: false
---
### 내용

- 위젯에서 Intent 를 사용하기위한 IntentHandler 에서 default[[parameter type](for:)](for:) 메서드로 위젯을 추가할 때 기본값을 설정하는 메서드를 다루겠습니다.
- **🚨 비동기 서버 통신  작업 이후에 결과를 가지고 함수를 리턴하는 메서드를 구현해야 합니다.**
- 그래서 **비동기적인 서버통신 작업의 종료 시점 이후에 처리해야 했습니다.**

```swift
    // 위젯 추가할때 호출. 기본값 설정.
    // ✅ 서버통신 후에 가장 첫번째 값을 기본값으로 리턴.
    // 서버통신 후에 값이 없다면 기본값으로 nil 리턴.
    func defaultMyCard(for intent: MyCardIntent) -> MyCard? {
        var myCard: MyCard?

        // ✅ DispatchGroup 사용.
        let group = DispatchGroup()
        
        DispatchQueue.global().async(group: group) { [weak self] in
            // ✅ 해당 블록이 disaptch group 에 진입함을 나타냄
            group.enter()
            
            self?.cardListFetchWithAPI { [weak self] result in
                switch result {
                case .success(let result):
                    if let result {
                        // ✅ 위젯의 configuration 을 구성
                        myCard = MyCard(identifier: self?.cardItems?[0].cardUUID ?? "", display: self?.cardItems?[0].cardName ?? "")
                        myCard?.userName = self?.cardItems?[0].userName
                        myCard?.cardImage = self?.cardItems?[0].cardImage
                    }
                case .failure(let err):
                    print(err)
                }
                // ✅ dispatch group 내의 블럭이 실행 종료됨을 나타냄
                group.leave()
            }
        }
        
        _ = group.wait(timeout: .now() + 60)
        
        return myCard
    }
```

**제가 원한 결과는 비동기 서버통신 작업이 시작하고 함수를 리턴하는 것이 아닌 서버통신 작업이 끝나기를 기다린 후에 결과를 가지고 `MyCard` 자료형으로 리턴하는 것입니다.**

(completion handler 를 통해서 비동기 서버통신 작업을 동기적으로 실행할 수 있지만, 위의 경우는 `defaultMyCard(for:)` 메서드에서 리턴값을 원했기 때문에 서버통신 결과를 기다렸다가 동기적으로 리턴해주어야 했습니다.)

이때 원하는 작업의 종료 시점을 알기 위해서 **DispatchGroup** 의 **enter, leave** 를 사용하였습니다.

**그래서 cardListFetchWithAPI 의 completion handler 에서 configuration 구성 후에 `group.leave()` 메서드로 작업의 종료를 알리도록 하였습니다.**

**추가적으로,** `wait` 를 사용하여 60초간 group 의 작업을 기다리고, 서버통신의 결과를 기다리다가 오래걸린다면 nil 을 반환할 수 있도록 구현해보았습니다. 

### 결과

<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/6ca30c5e-a586-4d27-a333-ab699f2ba1ad" width ="250">


