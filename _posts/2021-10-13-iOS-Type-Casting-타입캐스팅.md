---
title:  "iOS) Type Casting(타입캐스팅)"
categories:
- iOS

date:   2021-10-13  14:42:00 +0900
author_profile: false
---
**내용**

- 다운캐스팅
- 업캐스팅
- 타입 체킹

**타입캐스팅**은 instance 의 유형을 확인하거나 해당 instance를 자체 클래스 계층 구조에서 부모 클래스 또는 자식 클래스로 처리하는 방법이다.

연산자에는 is 와 as 가 있다.

# 다운캐스팅

**다운캐스팅**은 부모 클래스의 타입을 자식 클래스의 타입으로 낮춰서 형변환하는 것을 다운캐스팅이라고 말한다.(부모 클래스 인스턴스를 자식 클래스의 타입으로 참조한다.)  다운캐스팅에는 2가지 방법이 있다.

- as? : 변환이 성공 시 옵셔널 값을 가지며 실패 시 nil 반환
- as! : 강제 타입 변환 시도. 성공 시 언래핑된 값을 가지며 실패 시 런타임 에러 발생

## 🧊 as? as!

예시를 살펴봐보자 다음과 같이 화면 전환할 때 as? 를 사용한다. 

<img src ="https://user-images.githubusercontent.com/69136340/137073563-95fe7948-efaa-49aa-bf0f-c3a295af5d55.png" width ="500">

우선 instantiateViewController() 메서드의 리턴값이 무엇인지 확인해보자.

<img width="500" alt="2" src="https://user-images.githubusercontent.com/69136340/137073580-7c0eacf7-7a75-4a0a-a59a-8bb2dec7cbc7.png">

UIViewController 를 반환한다. 즉, 위의 코드는 UIViewController 를 상속받는 ResultViewController 로 다운캐스팅하겠다는 의미이다. 그렇다면 왜 다운캐스팅할까?

**쓰지 않으면 에러가 발생할까? 아니다. 가능하다!**

<img src ="https://user-images.githubusercontent.com/69136340/137073699-3bea374f-ee72-4877-ab41-774b61f9f9c7.png" widht ="500">

위의 코드처럼 다운캐스팅 없이도 사용이 가능하다! 하지만 뭔가 달라졌다. ResultViewController 의 message 프로퍼티에 접근할 수 없어서 주석처리를 해주었다.

**왜??** UIViewController 를 상속받는 ResultViewController 는 message 프로퍼티는 있지만 UIViewController 에는 message 프로퍼티가 없기 때문이다.

그래서 **다운캐스팅은** 이런식으로 부모 클래스가 아닌 자식 클래스에만 선언된 특정 메서드, 프로퍼티에 접근하기 위해서 사용된다.

# 업캐스팅

**업캐스팅**은 자식 클래스에서 부모 클래스로 형변환하는 것이다.(자식 클래스 인스턴스를 부모 클래스의 타입으로 참조한다.) 업캐스팅은 as 를 통해서 진행된다.

업캐스팅 성공 시 업캐스팅 한 클래스의 메서드와 프로퍼티에만 접근할 수 있다.

- as : 타입변환이 확실하게 가능한 경우만 사용.(자기자신 혹은 업캐스팅) 그 외에는 컴파일 에러.

## 🧊 as

**컴파일 시점**에서 업캐스팅 가능여부를 결정하기 때문에 무조건 성공한다. 

그래서 as 를 사용해도 되고 직접 타입을 명시해도 된다.

```swift
// 1. as를 사용한 업캐스팅
let human1 = Student() as Human
 
// 2. Type Annotation을 사용한 업캐스팅
let human2: Human = Student()
```

- 추가적으로 as 는 패턴 매칭에서도 사용된다.

# 타입체킹

## 🧊 is

런타임에서 타입을 체킹한다. 표현식이 타입과 동일하거나 자식 클래스일 경우 `true`. 아닐경우 `false` 를 반환한다. 

```swift
let char: Character = "A"
 
char is Character       // true
char is String          // false
```

위와 같이 내가 원하는 타입인지 확인할 때도 사용하고 자식 클래스인지 확인할 때도 사용한다.

# Any, AnyObject에 대한 Type casting

`Any` 와 `AnyObject` 타입을 사용할 경우 상속 관계가 아니어도 예외적으로 타입캐스팅 가능. 런타임 시점에 타입이 결정된다.

- Any: 모든 type을 포함하여 모든 유형의 instance 나타낼 수 있음.
- AnyObject: 모든 class type의 instance 나타낼 수 있음.(즉, 클래스 타입만 저장할 수 있다.)

앞서 as 는 패턴매칭에서도 사용된다고 말했는데 Any 와 AnyObject 에 대해서 다음과 같이 사용가능하다.

```swift
for thing in things {
    switch thing {
    case _ as Int:
        print("Int Type!")
    case _ as Double:
        print("Double Type!")
    case _ as String:
        print("String Type!")
    case _ as Human:
        print("Human Type")
    case _ as () -> ():
        print("Closure Type")
    default:
        print("something else")
    }
}
```

### 출처

[[iOS - swift 공식 문서] 18. Type Casting (타입 캐스팅, 다운캐스팅, 업캐스팅)](https://ios-development.tistory.com/621)

[Swift) is, as - 타입 캐스팅 (Type Casting)](https://babbab2.tistory.com/127)

[Swift) Any와 AnyObject 알아보기](https://babbab2.tistory.com/128?category=828998)
