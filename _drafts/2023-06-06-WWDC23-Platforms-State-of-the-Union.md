---
title: (WWDC23) Platforms State of the Union
author: Cody
date: 2023-06-06 00:34:00 +0800
categories: [iOS]
tags: [Swift, Swift5.9, WWDC, WWDC23]
pin: false
mermaid: true
---
`Platforms State of the Union`을 훝고 갑니다.

추후 각 세션 정리 예정입니다.

## Swift

### Swift Macros

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/44042f11-9715-4bba-b361-f7c313b1d5b5)


`Boilerplate` 코드를 좀 더 깔끔하게 만들어 줄 수 있는 방법입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/942f0a99-f228-4a5f-85c7-43a4571f5e69)

위처럼 `@`가 붙은 attribute일 수도 있고, 독립적으로 `#`이 붙은 형태일 수도 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/192dc9e9-a8dd-4f90-afc4-ef71f3e3e8a5)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/88a9a845-f8c1-4e77-b322-68198559e46a)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6658dc4e-c3e8-4f2c-9633-f55c3287b60f)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/61d58f08-a9de-4e57-9590-c34f089f9a4d)

이 `#URL Macro`는 `URL`을 초기화하여 언래핑시켜주는 코드인데, 해당 `Macro`에서 `Expand Macro`를 통해 확인이 됩니다.

그 뿐만 아니라 유효한 `URL String`인지도 컴파일 타임에 체크해주고, 올바른 코드를 작성할 수 있도록 커스텀 피드백을 할 수 있게 해줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4402032f-92f8-431d-8002-fc87d639ec58)

`fetchContent(_:completion:)` 함수에 `async/await`을 사용하고 싶을 땐, 위처럼 `@AddAsync` 매크로를 붙여서 쉽게 작성할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/318de7e3-4de0-4c89-8c33-430677c10375)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e397d60d-0796-4369-8222-0fabf796a24e)

`Quick Help`로 `Macro` 내용을 확인할 수 있고,

내부적으로 어떻게 동작하는지를 보고 싶을 땐, `breakpoint`를 걸고 비동기함수를 실행하면 에디터에서 해당코드를 확장해서 보여줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/57b9d3d4-ce0a-4a54-ae01-635d77d529e2)

### `Swift와 C++간 상호운용성`

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ad4f401f-496d-44a8-8aa5-95c436e45265)

중간 브릿징 레이어 없이 두 언어를 모두 사용, 컴파일러 플래그를 설정해서 클래스, 함수, 벡터와 같은 템플릿 전문화를 서로 공유하며 사용가능합니다.

### SwiftUI

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8bec8e35-0e0e-4e29-8fe0-5ae60b9a1b66)

`Swift Charts`의 파이차트 지원 및 차트 선택, 새 `Inspector API`, `Mapkit` 지원, `Overlay`, `Look around` 등이 추가되었지만, `애니메이션 개선`에 중점을 두었습니다. 애니메이션을 처음으로 되돌리고, 중단시키고, 취소하는 등의 고급기능들이 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a11a1bab-9d03-4ff7-af1d-54ce2f7e0609)

duration, bounce 파라미터를 가진 spring 애니메이션은 사용자 제스춰의 속도를 자동으로 반영해서 자연스럽게 전환시켜줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b1ed8749-33ab-40d3-b899-6151b4779ec8)

`SF Symbols`도 `SwiftUI`에선 애니메이션 효과를 줄 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1de29562-f1a8-43cd-a776-570fa030e8c9)

그리고 `Multi-part Animation`의 경우 `AnimationPhase`라는 새 API를 이용해서 단 몇줄로 정교한 애니메이션을 줄 수 있게 되었습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b96c4a15-53ad-43a8-8a13-caf612c14253)

이제부터 `SwiftUI`와 `Mapkit API`는 키프레임을 지원하여 무엇이든 애니메이션할 수 있습니다!

특정시간에 여러 속성을 정의하고, 위처럼 중간중간에 카메라 `pitch`, `heading`, `position` 을 독립적으로 애니메이션처리를 할 수 있습니다.

### Data Flow

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/bd9b1707-d21f-437c-97fa-098bbf65d464)

위와 같이 사용하던 기존의 `@Published` 프로퍼티와 `ObservableObject`를

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a7731310-4e58-4007-84b7-8b0d95d8be05)

`Swift`의 새 `Macro`를 적용하여 이렇게 심플하게 사용이 가능해집니다!

