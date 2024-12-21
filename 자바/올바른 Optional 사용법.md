# **🤔 Optional 사용목적**

**Optional**은 **메서드의 리턴 타입으로 `Null일 수도 있음`을 명확히 표현하는 용도**

### Optional 사용하기 전 주의

반환 값에서 Null을 반환할 수 있다고 알릴 때 Optional을 사용한다고 했습니다. 그러나 애초에 Null을 반환하는 행위 자체가 안티 패턴입니다. `Optional을 사용하기 전`에 Null을 반환하는 `메소드를 수정`하는 것이 좋습니다. `ex) 빈 값 반환, -1 반환`

# **😲 Optional 올바른 사용법**

### 1. Optional 객체에 null를 할당하지마라

반환 값을'null'로 반환할 경우 서비스 로직에서NullPointerException이 발생할 수 있기 때문에 Optional을 사용합니다. 즉, 서비스 로직에서 Optional을 반환받으면 Optional이 감싸고 있는 값이'null'일 경우에 대한 로직을 추가하여NullPointerException을 방지할 수 있게 됩니다. 

이러한 Optional 객체에 'null'을 할당하게 되면 Optional을 사용하는 의도가 없어지게 됩니다. 서비스 로직에서 반환받은 Optional이'null'일 때 사용하게 되면'null'을 참조하게 되니 결국엔NullPointerException이 발생하게 됩니다. 따라서 Optional 객체에'null'을 할당하지 않고 빈(empty) Optional 객체를 반환하 Optional.empty()를 사용해야 합니다.

### 2. ifPresent - get 은 orElse 나 orElseXX로 대체해라

 Optional 객체의 값 존재유무에 따른 로직은Optional.orElse(),Optional.orElseXX 를 사용하는 것을 추천합니다. Optional에서 제공하는 다양한 API들을 사용하게 되면 if문 등을 사용하지 않기 때문에 코드의 가독성이 증가하게 됩니다. 따라서 값이 존재여부에 따른 분기처리를 하지 않는 경우라면 Optional에서 제공하는 API를 사용하는 것이 좋습니다.

### 3. orElse(new ...) 대신 orElseGet(() -> new ...)

orElse(...)에서 ...는 Optional에 값이 있든 없든 **무조건 실행된다.** 따라서 ...가 새로운 객체를 생성하거나 새로운 연산을 수행하는 경우에는 orElse() 대신 **orElseGet()**을 써야한다.

```java
// 안 좋음
Optional<Member> member = ...;
return member.orElse(new Member());  // member에 값이 있든 없든 new Member()는 무조건 실행됨

// 좋음
Optional<Member> member = ...;
return member.orElseGet(Member::new);  // member에 값이 없을 때만 new Member()가 실행됨

// 좋음
Member EMPTY_MEMBER = new Member();
...
Optional<Member> member = ...;
return member.orElse(EMPTY_MEMBER);  // 이미 생성됐거나 계산된 값은 orElse()를 사용해도 무방
```

### 4. Optional을 필드에서 사용하지 말아라

- Optional 클래스는 `Serializable`을 구현하지 않았습니다. JPA 엔티티나 DTO 등 직렬화가 필요한 클래스에서 문제 발생할 수 있습니다.
- 다시 한 번 강조합니다!! `Optional은 메서드 반환 타입으로 사용하기 위해 설계됨`

### 5. Primitive Type을 값으로 갖는 Optional은 OptionalInt, OptionalLong, OptionalDouble을 사용해라

원시 타입(Primitive Type)을 값으로 갖는 Optional은 박싱/언박싱에 따른 오버헤드가 생기게 됩니다. 따라서 원시 타입을 값으로 갖는 Optional을 사용할 때에는OptionalInt,OptionalLong,OptionalDouble을 사용하여 오버헤드를 줄일 수 있습니다. 

**+) 박싱/언박싱이란?**

```java
// 박싱: primitive type -> Wrapper class
int primitive = 10;
Integer boxed = primitive;    // 자동 박싱

// 언박싱: Wrapper class -> primitive type
Integer wrapper = new Integer(10);
int primitive = wrapper;      // 자동 언박싱
```

**원시 타입 갖는 Optional 올바른 사용법**

```java
// Optional 사용 시
Optional<Integer> opt = Optional.of(10);  // 박싱 발생
OptionalInt opt = OptionalInt.of(10);     // 박싱 없음

// 박싱/언박싱 x
OptionalInt intOp = ...;
OptionalLong longOp = ...;
OptionalDouble doubleOp = ...;
```

### 6. Optional 대신 비어있는 컬렉션 반환

Optional은 비싸다. 컬렉션을 Optional로 감싸서 반환하지 말고 비어있는 컬렉션을 반환하자.

+) 컬렉션을 반환하는 Spring Data JPA Repository 메서드는 null을 반환하지 않고 비어있는 컬렉션을 반환해주므로 Optional로 감싸서 반환할 필요가 없다.

```java
// 안 좋음
List<Member> members = team.getMembers();
return Optional.ofNullable(members);

// 좋음
List<Member> members = team.getMembers();
return members != null ? members : Collections.emptyList();
```

**참고:**

https://dzone.com/articles/using-optional-correctly-is-not-optional
