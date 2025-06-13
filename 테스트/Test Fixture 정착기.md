프로젝트를 진행하면 여러 Test Fixture 방식들을 사용해봤지만 방식만의 장단점이 있어 어느 방식을 선택해야 할지 모호했습니다. 이번 기회에 자주 사용되는 Test Fixture 방식들을 살펴보고 앞으로 개인 프로젝트에서 사용할 Test fixture 방식을 정하겠습니다.

### **1. 인라인 객체 생성**

가장 단순하고 원시적인 방법입니다. 모든 테스트 메서드 안에서 `new` 키워드나 빌더를 사용해 직접 객체를 생성합니다.

- **방식**: `@Test` 메서드 내부에 필요한 모든 객체를 즉석에서 생성합니다.
- **장점**:
    - **극강의 단순함**: 별도의 클래스나 파일이 필요 없어 가장 빠릅니다.
    - **테스트 독립성**: 테스트 메서드 하나만 보면 모든 준비 과정과 검증 과정을 이해할 수 있습니다.
- **단점**:
    - **높은 중복성**: 비슷한 객체를 여러 테스트에서 생성하면 똑같은 코드가 계속 반복됩니다.
    - **낮은 유지보수성**: 객체의 생성자나 필드가 변경되면 관련된 모든 테스트 코드를 일일이 찾아 수정해야 합니다.
    - **가독성 저해**: Fixture 생성 코드가 길어지면 정작 중요한 테스트 로직이 한눈에 들어오지 않습니다.
- **적용 시점**:
    - 정말 간단한 객체이거나, 해당 테스트에서만 사용되는 매우 특수한 객체일 때 제한적으로 사용합니다. 예를 들어 저는 **Request**나 **Response**를 생성할 때 사용하는 방식입니다.
- **Java 예시**

```java
@Test
void 멤버를_성공적으로_생성한다() throws Exception {
    // given: 테스트에 필요한 request DTO를 메서드 내에서 직접 생성 (인라인 생성)
    final MemberCreateRequest request = new MemberCreateRequest("테스트유저", "test@example.com");

    // when: MockMvc를 사용하여 실제 POST 요청을 시뮬레이션
    final ResultActions resultActions = mockMvc.perform(
       post("/api/members") // POST 요청을 보낼 엔드포인트
            .contentType(MediaType.APPLICATION_JSON) // 요청 본문의 타입은 JSON
            .content(objectMapper.writeValueAsString(request)) // request 객체를 JSON 문자열로 변환하여 body에 담음
        );
        
        // then
        .
        .
        .
        .
    }
```

### **2. 외부 파일로부터 데이터 로드 (e.g., JSON, YAML, SQL)**

테스트 데이터를 코드와 완전히 분리하여 외부 파일로 관리하는 방식입니다.

- **방식**: `test/resources` 같은 곳에 `.json`이나 `.sql` 파일을 두고, 테스트 실행 시 이 파일을 읽어 객체로 변환하거나 DB에 삽입합니다.
- **장점**:
    - **데이터와 코드의 완벽한 분리**: 비개발자(QA 등)도 테스트 데이터를 쉽게 확인하고 수정할 수 있습니다.
    - **복잡한 데이터 표현 용이**: 복잡한 객체 그래프나 대량의 데이터를 표현하기에 좋습니다.
    - **통합 테스트에 강력함**: 특정 DB 상태를 재현해야 하는 통합 테스트에서 매우 효과적입니다.
- **단점**:
    - **초기 설정 복잡**: 파일 I/O, JSON 파싱(Jackson/Gson) 등 부가적인 설정과 코드가 필요합니다.
    - **유연성 부족**: "이 테스트에서는 이름만 살짝 바꾸고 싶은데?"와 같은 작은 변화를 주기가 번거롭습니다.
    - **성능 저하**: 매번 파일을 읽고 파싱하는 과정에서 오버헤드가 발생할 수 있습니다.
- **적용 시점**:
    - 여러 테이블의 데이터를 조합해야 하는 서비스 계층의 통합 테스트나, 복잡한 쿼리를 검증해야 하는 리포지토리 계층의 테스트에 적합합니다.
- **JAVA** 예시

```java

    @Test
    @Sql("/sql/member-test-data.sql") // 이 테스트 메서드 실행 전에 해당 SQL 파일을 실행합니다.
    void ID로_멤버를_성공적으로_조회한다() {
        // given
        // 별도의 given 코드가 필요 없습니다. 
        // @Sql 어노테이션이 ID가 1L인 멤버를 DB에 미리 넣어주는 'given' 역할을 대신합니다.
        final Long memberId = 1L;

        // when
        final MemberResponse response = memberService.findMemberById(memberId);

        // then
        assertThat(response).isNotNull();
        assertThat(response.getId()).isEqualTo(memberId);
        assertThat(response.getName()).isEqualTo("테스트유저");
        assertThat(response.getEmail()).isEqualTo("test@example.com");
    }
```

