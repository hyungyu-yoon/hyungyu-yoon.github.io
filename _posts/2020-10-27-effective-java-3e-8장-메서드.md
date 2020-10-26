---
title: effective java 3/E - 7장. 람다와 스트림
tags: 
 - Java
key: 13
---


# 8장 메서드

##### 8장에서 배울 내용

* 메서드를 설계할 때 주의할 점
* 매개변수와 반환값을 어떻게 처리해야 하는가
* 메서드 시그니처는 어떻게 설계해야 하는가
* 문서화는 어떻게 해야 하는가
* 메서드 뿐 아니라 생성자에도 적용이된다.
* 사용성, 견고성, 유연성에 집중한다.



___



## 아이템 49: 매개변수가 유효한지 검사하라

* 메서드와 생성자 대부분은 입력 매개변수의 값이 특정 조건을 만족하기를 바란다.
  ex) 인덱스 값은 음수가 안 되며, 객체 참조는 null이 아니어야 한다.
* 이런 제약은 반드시 문서화해야 하며 메서드 몸체가 시작되기 전에 검사해야 한다.
  "오류는 가능한 한 빨리 (발생한 곳에서) 잡아야 한다." 는 일반 원칙의 한 사례이기도 하다.

메서드 몸체가 실행되기 전에 매개변수를 확인한다면 잘못된 값이 넘어왔을 때 즉각적이고 깔끔한 방식으로 예외를 던질 수 있다.

##### 매개변수 검사를 제대로 하지 못하면 몇 가지 문제가 생길 수 있다.

1. 메서드가 수행되는 중간에 모호한 예외를 던지며 실패할 수 있다.
2. 메서드가 잘 수행되지만 잘못된 결과를 반환할 때다.
3. 메서드는 문제없이 수행됐지만, 어떤 객체를 이상한 상태로 만들어놓아서 미래의 알 수 없는 시점에 이 메서드와 관련없는 오류를 낼 때다.

##### <span style="color:red">⊗ 매개변수 검사에 실패하면 실패 원자성을 어기는 결과를 낳을 수 있다.</span>

##### public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화 해야한다.(@throws 자바독 태그 사용)

* 보통은 IllegalArgumentException, IndexOutOfBoundsException, NullPointerException 이다.
* 문서화 한다면 제약을 어겼을 떄 발생하는 예외도 함께 기술해야 한다.
* 이런 간단한 방법으로 API 사용자가 제약을 지킬 가능성을 크게 높일 수 있다.

##### - 예제

~~~java
/*
 * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
 *
 * @param m 계수 (양수여야 한다.)
 * @return 현재 값 mod m
 * @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
 */
public BigInteger mod(BigInteger m){
        if(m.signum() <= 0){
                throws new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
        }
        ...
}
~~~

* 이 메서드는 m이 null이면 m.signum() 호출 때 NullPointerException을 던진다.
* 그러나 메서드 설명에 NullPointerException을 던진다는 말은 없다.
* 그 이유는 메서드 개별이아닌 클래스 수준에서 기술했기 때문이다.
* 클래스 수준 주석은 그 클래스의 public 메서드에 적용되므로 각각 작성하는 것보다 훨씬 깔끔하다.

##### 자바 7에서 추가된 java.util.Objects.requireNonNull 메서드는 null 검사를 수행하여 유연하고 사용하기 편하다.

~~~java
this.strategy = Objects.requireNonNull(strategy, "전략");
~~~

* 원하는 예외 메세지를 지정할 수 있다.
* 입력을 그대로 반환하므로 값을 사용하는 동시에 null 검사를 수행할 수 있다.
* 반환 값은 무시하고 순수한 null 검사 목적으로 사용해도 된다.

##### 공개되지 않은 메서드라면 개발자가 메서드가 호출되는 상황을 통제 할 수 있다.

* 오직 유효한 값만이 메서드에 넘겨지리라는 것을 보증할 수 있고, 그렇게 해야한다.

* public이 아닌 메서드라면 단언(assert)을 사용해 매개변수 유효성을 검증할 수 있다.

  ~~~java
  private static void sort(long a[], int offset, int length) {
      assert a != null;
      assert offset >= 0 && offset <= a.length;
      assert length >= 0 && length <= a.length - offset;
      ... // 계산 수행
  }
  ~~~

  단언문들은 자신이 단언한 조건이 무조건 참이라고 선언한다는 것이다.
  단언문은 일반적인 유효성 검사와는 다르다.

  1. 실패하면 AssertionError를 던진다.
  2. 런타임에 아무런 효과도, 아무런 성능 저하도 없다.(단, java를 실행할 때 -ea, --enableassertions 를 설정하면 런타임에 영향이 있다.)

##### 메서드가 직접 사용하지는 않으나 나중에 쓰기 위해 저장하는 매개변수는 특히 더 신경써서 검사해야 한다.