위처럼 작성하면 모든 `public` 프로퍼티들은 자동으로 `Published`가 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/5cb52cbf-da8f-4700-b6cc-634dac5347c9)

그리고 해당 `@Observable` 객체를 사용할 때 기존처럼 `@ObservedObject` 프로퍼티 래퍼를 사용할 필요없이,

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8794b64e-ab77-4016-a170-40b45024802d)

이렇게 직접 참조하기만 해도 됩니다!

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4c0c5853-7ab0-48eb-be85-44493ea72b05)

`@Observable` 매크로는 `View`에서 사용하는 특정 프로퍼티가 변경될 때 `View`의 `Body`가 `re-evaluated`됩니다. `View`에서 사용하지 않는 필드를 수정할 때는 `invalidation`이 전혀 발생하지 않습니다.

### SwiftData

`CoreData`는 `Objective-C`를 기반으로 만들어져 `Swift`의 이점을 충분히 사용할 수가 없기 때문에

`Swift`에 맞춘 데이터모델링 및 관리를 위한 `SwiftData` 프레임워크가 만들어졌습니다.

외부 파일 포멧과 상관없이 코드에만 집중할 수 있게 만들어졌습니다. `Swift`의 새 `Macro` 시스템을 사용한 `Streamlined API`를 제공합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c3df3853-7367-4613-9774-d99db2f3f30a)
위와 같이 `Swift`로 작성된 모델이 있을 때,

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4206301c-e609-4ef5-a5b0-9dca0e0ad85a)
`@Model` `Marcro`를 붙여줄 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7f3ad1f4-2cfe-49df-ab2e-2925a2f7113a)

이 한줄 코드를 붙여줌으로써 `지속성 자동 활성화`, `iCloud` 싱크, `Undo`, `Redo`, `Spotlight` 검색 등을 쓸 수 있게 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/eeb1c384-3e97-4538-88fd-83e65b07dbdb)

고유값을 가지는 등 특정 속성을 붙이고 싶을 땐 위처럼 프로퍼티에 annotation을 붙여주면 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/39d851ab-5ebc-4d0e-a156-3db4ae4ba345)

Codable 프로토콜을 사용한 enum, struct도 SwiftData에서 사용할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ed2510f2-e6a6-417e-91a5-befdaba4e6c7)

위처럼 `SwiftData`에 사용할 모델에 `@Model` `Macro`를 달아주고,

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/af1fd18e-9297-4dc7-bc8e-43f12720bcd0)

앱 루트에서 `SwiftData`의 컨테이너를 설정하기 위한 `modifier`를 추가합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4ffc3ab3-f7a8-4caa-ba3d-e66b2a4da7bd)

그리고 사용할 View에서 `modelContext`를 `@Environment`로 주입하여 지속되도록 해줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/342e0dcd-0ef5-4eca-8f66-deab88599409)

마지막으로 데이터를 사용할 `View`에서 데이터를 연결하여 사용하면 됩니다.

이 때 새로운 `@Query` 프로퍼티 래퍼를 사용합니다. 저장된 데이터를 불러오기 때문에 프로토타입은 필요가 없습니다.

위젯도 같은 방식으로 `SwiftData`를 사용할 수 있습니다. `Shared Container`가 활성화 되었다면 `SwiftData`가 자동으로 동일한 API를 사용하여 위젯에서 같은 데이터에 직접 액세스할 수 있도록 합니다.

### WidgetKit

iOS17용으로 업데이트하면 간단한 변경으로 Standby에서 위젯을 보여줄 수 있게 됩니다. 이 위젯은 iPad 잠금화면에서도 사용할 수 있고, MacOS Sonoma의 바탕화면용으로도 사용할 수 있게 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b24e3496-a9f1-45ab-8409-fc37aa89bd69)

그리고 위젯에서 상호작용을 지원합니다.

위젯의 코드는 컨텐츠 생성을 위해 비동기적으로 실행되고, 빌드된 `SwiftUI`의 `View`는 아카이브에 저장됩니다.

후에 위젯을 그리게 될 때, 아카이브를 불러오고 백그라운드에서 렌더링한 후 시스템UI의 일부로 표시됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4b4ae3a6-91ba-4d92-8b81-0f3a634831c4)

사용자가 버튼을 탭하면 해당 `Extension`이 다시 실행되어 작업을 처리하고 다시 `UI`를 업데이트합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/bd219acb-4c47-4636-88c3-789bd9b8e091)