### **3. 파라미터화된 팩토리 (Parameterized Factory)**

- **방식:** 테스트 데이터 생성을 전담하는 `Fixture` 클래스(예: `MemberFixtures`)를 만듭니다.이 클래스 안에 생성 `static` 메서드를 여러 개 선언합니다. 각 생성자 메서드는 서로 다른 매개변수 목록을 가집니다. 메서드 내부에서는 전달받은 파라미터와 미리 정의된 기본값을 조합하여 객체를 생성하고 반환합니다.
- **장점:**
    - **높은 편의성**: `createWithName("김철수")`, `createWithAge(30)`처럼, 변경하고 싶은 값만 파라미터로 전달하면 되므로 매우 편리하고 코드가 짧아집니다.
    - **기본값 캡슐화**: 이름이나 나이 같은 기본값을 `Fixture` 클래스 안으로 숨길 수 있어 테스트 코드에서는 핵심적인 값에만 집중할 수 있습니다.
- **단점:**
    - **조합 폭발**: 커스터마이징이 필요한 필드가 늘어날수록, 지원해야 하는 파라미터 조합의 수가 기하급수적으로 증가합니다. 필드가 4~5개만 되어도 모든 조합의 메서드를 만드는 것은 거의 불가능에 가깝습니다.
    - **유연성 부족**: 미리 정의해두지 않은 새로운 조합(예: 이름과 주소만 변경)이 필요할 때마다 `Fixture` 클래스에 새로운 `create` 메서드를 계속 추가해야 합니다. Fixture 클래스가 금방 비대해지고 관리가 어려워집니다.
- **JAVA** 예시

```java
public class UserFixtures {

private static final String DEFAULT_NAME = "기본이름";
private static final int DEFAULT_AGE = 20;

// 파라미터가 없는 기본 create() 메서드
public static User create() {
    return new User(DEFAULT_NAME, DEFAULT_AGE);
}

// 이름만 지정하고 싶을 때 사용하는 오버로딩된 create(String) 메서드
public static User create(String name) {
    return new User(name, DEFAULT_AGE);
}

// 나이만 지정하고 싶을 때 사용하는 오버로딩된 create(int) 메서드
public static User create(int age) {
    return new User(DEFAULT_NAME, age);
}

// 이름과 나이를 모두 지정하고 싶을 때 사용하는 오버로딩된 create(String, int) 메서드
public static User create(String name, int age) {
    return new User(name, age);
}

```

### **4. 오브젝트 마더 패턴 (Object Mother Pattern)**

Martin Fowler가 제시한 Object Mother 패턴은 테스트할 인스턴스를 만들어주는 클래스를 의미합니다. 이 패턴은 복잡한 객체 생성 로직을 중앙화하여 관리할 수 있게 해줍니다.

- **방식**: 특정 도메인 상황에 맞는 객체를 생성하는 `static factory method`를 모아둔 클래스를 만듭니다.
- **장점**:
    - **재사용성**: 여러 테스트에서 동일한 종류의 객체를 쉽게 재사용할 수 있습니다.
    - **의도 명확화**: `createVerifiedUser()`, `createUnverifiedUser()`처럼 메서드 이름으로 객체의 상태와 의도를 명확히 표현할 수 있습니다.
- **단점**:
    - **유연성 부족**: "인증됐지만 이메일은 없는 유저"처럼 미묘하게 다른 객체가 필요할 때마다 새로운 메서드를 계속 추가해야 해서 클래스가 비대해질 수 있습니다.
- **적용 시점**:
    - 앱 내에서 사용되는 매우 전형적이고 고정된 상태의 객체(예: 기본 관리자, 손님 유저)를 여러 테스트에서 반복적으로 사용해야 할 때 유용합니다.
- **Java 예시**

```java
public class UserMother {
    public static User anUnverifiedUser() {
        return User.aUser() // 내부적으로 빌더 패턴과 조합
                   .isVerified(false)
                   .build();
    }

    public static User aVerifiedUserWithNoEmail() {
        return User.aUser()
                   .email(null)
                   .isVerified(true)
                   .build();
    }
}
// 테스트 코드에서 사용
User user = UserMother.anUnverifiedUser();
```

### **5. Enum 기반 Fixture 패턴**

