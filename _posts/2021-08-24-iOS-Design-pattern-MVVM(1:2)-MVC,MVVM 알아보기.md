---
title:  "iOS) Design pattern MVVM(1/2) - MVC, MVVM 알아보기"
categories:
- iOS

date:   2021-08-24  00:07:00 +0900
author_profile: false
---
# 🌂 Design Pattern

- 디자인 패턴을 정하게 되면 모든 클래스와 속성을 구조적으로 정리가능하며 팀 작업 시 원활한 의사소통과 코드 수정이 가능하다.

### ❗️ MVC(Model-View-Controller)

- Model : 데이터에 관한 로직 담당(데이터 값 변경 및 관리)
- View : 사용자에게 보여지는 화면 담당(UI)
- Controller : Model 과 View 연결(Model 값을 View 에 보여줌)
- 오리지날 MVC 패턴은 iOS 개발에 적합하지 않아(Model,View,Controller 가 너무 밀접하게 연관) 애플에서는 CocoaMVC 패턴을 제시했다.
- Controller 가 View 와 Model 의 중재자 역할을 하여 View 와 Model 에 독립성 부여.

<img src ="https://user-images.githubusercontent.com/69136340/130470778-120ce38d-0675-4ed1-9990-cafdf405d0c8.PNG" width ="500">

- 하지만 애플의 CocoaMVC 패턴에서는 View 와 Controller 의 역할 분리가 완벽하지 않다.
    - Controller 역할을 수행하는 UIViewController 은 Controller 가 View 를 포함하는 것은 물론 View 의 life-cycle 까지 관리하기 때문에 무거워지고, 분리가 어렵고, 재사용도 어렵고, 각각 테스트하기도 어렵다.
    - viewDidLoad 와 같은 life-cycle 이벤트 관리, @IBAction 등의 콜백처리 등으로 Massive ViewController 가 된다.
    - 그래서 실제로는 아래와 같은 구조가 된다.

<img src ="https://user-images.githubusercontent.com/69136340/130470768-d6a52036-2ac1-4f6d-afdb-e69bcfd18329.PNG" width = "600">

### 장단점

- 장점 : 생산성이 높다. 역할 분담하여 빠르게 구현. 확장성이 높다.
- 단점 : Apple's MVC 의 경우 많은 코드들이 Controller 에 집중되어 커지게 된다. 후에 유지보수가 어렵다는 문제 존재.

---

### ❗️ MVVM(Model-View-ViewModel)

- MVC 패턴의 Controller 가 커지는 문제를 View Model 을 둠으로써 해결하고자 한다.
- Model 에서 가져온 View 에 업데이트할 데이터를 View Model 이 처리하게함으로써 Controller 가 복잡해지는 것을 줄여준다. 즉, View Model 은 presentation logic 을 다루게 되지만 UI 는 다루지 않는다.

<img src ="https://user-images.githubusercontent.com/69136340/130470849-467f3eb2-81bd-4da5-9238-e6fc592c3148.PNG" width ="600">

- MVVM 에서 View Model 은 View 와 1:n 관계를 이룬다. View 와 View Model 간에는 데이터 바인딩을 통해 특정 View 의 속성과 View Model 속성을 연결한 뒤 View Model 속성이 변경되면 자동으로 View 를 업데이트.

**Data Binding**

:  View Model과 View 는 서로에게 데이터의 변경을 알려줄 수 있는 방법이 필요하다. 이것이 바로 Data Binding 이다. 

### Model

- Model 은 View 나 View Model 과 관계없이 독립적으로 구성. 앱이 사용자에게 보여지는 모습이나 제공방식과 관련이 없다.

### View

- 사용자가 볼 수 있는 요소들. 이벤트 발생 시  View 는 View Model 에 알린다. View 는 View Model 에만 접근 가능. View 의 event 를 View Model 에게 알려주면 View Model 이 Model 을 업데이트 시킨다.

### View Model

- Model 과 View 의 중개자. View Model 은 View 의 상태를 저장하고, View 에서 발생하는 액션에 따라 수행할 앱의 기능을 정의하는 명령을 구현. View Model 은 View 가 어떤 모습인지 전혀 알지 못한다.

### 장단점

- 장점 : ViewModel 은 View 와 독립적이다. 즉 UIKit 관련 코드가 없다. 따라서 MVC 모델보다 유닛 테스팅하기가 좋다.(View 와 View Model 간의 의존성이 없기 때문)
- 단점 : ViewModel 설계가 만만치 않다. View 에 대한 처리가 복잡해질수록 ViewModel 이 오버스펙이 될 수 있다.

### 출처:

[https://eunjin3786.tistory.com/31?category=837198](https://eunjin3786.tistory.com/31?category=837198)
