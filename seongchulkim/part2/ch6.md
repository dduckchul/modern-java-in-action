# 스트림으로 데이터 수집
* reduce 처럼 collect 역시 다양한 리듀싱 연산을 수행할 수 있다.
* 다양한 요소 누적 방식은 Collector 인터페이스에 있음

## 컬렉터란 무엇인가?
* 컬렉터 인터페이스는 스트림의 요소를 어떤 식으로 도출할지 지정한다.
* ex) toList, groupingBy

### 고급 리듀싱 기능을 수행하는 컬렉터
* collect로 결과를 수집하는 과정을 간단하면서도 유연한 방식으로 정의할 수 있다.
* 스트림에 collect를 호출하면 스트림의 요소에 리듀싱 연산이 수행
* Collectors 유틸리티 클래스는 자주 사용하는 컬렉터 인스턴스를 생성할 수 있는 정적 팩터리 메서드를 제공한다.
  * ex) Collectors.toList, Collectors.toMap

### 미리 정의된 컬렉터
* Collectors에서 제공하는 메서드의 기능은 크게 3가지
  * 스트림 요소를 하나의 값으로 리듀스하고 요약
  * 요소 그룹화
  * 요소 분할

### 리듀싱과 요약
* 스트림에서 최대값과 최소값 검색
  * 칼로리가 가장 높은 요리 찾을때
  * Collectors.maxBy,Collectors.minBy로 최대값, 최소값 탐색 가능

* 요약 연산
  * Collectors 클래스는 Collectors.summingInt라는 특별한 요약 팩터리 메서드를 제공한다.
  * summingInt는 객체를 Int로 매핑하는 함수를 인수로 받는다
  * summingInt의 인수로 전달된 함수는 객체를 int로 매핑한 컬렉터를 반환
  * summingLong, summingDouble 모두 비슷한 역할
  * 평균 값으로는 Collectors.averagingInt, averagingLong, averagingDouble 등이 있다.
  * summarizingInt는 카운트, 평균, 합계, 최대, 최소값 모두 계산해주는 IntSummaryStattistics 클래스를 반환해준다.

* 문자열 연결
  * Collectors.joining() -> toString 호출하여 추출한 모든 문자열을 하나의 문자열로 연결하여 반환한다.
  * 인자를 전달하면 사이에 넣을수 있도록 오버로드된 joining 메서드도 존재한다.

* 범용 리듀싱 요약 연산
  * 위의것들 모두 Collectors.reducing으로 구현 가능
  * 범용 팩터리 메서드 대신 특화된 컬렉터를 사용하는 이유는 편의성 + 가독성 때문
  * reducing은 인수 세 개를 받는다
    * 첫번째 인수는 리듀싱 연산의 시작값 or 스트림에 인수가 없을때는 반환값이다 (숫자 연산의 경우 0이 적합)
    * 두 번째 인수는 변환 함수
    * 세 번째 인수는 BinaryOperator
  * 인수가 두개인 reducing은 Optional\<T>로 구성되니 참고~

#### 컬렉션 프레임워크 유연성 : 같은 연산도 다양하게 수행 가능함
  * reducing 컬렉터를 이용한 예제에서 람다 표현식 되신 Integer의 sum메서드 참조 이용하면 코드를 좀더 단순화 가능하다.
  ``` java
  // summingInt
  int totalCalories = menu.stream()
  .collect(reducing(0,Dish::getCalories,Integer::sum));
  // count
  public static <T> Collector<T, ?, Long> counting() {
    return reducing(0L, e -> 1L, Long::sum);
  }
  ```

## 그룹화
* 팩토리 메서드 Collectors.groupingBy를 이용하여 메뉴를 쉽게 그룹화
* 이 함수를 기준으로 스트림이 그룹화 되므로, 분류함수라고 부른다.
* 단순한 속성 접근자 대신 더 복잡한 분류 기준이 필요한 상황에서는 메서드 참조를 사용할 수 없다.
* 이때는 메서드 참조 대신 람다 표현식으로 필요한 로직을 구현할것
``` java
public enum CaloricLevel {DIET, NORMAL, FAT}
Map<CaloricLevel> dishedByCalroricLevel = menu.stream().collect(
    groupingBy(dish -> {
        if(dish.getCalories() <= 400) return CaloricLevel.DIET;
        else if(dish.getCalories() <= 700) return CaloricLevel.NORMAL;
        else return CaloricLevel.FAT;
    })
)
```

### 그룹화된 요소 조작
* 그룹을 필터링할때, 미리 filter() 메서드를 사용한다면..? -> 스트림에서 이미 빠져서 키 값도 안만들어져서 그룹이 없어짐
  * groupingBy에서 predicate를 오버로딩 한 버전을 지원하므로, 거기서 필터링 한다면..? 비어있는 키의 그룹을 만들 수 있다!!
