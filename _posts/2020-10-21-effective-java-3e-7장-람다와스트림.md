---
title: effective java 3/E - 7장. 람다와 스트림
tags: 
 - Java
key: 12
---

# 7장 람다와 스트림

##### 7장에서 배울 내용

* 자바 8에서 함수형 인터페이스, 람다, 메서드 참조라는 개념이 추가되고 함수 객체를 더 쉽게 만들 수 있다.
* 스트림 API 까지 추가되어 데이터 원소의 시퀀스 처리를 라이브러리 차원에서 지원하기 시작했다.
* 이 기능들을 효과적으로 사용하는 방법들을 알아본다.



___



## 아이템 42: 익명 클래스보다는 람다를 사용하라

예전에는 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스(드물게는 추상 클래스)를 사용했다. 이런 인터페이스 함수를 객체라고 하여, 특정 함수나 동작을 나타내는데 썼다.
JDK 1.1 이후 함수 객체를 만드는 주요 수단은 익명 클래스가 되었다. 

##### 익명 클래스의 인스턴스를 함수 객체로 사용 - 낡은 기법

~~~java
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2){
        return Integer.compare(s1.length(), s2.length());
    }
});
~~~

Comparator 인터페이스가 정렬을 담당하는 추상 전략을 뜻하며, 문자열을 정렬하는 구체적인 전략을 익명 클래스로 구현했다. 하지만, 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않다.

지금은 함수형 인터페이스라 부르는 이 인터페이스들의 인스턴스를 **람다식(lambda expression; 람다)**을 사용해 만들 수 있다.

##### 람다식을 함수 객체로 사용 - 익명클래스 대체

~~~java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
~~~

* 람다 매개변수 s1, s2와 반환 값의 타입은 각각 (Comparator\<String\>), String, int 지만 코드에는 언급이 없다. 
* 컴파일러가 문맥을 보고 타입을 추론해준다.
* 컴파일러가 타입을 결정하지 못할 때는 직접 명시해야 한다.
* **타입을 명시해야 코드가 더 명확할 때만 제외하고, 람다의 모든 매개변수 타입은 생략하자**

##### 람다 자리에 비교자 생성 메서드 사용 - 더 간결하게 만들 수 있다.

~~~java
Collections.sort(words, comparingInt(String::length));
~~~

##### 자바 8에서 List 인터페이스에 추가된 sort - 더 더 간결하다.

~~~java
words.sort(comparingInt(String::length));
~~~

##### 아이템 34의 Operation 열거 타입 예제 - 람다로 구현

~~~java
public enum Operation {
    PLUS  ("+", (x, y) -> x + y),
    MINUS ("-", (x, y) -> x - y),
    TIMES ("*", (x, y) -> x * y),
    DIVIDE("/", (x, y) -> x / y);

    private final String symbol;
    private final DoubleBinaryOperator op;

    Operation(String symbol, DoubleBinaryOperator op) {
        this.symbol = symbol;
        this.op = op;
    }

    @Override public String toString() { return symbol; }

    public double apply(double x, double y) {
        return op.applyAsDouble(x, y);
    }

    // 아이템 34의 메인 메서드
    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values())
            System.out.printf("%f %s %f = %f%n",
                    x, op, y, op.apply(x, y));
    }
}
~~~

* 람다 기반 Operation 열거 타입은 상수별 클래스 몸체는 사용할 이유가 없다고 느낄 수 있지만, 그렇지는 않다.
* 람다는 이름이 없고 문서화도 못한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야 한다.
* 람다는 한줄일 때 가장 좋고 세 줄 안으로 작성하는 것이 좋다. 이상으로는 가독성이 나빠진다.
* 람다가 길거나 읽기 어려우면 간단히 줄여보거나 사용하지 않도록 리팩터링을 해야한다.
* 열거 타입 생성자에 넘겨지는 인수들의 타입도 컴파일타임에 추론되어 생성자 안의 람다는 열거 타입의 인스턴스 멤버에 접근할 수 없다.
* 상수별 동작을 단 몇 줄로 구현하기 어렵고, 필드나 메서드를 사용하는 상황이면 상수별 클래스 몸체를 사용한다.

##### 람다로 대체할 수 없는 곳

* 람다는 함수형 인터페이스에만 쓰인다.
* 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수없어 익명 클래스를 사용한다.
* 추상 메서드가 여러 개인 인터페이스의 인스턴스를 만들 때 익명 클래스를 사용한다.
* 함수 객체가 자신을 참조해야 한다면 익명 클래스를 사용한다. (람다의 this는 바깥 클래스, 익명 클래스는 자신을 참조)
* 람다를 직렬화하는 일은 극히 삼가야 한다.



##### 핵심 정리

> 자바가 8로 판올림되면서 작은 함수 객체를 구현하는 데 적합한 람다가 도입되었다. **익명 클래스는 (함수형 인터페이스가 아닌) 타입의 인스턴스를 만들 때만 사용하라**. 람다는 작은 함수 객체를 아주 쉽게 표현할 수 있어 (이전 자바에서는 실용적이지 않던) 함수형 프로그래밍의 지평을 열었다.



___



## 아이템 43: 람다보다는 메서드 참조를 사용하라

람다가 익명 클래스보다 나은 점 중에서 가장 큰 특징은 간결함이다. 그런데 자바에는 함수 객체를 람다보다도 더 간결하게 만드는 방법이 있으니, **메서드 참조(method reference)** 이다.

##### 임의의 키와 Integer 값의 매핑을 관리하는 프로그램의 일부

키가 맵안에 없다면 키와 숫자 1을 매핑하고 이미 있다면 기존 매핑 값을 증가시킨다.

~~~java
map.merge(key, 1, (count, incr) -> count + incr);
~~~

* 자바 8때 Map에 추가된 merge 메서드를 사용했다. 
* merge 메서드는 키, 값, 함수를 인수로 받으며 주어진 키가 맵안에 없다면 주어진 (키, 값) 쌍을 그대로 저장한다.
* 키가 이미 있다면 함수를 현재 값과 주어진 값에 적용한 다음, 그 결과로 현재 값을 덮어쓴다. 맵에 (키, 함수의 결과) 쌍을 저장한다.
* 매개변수인 count와 incr은 크게 하는 일이 없어 공간을 꽤 차지한다.
* 이 람다는 두 인수의 합을 단순히 반환할 뿐이다.

##### 람다 대신 Integer 클래스의 sum 메서드를 사용한다.

