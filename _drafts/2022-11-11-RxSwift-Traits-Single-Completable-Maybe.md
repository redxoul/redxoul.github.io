---
title: (RxSwift) Traits - Single, Completable, Maybe
author: Cody
date: 2022-11-11 00:34:00 +0800
categories: [iOS, RxSwift]
tags: [Swift, RxSwift]
pin: false
mermaid: true
---
`Observable`은 `next`, `completed`, `error` 세 가지 이벤트를 방출하는 타입인데요.

하지만 항상 저 세 가지 이벤트가 모두 필요한 것은 아닙니다.

그래서 `RxSwift`는 상황에 맞는 `Observable`의 변형들을 제공하는데 이를 `Traits(특성)`이라고 합니다.

`Single`, `Completable`, `Maybe` 이렇게 세가지 `Traits`가 있습니다.

`Observable`을 사용해도 되지만,

이 `Traits`들은 이름을 읽고서 좀 더 쉽게 흐름을 파악할 수 있도록 해 주고,

`API`를 읽는 사람들에게 `코드의 의도를 명확하게 전달하는 방법을 제공`하는 것이 목적입니다.

## Single

`Single`은 `success(value)`, `failure(error)` 이벤트만 방출시킵니다.

(error(error)이벤트는 deprecated되고 failure로 변경되었습니다)

`success`는 `observable`의 `next`와 `completed`를 결합해놓은 것과 같습니다.

서버로부터 데이터를 `Fetch`해올 때 성공하고 값을 방출하거나, 실패하고 에러를 내보내는

`1회성 프로세스`에 유용한 `Trait`입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c0197f09-0734-4482-8f7f-81e3b0e30214)

아래는 리소스의 텍스트 파일을 읽어오는

`Single<String>` 을 반환하는 `loadText` 메서드의 예시입니다.

```swift
print("- - - - - Single (1) - - - - -")

let disposeBag = DisposeBag()

// 디스크를 읽을 때 발생할 수 있는 error를 모델링
enum FileReadError: Error {
    case fileNotFound, unreadable, encodingFailed
}

// Single<String>을 반환하는 loadText 구현
func loadText(from name: String) -> Single<String> {
    // Single을 만들고 반환
    return Single.create { observer in
    
        // 반환할 Disposable을 생성
        let disposable = Disposables.create()
        
        // 파일 이름에 대한 경로를 가져오고
        guard let path = Bundle.main.path(forResource: name, ofType: "txt") else {
            // file not found error를 Single에 추가하고 disposable을 반환
            observer(.failure(FileReadError.fileNotFound))
            return disposable
        }
        
        // 해당 파일에서 데이터를 가져오고
        guard let data = FileManager.default.contents(atPath: path) else {
            // unreadable error를 Single에 추가하고 disposable을 반환
            observer(.failure(FileReadError.unreadable))
            return disposable
        }
        
        // 데이터를 String으로 변환
        guard let contents = String(data: data, encoding: .utf8) else {
            // encodingFailed error를 Single에 추가하고 disposable을 반환
            observer(.failure(FileReadError.encodingFailed))
            return disposable
        }
        
        // Single에 컨텐츠를 Single의 success에 추가하고 disposable을 반환
        observer(.success(contents))
        return disposable
    }
}
```

위 `loadText`를 사용.

```swift
// 위 Single의 사용 예시
// 텍스트파일의 이름을 전달
loadText(from: "fileName")
    .subscribe {
        switch $0 {
        case .success(let string):
            print(string) // 읽어오기 성공
        case .failure(let error):
            print(error) // 읽어오기 실패
        }
    }
    .disposed(by: disposeBag)
```

`Single`을 만드는 또 다른 방법이 있습니다.

기존의 `Observable`을 `.asSingle()` operator를 통해 `Single`로 변환을 할 수 있습니다.

```swift
// Single<String>을 반환하는 loadText 구현 (2)
func loadText(from name: String) -> Single<String> {

    return Observable<String>.create { observer in
    
        let disposable = Disposables.create()
        
        guard let path = Bundle.main.path(forResource: name, ofType: "txt") else {
            observer.onError(FileReadError.fileNotFound)
            return disposable
        }
        
        guard let data = FileManager.default.contents(atPath: path) else {
            observer.onError(FileReadError.unreadable)
            return disposable
        }
        
        guard let contents = String(data: data, encoding: .utf8) else {
            observer.onError(FileReadError.encodingFailed)
            return disposable
        }
        
        observer.onNext(contents)
        observer.onCompleted()
        return disposable
    }
    .asSingle() // 생성한 Observable을 Single로 전환
}
```

하지만 `Observable`을 `asSingle`을 통해 `Single`로 전환했을 때 `몇 가지 문제가 발생`할 수 있습니다.

`Single`은 `success`, `error` 이벤트만 방출하고

`success`이벤트는 `Observable`의 `next`, `complete`이벤트를 합쳐놓은 것과 같다고 했죠?

