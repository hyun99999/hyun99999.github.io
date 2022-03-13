---
title:  "iOS) Diffable Data Source 알아보기"
categories:
- iOS

date:   2022-02-26  19:04:00 +0900
author_profile: false
---
# 👷Diffable Data Source 란?

- 먼저 `Diffable Data Source` 가 무엇인지에 대해서 간단하게 알고 넘어가보자!

> TableView(또는 CollectionView)를 그리기 위한 데이터를 관리하고 UI를 업데이트 하는 역할을 한다. `Data Source` 와 달리 데이터가 달라진 부분을 추적하여 **자연스럽게 UI를 업데이트한다.**
> 
- 기본적으로, `Diffable Data Source` 와 `Data Source` 의 역할은 같다. 그러나, `Diffable Data Source` 를 사용하면 table view 나 collection view 를 **간소화하게 업데이트가 가능**하다.
- `Data Source` 는 **Protocol** 이다. 반면에 `Diffable Data Source` 는 **Generic Class**이며, 해당 클래스가 `Data Source` 를 채택하고 있다.

## WWDC

[WWDC 2019 > Advances in UI Data Source](https://developer.apple.com/videos/play/wwdc2019/220) 에서 소개되었고, iOS 13 부터 사용 가능합니다. 그리고 [WWDC 2021 > Make blazing fast lists and collection views](https://developer.apple.com/videos/play/wwdc2021/10252/)[](https://velog.io/@ellyheetov/UI-Diffable-Data-Source) 에서 diffable data source 에 대한 개선사항이 소개되었다.

소개글을 살펴보고 정리해보았다.

### 🧑🏻‍💻 WWDC 2019

- UI Data Sources 를 사용하면 자동 비교를 통해서  table view 와 collection view items 의 업데이트를 간소화할 수 있습니다.
- 고품질의 애니메이션이 자동으로 이루어지고, 추가적은 코드가 필요하지 않습니다!
- 개선된 data source 메커니즘은 동기화 버그, 예외 및 충돌을 완전히 방지합니다!
- UI 데이터 동기화의 사소한 부분 대신 앱의 동적 데이터와 콘텐츠에 집중할 수 있도록 `identifiers` 와 `snapshots` 를 사용하는 간소화된 data model 에 대해 알아보세요.

### 🧑🏻‍💻 WWDC 2021

- 일관되게 부드러운 scrolling list 그리고 collection views: 셀의 lifecycle 을 탐색하고 해당 지식을 적용해서 거친 스크롤 및 누락된 프레임을 제거하는 방법을 배우세요.
- 또한 최적화된 이미지 로딩 및 automatic cell prefetching 을 통해서 전반적인 스크롤 경험을 개선하고 비용이 많이드는 장애를 피하는 방법을 보여줍니다......(생략)

### ❓이전에 사용된 UITableView/CollectionViewDataSource 의 역할은?

- section 과 row / item 의 수를 제공
- 각 행에 대한 셀을 제공
- section 의 header 와 footer 제공
- 데이터 변화에 반응

등등 역할을 하고 있었어요! 필수로 구현해야할 메서드도 존재했었죠! collection view 로 알아보자구여

- collection view 를 사용하기 위해서는 프로토콜의 두가지 메소드를 반드시 구현해주어야 했어요!

```swift
// MARK: UICollectionViewDataSource

// ✅ 각 section 의 아이템 갯수 제공.
func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
    return 1
}
// ✅ 각 행에 대한 셀 제공.
func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath) as? HomeCollectionViewCell else { return UICollectionViewCell() }

    return cell
}
```

### ❓Why?

그럼 왜? Diffable Data Source 가 등장한걸까요?

`WWDC 2019` 에서 잘 설명해주고 있는데요. 기존 Controller 와 UI 관계를 살펴보자구요.

Controller 가 웹서비스 응답을 받고, 델리게이트를 처리해요. 그리고 UI 에게 바뀌었다고 전달해요. 그리고 우리는 에러를 마주합니다.

<img src="https://user-images.githubusercontent.com/69136340/155838546-61c720b7-a7ce-4b32-94c0-21839bae7704.png" width="600">

그리고 Diffalbe Data Source 는 바로 다음 상황을 해결하기 위한 것입니다.

<img src="https://user-images.githubusercontent.com/69136340/155838550-4174e3f0-49cb-4568-8c91-9b47820b7676.png" width ="700">

- 섹션 수가 잘못되어 앱이 종료되는 경우에요! 위의 에러는 데이터의 변경 상황을 수동으로 동기화해야함을 의미해요. 따라서 `reloadData` 를 통해서 동기화해주어야 해요.

### ❓뭐가 문제죠?

가장 큰 문제는 UI 와 DataSource 역할을 하는 Controller 가 시간이 지남에 따라 변하는 자기들만의 버전인 truth 를 가지고 있다는 것입니다.(own version of the truth)

이 truth 가 서로 맞지 않게되면 위와 같은 에러가 나는것이죠! 이러한 접근방식은 오류가 발생하기 쉽습니다. centralize 된 truth 가 없기 때문입니다.

---

자 다시 돌아가볼게요!

WWDC 에서도 `reloadData` 를 통한 동기화를 괜찮다고 해요. 그러나 모두가 알다시피 `reloadData` 를 하게 되면 애니메이션 없이 나타납니다.

그리고 이는 사용자 경험(UX) 를 저하시킨다고 해요. 대신 `Diffable Data Source` 는 변경된 데이터 부분에 대해서 자연스러운 업데이트를 제공합니다. 그래서 위에서 그렇게 고품질의 애니메이션, 스크롤 경험 개선에 대해서 강조한 것이죠.

- data source

<img src="https://user-images.githubusercontent.com/69136340/155838606-70dd4d5d-c4f8-426f-9491-e290305218d0.gif" width ="250">

출처: [https://velog.io/@ellyheetov/UI-Diffable-Data-Source](https://velog.io/@ellyheetov/UI-Diffable-Data-Source)

- diffable data source

<img src="https://user-images.githubusercontent.com/69136340/155838892-483797b5-16e9-4388-b37d-733651d0b96c.gif" width ="250">

출처: [https://velog.io/@ellyheetov/UI-Diffable-Data-Source](https://velog.io/@ellyheetov/UI-Diffable-Data-Source)

### ❓그렇다면 기존에는 자연스러운 애니메이션이 없었나요?

- 기존은 특정 cell만 바뀐경우의 처리는 아래와 같이 처리했어요.
    - 방법 1) `performBatchUpdates()`의 클로저 블록에 데이터 변경 작업 추가.(insert, delete, reload, move 연산을 그룹으로 묶어서 animate 한다.)
    - 방법 2) `beginUpdates()`와 `endupdates()` 사이에 데이터 변경 작업 작성.(일부분만을 변화시키는 방법. 애니메이션 블록을 만들 수 있다.)

## ❓Diffable DataSource 에서는?

- iOS 13+ 에서는 [apply()](https://developer.apple.com/documentation/uikit/uitableviewdiffabledatasource/3375811-apply)) 를 통해서 위 작업들을 처리 가능해요!

<img src="https://user-images.githubusercontent.com/69136340/155838666-22a4f4d0-2b30-44c7-adaf-3bd6aa551eaf.png" width ="600">

### ✨ ****apply(_:animatingDifferences:completion:)****

- snapshot 의 데이터 상태를 반영하도록 UI 를 업데이트하고, 선택적으로 UI 변경사항에 애니메이션을 적용하고 completion handler 를 실행한다.

**Parameter**

- snapshot: table / collection view 에서 데이터의 새 상태를 반영하는 snapshot.
- animatingDifferences: `true` 인 경우, 시스템은 table / collection view 에 대한 업데이트를 애니메이션으로 만든다. `false` 인 경우, 시스템은 업데이트를 애니메이션하지 않는다.
- completion: 애니메이션이 완료되면 실행할 클로저. 시스템은 main queue 에서 이 클로저를 호출한다.

### ✨ Snapshot

- `Snapshot` 은 간단히 말해서 현재 UI state 의 truth 입니다.
- section 과 item 에 대해 unique identifiers 가 있습니다.
- IndexPath 가 아니라 unique identifiers 로 업데이트하게 됩니다.

<img src="https://user-images.githubusercontent.com/69136340/155838739-bcb1d544-a7e3-41c9-9f69-214d6ad2f16a.png" width="600">

### ❗️예제를 통해서 알아보자구요!

- FOO, BAR, BIF 로 구성된 Cureent Snapshot 이 존재합니다.

<img src="https://user-images.githubusercontent.com/69136340/155838922-8f6ec248-e20f-4ea4-8bdc-a00a9885db4f.png" width ="300">

- 그 후, Controller가 변경되었다고 가정해봅시다. 즉, apply() 할 수 있는 새로운 `Snapshot` 이 생긴거죠.

<img src="https://user-images.githubusercontent.com/69136340/155838929-b1edf0bf-2547-48e5-8e6a-64f97e51ee84.png" width ="600">

---


# 👷사용해보자!

## 1️⃣ UICollectionViewDiffalbeDataSource 알아보기

- Table View 도 거의 동일하기때문에 UICollectionView 로 진행하겠습니다!
- 개발자문서를 살펴봅시다!

### ✨ [UICollectionViewDiffableDataSource](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource)

데이터를 관리하고 collection view 에 대한 cells 을 제공하는데 사용하는 개체입니다.

**Declaration**

```swift
@MainActor class UICollectionViewDiffableDataSource<SectionIdentifierType, ItemIdentifierType> : [NSObject](https://developer.apple.com/documentation/objectivec/nsobject) where SectionIdentifierType : [Hashable](https://developer.apple.com/documentation/swift/hashable), ItemIdentifierType : [Hashable](https://developer.apple.com/documentation/swift/hashable)
```

**Overview**

diffable data source 개체는 collection view 개체와 함께 작동하는 특수한 유형의 data source 입니다. 데이터와 UI 에 대한 업데이트를 간단하고, 효율적인 방식으로 관리하느데 필요한  동작을 제공합니다. 또한 UICollectionViewDataSource 프로토콜을 준수하고 프로토콜의 모든 메서드에 대한 구현을 제공합니다.

**구현 순서**

1. Connect a diffable data source to your collection view.
2. Implement a cell provider to configure your collection view’s cells.
3. Generate the current state of the data.
4. Display the data in the UI.

1️⃣

diffable data source 를 collection view 와 연결하기 위해서는 `init(collectionView:cellProvider:)` 이니셜라이저를 사용해서 만들어주고, data source 와 연결하려는 collection view 를 전달합니다.

2️⃣

또한 각 셀을 구성하여 UI 에 데이터를 표시하는 방법을 결정하는 `cell provider` 를 전달합니다.

```swift
dataSource = UICollectionViewDiffableDataSource<Int, UUID>(collectionView: collectionView) {
    (collectionView: UICollectionView, indexPath: IndexPath, itemIdentifier: UUID) -> UICollectionViewCell? in
    // Configure and return cell.
}
```

3️⃣

그런 다음, `Snapshot` 을 구성하고 적용하여 데이터의 current state 를 생성하고 UI 에 데이터를 표시한다. 더 많은 내용은 [NSDiffableDataSourceSnapshot](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot) 를 참고하십시오.

🚧 **Important
-** diffable data source 로 구성한 후 collection view 에서 dataSource 를 변경하지 마세요.
- 만약 collection view 가 초기 구성한 후 새로운 data source 를 필요한다면, 새 collection view 와 diffable data source 를 만들고 구성합니다.

### ❓달라진 부분을 어떻게 알아차리는 걸까요?

결론부터 말하자면 `Hash Value` 를 사용합니다.

`Diffable Data Source` 를 사용하기 위해서는 다음 두가지 generic type 을 가집니다.

- section identifier
- item identifier

```swift
// Diffalbe Data Source
@MainActor class UICollectionViewDiffableDataSource<SectionIdentifierType, ItemIdentifierType> : [NSObject](https://developer.apple.com/documentation/objectivec/nsobject) where SectionIdentifierType : [Hashable](https://developer.apple.com/documentation/swift/hashable), ItemIdentifierType : [Hashable](https://developer.apple.com/documentation/swift/hashable)
```

```swift
// Snapshot
struct NSDiffableDataSourceSnapshot<SectionIdentifierType, ItemIdentifierType> where SectionIdentifierType : Hashable, ItemIdentifierType : Hashable
```

`SectionIdentifierType`, `ItemIdentifierType` 두개의 generic parameter로 결정됩니다.

자료형을 보다시피 두 파라미터는 반드시 `Hashable` 해야 해요. `Hashalbe` 하기 때문에 apply 할때 비교해서 추가 또는 삭제된 부분을 알아차리도록 하는 것입니다.

## 2️⃣ 코드를 짜보자!

### 1. Connect a diffable data source to your collection view.

DiffableDataSource 는 프로토콜이 아니라 Generic Class 라고 했습니다.

```swift
@MainActor class UICollectionViewDiffableDataSource<SectionIdentifierType, ItemIdentifierType> : [NSObject](https://developer.apple.com/documentation/objectivec/nsobject) where SectionIdentifierType : [Hashable](https://developer.apple.com/documentation/swift/hashable), ItemIdentifierType : [Hashable](https://developer.apple.com/documentation/swift/hashable)
```

- 이니셜라이저를 통해서 만들어주고 전달하면 됩니다.

```swift
@MainActor init(collectionView: UICollectionView, cellProvider: @escaping UICollectionViewDiffableDataSource<SectionIdentifierType, ItemIdentifierType>.CellProvider)
```

<img width="600" alt="7" src="https://user-images.githubusercontent.com/69136340/155838781-786ab144-bfb9-45fd-b173-ec23f10aa7d8.png">

Generic type 부터 지정해보겠습니다. `SectionIdentifierType` 과 `ItemIdentifierType` 을 설정하면됩니다.

💫 **SectionIdentifierType**

*많은 예제를 보니 Int 혹은 CaseIterable 을 채택하는 enum 을 사용하더라구요.* 당연하게도 `Hashable` 를 채택해서 사용해도 좋습니다!

```swift
enum Section: CaseIterable {
    case main
}
```

<img width="600" alt="8" src="https://user-images.githubusercontent.com/69136340/155838802-65cd59d7-b22f-4db4-8109-dcf4f3cd7343.png">

- cellProvider는 3개의 파라미터를 제공합니다. `collectionView`, `IndexPath`, `ItemIdentifierType` 입니다.
- data source 코드를 다른곳에서도 사용해야하기 때문에 전역변수로 만들어서 관리해줍니다.

```swift
var dataSource: UICollectionViewDiffableDataSource<Section, String>!

//...

self.dataSource = UICollectionViewDiffableDataSource<Section, String>(collectionView: self.collectionView) { collectionView, indexPath, itemIdentifier -> UICollectionViewCell? in
    // code
}
```

### ❓  Hashable 이라고 했는데 CaseIterable 은 왜 가능한가요?

`CaseIterable` 에 대해서 잠깐 알아봐요!

왜 채택할까요? `allCases` 라는 타입속성을 얻기 위함인데요. 컴파일러가 프로토콜 구현을 자동으로 제공하기 때문에 채택 선언만 해주면 바로 사용할 수 있어요!

> *allCases의 타입은 프로토콜의 선언을 따라 enum 자신을 원소로 가지는 Collection 타입으로 제한됩니다.*

`allCases` 는 assoicated value 가 하나라도 있다면 자동으로 `allCases` 프로퍼티를 만들어줄 수 없어요.

> Associated Value의 값이 다른 경우를 같은 case로 취급해야 하는지, 다른 case로 취급해야하는지 컴파일러가 판단할 수 없기 때문입니다. 따라서 이 경우는 직접 allCases 프로퍼티를 제공해줘야 합니다.

`Hashable` 을 잠깐 살펴볼까요!

> `Hashable` 은 associated value 없이 enum 을 정의하면 자동으로 `Hashable` 을 준수합니다.

즉, associated value 없는 enum 은 자동으로 `Hashable` 을 준수하고, `CaseIterable` 을 채택해서 자동으로 `allCasese` 라는 타입속성을 얻었다는것은 associated value 가 없다라는 것이니까 **Hashable 을 준수하는 것이지요!**

그래서 `CaseIterable` 을 채택하는 예제도 성공적으로 구현이 가능한 것이랍니다!

- 참고:

[[Swift] CaseIterable 이란?](https://woozzang.tistory.com/98)

[Hashable](https://velog.io/@wonhee010/Hashable)

---

💫 **ItemIdentifierType**

보여줄 item 이 String 이라고 가정하고 넣어주겠습니다.

- 이때 같은 값이 중복으로 들어가게 되면 충돌이 발생합니다! 그래서 중복된 값을 처리해주어야 합니다.

### ❓중복된 값 처리

1️⃣ 

UUID 를 활용한 프로퍼티 정의

```swift
struct ItemName: Hashable {
    let id = UUID()
    var name: String
}
```

2️⃣ 

ItemIdentifierType 변경

```swift
// var dataSource: UICollectionViewDiffableDataSource<Section, String>!
// 변경.
var dataSource: UICollectionViewDiffableDataSource<Section, ItemName>!
```

---

### 2. Implement a cell provider to configure your collection view’s cells.

- cell 을 만들고, cell 에 데이터를 넣어준다.

```swift
// cell 이 해당 collection view 에 register 된 상태여야 한다.
collectionView.register(CollectionViewCell.self, forCellWithReuseIdentifier: "cell")

self.dataSource = UICollectionViewDiffableDataSource<Section, String>(collectionView: self.collectionView) { collectionView, indexPath, itemIdentifier -> UICollectionViewCell? in
    guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath) as? CollectionViewCell else { return UICollectionViewCell() }
    // ✅ cell 초기화.
    cell.initCell(itemIdentifier)
    return cell
}

collectionView.dataSource = dataSource
```

### 3. Generate the current state of the data.

- `Snapshot` 을 구성해보자.

```swift
// ✅ 텍스트에 따라서 검색 결과가 등장하는 로직.
func performQuery(with filter: String?) {
// ✅ 텍스트가 들어있는 arr 에서 filter 로 시작하는 텍스트들을 저장.
    let filtered = self.arr.filter { $0.hasPrefix(filter ?? "") }
// ✅ Snpapshot 생성.
    var snapshot = NSDiffableDataSourceSnapshot<Section, String>()
// ✅ section 및 item 추가.
    snapshot.appendSections([.main])
    snapshot.appendItems(filtered)
// ✅ apply 를 통해서 snapshot 과 animatingDifferences 파라미터 전달.
// ✅ 여기서 사용하기 위해서 dataSource 를 전역선언함.
    self.dataSource.apply(snapshot, animatingDifferences: true)
}
```

### ❗️apply snapshot without animation

위에서 사용한 `apply(_:animatingDifferences:completion:)` 메서드의 동작이 iOS 15 부터 변경되었다.

- `animatingDifferences` 파라미터가 `true` 이면 애니메이션이 적용되고, `false` 면 내부적으로 `reloadData` 로 바뀌어서 동작했었다.

iOS 15 부터

- `animatingDifferences` 파라미터가 `false` 를 넘기는 경우도 바뀐 부분만 적용하고 `reloadData` 로 바뀌어 동작하지 않는다고 한다.
- 그리고 [applySnapshotUsingReloadData(_:completion:)](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3804470-applysnapshotusingreloaddata) 메서드가 추가되어서 바뀐 부분만 업데이트하는 것이 아니라 reload 하고 싶으면 해당 메서드를 호출해주면 된다.

---

### 4. Display the data in the UI

위에서 작성한 `performQuery(with)` 메서드를 통해서 UI 이 갱신이 필요한 작업마다 `snapshot` 과 함께 `apply()` 를 호출해서 UI 를 갱신해주면 됩니다!

---

- 출처
    
    [Apple Developer Documentation](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3255138-init)
    
    [Diffable Data Source in iOS 15](https://brunch.co.kr/@eunjin3786/245)
    
    [Diffable Datasource](https://zeddios.tistory.com/1197)
    
    [[iOS - swift] 1. Diffable Data Source - UITableViewDiffableDataSource (테이블 뷰)](https://ios-development.tistory.com/717)
    
    [UI Diffable Data Source란?](https://velog.io/@ellyheetov/UI-Diffable-Data-Source)
    
    [[Swift] CaseIterable 이란?](https://woozzang.tistory.com/98)
