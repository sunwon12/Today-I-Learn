### **상위 public 메소드로 하위 private 메소드를 테스트 할 때 문제점**

extractNumbers메소드에서 호출되는 메소드들은 다 private 메소드들입니다. 각 기능들을 테스트 하기 위해 extractNumbers메소드를 호출하여 결과중심의 테스트로 private 메소드들을 테스트하였습니다. 문제점은 테스트에 실패했을 때 어느 메소드에서 실패했는지 모르는 것입니다. `유지보수를 위해 잘게 메소드로 나누었지만`, 어느 메소드가 잘못됐는지 알지 못하는 `모순되는 상황에 봉착` 했습니다.

```java
```java
// private 메소드 양만 봐도 어지럽습니다..

public class StringParser {

    private static final String BLANK = "";
    private static final String DEFAULT_SEPARATOR_COMMA = ",";
    private static final String DEFAULT_SEPARATOR_COLON = ":";
    private static final String CUSTOM_SEPARATOR_DEFINITION_PREFIX = "//";
    private static final String CUSTOM_SEPARATOR_DEFINITION_SUFFIX = "\\n";
    private static List<String> separators = Arrays.asList(DEFAULT_SEPARATOR_COMMA, DEFAULT_SEPARATOR_COLON);

    public List<Integer> extractNumbers(String input) {
        if (isBlank(input)) {
            return Collections.singletonList(0);
        }

        String customSeparator = extractCustomSeparator(input);
        String numberString = removeCustomSeparatorDefinition(input);
        List<String> allSeparators = getAllSeparators(customSeparator);
        String[] numberStrings = splitByAllSeparators(numberString, allSeparators);

        return convertToNumbers(numberStrings);
    }

    private List<String> getAllSeparators(String customSeparator) {
		...
    }

    private String[] splitByAllSeparators(String input, List<String> allSeparators) {
        createSeparatorRegex(allSeparators);
		...
    }

    private String createSeparatorRegex(List<String> allSeparators) {
		...
    }

    private List<Integer> convertToNumbers(String[] numberStrings) {
		...
    }

    private int parseNumber(String number) {
	    ...
    }

    private boolean isBlank(String input) {
	    ...
    }

    private boolean isSeparator(String potentialSeparator) {
        ...
    }

    private String extractCustomSeparator(String input) {
       findCustomSeparatorEndIndex(input);
	    ...
    }

    private String removeCustomSeparatorDefinition(String input) {
        findCustomSeparatorEndIndex(input);
	    ...
    }

    private int findCustomSeparatorEndIndex(String input) {
	    ...
    }
}

```
```

---

### **private 메소드는 어떻게 테스트 할까**

- Do not test
말그대로 테스트를 하지 않는 방법
- Java `Reflection` 사용하기
동적으로 호출하여 접근 지시자를 우회하는 방법
- Spring `ReflectionTestUtils` 사용
동적으로 호출하여 접근 지시자를 우회하는 방법
- 접근 지시자 느슨하게 하기`private` 지시자를 `protected` 또는 기본 지시자로 변경하여 테스트에서 접근 가능하도록 만드는 방법
여기에 더해 _테스트를 위해서 지시자를 느슨하게 만들었다는 사실_을 명시적으로 선언하기 위해서 Guava의 `@VisibleForTesting` 어노테이션을 이용 가능
- 상위 public 메소드를 통해 테스트하기
    - 많은 하위 private 메소드를 담고 있다면, 어느 하위 private 메소드가 잘못된지 파악하기 힘듦
- **해당 객체의 private 메소드로서의 책임이 맞는지 반문해보기.**

마이클 C. 페더스 - <레거시 코드 활용 전략>  책에서 아래와 같이 나옵니다.

> 어떻게든 private 메서드를 테스트하고 싶다면, 그 메서드는 private이면 안된다"다. 메서드를 public으로 바꿔도 될지 마음에 걸린다면, 이 메서드는 별도의 책임의 일부로서 원래는 다른 클래스에 들어있어야 했던 것이다.
> 

 오히려 책임과 역할을 제대로 이해하지도 또 분리하지 못한 채, 정보은닉이라는 핑계로 `private`으로 가시성을 제약하고 안심했을 뿐이었던 것이다. 이 사례는 결국 `private` 접근 지시자를 `protected` 를 넘어서 `public` 으로 만들어야 하는 경우다.

---

### **객체 분리 예시**

객체 분리의 예시를 들기 위해 나누어보겠습니다. 아래 코드는 예시를 들기 위한 1차분리입니다.(더 나은 객체지향 형태가 있다고 생각하시더라도 참아주세요..) 

