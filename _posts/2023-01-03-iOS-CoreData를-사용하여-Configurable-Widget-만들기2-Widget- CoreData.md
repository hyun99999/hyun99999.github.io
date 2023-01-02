---
title:  "iOS) CoreData 를 사용하여 Configurable Widget 만들기 (2/3) - Widget + CoreData"
categories:
- iOS

date:   2023-01-03  03:00:00 +0900
author_profile: false
---
1. 프로젝트 세팅
✅ **2. Widget 만들기**
- **공유한 데이터를 보여줄 widget 을 구현.**
- **다크모드를 적용.**
- **CoreData 데이터 공유.**
3. Configurable Widget 만들기

## 1️⃣ 위젯 UI 구현

- 우선, 아래와 같은 위젯의 뷰를 정적인 데이터를 채워넣어 구현해보겠습니다.(다크모드도 구현하겠습니다.)

<img width="400" alt="1" src="https://user-images.githubusercontent.com/69136340/210258802-36901ad2-d816-4a56-93e9-d949fcc511d6.png">

```swift
struct MyCardEnytryView : View {
    var entry: MyCardProvider.Entry
    // ✅ 다크모드를 판단하기 위한 enviornment 변수.
    @Environment(\.colorScheme) var colorScheme
    
    // TODO: - entry 변수를 사용하여 동적으로 컨텐츠 대응. 지금은 정적으로 대응.
    
    var body: some View {
        ZStack {
            Color.white
            GeometryReader { proxy in
                HStack(spacing: 0) {
                    Image("imgCardWidget")
                        .resizable()
                        // ✅ aspectRatio(width: 너비, height: 높이, contentMode: .fill) 코드에서 지정하는 너비와 높이는 contentMode 에 대한 것이다.
                        // (전체 frame 이 아님. 그래서 이미지의 크기를 위해서는 추후에 frame 에 대한 코드를 작성해주면 된다.)
                        // 현재 aspect ratio 를 그대로 사용하고 싶다면 contentMode 파라미터만 채워주면 된다.
                        .aspectRatio(contentMode: .fill)
                        // ✅ GeometryReader 의 GeometryProxy 사용해서 부모 컨테이너의 높이 값을 사용.
                        .frame(width: proxy.size.height * (92 / 152), height: proxy.size.height)

                     // 👉 다크모드를 다룰 때 다시 살펴보겠습니다.
                     // Color.backgroundColor(for: colorScheme)
                }
            }
            VStack {
                HStack {
                    Text("cardName")
                        .font(.system(size: 15))
                        .foregroundColor(.init(white: 1.0, opacity: 0.8))
                        .padding(EdgeInsets(top: 12, leading: 10, bottom: 0, trailing: 0))
                        .lineLimit(1)
                    Spacer()
                    Image("logoNada")
                        .padding(EdgeInsets(top: 10, leading: 0, bottom: 0, trailing: 10))
                }
                Spacer()
                HStack {
                    Spacer()
                    Text("userName")
                        .font(.system(size: 15))
                        // 👉 다크모드를 다룰 때 다시 살펴보겠습니다.
                        // .foregroundColor(.userNameColor(for: colorScheme))
                        .padding(EdgeInsets(top: 0, leading: 10, bottom: 11, trailing: 10))
                        .lineLimit(1)
                }
            }
        }
    }
}
```

## 2️⃣ 다크모드를 적용

- `colorScheme` 파라미터를 통해서 **light, dark appearances** 에 해당하는 **ColorScheme** 를 전달하고, switch 문에서 분기처리 해주었습니다.
- **ColorScheme** 가 **@frozen** 키워드를 사용하지 않기 때문에 언젠가 미래에 case 가 추가될 수 있습니다. 이때 default 로 분기처리하면 추가된 case 에 대해서 안내받을 수 없기 때문에 **@unknown** 키워드를 사용하여, 경고를 통해 알 수 있도록 하였습니다.