~~~java
map.merge(key, 1, Integer::sum);
~~~

하지만 어떤 람다에서는 매개변수의 이름 자체가 좋은 가이드가 되기도 한다. 이런 람다는 길이는 더 길지만 메서드 참조보다 읽기 쉽고 유지보수도 쉬울 수 있다.

* 람다로 할 수 없는 일이라면 메서드 참조로도 할 수 없다. 그래도 메서드 참조를 사용하는 편이 보통은 더 짧고 간결하므로, 람다를 구현했을 때 너무 길거나 복잡하다면 메서드 참조가 좋은 대안이된다.

* 람다로 작성할 코드를 새로운 메서드에 담은 다음, 람다 대신 그 메서드 참조를 이용하는 식이다.

* 메서드 참조에는 기능을 잘 드러내는 이름을 지어줄 수 있고 친절한 설명을 문서에 남길 수 있다.

* 때론 람다가 메서드 참조보다 간결할 때가 있다. 람다를 사용하는 것이 더 간결하다면 람다가 더 나을 수 있다.

##### 메서드 참조의 5가지 유형

1. 정적 메서드를 가리키는 메서드 참조
2. 수신 객체(receiving object; 참조 대상 인스턴스)를 특정하는 한정적 인스턴스 메서드 참조
   * 한정적 참조는 근본적으로 정적 참조와 비슷하다. 함수 객체가 받는 인수와 참조되는 메서드가 받는 인수가 똑같다.
3. 수신 객체를 특정하지 않는 비한정적 인스턴스 메서드 참조
   * 비한정적 참조에서는 함수 객체를 적용하는 시점에 수신 객체를 알려준다. 이를 위해 수신 객체 전달용 매개변수가 매개변수 목록의 첫 번째로 추가되며, 그 뒤로는 참조되는 메서드 선언에 정의된 매개변수들이 뒤따른다.
   * 주로 스트림 파이프라인에서 매핑과 필터 함수에 쓰인다.
4. 클래스 생성자를 가리키는 메서드 참조
   * 팩터리 객체로 사용된다.
5. 배열 생성자를 가리키는 메서드 참조

| 메서드 참조 유형    | 예                     | 같은 기능을 하는 람다                                   |
| ------------------- | ---------------------- | ------------------------------------------------------- |
| 정적                | Integer::parseInt      | str -> Integer.parseInt(str)                            |
| 한정적 (인스턴스)   | Instant.now()::isAfter | Instant then = Instant.now();<br />t -> then.isAfter(t) |
| 비한정적 (인스턴스) | String::toLowerCase    | str -> str.toLowerCase()                                |
| 클래스 생성자       | TreeMap<K,V>::new      | () -> new TreeMap\<K, V\>()                             |
| 배열 생성자         | int[]::new             | len -> new int[len]                                     |



##### 핵심 정리

> 메서드 참조는 람다의 간단명료한 대안이 될 수 있다. **메서드 참조 쪽이 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하라.**



___



## 아이템 44: 표준 함수형 인터페이스를 사용하라

자바가 람다를 지원하면서 API를 작성하는 모범 사례도 크게 바뀌었다. 
상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴의 매력이 크게 줄었다. 이를 대체하는 현대적인 해법은 같은 효과의 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 것이다.
이 내용을 일반화해서 말하면 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다. 이때 함수형 매개변수 타입을 올바르게 선택해야 한다.

##### LinkedHashMap의 protected 메서드인 removeEldestEntry를 재정의하면 캐시로 사용할 수 있다.

맵에 새로운 키를 추가하는 put 메서드는 이 메서드를 호출하여 true가 반환되면 맵에서 가장 오래된 원소를 제거한다.

이 코드는 가장 최근 원소 100개를 유지한다.

~~~java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > 100;
}
~~~

* 이 코드를 다시 구현한다면 함수 객체를 받는 정적 팩터리나 생성자를 제공했을 것이다.
* size()를 이용해 원소 수를 알아내는데, removeEldestEntry 가 인스턴스 메서드라서 가능하다.
* 하지만 생성자에 넘기는 함수 객체는 이 맵의 인스턴스 메서드가 아니다. 
* 팩터리나 생성자를 호출할 때는 맵의 인스턴스가 존재하지 않기 때문이다. 
* 따라서 맵은 자기 자신도 함수 객체에 건네줘야 한다.

##### 함수형 인터페이스의 선언 - 불필요한 함수형 인터페이스

~~~java
@FunctionalInterface interface EldestEntryRemovalFunction<K,V> {
    boolean remove(Map<K,V> map, Map.Entry<K,V> eldest);
}
~~~

이 인터페이스는 잘 동작하지만 굳이 사용할 필요는 없다. 자바 표준 라이브러리에 이미 같은 모양의 인터페이스가 준비되어 있기 때문이다.

##### 필요한 용도에 맞는게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.

* API가 다루는 개념의 수가 줄어들어 익히기 더 쉬워진다.
* 표준 함수형 인터페이스들은 유용한 디폴트 메서드를 많이 제공하므로 다른 코드와의 상호운용성도 크게 좋아질 것이다.
* java.util.function 패키지에는 총 43개의 인스턴스가 담겨 있다.
* 기본 6개만 기억하면 나머지를 충분히 유추해 낼 수 있다.

##### 표준 함수형 인터페이스

| 인터페이스                                                   | 함수 시그니처       | 예                  |
| :----------------------------------------------------------- | ------------------- | ------------------- |
| UnaryOperator\<T\><br />반환값과 인수의 타입이 같은 함수     | T apply(T t)        | String::toLowerCase |
| BinaryOperator\<T\><br />반환값과 인수의 타입이 같은 함수    | T apply(T t1, T t2) | BigInteger::add     |
| Predicate\<T\><br />인수를 하나 받아 boolean을 반환하는 함수 | boolean test(T t)   | Collection::isEmpty |
| Function\<T,R\><br />인수와 반환 타입이 다른 함수            | R apply(T t)        | Arrays::asList      |
| Supplier\<T\><br />인수를 받지 않고 값을 반환(혹은 제공)하는 함수 | T get()             | Instant::now        |
| Consumer\<T\><br />인수를 하나 받고 반환값은 없는(특히 인수를 소비하는) 함수 | void accept(T t)    | System.out::println |

##### 기본 인터페이스는 기본 타입인 int, long, double 용으로 각 3개씩 변형이 생겨난다.

