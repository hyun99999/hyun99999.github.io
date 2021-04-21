---
title:  "iOS) modal 창 뒤에 View 없애지 않기"
categories:
- iOS

date:   2021-04-22 00:11:00 +0900
author_profile: false
---
### iOS) modal 창 뒤에 View 없애지 않기

`UIModalPresentationStyle` 중 `.fullScreen` 과 `.currentContext` 를 통해서 화면 위 끝까지 덮는 모달창을 구현했었다. 하지만 drag down dismiss 기능을 추가하니까 뒤에 화면이 검은색이 되었다. 즉 전에 있던 View 는 사라진 상태였다. 그래서 알아보았다.

- `.fullScreen` 과 `.currentContext` 는 뷰가 present 될 때 지시하는 뷰컨트롤러의 뷰를 없애버린다.
- `.overFullScreen` 과 `.overCurrentContext` 는 뷰를 컨텍스트에서 없애지 않고 유지한 상태에서 present 하기 때문에 새로운 뷰의 alpah 값을 조절하면 뒤의 뷰를 비치게 보일 수 있다.

### 해결
나는 alpha 값을 1 로 설정해서 `.overFullScreen` 를 통해 구현하기로 했다.

```swift
let myTabVC = UIStoryboard.init(name: "MyTab", bundle: nil)
guard let nextVC = myTabVC.instantiateViewController(identifier: "MyTabVC") as? MyTabVC else {
    return
}
//overFullScreen, overCurrentContext 는 반투명도를 조절해서 뒤의 view 를 볼 수 있다.
nextVC.modalPresentationStyle = .overFullScreen
nextVC.modalTransitionStyle = .coverVertical
nextVC.view.alpha = 1
self.present(nextVC, animated: true, completion: nil)
```
<img src = "https://user-images.githubusercontent.com/69136340/115578198-0a1e0880-a300-11eb-808c-202db99fcf5b.png" width ="250">

