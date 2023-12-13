---
title: UserDefaults와 PropertyListEncoder, PropertyListDecoder
author: Cody
date: 2022-08-08 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Swift, UserDefaults, PropertyListEncoder, PropertyListDecoder]
pin: false
mermaid: true
---

우리는 어떤 값을 저장해놓았다가,

앱을 재실행했을 때도 이를 꺼내어 해당 값에 따라서 동작하도록 하고 싶을 때가 있습니다.

방법은 여러가지가 있겠지만, 간단한 값들을 저장해놓을 때 `UserDefaults`라는 수단을 많이 사용합니다.

매우 간단한 방법으로 값을 저장하고 불러올 수 있기 때문입니다.

이렇게 저장을 해놓으면,

```swift
UserDefaults.standard.set("Honey Butter", forKey: "almond")
UserDefaults.standard.synchronize()
```

이렇게 불러와서 쓰면 됩니다.

```swift
if let almond = UserDefaults.standard.value(forKey: "almond") {
	print("Almond: ", almond) // Almond:  Honey Butter
}
```

그런데 저장해놓고 싶은 값이 그냥 값이 아니라,

아래와 같은 구조체의 객체라면 어떻게 될까요?

```swift
struct Alert: Codable {
    var id: String = UUID().uuidString
    var date: Date
    var isOn: Bool
    
    var time: String {
        let formatter = DateFormatter()
        formatter.dateFormat = "hh:mm"
        return formatter.string(from: date)
    }
    
    var meridiem: String {
        let formatter = DateFormatter()
        formatter.dateFormat = "a"
        formatter.locale = Locale(identifier: "ko")
        return formatter.string(from: date)
    }
}
```

`Alert` 객체를 생성해서 위와 같이 저장을 하려고 하면

```swift
let newAlert = Alert(date: date, isOn: true)

UserDefaults.standard.set(newAlert, forKey: "alert")
// (Error: Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: 'Attempt to insert non-property list object Drink.Alert(id: "5B8E2C5C-71AB-4589-BF09-A5CC2BAE584E", date: 2022-08-07 20:53:42 +0000, isOn: true) for key alert')
```

에러가 발생합니다.

'`Non-property list` 객체를 `insert`하려고 시도했다`'는 것이 원인입니다.

`UserDefaults`에는 `property list` 객체만 저장`되야만 하기 때문인데,

이를 해결하기 위해서

`PropertyListEncoder`, `PropertyListDecoder`가 있습니다.

객체를 `PropertyList`로 변환하기 위해서 `PropertyListEncoder`를 사용하고,

`PropertyList`를 객체로 변환하기 위해서 `PropertyListDecoder`를 사용합니다.

여기서 `PropertyList`란,

우리가 `App` 프로젝트 내에서 설정들을 세팅할 때 사용하는 `plist` 를 말합니다.

이전에 작성한 `Codable`글에서처럼,

([`Codable(Encodable, Decodable), CodingKey`](https://swiftycody.github.io/posts/Swift-Codable-Encodable-Decodable-CodingKey/) )

객체를 `Encode`, `Decode`하기 위해서는 `Codable` 프로토콜로 작성을 해야합니다.

위에서는 미리 `Alert`구조체를 `Codable`로 작성했습니다.

이제 `PropertyListEncoder`을 이용해서 아래와 같이 저장하면 됩니다.

`Encode`된 값은 `Data` 타입으로 저장되어집니다.

```swift
let newAlert = Alert(date: date, isOn: true)

UserDefaults.standard.set(try? PropertyListEncoder().encode(newAlert), forKey: "alert")
```

`PropertyListDecoder`로 불러올 때는 아래와 같이 작성합니다.

위에서 `Data`타입으로 `UserDefaults`에 저장이 되었기 때문에,

`Data`로 불러온 뒤, `PropertyListDecoder`로 `Decode`해서 사용하면 됩니다.

```swift
guard let data = UserDefaults.standard.value(forKey: "alert") as? Data, let alert = try? PropertyListDecoder().decode(Alert.self, from: data) else { return }
            
print("alert: ", alert) // alert:  Alert(id: "F5C120A2-6B44-430F-B150-A1FE69810223", date: 2022-08-07 21:12:54 +0000, isOn: true)
```