---
title:  "iOS) fastlane Fastfile 설정 - match"
categories:
- iOS

date:   2023-06-22  02:30:00 +0900
author_profile: false
---
### 내용

- TestFlight 와 릴리즈 앱 자동 배포를 위한 fastlane 설정
- fastlane code signing 방법 중 match 방식에서 fastlane 의 lane 설정
- 하나의 개발 팀으로 multiple target 에 대한 code singing 적용, slack 연동 목표

### ✅ 들어가기 전,

fastlane 에서는 cert and sigh 방법과 match 방법을 지원합니다. 이 둘은 방법이 다르기 때문에 lane 또한 다르게 작성될 수 밖에 없습니다. 

- cert and sigh 방법은 로컬에 certificate 와 private key(cert 사용), provisioning profile 을 확인(sigh 사용)하여 없다면 다운받고, 생성이  필요하다면 개발자 계정에 생성하는 `get_certificates` , `get_provisioning_profile` 명령어를 사용합니다.
- match 방법은 보통 private repositary 에 있는 동기화된 certificates 를 사용하기 위해서 해당 repositary 의 주소를 설정하기 위해 `fastlane match init` 으로 초기화 단계가 다릅니다.

### ✅ Setup

프로젝트 폴더에서 match 를 사용해서 시작합니다.

```swift
fastlane match init
```

***(code signing 과 fastalen match 설정에 대해서 다음 글에서 정리하였습니다.)***

