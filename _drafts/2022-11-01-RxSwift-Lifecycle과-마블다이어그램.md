---
title: (RxSwift) Lifecycle과 마블다이어그램
author: Cody
date: 2022-11-01 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---
지난 포스팅에서 정리한 Lifecycle을

ReactiveX 공식 사이트([https://reactivex.io](https://reactivex.io/))에서는 마블다이어그램(Marble Diagram)을 사용해서 설명하고 있습니다.

이 마블다이어그램을 잘 이해하면, Rx의 여러 Operator의 흐름을 이해하는데 도움이 됩니다.

마블다이어그램에서 각 상징들이 어떤 의미인지 설명하는 포스팅을 정리해봅니다.

- **화살표 라인**은 `Observable`을 나타내고, 왼쪽에서 오른쪽으로 시간이 흘러감을 표현해줍니다.
- 화살표에 그려진 **원(Marble)**들은 `Observable`이 값`(1, 2, 3, ...)`을 방출시키는 `next`이벤트를 의미합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/5934eff9-1693-472f-bd65-05bd75970d38)
  
  
- `Observable`에서 세번의 이벤트(tap)을 방출하고 나오는 **세로선**은 정상적으로 종료된 `completed`이벤트를 의미합니다.
(`completed`이벤트 후에는 아무것도 방출시킬 수 없습니다)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b12a5d37-7b77-4316-8761-acacdd21dc0a)

- 화살표 라인에 빨간색**X**는 `error`가 발생한 것입니다. `Observable`이 `error`이벤트를 내보냅니다.(`error`이벤트 후에는 아무것도 방출시킬 수 없습니다)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/504746b2-bb86-4edb-89ea-16ae67c80bf9)

- **네모난 박스**는 `Operator`를 의미합니다.(아래 예시는 하나의 `next`이벤트를 방출하고 종료하는 `just` Operator)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/069f2745-7992-4d74-851c-1e9f2fecf204)

- **위쪽을 향하는 화살표**는 `Subscribe`를 의미하고, **아랫쪽을 향한 화살표**는 **방출한 이벤트(`next`)**를 의미합니다.(`Observable`은 `Subscribe`하기 전엔 아무값도 방출시키지 않는다고 했죠?  
`1`값이 `next`이벤트로 발생했지만 `Subscribe`한 `Observer`가 없었고, 
그 이후에 `Subscribe`한 2번째 화살표가 방출된 `2`값을 받았습니다.  
그리고 `2`값을 방출한 뒤 `Subscribe`한 3번째 화살표와 이미 `Subscribe`중인 2번째 화살표가 `3`값을 받고`completed` 이벤트를 받고 종료되었습니다)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f0ba36c8-5d45-42d9-879d-2dda6b6f57c1)

- **라인의 색깔**은 **thread**를 나타냅니다.(예시의 observeOn은 작업의 수행을 어떤 스레드에서 작업할지 변환시켜주는 Operator.backgroundScheduler에서 할지 mainScheduler에서 할지 지정해 줄 수 있습니다.)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4e4ddc5e-5994-406f-baa4-2531acd126a3)