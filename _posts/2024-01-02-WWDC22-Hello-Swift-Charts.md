---
title: (WWDC22) Swift Charts
author: Cody
date: 2024-01-02 00:34:00 +0800
categories: [iOS, Swift]
tags: [Swift, Charts, WWDC22, WWDC]
pin: false
mermaid: true
---

[(WWDC22)Hello Swift Charts](https://developer.apple.com/videos/play/wwdc2022/10136/)  
[(WWDC22)Swift Charts: Raise the bar](https://developer.apple.com/videos/play/wwdc2022/10137/)

## Swift Charts Framework
- Apple이 디자인한 Chart를 만들기 위한 Framework.
- SwiftUI와 동일한 선언적 문법.

### Mark
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/89a08c49-525e-4789-af28-1174cd1c4613)  
차트에서 각 항목에 대한 요소를 `Mark`라고 함.

위 그림에서 `Mark`는 `Bar Mark`.

```swift
import Charts
import SwiftUI

struct StylesDetailsChart: View {
    var body: some View {
        Chart {
            BarMark(
                x: .value("Name", "Cachapa"),
                y: .value("Sales", 900)
            )
        }
    }
}
```
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a24fd9d1-3661-4246-be1d-2d85a77c5ce6) 
`.value`로 값을 넣으면 막대의 높이, 위치를 직접 설정하지 않고 자동으로 설정.  
Swift Charts 프레임워크는 막대 뿐만 아니라 막대의 길이가 무엇을 의미하는지 보여주는 `x`, `y`축의 막대 레이블도 자동으로 생성.

데이터가 여러개일 때 `Identifiable`한 데이터들을 `forEach`를 통해 그릴 수 있음.
```swift
struct Pancakes: Identifiable {
    let name: String
    let sales: Int
    
    var id: String { name }
}

let sales: [Pancakes] = [
    .init(name: "Cachapa", sales: 900),
    .init(name: "Injera", sales: 850),
    .init(name: "Crape", sales: 802)
]

struct StylesDetailsChart: View {
    var body: some View {
        Chart {
            Chart(sales) { element in
                BarMark(
                    x: .value("Name", element.name),
                    y: .value("Sales", element.sales)
                )
            }
        }
    }
}
```


`forEach`가 차트의 유일한 콘텐츠일 땐 `Chart 이니셜라이저`에 주입 가능.   
`Chart`는 `forEach`처럼 동작.
```swift
struct Pancakes: Identifiable {
    let name: String
    let sales: Int
    
    var id: String { name }
}

let sales: [Pancakes] = [
    .init(name: "Cachapa", sales: 900),
    .init(name: "Injera", sales: 850),
    .init(name: "Crape", sales: 802)
]

struct StylesDetailsChart: View {
    var body: some View {
        Chart(sales) { element in
            BarMark(
                x: .value("Name", element.name),
                y: .value("Sales", element.sales)
            )
        }
    }
}
```
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/a29f8cfb-df05-4cab-880b-3dac8e8a5f5e) 

`x`, `y`데이터를 바꿔주면 그에 맞춰서 적절한 축 스타일을 선택해서 그려줌
```swift
struct Pancakes: Identifiable {
    let name: String
    let sales: Int
    
    var id: String { name }
}

let sales: [Pancakes] = [
    .init(name: "Cachapa", sales: 900),
    .init(name: "Injera", sales: 850),
    .init(name: "Crape", sales: 802),
    .init(name: "Jian Bing", sales: 753),
    .init(name: "Dosa", sales: 654),
    .init(name: "American", sales: 618)
]

struct StylesDetailsChart: View {
    var body: some View {
        Chart(sales) { element in
            BarMark(
                x: .value("Sales", element.sales),
                y: .value("Name", element.name)
            )
        }
    }
}
```
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/79b96dce-e314-4e0d-9f6a-51f97cedd88f) 

### 다크모드, 디스플레이 크기, 방향 지원
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/6e1ba54a-11c4-4c87-8903-21d5909dbe20) 
![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/234b12a9-2397-4d53-9679-bf4486bf56b8) 


