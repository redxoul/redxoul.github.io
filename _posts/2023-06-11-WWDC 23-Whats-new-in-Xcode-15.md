---
title: (WWDC 23) What's new in Xcode 15(feat. Release Notes)
author: Cody
date: 2023-06-11 00:34:00 +0800
categories: [iOS]
tags: [Xcode, Xcode15, WWDC, WWDC23]
pin: false
mermaid: true
---
(`Xcode 15`부터는 최소 버전이 `iOS12` 입니다🤩 개발자의 `Inner peace!`)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/16454f36-3482-4ee3-b7c3-b0d7c45be569)

그리고 `Xcode`의 용량이 대폭 줄어들었습니다!

(하지만 시뮬레이터들 미포함 용량입니다)

## Xcode Download

이제 개발자 사이트에서 `Xcode`를 다운로드 받을 때, 함께 받을 시뮬레이터를 선택해서 받을 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/fcd83b20-af15-409e-bead-a134f9beabbe)

---

## Code Completion

코드 자동완성이 좀 더 똑똑해졌다고 합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6ed00348-b2ab-4106-a0c2-7f18935b5caf)

예시로 `PlantSummaryRow`라는 `Swift`파일을 생성하고, 해당 파일에서 `struct`를 작성할 때 파일명을 자동완성에서 추천해줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b3e09e82-4a3f-4274-ab53-d99d56eae63e)

일부 `completion`이 표시되는 방식을 개선.

기본 인수가 있는 함수를 호출할 때, 원하는 매개변수를 정확히 가져오는 것이 불편했던 것이 개선되었습니다.

기본 인수의 `가능한 모든 순열`을 보여줍니다.

예시에서는 `VStack`에서 `.frame` 수정자를 선택하고 오른쪽화살표(→)를 누르면 확장되어 해당 수정자에서 가능한 모든 인수의 순열을 보여줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3a5da4b2-95e4-4f8d-b5f9-a439f1eb8f64)

자동완성이 더 많은 컨텍스트를 인식하고 잘 추천해줍니다.

아까 작성한 `VStack`의 `frame` 수정자 위에 추가로 수정자를 작성하기 위해 `.`을 입력하면 `Xcode`가 `.padding`이 `View`에서 가장 많이 사용되는 수정자라는 것을 알기 때문에 `.padding()`을 제일 위에 추천해줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b4f0879e-53e3-4394-a129-20e1058e204d)

마찬가지로 `Text`에서 수정자를 입력하려고 할 때는 `.font(_:)`가 가장 먼저 추천이 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/82d36ca9-a14b-4709-a87e-8ed36979c95c)

`editor`의 제안은 주변의 코드도 고려합니다.

위에서 이미 `Text`에 `.font(_:)` 수정자를 추가했기 때문에, `Text`에 또다른 수정자를 추가하고자 할 때는 (font를 또 수정할 일은 적기 때문에)그 다음 추천으로 `.bold()`를 추천해줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/34b9bf84-277e-4686-a0fb-d07709f7097f)

위 같은 코드에서 현재 코드 앞부분에서 `coordinate`의 `latitude`를 먼저 입력했고, 보통 쌍으로 `longtitude`를 입력하기 때문에 상단에 추천을 해줍니다.

---

## Asset Catalog의 자동 완성

`Asset Catalog`에서도 자동 완성이 강화되었습니다!(대박)

`Color, Image Asset`은 이제 `Swift Symbols`로서 코드 자동완성이 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/20d19a68-c68b-4202-8625-db81d6c357ba)

이렇게 `Asset`이 있다면, 문자열로 입력하는 것이 아닌 `Swift`코드로 작성할 수 있게 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/00a67b36-5cc1-43cc-a1a7-544528808571)
Asset 의 이름을 바꾸면(Clouds -> MultipleClouds)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ce0fca91-a41e-4f80-b15e-7053161add96)
해당Asset 을 사용하는 코드에서 Error 를 발생

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/47198765-199b-4ca6-8385-ec0709372b5c)
Asset의 이름을 에디터에서 추천

