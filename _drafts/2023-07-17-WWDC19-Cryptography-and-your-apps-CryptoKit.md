---
title: (WWDC19) Cryptography and your apps (CryptoKit)
author: Cody
date: 2023-07-17 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, WWDC19, WWDC, CryptoKit]
pin: false
mermaid: true
---
CryptoKit Documentation: [https://developer.apple.com/documentation/CryptoKit](https://developer.apple.com/documentation/CryptoKit)

(WWDC19)Cryptography and your apps: [https://developer.apple.com/videos/play/wwdc2019/709/](https://developer.apple.com/videos/play/wwdc2019/709/)

(Cryptography and your apps 세션의 Introducing Apple CryptoKit 파트의 정리)

# CryptoKit에서 할 수 있는 것

- `Hash 함수`: SHA-256, SHA-384, SHA-512
- `Symmetric-Key Cryptography`
    - Message Authentication Code: HMAC
    - Authenticated Encryption: AES-GCM(Chacha20Poly1305)
- `Public-Key Cryptography`: Curve25519. P-256, P-384, P-512
    - Key Agreement
    - Signitures
- `Insecure Module`(안전하지 않은 모듈)
    - Hash Functions: MD5 ,SHA-1

## Hash 함수

- deterministic fixed-size digest(결정론적 고정 크기 다이제스트) 생성
- Swift의 Hashable과는 달리, CryptoKit의 Hash 함수는 collision resistance과 같은 암호화 프로퍼티를 제공 → 동일한 digest로 hash될 2개의 입력을 찾기 어렵다는 것을 의미.
- 파일의 무결성 확인:

```swift
let audioData = FileManager.default.contents(atPath: filePath)!

// hash하려는 오디오 데이터
let digest = SHA256.hash(data: audioData) // SHA256 hash함수로 digest를 계산
```

- 파일을 스트리밍하는 경우 입력 스트림에서 파일을 읽을 수 있음.

```swift
// digest를 점진적으로 계산하려는 케이스
let hasher = SHA256()
let fileStream = InputStream(fileAtPath: filePath)!
fileStream.open()
let bufferSize = 64000
let buffer = UnsafeMutablePointer<UInt8>.allocate(capacity: bufferSize)

while fileStream.hasBytesAvailable { // 해시하려는 데이터를 전달하고 이를 위해 업데이트 메서드를 한 번 또는 여러 번 호출
    let read = fileStream.read(buffer, maxLength: bufferSize)
    let bufferPointer = UnsafeRawBufferPointer(start: buffer, count: read)
    hasher.update(bufferPointer: bufferPointer)
}

let digest = hasher.finalize() // digest 반환
```

## Authenticated Encryption

- 인증(authentication)과 암호화(encryption)을 모두 제공.
- 인증이 광범위 공격을 방지: CryptoKit은 단일 API 호출로 인증과 암호화를 제공(인증과 암호화를 수동으로 결합시, 다양한 공격을 야기시킬 수 있음)

```swift
// 복호화키 초기화
let key = SymmetricKey(data: keyData)

// SealedBox 초기화
// nonce, 암호 텍스트 및 태그를 특정방식으로 결합해야하는 사양을 구현하는 작업을 수행하는 경우 이용할 수 있음.
// 특정 nonce값을 전달해야하는 프로토콜을 구현하는 경우 이를 지원.
guard let sealedBox = ChaChaPoly.SealedBox(combined: downloadedData) else {
    throw MapDownloaderError.invalidDownload
}

// SealedBox 오픈(authenticates + decrypts)
let mapData = try ChaChaPoly.open(sealedBox, using: key)
```

## Signatures

- Private Key를 이용한 데이터 인증하는데 서명을 사용.
- 서명을 사용하여 associated Public Key를 이용한 데이터 확인.
- 권한 부여 및 작업을 위해 서명을 사용하려는 예시. 2단계 인증, 송금 트랜잭션.)
    - 디바이스에서 개인키 생성→ associated 공개키 생성→ 공개키를 서비스에 전달, 등록→ 작업을 수행시 개인키로 트랜젝션 데이터에 대한 서명을 생성.→ 트랜잭션 데이터와 서명을 서버로 전달.→ 서버는 서명이 올바른지 확인 후 서명이 맞으면 작업을 진행.

```swift
// 개인키 생성
let privateKey = P256.Signing.PrivateKey()
let publicKeyData = privateKey.publicKey.compactRepresentation! // compact(압축) 표현으로 사용
// (공개키의 여려 representation을 지원함)

// 서버에 공개키 등록
// (생략)

// keychain에 개인키 저장
// (생략)

// 트랜잭션 데이터에 서명
let signature = try privateKey.signature(for: transactionData)
```

이 매우매우 중요한 키를 보호하기 위해 `Secure Enclave`가 필요.

## Secure Enclave

- [https://support.apple.com/ko-kr/guide/security/sec59b0b31ff/web](https://support.apple.com/ko-kr/guide/security/sec59b0b31ff/web)
- 추가 보안 계층을 제공하기 위해 메인 프로세서와 분리된 하드웨어 기반 키 관리자.
- TouchID, FaceID와 같은 중요한 시스템 기능의 일부로 사용됨.
- 사용법은 매우 간단. 위 키 생성코드에서 `SecureEnclave`를 사용하도록만 하면 끝

```swift
// Secure Enclave 사용가능 여부를 체크
guard SecureEnclave.isAvailable else {
// 사용할 수 없는 디바이스 처리
}

let privateKey = try SecureEnclave.P256.Signing.PrivateKey()

// SecureEnclave로 키 생성
let publicKeyData = privateKey.publicKey.compactRepresentation!

// keychain에 개인키 저장
// (생략)

let signature = try privateKey.signature(for: transactionData)
```

- `Secure Enclave`의 장점은 `키 사용을 제한`할 수 있다는 점.
- 예시
    - 생성할 키가 디바이스가 잠금해제된 경우에만 액세스할 수 있고, 이 장치에서만 사용할 수 있도록 제한
    - 개인키로 작업할 때(privateKeyUsage), 사용자의 존재(userPresence)가 필요하다고 명시
        - userPresence를 요구한다는 것은 사용자에게 생체 인증을 요청하거나 장치 암호를 입력하라는 메시지가 표시됨을 의미
    - 생성중인 키의 이니셜라이저에 전달만 하면 해당 정책이 적용됨.

```swift
let accessControl = SecAccessControlCreateWithFlags(
    nil,
    kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
    [.privateKeyUsage, .userPresence],
    nil
)!

let privateKey = try SecureEnclave.P256.Signing.PrivateKey(
    accessControl: accessControl
)
```

- 예시2
    - 인증이 필요한 이유에 대해 사용자에게 추가적인 컨텍스트를 제공할 수도 있음.
    - 마찬가지로 키생성 이니셜라이저에 컨텍스트를 전달하면 적용됨.

```swift
let accessControl = SecAccessControlCreateWithFlags(
    nil,
    kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
    [.privateKeyUsage, .userPresence],
    nil
)!

let authContext = LAContext()
authContext.touchIDAuthenticationAllowableReuseDuration = 10
authContext.localizedReason = "Authorizing $10 transfer to Bob"

let privateKey = try SecureEnclave.P256.Signing.PrivateKey(
    accessControl: accessControl,
    authenticationContext: authContext
)
```

# 성능

- CryptoKit은 corecrypto(Apple 기본 암호화 라이브러리) 위에 구축.
- CPU 설계팀이 어셈블리코드를 수동으로 튜닝.
    - side-channel resistance와 같은 corecrypto의 보안완화를 활용.
    - corecrypto는 FIPS인증을 받았기 때문에, FIPS 호환 방식으로 사용 가능.