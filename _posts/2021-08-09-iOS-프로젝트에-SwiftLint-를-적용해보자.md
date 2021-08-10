---
title:  "iOS) 프로젝트에 SwiftLint 를 적용해보자"
categories:
- iOS

date:   2021-08-09  01:41:00 +0900
author_profile: false
---
[SwiftLint](https://github.com/realm/SwiftLint) 오픈라이브러리를 프로젝트에 적용해보려고 합니다.

먼저 SwiftLint 는! 코딩 컨벤션을 모아둔 가이드 라인입니다. 이를 통해 swift 코드가 일관성을 가지도록 도와줍니다.

### 사용법

1.podfile 에 추가해준 후 `pod install` 진행한다.

```swift
pod 'SwiftLint'
```

2.TARGETS > Build Phase 에서 `+` 를 통해 새로운 Run Script 를 생성해준다.

<img src ="https://user-images.githubusercontent.com/69136340/128639172-f25e3c9d-759b-44b6-a312-dc30f2af4d09.png" width ="800">

3.Run Script 에 다음과 같이 적어준다.

```swift
${PODS_ROOT}/SwiftLint/swiftlint
```

<img src ="https://user-images.githubusercontent.com/69136340/128639170-43628bfb-e390-4a3e-b8d3-8d8fd71215e9.png" width ="800">

프로젝트 바로 아래에 `.swiftlint.yml` 파일을 생성해준다.

<img src ="https://user-images.githubusercontent.com/69136340/128639168-32132daf-277a-4832-921b-6529942bddb8.png" width ="600">

**주의: 아래와 같이 폴더 안이 아니라 프로젝트 바로 아래에 생성해준다.**

<img src ="https://user-images.githubusercontent.com/69136340/128639164-79e8e346-e156-400f-8f58-f50247b3e481.png" width ="300">

- 본인의 필요에 따라서 다음과 같이 작성해주면된다. excluded 는 제외하고 싶은 폴더나 파일을 작성하면 된다.
- 더 많은 disabled_rules 가 궁금하다면 아래의 주소를 참조하자.
[SwiftLint rule](https://realm.github.io/SwiftLint/rule-directory.html)

<img src ="https://user-images.githubusercontent.com/69136340/128639167-dd157be7-9b6e-40ba-bb77-1e6d1f5e4514.png" width ="400">

참고:

[iOS ) 내 프로젝트에 SwiftLint를 적용해보자](https://zeddios.tistory.com/447)
