---
title:  "iOS) Protocol 생성 시 AnyObject 상속받기"
categories:
- iOS

date:   2021-11-11  17:29:00 +0900
author_profile: false
---
Delegate 패턴을 사용하면서 Protocol 을 만들었다. 그런데 SwiftLint 에서 다음과 같이 경고가 발생했다!

<img src ="https://user-images.githubusercontent.com/69136340/141263795-2358a5fd-f568-4dcd-a095-2a97d4dd2330.png" width ="700">

> delegate 는 reference cycle 을 피하기 위해서  weak(약한 참조)를 사용해야 한다.
> 

<img src ="https://user-images.githubusercontent.com/69136340/141263812-45998cd1-7a10-4106-9045-2d60c9c4a629.png" width =" 700">

> delegate protocol 은 weak(약한 참조)를 위해서 클래스 전용 이어야 한다.
> 

### Delegate 를 weak 로 선언하자

인스턴스화 해서 참조할 때 Retain 이 생긴다고 해요. 그런데 protocol 은 클래스나 구조체처럼 인스턴스화 되는 것이 아닌데 왜 Retain cycle 이 생기는걸까요?

예를 들어 클래스 a와 b가 서로를 protocol 로 참조하게 되면 강한 참조가 되면서 Retain Cycle 이 발생하는거죠!

그래서 weak 를 선언해서 reference count 를 늘리지 않도록 해주는 것이 메무리 누수를 막을 수 있는 방법이랍니다.

### 그래서 프로토콜을 weak 로 약한 참조로 선언을 해주었더니 이제는 에러가 발생했다...?

<img src ="https://user-images.githubusercontent.com/69136340/141263819-07768210-42ae-44d4-a5e7-50d3bccfa07d.png" width ="700">

weak 를 사용하려는 타입이 만약 protocol 일 경우 class 를 상속받지 않으면 위의 오류를 발생시켰습니다. 

이게무슨.. ㅎ.. 자자.. 진정하고

우리가 프로토콜을 채택하는 해당 타입이 클래스인지 구조체인지 열거형인지 알 수 없습니다. 그래서 reference count 관리를 위한 weak 를 사용할 수 없기 때문에 나타나는 오류입니다.

그래서 프로토콜은 class 를 상속받아서 class 에만 사용할 프로토콜임을 알려줘야 합니다.

<img src ="https://user-images.githubusercontent.com/69136340/141263827-9c225eae-a2ad-4766-9293-defaa17e16de.png" width ="700">

class 가 곧 사용되지 않는다고 AnyObjcet 를 대신 사용하라고 해서 아래와 같이 코드를 수정해주었습니당.

- AnyObject : 모든 class type의 instance 나타낼 수 있음.(즉, 클래스 타입만 저장할 수 있다.)

<img src ="https://user-images.githubusercontent.com/69136340/141263832-5071d968-9fb7-4887-aa0d-e480635a5c75.png" width ="700">

출처 :

[Protocol에 class선언해주기](https://hyerios.tistory.com/16)

[[swift] 메모리 관리 - Retain Cycle : strong, weak, unowned](https://dev200ok.blogspot.com/2020/04/swift-retain-cycle-strong-weak-unowned.html)

[[iOS 면접질문] Delegate는 retain이 될까?](https://fomaios.tistory.com/entry/iOS-면접질문-Delegate는-retain이-될까)
