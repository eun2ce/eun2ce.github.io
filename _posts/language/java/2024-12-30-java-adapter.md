---
title: "[java] `adapter` 패턴에 대해"
date: 2024-12-30 15:16:00 +0900
categories: [ "language", "java" ]
tags: [ "adapter Pattern", "constant pool", "java", "프로그래밍" ]
pin: false
math: false
mermaid: false
---

`adapter`라고하면 일반적으로 변환을 핵심으로 한다.
**어댑터 패턴(Adapter Pattern)**은 두 개의 호환되지 않는 인터페이스를 연결하는 구조적 디자인 패턴을 말한다.

이 패턴을 사용하면, 서로 다른 인터페이스를 가진 시스템들이 서로 통신할 수 있게 되며, 기존 코드를 수정하지 않고도 새로운 기능을 추가하거나 외부 시스템과 호환할 수 있게 됩니다.


## 예시

```java
// Target 인터페이스
interface ElectronicDevice {
    void turnOn();
}

// Adapter: 기존 시스템의 클래스
class OldDevice {
    public void powerOn() {
        System.out.println("기존 전자기기가 켜집니다.");
    }
}

// Adapter: OldDevice 를 ElectronicDevice 인터페이스로 변환
class DeviceAdapter implements ElectronicDevice {
    private OldDevice oldDevice;

    public DeviceAdapter(OldDevice oldDevice) {
        this.oldDevice = oldDevice;
    }

    @Override
    public void turnOn() {
        // 기존의 powerOn 메서드를 호출
        oldDevice.powerOn();
    }
}

public class AdapterPatternExample {
    public static void main(String[] args) {
        // 기존 시스템의 객체
        OldDevice oldDevice = new OldDevice();

        // 어댑터를 사용하여 기존 시스템의 객체를 호환시킴
        ElectronicDevice device = new DeviceAdapter(oldDevice);
        
        // 클라이언트는 Target 인터페이스만 사용
        device.turnOn();
    }
}
```

### 어댑터 패턴의 장점

`OldDevice`는 기존 시스템 클래스이고, D`eviceAdapter`는 `OldDevice`의 `powerOn` 메서드를 turnOn 메서드로 변환하여 `ElectronicDevice` 인터페이스와 호환
클라이언트는 `ElectronicDevice`만 알면 되며, `OldDevice`의 구현을 몰라도 된다.

* 기존 코드 수정 없이 확장: 새로운 시스템을 기존 시스템과 연결하려면 어댑터만 추가
* 유연한 시스템 통합: 서로 다른 인터페이스를 가진 시스템을 쉽게 통합

