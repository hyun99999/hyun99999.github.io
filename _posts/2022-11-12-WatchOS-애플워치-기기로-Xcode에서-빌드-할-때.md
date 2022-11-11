---
title:  "WatchOS) 애플 워치 기기로 Xcode 에서 빌드 할 때"
categories:
- iOS

date:   2022-11-12  08:34:00 +0900
author_profile: false
---
[Apple Developer Documentation](https://developer.apple.com/documentation/watchos-apps/testing-custom-notification-interfaces)

위는 Notification interfaces 를 기기에서 빌드하는 방법을 소개해준 글입니다. 이를 통해 우리는 손목에 착용하지 않아도 어떤 조건으로 기기에서 빌드할 수 있는지 알 수 있습니다.

**기기를 손목에 착용하지 않은 상태에서 notification interfaces 를 테스트하려면 다음의 단계를 따르세요.**

- 애플 워치에서 손목 감지를 비활성화 합니다. companion iPhone 의 watch 앱 또는 watch 의 Setting 에서 설정할 수 있습니다. 옵션은 **Passcode > Wrist Detection** 에 있습니다.
- 애플 워치가 충전기에 연결되어 있지 않은지 확인합니다.
- iPhone 을 잠그세요.

watch-only app 을 만들더라도 내 iPhone 기기를 통해서 애플워치에 접근할 수 있었다.

이때 provisioning profile 에 애플 워치 기기를 추가하여 빌드할 수 있다는 창이 등장한다. 이를 통해 신뢰하고 빌드할 수 있는 기기로 추가할 수 있었다.
