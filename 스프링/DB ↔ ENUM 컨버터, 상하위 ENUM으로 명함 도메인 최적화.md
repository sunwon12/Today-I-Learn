## 1. 기존 설계의 한계

---

관심사는 별도의 테이블에서 관리되고 있었습니다. 이를 명함 데이터와 연결하기 위해 **관심사 테이블**과 **명함-관심사 매핑 테이블**이 필요했습니다

- **interest 테이블**: 관심사 데이터를 저장.
- **id_card_interest_mapping 테이블**: 명함(`id_card`)과 관심사(`interest`)를 연결.

**데이터 예시:**

`interest` 테이블:

| id | name |
| --- | --- |
| 1 | GAME |
| 2 | MOVIE |
| 3 | DRAMA |

`id_card_interest` 테이블:

| id_card_id | interest_id |
| --- | --- |
| 101 | 1 |
| 101 | 2 |
| 102 | 1 |

**문제점**

1. `id_card`테이블에 관심사를 담지 않고 `id_card_interest` 테이블에 별도로 저장함에 따라 `한 개의` `idCard`가 `여러 행`을 `차지`하기 때문에 `데이터 공간 낭비`라고 생각하였습니다
2. 명함 조회 시, 관심사 테이블과 매핑 테이블을 `조인`해야 했습니다.
3. 관심사 추가 및 수정 시, 테이블과 데이터를 `직접 수정`해야 했습니다.

이러한 문제를 해결하기 위해 **`상하위 카테고리 ENUM`**, **`DB Column ↔ ENUM List 컨버터`**를 도입하고, 설계를 리팩토링하였습니다.

## 2. 리팩토링 과정

---

### 2-1. 테이블삭제하고 ENUM으로 변경(상하위 ENUM 구조 적용)

---

관심사 데이터를 따로 테이블로 관리하지 않지만, 데이터 정합성을 보장할 수 있어 ENUM으로 변경하였습니다. 또한, 카테고리 특성상 상하위 구조가 존재하였기 때문에 **상위-하위 ENUM 구조**를 도입하였습니다.

이로 인해 두 가지 장점을 얻었습니다. 별도의 조인 없이 명함 테이블에서 관심사 정보를 바로 읽어올 수 있어, **`조회 성능 향상`** 및 코드 **`개발 용이성`**을 얻었습니다. 또한, 관심사 목록 변경 시 ENUM값만 수정하면 되기 때문에 **`데이터 관리`**에도 용이했습니다.

**코드 예시:**

```java
@Getter
@AllArgsConstructor
public enum InterestType {

    ENTERTAINMENT("엔터테인먼트", null),
        GAME("게임", ENTERTAINMENT), MOVIE("영화", ENTERTAINMENT), DRAMA("드라마", ENTERTAINMENT),

    SPORTS("스포츠", null),
        RUNNING("런닝", SPORTS), HEALTH("헬스", SPORTS);

    private String name;
    private InterestType upperType;
}

```

### 2-2. ENUM 컨버터 도입

---

| id_card_id | interest_id |
| --- | --- |
| 101 | 1 |
| 101 | 2 |
| 102 | 1 |

위와 데이터와 같이 한 개의 idcard가 중복해서 나타나는 기존 방식을 개선하기 위해, **DB ↔ ENUM List JPA컨버터**를 도입했습니다. 

**동작 원리:**

- **DB에 저장할 때:** ENUM 값을 데이터베이스에 쉼표(,)로 구분된 문자열로 저장.
- **DB에서 꺼낼 때:** 데이터베이스 문자열을 읽어와 ENUM 값으로 변환.

**코드 예시**

```java
@Converter
public class InterestConverter implements AttributeConverter<List<InterestType>, String> {
    private static final String DELIMITER = ",";

    @Override
    public String convertToDatabaseColumn(List<InterestType> attribute) {
        if (attribute == null || attribute.isEmpty()) {
            return null;
        }
        return attribute.stream()
                .map(Enum::name)
                .collect(Collectors.joining(DELIMITER));
    }

    @Override
    public List<InterestType> convertToEntityAttribute(String dbData) {
        if (dbData == null || dbData.isBlank()) {
            return new ArrayList<>();
        }
        return Arrays.stream(dbData.split(DELIMITER))
                .map(InterestType::valueOf)
                .collect(Collectors.toList());
    }
}

```

```java
@Convert(converter = InterestConverter.class)
@Column(name ="card_back_interest_list")
private List<InterestType> cardBackInterestTypeList;

```

**수정 후 테이블 구조**

| id_card_id | interest_type |
| --- | --- |
| 101 | “GAME, MOVIE, DRAMA” |
| 102 | “GAEM” |

## 3. 결론

---

ENUM 컨버터와 상위-하위 ENUM 구조를 활용해 명함 도메인을 최적화한 결과

**데이터 중복 제거**

리팩토링 후, 관심사 데이터를 직접 저장하면서 중복 데이터를 제거할 수 있었습니다. 동일한 관심사 목록이 여러 명함에 반복 저장되지 않아 데이터 크기가 감소했습니다.

**성능 최적화**

별도의 테이블 조인 없이, 명함 데이터에서 관심사 데이터를 바로 조회할 수 있어 조회 성능이 향상되었습니다.

**유지보수성 향상**

관심사 추가 및 변경 시, ENUM 값만 수정하면 되므로 개발 속도가 빨라지고 유지보수가 쉬워졌습니다. 새로운 관심사가 추가되어도 데이터베이스 스키마를 변경할 필요가 없습니다.