만약 `Asset`의 이름을 바꾸면(Clouds -> MultipleClouds) 빌드시 해당 `Asset`을 사용하는 코드에서 `Error`를 발생시켜주기 때문에 런타임에 이미지가 잘못들어가거나 누락되는 것을 컴파일 단에서 방지시켜줄 수 있습니다.

## String Catalog

`String Catalog`를 통해 `Localization`을 한 곳에 모아 중앙에서 검토하고 업데이트할 수 있도록 해줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/36a799b0-409d-4126-8f62-9c723790be82)

`Edit>Convert to String Catalog` 를 통해 `String Catalog`로 프로젝트를 마이그레이션하도록 할 수 있습니다.

해당 기능 사용시 변환 가능한 모든 `storyboard`, `strings`, `stringsdict` 파일을 보여주는 시트를 보여줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/dc9813cd-bf0c-49bc-bc6d-63bc9d0f89d8)

마이그레이션하게 되면 위처럼 모든 번역이 단일 에디터로 보여집니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2880af75-9400-4ceb-b5ad-faabfd79dd7f)

에디터 왼쪽 사이드바에서 지원하는 `언어의 번역 진행도`를 파악할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c3d3c402-7b17-413d-9342-9ababb572aa3)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/5bedc3b2-21f8-481b-a2bf-9bb3d6f3d205)

최신화하기도 쉽도록, 빌드할 때마다 소스코드로부터 모든 문자열을 가져옵니다.  
새 문자열을 추가하거나 제거되면 편집기가 영향을 받는 언어에 주석을 달아주고 관련 문자열에 배지를 표시해줍니다.  
(자세한 내용은 'Discover String Catalogs' 세션 참조)

---

## Documentation

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/58f16b18-4d67-4c43-ad57-e4f835691a7e)
문서를 더 쉽게 읽을 수 있도록 새 스타일이 적용되었습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4250dc33-0572-4811-981f-45d7f92de696)
문서화 주석을 작성할 때, `Editor>Assistant` 를 열고, `Documentation Preview`를 선택하면

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/89d12898-1804-40db-92e7-1933583d1474)
문서를 실시간으로 `Preview`로 실시간 확인할 수 있습니다.


![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/63d6aed0-0fa5-42b5-ab63-49f5487b0381)
`Documentation Catalog`에 이미지를 등록했다면 문서에서 `이미지도 첨부가 가능`해집니다.

(자세한 내용은 'Create rich documentation with Swift-DocC' 세션 참조)

---

## Swift Macros

(`Macro`는 여러 세션에서 다루네요!)

간결하고 이해하기 쉬운 코드를 만들기 위해 새로 도입된 기능입니다.

표현력이 풍부한 `API`를 작성하고 `boilerplate` 코드를 줄일 수 있도록 도와줍니다.

 `Xcode`에 잘 통합되어 완전한 가시성을 제공하여 `Macro` 생성 코드를 앱의 다른 코드들처럼 처리할 수 있게 해줍니다.

`Macro`는 `SDK`의 `Swift package`의 일부입니다.

`Swift standard Library`, `Foundation`, `Swift Data` 프레임워크와 같은 여러 Apple 자체 프레임워크에서 `Macro`를 활용하고 있습니다. (`@Observable`, `#Predicate`, `@Model` 등)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7f5cabc2-aae1-473e-9a90-4a7d6e0f9cf5)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ee54b83b-8b10-477a-8441-f325e75f1382)

다른 사람과 공유할 커스텀한 `Macro` `package`를 만들 수도 있습니다.

`Macro`의 장점은 보통의 Swift 코드를 생성한다는 점입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e656e16c-2dee-4055-9f7f-84bf1f0bae38)
Quick Action(Command+Shift+A)로 Expand Macro

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/5f0af57a-a932-4b5c-a3cd-8be862274488)

`Macro`가 수행하는 작업을 보거나 `Macro` 생성 코드에서 디버그하려는 경우 `Quick Action`으로 `Macro`를 바로 확장할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7bb79f05-e959-48e9-8457-351dc3997c3e)

필요하면 이렇게 확장된 `Macro`코드에 `Breakpoint`를 설정할 수도 있습니다.

(자세한 내용은 'Expand on Swift macros', 'Write Swift macros' 세션을 참조)

