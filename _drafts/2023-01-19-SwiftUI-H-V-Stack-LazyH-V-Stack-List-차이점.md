---
title: (SwiftUI) H(V)Stack, LazyH(V)Stack, List 차이점
author: Cody
date: 2023-01-19 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, SwiftUI, iOS13, HStack, VStack, LazyHStack, LazyVStack, List]
pin: false
mermaid: true
---

가장 큰 차이점은 재사용 여부.

## HStack, VStack
초기화 시점에 모든 View(Cell)을 생성  
개별 View를 하나씩 넣어줄 땐 최대 10개까지만 가능.  
스크롤을 포함하지 않음.

## LazyHStack, LazyVStack
초기화 시점에 모든 VIew(Cell)을 생성시키지 않음.  
`최대 index 31`까지의 데이터의 View(Cell)을 생성.  
스크롤을 포함하지 않음.

## List
초기화 시점에 모든 Cell을 생성시키지 않음. UITableView와 가장 유사하게 보여질 View(Cell)만 생성. View(Cell)의 추가, 삭제 기능도 있음.  
스크롤을 포함하고 있음. 외관도 테이블뷰와 유사함.