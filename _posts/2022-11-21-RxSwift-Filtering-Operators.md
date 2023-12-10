---
title: (RxSwift) Filtering Operators
author: Cody
date: 2022-11-21 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---

우리가 서버로부터 fetch해온 JSON데이터를 파싱하고 가공해서 사용하듯이,

생성한 `Observable`(혹은 `Subject`, `Relay`. 이하 `Observable`)을 `Subscribe`해서

방출된 값을 사용하기 전에 값을 가공해서 사용해야하는데,

`RxSwift`에서는 이를 위한 다양한 `operator`들을 제공하고 있습니다.

(https://reactivex.io/documentation/operators.html)

값을 선택적으로 받기 위한 `Filtering Operator`부터 정리해봅니다.

---

### filter

`Swift Collection` 타입에서 제공하는 `filter`와 동일한 기능의 `Operator`입니다.

`filter`클로저에서 `true`인 케이스의 `next`이벤트를 받도록 합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0c055674-5edf-41fd-8fe3-ac05c3233a6c)

```swift
let disposeBag = DisposeBag()

print("- - - - - filter - - - - -")
Observable.of("MintChoco", "Wasabi", "MintChoco", "HoneyButter", "MintChoco", "MintChoco")
    .filter { $0 != "MintChoco" }
    .subscribe(onNext: {
        print("eat", $0, "Almond")
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - filter - - - - -
eat Wasabi Almond
eat HoneyButter Almond
```

### ignoreElements

모든 `next`이벤트를 무시합니다. `error`, `complete`로 종료되는 이벤트만 받습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/fd0d9c50-00ee-4ff3-bbaf-b90349c735b0)

`ignoreElements`의 예시입니다.

```swift
print("- - - - - ignoreElements - - - - -")
let takeAlmond = PublishSubject<String>()

takeAlmond
    .ignoreElements()
    .subscribe {
        print("take Wasabi Almond")
    }
    .disposed(by: disposeBag)

takeAlmond.onNext("MintChoco Almond is here")
takeAlmond.onNext("MintChoco Almond is here")
takeAlmond.onNext("MintChoco Almond is here")

takeAlmond.onCompleted()
```

(출력)

```
- - - - - ignoreElements - - - - -
take Wasabi Almond
```

### elementAt

`subscribe`한 후 `n`번째 인덱스의 `Element`만 받습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a0f3545f-a899-4ab8-bff5-b92da86b79d0)

`elementAt`의 예시

```swift
print("- - - - - elementAt - - - - -")
let reTakeAlmond = PublishSubject<String>()

reTakeAlmond
    .element(at: 2)
    .subscribe(onNext: {
        print("eat", $0, "Almond")
    })

reTakeAlmond.onNext("MintChoco")
reTakeAlmond.onNext("MintChoco")
reTakeAlmond.onNext("Wasabi")
reTakeAlmond.onNext("MintChoco")
```

(출력)

```
- - - - - elementAt - - - - -
eat Wasabi Almond
```

---

### skip

`n`번 `next`이벤트를 `skip`시킵니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/60a30c1e-7f9d-4cde-974d-e1e23480ff7c)

```swift
print("- - - - - skip - - - - -")
Observable.of("MintChoco", "MintChoco", "MintChoco", "HoneyButter", "Wasabi", "Buldak")
    .skip(3)
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - skip - - - - -
HoneyButter
Wasabi
Buldak
```

### skipWhile

`while`클로저가 `true`인 동안 `skip`을 시키고, `false`가 된 순간부터 `next`이벤트를 받습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1682fab7-fe76-48a2-b5d6-4c0c2317cc45)

```swift
print("- - - - - skipWhile - - - - -")
Observable.of("MintChoco", "MintChoco", "MintChoco", "HoneyButter", "Wasabi", "Corn")
    .skip(while: {
        $0 == "MintChoco"
    })
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - skipWhile - - - - -
HoneyButter
Wasabi
Corn
```

### skipUntil

`skipUntil`은 조금 특이합니다.

`skipUntil`은 다른 `Observable`을 트리거로 설정해서,

트리거로 설정한 `Observable`이 값을 방출할 때까지 `skip`을 시킵니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/98f6d5e1-216a-4a78-b974-5aa7fad3fa12)

```swift
print("- - - - - skipUntil - - - - -")
let buyAlmond = PublishSubject<String>()
let soldoutMintChoco = PublishSubject<String>() // 트리거가 되어줄 PublishSubject
buyAlmond.skip(until: soldoutMintChoco)
    .subscribe(onNext: {
        print("buy", $0)
    })
    .disposed(by: disposeBag)

buyAlmond.onNext("MintChoco")
buyAlmond.onNext("MintChoco")

soldoutMintChoco.onNext("Soldout MintChoco")

buyAlmond.onNext("Wasabi")
buyAlmond.onNext("Corn")
buyAlmond.onNext("Buldak")
```

(출력)

```
- - - - - skipUntil - - - - -
buy Wasabi
buy Corn
buy Buldak
```

### take

`take`는 `n`개의 `Element`만 받도록 합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8f21e9c9-06a3-4f45-85e6-a546c18525f6)

```swift
print("- - - - - take - - - - -")
Observable.of("Buldak", "Wasabi", "HoneyButter", "MintChoco", "MintChoco", "MintChoco")
    .take(3)
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - take - - - - -
Buldak
Wasabi
HoneyButter
```

### takeWhile

`takeWhile`은 `while`클로저가 `true`인 동안만 `Element`를 받습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/02398b1e-b4d0-4bbc-ad5d-ad9ca9bb887f)

```swift
print("- - - - - takeWhile - - - - -")
Observable.of("Buldak", "Wasabi", "HoneyButter", "MintChoco", "MintChoco", "MintChoco")
    .take(while: {
        $0 != "MintChoco" // MintChoco가 나오면 종료
    })
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - takeWhile - - - - -
Buldak
Wasabi
HoneyButter
```

### takeUntil

`takeUntil`은 `skipUntil`과 비슷하게,

다른 `Observable`을 트리거로 설정하여, 트리거로 설정한 `Observable`이 값을 방출할 때까지만 값을 받습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2dafc270-2fb7-488d-bc09-9fd3faf4ef35)

(출력)

```
- - - - - takeUntil - - - - -
Wasabi
HoneyButter
```

### distinctUntilChanged

`distinctUntilChanged`는 기존에 받은 `Element`와 다른 `Element`가 들어왔을 때만 값을 받게 해주어

중복된 값을 받지 않도록 막아줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/412873f1-ae93-4477-8420-152670d42493)

```swift
print("- - - - - distinctUntilChanged - - - - -")
Observable.of("Buldak", "Wasabi", "Wasabi", "HoneyButter", "MintChoco", "MintChoco", "MintChoco", "Wasabi")
    .distinctUntilChanged()
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - distinctUntilChanged - - - - -
Buldak
Wasabi
HoneyButter
MintChoco
Wasabi
```