* Predicate는 IntPredicate, BinaryOperator는 LongBinaryOperator 가 되는 식
* Function의 변형은 매개변수화됐다. LongFunction\<int[]\> 은 long 인수를 받아 int[]을 반환한다.

##### Function 인터페이스에는 기본 타입을 반환하는 변형이 총 9개 더있다.

* 입력과 결과 타입이 모두 기본 타입이면 접두어로 SrcToResult를 사용한다.
  * long을 받아 int를 반환하면 LongToIntFunction (총 6개)
* 입력이 객체 참조이고 결과가 int, long, double인 변형들로, 입력을 매개변수화 하고 접두어 ToResult를 사용한다.
  * ToLongFunction\<int[]\> 은 int[] 인수를 받아 long을 반환한다. (총 3개)

##### 기본 함수형 인터페이스 중 3개에는 인수를 2개씩 받는 변형이 있다.

* BiPredicate\<T,U\>
* BiFunction\<T,U,R\>
  * 기본 타입을 반환하는 세 변형 
    * ToIntBiFunction\<T,U\>
    * ToLongBiFunction\<T,U\>
    * ToDoubleBiFunction\<T,U\>
* BiConsumer\<T,U\>
  * 객체 참조와 기본 타입 하나, 즉 인수를 2개 받는 변형
    * ObjDoubleConsumer\<T\>
    * ObjIntConsumer\<T\>
    * ObjLongConsumer\<T\>

##### BooleanSuppiler 인터페이스는 boolean을 반환하도록 한 Supplier의 변형이다.

* 표준 함수형 인터페이스 중 boolean을 반환하는 인터페이스이지만,

* Predicate와 그 변형 4개도 boolean 값을 반환할 수 있다.



총 43개로 실무에서 자주 쓰이는 함수형 인터페이스 중 상당수를 제공하며 범용적인 이름을 사용했다.
표준 함수형 인터페이스는 대부분 기본 타입만 지원한다. **기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자**

##### 표준 함수형 인터페이스를 직접 작성해야 할때는 언제인가?

* 표준 인터페이스 중 필요한 용도에 맞는게 없다면 직접 작성해야한다.

* 구조적으로 똑같은 표준 함수형 인터페이스가 있더라도 직접 작성해야 할 때가 있다.

  ##### Comparator\<T\> 인터페이스 예제

  구조적으로는 ToIntBiFunction\<T,U\> 와 동일하다. 하지만 Comparator가 독자적인 인터페이스로 살아남아야 하는 이유가 몇 개 있다.

  1. API에서 굉장이 자주 사용되는데, 지금의 이름이 그 용도를 아주 훌륭히 설명해 준다.
  2. 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있다.
  3. 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 듬뿍 담고 있다.

* 전용 함수형 인터페이스를 구현해야 할 지 고민 할 경우

  * 자주 쓰이며, 이름 자체가 용도를 명확히 설명해준다.
  * 반드시 따라야 하는 규약이 있다.
  * 유용한 디폴트 메서드를 제공할 수 있다.

* 전용 함수형 인터페이스를 직접 작성하기로 했다면, 아주 주의해서 설계해야 한다.

##### @FunctionalInterface 애너테이션의 목적

1. 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
2. 해당 인터페이스가 추상 메서드 오직 하나만 가지고 있어야 컴파일되게 해준다.
3. 그 결과 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.
4. **직접 만든 함수형 인터페이스에는 항상 @FunctionalInterface 애너테이션을 사용하라.**



##### 함수형 인터페이스를 사용할 때 주의점

서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안된다. 클라이언트에게 불필요한 모호함만 안겨줄 뿐이며, 이 모호함으로 인해 실제로 문제가 일어나기도 한다.
이런 문제를 가장 피하기 쉬운 방법은 서로 다른 함수형 인터페이스를 같은 위치의 인수로 사용하는 다중정의를 피하는 것이다.



##### 핵심 정리

> 이제 자바도 람다를 지원한다. 여러분도 지금부터는 API를 설계할 때 람다도 염두에 두어야 한다는 뜻이다. 입력값과 반환값에 함수형 인터페이스 타입을 활용하라. 보통은 java.util.function 패키지의 표준 함수형 인터페이스를 사용하는 것이 가장 좋은 선택이다. 단, 흔치 않지만 직접 새로운 함수형 인터페이스를 만들어 쓰는 편이 나을 수도 있음을 잊지 말자.



___



## 아이템 45: 스트림은 주의해서 사용하라

스트림 API는 다량의 데이터 처리 작업(순차적이든 병렬적이든)을 돕고자 자바 8에 추가되었다. 

이 API가 제공하는 추상 개념 중 핵심은 두 가지다.

1. 스트림(Stream)은 데이터 원소의 유한 혹은 무한 시퀀스(sequence)를 뜻한다.
2. 스트림 파이프라인(stream pipeline)은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다. 스트림의 원소들은 어디로부터든 올 수 있다. 대표적으로는 컬렉션, 배열, 파일, 정규표현식 패턴 매처(matcher), 난 수 생성기, 혹은 다른 스트림이 있다. 스트림 안의 데이터 원소들은 객체 참조나 기본 타입 값이다.

##### 스트림 파이프라인

소스 스트림 -> 하나 이상의 중간 연산(intermediate operation) -> 종단 연산(terminal operation)

각 중간 연산은 스트림을 어떠한 방식으로 변환(transform) 한다.

ex) 각 원소에 함수를 적용하거나 특정 조건을 만족 못하는 원소를 걸러낼 수 있다. 

* 중간 연산들은 모두 한 스트림을 다른 스트림으로 변환하는데, 변환된 스트림의 원소 타입은 변환 전 스트림의 원소 타입과 같을 수도 있고 다를 수도 있다. 

* 종단 연산은 마지막 중간 연산이 내놓은 스트림에 최후의 연산을 가한다. 원소를 정렬해 컬렉션에 담거나, 특정 원소를 하나 선택하거나, 모든 원소를 출력하는 식이다.

* 스트림 파이프라인은 지연 평가(lazy evaluation)된다. 평가는 종단 연산이 호출될 때 이뤄지며, 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 이러한 지연 평가가 무한 스트림을 다룰 수 있게 해주는 열쇠다.

##### 스트림 API는 메서드 연쇄를 지원하는 플루언트 API(fluent API)다.

