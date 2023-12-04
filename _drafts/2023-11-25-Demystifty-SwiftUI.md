---
title: (WWDC21) Demystify SwiftUI - Identity, Lifetime, 의존성
author: Cody
date: 2023-11-25 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, SwiftUI]
pin: false
comments: true
mermaid: true
math: false
---
[WWDC의 Demystify SwiftUI](https://developer.apple.com/videos/play/wwdc2021/10022)를 정리한 내용입니다.

SwiftUI는 선언적UI로서 고수준의 앱이 원하는 것을 Describe하면 SwiftUI가 이를 구현하는 방법을 정확히 결정.   
대부분의 경우 잘 작동하지만, 예상하지 못한 동작을 하는 순간은 생겨나는데.   
이런 순간에 원하는 결과를 얻기 위해 SwiftUI가 뒤에서 무엇을 하는지 이해하는 것이 도움이 됨.

## SwiftUI가 코드를 볼때 무엇을 보는가
- `Identity`: SwiftUI가 앱의 여러 업데이트에서 element를 동일하거나 별개로 인식하는 방법
- `Lifetime`: SwiftUI가 시간이 지남에 따라 View와 데이터의 존재를 추적하는 방법
- `의존성`: SwiftUI가 인터페이스를 업데이트해야하는 시기와 이유를 이해하는 방법

위 세가지 개념이 SwiftUI가 변경해야할 사항, 방법, 시기를 결정해서 화면에 표시되는 동적UI를 만드는 방법을 알려줌.

---
### Identity (줄여서 ID)
두 View가 같은 것인지에 대한 질문의 핵심은 `Identity`.   

서로 다른 두 화면 state의 두가지 View가 있을 때,
다른 state의 동일한 View인지? 완전히 다른 View인지? 의 답에 따라 두가지 View의 인터페이스가 전환되는 방식이 달라짐.
- 다른 state의 동일한 View라면: SwiftUI가 해당 View를 한 위치에서 다른 위치로 이동하는 등 해당 View에 새 state를 적용.
- 서로 다른 View라면: state 변경 시 페이드인/아웃과 같이 독립적으로 전환

동일한 ID를 공유하는 View는 동일한 개념적 UI element의 서로 다른 상태를 나타냄.
반대로 별개의 UI element를 나타내는 View는 항상 다른 ID를 가짐.

#### Identity의 유형
- `명시적 Identity`
  - Custom 혹은 Data기반 Identity를 사용.
  - 강력하고 유연하지만 누군가가 어딘가에서 모든 이름을 추적해야 함.
  - 명시적 ID의 한 예시는 UIKit에서 사용하는 pointer identity. SwiftUI는 pointer identity를 사용하는 것 대신 명시적으로 id를 사용.
  - List에서 ForEach를 작성할 때, id를 명시적으로 넣는 것이 예시.
    ```swift
    ForEach(..., id: \.someProperty) { ... }
    ```
  - ScrollViewReader에서 ScrollView 내부의 View의 id를 통해 해당 View로 스크롤 애니메이션을 동작하는 예시.
    ```swift
    ScrollViewReader { proxy in
      ScrollView {
        HeaderView(rescueDog)
          .id(headerID)

        Text(rescueDog.backstory)

        Button("Jump to top") {
          withAnimation {
            proxy.scrollTo(headerID)
          }
        }
      }
    }
    ```
  - 위 방식의 장점은 모든 View를 명시적으로 식별할 필요가 없고, Header와 같이 코드의 다른 곳에서 참조해야 하는 View만 식별하도록 할 수 있음.
  - 하지만 Identity가 명시적이지 않다고해서 Identity가 없는 것은 아님. 모든 View는 Identity를 가짐 -> `구조적 Identity`

- `구조적 Identity`
  - View 계층 구조에서의 Type과 위치에 따라서 View를 구분.
  - 사용자가 Identity를 지정하지 않아도 암시적 ID를 생성함.
  - 조건부 논리를 사용하는 경우의 구조적 Identity의 예시
    ```swift
    var body: some View {
      if rescueDogs.isEmpty {
          AdoptionDirectory(selection: $rescueDogs) // true View
      } else {
          DogList(rescueDogs) // false View
      }
    }
    ```
    조건문의 구조는 각 View를 식별하는 명확한 방법을 제공함.
    첫번째 View는 `true`일 때만 보이고, 두번째 View는 `false`일 때만 보여짐. 유사하게 보이더라도 어느 View가 어느 View인지 항상 알 수 있음. 하지만 SwiftUI가 이런 View가 현재 위치에 유지되고, 위치를 바꾸지 않음을 정적으로 보장할 수 있는 경우에만 동작함.

    SwiftUI는 View를 볼 때, Generic type을 봄. 위 경우 `true`, `false` 컨텐츠에 대한 Generic한 `_ConditionalContent` View로 변환됨.
    ```swift
    some View =
      _ConditionalContent<
        AdoptionDiretory, // true View
        DogList // false View
      >
    ```
    이 변환은 Swift의 `Result Builder` 타입 중 하나인 `ViewBuilder`에 의해 제공.
    View 프로토콜은 프로퍼티의 논리문에서 단일 Generic View를 구성하는 `ViewBuilder`의 body 프로퍼티를 암시적으로 래핑함.
    ```swift
    @ViewBuilder // 암시적 래핑
    var body: some View {
      if rescueDogs.isEmpty {
          AdoptionDirectory(selection: $rescueDogs) // true View
      } else {
          DogList(rescueDogs) // false View
      }
    }
    ```
    body 프로퍼티의 `some View` 리턴타입은 정적 composite타입을 나타내는 placeholder로, 코드를 복잡하지 않게 숨겨줌.
    이 Generic타입을 사용해서 SwiftUI는 `true View`가 항상 `AdoptionDiretory`이고, `false View`가 `DogList`임을 보장하여 뒷단에서 암시적이고 안정적인 ID가 할당될 수 있도록 해줌.

#### AnyView가 View구조에 미치는 영향
- 개의 품종을 나타내는 View를 얻기 위한 helper 함수의 예시
  ```swift
  func view(for dog: Dog) -> some View {
    var dogView: AnyView
    if dog.breed == .bulldog {
      dogView = AnyView(BulldogView())
    }
    else if dog.breed == .pomeranian {
      dogView = AnyView(PomeranianView())
    }
    else if dog.breed == .borderCollie {
      dogView = AnyView(BorderCollieView())
      if sheepNearby {
        dogView = AnyView(HStack {
          dogView
          sheepView()
        })
      }
    } else {
      dogView = AnyView(UnknownBreedView())
    }
    return dogView
  }
  ```
  함수의 각 조건부 분기는 서로 다른 종류의 View를 리턴하는데, Swift에서는 전체 함수에 대한 단일 리턴타입이 필요하기 때문에 이를 모두 `AnyView`로 래핑.
  > ⚠️ 불행히도 이는 SwiftUI가 작성한 코드의 조건부 구조를 볼 수 없다는 의미
  ```swift
  some View = AnyView
  ```
  대신 `AnyView`를 함수의 리턴타입으로 간주함. 이는 `AnyView`가 `type-erasing wrapper 타입`으로 불리기 때문. Generic 시그니쳐에서 래핑하는 View 타입을 숨김. 하지만 더 중요한 점은 사람도 읽기 어렵다는 점.

  위 코드를 단순화하고 SwiftUI가 더 많은 구조를 표시할 방법.
  ```swift
    ...
      } else if dog.breed == .borderCollie {
        dogView = AnyView(BorderCollieView())
        if sheepNearby {
          dogView = AnyView(HStack {
            dogView
            sheepView()
          })
        }
      }
    ...
  ```
  현재 `sheepNearby` 분기는 `BorderCollieView`와 함께 `SheepView`를 조건부로 추가하는 것.
  이를 `dogView` 밖에서 HStack을 조건부로 추가하는 대신 HStack 내부에 View를 조건부로 추가하여 단순화시킬 수 있음.
  ```swift
    ...
      } else if dog.breed == .borderCollie {
        dogView = AnyView(HStack {
          BorderCollieView()
          if sheepNearby {
            SheepView()
          }
        })
      }
    ...
  ```
  위처럼 바꾸게 되면 각 분기에서 단일 View만 반환을 하기 때문에 dogView 지역변수가 필요없어지고 각 분기 내부에서 리턴문으로 바꿀 수 있음.
  ```swift
  func view(for dog: Dog) -> some View {
    if dog.breed == .bulldog {
      return AnyView(BulldogView())
    }
    else if dog.breed == .pomeranian {
      return AnyView(PomeranianView())
    }
    else if dog.breed == .borderCollie {
      return AnyView(HStack {
        BorderCollieView()
        if sheepNearby {
          SheepView()
        }
      })
    } else {
      return AnyView(UnknownBreedView())
    }
  }
  ```
  그리고 이전 예시처럼 `return`과 `AnyView`를 제거하면 오류와 경고가 생김.
  ```swift
  func view(for dog: Dog) -> some View { // 오류. 함수는 단일 리턴타입을 요구함
    if dog.breed == .bulldog {
      BulldogView() // 경고
    }
    else if dog.breed == .pomeranian {
      PomeranianView() // 경고
    }
    else if dog.breed == .borderCollie {
      HStack { // 경고
        BorderCollieView()
        if sheepNearby {
          SheepView()
        }
      }
    } else {
      UnknownBreedView() // 경고
    }
  }
  ```
  `body` 프로퍼티는 암시적으로 `ViewBuilder`로 래핑하기 때문에 위 오류를 피할 수 있었음. 위 함수도 `ViewBuilder`를 명시적으로 래핑해주면 동일한 효과를 볼 수 있음!
  ```swift
  @ViewBuilder // 추가!
  func view(for dog: Dog) -> some View {
    if dog.breed == .bulldog {
      BulldogView()
    }
    else if dog.breed == .pomeranian {
      PomeranianView()
    }
    else if dog.breed == .borderCollie {
      HStack {
        BorderCollieView()
        if sheepNearby {
          SheepView()
        }
      }
    } else {
      UnknownBreedView()
    }
  }
  ```
  그리고 result의 타입 시그니쳐를 보면 `_ConditionalContent` 트리를 사용해서 함수의 조건부 로직을 정확히 복제해서 SwiftUI에 View에 대한 더 풍부한 View와 해당 요소의 ID를 제공함.
  ```swift
  some View =
    _ConditionalContent<
      BulldogView,
      PomeranianView
    >,
    _ConditionalContent<
      HStack<
        TupleView<(
          BorderCollieView,
          SheepView?
        )>
      >
    >
  ```
- 추가적인 개선
  - 위 `ViewBuilder` 함수의 최상위는 `dog`의 `breed`(품종)에 따른 분기인데 이를 `switch`문으로 바꾸어도 `ViewBuilder`가 지원됨.
  - `switch`문으로 바꾸면 코드를 이해하는게 더 쉬워짐.
  ```swift
  @ViewBuilder
  func view(for dog: Dog) -> some View {
    switch dog.breed {
    case .bulldog:
      BulldogView()
    case .pomeranian:
      PomeranianView()
    case .borderCollie:
      HStack {
        BorderCollieView()
        if sheepNearby {
          SheepView()
        }
      }
    default:
      UnknownBreedView()
    }
  }
  ```
  - 그리고 실제로 `switch`문은 조건문의 구문적 sugar이기 때문에 result View의 타입 시그니쳐도 변경전과 동일하게 유지됨.
- `AnyView`에 대한 정리
  - `AnyView`는 코드를 읽고 이해하기 여럽게 만들 수 있음. 가능하면 피할 것.
  - `AnyView`는 컴파일러에서 `정적 타입 정보`를 숨기기 때문에 유용한 진단 오류 및 경고가 코드에 표시되지 않을 수 있음.
  - 필요하지 않을 때 `AnyView`를 쓰면 성능이 저하될 수 있음. 가능하면 `AnyView`를 전달하는 대신, `Generic`을 사용해서 `정적 타입 정보`를 보존할 것.

---
### Lifetime
ID를 사용하면 시간이 지남에 따라 다양한 값에 대한 안정적인 element를 정의할 수 있음. 즉, 시간이 지남에 따라 연속성을 도입할 수 있음.   
ID는 시간이 지남에 따라 다양한 value를 단일 entity(하나의 View)로 연결함.
- 데시벨을 표시하는 DecibelView의 예시
  ```swift
  struct DecibelView: View {
    var 강도: Double
    var body: some View {
      // ...
    }
  }
  ```
  상위 View body의 evaluation를 통해 SwiftUI가 이 View에 대한 새로운 Value(25)를 생성
  ```swift
  var body: some View {
    DecibelView(강도: 25)
  }
  ```
  더 높은 소리가 감지되어 더 높은 강도 값으로 호출되고, View에 대한 새 값이 생성됨.
  ```swift
  var body: some View {
    DecibelView(강도: 50)
  }
  ```
  이는 동일한 View 정의에서 생성된 두개의 고유한 값. SwiftUI는 비교를 해보고 View가 변경되었는지 알기 위해 값의 복사본을 유지함. 하지만 그 이후에는 이전 값이 소멸됨.
  > ⚠️ 여기에서 이해해야 할 중요한 점은 `View 값`이 `View ID`와 다르다는 것!   
  `View value` != `View Identity`

  `View 값`은 일시적이므로 Lifetime에 의존해서는 안됨. 우리는 `View의 ID`만 통제할 것.
  View가 처음 생성되면 `Identity` 파트에서의 방식(명시적, 구조적)으로 ID를 할당하고, 시간이 지나면 업데이트에 따라 View에 대한 새로운 값이 생성됨.
  > `DecibelView(강도: 25)` -> `DecibelView(강도: 50)` -> `DecibelView(강도: 42)`

  하지만 SwiftUI 관점에서 이는 동일한 View를 나타냄. View의 ID가 변경되거나, View가 제거되면 lifetime이 끝남.

  ```swift
  struct SoundRecorder: View {
    @State var title = ""
    @StateObject var mic = Microphone()

    var body: some View {
      VStack {
        TextField("Title", text: $title)
        MicView(mic)
      }
    }
  }
  ```
  `State`, `StateObject`는 View의 `ID`에 관한 영구 저장소.   
  View의 ID가 처음 생성될 때 SwiftUI는 초기값으로 `State`, `StateObject`를 위한 메모리에 저장공간을 할당.  
  View의 lifetime동안 SwiftUI는 이 저장공간이 변경되고 View의 body가 다시 evaluate할 때 이 저장공간을 유지함.

- ID의 변화가 상태의 지속성에 미치는 영향
  ```swift
  var body: some View {
    if dayTime {
      SoundRecorder()
    }
    else {
      SoundRecorder()
        .nightTimeStyle()
    }
  }
  ```
  동일한 View이지만 분기가 있는 예시.  
  구조적 ID로 인해, 두 View가 다른 ID를 가지고 있을 것으로 보임.  
  body를 처음 evaluate하고 실제 분기에 들어갈 때, SwiftUI는 초기값(`title=""`)을 사용해서 state에 대한 영구 저장공간을 할당.  
  해당 View의 lifetime에 걸쳐 SwiftUI가 다양한 작업에 의해 변경될 때 state를 유지.  
  하지만 `dayTime`이 바뀌어 분기가 바뀌면? SwiftUI가 state의 초기값부터 false View(`else`)의 새 저장공간을 생성하고, true View(`dayTime`)에 대한 저장공간은 할당이 취소됨. 다시 `dayTime` 분기로 돌아가도 마찬가지.
  > 여기서 중요한 점은, ID가 변경될 때마다, state가 대체된다는 것.
  `State lifetime == View lifetime`  
  이는 View의 본질(=state)가 무엇인지 명확하게 분리하고 이를 ID와 연결할 수 있기 때문에 꼭 알고 있어야 함.

  데이터의 ID를 View에 대한 `명시적 ID`의 형태로 사용하는 데이터 기반의 세트가 있음 -> `ForEach`, `confirmationDialog`, `alert`, `List`, `Table`, `OutlineGroup`.

  ForEach의 예시
  ```swift
  ForEach(0..<5) { offset in
    Text("🐑\(offset)")
  }
  ```
  SwiftUI가 ViewBuilder에 의해 생성된 View를 식별하기 위해 `offset`을 사용함. 일정한 범위(`0..<`)가 View의 lifetime동안 `ID`가 안정적임을 보장. `ForEach`의 범위를 동적으로 하게 되면 경고를 보게 됨.
  ```swift
  ForEach(0..<sheeps) { offset in // Non-constant range: argument must be an integer literal
    Text("🐑\(offset)")
  }
  ```
  
  아래는 동적 데이터로 `ForEach`를 사용하는 예시
  ```swift
  struct cat {
    var tagID: UUID
  }

  ForEach(cats, id: \.tagID) { cat in
    ProfileView(cat)
  }
  ```
  명시적으로 `hashable`한 값을 id에 제공하면, 각 View에 `ID`를 할당시켜줌.
  ```swift
  struct cat: Identifiable {
    var tagID: UUID
    var id: UUID { tagID }
  }

  ForEach(cats) { cat in
    ProfileView(cat)
  }
  ```
  데이터에 View를 그릴 데이터에 안정적인 `ID`를 제공한다는 의미의 `Identifiable` 프로토콜을 채택하면 따로 `id`값을 제공하지 않아도 각 View에 `ID`를 할당시켜줌.


---
### 의존성
의존성은 SwiftUI가 UI를 업데이트하는 방법
```swift
struct DogView: View {
  @Binding var dog: Dog // 의존성
  var treat: Treat      // 의존성
  
  var body: some View {
    Button {
      dog.reward(treat)
    } label {
      PawView()
    }
  }
}
```
`dog`, `treat`의 의존성를 가진 `DogView`.  
의존성이 변경되면 View가 새 body를 생성함. body의 역할은 View의 계층구조를 만드는 것.
![SCR-20231202-shce](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3ae3c509-a32f-458d-8ec8-897d6f2b5b62)
`DogView`의 다이어그램.  
버튼을 누르게 되면 Action으로 `reward()`가 실행되고, `reward()`가 `treat`을 변화시키고 의존성이 변경되었기 때문에 `DogView`가 새로운 body를 그림.  
(SwiftUI의 데이터흐름 세션: Data Essentials in SwiftUI(WWDC20))

![SCR-20231203-gnbm](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/cae15026-7df6-4a71-98db-e21b4af818c1)
`dog`, `treat`에 대한 의존성이 `DogView`만이 아닐수도 있음. 각 자식View들도 각자 의존성을 가지거나 `dog`, `treat`에 의존성을 가질 수 있음.

![SCR-20231203-gowc](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2540a71d-4948-491c-902c-177f28d8e09d)
그래서 실제로는 트리 구조가 아닌, 위와 같은 그래프 구조가 되는데, 이를 `의존성 그래프`라고 부름. 이 구조는 새 body가 필요한 View만 효율적으로 업데이트 수 있도록 하기 때문에 중요함.

![SCR-20231203-gsyr](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/14fd7371-7471-4380-8ef9-e0b388de21f3)
의존성이 변경되면 해당 의존성에 연결된 View만 invalidate되고, 각 View의 body를 호출해서 새로운 body value를 생성시킴.

> - ID는 의존성 그래프의 중추.
> - 모든 View는 명시적ID이나 구조적ID를 갖고, 이 ID가 SwiftUI가 변경사항을 올바른 View로 라우팅하고 UI를 효율적으로 업데이트하는 방법.
> - 의존성의 여러 종류: `@Binding`, `@Environment`, `@State`, `@StateObject`, `@ObservedObject`

#### 명시적ID의 안정성의 중요성
```swift
enum Animal { case dog, cat }

struct Pet: Indentifiable {
  var name: String
  var kind: Animal
  var id: UUID { UUID() } // Not Stable
}

struct FavoritePets: View {
  var pets: [Pet]
  var body: some View {
    List {
      ForEach(pets) {
        PetView($0)
      }
    }
  }
}
```
위 코드에서 Pet은 ID를 가지고 있지만, 새로운 Pet이 추가될 때마다 화면이 깜빡거리는 버그가 존재.
위처럼 UUID를 생성해서 할당하면 데이터가 변경될 때마다 새로운 ID를 가지게 됨.  
UUID대신 배열 인덱스를 사용하는 것도 마찬가지로 안정적인 ID가 아님.

```swift
struct FavoritePets: View {
  var pets: [Pet]
  var body: some View {
    List {
      ForEach(pets, id: \.databaseID) { // stable ID
        PetView($0)
      }
    }
  }
}
```
안정적이기 위해서는 데이터베이스에서 가져온 ID나, Pet의 안정적인 프로퍼티에서 파생된 안정적인 ID를 사용해야 함.  
좋은 식별자의 속성 = `고유성`. 각 ID는 단일 View에 매핑되어야 하고, 이를 통해 애니메이션이 자연스럽고 성능도 원활하고, 계층구조의 의존성이 가장 효율적인 형태로 반영될 수 있음.  

```swift
struct TreatJar: View {
  var treats: [Treat]

  var body: some View {
    ScrollView {
      LazyVGrid(...) {
        ForEach(treats, id: \.name) {
          TreatCell($0)
        }
      }
    }
  }
}
```
위 코드에서는 ID를 name이 아닌 고유한 ID로 변경되어야 함
```swift
        ForEach(treats, id: \.serialNumber) {
```
> - 무작위 ID를 쓸 떄는 주의가 필요.
> - ID는 시간이 지나도 변경되어서는 안됨. 새로운 ID는 새로운 lifetime을 가진 새로운 항목을 나타냄.
> - ID는 고유해야 함. 여러 View가 ID를 공유할 수 없음.

#### 구조적 ID
```swift
ForEach(treats, id: \.sericalNumber) { treat in
  TreatCell(treat)
    .modifier(ExpirationModifier(date: treat.expiryDate))
}
// 현재날짜와 비교해서 View를 어둡게 해주는 ViewModifier
struct ExpirationModifier: ViewModifier {
  var data: Date
  func body(content: Content) -> some View {
    if data < .now {
      content.opacity(0.3)
    } else {
      content
    }
  }
}
```
위 코드에는 문제가 있음. 조건이 바뀌고 date가 만료가 되면, `if data < .now`부분에서 분기가 생겨 새로운 구조적ID를 갖게 됨.
문제를 해결하는 방법은 해당 분기를 없애는 것. 아래처럼 수정하면 단일 ID를 갖게됨.
```swift
      content.opacity(data < .now ? 0.3 : 1.0)
```
그리고 `opacity(1.0)`이 되었을 때는 불투명도가 `1.0`이 되어서 아무런 효과가 없어서 렌더링 결과에 아무 영향이 없어서 `비활성 수정자`가 되어 더욱 비용이 절감됨.
> - 구조적 ID를 불필요하게 발생시키면 성능저하, state손실, 의도치 않은 애니메이션이 생길 수 있음
> - 분기를 만들 때 다른 View가 생기는지, 동일한 View의 다른 state가 되는지 고민할 것.



## 요약
- SwiftUI가 View를 그리는데 주요 관점은 Identity, Lifetime, 의존성.
- Identity
  - Identity에는 명시적 ID, 구조적 ID가 있음.
  - ID는 시간이 지남에 따라 다양한 value를 단일 entity(하나의 View)로 연결함.
- Lifetime
  - View의 ID가 변경되거나, View가 제거되면 lifetime이 끝남.
  - View value는 일시적인 것이므로 lifetime에 의존하면 안됨.
  - View의 ID를 제어할 수 있고, ID를 사용해서 state의 lifetime의 스코프를 명확히 지정할 수 있음.
  - SwiftUI는 Identifiable 프로토콜을 최대한 활용하기 때문에 데이터에 대한 안정적인 ID를 선택할 것. 안정적이지 않은 ID는 View의 lifetime을 단축시킴.
- 의존성
  - SwiftUI는 의존성에 따른 그래프를 그리고, 그에 따른 새로 그려야할 View만 효율적으로 그림.
  - ID를 안정적으로 유지해야 View가 안정적일 수 있음.
  - View에 대한 분기문을 작성할 때, 구조적 ID가 잘 유지되는지 고민해야 함.