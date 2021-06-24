---
title:  "iOS) Core Data custom class"
categories:
- iOS

date:   2021-06-17 11:36:00 +0900
author_profile: false
---
Core Data custom class 를 설정해보고 relationship 을 활용해서 관계형 데이터베이스화 해보자.

미리 밝힌다. 관계형 데이터베이스를 이용해서 구성하는 것이 올바른 설계지만 커스텀 클래스를 써보고자 했기 때문에 사용해보았다. 결국 릴레이션쉽을 설정해주었다.

# Core Data custom class

### 상황

각 이야기에 해당하는 글이 존재하고 pageIndex 에 따라서 뷰컨에 이야기와 해당 글을 뿌려주고 싶었다...

<img width="1000" src="https://user-images.githubusercontent.com/69136340/122339866-89386180-cf7c-11eb-90d2-b77f426c2641.png">

<img width="1000" src="https://user-images.githubusercontent.com/69136340/122339858-863d7100-cf7c-11eb-8087-dfbe14c72b11.png">

위와 같이 entity 를 두개를 만들고 `StroyList` 의 `story` attribute 에 custom class 로 `StoryModel` 배열을 넣어주었다. 

- StoryModel : 타이틀, 디테일, 날짜 정보를 가지는 "이야기" 하나에 들어가는 "글" 이다.
- StoryList : StoryModel 의 배열(=글 목록)과 "이야기" 의 제목과 소제목 정보를 가짐.

(PenCake 는 이야기에 글들을 등록할 수 있는 앱)

사실 진짜 Core Data 시작과 지금 진행이 이게 돼..?? 의 연속이기때문에 그냥 간과하고 프로젝트를 진행했고 이야기 한개에 글들을 등록하는 `StroyModel` 만 사용했던 체제에서 여러 스토리를 다루는 `StoryList` 를 적용하니까 다음과 같은 오류가 나왔다.

### error

<img width="800" alt="스크린샷 2021-06-11 오후 3 10 10" src="https://user-images.githubusercontent.com/69136340/122339983-ad943e00-cf7c-11eb-952c-beedb30cb802.png">

iOS 13부터 Apple이 사용자 지정 Core Data 데이터 유형에 대해 NSSecureCoding (NSCoding 대신)을 사용하여 보안을 강화하기를 원했고 내가 설정한 custom class 의 자료형은 NSSecureCoding 이 아니기 때문이다.

그래서 `StoryModel` 역할을 해줄 커스텀 클래스를  NSSecureCoding 을 준수하도록 만들어서 적용시키기로 했다. 그리고 더이상 StroyModel 은 그 자체만으로써 저장하지 않고 StoryList 에서 저장을 하기 때문에 삭제하기로 했다.

→ `Writings` 라는 배열 `Writing` 를 가지고 있는 커스텀 클래스를 만들어주기로 했다.

## 해결

### 첫번째, custom class 에 `NSSecureCoding` 채택.

<img width="1000" alt="스크린샷 2021-06-11 오후 1 22 54" src="https://user-images.githubusercontent.com/69136340/122339987-af5e0180-cf7c-11eb-8de1-8b69e7408388.png">

Writing.swift

```swift
//
//  Writing.swift
//  PenCake-iOS-CloneCoding
//
//  Created by kimhyungyu on 2021/06/11.
//

import Foundation
import UIKit
//1
public class Writing: NSObject, NSSecureCoding {
    public static var supportsSecureCoding = true
    
    var title: String
    var detail: String
    var date: Date
    
    enum Key: String {
        case title = "title"
        case detail = "detail"
        case date = "date"
    }
    
    init(title: String, detail: String, date: Date) {
        self.title = title
        self.detail = detail
        self.date = date
    }
    //2
    public func encode(with coder: NSCoder) {
        coder.encode(title, forKey: Key.title.rawValue)
        coder.encode(detail, forKey: Key.detail.rawValue)
        coder.encode(date, forKey: Key.date.rawValue)
    }
    
    public convenience required init?(coder: NSCoder) {
        //3
        let mTitle = coder.decodeObject(forKey: Key.title.rawValue)
        let mDetail = coder.decodeObject(forKey: Key.detail.rawValue)
        let mDate = coder.decodeObject(forKey: Key.date.rawValue)
        
        self.init(title: mTitle as! String, detail: mDetail as! String, date: mDate as! Date)
    }

}
```

**주석**

1: NSSecureCoding 프로토콜 채택

2: 해당 자료형이 NSSecureCoding 프로토콜을 따르지 않기 때문에 NSString으로 변환 한 다음 인코딩한다.

3: Core Data에서 데이터를 검색 할 때 decodeObject (forKey :)를 사용하여 객체를 디코딩한다.

(참고: String,Date 을 사용하는 멤버들이라서 NSSecureCoding, encode 도 하지 않아도 될거같다. 왜냐면 Core Data 에서 String, Date 은 지원하는 자료형이 아닌가! 하지만 혹시 다른 플젝에서 쓸 때는 얘기가 달라지니까 같이 소개한다.)