* 파이프 라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다. 파이프라인 여러 개를 연결해 표현식 하나로 만들 수도 있다.
* 기본적으로 스트림 파이프라인은 순차적으로 수행된다.
* 파이프라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림 중 하나에서 parallel 메서드를 호출해주기만 하면 되나, 효과를 볼 수 있는 상황은 많지 않다.
* 스트림 API는 다재다능하여 사실상 어떠한 계산이라도 해낼 수 있다.
* 스트림을 제대로 사용하면 프로그램이 짧고 깔끔해지지만, 잘못 사용하면 읽기 어렵고 유지보수도 힘들어 진다.

##### 아나그램 코드 1 - 철자를 구성하는 알파벳이 같고 순서만 다른 단어

~~~java
public class IterativeAnagrams {
    public static void main(String[] args) throws IOException {
        File dictionary = new File(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        Map<String, Set<String>> groups = new HashMap<>();
        try (Scanner s = new Scanner(dictionary)) {
            while (s.hasNext()) {
                String word = s.next();
                groups.computeIfAbsent(alphabetize(word),
                        (unused) -> new TreeSet<>()).add(word);
            }
        }

        for (Set<String> group : groups.values())
            if (group.size() >= minGroupSize)
                System.out.println(group.size() + ": " + group);
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
~~~

##### - 코드 해석

* 이 프로그램은 사용자가 명시한 사전 파일에서 각 단어를 읽어 맵에 저장한다.
* 맵의 키는 그 단어를 구성하는 철자들을 알파벳순으로 정렬한 값이다.
* "staple"의 키는 "aelpst"가 되고 "petals"의 키도 "aelpst"가 된다. 두 단어는 아나그램이고, 아나그램끼리는 같은 키를 공유한다.
* 맵의 값은 같은 키를 공유한 단어들을 담은 집합이다.
* 사전 하나를 모두 처리하고 나면 각 집합은 사전에 등재된 아나그램들을 모두 담은 상태가 된다.
* 마지막으로 이 프로그램은 맵의 values() 메서드로 아나그램 집합들을 얻어 원소 수가 문턱값보다 많은 집합들을 출력한다.
* computeIfAbsent 메서드를 사용했는데, 이 메서드는 맵 안에 키가 있는지 찾은 다음, 있으면 단순히 그 키에 매핑된 값을 반환한다.



##### 아나그램 코드 2 - 스트림을 과하게 사용한 코드

~~~java
public class StreamAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                    groupingBy(word -> word.chars().sorted()
                            .collect(StringBuilder::new,
                                    (sb, c) -> sb.append((char) c),
                                    StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
~~~

* 코드 1번과 동일한 기능을 하지만 스트림을 과하게 사용한다.
* 코드는 확실히 짧지만 읽기 힘들고 이해하기 어렵다.
* 스트림에 익숙하지 않은 프로그래머는 더 어려울 것이다.
* 스트림을 과용하면 프로그램을 읽거나 유지보수 하기 어려워진다.

##### 아나그램 코드 3 - 스트림을 적절히 활용하자.

~~~java
public class HybridAnagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
~~~

* try 블록에서 사전 파일을 열고, 파일의 모든 라인으로 구성된 스트림을 얻는다.
* 스트림 변수의 이름을 words로 지어 스트림 안의 각 원소가 단어(word)임을 명확히 했다.
* 이 스트림의 파이프라인에는 중간 연산은 없으며 종단 연산에서는 모든 단어를 수집해 맵으로 모은다.
* values()가 반환한 값으로 부터 새로운 Stream\<List\<String\>\> 스트림을 연다.
* 새로운 스트림은 리스트들 중 원소가 minGroupSize 보다 적은 것을 필터링돼 무시된다.
* 마지막으로 종단 연산인 forEach는 살아남은 리스트를 출력한다.

##### 스트림을 처음 사용할 때 서두르지 말자!

* 스트림을 처음 사용하면 모든 반복문을 스트림으로 바꾸고 싶은 유혹이 있다. 
* 스트림으로 바꾸는 게 가능할 지라도 코드 가독성과 유지보수 측면에서는 손해를 볼 수 있다. 
* 스트림과 반복문을 적절히 조합하여 사용하는게 최선이며, 기존 코드는 스트림을 사용하도록 리팩터링하되 새 코드가 더 나아 보일때만 반영하자.

##### 스트림이 안성 맞춤인 것

* 원소들의 시퀀스를 일관되게 변환한다.
* 원소들의 시퀀스를 필터링한다.
* 원소들의 시퀀스를 하나의 연산을 사용해 결합한다.(더하기, 연결하기, 최솟값 구하기 등).
* 원소들의 시퀀스를 컬렉션에 모은다.
* 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

##### 스트림으로 처리하기 어려운 일

* 데이터가 파이프라인의 여러 단계를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근하기 어려운 경우다.
* 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조이기 때문이다.
* 원래 값과 새로운 값의 쌍을 저장하는 객체를 사용해 매핑하는 우회 방법도 있지만, 만족스러운 해법은 아니다.
* 가능한 경우라면, 앞 단계의 값이 필요할 때 매핑을 거꾸로 수행하는 방법이 나을 것이다.

##### 메르센 소수를 출력하는 프로그램

메르센 수는 2ᴾ - 1 형태의 수다. p가 소수이면 해당 메르센 수도 소수일 수 있는데, 이때의 수를 메르센 소수라고 한다.

~~~java
public class MersennePrimes {
    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }

    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }
}
~~~

* 메서드 이름 primes는 스트림의 원소가 소수임을 말해준다.

* 스트림을 반환하는 메서드 이름은 복수 명사로 쓰기를 강력 추천한다.

* 소수들을 사용해 메르센 수를 계산하고, 결과값이 소수인 경우만 남긴 다음, 결과 스트림의 원소 수를 20개로 제한해놓고, 작업이 끝나면 결과를 출력한다.

* 지수 p를 출력하길 원할 때 이 값은 초기 스트림에만 나타나므로 결과를 출력하는 종단 연산에서는 접근할 수 없다.

* 하지만 다행히 첫 번째 중간 연산에서 수행한 매핑을 거꾸로 수행해 메르센 수의 지수를 쉽게 계산할 수 있다.

* 지수는 단순히 숫자를 이진수로 표현한 다음 몇 비트인지 세면 나오므로 종단 연산을 다음과 같이 한다.

  ~~~
  .forEach(mp -> System.out.println(mp.bitLength() + ": " + mp));
  ~~~

##### 스트림과 반복 중 무엇을 써야 할지 알기 어려운 작업

카드 덱을 초기화 하는 작업

카드는 숫자(rank)와 무늬(suit)를 묶은 불변 값 클래스이고, 숫자와 무늬는 모두 열거 타입이다.

##### - 반복 방식으로 구현 - 데카르트 곱 방식

~~~java
public class Card {
    public enum Suit { SPADE, HEART, DIAMOND, CLUB }
    public enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN,
                       EIGHT, NINE, TEN, JACK, QUEEN, KING }

    private final Suit suit;
    private final Rank rank;

    @Override public String toString() {
        return rank + " of " + suit + "S";
    }

    public Card(Suit suit, Rank rank) {
        this.suit = suit;
        this.rank = rank;

    }
    private static final List<Card> NEW_DECK = newDeck();

    // 데카르트 곱 계산을 반복 방식으로 구현
    private static List<Card> newDeck() {
        List<Card> result = new ArrayList<>();
        for (Suit suit : Suit.values())
            for (Rank rank : Rank.values())
                result.add(new Card(suit, rank));
        return result;
    }

    public static void main(String[] args) {
        System.out.println(NEW_DECK);
    }
}
~~~

##### - 스트림 방식으로 구현

~~~java
// 데카르트 곱 계산을 스트림 방식으로 구현
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
            .flatMap(suit -> Stream.of(Rank.values())
            .map(rank -> new Card(suit, rank)))
            .collect(toList());
}
~~~

* 두 방식은 비슷하며 결국은 개인 취향과 프로그래밍 환경의 문제다.
* 확신이 서지 않는 독자는 첫 번째 방식을 사용하는게 더 안전할 것이다.
* 스트림 방식이 나아 보이고 동료들도 스트림 코드를 이해할 수 있고 선호한다면 스트림 방식을 사용하자.



##### 핵심 정리

> 스트림을 사용해야 멋지게 처리할 수 있는 일이 있고, 반복 방식이 더 알맞은 일도 있다. 그리고 수많은 작업이 이 둘을 조합했을 때 가장 멋지게 해결된다. 어느 쪽을 선택하든 확고부동한 규칙은 없지만 참고할 만한 지침 정도는 있다. 어느 쪽이 나은지가 확연히 드러나는 경우가 많겠지만, 아니더라도 방법은 있다. **스트림과 반복 중 어느 쪽이 나은지 확신하기 어렵다면 둘 다 해보고 더 나은 쪽을 택하라.**



___



## 아이템 46: 스트림에서는 부작용 없는 함수를 사용하라

* 스트림은 처음 봐서는 이해하기 어려울 수 있다. 원하는 작업을 스트림 파이프라인으로 표현하는 것조차 어려울지 모른다.
* 성공하여 프로그램이 동작하더라도 장점이 무엇인지 쉽게 와 닿지 않을 수도 있다.
* 스트림은 그저 또 하나의 API가 아닌, 함수형 프로그래밍에 기초한 패러다임이기 때문이다.
* 스트림이 제공하는 표현력, 속도, 병렬성을 얻으려면 API는 말할 것도 없고 이 패러다임까지 함께 받아들여야 한다.

##### 스트림 패러다임의 핵심

* 스트림 패러다임의 핵심은 계산을 일련의 변환으로 재구성하는 부분이다. 
* 이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다.
* 다른 가변 상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다.
* 스트림 연산에 건네는 함수 객체는 모든 부작용(side effect)이 없어야 한다.

##### 스트림 패러다임을 이해하지 못한채 API만 사용한 코드 - 따라하지 말 것

~~~java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
    words.forEach(word -> {
        freq.merge(word.toLowerCase(), 1L, Long::sum);
    });
}
~~~

