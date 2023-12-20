---
title: (Swift) Collection Type (3)Dictionary
author: Cody
date: 2022-04-17 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

Swift의 `Dictionary`타입은 `Foundation`의 `NSDictionary`를 bridge한 타입.

### 빈 Dictionary 생성

```swift
var namesOfIntegers = [Int: String]()
namesOfIntegers[16] = "sixteen"

namesOfIntegers = [:]

```

### 리터럴로 Dictionary 생성

```dart
var airports: [String: String] = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]

```

### Dictionary 접근, 변경

```swift
print("The airports dictionary contains \(airports.count) items.")

// 빈 Dictionary 확인
if airports.isEmpty {
    print("The airports dictionary is empty.")
} else {
    print("The airports dictionary is not empty.")
}

// 값 할당
airports["LHR"] = "London"
```