---
title: (WWDC23) Privacy Manifests
author: Cody
date: 2023-07-29 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Swift, WWDC23, WWDC, PrivacyManifests, Xcode15, Xcode]
pin: false
mermaid: true
---
[https://developer.apple.com/news/?id=z6fu1dcu](https://developer.apple.com/news/?id=z6fu1dcu) 

얼마전(7/27) 위와 같은 내용이 Apple Developer 공지로 올라왔습니다.  
   
올해(2023년) 가을부터 사유를 명시해야 하는 API(타사 SDK포함)를 사용하는 새로운 앱 또는 앱 업데이트를 App Store Connect로 업로드하는 경우, 앱 Privacy Manifests(개인정보 보호 목록)에 승인된 사유를 제공하지 않으면 알림(notice)을 받게 된다고 합니다.  
그리고 2024년 봄부터는 `반드시` 앱의 '개인정보 보호 목록'에 앱이 API를 사용하는 방식을 정확하게 반영하는 승인된 사유를 포함하고 있어야 `App Store Connect`로 업로드가 가능해집니다.  
   
`Privacy Manifests`라는게 있었군요?  
게다가 이번 `WWDC23`에서도 해당 세션이 있었다고 합니다?  
[https://developer.apple.com/videos/play/wwdc2023/10060/](https://developer.apple.com/videos/play/wwdc2023/10060/)

`Swift`도 `Xcode`도 `Macro`도 중요하지만, 제일 중요한 건 앱을 `App Store`에 올리는 것이니 샤샥 확인해봅니다.  

## Privacy Manifests

이제부터는 `App Store` 리뷰 가이드라인에 따라  
앱의 코드 상의 모든 데이터 수집 및 트래킹에 대한 사안들을 `Privacy Manifests`에 명시해야 합니다.  
   
여기에는 본 앱은 물론 앱에서 사용중인 `서드파티 SDK`에서의 데이터 수집 및 트래킹도 마찬가지입니다.  
`서드파티 SDK`가 `Privacy Manifests`를 포함시켜서 `SDK`를 제공을 하는 방법으로 이를 쉽고 명확하게 해결할 수 있습니다.  
본 앱의 `Privacy Manifest`에서는 `서드파티 SDK`가 수집하는 정보를 위해서 항목을 명시할 필요가 없습니다.  
   
Xcode15(beta)에서부터 새 파일 생성시 아래와 같이 App Privacy 템플릿(`.xcprivacy`)을 제공합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e545be75-ef05-40d3-8e99-64e134cfcf0b)


생성된 파일의 `App Privacy Configuration`을 채워넣으면 됩니다.  
   
아래는 떱떱디씨 영상에서의 예시입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/70be61ca-5b35-4911-af99-890327eff3cb)
WWDC23 Get started with privacy manifests

뭔가 기존 `info.plist`에 명시해주던 `privacy description`보다 더 디테일한 느낌이 옵니다.  
이 파일은  
`SDK가 수집하는 데이터 유형(Collected Data Type),`  
`각 데이터 유형이 사용되는 방식(Collection Purposes),`  
`사용자와 연결되는지 여부(Linked to User),`  
`앱 추적 투명성 정책에 정의된 대로 추적에 사용되는지 여부(Used for Tracking)`  
`를 선언하는 프로퍼티 목록`입니다.  
   
위 예시를 보자면  
`Name`(사용자 이름)을 수집하지만 트래킹에는 사용하지 않고 앱 기능상, 제품 개인화를 목적으로 사용  
`Photos And Videos`를 사용자와 연결시켜 사용하고 트래킹에는 사용하지 않고 앱 기능상 필요하여 사용  
`User ID`를 사용자와 연결시켜 사용하고, 트래킹은 하지 않으면 앱 기능상, 제품 개인화를 목적으로 사용한다고 되어 있고  
마지막으로 `Privacy Tracking`은 사용하지 않는다고 선언되어 있습니다.  
  
앱스토어에 제품 페이지마다 사용자들에게 제공하는 `'앱이 수집하는 개인정보'와 매우 비슷하지만 조금 더 항목들이 자세`합니다.  
   
