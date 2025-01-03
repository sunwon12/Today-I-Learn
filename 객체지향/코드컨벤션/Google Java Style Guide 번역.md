
본 글은 [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)를 번역한 글입니다.


# Google Java 스타일 가이드

## 목차

1. [소개](#1-소개)
2. [소스 파일 기본사항](#2-소스-파일-기본사항)
3. [소스 파일 구조](#3-소스-파일-구조)
4. [포맷팅](#4-포맷팅)
5. [네이밍](#5-네이밍)
6. [프로그래밍 관행](#6-프로그래밍-관행)
7. [Javadoc](#7-javadoc)

## 1. 소개

이 문서는 Java 프로그래밍 언어로 작성된 소스 코드에 대한 Google의 코딩 표준을 완전히 정의합니다. Java 소스 파일은 여기에 명시된 규칙을 준수할 때만 Google 스타일을 따른다고 말할 수 있습니다.

다른 프로그래밍 스타일 가이드와 마찬가지로, 여기서 다루는 문제는 형식에 관한 심미적 문제뿐만 아니라 다른 유형의 규칙이나 코딩 표준도 포함됩니다. 그러나 이 문서는 주로 우리가 보편적으로 따르는 엄격한 규칙에 초점을 맞추고 있으며, (사람이나 도구에 의해) 명확하게 시행할 수 없는 조언은 피합니다.

### 1.1 용어 참고사항

이 문서에서 달리 명시되지 않는 한:

1. '클래스'라는 용어는 "일반" 클래스, 열거형 클래스, 인터페이스 또는 애노테이션 타입(@interface)을 포괄적으로 의미합니다.
2. '멤버'(클래스의)라는 용어는 중첩 클래스, 필드, 메서드 또는 생성자를 포괄적으로 의미합니다. 즉, 초기화 블록과 주석을 제외한 클래스의 모든 최상위 내용을 의미합니다.
3. '주석'이라는 용어는 항상 구현 주석을 의미합니다. "문서 주석"이라는 문구를 사용하지 않고 대신 일반적인 용어인 "Javadoc"을 사용합니다.

### 1.2 가이드 참고사항

이 문서의 예제 코드는 비표준입니다. 즉, 예제가 Google 스타일을 따르긴 하지만, 코드를 표현하는 유일한 세련된 방법을 보여주는 것은 아닙니다. 예제에서 선택한 선택적 형식은 규칙으로 강제되어서는 안 됩니다.

## 2. 소스 파일 기본사항

### 2.1 파일명

소스 파일 이름은 포함된 최상위 클래스의 대소문자를 구분하는 이름(정확히 하나만 있음)과 .java 확장자로 구성됩니다.

### 2.2 파일 인코딩: UTF-8

소스 파일은 UTF-8로 인코딩됩니다.

### 2.3 특수 문자

#### 2.3.1 공백 문자

줄 종결자 시퀀스를 제외하고 ASCII 수평 공백 문자(0x20)만이 소스 파일의 어디에나 나타나는 유일한 공백 문자입니다. 이는 다음을 의미합니다:

1. 문자열과 문자 리터럴의 다른 모든 공백 문자는 이스케이프됩니다.
2. 탭 문자는 들여쓰기에 사용되지 않습니다.

#### 2.3.2 특수 이스케이프 시퀀스

특수 이스케이프 시퀀스(\b, \t, \n, \f, \r, \", \' 및 \\)가 있는 문자의 경우 해당 시퀀스가 사용됩니다.

#### 2.3.3 비 ASCII 문자

나머지 비 ASCII 문자의 경우 실제 유니코드 문자(예: ∞) 또는 동등한 유니코드 이스케이프(예: \u221e)가 사용됩니다. 선택은 코드를 더 쉽게 읽고 이해할 수 있게 하는 것에만 달려 있지만, 문자열 리터럴과 주석 외부에서 유니코드 이스케이프를 사용하는 것은 강력히 권장되지 않습니다.

팁: 유니코드 이스케이프를 사용하는 경우, 때로는 실제 유니코드 문자를 사용할 때도 설명 주석이 매우 도움이 될 수 있습니다.

예시:
```java
String unitAbbrev = "μs"; // 최상: 주석 없이도 완벽히 명확함
String unitAbbrev = "\u03bcs"; // "μs" - 허용되지만 이렇게 할 이유가 없음
String unitAbbrev = "\u03bcs"; // 그리스 문자 뮤, "s" - 허용되지만 어색하고 실수하기 쉬움
String unitAbbrev = "\u03bcs"; // 나쁨: 독자는 이것이 무엇인지 알 수 없음
```

팁: 일부 프로그램이 비 ASCII 문자를 제대로 처리하지 못할 것이라는 두려움 때문에 코드의 가독성을 떨어뜨리지 마세요. 그런 일이 발생하면 그 프로그램들이 고장 난 것이며 수정되어야 합니다.

## 3. 소스 파일 구조

소스 파일은 다음 순서로 구성됩니다:

1. 라이선스 또는 저작권 정보 (있는 경우)
2. 패키지 문장
3. 임포트 문장
4. 정확히 하나의 최상위 클래스

각 섹션은 정확히 한 개의 빈 줄로 구분됩니다.

### 3.1 라이선스 또는 저작권 정보 (있는 경우)

파일에 라이선스 또는 저작권 정보가 속한다면, 여기에 포함됩니다.

### 3.2 패키지 문장

패키지 문장은 줄 바꿈되지 않습니다. 열 제한(섹션 4.4, 열 제한: 100)은 패키지 문장에 적용되지 않습니다.

### 3.3 임포트 문장

#### 3.3.1 와일드카드 임포트 금지

와일드카드 임포트는 정적이든 아니든 사용되지 않습니다.

#### 3.3.2 줄 바꿈 금지

임포트 문장은 줄 바꿈되지 않습니다. 열 제한(섹션 4.4, 열 제한: 100)은 임포트 문장에 적용되지 않습니다.

#### 3.3.3 순서 및 간격

임포트는 다음 순서로 정렬됩니다:

1. 모든 정적 임포트를 단일 블록으로
2. 모든 비정적 임포트를 단일 블록으로

정적 임포트와 비정적 임포트가 모두 있는 경우 단일 빈 줄이 두 블록을 구분합니다. 임포트 문장 사이에 다른 빈 줄은 없습니다.

각 블록 내에서 가져온 이름은 ASCII 정렬 순서로 나타납니다. (참고: '.'는 ';' 앞에 정렬되므로 이는 임포트 문장이 ASCII 정렬 순서와 같지 않습니다.)

#### 3.3.4 클래스에 대한 정적 임포트 사용 금지

정적 중첩 클래스에 대해 정적 임포트를 사용하지 않습니다. 이들은 일반 임포트로 가져옵니다.

### 3.4 클래스 선언

#### 3.4.1 정확히 하나의 최상위 클래스 선언

각 최상위 클래스는 자체 소스 파일에 있습니다.

#### 3.4.2 클래스 내용의 순서

클래스의 멤버와 초기화 블록에 대해 선택하는 순서는 학습 가능성에 큰 영향을 미칠 수 있습니다. 그러나 이를 위한 단 하나의 올바른 방법은 없습니다; 다른 클래스들은 내용을 다른 방식으로 정렬할 수 있습니다.

중요한 것은 각 클래스가 유지 보수자가 설명할 수 있는 논리적 순서를 사용한다는 것입니다. 예를 들어, 새로운 메서드를 습관적으로 클래스의 끝에 추가하는 것은 "연대순"이라는 논리적 순서가 아닙니다.

#### 3.4.2.1 오버로드: 절대 분리하지 않음

같은 이름을 공유하는 클래스의 메서드는 중간에 다른 멤버 없이 연속적인 그룹으로 나타납니다. 여러 생성자도 마찬가지입니다(항상 같은 이름을 가집니다). 이 규칙은 메서드 간에 정적이나 private과 같은 수식어가 다를 때도 적용됩니다.

## 4. 포맷팅

용어 참고: block-like construct는 클래스, 메서드 또는 생성자의 본문을 가리킵니다. 배열 초기화 블록에 대한 섹션 4.8.3.1에 의해 모든 배열 초기화 블록은 선택적으로 block-like construct로 취급될 수 있습니다.

### 4.1 중괄호

#### 4.1.1 선택적 중괄호 사용

중괄호는 if, else, for, do 및 while 문에 사용됩니다. 본문이 비어 있거나 단일 문장만 포함하는 경우에도 마찬가지입니다.

람다 표현식의 중괄호와 같은 다른 선택적 중괄호는 선택적으로 유지됩니다.

#### 4.1.2 비어 있지 않은 블록: K & R 스타일

비어 있지 않은 블록과 block-like construct에 대해 중괄호는 Kernighan과 Ritchie 스타일("이집트 괄호")을 따릅니다:

- 여는 중괄호 앞에 줄 바꿈이 없습니다(아래에 자세히 설명된 경우 제외).
- 여는 중괄호 뒤에 줄 바꿈이 있습니다.
- 닫는 중괄호 앞에 줄 바꿈이 있습니다.
- 닫는 중괄호가 문장을 종료하거나 메서드, 생성자 또는 명명된 클래스의 본문을 종료하는 경우에만 닫는 중괄호 뒤에 줄 바꿈이 있습니다. 예를 들어, 중괄호 뒤에 else나 쉼표가 오는 경우에는 줄 바꿈이 없습니다.

예외: 이 규칙들이 세미콜론(;)으로 끝나는 단일 문장을 허용하는 경우, 문장 블록이 나타날 수 있으며 이 블록의 여는 중괄호 앞에 줄 바꿈이 있습니다. 이러한 블록은 일반적으로 switch 문 내부와 같이 지역 변수의 범위를 제한하기 위해 도입됩니다.

예시:

```java
return () -> {
  while (condition()) {
    method();
  }
};

return new MyClass() {
  @Override public void method() {
    if (condition()) {
      try {
        something();
      } catch (ProblemException e) {
        recover();
      }
    } else if (otherCondition()) {
      somethingElse();
    } else {
      lastThing();
    }
    {
      int x = foo();
      frob(x); 
    } 
   } 
};
```




열거형 클래스에 대한 몇 가지 예외는 섹션 4.8.1, 열거형 클래스에 제시되어 있습니다.

#### 4.1.3 빈 블록: 간결할 수 있음

빈 블록이나 block-like construct는 K & R 스타일일 수 있습니다(섹션 4.1.2에 설명된 대로). 또는 여는 중괄호 뒤에 문자나 줄 바꿈 없이 바로 닫을 수 있습니다({}). 단, 다중 블록 문(여러 블록을 직접 포함하는 문: if/else 또는 try/catch/finally)의 일부인 경우는 예외입니다.

예시:

```
  // 이것은 허용됩니다
  void doNothing() {}

  // 이것도 똑같이 허용됩니다
  void doNothingElse() {
  }
  // 이것은 허용되지 않습니다: 다중 블록 문에서는 간결한 빈 블록이 안 됩니다
  try {
    doSomething();
  } catch (Exception e) {}
```

### 4.2 블록 들여쓰기: +2 공백

새 블록이나 block-like construct가 열릴 때마다 들여쓰기가 2칸씩 증가합니다. 블록이 끝나면 들여쓰기는 이전 들여쓰기 레벨로 돌아갑니다. 들여쓰기 레벨은 블록 전체의 코드와 주석에 모두 적용됩니다. (섹션 4.1.2, 비어 있지 않은 블록: K & R 스타일의 예시를 참조하세요.)

### 4.3 한 줄에 하나의 문장

각 문장 뒤에는 줄 바꿈이 옵니다.

### 4.4 열 제한: 100

Java 코드의 열 제한은 100자입니다. "문자"는 모든 유니코드 코드 포인트를 의미합니다. 아래에 명시된 경우를 제외하고, 이 제한을 초과하는 모든 줄은 섹션 4.5, 줄 바꿈에 설명된 대로 줄 바꿈되어야 합니다.

예외:

1. 열 제한을 준수할 수 없는 줄(예: Javadoc의 긴 URL 또는 긴 JSNI 메서드 참조).
2. package 및 import 문(섹션 3.2 패키지 문장 및 3.3 임포트 문장 참조).
3. 셸에 복사하여 붙여넣을 수 있는 주석의 명령줄.
4. 매우 긴 식별자(드물게 필요한 경우)는 열 제한을 초과할 수 있습니다. 이 경우 주변 코드의 유효한 줄 바꿈은 google-java-format에 의해 생성된 대로 합니다.

### 4.5 줄 바꿈

용어 참고: 한 줄에 합법적으로 들어갈 수 있는 코드를 여러 줄로 나누는 것을 줄 바꿈이라고 합니다.

모든 상황에서 줄 바꿈을 정확히 어떻게 해야 하는지를 보여주는 포괄적이고 결정론적인 공식은 없습니다. 매우 자주 같은 코드 조각을 줄 바꿈하는 여러 가지 유효한 방법이 있습니다.

참고: 줄 바꿈의 전형적인 이유는 열 제한을 초과하지 않기 위해서지만, 실제로 열 제한 내에 들어갈 코드라도 작성자의 재량에 따라 줄 바꿈될 수 있습니다.

팁: 메서드나 지역 변수를 추출하면 줄 바꿈 없이 문제를 해결할 수 있습니다.

#### 4.5.1 어디서 줄 바꿈할 것인가

줄 바꿈의 핵심 지침은: 더 높은 구문 수준에서 줄 바꿈을 선호합니다. 또한:

1. 할당 연산자가 아닌 연산자에서 줄이 끊어질 때 줄 바꿈은 기호 이전에 옵니다. (이는 C++ 및 JavaScript와 같은 다른 언어의 Google 스타일에서 사용되는 관행과 다릅니다.)
   - 이는 다음과 같은 "연산자 유사" 기호에도 적용됩니다:
     - 점 구분자 (.)
     - 메서드 참조의 두 콜론 (::)
     - 타입 바운드의 앰퍼샌드 (<T extends Foo & Bar>)
     - catch 블록의 파이프 (catch (FooException | BarException e))
2. 할당 연산자에서 줄이 끊어질 때 줄 바꿈은 일반적으로 기호 이후에 오지만, 어느 쪽이든 허용됩니다.
   - 이는 향상된 for ("foreach") 문의 "할당 연산자 유사" 콜론에도 적용됩니다.
3. 메서드나 생성자 이름은 그 뒤에 오는 여는 괄호("(")에 붙어 있습니다.
4. 쉼표(",")는 그 앞에 오는 토큰에 붙어 있습니다.
5. 람다의 화살표 인접에서는 줄이 절대 끊어지지 않습니다. 단, 람다의 본문이 단일 중괄호 없는 표현식으로 구성된 경우 화살표 바로 뒤에 줄 바꿈이 올 수 있습니다. 예시:

```java
MyLambda<String, Long, Object> lambda =
    (String label, Long value, Object obj) -> {
        ...
    };

Predicate<String> predicate = str ->
    longExpressionInvolving(str);
```

참고: 줄 바꿈의 주된 목표는 명확한 코드를 갖는 것이지, 반드시 가장 적은 줄 수의 코드를 갖는 것은 아닙니다.

#### 4.5.2 들여쓰기 연속은 최소 +4 공백

줄 바꿈 시, 첫 줄(각 연속줄) 이후의 각 줄은 원래 줄에서 최소 +4만큼 들여씁니다.

여러 연속줄이 있을 때, 들여쓰기는 원하는 대로 +4 이상으로 다양화될 수 있습니다. 일반적으로, 두 개의 연속줄은 구문적으로 병렬 요소로 시작하는 경우에만 동일한 들여쓰기 수준을 사용합니다.

수평 정렬에 관한 섹션 4.6.3은 특정 토큰을 이전 줄과 정렬하기 위해 가변적인 수의 공백을 사용하는 것을 권장하지 않는 관행을 다룹니다.

### 4.6 공백

#### 4.6.1 수직 공백

다음과 같은 경우 항상 한 줄의 빈 줄이 나타납니다:

1. 클래스의 연속된 멤버 또는 초기화 블록 사이: 필드, 생성자, 메서드, 중첩 클래스, 정적 초기화 블록, 인스턴스 초기화 블록.
   - 예외: 두 개의 연속된 필드 사이의 빈 줄(그 사이에 다른 코드가 없는 경우)은 선택 사항입니다. 이러한 빈 줄은 필드의 논리적 그룹을 만들기 위해 필요에 따라 사용됩니다.
   - 예외: 열거형 상수 사이의 빈 줄은 4.8.1절에서 다룹니다.
2. 이 문서의 다른 섹션에서 요구하는 경우(예: 섹션 3, 소스 파일 구조 및 섹션 3.3, 임포트 문장).

클래스의 첫 번째 멤버나 초기화 블록 앞, 또는 마지막 멤버나 초기화 블록 뒤의 빈 줄은 권장되지도 금지되지도 않습니다.

여러 개의 연속된 빈 줄이 허용되지만, 절대 필요하지는 않습니다(또는 권장되지 않습니다).

#### 4.6.2 수평 공백

언어나 다른 스타일 규칙에 의해 요구되는 경우를 넘어, 리터럴, 주석 및 Javadoc을 제외하고 단일 ASCII 공백도 다음 위치에만 나타납니다:

1. if, for 또는 catch와 같은 예약어를 같은 줄에서 뒤따르는 여는 괄호("(")와 구분
2. else 또는 catch와 같은 예약어를 같은 줄에서 앞서는 닫는 중괄호("}")와 구분
3. 여는 중괄호("{") 앞에 두 가지 예외가 있습니다:
   - @SomeAnnotation({a, b}) (공백이 사용되지 않음)
   - String[][] x = {{"foo"}}; ({{, 사이에 공백이 필요하지 않음, 아래 항목 8에 의해)
4. 이항 또는 삼항 연산자의 양쪽. 이는 다음 "연산자 유사" 기호에도 적용됩니다:
   - 결합 타입 바운드의 앰퍼샌드: <T extends Foo & Bar>
   - 여러 예외를 처리하는 catch 블록의 파이프: catch (FooException | BarException e)
   - 향상된 for ("foreach") 문의 콜론 (:)
   - 람다 표현식의 화살표: (String str) -> str.length()
   하지만 다음에는 적용되지 않습니다:
   - 메서드 참조의 두 콜론 (::), 이는 Object::toString과 같이 작성됩니다
   - 점 구분자 (.), 이는 object.toString()과 같이 작성됩니다
5. ,:; 또는 캐스트의 닫는 괄호 ()) 뒤
6. // 로 시작하는 주석의 시작 전 모든 내용. 여러 개의 공백이 허용됩니다.
7. // 로 시작하는 주석의 시작과 주석 텍스트 사이. 여러 개의 공백이 허용됩니다.
8. 선언의 타입과 변수 사이: `List<String> list`  
9. 배열 초기화 블록의 중괄호 안쪽 양쪽에 선택적으로:
   new int[] {5, 6} 및 new int[] { 5, 6 } 모두 유효
10. 타입 어노테이션과 [] 또는 ... 사이

이 규칙은 줄의 시작이나 끝에 추가 공백을 요구하거나 금지하는 것으로 해석되지 않습니다; 이는 오직 내부 공백만을 다룹니다.

#### 4.6.3 수평 정렬: 절대 필요하지 않음

용어 참고: 수평 정렬은 코드에 추가 공백을 넣어 특정 토큰을 이전 줄의 다른 특정 토큰 바로 아래에 나타나게 하는 방식입니다.

이 방식은 허용되지만, Google 스타일에서 절대 요구되지 않습니다. 이미 사용된 곳에서 수평 정렬을 유지할 필요도 없습니다.

다음은 정렬하지 않은 예와 정렬한 예입니다:


```
private int x; // 이것도 괜찮습니다
private Color color; // 이것도 괜찮습니다

private int   x;      // 허용되지만 향후 편집으로
private Color color;  // 정렬이 깨질 수 있습니다
```

팁: 정렬은 가독성을 향상시킬 수 있지만, 그 자체를 위해 정렬을 유지하려는 시도는 향후 문제를 야기할 수 있습니다. 예를 들어, 한 줄만 수정하는 변경을 고려해 보세요. 이 변경으로 인해 이전의 정렬이 깨진다면, 주변 줄들에 추가적인 변경을 도입하여 다시 정렬하는 것은 중요하지 않습니다. 영향을 받지 않은 줄들에 형식 변경을 도입하는 것은 버전 기록을 손상시키고, 리뷰어의 속도를 늦추며, 병합 충돌을 악화시킵니다. 이러한 실제적인 우려사항들이 정렬보다 우선됩니다.

### 4.7 괄호 그룹화: 권장됨

선택적 그룹화 괄호는 작성자와 리뷰어가 괄호 없이는 코드가 잘못 해석될 합리적인 가능성이 없다고 동의하거나, 괄호가 코드를 더 쉽게 읽게 만들지 않을 것이라고 동의하는 경우에만 생략됩니다. 모든 독자가 전체 Java 연산자 우선순위 표를 외우고 있다고 가정하는 것은 합리적이지 않습니다.

### 4.8 특정 구조

#### 4.8.1 열거형 클래스

열거형 상수 뒤의 각 쉼표 다음에 줄 바꿈은 선택 사항입니다. 추가 빈 줄(보통 한 줄)도 허용됩니다. 다음은 한 가지 가능성입니다:

```java
private enum Answer {
  YES {
    @Override public String toString() {
      return "yes";
    }
  },

  NO,
  MAYBE
}
```

메서드가 없고 상수에 대한 문서가 없는 열거형 클래스는 선택적으로 배열 초기화와 같이 형식화될 수 있습니다(섹션 4.8.3.1 배열 초기화 참조).

```java
private enum Suit { CLUBS, HEARTS, SPADES, DIAMONDS }
```

열거형 클래스도 클래스이므로 클래스 형식화에 대한 다른 모든 규칙이 적용됩니다.

#### 4.8.2 변수 선언

##### 4.8.2.1 선언당 하나의 변수

모든 변수 선언(필드 또는 로컬)은 하나의 변수만 선언합니다: int a, b;와 같은 선언은 사용되지 않습니다.

예외: for 루프의 헤더에서는 여러 변수 선언이 허용됩니다.

##### 4.8.2.2 필요할 때 선언

지역 변수는 일반적으로 포함하는 블록이나 블록과 유사한 구조의 시작 부분에 습관적으로 선언되지 않습니다. 대신, 지역 변수는 그 범위를 최소화하기 위해 처음 사용되는 지점(합리적인 범위 내에서) 가까이에 선언됩니다. 지역 변수 선언에는 일반적으로 초기화 구문이 포함되거나, 선언 직후에 초기화됩니다.

#### 4.8.3 배열

##### 4.8.3.1 배열 초기화: "블록과 유사"할 수 있음

모든 배열 초기화는 선택적으로 "블록과 유사한 구조"로 형식화될 수 있습니다. 예를 들어, 다음은 모두 유효합니다(완전한 목록은 아님):

```java
new int[] {           new int[] {
  0, 1, 2, 3            0,
}                       1,
                        2,
new int[] {             3,
  0, 1,               }
  2, 3
}                     new int[]
                          {0, 1, 2, 3}
```

##### 4.8.3.2 C 스타일 배열 선언 금지

대괄호는 타입의 일부를 형성하며, 변수의 일부가 아닙니다: String[] args, String args[]가 아닙니다.

#### 4.8.4 switch 문

용어 참고: switch 블록의 중괄호 안에는 하나 이상의 문장 그룹이 있습니다. 각 문장 그룹은 하나 이상의 switch 레이블(case FOO: 또는 default:)로 구성되며, 그 뒤에 하나 이상의 문장이 옵니다(또는 마지막 문장 그룹의 경우 0개 이상의 문장).

##### 4.8.4.1 들여쓰기

다른 블록과 마찬가지로 switch 블록의 내용은 +2로 들여쓰기됩니다.

switch 레이블 다음에는 줄 바꿈이 있으며, 들여쓰기 수준이 +2 증가합니다. 마치 블록이 열린 것처럼 정확히 들여쓰기됩니다. 다음 switch 레이블은 이전 들여쓰기 수준으로 돌아갑니다. 마치 블록이 닫힌 것처럼 말입니다.

##### 4.8.4.2 Fall-through: 주석 처리

switch 블록 내에서 각 문장 그룹은 중단(break, continue, return 또는 예외 발생)되거나, 실행이 다음 문장 그룹으로 계속될 수 있거나 계속될 것임을 나타내는 주석으로 표시됩니다. fall-through 아이디어를 전달하는 모든 주석이면 충분합니다(일반적으로 // fall through). 이 특별한 주석은 switch 블록의 마지막 문장 그룹에는 필요하지 않습니다. 예:

```java
switch (input) {
  case 1:
  case 2:
    prepareOneOrTwo();
    // fall through
  case 3:
    handleOneTwoOrThree();
    break;
  default:
    handleLargeNumber(input);
}
```

case 1: 다음에는 주석이 필요하지 않고, 문장 그룹의 끝에만 필요합니다.

##### 4.8.4.3 default 케이스의 존재

각 switch 문은 코드가 없더라도 default 문장 그룹을 포함합니다.

예외: 열거형 타입에 대한 switch 문은 해당 타입의 모든 가능한 값을 다루는 명시적인 케이스가 있는 경우 default 문장 그룹을 생략할 수 있습니다. 이를 통해 IDE나 다른 정적 분석 도구가 누락된 케이스가 있는 경우 경고를 발행할 수 있습니다.

#### 4.8.5 어노테이션

##### 4.8.5.1 타입 사용 어노테이션

타입 사용 어노테이션은 주석이 달린 타입 바로 앞에 나타납니다. 어노테이션이 @Target(ElementType.TYPE_USE)로 메타 주석이 달린 경우 타입 사용 어노테이션입니다. 예:

```java
final @Nullable String name;

public @Nullable Person getPersonByName(String name);
```

##### 4.8.5.2 클래스 어노테이션

클래스에 적용되는 어노테이션은 문서화 블록 바로 다음에 나타나며, 각 어노테이션은 자체 줄에 나열됩니다(즉, 줄당 하나의 어노테이션). 이러한 줄 바꿈은 줄 바꿈(섹션 4.5, 줄 바꿈)을 구성하지 않으므로 들여쓰기 수준이 증가하지 않습니다. 예:

```java
@Deprecated
@CheckReturnValue
public final class Frozzler { ... }
```

##### 4.8.5.3 메서드 및 생성자 어노테이션

메서드 및 생성자 선언에 대한 어노테이션 규칙은 이전 섹션과 동일합니다. 예:

```java
@Deprecated
@Override
public String getNameIfPresent() { ... }
```

예외: 매개변수가 없는 단일 어노테이션은 선택적으로 서명의 첫 번째 줄과 함께 나타날 수 있습니다. 예:

```java
@Override public int hashCode() { ... }
```

##### 4.8.5.4 필드 어노테이션

필드에 적용되는 어노테이션도 문서화 블록 바로 다음에 나타나지만, 이 경우 여러 어노테이션(매개변수화될 수 있음)이 같은 줄에 나열될 수 있습니다. 예:

```java
@Partial @Mock DataLoader loader;
```

##### 4.8.5.5 매개변수 및 지역 변수 어노테이션

매개변수나 지역 변수에 대한 어노테이션 형식화에 대한 특정 규칙은 없습니다(물론 어노테이션이 타입 사용 어노테이션인 경우는 예외).

#### 4.8.6 주석

이 섹션에서는 구현 주석을 다룹니다. Javadoc은 섹션 7, Javadoc에서 별도로 다룹니다.

모든 줄 바꿈 앞에는 임의의 공백과 구현 주석이 올 수 있습니다. 이러한 주석은 해당 줄을 비어 있지 않은 것으로 만듭니다.

##### 4.8.6.1 블록 주석 스타일

블록 주석은 주변 코드와 동일한 수준으로 들여쓰기됩니다. /* ... */ 스타일이나 // ... 스타일일 수 있습니다. 여러 줄 /* ... */ 주석의 경우 후속 줄은 이전 줄의 *와 정렬된 *로 시작해야 합니다.

```java
/*
 * 이것은          // 그리고 이것도     /* 또는 이렇게도
 * 괜찮습니다.      // 괜찮습니다.        * 할 수 있습니다. */
 */
```

주석은 별표나 다른 문자로 그려진 상자에 둘러싸이지 않습니다.

팁: 여러 줄 주석을 작성할 때, 필요한 경우 자동 코드 포맷터가 줄을 다시 감쌀 수 있도록 /* ... */ 스타일을 사용하세요(단락 스타일). 대부분의 포맷터는 // ... 스타일 주석 블록의 줄을 다시 감싸지 않습니다.

#### 4.8.7 수식어

클래스 및 멤버 수식어는 Java 언어 사양에서 권장하는 순서로 나타납니다:

public protected private abstract default static final transient volatile synchronized native strictfp

#### 4.8.8 숫자 리터럴

long 값의 정수 리터럴은 대문자 L 접미사를 사용하며, 절대 소문자(숫자 1과 혼동을 피하기 위해)를 사용하지 않습니다. 예를 들어, 3000000000L 대신 3000000000l을 사용합니다.

## 5. 명명

### 5.1 모든 식별자에 공통되는 규칙

식별자는 ASCII 문자와 숫자만 사용하며, 아래에 언급된 소수의 경우에 밑줄을 사용합니다. 따라서 각 유효한 식별자 이름은 정규 표현식 \w+와 일치합니다.

Google 스타일에서는 특별한 접두사나 접미사를 사용하지 않습니다. 예를 들어, 이러한 이름은 Google 스타일이 아닙니다: name_, mName, s_name 및 kName.

### 5.2 식별자 유형별 규칙

#### 5.2.1 패키지 이름

패키지 이름은 모두 소문자로, 연속된 단어는 단순히 연결됩니다(밑줄 없음). 
예를 들어: com.example.deepspace, com.example.deepSpace나 com.example.deep_space가 아닙니다.

#### 5.2.2 클래스 이름

클래스 이름은 UpperCamelCase로 작성됩니다.

클래스 이름은 일반적으로 명사나 명사구입니다. 예를 들어, Character 또는 ImmutableList. 인터페이스 이름도 명사나 명사구일 수 있지만(예: List), 때로는 형용사나 형용사구일 수도 있습니다(예: Readable).

어노테이션 타입의 이름 지정에 대한 특정 규칙이나 잘 확립된 관행은 없습니다.

테스트 클래스의 이름은 Test로 끝납니다. 예를 들어, HashIntegrationTest. 단일 클래스를 다루는 경우, 그 클래스의 이름에 Test를 추가합니다. 예를 들어, HashImplTest.

#### 5.2.3 메서드 이름

메서드 이름은 lowerCamelCase로 작성됩니다.

메서드 이름은 일반적으로 동사나 동사구입니다. 예를 들어, sendMessage 또는 stop.

JUnit 테스트 메서드 이름에는 이름의 논리적 구성 요소를 구분하기 위해 밑줄이 나타날 수 있으며, 각 구성 요소는 lowerCamelCase로 작성됩니다. 한 가지 전형적인 패턴은 `<methodUnderTest>_<state>`, 예를 들어 pop_emptyStack. 테스트 메서드를 명명하는 단 하나의 정확한 방법은 없습니다.

#### 5.2.4 상수 이름

상수 이름은 UPPER_SNAKE_CASE를 사용합니다: 모든 대문자로, 각 단어는 단일 밑줄로 구분됩니다. 그러나 정확히 무엇이 상수인가요?

상수는 내용이 깊이 불변이고 메서드에 감지 가능한 부작용이 없는 정적 최종 필드입니다. 여기에는 기본 타입, 문자열, 불변 값 클래스 및 null로 설정된 모든 것이 포함됩니다. 인스턴스의 관찰 가능한 상태가 변경될 수 있다면 그것은 상수가 아닙니다. 단순히 객체를 절대 변경하지 않을 의도만으로는 충분하지 않습니다. 예시:

```java
// 상수
static final int NUMBER = 5;
static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");
static final Map<String, Integer> AGES = ImmutableMap.of("Ed", 35, "Ann", 32);
static final Joiner COMMA_JOINER = Joiner.on(','); // Joiner가 불변이므로
static final SomeMutableType[] EMPTY_ARRAY = {};

// 상수가 아님
static String nonFinal = "non-final";
final String nonStatic = "non-static";
static final Set<String> mutableCollection = new HashSet<String>();
static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
static final Logger logger = Logger.getLogger(MyClass.getName());
static final String[] nonEmptyArray = {"these", "can", "change"};
```

이러한 이름은 일반적으로 명사나 명사구입니다.

#### 5.2.5 비상수 필드 이름

비상수 필드 이름(정적이든 아니든)은 lowerCamelCase로 작성됩니다.

이러한 이름은 일반적으로 명사나 명사구입니다. 예를 들어, computedValues 또는 index.

#### 5.2.6 매개변수 이름

매개변수 이름은 lowerCamelCase로 작성됩니다.

공개 메서드에서 한 문자로 된 매개변수 이름은 피해야 합니다.

#### 5.2.7 지역 변수 이름

지역 변수 이름은 lowerCamelCase로 작성됩니다.

final이고 불변이더라도, 지역 변수는 상수로 간주되지 않으며 상수처럼 스타일을 지정해서는 안 됩니다.

#### 5.2.8 타입 변수 이름

각 타입 변수는 다음 두 스타일 중 하나로 명명됩니다:

- 단일 대문자, 선택적으로 단일 숫자가 뒤따름(예: E, T, X, T2)
- 클래스에 사용되는 형식의 이름(5.2.2 클래스 이름 참조) 뒤에 대문자 T가 옴(예: RequestT, FooBarT)

### 5.3 카멜 케이스: 정의

때로는 영어 구문을 카멜 케이스로 변환하는 합리적인 방법이 하나 이상 있을 수 있습니다. 예를 들어 "IPv6" 또는 "iOS"와 같은 약어나 특이한 구조가 있는 경우입니다. 예측 가능성을 높이기 위해 Google 스타일은 다음과 같은 (거의) 결정론적 체계를 지정합니다.

이름의 산문 형식으로 시작하여:

1. 구문을 일반 ASCII로 변환하고 아포스트로피를 제거합니다. 예를 들어, "Müller's algorithm"은 "Muellers algorithm"이 될 수 있습니다.
2. 이 결과를 단어로 나누고, 공백과 남아있는 구두점(일반적으로 하이픈)으로 분리합니다.
   - 권장: 어떤 단어가 이미 일반적인 사용에서 관례적인 카멜 케이스 모양을 가지고 있다면, 이를 구성 요소로 분리합니다(예: "AdWords"는 "ad words"가 됩니다). "iOS"와 같은 단어는 실제로 카멜 케이스가 아니므로 이 권장 사항이 적용되지 않습니다.
3. 이제 모든 것을 소문자로 만든 다음(약어 포함), 다음의 첫 글자만 대문자로 만듭니다:
   - ... 각 단어를 대문자로 시작하여 upper camel case를 만듭니다.
   - ... 첫 단어를 제외한 각 단어를 대문자로 시작하여 lower camel case를 만듭니다.
4. 마지막으로, 모든 단어를 하나의 식별자로 결합합니다.

원래 단어의 대소문자는 거의 완전히 무시됩니다. 예시:

| 산문 형식 | 정확한 표기 | 잘못된 표기 |
|----------|-----------|------------|
| "XML HTTP request" | XmlHttpRequest | XMLHTTPRequest |
| "new customer ID" | newCustomerId | newCustomerID |
| "inner stopwatch" | innerStopwatch | innerStopWatch |
| "supports IPv6 on iOS?" | supportsIpv6OnIos | supportsIPv6OnIOS |
| "YouTube importer" | YouTubeImporter<br>*YouTubeImporter | |

*허용되지만 권장되지 않습니다.

참고: 일부 단어는 영어에서 모호하게 하이픈이 사용됩니다. 예를 들어 "nonempty"와 "non-empty"는 모두 올바르므로 메서드 이름 checkNonempty와 checkNonEmpty도 마찬가지로 모두 올바릅니다.

## 6. 프로그래밍 관행

### 6.1 @Override: 항상 사용

@Override 어노테이션은 법적으로 가능할 때마다 메서드에 표시됩니다. 이는 슈퍼클래스 메서드를 오버라이드하는 클래스 메서드, 인터페이스 메서드를 구현하는 클래스 메서드, 슈퍼인터페이스 메서드를 재지정하는 인터페이스 메서드를 포함합니다.

예외: 부모 메서드가 @Deprecated인 경우 @Override를 생략할 수 있습니다.

### 6.2 예외 처리: 무시하지 않음

아래에 언급된 경우를 제외하고, 포착된 예외에 대해 아무것도 하지 않는 것은 매우 드물게 올바릅니다. (일반적인 대응은 로그를 남기거나, "불가능"하다고 간주되는 경우 AssertionError로 다시 던지는 것입니다.)

catch 블록에서 아무런 조치도 취하지 않는 것이 진정으로 적절한 경우, 그 이유를 주석으로 설명합니다.

```java
try {
  int i = Integer.parseInt(response);
  return handleNumericResponse(i);
} catch (NumberFormatException ok) {
  // 숫자가 아닙니다; 괜찮습니다, 그냥 계속 진행합니다
}
return handleTextResponse(response);
```

예외: 테스트에서 예외가 발생할 것으로 예상되는 경우, 포착된 예외의 이름이 expected로 시작하거나 expected인 경우 주석 없이 무시할 수 있습니다. 다음은 테스트 대상 코드가 예상 유형의 예외를 던지는지 확인하는 매우 일반적인 관용구이므로 여기서는 주석이 불필요합니다.

```java
try {
  emptyStack.pop();
  fail();
} catch (NoSuchElementException expected) {
}
```

### 6.3 정적 멤버: 클래스로 한정

정적 클래스 멤버에 대한 참조를 한정해야 할 때, 그 클래스의 참조나 표현식이 아닌 클래스의 이름으로 한정합니다.

```java
Foo aFoo = ...;
Foo.aStaticMethod(); // 좋음
aFoo.aStaticMethod(); // 나쁨
somethingThatYieldsAFoo().aStaticMethod(); // 매우 나쁨
```

### 6.4 Finalizers: 사용하지 않음

Object.finalize를 오버라이드하지 마세요.

팁: 절대로 finalize 메서드를 오버라이딩하지 마세요. finalize를 오버라이딩하면 발생할 수 있는 문제들을 피할 수 있습니다. 대신 자원 정리를 위해 try-with-resources나 try/finally 블록을 사용하세요.

## 7. Javadoc

### 7.1 형식

#### 7.1.1 일반 형식

Javadoc 블록의 기본 형식은 다음 예제와 같습니다:

```java
/**
 * 여러 줄의 Javadoc 텍스트는 여기에 작성되며,
 * 자동으로 줄 바꿈됩니다...
 */
public int method(String p1) { ... }
```

... 또는 이 한 줄 예제와 같습니다:

```java
/** 특히 짧은 Javadoc. */
```

기본 형식은 항상 허용됩니다. 한 줄 형식은 전체 Javadoc 블록(주석 마커 포함)이 한 줄에 들어갈 수 있을 때 대체될 수 있습니다. 이는 @return과 같은 블록 태그가 없는 경우에만 적용됩니다.

#### 7.1.2 단락

하나의 빈 줄 - 즉, 정렬된 선행 별표(*)만 포함하는 줄 - 은 단락 사이, 그리고 블록 태그 그룹이 있는 경우 그 앞에 나타납니다. 첫 번째 단락을 제외한 각 단락은 첫 단어 바로 앞에 `<p>`가 있으며, 그 뒤에 공백이 없습니다. 다른 블록 레벨 요소에 대한 HTML 태그(예: `<ul>` 또는 `<table>`)에는 `<p>`가 선행되지 않습니다.

#### 7.1.3 블록 태그

사용되는 표준 "블록 태그"는 @param, @return, @throws, @deprecated 순서로 나타나며, 이 네 가지 유형은 절대 빈 설명과 함께 나타나지 않습니다. 블록 태그가 한 줄에 맞지 않을 때, 연속 줄은 @의 위치에서 4개(또는 그 이상의) 공백으로 들여쓰기됩니다.

### 7.2 요약 단편

각 Javadoc 블록은 간단한 요약 단편으로 시작합니다. 이 단편은 매우 중요합니다: 클래스 및 메서드 색인과 같은 특정 컨텍스트에 나타나는 텍스트의 유일한 부분입니다.

이는 단편입니다 - 완전한 문장이 아닌 명사구 또는 동사구입니다. "{@code Foo}는 ..."로 시작하거나 "이 메서드는 ..."로 시작하지 않으며, "저장..."과 같은 완전한 명령형 문장 형태도 아닙니다. 그러나 이 단편은 마치 완전한 문장인 것처럼 대문자로 시작하고 구두점으로 끝납니다.

팁: 일반적인 실수는 단순히 /** @return 고객 ID */와 같은 형태로 Javadoc을 작성하는 것입니다. 이는 잘못된 것이며, /** 고객 ID를 반환합니다. */로 변경해야 합니다.

### 7.3 Javadoc 사용 위치

최소한 모든 public 클래스와 그러한 클래스의 모든 public 또는 protected 멤버에 대해 Javadoc이 존재합니다. 아래에 언급된 몇 가지 예외가 있습니다.

추가적인 Javadoc 내용도 섹션 7.3.4, 필수가 아닌 Javadoc에서 설명한 대로 존재할 수 있습니다.

#### 7.3.1 예외: 자명한 메서드

Javadoc은 "단순하고 명백한" 메서드(예: getFoo())의 경우 선택적입니다. 여기서 정말로 "foo를 반환합니다"라고 말하는 것 외에는 아무 말할 가치가 없는 경우에 한합니다.

중요: 이 예외를 인용하여 일반적인 독자가 알아야 할 수 있는 관련 정보를 생략하는 것은 적절하지 않습니다. 예를 들어, getCanonicalName이라는 메서드에 대해 일반적인 독자가 "정규 이름"이라는 용어가 무엇을 의미하는지 전혀 모를 수 있다면, 단순히 "/** 정규 이름을 반환합니다. */"라고만 말하기 위해 문서를 생략하지 마세요!

#### 7.3.2 예외: 오버라이드

상위 타입 메서드를 오버라이드하는 메서드에는 Javadoc이 항상 존재하지 않을 수 있습니다.

#### 7.3.4 필수가 아닌 Javadoc

다른 클래스와 멤버는 필요하거나 원하는 대로 Javadoc을 가질 수 있습니다.

구현 주석이 클래스나 멤버의 전반적인 목적이나 행동을 정의하는 데 사용될 때마다, 그 주석은 대신 Javadoc으로 작성됩니다(/** 사용).

필수가 아닌 Javadoc은 섹션 7.1.2, 7.1.3, 및 7.2의 형식 규칙을 엄격히 따를 필요는 없지만, 물론 권장됩니다.
