# 동적 파라미터화
- 리스트의 모든 요소에 대해서 '어떤 동작'을 수행할 수 있음
- 리스트 관련 직업을 끝낸 다음에 '어떤 다른 동작'을 수행할 수 있음
- 에러가 발생하면 '정해진 어떤 다른 동작'을 수행할 수 있음
#### 프레디케이트(predicate)
- 참 또는 거짓을 반환하는 함수
```java
public interface ApplePredicate {
  boolean test (Apple apple);
}
```

```java
static class AppleWeightPredicate implements ApplePredicate {
  @Override
  public boolean test(Apple apple) {
    return apple.getWeight() > 150;
  }
}
```

#### 추상적 조건으로 필터링
- 코드/동작 전달하기
```java
public class AppleRedAndHeavyPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return "red".equals(apple.getColor()) && apple.getWeight() > 150;
  }
}
```
```java
List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPRedicate());
```
- 한 메서드가 다른 동작을 수행하도록 재활용할 수 있다.

### 익명 클래스
- 이름이 없는 클래스이며 즉석에서 필요한 구현을 만들어서 사용할 수 있다.
```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
  public boolean test(Apple apple) {
    return RED.equals(apple.getColor());
  }
});
```
- 부족한 점: 장황한 코드, 개발자가 사용에 익숙하지 않음

### 람다 표현식
```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
```

### 리스트 형식의 추상화
```java
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
  List<T> result = new ArrayList<>();
  for (T e : list) {
    if (p.test(e)) {
      result.add(e);
    }
  }
}
```
### Comparator로 정렬
- Java 8의 List에는 sort 메소드가 포함되어 있으며 java.util.Comparator 객체를 이용하여 sort의 동작을 파라미터화 할 수 있다.
```java
inventory.sort(new Comparator<Apple>() {
  public int compare(Apple a1, Apple a2) {
    return a1.getWeight().compareTo(a2.getWeight());
  }
})
```
```java
inventory.sort((Apple apple) ->  return a1.getWeight().compareTo(a2.getWeight()));
```

### Runnable 실행
```java
Thread T = new Thread (() -> System.out.println("Hello world"));
```

### Callable 결과 반환
- ExecutorService를 이용하면 태스크를 스레드 풀로 보내고 결과를 Future로 저장할 수 있음
```java
Future<String> threadNAme = executorService.submit(() -> Thread.currentThread().getName());
```

### Summary
- 동적 파라미터화에서는 코드를 메서드 인수로 전달 가능하다.
- 동적 파라미터화를 이용하면 유지보수에 용이한 코드 작성이 가능하다
- 코드 전달기법을 이용하면 동작을 메서드의 인수로 전달할 수 있다.
