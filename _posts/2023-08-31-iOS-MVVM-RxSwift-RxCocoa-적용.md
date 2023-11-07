---
title:  "iOS) MVVM + RxSwift, RxCocoa 적용"
categories:
- iOS

date:   2023-08-31  18:28:00 +0900
author_profile: false
---
## ✅ RxSwift 는 mvvm 과 함께 쓰기로 유명한데 그 이유를 알아보자

리액티브 프로그래밍은 코드 업데이트가 변경사항을 자동으로 반영하도록 설정할 수 있다면 좋겠다는 생각에서 출발한 것입니다. 

그래서 데이터 스트림과 변경 사항의 전파를 중심으로 하는 비동기 프로그래밍이라고 정리할 수 있습니다. 이처럼 리액티브 프로그래밍을 사용하는 이유는 특정 상태를 유지하는 것보다 로직에 조금 더 집중할 수 있기 때문입니다.

이런 리액티브 프로그래밍의 사용을 할 수 있는 오픈 소스 라이브러리가 ReactiveX 이고, Swift 와 함께 사용할 수 있는 것이 바로 RxSwift 입니다.

그리고 RxSwift 는 mvvm 패턴에서 데이터 바인딩의 대표적인 방법으로 소개됩니다.

mvvm 패턴에서는 view model 을 통해서 view 와 model 의 양방향 바인딩이 가능합니다. 이때 RxSwift 는 이 양방향 바인딩을 도와주는 operator 들이 있기 때문에 쉽게 처리할 수 있기 때문에 주로 함께 사용되는 것이 그 이유입니다.

## ✅ RxCocoa 는 무엇인가요?

RxCococa 는 UI 에 집중할 수 있도록 애플리케이션을 제작하기 위한 도구들의 모음인 Cocoa Framework 를 Rx 와 합친 기능을 제공하는 라이브러리입니다. 즉, Rx 와 UI 관련 기능을 함께 래핑했다고 할 수 있습니다.

RxCocoa 를 사용하여 UI 구성요소에 반응형 확장(Reactive Extensions) 기능을 추가하여 UI 이벤트를 추가할 수 있습니다.

(데이터의 타입이 스트림을 타면서 변경되는 상태를 반응형이라고 한다.)

확인해보면 아래처럼 UI 요소들을 확장한 코드를 가지고 있습니다.

<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/a7e76e4a-1c1f-4b72-a91f-bd0bdb6af26e" width ="250">

## ✅ MVVM + RxSwift

구현해 볼 **내용**

- view 에 액션 추가. 터치 시에 해당 색상 변경. 터치가 벗어나거나 손가락이 떨어지면 색상 복귀
- 액션에 화면 전환 로직 추가
- 테스트 코드 작성

### MVVM

MVVM 의 단점은 정형화된 패턴이 없기 때문에 코드가 다 다를 수 있습니다. 그래서 input - ouput 을 적용한 MVVM 에 대해서 다뤄보려고 합니다.

MVVM 모델은 Model - View - View Model 입니다.

<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/c91d178e-413b-4cd8-ad3b-7574d080091d" width ="500">

model 에서 가져온 view 를 업데이트할 데이터를 view model 에서 처리하게 함으로써 Controller 가 복잡해지는 것을 줄여줍니다.

- **Data Binding :** View Model과 View 는 서로에게 데이터의 변경을 알려줄 수 있는 방법이 필요합니다. 이것이 바로 Data Binding 입니다.
- Model : data 를 담당.
    - Model 은 View 나 View Model 과 관계없이 독립적으로 구성
    - 앱이 사용자에게 보여지는 모습이나 제공방식과 관련이 없음
- View : 화면.
    - 뷰 레이아웃 관련 코드가 있음
    - 이벤트 발생 시에 View 는 View Model 에게 알림. View 에서만 View Model 에 접근 가능. 뷰 모델 프로퍼티 소유.
    - input-oupt 을 적용하면서 transform 함수를 가짐
        - 뷰의 input stream 을 View Model 로 전달하고, 반환값으로 output stream 을 뷰에 binding
