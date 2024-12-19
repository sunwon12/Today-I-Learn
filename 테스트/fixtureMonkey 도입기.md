# 🙊 Fixture Monkey 도입 이유

---

- 객체 생성
    - 객체가 가지고 있는 데이터 항목이 많아 `객체 생성 비용`이 크다.
    - 비즈니스 요구사항이 변경될 때 객체를 변경하고 테스트를 수정해야 할 수 있다.
- 엣지 케이스 발견
    - 엣지 케이스를 직접 찾아야 하고 동일한 테스트를 여러번 작성해야한다

## **🤔 Fixture Monkey가 뭔디**

---

https://naver.github.io/fixture-monkey/v1-0-0/   

Fixture Monkey는 테스트를 위한 객체 생성을 자동화해주는 `Jqwik` 기반 `PBT` 라이브러리입니다.

## **🤔 Property-based Testing (PBT)는 뭘까?**

---

Property-based Testing은 테스트 케이스를 일일이 작성하는 대신, 프로그램이 랜덤 속성(property)을 만들어주고 테스트하는 방식을 말합니다. 자동으로 `다양한 테스트 케이스`를 생성할 수 있고 `엣지 케이스를 발견`할 수 있다는 데에 장점이 있습니다.

```java
// Unit Test - 구체적인 예시로 테스트
@Test
void testAdd() {
    assertEquals(4, add(2, 2));
    assertEquals(0, add(0, 0));
}

// Property Test - 속성(규칙)으로 테스트
@Property
void additionCommutative(@ForAll int a, @ForAll int b) {
    assertEquals(add(a, b), add(b, a)); // 교환법칙 검증
}
```

### **Spring에서 사용할 수 있는 PBT 종류**

- Jqwik
    - JUnit 5와 통합
    - Spring Boot Test와 호환성 좋음
    - https://www.baeldung.com/java-jqwik-property-based-testing
    월 500만 이상의 방문자의 블로그인 Baeldung(Eugen Paraschiv의 블로그 / Spring 관련 비공식 권위자)에서도 jqwik를 쓰는 것을 확인할 수 있음
- junit-quickcheck
    - JUnit과 통합
    - Spring Test와 함께 사용 가능
    - JUnit 4 기반
- Kotest (Kotlin)
    - Kotlin + Spring Boot 프로젝트에서 사용
    - Property-based Testing 기능 포함
    - Spring Test 지원
- ScalaCheck
    - Spring + Scala 프로젝트에서 사용 가능
    - 하지만 실제로는 거의 사용되지 않음

## Fixture Monkey 객체 생성 방식

---

fixture Monkey는 `BeanArbitraryIntrospector`객체를 생성하는 기본 방법으로 사용합니다. `Introspector`Fixture Monkey가 객체를 생성하는 방법입니다. 

