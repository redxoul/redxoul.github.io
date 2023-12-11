---
title: (RxCocoa) bind, ControlProperty, Driver, Signal
author: Cody
date: 2022-12-29 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift, RxCocoa]
pin: false
mermaid: true
---

`RxSwift` 정리글에 이어서 `RxCocoa`를 정리를 조금 해봅니다.

`RxSwift`는 플랫폼에 구애받지 않고 익혀두면 어디에서도 적용할 수 있는 공통적인 `Reactive` 사양을 구현해 놓았다면,

`RxCocoa`는 `iOS` 개발에 (아직은) 많은 부분을 차지하고 있는 `Cocoa` `Framework`에 좀 더 특화되어 도움을 주는 클래스들이 있습니다.

이는 `UIKit`들에 반응형 확장(`.rx`)를 추가해서 다양한 이벤트를 `subscribe 할` 수 있게 해 줍니다.

### .rx

`RxCocoa`를 `import 하면`

`UIKit`의 요소들(`UIButton`, `UISwitch`, `UITableView`, ...)의 인스턴스에서

`.rx` 키워드를 통해 해당 `UI`의 동작을 `Reactive 하게` 처리할 수 있도록 해줍니다.

`UISwitch`의 예시입니다.

`UIKit`에서 `UISwitch`, `UIButton` 등 에서 사용자의 입력에 대응하려면 아래와 같이 `selector`를 이용했는데,

```swift
		lazy var someSwitch = UISwitch().then {
        $0.isOn = true
        $0.addTarget(self, action: #selector(switchTapped(sender:)), for: .valueChanged)
    }
    
    @objc func switchTapped(sender: UISwitch) {
        print("is", sender.isOn ? "On!" : "Off!")
    }
```

RxCocoa를 사용하면 `.rx` 를 통해 아래처럼 작성할 수 있습니다.

```swift
		let someSwitch = UISwitch().then {
        $0.isOn = true
    }
    
    
    func bind() {
        someSwitch.rx.isOn
            .observe(on: MainScheduler.instance)
            .subscribe(onNext: { [weak self] isOn in
                self?.someLabel1.text = isOn ? "is On!" : "is Off!"
            })
            .disposed(by: disposeBag)
    }
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/af720060-ee71-4535-b784-32612304186f)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9623ed34-afcb-4bd5-8ce7-be79c542be71)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c806e508-6ea1-40d1-91be-64b0f4bd0a37)

`.rx`는 `RxCocoa` `Extension`을 사용할 수 있도록 해주는 역할이고,

후에 커스텀하게 만들어서 사용할 때도 `rx`를 통해서 사용하게 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/bb02aa08-dde6-4942-8490-6c90900615fe)

`isOn`은 `ControlProperty`입니다.

`ControlProperty`는 `Property`의 변경사항을 시퀀스로 받아올 수 있게 해주는 `ObservableType` 입니다.

그래서 `subscribe`를 통해서 구독할 수 있게 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/fff6bf9c-1455-4a3a-8924-2e2640a3e15a)

### .bind(to:)

`RxCocoa`를 사용하는 여러 가지 이유 중 하나인 `.bind(to:)`입니다.

`.bind`는 `Observer Type`에 바인드시켜 줄 수 있습니다.

`Observer Type`은 값을 주입시켜줄 수 있는 타입입니다.
![Observer 타입을 바인딩 시켜주는 bind(to:)](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a8988476-1e17-451c-8d9c-dd0d4cdc434d)
Observer 타입을 바인딩 시켜주는 bind(to:)

여기서 `Binder`라는 개념이 나옵니다.

`Binder`는 새로운 값을 `accept`할 수는 있지만, `subscribe`할 수는 없는 무언가는 나타내는 유용한 구성이며, 값을 특정 구현이나 개체에 바인딩하는데 자주 사용됩니다.

`rx`확장에서 `Binder`는 `@dynamicMemeberLookup`을 통해서,

`Reactive<Base>`의 모든 `writable` 참조 `property`에 자동으로 만들어지도록 되어 있습니다.

`Binder`는 `Observer Type`으로서 `bind(to:)`를 통해 바인딩을 해줄 수 있다는 것이죠!

'`RxCocoa`가 지원하는 모든 클래스들의 `writable` `property`들은 `rx`확장을 통해 `Binder` 타입으로 쓸 수 있구나!' 하고 생각하면 되겠습니다.

(`@dynamicMemberLookup`도 추후에 정리해 보겠습니다)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f0acf875-88fa-4938-a89d-0d99a8f5ac12)

위의 `UISwitch`를 `UILabel`과 `Bind`시키도록 바꿔보면

아래처럼 작성할 수 있습니다.

```swift
		let someLabel = UILabel().then {
        $0.text = "is On!"
    }
    
    let someSwitch = UISwitch().then {
        $0.isOn = true
    }
    
    func bind() {
        someSwitch.rx.isOn
            .observe(on: MainScheduler.instance)
            .map { $0 ? "is On!" : "is Off!" }
            .bind(to: someLabel.rx.text)
            .disposed(by: disposeBag)
    }
