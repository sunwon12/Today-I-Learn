## **1. @Transactional이란**

---

`@Transactional`은 Spring에서 제공하는 선언적 트랜잭션 관리 방식입니다. 이 어노테이션 하나로 다음과 같은 복잡한 트랜잭션 처리 코드들을 자동으로 처리할 수 있습니다:

- 데이터베이스 커넥션 획득
- 트랜잭션 시작 (setAutoCommit(false))
    
    (비즈니스 로직 실행)
    
- 정상 완료시 커밋 또는 예외 발생시 롤백
- 커넥션 반환

## **2. @Transactional 동작 원리**

---

@Transactional의 동작은 Spring AOP를 기반으로 합니다.

프록시 객체는 다음과 같은 트랜잭션 관련 코드를 자동으로 삽입합니다:

### **1) 메서드 호출 전**

- 데이터베이스 연결 (`Connection` 객체 생성).
- 트랜잭션 시작 (`setAutoCommit(false)`).
- 트랜잭션 동기화 (현재 트랜잭션 정보를 ThreadLocal에 저장).

### **2) 메서드 실행 중**

- 타겟 객체의 메서드를 호출하여 실제 비즈니스 로직 수행.

### **3) 메서드 호출 후**

- 메서드가 성공적으로 실행되면 **트랜잭션 커밋** (`commit`).
- 예외가 발생하면 **트랜잭션 롤백** (`rollback`).
- 트랜잭션 동기화 해제 (ThreadLocal에서 트랜잭션 정보 제거).

 

크게 다음 두 가지 프록시 방식으로 구현됩니다:

![image](https://github.com/user-attachments/assets/55554381-ee03-4f0b-ac43-83a6eeca3e9d)

### 2.1 JDK Dynamic Proxy

---

```java
Object proxy = Proxy.newProxyInstance(
   ClassLoader, 
   Class<?>[],
   InvocationHandler
);
```

- JDK Proxy는 반드시 **인터페이스를 구현한 클래스**만 프록시를 생성할 수 있습니다.
    - 인터페이스가 반드시 있어야 함
- java.lang.reflect.Proxy를 사용하여 프록시 객체 생성
    - Reflection을 사용하기 때문에 성능이 떨어지는 단점

### 2.2 CGLIB Proxy

---

```java
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(Target.class); 
enhancer.setCallback(MethodInterceptor);

Object proxy = enchancer.create(); // proxy 생성
```

- **JDK Proxy의 제약을 극복**하고 **유연한 AOP와 빈 관리를 제공**하기 위함
- **바이트코드 조작 라이브러리**로, 동적으로 클래스를 상속받아 프록시 객체를 생성합니다.
- 클래스를 상속받아 프록시 객체 생성
- Spring Boot 2 이후의 기본 프록시 방식
- **final 메서드**는 오버라이딩할 수 없으므로, 프록시 적용이 불가능합니다.

`Enhander 의존성 추가`, `default 생성자 필요`, `target의 생성자 2번 호출`의 단점으로 인해 **Spring에서는 CGLIB를 권장하지 않았습니다.**

- 그러나 `Spring 3.2`버전부터 Spring Core 패키지에 `cglib가 포함`되었고,
- `Spring 4.0`부터  Objensis 라이브러리를 통해 CGLIB의 단점 개선하였습니다.
    - **Default 생성자 없이도 Proxy 생성 가능**
    - 생성자 호출 2번 문제 해결

+) 

Spring Boot 2.x는 Spring Framework 5.x를 기반으로 동작.

Spring Boot 3.x는 Spring Framework 6.x를 기반으로 동작.

### 2.3 ByteBuddy

---

스프링 4.3 이후, 스프링은 CGLIB 대신 **ByteBuddy**를 기본적으로 사용하기 시작했습니다.

- **ByteBuddy**는 CGLIB보다 성능과 유지보수성 측면에서 우수합니다.
    - **성능**: 프록시 생성 속도, 메모리 사용량, 클래스 로딩 효율성 등에서 ByteBuddy가 더 뛰어남.
    - **유지보수성**: ByteBuddy는 현재도 활발히 개발 및 개선되고 있지만, CGLIB는 유지보수가 사실상 중단된 상태입니다.

CGLIB은 여전히 **Spring Core에 포함되어 있으며**, 특별한 이유가 있거나 레거시 코드와 호환이 필요한 경우 사용할 수 있습니다.

## 3. 트랜잭션 처리 과정

---

트랜잭션 처리는 다음과 같은 순서로 진행됩니다:

