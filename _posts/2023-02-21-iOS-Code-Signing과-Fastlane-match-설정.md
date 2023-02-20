---
title:  "iOS) Code Signing 과 Fastlane match 설정"
categories:
- iOS

date:   2023-02-21  11:59:00 +0900
author_profile: false
---
### 내용

[iOS) 앱 자동 배포를 위한 Fastlane 설정](https://gyuios.tistory.com/241)

- 위의 글에서 Fastlane 을 설정했었습니다. 이때는 cert and sigh 방법을 통해서 code signing 을 작업하였습니다.
- code signing 은 무엇이 필요하고, match 방법에 대해서 알아보겠습니다.

## ✅ Code Signing(or Signing) 이란?

“***Code signing*(or *signing*) an app allows the system to identify who signed the app and to verify that the app has not been modified since it was signed.” 출처 - [What is app signing?](https://help.apple.com/xcode/mac/current/#/dev3a05256b8)**

코드 서명은 앱에 서명한 사람을 식별하고, 서명된 이후 앱이 수정되지 않았음을 확인할 수 있는 과정입니다.

또한, 이 서명을 통해 App Store Connect 에 업로드하고, TestFlight 또는 App Store 에 배포하기 위한 요구사항입니다. 

**“A *signing certificate* is a digital identity used for code signing during the build and archive process.” 출처 - [What is app signing?](https://help.apple.com/xcode/mac/current/#/dev3a05256b8)**

코드 서명을 위해서 digital identity 가 되는 **signing certificate** 가 사용됩니다.

## ✅ Code Signing 에서는 무엇이 필요할까요?

프로젝트의 **Signing & Capapbilities** 탭을 확인해보면 다음과 같이 Build Scheme 별로(해당 프로젝트는 Debug, Release 가 있음) Bundle Identifier 와 Profile 을 설정해야 합니다.

여기서 **Signing Certificate** 는 **Provisioning Profile** 을 통해서 설정됩니다.

<img width="700" alt="1" src="https://user-images.githubusercontent.com/69136340/220173275-488d6297-c2c1-4896-b539-edb9721d8265.png">

대부분의 경우 저는 **Automatically manage signing** 을 체크하고 사용을 했습니다. 이런 경우에 프로젝트를 만들게 되면 자동으로 app ID 와 profile, certificate 을 생성하고 업데이트하게 됩니다. 말그대로 자동으로 signing 을 관리해주는 것이죠.

이를 해제하게 되면 수동으로 Provisioning Profile 을 설정할 수 있습니다.

정리하자면, Code Signing(코드 서명)을 하기 위해서는 signing certificate(서명 인증서)가 필요하고, 이를 Xcode 에서 설정하기 위해서는 provisioning profile 이 필요합니다.

밑에서 언급하겠지만, provisioning profile 에는 certificate 가 포함되어 있습니다. 그래서 이 certificate 를 통해서 signing certificate(서명 인증서)로 code signing(코드 서명)할 수 있습니다.

결론적으로 iOS 앱을 개발하고 배포하기 위해서 즉, code signing 을 위해서는 **Certificate** 와 **Provisioning Profile** 이 필요합니다.

- **Certificate** : 개발자가 Apple 대신 앱을 실행할 수 있는 권리 허용. 즉, Apple 이 합법적인 개발자로 인정한 것을 증명합니다.
- **Provisioning Profile** : 디바이스가 앱을 신뢰할 수 있는지 확인.

### 👉 Certificate

<img width="500" alt="2" src="https://user-images.githubusercontent.com/69136340/220173346-15e07df9-0f13-4142-8c72-19573290e471.png">

- Run, Archive 를 하든 기본적으로 프로젝트를 빌드할 때는 Apple Development 인증서가 필요합니다.
- App Store 에 제출을 하기 위해서는 Apple Distribution 인증서가 필요합니다.

그렇기 때문에 개발에서 배포까지는 두 가지 인증서가 필요합니다.

인증서를 발급받기 위해서는 CSR(Certificate Siging Request)을 생성하여 Apple Developer 에서 인증서를 다운받을 수 있습니다.

CSR 을 생성하게되면 키체인에서 자동으로 **공개 키**와 **개인 키**를 생성합니다.

- 공개 키는 인증서를 만들기 위한 신청에 필요한 키입니다.
- 개인 키는 인증서와 합쳐져 하나의 **Signing Certificate(서명 인증서)**가 됩니다. 이 서명 인증서를 가지고 **code signing(코드 서명)**을 하게 됩니다.

<img src="https://user-images.githubusercontent.com/69136340/220173372-321483c1-b3f9-4184-b2c5-5dc98db65f9d.png" width ="500">

인증서를 더블클릭해서 설치하면 Keychain 에서 다음과 같이 살펴볼 수 있습니다. 인증서에는 공개 키가 포함되어 있습니다.

<img src="https://user-images.githubusercontent.com/69136340/220173423-ddb43c2f-fca7-4da9-afaf-05784f04a4cd.png" width ="600">

(제가 만든 인증서의 개인키와 타인이 만든 인증서에 대한 개인키가 다르게 표시되는 것을 확인할 수 있습니다.)

### 👉 Provisioning Profile

signing certificate 로 앱의 code signing 을 하게 되면 믿을 수 있는 앱이 됩니다. 하지만, 사용자의 디바이스에서 실행할 수 없습니다.

**다시 말해서 Apple 의 인증서가 아닌 Apple 이 발급한 인증서로 코드 서명한 앱을 디바이스에 설치하기 위해서는 프로비저닝 프로파일이 필요합니다.**

프로비저닝 프로파일에는 생성할 때 설정했던 앱 실행에 필요한 App ID, Certificates, Capabilities, Entitlements 와 디바이스 목록 등이 포함되어 있습니다.

**즉, 특정 기기에서 특정 앱을 실행할 수 있는지 확인하는 정보들이 담겨있다고 생각하면 됩니다.** 

- Xcode 에서 쉽게 확인할 수 있습니다.

<img width="600" alt="5" src="https://user-images.githubusercontent.com/69136340/220173495-47f4d4df-0b25-44c8-b42a-fa39cf931ba6.png">

- 아래 문서를 확인하면 더 자세하게 프로비저닝 프로필을 확인할 수 있습니다.

[Apple Developer Documentation - inside code signing provisioning profiles](https://developer.apple.com/documentation/technotes/tn3125-inside-code-signing-provisioning-profiles)

## ✅ 왜 서명 인증서와 프로비저닝 프로파일을 공유해서 사용하나요?

❓ ***당연할 수 있는 질문을 하나 던지고 생각해보고 지나가도 좋을 것 같습니다.***

팀 프로젝트를 진행할 때면 특정 Capabilities 가 필요한 경우가 있습니다. Push Notifications, Sign In with Apple, HealthKit 등이 있을거에요. 그럴때면 App ID 을 편집해서 Capabilities 를 설정해주어야 합니다. 이 정보를 공유해야겠죠.

또한, Bundle Identifier 가 동일해야 같은 앱으로써 배포할 수 있겠죠. App ID 를 설정할 때 Bundle Identifier 가 필요하고, 이를 통해 Provisioning Profile 을 만들게 됩니다.

개발자마다 서명 인증서가 다르다면 개발자마다 프로비저닝 프로파일을 만들어야할 것이에요.

인증서를 갱신해야될 순간도 올 것이에요.

**이 외에도 동일한 프로젝트 개발을 위해서 엮여있는 많은 이유가 있을 것이고, 서명 인증서와 프로비저닝 프로파일을 공유할 필요가 있습니다.**

## ✅ 서명 인증서를 어떻게 공유할까요?

**“To share a signing certificate with another person on your team, export the signing certificate, and on the other person’s Mac, double-click the exported file to install the signing certificate in the keychain.” 출처 - [What is app signing?](https://help.apple.com/xcode/mac/current/#/dev3a05256b8)**

signing certificate 를 export 해서 다른 개발자 Mac의 키체인에 설치하라고 안내하고 있습니다.

하지만, 많은 개발자와 많은 환경들에 signing certificate 을 직접 전달하는 방법은 관리가 되지 않습니다. 그렇다고 개발자에 맞춰서 인증서를 만들기에는 갯수가 제한되어 있습니다.

***그래서 Fastlane 에서는 두 가지 방법을 지원합니다.***

### match

- private keys(개인 키)와 certificates(인증서) 을 git 레포지토리에 저장하여 개발자의 기기 간에 동기화 가능합니다.
- match 를 시작하려면 기존 certificate 를 취소해야 합니다.
- 보통 private repositary 를 생성하여서 사용하는데 private repositary 에서 인증서만 한번 바꿔주면 되기 때문에 갱신 문제, 계정 문제를 고민할 필요가 없습니다.

### cert and sigh

- 기존 certificate 를 취소하고 싶지않지만, 자동화된 설정을 원하는 경우 cert and sigh 방법이 적합합니다.
- `cert` 는 유효한 certificate 와 private key 가 로컬에 설치되어 있는지 확인하고, `sigh` 는 설치된 certificate 와 일치하는 provisioning profile 로컬에 설치되어 있는지 확인합니다.
- 그리고 `get_certificates` , `get_provisioning_profile` 명령어를 lane 에 추가해주어야 합니다.(로컬에 설치되어 있는지를 판단하고, 인증서와 프로비저닝 프로파일을 생성하여 다운받아줍니다.)

**출처:** [https://docs.fastlane.tools/codesigning/getting-started/](https://docs.fastlane.tools/codesigning/getting-started/)

## ✅ match 설정

아래 문서를 참고해서 진행해보겠습니다.

[match - fastlane docs](https://docs.fastlane.tools/actions/match/)

다음의 목표를 가지고 설정해보겠습니다.

- git private repository 를 사용하여 certificate 와 provisioning profile 을 공유하도록 하겠습니다.
- multiple target 에 대응하도록 하겠습니다.

### 1️⃣ Setup

프로젝트 폴더에서 match 를 사용해서 시작합니다.

```swift
fastlane match init
```

code signing identities 를 Git reop, Google Cloud 또는 Amazon S3 에 저장할지 묻는 메시지가 표시됩니다.

<img width="600" alt="11" src="https://user-images.githubusercontent.com/69136340/220173711-6c2c3911-127c-4217-81e0-ce5e26f02c61.png">

certificates 와 profiles 를 저장할 private git repository 를 만들고, **Git Repo URL** 을 입력하라고 합니다.

입력하게 되면 현재 디렉터리에 아래와 같은 컨텐츠를 가진 Matchfile (또는 (./fastlane/ 폴더 안에)가 생성됩니다.

```swift
git_url("https://github.com/...")

storage_mode("git")

type("development") # The default type, can be: appstore, adhoc, enterprise or development
```

### 2️⃣ Run

처음 match 를 실행하기 전에 `match nuke` 명령어를 사용해서 기존의 프로파일들과 인증서를 지워야만 합니다.

```swift
fastlane match nuke development
fastlane match nuke distribution
```

다음의 명령어로 새로운 인증서와 프로파일들을 생성할 수 있습니다.

```swift
fastlane match appstore
fastlane match development
```

- Passphrase

**새 컴퓨터에서 match 를 처음 실행하게 되면** 다음과 같이 Git repository 의 암호를 묻는 메시지가 표시됩니다. 다른 컴퓨터에서 match 를 실행할 때 필요하므로 기억해두면 됩니다.

<img width="700" alt="12" src="https://user-images.githubusercontent.com/69136340/220173853-52ff38a8-c983-48bf-aa33-79d0130524d5.png">

- `match nuke` 명령어를 사용해서 다음과 같이 기존의 인증서와 프로파일이 말끔하게 삭제되었습니다.

<img width="600" alt="13" src="https://user-images.githubusercontent.com/69136340/220173943-f2102871-309b-44ca-92ae-c4cb6a304b3c.png">

- fastlane match 명령어를 통해서 다음과 같이 인증서와 프로파일을 새로 만들 수 있었습니다.

<img width="600" alt="스크린샷 2023-02-21 오전 2 55 30" src="https://user-images.githubusercontent.com/69136340/220173928-6c9e70bc-8c6c-4318-8684-82916242e628.png">

***그런데 우리는 multiple target 에 대응하기로 했습니다. Notification Service Extension 을 위한 인증서와 프로비저닝 프로파일도 만들어보겠습니다.***

### 3️⃣ Handle multiple targets

match 는 모든 bundle identifiers 에 대해서 동일한 Git repository 를 사용할 수 있습니다.

다른 bundle identifiers 를 가진 여러가지 타겟이 있는 경우, 쉼표로 리스트를 제공할 수 있습니다.

```swift
fastlane match appstore -a tools.fastlane.app,tools.fastlane.app.watchkitapp
```

다음과 같이 lane 을 생성해서 fastlane 을 사용할 수도 있습니다. `fastlane certificates` 를 실행하여 동기화할 수 있습니다.

```swift
lane :certificates do
  match(app_identifier: ["tools.fastlane.app", "tools.fastlane.app.watchkitapp"])
end
```

- 처음부터 이렇게 진행해도 되고, 저는 위에서 만들어 놓은 인증서와 프로파일에 추가해보겠습니다.

```swift
// 혹은 "fastlane match appstore -a com.notificationService" 실행.

lane :certificates do
  match(app_identifier: ["com.notificaionService"])
end
```

이렇게 작성하고 `fastlane certificates` 를 실행하게 되면 development 만 생성됩니다. **그 이유는 Matchfile 에 type 이 development 이기 때문입니다.**

```swift
git_url("https://github.com/...")

storage_mode("git")
# appstore 로 변경해서 실행해주면 됩니다.
type("appstore") # The default type, can be: appstore, adhoc, enterprise or development
```

- 업데이트할 필요도 있기 때문에 이쁘게 정리해보겠습니다. `fastlane certificates` 를 실행하면 둘 다 생성되게 됩니다.

```swift
lane :certificates do
    match(type: "appstore",
          app_identifier:["com.mainapp", "com.notificationService"])
    match(type: "development",
          app_identifier:["com.mainapp", "com.notificationService"])
end
```

***Git repo 를 확인해볼까요?***

### 4️⃣ Folder structure

match 를 처음 실행하게 되면 Git repo 에 두 가지 디렉토리가 생깁니다.

- certs 폴더는 모든 인증서와 개인 키를 포함합니다.
- profiles 폴더는 provisioning profiles 를 포함합니다.

또한, match 는 README.md 도 만들어줍니다.

<img width="600" alt="15" src="https://user-images.githubusercontent.com/69136340/220174059-b6ce1fb8-ade9-48b6-9e5d-dea838fcb718.png">

***설정이 끝났습니다. 다른 개발자들은 어떻게 실행할 수 있을까요?***

### 5️⃣ New machine

인증서와 프로비저닝 프로파일을 설정하기 위해서 다음의 명령어만 실행하면 됩니다.

```swift
fastlane match development

// 읽기 전용 모드에서 match 를 실행해서 새로운 인증서나 프로파일을 생성하지 않도록 할 수 있습니다.
fastlane match development --readonly
// 또한, Apple Developer Portal 에 접근하기 위해서 비밀번호를 묻지 않도록 --readonly 옵션을 사용할 수 있습니다. 이렇게 되면 다른 개발자들이 Apple Developer Portal 에 접근해서 실수로 인증서를 취소하지 않을 수 있습니다.
```

multiple targets 인 경우에는 다음과 같이 사용할 수 있습니다.

```swift
fastlane match development -a com.mainapp, com.notificationService --readonly
```

다음의 목표들을 달성할 수 있도록 lane 을 작성해보겠습니다. 

- 해당 프로젝트에서는 development 와 distribution 인증서를 생성했기 때문에 두 가지를 실행해야 합니다.
- multiple targets 을 대응해야 합니다.
- readonly 옵션을 추가해서 안전하게 관리해야 합니다.

```swift
lane :certificates do
    match(type: "appstore",
          app_identifier:["com.mainapp", "com.notificationService"],
          readonly: true)
    match(type: "development",
          app_identifier:["com.mainapp", "com.notificationService"],
          readonly: true)
end
```

### 6️⃣ private Git repo 초대

팀원들을 private Git repository 에 초대해야 접근할 수 있겠죠?

- 생성한 private Git repository 의 **Setting > Access** 에서 팀원들을 초대할 수 있습니다.
- `fastlane match development --readonly` 등의 match 작업을 실행할 때 git repository 주소를 입력하라고 하는데 해당 주소를 입력하면 되고, 위에서 설정한 passphrase 를 입력하면 됩니다.

### 💫 느낀점

sigh and cert 방식은 기존의 인증서를 지우지 않아도 되지만, 개발자에게 직접 파일을 넘겨주기 위한 설정의 과정이 필요해서 번거로웠던 것 같습니다.

match 를 적용한 프로젝트가 소규모 인원이었기 때문에 이렇게 중간에 도입해서 인증서를 새로 만들더라도 match 에서 private repo 를 활용한 방식이 더 적용하기 편했던 것 같습니다. 이후에 타겟이 추가되어 certificate 와 profile 을 다시 설치하는 과정도 손쉽게 할 수 있는 것이 장점이라고 느꼈습니다.

정리해보자면 개발자마다 인증서를 만들지 않아도 되고, 한 곳에서 관리하기 쉽고, 다른 개발자들도 쉽게 설치할 수 있는 것이 장점이라고 느꼈습니다. 기존의 인증서를 지워야 하지만 fastlane 에서 손쉽게 생성할 수도 있어 불편하지 않았습니다.

출처:

[[iOS] Certificate 와 Provisioning profile](https://sujinnaljin.medium.com/ios-certificate-%EC%99%80-provisioning-profile-e1b9455e8a51)

[Apple Developer Documentation](https://developer.apple.com/documentation/technotes/tn3125-inside-code-signing-provisioning-profiles)

[match - fastlane docs](https://docs.fastlane.tools/actions/match/)

[What is app signing?](https://help.apple.com/xcode/mac/current/#/dev3a05256b8)