```

스위치를 동작하면 상태값 `Bool`을 받아서 `map`으로 변환 뒤 `Label`의 `Text`에 바인딩을 해줍니다.

아래처럼 `Observer`이자 `Observable`인 `PublishRelay`를 통해서 `bind(to:)`를 해줄 수도 있습니다.

`MVVM`에서 `View`로부터 받은 사용자의 입력을 `ViewModel`로 전달받을 때 사용하는 방법입니다.

`View`로부터의 입력은 `View`가 사라질 때까지 종료되지 않고(`completed`, `error` 이벤트가 발생하지 않고) 값을 받기만 하기 때문에 `PublishRelay`를 사용합니다.

```swift
		let someLabel = UILabel().then {
        $0.text = "is On!"
    }

    let someSwitch = UISwitch().then {
        $0.isOn = true
    }

    let someLabelText = PublishRelay<String>()

    func bind() {
        someLabelText
            .observe(on: MainScheduler.instance)
            .subscribe(onNext: { [weak self] isOn in
                self?.someLabel.text = string
            })
            .disposed(by: disposeBag)
        
        someSwitch.rx.isOn
            .map { $0 ? "is On!" : "is Off!" }
            .bind(to: someLabelText)
            .disposed(by: disposeBag)
    }
```

### Driver와 Signal

`RxSwift`와 함께 `UI`작업을 할 때는 몇가지 특징이 있습니다.

1. 항상 메인스레드에서 동작을 시키기 때문에 `.observeOn(MainScheduler.instance)` 작업을 항상 해주게 됩니다.
2. 클로저 내부에서 `self`를 사용하게 되면 `순환참조`가 발생되어 메모리 누수가 발생할 수 있습니다.
3. `Data`를 처리하다가 `error`가 발생하면 시퀀스가 끊어져버릴 수 있고, 한번 끊어진 시퀀스는 재활용을 할 수 없게 됩니다. 이렇게 되면 오동작을 일으킬 수 있습니다. 그래서 항상 `.catchErrorJustReturn()`을 사용하게 됩니다.

위에 예시문에서도 `.observeOn(MainScheduler.instance)` 를 작성해주었지만,

`UI`작업을 하는 내내 위 처리들을 모두 하기엔 불편하기도 하고 코드 가독성도 좋지 않기 때문에

이와 같은 처리를 고려하지 않아도 되는 `Traits`들이 만들어져 있습니다.

바로 `Driver`와 `Signal`입니다.

`Driver``와 `Signal`은 다음과 같은 특징을 가지고 있습니다.

1. 항상 `MainScheduler`에서 실행되기 때문에 `observeOn`을 작성하지 않아도 됩니다.(백그라운드 스레드에서 `UI`변경이 되지 않을 수 있으므로 주의해야 합니다.)
2. 순환참조 해결을 위한 `[weak self]`, 혹은 `.withUnretained` 처리를 해주지 않아도 됩니다.
3. `Complete`, `Error`도 발생시키지 않기 때문에 시퀀스가 끊어지지 않습니다.

차이점은 `Driver`는  `subscribe`를 하면 새 `Observer`에게 `최신값을 replay`시켜주는 특징이 있고,

`Signal`은 최신값을 `replay`시켜주지는 않는다는 점입니다.

간단한 예제를 보겠습니다.

앞에서 작성한 스위치 예제를 `MVVM`에서 자주 쓰이는 형태로 바꾸었습니다.(Model은 생략)

먼저 `ViewModel`입니다. ViewModel에서는

`Switch`로부터 바인딩하여 값을 받을 `switchIsOn`이 `PublishRelay<Bool>`로 있고,

`switchIsOn`을 `map`으로 데이터 가공을 하고 `asDriver`를 통해 `Driver`로 전환하여 `View`로 값을 전달할 `labelText`가 ``Driver<String>``로 되어 있습니다.

`asDriver`는 `Obsevable`을 `.observeOn(MainScheduler.instance)`와 `.catchErrorJustReturn` 처리를 한번에 하여 `Driver`로 변환시켜주는 역할을 해줍니다.

```swift
class SomeViewModel {
    
    let disposeBag = DisposeBag()
    
    // ViewModel -> View
    let labelText: Driver<String>
    
    // ViewModel <- View
    let switchIsOn = PublishRelay<Bool>()
    
    init() {
        labelText = switchIsOn
            .map { $0 ? "is On!" : "is Off!" }
            .asDriver(onErrorJustReturn: "error!")
    }
}
```

View에서 ViewModel의 Driver와 바인딩을 시켜주기 위해서는 `.bind(to:)` 대신 `.drive()`를 사용합니다.

```swift
class SomeViewController: UIViewController {
    
    let disposeBag = DisposeBag()
    
    var viewModel = SomeViewModel()
    
    let someLabel = UILabel().then {
        $0.text = "is On!"
    }
    
    let someSwitch = UISwitch().then {
        $0.isOn = true
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        
        attribute()
        layout()
        bind(to: viewModel)
    }
    
    func attribute() {
        // (생략)
    }
    
    func layout() {
        // (생략)
    }
    
    func bind(to viewModel: SomeViewModel) {
        
        // ViewModel -> View
        viewModel.labelText
            .drive(someLabel.rx.text) // bind(to:) 대신 drive를 사용
            .disposed(by: disposeBag)
        
        // ViewModel <- View
        someSwitch.rx.isOn
            .bind(to: viewModel.switchIsOn)
            .disposed(by: disposeBag)
    }
    
}
```

(동작 결과는 위 예제들과 동일합니다)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f41ac1fc-4fa3-4bd2-b16a-409073a07470)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8363e87e-4a3c-45f4-b6ed-debee96455e9)

간단한 예제들로 `RxCocoa`에서 주로 쓰이는

`bind`, `ControlProperty`, `Driver`, `Signal`의 개념들을 정리해봤습니다.

샘플 코드: https://github.com/SwiftyCody/RxCocoaSample