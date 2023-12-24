---
title: (WWDC22) (5) Appstore Connect
author: Cody
date: 2022-06-21 00:34:00 +0800
categories: [iOS, iOS News and Updates]
tags: [Appstore Connect, WWDC, WWDC22]
pin: false
mermaid: true
---

`WWDC2022`에서 `App Store Connect`의 변경점을 정리해봅니다.

## 여러 항목을 하나의 제출로 그룹화

(이 내용은 실제 앱스토어 커넥트에서 변경된 사항을 직접봐야 감이 올거 같네요)

여러 Review item을 하나의 제출로 그룹화할 수 있게 되었습니다. Review item의 수나 유형에 상관없이 24시간 내에 검토가 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b47178da-6685-4e51-8360-6b93679e4ad8)

심사 후 Review item은 승인(Accept)되거나 거부(Reject)되는데, 제출한 모든 Review item이 승인이 될 때까지 Review submission이 승인되지 않습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8c6e6b0b-8568-48f9-84eb-8726f525d987)

거부된 Review item을 심사 제출하기 위한 2가지 방법이 있습니다.

첫번째는 거부된 Review item을 수정한 뒤 다시 심사 제출을 하는 방법입니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/47e673ff-1b2f-4880-ac9a-ec5f54d6e4fd)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0be01e1a-a5cc-4aae-a3f5-0cb2bb3b09fd)

두번째 방법은 거부된 item을 제거해서 승인된 item만 남기는 방법입니다.

제거한 Review item을 다시 심사받으려면 새 제출물의 일부로 포함해서 다시 제출을 해야합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/deea16d5-e25c-4240-a9db-9e0d21a8f7df)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/70850e2f-2953-4582-94f0-8649190aacc8)

Review item은 앱 버전, 앱 내 이벤트, 사용자 지정 제품 페이지, 제품 페이지 최적화 테스트 일 수 있습니다.

## 대부분의 경우 새 앱 버전없이 제출할 수 있도록 선택 가능

대부분의 경우 새 앱 버전없이 제출할 수 있도록 선택 가능합니다.

각 제출물에는 관련 플랫폼들이 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/47838b37-6f47-4ef7-ade6-53d180f989e1)

각 플랫폼은 특정 심사 항목 집합을 지원합니다. 대부분은 iOS 제출의 일부로 심사되고 그룹화가 됩니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/61b619f3-7a40-4daa-8765-be84e522ac6b)

플랫폼 당 하나의 진행중인 심사를 제출할 수 있고, 아래 예제에서는 세가지 버전의 앱을 모두 제출하기 위해 시도하고 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6ee10118-1e08-48d2-8838-325950409d73)

일단 iOS 앱 제출만 자세히 보자면, 앱 심사는 제출물에 있는 모든 항목을 앱 버전과 비교하여 검토하고 모든 항목이 일치하는지 확인합니다. 제출물에 앱버전이 있으면 심사에 사용되는 버전이 됩니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1dddc6dc-4b26-4ece-b67a-c255650a8f80)

하지만 이제 앱 버전없이 심사 제출이 가능해졌습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3ddbdbc3-e463-4224-8657-2ce4278eaf80)

물론 앱 버전없이 심사 제출을 하기 위해서는 이전에 승인된 버전의 앱이 필요합니다. 물론 제출되면 이전 버전에 대해 항목이 심사가 됩니다. 즉, 첫번째 iOS버전이 승인되고 새로운 앱 바이너리 없이 언제든지 앱 내 이벤트, 사용자 지정 제품 페이지 최적화 테스트를 제출할 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/300b5e66-859a-4dd9-9744-523d67da9c61)

## 전용 App Review 페이지

진행중인 제출물을 관리하고, App Review와 Communicate하며, 최근 완료된 제출물을 볼 수 있는 전용 App Review 페이지가 생겼습니다. App Store Connect에서 해당 앱 페이지의 왼쪽 메뉴 리스트에 App Review 링크에서 볼 수 있습니다.

전체 심사 워크플로우를 관리할 수 있습니다. 여기서 제출 내용에 대한 개요를 볼 수 있고, 항목을 클릭하면 디테일한 내용을 볼 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/593cb620-70fa-4344-9bd8-8de857d8933e)

웹에서 뿐만 아니라, iOS와 iPadOS에서의 App Store Connect앱에서도 Ready for Review 상태의 제출물을 제출할 수 있게 되었습니다. 심사 제출의 진행도도 추적할 수 있고, 항목을 제거할 수도 있습니다. 리젝 사유를 검토하고 앱 심사에 회신하여 제출 내용을 관리할 수 있을 뿐만 아니라 상태 업데이트에 대한 알림도 바로 받을 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/bd3423d6-f73a-4d1e-af4e-8b8db0598f1b)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/296ae246-f5aa-47a0-a003-fba9311da0e7)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/52fa2401-7bd5-4864-86e0-f728183e4698)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ce3d5087-0c09-4ab9-98d1-d5920681f1bc)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/dcffaae3-9512-4865-a841-c26cd1e0b735)

## App Store Connect API

앱 워크플로우를 커스터마이징하고 자동화하는데 유용한 App Store Conenct API가 있습니다.

작년엔 앱 클립, Xcode Cloud, 인 앱 이벤트, 커스텀 제품 페이지, 제품 페이지 최적화 등을 추가했었고,

올 여름엔 전체 인 앱 구매, 구독 라이프사이클을 관리하기 위한 API 집합, 고객 리뷰 및 개발자의 응답, 앱 중단 분석이 추가될 예정입니다.

1. 전체 인 앱 구매, 구독 라이프사이클을 관리하기 위한 API 집합

: 구독을 자체 리소스로 분류, 앱 내 구매나 구독을 생성, 편집, 삭제할 수 있는 모든 권한이 부여됩니다. 가격을 관리하고, 심사를 제출하며, 특별행사 및 프로모션 코드를 만들 수 있습니다.

이 새 API를 통해 인앱 구매 및 구독 워크플로우를 자동화하는 방법들을 만들 수 있을 것입니다

2. 고객 리뷰 및 개발자의 응답

: 고객 리뷰를 가져오고, 응답할 수 있는 기능이 추가되었습니다. 고객 상호작용을 중심으로 커스터마이징된 워크플로우를 구축할 수 있을 것입니다.

3. 앱 중단 분석

: 앱에서 중단을 식별하고 제거하는 것은 성능을 향상시키고 사용자 환경을 개선할 수 있을 것 입니다. 지금까지는 앱 중단 rate같은 수치만 볼 수 있었지만, 올 여름에는 앱 중단 진단 유형이 추가됩니다. 가장 많이 중단되는 위치를 찾아낼 수 있습니다. 그리고 Diagnostic logs relationship을 통해 중단 시그니쳐에 대한 자세한 스택 추적도 볼 수 있게 됩니다.

디테일한 내용은 아래 세션들에서 제공합니다.

Identify trends with Power and Performance API. Track down hangs with Xcode and on-device detection.

지난 4년간 RestAPI를 통해 App Store Connect를 자동화할 수 있도록 지원을 하고 있고, 올 가을부터는 XML피드를 제공하지 않을 것입니다. 앞으로는 App Store Connect API를 사용하도록 권장합니다.