---
title:  "WWDC22) Hello Swift Charts"
categories:
- iOS

date:   2022-10-26  20:52:00 +0900
author_profile: false
---
👉 Overview

시뮬레이터에서 dynamic interface 를 테스트 할 준비가 되면, notification interface 를 실행하기 위한 custom build scheme 를 생성합니다.
scheme 를 구성하기 위해서 test data 가 포함된 JSON 데이터 파일을 지정해주어야 합니다.
그러기 위해서 PushNotificationPayload.apns 파일을 적절하게 수정해주면 됩니다.. 여기에는 remote notification 을 시뮬레이션하기 위해 필요한 키가 들어있습니다.
👉 Add a Scheme

notification 을 앱에 추가할 때 테스트할 scheme 도 추가해야 합니다. 여러가지 category 의 notification 을 테스트하기 위해서는 각 category 에 대한 scheme 를 추가해야 합니다.

1

Edit Sheme > Run 에서 Watch Interface 를 Dynamic Notification 으로 설정하고, Notification Playload 를 설정해줍니다.

2

notification scheme 를 실행하면 해당 payload 가 워치로 전송됩니다. 또한, payload 파일을 아이폰, 애플 워치 시뮬레이터로 끌어놓아서 notification 을 트리거할 수 있습니다.

이 경우에는 Simulator Target Bundle 을 target 의 bundle ID 로 설정해야 합니다. 또한, 해당 기기가 알림을 수신할 수 있는 권한이 이미 요청되어 있어야 합니다.

마지막으로, 즉시 전달되는 것은 아닙니다. 최대 1분이 지연될 수 있습니다.

To add a new payload file:

3

이를 통해 프로젝트에 여러 payload 파일을 포함하고, build scheme 를 변경하면서 테스트를 단순화할 수 있습니다.

👉 Edit the JSON Payload

payload 파일을 수정해서 실제적으로 앱에 전달되는 notification content 와 일치시킬 수 있습니다. 자세한 내용은 Apple Developer - Generating a remote notification 을 참조하면 됩니다.

payload 파일에는 WatchKit Simulator Actions key 가 포함되어 있습니다. 이 키는 일반 notification’s payload 의 일부가 아닙니다. 각 action button 을 나타냅니다.

다음의 키가 포함될 수 있습니다.

title
(Required) action button 의 제목입니다.
identifier
(Required) 사용자가 선택한 action 을 식별하는 문자열입니다. 사용자가 버튼을 탭하면, 시스템은 notification center delegate 의 userNotificationCenter(_:didReceive:withCompletionHandler:) 메소드를 호출합니다. 시스템은 이 키의 값을 이 메소드에 전달된 UNNotificationResponse 객체의 actionIdentifier 프로퍼티 에 할당합니다.
behavior
(Optional) 이 키의 유일한 값은 문자열 textInput 입니다. 만약 이 키가 있다면, resulting button 이 text input 을 트리거 합니다.
destructive
(Optional) 값 1 또는 0, 여기서 1은 resulting button 이 destructive action 을 수행하는 것을 나타내는 방식으로 렌더링 됩니다. 값이 0이면 정상적으로 렌더링됩니다.
background
(Optional) 값 1 또는 0, 여기서 1은 버튼이 백그라운드에서 앱을 실행하도록 합니다.
👉 Send Test Notifications to the Watch

시스템이 애플 워치에 notification 을 전달할지 여부를 자동으로 결정하기 때문에(애플워치를 착용하게 되면 자동으로 전달된다.) 기기에서 테스트할 때 시스템이 예상대로 notification 을 애플워치로 라우팅하도록 기기의 상태를 설정해야 할 수 있습니다.

기기를 손목에 착용하지 않은 상태에서 notification interfaces 를 테스트하려면 다음의 단계를 따르세요.

애플 워치에서 손목 감지를 비활성화 합니다. companion iPhone 의 watch 앱 또는 watch 의 Setting 에서 설정할 수 있습니다. 옵션은 Passcode > Wrist Detection 에 있습니다.
애플 워치가 충전기에 연결되어 있지 않은지 확인합니다.
iPhone 을 잠그세요.
💫 테스트를 해봅시다!

watch 에서 빌드하게 되면 알림 권한을 다음과 같이 요청합니다.


바로 알림이 오게 됩니다.


짠



❓ Simulator Target Bundle

추가적으로, 빌드할 때 마다 말고, 앞서 살펴본 것처럼 apns 파일을 기기에 끌어 놓아서 받아보겠습니다.

이때, 그냥 집어넣게 되면 다음과 같은 에러가 나옵니다.

7

해당 target 의 bundle identifier 를 apns 파일에 Simulator Target Bundle key 로 추가해주면 됩니다.
