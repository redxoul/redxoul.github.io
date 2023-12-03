---
title: SSL(Secure Sockets Layer) Pinning
author: Cody
date: 2023-09-05 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [iOS, Xcode, Swift, SSL]
pin: false
mermaid: true
---
## SSL Pinning(이하 Pinning, 피닝)은?

호스트를 인증서 또는 공개키와 연결하는 프로세스입니다. 호스트의 인증서 또는 공개키를 알고 있으면 해당 호스트에 고정(Pinning)합니다.

즉, **미리 정의된 인증서** 또는 **공개키** 중 하나 또는 몇 개를 제외한 모든 인증서를 거부하도록 앱을 구성합니다. 앱이 서버에 연결할 때마다 서버 인증서를 고정된 인증서 또는 공개키와 비교합니다. 둘이 일치하는 경우에만 앱이 서버를 신뢰하고 연결을 설정합니다.

일반적으로 개발 시 서비스의 인증서 또는 공개키를 추가합니다. 즉, 모바일 앱은 앱 번들 내에 디지털 인증서 또는 공개키를 포함해야 합니다. 공격자가 핀을 오염시킬 수 없기 때문에 이 방법이 선호됩니다.

## SSL 인증서 Pinning이 필요한 이유?

일반적으로 TLS 세션 설정 및 유지 관리를 iOS에 위임합니다. 즉, 앱이 연결을 설정하려고 할 때 어떤 인증서를 신뢰하고 어떤 인증서를 신뢰하지 않을지 결정하지 않습니다. 앱은 전적으로 iOS Trust Store에서 제공하는 인증서에 의존합니다.

하지만 이 방법에는 단점이 있습니다. 공격자는 자체 서명된 인증서를 생성하여 iOS 신뢰 스토어에 포함하거나 루트 CA 인증서를 해킹할 수 있습니다. 이렇게 하면 공격자가 중간자 공격을 설정하고 앱과 주고받는 전송 데이터를 가로챌 수 있습니다.

Pinning을 통해 신뢰할 수 있는 인증서 집합을 제한하면 공격자가 앱의 기능 및 서버와 통신하는 방식을 분석할 수 없습니다.

## SSL Pinning 유형

**Pinning**을 구현할 때 2가지 방법이 있습니다.

두 가지 옵션 중에서 선택하는 것은 요구 사항과 서버 구성에 따라 다릅니다.

- **인증서 Pinning**: 서버의 인증서를 앱에 번들로 제공. 런타임에 앱은 서버의 인증서를 임베드한 인증서와 비교  
    → 서버가 인증서를 교체(변경)할 때 앱을 업로드해야 하며, 그렇지 않으면 앱이 동작할 수 없게 됨.
- **공개키 Pinning**: 인증서의 공개키를 검색하여 코드에 문자열로 포함. 런타임에 앱은 인증서의 공개키를 코드에 하드코딩된 공개키와 비교  
    → 공개키가 변경되지 않으므로 키 로테이션 정책을 위반할 수 있음.

### 인증서 고정 방식 SSL Pinning

아래는 인증서 고정방식의 **SSL Pinning**의 예시입니다.

```swift
import Foundation

class SSLPinning: NSObject {
    static let shared = SSLPinning()

    // 신뢰할 수 있는 인증서 정보를 저장하는 배열 생성
    let trustedCertificates: [SecCertificate] = [
        SecCertificateCreateWithData(nil, Data(base64Encoded: "certificate1")! as CFData)!,
        SecCertificateCreateWithData(nil, Data(base64Encoded: "certificate2")! as CFData)!
    ]

    func pin(url: URL, completion: @escaping (Data?, URLResponse?, Error?) -> Void) {
        let request = URLRequest(url: url)
        let session = URLSession(configuration: .default, delegate: self, delegateQueue: nil)
        let task = session.dataTask(with: request, completionHandler: completion)
        task.resume()
    }
}

extension SSLPinning: URLSessionDelegate {
    func urlSession(_ session: URLSession, didReceive challenge: URLAuthenticationChallenge, completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        // 인증서가 신뢰할 수 있는지 여부를 체크
        if let serverTrust = challenge.protectionSpace.serverTrust,
            SecTrustGetCertificateCount(serverTrust) > 0 {

            let serverCertificate = SecTrustCopyCertificateChain(serverTrust)!
            let serverCertificateData = SecCertificateCopyData(serverCertificate as! SecCertificate) as Data

            for trustedCertificate in trustedCertificates {
                let trustedCertificateData = SecCertificateCopyData(trustedCertificate) as Data
                if serverCertificateData == trustedCertificateData {
                    completionHandler(.useCredential, URLCredential(trust: serverTrust))
                    return
                }
            }
        }
        // 인증서를 신뢰할 수 없는 경우 request를 취소
        completionHandler(.cancelAuthenticationChallenge, nil)
    }
}
```