* 스트림, 람다, 메서드 참조를 사용했고, 결과도 올바르다. 하지만 절대 스트림 코드라 할 수 없다.
* 스트림 코드를 가장한 반복적 코드이다.
* 스트림 API의 이점을 살리지 못하여 같은 기능의 반복적 코드보다 길고, 읽기 어렵고, 유지보수에도 좋지 않다.
* 모든 작업이 종단 연산인 forEach에서 일어나는데, 이때 외부 상태를 수정하는 람다를 실행하면서 문제가 생긴다.
* forEach가 그저 스트림이 수행한 연산 결과를 보여주는 일 이상을 하는 것을 보니 나쁜 코드일 것 이다.

##### 스트림을 제대로 활용해 빈도표를 초기화한다.

~~~java
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
    freq = words.collect(groupingBy(String::toLowerCase, counting()));
}
~~~

* 스트림 API를 제대로 사용했고, 짧고 명확하다.
* forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고 계산하는 데는 쓰지 말자.

##### 코드 수집기(Collector) - 스트림을 사용하려면 꼭 배워야 하는 새로운 개념

* java.util.stream.Collectors 클래스는 메서드를 무려 39개를 가지고 있고, 그중에는 타입 매개변수가 5개나 되는 것도 있다.

* 익숙해지기 전까지 Collector 인터페이스는 잠시 잊고, 그저 축소(reduction) 전략을 캡슐화한 블랙박스 객체라고 생각하자.
* 축소는 스트림의 원소들을 객체 하나에 취합한다는 뜻이다. 수집기가 생선하는 객체는 일반적으로 컬렉션이며, "collector"라는 이름을 쓴다.
* 수집기를 사용하면 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다.
* 수집기는 총 3가지로 toList(), toSet(), toCollection(collectionFactory)이다.

##### 빈도표에서 가장 흔한 단어 10개를 뽑아내는 파이프라인 - 수집기 3가지 중 toList 사용

~~~java
List<String> topTen = freq.keySet().stream()
                .sorted(comparing(freq::get).reversed())
                .limit(10)
                .collect(toList());
~~~

> 마지막 toList는 Collectors의 메서드다. Collectors의 멤버를 정적 임포트하여 쓰면 스트림 파이프라인 가독성이 좋아져, 흔히들 이렇게 사용한다.

* 이 코드에서 어려운 부분은 sorted에 넘긴 비교자, 즉 comparing(freq::get).reversed() 뿐이다.
* comparing 메서드는 키 추출 함수를 받는 비교자 생성 메서드다.
* 한정적 메서드 참조이자, 여기서 키 추출 함수로 쓰인 freq::get은 입력받은 단어(키)를 빈도표에서 찾아(추출) 그 빈도를 반환한다.
* 그런 다음 가장 흔한 단어가 위로 오도록 비교자(comparing)를 역순(reversed)으로 정렬한다(sorted).