``` java
// 그룹지으면서 필터링
Map<Dish.Type, List<Dish>> caloricDishesByType = menu
.stream()
.collect(groupingBy(Dish::getType, filtering(dish -> dish.getCalories() > 500, toList())));
```

* 매핑 함수를 이용해 요소 변환도 가능
  * mapping 메서드는 또 다른 컬렉터를 인수로 받아서, 변환 가능
``` java
// 타입으로 그룹지어 메뉴 이름 리스트를 반환
Map<Dish.Type, List<String>> dishNamesByType = menu
.stream()
.collect(groupingBy(Dish::getType, mapping(Dish::getName, toList())));
```

* flatMapping을 이용하여 두가지 리스트를 한개로 합칠수도 있음

### 다수준 그룹화
* Collectors.groupingBy 는 일반적인 분류 함수와 컬렉터를 인수로 받는다 
  * groupingBy를 중첩하여 사용해서 여러개 그룹을 만들어줄 수 있음.
  * n수준 그룹화의 결과는 n수준 트리 구조로 표현되는 n수준 맵이된다.

### 서브그룹으로 데이터 수집
* 컬렉터의 형식에는 제한이 없다. toList()외에도 counting(), maxBy() 등등 모두 사용 가능

#### 컬렉터 결과를 다른 형식에 적용하기
* 마지막 그룹화 연산에서 맵의 모든 값을 Optional로 감쌀 필요가 없으므로, Collectors.collectingAndThen 으로 사용 가능
``` java
menu.stream().collect(
        groupingBy(Dish::getType,
            collectingAndThen(
                reducing((d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2),
                Optional::get)));
```
* groupingBy와 리듀싱, mapping 작업들도 많이한다

## 분할
* 분할함수라 불리는 특수한 그룹화 기능
* 분할함수는 boolean을 반환, 맵의 키는 boolean이다.
``` java
Map<Boolean, List<Dish>> partitionedMenu = menu.stream().collect(partitioningBy(Dish::isVegeterian));
```

* partitioningBy에는 두번째 인수를 컬렉터로 전달할수 있어서 다른것과 같이 쓸 수 있다
* 분할이란 특수한 종류의 그룹화
  * partitioningBy는 참/거짓만 포함하므로 더 간결

### 숫자를 소수와 비소수로 분할하기
* 정수 n -> 2~n까지의 자연수를 소수와 비소수로 나누는 프로그램을 구현.
* 주어진 수가 소수인지 아닌지를 판단하는 프레디케이트를 구현하면 편리하다.
  * 소수는 제곱근 이하의 수들만 검증하면 되니까 그런식으로 구현해보도록~ (제곱근 이상은 역순으로 한번더 검증하니까 더이상 검증 할 필요없음!)

* Collector의 정적 팩터리 메서드

|팩토리 메서드|반환 형식|
|----|---|
| toList | List\<T> |
| toSet | Set\<T> |
| toCollection | Collection\<T> |
| counting | Long |
| summingInt | Integer |
| averagingInt | Integer |
| summarizingInt | IntSummaryStatics |
| joining | String |
| maxBy | Optional\<T> |
| minBy | Optional\<T> |
| reducing | 리덕션 연산에서 반환하는 값 |
| collectiongAndThen | 다른 컬렉터 감싸고 그 결과에 변환 함수 적용 |
| groupingBy | Map<K, List\<T>> |
| partitioningBy | Map<Boolean, List\<T>>

* 이들 모든 컬렉터는 Collector 인터페이스를 구현한다.

## Collector 인터페이스
* 컬렉터 인터페이스는 리듀싱 연산 (컬렉터) 를 어떻게 구현할지 제공하는 메서드 집합으로 구성된다.
* 위에 보였던 소수 분류 문제를 다른 방법으로 해결하는 방법을 설명한다.

``` java
public interface Collector<T,A,R> {
  Supplier<A> supplier();
  BiConsumer<A,T> accumulator();
  Function<A,R> finisher();
  BinaryOperator<A> combiner();
  Set<Characteristics> characteristics(); 
}
```
* T는 수집될 스트림 항목의 제네릭
* A는 누적자, 즉 수집 과정에서 중간 결과를 누적하는 객체
* R은 수집 연산 결과 객체의 형식

### 컬렉터 인터페이스의 메서드 살펴보기
* 네개의 메서드는 collect 메서드에서 실행하는 함수를 반환, 다섯번째는 최적화를 이용해서 리듀싱 연산을 수행할건지 결정하도록 힌트 특성 집합을 제공한다.

* supplier 메서드 : 새로운 결과 컨테이너
  * 빈 결과로 이루어진 supplier를 반환
* accumulator 메서드 : 결과 컨테이너에 요소 추가하기
  * 리듀싱 연산을 수행하는 함수를 반환
