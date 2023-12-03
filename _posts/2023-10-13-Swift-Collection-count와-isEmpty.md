---
title: (Swift) Collection.count와 isEmpty
author: Cody
date: 2023-10-13 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---
[https://developer.apple.com/documentation/swift/collection/count-4l4qk](https://developer.apple.com/documentation/swift/collection/count-4l4qk%5D)

Collection의 Element를 세어주는 count의 복잡도는

기본적으로 O(n) 연산.

`RandomAccessCollection`을 준수하는 경우 O(1) 연산.

Collection이 비어있는지 여부를 체크할 때는

`Collection.count == 0` 으로 체크하는 것보다,

`Collection.isEmpty` 로 체크하는 것이 더 좋음.

```swift
if someArray.count == 0 { // ❌
   ...
}

if someArray.isEmpty { // ⭕️
   ...
}
```