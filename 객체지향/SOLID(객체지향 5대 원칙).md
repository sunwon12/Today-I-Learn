**Single Responsibilty Principle(단일 책임 원칙)**

---

**오직 하나의 책임만을 가져야 함을 의미합니다.** 그 이유로는 코드 변경의 기준이 책임이기 때문이고, 책임 분리가 이루어질 시, 다른 기능의 변경에 영향을 받지 않기 때문입니다.

**Open-Closed Principle**(개방 폐쇄 원칙)

---

**확장에는 열려있고, 변경에는 닫혀 있어야 함을 의미합니다**. 즉, 새로운 타입, 메소드를 추가하였을 때에는 용이하고 기존 코드에서는 변경이 최소화되어야 합니다. 여기서 중요한 것은 모든 것에 변경을 예측하여 필요하지 않은 확장성을 대비해 과도한 추상화를 도입할 경우, 코드 복잡성만 증가할 수 있기 때문에 You Aren't Gonna Need It (YAGNI) 원칙에 따라 필요할 때만 확장성을 고려하는 것이 중요합니다.

**Liskov Substitution Principle**(리스코프 치환 원칙)

---

서브 타입은 언제나 상위 타입으로 교체 될 수 있어야 함을 의미합니다. 즉, **모든 하위타입**이 **상위타입의 기대 동작을 준수해야 합니다.**

```java
//나쁜 예
public class Bird {
    public void fly() {
        System.out.println("Flying");
    }
}

public class Penguin extends Bird {
    
    // 펭귄은 날 수 없기 때문에 서브 타입이 상위 타입으로 교체될 수 없습니다.
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins can't fly!");
    }
}
```

**Interface Segregation Principle(인터페이스 분리 원칙)**

---

사용하지 않는 인터페이스는 분리해야 함을 의미합니다. 사용하지 않지만 의존성을 가지고 있다면 해당 인터페이스가 변경되는 경우 영향을 받기 때문입니다.

**Dependency Inversion Principle(의존성 역전 원칙)**

---

위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안 되며 추상화에 의존해야 함을 의미합니다.