* int 배열의 List 뷰를 반환하는 메서드를 Objects.requireNonNull 을 이용해 검사를 수행할 때 null을 건넨다면 NullPointerException이 발생한다.
* 만약 이 검사를 생략했다면 List를 반환받고 이 List를 사용하려할 때 NullPointerException이 발생할 것이다.
* 이렇게 되면 추적하기가 까다로워진다.
* 생성자 매개변수의 유효성 검사는 클래스 불변식을 어기는 객체가 만들어지지 않게 하는 데 꼭 필요하다.

##### 메서드 몸체 실행 전 매개변수 유효성을 검사한다는 규칙의 예외 사항

* 유효성 검사 비용이 지나치게 높을 때

* 계산 과정에서 암묵적으로 검사가 수행될 때

* ##### 암묵적 유효성 검사에 너무 의존하다간 실패 원자성을 해칠 수 있으니 주의하자.

##### <span style="color:blue">"매개변수에 제약을 두는게 좋다"는 것이 아니며 메서드가 받은 값으로 제대로 된 일을 한다면 제약은 적을 수록 좋다.</span>



##### 핵심 정리

> 메서드나 생성자를 작성할 때면 매개변수들에 어떤 제약이 있을지 생각해야 한다. 그 제약들을 문서화하고 메서드 코드 시작 부분에서 명시적으로 검사해야 한다. 이런 습관을 반드시 기르도록 하자. 그 노력은 유효성 검사가 실제로 오류를 처음 걸러낼 때 충분히 보상받을 것이다.



___



## 아이템 50: 적시에 방어적 복사본을 만들라

자바는 안전한 언어이지만 다른 클래스로부터의 침범을 아무런 노력 없이 다 막을 수 있는 것은 아니다.

##### 클라이언트가 불변식을 깨뜨리려 혈안이 되어 있다고 가정하고 방어적으로 프로그래밍해야 한다.

* 실제로도 악의적인 의도를 가진 사람들의 시스템의 보안을 뚫으려는 시도가 늘고 있다.
* 평범한 프로그래머도 순전히 실수로 클래스를 오동작하게 만들 수 있다.
* 적절치 않은 클라이언트로부터 클래스를 보호하는 데 충분한 시간을 투자하는 게 좋다.

##### 어떤 객체든 그 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능하다.

하지만 주의를 기울이지 안ㅇ흐면 자기도 모르게 내부를 수정하도록 허락하는 경우가 생긴다.

##### 예제 - 기간을 표현하는 클래스, 불변식을 지키지 못했다.

다음 클래스는 한번 값이 정해지면 변하지 않도록 할 생각이었다.

~~~java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각. 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + "가 " + end + "보다 늦다.");
        this.start = start;
        this.end   = end;
    }

    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }

    // 나머지 코드 생략
}
~~~

* 이 클래스는 불변처럼 보이지만 Date가 가변이라는 사실을 이용하면 어렵지 않게 불변식을 깨뜨릴 수 있다.

##### Period 인스턴스의 내부를 공격해보자.

~~~java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부를 수정했다.
~~~

* 자바 8 이후로 Date 대신 Instant를 사용하면 된다.

* ##### Date는 낡은 API이니 새로운 코드를 작성할 때는 더 이상 사용하면 안된다.

##### 내부를 보호하려면 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy)해야 한다.

~~~java
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end   = new Date(end.getTime());

    if (this.start.compareTo(this.end) > 0)
        throw new IllegalArgumentException(this.start + "가 " + this.end + "보다 늦다.");
}
~~~

* 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사한 점에 주목하자.

* 순서가 부자연스러워 보이지만 반드시 이렇게 작성해야 한다.

* 멀티스레딩 환경이라면 원본 객체의 유효성 검사를 한 후 복사본을 만드는 찰나에 다른 스레드가 원본 객체를 수정할 위험이 있다.

* ##### 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안된다.

* 방어적 복사로 앞선 공격은 막을 수 있지만 아직도 변경이 가능하다.

##### Period 인스턴스를 향한 두 번째 공격

~~~java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78);
~~~

* 가변 필드의 방어적 복사본을 반환하면 막을 수 있다.

##### 수정한 접근자 - 필드의 방어적 복사본을 반환한다.

~~~java
public Date start() {
    return new Date(start.getTime());
}

public Date end() {
    return new Date(end.getTime());
}
~~~

* Period는 자신 말고는 가변 필드에 접근할 방법이 없으니 확실하게 불변식을 지킬 수 있다.

##### 매개변수를 방어적으로 복사하는 목적이 불변 객체를 만들기 위해서만은 아니다.

* 클라이언트가 제공한 객체의 참조를 내부의 자료구조에 보관해야 할 때면 항시 그 객체가 잠재적으로 변경될 수 있는지를 생각해야 한다.
* 변경될 수 있는 객체라면 그 객체가 클래스에 넘겨진 뒤 임의로 변경 되어도 그 클래스가 문제없이 동작할지 따져보라.
* 확신할 수 없다면 복사본을 만들어 저장해야 한다.

##### 방어적 복사는 성능 저하가 따르고, 또 항상 쓸 수 있는 것도 아니다.

