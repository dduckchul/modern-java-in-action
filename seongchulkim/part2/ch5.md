# 스트림 활용
* 데이터를 어떻게 처리할지는 스트림 api가 관리하므로, 편리하게 데이터 관련 작업을 할 수 있다.
* 스트림 api는 반복 뿐 아니라 코드를 병렬로 실행할지 여부도 결정할 수 있다. (단일 스레드로 순차 반복하는 외부 반복으로는 달성 안됨)
* 스트림 api가 지원하는 연산을 살펴보자

## 필터링
### Predicate 필터링
* prediecate를 인수로 받아서 일치하는 모든 요소 반환하는 스트림 반환.
~~~ java
List<Dish> vegetarianMenu = menu
.stream()
.filter(Dish::isVegetarian)
.collect(toList());
~~~

### 고유 요소 필터링
* 고유 요소로 반환하는 distinct() 메서드도 지원한다.
* 고유 여부는 스트림에서 만듣 객체의 hashCode, equals로 결정

## 스트림 자르기
* 자바 9는 스트림 요소를 효과적으로 선택할 수 있도록 takeWhile, dropWhile 등 새로운 메서드 지원한다.

#### 리스트가 칼로리 순으로 정렬 되었다고 가정
* takeWhile()로 반복작업을 중단할 수 있다.
  * 무한 스트림을 포함한 모든 스트림에 predicate적용해 스트림 슬라이스 가능하다.
* dropWhile()로 나머지 요소 선택 가능
  * predicate가 처음으로 거짓이 되는 지점까지 발견된 요소를 버린다.

### 스트림 축소
* 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환하는 메서드 limit(n)
* 소스가 정렬되어 있지 않다면 limit()도 정렬 안됨

### 건너뛰기
* 처음 n개 요소를 제외한 스트림 반환하는 skip(n) 메서드

## 매핑
* 특정 데이터를 선택하는 map과 flatMap 메서드

### 스트림의 각 요소에 함수 적용하기
* 함수를 인수로 받는 map 메서드를 지원
* 기존 값을 고친다기 보다는 새로운 버전을 만든다는 개념, mapping에 가깝다.

### 스트림 평면화
* flatMap() -> 각 배열을 스트림이 아니라 스트림의 컨텐츠로 매핑한다
  * map(Arrays::stream)과 달리 하나의 평면화된 스트림 생성
  * flatMap메서드는 스트림의 각 값을 다른 스틀미으로 만든다음에, 모든 스트림을 하나의 스트림으로 연결

## 검색과 매칭
* allMatch, anyMatch, nonMatch, findFirst, findAny등 제공
* 적어도 한 요소와 일치하는지 확인 : anyMatch
* 스트림의 모든 요소와 일치하는지 검사 : allMatch
* 주어진 predicate와 일치하는 요소가 없는지 검사 : noneMatch (위와 반대)
* 위 Match는 boolean 반환하므로, 최종연산이다.

### 요소 검색
* findAny는 현재 스트림에서 임의의 요소 반환
* 쇼트서킷을 이용하여 결과 찾는 즉시 실행 종료

#### Optional 이란?
* 값이 존재하는지, 부재하는지 표현하는 컨테이너 클래스
* findAny 등이 아무 요소도 반환하지 않을 수도 있다!
  * null 포인터 반환하므로 Optional을 만들었다
  * 일단 값이 존재하는지 확인하고, 없을때 어떻게 처리할지 강제함.

### 첫번째 요소 찾기
* findFirst()사용
  * 그런데 findAny()도 처음 만족하는거 찾으면 종료인데..? 머가다름?
  * 병렬성 이용할때 꼭 첫번째 찾아야하면 findFirst()
  * 병렬실행할때는 첫번째 요소를 찾기 어렵기 때문.
  * 뭐가 처음 나오던 상관없다면 findAny()로 사용할것.

## Reducing
* 리스트의 전체 합계 구하는? Integer 같은 결과가 나올때까지 줄이는것을 리듀싱이라고 한다.

``` java
// binaryOperator 사용
int product = numbers.stream().reduce(0, (a,b) -> a + b);
// 더 간편하게
int sum = numbers.stream().reduce(0, Integer::sum);
```

### 초기값 없음
* 초기값을 받지 않도록 오버로드 된 reduce도 있으나, 이떄는 Optional객체를 반환한다 (스트림에 아무 요소도 없으면 합계 없으므로)

``` java
// 최대값 찾기
Optional<Integer> max = numbers.stream().reduce(Integer::max);
// 최소값 찾기
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```

