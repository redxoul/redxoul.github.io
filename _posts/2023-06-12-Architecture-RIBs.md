---
title: (Architecture) RIBs 정리
author: Cody
date: 2023-06-12 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Swift, RIBs, Architecture]
pin: false
mermaid: true
---
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/00c6092f-9ab3-4890-9e50-765a88b496e3){: width="50%" height="50%"}

 
[GitHub - uber/RIBs: Uber's cross-platform mobile architecture framework.](https://github.com/uber/RIBs)

`RIBs`는 `Uber`에서 만든 `Cross Platform` 아키텍쳐입니다.

→ 도입시 `iOS`와 `Android`가 동일한 아키텍쳐를 사용하여 `플랫폼간 협업 및 비즈니스 로직 코드를 교차 검증할 수가 있다는 이점`이 생깁니다.

`RIBs`는 여러 `Riblet`(리블렛, 하나의 기능단위. 이하 RIB)으로 구성되며 `Router`, `Interactor`, `Builder`의 약자로 RIB의 핵심 요소들을 의미합니다. 그 외에 `View`, `Presenter`, `Component`와 같은 요소도 있습니다.

# RIBs의 구성

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/5f1c2696-545b-4a1d-84a0-121db9270a8c)
출처: RIBs Wiki

## Builder
해당 `RIB`의 구성들(`Router`, `Interactor`, `View`, `Component`)을 생성하는 `Factory`로 `DI`를 정의하며, 자녀 `RIB`의 `Builder`도 인스턴스화 합니다. 각 `Component`들은 자신의 코드를 그대로 유지한 채 `DI` `Framework`(Swinject, Needle 등)을 `Builder`에서 알맞게 사용함으로써 `Component`들의 `Mockability`를 향상시켜줄 수도 있습니다. `Builder`는 `A`라는 `RIB`의 `Interactor`와 `B`라는 `RIB`의 `Router`를 생성시켜 `C`라는 `RIB`을 만드는 일도 가능하여 `재사용성을 극대화`시켜줍니다.

## Component
부모 `RIB`의 `Builder`가 자식 `RIB`의 `Builder`에게 `Component`를 통해 `Dependency Injection`을 할 수 있게 해줍니다.

## Interactor✨
비즈니스 로직을 담당합니다. Rx로 Subscribe를 하거나, State를 관리하거나, 데이터를 다루거나, 필요에 따라 Router에게 자식 RIB을 attach, detach할 것을 요청하는 등의 일을 수행합니다. 앱의 중심 로직입니다.

## Router
부모 `RIB`의 `Router`는 자식 `RIB`을 `attach`, `detach`하여 `RIBs의 논리적 트리`를 만들어줍니다. 두 `Interactor` 간의 추가적 레이어로 `Interactor`가 비즈니스 로직에 집중할 수 있게 해주며, `Mockability`도 최대한 높여주어 복잡한 `Interactor`의 로직도 테스트하기 쉽게 해줍니다.

## View(Controller)
`UI Layout`을 그리고 `ViewModel`을 통한 업데이트를 하거나 애니메이션 등을 처리합니다. `Uber`에서는 `View`를 그저 정보를 보여주기만 하도록 최대한 `'바보처럼'` 설계되었으니, `단위테스트가 필요한 로직을 넣지말 것`을 이야기합니다.

## Presenter
`View`로직이 생기면 `Optional`하게 필요해질 수 있는 요소로 `Interactor`와 `View` 사이에서 `model`의 변환을 담당합니다. `Interactor`에서 `View`로 넘어오는 `model`은 정보가 너무 많을수도, 부족할수도 있기 때문에 `Interactor`가 가지고 있는 `비즈니스 모델`을 `ViewModel`로 변환해주거나. 반대로 `View`에서 넘어오는 `Action`을 `Interactor`가 해석할 수 있도록 변환을 해줍니다. `Presenter`를 생략시 `Interactor`나 `View`가 그 역할을 해야합니다.

여기서 중요한 것은 대부분의 아키텍쳐들이 `View`를 필수요소로 가지고 있지만 RIBs에서는 필수요소가 아니라는 점입니다.  
`VIPER`와 같은 다른 아키텍쳐들과 마찬가지로 각각의 `Component`들에 적절한 역할을 나누어 기능을 구현하고자 만든 것은 같지만,

