---
title:  "iOS) Swift Package Manager 를 사용해보자"
categories:
- iOS

date:   2021-08-27  10:51:00 +0900
author_profile: false
---
## 💨 Swift Package Manager 를 사용해보자

Xcode 11 부터 애플에서 공식으로 지원하고 Xcode 에 내장된 패키지 매니저이다.

우선 Swift Package Manager(SwiftPM) 를 사용해서 SnapKit 를 설치해보자

1️⃣ [Targets] → [General] → [Frameworks, Libraries, and Embedded Content] → [+]

<img src ="https://user-images.githubusercontent.com/69136340/131058384-216ba893-633f-406b-8d73-d7e452ba7c67.png" width ="700">

2️⃣ Add Package Dependency...

<img src ="https://user-images.githubusercontent.com/69136340/131058388-e03643fe-4f04-480d-9004-fd26d9756e6a.png" width = "400">

3️⃣ 사용하고 싶은 라이브러리의 주소를 기입한다.(깃허브 주소를 기입해주었다.)

<img src ="https://user-images.githubusercontent.com/69136340/131058390-03fab7b6-a158-4cf6-a638-0a07e46330ef.png" width = "600">

4️⃣ 설치 시 원하는 버전, 브랜치 및 커밋을 설정할 수 있다.

<img src ="https://user-images.githubusercontent.com/69136340/131058393-da3d6bfb-8447-4ebc-81b4-6134f8ddd1f7.png" width ="600">

5️⃣ 원하는 package product 를 골라서 [Finish]


<img src ="https://user-images.githubusercontent.com/69136340/131058397-ee76bcbf-b04b-42ab-bef5-695f17172b87.png" width ="600">

SnapKit 이 제대로 설치된 것을 확인할 수 있었다.

<img src ="https://user-images.githubusercontent.com/69136340/131058401-ccbebe71-502b-478d-a694-4a66273ceadc.png" width ="800">

이렇게 SwiftPM 를 사용해보았다. 생각보다 설치하는 과정은 쉬웠다.

그렇다면 어떤 SwiftPM 의 장단점들이 있는지 알아보자.

## 💨 SwiftPM 의 장단점은?

### ❗️ 장점

- package 를 추가하기 쉽다. podfile 을 관리하지 않아도 된다.
- 써드파티의 CocoaPods 과는 달리 퍼스트파티 툴이다. 추가적인 설치가 필요없다.
- Xcode 사이드바에 명확하게 표시하고 패키지의 현재 버전도 보여준다.

### ❗️ 단점

- CocoaPods 가 생성하는 Pods/ 디렉토리에 해당하는 것이 없다. SwiftPM 을 사용하면 dependencies 가 프로젝트의 파생 데이터 디렉토리(~/Library/Developer/Xcode/DerivedData/...) 깊숙이 저장됩니다.
- 패키지를 업데이트할 때 all or nothing 의 선택지만 주어진다. 단일 패키지에 대한 버전을 업데이트 할 수 없다.
- SwiftPM 은 CocoaPods 와 같은 커뮤니티 기반 도구보다 느린 프로세스다. CocoaPods는 수정 사항 및 새로운 기능을 위해 새 버전을 빠르게 릴리즈할 수 있지만 SwiftPM은 Xcode 릴리즈 일정에 구속됩니다.
- SwiftPM 은 현재 Objective-C 와 Swift 가 혼합된 라이브러리를 지원하지 않는다.
- SwiftPM 은 현재 이미지, 스토리보드, 자산 카탈로그 등과 같은 리소스 번들을 지원하지 않습니다.

## 💨 종합

- 작고 간단한 프로젝트에서 나 혼자 개발을 하고 dependencies 를 다 소유하고 있다면 위의 단점들이 영향을 미치지 않기 때문에 SwiftPM 은 유용하다.
- 크고 복잡한 팀작업의 경우 CocoaPods 이 선호된다.
- 그럼에도 SwiftPM 은 가능성이 있다. 언급한 주이슈들은 쉽게 해결가능하다.(패키지별 업데이트 추가, linking 을 위한 패키지별 타켓 추가, 프로젝트 루트에 SwiftPM/ 디렉토리 유지하는 옵션 추가, 번들 리소스 포함 허용, Objective-C 와 Swift 혼합 허용 등)

### 출처 :

[[Swift 5.2][SwiftPM] Swift Package Manager를 이용하여 패키지를 통합 관리하기 - Proxy Module](http://minsone.github.io/ios/mac/swift-package-manager-proxy-modular)

[My experience replacing CocoaPods with SwiftPM](https://www.jessesquires.com/blog/2020/02/24/replacing-cocoapods-with-swiftpm/)