[iOS) Code Signing 과 Fastlane match 설정](https://gyuios.tistory.com/272)

이후, 다음의 명령어를 통해서 새로운 certificate 와 provisioning profile 을 선택된 storage 에 만들 수 있습니다.

**또한, Keychain 에 certificate, private key 가 설치되어 있는 동안  `~/Library/MobileDevice/Provisioning Profiles` 경로에 provisioing profile 이 설치됩니다.**

```swift
fastlane match appstore

fastlane match development
```

### ✅ Handle multiple targets

여러 개의 target 에서 다른 bundle identifiers 를 가진다면, 다음과 같이 콤마로 표현할 수 있습니다.

```swift
fastlane match appstore -a tools.fastlane.app,tools.fastlane.app.watchkitapp

// -a : app_identifier 설정하는 옵션.
```

다음의 목표들을 달성할 수 있도록 lane 을 작성해보겠습니다.(`fastlane install` 명령어로 사용)

- 해당 프로젝트에서는 development 와 distribution 인증서를 생성했기 때문에 두 가지를 실행해야 합니다.
- multiple targets 을 대응해야 합니다.
- readonly 옵션을 추가해서 안전하게 관리해야 합니다.(새로운 certificates or profiles 를 생성하지 않도록 함)

```swift
lane :install do
    match(type: "appstore",
          app_identifier:["com.mainapp", "com.notificationService"],
          readonly: true)
    match(type: "development",
          app_identifier:["com.mainapp", "com.notificationService"],
          readonly: true)
end
```

### ✅ lane 작성 - TestFlight Deployment

다음의 목표를 가지고 작성하였습니다.

- multiple targets 을 대응.
- 빌드 넘버 증가.
- 명령어 입력 시 파라미터를 전달하여 버전 설정 및 테스트 메모 설정.
- 외부 테스트가 아닌 내부 테스트만 진행.
- slack 연동.

```swift
# beta
  desc "Push a new beta build to the TestFlight"
  lane :beta do |options|
    increment_build_number(xcodeproj: "com.mainapp.xcodeproj")

    build_app(
      workspace: "com.mainapp.xcworkspace",
      scheme: "com.mainapp (Beta)",
      export_method: "app-store",
      export_options: {
        provisioningProfiles: {
          "com.mainapp" => "match AppStore com.mainapp",
          "com.mainapp.Widgets" => "match AppStore com.mainapp.Widgets",
          "com.mainapp.IntentsExtension" => "match AppStore com.mainapp.IntentsExtension",
          "com.mainapp.IntentsExtensionUI" => "match AppStore com.mainapp.IntentsExtensionUI"
        }
      }
    )

    if options[:changelog]
      upload_to_testflight(
        # 외부 테스트의 경우에 true 로 설정하고, groups 를 추가해야함.
        distribute_external: false, # default
        changelog: "changelog"
      )
    else
      upload_to_testflight(
        distribute_external: false, # default
        changelog: ""
      )
    end

    # version 사용 시 반영, 미사용 시 현재 프로젝트 버전 사용.
    if options[:version]
      increment_version_number(
        version_number: options[:version],
        xcodeproj: "com.mainapp.xcodeproj"
      )
    else
      version = get_version_number(
                  target: "mainapp" # multiple target 인 경우 설정.
                )
    end

    build = get_build_number

    slack(
      username: "",
      icon_url: "",
      message: "성공적으로 TestFlight 에 등록되었습니다!🔥",
      success: true, # default
      slack_url: "",
      payload: {
    "Version": version + "(" + build + ")"
      }
    )
  end
```

### 🚨 앱 스토어에 올리는 것이 아닌데 export_method: “app-store” 로 설정하나요?

`export_method` 의 경우 배포 방식에 대한 설정입니다. Xcode 에서 앱을 배포하기 위해서 **Archive** 과정을 진행하면 다음과 같이 선택할 수 있는 단계에 대한 설정입니다.

- 다음과 같이 TestFlight 과 App Store 배포를 위해서는 `app-store` 을 설정해주면 됩니다.

![스크린샷 2023-06-21 오후 10.29.24.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9862a3d4-9171-4d36-87d2-29f81fb15b33/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-06-21_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.29.24.png)

- **문제 상황:** lane 내에서 export_method 와 일치하지 않는 profile 을 매칭해줌.
- **문제 해결:** TestFlight 를 배포하기 위해서기 때문에 profile 을 AppStore 용으로 재설정함.

![스크린샷 2023-06-22 오전 1.15.46.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/85ebabdd-9865-4c6c-8621-3a76da12c321/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-06-22_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_1.15.46.png)

**참고**

- 파라미터를 전달하는 방법은 아래 링크를 참조.

[Lanes - fastlane docs](https://docs.fastlane.tools/advanced/lanes/)

- slack 관련 옵션은 아래 링크를 참조.

[slack - fastlane docs](https://docs.fastlane.tools/actions/slack/)

### ✅ lane 작성 - Release Deployment

다음의 목표를 가지고 작성하였습니다.

- multiple targets 을 대응.
- 빌드 넘버 증가.
- 명령어 입력 시 파라미터를 전달하여 버전 설정.
- slack 연동.

```swift
  # release
  desc "Push a new release build to the App Store"
  lane :release do |options|

    increment_build_number(xcodeproj: "com.mainapp.xcodeproj")

    build_app(
      workspace: "com.mainapp.xcworkspace",
      scheme: "com.mainapp (Release)"
      export_method: "app-store",
      export_options: {
        provisioningProfiles: {
          "com.mainapp" => "match AppStore main.app",
          "com.mainapp.Widgets" => "match AppStore com.mainap.Widgets",
          "com.mainapp.IntentsExtension" => "match AppStore com.mainapp.IntentsExtension",
          "com.mainapp.IntentsExtensionUI" => "match AppStore com.mainapp.IntentsExtensionUI"
        }
      }
    )

    upload_to_app_store(
        # ✅ metadata 는 업데이트 내역을 위해서 스킵하지 않음.
        skip_metadata: false, # default
        # ✅ screenshots 는 기존의 것 사용. 
        skip_screenshots: true,
        submit_for_review: true,
        automatic_release: true,
        # ✅ force: HTML report를 스킵.
        force: true,
        # ✅ precheck 에서 In-app 결제 제외.
        precheck_include_in_app_purchases: false,
        # ✅ IDFA 세팅 안함.
        submission_information: { add_id_info_uses_idfa: false }
      )

    if options[:version]
    # version 이 있으면 해당 버전으로 업데이트.
      increment_version_number(
        version_number: options[:version],
        xcodeproj: "com.mainapp.xcodeproj"
      )
    else
    # version 없으면 타겟의 버전 적용.
      version = get_version_number(
                  target: "mainapp"
                )
    end

    build = get_build_number

    slack(
      username: "",
      icon_url: "",
      message: "성공적으로 앱을 등록했습니다!💫",
      success: true, # default
      slack_url: "",
      payload: {
          "Version": version + "(" + build + ")"
      }
    )
  end
```

### ✅ 에러 발생 시 slack 연동

```swift
# error
  error do |lane, exception, options|
    version = get_version_number(
                target: "mainapp"
              )
    build = get_build_number 

    slack(
      username: "",
      icon_url: "",
      message: "에러 발생!!! 발생!!🚨 : #{exception}",
      success: false,
      slack_url: "https://hooks.slack.com/services/...",
      payload: {
          "Version": version + "(" + build + ")"
      }
    )
  end
```

### ✅ 사용

다음과 같이 사용할 수 있습니다.

```swift
fastlane beta version:"2.1.0" changelog:"테스트 메모"

fastlane beta # version 그대로, changelog 빈값.
```

### 참고

[fastlane docs](https://docs.fastlane.tools/)
