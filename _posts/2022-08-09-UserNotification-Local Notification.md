---
title: UserNotification(Local Notification)
author: Cody
date: 2022-08-09 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Swift, UserNotification]
pin: false
mermaid: true
---
앱에서 푸시 메세지를 띄우는 방법은

1. 푸시 서버로부터 받는 방법

2. 로컬 푸시메세지로 받는 방법이 있습니다.

그 중에 로컬에서 띄운 푸시메세지를 `UserNotification`이라고 합니다.

`UserNotification`을 띄우려면

`UNUserNotificationCenter`에 `UserNotificationRequest`를 추가해야하는데,

`UserNotificationRequest`는 `id`, `content`, `trigger` 세가지가 필요합니다.

`id`는 푸시메세지 `고유값`으로 해당 푸시메세지를 취소시키고 싶을 때 필요합니다.

`content`는 `title`, `subTitle`, `body`, `sound`, `badge`, `launchImageName` 등의 `푸시메세지의 실제 내용`입니다.

`trigger`는 푸시메세지를 띄울 `조건(방아쇠)`입니다.

`UNCalendarNotificationTrigger`(날짜, 시간), `UNTimeIntervalNotificationTrigger`(시간간격), `UNLocationNotificationTrigger`(위치)가 있으며 각 조건에 따라서 푸시메세지를 띄울 수 있습니다.

푸시메세지를 추가하는 예제입니다.

기본 시계앱에 있는 것과 같이 시간 설정에 따라 푸시알림을 주는 `UNCalendarNotificationTrigger`의 예시입니다.

`extension`을 통한 구현입니다.

```swift
extension UNUserNotificationCenter {
    func addNotificationRequest(date: Date, id: String) {
        let content = UNMutableNotificationContent()
        content.title = "로컬 푸시 제목"
        content.body = "로컬 푸시 내용"
        content.sound = .default
        content.badge = 1
        
        let component = Calendar.current.dateComponents([.hour, .minute], from: date)
        let trigger = UNCalendarNotificationTrigger(dateMatching: component, repeats: true)
        
        let request = UNNotificationRequest(identifier: id, content: content, trigger: trigger)
        
        self.add(request)
    }
}
```

알림을 삭제하고자 할 때는 `id`를 통해 삭제합니다.

여러 알림 동시에 삭제할 땐 해당 `id`들을 배열에 모두 넣어주면 됩니다.

```swift
// UNUserNotificationCenter 삭제
UNUserNotificationCenter.current().removePendingNotificationRequests(withIdentifiers: [id])
```

물론 `UserNotification`또한 일반 푸시메세지처럼 사용자의 허가를 받아야하기 때문에

푸시메세지 허가를 요청해야 하고,

```swift
var userNotificationCenter = UNUserNotificationCenter.current()
userNotificationCenter.delegate = self

let authorizationOptions = UNAuthorizationOptions(arrayLiteral: [.alert, .badge, .sound])

userNotificationCenter.requestAuthorization(options: authorizationOptions, completionHandler: { _, error in
    if let error = error {
        print("Error: Notification Authorization Request \(error.localizedDescription)")
    }
})
```

`UNUserNotificationCenterDelegate`를 구현해줍니다.

`completionHandler`에 배열로 `UNNotificationPresentationOptions`를 넣어주면 푸시메세지의 스펙을 정해줄 수 있습니다.

위에서 `.sound`를 넣어서 허가 요청을 했더라도, `completeHandler`에서 제외시켜주면 푸시메세지에서 소리가 안 나게 할 수 있습니다.

```swift
extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner, .list, .badge, .sound])
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        completionHandler()
    }
}
```

이렇게 작성을 하고,

푸시메세지 허용을 한 뒤, `UNUserNotificationCenter`에 `UserNotificationRequest`를 추가하면

아래와 같이 로컬 푸시가 옵니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1b6e9161-3d25-4395-b078-2b6ef3606833)