![image](https://github.com/user-attachments/assets/3006ff0f-851c-4af8-a1b9-d20d6194cbb4)

1. **Caller (호출자)**
    - 서비스나 비즈니스 로직을 호출하는 클라이언트가 **프록시 객체**를 호출합니다.
    - **직접 타겟 메서드를 호출하지 않고** AOP Proxy를 호출하게 됩니다.
2. **AOP Proxy (프록시 객체)**
    - `@Transactional`이 적용된 메서드가 호출되면 스프링이 생성한 프록시 객체가 실행됩니다.
    - 이 프록시는 트랜잭션 관련 로직을 수행하기 위해 **Transaction Advisor**로 제어를 넘깁니다.
3. **Transaction Advisor (트랜잭션 어드바이저/ TransactionInterceptor)**
    - 프록시는 **Advisor**(어드바이저)를 통해 **Advice**(로직)를 실행하는데, 트랜잭션 처리의 경우 **TransactionInterceptor**가 이 Advice를 담당합니다.
    - 트랜잭션 관리 로직이 실행됩니다.
        - **트랜잭션 시작**: 데이터베이스 연결을 열고 `setAutoCommit(false)` 설정.
        - 트랜잭션 컨텍스트를 설정한 뒤 다음 단계로 넘어갑니다.
4. **Custom Advisor(s) (사용자 정의 어드바이저)**
    - 트랜잭션과 상관없이 **커스텀 인터셉터**들이 실행될 수 있습니다.
    - 사용자가 추가한 AOP 로직이 트랜잭션 실행 전/후에 실행될 수 있습니다.
5. **Target Method (비즈니스 로직 실행)**
    - 프록시 객체는 최종적으로 타겟 메서드(비즈니스 로직)를 호출합니다.
    - 여기서 **비즈니스 로직**이 수행됩니다.
6. **Transaction Commit/Rollback (트랜잭션 종료)**
    - 비즈니스 로직 수행이 끝나면 **Transaction Advisor**로 다시 돌아갑니다.
        - 예외가 없으면 **commit** 실행.
        - 예외가 발생하면 **rollback** 실행.
    - 트랜잭션이 종료되고 리소스를 정리합니다.
7. **제어 반환 (Control Flow Back)**
    - 프록시 객체를 통해 원래의 호출자(Caller)에게 결과를 반환합니다.
    - 호출된 로직의 결과값이나 예외가 호출자에게 전달됩니다.

## 4. 주요 속성

---

`@Transactional`은 다양한 속성을 제공합니다:

- propagation: 트랜잭션 전파 방식 설정
- isolation: 트랜잭션 격리 수준 설정
- readOnly: 읽기 전용 트랜잭션 여부
- timeout: 트랜잭션 타임아웃 설정
- rollbackFor: 특정 예외 발생 시 롤백

### **4-1. 트랜잭션 전파 방식 (Propagation)**

---

**트랜잭션 전파 방식**은 트랜잭션이 호출되는 메서드 간에 **트랜잭션을 어떻게 관리할지**를 설정하는 옵션입니다.

1. **`REQUIRED`**
    - **기존 트랜잭션이 있으면 그 트랜잭션에 참여하고**, 없으면 새로 생성합니다.
    - **기본값**이며 가장 많이 사용됩니다.
2. **`REQUIRES_NEW`**
    - 기존 트랜잭션이 있더라도 **항상 새로운 트랜잭션**을 생성합니다.
    - 기존 트랜잭션은 **일시 정지**됩니다.
3. **`NESTED`**
    - **기존 트랜잭션 내에서 중첩된 트랜잭션**을 생성합니다.
    - **SAVEPOINT**를 생성해 롤백 시 중첩 트랜잭션의 부분만 롤백합니다.
    - **JDBC 드라이버와 데이터베이스가 SAVEPOINT를 지원해야 사용 가능합니다.**
4. **`MANDATORY`**
    - 반드시 기존 트랜잭션이 있어야 합니다.
    - 트랜잭션이 없으면 예외를 발생시킵니다.
5. **`SUPPORTS`**
    - 트랜잭션이 있으면 참여하고, 없으면 트랜잭션 없이 실행합니다.
    - **선택적 트랜잭션**입니다.
6. **`NOT_SUPPORTED`**
    - 기존 트랜잭션이 있으면 **일시 정지**하고 트랜잭션 없이 실행합니다.
    - 트랜잭션이 필요 없는 로직에 사용합니다.
7. **`NEVER`**
    - **트랜잭션 없이 실행해야만 합니다.**
    - 기존 트랜잭션이 있으면 예외를 발생시킵니다.

### **4-2. 트랜잭션 격리 수준 (Isolation Level)**

---

**트랜잭션 격리 수준**은 여러 트랜잭션이 동시에 실행될 때 데이터의 **일관성과 무결성**을 보장하기 위해 설정합니다.

- **격리 수준이 높을수록 데이터 일관성은 좋아지지만 성능은 저하**됩니다.

1. **`DEFAULT`**
    - 데이터베이스의 기본 격리 수준을 따릅니다.
    - 대부분의 DBMS는 **READ_COMMITTED**가 기본입니다.
2. **`READ_UNCOMMITTED`**
    - 다른 트랜잭션에서 커밋되지 않은 변경사항도 읽을 수 있습니다.
    - **Dirty Read** 발생 가능
    - 성능은 가장 좋지만 데이터 일관성은 낮습니다.
3. **`READ_COMMITTED`** (기본값)
    - 다른 트랜잭션에서 **커밋된 데이터만** 읽을 수 있습니다.
    - **Dirty Read**는 방지되지만 **Non-Repeatable Read**는 발생할 수 있습니다.
4. **`REPEATABLE_READ`**
    - 트랜잭션 동안 **같은 데이터를 읽으면 항상 동일한 결과를 보장**합니다.
    - **Dirty Read**와 **Non-Repeatable Read** 방지
    - 하지만 **Phantom Read**는 발생할 수 있습니다.
5. **`SERIALIZABLE`**
    - **가장 높은 격리 수준**으로 트랜잭션을 순차적으로 실행하는 것처럼 동작합니다.
    - **Dirty Read**, **Non-Repeatable Read**, **Phantom Read**를 모두 방지합니다.
    - 성능 저하가 심할 수 있습니다.

## 5. 사용 시 주의사항

---

1. private 메서드에는 적용되지 않습니다
2. 같은 클래스 내부 호출에는 프록시가 적용되지 않습니다

```java
@Service
public class MyService {

	  public void selfInvokeMethod() {
        transactionalMethod(); // 프록시를 거치지 않고 직접(this) 호출
    }
    
    @Transactional
    public void transactionalMethod() {
        System.out.println("Transaction started");
    }
}

```

1. RuntimeException과 Error만 기본적으로 롤백됩니다

# 글을 다 쓴 후 의문점

---

- 스프링에서 AOP 어떻게 활용하는지?

참고

---

https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html
