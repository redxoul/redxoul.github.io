---
title: CocoaPods로 Package 배포하기
author: Cody
date: 2023-01-23 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Apple, Cocoa Pods]
pin: false
mermaid: true
---
앞 포스팅에서 `Swift Package Manager`로 `Package` 배포를 해봤는데,

해당 패키지를 `CocoaPods`으로도 배포하고 싶어서 시도해보고 경험을 공유해봅니다.

1. 일단 아무 폴더에다 들어가서 터미널을 통해 아래 명령어를 써줍니다.

```
pod lib create (패키지명)
```

2. 그리고 5가지 질문에 자신의 패키지에 맞는 선택을 하면

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ed54d4e9-f4be-4472-9e6b-06d1b0c0538b)

pod templete이 해당 폴더에 만들어집니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/75463237-1b78-448c-abef-38d98bf5e9ed)

3. 저 폴더 내용 중 `podspec` 파일만 이미 만들어놨던 패키지 폴더에 복사하고,

`podspec` 파일을 열어서 해당 패키지 구조와 스펙에 맞게 내용을 수정해줍니다.

저는 version, summary, author를 수정하고,

homepage, source를 SPM으로 배포했던 github주소로 변경해주고,

source_files 위치를 패키지 구조에 맞게 수정해주었습니다.