명시해주어야 할 개인정보 수집 항목은 아래 페이지들에 명시가 되어있습니다.

1. [Describing data use in privacy manifests](https://developer.apple.com/documentation/bundleresources/privacy_manifest_files/describing_data_use_in_privacy_manifests)
2. [App privacy details on the App Store](https://developer.apple.com/app-store/app-privacy-details/#linked-data)

## Privacy Report

`App Store`에 제출하기 위해 앱을 빌드할 때 `Xcode15`에서는 앱 프로젝트의 모든 `Privacy Manifests`를 집계하고 선언된 데이터 사용을 요약하는 `개인 정보 보고서(Privacy Report)를 생성`할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/fa4c0e70-24d6-447a-8108-9bc9e26b370f)
WWDC23 Get started with privacy manifests

`Generate Privacy Report`로 내보내진 파일은 아래와 같은 `PDF`파일입니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/65398fad-f2e9-4059-86f7-bc086cf557fd)
WWDC23 Get started with privacy manifests

파일의 구성은 `App Store Connect`에 제공하는 `Privacy Nutrition Labels`와 유사하게 되어있어, 이 `Report`를 참조해서 앱 및 Dependency의 개인정보 보호 관행을 쉽게 검토하고, 작성할 수 있습니다.  
~~(자동으로 App Store Connect에 작성해준다는 건줄 알았는데 그건 아니었..ㅠ)~~  
   
 

## Tracking Domains

`Privacy Manifests`의 또다른 기능 중 하나는  
앱에서 네트워크 연결을 더 쉽게 이해하고 제어할 수 있도록 하여 `도메인 추적을 관리하는 데 도움`을 주는 것입니다.  
허가 없이 사람들을 추적할 의도는 없지만 추적에 사용되는 도메인에 의도하지 않은 네트워크 연결을 생성하는 극단적인 경우가 있을 수 있습니다.  
  
예를 들어 많은 `서드파티 SDK`는 사람을 추적하기 전에 `추적 권한 상태를 확인`합니다.  
`서드파티 SDK`는 라이브러리의 함수 또는 일부 구성 변경을 통해 달리 지정하지 않는 한 기본적으로 추적을 사용하도록 설정할 수 있는데, 이로 인해 엣지 케이스가 발생할 수 있습니다.  
 

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7ec1f763-7cf9-43d7-8122-50bbda48a3de)
WWDC23 Get started with privacy manifests

`앱 혹은 서드파티 SDK 개발자`가 사용자의 허가 없이 사용자를 추적하지 않도록 하기 위해 추적을 선언하는 `Privacy Manifests`에는 `Privacy Tracking Domain이 포함`됩니다.  
사용자가 추적 권한을 제공하지 않은 경우, `iOS 17`은 앱에 포함된 `Privacy Manifests`에 지정된 `Privacy Tracking Domain`에 대한 연결을 자동으로 차단합니다.  
이를 통해 이러한 엣지 케이스를 제거할 수 있습니다.  
   
 

## Instrument - Points of Interest

만약 트래킹에 사용하는 도메인이 트래킹하지 않는 기능에도 사용하는 도메인이라면 어떻게 해야할까요?  
사용자가 iOS17에서 추적권한을 주지 않으면 비트래킹을 위한 기능들도 모두 막히게 됩니다.  
앱 혹은 서드파티SDK 개발자가 tracking.example.com에서 추적 기능을 호스팅하고 non-tracking.example.com에서 비추적 기능을 따로 호스팅하는 방법을 고려해볼 수 있습니다.(넵??ㅇㅁㅇ!!)  
이 경우 개발자가 `Tracking Domain`에 연결되어 있는지 항상 인식하지 못할 수 있는데 `Instrument`의 `Points of Interest instrument`가 이를 도와줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c2de6b82-e8af-4a2d-8c8b-f8ca7899aace)
WWDC23 Get started with privacy manifests

이를 사용하면 도메인이 App Tracking Transparency 정책에 따라 사용이 되는지 여부를 볼 수 있습니다.  
 

