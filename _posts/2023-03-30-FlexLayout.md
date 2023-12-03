---
title: FlexLayout에 대한 정리
author: Cody
date: 2023-03-30 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Swift, FlexLayout, PinLayout]
pin: false
mermaid: true
---

[https://github.com/layoutBox/FlexLayout](https://github.com/layoutBox/FlexLayout)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/579ec85a-d77b-444f-8a9b-c0d929875682){: width="30%" height="30%"}


**FlexLayout**은 Facebook에서 만든 Flexbox Layout engine인 Yoga의 기능을

**Swift**에 맞게 구현한 고성능 레이아웃 프레임워크입니다.

아래는 **FlexLayout**이 **UIStackView**, **AutoLayout**, **Manual Layout**들과 비교해 얼마나 성능이 뛰어난지 비교한 그래프입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d299bdf9-ec21-492c-8cc3-2dff45ff9b59)

## **FlexLayout의 Setting**

**FlexLayout**은 **iOS 8.0**부터 지원하여 많은 프로젝트에서 도입하기 좋은 조건을 갖고 있습니다.

**CocoaPods**, **Carthage**, **Swift Package Manager**을 모두 지원합니다.

단 **SPM**으로 Dependency 설정 후 빌드시 아래와 같은 에러가 발생하는데요,

```
'YGEnums.h' file not found
```

프로젝트의 **Target -> Build Setting -> Apple Clang-Preprocessing -> Preprocessor Macros(`GCC_PREPROCESSOR_DEFINITIONS`**) 항목에 아래 구문을 넣어줍니다.

```
FLEXLAYOUT_SWIFT_PACKAGE=1
```

공식 문서를 보면 **PinLayout**이라는 프레임워크와 함께 사용할 수 있다고 설명합니다.

둘 다 layoutBox에서 만든 레이아웃 프레임워크로

**`PinLayout`**은 **AutoLayout**을 좀 더 편한 **`.pin`** 구문으로 쉽게 설정할 수 있게 해주는 역할이고(**SnapKit**과 유사합니다),

**`FlexLayout`**은 **UIKit**으로 **SwiftUI**처럼 작성할 수 있게 해주는 느낌입니다. 그래서 소스코드가 시각적으로 봤을 때 구조를 파악하기가 매우 쉬워집니다.

공식 문서의 예시도 그렇고, 다른 곳에서 작성한 설명글을 봤을 때

주로 최상단의 컨테이너 뷰를 메인뷰의 영역에 고정하는 역할로 **PinLayout**을 사용하는 듯합니다.

**결론** = **PinLayout**도 함께 **Dependency** 설정을 해주고, 함께 **import 해서** 사용합니다.

## **FlexLayout 작성의 기본 틀**

FlexLayout 작성의 기본 틀은 아래와 같습니다.

```swift
import Foundation
import FlexLayout
import PinLayout

class FlexLayoutSample: UIView {
    private let rootContainer = UIView() // 최상단 컨테이너

    override init(frame: CGRect) {
        super.init(frame: frame)

                // FlexLayout 작성
        setFlexLayout()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func layoutSubviews() {
        super.layoutSubviews()

        layout()
    }

    private func layout() {
        rootContainer.pin.all(self.pin.safeArea)
        rootContainer.flex.layout()
    }

    private func setFlexLayout() {
                // FlexLayout 작성

    }
}
```

(1) **FlexLayout**을 UIView(혹은 UIViewController) 초기화 시 작성해 주기

(2) 최상단 컨테이너(rootContainer)를 **pin**으로 메인뷰에 고정시키고, **`.flex.layout()`**을 호출시켜 주는 메서드(`layout()`) 작성

→  여기서 **`.flex.layout()`**은 **FlexLayout**에서의 **`layoutSubviews()`**와 유사한 역할의 메서드입니다.

**FlexLayout**에서는 레이아웃을 다시 그리기 원하는 특정 뷰에서 **`.flex.layout()`**을 호출하여 화면 일부만 다시 그리도록 구현할 수 있습니다.

(3) **layout()** 메서드를 레이아웃을 **refresh 할** 시점, 즉 **`layoutSubviews()`**에서 호출.

→ **UIViewController**였다면 **`viewDidLayoutSubviews()`**, **UITableViewCell**이었다면 **`sizeThatFits(_:)`**에서도 호출

