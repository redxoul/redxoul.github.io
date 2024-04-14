---
title: (Architecture) Clean Architecture
author: Cody
date: 2024-04-14 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Swift, Clean Architecture, Architecture]
pin: false
mermaid: true
---

## Clean Architecture
여러가지 아키텍처들이 있지만 이들의 공통적인 목표가 있음.

- 프레임워크에 독립적일 것.
- Testability. 비즈니스 규칙은 UI, DB, 웹 서버 또는 기타 외부요소 없이 테스트 가능.
- UI와 독립적. 시스템의 나머지 부분을 변경하지 않고도, UI를 쉽게 변경할 수 있음.  
(예를 들어, 비즈니스 규칙을 변경하지 않도고 웹 UI를 콘솔 UI로 교체 가능)
- DB와 독립적. 비즈니스 규칙 DB에 바인딩되지 않음.
- 외부 기관으로부터 독립적. 실제 비즈니스 규칙은 외부 세계에 대해 전혀 알지 못함.

아래의 다이어그램은 이러한 모든 아키텍처들을 하나의 실행가능한 아이디어로 통합하기 위한 시도.

## Dependency Rule
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/739219a4-5210-41c8-bfdb-6fff09ec5c21)

동심원의 바깥에서 안쪽으로 들어갈수록 고수준.  
바깥쪽 원은 매커니즘, 내부 원은 정책.

이 아키텍처를 동작하는 최우선 규칙은 'Dependency Rule'.   
소스 코드 의존성은 안쪽으로만 가리킬 수 있음. 내부원에서는 외부원에 대해서 알 수가 없어야 함.   
외부원에서 선언된 항목(함수, 클래스, 변수 등)의 이름은 내부원의 코드에서 언급되어서는 안됨.   
그리고 외부원에서 사용되는 데이터 형식은 내부원에서 사용되어서는 안되며, 특히 외부원의 프레임워크에 의해 생성된 것이라면 더욱 그러함.   
> 외부원의 어떤 것도 내부원에 영향을 미치지 말아야 함.
{: .prompt-tip }



![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/41a5898b-d1df-42e8-947d-f2bbcb877d66)
내부원일수록 추상적이고, 일반적이고, 변화가 없으며   
외부원일수록 구체적이고, 특정하고, 변화가 많음.

## 예제
영화검색을 해주는 Clean Arch 기반의 MVVM 예제.  
https://github.com/kudoleh/iOS-Clean-Architecture-MVVM

| 검색 | 검색결과 | 검색상세 |
| -- | -- | -- |
| ![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b13fba89-6631-476e-b2e5-771bde4e7114) | ![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/62982e4c-731d-40a0-b10a-165622b18b91) | ![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/298ccaca-df9f-4b8b-bc83-1e35c7242513) |

### Entities: Enterprise Business Rules
`엔티티`는 전사적 비즈니스 규칙을 캡슐화.   
메서드가 포함된 객체일 수도 있고, 데이터 구조나 함수 집합일 수도 있음.
전사적이지는 않지만 단일 앱인 경우, 앱의 비즈니스 객체가 됨.
운영 관점에서 특정 앱에 어떤 변경이 필요하더라도 `엔티티` 계층은 절대 영향을 주어서는 안됨.

예제에서는 `Movie` 구조체.
```swift
struct Movie: Equatable, Identifiable {
    typealias Identifier = String
    enum Genre {
        case adventure
        case scienceFiction
    }
    let id: Identifier
    let title: String?
    let genre: Genre?
    let posterPath: String?
    let overview: String?
    let releaseDate: Date?
}

struct MoviesPage: Equatable {
    let page: Int
    let totalPages: Int
    let movies: [Movie]
}
```

### Use Cases: Application Business Rules
앱 고유의 비즈니스 규칙을 캡슐화.   
`UseCase`는 엔티티로 들어오고 나가는 데이터 흐름을 조정하고, `엔티티`가 자신의 핵심 비즈니스 규칙을 사용해서 `UseCase`의 목적을 달성하도록 만듬.   
이 계층에서의 변경이 `엔티티`에 영량을 주어서는 안됨. DB, UI, 다른 공통 프레임워크와 같은 외부요소에서 발생한 변경이 `UseCase`에 영향을 줘서도 안됨.   
하지만 운영 관점에서 앱이 변경되면 `UseCase`가 영향을 받게 됨.

예제에서의 `SearchMoviesUseCase` 프로토콜.
```swift
protocol SearchMoviesUseCase {
    func execute(
        requestValue: SearchMoviesUseCaseRequestValue,
        cached: @escaping (MoviesPage) -> Void,
        completion: @escaping (Result<MoviesPage, Error>) -> Void
    ) -> Cancellable?
}


```

### Interface Adapter(Presenter, View, Controllers, Gateway)
어댑터는 데이터를 `UseCase와 엔티티에게 가장 편리한 형식`에서 `DB나 웹 같은 외부 agency에게 가장 편리한 형식`으로 변환.   
이 계층은 GUI의 MVC 아키텍처를 모두 포괄. Model은 데이터 구조에 지나지 않고, Controller에서 UseCase로 전달되고, UseCase에서 Presenter와 View로 돌아감.

예제에서의 `Coordinator`, `ViewModel` 및 `ViewController` 들이 이 계층에 속함.

### Framework & Driver
가장 바깥쪽 계층. DB, 웹, Devices 등.   
안쪽원과 통신하기 위한 접합 코드 외에는 작성할 코드가 없음.

예제에서의 Network 및 `DataMapping`, `PersistentStorages(CoreData, UserDefaults)`들이 이 계층에 속함.

### 원은 4개여야만 하나?
위 그림은 개념 설명을 위한 예시일 뿐, 더 많을수도 적을수도 있음.   
원이 몇개가 되었든, Dependency Rule이 항상 안쪽을 향하기만 하면 됨.

### 예제에서의 계층
![Dependency Direction](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/82d87974-ed74-432d-8ec7-050ff20c5923)

#### Domain 계층
Domain 계층은 `UseCase` + `Entity`가 해당되고, 의존성 역전을 위한 `Data Repository 인터페이스(Protocol)`를 포함.

#### Presentation 계층
Presentation 계층엔 `ViewModel`, `View`가 해당됨.

#### Data 계층
Data 계층에는 Domain 계층에서 정의된 인터페이스를 따르는 `Data Repository 구현체`, `API(Network)` 및 DTO(Decodable), Persistent DB(CoreData) 엔티티를 Domain 모델에 매핑하는 메서드 등이 포함됨.

#### 계층들간의 데이터 흐름
![Dependency Direction2](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/73ddba1b-eb65-4129-8be3-2b09d03bcb5d)
1. `View(UI)`는 `ViewModel(Presenter)`에서 메서드를 호출.
2. `ViewModel`이 `UseCase`를 실행.
3. `UseCase`는 사용자와 `Repository의 데이터`를 결합.
4. 각 `Repository`는 `원격 데이터(Network)`, `Persistent DB 스토리지 소스`나 `캐시`에서 데이터를 반환
5. 정보는 항목을 표시하는 `View(UI)`로 다시 흐름.
