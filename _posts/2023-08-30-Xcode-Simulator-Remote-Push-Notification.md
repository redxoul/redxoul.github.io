---
title: Xcode 시뮬레이터로 Remote Push Notification 테스트하기
author: Cody
date: 2023-08-30 00:34:00 +0800
categories: [iOS, iOS Dev]
tags: [Xcode, Simulator, Push Notification]
pin: false
mermaid: true
---
**Xcode 11.4**부터 시뮬레이터로 **Push Notification**을 보낼 수 있습니다.

## 준비물: apns 파일

아래와 같이 payload 내용이 담긴 `apns파일`을 준비합니다.

파일이름은 `payload.apns` 정도로 만들면 됩니다.

```shell
{
    "aps" : {
        "alert" : {
            "title" : "시뮬레이터로 Push Notification 보내기",
            "body" : "apns 파일로 간단하게"
        },
        "badge" : 5
    }
}
```

## 커맨드라인 입력

그리고 터미널을 열고, 커맨드라인을 입력하는데 3가지 정보가 필요합니다.

- 시뮬레이터 `타겟`
- 타겟 시뮬레이터에 설치된 앱 `BundleID`
- `apns 파일` 경로

```shell
xcrun simctl push (시뮬레이터 타겟) (시뮬레이터에 설치된 앱 BundleID) (apns 파일명)
```

현재 시뮬레이터가 하나가 실행중일 때는 아래처럼 `booted`를 타겟으로 커맨드를 입력합니다.

아래는 예시입니다.

```shell
xcrun simctl push booted com.swiftyCody.AppStoreSearch payload.apns
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/7668b577-05b9-4ac5-9db0-969158f64eb7)


## 시뮬레이터 타겟 찾기

**실행중인 시뮬레이터가 2개 이상**일 때는 `booted`가 아닌 **시뮬레이터 타겟**을 명시해주어야 합니다.

커맨드라인에 아래 커맨드를 실행하면,

```applescript
xcrun simctl list
```

아래처럼 실행중인 시뮬레이터 디바이스 정보가 나옵니다.

```shell
== Devices ==
-- iOS 16.4 --
    iPhone SE (3rd generation) (F00DEBC9-5252-4010-B0D6-2B348B6C2063) (Booted)
    iPhone 14 (0E85F20C-5F7D-41A6-A3E2-9815E7D0BE3C) (Shutdown)
    iPhone 14 Plus (F6F64B76-32D4-48E0-AE9E-DC2CD92DD441) (Shutdown)
    iPhone 14 Pro (FC4CB706-28A1-43D9-9C79-4957C49F4056) (Booted)
    iPhone 14 Pro Max (2F79B115-7394-4682-BFB3-6D6DBF155D58) (Shutdown)
    iPad Air (5th generation) (9550FD0D-F397-4A1E-9A25-9FBA528CBC29) (Shutdown)
    iPad (10th generation) (53276EB1-3889-4D5F-80CD-FF2E11CC910A) (Shutdown)
    iPad mini (6th generation) (5336723F-2978-4D41-A448-4B18A760506C) (Shutdown)
    iPad Pro (11-inch) (4th generation) (204F1DB1-E707-498D-B0D5-49F0A7823B9F) (Shutdown)
    iPad Pro (12.9-inch) (6th generation) (24BB75DA-DD3A-4A5A-9268-07A68A0B672B) (Shutdown)
```

실행중인 시뮬레이터 옆엔 `(Booted)`라는 정보가 나옵니다. 그 중 푸시를 보낼 디바이스의 `deviceID`를 넣어서 보내면 됩니다.

```shell
xcrun simctl push FC4CB706-28A1-43D9-9C79-4957C49F4056 com.swiftyCody.AppStoreSearch payload.apns
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/3597724a-2f5d-4126-a1fb-af5ab1e347dc)


아래처럼 `Simulator Target Bundle`을 `apns파일`에 포함시키면

```shell
{
    "aps" : {
        "alert" : {
            "title" : "시뮬레이터로 Push Notification 보내기",
            "body" : "apns 파일로 간단하게. 타겟 BundleID를 추가해서 보내기."
        },
        "badge" : 5
    },
    "Simulator Target Bundle": "com.swiftyCody.AppStoreSearch"
}
```

아래처럼 타겟 `BundleID`를 생략해도 푸시가 됩니다.

```angelscript
xcrun simctl push FC4CB706-28A1-43D9-9C79-4957C49F4056 payload.apns
```

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/8e69e90b-ee21-4bc1-ae5c-721cf44b1848)


## Drag & Drop

`Simulator Target Bundle`을 포함한 `apns파일`은 해당 시뮬레이터로 `Drag & Drop`으로 넣어도 푸시가 됩니다.

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/adf96853-e904-4b49-ac79-9d0e4560adb4)

![image](https://github.com/swiftycody/swiftycody.github.io/assets/9062513/ce48fa98-ba04-4652-b65e-9c82c60d5da8)
