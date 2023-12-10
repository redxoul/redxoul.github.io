---
title: (RxSwift) share(replay:scope:)
author: Cody
date: 2022-11-29 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---

`share` Operator를 언제 사용할지를 이해하기 위해서는

먼저 `Observable`의 기본특성을 이해해야 합니다.

이전 글([[RxSwift] RxSwift의 기본 요소 및 LifeCycle](https://swiftycody.github.io/posts/RxSwift-RxSwift%EC%9D%98-%EA%B8%B0%EB%B3%B8-%EC%9A%94%EC%86%8C-%EB%B0%8F-LifeCycle/) )에서 정리했지만,

`Observable`은 정의해놓은 것만으로는 아무값도 방출이 되지 않고

`subscribe`을 했을 때부터 값을 방출합니다.

```swift
// Create of
let numObservable = Observable.of(1,2,3,4,5).debug("")
```

```swift
// 출력 없음
```

`subscribe`를 하면? `subscribe`할 때마다 `Element`를 방출합니다.

```swift
let numObservable = Observable.of(1,2,3,4,5).debug("")
numObservable
    .subscribe() // 한번 subscribe
    .disposed(by: disposeBag)

numObservable
    .subscribe() // 두번 subscribe
    .disposed(by: disposeBag)
```

(두번 subscribe해서 두번 방출)

```
-> subscribed
-> Event next(1)
-> Event next(2)
-> Event next(3)
-> Event next(4)
-> Event next(5)
-> Event completed
-> isDisposed
-> subscribed
-> Event next(1)
-> Event next(2)
-> Event next(3)
-> Event next(4)
-> Event next(5)
-> Event completed
-> isDisposed
```

만약 아래처럼 서버로부터 response를 받아오는 Observable이 있을 때,

```swift
import RxCocoa

let response = Observable.from(["ReactiveX/RxSwift"])
  .map { urlString -> URL in
    return URL(string: "https://api.github.com/repos/\(urlString)/events")!
  }
  .map { url -> URLRequest in
    var request = URLRequest(url: url)
    return request
  }
  .flatMap { request -> Observable<(response: HTTPURLResponse, data: Data)> in
    // RxCocoa의 rx.response : 웹서버로부터 전체 응답을 받을 때마다 완료되는 Observable<(response: HTTPURLResponse, data: Data)>을 반환
    // delegate없이 바로 응답을 받을 수 있음.
    return URLSession.shared.rx.response(request: request)
  }
```

위 `Observable`을 여러군데에서 `subscribe`를 하게 된다면?

여러번 호출될 때마다 서버에 계속 요청을 하게 됩니다.

```swift
response
    .subscribe(onNext: {
        print($0.data)
    })

response
    .subscribe(onNext: {
        print($0.data)
    })
```

(출력)

2번 `subscribe`해서 2번 서버에 요청되었습니다.

```
curl -X GET 
"https://api.github.com/repos/ReactiveX/RxSwift/events" -i -v
Success (5ms): Status 200
67782 bytes

curl -X GET 
"https://api.github.com/repos/ReactiveX/RxSwift/events" -i -v
Success (16ms): Status 200
67782 bytes
```

이렇게 하지 않으려면 `share`가 필요해집니다.

`share(replay:scope)`를 사용하게 되면 `subscribe`할 때 기존에 만들어진 시퀀스가 있다면,

새로 생성되지 않고 처음 `subscribe`했을 때 방출된 `Element`를 공유해서 사용할 수 있게 됩니다.

```swift
let response = Observable.from(["ReactiveX/RxSwift"])
  .map { urlString -> URL in
    // URL로 mapping해서 방출하겠다고 명시
    return URL(string: "https://api.github.com/repos/\(urlString)/events")!
  }
  .map { url -> URLRequest in
    // 위에서 URL을 받아서 URLRequest로 다시 mapping
    var request = URLRequest(url: url)
    return request
  }
  .flatMap { request -> Observable<(response: HTTPURLResponse, data: Data)> in
    return URLSession.shared.rx.response(request: request)
  }
  .share(replay: 1) // share로 받아온 값을 공유
response
    .subscribe(onNext: {
        print($0.data)
    })

response
    .subscribe(onNext: {
        print($0.data)
    })
```

(출력)

한번만 서버에 요청하고 두번째에는 replay.

```
curl -X GET 
"https://api.github.com/repos/ReactiveX/RxSwift/events" -i -v
Success (432ms): Status 200
67782 bytes
67782 bytes
```

`share`에는 옵셔널로 `scope`를 설정할 수 있는데요.

`.whileConnected`는 `subscriber`가 존재할 때까지만 `buffer`를 가집니다. `subscriber`가 없어지면 바로 `dispose`됩니다.

`.forever`는 받은 `Element`가 영구적으로 유지가 됩니다. 해당 `Element`가 메모리에 계속 남게 될 수도 있으니 주의해서 사용해야 합니다.