- View Model : 뷰에 대한 모델.
    - View 의 모습이 어떤지 모름. 즉, View 와 View Model 간에 의존성이 없기 때문에 유닛 테스트를 하기에 좋음
    - transform 함수 : input stream 을 비즈니스 로직으로 처리한 후, Operator 로 가공하여 output stream 으로 반환
    - 뷰가 가지는 모델을 포함함(예를 들어, 화면에 표시되는 텍스트와 상태 등)
    - 비즈니스 로직을 담당하는 Model 프로퍼티 소유(Service 를 의미)
        - Model 의 역할이 비즈니스 로직까지이기 때문에 Entity + Service 로 보면되고 주로 역할에 따라 분리함
        - 서버 통신을 예시로 Repository 에서 서버에서 전달받은 jSON 자료형을 디코딩하여 Entity 로 변환
        - Service 역할은 Repository 에서 변환한 Entity 를 가공하는 비지니스 로직을 수행(예를 들어, Date 자료형의 dateFormat 을 변경하여 원하는 문자열로 가공. 이 문자열이 Model 에 들어감. 이 Model 객체를 View Model 에서 소유)

(아래 곰튀김님의 영상을 참고해서 MVVM 에 대한 내용을  추가로 작성하였습니다.)

https://www.youtube.com/watch?v=M58LqynqQHc&t=1283s

### input-ouput

<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/e1b1a2ba-6fae-43c2-a5e3-9beead8a51b6" width ="550">

