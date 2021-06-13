---
title:  "iOS) User Defaults vs Core Data"
categories:
- iOS

date:   2021-06-13 19:46:00 +0900
author_profile: false
---
# User Defaults vs Core Data

좋은 자료를 찾아서 해석해봤습니다

데이터를 저장한다는 것을 제외하고는 persistence solution 이 전혀 다른 User Defautls 와 Core Data!

### 🤨 User Defaults 는 언제 사용하나요?

설정이나 사용자의 기본 설정과 같은 데이터의 small data chunks 를 저장하는 데 이상적이다. property list 또는 plist 로 디스크에 저장된다. property list 와 plist 는 XML 파일 형태이다.

`UserDefaults` 클래스는 성능 향상을 위해서 런타임에 메모리에 내용을 저장한다.

key-value 저장소일뿐이다. 이렇게하면 쉽게 액세스할 수 있지만 key-value pairs 가 서로 명시적 관계가 없음을 의미하기도 한다.

### ❗️User Defaults 의 단점

- 대용량의 데이터 세트를 저장하는데에 적합하지 않다.

User Defaults 는 디스크에서 로드하고 메모리에 저장하기 때문에 데이터베이스가 작고 관리하기 쉬운 경우에 효율적이고 성능이 좋다.

- 이미지 데이터와 같이 설계되지 않은 다른 유형의 데이터를 저장하는데는 적합하지 않다.

User Defaults 는 Strings, numbers, Date objects, Data objects 자료형을 지원한다. 만약에 이미지 데이터를 저장하기 위해서는 Data 자료형으로 변환을 해주어야 한다.

### 🤨 Core Data 는 언제 사용하나요?

Core Data persistence store 의 데이터는 XML 파일 또는 SQLite 데이터베이스로 저장할 수 있다. 데이터를 메모리에 저장하거나 프로젝트 요구사항에 맞는 cutom persistent store 도 만들 수 있다. 

Core Data 의 다른 주요 이점은 관계형 데이터세트 지원이다. (Xcode 에는 data model editor 가 있다.) entities 를 정의하고 entities 간의 relationship 을 정의할 수 있다.

Core Data 는 앱의 요청에 따라서 필요한 정보만 가져온다. 이것은 User Defaults 와 크게 다른 부분이다. User Defaults 클래스는 성능향상을 위해서 적절하게 변경사항을 디스크에 비동기식으로 기록한다.

### ❗️Core Data 의 단점

- 러닝커브가 가장 큰 단점.
- 가벼운 User Defaults 와 비교해서 약간의 오버헤드가 있다.
- 대규모의 관계형 데이터를 저장하는 데 유효하지 임의의 관계없는 데이터를 저장하는데는 비효율적이다.

### keychain!

혹시나 API에 대한 암호 또는 액세스 토큰과 같은 민감한 정보를 저장해야하는 경우 keychain 이 적합하다.

### 출처

[Should You Use Core Data Or User Defaults](https://cocoacasts.com/ud-10-should-you-use-core-data-or-user-defaults)
