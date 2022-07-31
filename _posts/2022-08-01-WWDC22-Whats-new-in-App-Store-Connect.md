---
title:  "WWDC22) What's new in App Store Connect"
categories:
- iOS

date:   2022-08-01  01:02:00 +0900
author_profile: false
---
[What's new in App Store Connect - WWDC22 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2022/10043/)

***본 글은 WWDC 를 보고, 번역 및 요약 그리고 실행해보는 스터디 프로젝트의 일환입니다.***

### 소개하기

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/182034693-ce11566f-6717-46e3-b7a3-37a377c0b475.png">

[App Store Connect API - Apple Developer](https://developer.apple.com/kr/app-store-connect/api/)

<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/182034696-569ce8fc-5ec8-403b-9903-0be5c1993371.png">

app store connet 는 지난 몇 년간 성장하여 app store 의 모든 플랫폼에 걸쳐 앱을 생성하고, 관리하며 확장할 수 있습니다. 또한 지속적으로 App Store Connect 의 새로운 기능을 웹, iOS, iPadOS app 그리고 App Store Connect API 에 추가하고 있습니다. 작년에 출시된 기능인 in-app events(앱 내 구입) 인 맞춤형 제품 페이지, Mac 용 TestFlight 를 들어보셨을 것입니다.

## Recent Updates

여러분들이 놓쳤을지 모르는 최근 몇 가지 업데이트를 빠르게 알려드리겠습니다.

<img width="700" alt="3" src="https://user-images.githubusercontent.com/69136340/182034710-37fdeb48-9407-4976-81f6-a64e123ee370.png">


**TestFlight 로 테스트 그룹의 관리가 쉬워졌습니다.** 한 번의 클릭만으로도 신속하게 versions 이나 build groups 탭에서 직접 빌드에 테스터 그룹을 추가하거나 삭제할 수 있습니다.

**Apple Wallet 을 사용하는 앱을 전환할 수 있도록 업데이트 했습니다.**

**제출 과정이 앱 내 이벤트, 맞춤형 제품 페이지와 제품 페이지 최적화에 용이하도록 App Store 제출 환경을 출시하였습니다**. 이는 제출 워크플로우상의 주요 개선 사항입니다.

### 이제 조금 더 상세하게 알아 볼까요?

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/182034713-140fede7-a2f5-4598-b562-7fe9167d5741.png">

여러분은 다수의 항목은 그룹화하여 단일 제출할 수 있습니다.

새로운 앱 버전 없이도 제출을 선택할 수 있습니다.

전용 심사 페이지도 출시되었습니다. 이곳에서는 진행 중인 제출물을 관리하고 앱 심사와 커뮤니케이션하며 최근 완려된 제출물도 볼 수 있습니다.

### 1. Multiple items in one submission

제출 심사에서 항목을 그룹화하는 것이 어떤 의미인지 알아보겠습니다.

여러분이 여러 개의 custom product pages(맞춤형 제품 페이지)나 심사 항목을 스토어에 게시한다고 가정하겠습니다.

제출 심사의 일부로만 항목을 제출할 수 있기 때문에 첫 단계로 할 일은 이를 제출에 추가하는 것입니다. 
<img width="700" alt="5" src="https://user-images.githubusercontent.com/69136340/182034723-2896a249-a978-4e44-a5da-0edaf80fec35.png">

심사 제출은 앱 심사를 오가며 심사 항목들을 운반하는 차량과 같습니다. 여러 개의 항목을 그룹화하면 콘텍스트에서 모두 함께 심사받을 수 있는 장점이 있습니다.

이를 통해 일관성있고, 효율적인 심사가 가능합니다. 사실상 모든 심사 제출은 일반적으로 24시간 이내 항목의 수량과 유형에 상관없이 심사됩니다.

심사가 끝나면 각가의 항목은 수락 또는 거절로 표시됩니다.

<img width="638" alt="6" src="https://user-images.githubusercontent.com/69136340/182034875-ddd7238d-42f0-4ced-b1e4-9c82ed337b08.png">

항목이 하나라도 수락되지 않으면 모든 검토 항목이 승인되지 않습니다.

**제출물 중 거절된 항목이 있는 경우 계속 진행하는 두 가지 방법을 살펴보겠습니다.** 첫 번째는 거절된 심사 항목들을 편집하여 재제출하는 것입니다.

<img width="636" alt="7" src="https://user-images.githubusercontent.com/69136340/182034873-8b864074-ad57-4ff9-92ff-3fbab87f837b.png">

두 번째는 거절된 항목을 간단히 삭제하는 것입니다. 이렇게 되면 승인된 항목만 남게 되고 다시 한 번 심사 과정이 완료됩니다.
<img width="700" alt="8" src="https://user-images.githubusercontent.com/69136340/182034724-eddf398e-8c32-48ef-8d82-6f1868b8ff91.png">

### Review submission items

<img width="700" alt="9" src="https://user-images.githubusercontent.com/69136340/182034727-1690509c-1195-44a0-a6cf-1867f74cc2ce.png">

위의 항목들을 심사 제출할 수 있습니다.

## 2. Submit without a new app version

이것이 어떻게 운용되는지 이해하기 위해 새로운 제출에 대해 몇 가지 말씀드리겠습니다.

첫 째, 각각의 제출은 연계 플랫폼을 가집니다. 

<img width="700" alt="10" src="https://user-images.githubusercontent.com/69136340/182034735-b5853389-ee95-4042-8926-ee0435c58734.png">

또한, 각각의 플랫폼은 특정 심사 항목 세트를 지원합니다.

<img width="700" alt="11" src="https://user-images.githubusercontent.com/69136340/182034739-708065c1-70dd-4525-a62b-82a94d671e3b.png">

대부분의 항목은 iOS 제출의 일부로 심사되고 그룹화됩니다.

마지막으로, 플랫폼당 진행 중인 심사 제출은 하나만 가질 수 있습니다.

<img width="700" alt="12" src="https://user-images.githubusercontent.com/69136340/182034743-912e1439-2e44-4f97-8d00-356e670d2dc6.png">

위는 세 개의 버전의 앱을 모두 제출하려는 시도입니다.

<img width="700" alt="13" src="https://user-images.githubusercontent.com/69136340/182034746-bce9c899-8942-4624-9f6b-04bc86674442.png">

iOS 심사 제출을 자세히 살펴보겠습니다.

앱 심사는 제출물에 있는 모든 항목을 앱 버전과 비교 심사하여 모든 항목이 일치하는지 확인합니다.  제출된 앱 버전이 심사에 사용된 버전인지 확인합니다. 앞서 말했듯이 새 버전을 추가하지 않고 제출할 수 있다고 소개했습니다.

이렇게 하려면 이전에 승인된 버전의 앱이 필요합니다. 물론, 제출을 마치면 이 버전과 항목을 비교 심사합니다.

<img width="700" alt="14" src="https://user-images.githubusercontent.com/69136340/182034749-f75a04cf-146e-4a66-952a-5727b3903e36.png">

이 뜻은 앱 내 이벤트와 맞춤 제품 페이지, 제품 페이지 최적화 테스트를 첫 iOS 버전이 승인된 후 언제나 앱 바이너리 없이 제출할 수 있다는 것을 의미합니다. 

## 3. Dedicated app review submission page

App Store Connect 의 앱 심사 제출 페이지를 보여드리겠습니다.

앱 심사 페이지로 여기서 전체 심사 워크플로우를 관리할 수 있습니다.

<img width="700" alt="15" src="https://user-images.githubusercontent.com/69136340/182034765-551cc173-b4f1-4258-b2d8-d203716da33a.png">

이곳에서 제출 개요를 볼 수 있고, 좀 더 상세한 사항을 확인할 수 있습니다.

물론 웹 UI 를 사용하는 것도 좋지만 이동 중에 제출 상태를 추적하고 제출하면 좋지 않을까요? 그래서 향상된 제출 환경을 iPadOS 와 iOS 에서도 App Store Connect 을 통해 경험할 수 있습니다.

<img width="700" alt="16" src="https://user-images.githubusercontent.com/69136340/182034769-8ec9933d-31d8-4509-b087-c15842a9ad7e.png">

이제 심사 준비를 끝낸 제출물을 한 번의 탭으로 어디서든 제출할 수 있습니다.

<img width="700" alt="17" src="https://user-images.githubusercontent.com/69136340/182034778-2d926839-9042-43dd-9647-90e6785e4a5a.png">

제출 후에는 심사 진행률을 추적할 수 있습니다.

<img width="700" alt="18" src="https://user-images.githubusercontent.com/69136340/182034783-01f4b483-75c4-4d90-bbd3-471f4d7a39a4.png">

항목을 제거하거나 거절 사유를 보거나 앱 심사에 회신하는 등 제출물을 관리할 수 있습니다.

<img width="700" alt="19" src="https://user-images.githubusercontent.com/69136340/182034794-d781377c-0fbd-491b-8656-6ffb03dbf0e2.png">

이것이 향상된 App Store 제출 환경으로 현재 App Store Connect 에서 만나볼 수 있습니다.

### App Store Connect API

이를 통해 앱 워크플로우를 커스텀하고 자동화할 수 있습니다.

🧔‍♀️ : 저희는 항상 API 에 새로운 기능을 추가하기 위해 노력하고 있습니다. 지난 해에는 App Clip, Xcode Cloud, in-app events, custom product pages, product page optimization, 위에서 언급한 향상된 App Store submission experience 를 추가하였습니다.

올 여름 대규모 2.0 출시를 앞두고 API 의 리소스 수를 60% 까지 확장하고 있습니다.

<img width="700" alt="20" src="https://user-images.githubusercontent.com/69136340/182034796-bb486df5-2442-4368-87eb-a5e7134225cc.png">

올 여름 출시에는 쇄도했던 요청 사항들이 반영된 훌륭한 기능들이 몇 가지 추가됩니다.

<img width="700" alt="21" src="https://user-images.githubusercontent.com/69136340/182034814-33a6bfd2-023d-44c1-9632-c9e5a82afc42.png">

## 1. In-app purchases and subscriptions

<img width="700" alt="22" src="https://user-images.githubusercontent.com/69136340/182034950-9245ad1d-174f-4bad-8df3-01d25ad62595.png">

API 종합 세트부터 살펴보면 이를 통해 앱 내 구매 및 구독 라이프사이클 관리를 전체적으로 할 수 있습니다. 

먼저 subscription 을 자체 리소스로 분류하고 앱 내 구매 또는 구독을 생성, 편집, 삭제 할 수 있는 모든 권한을 부여 했습니다. 또한, 가격을 관리하고 심사를 위해 제출하고, 특별 오퍼와 프로포션 코드를 생성할 수 있습니다.

## 2. Customer reviews and developer responses

고객의 평가와 개발자의 응답을 할 수 있는 기능을 추가하였습니다.

목표는 고객 상호작용을 중심으로 좋은 커스텀 워크플로우를 구출할 수 있도록 하는 것입니다.

## 3. App hang diagnostics(앱 중단 진단)

<img width="700" alt="23" src="https://user-images.githubusercontent.com/69136340/182034955-2a0895ea-0b08-4503-a849-58f856e52c2c.png">

추가 reporting data 를 power and performance metrics and dagnostics API 에 추가하였습니다.

지금까지는 API 를 통해서만 앱의 hang rate(중단률) 과 같은 측정 기준을 볼 수 있었지만 올 여름부터 app hang 에 대한 새로운 diagnostic type 이 추가되엇습니다. 기존 diagnostic signatures resource 와 함께 사용하여 앱에서 가장 많이 중단되는 위치를 찾을 수 있습니다. 

뿐만 아니라 diagnostic logs relation ship 을 통해 상세한 추적도 볼 수 있습니다. 

이러한 API 운용 방식이나 이 데이터를 사용하여 앱 양식에 대한 추가적인 인사이트를 얻으려면 반드시 아래의 두 개의 세션을 보면 됩니다. 

### XML feed decommission

<img width="700" alt="24" src="https://user-images.githubusercontent.com/69136340/182034971-52463f3c-58d7-4029-b66c-3a6489966c73.png">

전반적으로 App Store Connect API 2.0 출시는 주요 milestone 입니다. 4년의 개발 끝에 REST API 를 App Store Connect 자동화의 미래로 받아들였습니다.

그 결과 올 가을부터 XML feed 를 폐기하기 시작할 것입니다. 그러므로 App Store Connect API 와 통합을 얼라인하기를 바랍니다. 

<img width="700" alt="25" src="https://user-images.githubusercontent.com/69136340/182034976-ceb8382b-a032-431f-a76f-6c7a9c0d06f4.png">

<img width="700" alt="26" src="https://user-images.githubusercontent.com/69136340/182034983-53d2301a-4f89-4a29-9d59-29648fbb5019.png">

