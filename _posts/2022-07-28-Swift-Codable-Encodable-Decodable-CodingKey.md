---
title: (Swift) Codable(Encodable, Decodable), CodingKey
author: Cody
date: 2022-08-30 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, Codable, Encodable, Decodable, CodingKey]
pin: false
mermaid: true
---
Stand alone으로 돌아가는 앱도 있겠지만,

우리가 사용하는 많은 앱들은 서버와 통신을 주고받으면서 동작을 합니다.

서버와 데이터를 주고받을 때 주로 JSON 데이터를 사용하는데요.

이 JSON을 다룰 때 매우 유용한 게 `Codable(Encodable, Decodable)`, `CodingKey` 입니다.

`Codable` 문서를 먼저 보면,
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/98d0ef02-89ef-4f93-8224-27247f1b64ae)

`Codable`은 `Encodable`, `Decodable`를 `typealias`를 한 것이며, `type` 혹은 제네릭 제약으로 `Codable`을 사용한다면 `Encodable`, `Decodable` 프로토콜을 모두 준수하게 된다.라고 되어있습니다.

`Encodable`은 자신을 외부 표현으로 인코딩할 수 있는 형식이라고 되어 있는데,

이는 앱 내 데이터 모델을 `JSON` 데이터로 변환할 수 있는 것이라고 보면 됩니다.

`Decodable`은 거꾸로 외부 표현에서 자신으로 디코딩할 수 있는 형식이라고 되어있는데,

이는 서버로부터 받은 `JSON` 데이터를 앱 내 데이터 모델로 변환할 수 있는 것이라고 볼 수 있습니다.

`openweathermap.org`의 `API`를 이용해서 날씨 데이터를 가져오는 예시입니다.

`APIKey`를 발급받고, 올바른 도시명과 함께 데이터를 요청하면 아래와 같은 `JSON` 데이터가 내려옵니다.

```swift
{
	"coord": {
		"lon": 126.9778,
		"lat": 37.5683
	},
	"weather": [
		{
			"id": 801,
			"main": "Clouds",
			"description": "few clouds",
			"icon": "02d"
		}
	],
	"base": "stations",
	"main": {
		"temp": 306.81,
		"feels_like": 308.76,
		"temp_min": 303.84,
		"temp_max": 306.81,
		"pressure": 1010,
		"humidity": 43,
		"sea_level": 1010,
		"grnd_level": 1004
	},
// 이하 데이터 생략
```

우리는 여기서 `weather`배열의 데이터와 `main`데이터만 가져와서 사용을 하려 합니다.

`Codable`프로토콜을 따르는 `struct`를 작성해봅니다.

위에서 미리 확인한 `JSON` 데이터에서 사용할 데이터의 이름과 타입을 맞춰서 작성해주면 됩니다.

```swift
struct WeatherInformation: Codable {
    let weather: [Weather]
    let main: Temperature
    let name: String
}

struct Weather: Codable {
    let id: Int
    let main: String
    let description: String
    let icon: String
}

struct Temperature: Codable {
    let temp: Double
    let feels_like: Double
    let temp_min: Double
    let temp_max: Double
}
```

서버에서 내려준 `JSON`의 데이터 이름들을 맞춰서 작성을 하긴 했지만,

`Swift`를 작성할 때는 보통 스네이크 표기법보다는 카멜 표기법을 사용하는 것이 일반적입니다.

그리고 `main` 같은 내용이 추측이 안 되는 이름보다는 다른 이름을 사용하고 싶기도 합니다.

이처럼 `서버에서 내려준 값을 앱에서 정의한 새로운 이름으로 바꿔서 디코딩하고 싶을 때`가 있는데

이때 필요한 게 `CodingKey` 프로토콜입니다.`

위에서 `Codable`로 정의한 `struct`에 `CodingKey`프로토콜을 따르는 `enum`을 정의해주면 됩니다.

```swift
struct WeatherInformation: Codable {
    let weather: [Weather]
    let temp: Temperature
    let name: String

    enum CodingKeys: String, CodingKey {
        case weather
        case temp = "main"
        case name
    }
}

struct Weather: Codable {
    let id: Int
    let main: String
    let description: String
    let icon: String
}

struct Temperature: Codable {
    let temp: Double
    let feelsLike: Double
    let minTemp: Double
    let maxTemp: Double

    enum CodingKeys: String, CodingKey {
        case temp
        case feelsLike = "feels_like"
        case minTemp = "temp_min"
        case maxTemp = "temp_max"
    }
}
```

그리고 `openweathermap.org`로 데이터를 요청하고,

받아온 `JSON` 데이터를 위에서 작성한 모델로 변환하여 사용하는 함수를 작성해봅니다.

```swift
func getCurrentWeather(cityName: String) {
        guard let url = URL(string: "https://api.openweathermap.org/data/2.5/weather?q=\(cityName)&appid=(apikey)") else {
            return
        }
        
        let session = URLSession(configuration: .default)
        session.dataTask(with: url) { [weak self] data, response, error in
            guard let data = data, error == nil else { return }
            let decoder = JSONDecoder()
            guard let weatherInfomation = try? decoder.decode(WeatherInformation.self, from: data) else { return }
            
            DispatchQueue.main.async {
                // 화면 갱신 코드
            }
        }.resume()
    }
```

위에서 받아온 `weatherInformation`을 확인해보면 아래와 같습니다.
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d130bff9-057b-4b0b-aae7-497be3a159c2)

우리가 `WeatherInformation`에 정의한 대로 데이터가 잘 `Decode` 된 것을 확인할 수 있습니다.

이제 `Decode`된 데이터로 화면에 잘 그려주기면 하면 됩니다!