이 아키텍쳐로 `MacOS`에서 `iPhone`의 위젯을 `Mac`에서 매끄럽게 표시해줍니다.
`Continuity`(위 아키텍쳐를 말하는 듯?) 덕분에 위젯 아카이브를 네트워크로 `Mac`에게 보내고 사용자 상호작용을 `iPhone`에서 처리하기 위해 다시 보낼 수 있게 되었습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/fdea9ff1-c115-44a3-b7f0-f93e83dc9ce3)

위젯을 `백그라운드를 식별`하고, `WidgetKit`에서 제공하는 기본값을 사용하도록 `padding`을 업데이트하기만 하면 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6c70eb8d-15f6-4aeb-83a6-de734668cb62)

`SwiftUI`의 스택 기반 레이아웃을 사용하면 시스템이 컨텍스트에 따라 위젯의 색상과 간격을 조정할 수 있습니다.

위젯에 SwiftUI 버튼이나 토글을 추가하여 상호 작용도 쉽게 채택할 수 있습니다. 이러한 컨트롤에서 앱 의도를 트리거하기 위한 새로운 지원은 필요에 따라 Extension을 시작합니다. 위젯의 콘텐츠가 업데이트되면 시스템은 Keynote의 Magic Move처럼 작동하는 전환 애니메이션을 트리거합니다. 이동된 요소는 새 위치로 이동하고 추가되거나 제거된 요소는 부드럽게 페이드 인/아웃됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/16fe1185-082f-4f49-a5ba-e0350ac0d918)

`컨테이너 백그라운드`를 설정하고,

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e787003f-b95a-4bc8-a2cf-3aa00687156d)

`padding`을 제거해서, `iOS17` 기본 위젯 패딩을 사용하도록 하면 됩니다.

`#Preview` 라는 `Swift Macro`를 사용하면 `Preview`에서 위젯을 타임라인으로 어떻게 변화하는지 전환 애니메이션까지 확인이 가능합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/233e35a3-3269-44c4-a237-7059531d1525)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/285062a1-c0f6-4939-b99e-1f47b9f88ee8)

trasition Modifier를 추가. `Preview Canvas`에서는 iPad 잠금화면 위젯, Standby 위젯일 때 어떻게 보여지는지 확인을 할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c6c7acaa-aabf-4ff6-b122-6a02dee2fe11)

`SwiftUI`는 배경이 제거될 때 내 위젯이 어떻게 보이는지 사용자 정의할 수 있는 새로운 `showsWidgetContainerBackground` 변수를 제공합니다. 배경이 제거됐을 때 `birdBath`를 보여주는 모습.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7a528637-cdc6-43ca-85ba-c645b47d0d20)

### `App Intent`

`App Shortcut`에 `Intent`를 래핑하면 `Spotlight` 결과에서 앱아이콘 옆에 대화식으로 보여줄 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b3eaca21-56fc-4fc5-9435-c5c38f12d849)

`shortTitle`, `이미지나 symbol`, 그리고 앱의 info.plist에서 `앱아이콘을 보완하는 배경색`을 제공하면 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d2adeacb-2ffa-4b54-94cb-41d581e90ca0)

`App Shortcut`은 업데이트된 `Shortcut` 앱에 표시되며, 여기서 사용자가 자동으로 실행되게 하거나 홈화면에 추가하거나, 직접 바로가기를 만드는데 사용할 수 있습니다.

### TipKit

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e03d421f-58dd-4b9e-829e-33e018c1996c)

사용자가 특정 기능을 잘 알지 못할 것이 예상될 때 `TipKit`을 사용하면 적시에 적절한 기능에 대해 사용자에게 지능적으로 교육함으로써 이 문제를 해결하는 데 도움이 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/191efe81-fc26-4b9b-b7d3-022cca64bfdc)

`TipKit`에는 사용자가 시스템 앱에서 보는 데 익숙한 것과 일치하는 템플릿이 포함되어 있으며 앱의 모양과 느낌에 맞게 쉽게 사용자 지정할 수 있습니다. 다른 장치에서 해당 Tip을 본 경우에 이미 본 팁을 다시 표시하지 않도록 빈도를 관리할 수도 있습니다.

### AirDrop

`iOS17`에서는 공유시트를 통하지 않고도 `AirDrop`을 통해서 컨텐츠를 다른 디바이스로 전달할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c4831d08-fb3a-4205-ab39-9b02e79d1cfc)

`SwiftUI`에서 `ShareLink`를 통하거나, `UIKit` 뷰 컨트롤러에서 `activityItemsConfiguration`을 적용할 수 있습니다.

(이후 하드웨어, Vision Pro에 대한 내용은 생략합니다)