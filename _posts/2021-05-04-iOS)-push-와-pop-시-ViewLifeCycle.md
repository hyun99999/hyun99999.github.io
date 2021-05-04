---
title:  "iOS) push 와 pop 시 ViewLifeCycle"
categories:
- iOS

date:   2021-05-04 19:40:00 +0900
author_profile: false
---
push 와 pop 시 ViewLifeCycle

### 문제
- viewDidLoad 에서 realm 에서 데이터를 가져와서 뷰에 뿌려주는 형식인데 이 경우 push 하고 pop 해서 뷰로 돌아올 경우 업데이트가 되지 않았다.

### 원인
- push 할 경우에 view 가 사라지지만 메모리에는 남아있다는 것을 놓쳤다. 즉, viewDidLoad 는 한번만 호출되는데 notificationToken 을 viewDidLoad 에서 설정해주었던 것이 문제였다.

- `viewWillDisappear` 에서 notificationToken 을 invalidate() 하기 때문에 뷰가 사라질 때 저장한 notificationToken 이 해제가 되었다.

- 그 후에 push 된 뷰에서 db 를 수정하고 pop 해서 메인 뷰를  `viewWillAppear` 해주기 때문에 `viewDidLoad` 에서 설정해둔 `reloadData()` 가 실행되지 않았고 db 가 업데이트가 안되는 현상이 보여졌던 것이다.

- push

<img src = "https://user-images.githubusercontent.com/69136340/116993114-f9af5a00-ad11-11eb-9c99-265a140b64bb.png" width = "500">

- pop

<img src ="https://user-images.githubusercontent.com/69136340/116993101-f4520f80-ad11-11eb-90cc-4cfe59bc7f6d.png" width ="500">

### 해결

`viewWillAppear` 에서 notificationToken 을 설정해서 pop 되어 뷰가 화면에 보여질 때 `reloadData()` 가 실행되도록 하였다.

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    realm = try? Realm()
    Lists = realm?.objects(ListModel.self)
    
    print("Lists = \(Lists)")
    
    notificationToken = Lists?.observe { change in
        print("change: \(change)")
        self.tableView.reloadData()
    }
}
```

그렇다면 present 할때는?

- present 할때는 정상적으로 업데이트가 가능하다. 이때는 뒤의 뷰가 사라지지 않아서 deinit 역시 하지 않기 때문이다.
