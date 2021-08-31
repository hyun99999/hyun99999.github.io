---
title:  "iOS) KakaoQRcode 클론코딩 - shake motion event(2/2) - CMMotionManager"
categories:
- iOS

date:   2021-08-29  22:32:00 +0900
author_profile: false
---
### 내용

-   카카오톡에서 두번 흔들면 qr 코드 뷰를 띄어주는 기능 클론코딩

두가지 방법에 대해서 알아볼 것이다.

1.  UIResponder 의 motion event 메서드를 재정의해서 사용
2.  CoreMotion Framework 사용

---

## 2️⃣ CoreMotion Framework 사용

# 시작 전

## 👏 원리

-   motion service 를 관리하는 `CMMotionManager` 를 활용.

## 👏 CMMotionManager

`[CMMotionManager](https://developer.apple.com/documentation/coremotion/cmmotionmanager)` 객체를 사용하여 장치의 온보드 센서에서 감지한 움직임을 보고하는 서비스를 시작합니다. 이 객체를 사용하여 4가지 유형의 모션 데이터를 수신합니다.

-   **Accelerometer data**, indicating the instantaneous acceleration of the device in three dimensional space.
-   **Gyroscope data**, indicating the instantaneous rotation around the device's three primary axes.
-   **Magnetometer data**, indicating the device's orientation relative to Earth's magnetic field.
-   **Device-motion data**, indicating key motion-related attributes such as the device's user-initiated acceleration, its attitude, rotation rates, orientation relative to calibrated magnetic fields, and orientation relative to gravity. This data is provided by Core Motion’s sensor fusion algorithms.

지정된 **update intervals 로 센서 데이터를 수신**하거나 **센서가 데이터를 수집하고 나중에 검색하기 위해서 저장**하도록 할수도 있다. 이 두가지 방법은 데이터가 더이상 필요없을 때 적절한 중지 메서드를 호출한다(`stopAccelerometerUpdates()`, `stopGyroUpdates()`, `stopMagnetometerUpdates()` 및 `stopDeviceMotionUpdates()`)

### 특정 intervals 로 motion 업데이트 처리

데이터를 수신하기 위해서 앱은 `OperationQueue` 의 인스턴스와 이러한 업데이트를 처리하기 위한 block handler 를 사용하는 start 메서드를 호출한다. 모션 데이터는 block handler 로 전달된다. 업데이트의 빈도는 interval 프로퍼티를 통해서 결정된다.