```swift
import SwiftUI

extension Color {
    static func backgroundColor(for colorScheme: ColorScheme) -> Color {
        switch colorScheme {
        case .light:
            return Color(red: 255.0 / 255.0, green: 255.0 / 255.0, blue: 255.0 / 255.0)
        case .dark:
            return Color(red: 19.0 / 255.0, green: 20.0 / 255.0, blue: 22.0 / 255.0, opacity: 0.5)
            // ✅ future version 에서 추가적으로 알려지지 않은 case 가 생겼을 때 사용하는 쪽에서 경고를 얻어볼 수 있는 편의 기능을 제공한다.
            // 이때 알려지지 않은 case 에 대한 대응을 할 수 있다.
        @unknown default:
            return Color(red: 255.0 / 255.0, green: 255.0 / 255.0, blue: 255.0 / 255.0)
        }
    }
    
    static func userNameColor(for colorScheme: ColorScheme) -> Color {
        switch colorScheme {
        case .light:
            return Color(red: 19.0 / 255.0, green: 20.0 / 255.0, blue: 22.0 / 255.0)
        case .dark:
            return Color(red: 255.0 / 255.0, green: 255.0 / 255.0, blue: 255.0 / 255.0)
        @unknown default:
            return Color(red: 19.0 / 255.0, green: 20.0 / 255.0, blue: 22.0 / 255.0)
        }
    }
}

// ✅ 위에서 사용한 코드를 다시 살펴보겠습니다.

Color.backgroundColor(for: colorScheme)

Text("userName")
    .foregroundColor(.userNameColor(for: colorScheme))

// ✅ 아래와 같이 삼항 연산자를 활용해서 사용할 수도 있습니다.
// .foregroundColor(colorScheme == .light ? Color(red: 19.0 / 255.0, green: 20.0 / 255.0, blue: 22.0 / 255.0) : Color(red: 255.0 / 255.0, green: 255.0 / 255.0, blue: 255.0 / 255.0))
```

## 3️⃣ 공유된 데이터를 사용해봅시다.

- 프로젝트를 만들 때 **CoreData** 를 사용하겠다고 체크하면 자동으로 **AppDelegate** 에 코드를 생성해줍니다.

```swift
// MARK: - Core Data stack

    lazy var persistentContainer: NSPersistentContainer = {
        /*
         The persistent container for the application. This implementation
         creates and returns a container, having loaded the store for the
         application to it. This property is optional since there are legitimate
         error conditions that could cause the creation of the store to fail.
        */
        let container = NSPersistentContainer(name: "WidgetsWithCoreDataTutorial_iOS")
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            if let error = error as NSError? {
                // Replace this implementation with code to handle the error appropriately.
                // fatalError() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.
                 
                /*
                 Typical reasons for an error here include:
                 * The parent directory does not exist, cannot be created, or disallows writing.
                 * The persistent store is not accessible, due to permissions or data protection when the device is locked.
                 * The device is out of space.
                 * The store could not be migrated to the current model version.
                 Check the error message to determine what the actual problem was.
                 */
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
        return container
    }()

    // MARK: - Core Data Saving support

    func saveContext () {
        let context = persistentContainer.viewContext
        if context.hasChanges {
            do {
                try context.save()
            } catch {
                // Replace this implementation with code to handle the error appropriately.
                // fatalError() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.
                let nserror = error as NSError
                fatalError("Unresolved error \(nserror), \(nserror.userInfo)")
            }
        }
    }
```

- containing app 에서 데이터를 저장하기 위해 CoreData entitiy 를 `CardDetail` 로 생성하겠습니다.
    - 이미지를 data 로 변환해서 `cardImage` attribute 로 저장하겠습니다.
    - widget extension 에서도 사용할 것이기 때문에 **target membership** 에서 체크해줍니다.

<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/210258839-18395d1b-eb5c-4090-ae49-5a7f541dc7a1.png">

### 👉 CoreData 저장 및 조회