* map과 reduce를 연결하는 기법을 map-reduce 패턴이라 한다.
* 쉽게 병렬화 할수 있음, 구글이 웹 검색에 적용하면서 유명해졌다.

#### reduce 메서드의 장점과 병렬화
* 단계적 반복으로 합계를 구하는 것과 reduce를 이용해서 합계를 구하는것은 어떤 차이가 있을까?
* reduce를 이용하면 내부 반복이 추상화 되면서 내부 구현에서 병렬로 reduce를 실행할 수 있게 된다.
* 반복적인 합계에서는 sum 변수를 공유해야 하므로 쉽게 병렬화 하기 어렵다.
* 강제로 동기화 시키면 결국 병렬화로 얻는 이득이 스레드간 경쟁으로 인해 상쇄됨
* 후반에 포크조인 프레임워크 사용하는법 / 병렬화 나온다

#### 스트림 연산 상태 없음과 상태 있음
* 스트림 연산은 각각의 내부적인 상태를 고려해야 한다.
* map, filter 등은 입력 스트림에서 각 요소를 받아 0 또는 결과를 출력 리스트로 보낸다. 즉 보통 상태가 없는 내부 상태를 갖지 않는 연산이다.
* reduce, sum, max같은 연산은 결과를 누적할 내부 상태가 필요하다.
* 예제에서는 int, double을 사용했으나, 스트림에서 처리하는 요소 수와 관계없이 내부 상태의 크기는 한정되어있다.
* sorted나 distinct 같은 연산은 filter나 map 처럼 스트림을 입력으로 받아 다른 스트림을 출력하는 것처럼 보이나, filter나 map과는 다름.
  * 정렬 / 중복 제거는 과거의 이력을 알아야 하므로 보든 요소가 버퍼에 추가 되어 있어야 한다.
  * 데이터 스트림의 크기가 크거나 무한이라면 무제가 생긴다.
  * ex) 첫번째로 가장 큰 소수를 반환해야 한다면? (소수의 역순 sort)
    * 이러한 연산을 내부 상태를 갖는 연산이라 한다 (stateful operation)

## 숫자형 스트림
### 기본형 특화 스트림
* 박싱 비용을 피할 수 있도록 IntStream, DoubleStream, LongStream 등을 제공한다.
* 스트림을 특화 스트림으로 변환할 때는 mapToInt, mapToDouble, mapToLong 세가지 메서드 제공
  * map과 정확히 같은 기능을 수행하지만 Stream\<T> 대신 특화 스트림을 반환한다.

### 객체 스트림으로 복원하기
``` Java
// 스트림 -> 숫자 스트림
IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
// 숫자 스트림 -> 일반 스트림
Stream<Integer>stream = intStream.boxed();
```
### 기본값 : OptionalInt
* IntStream에서 최댓값 찾을때 0이라는 기본값 때문에 잘못 나올 수있음.
* 스트림에 요소가 업슨ㄴ 상황과, 실제 최대값이 0인 상황을 구분? ->
OptionalInt 등 기본형 특화 Optional로 알수 있다.

### 숫자 범위
* 특정 범위의 숫자를 이용해야 하는 상황이 발생
* IntStream, LongStream에서 range, rangeClosed 제공
  * range는 시작, 종료값이 포함 안됨, rangeClosed는 결과에 포함

#### 피타고리안 수 만들기
* 위에거 총집합해서 예쁘게 만드네요~ 책을 볼것 ^^...

## 스트림 만들기
### 값으로 스트림 만들기
* Stream.of 정적메서드
* Stream.empty() 사용해서 빈 스트림 만들수도 있음

### null이 될수 있는 객체로 스트림 만들기
* Stream.ofNullable() -> Stream.empty 아니면 스트림 만들어 준다.

### 배열로 스트림 만들기
* Arrays.stream()

### 파일로 스트림 만들기
* 자바의 NIO도 스트림 API 사용할수 있도록 바뀌었음
* Files의 많은 정적 메서드가 스트림을 반환한다
* Stream은 AutoClosable이므로 finally는 불필요함

### 함수로 스트림 만들기
* 함수에서 스트림을 만들수 있는 Stream.iterate, Stream.generate를 제공
* 무한 스트림 (크기가 고정되지 않는 스트림)을 만들 수 있다.
* iterate와 generate에서 만든 스트림은 요청할때마다 값을 만든다.
* 보통은 무한하게 하지 않도록 limit(n)을 연결함.
* java 9 이상의 iterate는 Predicate를 지원한다.

### generate 메서드
* generator는 Supplier\<T>를 인수로 받아 새로운 값을 생성

