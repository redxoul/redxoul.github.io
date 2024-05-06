---
title: LSApplicationQueriesSchemes(canOpenUrl:)
author: Cody
date: 2022-05-29 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Xcode, iOS, iOS15]
pin: false
mermaid: true
---

iOS의 앱에서 타 앱을 호출하는 방법으로 `URLScheme`(이하 scheme, 혹은 스키마)가 있습니다.

앱에서는 project파일의 target->Info 에서 `URL Types` 를 설정하여 외부에서 앱을 호출할 수 있는 스키마를 설정해줄 수 있습니다.

+버튼을 누르면 여러개를 설정할 수도 있고, 어떤 스키마로 호출되는지에 따라 다른 동작을 하도록 구현할 수 있게 됩니다.

(하나의 스키마로도 추가로 프로퍼티를 받아서 동작에 분기를 시킬 수 있기 때문에 가급적 하나의 스키마로 처리하는게 더 좋습니다)
![호출되는 앱의 URL Types 설정](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/748da3ee-6767-4f13-a34e-5b25bdcc22a4)

호출되는 앱의 URL Types 설정

그러면 외부 앱에서 해당앱을 호출할 수 있게 되는데요.

호출하는 쪽에서는 앱의 `Info.plist`에 `LSApplicationQueriesSchemes` (`Array<String>`)항목에 호출할 앱의 스키마를 작성해줍니다.
![호출하는 앱의 LSApplicationQueriesSchemes 설정](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3099f263-fdfe-45c0-8ea3-8a738574b8b2)

호출하는 앱의 LSApplicationQueriesSchemes 설정

그리고 호출할 때는 `canOpenURL` 함수로 앱을 열 수 있는지를 먼저 체크한 후 열어주면 됩니다.

정확하지 않은 스키마를 호출했다거나, `LSApplicationQueriesSchemes`에 작성하지 않은 앱을 호출할 경우 `canOpenURL`에서 `false`를 받습니다.

그럴 땐 메세지를 띄워주거나, 앱스토어로 연결해주는 등의 동작을 해주면 됩니다.

```swift
if let url = URL(string: "someappscheme://"),  UIApplication.shared.canOpenURL(url) {
    UIApplication.shared.open(url)
}
else {
    // 앱을 열 수 없을 때의 동작
}
```

스키마로 앱을 호출하는 방식은 굉장히 오래전부터 있었던 방법입니다.

처음엔 `LSApplicationQueriesSchemes` 작성 없이도 다른 앱을 호출할 수 있었고, 애플이 `iOS9.0`부터는 이를 작성해야만 앱이 호출되도록 변경했습니다.

그리고 2021년 발표한 `iOS15`에서부터는 `LSApplicationQueriesSchemes`의 `갯수를 50개로 제한`을 두도록 변경되었습니다.

(참고: https://developer.apple.com/documentation/uikit/uiapplication/1622952-canopenurl?language=objc )

[
Apple Developer Documentation
 
developer.apple.com](https://developer.apple.com/documentation/uikit/uiapplication/1622952-canopenurl?language=objc)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/dfa2550a-46e3-4f60-b5d3-7723a5782220)

각종 소셜 연동(트위터, 페이스북, 카카오톡 등)을 하다보면 등록할 스키마들이 몇개씩 늘어나고, 다른 서비스들과 연동하다보면 50개가 넘는 일도 생길텐데요. iOS15부터는 의도한대로 동작을 하지 않을 수 있으니 관리가 필요해보입니다.

사실 URLScheme이 BundleID처럼 특정 앱에 종속되는 고유값이 아니라 어떤 앱에서든 스키마 설정을 할 수 있기 때문에 문제가 조금 있긴 합니다. 그래서 애플에서는 개발자가 스키마를 통한 앱 호출을 점차적으로 지양하고, Associated domains를 통한 유니버셜 링크 사용을 지향하도록 유도하는게 아닌가 싶습니다.