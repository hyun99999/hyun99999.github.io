---
title:  "RxSwift) withUnretained 와 이를 대체하는 subscribe(with:onNext:…)"
categories:
- iOS

date:   2023-07-05  16:30:00 +0900
author_profile: false
---
### 내용

- RxSwift 6(2021.01.01) 에서 추가된 `withUnretained` 와 이를 대체하기 위해 RxSwift 6.1(2021.02.11) 에서 추가된 `subscribe(with:onNext:onError:onCompleted:onDisposed:)` 에 대해서 알아보자.

## 📌 들어가기 전

`withUnretained` 는 [RxSwiftExt](https://github.com/RxSwiftCommunity/RxSwiftExt) 에서 만든 operator 이지만 RxSwift 6 부터 추가되었습니다.

**간단하게 먼저 언급하자면 클로저에서 retain cycle 을 피하고자 [weak self] 를 사용하는 것 대신 `withUnretained` operator 를 사용할 수 있습니다.**

그런데 **RxSwift 6.1** 에서 다음과 같은 api 를 추가되었습니다.(출처: https://github.com/ReactiveX/RxSwift/releases/tag/6.1.0)

> Add new `subscribe(with:onNext:onError:onCompleted:onDisposed:)` alternatives to `withUnretained`. This exists for all traits and types: `Observable`, `Driver`, `Signal`, `Infallible`, `Completable`, `Single`, `Maybe` https://github.com/ReactiveX/RxSwift/pull/2290

(`withUnretained` 를 대신하여 새롭게 `subscribe(with:onNext:onError:onCompleted:onDisposed:)` 를 추가)

그렇다면 **RxSwift 6.1 에서는 `withUnretained` 를 대체하기 위해서 왜 새로운 API 를 추가하였을까요?**

먼저, `withUnretained` 에 대해서 알아보겠습니다.

## ✅ withUnretained

클로저에서는 self 를 캡처하고 이때 발생하는 Strong Reference Cycle(강한 순환 참조)을 해결하기 위해서 캡처리스트를 사용할 수 있습니다.

그리고 weak(약한 참조) 를 가지기 위해서 다음과 같이 사용하였습니다.

```swift
.subscribe(onNext: { [weak self] data in
    guard let self = self else { return }
// 혹은 다음과 같이 옵셔널 체이닝 사용.
// self?.
```

클로저가 반복될 수록 [weak self] 가 반복적으로 사용되게 됩니다. 이를 `withUnretained` operator 로 쉽게 사용할 수 있도록 하였습니다.

```swift
    .withUnretained(self)
    // 튜플형태로 전달.
    .subscribe(onNext: { owner, data in
        owner.task(data)
    })
    .disposed(by: disposeBag)
```

그렇다면 여기서! `withUnretained` 는 어떤 방식으로 옵셔널 바인딩이 되는 걸까요? 코드를 살펴보겠습니다.

- guard-let 구문을 통해서 옵셔널 바인딩을 진행하고 있습니다.

```swift
public func withUnretained<Object: AnyObject, Out>(
        _ obj: Object,
        resultSelector: @escaping (Object, Element) -> Out
    ) -> Observable<Out> {
        map { [weak obj] element -> Out in
            guard let obj = obj else { throw UnretainedError.failedRetaining }

            return resultSelector(obj, element)
        }
        .catch{ error -> Observable<Out> in
            guard let unretainedError = error as? UnretainedError,
                  unretainedError == .failedRetaining else {
                return .error(error)
            }

            return .empty()
        }
    }
```

## ✅ **subscribe(with:onNext:onError:onCompleted:onDisposed:)**

`***withUnretained` 를 대체한 이유에 대해서 설명하는 PR 입니다. 일부를 번역해보겠습니다.***

https://github.com/ReactiveX/RxSwift/pull/2290

해당 PR 은 다음과 같습니다.

- Driver 에서 `withUnretained` 는 hard deprecated.(Driver 와 함께 사용하면 문제가 되기에 deprecated)
- `withUnretained` 에 대한 문서를 업데이트하여 `replaying` 동작을 조심하도록 한다.(문제가 되는 관련 이유 입니다.)
- **Observable, Driver, Signal, Infallible, Completable, Single, Maybe** 에 새로운 operator 인 `subscribe(with:onNext:onError:onCompleted:onDisposed:)` 추가.

다음의 고려하지 못한 사이드 이펙트가 있었습니다.

**작업 중인 스트림에 `share(replay: 1)` 작업이 있는 경우, replay buffer 는 값과 함께 self 를 버퍼링하므로 일부 시나리오에서 retain cycle 을 만들어낼 수 있습니다.**

Driver 는 특별하게 기본적으로 마지막 값을 반복하기 때문에 Driver 에 대해서 `withUnretained` 을 deprecated 해야 했습니다. 여전히 다른 traits 인 Observable, signal 에서는 사용할 수 있습니다.

중단과 함께 유사한 작업을 수행할 수 있도록 새로운 api 를 추가하였습니다.

```swift
driver
  .drive(with: self,
         onNext: { object, value in ... 
           // object is self, value is the emission from `driver`
         })

// available for all pieces in RxSwift and RxCocoa: Observable, Infallible, Driver, Signal, Completable, Single, and Maybe.
```

여기까지 PR 을 살펴 보았는데요.

RxSwift 6.1 에서 추가된 **subscribe(with:onNext:onError:onCompleted:onDisposed:)** 를 살펴봅시다.

다른 점은 바로 with 파라미터를 통해서 unretained reference 객체를 전달하게 됩니다.

```swift
public func subscribe<Object: AnyObject>(
        with object: Object,
        onNext: ((Object, Element) -> Void)? = nil,
        onError: ((Object, Swift.Error) -> Void)? = nil,
        onCompleted: ((Object) -> Void)? = nil,
        onDisposed: ((Object) -> Void)? = nil
    ) -> Disposable {
        subscribe(
            onNext: { [weak object] in
                guard let object = object else { return }
                onNext?(object, $0)
            },
            onError: { [weak object] in
                guard let object = object else { return }
                onError?(object, $0)
            },
            onCompleted: { [weak object] in
                guard let object = object else { return }
                onCompleted?(object)
            },
            onDisposed: { [weak object] in
                guard let object = object else { return }
                onDisposed?(object)
            }
        )
    }
```

- 이를 통해서 onNext 뿐만 아니라 onError, onCompleted, onDisposed 에도 약한 참조를 한 캡처리스트를 사용할 수 있습니다.(이전에는 onError 경우에는 `withUnretained` 를 사용하게 되도 약한 참조를 가지는 객체를 전달하지 못함.)
- 또한, guard-let 구문을 통해서 옵셔널 바인딩을 하는 것을 알 수 있습니다.

이 외에도 다음과 같이 사용하면 되겠습니다.

```swift
.bind(with: self, onNext: { owner, data in
        owner.task(data)
    })
    .disposed(by: disposeBag)

.asSignal()
      .emit(with: self, onNext: { owner, data in
          owner.task(data)
      }, onCompleted: { owner in
          // do something
      }, onDisposed: { owner in
          // do something
      })
      .disposed(by: disposeBag)

.asDriver()
    .drive(with: self, onNext: { owner, data in
        owner.task(data)
    }, onCompleted: { owner in
        // do something
    }, onDisposed: { owner in
        // do something
    })
    .disposed(by: disposeBag)
```

---

**참고:**

- https://github.com/ReactiveX/RxSwift/releases

- https://phillip5094.tistory.com/108
