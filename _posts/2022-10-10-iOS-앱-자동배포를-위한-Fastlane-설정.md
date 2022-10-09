---
title:  "iOS) 앱 자동 배포를 위한 Fastlane 설정"
categories:
- iOS

date:   2022-10-10  01:11:00 +0900
author_profile: false
---
### 내용

- TestFlight 와 릴리즈 앱 자동 배포 위한 Fastlane 설정

## 0️⃣ Fastlane 을 선택한 이유

## 1️⃣ Fastlane 설치

`brew install fastlane` 명령어를 통해서 HomeBrew 를 이용해 Fastlane 을 설치할 수 있습니다.

`gem install bundler` 명령어를 통해서 bundler 를 설치해줍니다. bundler 는 fastlane 을 업데이트할 때 사용됩니다.

## 2️⃣ Fastlane 기본 설정

사용할 Xcode 프로젝트가 있는 경로에서 `fastlane init` 명령어로 기본 설정을 시작할 수 있습니다.

### 👉 beta 테스트를 위한 TestFlight 앱의 자동배포

<img width="500" alt="1" src="https://user-images.githubusercontent.com/69136340/194746425-f4ff29d2-7a44-4606-a52b-e9fc936b2353.png">

- 1-4번까지 해당되는 설정을 선택할 수 있습니다.
- beta 테스트를 위한 TestFlight 를 만들기 위함이니 2번을 선택합니다.
- 그 다음단계에서 Build Scheme 를 선택하여 지정할  수 있습니다.

<img width="600" alt="2" src="https://user-images.githubusercontent.com/69136340/194746432-6e200b8a-01a9-4530-a1aa-d93c8fccb75a.png">

- Apple ID 와 비밀번호를 입력하고, 이중 인증 6자리도 입력해줍니다.
- 만약에 소속된 App Store Connect team 과 Developer Portal 이 여러개라면 고르면 됩니다.

### 👉 fastlane 설정 파일

프로젝트를 살펴보면 다음과 같이 파일들이 생성되어있습니다.

```swift
fastlane
    Appfile  : 앱의 번들 ID, app ID, team ID 정보가 담겨있음.
    Fastfile : 배포와 관련된 자동화 명령어들이 담겨있음.
Gemfile      : gem 목록
Gemfil.lock  : 의존성 버전 관리를 위한 정보가 담겨있음.
```

- 위에서 2번이 아닌 3번을 선택하게되면 메타데이터를 관리하겠냐고 묻는다. 이때 다운로드할 수 있다.
- 2번을 선택하더라도 해당 명령어로 추가할 수 있다.
    - 스크린샷 다운로드 : `fastlane deliver download_screenshots`
    - 메타데이터 다운로드 : `fastlane deliver download_metadata`

```swift
fastlane
    metadata
    screenshots
Deliverfile : 앱스토어 메타데이터 저장을 위한 파일
```

## ❓Code signing?

fastlane 에서는 앱 개발과 배포를 위한 certificate 와 provisioning profile 을 등록하기 위한 code signing 작업에 두 가지 방법을 지원합니다.

- certificate(인증서) : 개발자가 Apple 대신 앱을 실행할 수 있는 권리를 허용
- provisioning profile(프로바이저닝 프로필) : 디바이스가 앱을 신뢰할 수 있는지 확인

### match

- private keys(개인 키)와 certificates(인증서)를 git 레포지토리에 저장하여 개발자의 기기 간에 동기화 가능합니다.
- match 를 시작하려면 기존 certificate 를 취소해야 합니다.
- 보통 private repositary 를 생성하여서 사용하는데 private repositary 에서 인증서만 한번 바꿔주면 되기 때문에 갱신 문제, 계정 문제를 고민할 필요가 없습니다.

### cert and sigh

- 기존 certificate 를 취소하고 싶지않지만, 자동화된 설정을 원하는 경우 cert and sigh 방법이 적합합니다.
- `cert` 는 유효한 certificate 와 private key 가 로컬에 설치되어 있는지 확인하고, `sigh` 는 설치된 certificate 와 일치하는 provisioning profile 로컬에 설치되어 있는지 확인합니다.
- 그리고 `get_certificates` , `get_provisioning_profile` 명령어를 lane 에 추가해주어야 합니다.(인증서와 프로필을 다운받아준다.)

