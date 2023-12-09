---
title: iOS17부터는 Device Support 대신 CoreDevice를 통해 Xcode와 통신합니다.
author: Cody
date: 2023-09-22 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [iOS17, Xcode15, Xcode14.3, CoreDevice, DeviceSupport]
pin: false
mermaid: true
---
이번에 Xcode15와 iOS17의 정식 버전이 배포가 되었는데요.  
  
바로 Xcode를 최신버전으로 올려서 사용하면 좋지만 현업에서는 그렇게하지 못하는 케이스도 많이 있습니다.  
그래서 이런 경우 이전 Xcode버전에 최신버전 iOS의 **Device Support**파일을 넣어서 개발을 해왔습니다.  
  
이번 Xcode15에서는 이럴때 필요한 iOS17의 **Device Support**파일을 제공하고 있지 않고 있고, Github에도 올라와 있지 않습니다.  
  
대신 Xcode15에서 iOS17 디버깅을 할 수 있도록 내부 다운로드를 진행한 후  
아래 명령어를 터미널에서 실행하면

```bash
defaults write com.apple.dt.Xcode DVTEnableCoreDevice enabled
```

  
아래처럼 Xcode14에서 **CoreDevice**를 통해 디버깅할 수 있게 됩니다.
![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/7c8c70c6-b69b-4e8d-80c2-af7a15cd1eb0)


iOS17 이상부터는 새로운 디바이스 스택인 **CoreDevice**를 사용해서 디바이스와 통신을 하기 때문이라고 합니다.  
(그래서 더이상 iOS17 이상의 **Device Support** 파일은 없는거죠)  
Xcode15 이후로는 새 iOS 디버깅을 위해서 **Device Support**를 받거나 새 Xcode버전을 받지 않아도 되고, 시스템의 디바이스 스택만 업데이트되면 될 것으로 보입니다.  
  
위 명령어로 **CoreDevice**를 사용하기 위해서는 최소 Xcode14.3 이상이 필요합니다.  
  
참고: [Missing iOS 17 device support files](https://developer.apple.com/forums/thread/730947?answerId=756665022#756665022)
