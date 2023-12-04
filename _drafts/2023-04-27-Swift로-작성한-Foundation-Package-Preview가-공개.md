---
title: Swift로 작성한 Foundation Package Preview가 공개
author: Cody
date: 2023-04-27 00:34:00 +0800
categories: [iOS, iOS News and Updates]
tags: [Apple, iOS, Swift]
pin: false
mermaid: true
---

[Foundation Package Preview Now Available](https://www.swift.org/blog/foundation-preview-now-available/)

드디어 **Swift**로 작성된 **Foundation Package**의 **Preview**가 **Github**에 공개됐습니다.

아직 모두 완성된 것은 아니지만 초기 테스트와 컨트리뷰트를 위해 빠르게 공개되었다고 합니다.

공개된 요소들은 아래와 같습니다.

**FoundationEssentials**

- AttributedString
- Data
- Date
- DateInterval
- JSONEncoder
- JSONDecoder
- Predicate
- String extensions
- UUID

**Internationalization**

- Calendar
- TimeZone
- Locale
- DateComponents
- FormatStyle
- ParseStrategy

Calendar의 경우 일부 벤치마크에서 기존 대비 20%, FormatStyle을 사용한 날짜 포멧은 표준 날짜 및 시간 템플릿을 사용한 포멧 벤치마크에서 150% 성능 개선이 있다고 합니다. JSON 디코딩 시간도 200~500%로 개선되었다고 합니다.
