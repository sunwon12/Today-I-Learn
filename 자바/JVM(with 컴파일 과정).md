
## 1. 자바

---

**자바는 OS에 독립적인 특성을 가지고 있습니다.** 그게 가능한 이유는 JVM(Java Virtual Machine) 때문입니다. JVM의 어떤 기능 때문에 OS에 독립적으로 실행시킬 수 있는지 자바 컴파일 과정을 통해 알아보겠습니다.

## 2. 컴파일 과정

---

1. 자바 소스코드가(.java) 자바 컴파일러(javac)를 통해 자바 바이트 코드로 변환됩니다.(.class)
2. 컴파일된 바이트코드(.class)를 JVM의 클래스로더(Class Loader)에게 전달합니다.
3. 클래스 로더는 동적로딩(Dynamic Loading)을 통해 필요한 클래스들을 로딩 및 링크하여 **런타임 데이터 영역**(Runtime Data Area의 Method Area), 즉 JVM의 메모리에 올립니다.
    1. 메모리에 올리는 과정은 총 3가지 과정을 거칩니다. **로딩 과정**에서 메서드 영역에 클래스를 저장하고, **링크 과정**에서 구성 요소를 검증하고 메모리를 할당한 후에, **초기화 과정**에서 클래스 변수들을 적절한 값으로 초기화합니다.
4. 실행 엔진(Execution Engine)은 JVM 메모리에 올라온 바이트 코드들을 가져와서 실행합니다. 이 때 실행 엔진에서 기계어로 변환하는 방식에는 두 가지가 존재합니다