---

## Previews

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/806a4f11-7e92-425e-b130-a77866c97e0d)

`#Preview Macro`를 사용해서 Preview API를 더 쉽게 작성할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4879d884-ce6b-4407-879c-e395a9152a7c)

`UIKit, AppKit`에서도 `#Preview Macro`를 통해 `Preview`를 사용할 수 있게 되었습니다!

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a6076e4b-2570-44fd-be1e-05d7e7324fe3)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/576cf96b-ab06-479f-8901-39690c75d87a)

`Preview`로 위젯을 개발할 때 `API`가 `time-base` 위젯을 구축하기 위한 새 워크플로우를 도입합니다.

`Preview` 아래에 모든 항목을 표시하는 새로운 영역이 추가되었습니다.

해당 영역으로 위젯의 전환을 탐색해볼 수 있습니다.

(자세한 내용은 'Build programmatic UI with Xcode Previews' 세션 참조)

---

## Xcode Navigation

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/504c7eec-6230-4783-8dba-55349e2eabb6)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1ab95d8b-86e2-41ca-8afe-cf6065ec30bb)

북마크 기능! 입니다.

북마크를 원하는 곳에서 퀵액션으로 `Bookmark`를 하면, 네비게이션의 북마크에서 빠르게 찾아볼 수도 있고,

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/02bb986c-7883-4a45-a560-8eae35992603)

해당 북마크의 설명을 추가로 달 수도 있게 되었습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/20631697-3e8c-454c-bd17-2b6790a78935)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/005b736c-81b2-464e-a07f-3550e2798361)

`북마크를 정렬하거나 그룹화하여 정리`할 수도 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9da200d3-e7e5-4cb0-870c-56965a7171bb)

할일 목록처럼 왼편을 클릭해서 `체크표시`를 해둘수도 있고, 삭제도 가능합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d7f1ca34-189b-4a65-9c21-4dbfdd98cdd2)

찾기 쿼리를 북마크화할 수도 있습니다.

(위처럼 TODO 주석을 검색해서 Bookmark로 등록하고 새로고침 버튼으로 최신화)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/944964f5-96cd-47ee-8304-5994e6e25c8f)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/23b14f63-239a-430c-aa71-59770f9569e3)

검색 타입에 `Confoming Types`로 원하는 타입을 검색해서,  북마크화 할 수도 있습니다.

---

## Sharing (Source Control)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7be3c995-45ac-464a-9357-cb8d52bb4221)

모든 변경사항을 검토할 수 있는 `Changes Navigation`과 `Commit Editor`가 추가되었습니다!

(Source Control Navigator)

각 블럭 위아래의 핸들을 잡아내려서 더 많은 소스가 보이도록 할 수도 있습니다.

해당 화면에서 소스를 바로 수정하고 반영할 수도 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/629cd505-6269-42a8-aae9-cea3d0aa8a39)
파랑bar는 Staging된 수정사항, 흰색bar는 Unstaging 상태의 수정사항

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/87c3c373-a696-4a38-adde-d92f0061bf22)

이 화면에서의 내 편집 내용이 변경 표시줄과 `Changes Navigation`에 위처럼 표시됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/86e5709e-c156-4b76-8c8b-ae366c9e3ddc)

`Changes bar`를 눌러 `Stage Change`를 누르면 

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/12434c09-ca10-463d-b9a1-5ff46a773937)

수정사항이 `Staging`됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/57410cce-1e0e-47b2-90a2-a0a4487a1193)

`Staging`된 것을 `Unstaging` 시킬 수도 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7b143aa5-0d9a-4adb-83f4-fd5ef79164f2)

`Commit Message`를 작성하고 `Commit`을 올리면 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/492f349f-03ca-4a71-aa16-9ebe226ed68a)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e2705d0f-263b-4472-920a-fc56e88a1273)

커밋 뷰어에서 동료들과 수정사항을 확인하고, 내 커밋 내역을 클릭해서 `Push`를 할 수 있습니다.

---

## Testing

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6b657f35-da52-46ac-b460-4b1fce2f1a35)

`Test` 결과를 실시간으로 실행하거나 `reporting`할 때 45% 더 빨라졌습니다.

