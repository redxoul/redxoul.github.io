---
title: (RxSwift) Create Operators(3) defer
author: Cody
date: 2022-11-10 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---
## defer

`defer`로 생성된 `observable`은 다른 생성 operator들과 다르게,

`observer`가 `subscribe`를 호출하면 그 때부터 `observable`을 생성시킵니다.

그래서 `defer`는 `subscribe`할 때마다 다른 `observable`을 생성시키는 `Factory`를 구현할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/34096868-2c1d-4435-9ae8-0df05ed701af)

```swift
print("----- deferred -----")

let disposeBag = DisposeBag()
    
// 반환할 observable을 flip할 플래그
var flip = false
// defer를 사용. Int형 Observable을 반환하는 factory 생성.
let factory: Observable<Int> = Observable.deferred {
	// subscribe될 때마다 flip의 toggle이 발생
	flip.toggle()
	        
	// flip에 따라 다른 observable을 반환
	if flip {
		return Observable.of(1, 2, 3)
	} else {
		return Observable.of(4, 5, 6)
	}
}

// subscribe
for _ in 0...3 {
	factory.subscribe(onNext: {
		print($0, terminator: " ")
	})
	.disposed(by: disposeBag)

	print()
}
```

(출력)

```
----- deferred -----
1 2 3
4 5 6
1 2 3
4 5 6
```