### 접근성 지원
```swift
struct StylesDetailsChart: View {
    var body: some View {
        Chart(sales) { element in
            BarMark(
                x: .value("Sales", element.sales),
                y: .value("Name", element.name)
            )
            .accessibilityLabel(element.name)
            .accessibilityValue("\(element.sales) sold")
        }
    }
}
```

### 2개 이상의 데이터 표시

2개 이상의 데이터를 한번에 표시하기.   
범례는 `foregroundStyle(by:)`로 표시해줌.
```swift
struct SalesSummary: Identifiable {
    let weekday: Date
    let sales: Int
    var id: Date { weekday }
}

struct Series: Identifiable {
    let city: String
    let sales: [SalesSummary]
    
    var id: String { city }
}

let seriesData: [Series] = [
    .init(city: "Cupertino", sales: cupertinoData),
    .init(city: "San Francisco", sales: sanfranciscoData)
]

struct LocationsDetailsChart: View {
    var body: some View {
        Chart(seriesData) { series in
            ForEach(series.sales) { element in
                BarMark(
                    x: .value("Day", element.weekday, unit: .day),
                    y: .value("Sales", element.sales))
            }
            .foregroundStyle(by: .value("City", series.city))
        }
    }
}
```
![2개 이상의 데이터 표시](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b583166a-5e4c-4208-9b68-12518b39fe69) 
위 처럼 그리면 누적되어 보여줌.
각각을 나누어 그룹화하려면 `.position(by:)` 수정자를 사용
```swift
struct LocationsDetailsChart: View {
    var body: some View {
        Chart(seriesData) { series in
            ForEach(series.sales) { element in
                BarMark(
                    x: .value("Day", element.weekday, unit: .day),
                    y: .value("Sales", element.sales))
            }
            .foregroundStyle(by: .value("City", series.city))
            .symbol(by: .value("City", series.city))
            .position(by: .value("City", series.city)) // 각각을 그룹화
        }
    }
}
```
![2개 이상의 데이터를 그룹화](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/b4eaf6bc-c1ed-4ec9-8dad-5e862cde7ac3) 


### PointMark, LineMark

`Mark` 유형을 `PointMark`나
![PointMark](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/f707ade5-50c6-45f0-91fe-5292c4f71679) 

`LineMark`로 변경만 하면 그에 맞게 Mark를 그려줌.
![LineMark](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/39ede602-2d8b-46ea-9da8-69090348a5df) 


두 `Mark` 유형을 동시에 그려줄 수도 있음.
```swift
struct LocationsDetailsChart: View {
    var body: some View {
        Chart(seriesData) { series in
            ForEach(series.sales) { element in
                PointMark(
                    x: .value("Day", element.weekday, unit: .day),
                    y: .value("Sales", element.sales))
            }
            .foregroundStyle(by: .value("City", series.city))
            
            ForEach(series.sales) { element in
                LineMark(
                    x: .value("Day", element.weekday, unit: .day),
                    y: .value("Sales", element.sales))
            }
            .foregroundStyle(by: .value("City", series.city))
        }
    }
}
```

![PointMark와 LineMark를 동시에](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/e7c05004-e03a-4d1b-8344-b2ab1962d83e) 

### symbol
`.symbol(by:)`를 통해 `PointMark`의 각 데이터들을 다른 기호로 표시시켜줌.   
기호가 있으면 색맹인 사람들이 더 차트를 잘 읽을 수 있음.

```swift
struct LocationsDetailsChart: View {
    var body: some View {
        Chart(seriesData) { series in
            ForEach(series.sales) { element in
                PointMark(
                    x: .value("Day", element.weekday, unit: .day),
                    y: .value("Sales", element.sales))
            }
            .foregroundStyle(by: .value("City", series.city))
            .symbol(by: .value("City", series.city)) // 기호로 구분
            
            ForEach(series.sales) { element in
                LineMark(
                    x: .value("Day", element.weekday, unit: .day),
                    y: .value("Sales", element.sales))
            }
            .foregroundStyle(by: .value("City", series.city))
        }
    }
}
```
![symbol(by:)를 사용해서 기호로 구분](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/acecbc60-4ce8-4142-a595-bd18e767ae03) 

