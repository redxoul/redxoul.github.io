---
title: (Swift) 유니코드와 String
author: Cody
date: 2022-04-17 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

Swift의 네이티브 문자열 타입은 유니코드 스칼라 값으로 만들어져 있습니다.

하나의 유니코드를 21비트의 숫자로 구성되어 있습니다.

유니코드는 아래와 같이 결합을 시켜 사용할 수 있습니다.

### 유니코드의 결합

문자끼리의 결합

```swift
let eAcute: Character = "\u{E9}"  // é
let combinedEAcute: Character = "\u{65}\u{301}"  // e +  ́
// eAcute : é, combinedEAcute : é
```

문자와 심볼
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e3505f02-561a-4d52-bd9f-5734f0916c2b)

(원심볼이 제대로 나오지 않아 캡쳐)

심볼과 심볼

```swift
// 지역심볼문자인 U(1F1FA)와 S(1F1F8)를 결합
let regionalIndicatorForUS: Character = "\u{1F1FA}\u{1F1F8}"
// regionalIndicatorForUS : 🇺🇸
```

마지막으로 한글의 자모음 결합

```swift
// 한글의 “한”자를 단독으로 사용했을 때와 ㅎ,ㅏ,ㄴ의 자모를 따로 결합해서 사용
let precomposed: Character = "\u{D55C}"                // 한
let decomposed: String = "\u{1112}\u{1161}\u{11AB}"    // ㅎ, ㅏ, ㄴ
// precomposed : 한, decomposed 한
```

### 유니코드 문자열의 비교

유니코드 문자열의 비교는 결합된 문자열을 기준으로 비교가 됩니다.

```swift
// 유니코드는 결합된 문자열을 가지고 비교
// "Voulez-vous un café?" using LATIN SMALL LETTER E WITH ACUTE
let eAcuteQuestion = "Voulez-vous un caf\u{E9}?"

// "Voulez-vous un café?" using LATIN SMALL LETTER E and COMBINING ACUTE ACCENT
let combinedEAcuteQuestion = "Voulez-vous un caf\u{65}\u{301}?"

if eAcuteQuestion == combinedEAcuteQuestion {
    print("These two strings are considered equal")
}
// These two strings are considered equal 출력
```