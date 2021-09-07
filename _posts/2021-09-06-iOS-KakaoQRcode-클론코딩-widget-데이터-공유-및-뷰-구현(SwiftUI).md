---
title:  "iOS) Kakao QRcode 클론코딩 - widget 데이터 공유 및 뷰 구현(SwiftUI)"
categories:
- iOS

date:   2021-09-06  16:59:00 +0900
author_profile: false
---
### 내용

- 카카오톡 **QR코드, 프로필** 위젯을 만들어보겠다.
- 위젯과 앱간의 프로필(이름, 이미지) 데이터 공유로 다음과 같은 프로필 위젯 만들기

<img src ="https://user-images.githubusercontent.com/69136340/132179692-73b9eb77-9c8e-4a94-8064-315ce2f02073.jpeg" width ="250">

# 시작전

위젯은 기능이 제한적이며 interactive 하지도 않지만 우리는 앱과 위젯이 데이터를 공유하기를 원할수도 있다.

## 🤒 App 과 Extension 간의 관계

[App Extension Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html#//apple_ref/doc/uid/TP40014214-CH21-SW1) 를 살펴보면 App 과 Extension 간의 관계를 볼 수 있다.

<img src ="https://user-images.githubusercontent.com/69136340/132179725-27a452cb-8265-4c70-9fdf-14f57b1ddd62.png" width ="600">

extension's bundle 이 containing app's bundle 내에 중첩되더라도 실행중인 app extension 과 containg app 은 서로의 contatiner 에 접근할 수 없다.

하지만 데이터 공유를 활성화 할 수 있다. containin app 과 contained app extensions 의 App Groups 를 활성화 하고 앱에서 사용할 App Groups 를 지정한다.

App Groups 를 활성화면 app extension 과 containing app 모두 UserDefaults 를 사용해서 데이터를 공유할 수 있다. 그 후 새로운 UserDefaults 개체를 인스턴스화하고 App Groups 의 식별자를 전달하면된다.

그렇다면 구체적으로 데이터 공유하는 몇가지 방법에 대해서 알아보자.

## 🤒 UserDefaults 데이터 공유

[[UserDefaults] App과 Extension간 UserDefaults 공유하기](https://eunjin3786.tistory.com/213)

[iOS 14 WidgetKit 적용하기](https://yoo-kie.tistory.com/27)

## 🤒 Codable 데이터 UserDefaults 로 공유하기

[Sharing Object Data Between an iOS App and Its Widget](https://michael-kiley.medium.com/sharing-object-data-between-an-ios-app-and-its-widget-a0a1af499c31)

## 🤒 KeyChain 데이터 공유

[iOS Widget 개발: Keychain Access Group 으로 Data Share하기](https://medium.com/@flyhigh320/ios-widget-개발-keychain-access-group-으로-data-share하기-a52015fe08be)

## 🤒 CoreData 데이터 공유

[Sharing data with a Widget](https://useyourloaf.com/blog/sharing-data-with-a-widget/)

## 🤒 FileManager 데이터 공유

[How to read files created by the app by iOS WidgetKit?](https://stackoverflow.com/questions/63816153/how-to-read-files-created-by-the-app-by-ios-widgetkit/63816322#63816322)

**프로필 이미지와 이름**을 위젯으로 넘기고 싶었다.

- ❌ KeyChain 을 사용하기에는 비밀번호와 같은 credential 한 정보가 아니라서 적합하지 않다고 생각했다.
- ❌ CoreData 을 사용하기에는 복잡한 관계형 데이터에 적합한 CoreData 가 오버스펙이라고 생각했다.
- ❌ FileManager 을 사용하기에는 디바이스에 파일을 저장해야하는데 오버스펙이라고 생각했다.
- ⭕️ UserDefaults 을 사용해서 적은 정보를 표시하는 위젯을 표현하는 것이 적합하다고 생각했다.

## 🤒 프로젝트 설정

### 📌 App Groups 권한

1️⃣  우선 데이터를 공유하려면 App Groups 에 추가해야한다.

- main app Target → [Signing&Capabilities] → [+ Capability] → [App Groups 추가]

<img src ="https://user-images.githubusercontent.com/69136340/132179831-c78d6037-ef8c-40de-89e8-cb9a05ff901b.png" width = "600">

2️⃣  New Container 을 추가한다.

- group. 을 prefix 로 가지는 포멧이 제공된다. App Group Identifier 는 bundle identifier 처럼 유니크한 값이기 때문에 동일하게 역도메인 이름을 사용했다.

<img src ="https://user-images.githubusercontent.com/69136340/132179873-021e0293-0d5b-42cf-9a0c-4e7b2fe8122e.png" width = "350">

<img src ="https://user-images.githubusercontent.com/69136340/132179884-59cb1476-2e34-447c-9c40-954142c814c8.png" width ="500">

3️⃣ 프로젝트에 entitlements 파일이 추가됨을 확인해보자.

<img src ="https://user-images.githubusercontent.com/69136340/132180274-19806455-bc19-44e8-abdb-61f501684b25.png" width ="600">

4️⃣ 각 Target 에도 App Groups 를 추가하고 활성화한다.

<img src ="https://user-images.githubusercontent.com/69136340/132180283-51eb35a4-5018-4005-9dbe-f4c2306365f1.png" width = "600">


## 🤒 데이터 공유

### 📌 UserDefaults 데이터 공유

- UserDefaults 와 관련된 Extension 을 만들어서 사용해보자

```swift
extension UserDefaults {
    static var shared: UserDefaults {
        // ✅ App Groups Identifier 를 저장하는 변수
        let appGroupID = "group.hyun99999.KakaoQRcodeiOSCloneCoding"

        // ✅ 파라미터로 전달되는 이름의 기본값으로 초기화된 UserDefaults 개체를 만든다.
        // ✅ 이전까지 사용했던 standard UserDefaults 와 다르다. 공유되는 App Group Container 에 있는 저장소를 사용한다.
        // ✅ suitename : The domain identifier of the search list.
        return UserDefaults(suiteName: appGroupID)!
    }
}
```

### 📌 UIImage convert to Data

- UserDefaults 는 UIImage 자료형을 저장하지 않아서 Data 로 변환해야한다.

이건 따로 글을 써봤다.

[iOS) UserDefaults 에 image 저장하기](https://gyuios.tistory.com/101)

# 시작하기

## 🤒 프로필 위젯

- 프로필 사진과 이름을 위젯과 공유하기 위해서 MainViewController 를 수정했다.

<img src ="https://user-images.githubusercontent.com/69136340/132180476-c5ac798c-5456-4ee4-8257-b49858b13ff7.png" width = "250">

- MainViewModel.swift

view model 에 UserDefaults 값을 저장하는 로직을 추가해주었다.

```swift
// ✅ 이름과 프로필 사진을 저장하는 메서드.
// MARK: - UserDefaults

    // ✅ 앱과 위젯이 공유하는 UserDefaults 개체(shared)에 접근
    func setUserDefaults(String value: String, _ key: String) {
        UserDefaults.shared.set(value, forKey: key)
    }

    // ✅ UIImage 의 경우 Data 로 변환해서 UserDefaults 에 저장해주어야 한다.(UIImage 다루지 못함.)
    func setUserDefaults(UIImage value: UIImage, _ key: String) {
        let imageData = value.jpegData(compressionQuality: 1.0)
        UserDefaults.shared.set(imageData, forKey: key)
    }
```

- MainViewController.swift

```swift
// ✅ UserDefaults 에 저장하는 view model 메서드
private func setUserDefaults() {
        if let nameLabel = nameLabel.text {
            viewModel.setUserDefaults(String: nameLabel, "Name")
        }
        if let profileImage = profileImage.image {
            viewModel.setUserDefaults(UIImage: profileImage, "ProfileImage")
        }
}
// ✅ viewDidLoad() 에서 호출해주었다.
```

- KakaoWidget_iOS_CloneCoding.swift

저장한 UserDefault 값을 위젯으로 가져와서 사용해보자.

```swift
// profile widget
struct ProfileEntryView: View {
    var entry: Provider.Entry
    
    var body: some View {
        let nameUserDefaults = UserDefaults(suiteName: "group.hyun99999.KakaoQRcodeiOSCloneCoding")
        // ZStack 을 활용해서 뷰 구성.
        ZStack {
            if let name = nameUserDefaults?.string(forKey: "Name"),
               let profileImageData = nameUserDefaults?.data(forKey: "ProfileImage"),
               let profileImage = UIImage(data: profileImageData) {
                // ✅ 배경으로 이미지 설정
                Image(uiImage: profileImage)
                    .resizable()
                    .scaledToFill()

                // ✅ 좌측하단에 텍스트 설정    
                Text(name)
                    .foregroundColor(.white)
                    .font(.system(size: 17, weight: .bold, design: .default))
                    .alignmentGuide(HorizontalAlignment.center, computeValue: { dimension in
                        dimension[HorizontalAlignment.center] + 40
                    })
                    .alignmentGuide(VerticalAlignment.center, computeValue: { dimension in
                        dimension[VerticalAlignment.center] - 55
                    })
            }
        }
    }
}
```

**결과**

<img src ="https://user-images.githubusercontent.com/69136340/132180555-e4c2b400-afd0-4b10-bfc8-faeebcef96b7.png" width ="250">

## 🤒 QR코드 위젯

- KakaoWidget_iOS_CloneCoding.swift

 카카오톡 QR코드 위젯 이미지를 캡쳐해서 사용했다.

```swift
// QRcode widget
struct QRcodeWidgetEntryView : View {
    let url = URL(string: "qrcode")
    var entry: Provider.Entry

    var body: some View {
        // ✅ 큐알코드 위젯 이미지를 캡쳐해서 에셋에 추가했다.
        Image("qrcodeImage")
            .resizable()
            .scaledToFill()
            .widgetURL(url)
    }
}
```

**결과**

<img src ="https://user-images.githubusercontent.com/69136340/132180639-39c9e0c7-c51d-449d-8dcd-45c67b0fa96d.png" width ="250">

### 참고 :

[[UserDefaults] App과 Extension간 UserDefaults 공유하기](https://eunjin3786.tistory.com/213)

[Sharing data with a Widget](https://useyourloaf.com/blog/sharing-data-with-a-widget/)

### 깃허브

[GitHub - 28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu: 🧚 아요 미라클론코딩 김현규](https://github.com/28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu)
