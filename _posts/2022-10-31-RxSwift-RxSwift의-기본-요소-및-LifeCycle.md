---
title: (RxSwift) RxSwift의 기본 요소 및 LifeCycle
author: Cody
date: 2022-10-31 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---
### 1. Observable

`Observable<Element>` 형태로 사용되며, `Element`는 `방출(emit)하고자 하는 타입`입니다.

`Subscribe`된 후 `시간이 지남에 따라 유한, 무한의 이벤트를 방출하는 객체`입니다.

유한의 이벤트는 `URLSession`을 통한 데이터 패치 이벤트가 예시가 될 수 있고,

무한의 이벤트는 사용자의 응답을 기다리는 `UI`의 이벤트가 예시가 될 수 있습니다.

`next`, `error`, `completed` 이벤트를 방출합니다.

- `next`: 최신, 혹은 다음 Element와 함께 방출되는 이벤트입니다. `error`, `completed` 이벤트가 방출되기 전까지 계속 `Element`가 방출될 수 있습니다.
- `completed`: 성공적으로 이벤트 시퀀스가 종료됐다는 의미입니다. `Observable`이 더이상 추가적인 이벤트를 방출하지 않습니다.
- `error`: `error`와 함께 이벤트 시퀸스가 종료됐다는 의미입니다. `Observable`이 더이상 추가적인 이벤트를 방출하지 않습니다.

### 2. 시퀀스(Sequence)

`Observable`에서 데이터가 생기고 `Element`를 방출하는 과정. 혹은 방출하는 연속된 `Element`들을 말합니다.

`Rx`가 다양한 언어를 지원하고 있고, 다른 언어에서는 스트림(Stream)이라고도 부르지만

RxSwift에서는 대체적으로 `시퀀스`라고 합니다.

### 3. Operator

`Observable` 시퀀스에서 데이터들을 처리할 수 있게 해주는 API들입니다.

`Create`, `Transform`, `Filtering`, `Combine`, `ErrorHandling`, `Scheduling`, `Timebase` 등 다양한 종류가 있습니다.

### 4. 스케쥴러(Scheduler)

`DispatchQueue`, `OperatorQueue`와 같이 특정 작업의 실행 컨텍스트를 정의할 수 있게 해줍니다.

GCD를 이용해 주어진 대기열에서 코드를 직렬처리(SerialDispatchQueueScheduler), 병렬처리(ConcurrentDispatchQueueScheduler)를 지정하거나,

OperationQueueScheduler로 주어진 OperationQueue에서 Subscribe를 예약할 수 있습니다.

---

## RxSwift의 LifeCycle

### 1. Create

`Observable`을 생성하는 단계입니다.

`create`, `just`, `from`, `of` 등과 같은 Create Operator를 통해 생성할 수 있습니다.

내보낼 값, 이벤트를 정의해줍니다.

생성한 것만으로는 아무런 값도 방출하지 않습니다.

(함수를 정의만 해놓은 상태와 같습니다)

### 2. Operator를 통한 가공(transform, filter, merge, errorHandling, scheduling, ...)

create로 방출될 이벤트(`next`, `completed`, `error`)들을

`subscribe`에서 `next`이벤트를 받기 전에 `'가공 처리'`해주는 과정입니다.

`Transform`, `Filtering`, `Combine`, `ErrorHandling`, `Scheduling`, `Timebase` 등

다양한 처리를 쉽게 할 수 있도록 도와줍니다.

(마치 수학에서의 +, - , x, ÷부호와 같습니다. Subscribe하기 전에 목표하고자하는 값을 얻을 때까지 여러번 호출할 수 있습니다)

### 3. Subscribe

`Subscribe`를 하고부터 `Observable`은 이벤트(`next`, `completed`, `error`)를 방출하기 시작합니다.

(1)Create에서 방출된 값이 (2)Operator들을 통해 가공된 값을 받게 됩니다.

(함수를 호출해서 값을 받을 수 있는 상태가 된 것과 같습니다)

dispose되기 전까지 계속 값을 받을 수 있습니다.

### 4. Dispose

`Subscribe`를 하면 `Observable`은 `disposable`객체를 내놓습니다.

이름 그대로 '폐기(`dispose`)할 수 있는 것'이라는 뜻입니다.

`Observable`에서 종료 이벤트(`completed`, `error`)가 방출되면 `dispose`가 되고,

종료 이벤트가 방출되기 전이라도 원하는 타이밍에 `dispose`시켜 `Observable`의 동작을 종료시킬 수 있습니다.

한번 `dispose`된 `subscriber`는 다시 `next`이벤트를 통해 값을 받을 수 없게 됩니다.

`DisposeBag`에 `subscription`들을 넣어놓으면 `DisposeBag`이 `dealloc`될 때

넣어놓은 `subscription`들의 `dispose()`를 호출시켜줍니다.

제때 `dispose`를 호출시켜주지 않거나, `disposeBag`에 `subscription`들을 넣어두지 않으면(= `disposed(by:)`), 메모리 누수가 생길 수 있으니 주의해야 합니다.