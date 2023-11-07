---
title:  "iOS) DiffableDataSource 사용해서 collection view 를 업데이트해보자(개발자 문서)"
categories:
- iOS

date:   2023-10-17  12:53:00 +0900
author_profile: false
---
개발자 문서를 참고하여 단계별로 알아보고 진행하여 보겠습니다.

[Updating Collection Views Using Diffable Data Sources | Apple Developer Documentation](https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/updating_collection_views_using_diffable_data_sources)

### Overview

주로 UICollectionViewDataSource 를 채택하여 컬렉션 뷰를 채웁니다. 복잡한 데이터 추가, 삭제 및 이동 핸들링 과정을 피하기 위해서 UICollectionViewDiffalbeDataSource 객체를 사용할 수 있습니다.

diffable data source 가 저장하는 section 과 item identifiers 는 변하지 않고 안정적인 identifiers 입니다. 이는 UICollectionViewDataSource 의 안정적이지 않은 indices 와 index path 와 대비됩니다.

identifiers 를 가진 diffable data source 는 컬렉션 뷰 내의 위치(indices, index path)에 대한 지식 없이 section 과 item 을 참조할 수 있습니다.

identifier 를 사용하기 위해서 데이터 타입은 반드시 Hashable 프로토콜을 채택해야만 합니다. Hashing 을 사용해서 Set, Dictionary 및 snapshots 같은 데이터 컬렉션에서 값을 Key 를 사용하여 빠르고 효율적으로 제공할 수 있습니다.

identifiers 는 hashalbe 하고 equatable 하기 때문에 현재 스냅샷과 다른 스냅샷 간의 차이점을 확인할 수 있습니다.

**Important**

효율성을 높이기 위해서 해시 충돌이 발생하지 않도록 요구합니다. 가끔 일어나는 불가피한 충돌은 괜찮지만 성능 저하를 우려하여 최소한으로 유지하도록 요구합니다.

### 1️⃣ Define the Diffable Data Source

목록을 표시하기 전에 diffable data source 를 저장하기 위해서 인스턴스 변수를 정의합니다.

```swift
private var recipeListDataSource: UICollectionViewDiffableDataSource<RecipeListSection, Recipe.ID>!
// Identifier type 으로 RecipeListSection 을 item identifier type 으로 Recipe.ID 를 사용.
```

- 다음과 같이 단 하나의 섹션을 가지는 enum 타입을 정의하였습니다.

```swift
private enum RecipeListSection: Int {
    case main
}
```