[Getting Started - fastlane docs](https://docs.fastlane.tools/codesigning/getting-started/)

## 3️⃣ lane 생성

기능에 따라서 명령어를 작성하면 된다. 아래의 링크를 참조해도 되고, 여러 블로그에서 필수적인 명령어를 사용해서 참고해도 충분히 진행할 수 있었다.

[Available Actions - fastlane docs](https://docs.fastlane.tools/actions/)](https://docs.fastlane.tools/actions/)

## 👉 beta 테스트를 위한 TestFlight 앱의 자동배포

- 각 bundle id 에 대한 provisioning profile(프로젝트에서 notification service 를 사용하고 있었음) 매칭하지 않은 에러
- authorization 부여하지 않은 에러

✅ ***이하 lane 명령어는 위의 두 가지를 설정하지 않았기 때문에 에러가 날 예정입니다. 에러는 아래에서 살펴보겠습니다.***

```swift
default_platform(:ios)

platform :ios do

# ✅ 기본 플랫폼을 ios 로 설정하여 fastlane [lane name] 으로 쉽게 실행가능.(원래 fastlane ios [lane name] 으로 실행해야 함)
  desc "Push a new beta build to TestFlight"
  lane :beta do

    # ✅ Cert, Sigh 방식으로 인증할 때 사용하는 인증 함수다.
    # get_certificates로 인증서를, get_provisioning_profile로 프로필을 가져온다.
    get_certificates
    get_provisioning_profile

    # ✅ 자동으로 빌드 넘버를 증가
    increment_build_number(xcodeproj: "~.xcodeproj")

    # ✅ 빌드할 워크스페이스와 빌드 스키마 지정
    build_app(workspace: "~.xcworkspace", scheme: "~-beta")

    # ✅ TestFlight 업로드
    upload_to_testflight(
    # ✅ 업로드 후에 App Store Connect 에 올라가기 전까지 시간이 걸리는데 이걸 기다리고 싶지 않다면 true 로 설정.
      skip_waiting_for_build_processing: true
    )
  end
end

// ~ 는 프로젝트 이름입니다.
```

## ❗️ error

```swift
Exit status: 70
No provisioning profile provided
```
<img width="600" alt="3" src="https://user-images.githubusercontent.com/69136340/194746471-dee2f8a3-dc06-420e-b058-d7aa66415d65.png">

자세히 살펴보니 notification service 인 extension target 이 provisioning profile 을 찾지 못하고 있었다.

- https://github.com/fastlane/fastlane/issues/12826

- [Xcode Project - fastlane docs](https://docs.fastlane.tools/codesigning/xcode-project/)

- [iOS fastlane - Fabric, Firebase](https://leejigun.github.io/fastlaneFirebase)

match 방법을 사용하지 않고, cert 와 sigh 방법을 사용하기 때문에 위의 문서들을 참고해서 provisioning profile 을 매칭해주었다.

```swift
# ...

build_app(
      workspace: "Spark-iOS.xcworkspace",
      scheme: "Spark-beta",
      export_method: "app-store",
      export_options: {
          provisioningProfiles: { 
# ✅ apple developer 에서 provisioning profile 의 이름을 입력해주어야 한다.
              "com.example.bundleid" => "Provisioning Profile Name",
              "com.example.bundleid2" => "Provisioning Profile Name 2"
          }
      })

# ...
```

그래도 error 가 등장했다.

### ❗️error2

`Unable to upload archive. Failed to get authorization for username '~' and password.`

ipa 파일까지 생성했는데 TestFlight 를 등록하는 단계에서 위와 같은 authorization 이 없다는 에러가 등장하였다. 그래서 fastlane 의 authentication 문서를 살펴보았다.

[Authentication - fastlane docs](https://docs.fastlane.tools/getting-started/ios/authentication/)

- 그 중에서 authorization 을 부여하기 위한 방법 중 App Store Connect API key 를 생성하는 방법이 추천되어서 적용해보았다.

<img width="500" alt="4" src="https://user-images.githubusercontent.com/69136340/194766600-69da188e-a690-4d56-8905-c227fbebc180.png">

- `App Store Connect > 사용자 및 액세스 > App Store Connect API` 에서 key 를 생성하고 문서를 기반으로 json 파일을 만들어서 적용했다.

<img width="500" alt="5" src="https://user-images.githubusercontent.com/69136340/194766645-7a276c15-b671-40b6-a780-361df761870a.png">

- Issuer ID 와 key ID, 키(.p8)를 다운 받을 수 있다.

<img width="700" alt="6" src="https://user-images.githubusercontent.com/69136340/194766854-18ccb541-b4b4-4115-8576-41a25677f289.png">

- 아래 문서를 통해서 해쉬 옵션과 json 파일로 관리하는 방법을 알려준다.(이하 json 파일로 관리로 적용해봄)

[App Store Connect API](https://docs.fastlane.tools/app-store-connect-api/#using-fastlane-api-key-json-file)

- josn 파일을 생성하여 진행하였다.

<img width="500" alt="7" src="https://user-images.githubusercontent.com/69136340/194766724-63f46f5f-bb3b-4093-9bec-1a11e4f45d6a.png">

### ❗️error3

`/usr/local/Cellar/fastlane/2.210.1/libexec/gems/json-2.6.2/lib/json/common.rb:216:in `parse': \e[31m[!] 859: unexpected token at '{ (JSON::ParserError)`

- 파싱 에러가 등장해서 json 파일을 수정해주었다.(`-----BEGIN PRIVATE KEY-----` 뒤에 줄바꿈(`\n`)을 빼먹어서 추가해주었다.)

```swift
{
  "key_id": "(key ID)",
  "issuer_id": "(Issuer ID)",
  "key": "-----BEGIN PRIVATE KEY-----\n(key(.p8) 값 복사)\n-----END PRIVATE KEY-----",
  "duration": 1200,
  "in_house": false
}
```

### ❗️error4

`invalid curve name`

- key 파일의 내용을 제대로 옮겨주지 않아서 에러가 발생했다.
- key 파일은 줄바꿈을 포함하는데 이 부분도 작성해주지 않았다. `\n` 를 추가하여 해결했다.

<img width="400" alt="8" src="https://user-images.githubusercontent.com/69136340/194766814-a98a6ff0-0585-455e-aec1-0c53ddae5f7f.png">

**즉, 위의 key 파일처럼 값을 넣기 위해서 모든 줄바꿈에는 `\n` 이 추가되어야 한다.(어쩌다보니 줄바꿈으로 에러를 두 개나 만나게 되었다🥲)**

### ✅ 수출 규정 관련 문서

<img width="300" alt="9" src="https://user-images.githubusercontent.com/69136340/194766887-7601c327-5613-46d7-a181-7524a4ec6396.png">


지금까지의 과정은 100퍼센트 자동화가 아닙니다. 수출 규정 관련 문서를 해결하지 않으면 결국 App Store Connect 에서 수동으로 설정해주어야 하기 때문입니다.😵

- https 를 호출하기 때문에 아니요를 체크하게 되었습니다.

<img width="400" alt="10" src="https://user-images.githubusercontent.com/69136340/194766908-09fab687-2385-469d-8147-bdce34db6823.png">

프로젝트 info.plist 에서 `App Uses Non-Exempt Encryption` 을 `No` 라고 해주거나 코드로 `<key>ITSAppUsesNonExemptEncryption</key> <No/>` 를 작성해주면 됩니다.(이후에는 App store connect 에서 이를 묻지 않음.)

이후 진행 상태가 `제출 준비 완료` 설정되었습니다.

하지만, 테스트를 위해서는 외부테스팅에서 빌드를  선택해야합니다. 그렇기 때문에 아직 자동화가 100퍼센트 되지 않았습니다.

### ✅ 외부 테스팅 그룹 설정

```swift
upload_to_testflight(
  # ✅ 해당 파라미터가 true 일때는 `distribute_external` 작동하지 않습니다.
  # 어차피 밑에서 등장하는 changlog 로 작동하지 않기 때문에 주석처리하겠습니다.
  # skip_waiting_for_build_processing: true

  # ...

  # ✅ Should the build be distributed to external testers? If set to true, use of groups option is required
  distribute_external: true,

  # ✅ This is required when distribute_external option is set to true or when we want to add a tester to one or more external testing groups
  groups: ["BetaTestGroup1", "BetaTestGroup2"]
)
```

- `distribute_external` 을 true 로 설정해주면 groups 를 사용해야 합니다. 이를 통해 테스트 그룹을 설정할 수 있습니다.

외부 테스팅 그룹을 설정했기 때문에 fastlane 이 먼저 changelog(테스트 세부사항)을 적을 수 있다고 제안해줍니다.

<img width="500" alt="11" src="https://user-images.githubusercontent.com/69136340/194766919-11d22e46-04e4-4382-bed3-fd752222e1fd.png">

***이 방법도 있지만 lane 에서 테스트 세부사항을 작성할 수 있습니다. 매번 기다렸다가 입력할 수는 없으니깐요!***

### ✅ 테스트 세부사항 작성

**lane 에서 테스트 세부사항을 작성해봅시다.** 테스트 세부사항을 작성하여 TestFlight 를 배포하고 싶을 때는 `changelog` 를 사용하면 됩니다.

[testflight actions](https://docs.fastlane.tools/actions/testflight/)

```swift
upload_to_testflight(
  # ...
  # ✅ `changelog` 를 사용하게 되면 `skip_waiting_for_build_processing` 를 true 로 설정하더라도 무시되지 않고 프로세스를 기다려야 합니다.
    changelog: "- 수출 규정 문서 추가
- fastlane 에서 테스트 세부사항 작성"
# ✅ `notify_external_testers` 를 false 로 설정하지 않는다면 기본값(true)으로 외부 테스터들에게 알립니다.
)
```

- 외부 테스팅 그룹과 테스트 세부사항이 설정된 100퍼센트 자동화가 완성되었습니다!🎊🎉

<img width="300" alt="12" src="https://user-images.githubusercontent.com/69136340/194766963-4e16cd49-fab9-4161-9359-429fcfcb6267.png">

<img width="400" alt="13" src="https://user-images.githubusercontent.com/69136340/194766970-26dc0d20-6ad0-4452-a857-971920522357.png">

### 👉 결과적으로 작성하게 된 TestFlight 자동 배포 lane

```swift
default_platform(:ios)

platform :ios do
  desc "Push a new beta build to TestFlight"
  lane :beta do
    get_certificates
    get_provisioning_profile
    increment_build_number(xcodeproj: "Project.xcodeproj")
    build_app(
      workspace: "Project.xcworkspace",
      scheme: "Project-beta",
      export_method: "app-store",
      export_options: {
        provisioningProfiles: { 
          "com.example.bundleid" => "Provisioning Profile Name",
          "com.example.bundleid2" => "Provisioning Profile Name 2"
        }
      }
    )

# ✅ 이렇게 key 파일을 로컬 경로에 두고 사용할 수 있다.
#api_key = app_store_connect_api_key(
#    key_id: "...",
#    issuer_id: "...",
#    key_filepath: "/Users/username/Desktop/auth/AuthKey.p8",
#    duration: 1200,
#    in_house: false
#  )

    upload_to_testflight(
# ✅ api_key_path 를 지정해주는 대신 api_key 를 만들어서 작성할 수 있다.
    #  api_key: api_key,
    api_key_path: "fastlane/key.json",
    distribute_external: true,
    groups: ["SparkBetaTestGroup"],
    changelog: "- 수출 규정 문서 추가
- fastlane 에서 테스트 세부사항 작성"
    )
  end
end
```

- fastlane/key.json

```swift
{
  "key_id": "(key ID)",
  "issuer_id": "(Issuer ID)",
  "key": "-----BEGIN PRIVATE KEY-----\n(key(.p8) 값 복사)\n-----END PRIVATE KEY-----",
  "duration": 1200,
  "in_house": false
}
```

💫 **지금까지 TestFlight 자동 배포에 대해서 알아봤습니다. 아래의 fastlane 문서에서 upload_to_testflight 의 다른 파라미터를 확인할 수 있습니다.**

[upload_to_testflight](http://docs.fastlane.tools/actions/upload_to_testflight/#upload_to_testflight)

### 👉  릴리즈를 위한 앱의 자동배포

```swift
default_platform(:ios)

desc "Push a new release build to the App Store"
lane :release do |options|
# ✅ 매개변수를 넣어서
# fastlane release version:"2.1.0"
# 과 같이 사용할 수 있다.
    if options[:version]
      increment_version_number(version_number: options[:version])
      get_certificates
      get_provisioning_profile
      build_app(workspace: "~.xcworkspace", scheme: "~-beta")
      upload_to_app_store(
        api_key_path: "fastlane/key.json",
        # ✅ screenshots 는 기존의 것 사용. metadata 는 업데이트 내역을 위해서 스킵하지 않음.
        skip_metadata: false,
        skip_screenshots: true,
        submit_for_review: true,
        automatic_release: true,
        # ✅ force: HTML report를 스킵합니다.
        force: true
      )
    # ✅ if 문을 종료하기 위한 end
    end
  end
end
```

### ✅ 업데이트 내역 작성

아래의 release_notes 를 수정해주면 됩니다.

<img width="300" alt="14" src="https://user-images.githubusercontent.com/69136340/194767001-9d49e4fc-9b62-4ee9-b88f-b96cc082863f.png">

💫 **지금까지 App Store 자동 배포에 대해서 알아봤습니다. 아래의 fastlane 문서에서 upload_to_app_store 의 다른 파라미터를 확인할 수 있습니다.**

[upload_to_app_store](http://docs.fastlane.tools/actions/upload_to_app_store/#upload_to_app_store)

## 4️⃣ Slack 으로 알림 보내기

- `Incoming WebHooks` 앱을 설치하고 원하는 채널을 설정해줍니다.

<img width="600" alt="15" src="https://user-images.githubusercontent.com/69136340/194767010-eafaad46-89ba-4283-95dd-69b594d34cb5.png">

- WebHook URL 이 설정되는데 이것을 사용하면 가능합니다.
- 터미널에서 손쉽게 `fastlane action slack` 명령어로 가능한 파라미터를 확인할 수 있습니다. 혹은 아래의 주소에서 확인할 수 있습니다.

[slack actions](https://docs.fastlane.tools/actions/slack/)

```swift
default_platform(:ios)

platform :ios do
  desc "Push a new beta build to TestFlight"
  lane :beta do
    # ...
    # ✅ 프로젝트의 버전과 빌드 번호를 가져옴.
    version = get_version_number
    build = get_build_number
  end
# ✅ Slack 설정.
    slack(
      username: "유저이름",
      message: "TestFlight 배포 성공.",
      icon_url: "아이콘 이미지 주소",
      slack_url: "https://hooks.slack.com/...",
      payload: { "Version": version + "(" + build + ")" }
    )

# ✅ 에러 처리.
error do |lane, exception, options|
    slack(
      message: "에러 발생 : #{exception}",
      success: false,
      slack_url: "https://hooks.slack.com/..."
    )
  end
end
```

### ❗️ **엇! 중간에 멈춰요!**

<img width="300" alt="16" src="https://user-images.githubusercontent.com/69136340/194767057-7d185268-5cc6-405c-be67-da84b8836a18.png">

- `get_version_number` 명령어를 사용할 때 프로젝트에 target이 2개 이상이라면 다음과 같이 물어봅니다. 그런데 매번 해줄 수 없으니 아래와 같이 파라미터를 추가해줍니다.

```swift
// ✅ Target name, optional. Will be needed if you have more than one non-test target to avoid being prompted to select one
version = get_version_number(
  xcodeproj: "Project.xcodeproj",
  target: "App"
)
```

### 💫 느낀점

<img src="https://user-images.githubusercontent.com/69136340/194767070-d54a9f14-7085-4905-86f2-b78604cb5a83.png" width ="600">

fastlane 을 활용하여 Continous Delivery(지속적인 제공) 를 넘어 Continuous Deployment(지속적인 배포) 까지 성공적으로 도입해 보았다.(deployment 와 delivery 의 차이점은 deployment 는 프로덕션의 자동 배포한다는 점이 다르다.)

👉 **진행하는 과정은 길고 각 프로젝트마다 다른 옵션을 가지다보니 다양하고, 더 적합하고, 좋은 파라미터에 대해서 고려하는 점이 진행하면서 어려웠습니다. 그러나 아카이브를 단계별로 거치는 과정과 TestFlight 를 배포하려고 웹사이트를 들락날락하는 과정을 명령어 하나로 진행할 수 있는 점이 가장 편리했습니다. 즉, 모든 배포에 대해서 자동화 할 수 있다는 것은 엄청난 장점으로 다가왔습니다.**

### 🗞 출처

[올리브영 iOS 테스트앱 자동배포하기 | 올리브영 테크블로그](https://oliveyoung.tech/blog/2021-09-08/Automatic-Distribution-iOS-Test-App-To-Fastlane/)

[iOS 앱 배포 자동화를 위한 Fastlane 설치 및 구성 | DevSecOps 구축 컨설팅, 교육, 기술지원 서비스 제공](https://insight.infograb.net/blog/2022/06/27/setup-fastlane-for-ios-app)

[[iOS] fastlane 이용한 배포 자동화 (TestFlight 편)](https://velog.io/@parkgyurim/iOS-fastlane-TestFlight)

[iOS - 배포 자동화, Fastlane 시작부터 적용까지](https://medium.com/hcleedev/ios-%EB%B0%B0%ED%8F%AC-%EC%9E%90%EB%8F%99%ED%99%94-fastlane-%EC%8B%9C%EC%9E%91%EB%B6%80%ED%84%B0-%EC%A0%81%EC%9A%A9%EA%B9%8C%EC%A7%80-3d9107cdc3b4)

[[iOS] Fastlane을 적용하여 배포 자동화하기 - Kyungmo's Blog](https://kyungmosung.github.io/2021/11/15/fastlane/)

[[iOS] fastlane 이용한 배포 자동화 (match 편)](https://velog.io/@parkgyurim/iOS-fastlane-match)

[Fastlane (2) - Test결과를 Slack으로 보내기](https://zeddios.tistory.com/839)

[fastlane 적용하여 앱스토어 배포까지!](https://vapor3965.tistory.com/95)

[[FastLane]Auto upload TestFlight _ 02 (외부 테스팅 추가)](https://kwanghone.tistory.com/40)