- [**Introspector 종류**](https://naver.github.io/fixture-monkey/v1-0-0/docs/generating-objects/introspector/#builderarbitraryintrospector)
    - **`BeanArbitraryIntrospector (기본)`**
        - 리플렉션으로 && no-args && setter로 객체 생성
        - 기본 인트로스펙터
    - **`ConstructorPropertiesArbitraryIntrospector`**
        - 생성자를 통한 객체 생성
        - @ConstructorProperties 어노테이션 필요
        - record 타입도 지원
        - Lombok 설정으로도 사용 가능
    - **`FieldReflectionArbitraryIntrospector`**
        - 리플렉션으로 && no-args && getter/setter 중 하나로 객체 생성
        - non-final 변수는 getter/setter 불필요
        - **엔티티에 @Setter 놓는 것을 선호하지 않기 때문에 개인적으로 이 방식 선호**
    - **`BuilderArbitraryIntrospector`**
        - 빌더로 객체 생성
    - **`FailoverArbitraryIntrospector`**
        - 여러 인트로스펙터 조합 사용
        - 하나가 실패하면 다음으로 시도
        - 실패 로그 비활성화 가능
    - **`PriorityConstructorArbitraryIntrospector`**
        - 커스텀 생성자로 객체 생성
        - @ConstructorProperties 불필요 ←-ConstructorPropertiesArbitraryIntrospector와의 차이점

# 생성 객체 조작하는 법

---

```java
// 방법 1. **FixtureMonkeyBuilder**로 전연적으로 설정
FixtureMonkey fixtureMonkey =FixtureMonkey.builder()
																						...
																						...
																						.build()
Product product = fixtureMonkey.giveMe(Product.class)  // 바로 객체 반환			

// 방법 2. **ArbitraryBuilder**로 설정하는 단계 거쳐(현재 생성 객체 해당) 객체 반환
Product product = fixtureMonkey.giveMeBuilder() 
																.set()
																.sample 
```

## **FixtureMonkeyBuilder의 함수 이해하기**

---

```java
/* FixtureMonkeyBuilder 주요 메서드 정리 */

// 1. 기본 설정 관련
fixtureMonkey.builder()
   .defaultNotNull(true)                 // null 값 생성하지 않도록 설정 
   .nullableContainer(false)             // 컬렉션(List, Set 등)을 null로 생성하지 않음
   .nullableElement(false)               // 컬렉션의 요소를 null로 생성하지 않음(ex: List<String>의 String 요소)
   .enableLoggingFail(true)              // 객체 생성 실패 시 로그 출력 설정
   .useExpressionStrictMode()            // 표현식 엄격 모드 사용 - 정확한 필드 경로 검증
   .seed(123L)                           // 랜덤 시드값 설정 (재현 가능한 테스트를 위해)

// 2. 특정 타입/패키지 제외 설정  
   .objectIntrospector(FieldReflectionArbitraryIntrospector.INSTANCE)  // 필드 리플렉션 방식으로 객체 생성
   .addExceptGenerateClass(Board.class)                                // 특정 클래스 타입의 필드를 null로 설정
   .addExceptGenerateClasses(Board.class, User.class)                  // 여러 클래스 타입의 필드들을 null로 설정  
   .addExceptGeneratePackage("com.example")                           // 특정 패키지의 모든 클래스 필드를 null로 설정
   .pushExceptGenerateType(matcher)                                    // 커스텀 matcher로 제외할 타입 지정

// 3. 타입별 생성 규칙 등록
   .register(Long.class, fixtureMonkey -> fixtureMonkey.giveMeBuilder(Long.class)  // 타입과 하위타입 규칙
       .set("id", Arbitraries.longs().greaterOrEqual(0L)))
   
   .registerExactType(Integer.class, fixtureMonkey -> fixtureMonkey.giveMeBuilder(Integer.class)  // 정확한 타입에만 규칙
       .set("id", Arbitraries.integers().greaterOrEqual(0)))
   
   .registerAssignableType(Number.class, fixtureMonkey -> fixtureMonkey.giveMeBuilder(Number.class)  // 상위타입 포함 규칙
       .set("id", Arbitraries.integers().greaterOrEqual(0)))
   
   .registerGroup(MyArbitraryBuilderGroup.class)   // 그룹으로 정의된 여러 규칙들을 한번에 등록

// 4. 프로퍼티(필드) 생성 관련
   .defaultPropertyGenerator(customGenerator)                          // 기본 필드 생성 규칙 변경
   .pushPropertyGenerator(matcherOperator)                            // 새로운 생성 규칙 추가
   .pushAssignableTypePropertyGenerator(type, generator)              // 특정 타입에 대한 생성 규칙 추가
   .defaultPropertyNameResolver(resolver)                             // 프로퍼티 이름 해석 방식 설정
   .pushPropertyNameResolver(matcherOperator)                         // 프로퍼티 이름 해석 규칙 추가
   .defaultObjectPropertyGenerator(generator)                         // 객체 필드 생성 기본 규칙 설정
   .pushObjectPropertyGenerator(matcherOperator)                      // 객체 필드 생성 규칙 추가

// 5. 컬렉션과 컨테이너 관련
   .addContainerType(List.class, containerPropertyGenerator,          // 컬렉션 타입 생성 방식 설정
       introspector, valueFactory)                                    
   .pushContainerPropertyGenerator(matcherOperator)                   // 컬렉션 생성 규칙 추가
   .pushContainerIntrospector(introspector)                          // 컨테이너 검사기 추가
   .defaultDecomposedContainerValueFactory(factory)                   // 컨테이너 분해 방식 설정
   .addDecomposedContainerValueFactory(type, factory)                // 특정 타입의 컨테이너 분해 방식 추가
   .defaultArbitraryContainerInfoGenerator(generator)                 // 컨테이너 정보 생성 기본 설정
   .pushArbitraryContainerInfoGenerator(matcherOperator)             // 컨테이너 정보 생성 규칙 추가

// 6. Null 처리 관련
   .defaultNullInjectGenerator(generator)                             // null 주입 기본 규칙 설정
   .pushNullInjectGenerator(matcherOperator)                         // null 주입 규칙 추가
   .pushExactTypeNullInjectGenerator(type, generator)                // 정확한 타입에 대한 null 주입 규칙
   .pushAssignableTypeNullInjectGenerator(type, generator)           // 타입과 하위타입에 대한 null 주입 규칙

// 7. 검증 관련
   .arbitraryValidator(validator)                                     // 값 검증기 설정
   .javaConstraintGenerator(new JavaConstraintGenerator() {          // @NotNull 등 validation 어노테이션 처리
       @Override
       public void process(JavaConstraintGeneratorContext context) {
           if (context.hasAnnotation(NotNull.class)) {
               context.setNotNull(true);
           }
       }
   })
   .pushJavaConstraintGeneratorCustomizer(generator ->               // 제약조건 처리 방식 수정
       generator.customize(context -> {
           // 커스텀 처리 로직
       }))

// 8. 생성 재시도 및 최적화
   .generateMaxTries(100)                // 객체 생성 최대 시도 횟수 설정
   .generateUniqueMaxTries(50)           // 중복되지 않는 값 생성 최대 시도 횟수 설정
   .manipulatorOptimizer(optimizer)      // 객체 조작 최적화 설정

// 9. 플러그인과 확장
   .plugin(new SimpleValueJqwikPlugin()) // jqwik 플러그인 사용 (극단값 생성 방지)
   .build();                             // FixtureMonkey 인스턴스 생성
```

## ArbitraryBuilder의 함수 이해하기

---

```java
//----------------------- 특정 값 지정 -------------------------
// 1. 기본 if 조건 (단순 조건에 따른 고정값)
Product product = fixtureMonkey.giveMeBuilder(Product.class)
   .set("price", price > 1000 ? 2000 : 500)
   .sample();

// 2. InnerSpec 설정 (여러 필드의 고정값 설정)
InnerSpec basicSpec = new InnerSpec()
   .property("category", condition ? Category.BREAD : Category.CAKE)
   .property("price", isDiscounted ? 2000 : 5000)
   .property("soldout", stock == 0 ? true : false);

Product product = fixtureMonkey.giveMeBuilder(Product.class)
   .setInner(basicSpec)
   .sample();

//----------------------- 범위 내 랜덤 값 생성 -------------------------
// 1. set + Arbitraries 사용 (가장 직관적인 범위 지정)
Product product = sut.giveMeBuilder(Product.class)
   .set("price", Arbitraries.integers()
       .filter(price -> isDiscountDay() ? price < 5000 : price < 10000))
   .sample();

// 2. setPostCondition 사용 (조건에 따른 값 필터링)
Product product = fixtureMonkey.giveMeBuilder(Product.class)
   .setPostCondition("price", Integer.class, 
       (Predicate<Integer>)(price -> price > getMinimumPrice()))
   .sample();

// 3. setLazy 사용 (동적 값 계산)
Product product = fixtureMonkey.giveMeBuilder(Product.class)
   .setLazy("price", () -> calculatePrice(/* 조건에 따른 계산 */))
   .sample();

// 4. typedRoot 사용 (타입 기반 객체 전체 조건)
Product product = fixtureMonkey.giveMeBuilder(Product.class)
   .builder.customizeProperty(
       TypedExpressionGenerator.<Product>typedRoot(),
       arbitrary -> arbitrary.filter(product -> 
           product.getCategory() == Category.BREAD ? 
               product.getPrice() < 5000 : product.getPrice() < 10000)
   );

// 5. typedString 사용 (특정 필드 조건)
Product product = fixtureMonkey.giveMeBuilder(Product.class)
   .builder.customizeProperty(
       TypedExpressionGenerator.typedString("price"),
       arbitrary -> arbitrary.filter(price -> 
           isHoliday() ? (int)price < 5000 : (int)price < 3000)
   );

// 6. acceptIf 사용 (조건에 따른 필드 설정)
Product product = sut.giveMeBuilder(Product.class)
   .acceptIf(
       p -> p.getPrice() > 10000,
       builder -> builder.set("category", Category.CAKE)
   )
   .sample();
```

위 방법들 말고도 필드값을 설정하는 방법은 다양합니다. 그런 다소 복잡한 점들은 빼고 직관적인 것들만 정리했습니다. 위 방법 중 필드값을 세팅하는 법을 정해야 합니다. 판단의 기준으로는 `성능`, `편의성`, `지속성`을 생각해볼 수 있을 것 같습니다. 

**성능 측면**

- `setPostCondition` 함수는 특정 필드의 값 생성 후 조건 체크하는 방식으로 조건에 맞는 필드를 생성합니다. `acceptIf`는 객체 전체가 생성된 후 조건 체크하는 방법입니다. 이는 한 번에 생성하는 것이 아닌 생성을 반복함으로 다른 함수들보다 `성능이 낮습니다`. 이 외의 함수들은 생성하고 검증하는 것이 아닌, 값에 해당하는 객체를 한 번에 생성합니다.

```java
// setPostCondition: 생성된 값을 검증하고, 조건에 맞지 않으면 재생성
.setPostCondition("age", Integer.class, (Predicate<Integer>)(age -> age > 20))
// 내부적으로: 값 생성 -> 조건 체크 -> 실패시 다시 생성 -> 조건 체크... 반복

// Arbitraries: 처음부터 조건에 맞는 값만 생성
.set("id", Arbitraries.integers().greaterOrEqual(1))
// 내부적으로: 조건에 맞는 값을 바로 생성
```

**편의성 && 지속성 측면**

- Fixture Monkey는 아직 안정화되지 않은 오픈소스라고 생각합니다. 지금 껏 그래왔듯이 deprecated 되는 객체, 함수가 빈번할 거라 생각합니다. 참고로 다음 버전에는 다른 필드 값 설정 방법을 공개한다고 합니다. 그래서 최대한 `가장 기본적인 함수`를 `사용`하는 것이 좋다고 생각합니다.
- 가장 기본적인 함수라 하면 값을 지정할 때는 `set` 함수를 말합니다. 특정값을 지정할 때에는 그냥 set 함수만 쓰고, 범위 내에 랜덤값을 지정하는 `set함수 + Arbitraries` 를 쓸 생각입니다. 참고로 `Arbitraries`는 Fixture Monkey의 객체가 아닌 **`Jqwik`**의 객체입니다.

**결론**

- 특정 값 지정: `set`함수
- 범위 내 랜덤 값 생성: `set함수 + Arbitraries`

# 전역적 설정 과정

---

## 엔티티 id 값은 0 이상이면 좋겠다(전역적 설정)

---

저흰 위에서 FixtureBuilder를 이용해 전역적으로 필드 값을 설정하는 법을 봤습니다.  register 함수를 이용해 특정 필드값에 대한 설정을 해주고 static 변수인 fixtureMonkey를 여러 곳에서 사용하면 됩니다.

**방법1**

```java

class FixtureMonkeyConfig {

    
     public static FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
             // Long 타입과 그 하위 타입 필드를 찾아서 양수로 설정
             .register(Long.class,
                     fixtureMonkey -> fixtureMonkey.giveMeBuilder(Long.class))
              .build();
           
 }
```

**방법2**

```java

public class FixtureMonkeyConfig {

    public static final FixtureMonkey fixtureMonkey;

    static {
        fixtureMonkey = FixtureMonkey.builder()
                // id 값 0이상이도록
                .pushAssignableTypeArbitraryIntrospector(
                        Long.class,
                        context -> new ArbitraryIntrospectorResult(
                                ArbitraryUtils.toCombinableArbitrary(
                                        Arbitraries.longs().greaterOrEqual(0L)  // 항상 양수 생성
                                )
                        )
                )
                .build();
    }
}

```

### register vs pushAssignableTypeArbitraryIntrospector 함수 비교

- 단순히 값의 범위나 조건만 정하고 싶다면 pushAssignableTypeArbitraryIntrospector
- 복잡한 생성 로직이나 다른 타입과의 연관관계가 필요하다면 register

## 연관관계인 필드는 null이 아니면 좋겠다(전역적 설정)

---

```java

class FixtureMonkeyConfig {
     public static FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
	              // 연관관계 필드 Null 아니도록
                .pushNullInjectGenerator(
                        new MatcherOperator<>(
                                p -> p.getAnnotation(OneToMany.class) != null
                                        || p.getAnnotation(ManyToOne.class) != null
                                        || p.getAnnotation(OneToOne.class) != null,
                                objectPropertyGeneratorContext -> NOT_NULL_INJECT
                        )
                )
                .build();

 }
```

위코드처럼 처음 연관관계 필드는 null이 아니게 설정하였습니다. 그러나 fixture monkey로 객체를 생성할 때 null이 아니면 안되는 필드들이 많았습니다. 그래서 아래 코드처럼 주어진 함수를 이용하여 어떠한 거든 null이 안되도록 설정하여 주었습니다.

```java

class FixtureMonkeyConfig {
     public static FixtureMonkey fixtureMonkey = FixtureMonkey.builder()
	              // 연관관계 필드 Null 아니도록
                 .defaultNotNull(true)                   // null 값 허용 안 하도록
                .build();

 }
```

# 트러블 슈팅

---

## 1. NotNull로 설정하였지만 List 사이즈가 간혈적으로 0이 나온다

---

![image](https://github.com/user-attachments/assets/497f7d67-b866-4d9b-aa25-c26e9987bca3)

![image](https://github.com/user-attachments/assets/1290ba3b-a687-43a3-8a1f-35f83c55ceab)

제가 한 가지 간과한게 있습니다. `NotNull`과 `빈값`, `new ArrayList<>()`과는 다릅니다. 빈값과 
`new ArrayList<>()`는 `null`이 아닙니다. (fixture Monkey를 잠깐 불신해서 죄송합니다..) 아래 보시면 String도 가끔 빈값이 들어옵니다.  

![image](https://github.com/user-attachments/assets/af3034db-beb2-4a4c-84bd-3254fe57ea0e)

## 해결: ContainerPropertyGenerator 커스텀하여 Config에 추가

---

오픈소스를 커스텀하는 건 굉장히 어려운 일이었습니다. 어떤 클래스들을 어떻게 서야할지 몰라 막막했지만, `DefaultSingleContainerPropertyGenerator`(기존 컨테이너 프로퍼티를 다루는 생성기)의 구현을 참고하여 작성하였습니다. `A부터 Z까지 오픈소스를 파악하지 못할 때 기존 구현체를 참고하여 구현하면 좋은 것 같습니다.`

그후, `pushAssignableTypeContainerPropertyGenerator`함수를 이용하여 커스텀한 컨테이너 생성기를 FixtureMonkey Builder에 추가하여주었습니다.

```java

public class ListContainerPropertyGenerator implements ContainerPropertyGenerator {

    @Override
    public ContainerProperty generate(ContainerPropertyGeneratorContext context) {
        // 원본 property 가져오기
        Property property = context.getProperty();

        // List의 제네릭 타입 가져오기
        List<AnnotatedType> elementTypes = Types.getGenericsTypes(property.getAnnotatedType());
        if (elementTypes.isEmpty()) {
            throw new IllegalArgumentException(
                    "List must have 1 generics type for element. " +
                            "propertyType: " + property.getType() +
                            ", elementTypes: " + elementTypes
            );
        }

        // List의 요소 타입
        AnnotatedType elementType = elementTypes.get(0);

        // 1에서 3 사이의 랜덤한 크기 결정
        ArbitraryContainerInfo containerInfo = new ArbitraryContainerInfo(1, 3);
        int size = containerInfo.getRandomSize();

        // 결정된 크기만큼 요소 프로퍼티 생성
        List<Property> elementProperties = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            elementProperties.add(
                    new ElementProperty(
                            property,
                            elementType,
                            i,  // sequence
                            i   // index
                    )
            );
        }

        // 최종 ContainerProperty 반환
        return new ContainerProperty(elementProperties, containerInfo);
    }
}

// -------- FixtureMonkey Config 클래스 ------------
	
fixtureMonkey = FixtureMonkey.builder()
   .pushAssignableTypeContainerPropertyGenerator(
                        List.class,
                        new ListContainerPropertyGenerator()
                )
   .build();
```

![image](https://github.com/user-attachments/assets/30c6fafa-57d3-4727-b818-bb1afa8826aa)

## 2. 부모 엔티티와 자식 엔티티의 부모 필드가 일치하지 않음

---

![image](https://github.com/user-attachments/assets/3996d029-4038-45a7-8c0a-3a328d7a962b)

## **해결: Reflect으로 직접 필드에 접근**

---

```java
fixtureMonkey = FixtureMonkey.builder()
   .register(Object.class, fixtureMonkey ->
                        fixtureMonkey.giveMeBuilder(Object.class)
                                .setPostCondition(entity -> {
                                    try {
                                        // 현재 엔티티의 모든 필드를 가져옵니다 - TypeCache를 활용하여 성능 개선
                                        Map<String, Field> fields = TypeCache.getFieldsByName(entity.getClass());

                                        // 모든 필드를 순회하면서 양방향 관계를 처리합니다
                                        for (Field field : fields.values()) {
                                            // final이나 static, transient 필드는 처리하지 않습니다
                                            if (Modifier.isFinal(field.getModifiers()) ||
                                                    Modifier.isStatic(field.getModifiers()) ||
                                                    Modifier.isTransient(field.getModifiers())) {
                                                continue;
                                            }

                                            field.setAccessible(true);
                                            Object fieldValue = field.get(entity);
                                            if (fieldValue == null) continue;

                                            // OneToMany 관계 처리
                                            if (field.isAnnotationPresent(OneToMany.class) && fieldValue instanceof List<?> list) {
                                                for (Object child : list) {
                                                    // 자식 엔티티에서 부모를 참조하는 필드를 찾습니다
                                                    setParentReference(child, entity);
                                                }
                                            }

                                            // ManyToOne 관계 처리
                                            if (field.isAnnotationPresent(ManyToOne.class)) {
                                                // 부모 엔티티에서 자식 컬렉션을 찾아 현재 엔티티를 추가합니다
                                                setChildrenReference(fieldValue, entity);
                                            }

                                            // OneToOne 관계 처리
                                            if (field.isAnnotationPresent(OneToOne.class)) {
                                                setOneToOneReference(fieldValue, entity);
                                            }
                                        }
                                        return true;
                                    } catch (Exception e) {
                                        LOGGER.warn("Bidirectional relationship setup failed for entity: {}",
                                                entity.getClass().getName(), e);
                                        return false;
                                    }
                                }))
                                .build();
                                
 // 부모 참조를 설정하는 헬퍼 메서드
    private static void setParentReference(Object child, Object parent) throws Exception {
        Map<String, Field> childFields = TypeCache.getFieldsByName(child.getClass());

        for (Field childField : childFields.values()) {
            if (childField.isAnnotationPresent(ManyToOne.class) &&
                    childField.getType().isAssignableFrom(parent.getClass())) {
                childField.setAccessible(true);
                childField.set(child, parent);
                break;
            }
        }
    }

    // 자식 컬렉션에 대한 참조를 설정하는 헬퍼 메서드
    private static void setChildrenReference(Object parent, Object child) throws Exception {
        Map<String, Field> parentFields = TypeCache.getFieldsByName(parent.getClass());

        for (Field parentField : parentFields.values()) {
            if (parentField.isAnnotationPresent(OneToMany.class) &&
                    parentField.getType().equals(List.class)) {
                parentField.setAccessible(true);
                List<Object> children = (List<Object>) parentField.get(parent);
                if (children != null && !children.contains(child)) {
                    children.add(child);
                }
                break;
            }
        }
    }

    // OneToOne 관계를 설정하는 헬퍼 메서드
    private static void setOneToOneReference(Object target, Object source) throws Exception {
        Map<String, Field> targetFields = TypeCache.getFieldsByName(target.getClass());

        for (Field targetField : targetFields.values()) {
            if (targetField.isAnnotationPresent(OneToOne.class) &&
                    targetField.getType().isAssignableFrom(source.getClass())) {
                targetField.setAccessible(true);
                targetField.set(target, source);
                break;
            }
        }
    }
```

## 3. 데이터베이스에 부모 엔티티를 저장할 때 자식 엔티티가 저장 안되어있음

---

![image](https://github.com/user-attachments/assets/ec52e35b-ad87-4329-a75a-7082deacce11)

Member를 저장할 때 속성으로 가지고 있는 `자식 엔티티`가 데이터베이스에 저장이 안되어있어서  `Withdrawl Id`가 존재하지 않는다고 에러가 나옵니다. 이를 해결하기 위해서는 `Member`를 저장할 때 `자식 엔티티`도 저장되게 해야 합니다. 

## 해결

![image](https://github.com/user-attachments/assets/b4907629-acba-4aa0-a4c4-4849dc6d6242)

member를 저장할 때 연관 엔티티도 저장되게 영속성 전이를 설정해주어 해결하였습니다.

## 4. member 필드의 List<Withdrawl>의 각 요소 id 값들이 겹친다

---

![image](https://github.com/user-attachments/assets/6deffe1d-19c5-4854-b6b6-4ccf0c22f7e0)

## **해결**

![image](https://github.com/user-attachments/assets/0331f73a-b601-4bb1-b00e-4a85298c8ec2)

`문제 2번`에서 해결한 코드에 아래 코드를 추가해줌으로써 해결해주었습니다. `null`로 `Id`값을 설정해준 의도는 운영환경에서도 엔티티가 데이터베이스에 저장이 되어야 id값이 생성되기 때문입니다. 

# 글을 마치며

---

Fixture Monkey를 도입함으로써 테스트 코드 작성 비용이 줄은 거는 확실합니다. 그러나 아직까지 많은 사람이 쓰지 않은 만큼 `참고자료`가 많이 `부족`하고 공식문서에서도 자세히 나와있지 않아 다루기도 힘들고, 오류에 대응하기 힘듭니다. 테스트 코드를 편히 작성하기 위해 Fixture Monkey를 도입하였는데 이것을 다루기 위해 2주라는 시간을 투자했습니다. 배보다 배꼽이 큰 상황이라는 걸 느꼈습니다. 

그래서 앞서 말한 점들과 사전에 팀원들이 오픈소스를 단기간내에 파악할 수 있는지를 고려하여 도입해야 할 것 같습니다.

Fixture Monkey issue를 확인해보니 아직까지도 부족한 점들이 많이 남은 것 같습니다. 지속적으로 버전 업데이트를 하여 최신 버전의 Fixture Monkey를 사용하길 권장드립니다.

**참고**

- [Fixture Monkey 컨퍼런스 발표 영상](https://www.notion.so/144d70faa06880759471d73694a94c47?pvs=21)
- [전역적 설정 참조](https://github.com/skytin1004/fixture-monkey/blob/main/fixture-monkey/src/test/java/com/navercorp/fixturemonkey/test/FixtureMonkeyOptionsAdditionalTestSpecs.java#L89)