* 같은 패키지 내에서 호출자가 컴포넌트 내부를 수정하지 않으리라 확신하면 방어적으로 복사를 생략할 수 있다.
* 호출자에서 해당 매개변수나 반환값을 수정하지 말아야 함을 명확히 문서화 하는게 좋다.
* 다른 패키지에서 사용하더라도 통제권을 넘겨받은 클라이언트가 수정하는 일이 없다면 가능하다.

##### 

##### 핵심 정리

> 클래스가 클라이언트로부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사해야 한다. 복사 비용이 너무 크거나 클라이언트가 그 요소를 잘못 수정할 일이 없음을 신뢰한다면 방어적 복사를 수행하는 대신 해당 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시하도록 하자.



___



## 아이템 51: 메서드 시그니처를 신중히 설계하라

이번 아이템은 API 설계 요령들을 모아놓았다. 요령들을 잘 활용하면 배우기 쉽고, 쓰기 쉬우며, 오류 가능성이 적은 API를 만들 수 있을 것이다.

##### 메서드 이름을 신중히 짓자.

* 표준 명명 규칙을 따라야 한다. (아이템 68)
* 이해할 수 있고, 같은 패키지에 속한 다른 이름들과 일관되게 짓는 게 최우선 목표이다.
* 개발자 커뮤니티에서 널리 받아들여지는 이름을 사용한다. 긴 이름은 피한다
* 자바 라이브러리의 API 가이드를 참조하라.

##### 편의 메서드를 너무 많이 만들지 말자.

* 메서드가 너무 많은 클래스를 익히고, 사용하고, 문서화하고, 테스트하고, 유지보수 하기 어렵다. (인터페이스도)
* 메서드가 너무 많으면 이를 구현하는 사람과 사용하는 사람 모두를 고통스럽게 한다.
* 클래스나 인터페이스는 자신의 각 기능을 완벽히 수행하는 메서드로 제공해야 한다.
* 아주 자주 쓰일 경우에만 별도의 약칭 메서드로 만들고 확신이 서지 않으면 만들지 말자.

##### 매개변수 목록은 짧게 유지하자.

* 4개 이하가 좋다. 4개가 넘어가면 전부 기억하기 쉽지 않다.
* 같은 타입의 매개변수 여러 개가 연달아 나오는 경우가 특히 해롭다.
* 사용자가 순서를 기억하기 어렵고, 실수로 순서를 바꿔도 컴파일되고 실행된다. 단지 의도와 다르게 동작한다.

##### <span style="color: tomato">매개변수 목록을 짧게 줄여주는 세 가지 기술</span>

1. 여러 메서드로 쪼갠다.

   * 쪼개진 메서드 각각은 원래의 매개변수 목록의 부분집합을 받는다.

   * 메서드가 많아질 수도 있지만, 직교성을 높여 오히려 메서드 수를 줄여주는 효과도 있다.

     ##### ※직교성이 높다: 공통점이 없는 기능들이 잘 분리되어 있다. or 기능을 원자적으로 쪼개 제공한다.

     메서드가 쪼개다보면 자연스럽게 중복이 줄고 결합성이 낮아져 코드를 수정하기 쉬워진다.

     단, 무조건 작게 나누는 것이 능사는 아니며 다루는 개념의 추상화 수준에 맞게 조절해야 한다.

2. 매개변수 여러 개를 묶어주는 도우미 클래스를 만드는 것이다.

   * 일반적으로 이런 도우미 클래스는 정적 멤버 클래스로 둔다.
   * 매개변수 몇 개를 독립된 하나의 개념으로 볼 때 추천하는 기법이다.

3. 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용한다.

   * 매개변수가 많을 때, 특히 그중 일부는 생략해도 괜찮을 때 도움이 된다.
   * 먼저 모든 매개변수를 하나로 추상화한 객체를 정의하고, 클라이언트에서 이 객체의 세터 메서드를 호출해 필요한 값을 설정하게 하는 것이다.
   * 이때 각 세터 메서드는 매개변수 하나 혹은 서로 연관된 몇 개만 설정하게 만든다.
   * 클라이언트는 먼저 필요한 매개변수를 다 설정한 다음, excute 메서드를 호출해 앞서 설정한 매개변수들의 유효성을 검사한다.
   * 마지막으로, 설정이 완료된 객체를 넘겨 원하는 계산을 수행한다.

##### 매개변수의 타입으로는 클래스보다는 인터페이스가 더 낫다.

* 매개변수로 적합한 인터페이스가 있다면 그 인터페이스를 직접 사용하자.
* HashMap 대신 Map 을 사용하면 TreeMap, ConcurrentHashMap 등 어떤 Map의 구현체도 인수로 건넬 수 있다.
* 인터페이스 대신 클래스를 사용하면 클라이언트에게 특정 구현체만 사용하도록 제한하는 꼴이며, 혹시라도 입력 데이터가 다른 형태로 존재한다면 명시한 특정 구현체의 객체로 옮겨 담느라 비싼 복사 비용을 치러야 한다.

