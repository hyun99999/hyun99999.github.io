---
title:  "iOS) section 에 따라서 커스텀 셀 설정"
categories:
- iOS

date:   2021-04-28 23:41:00 +0900
author_profile: false
---
section 에 따라서 커스텀 셀 설정

### UITableViewDataSource

section 별로 custom cell 을 리턴해주면 된다.

- IndexPath 의 section 에 따라서 custom 한 cell 을 리턴해주면된다.

> 마찬가지로 didSelectRowAt 메서드에서도 section 에 따라서 cell 을 present 할 수 있다.

```swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {

    if indexPath.section == 0 {
        guard let topCell = tableView.dequeueReusableCell(withIdentifier: ScheduleListTopCell.identifier, for: indexPath) as? ScheduleListTopCell else { return UITableViewCell() }
        topCell.selectionStyle = .none

        return topCell
    }
    else if indexPath.section == 1 {
        guard let middleCell = tableView.dequeueReusableCell(withIdentifier: DetailCell.identifier, for: indexPath) as? DetailCell else { return UITableViewCell() }
        middleCell.accessoryType = .disclosureIndicator
        
        return middleCell
    }
    else {
        guard let bottomCell = tableView.dequeueReusableCell(withIdentifier: EditListCell.identifier, for: indexPath) as? EditListCell else {  return UITableViewCell() }
        bottomCell.accessoryType = .disclosureIndicator
        
        return bottomCell
    }
}
```