(+) 여기에 **SnapKit** 작성때와 마찬가지로, **SwiftUI**의 **Preview**를 함께 사용하면 더욱 편하게 작성을 할 수 있습니다.

([**UIViewRepresentable, UIViewControllerRepresentable 프로토콜(feat. PreviewProvider, SnapKit)**](https://www.notion.so/UIViewRepresentable-UIViewControllerRepresentable-feat-PreviewProvider-SnapKit-363863d6a02a4d3591f13000259ad4dd?pvs=21)  글 참고)

## **FlexLayout의 작성**

**FlexLayout**을 작성할 때 필요한 API 및 예제는 공식문서에 자세히 설명이 잘 되어있습니다.

여기서는 예시를 들어 어떤 식으로 작성하는지, 어떤 API를 사용했는지에 대한 설명 위주로 해보려 합니다.

아래와 같은 뷰를 그리는 예시입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/fc9916c8-b4fc-4195-a55f-87f3b5220849)

바깥쪽에서부터 한 단계씩 그려나갑니다.

우선 가장 바깥쪽, 최상단 컨테이너뷰에 패딩이 들어간 것이 보입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4b4f84bd-b5fb-4ce5-a0f9-003a490c0dd7)

```swift
private func setFlexLayout() {

    addSubview(rootContainer)

    rootContainer.flex.padding(16) // 패딩을 추가
}
```

**.padding()**

패딩을 줄 수 있습니다. **`float`**으로 설정할 수도 있고 **`%`**로도 설정할 수 있습니다.

**`.paddingTop`**, **`.paddingLeft`**, **`.paddingBottom`**, **`.paddingRight`**, **`.paddingHorizontal`**, `.**paddingVertical**` 등 다양한 SugarAPI들을 제공합니다.

그 안에 크게 2개의 뷰로 나뉩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0968d5aa-7dde-443d-901c-71c0be453aaa)

```swift
private func setFlexLayout() {

    addSubview(rootContainer)

    rootContainer.flex.padding(16).direction(.column).justifyContent(.spaceBetween).define { flex in

        flex.addItem(topContainer).height(60)

        flex.addItem(bottomContainer)
    }
}
```

**FlexLayout**에서 여러 속성을 줄 때는 **`.flex`** 뒤로 계속 이어서 chaining으로 작성할 수 있습니다.

**.define { flex in }**

**서브뷰를 정의**할 때는 **`.define { flex in }`** 으로 작성해 줍니다.

**.addItem()**

**서브뷰를 추가**할 때는 **define** 클로저 내에서 **`.addItem()`**으로 할 수 있습니다.

(**`addItem()`**으로 서브뷰를 추가하면 **`addSubview()`**를 따로 작성할 필요는 없습니다)

**.direction()**

**서브뷰의 방향**을 지정해 줄 때는 **`.direction()`**으로 해줍니다. 세로방향은 **`.column`** 가로방향은 **`.row`** 입니다. 기본값은 **`.column`** 이기 때문에 세로방향의 컨테이너의 경우 **`.direction(.column)`**을 생략해 줄 수 있습니다.

**.justifyContent()**

**서브뷰들의 간격과 배치**를 어떻게 할 것인지를 지정할 수 있습니다. **`.start`**, **`.end`**, **`.center`**, **`.spaceBetween`**, **`.spaceAround`**, **`.spaceEvenly`** 값을 넣어줄 수 있습니다(각 값은 공식문서를 참조). 이 뷰에서는 **`.spaceBetween`**을 넣어서 서브뷰들을 테두리에 붙이면서 spacing를 주도록 했습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7e31be1f-cea4-4d3e-9380-b79a8a82469a)

.justifyContent(.spaceBetween)

그리고 **`addItem()`**에 아무 뷰도 넣어주지 않을 수도 있습니다. 컨터이너의 역할만 하고, 해당 참조값이 필요하지 않은 UIView의 경우 **`addItem()`**에서 **생략**을 시켜주어도 됩니다.

아래는 위 코드에서 생략 가능 요소들을 적용한 코드입니다.

```swift
private func setFlexLayout() {

    addSubview(rootContainer)

    rootContainer.flex.padding(16).justifyContent(.spaceBetween).define { flex in

        flex.addItem().height(60)

        flex.addItem()
    }
}
```

