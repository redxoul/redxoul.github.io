---
title: TLS와 Digital Certificate
author: Cody
date: 2023-09-04 00:34:00 +0800
categories: [iOS, Dev Terms]
tags: [iOS, TLS, DigitalCertificate]
pin: false
mermaid: true
---
## TLS(Transport Layer Security)

TLS의 주요 목표는 두 당사자 간에 교환되는 메시지에 개인 정보 보호 및 무결성을 추가하는 것.

TLS를 사용하면 해당 데이터를 신뢰할 수 없는 제3자에게 노출시키지 않고 네트워크를 통해 데이터를 전송할 수 있습니다.

### TLS 연결의 3단계

1. 클라이언트에서 서버로 연결 시도
    - 클라이언트가 암호화에 사용할 수 있는 **Cipher Suite**와 함께 지원할 수 있는 TLS버전이 나열된 메세지를 서버로 전달
    - **Cipher Suite**: TLS를 통해 네트워크 연결을 보호하는 데 필요한 알고리즘 집합(참고: [https://en.wikipedia.org/wiki/Cipher_suite](https://en.wikipedia.org/wiki/Cipher_suite))
    - 서버에서는 선택한 **Cipher Suite**로 응답하고, 하나 이상의 Digital Certificate로 다시 보냄.
    - 클라이언트에서는 해당 Digital Certificate가 유효한지 확인. 서버가 신뢰할 수 있는 서버인지, 민감한 정보를 스누핑하려는 사람인지 확인.
2. 검증 단계(공개키 암호화, 비대칭 암호화)
    - 클라이언트에서는 `pre-master secret key`를 생성하고 서버의 공개키(Certificate에 포함된 공개키)로 encrypt.
    - 서버는 암호화된 pre-master secret key를 수신하고 개인키로 decrypt. 서버와 클라이언트는 각각 `pre-master secret key`를 기반으로 `master secret key`와 `session key`를 생성.
3. `master secret key`로 클라이언트, 서버가 서로 교환하는 정보를 decrypt, encrypt.

---

## Digital Certificate

Digital Certificate는 인증서를 소유한 서버에 대한 정보를 캡슐화한 파일로 Certificate Authority(CA)가 인증서를 발급하거나 자체 서명할 수 있습니다.

- 인증서를 발급: CA가 인증서를 발급하기 전에 앱에서 인증서를 사용할 때 모두 소유자의 신원을 확인해야 함.
- 자체 서명: 신원을 인증하는 동일한 엔티티가 인증서에 서명.

### Digital Certificate의 구조

인증서의 구조는 `X.509` 표준을 사용합니다. 주요 필드는 아래와 같습니다:

- **`Subject`: 주체**. CA가 인증서를 발급한 엔터티(컴퓨터, 사용자, 네트워크 장치 등)의 이름을 제공.
- **`Serial Number`: 일련 번호**. CA가 발급하는 각 인증서에 대한 고유 식별자를 제공.
- **`Issuer`: 발급자**. 인증서를 발급한 CA의 고유 이름을 제공.
- **`Valid From`: 유효기간**. 인증서가 유효하게 되는 날짜와 시간을 제공.
- **`Valid To`: 유효기간**. 인증서가 더 이상 유효하지 않은 것으로 간주되는 날짜와 시간을 제공.
- **`Public Key`: 공개키**. 인증서와 함께 제공되는 키 쌍의 공개 키를 포함.
- **`Algorithm Identifier`: 알고리즘 식별자**. 인증서를 서명하는 데 사용된 알고리즘을 나타냄.
- **`Digital Signature`: 디지털 서명.** 인증서의 진위 여부를 확인하는 데 사용되는 비트 문자열.

공개 키와 알고리즘 식별자로 구성된 쌍은 `Subject Public Key Info`를 나타냄.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/2a5efc3e-5300-45d0-8e69-a66ac01445b9)

`X.509` 인증서는 다르게 인코딩할 수 있으며, 이는 인증서의 모양에 영향을 줍니다.

- `Privacy Enhanced Mail`(`PEM`. 개인정보보호 강화 메일): 파일 확장명이 `.pem`인 `Base-64 인코딩`. 원래는 secure email에 사용되는 인코딩 포멧이었는데 더이상 email쪽에서는 잘 쓰이지 않고 인증서 또는 키값을 저장하는데 많이 사용. `-----BEGIN XXX-----`, `-----END XXX-----` 로 묶여있는 text file이 `PEM` 형식(담고 있는 내용에 따라 XXX 위치에 `CERTIFICATE`, `RSA PRIVATE KEY` 등이 들어감)
- `Distinguished Encoding Rules`(`DER`. 고유 인코딩 규칙): 파일 확장자가 `.cer,` `.der` 및 `.crt`인 `이진 인코딩`.
- `Public Key Cryptography Standards`(`PKCS`. 공개키 암호화 표준): 단일 파일에서 공개 및 비공개 개체를 교환하는 데 사용됩니다. 확장자는 `.p7b`, `.p7c`, `.p12,` `.pfx` 등.

### Digital Certificate 유효성 검사

CA로부터 인증서를 받으면 해당 인증서는 `chain of trust` 또는 `chain of certificate`의 일부가 됩니다.(chain of trust: [https://en.wikipedia.org/wiki/Chain_of_trust](https://en.wikipedia.org/wiki/Chain_of_trust))

체인에 있는 인증서의 수는 CA의 계층 구조에 따라 다릅니다. 2단계 계층 구조가 가장 일반적입니다.

발급 CA는 사용자 인증서에 서명하고 루트 CA는 발급 CA의 인증서에 서명합니다. 루트 CA는 자체 서명되며 앱은 마지막에 이를 신뢰해야 합니다.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/0ece2ace-16df-4244-a5d3-3fd05f24ac96)


인증서 유효성 검사 중에 앱이 확인:

- The date of evaluation: 인증서가 유효하려면 인증서의 `Valid From` 와 `Valid To` 필드 사이에 있어야 함.
- 다음 발급 CA 또는 중간 CA의 공개키를 찾아서 디지털 서명을 확인. 이 프로세스는 루트 인증서에 도달할 때까지 계속됨.

> 참고: iOS는 잘 알려진 모든 루트 CA 인증서를 Trust Store에 보관. iOS에 사전 설치되어 제공되는 신뢰할 수 있는 루트 인증서를 알고 싶으면 [Available trusted root certificates for Apple operating systems](https://support.apple.com/en-us/HT209143)을 참조.

참조

[Preventing Man-in-the-Middle Attacks in iOS with SSL Pinning](https://www.kodeco.com/1484288-preventing-man-in-the-middle-attacks-in-ios-with-ssl-pinning)

[Wikipedia: Cipher suite](https://en.wikipedia.org/wiki/Cipher_suite)