---
title:  "iOS) Face ID & Touch ID - Biometrics Authentication(생체인식 인증)"
categories:
- iOS

date:   2021-09-06  17:30:00 +0900
author_profile: false
---
😨 Face ID 를 활용한 오싹한 생체인식 인증 실험

# 시작 전

## 😃 [Local Authentication](https://developer.apple.com/documentation/localauthentication/)

Authenticate users biometrically(생체인식)  또는 이미 알고 있는 passphrase(암호)로 사용자를 인증합니다.

**Overview**

많은 사용자가 TouchID 또는 FaceID 와 같은 생체인식에 의존해서 장비에 손쉽게 접근할 수 있다. 대체 옵션으로 생체 인식이 없는 경우 암호가 비슷한 용도로 활용된다. `LocalAuthentication` 프레임워크를 사용하여 앱에서 이러한 메커니즘을 활용하고 이미 구현한 인증 절차를 확장할 수 있다.

<img src = "https://user-images.githubusercontent.com/69136340/132942269-f0859ac9-2896-4bb9-a361-7a12452b7b4f.png" width ="600">

보안을 극대화하기 위해서 underlying authentication data(기본 인증 데이터)에 접근할 수 없다. 예를들어 지문 이미지에 접근할 수 없다는 뜻이다. 시스템의 나머지 부분과 격리된 하드웨어 기반 보안 프로세서인 Secure Enclave는 이 데이터를 운영 체제의 손마저 닿지 않는 곳에 관리합니다.

**대신 특정 policy 를 지정하고 사용자에게 인증을 원하는 이유를 알려주는 메시징을 제공**합니다. 그런 다음 Secure Enclave 와 조정하여 작업을 수행한다. **이후에는** **성공과 실패를 나타내는 Boolean 결과만 얻는다.**

## 😃 이제 애플에서 제공하는 예제코드에 대해서 알아보자