오브젝트 마더 패턴 + ENUM이라고 말할 수 있습니다.

- **방식:** 특정 도메인의 Fixture를 관리할 `enum`을 생성합니다. (예: `UserFixture`)`USER`, `ADMIN`, `SELLER` 등 필요한 객체의 '역할'이나 '상태'를 `enum` 상수로 정의합니다. 각 `enum` 상수는 객체를 생성하는 데 필요한 속성 값들을 생성자를 통해 내부 필드로 가지고 있습니다. `enum` 내부에 `getUser()`와 같은 메서드를 구현하여, 각 상수가 가진 데이터로 실제 객체를 생성하여 반환하도록 만듭니다.
- **장점**
    - **가독성과 의도 명확성**: `UserFixture.SELLER.getUser()`라는 코드만 봐도 "판매자(SELLER) 역할을 하는 유저 객체가 필요하구나"라는 의도가 아주 명확하게 드러납니다.  테스트 객체의 상태를 명확하게 파악할 수 있습니다.
    - **타입 안전성 (Type Safety)**: `enum`에 정의된 상수 외에는 다른 타입을 사용할 수 없으므로, 오타 등으로 잘못된 Fixture를 만드는 실수를 컴파일 시점에 방지할 수 있습니다.
- **단점**
    - **유연성 부족 (가장 큰 단점)**: 이 패턴은 **미리 정의된 고정된 객체**를 생성하는 데 특화되어 있습니다. "판매자인데, 닉네임만 다른 유저"처럼 테스트마다 미세한 속성 변경이 필요한 경우에는 매우 부적합합니다. 새로운 `enum` 상수를 추가하거나 `getUser` 메서드에 파라미터를 추가해야 하는데, 이는 `enum`의 본래 의도를 해칩니다.
    - **조합 폭발 문제**: 필요한 객체의 상태 조합이 많아지면 `enum` 상수의 수가 계속 늘어나 관리가 복잡해질 수 있습니다.

```java
public enum UserFixture {
    USER("user@gmail.com", "password1!", "userNickname", 0L, Role.ASSOCIATE, false),
    ANOTHER_USER("user@gmail.com", "password1!", "anotherUserNickname", 0L, Role.ASSOCIATE, false),
    SELLER("seller@gmail.com", "password1!", "sellerNickname", 1000L, Role.ASSOCIATE, false),
    BUYER("buyer@gmail.com", "password1!", "buyerNickname", 1000L, Role.ASSOCIATE, false);

    private final String username;
    private final String password;
    // ... 기타 필드

    public User getUser() {
        return User.builder()
            .username(username)
            .password(password)
            .nickname(nickname)
            .account(account)
            .role(role)
            .isDeleted(isDeleted)
            .build();
    }
}
```

### **6. Test Data Builder 패턴**

Builder 패턴을 활용한 Test Data Builder는 유연한 객체 생성을 가능하게 합니다. 이 패턴은 필요한 경우 특정 필드만을 수정해서 테스트가 가능하다는 장점이 있습니다. Fixture 클래스가 완성된 객체가 아닌, **미리 기본값이 설정된 빌더 객체를 반환**하는 것이 핵심입니다.

- **방식:** 테스트 데이터 생성을 전담하는  `Fixture` 클래스를 만듭니다. 이 클래스 안에  `builder()`와 같은 `static` 메서드를 정의합니다. 이 메서드는 객체를 직접 생성해서 반환하는 대신, **유효한 기본값들이 이미 설정된 `Builder` 객체**를 반환합니다. 테스트 코드에서는 이 빌더를 받아서, 필요한 속성만 추가로 수정한 뒤 마지막에 `.build()`를 호출하여 최종 객체를 생성합니다.
- **장점:**
    - **최고의 유연성과 가독성**: 테스트의 의도를 명확하게 표현할 수 있습니다. 특정 값의 변경이 필요하면 체이닝을 통해 "어떤 값이 이 테스트의 핵심 조건인지" 명확히 보여줍니다.
    - **유지보수 용이성**: 도메인 객체에 새로운 필드가 추가되어도 테스트 코드가 깨지는 것을 방지할 수 있습니다.
- **단점:**
    - 도메인(`User`)에 빌더를 생성하는 것이 아니기 때문에 빌더메소드를 속성마다 작성해야 합니다.
