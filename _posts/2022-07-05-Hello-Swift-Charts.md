---
title:  "iOS) Hello Swift Charts"
categories:
- iOS

date:   2022-07-01  04:37:00 +0900
author_profile: false
---
[WWDC22 - Hello Swift Charts](https://developer.apple.com/videos/wwdc2022/?q=hello)

****본 글은 WWDC 를 보고, 번역 및 요약 그리고 실행해보는 스터디 프로젝트의 일환입니다.***

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/177300909-841eb2dd-71e4-49f5-8124-7c7d194f8088.png">

SwiftUI 에서 차트를 만들기 위한 Apple 의 새로운 프레임워크입니다. 

(iOS 16.0+, iPad 16.0+, macOS 13.0+, Mac Catalyst 16.0+, tvOS 16.0+, watchOS 9.0+ 부터 사용이 가능합니다.)

Apple 에서는 시각화를 위한 모범 사례를 연구하는데 수년을 보냈습니다. 차트는 특정 시간 범위에 대한 트랜드와 주가 변동, 마지막 운동 중 심박수, 저녁 시간에 시원해질 때와 같은 데이터에 대한 추가적으로 유용한 컨텍스트를 표시할 때 가장 잘 작동하는 것을 배웠습니다.

<img width="700" alt="2" src="https://user-images.githubusercontent.com/69136340/177300942-d0590041-a5c9-4288-9547-0834230524e5.png">

그리고 이것들은 모든 플랫폼의 많은 예 중 일부일 뿐입니다.

<img width="700" alt="3" src="https://user-images.githubusercontent.com/69136340/177301034-edba3825-6ab3-405c-abb8-24d9164de1ce.png">

여러분에게 앱에서 유익하고 즐거운 차트를 만들 수 있도록 새로운 프레임워크를 소개하게 되어 기쁩니다.

## Say hell to Swift Charts

<img width="700" alt="4" src="https://user-images.githubusercontent.com/69136340/177301114-226daa2c-5ef8-4710-921d-e17c096ebbba.png">

Swift Charts 는 Apple 이 디자인한 차트를 만들기 위한 유연한 프레임워크입니다. SwiftUI 와 동일한 선언적 구문을 사용하므로 이미 Swift Charts 의 언어를 알고 있습니다. 

오늘은 Swift Charts 를 사용하여 팝업 팬케이크 푸드 트럭이 앱으로 판매를 추적하는 것을 돕는 차트를 만들어 보겠습니다. 트럭은 cachapa(카차파), injera(인제라), crepe(크레이프), jian bing(지안빙), dosa(도사), american pancake 제공합니다.
<img width="700" alt="5" src="https://user-images.githubusercontent.com/69136340/177301194-c0df47a9-78e7-41c5-8cb6-ed90a777bc2c.png">

푸드트럭은 지난 30일 동안 4500개 이상의 팬케이크를 제공했습니다. cachapa 가 가장 인기가 있었고 앱은 이미 제목에서 가장 중요한 정보를 보여줍니다. 

<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/177301226-9310ca56-1989-4ef7-9e42-e1a98cb4af81.png">

여섯 개의 팬케이크에 대한 자세한 분석을 보여주는 차트를 추가해 보겠습니다.

<img width="700" alt="7" src="https://user-images.githubusercontent.com/69136340/177301301-41ea3cfa-ef6d-4aac-86ba-0493f60946a5.png">

Swift Charts 에서는 구성별로 차트를 작성합니다. 막대 차트의 주요 구성 요소는 `bar(막대)` 입니다.   이러한 시각적 요소를 `marks` 라고 부릅니다. Xcode 로 이동해서 차트를 만들어 봅시다.

## Jump into Xcode

Charts 를  추가하는 것으로 시작합니다. bar 를 만들기 위해서 BarMark 를 Chart 안에 추가합니다. bar 를 cachapa 의 수대로 표시하기 위해서 이름과 판매량을 설정해야 합니다.

우리는 팬케이크의 이름에서 값이 파생되도록 bar 의 x위치를 설정했습니다.

<img width="700" alt="8" src="https://user-images.githubusercontent.com/69136340/177301654-899cf52a-9c0e-4264-b450-cc8d3e2d7502.png">

`.value` 메서드의 첫 번째 인수는 값에 대한 설명이고, 두 번째 인수는 값 자체입니다. 이제 preview 에 단일 막대가 표시됩니다. 

<img width="700" alt="9" src="https://user-images.githubusercontent.com/69136340/177301678-1126c8d7-e705-4fb9-98d0-07b2b54bda02.png">

y 속성으로 판매된 cachapa 의 수가 설정되어야 합니다. .value 를 사용하여 막대의 길이를 나타내는 x축과 y축의 레이블도 자동으로 생성합니다. 두 번째 막대를 추가해 보겠습니다.

<img width="700" alt="10" src="https://user-images.githubusercontent.com/69136340/177301716-5acd6af0-81cc-4ae3-bde9-f3f522c7b516.png">

개별 mark 를 만들고 앱에 표시되는 것을 보니 좋네요! 하지만 일반적으로 구조체 배열과 같은 컬렉션으로 구동되는 차트를 만들고 싶습니다. 먼저, 팬케이크 판매를 위한 구조체를 추가하는 것으로 시작해보겠습니다.

우리는 반복적으로 사용하고 싶기 때문에 Identifiable 로 가능하게 합니다. 그리고 name 을 반환하는 id 연산 프로퍼티를 정의합니다. 

<img width="700" alt="11" src="https://user-images.githubusercontent.com/69136340/177301749-bcc2ccb5-d63a-4bac-a5ae-1b8e29e06482.png">

이제 데이터 세트로 팬케이크 배열을 만들 수 있습니다. 외부 데이터 소스에서 로드할 수 있지만 여기서는 코드에서 정의하겠습니다. cachapa 와 injera 외에 crepe 도 추가하겠습니다. `ForEach` 를 사용하여 막대 차트를 만들 수 있습니다.

<img width="700" alt="12" src="https://user-images.githubusercontent.com/69136340/177301799-421767ba-977f-4dc5-a78d-643a89e4e497.png">

`ForEach` 가 차트의 유일한 컨텐츠인 경우 데이터를 Chart 이니셜라이저에 직접 넣을 수도 있습니다.

이제 jian bing, dosa 및 american pancake 에 대해 추가해보겠습니다.

<img width="700" alt="13" src="https://user-images.githubusercontent.com/69136340/177301818-4f267825-7652-472c-bf1f-3d2d0ad731e4.png">

레이블들이 서로 가까워지고 있는 것을 볼 수 있습니다. x 와 y 를 교환하여 각 bar 에 더 많은 공간을 줄 수 있습니다. Swift Charts 프레임워크는 자동으로 적절한 축 스타일을 선택하여 차트를 아름답게 만듭니다.

<img width="700" alt="14" src="https://user-images.githubusercontent.com/69136340/177301859-61bce000-412d-490c-861d-baefbca2a942.png">

그리고 Xcode 의 새로운 변형 기능을 사용해서 다크 모드와 다양한 글꼴 크기, 장치 크기 및 방향에 적응하고 accessibility 를 지원한다는 것을 알 수 있습니다.

<img width="700" alt="15" src="https://user-images.githubusercontent.com/69136340/177301904-b717bd3f-f38f-4807-8dfa-941aacc1c361.png">

시각적 표현의 데이터에 액세스하려면 볼 수 있어야 합니다. Swift Charts 는 시각화의 데이터를 `VoiceOver` 에 노출하여 모든 사람이 인기 있는 팬케이크의 세부사항을 탐색할 수 있도록 제공합니다.

- 7분 35초에서 확인할 수 있습니다.

<img width="700" alt="스크린샷 2022-07-05 오후 6 58 18" src="https://user-images.githubusercontent.com/69136340/177302891-fdba3999-0f53-4347-b0c5-c17ff25df671.png">

오른쪽으로 스와이프하며 순차적으로 막대 그래프의 정보를 들을 수 있다.

이 차트는 Apple 이 2021 에 발표한 sonifications(음양화) 를 포함하여 Audio Graphs 를 지원합니다.

- 7분 55초에서 확인할 수 있습니다.

<img width="700" alt="스크린샷 2022-07-05 오후 6 59 26" src="https://user-images.githubusercontent.com/69136340/177303135-e0fbbce3-9d74-4776-88dd-2501b631a07b.png">

play audio graphs 를 통해 비프음의 높이에 따라서 정보를 들을 수 있다.

우리는 프드 트럭 앱에 정보를 제공하고 접근 가능한 차트를 추가하기 위해 Swift Charts 를 사용했습니다. 차트는 여러가지 스타일의 팬케이크를 얼마나 판매했는지 보여줍니다. 

<img width="700" alt="16" src="https://user-images.githubusercontent.com/69136340/177303349-6c6cedaa-a2a7-4be9-9b17-11e1a9b74569.png">

각 스타일에 대한 요약 외에도 푸드 트럭에는 일별 판매 데이터도 있습니다. 

<img width="700" alt="17" src="https://user-images.githubusercontent.com/69136340/177303355-9de1bbd3-a6e2-4136-b3d7-685602b58aa3.png">

트럭은 Cupertino 와 San Francisco 에 주차할 수 있습니다. 우리는 차트를 사용하여 팬케이크 트럭이 주중에 주차할 위치를 결정하는데 도움이 되고자 합니다. 이 질문에 답하기 위해 데이터를 두 도시의 time series 를 시각화해 보겠습니다.

<img width="700" alt="18" src="https://user-images.githubusercontent.com/69136340/177303415-a229db60-413f-47b4-9535-ad7794742e46.png">

앞으로 다양한 디자인을 탐색하는 것이 얼마나 쉬운지 차트를 세번 반복해보겠습니다.

- Bar graph for Cupertino
- Add the data for San Francisco, add a Picker
- Finally, combine the data into one multiseries line chart

<img width="700" alt="19" src="https://user-images.githubusercontent.com/69136340/177303692-e5e470e2-e677-4f59-a71a-e651e00d9a90.png">

### Bar graph for Cupertino

요일별로 판매된 판케이크의 평균 수를 보여주는 bar 차트를 만들겠습니다.

<img width="700" alt="20" src="https://user-images.githubusercontent.com/69136340/177303712-565c456b-9bcd-4e63-8ac0-6ee794541736.png">

판매 데이터에는 날짜로 저장된 요일과 판대된 팬케이크의 수가 있습니다. 

<img width="700" alt="21" src="https://user-images.githubusercontent.com/69136340/177303722-c42ab028-82f6-41f9-aac1-b83bbddd9ec4.png">

날짜에서 파생되도록 막대의 x 위치를 설정했습니다. 판매에서부터 파생되도록 높이를 설정해보겠습니다.

### Add the data for San Francisco, add Picker

두 번째 반복에서는 San Francisco 데이터를 추가해 보겠습니다. 샌프란시스코의 판매 데이터는 sfData 변수에 있습니다. 우리는 두 도시 사이를 toggle 하고 각 도시에 대한 막대 차트를 보고 싶습니다.

state 변수에 city 를 추가하는 것으로 시작합니다.

<img width="639" alt="22" src="https://user-images.githubusercontent.com/69136340/177303787-88910e56-4e2e-406d-a74e-260e8b79a401.png">

그런 다음 SwiftUI picker 를 추가합니다. 그리고 데이터를 Cupertino 와 San Francisco 사이를 전환하는 데이터로 바꾸는 것입니다. 토글을 전환하면 차트가 두 도시 간에 전환됩니다.

<img width="639" alt="23" src="https://user-images.githubusercontent.com/69136340/177303809-ed2624dd-cb68-43e7-a2b8-25c34cd37894.png">

Swift Charts 는 SwiftUI 애니메이션과 함께 작동하므로 easyInOut 으로 전환 애니메이션을 지정할 수 있습니다.

<img width="638" alt="24" src="https://user-images.githubusercontent.com/69136340/177303895-87fcd9ea-373f-4bb2-ba06-195454d6911a.png">

### Finally, combine the data into one multiseries line chart

마지막 반복으로 두 시리즈를 모두 꺽은 선형 차트로 표시할 것입니다. 두 도시의 데이터는 일련의 배열로 되어 있습니다. 두 시리즈를 모두 보여주기 전에 Cupertino 데이터에 중점을 두겠습니다.

<img width="700" alt="25" src="https://user-images.githubusercontent.com/69136340/177303947-760d399a-7f9f-4c83-864c-e5a6829bcc9f.png">

Chart 이니셜라이저는 ForEach 처럼 작동할 수 있다고 했습니다. 그런 다음 Cupertino 에 대한 특정 데이터를 Series 의 sales data 로 바꿀 수 있습니다.

<img width="700" alt="26" src="https://user-images.githubusercontent.com/69136340/177303958-04882ce4-6f2c-41ed-835f-f4b43a565deb.png">

두 도시의 데이터를 구분하기 위해서 bar 의 색상을 정할 수 있습니다. 이를 표시하기 위해서 Swift Charts 는 차트 아래에 범례를 만듭니다.

<img width="700" alt="27" src="https://user-images.githubusercontent.com/69136340/177304010-7bb80354-d21a-4f8d-9d21-518cf0ae7250.png">

San Francisco 데이터를 추가해 보겠습니다.

<img width="700" alt="28" src="https://user-images.githubusercontent.com/69136340/177304026-6eb121be-2b81-4357-9eef-a6c0417b52c5.png">

preview 에서 볼 수 있듯이 Swift Charts 는 자동으로 San Francisco 의색상을 정합니다.

Charts 에는 특정 컨텍스트에 대한 데이터가 표시되며 데이터나 질문이 변경되면 시각화도 변경해야 할 수 있습니다. Swift Charts 는 빠르게 변경하여 다양한 디자인을 탐색할 수 있도록 해줍니다. stacked bar chart(누적 막대형 차트)는 하루 평균 총 판매액을 표시하는데 적합합니다.

하지만, 두 도시를 비교하고 싶다면 point 또는 line chart 가 더 나을 것입니다. mark type 을 BarMark 에서 PointMark 로 변경하거나 LineMark 로 변경할 수 있습니다.

<img width="700" alt="29" src="https://user-images.githubusercontent.com/69136340/177304056-7c8c840e-0eb9-4795-a4c1-b3870014909a.png">

<img width="700" alt="30" src="https://user-images.githubusercontent.com/69136340/177304160-bfe20391-d262-41ec-81d8-84788dde8f75.png">

차트는 여러 mark 를 결합할 수 있습니다. 예를 들어 PointMark 를 추가할 수 있습니다. 

<img width="700" alt="31" src="https://user-images.githubusercontent.com/69136340/177304216-391bd000-200d-406a-8dab-d5048a6eb4fa.png">

series 를 색상 없이 구분할 수 있도록 도시에서 파생된 symbol 을 설정해보겠습니다.

<img width="700" alt="32" src="https://user-images.githubusercontent.com/69136340/177304223-7ae2a6e5-81b2-4776-adf7-748eb4dafb73.png">

이제 각 점은 색상과 기호로 도시를 나타냅니다. 선에 점을 표시하는 것이 일반적이기 때문에 Swift Charts 는 LineMark 에 symbol modifier 를 적용하는 축약형이 있습니다. 점의 스타일은 선에 맞게 조정됩니다.

<img width="700" alt="33" src="https://user-images.githubusercontent.com/69136340/177304279-c68c5308-ab36-4bd4-9879-e8b5eb3ab54c.png">

이 차트는 훌륭합니다. 일주일 동안의 판매 트랜드를 쉽게 비교할 수 있습니다.

Swift Charts 를 사용하여 쉽고 빠르게 반복하는 동시에 차트를 앱의 고유한 스타일에 매끄럽게 통합할 수 있을 만큼 유연하게 만드는 방법에 대해서 다시 살펴보겠습니다.

Swift 에서는 Mark Properties 가 있는 Marks 구성으로 차트를 작업합니다. Pancake app 에서 3개의 다른 mark 와 4개의 mark properteis 로 차트를 구성했습니다.

<img width="400" alt="34" src="https://user-images.githubusercontent.com/69136340/177304329-dea853d1-992f-4905-b772-9189f6b1212c.png">

예를 들어 x 축과 y 축이 있는 bar mak 로 간단한 막대 차트를 만들 수 있습니다.

<img width="700" alt="35" src="https://user-images.githubusercontent.com/69136340/177304513-8e4ab836-db5c-410f-9537-b708a51aa85c.png">

Ehgks, points 또는 line charts 처럼 디자인을 빠르게 탐색할 수 있습니다.

<img width="700" alt="스크린샷 2022-07-05 오후 7 07 08" src="https://user-images.githubusercontent.com/69136340/177304473-791a7aaa-2ed7-48f0-95f0-5eba04f9b198.png">

또한, `foregroundStyle` 과 같은 속성을 추가해서 line chart 의 multiple series 를 보여줄 수 있습니다.

<img width="700" alt="36" src="https://user-images.githubusercontent.com/69136340/177304580-27ae912c-9379-49ca-b15e-4aeceedc6ea7.png">

그리고 chart 는 하나의 mark 만 가질 필요가 없습니다. 우리는 points 와 lines 를 조합했습니다. 

<img width="700" alt="37" src="https://user-images.githubusercontent.com/69136340/177304605-c7cb1174-891b-4098-a3ff-e244c72fb04f.png">

Swift Charts 는 오늘 사용하는 것보다 훨씬 더 많은 marks 와 mark properties 를 지원합니다. 또한, 확장 가능하며 커스텀 marks 를 추가할 수 있습니다.

<img width="700" alt="38" src="https://user-images.githubusercontent.com/69136340/177304692-ac8b17c3-3ddd-499a-864a-362b3f56a4a1.png">

marks 와 mark properties 는 적은 수의 devlarative building blocks 으로 광범위한 차트 디자인을 표현할 수 있습니다.

<img width="700" alt="39" src="https://user-images.githubusercontent.com/69136340/177304736-df2fdc4a-5268-4603-a72a-c83beb72ab05.png">

이러한 building block 을 조합하여 더 거대한 데이터의 시각화를 만들 수 있습니다.

그리고 이미 보여드린 것 처럼 dark mode, different device screen sizes, Dynamic Type, VoiceOver 그리고 Audio Graphs 를 무료로 지원합니다. 추가로 High-Contrast mode(accessibility 대비 설정) 를 지원합니다. 마지막으로 Swift Charts 는 여러 loacle 들에서 작동하며 multiplatform 입니다. 동일한 코드는 Apple 플랫폼들에서 모두 작동합니다. 동일한 customizations 가 모든 곳에서 동작하므로 각 플랫폼에 맞게 조정할 수 있습니다. 

<img width="700" alt="40" src="https://user-images.githubusercontent.com/69136340/177304810-e650ef2d-f57f-4269-98ca-ca54db439c4a.png">

<img width="700" alt="41" src="https://user-images.githubusercontent.com/69136340/177304836-197b923c-a768-4de7-96e1-703e5305b0d5.png">