![image](https://github.com/user-attachments/assets/c4178b75-b501-4b1c-9c22-7851344f9240)

### 2.1 기계어 변화 두가지 방식

---

클래스 실행엔진이 JVM 메모리에 올라온 바이트 코드들을 가져와서 기계어로 변환하는데, 두 가지 방식이 있다고 하였습니다. 두 가지 방식으로는 인터프리터와 JIT 컴파일러가 있습니다. 

1. **인터프리터**
    - 바이트 코드 명령어를 **하나씩** 읽어서 해석하고 실행합니다.
    - 하나하나의 실행은 빠르나, 전체적인 실행 속도가 느리다는 단점이 있습니다
2. **JIT 컴파일러 (Just-In Time Compiler)**
    - 인터프리터의 단점을 보완하기 위해 도입된 방식입니다.
    - 바이트 코드 **전체를** **컴파일**하여 바이너리 코드로 변경하고 이후에는 해당 메서드를 더이상 인터프리팅 하지 않고, 바이너리 코드로 직접 실행하는 방식입니다.
    - 하나씩 인터프리팅하여 실행하는 것이 아니라 바이트 코드 전체가 컴파일 된 바이너리 코드를 실행하는 것이기 때문에 전체적인 실행속도는 인터프리팅 방식보다 빠릅니다
    - 그러나 인터프리팅 방식보다는 훨씬 오래 걸리므로 한번만 실행하면 되는 코드는 인터프리팅하는 것이 유리합니다

### 2.2 🤔 그럼 인터프리터, JIT 컴파일러는 각각 언제 쓰일까요?

---

- 처음에는 인터프리터로 실행
- JVM이 호출 빈도 모니터링
- 임계값 도달 시 JIT 컴파일 수행
- 이후 해당 코드는 컴파일된 버전으로 실행

## 3. JVM 구조

---

앞서 컴파일 되는 과정을 통해 JVM 구조를 맛보기 알아보았습니다. 지금부터는 JVM 구조를 자세히 알아보겠습니다. (컴파일 과정에서 클래스 로더와 실행엔진을 알아봤으니 둘은 제외하겠습니다.)

![image](https://github.com/user-attachments/assets/2c0d3434-60cc-4caf-b03a-47a84bdf987b)

### 3.1 가비지 컬렉터(Garbage Collector, GC)

---

가비지 컬렉터(GC)는 JVM의 **Heap 영역**에서 사용하지 않는 객체를 자동으로 회수하여 메모리를 관리합니다. 개발자는 명시적으로 메모리를 해제하지 않아도 되며, GC가 이를 처리합니다.

단, Full GC가 발생하는 경우에는 모든 스레드를 중지 시킵니다.

### 3.2 Runtime Data Area

---

런타임 데이터 영역은 JVM 메모리 영역으로 자바 애플리케션을 실행할 때 사용되는 데이터들을 적재하는 공간입니다. 런타임 데이터 영역은 Method Area, Heap Area, Stack Area, PC register, Native Method Stack으로 나뉘어집니다.  

이 때 Method Area, Heap Area는 모든 쓰레드가 공유하는 영역이고, 나머지 Stack Area, PC Register, Native Method Stack은 각 쓰레드마다 생성되는 개별 영역입니다.

![image](https://github.com/user-attachments/assets/f3067a77-0365-46fd-a035-7e76c089ae4d)

### 3.2.1 Method Area(Static Area)

---

- 클래스와 메소드, static변수와 상수(final) 정보 등이 저장되는 공간입니다
- JVM이 종료 시(프로그램이 종료 시) 메모리에서 해제 된다. 즉 프로그램이 종료되기 전까진 메모리 상에 존재합니다.

### 3.2.2 Heap Area

---

- new 명령어를 통해 생성한 인스턴스와 배열 등의 참조형 변수정보가 저장되는 공간입니다
- 물론 Method Area에 올라온 클래스들만 생성이 가능합니다
- GC(Garbage Collection)의 대상이 됩니다.

![image](https://github.com/user-attachments/assets/77479f83-292d-450c-93de-20e112215251)

이렇게 다섯가지 영역(Eden, survivor 0, survivor 1, Old, Permanent)으로 나뉜 힙 영역은 다시 물리적으로Young Generation 과Old Generation영역으로 구분되게 되는데 다음과 같습니다.

- **Young Generation** : 생명 주기가 짧은 객체를 GC 대상으로 하는 영역.
    - **Eden** : new를 통해 새로 생성된 객체가 위치. 정기적인 쓰레기 수집 후 살아남은 객체들은 Survivor로 이동
    - **Survivor 0 / Survivor 1** : 각 영역이 채워지게 되면, 살아남은 객체는 비워진 Survivor로 순차적으로 이동
- **Old Generation** : 생명 주기가 긴 객체를 GC 대상으로 하는 영역. Youn Generation에서 마지막까지 살아남은 객체가 이동

### 3.2.3 Stack Area

---

- 클래스 내의 메소드에서 사용되는 정보들이 저장되는 공간입니다
- 매개변수, 지역변수, 리턴값 등이 저장되며 LIFO(Last In First Out) 방식으로 메소드 실행 시 저장되었다가 실행이 완료되면 제거됩니다
- 임시 저장공간으로 생각하면 됩니다

member라는 변수는 Stack에 저장되고 객체는 Heap에 저장됩니다. 각 쓰레드가 값을 수정할 때는 자신이 가지고 있는 Stack에 있는 member 변수로 Heap에 있는 객체를 수정하는 것입니다. 만약 A메소드가 끝나서 메소드 내에 있는 지역 변수들이 사라질 때, heap 영역에 있는 값들은 참조가 끊기니 GC의 대상이 되는 것입니다.(해당 객체가 A메소드에서만 참조한다고 가정했을 때)

![image](https://github.com/user-attachments/assets/d8c47a4b-aa2d-4634-9d4b-5080a87b2eaf)

단, 데이터 타입에 따라 데이터를 저장하는 방식이 다릅니다. 기본(원시)타입의 변수는 stack이 가지고, 참조 타입의 변수는 Heap이나 Method(Static)에 객체가 저장됩니다.

![image](https://github.com/user-attachments/assets/9d750bad-8209-4a0d-89b5-1123f9f2bb59)

### 3.2.4 PC register

---

PC register는 현재 실행중인 JVM 명령어 주소값이 저장되는 공간입니다. 책갈피 같은 역할을 하는 공간입니다.  Stack과 비슷하다고 느낄 수도 있습니다. Stack은  main() → methodA() → methodB() 와 같은 정보를 기억하는 것이고 PC Register는 정확한 정보를 기억합니다. 같은 메소드 내에서 몇 번째 줄을 실행 중이었는지를 기억합니다.

- Stack: "method()"가 호출됐다는 정보
- PC Register: "현재 0x1235 주소의 명령어 실행 중"

즉, Stack은 메소드 호출 관계를, PC Register는 현재 실행 중인 정확한 명령어 위치를 추적합니다.

```java
public void method() {
    int a = 1;     // 바이트코드 주소 0x1234
    int b = 2;     // 바이트코드 주소 0x1235
    int c = a + b; // 바이트코드 주소 0x1236
}
```

### 3.2.5 Native Method Stack

---

- 자바 외 다른 언어의 호출을 위해 할당되는 영역입니다
- 자바에서 C/C++의 메소드를 호출할 때 사용하는 Stack 영역이라고 생각하면 됩니다
