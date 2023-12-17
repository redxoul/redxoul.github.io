---
title: (WWDC22) (4) App Intents, PassKey
author: Cody
date: 2022-06-12 00:34:00 +0800
categories: [iOS, iOS News and Updates]
tags: [App Intents, PassKey, WWDC, WWDC22]
pin: false
mermaid: true
---
`WWDC2022`(Platforms State of the Union)에서 추가된 `App Intents, Passkey`의 내용을 정리해봅니다.

개별 세션의 내용을 참조한 내용들도 있으며, 글은 계속 수정될 수 있습니다.

### App Intents Framework

앱의 기능을 시스템에서 사용가능하게 해서 사용자가 Siri와 단축어로 앱을 자동화하여 사용할 수 있게 해줍니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d46ef205-4dd4-4740-b40c-7b5c08ee944e)

사용자가 사용자화 단축어로 앱들의 기능들을 재조합할 수 있게 해줍니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9dfbbea2-a4c4-49c6-b738-0abd5b04d41d)

기존엔 사용자가 직접 단축어를 시리에 추가해야 사용할 수 있었지만, iOS16에서는 `App Intents Framework`로 이를 자동화시켰습니다.App Intents가 단축어와 합쳐져 `App 단축어`가 됨. 사용자가 별도 설정없이 Siri에서 사용 가능.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ada5a1ab-b3c6-4576-a0e5-ca0568ee599b)

사용자가 별도의 설정없이 시리로 바로 앱의 동작을 실행

Siri뿐만 아니라 시스템 전반에서 앱의 기능을 쓸 수 있게 해줍니다.Spotlight에서 검색할 때 해당 Intents를 적용한 단축기능이 앱 제안 바로 아래에 뜨게 됩니다.

이를 위해서 Donation 같은 추가적인 API를 적용할 필요가 이제 없습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/aa7a1ae1-cb87-4a6b-bd5b-ddfad5ee22ed)

단축어 앱에서도 바로 사용 가능합니다.`App Intents`는 iOS10이상에서 사용하는 기존의 `SiriKit intents` 프레임워크의 다음 단계로

기존에 Intents를 채택해 위젯이나 미디어, 메세징과 같은 도메인과 통합했다면 `SiriKit Intents` 프레임워크를 계속 사용하기를 권장한다고 합니다.

Siri와 단축어를 위한 사용자화 Intents를 개발하는 개발자들은 `App Intents`로 업그레이드를 하는 것이 좋다고 합니다.기존 `SiriKit Intents`를 만들었다면 `IntentDefinition` 파일에서 Convert버튼을 누르면 변환됩니다.

### PassKey
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ebac5c0a-bb74-4333-a42a-452b3ebebf4f)

`FIDO Alliance`(Apple, Google, Microsoft, 등) 내의 여러 플랫폼 업체들과 협력하여 호환되도록 개방형 표준으로 만든 FIDO 솔루션입니다.
(관련 기사: https://www.apple.com/kr/newsroom/2022/05/apple-google-and-microsoft-commit-to-expanded-support-for-fido-standard/)

기존 비밀번호의 문제점들(피싱, 계정 간 재사용, 웹사이트 유출 등)을 해결하기 위해 만들었습니다.

사용자는 기존의 친숙한 자동채우기 스타일의 UI, 생채인증(FaceID, TouchID) UI를 통해 비밀번호 및 계정에 대한 설정을 할 수 있게 됩니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/5ca6a172-8d63-45ca-9730-99576b128466)

`PassKey`를 통해 계정을 설정할 때는 사이트의 복잡성 요구사항에 맞춘 비밀번호 생성을 할 필요가 없습니다. 알아서 해당 앱이나 웹사이트 만을 위한 키를 생성하고 생체인증으로 보호합니다. (애초에 사용자가 암호를 생성하지 않았기 때문에)잊어버릴 수도, 재사용할 수도, 추측도 할 수가 없게 되었습니다.

생성 시 ID만 입력하고, iCloud KeyChain에 `PassKey`를 저장하며 모든 Apple기기와 동기화됩니다.

로그인 시에는 이미 저장된 `PassKey`가 있다면 ID입력할 필요도 없이 `PassKey`인증 1단계로 간단하게 로그인할 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/12dd7649-2bb3-4467-bb30-64b962745dd4)