다음은 한 단계 더 아래의 서브뷰들입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3f2ba333-4d81-43a3-85ef-adf60bd2d291)

```swift
private func setFlexLayout() {

    addSubview(rootContainer)

    rootContainer.flex.padding(16).justifyContent(.spaceBetween).define { flex in
        flex.addItem().direction(.row).justifyContent(.spaceBetween).alignItems(.center).height(60).define { flex in
            flex.addItem(iconImageView).size(60) // 아이콘
            flex.addItem().width(8) // spacer
            flex.addItem() // 타이틀, 서브타이틀, 별점
            flex.addItem().width(8).grow(1) // spacer
            flex.addItem(getButton).width(70).height(30) // 받기 버튼
        }

        flex.addItem().direction(.row).alignItems(.center).justifyContent(.spaceBetween).define { flex in
                        // 스크린샷들
            screenshots.forEach { screenshot in
                flex.addItem(screenshot).aspectRatio(392/696).width(32%)
            }
        }
    }
}
```

**.alignItems()**

**서브뷰들의 정렬 방법**을 지정해줍니다. **`.stretch`**, **`.start`**, **`.end`**, **`.center`**, **`.baseline`** 이 있고 기본값은 **`.stretch`**입니다. 때문에 위처럼 **`.center`**로 지정해주지 않으면 '받기 버튼'이 상단에 붙어버리게 됩니다.

**.height(), .width(), .size()**

**크기**를 지정해 줄 때 사용합니다. 크기값을 **`float`**으로 넣어줄 수도 있고, **`%`**로 넣어줄 수도 있습니다.

아래쪽 스크린샷의 경우 부모뷰(컨테이너)의 32% 너비로 그려주도록 `.width(32%)`로 설정해 주었습니다.

**.addItem()**

위에서도 설명했지만 참조가 필요 없는 UIView의 경우 뷰 값을 생략하여 사용할 수 있습니다.

위 코드에서처럼 **Spacer**역할로도 사용할 수 있습니다.

**.grow()**

item이 **얼마나 커질 수 있는지** 지정해 줄 수 있습니다. **`0`**, **`1`**, **`2`** 값을 넣어줄 수 있고 기본값은 **`0`**입니다.

**`0`**일 땐 컨테이너가 커져도 item의 크기는 본래 크기를 유지합니다.

**`1`**일 땐 컨테이너가 커지는 만큼 커집니다.

**`2` 이상일** 땐 나머지 item들보다 1:n배만큼 커질 수 있습니다.

위 예제에서는 '받기 버튼' 왼쪽 spacer뷰에 **`.grow(1)`**로 적용하여 그 사이 공간을 채우도록 만들었습니다.

(반대의 개념으로 **`.shrink()`**가 있는데 다음 단계에서 설명 예정)

**.aspectRatio()**

해당 item의 **가로, 세로의 비율**을 설정할 수 있습니다.

예제에서는 스크린샷의 비율(392/696)로 설정하고 width도 `%`로 해주어 디바이스 해상도와 상관없이 비율을 유지하며 크기를 잡을 수 있도록 설정했습니다.

다음은 타이틀, 서브타이틀, 별점이 있는 서브뷰입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/050ecf38-4a17-4227-90ea-92a5f3847387)

```swift
private func setFlexLayout() {

    addSubview(rootContainer)

    rootContainer.flex.padding(16).justifyContent(.spaceBetween).define { flex in

        flex.addItem().direction(.row).justifyContent(.spaceBetween).alignItems(.center).height(60).define { flex in
            flex.addItem(iconImageView).size(60) // 아이콘
            flex.addItem().width(8) // spacer
            flex.addItem().shrink(1).define { flex in
                flex.addItem(titleLabel).height(16) // 타이틀
                flex.addItem().height(4) // spacer
                flex.addItem(subTitleLabel).height(12) // 서브타이틀
                flex.addItem().height(4) // spacer
                flex.addItem().direction(.row).alignItems(.end).define { flex in
                    flex.addItem(starRateView).height(14).width(57) // 별점
                    flex.addItem().width(16) // spacer
                    flex.addItem(ratingLabel).height(13).width(60) // 평가수
                }
            }
            flex.addItem().width(8).grow(1) // spacer
            flex.addItem(getButton).width(70).height(30) // 받기 버튼
        }
        flex.addItem().direction(.row).alignItems(.center).padding(0).justifyContent(.spaceBetween).define { flex in
                        // 스크린샷들
            screenshots.forEach { screenshot in
                flex.addItem(screenshot).aspectRatio(392/696).width(32%)
            }
        }
    }
}
```