`Test Navigator`는 `Test Plan`을 중심으로 구성되어 관심 있는 `Test`를 더 쉽게 찾을 수 있습니다.  
필터를 사용하여 예상되는 실패와 같은 결과 유형을 기반으로 테스트를 찾을 수도 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/657c831f-abc4-413a-8fd7-122a53e60ef6)

`Xcode`, `Xcode Cloud`에서 `Test`를 실행하고 나면 결과가 요약된 `Test Report`가 제공됩니다.

- `Top Insights`: 결과에 대한 패턴 기반 분석.
- `Tests`: Test 수행 방법 요약. 서로 다른 장치, 구성에서 얼마나 많은 Test가 통과/실패했는지 파악.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/04a9ba46-04fc-4f9b-b4ff-733453ba1bf5)
`Insights`: Test결과를 분석하여 이전에는 확인하기 어려웠을 수 있는 잠재적으로 관련된 오류를 식별. 또한 전체 제품군이 결과를 반환하는 데 더 오래 걸릴 수 있는 Test 실행에 대해 경고.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2d6c2f34-8b28-4286-8ab3-1bf497173688)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b80fb925-0778-449d-80ef-46d3f7be08a3)

`Overview`에서 결과 유형, `run` `destination` 및 `Test Plan configuration`에 대한 필터와 함께 모든 `Test` 실행을 표시하는 `Test list`으로 이동할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8b06a1f3-49d5-426f-9775-4c84d0a16150)
  
`Test`에서 동일한 오류 메시지와 함께 어떤 오류가 있는지(여러 언어에서 계정 버튼을 탭하지 못한 것)를 확인했습니다.  
자세히 알아보려면 개별 클래스를 살펴보거나 개별 `Test` 메서드에 대한 새로운 전용 `Test` 세부 정보 보기로 이동할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a9b7b31d-77c4-496e-be38-edaf3a25d2cc)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/93e62996-240a-4e42-b6d5-b336c7b9493b)

`Test` 세부 정보 보기에는 모든 실행의 분석 및 전용 성능 메트릭 탭을 포함하여 결과 데이터를 탐색하는 다양한 방법에 대한 탭이 포함되어 있습니다.

새로운 자동화 탐색기를 사용하면 `UI Test` 실패를 디버깅이 좀 더 쉬워집니다.  
탐색기는 대화형이라서 `Test` 재생을 보거나 타임라인을 스크러빙할 수 있습니다.  
터치 또는 마우스 이벤트가 비디오에 오버레이됩니다.  
내 `Test`가 계정 버튼을 탭하지 못한 경우와 같이 실패 지점에서 내 앱의 `UI` 계층 구조를 검사할 수 있습니다.

(자세한 내용은 'Fix failures faster with Xcode test reports' 세션 참조)

---

## Debugging

`OSLog`: 런타임에 대한 정보를 캡쳐하기 위한 도구. 로그 출력을 체계적으로 유지하는 구조화되고 사용자 정의 가능한 `Logging` 메커니즘을 제공합니다.

```swift
import OSLog

let logger = Logger(subsystem: "BackyardBirdsData", category: "Account")

func login(password: String) -> Error? {
    var error: Error? = nil
    logger.info("Logging in user '\(username)'...")

    // ...

    if let error {
        logger.error("User '\(username)' failed to log in. Error: \(error)")
    } else {
        loggedIn = true
        logger.notice("User '\(username)' logged in successfully.")
    }
    return error
}
```
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/87bab9d3-84fc-4b25-9fa7-037b484eb64b)

그리고 `Xcode 15`의 콘솔은 하위 시스템 범주 및 심각도와 같은 로그 데이터에 대한 복잡한 필터링을 수행하는 기능을 포함하여 `OSLog`에 대한 전체 지원을 도입합니다.

로그 표시가 매우 깔끔해졌습니다. 추가 데이터가 깔끔하게 숨겨져 있는 로그 콘텐츠에 초점을 맞춥니다.

각 로그 항목의 배경색은 심각도를 나타내므로 긴 로그 출력 스트림에서 중요한 메시지를 쉽게 검색할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/61bcd917-9f72-4123-ac20-9ecf721a5a0d)

