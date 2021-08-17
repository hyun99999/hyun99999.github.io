---
title:  "iOS) 애플리케이션의 Life Cycle"
categories:
- iOS

date:   2021-08-17  13:40:00 +0900
author_profile: false
---
# App's Life Cycle

## 1. iOS 13 이후
*- UISceneDelegate 객체를 사용하여 scene-based app 의 life-cycle 에 응답*
- 앱이 scenes 를 지원하는 경우 UIKit 은 각각에 대해 별도의 life-cycle 이벤트를 제공.
  > scene : 기기에서 실행되는 app's UI 의 한 인스턴스.
- app 에 대한 여러개의 scenes 을 만들고 별도로 보여주거나 숨길 수 있다. 왜냐면 고유한 life-cycle 이 존재하기 때문이다. 예를들어 한 장면은 foreground 에 있고 나머지 장면들은 background 혹은 일시중단 될 수 있다.
> Scene support 는 opt-in(선택적) 기능이다. Info.plist 파일에 UIApplicationSceneManifest 키를 추가하면 사용 가능.
  > - 멀티 윈도우 기능을 이야기한다. 아이패드 등에서 활용.
- 유저나 시스템이 앱의 새로운 scene 을 요청하면 UIKit 은 이를 생성하고 'Unattached' 상태로 둔다.
- 유저의 요청된 scenes 은 'Foreground' 로 빠르게 옮겨진다. 시스템이 요청한 scene 은 'Background' 로 이동하여 이벤트를 처리한다.
  - 예를 들어 시스템은 'Background' 에서 scene 을 실행하여 location event 를 수행할 수 있다.
- 유저가 앱을 닫으면(dismiss) UIKit 은 관련 scene 을 'Background' 상태로 이동하고 'Suspended' 된다. UIKit 은 리소스를 회수하기 위해 'Background' 나 'Suspended' 된 scene 연결을 끊고 'Unattached' 상태로 반환한다.

- 다음은 상태전환을 보여준다.
<img src ="https://user-images.githubusercontent.com/69136340/104924152-619a9880-59e0-11eb-9672-067e2ccfe87a.PNG" width="500">

  > - Unattached (시스템이 connection notification 을 주기 전까지는 이 상태 유지. 메모리를 점유하고 실행중인 상태.)
  > - ForegroundInactive (앱이 실행중, event 를 받지 않는 상태)
  > - ForegroundActive (앱이 실행중, event 를 받고있는 상태. 다른 상태로 전환되는 동안에 이 상태를 통과하게 된다. 예를 들어 시스템 알람이 오거나 창을 내리거나 app-switching 상태에 있는 경우 등)
  > - Background (앱이 백그라운드에서 실행되고 있는 상태)
  > - Suspended (앱이 백그라운드에 있으며, 아무것도 실행되지 않는 상태. 메모리 점유중하지만 대기 상태)

- App delegate 와 차이점
  - background에서만 Unattached가 됩니다. (AppDelegate 에서는 suspended 나 background 상태에서 not running 가능.)
  - 화살표들의 차이
    - Scene delegate에서는 Suspended에서 바로 Inaction로 가는 화살표 없음.
    - Scene delegate에서 Unattached에서 inactive 될때 실선. inactive에서 active가 될때 점선.
    - Scene delegate에서 Unattached에서 background가 될때 실선입니다.
> - *점선의 경우에는 특별한 event 없이도 시스템이 자동으로 수행해주는 상태의 전환이라고 할 수 있다.*
> - *실선의 경우에는 사용자나 시스템에 의해 발생한 event로 인해서 발생하는 상태의 전환이라고 할 수 있다.*

## 2. iOS 12 이전 
*- UIApplicationDelegate 객체를 사용하여 life-cycle 에 응답*
- iOS 12 이하 버전에서는 scene 지원하지 않는다. UIKit 은 모든 life-cycle 이벤트를 UIApplicationDelegate 로 전달한다.
- launch 후 시스템은 UI 가 화면에 표시되는지 여부에 따라서 app 을 inactive 또는 background state 로 전환.
- foreground 로 런칭할 때 시스템은 자동으로 active 상태로 전환. 그 후에는 app 이 종료전까지 active 와 background 에서 변동한다.

<img src ="https://user-images.githubusercontent.com/69136340/104924154-62332f00-59e0-11eb-8dcb-1ffd9f22b7d8.PNG" width="500">

  > - Not Running (앱이 시작되지 않았거나 시스템에 의해 종료된 상태. 메모리에도 없고 프로세스 관점에서도 아무것도 실행되지 않는다.)
  > - Inactive (앱이 실행 중, event를 받지 않는 상태)
  > - Active (앱 실행 중, event 받는 상태)
  > - Background (앱이 백그라운드에 있지만 코드가 실행되고 있는 상태)
  > - Suspended (앱이 백그라운드에 있으며, 아무것도 실행되지 않는 상태. 메모리 점유중하지만 대기 상태) 

## scene, window, view 의 관계
> - *UIScene 은 app UI 의 객체이며, window 와 view controller(View) 를 포함*
> - *UIWindow 는 app UI 의 뒷 배경. view 에 이벤트를 보내는 객체이다*
> - *UIView 는 화면에서 사각형 모양의 content 를 관리하는 객체이다*


- 출처 : https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle
- 출처 : https://developer.apple.com/documentation/uikit/app_and_environment/scenes/specifying_the_scenes_your_app_supports
- 출처 : https://jercy.tistory.com/11?category=728321
- 출처 : https://velog.io/@minni/iOS-Application-Life-Cycle