- **적용 시점**
    - **거의 모든 상황에 적용 가능한 가장 이상적인 패턴**입니다. 프로젝트의 규모가 커지고 테스트 케이스가 복잡해질수록 이 패턴의 진가가 드러납니다.
    - 대부분의 테스트에서는 표준적인 객체를 사용하지만, 일부 테스트에서는 특정 필드 몇 개만 수정해야 하는 경우가 빈번할 때 가장 효과적입니다.
    - 객체의 필드가 많고 생성 로직이 복잡할 때, 테스트 코드의 `given` 단계를 매우 깔끔하게 유지시켜 줍니다.
- **JAVA** 예시

```java
// 1. 테스트 코드에만 존재하는 PersonBuilder 클래스
public class PersonBuilder {
    private String name = "기본이름"; // 합리적인 기본값 설정
    private int age = 30;
    private double weight = 70.0;
    private double height = 1.75;

    public PersonBuilder withName(String name) {
        this.name = name;
        return this; // 메서드 체이닝을 위해 자기 자신을 반환
    }

    public PersonBuilder withWeight(double weight) {
        this.weight = weight;
        return this;
    }
    
    // ... 다른 필드들을 위한 with... 메서드들 ...

    public Person build() {
        return new Person(name, age, weight, height, 0);
    }
}

// 2. 테스트 코드에서의 사용
@Test
void testBMICalculation() {
    // given
    // 키와 몸무게만 이 테스트의 관심사! 다른 값은 알 필요 없음.
    Person person = new PersonBuilder()
            .withWeight(80.0)
            .withHeight(1.80)
            .build();

    // when & then
    // ...
}
```

### 7. **외부 라이브러리 사용하기 (e.g. Fixture Monkey)**

**Fixture Monkey**, AutoFixture 등과 같은 라이브러리를 사용하여 객체 생성을 자동화하는 라이브러리입니다. 개발자가 필드 값을 일일이 지정하지 않아도, 라이브러리가 객체의 구조를 분석하여 임의의(arbitrary) 데이터로 채워진 객체 인스턴스를 자동으로 생성해 줍니다.

- **방식:** `Fixture Monkey` 같은 라이브러리 의존성을 프로젝트에 추가합니다. 테스트 코드에서 `FixtureMonkey` 인스턴스를 생성합니다. 필요에 따라 플러그인이나 전역 설정을 추가할 수 있습니다. `giveMeOne(User.class)`와 같은 간단한 메서드를 호출하여, 모든 필드가 임의의 값으로 채워진 객체를 생성합니다.
- **장점:**
    - **압도적인 생산성**: 따로 fixture 클래스를 작성하지 않더라도`fixtureMonkey.giveMeOne(User.class)` 한 줄이면 모든 필드가 채워진 객체가 생성됩니다.
    - **유지보수 용이성**: 도메인 객체에 새로운 필드가 추가되어도, 테스트 코드는 수정할 필요가 없습니다. 라이브러리가 알아서 새 필드를 채워주기 때문입니다.
- **단점**
    - **학습 곡선**: 모든 팀원이 라이브러리의 사용법과 컨벤션을 학습해야 합니다.
    - **복잡한 커스터마이징**: 엔티티 간 양방향 관계 설정이나 복잡한 비즈니스 규칙을 만족하는 객체를 생성하려면 라이브러리의 고급 기능을 사용해야 하며, 이 과정이 수동으로 빌더를 만드는 것보다 더 복잡할 수 있습니다.
    - **예측 불가능성**: 데이터가 임의로 생성되므로, 특정 랜덤 값(예: 너무 긴 문자열) 때문에 의도치 않게 테스트가 실패하는 경우가 발생할 수 있어 테스트의 결정성이 떨어질 수 있습니다.
- **적용 시점**
    - 테스트 대상 객체의 필드 값이 무엇인지는 중요하지 않고, 단지 유효한 객체 인스턴스 자체가 필요할 때 유용합니다.
    - 다양한 임의의 값으로 여러 번 테스트하여 엣지 케이스를 발견하려는 테스트에 사용할 수 있습니다.

## 나의 결론: **Test Data Builder이지만 나의 경험을 곁들인 커스터마이징**

```java
public class ChatRoomFixture {

    /**
     * Domain/Service 테스트용 Fixture. 랜덤 ID를 가진 ChatRoom 객체를 생성합니다.
     */
    public static ChatRoom create() {
        return builder().build();
    }

    public static ChatRoomBuilder builder() {
        long randomId = ThreadLocalRandom.current().nextLong(1, 100000);
        return builderWithoutId()
                .id(randomId);
    }

    /**
     * Repository/API 테스트용 Fixture. ID가 없는 ChatRoom 객체를 생성합니다.
     */
    public static ChatRoom createWithoutId() {
        return builderWithoutId().build();
    }

    public static ChatRoomBuilder builderWithoutId() {
        long randomNumber = ThreadLocalRandom.current().nextLong(1, 100000);
        return ChatRoom.builder()
                .id(null)
                .name("Test ChatRoom " + randomNumber)
                .description("Test ChatRoom " + randomNumber + " description")
                .category(ChatCategory.BESTSELLER)
                .participantCount(0);
    }
} 

```