여러 플랫폼(Apple, Google, Microsoft, 등)이 채택한 공개 업계 표준을 기반으로 하기 때문에 Apple기기에서 생성한 `PassKey`로 웹사이트에서도 로그인이 가능합니다.

타 플랫폼에서는 웹사이트에서 사용자가 ID를 제출하고, 휴대폰으로 로그인 옵션 선택 후,
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3beb8ccc-a5bd-46e8-ab7b-eb07b758d324)

QR코드로 스캔하고,
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a53f19e4-cfb4-4e26-8bf1-a2cd07fb982e)

Apple기기와 PC가 통신 후, 생체인증을 하면 끝입니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ff92bbad-3f3d-43fa-a468-86364eaf5673)

간단해보이지만, QR코드를 스캔하는 순간 이면에서는 로컬키 협의가 이뤄지고, 근접성을 증명하고, 종단간 암호화된 통신채널을 만듭니다.

맥에서는 터치아이디 한번으로 사이트에 더 빠르게 로그인 가능합니다.

공개키 암호 방식을 기반하여 서버에서의 유출이 될 수 없습니다. 서버는 공개키만 저장하여 웹사이트의 위험부담을 줄여줍니다.

앱과 사이트에 절대적으로 묶여있어 엉뚱한 웹사이트에 속아 해당 패스키를 사용할 일이 없고 사용자가 겪는 자격증명 피싱이 불가능해집니다.

### 앱과 웹사이트에 패스키 적용
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/2a93f3d7-141b-4b8a-b865-591b50268d5f)

PassKey의 적용은 간단합니다.

#### 웹사이트에 적용
- 계정 백엔드가 모든 계정마다 고유한 공개키를 저장하도록 하고 인증요청을 보내도록 설정합니다.
- 웹사이트, 앱에서 사용자에게 패스키를 제안하고, 패스키 생성과 로그인을 위한 API(WebAuthn API, AuthenticationServices Framework API)를 적용해주면 됩니다.

(자세한 내용은 Meet passkeys 세션 참조. https://developer.apple.com/videos/play/wwdc2022/10092/)

#### 앱에서 적용
- 앱에서는 `webCredentials service`를 설정하여 `associated domain`을 설정하고
- ID입력 TextField에 textContentType을 .userName으로 설정해줍니다. 이렇게 해주면 시스템이 `PassKey`를 제안할 위치를 알아챌 수 있습니다.
- 그리고 서버로부터 받아온 challenge를 `ASAuthorizationPlatformPublicKey`를 이용하여 provider와 request를 생성하고, `performAutoFillAssistedRequests`를 실행합니다.(WebAuthn 용어에서 `Assertion`은 `로그인`. `Assertion Request`는 `기존 Passkey로 로그인하기 위한 요청`)
- 해당 디바이스에 `Passkey`가 있거나, 가까운 디바이스로 `Passkey`로그인을 하는 경우 위 처리로 모두 프로세스 진행 가능합니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b0e0a1a7-6ded-42f4-a527-b90d77f842de)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d6c779b0-9265-468b-b185-0c40a6c8ec32)

- 아직 `Passkey`가 없는 경우, ID입력 TextField 아래에 Password 입력을 위한 TextField를 추가로 띄워주어 기존 방식으로 진행하게 하면 사용자가 자연스럽게 넘어갈 수 있게 할 수 있습니다.
- Autofill요청을 Modal요청으로 바꾸고 싶으면 `performAutoFillAssistedRequests()`를 `performRequests()`로 전환하기만 하면 됩니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3b14aa03-2225-4881-aa5f-c18f6a444676)

- 디바이스에 해당 앱에 대한 여러 `Passkey`가 있을 수 있습니다. 기본적으로 사용가능한 모든 ID가 표시되는데, ID TextField에 입력한 ID로 제한을 하고 싶으면 ID를 서버로 전달해서 `credentialIDs`를 받아온 뒤 request에 세팅하면 해당 ID에 대한 `Passkey`만 보이도록 할 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/0b835004-b7bd-4e4c-9d9e-60c55b576cee)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/830f4657-169b-45ca-9eca-bd021ae8e701)

- 디바이스에 저장된 ID, Passkey, AppleID 로그인 중 선택하게 하는 Modal을 띄울 수도 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/473525ba-8ce9-45d1-89ba-0fe90b2e827c)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/cebccc7d-694f-40ad-a878-8d0ac7ca5472)
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/53471929-039f-4cc8-a497-ef402eef9fc6)