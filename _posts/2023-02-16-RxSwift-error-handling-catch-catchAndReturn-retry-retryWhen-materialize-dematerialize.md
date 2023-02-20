---
title:  "RxSwift) error handling - catch, catchAndReturn, retry, retryWhen, materialize, dematerialize"
categories:
- iOS

date:   2023-02-16  11:59:00 +0900
author_profile: false
---
## ✅ error handling - catch, catchAndReturn, retry, retryWhen, materialize, dematerialize

1. catch : error 를 새로운 Observable 또는 값으로 처리
2. retry : 재시도
3. materialize / dematerialize : sequence 를 제어해서 처리

### onError 에서 처리하면 되지 않나요?

Error 이벤트는 Observable 을 종료시키게 됩니다. 그래서 종료시키지 않고 다음과 같이 이벤트를 발생키시고 completed 가 발생되도록 error handling 을 하고자 합니다.

<img width="500" alt="1" src="https://user-images.githubusercontent.com/69136340/219395308-71c576e4-3359-4fa0-8be4-9c6cf49e9903.png">

### catch

- catch

```swift
extension ObservableType {

    /**
     Continues an observable sequence that is terminated by an error with the observable sequence produced by the handler.
     */
    public func `catch`(_ handler: @escaping (Swift.Error) throws -> Observable<Element>)
        -> Observable<Element> {
        Catch(source: self.asObservable(), handler: handler)
    }
}
```

<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/219395369-7add6f2a-f3b3-49c6-8fc6-dd43084d51ec.png">

next, completed 이벤트를 방출하지만 error 이벤트가 방출되면 기존의 Observable 에서 새로운 Observable 로 바꾸어서 핸들링할 수 있습니다.

예를 들어, 서버 통신 시에 next, completed  이벤트를 방출하다가 error 이벤트가 방출되면 내가 정해둔 동작으로 대체할 수 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/219395419-fcd2c68c-868e-4ff2-b909-e56cce98a4f4.png" width ="500">

```swift
let subject = PublishSubject<Int>()
let recovery = PublishSubject<Int>()

subject.catch { _ in
        recovery
    }
    .subscribe { print($0) }
    .disposed(by: disposeBag)

subject.onError(MyError.error)
subject.onNext(1) // Error 이벤트는 Observable 을 종료시키기 때문에 방출되지 않음.

recovery.onNext(2) // 새로운 Observable 이 next(2)을 방출함.
recovery.onCompleted()
```

- catchAndReturn

RxSwift 6.0 에서 `catchErrorJustReturn` 이 `catchAndReturn` 으로 이름이 바뀌었습니다.

