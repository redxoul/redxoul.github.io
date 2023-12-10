---
title: (Swift) String(_:radix:uppercase:)
author: Cody
date: 2023-01-17 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---
코딩테스트 풀이할 때 유용했던 ``String`` 생성자입니다.

입력한 `value`를 `N`진수(`radix`)로 변환해줍니다.

`11`진수 이상일 때 `uppercase`로 반환해주는 `parameter`도 제공합니다.

아래는 예시입니다.

```swift
String(10, radix: 2) // 1010
String(999999, radix: 16) // f423f
String(999999, radix: 16, uppercase: true) // F423F
```