- Recipe 구조체는 Identifiable 프로토콜을 채택하여 id 값을 포함하고 있습니다. [Recipe.ID](http://recipe.id/) 는 Hashable 하기 때문에 identifier type 으로 설정될 수 있습니다.

```swift
struct Recipe: Identifiable, Codable {
    var id: Int
    var title: String
    var prepTime: Int   // In seconds.
    var cookTime: Int   // In seconds.
    var servings: String
    var ingredients: String
    var directions: String
    var isFavorite: Bool
    var collections: [String]
    fileprivate var addedOn: Date? = Date()
    fileprivate var imageNames: [String]
}
```

모든 스냅샷에 Recipe 전체 데이터가 아닌 [Recipe.ID](http://recipe.id/) 값만 포함되었습니다. 이러한 접근 방식은 컬렉션 뷰에 레시피를 표시할 때 최대 성능을 낼 수 있도록 diffable data source 를 최적화할 수 있습니다.

### 2️⃣ Configure the Diffable Data Source

UICollectionViewDiffableDataSource 인스턴스를 만들고 cell provider 를 설정합니다. 이는 컬렉션 뷰를 윈한 cell 을 구성하고 반환하는 클로저입니다.

다음은 CellRegistration 을 만들어서 진행한 예시 코드입니다.

```swift
private func configureDataSource() {
    // Create a cell registration that the diffable data source will use.
    let recipeCellRegistration = UICollectionView.CellRegistration<UICollectionViewListCell, Recipe> { cell, indexPath, recipe in
        var contentConfiguration = UIListContentConfiguration.subtitleCell()
        // ...
        cell.contentConfiguration = contentConfiguration
        // ...
    }

    // Create the diffable data source and its cell provider.
    recipeListDataSource = UICollectionViewDiffableDataSource(collectionView: collectionView) { collectionView, indexPath, identifier -> UICollectionViewCell in
        // `identifier` is an instance of `Recipe.ID`. Use it to
        // retrieve the recipe from the backing data store.
        let recipe = dataStore.recipe(with: identifier)!
        return collectionView.dequeueConfiguredReusableCell(using: recipeCellRegistration, for: indexPath, item: recipe)
    }
}
```

- UICollectionViewCell 을 만들어서 진행하는 경우, 저는 아래와 같이 진행하였습니다.

```swift
// viewController.swift

    private var diffableDataSource: UICollectionViewDiffableDataSource<Section, ReceivedTag>?   

    // ...

    private func configureDataSource() {
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
```

### 3️⃣ Load the Diffable Data Source with Identifiers

data source 에 데이터의 초기 로드를 수행하여 컬렉션 뷰를 채웁니다.

해당 메소드는 [NSDiffableDataSourceSnapshot](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot) 인스턴스를 생성합니다. [applySnapshotUsingReloadData(_:)](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3804469-applysnapshotusingreloaddata) 메소드를 호출하여 snapshot 을 data source 에 적용하고 차이점을 계산하거나 변경사항에 애니메이션 없이 컬렉션 뷰를 재설정 합니다.

```swift
private func loadRecipeData() {
    // Retrieve the list of recipe identifiers determined based on a
    // selected sidebar item such as All Recipes or Favorites.
    guard let recipeIds = recipeSplitViewController.selectedRecipes?.recipeIds()
    else { return }
    
    // Update the collection view by adding the recipe identifiers to
    // a new snapshot, and apply the snapshot to the diffable data source.
    var snapshot = NSDiffableDataSourceSnapshot<RecipeListSection, Recipe.ID>()
    snapshot.appendSections([.main])
    snapshot.appendItems(recipeIds, toSection: .main)
    recipeListDataSource.applySnapshotUsingReloadData(snapshot)
}
```

### 4️⃣ Insert, Delete, and Move Items

이전에는 데이터의 초기 로드에 대해 살펴봤다면 이번에는 데이터가 더해지거나 삭제되거나 순서를 변경에 대해 알아보겠습니다.

data collection 에 대한 변경 사항을 처리하기 위해 앱은 현재 상태를 나타내는 새 snapshot 을 생성하고 diffable data source 에 적용합니다. 이를 현재 snapshot 과 변경사항을 비교해서 필요한 삽입, 삭제, 이동을 수행하게 됩니다.

data collection 의 변경을 모니터링하지 않지만, 새로운 snapshot 을 적용하여 data 의 변화를 추적하는 것은 앱의 몫입니다. 이때, 데이터의 변경사항을 NotificationCenter 혹은 Combine 을 사용해서 앱에 알릴 수 있습니다.

[apply(_:animatingDifferences:)](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3375795-apply) 메소드는 표시되는 데이터를 완전히 재설정하는 것이 아닌 incremental updates 를 수행합니다. 그리고 animatingDifferences 를 true 로 설정하여 애니메이션을 적용할 수 있습니다.

```swift
var snapshot = NSDiffableDataSourceSnapshot<RecipeListSection, Recipe.ID>()
snapshot.appendSections([.main])
snapshot.appendItems(selectedRecipeIds, toSection: .main)
recipeListDataSource.apply(snapshot, animatingDifferences: true)
```

### 5️⃣ Update Existing Items

기존 아이템의 프로퍼티를 처리하기 위해 앱은 diffable data source 에서 현재 snapshot 을 가져와 [reconfigureItems(_:)](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot/3804468-reconfigureitems) 또는 [reloadItems(_:)](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot/3375783-reloaditems) 을 호출합니다.

이때에도 역시나 diffable data source 가 추적하는 것이 아닌 앱이 수행합니다.

예시 코드에서는 단일 레시피의 데이터 변경을 구현하고 있습니다. 하나의 레시피만 변경되었으므로 컬렉션 뷰의 전체 레시피 목록을 업데이트할 필요가 없습니다. 변경된 레시피를 표시하는 셀만 업데이트합니다.

아래 예시 코드는 notification 이 제공하는 레시피 ID 를 사용해 reconfigureItems(_:) 메소드를 통해 특정 레시피를 업데이트 합니다.

```swift
@objc
private func recipeDidChange(_ notification: Notification) {
    guard
        // Get `recipeId` from from the `userInfo` dictionary.
        let userInfo = notification.userInfo,
        let recipeId = userInfo[NotificationKeys.recipeId] as? Recipe.ID,
        // Confirm that the data source contains the recipe.
        recipeListDataSource.indexPath(for: recipeId) != nil
    else { return }
    
    // Get the diffable data source's current snapshot.
    var snapshot = recipeListDataSource.snapshot()
    // Update the recipe's data displayed in the collection view.
    snapshot.reconfigureItems([recipeId])
    recipeListDataSource.apply(snapshot, animatingDifferences: true)
}
```

**reloadItems(_:) vs reconfigureItems(_:)**

reconfigureItems(_:) 메소드는 기존 셀을 재구성하기 때문에 prepareForReuse 를 호출하지 않습니다.

existing  cell 을 새로운 cell 로 바꾸지 않고 내용을 업데이트하려면 reladItems(_:) 대신 reconfigureItems(_:) 메소드를 사용합니다. 즉, 성능을 위해서 새로운 cell 로 교체(다른 타입의 셀 반환)해야하는 필요가 없다면 reload 하는 대신 reconfigure 를 선택할 수 있습니다.

### 6️⃣ **Populate Snapshots with Lightweight Data Structures**

identifiers 를 저장하는 또 다른 접근 방식에는 diffable data source 와 snapshot 을 lightweight 의 데이터 구조체들로 populate(채우는) 것도 있습니다.

이러한 접근 방식은 빠른 프로토타이핑이나 정적인 프로퍼티를 가진 items 의 경우 편리하고 적합할 수 있지만, 장단점을 가집니다. 예를 들어, 변할 수 있는 구조체의 모든 프로퍼티가 Hashable, Equatable 이 구현되어야 합니다. 

다음의 예시 코드는 lightweight 데이터 구조체를 사용하여 사이드바를 정의합니다.

```swift
private struct SidebarItem: Hashable {
    let title: String
    let type: SidebarItemType
    
    enum SidebarItemType {
        case standard, collection, expandableHeader
    }
}
```

프로퍼티의 값이 변경되지 않으므로 SidebarItem 구조체로 snapshot 을 채우는 것이 가능한 사례입니다.

```swift
private func createSnapshotOfStandardItems() -> NSDiffableDataSourceSectionSnapshot<SidebarItem> {
    let items = [
        SidebarItem(title: StandardSidebarItem.all.rawValue, type: .standard),
        SidebarItem(title: StandardSidebarItem.favorites.rawValue, type: .standard),
        SidebarItem(title: StandardSidebarItem.recents.rawValue, type: .standard)
    ]
    return createSidebarItemSnapshot(.standardItems, items: items)
}

// StandardSidebarItem
private enum StandardSidebarItem: String, CaseIterable {
    case all = "All Recipes"
    case favorites = "Favorites"
    case recents = "Recents"
}
```

<img width="400" alt="스크린샷 2023-10-17 오전 10 57 19" src="https://github.com/hyun99999/algorithm-Swift/assets/69136340/c3da4792-9f47-4f16-bcc9-46ac762ba7d4">

이러한 접근 방식의 단점은 diffable data source 가 더이상 ID 를 추적할 수 없는 것입니다. existing item 이 변경될 때 diffable data source 는 이전 항목을 삭제하고 새 항목 추가로 간주합니다. 결과적으로 컬렉션 뷰는 item 과 관련된 중요한 상태를 잃어버립니다. 예를 들어, diffable data source 관점에서 앱이 해당 item 를 삭제하고 대신 새 item 을 추가하기 때문에 선택한 item 이 선택 취소됩니다.

또한, animatingDifferences 가 true 인 경우에 snapshot 을 apply 하면 모든 변화는 애니메이션 과정을 필요로 합니다. 이는 성능에 해롭고, 셀 내에서 애니메이션을 포함한 UI상태가 손실될 수 있습니다.

추가적으로, identifiers 를 사용해야 하기 때문에 reconfigureItems(_:) 또는 reloadItems(_:) 메서드를 사용하는 것이 불가능합니다.

existing items 을 업데이트하는 유일한 메커니즘은 새로운 데이터 모델을 포함하는 새로운 snapshot 을 적용하는 것입니다.

items 변경되지 않거나 item 의 identifier 가 중요하지 않은 경우에만 해당 접근 방식을 사용하세요.
