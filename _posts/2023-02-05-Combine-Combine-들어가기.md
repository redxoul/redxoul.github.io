---
title: (Combine) Combine 들어가기
author: Cody
date: 2023-02-05 00:34:00 +0800
categories: [iOS, Combine]
tags: [Swift, Combine, WWDC19, WWDC, iOS13]
pin: false
mermaid: true
---
아래는 `Apple` 공식문서의 설명을 번역했습니다.

(https://developer.apple.com/documentation/combine)

---

> Customize handling of asynchronous events by combining event-processing operators.
> 
> `이벤트 처리 operator를 결합하여, 비동기 이벤트 처리를 Customize 하는 Framework.`
> 

`Combine` `framework는` `시간 경과에 따른 값 처리를 위한 선언적 Swift API를 제공`합니다. 이 값들은 많은 종류의 비동기 이벤트들을 나타낼 수 있습니다. Combine은 `publisher`가 시간이 지남에 따라 변경될 수 있는 값을 노출하고, `subscriber`가 publisher로부터 이러한 값들을 받을 수 있도록 선언합니다.

- `Publisher` 프로토콜은 시간 경과에 따라 시퀀스 값들을 전달받을 수 있는 타입을 선언합니다. `Publisher`는 `upstream Publisher`로부터 받은 값에 따라 작업하고 다시 `publish` 할 수 있는 `Operator`를 가지고 있습니다.
- `Publisher`의 체인 끝에, `Subscriber`는 `element`를 받을 때 해당 `element`에 대한 작업을 수행합니다. `Publisher`는 `Subscriber`가 명시적으로 `request` 한 경우에만 값을 내보냅니다. 이렇게 하면 `Subscriber` 코드가 연결된 `Publisher`로부터 이벤트를 받는 속도를 제어할 수 있습니다.

`Timer`, `NotificationCenter`, `URLSession을` 포함한 여러 `Foundation Type`이 `Publisher`를 통해 기능을 보여줍니다. 그리고 `Combine`은 `Key-Value Observing`을 준수하는 모든 `Property`에 대해 기본 제공 `Publisher`를 제공합니다.

여러 `Publisher`의 출력을 결합(combine)하고 상호작용을 조정할 수 있습니다. 예를 들어 텍스트필드의 `Publisher`에서 업데이트를 `subscribe` 하고 텍스트를 사용하여 URL요청을 할 수 있습니다. 그리고 다른 `Publisher`를 사용하여 응답을 처리하고 앱을 업데이트하는 데 사용할 수 있습니다.

`Combine`을 채택하면, 이벤트 처리 코드를 중앙 집중화하고 중첩된 클로저 및 컨벤션기반 콜백과 같은 번거로운 기술을 제거하여 코드를 읽고 유지보수하기 쉽게 만들 수 있습니다.

---

`Apple`의 공식문서 설명을 보면

`RxSwift`를 익혔던 개발자라면 매우 익숙한 개념들이 많이 나옵니다.

`시간경과에 따른 값 처리, 비동기 처리, operators`가 나오다가

`Publisher`라는 새 개념이 나오지만 읽어보면 `Observable`과 유사한 개념으로 보이고,

반가운 `Subscriber`가 동일하게 나옵니다.

기존 `iOS`개발에서 비동기 처리를 위한 `API`들은

`NotificationCenter`, `Delegate Pattern`, `Key-Value Observing`, `클로저`, `selector`, `IBTarget/IBAction`, `URLSession` 등 다양하며 제각각의 인터페이스로 구현되어 사용성이 좋지 않았습니다.

이 비동기 인터페이스들을 모두 `시간이 지남에 따라 비동기적으로 받는 이벤트`로 보고 `하나의 공통된 API로 처리`할 수 있도록 만들어 준 것이 `Combine`입니다.

`RxSwift`와 같은 목적에서 같은 개념으로 만들어졌다고 보입니다.

`RxSwift`는 더 긴 시간 이어져와서 더 풍부한 `API`를 가졌고 최소 타겟 버전이 낮다는 장점이 있고,

`Combine`은 그렇지는 않겠지만 `Apple` 본가(?)에서 만든 덕에 성능과 기존 `Foundation`, `UIKit` `API`와의 연동성이 좋을테죠?

최소 타겟에 따른 차이도 크기 때문에 당장에 `combine`을 적극 도입된 곳이 많지 않을 수도 있지만,

이제 기존 앱들의 최소 타겟이 조금씩 올라가 `iOS 13`이 되어가는 시기가 다가오기 때문에

`RxSwift+RxCocoa`와 `Combine+SwiftUI`를 비교해 가면서 익혀두는 재미가 있을 듯합니다.

`RxSwift`의 정리처럼 연달아서 올리지는 않고,

하나씩 하나씩 알게 될 때마다 정리를 해서 포스팅해 봐야겠습니다:)