위 코드는 `SecCertificate` 객체로 표시되는 신뢰할 수 있는 인증서 배열을 포함하는 SSLPinning이라는 클래스를 정의해줍니다.

이 클래스에는 URL과 completion 핸들러를 받는 `pin(url:completion:)`이라는 함수도 있습니다.

SSLPinning 클래스는 `urlSession(_:didReceive:completionHandler:)` 메서드를 구현해야 하는 `URLSessionDelegate` 프로토콜을 따릅니다. 이 메서드는 서버가 일반적으로 인증 요청인 challenge를 앱에 보낼 때 호출됩니다. 이 메서드는 서버에서 제공한 인증서를 신뢰할 수 있는 인증서 배열과 비교하여 신뢰할 수 있는지 확인합니다. 일치하는 항목이 발견되면 `.useCredential` 및 `URLCredential(trust: serverTrust)`을 `completionHandler`에 전달하여 요청을 완료하고, 일치하지 않으면 `.cancelAuthenticationChallenge` 및 `nil`을 `completionHandler`에 전달하여 request을 취소합니다.

### 공개키 고정 방식 SSL Pinning

고정된 CA 공개키는 중간 또는 루트 인증서에 인증서 체인에 나타나야 합니다. Pinning된 키는 항상 도메인 이름과 연결되며, 앱은 Pinning Requirement가 충족되지 않으면 해당 도메인에 대한 연결을 거부합니다.

예를 들어 example.org 도메인 이름에 연결할 때 특정 CA 공개키가 있어야 하도록 하려면 앱의 `Info.plist` 파일에 다음 항목을 추가하면 됩니다.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/76050cba-e1d7-43a8-bb46-703bac790d34)


```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSPinnedDomains</key>
    <dict>
        <key>example.org</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSPinnedCAIdentities</key>
            <array>
                <dict>
                    <key>SPKI-SHA256-BASE64</key>
                    <string>r/mIkG3eEpVdm+u/ko/cwxzOMo1bk4TyHIlByibiA5E=</string>
                </dict>
            </array>
        </dict>
    </dict>
</dict>
```

위 예에서 고정된 공개키는 example.org 및 math.example.org, history.example.org와 같은 하위 도메인과도 연결되지만 advanced.math.example.org 또는 ancient.history.example.org와는 연결시키지 않습니다.

공개키는 `X.509 인증서`의 `DER`로 인코딩된 `ASN.1 Subject 공개키 Info Struct`의 Base64로 인코딩된 SHA-256 digest로 표현됩니다. ca.pem 파일에 저장된 아래와 같은 PEM 인코딩된 공개키 인증서가 있다고 가정하면 openssl 명령을 사용하여 해당 SPKI-SHA256-BASE64 값을 계산할 수 있습니다.

```shell
-----BEGIN CERTIFICATE-----
MIIDrzCCApegAwIBAgIQCDvgVpBCRrGhdWrJWZHHSjANBgkqhkiG9w0BAQUFADBh
MQswCQYDVQQGEwJVUzEVMBMGA1UEChMMRGlnaUNlcnQgSW5jMRkwFwYDVQQLExB3
d3cuZGlnaWNlcnQuY29tMSAwHgYDVQQDExdEaWdpQ2VydCBHbG9iYWwgUm9vdCBD
QTAeFw0wNjExMTAwMDAwMDBaFw0zMTExMTAwMDAwMDBaMGExCzAJBgNVBAYTAlVT
MRUwEwYDVQQKEwxEaWdpQ2VydCBJbmMxGTAXBgNVBAsTEHd3dy5kaWdpY2VydC5j
b20xIDAeBgNVBAMTF0RpZ2lDZXJ0IEdsb2JhbCBSb290IENBMIIBIjANBgkqhkiG
9w0BAQEFAAOCAQ8AMIIBCgKCAQEA4jvhEXLeqKTTo1eqUKKPC3eQyaKl7hLOllsB
CSDMAZOnTjC3U/dDxGkAV53ijSLdhwZAAIEJzs4bg7/fzTtxRuLWZscFs3YnFo97
nh6Vfe63SKMI2tavegw5BmV/Sl0fvBf4q77uKNd0f3p4mVmFaG5cIzJLv07A6Fpt
43C/dxC//AH2hdmoRBBYMql1GNXRor5H4idq9Joz+EkIYIvUX7Q6hL+hqkpMfT7P
T19sdl6gSzeRntwi5m3OFBqOasv+zbMUZBfHWymeMr/y7vrTC0LUq7dBMtoM1O/4
gdW7jVg/tRvoSSiicNoxBN33shbyTApOB6jtSj1etX+jkMOvJwIDAQABo2MwYTAO
BgNVHQ8BAf8EBAMCAYYwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUA95QNVbR
TLtm8KPiGxvDl7I90VUwHwYDVR0jBBgwFoAUA95QNVbRTLtm8KPiGxvDl7I90VUw
DQYJKoZIhvcNAQEFBQADggEBAMucN6pIExIK+t1EnE9SsPTfrgT1eXkIoyQY/Esr
hMAtudXH/vTBH1jLuG2cenTnmCmrEbXjcKChzUyImZOMkXDiqw8cvpOp/2PV5Adg
06O/nVsJ8dWO41P0jmP6P6fbtGbfYmbW0W5BjfIttep3Sp+dWOIrWcBAI+0tKIJF
PnlUkiaY4IBIqDfv8NZ5YBberOgOzW6sRBc4L0na4UU+Krk2U886UAb3LujEV0ls
YSEY1QSteDwsOoBrp+uvFRTp2InBuThs4pFsiv9kuXclVzDAGySj4dzp30d8tbQk
CAUw7C29C79Fv1C5qfPrmAESrciIxpg0X40KPMbp1ZWVbd4=
-----END CERTIFICATE-----
```

