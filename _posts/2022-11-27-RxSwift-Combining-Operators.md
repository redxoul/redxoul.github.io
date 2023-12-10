---
title: (RxSwift) Combining Operators
author: Cody
date: 2022-11-27 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---

이번 글에서는 두개 이상의 `Observable`을 결합하기 위한 `Combining Operator`들을 정리해보려 합니다.

### zip

`zip`은 묶인 `Observable`들의 결과값을 쌍으로 묶어서 내보내줍니다.

쌍이 맞지 않은 결과값은 쌍이 맞을 때까지 방출될 수 없게 됩니다.

`zip`으로 묶이는 `Observable`들은 데이터 타입이 달라도 상관이 없습니다.
![짝이 맞지 않는 5는 방출이 되지 않음](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d8969957-cf5b-4371-bb8f-014348eda36e)
짝이 맞지 않는 5는 방출이 되지 않음

```swift
print("- - - - - zip - - - - -")
enum Whose {
    case mine
    case yours
}

let almond = Observable<String>.of("HoneyButter", "Wasabi", "MintChoco", "Corn", "Buldak")
let whose = Observable<Whose>.of(.mine, .mine, .yours, .mine, .mine)

let almondEating = Observable
    .zip(almond, whose) { almond, whose in
        return almond + " is \(whose)"
    }

almondEating
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - zip - - - - -
HoneyButter is mine
Wasabi is mine
MintChoco is yours
Corn is mine
Buldak is mine
```

### merge

2개 이상의 `Observable`을 하나의 `Observable`로 만들어주며, `merge`하려는 `Observable`의 타입이 같아야 합니다.

`merge`한 시퀀스들을 동시에 `subscribe`하고 `element`가 들어오는 즉시 방출되어 어떤 것이 먼저 들어올지 순서를 알 수는 없습니다.

내부 시퀀스들이 모두 `complete`가 될 때 `merge`시퀀스가 종료가 되며,

내부 시퀀스들 중 하나가 `error`를 방출시키면 `merge Observable`은 바로 `error`를 `relay`하고 종료시켜버립니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ab311070-933d-482f-9a86-4ebb77bc837a)

```swift
enum Almond {
    case honeyButter
    case wasabi
    case mintChoco
    case buldak
    case corn
    case garlic
    case onion
    case caramel
}

print("- - - - - merge(1) - - - - -")

let almond1 = Observable<Almond>.from([.honeyButter, .wasabi, .mintChoco, .buldak])
let almond2 = Observable<Almond>.from([.corn, .garlic, .onion, .caramel])

Observable.of(almond1, almond2)
    .merge()
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

`merge Observable`로 방출되는 순서를 예측할 수 없는 것을 확인.

```
- - - - - merge(1) - - - - -
honeyButter
wasabi
corn
mintChoco
garlic
buldak
onion
caramel
```

`merge`가 `내부 Observable`들을 동시에 `subscribe`한다고 했는데요.

만약 동시에 `subscribe`할 갯수를 제한하고 싶다면 `merge(maxConcurrent:)`를 사용하면 됩니다.

`maxConcurrent`만큼의 `Observable`만 동시에 `subscribe`하게 됩니다.

네트워크 요청이 많을 때 리소스 제한을 둔다거나, 연결될 수를 제한하고자 할 때 유용합니다.

아래처럼 `maxConcurrent`를 `1`로 설정하면,

첫번째 Observable(`almond1`)이 끝나야 두번째 Observable(`almond2`)를 `subscribe`하게 되어 순서가 보장됩니다.

```swift
print("- - - - - merge(2) - - - - -")
Observable.of(almond1, almond2)
    .merge(maxConcurrent: 1)
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

첫번째 시퀀스가 다 끝나고 두번째 시퀀스가 오는 것을 확인.

```
- - - - - merge(2) - - - - -
honeyButter
wasabi
mintChoco
buldak
corn
garlic
onion
caramel
```

### combineLatest

2개 이상의 `Observable`을 묶어주는데, 각 `Observable`이 마지막으로 방출한 `Element`를 묶어서 방출시켜줍니다.

최대 8개까지 묶는 것이 가능합니다.

여러개의 `TextField`값을 하나의 `Observable`로 조합하여 관리하는 상황에서 유용합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d8bb450f-eac3-43b6-a37c-ac5866dc727b)

```swift
print("- - - - - combineLatest - - - - -")
let dateFormat = PublishSubject<DateFormatter.Style>()
let current = Observable<Date>.of(Date())

let currentDateString = Observable
    .combineLatest(
        dateFormat,
        current,
        resultSelector: { format, date -> String in
            let dateFormatter = DateFormatter()
            dateFormatter.dateStyle = format
            return dateFormatter.string(from: date)
        }
    )

currentDateString
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

dateFormat.onNext(.short)
dateFormat.onNext(.long)
```