##### Collectors의 나머지 36개 메서드

* 이 중 대부분은 스트림을 맵으로 취합하는 기능으로 진짜 컬렉션에 취합하는 것보다 훨씬 복잡하다.

* 스트림의 각 원소는 키 하나와 값 하나에 연관되어 있다. 그리고 다수의 스트림 원소가 같은 키에 연관 될 수 있다.

##### toMap

* 가장 간단한 맵 수집기는 toMap(keyMapper, valueMapper)로 스트림 원소를 키에 매핑하는 함수와 값에 매핑하는 함수를 인수로 받는다.

  ##### 열거 타입 상수의 문자열 표현을 열거 타입 자체에 매핑하는 fromString 구현

  ~~~java
  private static final Map<String, Operation> stringToEnum = 
      Stream.of(values()).collect(toMap(Object::toString, e -> e));
  ~~~

  toMap 형태는 스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다.

  스트림 원소 다수가 같은 키를 사용한다면 파이프라인이 IllegalStateException을 던지며 종료될 것이다.

* 더 복잡한 형태의 toMap이나 groupingBy는 이런 충돌을 다루는 다양한 전략을 제공한다.

  * toMap에 키 매퍼와 값 매퍼는 물론 병합(merge) 함수까지 제공할 수 있따.

* 인수 3개를 받는 toMap은 어떤 키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다.

  ##### 다양한 음악가의 앨범들을 담은 스트림을 가지고, 음악가와 그 음악가의 베스트 앨범을 연관 짓고 싶다.

  ~~~java
  Map<Artist, Album> topHits = albums.collect(
      toMap(Album::artist, a->a, maxBy(comparing(Album::sales))));
  ~~~

* 인수가 3개인 toMap은 충돌이 나면 마지막 값을 취하는 수집기를 만들때도 유용하다.

  ##### 마지막에 쓴 값을 취하는 수집기

  ~~~java
  toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
  ~~~

* 마지막 toMap은 네 번째 인수로 맵 팩터리를 받는다. 이 인수로는 EnumMap이나 TreeMap처럼 원하는 특정 맵 구현체를 직접 지정할 수 있다.

* 세 가지 toMap에는 변종이 있다. 그 중 toConcurrentMap은 병렬 실행 된 후 결과로 ConcurrentHashMap 인스턴스를 생성한다.



##### groupingBy

* 이 메서드는 입력으로 분류 함수(classifier)를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다.
* 분류 함수는 입력받은 원소가 속하는 카테고리를 반환한다.
* 그리고 이 카테고리가 해당 원소의 맵 키로 쓰인다.
* 다중정의된 groupingBy 중 형태가 가장 간단한 것은 분류 함수 하나를 인수로 받아 맵을 반환한다.
* 반환된 맵에 담긴 각각의 값은 해당 카테고리에 속하는 원소들을 모두 담은 리스트다.
* groupingBy가 반환하는 수집기가 리스트 외의 값을 갖는 맵을 생성하게 하려면, 분류 함수와 함께 다운스트림(downstream) 수집기도 명시해야 한다.
* 다운스트림의 수집기의 역할은 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생산하는 일이다.



##### partitioningBy

* 분류 함수 자리에 프레디키트(predicate)를 받고 키가 Boolean인 맵을 반환한다.

##### counting



##### 이름이 summing, averaging, summarizing으로 시작하는 메서드

int, long, double 스트림용으로 하나씩 존재

##### 다중 정의된 reducing 메서드, filtering, mapping, flatMapping, collecting, AndThen

##### minBy, maxBy

인수로 받은 비교자를 이용해 스트림에서 값이 가장 작거나 큰 원소를 찾아 받환한다.

##### joining

문자열 등의 CharSequece 인스턴스의 스트림에만 적용할 수 있다.



##### 핵심 정리

> 스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다. 스트림뿐 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다. 종단 연산 중 forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다. 계산 자체에는 이용하지 말자. 스트림을 올바로 사용하려면 수집기를 잘 알아둬야 한다. 가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining이다.



 책에서는 Collectors들을 나열하며 설명을 하였으나 양이 많고 이해하기 어려운 부분이 있다. stream.Collectors의 API 문서를 추가로 보거나 관련 자료들을 찾아보며 학습을 해야겠다. 
기회가 된다면 먼저 핵심 정리에서 중요하다고 말한 toList, toSet, toMap, groupingBy, joining을 보고 정리하도록 하겠다.



___



## 아이템 47: 반환 타입으로는 스트림보다 컬렉션이 낫다

원소 시퀀스, 일련의 원소를 반환하는 메서드

반환 타입으로 Collection, Set, List, Iterable, 배열을 썼다.

기본적은 컬렉션 인터페이스이고 Collection을 구현할 수 없을 때 Iterable을, 기본 타입이거나 성능에 민감하면 배열을 썼다.

자바 8 에서 스트림 개념 생기면서 이 선택이 아주 복잡하게 되었다.

##### 스트림은 반복을 지원하지 않는다.

* 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다. 
* API를 스트림만 반환하도록 짜놓으면 반환된 스트림을 for-each로 사용할 수 없다.
* Stream이 Iterable을 확장하지 않았기 때문이다.

##### 스트림의 iterator 메서드에 메서드 참조를 건넨다면? - 자바 타입 추론의 한계로 컴파일 되지 않는다.

~~~java
for(ProcessHandle ph : ProcessHandle.allProcesses()::iterator) {
    // 프로세스를 처리한다.
}
~~~

스트림으로 프로세스를 받는 메서드로 iterator를 참조하는 방법이다.

이 코드는 타입의 추론을 하지 못해 컴파일 오류를 낸다.

##### 스트림을 반복하기 위한 '끔찍한' 우회 방법

~~~java
for (ProcessHandle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
    // 프로세스를 처리한다.
}
~~~

동은 하지만 직접 형변환을 해야 하기 때문에 난잡하고 직관성이 떨어진다.

##### Stream\<E\> 를 Iterable\<E\>로 중개해주는 어댑터 - 자바의 타입 추론이 문맥을 잘 파악하여 따로 형변환하지 않아도 된다.

~~~java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return stream::iterator;
}
~~~

~~~java
for (ProcessHandle ph : iterableOf(ProcessHandle.allProcesses())) {
    // 프로세스를 처리한다.
}
~~~

어댑터를 사용하면 어떤 스트림도 for-each 문으로 반복할 수 있다.