`RIBs`만의 이점은 여기에 있습니다.  
`RIBs`에서는 `View`가 필수요소가 아니기 때문에 `Router`, `Interactor`, `Builder`만으로 구성된 비즈니스 로직만을 위한 `RIB`노드를 만들 수 있습니다. 이는 `RIBs`는 `View`를 중심으로 트리가 만들어지는 것이 아닌 비즈니스 로직을 중심으로 `State`에 따른 트리가 만들어진다는 의미입니다.

---

# State Tree

아래 예시는 'Multiplatform Architecture RIBs in Swift'영상에서 나온 타다 서비스의 `State Tree`입니다.

`RIBs` 아키텍쳐는 아래에서처럼 `View`에 의해서 `State`가 변하는 것이 아닌 `비즈니스 로직에 의해 State가 변해야`는 것을 강조하고 `State` 트리에 따라 앱의 흐름이 만들어집니다.

(`View Tree`보다 더 depth가 깊고 디테일합니다)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3c12c4fc-0732-47ab-bff2-4c70de724fb7)
출처: Multiplatform Architecture RIBs in Swift (Youtube)

각각이 하나의 `RIB`이고 `View`가 있기도 하고 없기도 합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1fe247ac-dc5a-41b3-81ca-6b605663acd0)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4c053259-ff02-480c-87fc-fa3349a05063)

Root RIB에서 시작하여

로그인 정보 저장 여부에 따라 LoggedOut RIB을 Attach하거나, Main RIB을 Attach하게 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2503a07e-d7d8-46b0-9ed7-9c9c9667f622)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f9f27e02-5be9-4c32-8f05-27c0b7395417)


`Home`에서 호출(Request)을 누르면 Request RIB의 판단에 의해서 LocationEditor RIB을 생성해 Attach하고,

LocationEditor RIB에서 사용자가 출발지, 도착지의 위치를 모두 결정하고 나면,

경로를 확인하는 Confirm RIB이 만들어져 Attach되고, LocationEditor RIB이 Detach됩니다.

Confirm RIB 아래로 Coupon RIB Card RIB이 Attach하게 됩니다.

아직 Request하기 전이기 때문에 상단에 Request RIB이 역할을 하고 있습니다.

사용자가 '호출하기'버튼을 누르면

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a6a30c74-3d5e-481c-a709-ddf316ce1f43)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e6a15806-2845-489a-a8bc-847c1f7cdf21)

Confirm RIB이 Detach되고, Refinement(위치조정) RIB이 Attach됩니다.

그리고 Refinement에서 '확인'을 누르면, Request RIB이 Detach되고 Ride RIB이 Attach됩니다.

RIDE RIB에선 View가 없고, 서버에서의 유저 State가 Pending State인지, Matched State인지, DroppedOff State인지를 받아서 각각의 `RIB`을 Attach, Detach해주는 역할을 합니다.

---

# Scopes

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d271eff5-c82f-461c-a6db-7890ca7e4ea8)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/65fce72e-18ff-4d9f-83b9-7cccb65dee0c)


아랫쪽은 `State`가 아닌 `View`에 따른 `tree` 형태일 때입니다.  
깊이가 얕아지고, MainView하나에서 관리해야하는 서브뷰들이 많아지게 되어 MainView가 복잡해지게 됩니다.  
`RIBs`에서는 비즈니스 로직에 의해 `State`를 관리하고, 해당 State에 따라 `Scope`를 가두어서 관리를 할 때 더 쉬운 코드가 될 수 있도록 해줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/5cbffcea-3095-4a0c-a722-28ac746e387a)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d648b156-d0c6-4b89-9dd9-5f48ab1722f1)


윗쪽은 앱을 처음 진입했을 때 만들어진 아무런 `State`가 없는 상태의 Root RIB입니다.

로그인 정보(AuthToken, UserDTO)가 있다면 그 아래의 Main RIB을 attach하고 해당 정보를 넘겨줄 수 있고,

해당 Main RIB은 항상 AuthToken, UserDTO는 항상 있다고 가정하여(non-optional) 옵셔널 바인딩 등을 체크하지 않고 깔끔하게 코드가 작성해집니다.