그래서 StringParser에서 아래 메소드들을 Separator로 따로 분리시켰습니다.
또한 static final로 선언한 분리자 관련 속성들도 분리시켰습니다.

1. 커스텀 분리자를 추출한다
2. 커스텀 분리자를 제거한다
3. 모든 커스텀 분리자를 가져온다
4. 커스텀 분리자로 문자에서 숫자를 분리하다

```java
public class StringParser {

	private static final String BLANK = "";
	private final Separator separator = new Separator();

	public List<Integer> extractNumbers(String input) {
		if (isBlank(input)) {
		return Collections.singletonList(0);
	}

		String customSeparator = separator.extractCustomSeparator(input);
		String numberString = separator.removeCustomSeparatorDefinition(input);
		List<String> allSeparators = separator.getAllSeparators(customSeparator);
		String[] numberStrings = separator.splitByAllSeparators(numberString, allSeparators);

		return convertToNumbers(numberStrings);
	}

	private List<Integer> convertToNumbers(String[] numberStrings) {
		return Arrays.stream(numberStrings)
		.map(this::parseNumber)
		.toList();
	}

	private int parseNumber(String number) {
		try {
		return Integer.parseInt(number);
		} catch (NumberFormatException e) {
		throw new IllegalArgumentException("유효하지 않은 숫자 형식입니다: " + number);
		}
	}

	public boolean isBlank(String input) {
		return input.equals(BLANK);
	}
}

```

```java

public class Separator {

	private static final String DEFAULT_SEPARATOR_COMMA = ",";
	private static final String DEFAULT_SEPARATOR_COLON = ":";
	private static final String CUSTOM_SEPARATOR_DEFINITION_PREFIX = "//";
	private static final String CUSTOM_SEPARATOR_DEFINITION_SUFFIX = "\\\\n";
	private static List<String> separators = Arrays.asList(DEFAULT_SEPARATOR_COMMA, DEFAULT_SEPARATOR_COLON);

	public String extractCustomSeparator(String input) {
		int endIndex = findCustomSeparatorEndIndex(input);
		if (endIndex == -1) {
		return null;
		}
		return input.substring(CUSTOM_SEPARATOR_DEFINITION_PREFIX.length(), endIndex);
	}

	public String removeCustomSeparatorDefinition(String input) {
		int endIndex = findCustomSeparatorEndIndex(input);
		if (endIndex == -1) {
		return input;
		}
		return input.substring(endIndex + CUSTOM_SEPARATOR_DEFINITION_SUFFIX.length());
	}

	private int findCustomSeparatorEndIndex(String input) {
		if (!input.startsWith(CUSTOM_SEPARATOR_DEFINITION_PREFIX)) {
		return -1;
		}
		int endIndex = input.indexOf(CUSTOM_SEPARATOR_DEFINITION_SUFFIX);
		if (endIndex == -1) {
		throw new IllegalArgumentException("커스텀 구분자 형식이 올바르지 않습니다.");
		}
		return endIndex;
	}

	public List<String> getAllSeparators(String customSeparator) {
		List<String> allSeparators = new ArrayList<>(separators);
		if (customSeparator != null) {
		allSeparators.add(customSeparator);
		}
		return allSeparators;
	}

	public String[] splitByAllSeparators(String input, List<String> allSeparators) {
		String regex = createSeparatorRegex(allSeparators);
		return input.split(regex);
	}

	private String createSeparatorRegex(List<String> allSeparators) {
		String separatorPattern = allSeparators.stream()
		.map(Pattern::quote)
		.collect(Collectors.joining("|"));
		return "(" + separatorPattern + ")";
	}

}

```

---

### **메소드를 private으로 설정할 때 고려해야 할 점**

- private 메소드 사용 원칙:
    - 클래스 내부에서만 사용되는 메소드
    - 구현 세부사항을 캡슐화하기 위해
    - 외부에 노출될 필요가 없는 헬퍼 메소드
- 하위 메소드이지만, private을 고려해야 하는 상황
    - 모든 하위 메소드가 자동으로 private이 되어야 하는 것은 아닙니다.
    - 외부에서 호출될 재사용 가능성을 고려해야 합니다.

---

참고
[https://blog.kmong.com/private-메서드는-어떻게-테스트-하나요-2069f2e2216e](https://blog.kmong.com/private-%EB%A9%94%EC%84%9C%EB%93%9C%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%95%98%EB%82%98%EC%9A%94-2069f2e2216e)
