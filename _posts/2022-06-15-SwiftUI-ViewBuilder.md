---
title:  "SwiftUI) ViewBuilder"
categories:
- iOS

date:   2022-06-17  17:12:00 +0900
author_profile: false
---
### ViewBuilder

Closure에서 View를 구성하는 custom parameter attribute

그럼이제 앞에서 말한 예를들어 HStack View Builder 는 무엇이냐!

다음은 HStack 의 이니셜라이저입니다.

```swift
@inlinable public init(alignment: VerticalAlignment = .center,
                    spacing: CGFloat? = nil,
                    @ViewBuilder content: () -> Content)
```

어... body 라는 단어는 뭔가 있나요? 왜 `var body` 만 가능해요?
<img width="600" alt="11" src="https://user-images.githubusercontent.com/69136340/174251600-57e3fd3d-dc9f-4bd1-9e83-4df0b911530e.png">

***body 를 사용하지 않으니 protocol 을 채택하지 못한다고 에러가 나오네요!***

**body 는 암시적으로 @ViewBuilder 로 선언되어있기 때문에 클로저에서 뷰를 구성할 수 있어요!**

```swift
import SwiftUI

struct ViewA: View {

    var body: some View {
        Text("Hello, World!")
    }
}

struct ViewA_Previews: PreviewProvider {
    static var previews: some View {
        ViewA()
    }
}
```

출처:

[SwiftUI ) ViewBuilder](https://zeddios.tistory.com/1324)

### body

앞서 언급한 것처럼 body 에는 @ViewBuilder 가 선언되어 있습니다.

<img width="500" alt="22" src="https://user-images.githubusercontent.com/69136340/174251604-6bc438da-cb90-43ae-927a-e24da229dc5e.png">

_개발자 문서를 통해서 좀 더 알아봐요!_

Structure

## ViewBuilder

**Overview**

일반적으로 ViewBuilder 를 자식 뷰 생성 클로저 매개변수의 매개변수 속성으로 사용하여 해당 클로저가 여러 자식뷰를 제공할 수 있도록 합니다. 예를들어, 다음 contextMenu 함수는 view builder 를 통해 하나 이상의 뷰를 생성하는 클로저를 허용합니다.

```swift
func contextMenu<MenuItems: View>(
    @ViewBuilder menuItems: () -> MenuItems
) -> some View
```

이 함수는 다음과 같이 여러 자식 뷰를 제공할 수 있습니다.

```swift
myView.contextMenu {
    Text("Cut")
    Text("Copy")
    Text("Paste")
    if isSymbol {
        Text("Jump to Definition")
    }
}
```

[Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/viewbuilder)

이렇게 자신만의 뷰 컨테이너를 만들 수 있다.

```swift
struct ContentView: View {
    var body: some View {
        manyTexts
        MyVStack {
            HStakc {
                // ...
            }
        }
    }

    @ViewBuilder
    var manyTexts: some View { 
        Text("Zedd")
        Text("Zedd")
        Text("Zedd")
        Text("Zedd")
    }
}
// 메서드도 동일.
```

```swift
struct MyVStack<Content: View>: View {
    let content: () -> Content    // 클로저 표현식을 담는 변수
    init(@ViewBuilder content: @escaping () -> Content) {
        self.content = content
    } // 클로저 표현식({})를 인자로 받음으로서 다수의 하위뷰들을 받아 content변수에 넣음.
    
    var body: some View {
        VStack(spacing: 10) {
            content()
        }
        .font(.largeTitle)
    }
}
```

view builder 는 buildBlock 이라는 타입 메서드를 통해서 자식 뷰로 작성된 단일 뷰를 전달하게 됩니다.

2개 이상의 뷰일때는요? 바로 `TupleView` 라는 타입을 리턴합니다.

그런데 아래와 같이 C9 까지 밖에 없죠? 

<img width="400" alt="33" src="https://user-images.githubusercontent.com/69136340/174251836-41a33f92-5592-430f-98cf-8c0fea984104.png">

즉, 10개를 넘는 뷰를 넣을 경우 컴파일 에러가 발생하게 됩니다.

<img width="400" alt="44" src="https://user-images.githubusercontent.com/69136340/174251842-6e2928dd-18a3-44d7-9916-7518880a16bc.png">