메타데이터 필드는 기본적으로 숨겨져 있지만 클릭 몇 번이면 됩니다.  
보고 싶은 필드만 선택할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/553f3534-c1eb-44e9-a974-9d386d7ac2c8)

특정 항목을 찾고 있는 경우 필터 필드를 사용하여 검색 범위를 좁힐 수 있습니다.  
메타데이터 또는 로그의 전체 텍스트를 필터링할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/41d55fca-7943-4320-af7d-4f86c40f35d1)

로그 항목에서 이를 생성한 코드 줄로 바로 이동할 수도 있습니다.

(자세한 내용은 'Debug with structured logging' 세션 참조)

---

## Distribution 배포

배포 프로세스가 개선되었습니다.

`Xcode Cloud`에서는 빌드버전 관리, 앱 서명, 배포 프로파일 등을 자동으로 관리합니다.

`Xcode 15`에서는 추가로 2가지 요소를 더 처리합니다.

(1) TestFlight 테스트 세부정보: 소스코드 바로 옆에 테스트 노트를 포함하기 위한 지원을 추가합니다. 이는 배포를 위해 TestFlight 빌드에 자동으로 첨부되므로 메모가 앱 바로 옆에 테스터에게 표시됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2a36440d-5b64-4465-abe9-8f3ccceb56a6)

(2) Mac용 앱에 대한 공증(Notarize)을 지원:

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/eab41fb9-1d9a-4371-9736-933d565c29b3)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/89df2fca-3ef9-43bc-8c84-0c16f7b17207)

워크플로우에 `Notarization post action`을 추가하기만 하면 `Xcode Cloud`가 나머지 작업을 수행합니다. 빌드가 완료되면 앱이 자동으로 `notarize`되고 `staple`되며 다운로드할 준비가 됩니다.

앱 공증은 사용자에게 매우 중요합니다. 앱이 변조되지 않았음을 알려줍니다.

그러나 의존하는 `SDK` 및 `Framework`의 무결성을 신뢰할 수 있는 것도 그만큼 중요합니다.  
이러한 보증을 제공하기 위해 `Xcode`는 `XCFrameworks`에 대한 서명 검증을 도입합니다.  
작성자는 `Framework`의 콘텐츠에 디지털 서명을 할 수 있으며 `Xcode`에서 이러한 서명을 바로 확인할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3cf65138-5546-4056-9fec-4aca3dbebea5)

`Framework`의 `inspector`에는 새로운 서명 조각이 있습니다.  
누가 `Framework`를 생성하고 서명했는지 정확히 알려줍니다.

따라서 `Framework`를 업데이트할 때 변경되면 문제에 대한 명확한 경고를 받지만 그보다 더 많은 것이 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6c825e3e-6d93-42cc-9465-3bc0e8c4e2bf)

작성자는 이제 Framework에 Privacy Manifest(PrivacyInfo)를 포함할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/20ba5fb2-91a6-4965-b887-b4f2722c1680)

이 `Manifest`는 `Framework`가 중요한 데이터를 사용하고 보호하는 방법을 정확하게 자세히 설명합니다.  
`Privacy Manifest`는 `Framework`와 함께 번들로 제공되므로 서명된 패키지의 일부이기도 하며 해당 콘텐츠가 작성자로부터 직접 온 것임을 신뢰할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/55a549af-3757-4ca0-9834-4ab22515b1d3)

`Xcode`를 사용하여 모든 `Manifest`를 내 앱에 대한 완전한 `Privacy Report`로 요약할 수 있습니다.

이 `Report`는 사용자에게 정확한 정보를 제공할 수 있도록 `App Store Connect`에서 `Privacy Nutirution Label`을 쉽게 작성할 수 있도록 설계되었습니다.  
그리고 `Apple`은 개인 정보에 영향을 미치는 `SDK`와 협력하여 모든 중요한 의존성이 이 귀중한 정보를 제공하도록 합니다.

(자세한 내용은 'Verify app dependencies with digital signatures', 'Get started with privacy manifests' 세션 참조)

## Xcode Distribution

