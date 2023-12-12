---
title: (SwiftUI) Property Wrapper (3) @EnvironmentObject, @Environment
author: Cody
date: 2022-09-02 00:34:00 +0800
categories: [iOS Swift]
tags: [Swift, SwiftUI, iOS13]
pin: false
mermaid: true
---

### @EnvironmentObject
SwiftUI에서는 환경값이 바뀌었을 때,  
해당값에 따라서 뷰를 그릴 수 있게 해주는 `@EnvironmentObject`라는 Property Wrapper를 제공합니다.

`@EnvironmentObject` Property Wrapper는 이전글에서 다룬 `ObservableObject` 프로토콜을 따르는 프로퍼티에만 사용할 수 있습니다.

이 값은 부모뷰로부터 주입된 값으로, 자식뷰들에서는 어디에서든 사용할 수 있습니다.

개발을 하다보면 A뷰->B뷰->C뷰->... 로 계속 같은 데이터를 넘겨주고,

데이터가 변경될 때 이에 따라 뷰를 그려야 할 상황이 생기는데,

이런 경우 매우 유용하게 작성할 수 있게 해주는 방법입니다.

간단한 예제로 이전글에서 썼던 `Counter`, `CounterModelView` 예제를 이어서 작성해보겠습니다.

(이전글: [`[Swift, SwiftUI] Property Wrapper (2) @State/@Binding, @Published/@ObservedObject`](https://www.notion.so/Swift-SwiftUI-Property-Wrapper-2-State-Binding-Published-ObservedObject-90464fb942c24d389c57ddd44fc93f88?pvs=21) )

`CounterModelView`를 `@EnvironmentObject`로 가지고 있는 `CountResultView`를 작성하고,

(이 때, `@EnviromentObject`를 가진 `View`의 `PreviewProvider`에서 `environmentObject`를 넘겨주어야 `Preview`를 문제없이 그려줍니다)

```swift
struct CountResultView: View {
    @EnvironmentObject var countModel: CounterViewModel
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Your Count: \(countModel.count)")
                .font(.title)
            Button {
                countModel.count = 0
            } label: {
                Text("초기화")
            }

        }
    }
}

struct CountResultView_Previews: PreviewProvider {
    static var previews: some View {
    	// PreviewProvider에서 environmentObject를 넘겨주어야 Preview를 그려줍니다)
        CountResultView()
            .environmentObject(CounterViewModel())
    }
}
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/cf73b776-b13b-4dd9-9e15-e2ccad54fed6)

`Counter` 뷰에서 `CountResultView`를 생성할 때,

`.environmentObject()` Modifier를 통해 `viewModel`을 주입시켜줍니다.

```swift
struct Counter: View {
    @ObservedObject var viewModel = CounterViewModel()
    
    var body: some View {
        VStack(spacing: 30) {
            HStack(spacing: 20) {
                Button {
                    viewModel.count = viewModel.count-1 >= 0 ? viewModel.count-1 : 0
                } label: {
                    Image(systemName: "minus.square")
                        .font(.system(size: 50))
                }
                
                Text("\(viewModel.count)")
                    .font(.system(size: 50))
                
                Button {
                    viewModel.count += 1
                } label: {
                    Image(systemName: "plus.square")
                        .font(.system(size: 50))
                }
            }
            
            NavigationLink {
                CountResultView()
                    .environmentObject(viewModel) // EnvironmentObject를 주입
            } label: {
                Text("CountEnd")
                    .font(.title)
                    .foregroundColor(.blue)
            }

        }
    }
}
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2889b5e5-7866-4ab1-9a9b-4c8b53ac2802)

그리고 `CountResultView`를 호출하면,

넘겨준 `viewModel`의 `count`값이 뷰에 잘 반영된 것을 볼 수 있고,

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e0398f42-2792-4afb-ab99-b65979650ec3)

`CountResultView`에서 초기화를 눌렀을 때,

`CountResultView`와 `Counter`뷰 모두 잘 반영이 된 것을 확인할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/aba0b435-6142-4dfd-8d06-fb0b3ecb70db)

### @Environment

위의 `@EnvironmentObject`와 유사하게 뷰환경에 저장된 값을 읽어올 수 있는 `@Envirnment` Property Wrapper가 있습니다.

시스템에서 제공해주는 지역(`\.locale`), 다크모드(`\.colorScheme`), 스케일(`\.displayScale`) 등과 같은 값들이 이런 환경값들입니다.

이 `@Environment` Property Wrapper는 `읽기전용`으로,

`EnvironmentValues`의 `KeyPath`(`\.locale`,` `\.colorScheme`, ...)을 통해서 읽을 값을 나타냅니다.

`@EnvironmentObject`처럼 값이 변경되었을 때, `뷰의 일부를 업데이트` 시킬 수 있고,

뷰의 `.environment(_:_:)` Modifier를 통해 `오버라이드`하거나 `커스텀 `EnvironmentValue`를 설정`할 수 있습니다.

시스템에서 제공해주는 `ColorScheme` `@Environment`의 예시입니다.

앞에서 작성한 카운터가 또 수고해주었습니다.

`@Environment` 값 중에 `\.colorScheme`값을 `colorScheme` 프로퍼티`를 통해 읽어옵니다.

```swift
struct Counter: View {
    @ObservedObject var viewModel = CounterViewModel()
    @Environment(\.colorScheme) var colorScheme: ColorScheme // ColorScheme Environment 읽어옴
    
    var body: some View {
        VStack(spacing: 30) {
            HStack(spacing: 20) {
                Button {
                    viewModel.count = viewModel.count-1 >= 0 ? viewModel.count-1 : 0
                } label: {
                    Image(systemName: "minus.square")
                        .font(.system(size: 50))
                }
                
                Text("\(viewModel.count)")
                    .font(.system(size: 50))
                    .foregroundColor(colorScheme == .light ? .brown : .green) // colorScheme에 따른 뷰 업데이트
                
                Button {
                    viewModel.count += 1
                } label: {
                    Image(systemName: "plus.square")
                        .font(.system(size: 50))
                }
            }
        }
    }
}
```

`ColorScheme`의 `.light`모드, `.dark`모드에 따라서 폰트 색상이 잘 변경되는 것을 확인할 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/20ef068b-9158-41b4-b036-4a6337b0327d)

### 커스텀 EnvironmentValue를 설정

먼저 `EnvironmentKey`를 설정해주어야 합니다.

`EnvironmentKey` 프로토콜을 따르는 커스텀 구조체를 만들고, `defaultValue`를 필수적으로 설정해주어야 합니다.

`KeyPath`로 읽어올 때 기본적으로 이 `defaultValue`를 읽어오게 됩니다.

그리고 `EnvironmentValues`를 `extension`하여 커스텀 `EnvironmentValue`를 읽어올 프로퍼티를 설정해줍니다.

```swift
//  MyEnviroment.swift
import SwiftUI

// defaultValue 설정
private struct MyEnvironmentKey: EnvironmentKey {
    static let defaultValue: String = "Default value"
}

// customValue를 읽어올 프로퍼티 설정
extension EnvironmentValues {
    var myCustomValue: String {
        get { self[MyEnvironmentKey.self] }
        set { self[MyEnvironmentKey.self] = newValue }
    }
}
```

그리고 사용할 때는 값을 주입할 뷰의 `.environment(_:_:)`
 Modifier로 커스텀 `EnvironmentValue` 를 주입하고,

```swift
Counter()
    .environment(\.myCustomValue, "Custom String")
```

사용할 뷰에서는 아래와 같이 값을 읽어와서 사용하면 됩니다.

```swift
@Environment(\.myCustomValue) var customValue: String
```