선에 점을 표시하는 것이 일반적이기 때문에, `LineMark`에 `symbol(by:)` 수정자를 적용하면 Point가 함께 그려짐.
```swift
struct LocationsDetailsChart: View {
    var body: some View {
        Chart(seriesData) { series in
            ForEach(series.sales) { element in
                LineMark(
                    x: .value("Day", element.weekday, unit: .day),
                    y: .value("Sales", element.sales))
            }
            .foregroundStyle(by: .value("City", series.city))
            .symbol(by: .value("City", series.city))
        }
    }
}
```
![Image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4e7c3bfa-5632-4b45-a480-c95e45123e6d) 

### 보간법(interpolation)
`LineMark`의 경우 `interpolation`으로 더 매끄러운 선을 그려줄 수 있음.
```swift
struct LocationsDetailsChart: View {
    var body: some View {
        Chart(seriesData) { series in
            ForEach(series.sales) { element in
                LineMark(
                    x: .value("Day", element.weekday, unit: .day),
                    y: .value("Sales", element.sales))
            }
            .foregroundStyle(by: .value("City", series.city))
            .symbol(by: .value("City", series.city))
            .interpolationMethod(.catmullRom) // 보간법 적용
        }
    }
}
```
![Image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/4ce26fca-e9a0-4714-9aad-2b033c8451cc) 

### 최소값과 최대값의 시각화
`yStart`, `yEnd`를 통해 최소값과 최대값을 시각화할 수 있음.
```swift
struct MonthlySalesChart: View {
    var body: some View {
        Chart(SalesData.last12Months, id: \.month) {
            AreaMark(
                x: .value("Month", $0.month, unit: .month),
                yStart: .value("Daily Min", $0.dailyMin),   // 최소값
                yEnd: .value("Daily Max", $0.dailyMax)      // 최대값
            )
            .opacity(0.3)
            
            LineMark(x: .value("Month", $0.month, unit: .month),
                     y: .value("Daily Average", $0.dailyAverage))
        }
    }
}
```
![Image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/963a1abd-83d2-4c8c-a6ce-b9ec664f11d3) 


위에서 사용했던 `Barmark`, `LineMark`, `PointMark`와   
`x`, `y`, `foregroundStyle`, `symbol` 속성 외에도   

`area`, `rule`, `rectangle` Mark와   
`symbolSize`, `lineStyle` 속성 들도 있음.
![SCR-20240108-ghgn](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/13d3a4be-96b0-4c77-ba59-bda5c7e39294)

### RectangleMark, RuleMark
위 차트에서 AreaMark를 BarMark로 LineMark를 RectangleMark로 변경하고,   
RuleMark를 추가.
```swift
struct MonthlySalesChart: View {
    let averageValue = 137
    var body: some View {
        Chart {
            ForEach(SalesData.last12Months, id: \.month) {
                BarMark(
                    x: .value("Month", $0.month, unit: .month),
                    yStart: .value("Daily Min", $0.dailyMin),   // 최소값
                    yEnd: .value("Daily Max", $0.dailyMax)      // 최대값
                )
                
                RectangleMark(
                    x: .value("Month", $0.month, unit: .month),
                    y: .value("Daily Average", $0.dailyAverage),
                    height: 2
                )
            }
            .opacity(0.3)
            RuleMark(
                y: .value("Average", averageValue)
            )
            .lineStyle(StrokeStyle(lineWidth: 3))
        }
    }
}
```
![Image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d3f9ea31-6b65-4c48-92e9-4db5bc9100e1) 

### 축 값의 변경
아래와 같이 작성하면 x축의 월 표시가 모두 표시가 되지 않음.
```swift
struct MonthlySalesChart: View {
    let averageValue = 137
    var body: some View {
        Chart {
            ForEach(SalesData.last12Months, id: \.month) {
                BarMark(
                    x: .value("Month", $0.month, unit: .month),
                    y: .value("Sales", $0.sales)
                )
            }
        }
    }
}
```
![Image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8fe3c532-80d0-4669-a482-bcccc5e3f33b) 

