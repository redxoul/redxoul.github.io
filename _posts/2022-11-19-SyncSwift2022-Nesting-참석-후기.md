---
title: SyncSwift2022 Nesting 참석 후기
author: Cody
date: 2022-11-19 00:34:00 +0800
categories: [Dev life]
tags: [etc]
mermaid: true
---

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0ce978ab-50ec-4189-88c9-dfb8b3496665)
2022년11월12일 포항에서 계최한 **SyncSwift2022: Nesting** 컨퍼런스에 참석했습니다.

일주일이나 지나서 후기를 남겨봅니다👀  
실시간 온라인으로도 진행했던 AsyncSwift는 계속 온라인으로 참여를 했었는데,  
이번 SyncSwift 컨퍼런스는 오프라인으로만 진행을 한다고 했지만  
너무 가보고 싶었기에 포항으로 가기로 했습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b514a84f-2e7e-4597-a3f9-f69fd88eec71)

지하철 첫차를 타고, KTX를 타고, 택시를 타니 겨우 세이프!  
오프라인으로 개발행사를 가보는 것이 처음이고,  
회사 밖의 다른 iOS개발자들을 만나는 것도 처음이라 두근두근합니다🤗

포항공대 입구에서 걷다보면 나오는 체인지업그라운드.   

| | |
| -- | -- |
|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/865172d9-3336-4ac4-8ffa-1bd324aebb3c)|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/99b256bb-e49f-4417-ba60-acd84137f9a9)|

내부도 인상적이었던 공간.   

| | |
| -- | -- |
|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/14d857c5-12a1-47fd-bfcd-889bb0d3709d)|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b5db0bef-eb72-4c52-807d-afde251a648f)|

Talk Session이 진행된 Media Wall 공간.   

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2e4e0102-e5a9-4336-b20e-5b96e876b035)

이벤트홀 입구   

| | |
| -- | -- |
|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/15c79502-2766-411b-a900-6d2539b98d1e)|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/61237eb6-f7a7-4305-a0bd-9cf089084768)|
|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/316968ce-aeaf-496b-95b8-92c5180d902e)|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0bf4d682-c714-4f27-a6a7-9a687fd4d6df)|

입구에서 참석자 목걸이 및 굿즈들을 받고 들어갑니다.   
목걸이 안쪽에는 전체 세션의 시간표가 있어 보기 편리해서 좋았습니다.   

| | |
| -- | -- |
| ![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/638e01e4-5c37-41d8-9dae-15d553740f5d) | ![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9c7403b3-900e-47d5-85c3-e961382c86f2) |

런치 스페셜 세션에 홍길동님은 과연?  
차례를 보면 느껴지듯이 테크 쪽의 발표(Swift 개발)가 메인이지만,  
함께 생태계를 이끌어가는 디자이너 분들도 함께 시간을 가졌습니다.

---

### 세션
(세션 요약은 개발 세션 위주로 정리했습니다)

#### MVI 패턴과 어울리는 SwiftUI 화면이동 라이브러리 만들기

플리토의 문상봉 님이 발표를 해주셨습니다.  
`SwiftUI`를 써보면 느끼는 불편함 중 하나가 화면이동에 쓰이는 `NavigationLink`입니다.  
화면이동의 역할을 `View`가 담당할 수 밖에 없고, 스택관리가 어려운데.  
플리토에서는 앱에 `SwiftUI`적용을 하면서 `MVI패턴`+`SideEffect 레이어`를 적용하고,  
`SwiftUI`의 `NavigationLink`의 단점을 극복하기 위해 직접 `LinkNavigator`라는 라이브러리를 제작하셨다고 합니다🤩  
기존 `UINavigationController`의 기능을 모두 `SwiftUI`에서 쓸 수 있고, 다양한 네비게이션 스택 제어를 지원한다고 합니다.

