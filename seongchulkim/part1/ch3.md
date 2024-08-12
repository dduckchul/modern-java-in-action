# Ch.3 람다 표현식
## 람다란 무엇인가?
* 익명 - 보통 이름이 없음
* 함수 - 특정 클래스에 종속되지 않음, 그러나 메서드처럼 파라미터 리스트, 바디, 반환형식, 예외 리스트를 포함
* 전달 - 람다 표현식을 메서드 인수로 전달하거나 변수로 지정할 수 있다
* 간결성 - 익명 클래스처럼 자질구레한 코드를 구현할 필요가 없다.
### 결론적으로 코드가 간결하고 유연해 진다

``` java
// comparator 객체
Comparator<Apple> = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight().compareTo(a2.getWeight());
    }
}

Comparator<Apple> byWeight = (Apple a2, Apple a2) -> a1.getWeight.compareTo(a2.getWeight());
```
* 람다는 위 처럼 세 부분으로 이뤄진다
  * 람다 파라미터 (메서드 파라미터와 동일)
  * 화살표 (파라미트와 바디를 구분)
  * 바디 (람다의 반환값)

### 표현식 스타일 람다
* (parameters) -> expression
### 블록 스타일 람다
* (parameters) -> {statements;}

## 어디에 어떻게 람다를 사용할까?
### 함수형 인터페이스
* 오직 하나의 추상 메서드만 지정하는 것
  * ex) comparator, runnable
* 함수형 인터페이스로 함수형 인터페이스의 추상 메서드 구현을 직접 전달할 수 있음
* 전체 표현식을 함수형 인터페이스의 인스턴스로 취급 할수 있다.

### 함수 디스크립터
* 추상 메서드 시그니쳐는 람다 표현식의 시그니쳐
* 시그니쳐를 표현하는 메서드를 함수 디스크립터라고 한다
* 자바 새로운 api중에는 @FunctionalInterface 로 선언한 것들 있음

## 실행 어라운드 패턴에 람다 적용하기
* 실행적인 예제 -> 순환패턴 자원 열고, 닫는 (보통 DB)
* 설정과 정리 작업은 비슷하다
* try-with-resources로 예제 구행

``` java
  public static String processFileLimited() throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(FILE))) {
      return br.readLine();
    }
  }
```
1. 동작 파라미터화를 기억할것
  * processFiles 을 동작 파라미터화 할것
  * ex)     String oneLine = processFile((BufferedReader b) -> b.readLine());

2. 함수형 인터페이스 사용해서 동작 전달할것
  * 함수형 인터페이스 자리에 람다를 사용할 수 있다.
``` java
  public interface BufferedReaderProcessor {

    String process(BufferedReader b) throws IOException;

  }
```
3. 동작 실행
* bufferedReader를 스트링으로 반환하는 시그니쳐를 만들었기 때문에, 일치하는 람다를 전달 할 수 있다.
``` java
  public static String processFile(BufferedReaderProcessor p) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(FILE))) {
      return p.process(br);
    }
  }
```

4. 람다 전달
* 이제 한 코드로 유연하게 두개의 실행을 처리할 수 있음
``` java
    String oneLine = processFile((BufferedReader b) -> b.readLine());
    System.out.println(oneLine);

    String twoLines = processFile((BufferedReader b) -> b.readLine() + b.readLine());
    System.out.println(twoLines);
```

## 함수형 인터페이스 사용
* Predicate
  * test 를 사용해 T 객체 사용하여 불리언을 반환
* Consumer
  * accept를 사용해 T 사용해서 Void 를 반환
* Function
  * T를 인수로 받아서 제네릭 형태 R을 반환 apply 정의
* 기본형 특화
  * 오토박싱 형태를 피할수 있도록 특별한 버전도 제공한다
  * intPredicate, intConsumer 등등등...
* 자세한 예제들은 책을 참고하도록!

