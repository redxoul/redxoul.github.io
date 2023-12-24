---
title: (WWDC22) (2) SwiftUI & Charts
author: Cody
date: 2022-06-12 00:34:00 +0800
categories: [iOS, iOS News and Updates]
tags: [SwiftUI, Charts, WWDC, WWDC22]
pin: false
mermaid: true
---

`WWDC2022`(Platforms State of the Union)에서 `SwiftUI`의 변경점을 정리해봅니다.

개별 세션의 내용을 참조한 내용들도 있으며, 글은 계속 수정될 수 있습니다.

## Navigation Stack

새 내비게이션 API. 가장 알맞은 네비게이션 스타일을 쉽게 표현하게 해줍니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/50147615-86ac-4427-9d39-318c3de75c46)

선택사항을 쉽게 저장하고 복구할 수 있고, 네비게이션 스택의 내용 전체를 대체할 수도 있습니다.

이 기능으로 인해 앱 실행단계 설정, 사이즈 클래스간의 전환 관리, 딥링크 응답 같은 동작을 쉽게 처리가 가능합니다.

## NavigationSplitView

선택사항을 추적하는 Sidebar, NavigationStack이 포함된 뷰입니다.

Sidebar 선택사항이 변경되면 콘텐츠도 변경시켜 줍니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/259c0139-f940-4e2c-9ab3-fe6263e0c30b)

## Scene API

맥용 앱을 쉽게 작성하기 위한 Window 생성 및 관리를 위한 API들이 추가되었습다.

Window 추가, 바로가기 설정, 기본 크기, 위치, 크기 조정 등을 위한 Modifier들이 추가되었습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f0c50081-2b09-4069-94df-a9e6cd553ab6)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9bf94706-1fb9-4a96-9094-5f1effce1140)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f4e2821c-9de1-422b-a479-67fc6d19a59d)

위치나 사이즈를 조절하면 SwiftUI가 이를 자동으로 기억해 줍니다.

`.presentationDetents()` Modifier로 크기 조정 가능(Mac용으로 만든 View를 iOS로 적용하기 위함으로 보여집니다)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8d948528-e270-482e-8249-681e2c4010dc)

## MenubarExtra

메뉴바의 아이콘 추가를 위해 `MenubarExtra`가 새로 추가되었습니다.

SwiftUI로 간단하게 Menubar의 기능을 추가할 수가 있게 되었습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1ff0807f-68c4-4f31-92e8-af7204d0b1b8)


## Form

(Grouped)`Form`을 작성할 수 있습니다.

새 MacOS Ventura의 설정 화면도 SwiftUI의 Form으로 작성되어져 있다고 합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/41f01b56-42a4-45e3-a575-102d1628796b)

그리고 해당 API는 iOS, iPadOS에서도 그대로 사용이 가능합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/454386ff-22ad-4d80-a5c8-dda3ae77f5b8)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/29e5560f-6bd6-4b2e-9ef0-f98f3879dce6)


## Controls 개선

SwiftUI의 컨트롤들을 개선되었습니다.

개선된 컨트롤들은 위에서 소개한 Forms에서 사용하기 편리하도록 되어있습니다.

`TextField`에서 `axis`(축) 매개변수가 추가되었습니다.

예를 들어 axis를 .vertical로 지정하면 텍스트에 맞도록 수직으로 높이가 늘어나게 됩니다.

`.lineLimit` Modifier를 지정할 때 범위연산자로 지정해 줄 수도 있습니다.

긴 텍스트의 경우 TextEditor 사용을 추천.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d2113491-bee0-4bfd-9236-ee3fd0e1a819)

`MultiDatePicker`가 추가되었습니다.

이제 DatePicker하나로 여러 날짜를 입력받기가 쉬워졌습니다!
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b5b7143b-f05a-4e26-917e-2d6ac76d83ee)

`Mixed-state Toggles`가 추가되었습니다.

DisclosureGroup내의 각 toggle들은 각자의 단일값과 바인딩되어 있고,

label 내의 집계 toggle은 모든 값이 일치하지 않을 경우 혼합상태를 표시하는 모든 바인딩을 수집합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f66d4d02-5aeb-4386-acfc-7bd66ed9c78c)

`Formatted Stepper`가 추가되었습니다.

Stepper는 WatchOS에서도 그대로 사용가능합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/bde44e4b-83cb-47ec-a2e1-c0717e10d67e)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6bf9e74c-b010-485a-92f6-bf614a98491c)


## Tables

2021년에 소개했던 MacOS용 Table을 iPadOS에서도 사용할 수 있게 되었습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/5885c079-d72b-4b78-b9e5-5ccec31263c4)

iOS에서도 Table을 사용할 수는 있지만, 한 화면에서 기본 Column만 보여지게 된다고 합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/457e1f11-27b5-47ce-9ec7-472b14e66315)