## Required Reason APIs

`Fingerprinting`은 피하면서 사용자에게 도움이 되는 중요한 사용 케이스를 지원하기 위해 `Required Reason APIs`라고 하는 API의 새로운 카테고리가 생겨났습니다.  
(Fingerprinting: 쿠키없이 그 외의 정보로 사용자를 추적하는 기법)  
   
각 카테고리에는 사용 케이스에 따라 이러한 API에 액세스할 수 있는 승인된 Reasons 목록이 있습니다.  
예를 들어 `Required Reason APIs` 중 하나는 파일 시스템의 여유 공간을 나타내는 `NSFileSystemFreeSize`입니다.  
이 `Required Reason API`는 파일을 디스크에 쓰기 전에 이 API를 사용하여 디스크 공간이 충분한지 확인하는 것을 지원합니다.  
   
그 밖에 `Required Reason API`는 아래 문서에 정리되어 있고 앞으로 계속 업데이트될 예정이라고 합니다.  
[https://developer.apple.com/documentation/bundleresources/privacy_manifest_files/describing_use_of_required_reason_api](https://developer.apple.com/documentation/bundleresources/privacy_manifest_files/describing_use_of_required_reason_api)

> 위 Required Reason API 문서에는 `File timestamp APIs, System boot time APIs, Disk space APIs, Active keyboard APIs,` `User defaults APIs`가 포함되어 있습니다.
{: .prompt-tip }

   
위 `Required Reason API`를 사용하여 반환된 데이터들은 다른 용도로 사용하면 안 됩니다.  
`Required Reason API`를 사용하는 이유를 명확하게 설명하고 `서드파티 SDK` 개발자가 동일한 작업을 쉽게 수행할 수 있도록 `Privacy Manifests`에 이 정보가 포함시켜야 합니다.  
`Required Reason API`에 액세스하는 `앱 또는 서드파티 SDK`는 `API 카테고리` 및 `API를 사용하는 모든 이유를 선언`해야 합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/14a5e881-13b2-4175-8c23-28c05e9dca10)
WWDC23 Get started with privacy manifests

### Privacy-impacting SDKs

사용자 개인 정보 보호에 특히 큰 영향을 미치는 일부 서드파티 SDK를 식별해서 이를 `Privacy-impacting SDKs`라고 부르게 됩니다.  
이러한 `서드파티 SDK 및 향후 업데이트 목록은 Apple 개발자 문서에 게시될 예정`입니다.  
`Privacy-impacting SDK`에서 정보를 얻는 것이 특히 중요하기 때문에 `Privacy-impacting SDK`를 포함하는 앱은 `Privacy Manifests` 와 함께 `해당 SDK의 사본을 포함`해야 합니다.  
(`Privacy-impacting SDKs` 문서 참조..이지만 아직 문서가 보이지 않습니다?)  
   
그리고 앞으로 `서드파티 SDK를 배포할 때는 아래 2가지를 반드시 포함`시켜야 합니다.

- `Privacy Manifest`
- `서명`: Xcode15에서 지원. SDK의 무결성을 확인.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9c47e212-f888-4146-87c3-ca0b14e5ca3b)
WWDC23 Get started with privacy manifests

   
SDK 서명을 위한 내용은 아래 세션을 확인하면 됩니다.  
([WWDC23. Verify app dependencies with digital signatures](https://developer.apple.com/videos/play/wwdc2023/10061/))  
 

---

2023년 가을부터 업데이트 및 신규 앱을 `App Store Connect`로 올릴때,  
`Privacy-impacting SDK`를 사용하는 앱의 경우 `해당 SDK에 Privacy Manifests 와 서명이 포함되어 있는지 체크`하고,  
포함되어 있지 않으면 개발자에게 이메일을 전송합니다.  
`Required Reason API`를 선언하지 않고 해당 API를 사용하는 앱에 대해서도 개발자에게 이메일을 전송합니다.  
`2024년 봄부터 해당 이슈가 리뷰 가이드라인에 포함될 것`이기 때문에 그 전에 해당 문제를 해결해야 합니다.