## 형식 검사, 형식 추론, 제약
### 형식 검사
* 기대되는 람다 표현식의 형식을 대상 형식 이라고 한다.
* 컨텍스트를 이용하여 람다의 형식을 추론 할 수 있다.
### 같은 람다 다른 함수형 인터페이스
* 대상 형식이라는 특징 때문에 같은 람다 표현식이라도 호환되는 추상 메서드를 가진 다른 함수형 인터페이스로 사용 될 수 있음
### 형식 추론
* 대상 형식을 이용해서 함수 디스크립터를 알수 있기 때문에, 람다의 시그니쳐를 할 수 있다. 따라서 람다 파라미터의 형식을 추론 할 수 있다.
* 람다 파라미터가 여러개일때 가독성이 좋아진다. 정해진 규칙은 없음 잘 판단해서 적용할것
### 지역 변수 사용
* 익명 함수처럼 자유 변수를 활용 할 수 있다
* 람다 캡쳐링이라고도 함
* 다만 final이나 실질적 final인 변수만 가능함.
    * 파이널에만 사용가능한 이유는 지역변수는 태생부터 스택에 존재하기때문
    * 따라서 복제본만 저장 가능하다
    * 이런 제약이 있으므로 외부 변수를 변화시키는 것에 제동 걸수있음

## 메서드 참조
* 기존의 메서드 정의를 재활용 하여 람다를 사용할 수 있다.
* 떄로는 람다보다 메서드 참조하는게 훨씬 좋을 수 있음
``` java
    inventory.sort((a1, a2) -> a1.getWeight() - a2.getWeight());

    inventory.sort(comparing(Apple::getWeight));
```
* 메서드 참조는 특정 메서드만을 호출하는 람다의 축약형
* 명시적으로 메서드명을 참조함으로써 가독성을 높일 수 있다.

* 메서드 참조는 세가지 유형으로 구분 가능
  * 정적 메서드 참조 
  * 다양한 형식의 인스턴스 메서드 참조
  * 기존 객체의 인스턴스 메서드 참조
* 생성자, 배열 생성자, super 호출 등에 사용하는 특별한 메서드 참조도 있음


## 람다 표현식을 조합할 수 있는 유용한 메서드 - default method
* Comparator 조합
  * 역정렬 (reversed)
  * 정렬 거꾸로 하고싶다면? 새로 만들 필요없음
  * 인터페이스에서 비교자의 순서를 뒤바꾸는 reversed를 지원함.
  ``` java
  // 비교자 구현 재사용 해서 역순 정렬 구현
  inventory.sort(comparing(Apple::getweight).reversed())
  ```
* Comparator 연결
  * 무게가 같은 두 사과가 존재할경우?
  * 비교 결과를 다듬을 수 있는 두번째 comparator 만든다
  * thenComparing 은 첫번째가 같다고 판단되면 두번째 비교자에 전달한다.
  ``` java
  // 같을경우 두번째 비교자에 전달
  inventory.sort(
    comparing(Apple::getWeight())
      .reversed()
      .thenComparing(Apple::getCountry)
    )
  ```

* Predicate 조합
  * negate (기존 프레디케이트 결과 반전), or, and를 지원
  * 단순한 람다표현식으로 복잡한 Predicate 만들 수 있다.

* Function 조합
  * andThen, compose 두가지 디폴트 메서드 제공
  ``` java
  Function<Integer, Integer> f = x -> x + 1;
  Function<Integer, Integer> g = x -> x * 2;
  Function<Integer, Integer> h = f.andThen(g);
  int result = h.apply(1); // g(f(x))를 수행, 4 반환

  Function<Integer, Integer> h2 = f.compose(g);
  int result = h2.apply(1); // f(g(x))를 수행, 3 반환
  ```

## 비슷한 수학적 개념은 넘깁시다 후후

### 마무리
* 람다 표현식은 익명 함수의 일종, 이름은 없지만 파라미터, 바디, 반환값을 가지며 예외를 던질 수 있다.
* 람다 표현식으로 간결한 코드를 만들 수 있다.
* 함수형 인터페이스는 하나의 추상 메서드만을 정의하는 인터페이스이다.
* 함수형 인터페이스를 기대하는 곳에서만 람다 표현식을 사용 가능하다.
* 람다 표현식을 이용해서 함수형 인터페이스의 추상 메서드를 즉석으로 제공 할 수 있다.
* 람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급된다
* java.util.function 패키지는 자주 사용하는 다양한 함수형 인터페이스를 제공한다.
* 박싱 동작을 피할 수 있는 기본형 특화 인터 페이스를 제공한다.
* 실행 어라운드 패턴을 사용하면 유연성과 재사용성을 추가로 얻을 수 있다.
* 메서드 참조를 사용하면 기존의 메서드 구현을 재사용하고 직접 전달할 수 있다.
* Comparator, Predicate, Function 같은 함수형 인터페이스는 람다 표현식을 조합할 수 있는다양한 디폴트 메서드를 제공한다.