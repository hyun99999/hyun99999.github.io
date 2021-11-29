---
title:  "iOS) Share and Action Extension 사용해보기(1) - HIG 문서 읽어보기"
categories:
- iOS

date:   2021-11-30  02:05:00 +0900
author_profile: false
---

1. **HIG 문서 읽어보기**
2. Action Extension 사용해보기
3. Share Extension 사용해보기

먼저 HIG 에서 `Share & Action Extension` 이 무엇인지 부터 알아보자구요!

[Sharing and Actions - Extensions - iOS - Human Interface Guidelines - Apple Developer](https://developer.apple.com/design/human-interface-guidelines/ios/extensions/sharing-and-actions/)

HIG 문서에서는 Sharing 과 Actions 를 같이 소개하고 있어요! 아마도 Activity view 에서 표시되기 때문인 것 같네요!

다음은 HIG 문서를 옮겨놓은 글입니다.

**Share extensions** 은 현재 context 를 apps, social media accounts, 다른 서비스로 공유하는 편리한 방법을 제공한다. **Action extensions** 은 북마크 추가, 링크 복사, 이미지 저장과 같은 컨텐츠별 작업을 제공한다.

사람들은 activity view 표시하기 위해서 앱에서 버튼을 탭해서 share extensions 와 action extensions 에 액세스 한다. activity view 에는 현재 context 와 관련된 extensions 만 표시합니다. 예를 들어 비디오를 편집하는 동안 텍스트 조작 액션을 볼 수 없다. activity view 내에서 share extensions 는 action extensions 위에 나열된다.

- 잘.. 이해가 되지는 않네요..! 근데 우리가 아예 모르는게 아닙니다! 바로 아래보여지는 뷰가 activity view 이고 위에는 share extensions 아래는 action extensions 입니다.

<img width="250" alt="1" src="https://user-images.githubusercontent.com/69136340/143911270-e4472122-19c7-4c44-ac16-86b4af4cad8e.png">

**집중된 단일 작업을 활성화.** extesion 은 mini-app 이 아닙니다. 현재 context 와 관련된 좁은 범위의 작업을 수행합니다.

**친숙한 인터페이스를 만드십시오.** share extensions 의 경우, 시스템에서 제공하는 구성 뷰는 친숙하고 시스템 전반에 대해서 일관된 sharing 경험을 제공합니다. 가능하면 사용하십시오.

action extesions 의 경우, 앱 이름을 포함하거나 recognizable 하고 앱의 자연스러운 extensions 처럼 느끼게 인터페이스를 디자인하세요.

**상호작용을 간소화하고 제한합니다.** 최고의 extesions 은 사람들이 몇 단계만 거치면 작업을 수행할 수 있는 것입니다. 예를 들어, share extesion 은 탭 한번으로 social media account 에 이미지를 즉시 게시할 수 있습니다. 필요한 경우에만 인터페이스를 제공하십시오.

**extesion 위에 모달 뷰를 배치하지 마시오.** extensions 는 기본적으로 모달뷰 내에 표시됩니다. alert 는 extension 위에 의미 있을 수 있지만 추가적인 모달뷰를 계층화하지 마십시오.

**긴 작업의 진행상황을 표시하려면 main app 을 사용하세요.** activity view 는 sharing 또는 action 시작한 후 즉시 사라져야 합니다. 시간이 많이걸리는 작업은 백그라운드에서 계속되어야 하고 main app 은 이러한 작업을 확인할 수 있는 방법을 제공해야 합니다. 이를 위해서 notifications 를 사용하지 마세요. 사람들은 작업이 완료될 때마다 notification 을 보고 싶어하지 않지만 문제가 있는경우 알려주는 것은 괜찮아 합니다.

extension icon 에 템플릿 이미지 사용하세요. 템플릿 이미지는 아이콘을 만들기 위해서 mask 를 사용합니다. 적절한 transparency 와 antialiasing 이 있는 흑백을 사용하고 그림자를 포함하지 마십시오. 템플릿 이미지는 약 70px * 70px  크기의 중앙에 위치해야만 합니다.

- For additional guidelines, see [Activity Views](https://developer.apple.com/design/human-interface-guidelines/ios/views/activity-views/). For developer guidance, see [Share](https://developer.apple.com/library/content/documentation/General/Conceptual/ExtensibilityPG/Share.html) and [Action](https://developer.apple.com/library/content/documentation/General/Conceptual/ExtensibilityPG/Action.html) in [App Extension Programming Guide](https://developer.apple.com/library/content/documentation/General/Conceptual/ExtensibilityPG/index.html).

<aside>
👻 TIP
Share extensions automatically use your app icon, instilling confidence that the extension is in fact provided by your app.
</aside>
