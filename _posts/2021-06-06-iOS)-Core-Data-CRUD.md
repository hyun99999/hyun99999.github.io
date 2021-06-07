---
title:  "iOS) Core Data CRUD"
categories:
- iOS

date:   2021-06-06 21:04:00 +0900
author_profile: false
---
# Core Data CRUD

`NSManagedObjectContext` 여러가지 transaction 메소드를 살펴보자

- manager class

AppDelegate.swift 에서 persistentContainer 를 계속 가져왔지만 매니저 클래스를 만들어서 사용하자

```swift
import Foundation
import CoreData

class PersistenceManager {
    static var shared: PersistenceManager = PersistenceManager()
    
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "Model")
        container.loadPersistentStores(completionHandler: { (storeDescription, error) in
            if let error = error as NSError? {
                fatalError("Unresolved error \(error), \(error.userInfo)")
            }
        })
        return container
    }()
    
    var context: NSManagedObjectContext {
        return self.persistentContainer.viewContext
    }
}
```

여기에 기능에 맞는 코드를 추가할 것이다.

그전에 `@discardableResult` 에 대해서 사용할 것이기 때문에 공부해보자.

- 문구 그대로 결과를 버릴 수 있는 의미

함수의 return 값을 discadable 시킬 수 있다. 즉, return 값을 사용하지 않아도 warning 메시지가 나오지 않는다.

insertStory(), deleteAll(), delete() 에서만 선언해준다. 

- fetch

```swift
//fetch
    func fetch<T :NSManagedObject>(reqeust: NSFetchRequest<T>) -> [T] {
        do {
            let fetchResult = try self.context.fetch(reqeust)
            return fetchResult
        } catch {
            print(error.localizedDescription)
            return []
        }
    }

// 사용
// let request: NSFetchRequest<StoryModel> = StoryModel.fetchRequest()
// let fetchResult = PersistenceManager.shared.fetch(reqeust: request)
```

- insert

```swift
//insert data
     @discardableResult
    func insertStory(story: Story) -> Bool {
        let entity = NSEntityDescription.entity(forEntityName: "StoryModel", in: self.context)
        if let entity = entity {
            let managedObject = NSManagedObject(entity: entity, insertInto: self.context)
            
            managedObject.setValue(story.title, forKey: "title")
            managedObject.setValue(story.detail, forKey: "detail")
            managedObject.setValue(story.date, forKey: "date")
            
            do {
                try self.context.save()
                return true
            } catch {
                print(error.localizedDescription)
                return false
            }
        } else {
            return false
        }
    }

// 사용
// let storyOne = Story(title: "story1", detail: "detail1", date: "date1")
// PersistenceManager.shared.insertStory(story: storyOne)
```

- delete

```swift
// delete
    @discardableResult
    func delete(object: NSManagedObject) -> Bool {
        self.context.delete(object)
        do {
            try context.save()
            return true
        } catch {
            return false
        }
    }

// 사용
// let request: NSFetchRequest<StoryModel> = StoryModel.fetchRequest()
// let fetchResult = PersistenceManager.shared.fetch(reqeust: request)
// PersistenceManager.shared.delete(object: fetchResult.last!)
```

- delete all

```swift
// delete all
    @discardableResult
    func deleteAll<T: NSManagedObject>(request: NSFetchRequest<T>) -> Bool {
        let request: NSFetchRequest<NSFetchRequestResult> = T.fetchRequest()
        let delete = NSBatchDeleteRequest(fetchRequest: request)
        do {
            try self.context.execute(delete)
            return true
        } catch {
            return false
        }
    }

// 사용
// let request: NSFetchRequest<StoryModel> = StoryModel.fetchRequest()
// PersistenceManager.shared.deleteAll(request: request)
// let arr = PersistenceManager.shared.fetch(request: request)
// if arr.isEmpty {
//    print("clean") // Print "clean"
// }
```

- count

```swift
// count
    func count<T: NSManagedObject>(request: NSFetchRequest<T>) -> Int? {
        do {
            let count = try self.context.count(for: request)
            return count
        } catch {
            return nil
        }
    }

// 사용
// let request: NSFetchRequest<StoryModel> = StoryModel.fetchRequest()
// print(PersistenceManager.shared.count(request: request))
```

---