그래서 `Observable`을 `asSingle`을 사용해 전환했을 때, 위 예제와 같이

(1) `next`이벤트 이후에 `completed`이벤트가 발생하지 않으면 `observer`가 `success`이벤트를 받을 수가 없습니다. 그리고

(2) `next`이벤트가 발생하기 전에 `completed`이벤트가 먼저 발생하면 방출하기로 한 데이터가 없기 때문에 `observer`는 `error`이벤트를 받게 됩니다.

---

## Completable

`Completable`은 `completed`와 `error(error)` 이벤트만 방출시킵니다.

어떤 값도 방출시키지 않기 때문에 파일저장 같은 `작업의 성공, 실패만 필요한 경우에 적합`합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c0bf3a65-8542-484a-915b-6f262706c344)

```swift
func saveText(fileName: String, text: String) -> Completable {
    // Completable을 만들어 반환
    return Completable.create { observer -> Disposable in
        let disposable = Disposables.create()
        
        let fileManager = FileManager.default
        
        let documentURL = fileManager.urls(for: .documentDirectory, in: .userDomainMask).first
        
        guard let directoryURL = documentURL?.appendingPathComponent("FolderName") else {
            observer(.error(TraitsError.completable))
            return disposable
        }
        
        do {
            try fileManager.createDirectory(atPath: directoryURL.path, withIntermediateDirectories: false)
        } catch let error {
            observer(.error(error))
            return disposable
        }
        
        let filePath = directoryURL.appendingPathComponent(fileName)
        
        do {
            try text.write(to: filePath, atomically: false, encoding: .utf8)
        } catch let error {
            observer(.error(error))
            return disposable
        }
        
        observer(.completed) // 최종 저장 끝!
        return disposable
    }
}
```

이렇게 만든 파일저장용 `Completable`을 `subscribe`한 뒤

`completed`, `error` 이벤트에서 각각 적절한 사용자 피드백을 주면 됩니다.

```swift
saveText(fileName: "someFile", text: "someText")
    .subscribe (
        onCompleted: {
        	// 사용자에게 성공 메세지를 띄워준다
            print("completed")
        },
        onError: {
        	// 사용자에게 적절한 에러 메세지를 띄워준다
            print("error: \($0)")
        }
    )
    .disposed(by: disposeBag)
```

## Maybe

`Maybe`는 앞에서 설명한 `Single`과 `Completable`을 합쳐놓은 개념입니다.

`success(value)`, `completed`, `error(error)` 중 하나를 방출합니다.

기본적으로 `Single`같지만 아무값도 방출하지 않고 `Completed`로 종료시킬 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0eec3f95-0d94-4620-8850-5f09e11a2cbb)

```swift
print("- - - - - Maybe (1) - - - - -")
Maybe<String>.just("Maybe")
    .subscribe(
        onSuccess: {
            print($0)
        },
        onError: {
            print("error: \($0)")
        },
        onCompleted: {
            print("completed")
        },
        onDisposed: {
            print("disposed")
        }
    )
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - Maybe (1) - - - - -
Maybe
disposed
```

`Maybe`도 `Single`처럼 `Observable`을 `Maybe`로 변환해주는 `asMaybe()`가 있습니다.

```swift
print("- - - - - Maybe (2) - - - - -")
Observable<String>.create { observer -> Disposable in
        observer.onError(TraitsError.maybe)
        return Disposables.create()
    }
    .asMaybe()
    .subscribe(
        onSuccess: {
            print($0)
        },
        onError: {
            print("error: \($0)")
        },
        onCompleted: {
            print("completed")
        },
        onDisposed: {
            print("disposed")
        }
    )
    .disposed(by: disposeBag)
```

(출력)

```
- - - - - Maybe (2) - - - - -
error: maybe
disposed
```

`asMaybe` 또한 `asSingle`과 같은 문제점을 가지고 있습니다.

`Observable`에서 `next`이벤트를 방출했어도

`completed`이벤트를 방출하기 전까지 `asMaybe`의 `success(value)`이벤트를 받을 수가 없습니다.

---

`asMaybe`, `asSingle`을 이해하지 않고 한 번만 이벤트를 받기 위해 전환을 했다가

영원히 `success(value)`이벤트를 못 받거나 `error(error)`이벤트를 받아서 의도하지 않게 동작할 수 있으니 주의해야 합니다.

그래서 `asSingle`, `asMaybe`로 전환해서 쓸 일이 있을 때는

(1) 가능하면 처음부터 `Single`이나 `Maybe`로 작성을 하거나

(2) 적어도 `유한한 Observable`에만 `asSingle`, `asMaybe`로 전환해서 사용하는 것이 좋습니다. `유한한 Observable`은 반드시 `completed`를 방출시키는 `Observable`입니다. 그러면 받아야 할 `success(value)`를 못 받는 일이 생기진 않겠죠?(언젠간 받겠죠)