(Github: https://github.com/interactord/LinkNavigator)
(WWDC2022에서 발표한 NavigationStack으로 많은 개선이 생겼지만, 이걸 쓰려면 앱 타겟이..ㅎ)

#### Tuist... 한번 써봤는데요?
Apple Developer Academy의 이명환 멘티님의 경험담으로 한 발표였습니다.  
`Tuist`를 소개하고, 이의 장단점을 공유해주셨습니다.  
`Tuist`는 Xcode프로젝트를 생성, 유지보수, 상호작용을 용이하게 해주는 CLI 도구입니다.  
장점으로는 `pdxproj` conflict가 발생할 일이 없다고 하고, 이를 기반으로 모듈화를 해볼수 있다고 합니다.  
그리고 artifacts라는 것 내부에 dependancy가 존재하게 되는데 이를 통해 한눈에 파악하기 쉽고, 캐싱을 통해 빌드속도를 엄청 줄여준다고 하네요.

단점으로는 branch가 바뀔때마다 매번 tuist generate 명령을 실행. 그리고 파일이름이 알파벳순으로만 정렬이 되고,  
주로 한명이 이 환경을 세팅하고 나머지 팀원들이 따라가게 되어, 팀원들이 세팅한 사람에게 dependancy가 생기게 된다는 문제(?)를 알려주셨습니다.  
Tuist라는 도구와 모듈화에 관심이 있었는데, 발표를 통해 알게 되어 좋았습니다.

#### SwiftUI로 앱 서비스 출시까지
Apple Developer Academy의 박건호 멘티님의 발표였습니다.  
SwiftUI로 merging이라는 깃헙앱을 출시한 경험담의 공유였습니다.  
요즘 커뮤니티에 돌고있는 'SwiftUI에서 MVVM을 쓰는 것을 멈춰라'라는 이슈도 잠시 이야기하고.  
문상봉님 발표에서 소개해주셨던 `MVI패턴`의 적용에 대한 내용을 얘기해주셨습니다.

#### Launch Special Session
모 소속의 홍길동님의 세션이었습니다. 세션리스트에 숨겨진 분이셨는데 놀라운 분이셨다는ㅎ  
App 배포에 대한, 특히 기업용 앱 배포, MDM, ABM에 관한 내용을 자세하게 얘기해주셨는데요.  
저는 회사에서 엔터프라이즈 계정도 사용중이기도 하고, 이전부터 관심이 있었던 내용이라 너무 좋았습니다!  
기업용앱 배포에도 다양한 방법이 있고 각각에 특징을 너무 잘 알려주셨습니다.

#### 개인앱 개발: 이렇게만 따라하면 실패할 수 있다!
원티드랩의 김찬우님의 발표였습니다.  
(사실 발표자료가 수정본이 아니었어서 가장 마지막 발표로 바뀌셨습니다. 하지만 엄청 발표를 잘 해주셨어요!)  
세션 제목이 명확합니다. 앱개발을 하는 사람이라면 공감하는 점..  
바로 '개인앱 개발을 완성이라도 해야 실패를 하는데, 왜 우리는 실패조차 하지 못할까?'를 짚어주셨습니다.  
(다들 Mac에 만들다가 중단한 프로젝트 폴더 몇개쯤은 있잖아요..?😭)  

- 제품을 명확하게 정의내릴 수 없는 경우: 한문장으로 앱을 정의했을 때 내가 생각하는 제품과 상대방에 생각하는 제품의 격차가 적을수록 정의내리기 쉽다.
- 앱 개발의 목적이 기술 배우기인 경우: 기술 스택에 대한 집착. 기술의 러닝커브 등으로 속도와 의지가 줄음.. 좋은 제품을 만들기 위한 접근법은 아니며, 기술이 동기가 되면 기술에 흥미를 잃는 순간 앱개발도 멈추게 된다.
- 걱정과 대비가 과한 경우: 김찬우님은 개인앱을 위해 변리사까지 쓰셨다고ㅠㅠ 일단 만들고 걱정하자!  
MVP를 통한 빠른 의도의 검증과 성공의 실마리에 대한 내용을 공유해주셨습니다.

#### Swift Concurrency 적용기
Line의 김윤재님의 발표였습니다.  
기존 GCD의 문제점과 이를 해결한 Swift Concurrency에 대한 디테일한 내용을 설명해주셨습니다.  
GCD와 Concurrency의 차이점을 보여주시고, 
각각을 모두 이해하고 컨버전을 해야함을 알려주셨습니다.  
(https://engineering.linecorp.com/ko/blog/about-swift-concurrency/)

#### 당신이 TDD를 시도했다가 포기해 봤다면
뱅크샐러드의 류성두님의 발표였습니다.
개인적으로 회사에서 매우 하고 싶은 내용이었고 필요한 내용이었어서 너무 좋았습니다.  
(노션으로 가득 정리를 해서 다 적긴 어렵고)  
TDD실패의 이유와 이를 해결하기 위한 방법으로 모듈화 병렬화 등을 설명해주시고, 이를 통해 얻는 것들을 알려주시고,  
TDD를 하고 싶지만 무엇을 테스트해야하는지 모르겠는 개발자를 위한 설명,  
접근성 경험을 테스트해야 한다, `Test Code는 믿을 수 있는 '문서'`이다..등  
TDD를 위한 좋은 내용이 한가득이었던 세션이었네요🤩

#### Modular Archtecture 시작하기
당근마켓의 오강훈님의 발표였습니다.  
너무 알고 싶고, 적용하고 싶은 내용이었기에 기대하고 들었던 내용이었습니다.  
Modular Programming은 독립적이고 교체 가능한 모듈로 분리하는 것을 강조하는 소프트웨어 설계 기법으로.  
왜 모듈(Library, Framework, Package..)로 분리하는가?  
팀이 커져감에 따라 생산성이 떨어지게 되는데,  
모듈화를 통해 무거운 모듈 또한 Mock으로 개발이 가능해지고,  
여러 타겟으로 나누어 병렬빌드를 통해 빌드속도를 증가시키고,  
프로젝트 복잡도를 낮추고, 코드간 의존도를 낮추고, 격리된 환경에서 개발할 수 있게 된다.  
의존성 주입이 강제화되어 테스트 작성이 편리해진다.  
그 밖에 재사용/도메인 관점에 따른 분리, 모듈 계층 구조, 모듈 분리 순서, 모듈화를 언제 어디서부터 시작하는가, 추가로 학습해야 할 것들에 대해 이야기를 나눠주셨습니다.  

- 추가로 학습해야 할 것들: Xcode에서 빌드 실행시 어떤 일이 일어나는지 알아보기. SwiftPackage, Dynamic Framework, Static Library 차이를 알기. 의존성 역전, 의존성 주입 알기. 구현을 위한 편리한 도구 알아보기(Tuist, SPM, Mockolo, Needle, Swinject, ...)

---

### After Party
After Party는 메인 이벤트 홀이 정리되어 같은 장소에서 진행되었습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4ff2d15e-0b81-4521-abd6-5157ffc27eea)

맛있는 음식을 먹으면서 자유롭게 다른 개발자들과 이야기를 나누는 시간이었습니다.  
처음에도 썼지만 회사 밖에서 다른 iOS개발자를 만나는게 처음이었는데요.  
아카데미의 멘토, 멘티분들도 함께 즐기고, 다른 회사의 개발자분들도 만나보고 얘기 나눌 수 있는 시간이었어서 매우 즐거웠습니다.  
시간이 금방 지나가서 너무 아쉬웠습니다.  
다음에 이런 자리가 있으면 더 적극적으로 많이 만나보고 얘기를 나눠야겠다는 생각을 했습니다.  
그리고 발표해주시는 연사님들을 보니,  
나도 다른 사람들에게 좋은 경험과 지식을 나눠주는 사람이 되고 싶다는 생각,  
그리고 그런 동료가 되고 싶고 그런 동료를 갖고 싶다는 생각,  
연사님들을 동료로서 함께 일하는 분들, 함께 성장하는 분들 왕부럽다는 생각,  
등 많은 생각을 하게 되었네요.  

---

후기는 여기까지 입니다.

만났던 분들(카뱅의 sean.jeong님, 원티드랩의 찬주님, 뱅샐의 류성두님!, 라인의 코든님 내일날씨맑음님, 인포마이닝의 Lucas님, 아카데미의 ViVi님 머피님 Yeni님, 타스님, Kkoma님 등등 많은 분들) 너무 반가웠고 다음에 좋은 인연으로 다시 만나면 좋겠습니다😉