##### boolean보다는 원소 2개짜리 열거 타입이 낫다

* 열거 타입을 사용하면 코드를 읽고 쓰기가 더 쉬워진다.
* 나중에 선택지를 추가하기도 쉽다.



___



## 아이템 52: 다중정의는 신중히 사용하라

##### 컬렉션을 집합, 리스트 그 외로 구분하고자 만든 프로그램 - 오류!

~~~java
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
~~~

* "집합", "리스트", "그 외" 를 차례로 출력할 것 같지만, 실제로 수행하면 "그 외" 만 3번 출력한다.

* ##### 다중정의(overloading, 오버로딩) 된 세 classify 중 어느 메서드를 호출할지가 컴파일타임에 정해지기 때문이다.

* 컴파일 타임에 for의 c는 항상 Collection\<?\> 타입이다. 

* 런타임에는 타입이 매번 달라지지만, 호출할 메서드를 선택하는 데는 영향을 주지 못한다.

* 따라서 컴파일타임의 매개변수 타입을 기준으로 항상 세 번째 메서드만 선택하는 것이다.

##### 재정의한 메서드는 동적으로 선택되고, 다중정의한 메서드는 정적으로 선택되기 때문이다.

* 메서드를 재정의했다면 해당 객체의 런타임 타입이 어떤 메서드를 호출할지의 기준이 된다.
* 재정의란 상위 클래스가 정의한 것과 똑같은 시그니처의 메서드를 하위 클래스에서 다시 정의한 것이다.
* 메서드를 재정의하고 하위 클래스의 인스턴스에서 메서드를 호출하면 재정의한 메서드가 실행된다.

##### 재정의된 메서드 호출 메커니즘 - 이 프로그램은 무엇을 출력할까?

~~~java
class Wine {
    String name() { return "포도주"; }
}

class SparklingWine extends Wine {
    @Override String name() { return "발포성 포도주"; }
}

class Champagne extends SparklingWine {
    @Override String name() { return "샴페인"; }
}

public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
                new Wine(), new SparklingWine(), new Champagne());

        for (Wine wine : wineList)
            System.out.println(wine.name());
    }
}
~~~

* Wine 메서드의 name은 SparklingWine과 Chanpagne에서 재정의 된다.
* 이 프로그램은 "포도주", "발포성 포도주", "샴페인"을 차례로 출력한다.
* 가장 하위에서 재정의한 메서드가 실행되는 것이다.

##### 다중정의된 메서드 사이에서는 객체의 런타임 타입은 전혀 중요하지 않다.

* 선택은 컴파일타임에 오직 매개변수의 컴파일 타입에 의해 의뤄진다.

##### 앞서의 CollectionClassifier의 다중정의 해결법 - 정적 메서드

~~~java
public static String classify(Collection<?> c) {
    return c instanceof Set  ? "집합" :          
           c instanceof List ? "리스트" : "그 외";
}
~~~

* 하나의 classify로 합치고 instanceof로 명시적 검사를 한다.

* 다중정의에서 헷갈릴 수 있는 코드는 작성하지 않는게 좋다.

* API 사용자가 매개변수를 넘기면서 어떤 다중정의 메서드가 호출될지 모른다면 프로그램이 오동작하기 쉽다.

* 런타임에 이상하게 행동할 것이며 API 사용자들은 문제를 진단하느라 시간을 소비할 것이다.

* ##### 그러므로 다중정의가 혼동을 일으키는 상황을 피해야 한다.

##### 안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자.

* 가변인수를 사용하는 메서드라면 다중정의를 아예 하지 말아야 한다. (아이템 53)

* 이 규칙만 잘 따르면 어떤 다중정의 메서드가 호출될지 헷갈릴 일은 없다.

* ##### 다중정의하는 대신 메서드 이름을 다르게 지어주는 길도 항상 열려 있다.

  * ObjectOutputStream 클래스의 writeBoolean, writeInt, writeLong ... 같은 식으로 정의

##### 생성자는 이름을 다르게 지을 수 없다.

* 정적 팩터리라는 대안을 활용할 수 있는 경우가 많다.

##### 그래도 같은 수의 매개변수를 받아갈 경우가 있을 수 있다.

* 매개변수 중 하나 이상이 "근본적으로 다르다"면 헷갈일 일이 없다.
* 근본적으로 다르다는 것은 두 타입의 (null이 아닌) 값을 서로 어느쪽으로든 형변환 할 수가 없다는 뜻이다.
* 이 조건을 충족하면 어느 다중정의 메서드를 호출할지 런타임 타입만으로 결정된다.
* 따라서 컴파일타임 타입에는 영향을 받지 않고, 혼란을 주는 주된 원인이 사라진다.
* ex) ArrayList 에는 int를 받는 생성자가 있고, Collection을 받는 생성자가 있다. 
  둘을 헷갈일 일은 없다.

##### 자바 4까지 모든 기본 타입이 모든 참조 타입과 근본적으로 달랐지만, 자바 5에서 오토박싱이 도입되면서 상황이 바뀌었다.