버그 수정, 새 `feature` 브랜치 작업을 할 때, 팀원에게만 배포하고 싶을 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/fefd5286-50e0-4027-b12c-ae50b87eea73)

이제 `Xcode 15`는 `TestFlight 내부 테스트 배포 옵션을 지원`합니다.  
`TestFlight 내부 빌드`는 팀에서만 사용할 수 있으므로 실수로 고객에게 릴리스할 수 없습니다.  
만들기 쉽습니다.  
`App Store Connect`를 통해 앱을 배포할 때 `"TestFlight internal testing only"` 옵션을 선택하기만 하면 됩니다.

(Xcode Organizer 창 TestFlight Internal Only 배포 지원은 현재 사용할 수 없다고 Release Notes에 안내.)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c61d469e-4c69-43b5-83bd-e06354754a75)

`Xcode`에서 `Distribute`할 때, 가장 일반적인 배포 방법과 권장 설정 세트를 번들로 제공합니다.  
`TestFlight Internal Only`을 포함하여 이러한 새 옵션 중 하나를 선택하고 `Distribute`를 클릭하면 완료됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3632a66b-0c2b-4f88-b216-44dc89d33839)

`App Store Connect`를 통해 배포하는 경우 이제 빌드 상태에 대한 데스크톱 알림도 받게 됩니다.  
따라서 앱을 테스트할 준비가 되면 즉시 알림을 받게 됩니다.  
이러한 업데이트를 통해 앱을 빠르게 반복하고 배포할 수 있으므로 팀원, 테스터 및 사용자와 원활하고 빠르게 긴밀하게 작업할 수 있습니다.

(자세한 내용은 'Simplify distibution with Xcode and Xcode Cloud' 세션 참조)

---

## Xcode 15 Beta Release Notes

('What's new in Xcode 15' 세션에서 다루지 않은 Release Notes의 내용을 정리합니다)

### `Overview`

- `iOS 12` 이상, `tvOS` 12 이상 및 `watchOS` 4 이상에서 온디바이스 디버깅을 지원합니다. `Xcode 15` 베타에는 `macOS Ventura 13.3 이상`이 실행되는 `Mac`이 필요합니다.

### `Playgrounds`

- `Playgrounds` 콘솔은 `Xcode 15`의 새 콘솔을 사용하며 인라인 찾기와 같은 기능을 제공합니다.
- `Xcode Playgrounds`의 `Result Sidebar`는 행의 모든 ​​표현식에 대한 요약을 표시하고 새로운 컨트롤을 사용하면 각 표현식에 대한 세부 정보가 포함된 팝오버를 볼 수 있습니다.
- `Playgrounds Map` 템플릿이 최신 `MapKit API`를 사용하도록 업데이트되었습니다.

### `Preview`

- `@objc annotation`을 사용할 때 미리보기가 실패하는 문제가 해결되었습니다.
- 파일을 탐색할 때 캔버스 모드, 장치 설정 및 선택한 미리 보기 장치와 같은 캔버스 설정이 유지됩니다.

### `Signing and Distribution`

- `Xcode` 및 `Xcode Cloud`의 `"Manage version and build number"` 배포 옵션이 앱의 `Framework Dependencies`의 버전 및 빌드 번호를 덮어쓰는 문제가 해결되었습니다. 앱을 배포할 때 `Framework Dependencies`은 원래 버전과 빌드 번호를 유지합니다.
- `Xcode Organizer` 창 `TestFlight Internal Only` 배포 지원은 현재 사용할 수 없습니다.

### `Source Editor`

- `#if...#endif` 블록의 비활성 코드가 이제 흐리게 표시됩니다. 이것은 `Text Editing>Display` 기본 설정 에서 비활성화할 수 있습니다.
- 코드를 별도의 줄로 분할하는 `"Format to multiple lines"` 리팩터링 작업이 추가됐습니다.

### `Swift Package`

- `File > New > Package…` 메뉴 명령을 사용하여 `다양한 유형의 Swift 패키지를 생성`할 수 있습니다. 템플릿에는 `Macro`, `Package Plugin` 및 `swift-argument-parser를 사용하도록 구성된 command line tool configure가 포함`됩니다.