저는 처음에는  **오브젝트 마더 패턴**을 도입했습니다. `UserMother.createDefaultUser()` 같이 사용한 것입니다. 확실히 중복은 줄고, 테스트 코드도 깔끔해졌습니다. 근데 또 다른 문제가 생겼습니다. 테스트 케이스가 복잡해지면서 '인증은 됐지만 이메일은 없는 유저', '어드민 권한을 가진 유저' 같이 미묘하게 다른 객체들이 필요해졌고, Mother 클래스는 `createAdminUser()`, `createUnverifiedUserWithKakaoEmail()` 같은 메서드들로 점점 비대해져 재사용성이 오히려 떨어지는 느낌이었습니다. 그래서 **파라미터화된 팩토리를 사용했더니** 조합에 따라 메소드를 추가해야 하면 관리의 복잡성이 늘어나 배보다 배꼽이 더 큰 상황을 겪었습니다.

고민 끝에 제가 도달한 결론은 “**`Test Data Builder`을 커스터마이징해서 쓰자”**였습니다. 기존 Test Data Builder는 도메인 코드를 건드리지 않아 테스트 코드 내에서 withName()과 같이 메소드를 구현해야 하는 단점이 있었습니다. 그러나 builder를 메인 코드에서 사용하고 있기도 했고 도메인에서 사용하지 않더라도 생성자로 제약해야 하는 규칙이 없다면 @Builder를 추가해도 된다고 생각하여 기존 Test Data Builder 방식의 메소드는 작성하지 않았습니다.

**두 번째**로는 Test Data Builder를 사용하되 오브젝트 마더의 간편성도 이용하는 메소드를 하나 더 만들었습니다.

- `ChatRoomFixture.create()`: "그냥 기본 채팅방 하나 줘. 상세 내용은 중요하지 않아." 라는 의도가 명확히 보입니다.
- `ChatRoomFixture.builder().name("새로운 채팅방").build()`: 테스트 코드만 봐도 "아, 이 테스트는 채팅방 이름이 중요한 케이스구나" 하고 바로 이해가 됩니다. '어떤' 부분이 이 테스트의 핵심인지 드러내줍니다.

오브젝트 마더의 간편성과 Test Data Builder의 **재사용성/의도 명확성**이라는 두 마리 토끼를 다 잡을 수 있었습니다.

**세 번째**로는 `create()`, `createWithoutId()` 혹은 `builder()`, `builderWithoutId()`처럼 **영속성 상태(ID 유무)에 따른 분리**하였습니다. 저는 외부 종속성이 없는 Domain/Service단위테스트와 외부 종속성이 있는 Repository/통합테스트를 구분하여 테스트합니다. `createWithoutId()`, `builderWithoutId()`는 id를 제외하고 객체를 반환합니다. 이는 **Repository/통합 테스트**에서 **`save()`**시 **Sequence Id를 부여하기 때문에 CasCade로 자식 엔티티를 저장할 때  id가 있으면 충돌할 수도 있기 때문에 방지한 것입니다.** 

**네 번째**로는 비지니스 요구사항 (e.g. isFalse = false)에 따라 기본값을 설정해야 부분을 제외하고는 랜덤값을 부여했습니다. id와 같이 대부분의 속성들은 겹치면 안되거나 겹치지 않는 것을 의도하는 테스트가 많아 이렇게 설정하였습니다.

결국 이러한 방식들로 **‘개발 편의성’**과 **'안정적인 기본값'** 과 **'유연한 커스터마이징'를** 확보했습니다. Fixture 생성의 복잡함은 `Fixture` 클래스 안에 캡슐화하고, 테스트 코드에서는 오직 테스트의 '**의도**'에만 집중할 수 있게 만들어 줬습니다!!

---

**참고자료**

https://medium.com/better-programming/why-you-should-use-test-data-builders-714eb9de20c1

https://jaehoney.tistory.com/274

[https://inpa.tistory.com/entry/GOF-💠-빌더Builder-패턴-끝판왕-정리](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EB%B9%8C%EB%8D%94Builder-%ED%8C%A8%ED%84%B4-%EB%81%9D%ED%8C%90%EC%99%95-%EC%A0%95%EB%A6%AC)

https://oliveyoung.tech/2024-04-01/testcode-use-fixture-monkey/