- widget extension 과 데이터를 공유하기 위해서 **AppGroup** 을 사용하겠습니다. 사용자 정의 `CoreDataManager` 클래스를 추가하겠습니다.(이 역시 widget extension 에서도 사용할 것이기 때문에 **target membership** 에서 체크해줍니다.)
- 기본적인 저장 및 조회에 대한 기능을 구현하겠습니다.

```swift
// ✅ AppDelegate 에 있던 코드를 옮겨 CoreDataManager 라는 클래스를 구현하였습니다.
import CoreData
import UIKit

class CoreDataManager {
    // 👉 앱의 라이프사이클 동안 한번 생성하게되면 중복되지 않고 어디서든 동일한 인스턴스를 호출할 수 있도록 싱글톤 패턴을 적용.
    static let shared = CoreDataManager()
    private init() { }
    
    // ✅ AppGroup 을 활용하여 CoreData 로 저장한 데이터를 공유.
    private let appGroup = "group.WidgetsWithCoreDataTutorial-iOS"
    
    lazy var persistentContainer: NSPersistentContainer = {
        
        // ✅ App Group identifier 와 연결된 container directory 를 반환. 즉, 해당 group 의 공유 directory 의 파일 시스템 내 위치를 지정하는 NSURL 인스턴스를 반환.
        guard let url = FileManager.default.containerURL(forSecurityApplicationGroupIdentifier: appGroup) else { fatalError("Shared file container could not be created.") }
        
        let storeURL = url.appending(path: "WidgetsWithCoreDataTutorial_iOS.sqlite")
        
        // ✅ persistent store 를 생성 및 로드하는데 사용되는 description object.
        let storeDescription = NSPersistentStoreDescription(url: storeURL)

        let container = NSPersistentContainer(name: "WidgetsWithCoreDataTutorial_iOS")
        
        // ✅ 생성된 container 에서 사용하는 persistent store 의 타입을 재정의하려면 NSPersistentStoreDescription 배열로 설정할 수 있다.
        container.persistentStoreDescriptions = [storeDescription]
        
        // ✅ container 가 persistent store 를 로드하고, CoreData stack 을 생성 완료하도록 지시한다.
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            if let error = error as NSError? {
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
        return container
    }()
}

// MARK: - extensions
extension CoreDataManager {
    // ✅ scene 이 foreground 에서 background 로 전환될 때 호출되는 sceneDidEnterBackground(_ scene:) 메서드가 SceneDelegate 에서 구현되어있습니다.
    // 해당 메서드가 호출될 때, CoreData 의 변경사항을 저장하기 위해 saveContext() 메서드를 호출해야 합니다.(AppDelegate 에 있던 코드 그대로 사용하였습니다.)
    func saveContext () {
        let context = persistentContainer.viewContext
        if context.hasChanges {
            do {
                try context.save()
            } catch {
                let nserror = error as NSError
                fatalError("Unresolved error \(nserror), \(nserror.userInfo)")
            }
        }
    }
    
    // 👉 아래에서 살펴보겠습니다.
    func fetch(entityName: String) -> [NSManagedObject] {
        // ...
    }
}
```

- widget extension 에서 데이터를 조회할 수 있도록 메서드를 구현하겠습니다.

```swift
func fetch(entityName: String) -> [NSManagedObject] {
        let viewContext = persistentContainer.viewContext

        // ✅ entityName 파라미터에 따라서 해당 entity 를 조회하는 request 생성.
        let fetchReqeust = NSFetchRequest<NSManagedObject>(entityName: entityName)
        
        do {
            // ✅ 조회.
            let fetchResult = try viewContext.fetch(fetchReqeust)
            
            return fetchResult
        } catch {
            let nserror = error as NSError
            fatalError("Unresolved error \(nserror), \(nserror.userInfo)")
        }
    }
```

### 👉 적용해보자.

- `ViewController.swift` 에서 다음과 같이 CoreData 를 활용하여 데이터를 저장하는 메서드를 구현하였습니다.

