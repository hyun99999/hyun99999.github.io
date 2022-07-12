---
title:  "WWDC22) What's new in the Photos picker"
categories:
- iOS

date:   2022-07-12  17:05:00 +0900
author_profile: false
---
[What's new in the Photos picker - WWDC22 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2022/10023/)

***본 글은 WWDC 를 보고, 번역 및 요약 그리고 실행해보는 스터디 프로젝트의 일환입니다.***

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/178441324-f2a76393-e3d5-4ac3-90f3-31f35216e159.png">

오늘은 system Photos picker 개선된 몇 가지 사항에 대해 이야기하고자 합니다.

system Photos picker 는 대부분의 앱이 iOS 에서 사진, 비디오에 액세스하는 가장 좋은 방법입니다. 

<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/178441365-a3adf5ca-431c-4219-a4c2-a1bd2847431f.png">

picker 는 process 없이 돌아가므로 라이브러리 액세스를 요청할 필요가 없습니다. 직관적인 UI 와 사용하기 쉬운 API 가 있습니다. PHPicker API 에 익숙하지 않은 경우 이전 연도 WWDC 세션에서 자세히 볼 수 있습니다.

<img width="700" alt="3" src="https://user-images.githubusercontent.com/69136340/178441428-d2b5cd68-af12-4415-bbad-8aa06f47c9ed.png">

# Overview

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/178441448-54a388ce-ac04-4931-b049-a4251246aef6.png">

- 오늘 세션에서는 picker 에 추가한 새로운 기능에 대한 개요로 시작하겠습니다.
- 다음은 현재 지원하는 추가적인 플랫폼과 프레임워크에 대해서 이야기하겠습니다.

**🥽 *Let’s dive in!***

# New features

## PHPickerFilter

picker 는 도입된 이후로 이미지와 비디오, 라이브 포토 간의 필터링을 지원합니다.

<img width="700" alt="5" src="https://user-images.githubusercontent.com/69136340/178441493-efc9fc3e-7955-40de-8c8f-640854f1842d.png">

그러나 일부 앱에서는 다른 요구 사항이 있을 수 있습니다. 예를 들어, screenshot-stitching app 은 오직 스크린샷만 피커에서 보여주려고 합니다.

올해 추가한 새로운 screenshots fileter 로 이제 가능해졌습니다. screenshots 외에도 screen recordings 와 slo-mo videos 와 같은 다른 asset 타입들도 추가했습니다.

<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/178441520-f49e101a-7d2d-47c7-b484-00c4313db998.png">

`PHAsset.PlaybackStyle` 을 사용하여 새 필터를 만드는 방법도 있습니다. 

<img width="700" alt="7" src="https://user-images.githubusercontent.com/69136340/178441563-bede5807-e556-4059-8e5c-a4db61a1ce2e.png">

cinematic videos, depth effect photos 및 bursts 를 제외한 모든 새로운 필터들이 backport 됩니다.(**backport**: 최신 버전에서 가져와서 이전 버전으로 적용하는 작업)

즉, 위의 세가지는 iOS 16부터 지원합니다.

앱이 iOS 15 를 타겟팅하는 경우 여러분들은 iOS 16 SDK 로 컴파일을 하는 동안 계속 사용할 수 있습니다.

## Compound filters

Compound filters 의 경우 기존의 `any` 에 추가하여 `all` 과 `not` 도 사용됩니다. 이 역시 iOS 15 로 백포트되었습니다.

<img width="700" alt="8" src="https://user-images.githubusercontent.com/69136340/178441600-91e9dfd2-d0a0-4afd-b325-09ac462dbcc4.png">

몇가지 코드 예제를 살펴보겠습니다.

<img width="700" alt="9" src="https://user-images.githubusercontent.com/69136340/178441734-0f5bf4ec-fe1e-45ff-9d3f-1ac419237edd.png">

비디오와 live photo 를 표시하려면 `any` 와 결합할 수 있습니다. 또는 스크린샷만 표시하고 싶을 수도 있습니다. 스크린샷 없이 모든 이미지를 표시하려면 images 와 screenshots filters 를 `all` 과 `not` 을 사용하여 결합할 수 있습니다. 마지막 예시에서는 iOS 16 이상을 타겟팅하는 경우  `.cinematicVideos` 필터를 사용할 수 있습니다. 

<img src="https://user-images.githubusercontent.com/69136340/178441759-62a2fc84-27d6-4ea2-8aa8-f9c6d205b4db.png" width ="500">

---

<img src="https://user-images.githubusercontent.com/69136340/178441884-2dd4c909-6d13-4c8a-9a65-8c959bd10562.png" width ="500">

---

<img src="https://user-images.githubusercontent.com/69136340/178441954-1f35641a-9e37-43ec-a60e-67d86ad32194.png" width="500">

**다음으로 half-height picker 와 관련된 개선 사항에 대해 이야기 해보겠습니다.**

## half-height picker

iOS 15 UIKit 에는 picker 를 절반 높이 모드로 표시하는데 사용할 수 있는 새로운 `UISheetPresentationController` API 가 있습니다. 이미 많은 경우에 훌륭하게 작동합니다. 하지만, 일부는 선택한 assets 를 조정하고 이러한 변경사항이 picker 에 다시 반영되도록 커스텀UI 를 구현하고자 할 수 있습니다. 

<img width="700" alt="13" src="https://user-images.githubusercontent.com/69136340/178442046-59355ac0-96d3-4d67-bf83-736f577b04c6.png">

iOS 16에서는 asset identifiers 를 통해서 에셋을 선택해제 할 수 있습니다.

<img width="700" alt="14" src="https://user-images.githubusercontent.com/69136340/178442065-4c950784-7329-41f2-97de-4b3491d4cb16.png">

위의 그림처럼 두 번째 사진은 `deselectAssets` 가 호출된 후 picker 에서 더 이상 선택되지 않습니다. `moveAesset` 메서드를 사용하여 assets 를 재정렬할 수도 있습니다.

<img width="700" alt="15" src="https://user-images.githubusercontent.com/69136340/178442158-450d0381-f8e2-4224-8e8d-8cee7face212.png">

우리는 picker 의 새로운 기능에 익숙해졌습니다. 플랫폼 지원에 대해서 이야기 해보겠습니다.

# Platform support

현재 system Phots picker 는 오직 iOS 와 iPadOS 에서만 사용할 수 있습니다. 올해 macOS 와 watchOS 라는 두 가지 추가 플랫폼에 도입할 예정입니다. iPadOS picker 도 iPad 만을 위한 새로운 디자인으로 업데이트 되었습니다.

<img width="700" alt="16" src="https://user-images.githubusercontent.com/69136340/178442189-02af535c-5a82-4e95-8a91-78017ea7fba2.png">

## iPadOS

먼저 새로운 iPad UI 을 살펴보겠습니다.

<img width="700" alt="17" src="https://user-images.githubusercontent.com/69136340/178442250-13865fff-711f-4685-9222-14cc169ba68f.png">

picker 는 이제 큰 iPad 디스플레이의 이점을 활용하기 위해서 sidebar 가 표시됩니다. 사이드 바를 상요하여 다른 컬렉션간의 빠른 탐색을 할 수 있습니다. 그러나 Split Screen mode 에서는 공간이 충분하지 않으면 기존의 compact picker UI 로 대체됩니다.

*(아래는 현재 iPadOS 에서 제공하는 compact picker UI 입니다.)*

<img src="https://user-images.githubusercontent.com/69136340/178442288-5320778f-47f1-49f1-a2af-278b7884516c.png" width = "700">

> 사진 출처: [https://www.idownloadblog.com/2020/08/11/enhanced-image-picker-iphone-ipad/](https://www.idownloadblog.com/2020/08/11/enhanced-image-picker-iphone-ipad/)

## macOS

macOS picker 는 Mac-style controls 이 있는 sidebar 를 가집니다.

<img width="700" alt="19" src="https://user-images.githubusercontent.com/69136340/178442419-18796410-bc23-4381-afbe-ca1157896d03.png">

그리고 iOS picker 처럼 multiple selection, grid 의 fluid zooming 를 지원하며 사람, 장소 등과 같은 항목을 검색할 수 있는 검색기능이 있습니다.

새로운 picker UI 는 NSOpenPanel 을 사용할 수 있습니다. 이를 사용하여 system photo library 에서 iCloud Photos 에 저장된 에셋을 포함하여 선택할 수 있습니다. 여러분의 앱은 어떤 adoption work 없이 새로운 UI 가 무료로 제공될 것입니다.

<img src="https://user-images.githubusercontent.com/69136340/178442444-eeaf0b4a-d4d7-4c80-b321-6f7cb89d63ef.png" width ="700">

### When to use NSOpenPanel

drag and drop 이 NSOpenPanel picker 에서 지원됩니다. iOS, iPadOS 및 macOS 의 표준 picker  에서도 지원됩니다. 앱에서 몇개의 이미지 혹은 비디오를 선택해야 하는 경우 NSOpenPanel API 만 있으면 됩니다.

그러나, photo library 에서 선택한 파일들은 시스템에 의해 언제나 삭제될 수 있다는 것을 명심해야 합니다. 장기적으로 사용하려면 앱에서 관리하는 위치에 복사해야 합니다.

<img width="700" alt="21" src="https://user-images.githubusercontent.com/69136340/178442547-0e7eb1cd-555f-4eae-96a8-40e9702f560c.png">

### When to use PHPicker

media-centric(미디어 중심) macOS 앱의 경우, 최고의 사용자 경험을 위해 기본적으로 새로운 Photos picker 를 사용하는 것이 좋습니다. 하지만, 앱은 NSOpenPanel API 를 사용하여 파일 시스템에서 에셋을 선택해야하는 대체 방법을 계속 제공 해야 합니다. 고객이 여전히 photo libraries 외부의 에셋을 선택하고 싶어하는 경우가 있습니다.

<img width="700" alt="22" src="https://user-images.githubusercontent.com/69136340/178442574-c6b2207b-40af-4ae9-a690-2b4bfb07915e.png">

## watchOS

마지막으로 watchOS 에 대해서 알아보겠습니다.

처음으로 새로운 API 를 통해서 저장된 이미지에 액세스할 수 있습니다. watchOS picker 는 iOS 와 macOS picker 처럼 프로세스가 필요하지 않으므로 라이브러리 액세스를 요청할 필요가 없습니다.

iOS picker 와 유사한 UI 를 가지고 있지만 더 작은 화면에 최적화되어 있습니다. 그리드 또는 컬렉션 별로 사진을 탐색할 수 있습니다. 

<img src="https://user-images.githubusercontent.com/69136340/178442629-3f12decd-4c52-45c6-b079-f1556a52584e.png" width ="700">

선택 순서를 표시하고 선택 제한을 지정하도록 picker 를 구성할 수 있습니다.

하지만, iOS 와 macOS 와 달리 watchOS picker 는 이미지만 표시됩니다. 장비에 500개 이상의 이미지가 있는 경우 가장 최근의 500개 이미지만 표시됩니다.

<img width="700" alt="24" src="https://user-images.githubusercontent.com/69136340/178442657-d4d35b82-0b16-4e50-9d32-6df4a9601a63.png">

# Framework

PHPickerViewController 는 UIKit 기반인데 어떻게 macOS 나 watchOS 앱에서 사용할 수 있을까요?

짐작하셨겠지만 이제 AppKit 과 SwiftUI 에서 새로운 picker API 를 사용할 수 있습니다.

*(AppKit 은 macOS 를 위한 유저 인터페이스를 구현하는데 필요한 objects 를 포함합니다.)*

<img width="700" alt="25" src="https://user-images.githubusercontent.com/69136340/178442760-a6e03ec8-b418-4be5-b2e9-b993b8a51b75.png">

먼저, AppKit API 를 살펴보겠습니다.

## AppKit

UIKit API 와 매우 유사합니다. 

<img width="700" alt="26" src="https://user-images.githubusercontent.com/69136340/178442857-45112893-f50b-4660-ab15-52b883b10b65.png">

동일한 PHPickerConfiguration 타입과 프로퍼티에 액세스 할 수 있습니다. 약간의 차이점은 PHPickerViewController 가 AppKit 앱을 위한 NSViewController 하위 클래스라는 것입니다.

(NS… : NextSoftware 는 objective-C 의 라이선스를 받아 NEXTSTEP 이라는 개발 환경과 라이브러리를 개발하였다. 이후 Next Computer 와 Sun Microsystems 는 NEXTSTEP 시스템의 표준 규격을 발표하고 이를 OPENSTEP 으로 명명한다. 이후 애플은 넥스트 소프트웨어를 인수하고 NEXTSTEP/OPENSTEP 의 개발환경을 애플 버전에서 Cocoa(코코아)라고 칭했다. NS 접두사는 NEXTSTEP 때 작성된 코드와의 호환을 유지하기위해 애플이 사용하고 있다.)

AppKit 기반의 PHPicker 의 도입으로 legacy media libraray browser 에서 벗어날 때입니다. PHPicker 는 훨씬 더 강력합니다. UIKit 과 AppKit 앱을 동시에 작업할 때 유지관리가 쉽습니다.

<img width="700" alt="27" src="https://user-images.githubusercontent.com/69136340/178442886-f38224ac-fb86-4f8e-b40a-bacbdc460f89.png">

## SwiftUI

iOS picker 를 기억하시나요? 몇 줄의 SwiftUI 코드로 표시할 수 있습니다.

<img width="700" alt="28" src="https://user-images.githubusercontent.com/69136340/178442987-9b56bda3-3c94-4d95-ae3c-810698f973bc.png">

더 중요한 것은 iOS, iPadOS, macOS 그리고 watchOS 와 같이 picker 가 지원되는 모든 플랫폼에서 SwiftUI PhotosPicker API 에 액세스할 수 있다는 것입니다. picker 는 자동으로 플랫폼, 앱의 구성, 사용 가능한 스크린 공간에 따라 최적의 레이아웃을 자동으로 선택합니다.

picker UI 에 대해서 걱정할 필요가 없으므로 앱을 개선하는데 집중할 수 있습니다.

### Loading photos and videos

데모를 통해 새로운 API 를 자세히 살펴보기 전에 먼저 선택한 사진들과 비디오들을 로드하는 방법에 대해 이야기해야 합니다.

SwiftUI 바인딩을 통해 받는 선택 항목에 대해서 placeholder 객체만 포함합니다. 요청 시에 에셋 데이터를 즉시 로드해야할 수 있습니다. 하지만 일부 에셋 데이터들은 즉시 로드되지 않습니다. 

예를 들어 picker 가 iCloud  Photos 로부터 데이터를 다운로드하려고 했지만 기기가 인터넷과 연결되지 않은 경우와 같이 오류가 발생할 때는 로드 작업이 실패할 수 있습니다. 일부 대용량 동영상과 같은 다운로드가 오래걸리는 큰 파일의 경우는 loading indicator 대신 항목별 inline loading UI 를 권장합니다.

<img width="700" alt="29" src="https://user-images.githubusercontent.com/69136340/178443077-a29a93d7-22cf-4031-97e1-a4bd8a34816e.png">

PhotosPicker 는 앱과 extensions 간의 데이터 전송을 위한 새로운 SwiftUI 프로토콜인 `Transferable` 을 사용합니다. Transfeable 을 통해 SwiftUI 이미지를 직접 로드할 수 있지만 advanced use cases 의 경우 로드하려는 데이터 타입을 완전히 제어하기 위해 Transfeable 프로토콜을 준수하는 모델 객체를 정의해야 합니다.

Transferable 에 대한 자세한 내용은 `Meet Transferable` 세션에서 확인할 수 있습니다.

<img width="700" alt="30" src="https://user-images.githubusercontent.com/69136340/178443141-8c3fb165-45fe-46c4-9858-ceb6ddafc8c6.png">

만약 앱이 동시에 많은 아이템을 처리하거나 비디오와 같은 큰 에셋을 처리해야 하는 경우에는 동시에 모든 것들을 메모리에 로드하는 것이 불가능할 수 있습니다. 메모리 사용량을 줄이기 위해서 FileTransferRepresentation 을 사용하여 선택한 에셋을 파일로써 로드할 수 있습니다. 

에셋을 파일로써 로드할 때 lifecycles 를 관리해야 한다는 것을 염두에 둬야합니다. 파일을 받으면 앱 디렉토리에 복사하고 더 이상 필요하지 않으면 삭제해야 합니다.

<img width="700" alt="31" src="https://user-images.githubusercontent.com/69136340/178443168-a2a21bf7-8db5-46ff-82c9-e0b01440e0da.png">

자, 이제 데모를 할 시간입니다!

# Demo

이미 계정 프로필 페이지를 표시하는 데모 앱을 설정했습니다. 현재 프로필 이미지는 placeholder 아이콘입니다. 우리는 PhotosPicker API 를 사용하여 프로필 이미지를 변경하는 edit button 추가하고자 합니다.

프로필 이미지 뷰는 view model 에 정의된 image state 에 응답할 수 있으므로 picker selection 이 수신될 때 이미지 상태를 업데이트하기만 하면 됩니다.

<img width="700" alt="32" src="https://user-images.githubusercontent.com/69136340/178443274-6b78df77-f1fc-4c56-ae4e-7d58aa898477.png">

먼저, view model 로 가서 imageSelection 프로퍼티를 추가해 보겠습니다. 

<img width="700" alt="33" src="https://user-images.githubusercontent.com/69136340/178443291-0d3e1c2e-2cd3-414f-8736-e3c5ac7122b5.png">

selection 바인딩으로 PhotosPicker API 에 전달됩니다. 이제 프로필 이미지 뷰로 돌아가서 picker 를 가져오는 오버레이 버튼을 추가해봅시다.

<img width="700" alt="34" src="https://user-images.githubusercontent.com/69136340/178443330-551c3a83-586e-45f8-8b65-0ad62c249fb9.png">

빌드하고 실행하면 편집 버튼을 눌러 picker 를 불러올 수 있습니다. 

이제 image selection 과 image state 를 연결해 보겠습니다. view model 로 돌아가서 image selection did set 을 수신할 수 있습니다. image selection 이 nil 이라면 image state 를 공백으로 설정합니다.

아직 loadTransferable 메서드를 구현하지 않았기 때문에 컴파일 에러를 볼 수 있습니다.

<img width="700" alt="35" src="https://user-images.githubusercontent.com/69136340/178443350-ecd4d88a-4b46-4a43-afc0-d06a31eab063.png">

구현은 간단합니다. completion handler 에 응답하고 image state 를 업데이트하면 됩니다.

<img width="700" alt="36" src="https://user-images.githubusercontent.com/69136340/178443401-ea155a77-df46-44ed-b0ad-270634e8c034.png">

PhotosPickerItem 의 `loadTransferable(type:completionHandler:)` 메서드를 사용하여 컴플리션 핸들러를 통해 지정한 타입의 인스턴스로 로드하도록 구현했습니다.

<img src="https://user-images.githubusercontent.com/69136340/178443450-2e4761b8-a6cd-49fe-9bc8-0d15bce5af7d.png" width ="700">

편집 버튼을 탭하고 이미지를 선택할 수 있습니다. 예상대로 작동합니다!

<img width="700" alt="38" src="https://user-images.githubusercontent.com/69136340/178443462-91501269-d1b2-4a11-82f4-f5f76d81bf72.png">

사실 프로젝트는 이미 macOS 에서도 실행할 수 있도록 설정되어 있습니다. 컴파일 해봅시다.

picker 를 열고 이미지를 선택하면 앱에 반영된 것을 볼 수 있을 것입니다.

<img width="700" alt="39" src="https://user-images.githubusercontent.com/69136340/178443524-8aba4134-d1ff-4ab6-8873-5e24830e25cc.png">

iOS 와 macOS 에서 데모를 보았지만 동일한 코드가 watchOS 에서도 작동합니다. 그러나 명심해야할 것이 몇 가지 있습니다.

watchOS picker 는 간단한 흐름과 짧은 인터렉션으로 디자인되어있습니다. 이미지는 기기의 사이즈에 따라 크기가 조정됩니다. 일반적으로 페어링된 iPhone 에 동기화됩니다. 

<img width="700" alt="40" src="https://user-images.githubusercontent.com/69136340/178443558-17a4cc26-76bf-4763-af3d-24d0528ce0ca.png">

그러나 Family Setup(가족 설정) 을 사용하면 자신의 iPhone 이 없는 가족 구성원이 Apple Watch 의 기능과 이점을 즐길 수있습니다. 기기가 Family Setup 모드인 경우 picker 를 사용하여 iCloud Photos 에 있는 최근 1000개의 이미지를 선택할 수 있습니다. picker 는 인터넷에서 일부 이미지를 다운로드해야하고, 이 경우 picker 가 닫히기 전 loading UI 가 보여집니다.

<img width="700" alt="41" src="https://user-images.githubusercontent.com/69136340/178443571-cd7d86da-8101-4430-a11b-918a9e5332b3.png">

여러분이 가시기 전에 앱이 사지과 비디오에 액세스할 수 있는 가장 좋은 방법으로 systme Photos picker 를 만들기 위해 최선을 다하고 있다는 것을 말씀드리고 싶습니다. 당신이 커스텀 picker 를 여전히 사용하고 있는 경우 전환하는 것이 좋습니다. 감사합니다!

<img width="700" alt="42" src="https://user-images.githubusercontent.com/69136340/178443614-97ae64da-c539-4986-8f5f-76f63a7594e5.png">
