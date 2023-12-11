---
title: (RxCocoa) DelegateProxy
author: Cody
date: 2023-01-01 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift, RxCocoa]
pin: false
mermaid: true
---

`iOS` 개발을 하면

반드시 마주하게 되는 패턴 중 하나는 `Delegate` 패턴입니다.

`Cocoa`에서 `Delegate`는 보통

`DataSource`(셀 구현, 셀/섹션 개수, 높이 등)를 대신 구현하게 하거나,

`비동기 응답`(사용자의 셀 선택, Location 업데이트 등)을 받기 위한 메서드를 구현하도록 합니다.

위와 같은 `Delegate` 패턴을

`Rx`를 통해 사용할 수 있도록 만들어주는 방법이 바로 `DelegateProx`입니다.

`DelegateProxy`의 구현을 `MKMapView`를 예시로 보겠습니다.

### 0. `extension`으로 `HasDelegate`를 따르도록 작성
이미 `delegate`가 있는 클래스를 대상으로 하기 때문에 `extension`에 따로 구현해줄 것은 없습니다.

```swift
extension MKMapView: HasDelegate {}
```

### 1. 아래 3가지를 `상속받는 class`(~Proxy로 네이밍)를 필요로 합니다.
- `DelegateProxy<Delegate가 있는 클래스 이름, Delegate 이름>`
- `DelegateProxyType`
- `Delegate 이름`

### 2. 그리고 해당 `class`에 아래 메서드를 구현

```swift
class RxMKMapViewDelegateProxy: DelegateProxy<MKMapView, MKMapViewDelegate>, DelegateProxyType, MKMapViewDelegate {
    
    weak public private(set) var mapView: MKMapView?
    
    public init(mapView: ParentObject) {
        self.mapView = mapView
        super.init(parentObject: mapView,
                   delegateProxy: RxMKMapViewDelegateProxy.self)
    }
    
    static func registerKnownImplementations() {
        register { RxMKMapViewDelegateProxy(mapView: $0) }
    }
}
```

### 3. `Reactive`를 `extension`하여 아래의 것들을 구현해 줍니다.

- (위에서 만들어주었던)`DelegateProxy`로 `delegate 계산프로퍼티` 구현
- (위에서 만들어주었던)`DelegateProxy`로 `setDelegate 메서드` 구현
- `delegate`를 `rx로 바꾸어 사용할 메서드들` 구현

```swift
public extension Reactive where Base: MKMapView {
    var delegate: DelegateProxy<MKMapView, MKMapViewDelegate> {
        RxMKMapViewDelegateProxy.proxy(for: base)
    }
    
    func setDelegate(_ delegate: MKMapViewDelegate) -> Disposable {
        RxMKMapViewDelegateProxy.installForwardDelegate(
            delegate,
            retainDelegate: false,
            onProxyForObject: self.base
        )
    }
    
    // mapView(_:didUpdate:)를 위한 구현
    var didUpdateUserLocation : ControlEvent<CLLocationCoordinate2D> {
        let source = delegate
            .methodInvoked(#selector(MKMapViewDelegate.mapView(_:didUpdate:)))
            .map { parameters in
                return (parameters[1] as? MKUserLocation)?.coordinate ?? CLLocationCoordinate2D()
            }
        
        return ControlEvent(events: source)
    }
    
    // mapView(_:regionDidChangeAnimated:)를 위한 구현
    var regionDidChange: ControlEvent<CLLocationCoordinate2D> {
        let source = delegate
            .methodInvoked(#selector(MKMapViewDelegate.mapView(_:regionDidChangeAnimated:)))
            .map { parameters in
                return (parameters[0] as? MKMapView)?.centerCoordinate ?? CLLocationCoordinate2D()
            }
        
        return ControlEvent(events: source)
    }
}
```
### 4 RxCocoa에서 익숙한 `.rx`를 통해 위에서 구현한 메서드를 사용

```swift
		// ViewModel
    let currentLocation = PublishRelay<CLLocationCoordinate2D>() // 사용자 위치
    let mapCenterPoint = PublishRelay<CLLocationCoordinate2D>() // 지도 중심
```

```swift
class LocationInfoViewController: UIViewController {
    lazy var mapView = MKMapView()
    
    // (나머지 구현 생략) ...
    
    func bind(_ viewModel: LocationInfoViewModel) {
    
        // DelegateProxy와 Binding
        mapView.rx.regionDidChange
            .bind(to: viewModel.mapCenterPoint)
            .disposed(by: disposeBag)
        
        mapView.rx.didUpdateUserLocation
            .bind(to: viewModel.currentLocation)
            .disposed(by: disposeBag)
    }
}
```

`DelegateProxy`에 대한 글은 여기까지입니다.

전체 코드 및 사용예는 아래 `Repo`에서 확인하실 수 있습니다.
[GitHub - SwiftyCody/SearchCVS: Kakao RestAPI를 이용한 RxCocoa 예제](https://github.com/SwiftyCody/SearchCVS)