Writings

```swift
//
//  Writings.swift
//  PenCake-iOS-CloneCoding
//
//  Created by kimhyungyu on 2021/06/11.
//

import Foundation
public class Writings: NSObject, NSSecureCoding {
    public static var supportsSecureCoding = true
    
    public var writings: [Writing] = []
    
    enum Key: String {
        case writings = "writings"
    }
    
    init(writings: [Writing]) {
        self.writings = writings
    }
    
    public func encode(with coder: NSCoder) {
        coder.encode(writings, forKey: Key.writings.rawValue)
    }
    
    public convenience required init?(coder: NSCoder) {
        let mWritings = coder.decodeObject(forKey: Key.writings.rawValue)
        
        self.init(writings: mWritings as! [Writing])
    }
}
```

### 두번째, `StoryList` 의 `story` attribute 의 `NSSecureUnarchiveFromDataTransformer` subclass 만들기

```swift
//
//  StoryAttributeTransformer.swift
//  PenCake-iOS-CloneCoding
//
//  Created by kimhyungyu on 2021/06/11.
//

import Foundation

class StoryAttributeTransformer: NSSecureUnarchiveFromDataTransformer {
        //1
    override static var allowedTopLevelClasses: [AnyClass] {
        [Writings.self]
    }
    //2
    static func register() {
        let className = String(describing: StoryAttributeTransformer.self)
        let name = NSValueTransformerName(className)
        let transformer = StoryAttributeTransformer()
        
        ValueTransformer.setValueTransformer(transformer, forName: name)
    }
}
```

- 위의 클래스는 NSSecureUnarchiveFromDataTransformer 하위 클래스로 Core Data에서 검색 할 때 NSData의 값을 올바른 데이터 유형으로 변환하는 방법을 알고 있습니다.

**주석**

1: 여기서는 값 변환 프로세스에서 최상위 클래스이므로 allowedTopLevelClasses 배열에 Writings 클래스를 추가합니다.

2: custom transformer 를 등록하기 위한 메서드.

### 세번째, `AppDelegate.swift` 에 custom transformer 를 등록

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        
                StoryAttributeTransformer.register()

        return true
    }
```

**error**

위 과정을 빼먹으면 아래와 같은 에러를 마주한다.

<img width="862" alt="스크린샷 2021-06-11 오후 3 10 10" src="https://user-images.githubusercontent.com/69136340/122340071-c69cef00-cf7c-11eb-9241-8e61bb5648e4.png">

### 네번째, `Transformer` 와 `Custom Class` 설정.

<img width="1125" alt="스크린샷 2021-06-11 오후 3 03 05" src="https://user-images.githubusercontent.com/69136340/122340079-c8ff4900-cf7c-11eb-8c48-7d60e2090f1f.png">

custom class 설정에 내가 만든 custom class 를 넣어보는 과정이 끝났다.

[How to save an array of custom data types in Core Data with Transformable and NSSecureCoding in iOS - Lexicon Digital](https://www.lexicon.com.au/blog/how-to-save-an-array-of-custom-data-types-in-core-data-with-transformable-and-nssecurecoding-in-ios)

### 결론

위와 같은 방법으로 Core Data 에서 기본적으로 사용하지 못하는 UIColor, CGPoint 를 멤버로 가지는 커스텀 클래스를 만들어서 사용할 수 있다.

하지만 CRUD 는...? 기존의 방법은 entity 를 가져와서 그 entity 에 setValue() 로 값을 추가해주는 방식이었는데 나는 지금 entity 안의 Writings 안에 Story 를 CRUD 해야한다.

→ 릴레이션으로 풀어가보자

## Relationship

<img width="1000" alt="스크린샷 2021-06-12 오전 12 20 43" src="https://user-images.githubusercontent.com/69136340/122340147-e0d6cd00-cf7c-11eb-8ec4-b5e32ed407e2.png">

<img width="1000" alt="스크린샷 2021-06-12 오전 12 20 25" src="https://user-images.githubusercontent.com/69136340/122340153-e3d1bd80-cf7c-11eb-92ec-64d90993ac26.png">

<img width="450" alt="_2021-06-12__12 08 05" src="https://user-images.githubusercontent.com/69136340/122340167-e7fddb00-cf7c-11eb-855c-c9c9dda64123.png">


### 문제

위의 방식으로 진행시에 NSSet 으로 저장이 되기 때문에 디비를 설정시에 자료형이 set 이라면 복수 값과 순서가 없기 때문에 무리라고 판단하고 여기까지만 진행했다.

[Mastering In CoreData (Part 5 Relationship Between Entities in Core Data)](https://ali-akhtar.medium.com/mastering-in-coredata-part-5-relationship-between-entities-in-core-data-b8fea1b50efb)

### CRUD

릴레이션을 활용한 CRUD.

[Mastering In CoreData (Part 6 Relationship Between Entities in Core Data CRUD Operation)](https://ali-akhtar.medium.com/mastering-in-coredata-part-6-relationship-between-entities-in-core-data-crud-operation-87138fdf6fea)