```
Pod::Spec.new do |s|
  s.name             = 'AESwift'
  s.version          = '1.0.1'
  s.summary          = 'Data and NSData Extension made for AES encryption/decryption only with Swift.'
  s.homepage         = 'https://github.com/redxoul/AESwift'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'redxoul' => 'soldolly@gmail.com' }
  s.source           = { :git => 'https://github.com/redxoul/AESwift.git', :tag => s.version.to_s }
  s.ios.deployment_target = '10.0'
  s.swift_version = "5.0"

  s.source_files = 'Sources/AESwift/`/*'
end
```

4. 여기까지 하고 `tag`를 '`숫자로만 된 버전`명(1.0.1)'으로 만들고 Push를 해줍니다.

Push된 태그로 github에서 release를 시켜줍니다. `Cocoapods`에서는 숫자만으로 버전이 되어 있어야 배포가 된다고 합니다.

5. 이제 `Cocoapods`에 register를 하는 과정입니다.

터미널을 열고 아래 명령어를 실행해줍니다.

```
pod trunk register (email 주소) (user 이름)
```

그러면 아래처럼 이메일 인증을 하라는 메세지가 나오고, 이메일이 옵니다.

이메일 내용의 링크를 누르면 인증이 됩니다.

```
[!] Please verify the session by clicking the link in the verification email that has been sent to soldolly@gmail.com
```

6. 이제 터미널에서 패키지가 있는 폴더로 가서 Pod 배포를 위한 유효성, 무결성 검사를 위해 아래 명령어를 실행합니다.

```
pod lib lint
```

그러면 어떤 부분을 더 수정해야하는지 빨간글씨로 알려줍니다.

```
 -> AESwift
 -> AESwift (1.0.1)
    - WARN  | [iOS] swift: The validator used Swift `4.0` by default because no Swift version was specified. To specify a Swift version during validation, add the `swift_versions` attribute in your podspec. Note that usage of a `.swift-version` file is now deprecated.
    - NOTE  | xcodebuild:  note: Using codesigning identity override: -
    - NOTE  | [iOS] xcodebuild:  note: Building targets in dependency order
    - NOTE  | [iOS] xcodebuild:  /var/folders/_n/1kbpw1917mn44yxrh9jlm83w0000gn/T/CocoaPods-Lint-20230123-23136-tzr2wi-AESwift/App.xcodeproj: warning: The iOS Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of supported deployment target versions is 11.0 to 16.2.99. (in target 'App' from project 'App')
    - NOTE  | [iOS] xcodebuild:  Pods.xcodeproj: warning: The iOS Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of supported deployment target versions is 11.0 to 16.2.99. (in target 'AESwift' from project 'Pods')
    - NOTE  | [iOS] xcodebuild:  note: Metadata extraction skipped. No AppIntents.framework dependency found. (in target 'AESwift' from project 'Pods')
    - NOTE  | [iOS] xcodebuild:  note: Metadata extraction skipped. No AppIntents.framework dependency found. (in target 'App' from project 'App')
    - NOTE  | [iOS] xcodebuild:  Pods.xcodeproj: warning: The iOS Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of supported deployment target versions is 11.0 to 16.2.99. (in target 'Pods-App' from project 'Pods')
```

```
[!] AESwift did not pass validation, due to 1 warning (but you can use `--allow-warnings` to ignore it).
```

`WARN`으로 알려준 부분을 수정하면 됩니다.

저는 처음에 `podspec`에서 `swift_version`을 명시하지 않아서 위와 같이 경고가 떴고,

해당 사항을 명시해준 후에 다시 `pod lib lint`를 해주니 아래와 같이 패스가 되었습니다.

```
AESwift passed validation.
```

7. 이제 터미널에서 패키지 경로로 들어가서 아래 명령어를 실행해줍니다.

```
pod trunk push
```

6단계에서 이미 validate 검사를 다 해주었기 때문에 아래와 같이 진행이 됩니다.

```
[!] Found podspec `AESwift.podspec`
Updating spec repo `trunk`
Validating podspec
 -> AESwift (1.0.3)
    - NOTE  | xcodebuild:  note: Using codesigning identity override: -
    - NOTE  | [iOS] xcodebuild:  note: Building targets in dependency order
    - NOTE  | [iOS] xcodebuild:  Pods.xcodeproj: warning: The iOS Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of supported deployment target versions is 11.0 to 16.2.99. (in target 'Pods-App' from project 'Pods')
    - NOTE  | [iOS] xcodebuild:  Pods.xcodeproj: warning: The iOS Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of supported deployment target versions is 11.0 to 16.2.99. (in target 'AESwift' from project 'Pods')
    - NOTE  | [iOS] xcodebuild:  note: Metadata extraction skipped. No AppIntents.framework dependency found. (in target 'AESwift' from project 'Pods')
    - NOTE  | [iOS] xcodebuild:  note: Metadata extraction skipped. No AppIntents.framework dependency found. (in target 'App' from project 'App')
    - NOTE  | [iOS] xcodebuild:  /var/folders/_n/1kbpw1917mn44yxrh9jlm83w0000gn/T/CocoaPods-Lint-20230123-24556-glnkfe-AESwift/App.xcodeproj: warning: The iOS Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 10.0, but the range of supported deployment target versions is 11.0 to 16.2.99. (in target 'App' from project 'App')

Updating spec repo `trunk`

--------------------------------------------------------------------------------
 🎉  Congrats

 🚀  AESwift (1.0.3) successfully published
 📅  January 23rd, 09:33
 🌎  https://cocoapods.org/pods/AESwift
 👍  Tell your friends!
--------------------------------------------------------------------------------
```

8. 이제 `Cocoapods`로 `pod install`이 잘 되는지 확인해봅니다!

(`Cocoapods`에 등록이 되기까지 시간이 조금 걸릴 수 있습니다. 저는 몇 시간이 걸린 느낌이네요.)

pod install도 잘 되고,

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3c707f2f-2408-4643-aa0c-bbae2eb8b4d6)

사용도 아주 잘 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/dff6068a-cbf6-4b58-8798-b6fad639ddc9)

(시간이 좀 걸렸지만)
CocoaPods.org에 등록이 되면 아래처럼 검색이 됩니다
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/28857553-9139-475c-89b3-7fb02a6c0e1d)

여기까지 `Swift Package Manager`와 `CocoaPods`으로 패키지를 배포하게 된 경험을 공유해봤습니다.