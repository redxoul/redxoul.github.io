---
title: (Algorithm) 유클리드 호제법(Euclidean algorithm) - 최대공약수GDC를 구하는 알고리즘
author: Cody
date: 2022-01-26 00:34:00 +0800
categories: [Coding test]
tags: [iOS, Algorithm]
pin: false
mermaid: true
---

2개의 자연수의 최대공약수(Greatest Common Divisor. GCD)를 구하는 알고리즘.  
`두개의 자연수 a, b. a>b, a%b=r 일 때,
GCD(a, b) = GCD(b, r)과 같고 r이 0이 되었을 때 그 때 b가 최대공약수이다.`
라는 원리를 이용한 알고리즘.

> ex) GCD(24, 16) → GCD(16, 8) → GCD(8, 0) → 최대공약수 = 8
{: .prompt-info }

### 재귀함수로 구현

```swift
func GCD(_ a: Int, _ b: Int) -> Int {
        if b == 0 {
            return a
        }
        return GCD(b, a % b)
}
```

### 반복문으로 구현

```swift
func GCD(_ a: Int, _ b: Int)
{
    while(b != 0){
        var r = a % b
        a = b;
        b = r;
    }
}
```

### 다항식일 경우

a, b, c가 있을 때 a, b의 최대공약수(x)를 먼저 구하고, c, x의 최대공약수를 다시 구하면 됨.

```swift
func GCD3(_ a: Int, _ b: Int, _ c: Int) -> Int {
    return GCD(GCD(a, b), c)
}
```