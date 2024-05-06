---
title: (Swift) stride 함수
author: Cody
date: 2022-05-01 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

`stride`는

시작값(`from`)에서부터

끝값을 포함(`through`)하거나, 미포함(`to`)할 때까지

지정한 양(`by`)만큼 늘어나는 시퀸스를 반환하는 [제네릭](https://swiftycody.github.io/posts/Swift-Generic/) 함수입니다.

```swift
// stride(from:to:by)는 to를 포함하지 않음
for i in stride(from: 0, to: 5, by: 1) {
    print(i) // 0, 1, 2, 3, 4
}

// stride(from:through:by)는 through를 포함
for i in stride(from: 0, through: 5, by: 1) {
    print(i) // 0, 1, 2, 3, 4, 5
}
```
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/bf706dc8-5454-4ebf-844c-d7fccad5c319)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2589b028-a67c-4248-80c3-9abdbd90ab1d)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/37b46e9b-bdc3-473f-a708-9f932f6a89fe)