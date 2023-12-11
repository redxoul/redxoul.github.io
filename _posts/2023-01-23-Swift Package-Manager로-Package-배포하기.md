---
title: Swift Package Manager로 Package 배포하기
author: Cody
date: 2023-01-23 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Apple, Swift Package Manager]
pin: false
mermaid: true
---
개발할 때 유용한 패키지들을

`Swift Package Manager`, `CocoaPods`, `Carthage`를 통해서 `Dependancy` 세팅을 자주 하는데요.

매번 유용한 라이브러리들을 사용하기만 하면서,

이런 건 어떻게 배포하는 걸까? 싶었는데 이번 기회에 알아보게 되었습니다.

생각보다 쉽게 배포가 되었습니다.

### 1. `Github`에서 `Public Repo` 생성
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a7f91d9a-928a-4585-abe9-cf928338d6c2)

### 2. 로컬에 클론 받기

### 3. 터미널에서 클론 받은 경로로 들어가서 아래 명령어 실행

```
swift package init
```

실행하고 나면 아래와 같이 파일들이 생깁니다.

(`LICENSE` 파일은 `Github`에서 생성되어 클론받은 파일)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/1a2e26bc-e474-47f3-baf3-d8579fa93ce1)

### 4. `Package.swift`, `README.md` 및 `Sources`, `Tests` 작성

저는 Package.swift 파일은 거의 그대로 두고,

README는 라이브러리에 대한 설명을 마크다운으로 적절히 넣어주었습니다.

Sources에 파일은 배포할 내용으로 작성해 줍니다.

Tests는 배포하지 않으려면 삭제하고, Package.swift에서도 타겟을 제거해 줍니다.

저는 배포할 내용을 테스트해 볼 수 있는 간단한 Test코드를 작성해 두었습니다.

### 5. (Xcode, SourceTree, Fork 등) 각자 사용하는 툴로 소스를 Push

여기까지만 해주어도, `Xcode`의 `SPM`에서 `Dependancy` 설정으로 내려받아서 확인할 수 있습니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f4e1a613-badb-4685-9471-1da7f1962d26)

하지만 여기까지만 하면 사용자는 main브랜치만 받게 됩니다.

추가로 publish release까지 진행하여, 버전관리까지 해봅니다.

### 6. 버전으로 release 할 버전명을 태그를 만들어주고, Push

### 7. Github 레포에서 `Release -> 'Draft a new release'`

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/32bdf2a0-6808-4179-b3bf-6d1152f811ee)

### 8. 아까 `Push 해둔` 태그를 선택해 주고,
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b6170db7-da39-4f6c-9154-cd6908bb53b4){: width="50%" height="50%"}

release 내용을 작성해 주고 하단에 `Publish release`를 해줍니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8bca0339-7393-4667-bd9f-9c3f2352b7f7)

여기까지 하면 release 배포가 끝나고,

사용자는 `main` 브랜치가 아닌 원하는 버전에 맞춰서 받을 수 있게 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a3a936cc-f534-4a08-a1b2-374b8c6fa005)

이렇게 `SPM`으로 패키지를 배포한 결과는 아래와 같습니다.

(이 패키지를 이제 `CocoaPods`으로도 함께 배포할 수 있도록 추가해 보고 포스팅을 다시 해보겠습니다)
