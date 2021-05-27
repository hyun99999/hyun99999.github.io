---
title:  "iOS) Core Data"
categories:
- iOS

date:   2021-05-27 23:04:00 +0900
author_profile: false
---
# Core Data 에 대해서

Core Data 는 iOS 기본 프레임워크이다.

UserDefaults 는 app setting 같은 간단한 정보를 저장하기에 적합하다면

Core Data 는 복잡하고 큰 user data 를 저장하기에 적합하다. 예를 들어 내 아이폰 내에서의 검색 기록과 같이 시간과 키워드에 해당하는 정보를 저장하기에 UserDefaults 보다 적합할 수 있다.

'**Core Data 는 Database 인가?'**

아니다. 데이터를 유지하기 위한 API 도 아니다. 객체 그래프를 관리하는 Framework 이다. 

이 객체 그래프를 디스크에 저장하여 Persistence 기능을 이용할 수 있는 것이다.

보통 영구적으로 저장하기 위해서 Core Data 를 사용하는데 Persistence 는 Core Data 의 기능 중 하나일뿐이다. 그렇기 때문에 Persistence  기능을 사용하지 않고도 Core Data 를 이용할 수 있다.

---

Core Data 가 속한 Core Service 계층과 다른 계층을  iOS Architecture 를 알아보자

<img src ="https://user-images.githubusercontent.com/69136340/119841596-e96d5200-bf40-11eb-9dad-ebbee93ae111.png" width ="200">
   
**Cocoa Touch**

- 화면의 그래픽 UI 및 터치의 기능을 제공하는 Framework
- UIKit, MapKit, GameKit, MessageUI, MessageUI, AddressBookUI

**Media**

- 그래픽이나 오디오, 비디오 등 멀티미디어의 기능을 제공하는 Framework
- AvFoundation, MediaPlayer, Core Animation, Core Graphics

**Core Service**

- GPS 나침반, 가속도 센서나 자이로스코프 센서와 같이 디바이스의 하드웨어 특성에 기반한 서비스 제공
- Core Location, Core Data, Core Motion, Foundation

**Core OS**

- 하드웨어와 가장 가까이 있는 최하위 계층
- C 기반의 저수준의 API로 이루어져 있음.
- 커널, 파일 시스템, 전원 관리, 디바이스 드라이버 등이 포함

---

# Core Data

단일장치에 대한 캐시데이터 유지 혹은 CloudKit 을 사용해서 여러장치에 데이터 동기화

---

### Overview

Core Data 를 사용해서 데이터의 오프라인 사용, 임시데이터 캐시, 단일기기에서 undo functionality 추가한다.

단일 iCloud 계정에서 여러장치에서 데이터를 동기화힉 위해서 Core Data 는 자동으로 스키마를 Cloud kit container 에 미러링한다.

Core Data 의 Data Model editor 를 통해서 데이터의 types, relationships 와 각 class 의 정의를 생성합니다. Core Data 는 런타임때 객체 인스턴스를 관리해서 다음과 같은 기능을 제공할 수 있다.

### Persistence

Core Data 는 객체를 저장소에 매핑하는 세부정보를 추상화해서 데이터베이스를 직접 관리하지 않고도 Swift 와 Objective-C 의 데이터를 쉽게 저장할 수 있다.

<img src ="https://user-images.githubusercontent.com/69136340/119841701-fee27c00-bf40-11eb-9c39-46b341c6caf2.png" width = "200">

### Undo and Redo of Individual or Batched Changes

Core Data 의 undo manager tracks 는 변경 사항을 추적해서 개별적, 그룹, 한번에 모두 롤백할 수 있어서 쉽게 undo(실행취소), redo(다시 실행)지원을 앱에 추가할 수 있다.

<img src ="https://user-images.githubusercontent.com/69136340/119841898-2c2f2a00-bf41-11eb-8b4d-7931ec59437f.png" width = "800">

### Background Data Tasks

background 에서 JSON 을 객체로 파싱하는 것과 같이 잠재적으로 UI-blocking data tasks 를 수행한다. 그런다음 결과를 캐시하거나 저장하여 서버 왕복을 줄일 수 있다.

<img src ="https://user-images.githubusercontent.com/69136340/119841968-3a7d4600-bf41-11eb-8573-58335ecccc7e.png" width = "200">

### View Synchronization

Core Data 는 table 과 collection views 의 data sources 를 제공함으로써 뷰와 데이터를 동기화 상태로 유지하는것 역시 도움이 된다.

### Versioning and Migration

Core Data 는 data model 의 versioning 과 앱이 발전함에 따라 user data 를 migrating 한다.

*(Core Data 는 현재 data model 을 가장 최신으로 유지하는데 도움이되는  built-in data migration tool 이 있다.)*