```swift
// 기존 방식의 코드
class AuthenticateNetworkRequest: NetworkRequest {
    private var authToken: String?
    
    public func makeRequest() {
        if let authToken = authToken {
            addAuthToken(authToken)
        }
        super.makeRequest()
    }
    
    ...
}
```

```swift
// Main RIB으로부터 주입되어 항상 존재하는 authToken으로 작성
class AuthenticateNetworkRequest: NetworkRequest {
    private var authToken: String
    
    public func makeRequest() {
        addAuthToken(authToken)
        super.makeRequest()
    }
    
    ...
}
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/c765508f-bfaf-4e22-8cf5-dbfdbcde3842)

Ride 상태가 되어 Main RIB에 Ride RIB이 Attach된 상태.

이 상태가 되었다는 것은 `Main`에서 `Ride` 관련 객체를 `Ride`의 자식 `RIB` `Tree`에게 전달했다는 의미이고,

`Ride Scope`에 갖혀있는 모든 자식 `RIB`들은 `Ride` 객체가 있다는 가정하에 코드를 작성할 수 있게 됩니다.

```swift
// 기존 방식의 코드
public func refresh() {
    if let ride = self.ride {
        getRideStatus(ride.id)
    } else {
        getUserStatus(self.user!.id)
    }
}
```

```swift
// 항상 존재하는 ride 객체로 작성
public func refresh() {
    getRideStatus(ride.id)
}
```

---

# RIB들의 Communications

`Scope`에 따라 개발을 하다보면

어떤 `Interactor`가 다른 `RIB`에게 이벤트를 보내야할 때는 어떻게 할까요?

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d536cb81-44cf-4651-ae8f-a073232c627b)

위 예시는 Main RIB의 서브 `RIB`인 Home RIB과 Request RIB 사이에서 커뮤니케이션이 필요하게 된 상황입니다.

저런 식의 코드를 작성하기 보다는 `RIBs`에서 제안하는 Communication 방식이 있습니다.

크게 `Upwards`, `Downwards` Communication이 있습니다.

## 1. Upwards Communication

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ebcb51ec-dead-48a9-8fec-69849a3a2f2e)

`Upwards Communication`의 방법으로는 `Listener interface`를 사용하도록 합니다.

자신(`Home`)의 `Listener interface`를 선언해놓고, 그 주입받은 `Listener`를 동작시킴으로써 `communication`을 합니다.

`RIB Tree`상 부모 `RIB`은 자식 `RIB`을 알지만, 자식 `RIB`은 부모 `RIB`을 모릅니다.

자식 `RIB`이 부모 `RIB`의 특정 기능을 직접 호출하지 못하게 막아놓음으로써, 부모와 자식간의 `coupling`을 최대한 없앨 수 있습니다.

## 2. Downwards Communication

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2ca91ae0-c446-4404-a68e-c6692bb5cab9)

자식 `RIB`이 여러개가 있을 수 있기 때문에 `Downwards Communication`은 `Listener interface`로 구현할 수가 없습니다.  
위에서 다뤘던 `Scope`로 주입받는 것이 중요한데, `Downwards`로는 주입을 받을 수가 없습니다. `Property`로 `Listener`를 선언해서 해당 리스너를 호출할 수도 있지만 이는 `RIBs` 아키텍쳐가 추구하는 방향이 아닙니다.  
그래서 `Downwards`로는 `Rx`를 이용해서 전달합니다. 시퀀스로 이벤트를 전달하고 방출된 이벤트를 받는 방식입니다.  
(`RIBs`를 fork해서 만든 `ModernRIBs`는 `Rx`대신 `Combine`을 사용하도록 되어있습니다)  
그러면 Main은 Location이벤트를 만들어 계속 방출하고, 자식 RIB(Home, Request) 중 필요하면 이를 Subscribe해서 사용합니다.

---

`RIBs`는 이렇게 `Scope`를 가두고, `RIB`간의 `Communication` 방식을 제한하는 방법으로 `공동작업` 결과를 매우 끌어올려줍니다.

각 `RIB`은 의존성만 제대로 주입이 되어 있다면, 각각 독립적으로 코드를 작성할 수 있기 때문입니다.

이는 `View`가 포함되어 있는 `모듈화`도 할 수 있게 해주어 `모듈화`를 극대화할 수 있게 환경을 만들어줍니다.