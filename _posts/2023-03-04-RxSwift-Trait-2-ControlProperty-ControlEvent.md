---
title:  "RxSwift) Trait(2) ControlProperty, ControlEvent"
categories:
- iOS

date:   2023-03-04  17:32:00 +0900
author_profile: false
---
# RxCocoa traits

RxSwift 문서를 정리 및 요약해보겠습니다. (출처  - [RxCocoa Traits](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/Traits.md#rxcocoa-traits))

ControlProperty 와 ControlEvent 는 UI 요소의 속성을 나타내는 Observable/ObservableType 의 trait 입니다.

## 👉 ControlProperty

ControlProperty 는 ObservableType 과 ObserverType 을 동시에 채택하는 ControlPropetyType 을 채택합니다.

**Subject 와 같이 프로퍼티에 새로운 값을 관찰할 수도 방출할 수도 있습니다.**

```swift
public struct ControlProperty<PropertyType> : ControlPropertyType {
    ...
}

/// Protocol that enables extension of `ControlProperty`.
public protocol ControlPropertyType : ObservableType, ObserverType {

    /// - returns: `ControlProperty` interface
    func asControlProperty() -> ControlProperty<Element>
}
```

예를 들어, `UITextField+Rx.Swift` 에서는 다음과 같이 text 를 ControlProperty 로 가집니다.

```swift
extension Reactive where Base: UITextField { ...
    public var text: ControlProperty<String?> {
        return value
    }
...
}
```

Observable 채택한 text 에 대한 변경사항을 관찰하기 위해서 subscribe 를 수행할 수 있습니다.

**ObserverType 채택하였기 때문에 다음과 같이 사용할 수 있습니다.**

drive(_:) 와 bind(to:) 는 ObserverType 을 채택한 객체를 파라미터로 전달할 수 있습니다.

```swift
.drive(textfield.rx.text)

.bind(to: textfield.rx.text)
```

**위의 예시처럼 ControlProperty 는 시퀀스가 오직 초기 control value 와 사용자가 시작한 값 변경만을 나타냅니다.**

속성은 다음과 같습니다.

- never fails
- never errors out
    - UI 작업에 특화되있는 RxCocoa 의 Trait 이기 때문에 스트림이 끊기지 않도록 실패를 하지 않도록 합니다.
- `share(replay: 1)` behavior
    - 구독 시에 마지막 요소가 있는 경우 즉시 replay 됩니다.
- 할당 해제될때 `Complete` 방출됩니다.
- `MainScheduler.instance` 에 이벤트를 전달합니다.
    - `ControlEvent` 의 구현은 이벤트 시퀀스가 main scheduler(`subscribeOn(ConcurrentMainScheduler.instance)` 동작)에서 구독되도록 합니다.

좀 더 예시 코드를 살펴보겠습니다. `UISearchBar+Rx.swift` 에서는 text 를

```swift
 extension Reactive where Base: UISearchBar {
    /// Reactive wrapper for `text` property.
    public var value: ControlProperty<String?> {
        let source: Observable<String?> = Observable.deferred { [weak searchBar = self.base as UISearchBar] () -> Observable<String?> in
            let text = searchBar?.text
            
            return (searchBar?.rx.delegate.methodInvoked(#selector(UISearchBarDelegate.searchBar(_:textDidChange:))) ?? Observable.empty())
                    .map { a in
                        return a[1] as? String
                    }
                    .startWith(text)
        }

        let bindingObserver = Binder(self.base) { (searchBar, text: String?) in
            searchBar.text = text
        }
        
        return ControlProperty(values: source, valueSink: bindingObserver)
    }
}
```

UISegmentedControl 에서는 `selectedSegmentIndex` 를 래핑하고 있습니다.

```swift
extension Reactive where Base: UISegmentedControl {
    /// Reactive wrapper for `selectedSegmentIndex` property.
    public var selectedSegmentIndex: ControlProperty<Int> {
        value
    }
    
    /// Reactive wrapper for `selectedSegmentIndex` property.
    public var value: ControlProperty<Int> {
        return UIControl.rx.value(
            self.base,
            getter: { segmentedControl in
                segmentedControl.selectedSegmentIndex
            }, setter: { segmentedControl, value in
                segmentedControl.selectedSegmentIndex = value
            }
        )
    }
}
```

## 👉 ControlEvent

`ControlEvent` 는 `ObservableType` 을 채택하는 `ControlEventType` 을 채택합니다.

```swift
public struct ControlEvent<PropertyType> : ControlEventType {
    ...
}

/// A protocol that extends `ControlEvent`.
public protocol ControlEventType : ObservableType {

    /// - returns: `ControlEvent` interface
    func asControlEvent() -> ControlEvent<Element>
}
```

속성은 다음과 같습니다.

- never fails
- never errors out
    - UI 작업에 특화되있는 RxCocoa 의 Trait 이기 때문에 스트림이 끊기지 않도록 실패를 하지 않도록 합니다.
- 구독 시 초기 값을 보내지 않습니다.
- 할당 해제될때 `Complete` 방출됩니다.
- `MainScheduler.instance` 에 이벤트를 전달합니다.
    - `ControlEvent` 의 구현은 이벤트 시퀀스가 main scheduler(`subscribeOn(ConcurrentMainScheduler.instance)` 동작)에서 구독되도록 합니다.

예를 들어, `UIButton+Rx.Swift` 

```swift
extension Reactive where Base: UIButton {
    
    /// Reactive wrapper for `TouchUpInside` control event.
    public var tap: ControlEvent<Void> {
        controlEvent(.touchUpInside)
    }
}
```

`UICollectionView+Rx.swift` 에서도 확인할 수 있습니다.

```swift
extension Reactive where Base: UICollectionView {

    /// Reactive wrapper for `delegate` message `collectionView:didSelectItemAtIndexPath:`.
    public var itemSelected: ControlEvent<IndexPath> {
        let source = delegate.methodInvoked(#selector(UICollectionViewDelegate.collectionView(_:didSelectItemAt:)))
            .map { a in
                return a[1] as! IndexPath
            }

        return ControlEvent(events: source)
    }
}
```

`tap`, `itemSelected` 의 변경사항을 관찰하기 위해서 subscribe 을 사용할 수 있습니다.

**출처 :**

[https://github.com/ReactiveX/RxSwift/blob/main/Documentation/Traits.md](https://github.com/ReactiveX/RxSwift/blob/main/Documentation/Traits.md)

[[RxSwift] ControlProperty, ControlEvent](https://inuplace.tistory.com/1101)

[https://jusung.github.io/shareReplay/](https://jusung.github.io/shareReplay/)
