<img width="327" alt="image" src="https://github.com/user-attachments/assets/3e977c79-c72d-48a9-bfd2-b86139ad3f68" />

# 1. AOP(Aspect-Oriented Programming)

---

AOP는 관점 지향 프로그래밍을 뜻합니다. 애플리케이션의 공통 관심사(로깅, 트랜잭션 관리, 보안, 성능 모니터링 등…)를 분리하여 모듈화하는 것입니다.

이로 인해 클래스들은  SRP(Single Reponsibility Principle)에 부합하게 하나의 책임만을 가질 수 있습니다.

## 1.2 AOP의 주요 개념

---

- **Aspect**: 공통 관심사를 캡슐화한 모듈입니다. 예를 들어, 트랜잭션 관리 Aspect, 로깅 Aspect 등.
- **Advice**: Join Point에서 실행될 구체적인 동작입니다. 예: "메서드 실행 전에 로깅하라".
    - **Join Point**: Aspect를 적용할 수 있는 실행 지점입니다. 메서드 호출, 객체 생성 등.
    - **Pointcut**: Advice가 적용될 Join Point를 정의하는 표현식입니다.
- **Target**: Advice가 적용될 대상
- **Weaving**: Aspect 를 Target에 연결시켜 AOP 객체로 만드는 바이트 코드 및 객체 조작과정

# 2. **Spring AOP 동작 원리**

---

스프링은 AOP는 프록시 방식으로 동작합니다. 객체를 직접적으로 참조하는 것이 아니라, 해당 객체를 대행(proxy)하는 객체를 통해 대상 객체에 접근하는 방식을 말합니다.