**.shrink()**

item이 얼마나 줄어들 수 있을지 지정해 줄 수 있습니다. **`0`**, **`1`**, **`2`** 값을 넣어줄 수 있고 기본값은 **0**입니다.

(위에서 설명했던 `grow()`의 반대 개념입니다)

**`0`**일 땐 컨테이너가 줄어들어도 item의 크기는 본래 크기를 유지합니다.

**`1`**일 땐 컨테이너가 줄어드는 만큼 줄어듭니다.

**`2` 이상**일 땐 나머지 item들보다 n:1 만큼 작아질 수 있습니다.

위 예시에서는 타이틀, 서브타이틀이 numberOfLines가 1로 설정되어 있는데, text가 길어지면 컨테이너 밖으로 넘어갈 수가 있어서, 해당 컨테이너에 **`.grow(1)`**을 설정해 주었습니다.

---

그 밖에 잘 사용하는 설정

**.backgroundColor()**

**UIColor**값을 넣어 배경색을 지정해 줄 수 있습니다.

view나 컨테이너 등에 지정해 주어 영역을 확인하기에 유용합니다.

**.margin()**

마진을 정해줄 수 있습니다. **float**으로 설정할 수도 있고 **%**로도 설정할 수 있습니다.

**padding**과 마찬가지로 **`.marginTop`**, **`.marginLeft`**, **`.marginBotton`**, **`.marginRight`**, **`.marginHorizontal`**, **`.marginVertical`** 등의 SugarAPI를 제공합니다.

**.position()**

위치를 **`.relative`**(상대적위치)로 할 것인지, **`.absolute`**(절대적위치)로 할 것인지 설정할 수 있습니다. 기본값은 **`.relative`**로 그동안 위에서 작성했던 것들은 모두 상대적으로 작성한 것입니다.

**`.absolute`**로 설정한 **item**은 부모뷰의 좌표를 기준으로 위치를 따로 잡아주어야 합니다. **`.absolute`**로 설정한 **item**은 **`.relative`**로 설정된 다른 **item**들의 위치 설정에 영향을 주지 않습니다.

**`.top()`**, **`.left()`**, **`.bottom()`**, **`.right()`**, **`.start()`**, **`.end()`**, **`.vertically()`**, **`.horizontally()`**, **`.all()`** 로 절대적 위치를 잡아줄 수 있습니다. 각 설정은 **`float`**과 **`%`**로 넣을 수 있습니다.

**.minWidth(), .maxWidth(), .minHeight(), .maxHeight()**

**item**의 **너비, 높이의 최대, 최소값**을 설정해 줄 수 있고, **`float`**과 **`%`**로 정할 수 있습니다.

## **ScrollView의 세팅**

아래는 스크롤뷰 세팅의 예시입니다.

```swift
import Foundation
import FlexLayout
import PinLayout
import Then

class FlexScrollSample: UIViewController {

    private let scrollView = UIScrollView()

    private let contentContainer = UIView()

    init() {
        super.init(nibName: nil, bundle: nil)

                // FlexLayout 작성
        setFlexLayout()
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()

        layout()
    }

    private func layout() {
        scrollView.pin.all()

        contentContainer.pin.top().left().right()

        contentContainer.flex.layout(mode: .adjustHeight)

        scrollView.contentSize = contentContainer.frame.size
    }

    private func setFlexLayout() {
                // FlexLayout 작성
        contentContainer.flex.paddingHorizontal(16).define { flex in
            (0...100).forEach {
                let text = "\($0)"
                let label = UILabel().then {
                    $0.text = text
                }
                flex.addItem(label)
            }
        }
                // FlexLayout 작성 끝

        scrollView.addSubview(contentContainer)
        view.addSubview(scrollView)
    }
}
```

요지는 **`layout()`**에서 **contentContainer**뷰와 **scrollView**를 세팅하는 부분입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6a5a7498-1752-4b89-9366-97d811a9612c){: width="50%" height="50%"}