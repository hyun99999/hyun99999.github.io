---
title:  "iOS) Dispatch(1) - Dynamic / Static Dispatch"
categories:
- iOS

date:   2022-08-10  14:50:00 +0900
author_profile: false
---
# Method Dispatch

method dispatch 는 Swift 에서 메서드를 호출할 때 현재 메모리에서 어떻게 어떤 메소드를 실행시킬지를 결정할때 사용하는 방법입니다.

클래스의 dispatch 과정을 예시를 들어봅시다.

```swift
class Animal {
    func bark() {
        print("bark!")
    }
}

class Cow: Animal {
    func bark() {
        print("moo!")
    }
}

class Dog: Animal { }

let animal: Animal = Animal()
animal.bark()

let cow: Animal = Cow()
cow.bark()

let dog: Animal = Dog()
dog.bark()
```

***인스턴스 animal, cow, dog 가 bark() 메소드를 호출할 때 어떤 결과가 나올까요? 즉, 어떤 클래스의 메소드를 호출할까요?*** 

**이때 *컴파일러는 클래스의 메소드가 자식 클래스에서 오버라이딩이 될 경우를 대비해서 런타임에 참조를 확인합니다.*** 

즉, 클래스는 오버라이딩이 될 수 있는 가능성이 존재하기 때문에 Dog 클래스처럼 오버라이딩이 되지 않았더라도 런타임 때 어떤 메소드를 실행시킬지 결정합니다. 이것이 바로 method dispatch 중 `**dynamic dispatch**` 입니다.

method dispatch 에는 `dynamic dispatch` 외에도 `static dispatch` 가 있습니다. 알아봅시다!

# Static / Dynamic Dispatch

- **Static Dispatch(Direct Call)**

**컴파일 타임**에 컴파일러가 어떤 클래스의 메소드를 실행할지 결정하여 빠르게 수행됩니다.

- **Dynamic Dispatch(Indirect Call)**

**런타임**에 어떤  클래스의 메소드를 실행할지 결정합니다.

## Reference / Value / Protocol Type 의 dispatch

Swift 에서 각 타입들의 어떤 dispatch 를 사용하는지 알아봅시다.

- Reference Type

앞서 보았던 예시의 결과는 어떻게 나올까?

```swift
class Animal {
    func bark() {
        print("bark!")
    }
}

class Cow: Animal {
    func bark() {
        print("moo!")
    }
}

class Dog: Animal { }

let animal: Animal = Animal()
animal.bark()
// ✅ bark!

let cow: Animal = Cow()
cow.bark()
// ✅ moo!

let dog: Animal = Dog()
dog.bark()
// ✅ bark!
```

cow 의 경우 Animal 타입이지만 Cow 타입의 인스턴스를 할당했습니다. 이때 업캐스팅하기 때문에 Animal 클래스의 bark() 메소드가 아닌 Cow 클래스의 bark() 메소드를 호출하기로 **런타임**에 결정합니다.

dog 의 경우 Animal 타입이지만 Dog 타입의 인스턴스를 할당했습니다. 이때 Dog 에는 bark() 메소드가 없지만 상속받은 부모 클래스 Animal 의 bark() 메소드가 호출됩니다.

이처럼 참조 타입의 Class 는 상속이 가능합니다. 따라서 상속과 오버라이드를 한 함수를 호출 할 **가능성**이 있기 때문에 `dynamic dispatch` 를 수행합니다.

### ***그렇다면 이 참조 타입의 dynamic dispatch 은 어떻게 이루어지는걸까요?***

클래스에는 메소드를 호출할 때 실질적으로 어떤 메소드를 호출하는지 결정하는 역할을 하는 **함수 포인터**가 담긴 **vTable** 이 있습니다. 

상속을 통해 자식 클래스에는 부모 클래스의 **vTable** 의 복사본이 존재합니다. 오버라이딩하지 않은 메소드가 있다면 부모 클래스의 함수 포인터가 그대로 담겨 있게 됩니다.

이를 통해 런타임 때 클래스의 **vTable** 의 해당 함수 포인터를 확인하고, 자식 클래스의 메소드인지 부모 클래스의 메소드인지 결정할 수 있게 됩니다.

자식 클래스가 새 메소드를 추가하면 해당 메소드 포인터는 vTable 의 끝에 추가됩니다.

***dynamic dispatch 는 vTable 에서 함수 포인터를 찾아 메모리 주소를 읽고 접근해야하기 때문에 성능상의 손해를 겪게 됩니다.***

- Value Type

값 타입은 상속을 할 수 없기 때문에 오버라이딩 될 가능성이 없고, `static dispatch` 를 수행합니다.

- Protocol Type

프로토콜을 통해 호출하는 메소드는 프로토콜을 채택한 타입들이 구현한 메소드입니다. 이때는 메소드를 구현하고 있음이 보장됩니다. 혹은 protocol 의 extension 에서는 메소드의 기본 구현을 작성할 수 있습니다.

**감이 오시나요? 이때 해당 타입에서 구현이 됐는지 안됐는지를 확인해야하기 때문에 `dynamic dispatch` 를 수행합니다.**

그런데 앞서 reference type 과는 달리 vTable 을 조회하지 않습니다. 대신 프로토콜이 보유한 정보를 가진 **witness table** 을 통해서 `dynamic dispatch` 를 수행할 수 있게 됩니다.

(***위에서 언급된 클래스의 메소드 뿐만 아니라 상속을 통해 오버라이딩이 가능한 프로퍼티 역시 동일하게 dynamic dispatch 를 통해서 결정됩니다.)***

### 출처

[Swift) Static Dispatch & Dynamic Dispatch (1/2)](https://babbab2.tistory.com/143)

[[iOS - swift] Static Dispatch, Dynamic Dispatch 성능 최적화 방법, Witness Table, (final, private을 사용하는 이유)](https://ios-development.tistory.com/816)

[[SWIFT] Swift Method Dispatch - Dynamic, Static](https://dongminyoon.tistory.com/65)