Table에서 아무것도 선택하지 않았을 때, 1개 선택했을 때, 여러개를 선택했을 경우에 대한 컨텍스트 메뉴를 설정할 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/562eb4e7-23f0-4b5d-99b8-30e224b9bacd)


## iPad에서의 도구모음

iPadOS의 앱에서 상단 도구모음이 MacOS처럼 바뀌었습니다.

사용자는 이제 사용자 지정 재정렬을 할 수 있고, MacOS에서 사용했던 것과 동일한 API의 명시적 식별자로 구현할 수 있습니다.

(물론 모든 것을 지원하는 것은 아니라고 합니다)

사용자 지정 도구모음 구성을 자동으로 저장하고 복원하는 UI를 지원합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6bbf0493-603e-48fa-a5cd-14d0e1f0a6a5)

## 기존 .searchable Modifier 개선

기존 SwiftUI에서 제공하던 `.searchable` Modifier에  `scopes` 기능이 추가되었습니다.

scopes를 설정하면, 도구모음 아래에 segment같은 바가 생기고 결과를 필터링하는데 도움을 줄 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ada1d9f0-24e0-4096-a08a-576d1eb8fdb7)

해당 컨트롤은 iOS에서는 세그먼트 컨트롤과 같은 형태로 보여지게 됩니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8e3a51da-1177-4766-9ef9-5a494b252cdb)

## PhotosPicker

멀티플랫폼용 PhotosPicker가 추가되었습니다. toolbar가 아니더라도 앱 어디에라도 사용할 수 있습니다.

선택할 컨텐츠 유형에 대한 필터링도 할 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/60172f56-cb83-4120-bcd0-de4fab050db4)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8b00d614-9aeb-46ba-abdf-c6c836c4d5ff)

## ShareLink API

ShareLink가 추가되었습니다. 이 또한 멀티플랫폼 사용 가능한 표준 인터페이스입니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ac086503-5888-452f-b97f-50a1ab70f37a)

툴바는 물론, 컨텍스트 메뉴에도 넣을 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2f013784-e9c3-433b-b151-72c0c571f3be)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7ef51094-10d5-41bd-8f67-76e844f3ab41)

## Transferable Protocol

위에서 소개했던 PhotosPicker와 ShareLink API는 Transferable Protocol을 따릅니다.

Transferable 타입은 드래그앤드롭과 같은 UI의 기능을 사용할 수 있도록 해주는데,

이 경우 payload 타입을 허용하는 새로운 dropDestination API를 사용합니다.

completion 블록에서 drop한 위치와 함께, 수신된 이미지들의 컬렉션을 전달받습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2b18c9e5-820b-4d1d-9c46-062b51eb0f67)

String, Data, URL, Attributed String, Image 등 많은 표준 타입들이 Transferable Protocol을 따릅니다.

이 타입 외에도 커스터마이징한 Transferable 타입을 정할 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/55c40532-d8d2-463d-8363-4e159da72bed)


## Shape Styles

기존 Shape Style들에 개선점이 있습니다.

Color에 `.gradient` 속성이 추가되었습니다. 해당 속성을 사용하면 지정한 색상으로부터 파생된 미묘한 그라데이션이 적용됩니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/82945874-5c48-4ed2-84b8-9753d1ddd276)

그리고 `.shadow` 속성도 생겼습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d9356a49-bb01-45a3-9aed-e3777d5e222f)

기존 SwiftUI의 shape들 뿐만 아니라, `SF Symbols`에도 적용이 가능합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0ad53145-aae3-4acc-9959-0446f144fb9f)


## Grid API

앱 인터페이스의 레이아웃 제어도 개선되었습니다.

기존에 `VStack`, `HStack`을 이용한 레이아웃 모델은 일반적인 레이아웃에서 쓰기 좋지만,

더 유연한 게 필요할 때가 있고 이를 위해서 `Grid API`가 추가되었습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7c969603-392b-4256-b6ae-fa780e5aef89)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/89522ffa-585d-4311-92b1-6fcd0c9a3736)

## Custom Layout API

원하는 어떤 레이아웃도 제작할 수 있는 유연성을 제공합니다.

예시로 '유동 레이아웃'을 만들 수 있습니다.

뷰가 신문기사처럼 배열되고, 공간이 더 필요해지면 다음 단으로 넘어가는 레이아웃이 가능.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/01b4596a-bef9-4705-a8ed-4e905aca24b4)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b7a77e6e-942c-4342-b0e3-a1b5ec71cda1)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c5ed7c56-5f4d-4a5b-8a0a-c7cb39c79e11)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/199dd2f7-fe7d-498f-85ba-97b53928d89f)

