---
title:  "iOS) Moya 에서 Plugin 으로 Token 갱신하기"
categories:
- iOS

date:   2022-04-11  10:41:00 +0900
author_profile: false
---
### 내용

- Custom Moya Plugin 을 활용해서 Refresh Token 으로 Access Token 갱신하기

# 서버통신 과정

1️⃣ 모든 서버통신 + 액세스 토큰 

2️⃣  액세스 토큰 **`만료 o`** (40x status code)

3️⃣ `didReceive` 에서 서버통신 (액세스 토큰 + 리프레쉬 토큰) 바디에 담아 보내기

→ 3-1 리프레쉬 토큰 `**만료 x**` (200 status code)→ 액세스 토큰, 리프레쉬 토큰 **`갱신`**

→ 3-2 리프레쉬 토큰 `**만료 o**` (40x status code)

4️⃣ 리프레쉬 토큰, 액세스토큰 삭제 및 로그인 화면으로 보내기

## 📡 Plugin 커스텀해서 해결하기

**Plugins**

Moya plugins are used to modify requests and responses or perform side-effects. A plugin is called:

- (`prepare`) after Moya has resolved the `TargetType` to a `URLRequest`. This is an opportunity to modify the request before it is sent (e.g. add headers).
- (`willSend`) before a request is about to be sent. This is an opportunity to inspect the request and perform any side-effects (e.g. logging).
- (`didReceive`) after a response has been received. This is an opportunity to inspect the response and perform side-effects.
- (`process`) before `completion` is called with the `Result`. This is an opportunity to make any modifications to the `Result` of the `request`.

**출처:** 

[Moya/Plugins.md at master · Moya/Moya](https://github.com/Moya/Moya/blob/master/docs/Plugins.md)

# 📡 해결 과정

### 개선 코드

```swift
import Foundation
import Moya

final class MoyaLoggerPlugin: PluginType {
    // 🔥 Request 가 전송되기 전.
    func willSend(_ request: RequestType, target: TargetType) {
        guard let httpRequest = request.request else {
            print("--> 유효하지 않은 요청")
            return
        }
        let url = httpRequest.description
        let method = httpRequest.httpMethod ?? "unknown method"
        var log = "----------------------------------------------------\n[\(method)] \(url)\n----------------------------------------------------\n"
        log.append("API: \(target)\n")
        if let headers = httpRequest.allHTTPHeaderFields, !headers.isEmpty {
            log.append("header: \(headers)\n")
        }
        if let body = httpRequest.httpBody, let bodyString = String(bytes: body, encoding: String.Encoding.utf8) {
            log.append("\(bodyString)\n")
        }
        log.append("------------------- END \(method) --------------------------")
        print(log)
    }
    
    // 🔥 Response 를 받은 후.
    func didReceive(_ result: Result<Response, MoyaError>, target: TargetType) {
        switch result {
        case let .success(response):
            onSuceed(response, target: target, isFromError: false)
        case let .failure(error):
            onFail(error, target: target)
        }
    }
    
    func onSuceed(_ response: Response, target: TargetType, isFromError: Bool) {
        let request = response.request
        let url = request?.url?.absoluteString ?? "nil"
        let statusCode = response.statusCode
        
        var log = "------------------- 네트워크 통신 성공(isFromError: \(isFromError)) -------------------"
        log.append("\n[\(statusCode)] \(url)\n----------------------------------------------------\n")
        log.append("API: \(target)\n")
        response.response?.allHeaderFields.forEach {
            log.append("\($0): \($1)\n")
        }
        if let reString = String(bytes: response.data, encoding: String.Encoding.utf8) {
            log.append("\(reString)\n")
        }
        log.append("------------------- END HTTP (\(response.data.count)-byte body) -------------------")
        print(log)
        
        // 🔥 401 인 경우 리프레쉬 토큰 + 액세스 토큰 을 가지고 갱신 시도.
        switch statusCode {
        case 401:
            let acessToken = UserDefaults.standard.string(forKey: Const.UserDefaultsKey.accessToken)
            let refreshToken = UserDefaults.standard.string(forKey: Const.UserDefaultsKey.refreshToken)
            // 🔥 토큰 갱신 서버통신 메서드. 
            userTokenReissueWithAPI(request: UserReissueToken(accessToken: acessToken ?? "",
                                                              refreshToken: refreshToken ?? ""))
        default:
            return
        }
    }
    
    func onFail(_ error: MoyaError, target: TargetType) {
        if let response = error.response {
            onSuceed(response, target: target, isFromError: true)
            return
        }
        var log = "네트워크 오류"
        log.append("<-- \(error.errorCode) \(target)\n")
        log.append("\(error.failureReason ?? error.errorDescription ?? "unknown error")\n")
        log.append("<-- END HTTP")
        print(log)
    }
}

// 🔥 Network.
extension MoyaLoggerPlugin {
    func userTokenReissueWithAPI(request: UserReissueToken) {
        UserAPI.shared.userTokenReissue(request: request) { response in
            switch response {
            case .success(let data):
                // 🔥 성공적으로 액세스 토큰, 리프레쉬 토큰 갱신.
                if let tokenData = data as? UserReissueToken {
                    UserDefaults.standard.set(tokenData.accessToken, forKey: Const.UserDefaultsKey.accessToken)
                    UserDefaults.standard.set(tokenData.refreshToken, forKey: Const.UserDefaultsKey.refreshToken)
                    
                    print("userTokenReissueWithAPI - success")
                }
            case .requestErr(let statusCode):
                // 🔥 406 일 경우, 리프레쉬 토큰도 만료되었다고 판단.
                if let statusCode = statusCode as? Int, statusCode == 406 {
                    // 🔥 로그인뷰로 화면전환. 액세스 토큰, 리프레쉬 토큰, userID 삭제. 
                    let loginVC = UIStoryboard(name: Const.Storyboard.Name.login, bundle: nil).instantiateViewController(withIdentifier: Const.ViewController.Identifier.loginViewController)
                    UIApplication.shared.windows.first {$0.isKeyWindow}?.rootViewController = loginVC
                    
                    UserDefaults.standard.removeObject(forKey: Const.UserDefaultsKey.accessToken)
                    UserDefaults.standard.removeObject(forKey: Const.UserDefaultsKey.refreshToken)
                    UserDefaults.standard.removeObject(forKey: Const.UserDefaultsKey.userID)
                }
                print("userTokenReissueWithAPI - requestErr: \(statusCode)")
            case .pathErr:
                print("userTokenReissueWithAPI - pathErr")
            case .serverErr:
                print("userTokenReissueWithAPI - serverErr")
            case .networkFail:
                print("userTokenReissueWithAPI - networkFail")
            }
        }
    }
}
```
