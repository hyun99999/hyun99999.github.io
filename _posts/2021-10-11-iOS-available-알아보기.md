---
title:  "iOS) available 알아보기"
categories:
- iOS

date:   2021-10-11  21:36:00 +0900
author_profile: false
---
# 💃 available 알아보기

`available` 을 사용하여 특정 Swift 버전 또는 특정 플랫폼 및 OS버전과 관련된 선언의 생명주기를 나타낸다.

사용가능한 속성(attribute)은 항상 두개 이상의 쉼표로 구분된 attribute argument 목록과 함께 나타난다.

이러한 argument는 다음 플랫폼 또는 언어 이름 중 하나로 시작한다.

- `iOS`
- `iOSApplicationExtension`
- `macOS`
- `macOSApplicationExtension`
- `watchOS`
- `watchOSApplicationExtension`
- `tvOS`
- `tvOSApplicationExtension`
- `swift`

## #available ?

#available 은 다음과 같이 사용되고 `*` 필수이다. Bool 을 반환하는 런타임 검사이다. 그래서 런타임 중에 모드를 변경해도 반영이 된다.

```swift
// ✅ if 에서 사용
if #available(iOS 14, *) {
    print("This code only runs on iOS 14 and up")
} else {
    print("This code only runs on iOS 13 and lower")
}

// ✅ guard 에서 사용
guard #available(iOS 14, *) else {
    print("Returning if iOS 13 or lower")
    return
}

print("This code only runs on iOS 14 and up")
```

## @available ?

메서드를 Swift, OS 버전 또는 플랫폼의 버전에 따라서 제한할 수 있다.

`@available` 은 함수(메소드), 클래스 또는 프로토콜 앞에 놓인다. 타입 또는 프로토콜이 적용되는 플랫폼 및 OS를 나타낸다.

`#available` 과 다르게, 컴파일 타임에 경고 또는 오류를 생성합니다.

```swift
@available(iOS 12, *)
func setupDoneButton() { }

// 다음과 같이 사용 가능
@available(iOS 12.0, macOS 10.12, *)
func setupDoneButton() { }
```

deployment target 즉, 최소한으로 지원하는 OS 버전이 10.0 인 경우에 위의 `setupDoneButton()` 메서드는 다음과 같이 사용된다.

```swift
if #available(iOS 12, *) {
     self.setupDoneButton()
} else {
    // Fallback on earlier versions
}
```

### 출처

[iOS ) available](https://zeddios.tistory.com/647)
