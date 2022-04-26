---
title:  "SwiftUI) Podcasts clone coding - Menu"
categories:
- iOS

date:   2022-04-26  22:38:00 +0900
author_profile: false
---
### 내용

- SwiftUI 사용해서 Podcasts 앱의 클론코딩을 진행했다.
- 아래의 Menu 를 구현해보자!

<img src="https://user-images.githubusercontent.com/69136340/165288586-6152d1c4-4acb-4a71-9595-043ebbc3b06a.jpeg" width ="250">

해당 컴포넌트는 `Menu` 이다. 개발자 문서를 통해서 알아보자.

## Menu

A control for presenting a menu of actions.

### **Overview**

다음 코드는 세 개의 버튼으로 구성된 Menu 와 세 개의 버튼을 포함하는 하위 메뉴를 보여줍니다.

```swift
// action 은 커스텀 메서드입니다.
Menu("Actions") {
    Button("Duplicate", action: duplicate)
    Button("Rename", action: rename)
    Button("Delete…", action: delete)
    Menu("Copy") {
        Button("Copy", action: copy)
        Button("Copy Formatted", action: copyFormatted)
        Button("Copy Library Path", action: copyPath)
    }
}
```

<img src="https://user-images.githubusercontent.com/69136340/165288278-8f1ec1f9-58f4-4bda-b2c1-d49577bced38.mov" width ="250">

