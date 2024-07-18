# 동작 파라미터화 코드 전달하기
* 동작 파라미터화를 사용하면 변화하는 요구사항에 잘 대응 할 수 있다.
* 동작 파라미터화란?
  * 아직은 어떻게 실행 할지 결정하지 않은 코드, 나중에 프로그램에서 호출한다.
  * 변화하는 요구사항에 유연하게 대응할수 있도록 코드를 구현하는 방법

## 녹색 사과 필터링
```
  enum Color {Red, Green}

  public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getColor() == Color.GREEN) {
        result.add(apple);
      }
    }
    return result;
  }
```
* 여기서 새로운 색상을 필터링 하고싶다면 메서드를 하나 더 만들어야함..
* 거의 비슷한 반복 코드가 있다면 코드를 추상화 할 것

### 색을 파라미터화 하기
```
  public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getColor() == color) {
        result.add(apple);
      }
    }
    return result;
  }
```
* 색을 인자로 넘겨줘서 좀더 유연하게 대응

* 그런데, 갑자기 무게로 필터링이 필요하다면..?

```
  public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (apple.getWeight() > weight) {
        result.add(apple);
      }
    }
    return result;
  }
```
* 또 이런 별도의 메서드를 만들어줘야할까..?
* 색 필터링, 무게 필터링 하는 외의 부분은 거의 다 비슷함

### 가능한 모든 속성으로 필터링하기
* 플래그를 사용해서 모든 속성을 파라미터로 전달하는 코드..
  * 요구사항 바뀌었을떄 유연하게 대응 불가능
  * 메서드 파라미터가 무엇을 의미하는지도 파악 불가능..

## 동작 파라미터화
* 변화하는 요구사항에 좀더 유연하게 대응할수 있는 방법이 필요하다
* 참 거짓을 반환하는 Predicate 함수

```
  interface ApplePredicate {

    boolean test(Apple a);

  }

  static class AppleWeightPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getWeight() > 150;
    }

  }

  static class AppleColorPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
      return apple.getColor() == Color.GREEN;
    }

  }
```
* filter 메서드를 다르게 적용하여 동작하도록 만들어준다
* 전략 디자인 패턴 -> 알고리즘을 캡슐화하는 알고리즘 패밀리들을 정의, 런타임에 알고리즘을 선택하는 기법
* 이렇게 짠다면 엔지니어링 적으로 큰 이득을 얻을 수 있다.

### 추상적 조건으로 필터링
```
  public static List<Apple> filter(List<Apple> inventory, ApplePredicate p) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
      if (p.test(apple)) {
        result.add(apple);
      }
    }
    return result;
  }
```
* 뭔가 필터링을 추가해야 한다면, 새로운 클래스를 정의하면 끝!
* ApplePredicates 만 전달하고, filterApples는 그 안의 동작을 전달 한다.

* 동작 파라미터화의 강점
  * 컬렉션의 탐색 로직과 필터링 하는 로직을 분리 할 수 있다.
  * 한 메서드가 다른 여러가지 동작들을 실행할수 있도록 해짐
  * 유연한 API를 만드는데 큰 도움 준다

## 복잡한 과정 간소화
* 익명 클래스
  * 익명 클래스를 사용하면 클래스의 선언과 인스턴스화를 동시에 수행 가능
  * 블록 내부에 선언한 클래스 이다

### 익명 클래스의 사용
```
    List<Apple> redApples2 = filter(inventory, new ApplePredicate() {
      @Override
      public boolean test(Apple a) {
        return a.getColor() == Color.RED;
      }
    });
```
* FX자바등 GUI인터페이스에서 많이 쓰던 방식
* 이벤트 핸들러 등에서는 여전히 많이 쓰임
* 코드를 줄일수는 있지만 결국 객체를 명시적으로 만들고, 새로운 동작 정의하는 메서드 구현해야 하는 코드가 늘어난다.
* 자바 8에서는 람다를 통해 간단해짐

### 람다 표현식 사용
```
List<Apple> result = filterApples(inventory, (Apple apple) -> Red.equals(apple.getColor))
```

### 리스트 형식으로 추상화
```
public interface Predicate<T> {
    boolean test(T t)
}

public static <T> List<T> filter(List<T> list, Predicate<T> p){
    List<T> result = new ArrayList<>();
    for (T e : list){
        if(p.test(e)){
            result.add(e);
        }
    }
    return result;
}

List<Apple> redApples = filter(inventory, (Apple apple) -> Red.equals(apple.getColor()))
```
위와 같이 쓸 수 있다~

### 실전에서는..?
* Comparator 객체의 Sort 정의
* Runnable run 의 동작
* Callable의 call 동작
* EventHandler 의 handle 동작

## 동작 파라미터 정리
* 내부적으로 다양한 동작을 수행할 수 있도록 인수로 전달한다.
* 동작 파라미터화를 이용하면 변화하는 요구사항에 더 잘 대응할수 있는 코드 구현 가능하다
* 동작을 메서드의 인수로 전달 할 수 있다. 그러나 자바 8 이전에는 좀 깔끔하지 못했음.
* 자바 Api 들의 많은 메서드는 다양한 동작으로 파라미터화 할 수있다.