~~~java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list);
    }
}
~~~

* set의 remove(i)의 시그니쳐는 remove(Object)이다. 따라서 0, 1, 2를 제거하여 [-3, -2, -1] 로 출력된다.

* list의 remove(i)는 다중정의된 remove(int index) 이다. 지정된 인덱스의 원소를 제거하게 된다.
  따라서 [-2, 0, 2]를 출력한다.

* 이를 해결하기 위해 list.remove(i)의 인수를 Integer로 형변환하여 올바른 다중정의 메서드를 선택하도록 한다.

  ~~~java
  list.remove((Integer) i); // 또는 remove(Integer.valueOf(i))
  ~~~

* 제네릭과 오토박싱이 등장하면서 두 메서드의 매개변수 타입이 근본적으로 다르지 않다. 

* 이를 주의해서 사용해야 한다.

##### 자바 8에서 도입한 람다와 메서드 참조 역시 다중정의 혼란을 키웠다.

~~~java
// 1번 Thread의 생성자 호출
new Thread(System.out::println).start();

// 2번 ExecutorService의 submit 메서드 호출
ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(System.out::println);
~~~

* 1,2 번이 모습은 비슷하지만 2번은 컴파일 오류가 난다.

* submit 다중정의 메서드 중에는 Callable\<T\> 를 받는 메서드도 있기 때문이다.

* System.out::println 은 부정확한 메서드 참조이다.

* 암시적 타입 람다식이나 부정확한 메서드 참조 같은 인수 표현식은 목표 타입이 선택되기 전에는 그 의미가 정해지지 않기 때문에 적용성 테스트 때 무시된다.

* ##### 메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수를 받아서는 안 된다.

* 이는 서로 다른 함수형 인터페이스라도 서로 근본적으로 다르지 않다는 뜻이다.



##### 핵심 정리

> 프로그래밍 언어가 다중정의를 허용한다고 해서 다중정의를 꼭 활용하란 뜻은 아니다. 일반적으로 매개변수 수가 같을 때 다중정의를 피하는 게 좋다. 상황에 따라, 특히 생성자라면 이 조언을 따르기 불가능할 수 있다. 그럴 때는 헷갈릴 만한 매개변수는 형변환하여 정확한 다중정의 메서드가 선택되도록 해야한다. 이것이 불가능하면, 예컨대 기존 클래스를 수정해 새로운 인터페이스를 구현해야 할 때는 같은 객체를 입력받는 다중정의 메서드들이 모두 동일하게 동작하도록 만들어야 한다. 그렇지 못하면 프로그래머들은 다중정의된 메서드나 생성자를 효과적으로 사용하지 못할 것이고, 의도대로 동작하지 않는 이유를 이해하지도 못할 것이다.



___



## 아이템 53: 가변인수는 신중히 사용하라

가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다. 가변 인수 메서드를 호출하면 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다.

##### 간단한 가변인수를 활용 예

입력받은 int 인수들의 합을 계산하는 코드

~~~java
static int sum(int... args){
    int sum = 0;
    for (int arg : args){
        sum += arg;
    }
    return sum;
}
~~~

##### 인수 1개 이상이어야 하는 가변인수 메서드 - 잘못 구현한 예!

~~~java
static int min(int... args){
    if(args.length == 0)
        throws new IllegalArgumentException("인수가 1개 이상 필요합니다.");
        
    int min = args[0];
    for (int i = 1; i < args.length; i++){
        if (args[i] < min)
            min = args[i];
    }
    return min;
}
~~~

* 인수를 0개만 넣어 호출하면 런타임에 실패하고 코드도 지저분하다.
* args 유효성 검사를 명시적으로 해야하고, min의 초깃값을 Integer.MAX_VALUE로 설정하지 않고는 for-each 문을 사용할 수 없다.

##### 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법

