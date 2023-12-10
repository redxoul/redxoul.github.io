---
title: UIViewRepresentable, UIViewControllerRepresentable 프로토콜(feat. PreviewProvider, SnapKit)
author: Cody
date: 2022-08-15 00:34:00 +0800
categories: [iOS Swift]
tags: [Swift, SwiftUI, iOS13]
pin: false
mermaid: true
---
SwiftUI가 최근 버전에 와서는 조금 쓸만해졌지만,
쓸만해진 SwiftUI를 쓰려면 Deployment target version이 좀 높아져야 하기도 하고,
타겟 버전을 높였다고 해도 기존 프로젝트에서 많이 사용 중인 UIKit을 모두 SwiftUI로 변경하기엔 어려움이 있기도 합니다.  
그래서 프로젝트들이 아직까지는 UIKit을 많이 사용 중입니다.

Apple에서는 SwiftUI를 밀고 있으면서도
UIKit을 SwiftUI에서 재사용하거나,
최근(WWDC2022)에는 UIKit(UITableview)에 SwiftUI를 녹여서 쓸 수 있는 방법을 제시하고 있습니다.

그중에서 `SwiftUI에서 UIKit을 사용할 수 있도록 제공하는 방법`이
`UIViewRepresentable`, `UIViewControllerRepresentable` 프로토콜입니다.
`UIViewRepresentable` 프로토콜은 `UIView`를
`UIViewControllerRepresentable` 프로토콜은 `UIViewController`를 SwiftUI에서 사용할 수 있도록 해줍니다.
두 프로토콜들은 iOS 13.0부터 사용 가능합니다.

(각각의 Define을 스크린샷으로 담기엔 문서화 주석이 많아서)
프로토콜에서 꼭 구현해야 할 요소들만 정리를 해보자면 아래와 같습니다.
```swift
// UIViewControllerRepresentable 프로토콜
public protocol UIViewControllerRepresentable : View where Self.Body == Never {
    associatedtype UIViewControllerType : UIViewController
    func makeUIViewController(context: Self.Context) -> Self.UIViewControllerType
    func updateUIViewController(_ uiViewController: Self.UIViewControllerType, context: Self.Context)
}
```

```swift
// UIViewRepresentable 프로토콜
public protocol UIViewRepresentable : View where Self.Body == Never {
    associatedtype UIViewType : UIView
    func makeUIView(context: Self.Context) -> Self.UIViewType
    func updateUIView(_ uiView: Self.UIViewType, context: Self.Context)
}
```

사용 예시를 들어보겠습니다.

`SomeViewController`이라는 `UIViewController`를 작성했습니다.

`SnapKit`과 `Then` 라이브러리를 사용하여 작성했습니다.

(본 포스팅에 이어서 `SnapKit` 포스팅도 예정에 있습니다)

(Then 라이브러리 포스팅: [Then 라이브러리](https://swiftycody.github.io/posts/Then-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC/) )

```swift
class SomeViewController: UIViewController {
    let someView = UIView().then {
        $0.backgroundColor = .systemPink
        $0.layer.cornerRadius = 10
        $0.layer.borderWidth = 10
        $0.layer.borderColor = UIColor.purple.cgColor
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.drawSubviews()
    }
    
    func drawSubviews() {
        self.view.addSubview(someView)
        
        someView.snp.makeConstraints {
            $0.width.height.equalTo(100)
            $0.center.equalToSuperview()
        }
    }
}
```

그리고 `SomeViewController`를 위한 `UIViewControllerRepresentable` 프로토콜을 구현해보면 아래와 같습니다.

`updateUIViewController(_:, context:)`는 `SomeViewController`를 사용하는 SwiftUI에서 업데이트가 일어날 때 호출되는 함수로 `SomeViewController`가 변화에 대응할 수 있도록 해줍니다.

```swift
struct SomeViewControllerContainer: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> UIViewController {
        let someViewController = SomeViewController()
        
        return someViewController
    }
    
    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
        
    }
    
    typealias UIViewControllerType = UIViewController
}
```

이제 SwiftUI에서 `SomeViewController`를 사용할 준비가 끝났습니다.

SwiftUI를 빠르게 확인할 수 있는 방법은 바로 직전에 정리했던 포스팅에서 소개한 `PreviewProvider`입니다.

([[SwiftUI] PreviewProvider 프로토콜, Preview (feat. Xcode14(Beta))](https://swiftycody.github.io/posts/SwiftUI-PreviewProvider-%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C-Preview/) )

`PreviewProvider`에서 `SomeViewControllerContainer`를 작성해보면 `Preview`에서 바로 확인해 볼 수가 있습니다.

```swift
struct SomeViewController_Previews: PreviewProvider {
    static var previews: some View {
        SomeViewControllerContainer().edgesIgnoringSafeArea(.all)
    }
}
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/36693c3d-e5ff-453a-900b-90c4be794ab3)

(UIViewRepresentable은 UIViewControllerRepresentable과 유사하기 때문에 생략합니다)

여기까지가 UIViewRepresentable, UIViewControllerRepresentable 프로토콜에 대한 포스팅이었습니다.

이렇게 보면 위의 `두 프로토콜들과 SnapKit 라이브러리, 그리고 SwiftUI의 PreviewProvider 프로토콜과의 조합`이 굉장히 좋다는 것을 알 수 있습니다.

기존의 Autolayout만 썼을 때의 단점은 '스토리보드만으로 Autolayout을 다 커버할 수 없다'.

그래서 결국 코드로 Autolayout을 추가로 작성을 해주어야 하는데?

그러면 '화면구성 및 애니메이션 등에 대한 구현이 스토리보드와 코드로 나뉘어져 코드의 가독성이 매우 떨어지게' 됩니다.

코드로만 Autolayout을 작성하게 되면 'Constraint들이 엄청나게 많아지게 되고 직관적이지가 않게 되는 문제'가 발생하는데.

`많고 복잡해지는 Constraint 작성을 간단하게 만들어주는 도구`가 `SnapKit 라이브러리`이고,

`코드로 작성한 Autolayout을 직관적으로 보여주게 해주는 도구`가 `PreviewProvider 프로토콜`이 되어줍니다.

조만간 이 SnapKit에 대한 포스팅을 올려보도록 하겠습니다.