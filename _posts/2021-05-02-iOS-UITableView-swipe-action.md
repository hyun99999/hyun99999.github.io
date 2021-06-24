---
title:  "iOS) UITalbeViewCell SwipeAction"
categories:
- iOS

date:   2021-05-02 14:01:00 +0900
author_profile: false
---
UITalbeViewCell SwipeAction

### UITableViewDatasource - trailingSwipeActionConfigurationForRowAt

<img src = "https://user-images.githubusercontent.com/69136340/116802652-629aa480-ab4f-11eb-8fe8-d1f51c9424ff.png" width="250">

```swift
func tableView(_ tableView: UITableView, trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath) -> UISwipeActionsConfiguration? {
    //info
    let info = UIContextualAction(style: .normal, title: "세부사항") { action, view, completion in
    //세부사항을 클릭하면 모달창을 띄우게 함.
        guard let nextVC = self.storyboard?.instantiateViewController(identifier: DetailReminderVC.identifier) as? DetailReminderVC else {
                return
            }
        //modally
        self.present(nextVC, animated: true, completion: nil)
    }
    //delete 
    let delete = UIContextualAction(style: .destructive, title: "삭제") { action, view, completion in
    //tableContents 라는 배열에 정보를 저장했다.
        self.tableContents.remove(at: indexPath.row)
        tableView.deleteRows(at: [indexPath], with: .automatic)
    }

    return UISwipeActionsConfiguration(actions: [delete, info])
}
```
