## **\# 논리 오류를 알게된 경위**

collectionLike를 데이터베이스에서 찾고 없으면 collectionLike를 새로 만들어 저장하는 코드를 짰다. 코드 검사에서 orElse에 있는 collectionLikeRepository.save()가 항상 호출된다고 위험을 알렸다.

```
    @Transactional
    public void saveCollectionLike(Long memberId, Long collectionId) {
        Member member = memberRepository.findByIdOrElseThrow(memberId);
        Collection collection = collectionRepository.findByIdOrElseThrow(collectionId);

        collectionLikeRepository.findByMemberAndCollection(member, collection)
                .orElse(collectionLikeRepository.save(
                        new CollectionLike(member, collection)));
    }
```

## **\# orElse()가 null이 아니어도 함수를 실행하는 이유**

orElse함수와 orElseGet함수의 구현은 아래와 같다.

```
/**
 * Return the value if present, otherwise return {@code other}.
 *
 * @param other the value to be returned if there is no value present, may
 * be null
 * @return the value, if present, otherwise {@code other}
 */
public T orElse(T other) {
  return value != null ? value : other;
}

/**
  * Return the value if present, otherwise invoke {@code other} and return
  * the result of that invocation.
  *
  * @param other a {@code Supplier} whose result is returned if no value
  * is present
  * @return the value if present otherwise the result of {@code other.get()}
  * @throws NullPointerException if value is not present and {@code other} is
  * null
  */
public T orElseGet(Supplier<? extends T> other) {
  return value != null ? value : other.get();
}
```

**결론부터 말하자면**

-   **orElse 메소드는 해당 값이 null 이든 아니든 관계없이 항상 불린다.**
-   **orElseGet 메소드는 해당 값이 null 일 때만 불린다**

```
@Test
@DisplayName("notNull테스트")
void name() {
    String name = "snack";
    String elseName = Optional.ofNullable(name).orElse(anyName());
    System.out.println(elseName);

    String elseGetName = Optional.ofNullable(name).orElseGet(this::anyName);
    System.out.println(elseGetName);
}

private String anyName() {
    System.out.println("reach anyName");
    return "anyName";
}
```

위 테스트 코드를 돌려보면 

**reach anyName**  
**snack**  
**snack**

```
@Test
@DisplayName("null테스트")
void name2() {
    String name = null;
    String elseName = Optional.ofNullable(name).orElse(anyName());
    System.out.println(elseName);

    String elseGetName = Optional.ofNullable(name).orElseGet(this::anyName);
    System.out.println(elseGetName);
}
```

**reach anyName**  
**anyName**  
**reach anyName**  
**anyName**

그럼 orElse()함수가 항상 파라미터 안에 있는 함수를 호출하는 이유는 뭘까?  쉽게 생각하면 함수를 파라미터로 전달해줬기 때문이다. 

```
public T orElse(anyName()) {
  return value != null ? value : anyName();
}
```

반면 orElseGet은 ohter.get으로 null일 때만 함수를 실행한다. other 자체로는 함수가 실행이 안된다.

```
public T orElseGet(Supplier<? extends T> other) {
  return value != null ? value : other.get();
}
```

## **\# 결론**

```
 @Transactional
    public void saveCollectionLike(Long memberId, Long collectionId) {
        Member member = memberRepository.findByIdOrElseThrow(memberId);
        Collection collection = collectionRepository.findByIdOrElseThrow(collectionId);

        collectionLikeRepository.findByMemberAndCollection(member, collection)
                .orElseGet(() -> collectionLikeRepository.save(new CollectionLike(member, collection)));
    }
```

-   orElse는 상수,변수값을 그대로 반환받을 때만 쓰고
-   orElseGet는 함수를 파라미터로 넘길때만 쓰자