* finisher 메서드 : 최종 변환값을 결과 컨테이너로 적용하기
  * 스트림 탐색을 끝내고 누적자 객체를 최종 결과로 변환, 누적 과정을 끝낼때 호출할 함수를 반환한다. 
  * ToList의 경우에는 누적자 개게착 이미 최종 결과이다.
  * 이때는 finisher 메서드가 항등 함수를 반환한다.
* combiner 메서드 : 두 결과 컨테이너 병합
  * 스트림의 서로 다른 서브 파트를 병렬로 처리할 때 이 누적자가 어떻게 결과를 처리할 지 정의한다.
  * 네번째 메서드를 이용하면 스트림의 리듀싱을 병렬로 수행 가능하다.
    * 스트림의 리듀싱을 병렬로 수행할때 포크/조인 프레임워크, Spliterator를 사용한다.
    * 스트림을 분할해야 하는지 정의하는 조건이 거짓으로 바뀌지 전까지 원래 스트림을 재귀적으로 분할한다. (일반적으로 프로세싱 코어의 갯수를 초과하는 병렬작업은 효율적이지 않다.)
    * 모든 서브스트림의 각 요소에 리듀싱 연산을 순차적으로 적용해서 서브스트림을 병렬로 처리할 수 있다.
    * 컬렉터의 combiner 메서드가 반환하는 함수로 모든 부분 결과를 쌍으로 합친다.
* Characteristics 메서드
  * 컬렉터의 연산을 정의하는 불볍 집합을 반환.
  * 스트림을 병렬로 리듀스 할 것인지, 병렬로 리듀스한다면 어떤 최적화를 선택해야 할지 힌트를 제공.
  * 다음 세 항목을 포함하는 열겨형이다
    * UNORDERED : 결과가 스트림 요소의 순서에 영향을 받지 않음
    * CONCURRENT : 다중 쓰레드에서 accumulator 함수를 동시에 호출 할 수 있으며, 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수 있다. 
      * UNORDERED를 같이 설정하지 않았다면 데이터가 정렬되어 있지 않은 상황에서만 병렬 리듀싱을 사용 가능하다.
    * IDENTITY_FINISH : 단순히 identitiy를 적용, 누적자 객체를 바로 사용할 수 있다.

### 컬렉터 구현 안만들고 커스텀하게 수지하기
* IDENTITY_FINISH 수집에서는 Collector 인터페이스를 완전 새로 구현하지 않고도 같은 결과를 얻을 수 있다. 
* Stream은 세 함수를 인수로 받는 collect를 오버로드한다.
``` java
List<Dish> dishes = menu.stream().collect(
  ArrayList::new, //supplier
  List::add, // accumulator
  List::addAll // combiner
)
```

## 커스텀 컬렉터를 구현하여 성능 개선하기
* 위 소수 예제 개선하기
### 소수로만 나누기
* 제수를 현재 숫자 이하에서 발견한 소수로 제한 할 수 있다.
  * 주어진 숫자가 소수인지 아닌지 접근해야함
  * 위 컬렉터 들에서는 컬렉터 수집 과정에서 부분 결과에 접근 할 수 없다.
  * 커스텀 컬렉터 클래스로 해결 가능하다.
  * 다른것은 동일하지만, accumlator에서 차이남
  ``` java
  public static boolean isPrime(List<Integer> primes, Integer candidate) {
    double candidateRoot = Math.sqrt(candidate);
    return primes.stream().takeWhile(i -> i <= candidateRoot).noneMatch(i -> candidate % i == 0);
  }
  // Biconsumer로 현재까지 중간 결과인 List<Integer>로 접근하여 소수 리스트 얻어낼 수 있다
  public BiConsumer<Map<Boolean, List<Integer>>, Integer> accumulator() {
    return (Map<Boolean, List<Integer>> acc, Integer candidate) -> {
      acc.get(isPrime(acc.get(true), candidate))
          .add(candidate);
    };
  }
  ```
* 알고리즘이 순차적이라서 병렬로 구현할 수는 없음

``` java
  public static void main(String ... args) {
    Instant instant = Instant.now();
    partitionPrimes(10000000);
    Instant instant2 = Instant.now();
    System.out.println("Time taken: " + (instant2.toEpochMilli() - instant.toEpochMilli()) + "ms");
    partitionPrimesWithCustomCollector(10000000);
    Instant instant3 = Instant.now();
    System.out.println("Time taken: " + (instant3.toEpochMilli() - instant.toEpochMilli()) + "ms");
  }
```

* 성능 테스트 해보니 아주 작은 그룹 에서는 기존게 유리하고, 큰 숫자에서는 커스텀이 유리한걸로 나옴 (100만개는 기존거가 유리!?)
  * 아마도..? 중간 연산 결과를 접근하는 펑션이, 아주 많은 소수를 탐색할때는 유리한듯 
  * 그러나 많지 않다면 중간 리스트에 접근하는것 저체가 퍼포먼스에 저하를 불러일으키는듯.