```swift
import CoreData
import UIKit

class ViewController: UIViewController {

    // MARK: - Components
    @IBOutlet weak var backgroundImageView: UIImageView!
    @IBOutlet weak var cardNameLabel: UILabel!
    @IBOutlet weak var nameLabel: UILabel!
    
    // MARK: - View Life Cycle
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 여러 데이터를 저장하기 위해서 viewDidLoad 에서 다음과 같이 함수를 호출하였습니다.
        save(cardNameLabel.text, userName: nameLabel.text, image: backgroundImageView.image ?? UIImage())
        save("두 번째 카드", userName: "두현규", image: UIImage(named: "imgCardWidget") ?? UIImage())
    }
}

// MARK: - Extension
extension ViewController {
    /* 저장
    1. NSManagedObjectContext를 가져온다.
    2. 저장할 Entity 가져온다.
    3. NSManagedObject 생성한다.
    4. NSManagedObjectContext 저장해준다.
    */
    func save(_ cardName: String?, userName: String?, image: UIImage) {
        // ✅ CoreDataManager 의 persistentContainer 를 가져와서 entity 생성.
        let viewContext = CoreDataManager.shared.persistentContainer.viewContext
        let entity = NSEntityDescription.entity(forEntityName: "CardDetail", in: viewContext)
        
        // ✅ NSManagedObject 생성.
        if let entity {
            let card = NSManagedObject(entity: entity, insertInto: viewContext)
            card.setValue(cardName, forKey: "cardName")
            card.setValue(userName, forKey: "userName")
            if let imageData = image.pngData() {
                card.setValue(imageData, forKey: "cardImage")
            }
            
            // ✅ NSManagedObjectContext 저장.
            do {
                try viewContext.save()
            } catch {
                let nserror = error as NSError
                fatalError("Unresolved error \(nserror), \(nserror.userInfo)")
            }
        }
    }
}
```

- widget 에서 CardDetail entity 를 조회하고 정적으로 반영하겠습니다.(Provider 를 사용하는 동적인 작업은 다음 글에서 진행하겠습니다. 주석처리한 코드를 사용해서 동적으로 반영할 수 있습니다.)

```swift
struct MyCardEntry: TimelineEntry {
    let date: Date
    let detail: MyCardDetail
}

struct MyCardEnytryView : View {
    var entry: MyCardProvider.Entry

    // ...
    
    // ✅ CoreData 로 저장한 데이터 조회.
    let cardDetail = CoreDataManager.shared.fetch(entityName: "CardDetail")
    
    var body: some View {

        // ...

        // ✅ cardDetail 배열에서 두 번째 요소를 사용하여 정적으로 위젯에 반영해보겠습니다.
        // value(forKey:) - Returns the value for the property specified by key.
        let cardImage = UIImage(data: cardDetail[1].value(forKey: "cardImage") as? Data ?? Data()) ?? UIImage()
        Image(uiImage: cardImage)
    //  Image(uiImage: entry.detail.cardImage)

        // ...

        let cardName = cardDetail[1].value(forKey: "cardName") as? String ?? ""
        Text(cardName)
    //  Text(entry.detail.cardName)

        // ...

        let userName = cardDetail[1].value(forKey: "userName") as? String ?? ""
        Text(userName)
    //  Text(entry.detail.userName)
    }
}
```

### 👉 결과

<img src="https://user-images.githubusercontent.com/69136340/210258861-610eeaad-4755-40e5-9579-0073f941f14b.png"
 width ="250">

***다음 글에서는 이렇게 불러올 수 있는 데이터를 사용해서 위젯 편집할 때 선택 목록을 구성해보겠습니다.***

## 👉 GitHub

https://github.com/hyun99999/WidgetsWithCoreDataTutorial-iOS

**참고:** 

[sharing data with a widget](https://useyourloaf.com/blog/sharing-data-with-a-widget/)

[Sharing Core Data Storage with App Extensions | eyupgoymen](https://www.eyupgoymen.com/articles/sharing-core-data-storage-with-extensions/)
