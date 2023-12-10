---
title: (Architecture) MVI Pattern
author: Cody
date: 2022-11-23 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Swift, Architecture, MVI]
pin: false
mermaid: true
---

이번에 다녀온 `SyncSwift2022` 컨퍼런스에서`MVI` 패턴에 대한 언급이 2회 등장했습니다.

두번 모두 `SwiftUI`에 대한 설명중에

`선언적 UI인 SwiftUI`에서는 `MVVM`보다는 `MVI`패턴이 더 어울린다는 내용이었습니다.

여기서 `MVI`는 `Model`, `View`, `Intent`를 의미하고,
아래와 같은 단방향 데이터 흐름을 보여주는 패턴입니다.

`View`로부터 사용자가 `UI event`를 일으키면,

그 `action`을 받아서 `Intent`가 `Model`의 `State`를 update시키고,

그 update된 `Model`의 `State`를 `View`가 반영해서 사용자에게 보여주는 흐름입니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8b17a8b8-c860-4f32-a07d-2f5ce55cc926)

(SyncSwift2022에서 문상봉님 세션에서는 `Intent`역할을 좀더 세분화해서 `SideEffect`레이어를 추가해서 적용을 하셨다고 소개를 해주셨습니다)

위의 단방향 데이터 흐름은

애플의 `WWDC2019` '`Data Flow Through SwiftUI`'세션에서 보여주었던 흐름과 일치합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7c838da2-0196-4652-ae23-2c9f57d170dc)

`ViewModel`자리에 `Intent`가 있어서 비슷해보이지만,

단방향 데이터 흐름에 주목해서 보시면 조금 다릅니다.

꼭 `SwiftUI`로만 `MVI`를 할 필요는 없죠?

`RxSwift`와 `UIKit`으로 구현한 `MVI`의 간단한 예제를 올려봅니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/51db81b3-a4fd-49d5-8e96-7800021c2c5a)

사용자가 [Get Poster!]버튼을 누를 때마다 (`User`가 `UI Event`를 일으키면)
랜덤으로 이미지를 받아오는 `API`로부터 이미지를 받아와서 (그 `Action`을 `Intent`가 받아서 `Model`의 `State`를 `Update`시키고)
화면 포스터영역을 갱신시켜 보여주는 (`Update`된 `Model`의 `State`를 `View`에 반영)
단순한 앱입니다.

먼저 `Model`입니다.
`getImage` 함수를 호출하면 `Single`로 `UIImage`를 방출합니다.
(lorem.space 참 고마운 API입니다🤩)

```swift
import Foundation
import RxSwift
import UIKit
import Alamofire

struct ImageModel {
    func getImage() -> Single<UIImage> {
        return Single.create { observer -> Disposable in
            AF.request("https://api.lorem.space/image/movie")
                .responseData { response in
                    switch response.result {
                    case let .success(imageData):
                        if let image = UIImage(data: imageData) {
                            observer(.success(image))
                        }
                        else {
                            fatalError()
                        }
                    case let .failure(error):
                        observer(.failure(error))
                    }
                }
            
            return Disposables.create()
            
        }
    }
}
```

그 다음은 `Intent`입니다.
버튼의 액션을 `ImageModel`로 전달하며, Update된 상태(Image)를 `PublishRelay`로 방출시키고,
`PublishRelay`로 방출된 `Image`를 `View`의 `ImageView`로 갱신시켜주는 역할을 하고 있습니다.

```swift
import Foundation
import RxSwift
import RxRelay
import UIKit

class PosterViewIntent {
    let imageObserver = PublishRelay<UIImage>()
    let imageModel = ImageModel()
    let disposeBag = DisposeBag()
    
    var viewController: PosterViewController?
    
    func bind(to viewController: PosterViewController) {
        self.viewController = viewController
        
        imageObserver.subscribe(on: MainScheduler.instance)
            .subscribe { image in
                viewController.imageView.image = image
                viewController.imageView.backgroundColor = .clear
            }
            .disposed(by: disposeBag)
    }
    
    func getImageButtonTouched() {
        imageModel.getImage()
            .subscribe(onSuccess: { image in
                self.imageObserver.accept(image)
            }, onFailure: { error in
                print("error:", error)
            })
            .disposed(by: disposeBag)
    }
}
```

마지막으로 `View`입니다.
`View`에서는 layout을 그려주고 `Intent`와 바인딩만 해주면 끝입니다.

```swift
import UIKit
import SnapKit
import Then
import RxSwift
import RxCocoa

class PosterViewController: UIViewController {
    
    let disposeBag = DisposeBag()
    let intent = PosterViewIntent()
    
    let imageView = UIImageView().then {
        $0.backgroundColor = .lightGray
        $0.contentMode = .scaleAspectFit
    }
    
    let button = UIButton().then {
        $0.backgroundColor = .blue
        $0.layer.cornerRadius = 10
        $0.setTitle("Get Poster!", for: .normal)
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        
        layout()
        bind()
    }
    
    func layout() {
        [imageView, button]
            .forEach { view.addSubview($0) }
        
        imageView.snp.makeConstraints {
            $0.top.equalTo(view.safeAreaLayoutGuide.snp.top)
            $0.leading.trailing.equalToSuperview()
            $0.height.equalTo(imageView.snp.width)
        }
        
        button.snp.makeConstraints {
            $0.top.equalTo(imageView.snp.bottom).offset(20)
            $0.leading.trailing.equalToSuperview().inset(20)
            $0.height.equalTo(40)
        }
    }
    
    func bind() {
        intent.bind(to: self)
        button.rx.tap.bind {
            self.intent.getImageButtonTouched()
        }
        .disposed(by: disposeBag)
    }
}
```

샘플 소스는 아래에 공유되어 있습니다.

개발 편의를 위해 `SnapKit`, `Then`, `SwiftUI Preview`도 사용되었습니다.

https://github.com/SwiftyCody/RxMVISample