`.chartXAxis` 수정자로 `AxisMark`를 일정간격(stride)으로 표시.
```swift
struct MonthlySalesChart: View {
    let averageValue = 137
    var body: some View {
        Chart {
            ForEach(SalesData.last12Months, id: \.month) {
                BarMark(
                    x: .value("Month", $0.month, unit: .month),
                    y: .value("Sales", $0.sales)
                )
            }
        }
        .chartXAxis {
            AxisMarks(values: .stride(by: .month))
        }
    }
}
```
![Image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/d1e7e559-7350-4af4-8217-578f21760c22) 

위에서는 각 항목이 너무 길어서 잘림. `AxisValueLabel`을 통해 format을 좁은 형식을 쓰도록 설정해줄 수 있음.
```swift
struct MonthlySalesChart: View {
    let averageValue = 137
    var body: some View {
        Chart {
            ForEach(SalesData.last12Months, id: \.month) {
                BarMark(
                    x: .value("Month", $0.month, unit: .month),
                    y: .value("Sales", $0.sales)
                )
            }
        }
        .chartXAxis {
            AxisMarks(values: .stride(by: .month)) { value in
                AxisGridLine()
                AxisTick()
                AxisValueLabel(format: .dateTime.month(.narrow))
            }
        }
    }
}
```
![Image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/832cbe25-f7e6-44d6-93a1-b3b3dbd84194) 

Y축의 위치를 앞쪽(`.leading`)으로 적용.
```swift
struct MonthlySalesChart: View {
    let averageValue = 137
    var body: some View {
        Chart {
            ForEach(SalesData.last12Months, id: \.month) {
                BarMark(
                    x: .value("Month", $0.month, unit: .month),
                    y: .value("Sales", $0.sales)
                )
            }
        }
        .chartXAxis {
            AxisMarks(values: .stride(by: .month)) { value in
                AxisGridLine()
                AxisTick()
                AxisValueLabel(format: .dateTime.month(.narrow))
            }
        }
        .chartYAxis {
            AxisMarks(position: .leading)
        }
    }
}
```
![Image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/5a20b023-dfb0-4d06-a8ef-7473d6b55446) 

경우에 따라서는 그래프의 모양, 개요만 보여주고, X, Y축의 정보는 보여주고 싶지 않을 수도 있음.   
이럴 떄는 hidden 처리가 가능.
```swift
struct MonthlySalesChart: View {
    let averageValue = 137
    var body: some View {
        Chart {
            ForEach(SalesData.last12Months, id: \.month) {
                BarMark(
                    x: .value("Month", $0.month, unit: .month),
                    y: .value("Sales", $0.sales)
                )
            }
        }
        .chartXAxis {
            AxisMarks(values: .stride(by: .month)) { value in
                AxisGridLine()
                AxisTick()
                AxisValueLabel(format: .dateTime.month(.narrow))
            }
        }
        .chartYAxis(.hidden)
        .chartXAxis(.hidden)
    }
}
```
![Image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/71d22635-b65d-4c2f-827a-9dfd6c793451) 

### chartPlotStyle
플롯 영역에 정확한 크기나 종횡비를 설정하고 싶을 때는 `.chartPlotStyle` 사용.  
`.background`로 색상을 주거나, `.border`로 테두리 설정을 할 수도 있음.
```swift
struct StylesOverviewChart: View {
    let numberOfCategories = TopStyleData.last30Days.count
    
    var body: some View {
        Chart(TopStyleData.last30Days, id: \.name) { element in
            BarMark(x: .value("sales", element.sales),
                    y: .value("name", element.name))
        }
        .foregroundColor(.pink)
        .chartPlotStyle { plotArea in
            plotArea
                .frame(height: 60.0 * CGFloat(numberOfCategories))
                .background(.pink.opacity(0.2))
                .border(.pink, width: 1)
        }
    }
}
```
![Image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3fa2d474-e866-40d2-8070-dfa1975cbe61) 