**Core Data**
[Apple Developer Documentation](https://developer.apple.com/documentation/coredata)

---

# Core Data Stack

app's model layer 관리, 유지

---

### Overview

Core Data 는 app's model layer 을 공동으로 지원하는 클래스 집합을 제공한다.

- `NSManagedObjectModel` 의 인스턴스는 properties 와 relationships 를 포함하여 app's types 를 설명한다.
- `NSManagedObjectContext` 의 인스턴스는 app's types 의 인스턴스에 대한 변화를 추적한다. 이 객체를 통해서 create, delete, fetch, update 작업을 할 수 있다.

    (core data는 메모리에 로드된 상태로 처리되는데, 이 때의 메모리가 context 를 의미)

- `NSPersistentStoreCoordinator` 의 인스턴스는 store 에서 app's types 의 인스턴스를 저장하고 가져온다.

    (Context 와 Container 사이를 통신한다.)

<img src ="https://user-images.githubusercontent.com/69136340/119842331-8c25d080-bf41-11eb-979a-5e0e61066f9c.png" width = "800">

출처 : [https://developer.apple.com/documentation/coredata/setting_up_a_core_data_stack](https://developer.apple.com/documentation/coredata/setting_up_a_core_data_stack)

![https://www.vadimbulavin.com/assets/images/posts/core_data_stack_1.svg](https://www.vadimbulavin.com/assets/images/posts/core_data_stack_1.svg)

출처 [https://www.vadimbulavin.com/core-data-stack-swift-4/](https://www.vadimbulavin.com/core-data-stack-swift-4/)

`NSPersistentContainer` 인스턴스를 사용해서 model, context, store coordinator 를 동시에 설정한다.

(전체적으로 관리하는 최상단의 객체)

**Core Data Stack**
[Apple Developer Documentation](https://developer.apple.com/documentation/coredata)

---

# **Setting Up a Core Data Stack**

### Initialize a Persistent Container

일반적으로 app's startup 동안 Core Data 가 초기화된다. persistent container 를 `lazy variable` 로 만들어서 delegate 에서 처음 사용할때까지 연기시킨다. 

새 Xcode 프로젝트를 생성할 때 Core Data checkbox 를 선택했다면 `AppDelegate` 에 setup code 가 자동으로 포함되어있을 것이다.

1. `NSPersistentContainer` 를 lazy variable 로 선언.
2. data model filename 를 이니셜라이저에 전달해서 persistent container instance 를 생성
3. persistent stores 를 로드한다. 존재하지 않을 경우 store 를 만든다.

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {

    ...

    lazy var persistentContainer: NSPersistentContainer = {        
        let container = NSPersistentContainer(name: "DataModel")
        container.loadPersistentStores { description, error in
            if let error = error {
                fatalError("Unable to load persistent stores: \(error)")
            }
        }
        return container
    }()

    ...
}
```

생성되면 persistent container 는 해당 `managedObjectModel`, `viewContext`, `persistentStoreCoordinator` 속성에 각각  model, context, store coordinator 인스턴스에 대한 참조를 가진다.

<img src ="https://user-images.githubusercontent.com/69136340/119842512-b4adca80-bf41-11eb-92b7-7b2cf9f0e237.png" width = "800">

NSPersistentContainer 정의.

container 에 대한 참조를 user interface 에 전달할 수 있다.

### Pass a Persistent Container Reference to a View Controller

app's root view controller 에서 Core Data 를 import 하고 persistent container 참조하는 변수를 만들어라

```swift
import UIKit
import CoreData

class ViewController: UIViewController {

    var container: NSPersistentContainer!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        guard container != nil else {
            fatalError("This view needs a persistent container.")
        }
        // The persistent container is available.
    }
}
```

delegate 로 돌아간다. `application(_:didFinishLaunchingWithOptions:)` 에서 app window's rootViewController 을 root view controller type 으로 downcast 한다.

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {

    ...

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {        
        if let rootVC = window?.rootViewController as? ViewController {
            rootVC.container = persistentContainer
        }        
        return true
    }

    ...
}
```

persistent container 를 추가적인 view controllers 에 전달하려면 각 뷰컨에서 container variable 생성을 반복하고 이전 뷰컨의 `prepare(for:sender:)` 에서 해당 값을 설정한다.

```swift
class ViewController: UIViewController {

    ...
    
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if let nextVC = segue.destination as? NextViewController {
            nextVC.container = container
        }
    }
}
```

### Subclass the Persistent Container

`NSPersistentContainer` 는 subclass 로 지정된다. subclass 는 subset of data 와 data 를 디스크에 유지하기위한 호출과 같은 Core Data 관련 코드를 넣을 수 있는 장소다.

```swift
import CoreData

class PersistentContainer: NSPersistentContainer {    

    func saveContext(backgroundContext: NSManagedObjectContext? = nil) {
        let context = backgroundContext ?? viewContext
        guard context.hasChanges else { return }
        do {
            try context.save()
        } catch let error as NSError {
            print("Error: \(error), \(error.userInfo)")
        }
    }    
}
```

위의 코드는 변경사항이 있을때만 context 에 저장해서 성능을 향상시키기 위한 `saveContext` 함수를 추가하는 예시이다.

**Setting Up a Core Data Stack**
[Apple Developer Documentation](https://developer.apple.com/documentation/coredata/setting_up_a_core_data_stack)

---

# **Creating a Core Model**

**Creating a Core Model**
[Apple Developer Documentation](https://developer.apple.com/documentation/coredata/creating_a_core_data_model)
