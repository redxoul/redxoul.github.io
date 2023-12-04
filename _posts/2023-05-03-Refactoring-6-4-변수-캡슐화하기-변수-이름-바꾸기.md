---
title: (Refactoring) 6-4 변수 캡슐화하기, 변수 이름 바꾸기
author: Cody
date: 2023-05-03 00:34:00 +0800
categories: [Refactoring]
tags: [Swift, Refactoring]
pin: false
mermaid: true
---
## **변수 캡슐화하기**

데이터는 참조하는 모든 부분을 한 번에 바꿔야 코드가 제대로 동작하는 까닭에 함수보다 까다롭습니다.

유효범위가 넓어질수록 다루기 어려워집니다. 전역 데이터가 골칫거리인 이유.

그래서 접근 범위가 넓은 데이터를 옮길 때는 먼저 그 데이터로의 접근을 독점하는 식의 **'캡슐화'**하는 것이 좋을 때가 많습니다.

캡슐화를 하면 데이터를 변경하고 사용하는 코드를 감시할 수 있는 확실한 통로가 되어주기 때문에 데이터 변경 전 검증이나, 변경 후 추가로직을 쉽게 끼워 넣을 수 있습니다.

'불변 데이터'는 가변 데이터보다 캡슐화할 이유가 적습니다. 데이터가 변경될 일이 없어서 갱신 전 검증 같은 추가 로직이 자리할 공간을 마련할 필요가 없기 때문이죠. 그래서 원본 데이터를 참조하는 코드를 변경할 필요도 없고, 데이터를 변형시키는 코드를 걱정할 일도 없습니다. 불변성은 강력한 '방부제'인 셈.

(1) 변수로의 접근과 갱신을 전담하는 캡슐화 함수들 작성

(2) 정적 검사 수행

(3) 변수를 직접 참조하던 부분을 모두 적절한 캡슐화 함수 호출로 변경. 바꿀 때마다 테스트.

(4) 변수의 접근 범위를 제한

(5) 테스트

(6) 변수 값이 레코드라면 '레코드 캡슐화하기(chapter7)'를 적용할지 고려

(책과는 조금 다른 내용일 수 있음. 불변의 데이터를 복사하여 사용해야한다는 내용에 초점)

**`defaultOwner`**를 **`private`** 접근자로 제한하고, **`getDefaultOwner()`**로만 접근하도록 만든 예시입니다.

```swift
class Owner {
    var firstName: String
    var lastName: String

    init(firstName: String, lastName: String) {
        self.firstName = firstName
        self.lastName = lastName
    }
}
```

```swift
private var defaultOwner = Owner(firstName: "마틴", lastName: "파울러")

func getDefaultOwner() -> Owner {
    return defaultOwner
}
```

위 코드의 문제점은 **`getDefaultOwner()`**로 받아온 **`defaultOwner`**을 참조값으로 반환하기 때문에

아래와 같이 다른 곳에서 사용할 때 **원본값이 변경**된다는 점에 있습니다.

변경될 수 있는 값을 여러 곳에 전달하는 것은 예상치 못한 결과를 만들어낼 수 있습니다.

```swift
let use = {
    var owner = getDefaultOwner()
    owner.firstName = "엘리"

    print(owner.firstName, owner.lastName) // 엘리 파울러
        print(getDefaultOwner().firstName, getDefaultOwner().lastName) // 엘리 파울러(원본이 변경)
}

use()
```

사실 위와 같은 결과가 생긴 것은 **Owner**가 **class**로 만들어졌기 때문입니다.

**Owner**를 **struct**로 작성해주면 **defaultOwner**는 값 타입이 되기 때문에 복사된 값이 전달되어 위와 같은 문제를 막을 수 있습니다.

```swift
struct Owner { // struct로 작성
        var firstName: String
    var lastName: String

    init(firstName: String, lastName: String) {
        self.firstName = firstName
        self.lastName = lastName
    }
}
```

```swift
let use = {
    var owner = getDefaultOwner()
    owner.firstName = "엘리"

    print(owner.firstName, owner.lastName) // 엘리 파울러
        print(getDefaultOwner().firstName, getDefaultOwner().lastName) // 마틴 파울러
}

use()
```

보통 저런 **데이터 모델**의 경우 **struct**를 사용하는 이유이기도 합니다. **상속**이나 **Objective-C와 호환** 등과 같은 이유가 있지 않으면 **struct**를 사용. 초기화 시에만 값을 설정하고난 후 사용할 때 값의 무분별한 변경을 방지하려면 let으로 설정하여 데이터를 안전하게 제한을 둘 수 있습니다.

## **변수 이름 바꾸기**

변수 이름짓기는 프로그래밍의 핵심.

아래는 극단적인 예시.

```swift
let a = height * width
```

a가 어떤 역할인지 명확치 않아 다른 라인에서 a라는 변수명을 보고 다시 정의부분을 확인해야만 하고, 보통은 너비*높이 순서로 표현.

```swift
let area = width * height
```

아래는 너무 축약된 이름의 예시.

```swift
let cpyNm = "애플"
```

cpy? copy? 너무 축약하면 의미를 오해할 수가 있음.

```swift
let companyName = "애플"
```

아래는 유추를 해야만 하는 네이밍.

```swift
var tpHd = "제목없음"
let result = "<h1>\(tpHd)</h1>"
```

구체적인 네이밍으로 변경

```swift
var pageTitle = "제목없음"
let result = "<h1>\(pageTitle)</h1>"
```