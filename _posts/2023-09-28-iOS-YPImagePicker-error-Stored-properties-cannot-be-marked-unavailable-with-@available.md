 ---
title:  "iOS) YPImagePicker error :  Stored properties cannot be marked unavailable with `@available`"
categories:
- iOS

date:   2023-09-28  15:43:00 +0900
author_profile: false
---
- Xcode 15(15A240d) 업데이트 이후 YPImagePicker 오픈소스 라이브러리에서 저장 프로퍼티가 `@available` 를 가지며 생기는 에러가 있었습니다.

<img width="1351" alt="스크린샷 2023-09-26 오전 12 16 24" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/acc03b8d-176b-4e8c-9ef1-86515c3954b4">

이를 fork 해서 직접 저장 프로퍼티를 삭제해볼까 생각도 해보고 이번 Xcode 업데이트와도 관련있는거 같아서 기다려봤습니다.

그러던 중 2023.9.26 기준으로 업데이트 되었습니다!

<img width="600" alt="스크린샷 2023-09-28 오후 2 51 50" src="https://github.com/TeamNADA/NADA-iOS-ForRelease/assets/69136340/b53507e9-061c-4a3d-adc4-9e48a1caa06a">

> https://github.com/Yummypets/YPImagePicker/releases/tag/5.2.2
> 

아래 커밋 기록에서 `@available` 를 가지는 저장 프로퍼티가 사라졌습니다.
해당 에러를 겪게 되면 `pod update` 를 통해서 YPImagePicker 버전을 5.2.2 로 업데이트하면 됩니다.(cocoaPods 1.13.0 기준)

> https://github.com/Yummypets/YPImagePicker/commit/e59d7a6c16231f391dd51dbda087936cda8b56fe
>
