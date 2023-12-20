---
title: (Swift) Collection Type (2)Set
author: Cody
date: 2022-04-17 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift]
pin: false
mermaid: true
---

`Set`은 `Array`와 유사하지만 같은 값을 또 넣을 수 없다는 특징이 있는 `Collection Type`입니다.

`Set` 형태로 저장되기 위해서는 반드시 타입이 `hashable`이어야 합니다.

Swift에서 `String`, `Int`, `Double`, `Bool` 같은 기본 타입은 기본적으로 `hashable`입니다.

Swift에서 `Set` 타입은 `Set`으로 선언.

### 빈 Set 생성

```swift
var letters = Set<Character>()
letters.insert("a")

letters = []

```

### 배열 리터럴로 Set 생성

```swift
var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"]

// 타입 추론으로 생략 가능
var favoriteGenres: Set = ["Rock", "Classical", "Hip hop"]

```

### Set 접근, 변경

```swift
print("I have \(favoriteGenres.count) favorite music genres.")

// 빈 값 확인
if favoriteGenres.isEmpty {
    print("As far as music goes, I'm not picky.")
} else {
    print("I have particular music preferences.")
}

// 값 추가
favoriteGenres.insert("Jazz")

// 값 삭제
if let removedGenre = favoriteGenres.remove("Rock") {
    print("\\(removedGenre)? I'm over it.")
} else {
    print("I never much cared for that.")
}

// 값 확인
if favoriteGenres.contains("Funk") {
    print("I get up on the good foot.")
} else {
    print("It's too funky in here.")
}
// It's too funky in here.

```

### Set 순회

```swift
for genre in favoriteGenres {
    print("\(genre)")
}
// Classical
// Hip hop
// Jazz

```

### Set의 명령어

```swift
let oddDigits: Set = [1, 3, 5, 7, 9]
let evenDigits: Set = [0, 2, 4, 6, 8]
let singleDigitPrimeNumbers: Set = [2, 3, 5, 7]

oddDigits.union(evenDigits).sorted()
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
oddDigits.intersection(evenDigits).sorted()
// []
oddDigits.subtracting(singleDigitPrimeNumbers).sorted()
// [1, 9]
oddDigits.symmetricDifference(singleDigitPrimeNumbers).sorted()
// [1, 2, 9]
```
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9c19845e-02ea-439b-8a56-d01f0404d192)

#### Set 멤버 확인 및 동등 비교

`Set`의 동등비교와 맴버 여부를 확인하기 위해 각각 `==` 연산자와 `isSuperset(of:)`, `isStrictSubset(of:)`, `isStrictSuperset(of:)`, `isDisjoint(with:)` 메서드를 사용.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/88085d4a-7c76-441d-94f5-c64b26b9bc83)

- `isSubset(of:)`: `부분집합`인지 확인. 동일한 Set일 때도 true

- `isStrictSubset(of:)`: `진부분집합`인지 확인. 동일하지 않은 Set일 때만 true

- `isSuperset(of:)`: `상위집합`인지 확인. 동일한 Set일 때도 true

- `isStrictSuperset(of:)`: `진상위집합`인지 확인. 동일하지 않은 Set일 때만 true

- `isDisjoint(with:)`: 두집합이 `겹치는 원소가 없을 때` true

```swift
let houseAnimals: Set = ["🐶", "🐱"]
let farmAnimals: Set = ["🐮", "🐔", "🐑", "🐶", "🐱"]
let cityAnimals: Set = ["🐦", "🐭"]

houseAnimals.isSubset(of: farmAnimals)
// true
farmAnimals.isSuperset(of: houseAnimals)
// true
farmAnimals.isDisjoint(with: cityAnimals)
// true
```