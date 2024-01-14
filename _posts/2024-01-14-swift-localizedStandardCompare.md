---
title: (Swift) localizedStandardCompare
author: Cody
date: 2024-01-14 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

## localizedStandardCompare(_:)

[localizedStandardCompare(_:)](https://developer.apple.com/documentation/foundation/nsstring/1409742-localizedstandardcompare)  
Finder의 파일 정렬처럼 String을 비교해주는 메서드.

### 예시

아래는 기존 `sorted()` 메서드의 예시.
```swift
let fileNames = [
    "파일이름 100.txt",
    "파일이름 5.txt",
    "파일이름 20.txt"
]

let sortedFileName = fileNames.sorted()
sortedFileName
    .forEach {
        print($0)
    }
```
결과
```
파일이름 (100).txt
파일이름 (20).txt
파일이름 (5).txt
```
하지만 우리가 파일 정렬에서 원하는 건 파일명 뒤의 숫자가 텍스트 순서가 아닌 Integer의 순서로 정렬이 되는 것을 원함.  
이럴 때 `sorted(by:)`의 클로저에 `localizedStandardCompare(_:)`메서드를 사용하면,
```swift
let fileNames = [
    "파일이름 100.txt",
    "파일이름 5.txt",
    "파일이름 20.txt"
]

let localizedStandardCompareFileName = fileNames.sorted {
    $0.localizedStandardCompare($1) == .orderedAscending
}
localizedStandardCompareFileName
    .forEach {
        print($0)
    }
```
우리가 원하는 파일 정렬의 규칙으로 정렬이 됨.
```
파일이름 (5).txt
파일이름 (20).txt
파일이름 (100).txt
```

### discussion
> 이 정렬의 정확한 동작은 locale에 따라 다르다고 함.
{: .prompt-warning }