![image](https://github.com/user-attachments/assets/1236ab2a-d337-4827-95aa-d1942d6a9dbb)

 

크게 다음 두 가지 프록시 방식으로 구현됩니다:

![image](https://github.com/user-attachments/assets/ef1a9d42-aff5-45b1-ad70-15e70afaefed)

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

자세한 내용은 [링크](https://gmoon92.github.io/spring/aop/2019/02/23/spring-aop-proxy-bean.html) 참조

### 2.2 CGLIB Proxy

---

```java
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(Target.class); 
enhancer.setCallback(MethodInterceptor);

Object proxy = enchancer.create(); // proxy 생성
```

- **JDK Proxy의 제약을 극복**하고 **유연한 AOP와 빈 관리를 제공**하기 위함
- 바이트코드 조작 라이브러리를 활용하여 **런타임 시점**에 대상 클래스를 상속받아 동적으로 **프록시 객체**를 생성합니다.
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

# 3. 스프링이 제공하는 AOP 사용법

---

1. **기본적인 Aspect 작성 예시**

```java
@Aspect
@Component
public class LoggingAspect {
    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    
    // 모든 서비스 클래스의 모든 메서드에 적용
    @Around("execution(* com.example.service.*.*(..))")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        
        Object result = joinPoint.proceed();
        
        long executionTime = System.currentTimeMillis() - start;
        logger.info("{} executed in {} ms", joinPoint.getSignature(), executionTime);
        
        return result;
    }
}
```

- 포인트컷 표현식을 보고 일치하는 클래스들은 프록시를 만들어서 빈으로 등록
- 런타임 시 포인트컷에 일치하는 메서드들은 애스펙트에 정의해 놓은 어드바이스 로직을 실행
- 타겟의 메서드를 호출

1. **특정 어노테이션이 붙은 메서드에만 AOP를 적용하는 예시:**

```java
// 커스텀 어노테이션 정의
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime {
}

// 서비스 클래스에서 사용
@Service
public class UserService {
    @LogExecutionTime
    public User findUser(Long id) {
        // 비즈니스 로직
        return userRepository.findById(id);
    }
}

// Aspect 정의
@Aspect
@Component
public class LoggingAspect {
    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    
    @Around("@annotation(com.example.annotation.LogExecutionTime)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        // ... (이전과 동일)
    }
}
```

# 4. Spring AOP와 AspectJ 비교

---

스프링 AOP와 AspectJ의 가장 큰 `차이점`은 `위빙 방식`에 있습니다. 스프링 AOP는 주로 프록시 기반의 런타임 위빙을 사용하여 AOP를 구현합니다. 이는 스프링 빈에만 AOP를 적용할 수 있으며, 메소드 실행 시 프록시를 통해 추가 기능을 실행하는 방식입니다. 반면, AspectJ는 컴파일 타임, 로드 타임에 AOP를 적용할 수 있어 더 다양한 시나리오에서 사용할 수 있습니다.

AspectJ는 바이트코드를 직접 조작하여 AOP를 구현하기 때문에, 스프링 AOP보다 더 세밀한 제어가 가능합니다. 예를 들어, 생성자 호출, 필드 접근과 같은 다양한 조인 포인트에서 AOP를 적용할 수 있습니다. 이러한 차이로 인해 `AspectJ`는 스프링 AOP보다 더 강력한 AOP 구현이 가능하지만, 설정과 사용이 복잡하고 `학습 곡선`이 더 `높습니다.`

## 4.1 Weaving 시점

---

- **Spring AOP**: Spring AOP는 pure Java로 구현되어 특별한 컴파일 프로세스를 사용하지 않고 `런타임`에서 Weaving을 진행합니다. 런타임 과정에서 AOP를 구현하기 위해서 AOP Proxy를 사용합니다.
- **AspectJ**: AspectJ는 Java 언어의 확장 기능을 사용하여 구현되어 AspectJ를 위한 Compiler가 사용됩니다. 이에 따라서 AspectJ는 3가지 유형의 Weaving을 활용할 수 있습니다.
    - Compile-time Weaving: 소스코드 컴파일 과정에서 Weaving을 진행하기
    - Post-compile Weaving(Binary Weaving): `.class` 파일과 `.jar` 파일 기반의 Weaving 진행하기
    - Load-time weaving: class Loader가 `.class` 파일을 JVM에 로드할 때 Weaving 진행하기

정리하자면 Spring AOP는 런타임에서 Weaving을 AspectJ는 `컴파일` 및 `클래스 로드 시간`에 Weaving을 한다는 점에서 차이가 존재합니다.

## 4.2 Join Point 위치

---

Spring AOP는 오로지 **Spring Bean**에서의 메서드 실행에 대한 Join Point만 지원하며 필드 참조나 할당에 대한 지원은 AspectJ를 사용하라고 안내하고 있습니다. 프레임워크에 따른 사용 가능한 Join Point를 열거해보면 아래 표와 같습니다.

| **Join Point** | **Spring AOP** | **AspectJ** |
| --- | --- | --- |
| 메서드 호출 | No | Yes |
| 메서드 실행 | Yes | Yes |
| 생성자 호출 | No | Yes |
| 생성자 실행 | No | Yes |
| 정적 초기화 실행 | No | Yes |
| 객체 초기화 | No | Yes |
| 필드 참조 | No | Yes |
| 필드 할당 | No | Yes |
| 핸들러 실행 | No | Yes |
| 어드바이스 실행 | No | Yes |

왜 Spring AOP가 메서드 실행에 대한 Join Point만을 지원할 수 있는지 생각해보면 런타임에서 Weaving을 실행하기 때문에 Proxy를 활용한 우회로써만 AOP 구현이 가능하다는 점을 생각해보면 이해할 수 있습니다. AspectJ는 Compile 및 Class Loader 과정에서 AOP를 구현할 수 있으므로 부가적인 기능을 모든 곳에 추가할 수 있지만 Spring AOP는 오로지 런타임에서만 동작하기 때문에 메서드 실행에서 제한적으로 사용이 가능합니다.

## 4.3 성능

---

당연히 컴파일 단계에서의 Weaving을 진행하는 AspectJ가 런타임 단계에서 Weaving을 진행하는 Spring AOP보다 빠릅니다. Spring AOP의 경우 애플리케이션이 시작할 때 프록시가 생성되는 과정이 추가로 필요하기 때문입니다.

> 밴치마크에 따르면 AspectJ가 Spring AOP보다 8~35배 정도의 속도 차이를 보인다는 것을 알 수 있습니다.
>
