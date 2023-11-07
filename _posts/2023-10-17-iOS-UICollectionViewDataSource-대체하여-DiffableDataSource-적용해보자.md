---
title:  "iOS) UICollectionViewDataSource 대체하여 DiffableDataSource 적용해보자"
categories:
- iOS

date:   2023-10-17  13:16:00 +0900
author_profile: false
---
## 내용

- UICollectionViewDataSource 를 대체하여 DiffableDataSource 를 적용해보자
- 코드 중심적으로 정리하여 추후에 사용할 때 도움이 되어보자

## 👉 들어가기 전

관련 글은 다음에 정리해뒀습니다.

[iOS) Diffable Data Source 알아보기](https://gyuios.tistory.com/153)

[iOS) DiffableDataSource 사용해서 collection view 를 업데이트해보자(개발자 문서)](https://gyuios.tistory.com/300)

## 👉 적용해보자

- 사용할 데이터 모델을 살펴보겠습니다.

```swift
public struct ReceivedTag: Codable {
    let adjective: String
    let cardTagID: Int
    // ...
    
    enum CodingKeys: String, CodingKey {
        case adjective, icon, noun
        case cardTagID = "cardTagId"
        // ...
    }
}
```

기존의 데이터 모델은 위와 같습니다. 이를 diffableDataSource 에 사용하기 위해서는 **Hashable 을 채택한 데이터 모델을 요구합니다.**

이는 모델이 갱신될때 비교해야하기 때문입니다.

<img width="700" alt="1" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/ddb91dc0-f099-45da-91cb-cffc1c91b026">

우선, 하나의 컬렉션뷰 섹션을 사용할 것이기 때문에 **SectionIdentifierType** 역할을 하기 위해서 다음과 같이 데이터 모델을 생성하였습니다.

```swift
private enum Section: Hashable {
    case main
}

// ✅ 많은 예제가 다음과 같이 CaseIterable 프로토콜을 채택합니다. 이것도 되는걸까요?
private enum Section: CaseIterable {
// CaseIterable 을 채택하면 자동으로 allCasese 를 얻게되는데 이때, 조건이 associated type 이 없는 경우입니다.
// 즉, CaseIterable 을 채택할 수 있다면 associaated type 없다는 것입니다.
// enum 타입에서 associated type 이 없다면 자동으로 Hashable 을 준수하게됩니다.
    case main
}
```

어떻게 해야 Hashable 을 준수할 수 있을까요? 기존의 데이터 모델이 Hashable 을 채택하도록 Hashable 에 대해서 알아봅시다.

(프로토콜을 채택하기 위해서 implement(구현)하는 것을 conformance(준수한다)라고 표현합니다.)

## 👉 Hashable 준수하기

Hashable 프로토콜은 

기본적으로 Strings, integers, floating-point, boolean values, sets 는 Hashable 을 채택합니다. optionals 이라던지 arrays 와 같은 타입들의 경우는 해당 타입이 hashable 하다면 자동으로 hashable 합니다.

커스텀 타입이 Hashable 을 다음과 같이 채택할 수 있습니다.

- associated value 가 없는 enum 타입은 자동으로 Hashable 를 준수
- hash(into:) 메소드를 구현하여 커스텀 타입이 Hashable 준수하도록 할 수 있습니다.
- Hashable 프로토콜은 Equatable 프로토콜을 상속하므로 해당 프로토콜도 준수해야 합니다.

Hashable 을 준수하기 위해서 구현해야하는 hash(into:) 메소드를 컴파일러가 자동으로 제공하는 경우도 있습니다.

- 구조체의 경우 저장된 프로퍼티들이 모두 Hashable.
- Hashable 한 associated value 를 가진 enum 타입.

(출처: https://developer.apple.com/documentation/swift/hashable)

클래스인 경우에 Hashable 을 준수하도록 해보겠습니다.

```swift
public class TagDataModel: Codable, Hashable {
    let name: String
    let id: Int
    
    public func hash(into hasher: inout Hasher) {
        hasher.combine(name)
        hasher.combine(id)
    }
    // hash(into:) 만 구현하면 Equatable 프로토콜을 채택하라는 에러 메시지가 뜸.    
    
    // Equatable 을 채택하는 Hashable 을 채택하기 때문에 구현해야 함.    
    public static func == (lhs: TagDataModel, rhs: TagDataModel) -> Bool {
        return lhs.name == rhs.name && lhs.id == rhs.id
    }
}
```

---

***다시 돌아와서 데이터 모델들을 수정해보겠습니다.***

저장 프로퍼티들이 Hashable 한 구조체의 경우 Hashable 을 준수하기 위해서 구현해야하는 바가 없기 때문에 다음과 같이 Hashable 프로토콜을 채택하겠습니다.

```swift
public struct ReceivedTag: Codable, Hashable {
    // ...
}
```

하지만, 데이터 모델 안에 기본 자료형이 아닌 다른 데이터 모델들이 들어있을 수 있습니다. 

TagDataModel 이 Hashable 하지 않기 때문에 ReceivedTag 데이터 모델은 Hashable 을 준수할 수 없습니다.

<img width="600" alt="2" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/46f338a8-0dff-428d-9afa-1b158553da0e">

이때는 다음과 같이 구현해줘야 합니다.

<img width="300" alt="3" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/dc1007d2-31a8-45e1-9a13-9cab9facd0bf">

Hashable 프로토콜을 채택하게 된 데이터 모델을 사용하여 다음과 같이 diffableDataSource 변수를 만들 수 있습니다.

```swift
private var diffableDataSource: UICollectionViewDiffableDataSource<Section, ReceivedTag>?   
```

- data source 를 대체하여 적용해보겠습니다.

```swift
// viewController.swift

    private var diffableDataSource: UICollectionViewDiffableDataSource<Section, ReceivedTag>?   

    // ...

    private func setDelegate() {
        collectionView.delegate = self
//        collectionView.dataSource = self
        collectionView.register(TagCVC.self, forCellWithReuseIdentifier: "TagCVC")
        
        diffableDataSource = UICollectionViewDiffableDataSource<Section, ReceivedTag>(collectionView: collectionView) { [weak self] collectionView, indexPath, itemIdentifier in
            guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "TagCVC", for: indexPath) as? TagCVC else { return UICollectionViewCell() }
            
            // 아래와 같이 동일한 동작으로 사용할 수 있습니다.
            cell.initCell(itemIdentifier.name)
            cell.initCell(receivedTags[indexPath.item].name)
            
            return cell
        }
        
        collectionView.dataSource = diffableDataSource
    }

// 초기 구성 후, 새로운 data source가 필요하다면 새로운 collection view 와 data source 를 만들어야 한다.
```

초기 데이터를 로드했을 때 컬렉션 뷰를 세팅해보겠습니다.

컬렉션 뷰에 데이터를 보여주기 위해서는 apply… 로 시작하는 4개의 메소드를 사용할 수 있습니다.

<img width="500" alt="4" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/d58f0f5b-c04d-4d03-9cd3-1bdad84703dd">

- **[applySnapshotUsingReloadData(_:)](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3804469-applysnapshotusingreloaddata)** : iOS 15 +. 차이점을 계산하거나 변경 사항에 애니메이션을 적용하지 않고, 스냅샷의 상태만 반영하도록 UI 를 재설정하는 메소드입니다.
- **[applySnapshotUsingReloadData(_:completion:)](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3804470-applysnapshotusingreloaddata)** : iOS 15 +. applySnapshotUsingReloadData(_:) 동일하고, 애니메이션 적용 후 completion handler 실행.
- **[apply(_:animatingDifferences:)](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3795617-apply)** :  iOS 15 +. UI 를 업데이트하고, 선택적으로 변경사항에 애니메이션을 적용합니다.
- **[apply(_:animatingDifferences:completion:)](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3375795-apply)** : iOS 13 +. apply(_:animatingDifferences:) 동일하고, 애니메이션 적용 후 completion handler 실행. 

[WWDC 2021 > Make blazing fast lists and collection views](https://developer.apple.com/videos/play/wwdc2021/10252/)을 살펴보면 iOS 13 이상 사용 가능한 apply(_:animatingDifferences:completion:) 과 iOS 15이상 사용 가능한 apply(_:animatingDifferences:) 의 동작 차이에 대해서 설명해줍니다.(2분40초부터)
- iOS 15 이전에는 animatingDifferences 파라미터가 false 인 경우에는 내부적으로 reloadData() 바뀌어서 동작하였습니다.(컬렉션 뷰의 모든 셀을 재생성해야 했기 때문에 퍼포먼스가 좋지 않았다고 합니다.)
- iOS 15 부터는 animatingDifferences 파라미터가 false 인 경우에도 차이만 적용하고 다른 extra work 를 하지 않도록 동작한다고 합니다.
- 그리고 해당 completion 파라미터는 nil 기본값을 가지고 있기 때문에 apply(_: animatingDifferences:) 를 사용하게되면, iOS 15 이전에는 apply(_:animatingDifferences:completion:) 메소드가 호출될 것이고 iOS 15 부터는 apply(_:animatingDifferences:) 가 호출될 것입니다. 즉, 코드 변화 없이 적용될 수 있습니다.

---

**초기 데이터 로드에서 applySnapshotUsingReloadData(_:) 메소드를 사용하여 변경사항  비교 없이 스냅샷의 상태를 반영하고, UI 업데이트를 위해서 apply(_:animatingDifferences:) 메소드를 호출할 수 있습니다.**

하지만, 저는 초기 데이터 로드 과정없이 서버 통신 후의 데이터 로드 과정을 가져갔기 때문에 다음과 같이 apply(_:animatingDifferences:) 메소드를 사용하여 적용하였습니다.

- snapshot 을 갱신하겠습니다.

```swift
private func setCollectionView() {
    var snapshot = NSDiffableDataSourceSnapshot<Section, ReceivedTag>()
        
    snapshot.appendSections([.main])
        
    if let receivedTags {
        snapshot.appendItems(receivedTags)
    }

    diffableDataSource?.apply(snapshot, animatingDifferences: true)
}

// 서버 통신 후 데이터를 반영하는데 사용.
RxAlamofire.requestData(.get, url)
    .map { $1 }
    .bind(with: self, onNext: { owner, data in
        owner.receivedTags = data
        DispatchQueue.main.async {
         // owner.collectionView.reloadData()
            owner.setCollectionView()
        }
    })
    .disposed(by: disposedBag)
```

애니메이션이 생겨 collection view 를 reloadData 하여 갑자기 변하는 것보다 자연스러워졌습니다.

<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/cb1a2304-5d83-4117-aba6-5d1e02ce6971" width ="250">

## 👉 느낀 점

- UICollectionViewDataSource 에서 필수록 구현해야했던 두 메소드를 구현하지않아도 되었고, 애니메이션을 위해서 별도의 코드없이도 자연스럽게 적용할 수 있었습니다. 채택하기 위해 요구하는 메소드를 구현해야하는 과정에서 diffable data source 객체를 만들어서 할당하는 과정이 새로워서 직관적이게 느껴졌던 것 같습니다.
- 또한, UICollectionViewDataSource 를 사용할때와 달리 해싱으로 item 에 접근하기 때문에 셀을 재구성하거나 새롭게 교체하거나 ID 를 활용하는 성능을 향상시킬 수 있는 여지가 주어지는 점도 장점으로 느껴졌습니다.
- 반면, 복잡한 프로세스는 아니지만 identifier 를 사용하기 위해서 데이터 타입은 반드시 Hashable 프로토콜을 채택해야만 하는 점이 있었습니다.

- 아래와 같이 섹션 수가 잘못되어 앱이 종료되는 경우를 방지할 수 있었습니다. 데이터의 변경 상황을 동기화하기 위해서 reloadData() 를 사용하게 됩니다. 이때 시간이 지남에 따라 UI 와 DataSource 역할을 하는 controller 가 own version of the truth 를 가지게 됩니다. 이 centralize 된 truth 가 없기 때문에 서로의 truth 가 맞지 않아 아래의 에러가 나는 것을 방지할 수 있었습니다.

<img src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/977aaaae-9bd4-45d5-87a2-24af6e176022" width ="700">

(출처 : [WWDC 2019 > Advances in UI Data Source](https://developer.apple.com/videos/play/wwdc2019/220))

- diffable data source 가 저장하는 section 과 item identifiers 는 변하지 않고 안정적인 identifiers 입니다. 이는 UICollectionViewDataSource 의 안정적이지 않은 indices 와 index path 와 대비됨을 경험하였습니다.

- identifiers 를 가진 diffable data source 는 컬렉션 뷰 내의 위치(indices, index path)에 대한 지식 없이 section 과 item 을 참조할 수 있었습니다.
