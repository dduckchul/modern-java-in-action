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