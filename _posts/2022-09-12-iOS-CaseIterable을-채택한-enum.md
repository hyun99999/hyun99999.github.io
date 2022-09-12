---
title:  "iOS) CaseIterable 을 채택한 enum"
categories:
- iOS

date:   2022-09-12  22:46:00 +0900
author_profile: false
---
열거형에 포함된 모든 케이스에 대해서 순회하고 싶을 때가 있습니다.

이때는 `CaseIterable` 프로토콜을 채택하면 `allCase` 라는 타입 프로퍼티를 통해 모든 케이스에 대한 컬렉션을 생성할 수 있습니다.

## ✅ CaseIterable 을 채택하여 enum 순회

```swift
import Foundation

enum School: CaseIterable {
    case elementary
    case middle
    case high
    case college
}

let allCase: [School] = School.allCases
print(allCase)

// [School.elementary, School.middle, School.high, School.college]
```

이때 원시값을 갖는 열거형을 만들기 위해서 자료형을 상속받았는데 위치가 중요합니다.

<img width="600" alt="1" src="https://user-images.githubusercontent.com/69136340/189670077-c6246741-011c-4675-a460-f0c4622a0498.png">

**열거형이 상속받을 원시값을 위해서 항상 첫 번째에 위치해야 합니다.**

단순히 `CaseIterable` 을 채택해주는 것만으로는 `allCases` 를 사용할 수 없는 복잡한 경우가 있습니다.

## 1️⃣ 플랫폼별로 조건을 

<img width="600" alt="2" src="https://user-images.githubusercontent.com/69136340/189670103-e008ff44-13a7-44ad-ba8e-876be84f247b.png">

추가하는 경우 available 로 특정 케이스를 플랫폼에 따라 사용할 수 없는 경우가 생기면  `CaseIterable` 프로토콜을 채택하는 것만으로는 `allCases` 프로퍼티를 사용할 수 없습니다.

이때는 직접 `allCases` 프로퍼티를 구현해 주어야 합니다.

```swift
import Foundation

enum School: String, CaseIterable {
    case elementary = "초등학교"
    case middle = "중학교"
    case high = "고등학교"
    case college = "대학교"
    // deprecated : 더이상 사용되지 않는(경고) 첫번째 버전.
    // obsoleted : 폐기되어 사용할 수 없는(에러) 첫번째 버전.
    @available(iOS, deprecated: 13.0, obsoleted: 14.0)
    // -> iOS 13 부터는 사용되지 않는 deprecated 경고.
    // -> iOS 14 부터는 사용할 수 없는 에러.
    case graduate
    
    static var allCases: [School] {
        let all: [School] = [.elementary,
                             .middle,
                             .high,
                             .college]
        
        #if os(iOS)
        return all
        #else
        return all + [.graduate]
        #endif
    }
}

let allCase: [School] = School.allCases
print(allCase)

// ✅ macOS Command Line Tool 로 만들었기 때문에 graduate 케이스까지 출력.
// [School.elementary, School.middle, School.high, School.college, School.graduate]
```

## 2️⃣ 케이스가 연관 값을 갖는 경우

<img width="700" alt="3" src="https://user-images.githubusercontent.com/69136340/189670214-62f6c78d-b295-44bf-9392-f4e994806efb.png">

enum 형의 케이스가 연관 값을 갖는 경우 즉, 파라미터를 전달하는 경우에는 `CaseIterable` 프로토콜을 채택하여도 `allCases` 프로퍼티를 사용할 수 없습니다.

이때도 마찬가지로 직접 `allCases` 프로퍼티를 구현해 주어야 합니다.

```swift
import Foundation

enum PastaTaste: CaseIterable {
    case cream, tomato
}

enum PizzaDough: CaseIterable {
    case cheeseCrust, thin, original
}

enum PizzaTopping: CaseIterable {
    case pepperoni, cheese, bacon
}

enum MainDish: CaseIterable {
    case pasta(taste: PastaTaste)
    case pizza(dough: PizzaDough, topping: PizzaTopping)
    // 파라미터로 enum 이 아닌 경우
    case chiken(withSauce: Bool)
    // 파라미터가 없는 경우
    case rice
    
    static var allCases: [MainDish] {
        return PastaTaste.allCases.map(MainDish.pasta)
        + PizzaDough.allCases.reduce([]) { (result, dough) -> [MainDish] in
            result + PizzaTopping.allCases.map { topping -> MainDish in
                MainDish.pizza(dough: dough, topping: topping)
            }
        }
        + [true, false].map(MainDish.chiken)
        + [MainDish.rice]
    }
}

print(MainDish.allCases)

// [MainDish.pasta(taste: PastaTaste.cream), MainDish.pasta(taste: PastaTaste.tomato), MainDish.pizza(dough: PizzaDough.cheeseCrust, topping: PizzaTopping.pepperoni), MainDish.pizza(dough: PizzaDough.cheeseCrust, topping: PizzaTopping.cheese), MainDish.pizza(dough: PizzaDough.cheeseCrust, topping: PizzaTopping.bacon), MainDish.pizza(dough: PizzaDough.thin, topping: PizzaTopping.pepperoni), MainDish.pizza(dough: PizzaDough.thin, topping: PizzaTopping.cheese), MainDish.pizza(dough: PizzaDough.thin, topping: PizzaTopping.bacon), MainDish.pizza(dough: PizzaDough.original, topping: PizzaTopping.pepperoni), MainDish.pizza(dough: PizzaDough.original, topping: PizzaTopping.cheese), MainDish.pizza(dough: PizzaDough.original, topping: PizzaTopping.bacon), MainDish.chiken(withSauce: true), MainDish.chiken(withSauce: false), MainDish.rice]
```

### 출처:

SWIFT 스위프트 프로그래밍 3판 - 야곰

[[swift enum, CaseIterable로 열거형타입 배열처럼 다루기](https://0urtrees.tistory.com/197)](https://0urtrees.tistory.com/197)

[[Apple Developer Documentation - CaseIterable](https://developer.apple.com/documentation/swift/caseiterable)](https://developer.apple.com/documentation/swift/caseiterable)