##### API가 Iterable만 반환하면  스트림 파이프라인에서 처리하지 못한다 - Iterable\<E\> 를 Stream\<E\> 로 중개해주는 어댑터

~~~java
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
    return StreamSupport.stream(iterable.spliterator(), false);
}
~~~

* 객체 시퀀스를 반환하는 메서드를 작성하는데, 이 메서드가 오직 스트림 파이프라인에서만 쓰일 걸 안다면 마음 놓고 스트림을 반환하게 해주자.
* 반환된 객체들이 반복문에서만 쓰일 걸 안다면 Iterable을 반환하자.
* 공개 API를 작성할 때는 스트림 파이프라인을 사용하는 사람과 반복문에서 쓰려는 사람 모두를 배려해야 한다.



##### Collection 인터페이스는 반복과 스트림을 동시에 지원한다.

원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는 게 일반적으로 최선이다.

단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다.

##### 반환할 시퀀스가 크지만 표현을 간결하게 할 수 있따면 전용 컬렉션을 구현하는 방법을 검토하자.

주어진 집합의 멱집합(한 집합의 모든 부분집합을 원소로 하는 집합)을 반환하는 상황

원소의 개수가 n이면 멱집합의 원소 개수는 2ⁿ 개가 된다. 이를 표준 컬렉션 구현체에 저장하려는 것은 위험하다.

AbstractList를 이용하면 훌륭한 전용 컬렉션을 손쉽게 구현할 수 있다.

~~~java
public class PowerSet {
    // 입력 집합의 멱집합을 전용 컬렉션에 담아 반환한다.
    public static final <E> Collection<Set<E>> of(Set<E> s) {
        List<E> src = new ArrayList<>(s);
        if (src.size() > 30)
            throw new IllegalArgumentException(
                "집합에 원소가 너무 많습니다(최대 30개).: " + s);
        return new AbstractList<Set<E>>() {
            @Override public int size() {
                // 멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱 것과 같다.
                return 1 << src.size();
            }

            @Override public boolean contains(Object o) {
                return o instanceof Set && src.containsAll((Set)o);
            }

            @Override public Set<E> get(int index) {
                Set<E> result = new HashSet<>();
                for (int i = 0; index != 0; i++, index >>= 1)
                    if ((index & 1) == 1)
                        result.add(src.get(i));
                return result;
            }
        };
    }

    public static void main(String[] args) {
        Set s = new HashSet(Arrays.asList(args));
        System.out.println(PowerSet.of(s));
    }
}
~~~

* 멱집합을 구성하는 각 원소의 인덱스를 비트 벡터로 사용하는 것이다.
* 인덱스의 n번째 비트 값은 멱집합의 해당 원소가 원래 집합의 n번째 원소를 포함하는지 여부를 알려준다.
* 0부터 2ⁿ - 1 까지의 이진수와 원소 n개인 집합의 멱집합과 자연스럽게 매핑된다.



##### AbstractCollection을 활용해서 Collection 구현체를 작성할 때는 Iterable용 메서드 외 2개만 더 구현하면 된다.

contains와 size 다.

이 둘을 구현하는 게 불가능할 때는 컬렉션보다는 스트림이나 Iterable을 반환하는 편이 낫다.



##### 입력 리스트의 부분리스트를 모두 반환하는 메서드

~~~java
public class SubLists {
    // 입력 리스트의 모든 부분리스트를 스트림으로 반환한다.
    public static <E> Stream<List<E>> of(List<E> list) {
        return Stream.concat(Stream.of(Collections.emptyList()),
                prefixes(list).flatMap(SubLists::suffixes));
    }

    // 첫 번째 원소를 포함하는 부분리스트 프리픽스 (a,b,c) -> (a), (a,b), (a,b,c)
    private static <E> Stream<List<E>> prefixes(List<E> list) {
        return IntStream.rangeClosed(1, list.size())
                .mapToObj(end -> list.subList(0, end));
    }

    // 마지막 원소를 포함하는 부분리스트 서픽스 (a,b,c) ->  (a,b,c), (b,c), (c)
    private static <E> Stream<List<E>> suffixes(List<E> list) {
        return IntStream.range(0, list.size())
                .mapToObj(start -> list.subList(start, list.size()));
    }
  
    // 입력 리스트의 모든 부분리스트를 스트림으로 반환한다(빈 리스트는 제외)
    public static <E> Stream<List<E>> of(List<E> list) {
        return IntStream.range(0, list.size())
                .mapToObj(start ->
                    IntStream.rangeClosed(start + 1, list.size())
                            .mapToObj(end -> list.subList(start, end)))
                .flatMap(x -> x);
    }

    public static void main(String[] args) {
        List<String> list = Arrays.asList(args);
        SubLists.of(list).forEach(System.out::println);
    }
}
~~~

스트림을 반환하는 두 가지 구현이며 모두 쓸만하다.

하지만, 반복을 사용하는 게 더 자연스러운 상황에서는 스트림보다 반복을 사용하는게 좋을 수 있다.



##### 핵심 정리

> 원소 시퀀스를 반환하는 메서드를 작성할 때는, 이를 스트림으로 처리하기를 원하는 사용자와 반복으로 처리하길 원하는 사용자가 모두 있을 수 있음을 떠올리고, 양쪽을 다 만족시키려고 노력하자. 컬렉션을 반환할 수 있다면 그렇게 하라. 반환 전부터 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들어도 될 정도로 원소 개수가 적다면 ArrayList 같은 표준 컬렉션에 담아 반환하라. 그렇지 않으면 앞서의 멱집합 예처럼 전용 컬렉션을 구현할지 고민하라. 컬렉션을 반환하는 게 불가능하면 스트림과 Iterable 중 더 자연스러운 것을 반환하라. 만약 나중에 Stream 인터페이스가 Iterable을 지원하도록 자바가 수정된다면, 그때는 안심하고 스트림을 반환하면 될 것이다.(스트림 처리와 반복 모두에 사용할 수 있으니)



___



## 아이템 48: 스트림 병렬화는 주의해서 적용하라

동시성 프로그램 측면에서 자바는 항상 앞서갔다. 

* 처음 부터 스레드, 동기화, wait/notify를 지원했다.
* 자바 5부터 동시성 컬렉션인 java.util.concurrent 라이브러리와 실행자(Excecutor) 프레임워크 지원했다.
* 자바 7부터는 고성능 병렬 분해 프레임워크인 포크-조인 패키지를 추가했다.
* 자바 8부터는 parallel 메서드만 한 번 호출하면 파이프라인을 병렬 실행할 수 있는 스트림을 지원했다.

