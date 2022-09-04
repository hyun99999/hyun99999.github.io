---
title:  "CS) OOP(Object-Oriented Programming) 4가지 특징"
categories:
- iOS

date:   2022-09-02  15:36:00 +0900
author_profile: false
---
각 언어들이 지원하는 클래스를 통해서 OOP 를 구현할 수 있는 것이지 OOP 의 객체가 곧 클래스는 아니다.

OOP 의 4가지 특징

1️⃣ 캡슐화, Encapsulation

즉 캡슐 안에 담는다고 하여 '캡슐화' 라고 부르며, 여기서 말하는 단위나 캡슐이 곧 프로그래밍 언어에서 사용되는 객체
이다.

변수와 함수를 하나로 묶는 것.
데이터를 외부에서 접근하는 것을 방지하고, 메서드를 통해서만 접근할 수 있게 한다.
2가지 관점
데이터 캡슐화(상태-행위 캡슐화) : 객체는 상태와 행동을 하나의 단위로 묶는 자율적 실체
은닉화 : 외부에서 객체의 상태를 변경할 수 없도록 은닉. ex. 접근 제어자
객체가 다른 객체로 부터 독립적이고, 이것은 복잡도가 낮아지는 것으로 이어진다. 복잡성이 줄면 코드의 재사용성을 높일 수 있다.
class Grade {
    constructor (score) {
        this.score = score;
        this.curve = 10 
    }

    int getTotal() {
        return this.score * this.curve
    }
}

const myGrade = new Grade(9);
myGrade.getTotal() // 90
2️⃣ 상속, Inheritance

상속은 이미 만들어놓은 객체의 특징을 새로운 객체가 이어받아 사용할 수 있도록 하는 것이다.
자식 클래스가 부모 클래스의 특성과 기능을 물려받는 것
기능의 일부분을 변경하는 경우 자식 클래스에서 상속받아 수정 및 사용함
상속은 캡슐화를 유지, 클래스의 재사용이 용이하도록 해 준다.
class letterGrade extends Grade {
    char getLetter () {
        return (this.score >= 60 ? 'P' : 'F')
    }
} 

const hisGrade = new letterGrade(5);

console.log(hisGrade); // {score: 5, curve:10}
hisGrade.getLetter(); // 'F'
3️⃣ 추상화, Abstraction

객체 안의 공통적인 특징을 추출하여 하나의 개념 또는 기능으로 정의함으로써 인터페이스 와 구현을 분리하는 특징이다.
객체를 묘사하고, 유사성만을 표현하며 상세 사항은 다르게 구현되도록 할 수 있다.
추상화의 장점은 객체 내부의 변화가 외부에 미치는 영향력을 줄일 수 있다는 것이다.
외부에 구현된 인터페이스는 그대로 유지된다면, 내부의 로직이나 값이 어떻게 바뀌든 이를 활용하여 쓰는 프로그래머는 기존 명칭을 그대로 활용하여 값을 산출할 수 있다.
4️⃣ 다형성, Polymorphism

여러 객체가 동일한 요청(메시지)에 대해서 서로 다른 방식으로 응답할 수 있도록 만드는 것.
오버로딩(Overloading) : 같은 메서드 이름이지만 다른 인자 목록을 사용한다. 다수의 메서드를 만들수 있다.
오버라이딩(Overriding) : 부모 클래스의 메소드를 자식 클래스의 용도에 맞게 재정의하여 코드의 재사용성을 높임
동적 결합(Dynamic binding) : 더 이상 변경할 수 없는 구속(bind) 상태가 되는 것.
프로그램 내에서 변수, 배열, 라벨, 절차 등의 명칭, 즉 식별자(identifier)가 그 대상인 메모리 주소, 데이터형 또는 실제값으로 배정되는 것이 이에 해당되며,

컴파일 시에 확정되는 바인딩을 정적 바인딩(static binding)이라 하고,

런타임 시에 바인딩되는 것을 동적 바인딩(dynamic binding)이라고 한다.

자바에서는 동적바인딩을 하고 이로인해 상속과 다형성이 가능하다

Swift 에서는 이를 dynamic dispatch 라고 부른다.

public class Car {
    public void getName() {
        System.out.println("car");
    }
}

public class Sonata entends Car {
    public void getName() {
        System.out.println("sonata");
    }
}

public class Test {
    public static void main(String[] arg) {
        Car car = new Car();
        Car sonata = new Sonata();

// 어떤 메서드를 호출하는 지 모르므로 런타임시간에 바인딩 되므로 동적 바인딩
        car.getName();
        sonata.getName();
    }
}
출처

📚OOP의 5원칙과 4가지 특성

OOP의 4가지 특징

[CS] OOP (객체지향) & Functional (함수형) 프로그래밍 기초