위 `ASN.1 Subject Info Struct`를 디코딩한 값은 아래와 같습니다.

(decoder: [https://lapo.it/asn1js/](https://lapo.it/asn1js/))
```
Certificate SEQUENCE (3 elem)
  tbsCertificate TBSCertificate SEQUENCE (8 elem)
    version [0] (1 elem)
      Version INTEGER 2
    serialNumber CertificateSerialNumber INTEGER (124 bit) 10944719598952040374951832963794454346
    signature AlgorithmIdentifier SEQUENCE (2 elem)
      algorithm OBJECT IDENTIFIER 1.2.840.113549.1.1.5 sha1WithRSAEncryption (PKCS #1)
      parameters ANY NULL
    issuer Name SEQUENCE (4 elem)
      RelativeDistinguishedName SET (1 elem)
        AttributeTypeAndValue SEQUENCE (2 elem)
          type AttributeType OBJECT IDENTIFIER 2.5.4.6 countryName (X.520 DN component)
          value AttributeValue [?] PrintableString US
      RelativeDistinguishedName SET (1 elem)
        AttributeTypeAndValue SEQUENCE (2 elem)
          type AttributeType OBJECT IDENTIFIER 2.5.4.10 organizationName (X.520 DN component)
          value AttributeValue [?] PrintableString DigiCert Inc
      RelativeDistinguishedName SET (1 elem)
        AttributeTypeAndValue SEQUENCE (2 elem)
          type AttributeType OBJECT IDENTIFIER 2.5.4.11 organizationalUnitName (X.520 DN component)
          value AttributeValue [?] PrintableString www.digicert.com
      RelativeDistinguishedName SET (1 elem)
        AttributeTypeAndValue SEQUENCE (2 elem)
          type AttributeType OBJECT IDENTIFIER 2.5.4.3 commonName (X.520 DN component)
          value AttributeValue [?] PrintableString DigiCert Global Root CA
    validity Validity SEQUENCE (2 elem)
      notBefore Time UTCTime 2006-11-10 00:00:00 UTC
      notAfter Time UTCTime 2031-11-10 00:00:00 UTC
    subject Name SEQUENCE (4 elem)
      RelativeDistinguishedName SET (1 elem)
        AttributeTypeAndValue SEQUENCE (2 elem)
          type AttributeType OBJECT IDENTIFIER 2.5.4.6 countryName (X.520 DN component)
          value AttributeValue [?] PrintableString US
      RelativeDistinguishedName SET (1 elem)
        AttributeTypeAndValue SEQUENCE (2 elem)
          type AttributeType OBJECT IDENTIFIER 2.5.4.10 organizationName (X.520 DN component)
          value AttributeValue [?] PrintableString DigiCert Inc
      RelativeDistinguishedName SET (1 elem)
        AttributeTypeAndValue SEQUENCE (2 elem)
          type AttributeType OBJECT IDENTIFIER 2.5.4.11 organizationalUnitName (X.520 DN component)
          value AttributeValue [?] PrintableString www.digicert.com
      RelativeDistinguishedName SET (1 elem)
        AttributeTypeAndValue SEQUENCE (2 elem)
          type AttributeType OBJECT IDENTIFIER 2.5.4.3 commonName (X.520 DN component)
          value AttributeValue [?] PrintableString DigiCert Global Root CA
    subjectPublicKeyInfo SubjectPublicKeyInfo SEQUENCE (2 elem)
      algorithm AlgorithmIdentifier SEQUENCE (2 elem)
        algorithm OBJECT IDENTIFIER 1.2.840.113549.1.1.1 rsaEncryption (PKCS #1)
        parameters ANY NULL
      subjectPublicKey BIT STRING (2160 bit) 001100001000001000000001000010100000001010000010000000010000000100000…
        SEQUENCE (2 elem)
          INTEGER (2048 bit) 285593844427928762732802743986205789797337868177841749601124001697190…
          INTEGER 65537
    extensions [3] (1 elem)
      Extensions SEQUENCE (4 elem)
        Extension SEQUENCE (3 elem)
          extnID OBJECT IDENTIFIER 2.5.29.15 keyUsage (X.509 extension)
          critical BOOLEAN true
          extnValue OCTET STRING (4 byte) 03020186
            BIT STRING (7 bit) 1000011
        Extension SEQUENCE (3 elem)
          extnID OBJECT IDENTIFIER 2.5.29.19 basicConstraints (X.509 extension)
          critical BOOLEAN true
          extnValue OCTET STRING (5 byte) 30030101FF
            SEQUENCE (1 elem)
              BOOLEAN true
        Extension SEQUENCE (2 elem)
          extnID OBJECT IDENTIFIER 2.5.29.14 subjectKeyIdentifier (X.509 extension)
          extnValue OCTET STRING (22 byte) 041403DE503556D14CBB66F0A3E21B1BC397B23DD155
            OCTET STRING (20 byte) 03DE503556D14CBB66F0A3E21B1BC397B23DD155
        Extension SEQUENCE (2 elem)
          extnID OBJECT IDENTIFIER 2.5.29.35 authorityKeyIdentifier (X.509 extension)
          extnValue OCTET STRING (24 byte) 3016801403DE503556D14CBB66F0A3E21B1BC397B23DD155
            SEQUENCE (1 elem)
              [0] (20 byte) 03DE503556D14CBB66F0A3E21B1BC397B23DD155
  signatureAlgorithm AlgorithmIdentifier SEQUENCE (2 elem)
    algorithm OBJECT IDENTIFIER 1.2.840.113549.1.1.5 sha1WithRSAEncryption (PKCS #1)
    parameters ANY NULL
  signature BIT STRING (2048 bit) 110010111001110000110111101010100100100000010011000100100000101011111…
```

Pinning 구성에 중복성을 도입하려면 여러 개의 공개키를 도메인 이름에 연결할 수도 있습니다.

![image](https://github.com/redxoul/redxoul.github.io/assets/9062513/2d55bf2c-7b99-4410-b150-503e65d6bcbe)


```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSPinnedDomains</key>
    <dict>
        <key>example.org</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSPinnedCAIdentities</key>
            <array>
                <dict>
                    <key>SPKI-SHA256-BASE64</key>
                    <string>r/mIkG3eEpVdm+u/ko/cwxzOMo1bk4TyHIlByibiA5E=</string>
                </dict>
            </array>
        </dict>
        <key>example.net</key>
        <dict>
            <key>NSPinnedLeafIdentities</key>
            <array>
                <dict>
                    <key>SPKI-SHA256-BASE64</key>
                    <string>i9HaIScvf6T/skE3/A7QOq5n5cTYs8UHNOEFCnkguSI=</string>
                </dict>
                <dict>
                    <key>SPKI-SHA256-BASE64</key>
                    <string>i9HaIScvf6T/skE3/A7QOq5n5cTYs8UHNOEFCnkguSI=</string>
                </dict>
            </array>
        </dict>
    </dict>
</dict>
```

예를 들어 example.net 서버 인증서에 대한 여러 공개키를 고정하려면 앱의 `Info.plist` 파일에 배열의 항목으로 개별 항목을 추가하면 됩니다. example.net 연결에 대한 Pinning Requirement를 충족하려면 서버 인증서에 이러한 키 중 하나가 포함되어야 합니다.

참고:

[Identity Pinning: How to configure server certificates for your app](https://developer.apple.com/news/?id=g9ejcf8y)

[SSL Pinning in iOS](https://arvindcs.medium.com/ssl-pinning-in-ios-30ee13f3202d)

[Preventing Man-in-the-Middle Attacks in iOS with SSL Pinning](https://www.kodeco.com/1484288-preventing-man-in-the-middle-attacks-in-ios-with-ssl-pinning)