[https://github.com/ReactiveX/RxSwift/releases/tag/6.0.0-rc.1](https://github.com/ReactiveX/RxSwift/releases/tag/6.0.0-rc.1)

```swift
extension ObservableType {
    /**
     Continues an observable sequence that is terminated by an error with a single element.

     - seealso: [catch operator on reactivex.io](http://reactivex.io/documentation/operators/catch.html)

     - parameter element: Last element in an observable sequence in case error occurs.
     - returns: An observable sequence containing the source sequence's elements, followed by the `element` in case an error occurred.
     */
    public func catchAndReturn(_ element: Element)
        -> Observable<Element> {
        Catch(source: self.asObservable(), handler: { _ in Observable.just(element) })
    }
}
```

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/219396386-d78dd610-bee2-4a90-9efd-cea0a7a2b0ca.png">

error 로 종료된 Observable 시퀀스가 단일 element 을 방출합니다.

error 가 발생했을 때 기본값을 방출하거나 로컬의 캐싱된 데이터를 방출할 때 사용됩니다.

error 의 종류와 상관없이 단일 element 를 방출하는 단점이 존재.

```swift
let subject = PublishSubject<Int>()

subject
    .catchAndReturn(0)
    .subscribe { print($0) }
    .disposed(by: disposeBag)

subject.onError(MyError.error) // next(0)
```

## retry

- retry

Observable 이 Error 이벤트를 방출하면 성공할 때까지 계속 시도하거나 최대 재시도 횟수를 지정할 수 있습니다.

```swift
extension ObservableType {

    /**
     Repeats the source observable sequence until it successfully terminates.
     */
    public func retry() -> Observable<Element> {
        CatchSequence(sources: InfiniteSequence(repeatedValue: self.asObservable()))
    }
```

<img width="700" alt="5" src="https://user-images.githubusercontent.com/69136340/219396545-471eb98c-8552-493d-a2de-ef20954a3bb2.png">

maxAttemptCount 파라미터로 최대 retry 횟수를 지정할 수 있다. 이때는 처음 시도 횟수도 포함한다.

```swift
extension ObservableType {
    /**
     Repeats the source observable sequence the specified number of times in case of an error or until it successfully terminates.
     */
    public func retry(_ maxAttemptCount: Int)
        -> Observable<Element> {
        CatchSequence(sources: Swift.repeatElement(self.asObservable(), count: maxAttemptCount))
    }
}
```

<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/219396590-e3bddda3-3ec2-45c8-bf0d-a79ecdfca5d9.png">

<img src="https://user-images.githubusercontent.com/69136340/219396641-0df449d5-92b5-46e0-9dea-e3adf4dde7bd.png" width ="500">

```swift
var attemptCount = 1

let source = Observable<Int>.create { observer in
    let currentAttemptCount = attemptCount

    print("#START => \(currentAttemptCount)")

    // 2번째 시도까지 에러 방출.
    if attemptCount < 3 {
      // 다른 경우 : 6번째 시도까지 에러 방출. -> retry(5). 최대 5번의 시도이기 때문에 6번째에서 error(myError) 출력.
        // if attemptCount < 7 {
        observer.onError(MyError.myError)
        attemptCount += 1
    }
    
    observer.onNext(1)
    observer.onCompleted()

    return Disposables.create {
        print("#END => \(currentAttemptCount)")
    }
}

source
    // 최대 4번의 재시도.
    .retry(5)
    .subscribe {
        print($0)
    }
    .disposed(by: disposeBag)

/* 
#START => 1
#END => 1
// Error 이벤트가 방출되었기 때문에 재시도.
#START => 2
#END => 2
// Error 이벤트가 방출되었기 때문에 재시도.
#START => 3
next(1)
completed
#END => 3
*/

// 다른 경우에는 다음을 출력.
/*
#START => 1
#END => 1
#START => 2
#END => 2
#START => 3
#END => 3
#START => 4
#END => 4
#START => 5
#END => 5
error(myError)
*/
```

- retry(When:)

RxSwift 6.0 에서 `retryWhen(_:)` 이 `retry(when:)` 으로 이름이 바뀌었습니다.

[https://github.com/ReactiveX/RxSwift/releases/tag/6.0.0](https://github.com/ReactiveX/RxSwift/releases/tag/6.0.0)

```swift
extension ObservableType {
    /**
     Repeats the source observable sequence on error when the notifier emits a next value.
     If the source observable errors and the notifier completes, it will complete the source sequence.
     */
    public func retry<TriggerObservable: ObservableType, Error: Swift.Error>(when notificationHandler: @escaping (Observable<Error>) -> TriggerObservable)
        -> Observable<Element> {
        RetryWhenSequence(sources: InfiniteSequence(repeatedValue: self.asObservable()), notificationHandler: notificationHandler)
    }
```

<img width="700" alt="9" src="https://user-images.githubusercontent.com/69136340/219396950-7b635932-09d0-4cc0-a0a6-a422fd36a4de.png">

trigger observable 을 만들어서 반환하게 되는데 이는 error 이벤트가 방출되었을 때 새로운 observable 을 만들고 해당 observable 의 next 이벤트가 방출되면 재시도하게 됩니다.

예를 들어, 서버통신이 실패한 뒤에 특정 시간 뒤에 다시금 재시도하는 Observable 을 만들 수도 있습니다.

즉, trigger observable 은 retry 를 trigger 하는데 사용됩니다.

retry 처럼 바로 재시도하는 것이 아니라 trigger observable 을 통해 retry 시점을 특정할 수 있습니다.

### materialize / dematerialize

- materialize

materialize 는 시퀀스를 Event enum sequence(next, error, completed) 로 변환할 수 있습니다.

• `Observable<Int>` → `Observable<Event<Value>>`

```swift
extension ObservableType {
    /**
     Convert any Observable into an Observable of its events.
     */
    public func materialize() -> Observable<Event<Element>> {
        Materialize(source: self.asObservable())
    }
}
```

<img width="500" alt="10" src="https://user-images.githubusercontent.com/69136340/219397023-bfd60f61-8f22-4a01-987d-d77a2ea1d7eb.png">


<img src="https://user-images.githubusercontent.com/69136340/219397048-6d8b1987-dad2-481a-9d29-c306f6b7e4c1.png" width ="500">

materialize 를 사용하면 1 과 2, completed 혹은 error 가 이벤트로 변환되서 Event<Data>, Event<Error>, Event<Completed> 가 됩니다. 따라서 onNext, onError, onCompleted 각각의 함수로 핸들링할 수 있습니다.

- dematerialize

materialize 로 변환한 Observable 을 원래 폼으로 전환할 수 있습니다.

• `Observable<Event<Value>>` → `Observable<Int>` 

```swift
extension ObservableType where Element: EventConvertible {
    /**
     Convert any previously materialized Observable into it's original form.
     */
    public func dematerialize() -> Observable<Element.Element> {
        Dematerialize(source: self.asObservable())
    }

}
```
<img width="500" alt="스크린샷 2023-02-11 오후 3 06 05" src="https://user-images.githubusercontent.com/69136340/219397244-3b669efe-2b82-4f05-a503-a3d4789a49ee.png">

<img scr="https://user-images.githubusercontent.com/69136340/219397268-77d32923-f4af-482d-82bf-2188ff92e578.png" width="500">

### 어떻게 error handling 하나요?

materialize 와 dematerialize 를 사용해서 실패하는 sequence 를 디버깅해서 다룰 수 있습니다.

이는 third party 프레임워크에서 생성된 것처럼 시퀀스가 제한되거나 제어할 수 없을 때 해결할 수 있습니다.

또한, observable 를 제어할 수 없고, 외부 시퀀스가 종료되지 않도록 오류 이벤트를 처리하는 경우가 있습니다. materialize 와 dematerialize 를 함께 사용하면 observable 을 변환하여 이벤트를 처리하고 다시금 observable 로 변환시킬 수 있습니다.

- 아래 코드는 error 이벤트의 경우 next 이벤트로 바꿔주고 있습니다.

```swift
let observable = Observable<Int>
    .create { observer -> Disposable in
        observer.onNext(1)
        observer.onNext(2)
        observer.onNext(3)
        observer.onError(NSError(domain: "", code: 100, userInfo: nil))
        observer.onError(NSError(domain: "", code: 200, userInfo: nil))
        return Disposables.create { }
}

observable
    .materialize()
    .map { event -> Event<Int> in
        switch event {
        case .error:
            return .next(999)
        default:
            return event
        }
    }
    .dematerialize()
    .subscribe { event in
        print(event)
    }
    .disposed(by: disposeBag)

// 출력 결과
next(1)
next(2)
next(3)
next(999)
completed
```

**출처**
    
[4. Error Handling - catch, catchAndReturn, retry](https://roniruny.tistory.com/269)
    
[https://github.com/fimuxd/RxSwift](https://github.com/fimuxd/RxSwift)
    
[Materialize/dematerialize](https://shoveller.tistory.com/entry/Materializedematerialize)
    
[RxSwift, Error 처리](https://brunch.co.kr/@tilltue/8)
    
[RxSwift) Error Handling Operators](https://velog.io/@hansangjin96/RxSwift-Error-Handling-Operators)