[Apple Developer Documentation](https://developer.apple.com/documentation/localauthentication/logging_a_user_into_your_app_with_face_id_or_touch_id)

### Overview

사용자는 Touch ID 와 Face ID 와 같은 인증 매커니즘을 통해서 장치에 쉽게 접근할 수 있기 때문에 좋아한다. `LocalAuthentication` 프레임워크를 채택하면 일반적인 경우 사용자 인경 경험을 간소화하는 동시에 생체인식을 사용할 수 없을 때를 대비한 대체 옵션도 제공한다.

### Set the Face ID Usage Description

biometrics 를 사용하는 모든  프로젝트에서  Info.plist 파일에 `NSFaceIDUsageDescription` 키를 가진다. 이 키가 없으면 시스템은 앱이 Face ID 를 사용을 허락하지 않는다. 키의 값은 사용자가 처음으로 Face ID 를 사용할 때 시스템이 보여주는 문자열이다. 문자열은 앱이 이 인증 매커니즘을 액세스해야 하는 이유를 명확하게 설명해준다. 시스템은 Touch ID 에 대한 유사한 사용 설명을 필요로 하지 않는다.

(key 로 Privacy - Face ID Usage Description 를 사용해도 가능하다. 어차피 위의 키값을 입력하면 변환된다.)

### Create and Configure a Context

앱에서 LAContext 인스턴스를 사용해서 생체인식 인증을 사용할 수 있다. (LAContext 는 Secure Enclave 와 앱과의 상호작용을 중개한다.) context 를 생성하며 시작한다 :

```swift
var context = LAContext()
```

context 에서 사용하는 메시징을 커스텀해서 유저를 흐름으로 안내할 수 있다. 예를 들어, 다양한 alert view 에 나타나는 Cancel button 에 대한 커스텀 메시지를 설정할 수 있다.

```swift
context.localizedCancelTitle = "Enter Username/Password"
```

이것은 사용자가 버튼을 탭하면 일반 인증 절차로 되돌아갈 것을 이해하는데 도움이 된다.

### Test Policy Availability

인증을 시도하기 전에 [canEvaluatePolicy(_:error:)](https://developer.apple.com/documentation/localauthentication/lacontext/1514149-canevaluatepolicy) 메서드를 호출해서 인증할 수 있는지 테스트할 수 있다.

```swift
var error: NSError?
if context.canEvaluatePolicy(.deviceOwnerAuthentication, error: &error) {
```

테스트 할 LAPolicy 열거에서 값을 선택한다. poicy 는 인증이 작동하는 방식을 제어한다. 예를 들어, 위의 코드에서 사용된 `LAPolicy.deviceOwnerAuthentication` 은 생채인식이 실패하거나 사용할 수 없는 경우 passcode 로 되돌릴 수 있다. 또는 passcode 로 돌릴 수 없는 `LAPolicy.deviceOwnerAuthenticationWithBiometrics` 로 지정할 수도 있다.

### Evaluate a Policy

인증할 준비가 되면 이미 테스트한 policy 와 동일한 policy 를 사용해서 [evaluatePolicy(_:localizedReason:reply:)](https://developer.apple.com/documentation/localauthentication/lacontext/1514176-evaluatepolicy) 를 호출한다. 

```swift
let reason = "Log in to your account"
context.evaluatePolicy(.deviceOwnerAuthentication, localizedReason: reason ) { success, error in

    if success {

        // Move to the main thread because a state update triggers UI changes.
        DispatchQueue.main.async { [unowned self] in
            self.state = .loggedin
        }

    } else {
        print(error?.localizedDescription ?? "Failed to authenticate")

        // Fall back to a asking for username and password.
        // ...
    }
}
```

Touch ID 의 경우 또는 사용자가 passcode 를 입력하면 시스템은 위의 메소드에서 제공하는 `reason` 을 인증 이유로 표시한다.

(다음과 같이 "Log in to your account" 문자열이 passcode 사용 시 인증 이유로 표시된다.)

<img src ="https://user-images.githubusercontent.com/69136340/132942295-411b7593-787e-4160-9ac6-2e8c1308589c.jpeg" width ="250">

앱이 사용자에게 인증을 요청하는 이유에 대해서 운영하는 모든 지역에 localized 된 명확한 설명을 제공하는 것은 중요하다. 앱 이름은 위의 사진처럼 이유 앞에 이미 나타나므로 메시지에 포함시킬 필요가 없다.

### Optionally, Adjust Your User Interface to Accommodate Face ID

생체 인식 스캔 작업을 제외하고 Face ID 는 Touch ID 와 중요한 차이점을 보여준다.

- 앱이 Touch ID 를 사용하려고 하면 손가락을 제시하라는 시스템 메시지가 표시된다. 사용자는 프롬프트를 취소하여 작업에 대해 생각하고 취소할 시간이 있다. Face ID 를 호출할때는 기기가 바로 사용자의 얼굴을 스캔하기 시작한다. 사용자는 취소할 수 있는 마지막 기회를 얻지 못한다.

이러한 동작 차이를 수용하기 위해서 생체 인식 종류에 따라 다른 UI 를 제공할 수 있다.

아래와 같은 애플 샘플 앱의 경우 버튼을 탭하면 즉각적인 Face ID 스캔이 발생하는 것을 경고메시지 텍스트 라벨을 포함하여 다른 UI 로 제공한다. 

<img src ="https://user-images.githubusercontent.com/69136340/132942306-1d4d8797-4ec0-461f-b7f9-352fce16a504.png" width ="250">
context 의 biometryTpe 파라미터를 읽어서 장치가 어떤 biometry 를 지원하는지 테스트할 수 있다.

(다음의 코드는 **Face ID 를 지원하는 않는 디바이스의 경우** 혹은 이미 로그인된 경우 Face ID 인증 라벨을 hidden 시키는 코드이다.)

```swift
faceIDLabel.isHidden = (state == .loggedin) || (context.biometryType != .faceID)
```

(다음은 Face ID 를 지원하지 않는 iPhone SE 2세대이다. 실행했을 경우에 라벨이 hidden 된 것을 볼 수 있다.)

<img src="https://user-images.githubusercontent.com/69136340/132942307-8294d53b-7b59-4ae1-961c-671eb526baea.png" width ="250">

### Provide a Fallback Alternative to Biometrics

다양한 이유로 인증은 실패하거나 사용할 수 없는 경우가 있다.

- The user’s device doesn’t have Touch ID or Face ID.
- The user isn’t enrolled in biometrics, or doesn’t have a passcode set.
- The user cancels the operation.
- Touch ID or Face ID fails to recognize the user.
- You’ve previously invalidated the context with a call to the [invalidate()](https://developer.apple.com/documentation/localauthentication/lacontext/1514192-invalidate) method.(invalidate() 메서드를 호출해서 context 를 무효화하는 것을 말한다. )

가능한 error 조건의 전체 목록은 [LAError.Code](https://developer.apple.com/documentation/localauthentication/laerror/code/) 를 참조하자.

샘플 앱은 대체 인증을 구현하지 않았다. 실제 앱에서는 local authentication error 가 발생하면 사용자 이름과 비밀번호를 묻는 것과 같은 고유한 인증 체계로 대체한다. (즉, 유일한 인증옵션으로 대하지 말라는 것.)

이미 하는 일에 보완책으로 biometrics 를 제공해라. 유일한 인증 옵션으로 biometrics 를 의존하지 마라.

## 😃 [LAError](https://developer.apple.com/documentation/localauthentication/laerror)

LocalAuthentication 프레임워크에서 발생한 오류.

### 😃 [LAError.Codes](https://developer.apple.com/documentation/localauthentication/laerror/code)

policy 평가에 실패했을 때 LocalAuthentication 프레임워크가 반환하는 Error codes 이다.

내용을 보면 Biometry Failure 에 해당하는 에러코드가 있다.

# 시작하기

아이폰 기기가 Face ID 를 지원해야하고 Face ID 를 사용하고 있는 상황에서 가능하다. 

## 😃 프로젝트 설정

- Info.plist 파일

<img src ="https://user-images.githubusercontent.com/69136340/132942409-3a5b0e7e-bd27-4806-84c5-fd0c121bda3d.png" width ="700">

- Face ID 사용

```swift
import UIKit
import LocalAuthentication

class ViewController: UIViewController {
    
    enum AuthenticationState {
        case loggedin, loggedout
    }
    
    // ✅ 현재 로그인 상태에 따른 UI 변화
    var state = AuthenticationState.loggedout {
        didSet {
            if state == .loggedin {
                loginButton.setTitle("Logout", for: .normal)
            } else {
                loginButton.setTitle("Login", for: .normal)
            }
        }
    }
    
    @IBOutlet weak var loginButton: UIButton!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        loginButton.setTitle("Login", for: .normal)
    }
    
    @IBAction func touchLoginButton(_ sender: Any) {
        if state == .loggedin {
            
            state = .loggedout
        
        } else {
            // ✅ LAContext 로 Secure Enclave 와 앱과의 상호작용을 중개.
            let context = LAContext()
            
            // ✅ alert view 에서 cancel button 메시지.
            context.localizedCancelTitle = "Enter Username/Password"
            
            var error: NSError?
            
            // ✅ 지정한 policy 로 biometrics 인증이 가능한지 테스트.
            if context.canEvaluatePolicy(.deviceOwnerAuthentication, error: &error) {
                
                let reason = "Log in to your account"
                
                // ✅ 지정한 policy 로 biometrics 인증 시작.
                context.evaluatePolicy(.deviceOwnerAuthentication, localizedReason: reason ) { success, error in
                    if success {
                        // ✅ state 업데이트는 UI 변화를 일으켜서 main thread 에서 처리해야한다.
                        DispatchQueue.main.async { [unowned self] in
                            self.state = .loggedin
                            print("state : \(self.state)")
                        }
                    } else {
                        print(error?.localizedDescription ?? "Failed to authenticate")
                        
                        // Fall back to a asking for username and password.
                        // ...
                    }
                }
            }
        }
    }
}
```

## 시뮬레이터에서 Face ID 실행해보기

- Features → Face ID → Enrolled 로 활성화해준다.
- (활성화 이전에 생체인식 인증을 시도하면 passcode 를 입력받으려 한다.)

<img src ="https://user-images.githubusercontent.com/69136340/132942442-cf22c39b-af0c-4e9d-905e-6f3b13839f11.png" width ="450">

- Face ID 를 매칭 / 비매칭 선택할 수 있다.

<img src ="https://user-images.githubusercontent.com/69136340/132942444-b9e7d2af-465b-4dc4-932d-910acf01f34b.png" width ="450">

### 결과
<img src ="https://user-images.githubusercontent.com/69136340/132942657-925d5a70-5982-4abb-af10-89b65535e1b5.gif" width ="250">

### 출처

[Apple Developer Documentation](https://developer.apple.com/documentation/localauthentication/)