다음 컨텐츠의 크기가 첫번째 단을 넘어가면, 그 다음 단으로 컨텐츠가 넘어감.

시계처럼 뷰가 둥글게 펼쳐지는 레이아웃도 제작이 가능합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7eb56fe1-cc48-4cab-89ab-181012aad3ab)

`Custom Layout API`는 쉽게 레이아웃 로직을 재사용할 수 있게 해주고 뷰 코드를 더 단순하고 읽기 쉽게 만들어 줍니다.

`Layout Protocol`을 사용하여 효율적인 레이아웃을 작성할 수 있으며. `AnyLayout` 타입을 이용해서 레이아웃들을 쉽게 전환할 수 있도록 도와줍니다.

(디테일한 내용은 'Compose custom layouts with SwiftUI' 세션을 참조)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b0c6d3ac-9f13-4112-a2a6-ee0dc8e72aee)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a2a5daa4-c341-4a84-90f7-d3fc0de842d0)

`Half sheet`가 네이티브 뷰로 새로 추가되었습니다(bottom sheet를 이제 커스텀으로 만들지 않아도 되네요!?)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4ee108bb-5784-4f7d-ad9d-3a5bf2632013)

## UIHostingConfiguration

기존 앱에 SwiftUI를 점진적으로 적용하기 쉽도록 SwiftUI의 뷰를 열 수 있는 특별한 CollectionView Cell이 도입되었습니다.

UIKit 으로 만든 앱에 CollectionView가 이미 있다면 SwiftUI식 선언형 구문으로 커스텀 셀을 작성 가능하고,

기존 UIKit의 UICollectionView의 여러 기능(쓸어넘기기, 셀 배경 등)을 지원합니다.

(기존의 UIViewRepresentable, UIViewControllerRepresentable외에도 `UIHostingConfiguration`을 통해 UIKit에서 SwiftUI(식 구문)을 사용할 수 있게 함으로써, 점차 SwiftUI로 넘어갈 수 있게 지원해주려는 것 같습니다)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/df450850-076d-4ccd-938c-e0e8c2f15579)

## AlertView에 TextField를 추가
이제 ViewBuilder를 따르는 alert Modifier를 통해서 TextField를 추가할 수가 있습니다.

## Label을 포함한 View를 ViewBuilder가 자동 정렬 및 스타일링
Label을 가지고 있는 Control, Section, 기타 View들의 경우 ViewBuilder컨텐츠가 여러 View들을 제목, 부제목과 같은 계층적 요소로 자동으로 정렬하고 스타일링 합니다. 계층이 아닌 수평으로 정렬하도록 의도하고 싶다면 HStack을 써야 합니다.

## List 개선
List에서 Section Footer를 지원합니다.

## Text, Image 변경에 따른 애니메이션
이제 Text와 Image가 변경되었을 때 애니메이션이 기본적으로 적용됩니다.

이를 적용하고 싶지 않다면, `.contentsTransition(.identity)`를 사용하면 됩니다.

## List, Form에서 스크롤시 소프트웨어 키보드 자동 Dismiss

List, Form에서 스크롤을 할 때 소프트웨어 키보드가 활성화 되어 있었다면, 자동으로 키보드가 Dismiss 됩니다.

이를 적용하고 싶지 않다면, `.scrollDismissesKeyboard(.never)`를 사용하면 됩니다.

## Swift Charts Framework
SwiftUI기반. SwiftUI와 동일한 선언형 구문으로 시각정보를 담은 코드를 쉽게 읽고 쓸 수 있도록 되어있습니다.

앱에 알맞는 정보표시 방식을 사용자화하여 차트를 작성할 수 있게 되었습니다
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d895ae30-1e04-43d3-bfb5-281171709493)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c0d8960e-61a1-4a83-8186-e613554c6028)

라인, 바 차트, 히트맵, 스트림 그래프 등 다양한 그래프를 디테일하게 그릴 수 있습니다.

차트의 접근성도 SwiftUI처럼 지원합니다. 사용자화가 쉽다고 하네요.

차트에 애니메이션을 넣을 수 있고, 데이터를 실시간으로 반영해서 차트가 변경됩니다.

## ViewThatFits

가용공간에 따라 `HStack`이나 `VStack`으로 전환하게 해줍니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1951e7a6-8a88-46d3-85e6-dcebdaa7345e)

## Preview

- 대량으로 편집할 때, Preview가 더이상 일시중지되지 않습니다. 대신 업데이트 빈도를 동적으로 조정합니다.
- 이제 PreviewProvider를 수정하지 않고도 SwiftUI의 Preview에 추가된 컨트롤을 통해 다양한 Variants(Darkmode, Orientation, Dynamic Type)을 확인할 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2a4e5a16-d7bd-45bd-bbbe-27d0b0c85ca4)