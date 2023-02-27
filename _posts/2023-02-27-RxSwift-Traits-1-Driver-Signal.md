---
title:  "RxSwift) Traits(1) - Driver, Signal"
categories:
- iOS

date:   2023-02-27  17:32:00 +0900
author_profile: false
---
RxCocoa 에 있는 Traits 에 대해서 알아보자

### Traits?

Rx 를 보다 직관적이게 사용할 수 있는 강력한 방법이 있습니다.

Traits 는 선택적인 방법일뿐입니다. 핵심은 RxSwift 와 RxCocoa 에서 형변환되지 않은 Observable 시퀀스를 자유롭게 사용할 수 있습니다.

다음과 같이 Traits 의 코드를 살펴보면 단일 읽기 전용 Observable 시퀀스 프로퍼티를 가진 wrapper struct 라는 것을 알 수 있습니다.

```swift
struct Single<Element> {
    let source: Observable<Element>
}

struct Driver<Element> {
    let source: Observable<Element>
}
...
```

# RxCocoa traits

RxSwift 문서를 정리 및 요약해보겠습니다. (출처  - [RxCocoa Traits](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/Traits.md#rxcocoa-traits))

## 👉 Driver

UI 계층에서 반응형 코드를 작성하는 직관적인 방법을 제공하거나 앱을 Driving 하는 데이터 스트림을 모델링하려는 경우에 사용됩니다.

- error 를 방출하지 않습니다. 즉, 시퀀스 오류가 발생하더라도 앱은 input 에 대해서 반응하지 않습니다.
- main scheduler 에서 관찰이 발생합니다.
- 사이드 이펙트를 공유합니다.(`share(replay: 1, scope: .whileConnected)`)
    - 기본값으로 가장 최근에 방출했던 아이템의 값을 방출합니다.
    - 구독할 때마다 새로운 Observable 이 생성되지 않고, 공유할 수 있음.
    - `.whileConnected` : Subscriber 가 0개가 되고, disposed 되면 대상이 비워짐.

**Why is named Driver**

- “Drive UI from CoreData model.”
- “Drive UI using values form other UI elements ….”
- 앱을 drive 하는 시퀀스를 모델링하는 것에 의도되었습니다.

다음의 전형적인 코드에서 사용방법을 알아보겠습니다.

```swift
let results = query.rx.text
    .throttle(.milliseconds(300), scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
    }

results
    .map { "\($0.count)" }
    .bind(to: resultCount.rx.text)
    .disposed(by: disposeBag)

results
    .bind(to: resultsTableView.rx.items(cellIdentifier: "Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .disposed(by: disposeBag)
```

이 코드의 의도된 동작은 다음과 같습니다.

- Throttle user input.
- 서버에 접속해서 유저 결과의 리스트를 조회합니다.
- 결과를 두 개의 UI 요소(resultsTableView, resultCont label)에 바인딩합니다.

### 🚨 **이 코드의 문제는 무엇이 있을까요?**

- 만약, `fetchAutoCompleteItems` observable 시퀀스 오류(연결 실패 또는 파싱 에러)가 발생하면 이 오류는 모든 항목에 대해서 바인딩을 해제하고 UI는 더 이상 새 쿼리에 대해서 응답하지 않습니다.
    - 👉 즉, 시퀀스 오류가 발생하면 바인딩이 해제되기 때문에 UI 스트림이 끊어져 버린다는 것입니다.
- 만약, featchAutoCompleteItems 가 일부 백그라운드 스레드에서 결과를 반환한다면 결과는 non-deterministic crashes(비결정적인 충돌)를 일으킬 수 있는 백그라운드 스레드의 UI 요소에 바인딩됩니다.
    - 👉 즉, UI 요소의 업데이트는 main thread 에서 실행되어야 하는데 백그라운드 스레드에서 반환된 results 을 별도의 처리 없이 UI요소에 바인딩하고 있다는 것입니다.
- results 는 두 개의 UI 요소에 바인딩됩니다. 이것은 각 쿼리에 대해 두 개의 HTTP request 가 생성되고, 이는 의도된 동작이 아닙니다.
    - 👉 즉, 한 번의 HTTP request 를 통해 반환된 하나의 결과를 두 개의 UI 요소에 바인딩하고자 했는데 위의 코드는 UI 요소에 바인딩하는 두 개의 bind 코드에 대해서 각각 HTTP request 가 생성되게 된다는 것입니다.

좀 더 적절하게 코드를 변경해보겠습니다.

```swift
let results = query.rx.text
    .throttle(.milliseconds(300), scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
            .observeOn(MainScheduler.instance)  // ✅ results are returned on MainScheduler
         // .catchErrorJustReturn([])           // 👉 RxSwift 6 부터 이름이 변경되었습니다.
            .catchAndReturn([])                 // ✅ in the worst case, errors are handled
    }
    .share(replay: 1)                           // ✅ HTTP requests are shared and results replayed to all UI elements

results
    .map { "\($0.count)" }
    .bind(to: resultCount.rx.text)
    .disposed(by: disposeBag)

results
    .bind(to: resultsTableView.rx.items(cellIdentifier: "Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .disposed(by: disposeBag)
```

위의 변경점들이 충족되었음을 증명하는 더 간단한 방법이 있습니다. 아래 코드도 살펴보겠습니다.

```swift
let results = query.rx.text.asDriver()        // ✅ This converts a normal sequence into a `Driver` sequence.
    .throttle(.milliseconds(300), scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
            .asDriver(onErrorJustReturn: [])  // ✅ Builder just needs info about what to return in case of error.
    }

results
    .map { "\($0.count)" }
    .drive(resultCount.rx.text)               // ✅ If there is a `drive` method available instead of `bind(to:)`,
    .disposed(by: disposeBag)                 // that means that the compiler has proven that all properties
                                              // are satisfied.
results
    .drive(resultsTableView.rx.items(cellIdentifier: "Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .disposed(by: disposeBag)
```

**첫 번째** `asDriver` 메서드는 `ControlProperty` trait 를 trait 로 변환합니다.

```swift
query.rx.text.asDriver()
```

`Driver` 는 `ControlProperty` 의 모든 속성들과 더 많은 속성을 가집니다. 이때는 단지 기본 observable sequence 가 Driver 로 래핑됩니다.

**두 번째** `asDriver` 메서드로 모든 observable sequence 는 3가지 속성을 만족하는 `Driver` 로 변환될 수 있습니다.

```swift
.asDriver(onErrorJustReturn: [])
```

- error 가 발생할 수 없습니다.
- main scheduler 에서 관찰합니다.
- 사이드 이펙트를 공유합니다.(`share(replay: 1, scop: .whileConnected)`)

`.asDriver(onErrorJustReturn: [])` 코드는 다음 코드와 동일합니다.

**(현재는 이름이 바뀌고, 사용되지 않는 코드도 있으니 구조만 파악하는데 참고하시면 될 것 같습니다.)**

```swift
let safeSequence = xs
  .observeOn(MainScheduler.instance)        // observe events on main scheduler
  .catchErrorJustReturn(onErrorJustReturn)  // can't error out
  .share(replay: 1, scope: .whileConnected) // side effects sharing

return Driver(raw: safeSequence)            // wrap it up
```

- `asDriver` 구현부를 살펴보면 알 수 있습니다.

```swift
extension ObservableConvertibleType {
    /**
    Converts observable sequence to `Driver` trait.
    
    - parameter onErrorJustReturn: Element to return in case of error and after that complete the sequence.
    - returns: Driver trait.
    */
    public func asDriver(onErrorJustReturn: Element) -> Driver<Element> {
        let source = self
            .asObservable()
            .observe(on:DriverSharingStrategy.scheduler)
            .catchAndReturn(onErrorJustReturn)
        return Driver(source)
    }
```

마지막은 `bind(to:)` 대신 drive 를 사용한 코드입니다.

`drive` 는 Driver trait 에서만 정의됩니다.

**즉, 코드 어딘가에 drive 가 보이면 observable sequence 는 error 를 발생하지 않으며 main thread 에서 관찰하고, 이는 UI 요소에 바인딩하기 안전하다는 것을 의미합니다.**

## 👉 Signal

Signal 은 Driver 와 비슷하지만 한 가지 차이점이 있습니다. 구독 시 최신 이벤트를 replay 하지 않지만, subscribers 는 시퀀스의 computational resources 를 여전히 공유합니다.

***(코드의 주석에서는 이를 “observer 들의 개수에 따라 참조 카운트가 올라갑니다.” 라고 표현합니다. 즉, 시퀀스를 drive 와 동일하게 공유한다는 의미입니다.)***

이는 반응형 방식으로 명령형 이벤트를 모델링하는 빌더 패턴으로 간주할 수 있습니다.

- error 를 방출하지 않습니다.
- main scheduler 에 이벤트를 전달합니다.
- computational resources 를 공유(`share(scope: .whileConnected)`)(시퀀스를 공유한다.)
- 구독 시 요소를 replay 하지 않습니다.
    - 이는 아래의 구현부를 확인하면 share 에 replay 파라미터를 넘기지 않는데 이때 파라미터의 기본값이 0입니다.
    - replay 가 0이라는 것은 replay buffer 의 최대 개수가 0이라는 것입니다.

```swift
public typealias Signal<Element> = SharedSequence<SignalSharingStrategy, Element>

public struct SignalSharingStrategy: SharingStrategyProtocol {
    public static var scheduler: SchedulerType { SharingScheduler.make() }
    
    public static func share<Element>(_ source: Observable<Element>) -> Observable<Element> {
        source.share(scope: .whileConnected)
    }
}
```

**출처:** 

[[RxSwift] Share(replay:)](https://jusung.github.io/shareReplay/)

[RxSwift/Traits.md at main · ReactiveX/RxSwift](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/Traits.md)
