---
title:  "WWDC22) Discover PhotoKit change history"
categories:
- iOS

date:   2022-09-29  22:23:00 +0900
author_profile: false
---
[Discover PhotoKit change history - WWDC22 - Videos - Apple Developer](https://developer.apple.com/videos/play/wwdc2022/10132/)

***본 글은 WWDC 를 보고, 번역 및 요약 그리고 실행해보는 스터디 프로젝트의 일환입니다.***

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/193038873-96cee48b-3a4f-4929-bb00-c6bd96b7c31c.png">

- 앱에서 Phots 변경 기록에 접근하는 방법에 대해 알아보겠습니다.

<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/193037464-7ad69528-768a-4370-b394-045cc1f6d121.png">

PhotoKit 은 사진 라이브러리에 저장된 사진, 영상 및 앨범에 접근하고 업데이트하기 위해 풍부한 API 를 제공합니다. PhotoKit 은 깊은 수준으로 Photos 접근과 통합이 필요한 앱을 위해 설계되었습니다.

<img width="700" alt="3" src="https://user-images.githubusercontent.com/69136340/193039046-e0d94aba-1b08-4007-b981-4b94ce7585a0.png">

사진 관리 또는 편집, 사용자 지정 카메라 앱이나 사진 라이브러리를 독특하게 탐색할 수 있는 방법을 제공하는 앱들이 있을 것입니다.

이런 유형의 앱들은 사진 라이브러리가 시간이 지남에 따라 어떻게 변경되는지 모니터링해서 Photos 의 경험을 자세히 반영할 수 있습니다. 

예를 들어, 소셜 하이킹 앱을 만들었다고 해보겠습니다. 친구와 하이킹 다녀온 사진을 공유하고, 편집하는 앱입니다.

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/193039791-18ce838a-7dc6-4748-9249-41174b055a8a.png">

이 앱은 산에서의 경험을 콜라주(위와 같이 화면을 구성하는 기법)를 만들기 위해 최근 하이킹 운동의 시작과 끝 타임스탬프에서 사진을 모읍니다. 이는 사진 라이브러리에서 선택한 사진과 동기화된 상태로 유지됩니다. 예를 들어, 친구에게서 하이킹 사진을 받는다면 앱은 이 업데이트를 사용하여 새로운 콜라주를 생성할 것입니다.

<img width="700" alt="5" src="https://user-images.githubusercontent.com/69136340/193039699-4b5f0590-a0c5-4476-971a-bc28a64d5071.png">

지금까지 앱이 새로 삽입된 에셋과 이전 하이킹 콜라주의 변경사항을 발견하려면 열련의 fetch 를 수행해야 했습니다.

앱은 마지막 앱 실행 날짜보다 늦게 생성된 에셋을 가져올 수 있습니다. 에셋 업데이트와 삭제를 결정하는 건 더 어려웠습니다.

<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/193039826-2ae12e13-96bb-4e18-bd41-4848e6fe9ad0.png">

- 업데이트 : 앱은 모든 콜라주에 있는 에셋을 refatch 하고, 에셋 업데이트를 위해 수정 날짜를 확인해야 하는데 Photos 내부의 처리 활동 때문에 에셋 수정 날짜가 바뀔 수 있기 때문에 잘못 탐지할 수 있습니다.
 
<img width="700" alt="7" src="https://user-images.githubusercontent.com/69136340/193039905-54927e3d-5a85-45dd-8e07-a71dbbd70193.png">

- 삭제 : 추적된 모든 에셋을 가져와야하고, 반환되지 않은 에셋과 비교해 무엇이 삭제되었는지 알아야하기 때문에 더 어렵습니다.

전체적으로 앱을 실행할 때마다 세 가지 검사를 별도로 해야하는데 만약 앱이 많은 양의 에셋을 표시하는 경우 특히나 비용이 많이 듭니다.

<img width="700" alt="8" src="https://user-images.githubusercontent.com/69136340/193039958-2bfd33a3-ae24-437e-8c99-5c1f6331e780.png">

**불확실한 결과를 일일이 가져오고 확인을 하는 대신 API 호출로 무엇이 변했는지 정확하게 아는 방법이 있다면 어떨까요?**

새로운 변경 내역 API 를 통해 쉽게 사진 라이브러리의 오프라인 업데이트를 추적할 수 있습니다. 

<img width="700" alt="9" src="https://user-images.githubusercontent.com/69136340/193040291-1950ca2a-c7ac-470c-af9f-17bfd296418e.png">

# 1️⃣ New change history API

변경 내역은 사진 라이브러리에 대한 삽입, 업데이트, 삭제 처럼 변경 타임라인으로 구성됩니다.

<img width="700" alt="10" src="https://user-images.githubusercontent.com/69136340/193040359-218a5fa5-c8a9-443b-8fa5-c8d83727a9e4.png">

위의 타임라인 예시를 보면 3일간의 변경 내역에 에셋, 앨범, 폴더가 변경되었습니다. 어떻게 변경 사항이 발생한 것을 알 수 있을까요?

**👾 : persistent change token 을 사용하면 됩니다!**

<img width="700" alt="11" src="https://user-images.githubusercontent.com/69136340/193040418-937023a2-48b8-4bd5-b197-157904bcb5ec.png">

- 특정 시점의 사진 라이브러리의 상태를 나타내느 토큰입니다.
- 토큰은 앱을 실행해도 유지됩니다.
- third-party app 변경사항을 포함하여 해당 토큰 이후 발생한 변경사항을 가져오는데 사용할 수 있습니다.
- 앱이 제한된 라이브러리 모드일 땐 user-selected PhotoKit 객체들에 대한 변경 사항만 반환됩니다.
- 토큰은 장치에 로컬이며 언제나 사진 라이브러리 인스턴스에서 접근할 수있습니다.

<img width="700" alt="12" src="https://user-images.githubusercontent.com/69136340/193040550-9f991ee0-d68b-4f08-81a2-d5433a955ede.png">

 새로운 API 는 PhotoKit 을 지원하는 모든 플랫폼에서 사용 가능합니다. 

<img width="700" alt="13" src="https://user-images.githubusercontent.com/69136340/193040600-4516216a-430a-411e-804a-1c9b92ccf129.png">

앱이 실행되고 사진 라이브러리와 함께 작동하면 여러분은 persistent change token 을 앱 내에 저장할 수 있습니다. 

👉 **Photos 객체에 대한 변경 세부 정보를 세 가지 유형으로 가져올 수 있습니다.**

<img width="700" alt="14" src="https://user-images.githubusercontent.com/69136340/193040645-134746fb-68b1-448c-a45c-ba233594d3cd.png">

👉 **코드로 살펴보겠습니다.**

- 마지막으로 저장된 token 를 사용해 persistent changes 를 fetch 합니다.

<img width="700" alt="15" src="https://user-images.githubusercontent.com/69136340/193040715-c9f2d21f-1454-4a6f-a4e6-cc6a47bbc981.png">

- persistent changes 를 열거하고 변경 세부사항에 대해서 가져옵니다.

<img width="700" alt="16" src="https://user-images.githubusercontent.com/69136340/193040785-7101e16f-1281-4d8c-a94c-5c9eef55bf72.png">

- 이런 변경 정보는 updated, deleted, inserted local identifiers 를 제공합니다.

<img width="700" alt="17" src="https://user-images.githubusercontent.com/69136340/193040884-6344b37b-1f01-4d97-a7e0-7dd34ef5730b.png">

- 변경사항을 처리하고, 마지막 변경 토큰을 저장해서 나중에 사용할 수 있습니다.

👉 **새로운 persistent history API 와 기존의 change observer API 를 비교해 보겠습니다.**

<img width="700" alt="18" src="https://user-images.githubusercontent.com/69136340/193040946-92179ed8-7f91-41ef-ac8b-86653a3d92be.png">

- PHChanges 는 메모리 내 fetch 결과를 처리해서 앱이 실행되는 동안 사진 라이브러리에 실시간 변경 사항을 기록하는데 사용됩니다.
- 반면에, persistent history 는 장기적으로 실행 중인 변경사항을 사진 라이브러리에 기록하고, 앱이 활성화가 끝난 이후의 변경사항을 보고하는데 사용할 수 있습니다.

여러분들은 요구사항에 따라 위의 API 를 사용할 수 있습니다. 하이킹 앱으로 돌아가 봅시다.

<img width="700" alt="19" src="https://user-images.githubusercontent.com/69136340/193041041-a5953f34-a245-4284-a9b1-22707af7eb9b.png">

이제 하이킹 콜라주를 만들고 업데이트 하기 위해 persistent change API 를 사용해서 에셋을 추적하려고 합니다.

- 먼저, 마지막으로 저장된 토큰을 사용해 persistent change 를 fetch 합니다. (1-2)
- 다음으로 persistent changes 를 반복해 관련 에셋 변경 정보를 파악한 후 updated, deleted, inserted local identifiers를 처리합니다. (3)

이제 변경 정보에서 앱에 영향을 미치는 라이브러리 변경 사항을 식별해야 합니다. 이미 우리는 updated, deleted, inserted local identifiers 를 식별했습니다.

👉 **어떻게 이걸 반영해 업데이트 할까요?**

- insert

<img width="700" alt="20" src="https://user-images.githubusercontent.com/69136340/193041464-8d0ec558-1fa1-494f-b015-330c6152fbf9.png">

insertedIdentifiers 를 사용해서 삽입된 에셋을 가져오고 어떤 에셋이 삽입되었는지 알 수 있습니다.

<img width="700" alt="21" src="https://user-images.githubusercontent.com/69136340/193041512-6f09c36e-33ed-421a-934a-47fa232d0b44.png">

하이킹 시작과 종료와 생성 날짜를 비교하며 하이킹 타임스탬프 사이의 변화를 확인할 수 있습니다.

- update

<img width="700" alt="22" src="https://user-images.githubusercontent.com/69136340/193041554-0f86284d-7f27-4fed-aafa-c28e1adc68a4.png">

업데이트 된 에셋들에 수정이 있을 수 있으니 `hasAdjustments` API 를 사용해 UI 에 에셋을 다시 그려야하는지 확인할 수 있습니다.

- delete

<img width="700" alt="23" src="https://user-images.githubusercontent.com/69136340/193041631-c44fb90d-0652-4cfa-9ea3-eb8f0e96a5c2.png">

deleteIdentifier 를 사용해 재생성이 필요한 콜라주를 결정할 수 있습니다.

**👾 : 이제 변경 사항은 모두 처리했고, 앱은 최신 상태입니다.**

새로운 API 를 사용할 때 주의해야 할 사항이 몇 가지 있습니다.

## ✅ Considerations

<img width="700" alt="24" src="https://user-images.githubusercontent.com/69136340/193041770-2ec5b500-19dd-406f-b2f0-49073754d685.png">

- 무엇이 중요한 변경사항인지 결정하고 그러한 변경사항만 체크하는 것입니다.

업데이트, 삽입된 에셋에 대해 성능 향상을 보이기 위해서는 여러 개의 작은 요청 대신 하나의 큰 fetch 가 좋습니다.

- Photo 라이브러리는 보이지 않는 곳에서 처리 및 동기화로 많이 변경될 수 있기 때문에 앱을 자주 실행하지 않는 경우는 많은 변경 사항을 열거하게 될 수 있습니다.
- 이때문에 우리는 UI 가 차단되지 않도록 background 스레드에서 변경 내역 요청을 권장합니다.

## ✅ Handling errors

persistent changes 를 fetch 할 때 오류 두 가지가 있습니다. 

<img width="700" alt="25" src="https://user-images.githubusercontent.com/69136340/193041835-bf1e7ae6-8a5c-4410-ab12-68907a9af19e.png">

- change token 이 가능한 변경 내역보다 오래된 경우는 만료된 change token erorr 가 반환됩니다.
- 어떤 경우에는 persistent change 가 발생한 변경 사항을 완전히 재구성했는지 확신할 수 없고, 변경 세부 정보를  쓸 수 없다는 에러를 반환할 수 있습니다.

<img width="700" alt="26" src="https://user-images.githubusercontent.com/69136340/193041880-fb152b5f-89c0-4e61-9948-84a014c49e03.png">

위의 경우 앱이 최신 상태인지 확인하기 위해 사진 라이브러리에서 추적된 객체를 refetch 하는 것을 권합니다. 

# 2️⃣ Additional PhotoKit APIs

새로운 PhotoKit API 가 몇 가지 더 있습니다.

<img width="700" alt="27" src="https://user-images.githubusercontent.com/69136340/193041955-d11abd36-d36d-412a-a9cb-286d803bbe18.png">

<img width="700" alt="28" src="https://user-images.githubusercontent.com/69136340/193042018-d0f1fd12-7eae-4202-ba33-265abb13518f.png">

PhotoKit 은 미디어 하위 유형 및 스마트 앨범에서 cinematic 비디오에 접근을 지원합니다. 

👉 **여기에 새로운 오류 코드도 두 개 있습니다.**

<img width="700" alt="29" src="https://user-images.githubusercontent.com/69136340/193042274-9bef5a6a-5900-4232-9cd9-810e600f0ae8.png">

- 사진 라이브러리 번들이 macOS 의 File Provider sync root directory 에 있는 경우 라이브러리가 손상될 수 있고, 변경 작업을 수행할 때 오류가 반환됩니다.
- 네트워크 문제로 에셋 리소스를 찾을 수 없는 경우에는 요청에 네트워크 오류가 반환됩니다.

**👾 : Photos picker 관한 올해 세션도 있습니다!**

<img width="700" alt="30" src="https://user-images.githubusercontent.com/69136340/193042325-72886210-ea14-403e-adbe-8ec770e0ba66.png">

<img width="700" alt="31" src="https://user-images.githubusercontent.com/69136340/193042375-8ec607a3-67a8-4d6c-8042-652ae1fc5f41.png">
