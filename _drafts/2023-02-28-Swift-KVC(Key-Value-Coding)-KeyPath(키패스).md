---
title: (Swift) KVC(Key-Value Coding), KeyPath(키패스)
author: Cody
date: 2023-02-28 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, Swift5.2, KVC, KeyPath]
pin: false
mermaid: true
---
## **Key-Value Coding (KVC)**

**`Key-Value Coding`**은 객체의 프로퍼티를 **Key-value** 쌍으로 접근할 수 있도록 해주는 **Objective-C 문법**입니다.

**`KVC`**를 사용하면 속성의 이름을 **문자열로 참조하여 런타임에 동적으로 객체의 프로퍼티 값을 설정하거나 읽을 수** 있습니다.

**Swift**에서도 사용할 수 있지만, **Objective-C** 런타임에 의존하기 때문에 프로퍼티 선언 시 앞에 **`@objc`** 어노테이션을 붙여줘야 하며,

**`NSObject`**의 서브클래스에서만 사용이 가능합니다.

```swift
class Person: NSObject { // NSObject 서브클래스
        @objc var name: String? // @objc 어노테이션
}
```

위와 같이 선언된 클래스의 프로퍼티는 아래처럼 **KVC**로 접근이 가능합니다.

```swift
let person = Person()
person.value(forKey: "name") // nil
person.name = "Cody"
person.value(forKey: "name") // Cody
```

만약 없는 프로퍼티명으로 접근하려 하면 런타임 에러가 발생하므로 주의해야 합니다.

```swift
person.value(forKey: "age") // runtime error(error: Execution was interrupted, reason: signal SIGABRT.)
```

## **KeyPath**

**`KeyPath`**는 값에 대한 참조가 아닌, **프로퍼티에 대한 참조를 나타내는 타입**입니다.

위의 **Person**의 **name** 프로퍼티를 참조하는 예시입니다.

```swift
let personNameKeyPath = \Person.name
print(personNameKeyPath) // Swift.ReferenceWritableKeyPath<__lldb_expr_47.Person, Swift.Optional<Swift.String>>
```

**person**인스턴스에서 **keyPath**를 통해 값을 받아오는 방법입니다.

**subscript(keyPath:)**를 통해서 값에 접근합니다.

```swift
let personName = person[keyPath: personNameKeyPath]
print(personName) // Optional("Cody")
```

**Person** 클래스에 친구 정보를 넣도록 수정하겠습니다.

```swift
class Person: NSObject {
    @objc var name: String

    @objc var friend1: Person?
    @objc var friend2: Person?

    init(name: String) {
        self.name = name
    }
}
```

그리고 **Cody**라는 **name**의 **Person**인스턴스를 만들고, **Charlie**와 **Choi**라는 친구를 넣어보겠습니다.

```swift
let charlie = Person(name: "Charlie")
let choi = Person(name: "Choi")

let cody = Person(name: "Cody")
cody.friend1 = charlie
cody.friend2 = choi
```

함수를 통해 **keyPath**를 사용하지 않고 **cody**의 **friend1**, **friend2**의 정보를 가져오려면 아래와 같이 작성해야 합니다.

```swift
func getFriend1(person: Person) -> Person? {
    return person.friend1
}

func getFriend2(person: Person) -> Person? {
    return person.friend2
}

getFriend1(person: cody)?.name // Charlie
getFriend2(person: cody)?.name // Choi
```

**keyPath**를 사용하면 위 두 함수를 하나로 작성할 수 있습니다.

```swift
func getFriend(person: Person, keyPath: KeyPath<Person, Person?>) -> Person? {
    return person[keyPath: keyPath]
}

getFriend(person: cody, keyPath: \Person.friend1)?.name // Charie
getFriend(person: cody, keyPath: \.friend2)?.name // Choi
```

위에 예시처럼

**`Person`**의 **`friend1`**의 **keyPath**는 **`\Person.friend1`**으로 쓸 수도 있고,

**`getFriend()`**함수 파라메터 정의에서 **KeyPath**에 **`Person`**타입을 받는다고 명시를 했기 때문에 **`Person`**을 생략하고 `**\.friend1**` 으로 쓸 수도 있습니다.

(Swift5.2 이상) 컬렉션타입에서 고차함수에서 **keyPath**를 활용할 수 있습니다.

```swift
let persons = [cody, charlie, choi]

// 키패스를 사용하지 않고
let names = persons.map { $0.name } // ["Cody", "Charlie", "Choi"]

// 키패스를 사용해서
let namesUsingKeyPath = persons.map(\.name) // ["Cody", "Charlie", "Choi"]
```

**KVC**를 위해 **`@objc`** 어노테이션을 붙인 프로퍼티에 추가로 **dynamic** 어노테이션을 붙여주면 **KVO(Key-Value Observing)**을 사용할 수 있습니다. (**KVO**는 [이전글](https://www.notion.so/Swift-KVO-Key-Value-Observing-acb10a09040e46dcb4f585b80ebb7b8f?pvs=21) 링크를 참조)