(출력)

```
- - - - - combineLatest - - - - -
11/26/22
November 26, 2022
```

### concat

`concat`은 두개이상의 시퀀스를 이어붙이고 싶을 때 사용합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6ca158e5-f1ce-430b-8024-1cfe23200251)

`concat`에서는 두가지 방법이 있는데

첫번째 방법은 새 `Observable`에 `concat`으로 붙일 `Observable`들을 배열로 넣는 방법입니다.

```swift
print("- - - - - concat - - - - -")
let package1 = Observable<Almond>.of(.wasabi, .onion, .honeyButter)
let package2 = Observable<Almond>.of(.caramel, .corn, .garlic)

let packageSet = Observable
    .concat([package1, package2])

packageSet
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

두번째 방법은 첫번째 `Observable`에 `concat`으로 뒤에 따라 붙일 `Observable`을 넣는 방법입니다.

```swift
package1
    .concat(package2)
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - concat - - - - -
wasabi
onion
honeyButter
caramel
corn
garlic
```

### startWith

`startWith`는 `concat`의 변형으로

`Observable`을 사용할 때 초기값으로 확실히 보장받고 싶은 `Element`가 있을 때 유용합니다.

`startWith`의 위치는 `subscribe`이전이라면 어느곳이든 상관없습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f6455bbc-a766-48f2-bf44-761bab8ce9fb)

```swift
print("- - - - - startWith - - - - -")
let almondPackage = Observable<Almond>.of(.honeyButter, .wasabi, .corn)

almondPackage
    .enumerated()
    .map { index, element in
        "(\(index+1)) \(element)"
    }
    .startWith("(0) \(Almond.garlic)")
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - startWith - - - - -
(0) garlic
(1) honeyButter
(2) wasabi
(3) corn
```

### concatMap

`concatMap`은 이전글에서 정리한 `flatMap`과 유사하지만,

각각의 시퀀스가 다음 시퀀스가 구독되기 전에 모두 `Complete`되는 것을 보장해줍니다.

첫번째 들어온 `내부 Observable`이 종료될 때까지 두번째 들어온 `내부 Observable`이 `subscribe`되는것을 막아줍니다.

첫번째 들어온 `내부 Observable`이 종료되면 두번째 `내부 Observable`을 `subscribe`하기 시작합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/56e15ba2-1bf7-4435-92a6-40ea1b256e26)

```swift
print("- - - - - concatMap - - - - -")
struct Taster {
    var tasting: PublishSubject<Almond>
}

let almondTasting = Taster(tasting: PublishSubject<Almond>())
let almondTasting2 = Taster(tasting: PublishSubject<Almond>())

let tasting = PublishSubject<Taster>()

tasting
    .concatMap { Taster in
        Taster.tasting
    }
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

tasting.onNext(almondTasting)
tasting.onNext(almondTasting2)

almondTasting.tasting.onNext(.honeyButter)

almondTasting2.tasting.onNext(.mintChoco) // 첫번째 subscribe가 완료되기 전이라 무시
almondTasting.tasting.onNext(.corn)
almondTasting.tasting.onNext(.buldak)
almondTasting.tasting.onCompleted() // 첫번쨰 subscribe 완료
almondTasting2.tasting.onNext(.caramel)
almondTasting2.tasting.onNext(.garlic)
```

(출력)

첫번째 `subscribe`(almondTasting)가 완료되기 전에 `next`이벤트를 한 두번째 `Observable`은 아직 `subscribe`전이기 때문에 출력이 되지 않은 것을 확인할 수 있습니다.

만약 `subscribe`전에 방출한 `Element`를 받고 싶다면 `BehaviorSubject`나 `ReplaySubject`를 사용하면 됩니다.

```
- - - - - concatMap - - - - -
honeyButter
corn
buldak
caramel
garlic
```

### withLatestFrom

`withLatestFrom`은 트리거 역할을 하는 `Observable`의 값이 방출이 될 때,

해당 `Element`를 `Second Observable`의 최신값과 함께 방출되도록 하는 `Operator`입니다.

화면에서 사용자 입력을 받고, 버튼을 눌렀을 때 최신 입력값을 전달하고자 하는 상황에서 쓰일 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b967ce51-1657-454b-bb0a-d6784653a1b1)

```swift
print("- - - - - withLatestFrom - - - - -")
enum CasherAction {
    case scan
}