~~~java
static int min(int firstArg, int... remainingArgs){
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
~~~

* 가변인수는 인수 개수가 정해지지 않았을 때 아주 유용하다.

##### 성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다.

* 가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화한다.

* 가변인수의 유연성이 필요할 때 선택할 수 있는 멋진 패턴

  ##### 메서드 호출의 95%가 인수를 3개 이하로 사용한다.

  ~~~java
  public void foo() {}
  public void foo(int a1) {}
  public void foo(int a1, int a2) {}
  public void foo(int a1, int a2, int a3) {}
  public void foo(int a1, int a2, int a3, int... rest) {}
  ~~~

  메서드를 다중정의 하여 95%를 담당하고 인수가 4개인 메서드로 5%로를 담당한다.
  꼭 필요한 특수 상황에서는 성능적으로 최적화 할 수 있다.

* EnumSet의 정적 팩터리도 이 기법을 사용해 열거 타입 집합 생성 비용을 최소화 한다.



##### 핵심 정리

> 인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다. 메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자.



___



## 아이템 54: null이 아닌, 빈 컬렉션이나 배열을 반환하라

##### 컬렉션이 비어있으면 null을 반환하는 코드 - 따라 하지 말 것!

~~~java
priavate final List<Cheese> cheesesInStock = ...;

/**
 * @return 매장 안의 모든 치즈 목록을 반환한다.
 *     단, 재고가 하나도 없다면 null을 반환한다.
 */
public List<Cheese> get Cheeses() {
    return cheesesInStock.isEmpty() ? null
        : new ArrayList<>(cheesesInStock); 
}
~~~

* null을 반환한다면, 클라이언트는 null을 처리하는 코드를 추가적으로 작성해야 한다.

~~~java
List<Cheese> cheese = shop.getCheeses();
if (cheese != null && cheeses.contains(Cheese.STILTON))
    System.out.println("좋았어, 바로 그거야.");
~~~

* 컬렉션이나 배열 같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용하면 위 코드 처럼 방어코드를 넣어주어야 한다.
* 방어 코드를 빼먹는다면 오류가 발생할 수 있다.
* null을 반환하려면 반환하는 쪽에서도 이 상황을 특별히 취급해줘야 해서 코드가 더 복잡해 진다.

##### 빈 컨테이너를 할당하는 데도 비용이 드니 null을 반환하는 것이 낫다는 주장도 있다.

하지만 이는 두 가지 면에서 틀린 주장이다.

1. 성능 분석 결과 이 할당이 성능 저하의 주범이라고 확인되지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 못된다.

2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

   ##### 빈 컬렉션을 반환하는 올바른 예

   ~~~java
   public List<Cheese> get Cheeses() {
       return new ArrayList<>(cheesesInStock);
   }
   ~~~

   * 사용 패턴에 따라 빈 컬렉션 할당이 성능을 눈에 띄게 떨어 뜨릴 수도 있다.
   * 해결 법은 매번 똑같은 빈 '불변' 컬렉션을 반환하는 것이다.
   * Collections.emptyList 메서드가 예이다.
   * 단, 이 역시 최적화에 해당하니 꼭 필요할 때만 사용하자.
   * 최적화가 필요하다고 판단되면 수정 전과 후의 성능을 측정하여 실제 성능이 개선 되는지 꼭 확인하자.

   ##### 최적화 - 빈 컬렉션을 매번 새로 할당하지 않도록 했다.

   ~~~java
   public List<Cheese> getCheese() {
       return cheesesInStock.isEmpty() ? Collections.emptyList()
           : new ArrayList<>(cheesesInStock);
   }
   ~~~

##### 배열을 쓸 때도 마찬가지이다.

* 절대 null을 반환하지 말고 길이가 0인 배열을 반환하라.
* 보통은 단순히 정확한 길이의 배열을 반환하기만 하면 된다.

~~~java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
~~~

* 이 방식이 성능을 떨어뜨릴 것 같다면 길이 0짜리 배열을 미리 선언하고 그 배열을 반환한다.

~~~java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
    return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
~~~

* 단순히 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하는 것은 추천하지 않는다.



##### 핵심 정리

> null이 아닌, 빈 배열이나 컬렉션을 반환하라. null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다. 그렇다고 성능이 좋은 것도 아니다.



___



## 아이템 55: 옵셔널 반환은 신중히 하라

##### 자바 8 전에 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할 수 있는 선택 두 가지

* 예외를 던진다.
  * 예외는 진짜 예외적인 상황에서만 사용해야 한다.
  * 예외를 생성할 때 스택 추적 전체를 캡처하므로 비용이 만만치 않다.
* null을 반환한다.
  * null을 반환할 수 있는 메서드를 호출할 때는 별도의 null처리 코드를 추가해야 한다.
  * null 처리를 무시하고 반환된 null 값을 어딘가에 저장해두면 언젠가 NullPointerException이 발생할 수 있다.

##### 자바 8에서 생긴 Optional\<T\> 

* null이 아닌 T 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다.
* 아무것도 담지 않은 옵셔널은 **비었다** 고 말한다.
* 반대로 어떤 값을 담은 옵셔널은 **비지 않았다** 라고 말한다.
* 옵셔널은 원소 최대 1개를 가질 수 있는 **불변** 컬렉션이다.
* 보통은 T를 반환해야 하지만 특정 조건에서는 아무것도 반환하지 않아야 할 때 Optional\<T\> 를 반환하도록 선언한다.
* 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 작다.

##### 컬렉션에서 최댓값을 구한다 - 컬렉션이 비어있을 때 예외를 던진다.

~~~java
public class Max {
    public static <E extends Comparable<E>> E max(Collection<E> c) {
        if (c.isEmpty())
            throw new IllegalArgumentException("빈 컬렉션");

        E result = null;
        for (E e : c)
            if (result == null || e.compareTo(result) > 0)
                result = Objects.requireNonNull(e);

        return result;
    }
}
~~~

* 이 메서드에 빈 컬렉션을 건네면 IllegalArgumentException을 던진다. 
* Optional\<E\> 를 반환하는 것이 더 낫다.

##### 컬렉션에서 최댓값을 구한다. - Optional\<E\> 로 반환한다.

~~~java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return Optional.of(result);
}
~~~

