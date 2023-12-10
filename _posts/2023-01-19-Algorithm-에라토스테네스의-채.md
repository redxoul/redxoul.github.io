---
title: (Algorithm) 에라토스테네스의 채(Sieve of Eratosthenes)
author: Cody
date: 2023-01-19 00:34:00 +0800
categories: [Coding test]
tags: [iOS, Algorithm]
pin: false
mermaid: true
---
1에서부터 N까지 존재하는 소수를 찾는 알고리즘입니다.

고대 그리스 수학자 에라토스테네스가 발견하였으며,

구하는 방법이 마치 채에 거르는 것과 같아 `'에라토스테네스의 채'`라고 불립니다.

예를 들어 1~100사이의 소수를 찾는다고 하면,

1. 1은 합성수가 아니므로 지워줍니다.
2. 2부터 시작해서 자신을 제외한 2의 배수를 모두 지워줍니다.
3. 3부터 시작해서 자신을 제외한 3의 배수를 모두 지워줍니다.
4. (4는 2단계에서 지워졌으므로)5부터 시작해서 자신을 제외한 5의 배수를 모두 지워줍니다.
5. (6은 2단계에서 지워졌으므로)7부터 시작해서 자신을 제외한 7의 배수를 모두 지워줍니다.
6. 위 과정을 계속 반복합니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/fe335d76-1233-4461-99be-27aeda9aafd3)

코드로 작성하면 아래와 같습니다.

```swift
// '에라토스테네스의 채'로 1~N범위의 prime을 찾기
public func sieveOfEratosthenes(_ N: Int) -> [Int] {
    var isPrime = Array(repeating: true, count: N+1) // N의 범위만큼의 배열
    isPrime[0] = false // 0, 1은 소수 아님
    isPrime[1] = false
    var prime: [Int] = []

    for i in 2...N {
        if isPrime[i] {
            prime.append(i)
            for j in stride(from: i*i, through: N, by: i) {
                isPrime[j] = false
            }
        }
    }

    return prime
}
```