-   **Gyroscope.** Set the [gyroUpdateInterval](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616160-gyroupdateinterval) property to specify an update interval. Call the [startGyroUpdates(to:withHandler:)](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616104-startgyroupdates) method, passing in a block of type[CMGyroHandler](https://developer.apple.com/documentation/coremotion/cmgyrohandler). Rotation-rate data is passed into the block as [CMGyroData](https://developer.apple.com/documentation/coremotion/cmgyrodata) objects.
-   **Magnetometer.** Set the [magnetometerUpdateInterval](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616089-magnetometerupdateinterval) property to specify an update interval. Call the [startMagnetometerUpdates(to:withHandler:)](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1615968-startmagnetometerupdates) method, passing a block of type [CMMagnetometerHandler](https://developer.apple.com/documentation/coremotion/cmmagnetometerhandler). Magnetic-field data is passed into the block as [CMMagnetometerData](https://developer.apple.com/documentation/coremotion/cmmagnetometerdata) objects.
-   **Device motion.** Set the [deviceMotionUpdateInterval](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616065-devicemotionupdateinterval) property to specify an update interval. Call the [startDeviceMotionUpdates(using:)](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616107-startdevicemotionupdates) or [startDeviceMotionUpdates(using:to:withHandler:)](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616176-startdevicemotionupdates) or [startDeviceMotionUpdates(to:withHandler:)](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616048-startdevicemotionupdates) method, passing in a block of type [CMDeviceMotionHandler](https://developer.apple.com/documentation/coremotion/cmdevicemotionhandler). With the former method (new in iOS 5.0), you can specify a reference frame to be used for the attitude estimates. Rotation-rate data is passed into the block as [CMDeviceMotion](https://developer.apple.com/documentation/coremotion/cmdevicemotion) objects.

### Motion Data 의 주기적 샘플링

주기적 샘플링으로 모션 데이터를 처리하기 위해서 앱은 argument 가 없는 start 메서드를 호출합니다. 그리고 지정된 모션 데이터 유형의 프로퍼티가 가진 모션 데이터에 주기적으로 접근한다. 이 접근 방식은 게임과 같은 앱에서 권장되는 접근방식이다.

accererometer data 를 block 에서 처리하면 오버헤드가 발생하고 대부분의 게임 앱은 프레임을 렌더링할 때 모션 데이터의 최신 샘플에만 관심이 있습니다.

-   **Accelerometer.** Call [startAccelerometerUpdates()](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616171-startaccelerometerupdates) to begin updates and periodically access [CMAccelerometerData](https://developer.apple.com/documentation/coremotion/cmaccelerometerdata) objects by reading the [accelerometerData](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1615992-accelerometerdata) property.
-   **Gyroscope.** Call [startGyroUpdates()](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616156-startgyroupdates) to begin updates and periodically access [CMGyroData](https://developer.apple.com/documentation/coremotion/cmgyrodata) objects by reading the [gyroData](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616154-gyrodata) property.
-   **Magnetometer.** Call [startMagnetometerUpdates()](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1615961-startmagnetometerupdates) to begin updates and periodically access [CMMagnetometerData](https://developer.apple.com/documentation/coremotion/cmmagnetometerdata) objects by reading the [magnetometerData](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616032-magnetometerdata) property.
-   **Device motion.** Call the [startDeviceMotionUpdates(using:)](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616107-startdevicemotionupdates) or [startDeviceMotionUpdates()](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616110-startdevicemotionupdates) method to begin updates and periodically access [CMDeviceMotion](https://developer.apple.com/documentation/coremotion/cmdevicemotion) objects by reading the [deviceMotion](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616040-devicemotion) property. The `startDeviceMotionUpdates(using:)` method (new in iOS 5.0) lets you specify a reference frame to be used for the attitude estimates.

### Hardware Availability and State

hardware 기능(ex. gyroscope) 을 장치에서 사용할 수 없는 경우 해당기능과 관련된 start 메서드를 호출해도 효과가 없다. 적당한 프로퍼티로 하드웨어 기능이 사용가능하거나 활성화되어있는지 확인 할 수 있다.

device-motion 서비스의 경우, [`isDeviceMotionAvailable`](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616094-isdevicemotionavailable), [`isDeviceMotionActive`](https://developer.apple.com/documentation/coremotion/cmmotionmanager/1616158-isdevicemotionactive) 프로퍼티의 값을 체크해서 확인 가능하다.

# 시작하기

우리는 Device-motion data 를 수신해서 사용해보도록 하겠다.

-   MainViewController.swift

```
import UIKit
import SnapKit
// ✅ CMMotionMager 를 사용하기 위해서 import
import CoreMotion

class MainViewController: UIViewController {

// ...

    // MARK: - View Life Cycle

    override func viewDidLoad() {
        super.viewDidLoad()
        configUI()
        setLayout()

        // CMMotionManager 로 motion 을 캡쳐할 것이다
        let coreMotionManager = CMMotionManager()
        getDeviceMotion(coreMotionMager: coreMotionManager)
    }
}

// MARK: - Extensions

extension MainViewController {

    private func configUI(){
        // ...
    }

    private func setLayout() {
        // ...
    }

    @objc
    private func presentToQRCodeVC() {
        let nextVC = QRCodeViewController()
        nextVC.modalPresentationStyle = .overFullScreen
        present(nextVC, animated: true, completion: nil)

    }

// ✅ **shake motion detect with CMMotionMager**

    private func getDeviceMotion(coreMotionMager: CMMotionManager) {
        if coreMotionMager.isDeviceMotionAvailable {
            coreMotionMager.deviceMotionUpdateInterval = 1
            coreMotionMager.startDeviceMotionUpdates(to: .main
            ) { _, _  in
                print("shake")
                // qrcode 뷰가 dismiss 되더라도 motion은 계속 update 되야하기 때문에 stop 메서드를 주석처리해주었다.
//                coreMotionMager.stopDeviceMotionUpdates()
                self.presentToQRCodeVC()
            }
        }
    }
}
```

### 출처 :

[iOS ShakeGesture](https://blog.lenskart.com/ios-shakegesture-ecece76d88cf)

### 깃허브

[GitHub - 28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu: 🧚 아요 미라클론코딩 김현규](https://github.com/28th-SOPT-iOS-CloneCoding/MiraClone-KimHyunGyu)
