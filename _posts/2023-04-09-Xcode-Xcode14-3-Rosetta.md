---
title: (Xcode) Xcode 14.3부터는 Rosetta로 실행하는 것을 지원하지 않습니다.
author: Cody
date: 2023-04-09 00:34:00 +0800
categories: [iOS, iOS News and Updates]
tags: [Xcode, Xcode14.3]
pin: false
mermaid: true
---
이번 [Xcode14.3 Release Note](https://developer.apple.com/documentation/xcode-release-notes/xcode-14_3-release-notes)를 보면 아래와 같은 **General Deprecation**이 있습니다.

> Xcode isn’t supported under Rosetta. See Developer Technote “Resolving architecture build errors on Apple silicon“ for more information. (92772361)  
>⚠️ Xcode 14.3부터는 Rosetta로 실행하는 것을 지원하지 않습니다.

| Xcode14.2 | Xcode14.3 |
| -- | -- |
|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1f56dd6e-0e28-4f23-911e-87c38a5ced69)|![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/84624634-72a9-4c75-ae61-d3fbfe60c8d7)|
(좌) Xcode 14.2 이하에서 Rosetta로 실행 지원 / (우) Xcode 14.3부터 Rosetta로 실행 미지원

저는 그동안 아래 글에서 **Apple Silicon** 환경(**M1**, **M2**, ...)에서 발생할 수 있는 문제점 및 해결 방법을 정리하고 계속 글을 갱신하고 있었는데요.

[**애플 실리콘(M1, M2, ...) 환경에서의 빌드 환경 문제**](https://swiftycody.github.io/posts/%EC%95%A0%ED%94%8C-%EC%8B%A4%EB%A6%AC%EC%BD%98-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EB%B9%8C%EB%93%9C-%ED%99%98%EA%B2%BD-%EB%AC%B8%EC%A0%9C/) 

어떤 문제는 **Xcode**를 **Rosetta**로 실행하여 우회적으로 해결할 수 있었는데,

**Xcode 14.3**부터는 더이상 **Rosetta** 실행을 지원하지 않기 때문에

이제는 해당 문제를 발생시키는 모듈이 직접적으로 해결이 되어야(**Silicon**을 정식 지원을 해야) 할 듯합니다.

(https://developer.apple.com/documentation/technotes/tn3117-resolving-build-errors-for-apple-silicon)

FirebaseCrashlytics의 upload-symbols 스크립트 실행과 같은 것은 수동으로 DSYM을 올려주어도 되긴 하는데,

배포 자동화 프로세스에서 꼭 필요한 상황이라면 문제가 될 수 있을 거 같네요.

(회사에서는 아직 Xcode 14.3을 사용하지 않고 있는데, 해당 문제가 해결이 되었는지 확인을 해봐야겠습니다)