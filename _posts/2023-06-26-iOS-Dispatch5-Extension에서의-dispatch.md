 ---
title:  "iOS) Dispatch(5) - Extension 에서의 dispatch"
categories:
- iOS

date:   2023-06-26  17:30:00 +0900
author_profile: false
---
Reference / Value / Protocol Type 의 Extension 할 경우 dispatch 방법이 달라지기도 합니다. 알아봅시다!

- **value type** 의 extension 에서 구현한 메소드는 상속 가능성이 없기 때문에 **static dispatch** 로 동작.
- **reference type** 의 extension 에서 구현한 메소드는 서브 클래스에서 재정의가 불가능하다.(이를 위해서는 **@objc** 를 통해 해결할 수 있다.) 그래서 extension 의 메소드가 호출되는 것이 보장되기 때문에 **static dispatch** 로 동작.
- protocol 의 extension 에서 구현한 메소드는 두 가지로 나뉜다.
    
    1.extension 에 default 구현을 해둔 경우는 해당 메소드에 대한 구현이 필수적이지 않기 때문에 **dynamic dispatch** 로 동작.
    
    ```swift
    protocol A {
        func sayA()
    }
    
    extension A {
        // default 구현
        func sayA() {
            print("A")
        }
    }
    ```
    
    2.protocol 에 없는 메소드를 extnesion 으로 구현했고, **상수, 변수의 타입이 해당 protocol 이라면 동일한 이름의 메서드가 구현되어 있어도 해당 protocol 에 대한 메소드가 호출되어서 static dispatch 로 동작.**
    
    ```swift
    protocol Human { }
     
    extension Human {
        func sayHello() {
            print("Hello Human!")
        }
    }
     
    class Student: Human { }
    
    class Teacher: Human {
        func sayHello() {
            print("Hello Teacher!")
        }
    }
    
    // Human 타입의 변수 a 는 static dispatch 로 호출.
    var a: Human = Student.init()
    a.sayHello()       // ✅ Hello Human!
     
    a = Teacher.init()
    a.sayHello()       // ✅ Hello Human!
    
    // Teacher 클래스 내에서 선언한 메서드 호출.
    let b: Teacher = Teacher.init()
    b.sayHello()       // ✅ Hello Teacher!
    // (class 에서 선언함. 별도의 final 키워드가 없기 때문에 상속을 통한 오버라이딩 여지가 있다고 판단하여 dynamic dispatch 로 호출.)
    
    // 출처 : https://babbab2.tistory.com/144?category=828998
    ```
    

### 출처

[Swift) Static Dispatch & Dynamic Dispatch (2/2)](https://babbab2.tistory.com/144?category=828998)
