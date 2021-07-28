---
title:  "iOS) UISearchController 의 검색 로직을 알아보자"
categories:
- iOS

date:   2021-07-27  17:28:00 +0900
author_profile: false
---
먼저, UISearchController 에 대해서 읽어보자.

[iOS ) UISearchController (2)](https://zeddios.tistory.com/1199)

**핵심 로직) SearchBar에 Text가 업데이트 될 때 마다 검색 결과를 필터링**

```swift
extension ViewController: UISearchResultsUpdating {
    func updateSearchResults(for searchController: UISearchController) {
        guard let text = searchController.searchBar.text?.lowercased() else { return }
        self.filteredArr = self.arr.filter { $0.lowercased().contains(text) }
    }
}

// 출처: https://zeddios.tistory.com/1199 [ZeddiOS]
```

- 추가적으로 apple developer 에서 제공하는 UISearchController 예시 프로젝트를 다운받아서 코드를 살펴보았다. 그랬더니 위의 글에서 소개했던 검색 결과를 보여주는 알고리즘이 조금은 달라서 살펴보려고 한다.

[Apple Developer Documentation](https://developer.apple.com/documentation/uikit/view_controllers/displaying_searchable_content_by_using_a_search_controller)

**핵심 로직) SearchBar에 Text가 업데이트 될 때 마다 검색 결과를 필터링**

`NSComparisonPredicate` 와 함께 `UISearchResultsUpdating` 프로토콜을 사용하여 그룹에서 검색 결과를 필터링합니다. `NSComparisonPredicate` 는 검색 기준을 사용하여 데이터를 가져오거나 필터링하는 방법을 지정하는 기본 클래스입니다. 검색 기준은 제품 제목, 출시 연도 및 가격의 조합이 될 수 있는 search bar 에 사용자가 입력한 내용을 기반으로 한다.

```swift
extension MainTableViewController: UISearchResultsUpdating {

    // ...

    func updateSearchResults(for searchController: UISearchController) {
        // Update the filtered array based on the search text.
        let searchResults = products

                // 1
        // Strip out all the leading and trailing spaces.
        let whitespaceCharacterSet = CharacterSet.whitespaces
        let strippedString =
            searchController.searchBar.text!.trimmingCharacters(in: whitespaceCharacterSet)
        let searchItems = strippedString.components(separatedBy: " ") as [String]
                
                // 2
        // Build all the "AND" expressions for each value in searchString.
        let andMatchPredicates: [NSPredicate] = searchItems.map { searchString in
            findMatches(searchString: searchString)
        }

                // 3
        // Match up the fields of the Product object.
        let finalCompoundPredicate =
            NSCompoundPredicate(andPredicateWithSubpredicates: andMatchPredicates)

        let filteredResults = searchResults.filter { finalCompoundPredicate.evaluate(with: $0) }
                
                // 4
        // Apply the filtered results to the search results table.
        if let resultsController = searchController.searchResultsController as? ResultsTableController {
            resultsController.filteredProducts = filteredResults
            resultsController.tableView.reloadData()
        }
    }
}
```

**주석)**

1 : 검색을 준비하기 위해서 search bar content 는 선행 및 후행 공백문자로 잘린다.

2 : findMatches() 메서드를 통해서 띄어쓰기로 구분된 searchString 으로 OR predicate 를 만들고 "AND" 표현식 작성을 준비한다. (custom 메서드이고 아래에 첨부해뒀다.) OR predicate 를 만드는 이유는 검색어가 리스트의 속성 하나만이 아닌 리스트가 가진 name, yearIntroduced, introPrice 속성에 대해서 검색이 가능하도록 로직을 구성했기 때문이다.(ex) Gladiolus 51.99 2001 라는 정보에 대해서 Gladioulus / 51.99 / 2001 무엇으로 검색해도 나오도록 구성.)

3 : OR predicate 를 AND predicate 를 만들고 searchResults 에 매치시킨다. 

`NSCompoundPredicate` - 다른 predicate 의 논리적 조합을 평가하는 특수 predicate 이다.

`NSCompoundPredicate(andPredicateWithSubpredicates:)`

init(andPredicateWithSubpredicates subpredicates: [NSPredicate]) : 주어진 배열을 통해서  predicate 들을 AND 연산하고 새 predicate 를 만든다.

4 : 검색 결과는 필터링된 목록(filteredResults)으로 searchResultsController 에 전달된다.

- findMatches() : 파라미터 searchString 으로 OR predicate 를 반환하는 함수

```swift
private func findMatches(searchString: String) -> NSCompoundPredicate {
        /** Each searchString creates an OR predicate for: name, yearIntroduced, introPrice.
            Example if searchItems contains "Gladiolus 51.99 2001":
                name CONTAINS[c] "gladiolus"
                name CONTAINS[c] "gladiolus", yearIntroduced ==[c] 2001, introPrice ==[c] 51.99
                name CONTAINS[c] "ginger", yearIntroduced ==[c] 2007, introPrice ==[c] 49.98
        */
        var searchItemsPredicate = [NSPredicate]()
        
        /** Below we use NSExpression represent expressions in our predicates.
            NSPredicate is made up of smaller, atomic parts:
            two NSExpressions (a left-hand value and a right-hand value).
        */
        
        // Name field matching.
        let titleExpression = NSExpression(forKeyPath: ExpressionKeys.title.rawValue)
        let searchStringExpression = NSExpression(forConstantValue: searchString)
        
        let titleSearchComparisonPredicate =
            NSComparisonPredicate(leftExpression: titleExpression,
                                  rightExpression: searchStringExpression,
                                  modifier: .direct,
                                  type: .contains,
                                  options: [.caseInsensitive, .diacriticInsensitive])
        
        searchItemsPredicate.append(titleSearchComparisonPredicate)
        
        let numberFormatter = NumberFormatter()
        numberFormatter.numberStyle = .none
        numberFormatter.formatterBehavior = .default
        
        // The `searchString` may fail to convert to a number.
        if let targetNumber = numberFormatter.number(from: searchString) {
                        // Use `targetNumberExpression` in both the following predicates.
                        let targetNumberExpression = NSExpression(forConstantValue: targetNumber)
            
                        // The `yearIntroduced` field matching.
            let yearIntroducedExpression = NSExpression(forKeyPath: ExpressionKeys.yearIntroduced.rawValue)
                        let yearIntroducedPredicate =
                            NSComparisonPredicate(leftExpression: yearIntroducedExpression,
                                                  rightExpression: targetNumberExpression,
                                                  modifier: .direct,
                                                  type: .equalTo,
                                                  options: [.caseInsensitive, .diacriticInsensitive])
            
                        searchItemsPredicate.append(yearIntroducedPredicate)
            
                        // The `price` field matching.
                        let lhs = NSExpression(forKeyPath: ExpressionKeys.introPrice.rawValue)
            
                        let finalPredicate =
                            NSComparisonPredicate(leftExpression: lhs,
                                                  rightExpression: targetNumberExpression,
                                                  modifier: .direct,
                                                  type: .equalTo,
                                                  options: [.caseInsensitive, .diacriticInsensitive])
            
                        searchItemsPredicate.append(finalPredicate)
        }
        
        let orMatchPredicate = NSCompoundPredicate(orPredicateWithSubpredicates: searchItemsPredicate)
        
        return orMatchPredicate
    }
```
