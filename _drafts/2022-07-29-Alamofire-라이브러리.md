---
title: Alamofire 라이브러리(feat. Charts)
author: Cody
date: 2022-07-29 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Swift, Alamofire]
pin: false
mermaid: true
---
`Alamofire`는 swift기반의 네트워킹을 도와주는 라이브러리입니다.

`URLSession`을 기반으로 하는 라이브러리로,

네트워킹 작업을 단순화시켜주고 네트워킹을 위한 다양한 메서드와 `JSON`파싱 등을 제공해줍니다.

- 연결가능한 `Request`, `Response`메서드를 제공
- `Combine`을 지원
- `URL`, `JSON`형태의 매개변수 인코딩을 지원
- 파일, 데이터, 스트림, `MultipartForm` 데이터 등의 업로드
- `Request`, `Resume` `Data`로 파일 다운로드
- `URLCredential`로 인증
- `HTTPResponse` 검증
- `Upload`, `Download` 진행률 클로저
- `cURL` `Command` `Output`
- 동적으로 `Request`를 조정 및 재시도
- `TLS`인증서 및 공개키 고정
- `Network` `Reachability`

`Alamofire`를 사용하면 코드가 간소화되고, 가독성이 매우 좋아지며, 여러가지 기능들을 직접 만들지 않고 쉽게 사용할 수 있어 많은 개발자들이 사용하고 있습니다.

아래는 `Github` 주소입니다.

`Alamofire`는 `SPM`(Swift Package Manager)도 지원하여 Xcode 내에서 쉽게 Dependancy를 설정할 수 있습니다.

[GitHub - Alamofire/Alamofire: Elegant HTTP Networking in Swift](https://github.com/Alamofire/Alamofire)

가장 많이 쓰이게 되는 `request`를 살펴보면,

필수값인 `convertible` 과

`method`, `parameters`, `encoder`, `headers`, `interceptor`, `requestModifier` 매개변수가 있고

`method`는 `get`, `encoder`는 `URLEncodedFormParameterEncoder.default`가 기본값으로 되어 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a9384bac-b6c1-4bb1-ae14-2b6dd690601d)

간단한 예제를 보겠습니다.

`Corona-19-API`(https://api.corona-19.kr/)의 API를 이용하여 전국 지역별 코로나 현황을 가져오는 예제입니다.

`API Key`를 발급받고, 요청을 하면 아래와 같은 `JSON`응답값이 내려옴을 확인할 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/42b601e8-87c1-49ff-b7e3-e41ce2026433)

이 데이터에 맞춰 Model을 준비해주고,

```swift
struct CityCovidOverview: Codable {
    let korea: CovidOverview
    let seoul: CovidOverview
    let busan: CovidOverview
    let daegu: CovidOverview
    let incheon: CovidOverview
    let gwangju: CovidOverview
    let daejeon: CovidOverview
    let ulsan: CovidOverview
    let sejong: CovidOverview
    let gyeonggi: CovidOverview
    let gangwon: CovidOverview
    let chungbuk: CovidOverview
    let chungnam: CovidOverview
    let jeonbuk: CovidOverview
    let jeonnam: CovidOverview
    let gyeongbuk: CovidOverview
    let gyeongnam: CovidOverview
    let jeju: CovidOverview
}

struct CovidOverview: Codable {
    var countryName: String
    var newCase: String
    var totalCase: String
    var recovered: String
    var death: String
    var percentage: String
}
```

이제 `Alamofire`를 `import`해주고 아래와 같이 요청을 해줍니다.

매우 간단한 코드로 요청부터 응답값 확인, 디코딩까지 작성이 가능합니다.

요청은 `get`으로 해주고, 파라메터는 `Dictionary`에 넣어서 준비해주면 됩니다.

```swift
func fetchCovidOverview(completionHandler: @escaping (Result<CityCovidOverview, Error>) -> Void) {
        let url = "https://api.corona-19.kr/korea/country/new/"
        let param = ["serviceKey": "(APIKey)"]
        
        AF.request(url, method: .get, parameters: param)
            .responseData { response in
                switch response.result {
                case let .success(data):
                    do {
                        let decoder = JSONDecoder()
                        let result = try decoder.decode(CityCovidOverview.self, from: data)
                        completionHandler(.success(result))
                    } catch {
                        completionHandler(.failure(error))
                    }
                    
                case let .failure(error):
                    completionHandler(.failure(error))
                }
            }
    }
```

값이 잘 들어온 것을 확인할 수 있습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b31e3b18-ec8b-4aad-a726-88f974d55a1e)

받아온 값으로

이번 WWDC2022에서 소개된 `Charts` framework를 이용해서 `BarMark` 차트를 그려봤습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/9d5c3379-47b2-4ed9-b735-658f55ab8b31)