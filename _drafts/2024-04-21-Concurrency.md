---
title: (Swift) 동시성(Concurrency)
author: Cody
date: 2024-04-21 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Swift, Concurrency, 동시성]
pin: false
mermaid: true
---

WWDC21 [Meet async/await in Swift](https://developer.apple.com/videos/play/wwdc2021/10132/)

기존 UIKit의 UIImage에서는 썸네일을 생성하는 함수를 동기식, 비동기식을 모두 제공.   
```Swift
func preparingThumbnail(of size: CGSize) -> UIImage? // 동기식
func prepareThumbnail(of size: CGSize, completionHandler: @escaping (UIImage?) -> Void) // 비동기식
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b16e6501-a2a4-4a97-991d-adfbd64a0af6)
동기식 함수를 호출시, 스레드가 block되어 해당 함수가 완료될 때까지 기다림.   
`fetchThumbnail()`이라는 함수에서 `preparingThumbnail(of:)`를 호출하면 완료될 때까지 스레드가 다른 작업을 할 수 없게 됨.   

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/edca4dfc-a695-4710-a5a2-9da463273f3c)
비동기 함수 `prepareThumbnail(of:completionHandler:)`를 호출하면 실행되는 동안 스레드가 자유롭게 다른 작업이 가능.
작업이 완료되면 completionHandler를 호출하여 받은 이미지에 대한 처리를 할 수 있게 해줌.

기존 SDK들은 비동기 처리 후 completionHandler나 delegate 콜백을 통해 알려주고, 많은 경우 async로 표시되어 값만 반환함.   

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9c6cde42-aff1-4567-88ee-cee975a10e5c)
fetchThumbnail()함수의 예시.   
문자열로부터 `URLRequest`를 생성 후, `URLSession`의 `dataTask(with:completion:)`으로 request의 data를 가져오고, 가져온 data에서 `UIImage`를 초기화하고, 원본 이미지로부터 썸네일을 가져오는 예시.   
각각의 작업들은 이전의 작업 결과에 따라 달라지기 때문에 순차적으로 작업되어야 하는데.
일부 작업(`thumbnailURLRequest`, `UIImage`)은 값을 빠르게 반환하기 때문에 함수가 있는 스레드에서 동기 호출이 되는 것이 좋음.   
어떤 작업(`dataTask`, `prepareThumbnail`)은 시간이 걸리기 때문에 비동기 호출이 좋음.

```swift
func fetchThumbnail(for id: String, completion: @escaping (UIImage?, Error?) -> Void) {
    let request = thumbnailURLRequest(for: id) // 동기
    // 비동기
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
        if let error = error {
            completion(nil, error)
        } else if (response as? HTTPURLResponse)?.statusCode != 200 {
            completion(nil, FetchError.badID)
        } else {
            guard let image = UIImage(data: data!) else { // 동기
                // completion(nil, FetchError.badImage) // 작업자가 빼먹음
                return
            }
            // 비동기
            image.prepareThumbnail(of: CGSize(width: 40, height: 40)) { thumbnail in
                guard let thumbnail = thumbnail else {
                    // completion(nil, FetchError.badImage) // 작업자가 빼먹음
                    return
                }
                completion(thumbnail, nil)
            }
        }
    }
    task.resume()
}
```
위처럼 작성된 `fetchThumbnail(for:completion:)`의 호출자가 해당 작업이 완료되었을 때 실패하더라도 노티를 받기를 원했지만, 작업자가 completion을 호출하는 것을 빼먹게 되면 문제가 생길 수 있음.   
무슨 일이 일어나더라도 호출자에게 알리는 것은 매우 중요함.   
일반 함수는 error를 발생시켜 호출자에서 throw함. Swift는 함수를 통해 실행이 어떻게 되든 상관없이 값이 반환되지 않으면 오류가 발생하도록 보장하지만, 여기에서는 일반적인 error 처리 매커니즘을 사용할 수가 없음. 문제가 발생했을 때, completionHandler 내에서 error를 발생시킬 수 없음.   
> Swift가 (컴파일 시) 우리 작업을 체크해 줄 수가 없다는 의미.   
> completionHandler는 그저 클로저일 뿐. 항상 호출되는지 확인하고 싶지만 Swift에서 이를 강제할 수 있는 방법은 없음.

-> async/await을 통해 더 간결하면서 안전한 코드를 만들 수 있음.

### async/await
```swift
func fetchThumbnail(for id: String) async throws -> UIImage {
    
}
```
함수를 `async`로 표시할 때 키워드는 `throws` 앞에 와야하고 throw되지 않으면 `->`앞에 오면 됨.   
thumbnail이미지가 성공적으로 만들어지만 UIImage로 반환하고 오류발생시 throw됨.
```swift
func fetchThumbnail(for id: String) async throws -> UIImage {
    let request = thumbnailURLRequest(for: id) // 동기
    
    // data(for: request)
    // UIImage(data: data)
    // thumbnail
}
```
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0c7dc88b-4290-4b2f-bf16-abf212a08e92)
우선 동기 작업인 `thumbnailURLRequest(for:)` 작업을 수행. 스레드가 차단되어 수행.


```swift
func fetchThumbnail(for id: String) async throws -> UIImage {
    let request = thumbnailURLRequest(for: id)  
    let (data, response) = try await URLSession.shared.data(for: request)
    guard (response as? HTTPURLResponse)?.statusCode == 200 else { throw FetchError.badID }

    // UIImage(data: data)
    // thumbnail
}
```
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0f17748a-d634-4ba6-b877-d585907f66b0)
그리고 dataTask와 마찬가지로 Foundation이 제공하는 `data(for:)`를 호출하여 데이터를 비동기 다운로드.    
이 메서드는 awaitable한 메서드. 호출된 뒤 빠르게 스스로(`data(for:)`)를 일시정지하고 스레드 차단을 해지해서 스레드가 다른 작업도 수행할 수 있게 함.   
`try`가 붙어 있는 이유는 `data(for:)` 메서드가 `throws` 이기 때문.   
`throws` 표시된 함수를 호출하려면 `try`가 필요하듯이, `async`표시된 함수를 호출하려면 `await`이 필요함.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/20067019-b946-4f78-b34a-687eb78c0a9b)
데이터 다운로드가 완료되면 `data(for:)` 메서드가 재개되고 `fetchThumbnail`로 돌아감.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/595af49a-26dd-42d3-9f85-e461d36dd393)
이 시점에 `data(for:)`메서드가 반환하는 값이나 throw한 error가 유입되고,   
error가 유입되었다면 `fetchThumbnail(for:)`이 해당 오류를 발생시킴.   
그렇지 않으면 `let (data, response)`가 정의됨.
```swift
func fetchThumbnail(for id: String) async throws -> UIImage {
    let request = thumbnailURLRequest(for: id)  
    let (data, response) = try await URLSession.shared.data(for: request)
    guard (response as? HTTPURLResponse)?.statusCode == 200 else { throw FetchError.badID }
    let maybeImage = UIImage(data: data)

    // thumbnail
}
```
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f2e6947d-3aec-4fff-907d-8acf976af3ad)
그리고 다운받은 데이터로 `UIImage`를 생성하고,

```swift
func fetchThumbnail(for id: String) async throws -> UIImage {
    let request = thumbnailURLRequest(for: id)  
    let (data, response) = try await URLSession.shared.data(for: request)
    guard (response as? HTTPURLResponse)?.statusCode == 200 else { throw FetchError.badID }
    let maybeImage = UIImage(data: data)
    guard let thumbnail = await maybeImage?.thumbnail else { throw FetchError.badImage }
    return thumbnail
}
```
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1cb62e0a-38d4-48af-84b5-81e9938363c7)
이미지의 `thumbnail` 프로퍼티로 이미지 썸네일을 렌더링. `thumbnail` 프로퍼티를 수행하고 `fetchThumbnail(for:)`로 돌아오는 동안 다른 작업을 수행할 수 있게 됨.   

위 코드는 이전의 코드와 동작이 일치하지만, 더 간결하고 직선코드이며 문제 발생시 호출한 곳에 throw함.   
그리고 이전 코드처럼 실수로 completion 호출을 하지 않는 실수를 하지 않아도 되어 안전함.
> 함수뿐만 아니라 프로퍼티, 이니셜라이저도 `async`일 수 있음.

thumbnail 프로퍼티 코드 예시.
```swift
extension UIImage {
    var thumbnail: UIImage? {
        get async {
            let size = CGSize(width: 40, height: 40)
            return await self.byPreparingThumbnail(ofSize: size)
        }
    }
}
```
`Swift5.5`부터는 `getter`도 `throw`할 수 있고, `async`이면서 `throws`인 경우 `throws`키워드 앞에 `async`를 붙임.   
`getter`에만 `async`일 수 있고, `setter`는 안됨.   

```swift
for await id in staticImageIDsURL.lines {
    let thumbnail = await fetchThumbnail(for: id)
    collage.add(thumbnail)
}
let result = await collage.draw()
```
`await`는 for루프에서도 사용될 수 있음.