* 적절한 정적 팩터리를 사용해 옵셔널을 생성해주면 된다.

* 빈 옵셔널은 Optional.empty() 로 만든다.

* 값이 든 옵셔널은 Optional.of(value) 로 만든다.

* null 값을 허용하는 옵셔널은 Optional.ofNullable(value) 를 사용한다.

* ##### 옵셔널을 반환하는 메서드에서는 절대 null을 반환하지 말자.

##### 스트림의 종단 연산 중 상당수가 옵셔널을 반환한다. - max 예제의 스트림 버전

~~~java
public static <E extends Comparable<E>> Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder());
}
~~~



##### null을 반환하거나 예외를 던지는 대신 옵셔널 반환을 선택해야 하는 기준은?

* ##### 옵셔널은 검사 예외와 취지가 비슷하다. (아이템 71)

* 반환 값이 없을 수도 있음을 API 사용자에게 명확히 알려준다.

* 비검사 예외를 던지거나 null을 반환하면 사용자는 그 사실을 인지하지 못한다.

##### 옵셔널로 반환 값을 받지 못했을 때 클라이언트의 행동

##### <span style="color:tomato">옵셔널 활용 1 - 기본값을 정해둘 수 있다.</span>

~~~java
String lastWordInLexicon = max(words).opElse("단어 없음");
~~~

##### <span style="color:tomato">옵셔널 활용 2 - 원하는 예외를 던질 수 있다.</span>

~~~java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
~~~

##### <span style="color:tomato">옵셔널 활용 3 - 항상 값이 채워져 있다고 가정한다. </span>

~~~java
Element lastNobleGas = max(Elements.NOBLE_GASES).get();
~~~

##### 기본 값을 설정할 때 설정하는 비용이 아주 커서 부담이 될 때

* Supplier\<T\> 를 인수로 받는 orElseGet을 사용하면, 값이 처음 필요할 때 Supplier\<T\> 를 사용해 생성하므로 초기 설정 비용을 낮출 수 있다.

* 더 특별한 쓰임에 대비한 메서드 filter, map, flatMap, ifPresent

  * **ifPresent:** 옵셔널이 채워져 있으면 true, 비어 있으면 false를 반환한다.

    ##### 부모의 프로세스 ID를 출력하거나 없다면 "N/A"를 출력하는 코드

    ~~~java
    Optional<ProcessHandle> parentProcess = ph.parent();
    System.out.println("부모 PID: " + (parentProcess.isPresent() ?
            String.valueOf(parentProcess.get().pid()) : "N/A"));
    ~~~

    ##### 개선 코드

    ~~~java
    System.out.println("부모 PID: " +
        ph.parent().map(h -> String.valueOf(h.pid())).orElse("N/A"));
    ~~~



##### 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.

* 빈 Optional\<List\<T\>\> 를 반환하기 보다는 빈 List\<T\> 를 반환하는게 좋다.
* 빈 컨테이너를 그대로 반환하면 옵셔널 처리코드를 넣지 않아도 된다.



##### 어떤 경우에 T 대신 Optional\<T\> 를 반환할까?

기본 규칙은 결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 한다면 Optional\<T\> 를 반환한다.

* 성능이 중요한 상황에서는 옵셔널이 맞지 않을 수 있고, 이 상황에 처하는지 알아보기 위해 세심히 측정해보는 수밖에 없다.
* int, long, double 전용 옵셔널 클래스들 OptionalInt, OptionalLong, OptionalDobule 이 있으므로 박싱된 기본 타입을 담은 옵셔널을 반환하는 일은 없도록 하자.



##### 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는 게 적절한 상황은 거의 없다.

##### 핵심 정리

> 값을 반환하지 못할 가능성이 있고, 호출할 때마다 반환값이 없을 가능성을 염두에 둬야하는 메서드라면 옵셔널을 반환해야 하는 상황일 수 있다. 하지만 옵셔널 반환에는 성능 저하가 뒤따르니, 성능에 민감한 메서드라면 null을 반환하거나 예외를 던지는 편이 나을 수 있다. 그리고 옵셔널을 반환값 이외의 용도로 쓰는 경우는 매우 드물다.



___



## 아이템 56: 공개된 API 요소에는 항상 문서화 주석을 작성하라

API를 쓸모 있게 하려면 잘 작성된 문서도 곁들여야 한다. 
자바독은 소스 파일에서 문서화 주석이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해 준다.

##### 문서화 주석 작성법

##### API를 올바로 문서화하려면 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.

* 직렬화 할 수 있는 클래스라면 직렬화에 대해서도 작성한다.
* 공개 클래스에서는 절대 기본 생성자를 사용하면 안된다. 주석을 달 수가 없다.
* 유지보수까지 고려하면 대다수의 공개되지 않은 클래스, 인터페이스, 생성자, 메서드, 필드에도 문서화 주석을 달아야 한다.

##### 메서드용 문서화 주석에는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.

