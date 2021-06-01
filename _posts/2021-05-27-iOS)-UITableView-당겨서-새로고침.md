---
title:  "iOS) UITableView 당겨서 새로고침"
categories:
- iOS

date:   2021-05-27 23:44:00 +0900
author_profile: false
---
### UITableView 당겨서 새로고침

```swift
//MARK: - View Life Cycle
override func viewDidLoad() {
    //... 
    setUI()
}

//MARK: - @obcj Methods
@objc
func pullToRefresh(refresh: UIRefreshControl) {
    print("pullToRefresh()")
    refresh.endRefreshing()

    //새로고침 시 적용하고 싶은 코드.

    tableView.reloadData()
}

private func setUI() {
//...
// 당겨서 새로고침
        let refreshControl = UIRefreshControl()
// 이미지 안보이게 하기
//  refreshControl.tintColor = .clear
// 문구 넣기
//  self.refreshControl.attributedTitle = NSAttributedString(string: "당겨서 새로고침")
    refreshControl.addTarget(self, action: #selector(pullToRefresh(refresh:)), for: .valueChanged)
    tableView.refreshControl = refreshControl
}
```