자바로 동시성 프로그램을 작성하기가 점점 쉬워지고 있지만, 올바르고 빠르게 작성하는 일은 여전히 어려운 작업이다.

동시성 프로그래밍을 할 때는 안정성(safety)과 응답 가능(liveness) 상태를 유지하기 위해 애써야 하는데, 병렬 스트림 파이프라인 프로그래밍에서도 다를 바 없다.

##### 스트림을 사용해 처음 20개의 메르센 소수를 생성하는 프로그램 - 아이템 45의 예제

~~~java
public static void main(String[] args){
    primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
        .filter(mersenne -> mersenn.isProbablePrime(50))
        .limit(20)
        .forEach(System.out::println);
}

static Stream<BigInteger> primes() {
    return Stream.iterate(TWO, BigInteger::nextProbablePrime);
}
~~~

* 이 프로그램의 속도를 높이고 싶어 스트림 파이프라인의 parallel()을 호출하겠다고 생각해보자.
* 안타깝게도 이 프로그램은 아무것도 출력하지 못하면서 CPU는 90%나 잡아먹는 상태가 무한히 계속된다.(응답 불가)

##### 원인은? - 스트림 라이브러리가 이 파이프라인을 병렬화하는 방법을 찾아내지 못했기 때문이다.

* 데이터 소스가 Stream.iterate거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.
* 파이프라인 병렬화는 limit를 다룰 때 CPU 코어가 남는다면 원소를 몇 개 더 처리한 후 제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정한다.
* 원소 하나를 계산하는 비용이 대략 그 이전까지의 원소 전부를 계산한 비용을 합친 것만큼 든다.
* 따라서, 스트림 파이프라인을 마구잡이로 병렬화하면 안된다. 성능이 오히려 끔찍하게 나빠질 수 있다.



##### 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위 long 범위일 때 병렬화의 효과가 가장 좋다.

* 이 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어 일을 다수의 스레드에 분배하기 좋다.
* 나누는 작업은 Spliterator가 담당하며, Spliterator 객체는 Stream이나 Iterable의 spliterator 메서드로 얻을 수 있다.
* 이 자료구조들의 또 다른 중요한 공통점은 원소들을 순차적으로 실행할 때의 참조 지역성(locality of reference)이 뛰어나단 것이다.
  이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻이다.
* 참조 지역성이 낮으면 데이터가 캐시 메모리로 전송되는 시간을 기다리게 된다.
* 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열이다.

##### 스트림 파이프라인의 종단 연산의 동작 방식 역시 병렬 수행 효율에 영향을 준다.

* 종단 연산 중 병렬화에 가장 적합한 것은 축소(reduction)다.
* 축소는 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업이다.
* Stream의 reduce 중 하나, min, max, count, sum 같이 완성된 형태로 제공되는 메서드 중 하나를 선택해 수행한다.
* anyMatch, allMatch, nonMatch처럼 조건에 맞으면 바로 반환되는 메서드도 병렬화에 적합하다.
* Stream의 collect 메서드는 병렬화에 적합하지 않다.

##### 스트림을 잘못 병렬화하면 성능이 나빠질 뿐만 아니라 결과 자체가 잘못되거나 예상 못한 동작이 발생할 수 있다.

* 결과가 잘못되거나 오동작하는 것은 안전 실패(safety failure)라 한다.
* 안전 실패는 병렬화한 파이프라인이 사용하는 mappers, filters, 혹은 프로그래머가 제공한 다른 함수 객체가 명세대로 동작하지 않을 때 벌어질 수 있다.
* Stream 명세는 이때 사용되는 함수 객체에 관한 엄중한 규약을 정의해놨다.

##### 스트림 병렬화는 오직 성능 최적화 수단임을 기억해야 한다.

* 병렬화를 사용할 때 실제로 성능이 향상될지를 추정해보는 방법이 있다.
  스트림 안의 원소 수와 원소당 수행되는 코드 줄 수를 곱해보자.
  이 값이 최소 수십만은 되어야 성능향상을 맛볼 수 있다.

* 다른 최적화와 마찬가지로 변경 전후로 반드시 성능 테스트를 하여 병렬화를 사용할 가치가 있는 지 확인해야 한다.
* 잘못된 파이프라인 하나가 시스템의 다른 부분의 성능에 악영향을 줄 수 있음을 유의하자.



##### 우리가 스트림 파이프라인을 병렬화할 일은 그리 많지 않다.

스트림 병렬화가 효과를 보는 경우는 많지 않음을 알게된다.

조건이 잘 갖춰지면 parallel 메서드 호출 하나로 거의 프로세서 코어 수에 비례하는 성능향상을 만끽할 수 있다.

##### 스트림 파이프라인 병렬화가 효과를 제대로 발휘하는 예제

𝜋(n), n보다 작거나 같은 소수의 개수를 계산하는 함수

~~~java
public class ParallelPrimeCounting {
    // 소수 계산 스트림 파이프라인 - 병렬화가 아닐 때 코드
    static long pi(long n) {
        return LongStream.rangeClosed(2, n)
                .mapToObj(BigInteger::valueOf)
                .filter(i -> i.isProbablePrime(50))
                .count();
    }

    public static void main(String[] args) {
        System.out.println(pi(10_000_000));
    }
}
~~~

~~~java
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
            .parallel()
            .mapToObj(BigInteger::valueOf)
            .filter(i -> i.isProbablePrime(50))
            .count();
}
~~~

위의 예제에서는 단순히 parallel() 코드만으로 3배 이상 빨라지는 효과를 보았다.



##### 핵심 정리

> 계산도 올바로 수행하고 성능도 빨라질 거라는 확신 없이는 스트림 파이프라인 병렬화는 시도조차 하지 말라. 스트림을 잘못 병렬화하면 프로그램을 오동작하게 하거나 성능을 급격히 떨어뜨린다. 병렬화하는 편이 낫다고 믿더라도, 수정 후의 코드가 여전히 정확한지 확인하고 운영 환경과 유사한 조건에서 수행해보며 성능 지표를 유심히 관찰하라. 그래서 계산도 정확하고 성능도 좋아졌음이 확실해졌을 때, 오직 그럴 때만 병렬화 버전 코드를 운영 코드에 반영하라.



___

7장 람다와 스트림 끝...