* 상속용 클래스의 메서드가 아니라면 어떻게 동작하는지가 아니라 **무엇을 하는지** 기술해야 한다.
* 클라이언트가 해당 메서드를 호출하기 위한 전제조건(precondition)을 모두 나열해야 한다.
* 메서드가 성공적으로 수행된 후에 만족해야 하는 사후조건(postcondition)도 모두 나열해야 한다.
* 일반적으로 전제조건은 @throws 태그로 비검사 예외를 선언하여 암시적으로 기술한다.
* @param 태그를 이용해 그 조건에 영향받는 매개변수에 기술할 수도 있다.
* 부작용도 문서화 해야한다.
  * 시스템의 상태에 어떤 변화를 가져오는 것

##### 메서드의 계약(contract)을 완벽히 기술

* 모든 매개변수에 @param 태그
* 반환 타입이 void가 아니라면 @return 태그
* 발생할 가능성이 있는 모든 예외에 @throws 태그

@param 태그와 @return 태그의 설명은 뜻하는 값이나 반환값을 설명하는 명사구를 사용한다.

##### 예제 

~~~java
 /**
  * 이 리스트에서 지정한 위치의 원소를 반환한다.
  *
  * <p>이 메서드는 상수 시간에 수행됨을 보장하지 <i>않는다</i>. 구현에 따라
  * 원소의 위치에 비례해 시간이 걸릴 수도 있다.
  *
  * @param  index 반환할 원소의 인덱스; 0 이상이고 리스트 크기보다 작아야 한다.
  * @return 이 리스트에서 지정한 위치의 원소
  * @throws IndexOutOfBoundsException index가 범위를 벗어나면,
  * 즉, ({@code index < 0 || index >= this.size()})이면 발생한다.
  */
 E get(int index) {
     return null;
 }
~~~

* 주석은 Html로 변환되므로 \<p\>, \<i\> 태그가 문서에 반영이 된다.
* {@code ... 코드 ...} 는 코드용 폰트로 렌더링을 해준다. 여러 줄에 걸쳐 사용하려면 \<pre\> 태그로 감싼다.

##### 

##### 상속용으로 설계할 때는 자기사용 패턴에 대해서도 문서에 남겨 올바르게 재정의하는 방법을 알려주어야 한다.

* @impleSpec 태그로 문서화를 한다.

* 이 태그는 해당 메서드와 하위 클래스 사이의 계약을 설명하여, 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지를 명확히 인지하고 사용하도록 해줘야 한다.

* 자바독 명령줄에서 옵션을 추가해야 한다.

  ~~~shell
  -tag "impleSpec:a:Implemenation Requirements:"
  ~~~

##### 예제

~~~java
 /**
  * 이 컬렉션이 비었다면 true를 반환한다.
  *
  * @implSpec 이 구현은 {@code this.size() == 0}의 결과를 반환한다.
  *
  * @return 이 컬렉션이 비었다면 true, 그렇지 않으면 false
  */
 public boolean isEmpty() {
     return false;
 }
~~~



##### API 설명에 <, >, & 등의 Html 메타문자를 포함시키기

* {@literal} 태그로 감싼다.
* 이 태그는 HTML 마크업이나 자바독 태그를 무시하게 해준다.



##### 각 문서화 주석의 첫 번째 문장은 해당 요소의 요약 설명으로 간주된다.

* 요약설명은 반드시 대상의 기능을 고유하게 기술해야 한다.
* 한 클래스(혹은 인터페이스) 안에서 요약 설명이 똑같은 멤버(혹은 생성자)가 둘 이상이면 안된다.
* 요약 설명은 첫 번째 마침표 . 까지 요약 설명 된다.
* 메서드와 생성자의 요약 설명은 해당 메서드와 생성자의 동작을 설명하는 (주어가 없는) 동사구여야 한다.
* 클래스, 인터페이스, 필드의 요약 설명은 대상을 설명하는 명사절이어야 한다.



##### 문서화 주석에서 제네릭, 열거 타입, 애너테이션은 특별히 주의해야 한다.

* 제네릭 타입이나 제네릭 메서드를 문서화 할 때는 모든 타입 매개변수에 주석을 달아야 한다.
* 열거 타입을 문서화할 때는 상수들에도 주석을 달아야 한다.
* 애너테이션 타입을 문서화 할 때는 멤버들에도 모두 주석을 달아야 한다.



##### 클래스 혹은 정적 메서드가 스레드 안전하든 그렇지 않든 스레드 안전 수준을 반드시 API 설명에 포함해야 한다.

##### 직렬화할 수 있는 클래스라면 직렬화 형태도 기술해야 한다.



##### 핵심 정리

> 문서화 주석은 여러분 API를 문서화하는 가장 훌륭하고 효과적인 방법이다. 공개 API라면 빠짐없이 설명을 달아야 한다. 표준 규약을 일관되게 지키자. 문서화 주석에 임의의 HTML 태그를 사용할 수 있음을 기억하라. 단, HTML 메타문자는 특별하게 취급해야 한다.



___

8장 메서드 끝...