menu 타이틀을 이전 예시처럼 ✅ (참고)[LocalizedStringKey](https://developer.apple.com/documentation/swiftui/localizedstringkey)(문자열의 localization) 를 사용해서 만들거나, image 나 text 뷰 같은 다수의 뷰를 만드는 view builder 를 사용해서 만들 수 있습니다.

```swift
Menu {
    Button("Open in Preview", action: openInPreview)
    Button("Save as PDF", action: saveAsPDF)
} label: {
    Label("PDF", systemImage: "doc.fill")
}
```

<img width="300" alt="스크린샷 2022-04-26 오후 6 20 45" src="https://user-images.githubusercontent.com/69136340/165288390-98cc1924-fd64-4b88-8a8d-346c6d2e26ea.png">

### **Primary Action**

Menu 는 커스텀 `primary action` 으로 만들 수 있습니다. `primary action` 은  사용자가 control 의 body 를 taps 혹은 clicks 했을 때 수행되고 `menu presentation`(위와 같은 메뉴를 보여주는 기능) 은 long press 혹은 ✅ (참고)`menu indicator` 를 click 하는 것과 같은 보조 제스처에서 발생합니다.

다음 코드는 menu 에 표시되는 advanced options 와 함께 책갈피를 추가하는 menu 를 만드는 예시입니다.

```swift
Menu {
// 🔥 long press 혹은 menu indicator 를 click 해야 등장한다.
    Button(action: addCurrentTabToReadingList) {
        Label("Add to Reading List", systemImage: "eyeglasses")
    }
    Button(action: bookmarkAll) {
        Label("Add Bookmarks for All Tabs", systemImage: "book")
    }
    Button(action: show) {
        Label("Show All Bookmarks", systemImage: "books.vertical")
    }
} label: {
    Label("Add Bookmark", systemImage: "book")
} primaryAction: {
// 🔥 taps 혹은 clicks.
    addBookmark()
}
```

🚨 ***에러***

<img src="https://user-images.githubusercontent.com/69136340/165288778-b688d07b-9772-4ea6-a647-4b7fd19cd860.png" width ="500">

🙆‍♂️ ***해결: if #available(iOS 15.0, *)*** 

iOS 15 이상부터 primaryAction 을 사용할 수 있었고, #available 를 사용해서 분기처리 해주었습니다.

```swift
// 🔥 iOS 15 이상부터 사용가능.
if #available(iOS 15.0, *) {
    Menu {
        Button(action: {}) {
            Label("Add to Reading List", systemImage: "eyeglasses")
        }
        Button(action: {}) {
            Label("Add Bookmarks for All Tabs", systemImage: "book")
        }
        Button(action: {}) {
            Label("Show All Bookmarks", systemImage: "books.vertical")
        }
    } label: {
        Label("Add Bookmark", systemImage: "book")
    } primaryAction: {
        print("primary action")
    }
} else {
    // Fallback on earlier versions
}
```

### Styling Menus

[menuStyle(_:)](https://developer.apple.com/documentation/swiftui/view/menustyle(_:))modifier 를 사용해서 메뉴의 스타일을 변경할 수 있습니다. 아래의 코드는 custom style 을 적용하는 방법을 보여줍니다.

```swift
Menu("Editing") {
    Button("Set In Point", action: setInPoint)
    Button("Set Out Point", action: setOutPoint)
}
// custom style
.menuStyle(EditingControlsMenuStyle())
```

---

## 참고

### ✅ LocalizedKeyString

- LocalizedStringKey 인스턴스를 만들어서 문자열을 localize 할 수 있습니다.

UIKit 에서 `UILabel` 은 String 로 초기화를 합니다. 그러나 SwiftUI 에서 UILabel 역할을 하는 `Text` 는 `LocalizedKeyString` 을 통해서 초기화를 합니다. 즉, `NSLocalizedString` 를 사용하지않아도 자동으로 localization 되는 것입니다.

`Menu` 역시 `LocalizedStringKey` 를 사용해서 초기화 할 수 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/165289028-0fb7288f-8348-44ed-9651-4443c9dc0ad3.png" width ="500">

### ✅ menu indicator

`primary action` 을 사용하게 되면 menu presentation 은 long press 와 `menu indicator` 를 통해서 기능한다고 했습니다. 

****menuIndicator(_:)**** 

Sets the menu indicator visibility for controls within this view.

```swift
func menuIndicator(_ visibility: Visibility) -> some View
```

위의 modifier 를 통해서 menu indicator 가 있는 Menu 를 만들 수 있습니다.

- 화살표와 같은 indicator 가 등장한다고 하는데 왜인지.. 등장하지 않았습니다..!

```swift
Menu {
// long press 혹은 menu indicator 를 click 해야 등장한다.
    Button(action: addCurrentTabToReadingList) {
        Label("Add to Reading List", systemImage: "eyeglasses")
    }
    Button(action: bookmarkAll) {
        Label("Add Bookmarks for All Tabs", systemImage: "book")
    }
    Button(action: show) {
        Label("Show All Bookmarks", systemImage: "books.vertical")
    }
} label: {
    Label("Add Bookmark", systemImage: "book")
} primaryAction: {
    addBookmark()
}
// 🔥 menuindicator modifier 를 통해서 가시성을 설정할 수 있다.
.menuIndicator(.visible)
```

[Apple Developer Documentation - menuindicator(_:)](https://developer.apple.com/documentation/swiftui/link/menuindicator(_:))

---

## 클론코딩

- Podcast 클론코딩을 진행해보자.

<img src="https://user-images.githubusercontent.com/69136340/165288586-6152d1c4-4acb-4a71-9595-043ebbc3b06a.jpeg" width ="250">

```swift
Menu {
    Button(action: {
        print("Download Episode")
    }) {
        Label("Download Episode", systemImage: "arrow.down.circle")
    }
    Button(action: {
        print("Play Episode")
    }) {
        Label("Play Episode", systemImage: "play")
    }

    Divider()

    Button(action: {
            print("Play Next")
        }) {
            Label("Play Next", systemImage: "text.insert")
        }
    Button(action: {
             print("Play Last")
        }) {
            Label("Play Last", systemImage: "text.append")
        }
    Button(action: {
            print("Go to Show")
        }) {
            Label("Go to Show", systemImage: "airplayaudio")
        }
    Button(action: {
            print("Save Episode")
        }) {
            Label("Save Episode", systemImage: "bookmark")
        }

    Divider()

    Button(action: {
        print("Report a Concern")
    }) {
        Label("Report a Concern", systemImage: "exclamationmark.bubble")
    }
    Button(action: {
        print("Copy Link")
    }) {
        Label("Copy Link", systemImage: "link")
    }
    Button(action: {
        print("Share Episode...")
    }) {
        Label("Share Episode...", systemImage: "square.and.arrow.up")
    }
} label: {
        Image(systemName: "ellipsis")
            .foregroundColor(.secondary)
}
```

### 에러: Extra argument in call

<img src="https://user-images.githubusercontent.com/69136340/165289232-325b95dd-fe46-461c-b7ea-3423de898d27.png" width ="400">

Podcast 의 Menu 안에는 10개 넘는 Button 들이 필요했고, 다음과 같은 에러를 만나게 되었다. SwiftUI 에서는 View Builder 안에 최대 10개까지만 선언 할 수 있다.

### 해결: 다음과 같이 파일분할하여 진행하였다.

Divider 를 통해서 구분지었는데 해당 영역마다 역할이 있다면 해당 이름으로 지어도 되지만 순서를 나타내기 위해서 다음과 같이 작성했다. 

```swift
Menu {
    ThrdMenu()
    Divider()
    SecondMenu()
    Divider()
    FirstMenu()
    } label: {
        Image(systemName: "ellipsis")
            .foregroundColor(.secondary)
    }

// FristMenu
// Divider
// SecondMenu
// Divider
// ThirdMenu
// 순서로 뷰에 그려진다.
```

## 결과

(Go to Show 의 아이콘을 SF Symbol 에서 찾지 못했다..ㅜ 그래서 비슷한걸로 대체했습니당)

<img src="https://user-images.githubusercontent.com/69136340/165289326-6a370dff-8b9f-4aef-8131-95dc4310401b.png" width ="250">