let strangeCasher = PublishSubject<CasherAction>()
let customer = PublishSubject<Almond>()

strangeCasher
    .withLatestFrom(customer) { "\($0) \($1)" }
    .distinctUntilChanged() // 변동이 있을 때만 방출되도록 함
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

customer.onNext(.honeyButter) // scan 전 최신값
strangeCasher.onNext(.scan) // honeyButter와 scan 방출
customer.onNext(.wasabi)
customer.onNext(.corn)
customer.onNext(.buldak) // scan 전 최신값
strangeCasher.onNext(.scan) // buldak과 scan 방출
strangeCasher.onNext(.scan) // 변동이 없어서 아무것도 방출하지 않음
```

(출력)

```
- - - - - withLatestFrom - - - - -
scan honeyButter
scan buldak
```

### sample

`sample`은 위 예제에서 사용한 `withLatestFrom`과 `distinctUntilChanged`를 합쳐놓은 `Operator`입니다.

트리거 `Observable`이 방출이 되었을 때, `Second` `Observable`의 최신 `Element`와 함께 방출이 되지만, 값의 변동이 없다면 방출이 되지 않습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ec03f5fa-9488-49c7-b99b-d48160cea1a8)

```swift
print("- - - - - sample - - - - -")
let strangeCasher2 = PublishSubject<CasherAction>()
let customer2 = PublishSubject<Almond>()

customer2
    .sample(strangeCasher2)
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

customer2.onNext(.wasabi)
strangeCasher2.onNext(.scan) // 최신값 wasabi와 함께 방출
customer2.onNext(.corn)
strangeCasher2.onNext(.scan) // 최신값 corn과 함께 방출
customer2.onNext(.buldak)
customer2.onNext(.honeyButter)
strangeCasher2.onNext(.scan) // 최신값 honeyButter와 함께 방출
strangeCasher2.onNext(.scan) // 변경된 값이 없어서 아무것도 방출되지 않음
```

(출력)

```
- - - - - sample - - - - -
wasabi
corn
honeyButter
```

### amb

`amb`는 `ambiguous`(애매모호한)의 약자입니다.

두 `Observable` 중 어느것을 `Subscribe`할지 애매매호할 때 유용합니다.

두개의 `Observable`로부터 `Element`가 방출될 때를 기다렸다가, 먼저 `Element`를 방출한 `Observable`만 `Subscribe` 되고 나머지 `Observable`은 무시됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f78c9237-b808-4e13-a55d-99d21150c169)

```swift
print("- - - - - amb - - - - -")
let challenger1 = PublishSubject<String>()
let challenger2 = PublishSubject<String>()

let Referee = challenger1.amb(challenger2)

Referee
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

challenger2.onNext("1")
challenger1.onNext("2")
challenger1.onNext("3")
challenger2.onNext("4")
challenger1.onNext("5")
challenger2.onNext("6")
```

(출력)

```
- - - - - amb - - - - -
1
4
6
```

### switchLatest

`외부 Observable`이 마지막으로 방출한 `내부 Observable`에서만 `Element`가 방출됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a1344ba1-e15b-485f-b672-f7b287f1cb35)

```swift
print("- - - - - switchLatest - - - - -")
let student1 = PublishSubject<String>()
let student2 = PublishSubject<String>()
let student3 = PublishSubject<String>()

let handsUp = PublishSubject<Observable<String>>()

let classRoom = handsUp.switchLatest()

classRoom
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)

handsUp.onNext(student1)
student1.onNext("student1: 손들고 발표")
student2.onNext("student2: 손 안들고 말함") // 방출되지 않음
handsUp.onNext(student2)
student2.onNext("student2: 손들고 발표")
student1.onNext("student1: 손 안들고 말함") // 방출되지 않음
```

(출력)

```
- - - - - switchLatest - - - - -
student1: 손들고 발표
student2: 손들고 발표
```

### reduce

`Swift`의 `Collection`타입에서 제공하는 ``reduce``와 동일한 기능의 `Operator`입니다.

간단히 넘어갑니다.

```swift
print("- - - - - reduce - - - - -")
Observable.from((1...10))
    .reduce(0) { summary, newValue in
        return summary + newValue
    } // .reduce(0, accumulator: +) 로 줄일 수 있음
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - reduce - - - - -
55
```

### scan

`scan`은 `reduce`와 유사하지만, `reduce`의 과정마다 `Element`를 방출시켜줍니다.

```swift
print("- - - - - scan - - - - -")
Observable.from((1...10))
    .scan(0, accumulator: +)
    .subscribe(onNext: {
        print($0)
    })
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - scan - - - - -
1
3
6
10
15
21
28
36
45
55
```