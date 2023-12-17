---
title: (WWDC22) (3) WidgetKit, Messages Collaboration API
author: Cody
date: 2022-06-12 00:34:00 +0800
categories: [iOS, iOS News and Updates]
tags: [WidgetKit, Messages Collaboration API, WWDC, WWDC22]
pin: false
mermaid: true
---

`WWDC2022`(Platforms State of the Union)에서 `Swift`의 변경점을 정리해봅니다.

개별 세션의 내용을 참조한 내용들도 있으며, 글은 계속 수정될 수 있습니다.

### WidgetKit

WatchOS용 앱에서 사용하던 `WidgetKit`을 이제 iOS의 잠금화면용 Widget에서도 사용할 수 있게 되었습니다.

애플워치의 Complication을 iOS로 확장시켰습니다.

모든 위젯은 iOS, WatchOS에서 모두 작동. 기존 WatchOS용 WidgetKit을 그대로 사용할 수 있습니다.

플랫폼의 차이를 자동으로 관리합니다. 잠금화면의 위젯색상을 조정해서 가독성을 높였습니다.

기존 WatchOS에서 사용했던 것처럼 3가지 타입이 있습니다. `Circular`, `Retanglar`, `Inline`.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/162b2015-bac9-4ddb-b2d9-7017d1004447)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1b21913c-3321-4886-a1bb-349d34601b7c)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0d67bb74-62f9-4544-910c-9e08865f0432)

### Live Activities

`WidgetKit`으로 만들 수 있는 잠금화면에서 실시간으로 업데이트시켜줄 수 있는 뷰입니다.

업데이트 시 변화에 애니메이션을 줄 수 있습니다.

운동 경기의 스코어, Uber 택시가 도착하기 까지의 시간, 운동 타이머 등에 활용 예시가 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/69961cdb-d30f-400d-8f5b-949a7928c0d3)

### Messages Collaboration API

메세지, 페이스 타임으로 공유. 앱 내의 콘텐츠 링크를 공유할 때, 이 API로 그 링크를 협업용으로 설정 가능합니다.  
필요한 식별자를 제공하고. 수신인이 링크를 누르면 바로 접속 권한을 줄 수 있습니다.  
Messages의 아이디, 앱 아이디는 비공개로 남게 됩니다.  
공유시트, 드래그 앤 드롭 2가지 방식으로 협업 개시 가능합니다.  
대화가 시작되면 콘텐츠 업데이트에 관한 공지를 Messages에 게시할 수도 있습니다.  