출처 - [https://velog.io/@dawn_dancer/iOS-Rx-MVVM의-올바른-사용법-saebyuckchoom](https://velog.io/@dawn_dancer/iOS-Rx-MVVM%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-%EC%82%AC%EC%9A%A9%EB%B2%95-saebyuckchoom)

- 뷰로 부터 input 이 들어와서 뷰 모델에서 비즈니스 로직을 통해 output 으로 전달되는 흐름이 이해하기 쉬웠습니다.
- 뷰에서 출발한 스트림이 끊어지지 않고 변하면서 값이 전달되는 이러한 특성은 Rx 에서 반응형 프로그래밍을 나타내기에 적절합니다.
- subscribe 은 스트림의 종착점인 ViewController 에서 이루어지는 것이 좋습니다. 중간에 subscribe 을 통해서 스트림을 가공하는 형태가 아닌 Operator를 통해 데이터를 가공하거나 사이드 이펙트를 반영할 수 있습니다.

```swift
class MovieDetailViewModel {

    // 비즈니스 로직을 담당하는 객체(API Call)
    private let movieSearchService: MovieSearchService 

    init(movieSearchService: MovieSearchService) {
        self.movieSearchService = movieSearchService
    }

    struct Input { // View에서 발생할 Input 이벤트(Stream)들
        let trigger: Observable<Void> // viewWillAppear와 같은 trigger event
    }
    struct Output { // View에 반영시킬 Output Stream들
        let resultTitleText: Observable<String> // UILabel 등에 바인딩할 데이터 Stream
    }

// ✅ 추가적으로 프로토콜로 Input, Ouput 을 정의할 수도 있습니다.
// 이를 통해 의존성 역전 원칙(DIP)도 적용 가능합니다.

    func transform(input: Input) -> Output { // 뷰의 Input을 받아 Output으로 변형하는 메서드
        let resultTritleText = input.trigger
            .flatMap { _ in
                self.movieSearchService.fetchMovieDetail()
            }
            .map { $0.title }

        return Output(resultTitleText: resultTitleText)
    }
}

// 출처: https://velog.io/@dawn_dancer/iOS-Rx-MVVM의-올바른-사용법-saebyuckchoom
```

input-output 구조가 반복되기 때문에 다음과 같이 ViewModelType 프로토콜을 만들어주는 방법도 있다.

```swift
protocol ViewModelType {
    associatedtype Input
    associatedtype Output
    
    func transform(input: Input) -> Output
 }
```

### bind

`bind(to:)` 를 통해서 subscribe 수행할 수 있습니다.

`bind(to:)` 의 정의를 확인해보면 내부 동작에서 subcribe 하고 있는 것을 확인할 수 있습니다.

```swift
import RxSwift

extension ObservableType {
    /**
     Creates new subscription and sends elements to observer(s).
     In this form, it's equivalent to the `subscribe` method, but it better conveys intent, and enables
     writing more consistent binding code.
     - parameter observers: Observers to receives events.
     - returns: Disposable object that can be used to unsubscribe the observers.
     */
    public func bind<Observer: ObserverType>(to observers: Observer...) -> Disposable where Observer.Element == Element {
        self.subscribe { event in
            observers.forEach { $0.on(event) }
        }
    }
```

`bind` 와 이벤트를 핸들링하는 클로저에서 약한 참조를 사용하는 것에 대해서 여러 가지 방법을 살펴보겠습니다.

```swift
// bind 를 사용하면서 아래의 operator 는 사용하지 않아도 됩니다.
//.observeOn(MainScheduler.instance)
.bind(onNext: { [weak self] data in
    guard let self = self else { return }
    self.task1(data)
    self.task2(data)
})
.disposed(by: disposeBag)

// withUnretained 는 share(replay:1)과 같이 사용할 때 retain cycle 이 발생할 수 있다.
// 그래서 Driver 와는 사용할 수 없다고 함.
.withUnretained(self)
.bind(onNext: { owner, data in
    owner.task1(data)
    owner.task2(data)
})
.disposed(by: disposeBag)

// bind(with:onNext:)를 사용하면 약한 참조의 인스턴스를 전달할 수 있다.
.bind(with: self, onNext: { owner, data in
    owner.task1(data)
    owner.task2(data)
})
.disposed(by: disposeBag)

// self? 을 사용해야 한다면 다음과 같이 캡처 리스트를 생성하고 진행.
// bind(with:onNext:) 코드를 살펴보면 guard let 으로 강한 참조하여 전달하고 있음.
.bind(onNext: { [weak self] data in
    self?.task1(data)
    self?.task2(data)
})
.disposed(by: disposeBag)
```

### bind 와 drive 의 차이

- bind 와 drive 모두 main thread 에서 실행되기 때문에 UI 작업을 하기에 적절합니다.
- drive 는 Observable 에서 `asDriver()` 메소드를 통해 변환된 Driver 타입의 핸들링을 합니다.
    - Driver 는 RxCocoa 가 제공하는 UI 처리에 특화된 Observable 인 Traits 의 하나입니다.
- bind 는 Observer 가 추가될때마다 새로운 스트림이 실행됩니다. 반면, drive 는 스트림을 공유합니다. 그래서 하나의 스트림을 여러 곳에서 관찰하는 경우에 bind 보다 메모리의 이점이 존재합니다.
- bind 코드를 살펴보면 onError 가 없습니다. (Driver 는 error 를 방출하지 않기 때문에 bind 와 달리 애초에 없습니다.)
- 또한, drive 는 onCompleted, onDisposed 파라미터가 있기 때문에 사이드 이펙트를 처리하기에 용이합니다.

### 느낀점

공부를 하면서 더욱 느끼는 것이지만 패턴에는 표준이라는 것이 없다는 것이다. 개발자마다 패턴이 조금씩 달랐다. 조바심을 내지 않고 최대한 많은 부분들을 눈에 담는 것도 중요한 단계라고 생각이 들었다.

**출처 :** 

[[iOS] RxSwift를 이용한 Login 기능 구현](https://duwjdtn11.tistory.com/611)

[Getting Started With RxSwift and RxCocoa](https://www.raywenderlich.com/1228891-getting-started-with-rxswift-and-rxcocoa)

[[Rx] TIL - Rx 개념 재복습 및 총정리](https://roniruny.tistory.com/263)

[RxSwift #4 — RxSwift 를 이용한 MVVM 패턴](https://okanghoon.medium.com/rxswift-4-mvvm-with-rxswift-17a9b6d43746)

https://linux-studying.tistory.com/28

[https://velog.io/@dawn_dancer/iOS-Rx-MVVM의-올바른-사용법-saebyuckchoom](https://velog.io/@dawn_dancer/iOS-Rx-MVVM%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-%EC%82%AC%EC%9A%A9%EB%B2%95-saebyuckchoom)

https://duwjdtn11.tistory.com/628

https://jinsangjin.tistory.com/126
