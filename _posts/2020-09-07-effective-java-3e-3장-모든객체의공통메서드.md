---
title: effective java 3/E - 3장. 모든 객체의 공통 메서드
tags: 
 - Java
key: 8
---

# 3장 모든 객체의 공통 메서드

##### 3장에서 배울 내용

* final이 아닌 Object 메서드들을 언제 어떻게 재정의해야 하는지 다룬다.
* equals, hashCode, toString, clone, finalize(아이템 8에서 다룸)
* 성격이 비슷한 Comparable.comapreTo



## 아이템 10: equals는 일반 규약을 지켜 재정의하라

 equals는 재정의하기 쉬워 보이지만 잘못 정의하면 끔찍한 결과를 초래한다. 문제를 피하는 가장 쉬운 길은 아예 재정의 하지 않는 것이다.

##### 다음 상황에는 재정의하지 않는 것이 최선이다.

* 각 인스턴스가 본질적으로 고유하다.

* 인스턴스의 논리적 동치성(logical equality)을 검사할 일이 없다.

* 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.

* 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

  equals가 실수로라도 호출되는 것을 막으려면 다음처럼 구현해두자.

  ~~~java
  @Override
  public boolean equals(Object o){
  	throw new AssertionError();
  }
  ~~~



##### equals를 재정의해야 할 때는 언제인가?

객체 식별성(object identity; 두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하지만, 상위 클래스의 equals가 **논리적 동치성**을 비교하도록 재정의 되지 않았을 때이다.
**equals**로 비교하는 것은 객체가 같은 것이 아니라 **객체 안의 값(필드)**들이 같은지 확인하는 것이다.

* 싱글턴, enum은 유일한 객체를 생성하므로 equals를 재정의 할 필요는 없다.



##### equals를 재정의 할 때 반드시 따라야 할 일반 규약 - 동치관계 구현

* **반사성(reflexivity)**: null이 아닌 모든 참조 값 x에 대해, `x.equals(x)`  는 true다.
* **대칭성(symmetry)**: null이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)` 가 true이면 `y.equals(x)`도 true 이다.
* **추이성(transitivity)**: null이 아닌 모든 참조 값 x, y, z에 대해, `x.equals(y)` 가 true이고 `y.equals(z)` 가 true 이면  `x.equals(z)` 도 true 이다.
* **일관성(consistency)**: null이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)` 를 반복해서 호출하면 항상 true 또는 항상 false를 반환한다.
* **null-아님**: null이 아닌 모든 참조값 x에 대해 `x.equals(null)` 은 false 다.



동치관계란 집합을 서로 같은 원소들로 이뤄진 부분집합으로 나누는 연산.

##### 동치관계를 만족시키기 위한 5가지 요건을 살펴보자

* **반사성**은 단순히 말하면 객체는 자기 자신과 같아야 한다는 뜻이다.

* **대칭성**은 두 객체는 서로 동치 여부에 똑같이 답해야 한다는 뜻이다.

  ex) 잘못된 equals 재정의

  ~~~java
  public final class CaseInsensitiveString {
      private final String s;
  
      public CaseInsensitiveString(String s) {
          this.s = Objects.requireNonNull(s);
      }
  
      // 대칭성 위배!
      @Override
      public boolean equals(Object o) {
          if (o instanceof CaseInsensitiveString)
              return s.equalsIgnoreCase(
                      ((CaseInsensitiveString) o).s);
          if (o instanceof String)  // 한 방향으로만 작동한다!
              return s.equalsIgnoreCase((String) o);
          return false;
      }
  }
  ~~~

  ~~~java
  CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
  String s = "Polish";
  ~~~

  위의 두 객체를 비교해보자. `cis.equals(s)` 를 한다면 재정의된 equals 메서드의 두번 째 if문에서 String과 비교하는 로직이 있기 때문에 true가 반환될 것이다. 하지만 반대로 `s.equals(cis)` 를 할 때 String의 equals에서는 CaseInsensitiveString의 존재를 모르기 때문에 다른 객체로 판단하여 false를 반환할 것이다. 이것은 대칭성을 명백히 위반한다.

  ##### 올바른 equals 작성

  ~~~java
  @Override 
  public boolean equals(Object o) {
      return o instanceof CaseInsensitiveString &&
              ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
  }
  ~~~

  오직 같은 클래스의 인스턴스와 비교를 하여 대칭성을 유지한다.

* **추이성** 은 첫 번째 객체와 두 번째 객체가 같다면 첫 번째 객체와 세 번째 객체도 같아야 한다.
  ex) 2차원 점을 표현하는 클래스 Point

  ~~~java
  public class Point {
      private final int x;
      private final int y;
  
      public Point(int x, int y) {
          this.x = x;
          this.y = y;
      }
  
      @Override 
  		public boolean equals(Object o) {
          if (!(o instanceof Point))
              return false;
          Point p = (Point)o;
          return p.x == x && p.y == y;
      }
  }
  ~~~

   Point 객체간 비교하는 것은 문제 없을 것이다. 하지만 하위 클래스를 생성하여 상위 클래스에 없는 필드를 추가하는 상황을 보자.

  ~~~java
  public class ColorPoint extends Point {
      private final Color color;
  
      public ColorPoint(int x, int y, Color color) {
          super(x, y);
          this.color = color;
      }
  }
  ~~~

  ColorPoint에서 equals를 재정의 하지 않는다면 Point의 equals를 상속받아 활용한다. 하지만 상속받은 equals는 색상 정보는 무시한채 비교를 수행한다. equals 규약은 어기지 않았지만 정확한 비교를 할 수 없다.

  ##### 위배 1 : 비교 대상이 다른 ColorPoint이며 위치와 색상이 같을 때만 true를 반환하는 equals

  ~~~java
  // ColorPoint의 equals : 잘못된 코드 - 대칭성 위배!
  @Override 
  public boolean equals(Object o) {
      if (!(o instanceof ColorPoint)) // Point와 비교는 항상 false
          return false;
        return super.equals(o) && ((ColorPoint) o).color == color;
  }
  ~~~

  이 메서드는 Point와 ColorPoint를 순서를 바꾸어서 비교를 할 때 그 결과가 다를 수 있다. Point의 equals는 색상을 무시하고 ColorPoint는 ColorPoint의 인스턴스가 아니므로 항상 false를 반환한다.

  ##### 위배 2 : ColorPoint의 equals가 Point와 비교할 때 색상을 무시하는 equals 

  ~~~java
  // 잘못된 코드 - 추이성 위배!
  @Override 
  public boolean equals(Object o) {
      if (!(o instanceof Point))
          return false;
  
  		// o가 일반 Point면 색상을 무시하고 비교한다.
  		if (!(o instanceof ColorPoint))
  				return o.equals(this);
  
  		// o가 ColorPoint면 색상까지 비교한다.
      return super.equals(o) && ((ColorPoint) o).color == color;
  }
  ~~~

  이 메서드는 대칭성은 지키지만 추이성을 깨버린다

  ~~~java
  ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
  Point p2 = new Point(1, 2);
  ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
  ~~~

  이 예제에서 p1과 p2는 true, p2와 p3는 true 이지만, p1과 p3는 false로 추이성을 깨버린다.

  #### 해법은? : 결론적으로 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.

  

  ##### Point의 equals의 instance 검사를 getClass 검사로 바꾼다면

  ~~~java
  // 잘못된 코드 - 리스코프 치환 원칙 위배!
  @Override 
  public boolean equals(Object o) {
  		if (o == null || o.getClass() != getClass())
  				return false;
  		Point p = (Point) o;
  		return p.x == x && p.y == y;
  }
  ~~~

  > 리스코프 치환 원칙(Liskov subsituation principle)
  >
  > 하위 클래스는 부모 클래스 타입으로 치환을 할 수 있어야 한다.
  >
  > ex) SuperClass sb = new SubClass();

  이와 같은 equals는 같은 구현 클래스의 객체와 비교할 때만 true를 반환한다. 괜찮아 보이지만 실제로는 사용이 불가능하다 . Point의 하위 클래스는 정의상 여전히 Point이므로 어디서든 Point로 활용될 수 있어야 한다.

  ##### 위배 3: Point에 추가된 메서드 - 반지름이 1인 단위 원 안에 있는지 판별하는 메서드

  ~~~java
  private static final Set<Point> unitCircle = Set.of(new Point(0, -1),
     new Point(0, 1),
     new Point(-1, 0),
     new Point(1, 0)
  );
  
  public static boolean onUnitCircle(Point p) {
      return unitCircle.contains(p);
  }
  ~~~

  ~~~java
  // Point의 평범한 하위 클래스 - 값 컴포넌트를 추가하지 않았다.
  public class CounterPoint extends Point {
      private static final AtomicInteger counter =
              new AtomicInteger();
  
      public CounterPoint(int x, int y) {
          super(x, y);
          counter.incrementAndGet();
      }
      public static int numberCreated() { return counter.get(); }
  }
  ~~~

  onUnitCircle 메서드의 매개변수로 CounterPoint를 전달한다면 CounterPoint의 x, y 값과 무관하게 false를 반환할 것이다. Set에서 equals로 객체가 같은지 판단을 하기 때문에 Point에서 getClass를 사용하여 재정의한 equals는 같은 클래스의 객체일 때만 true를 반환하므로 CounterPoint는 false로 판단을 한다.

  ##### 하위 클래스에 값을 추가할 수는 없지만 비슷하게 사용할 수 있는 우회적인 방법 - 상속 대신 컴포지션을 사용하라(아이템 18)

  ~~~java
  public class ColorPoint {
      private final Point point;
      private final Color color;
  
      public ColorPoint(int x, int y, Color color) {
          point = new Point(x, y);
          this.color = Objects.requireNonNull(color);
      }
  
      /**
       * 이 ColorPoint의 Point 뷰를 반환한다.
       */
      public Point asPoint() {
          return point;
      }
  
      @Override public boolean equals(Object o) {
          if (!(o instanceof ColorPoint))
              return false;
          ColorPoint cp = (ColorPoint) o;
          return cp.point.equals(point) && cp.color.equals(color);
      }
  
      @Override public int hashCode() {
          return 31 * point.hashCode() + color.hashCode();
      }
  }
  ~~~

  Point를 상속하지 않고 private 필드로 선언하고 일반 Point를 반환하는 view 메서드를 추가하는 식이다.

* **일관성** 은 두객체가 같다면(수정 되지 않는 한) 앞으로도 영원히 같아야 한다는 것이다. 가변 객체는 비교 시점에 따라 서로 다를 수도 같을 수도 있는 반면, 불변 객체는 한번 다르면 끝까지 달라야 한다.

* **null-아님** 은 모든 객체과 null과 같지 않아야 한다는 뜻이다. equals에서 형변환에 앞서 instanceof 연산자로 입력 매개변수가 올바른 타입인지 확인할 때 null 체크가 된다.

  

##### equals 메서드를 구현하는 방법

1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.

2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.

3. 입력을 올바른 타입으로 형변환한다.

4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다. 기본 타입 필드는 == 연산자로 비교하고 참조 타입 필드는 각각의 equals 메서드로, Float.compare(float, float), Double.compare(double, double) 활용

5. 다 구현하였다면 대칭성, 추이성 일관성을 확인하고 테스트해보자.

   

##### 마지막 주의사항

* equals 메서드를 재정의할 땐 hashCode도 반드시 재정의하자(아이템 11)

* 너무 복잡하게 해결하려들지 말자

* Object 외의 타입을 매개변수로 받는 equals는 선언하지 말자.

* equals, hashCode를 자동으로 만들어주는 AutoValue 프레임워크 활용(Google)

  

##### 핵심 정리

>꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해준다. 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 다섯 가지 규약을 확실히 지켜나가며 비교해야 한다.



___



## 아이템 11:  equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다. 그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap, HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

##### Object 명세의 규약

>* equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇  번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
>* equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
>* equals(Object)가 두 객체를 다르다고 판단햇더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

HashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 두 번째다. 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.

equals로 두 객체가 같다고 판단하더라도 hashCode가 둘을 다르다고 판단하여 서로 다른 값을 반환한다.

##### ex) PhoneNumber의 인스턴스를 HashMap의 원소로 사용

~~~java
// 10-6 전형적인 equals 메서드의 예
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix   = rangeCheck(prefix,   999, "프리픽스");
        this.lineNum  = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override 
  	public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
}
~~~

~~~java
HashMap<PhoneNumber,String> hash = new HashMap<>();
PhoneNumber p1 = new PhoneNumber(123,123,1234);
PhoneNumber p2 = new PhoneNumber(123,123,1234);
hash.put(p1 ,"크리스");
System.out.println(hash.get(p2));   // null 반환!
~~~

두 객체는 논리적으로 동일한 객체이지만 HashMap은 hashCode가 다른 엔트리끼리는 동치성 비교를 시도조차 하지 않도록 최적화가 되어 있기 때문에 null을 반환한다.

##### 해결 방법 - hashCode를 재정의하자

ex) 최악의 hashCode 구현 방법 - 사용금지

~~~java
@Override
public int hashCode() {
    return 42;
}
~~~

이 코드는 동치인 모든 객체에서 똑같은 해쉬코드를 반환하여 적법한 것처럼 보이지만, 해시테이블의 버킷 하나에 담겨 연결리스트 처럼 동작한다. 성능이 O(n)으로 느려져 사용하기 어렵다.

##### 좋은 hashCode 재정의

~~~java
@Override public int hashCode() {
    int result = Short.hashCode(areaCode); 
    result = 31 * result + Short.hashCode(prefix); 
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
~~~

1. int result 선언 후 객체의 핵심 첫 번째 핵심 필드의 hashCode로 초기화
2. 나머지 핵심필드 각각에 대하여 다음 작업 수행
   * 기본 타입 필드라면 박싱 클래스의 hashCode() 사용
   * 참조 타입 필드라면 참조 타입의 hashCode를 사용하고 null 이면 0을 사용
   * 필드가 배열이면 원소 각각을 별도의 필드처럼 다루어 hasCode 구함
3. 2번에서 정한 hashCode 값을 계산하여 result를 갱신한다.
   * int result = 31 * result + 2번에서 계산된 hashCode;
     * 31의 자리는 보통 소수의 값으로 정한다.

hashCode를 구현하였다면 동치인 인스턴스에 대해 동일한 hashCode를 반환하는지 확인한다. 단위 테스트 검증

##### Objects 클래스는 hashCode를 계산해주는 정적 메서드 hash를 제공

~~~java
@Override 
public int hashCode() {
		return Objects.hash(lineNum, prefix, areaCode);
}
~~~

임의 개수의 매개변수를 받아 hashCode 값을 반환해주는 메서드. 하지만 성능이 아쉽기에 성능에 민감하지 않은 상황에 사용하자. 

##### 클래스가 불변이고 해시코드를 계산하는 비용이 크다면 캐싱을 고려한다.

~~~java
// 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야 한다.
private int hashCode; // 자동으로 0으로 초기화된다.

@Override 
public int hashCode() {
		int result = hashCode;
    if (result == 0) {
	    result = Short.hashCode(areaCode);
      result = 31 * result + Short.hashCode(prefix);
      result = 31 * result + Short.hashCode(lineNum);
      hashCode = result;
    }
    return result;
}
~~~

성능을 높인다고 해시코드를 계산할 때 핵심 필드를 생략하면 안된다.

hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 게산 방식을 바꿀 수도 있다.



##### 핵심 정리

>equals를 재정의할 때는 hashCode도 반드시 재정의해야 한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다. 재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다. 이렇게 구현하기가 어렵지 않지만 조금 따분한 일이기는 하다. 하지만 아이템 10에서 이야기한 AutoValue 프레임워크를 사용하면 멋진 equals와 hashCode를 자동으로 만들어준다.



___



## 아이템 12: toString을 항상 재정의하라

Object의 기본 toString은 단순히 `클래스이름@16진수해시코드` 형태로 반환을 해준다. toString의 일반 규약은 '간결하면서 사람이 읽기 쉬운 형태의 유익한 정보' 를 반환해야 한다. 

##### ''모든 하위 클래스에서 이 메서드를 재정의하라''

* toString을 잘 구현한 클래스는 사용하기에 훨씬 즐겁고, 그 클래스를 사용한 시스템은 디버깅하기 쉽다.

* 실전에서 toString은 그 객체가 가진 주요 정보를 모두 반환하는게 좋다.

* toSring을 구현할 때 반환값의 포맷을 문서화 할지 정해야 한다.

  포맷을 명시하든 아니든 여러분의 의도는 명확히 밝혀야 한다.

  ex) 포맷 명시 O

  ~~~java
   		/**
       * 이 전화번호의 문자열 표현을 반환한다.
       * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
       * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
       * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
       * <p>
       * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
       * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
       * 전화번호의 마지막 네 문자는 "0123"이 된다.
       */
      @Override
      public String toString() {
          return String.format("%03d-%03d-%04d",
                  areaCode, prefix, lineNum);
      }
  ~~~

  포맷을 정하였다면 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공해주면 좋다. 

  **단점으로는** 포맷을 한번 명시하면 평생 그 포맷에 얽히게 된다. 이를 사용하는 프로그래머들이 그 포맷에 맞추어 파싱하고, 새로운 객체를 만들고, 영속 데이터로 저장하는 코드를 작성할 것이다. 그 후에 만약 이 포맷이 바꾼다면 이 코드를 사용하던 코드는 엉망이 될 것이다.

  ex) 포맷 명시 X

  ~~~java
  /** 
   * 이 약물에 관한 대략적인 설명을 반환한다. 
   * 다음은 이 설명의 일반적인 형태이나, 
   * 상세 형식은 정해지지 않았으며 향후 변경될 수 있다. 
   * 
   * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면, 
   * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면 
   */ 
  @Override 
  public String toString() { .... }
  ~~~

* 포맷 명시 여부와 상관없이 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자

##### 

##### 핵심 정리

> 모든 구체 클래스에서 Object의 toString을 재정의하자. 상위 클래스에서 이미 알맞게 재정의한 경우는 예외다. toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다. toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.



___



## 아이템 13: clone 재정의는 주의해서 진행하라

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이지만, 아쉽게도 의도한 목적을 제대로 이루지 못했다. 가장 큰 문제는 clone 메서드가 Object에 있고 protected라는 것이다. 그래서 Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다.

##### Cloneable 인터페이스가 하는 일

Cloneable 인터페이스는 Object의 protected인 clone의 동작 방식을 결정한다. Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스를 호출하면 CloneNotSupportedException을 던진다.

##### 제대로 동작하는 clone 메서드를 가진 상위 클래스를 상속해 Cloneable을 구현

~~~java
@Override 
public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();  // 일어날 수 없는 일이다.
    }
}
~~~

Object의 clone은 Object를 반환하지만 PhoneNumber 클래스는 PhoneNumber 클래스를 반환하게 했다. 자바는 공변 반환 타이핑(convariant return typing)을 지원하니 이렇게 하는 것이 가능하고 권장하는 방식이다.

클라이언트에서 형변환을 하지 않아도 되게끔 super.clone()로 얻은 객체를 반환하기전 (PhoneNumber)로 형변환을 한다.



##### clone을 구현하는 클래스가 가변 객체를 참조하고 있을 때 구현 방법 - Stack 예제

~~~java
private Object[] elements; // stack 이 관리하는 참조 객체

// 가변 상태를 참조하는 클래스용 clone 메서드
@Override 
public Stack clone() {
   try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
~~~

super.clone()으로 만 복제를 하고 결과를 반환한다면 elements 필드는 원본과 똑같은 배열을 참조할 것이다. 그래서 위 코드처럼 가변 객체를 참조하고 있는 부분을 내부적으로 복사를 해주어야 한다. 배열의 경우 clone() 메서드를 사용해서 복사를 한다. 배열은 clone 기능을 제대로 사용하는 유일한 예라고 할 수 있다.

##### 해시테이블 clone 예제 - 버킷의 배열, 각 버킷은 키, 값을 담는 연결리스트의 첫 번째 엔트리 참조

~~~java
public class HashTable implements Cloneable  {
  private Entry[] buckets = ...;
  
  private static class Entry {
    final Object key;
    Object value;
    Entry next;

    Entry(Object key, Object value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }
  }
  //...
}
~~~

ex) 잘못된 clone - 가변 상태를 공유한다.

~~~java
@Override
public HashTable clone() {
		try {
      HashTable result = (HashTable) super.clone();
      result.buckets = buckets.clone();
      return result;
    } catch(CloneNotSupportedException e) {
      throw new Assertion();
    }
}
~~~

buckets의 배열은 clone으로 복사를 하지만 원본의 연결리스트를 참조하여 예기치 않은 동작을 할 가능성이 있다.

**해결 방법**: 복사를 위해 deepCopy를 진행한다.

~~~java
// 엔트리 자신이 가리키는 연결 리스트를 반복적으로 복사한다.
Entry deeCopy(){
		Entry result = new Entry(key, value, next);
		for(Entry p = result; p.next != null; p = p.next){
				p.next = new Entry(p.next.key, p.next.value, p.next.next);
		}
		return result;
}

@Override
public HashTable clone() {
		try {
      HashTable result = (HashTable) super.clone();// 객체의 모든 필드를 복사
      result.buckets = new Entry[buckets.length];  // 새로운 버킷 배열로 초기화
      for(int i = 0; i < buckets.length; i++){     // 엔트리 깊은 복사 진행
        if(buckets[i] != null){
          result.buckets[i] = buckets[i].deepCopy();
        }
      }
      return result;
    } catch(CloneNotSupportedException e) {
      throw new Assertion();
    }
}
~~~



##### 생성자에서는 재정의될 수 있는 메서드를 호출하지 말아야 한다.

만약 clone이 하위 클래스에서 재정의한 메서드를 호출하면 하위 클래스는 복제 과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 수 있다.



##### public인 clone 메서드에서는 throws 절을 없애야 한다.

검사 예외를 던지지 않아야 그 메서드를 사용하기 편하다.



##### 상속해서 쓰기 위한 클래스 설계 방식에서 상속용 클래스는 Cloneable을 구현해서는 안된다.

clone을 동작하지 않게 구현해놓고 하위 클래스에서 재정의하지 못하게 하는 법

~~~java
@Override
protected final Object clone() throws CloneNotSupportedException {
		throw new CloneNotSupportedException();
}
~~~



##### Cloneable을 구현한 스레드 안전 클래스를 작성할 때는 clone 메서드 역시 적절히 동기화해줘야 한다.

Object의 clone은 동기화를 신경쓰지 않았기에 super.clone 호출 외에 다른 할 일이 없더라도 재정의하고 동기화를 해야한다.



##### 복사 생성자와 복사 팩터리라는 더 나은 객체 복사 방식을 제공할 수 있다.

단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자이다.

~~~java
// 복사 생성자
public Yum(Yum yum){...}

// 복사 팩터리
public static Yum newInstance(Yum yum) {...}
~~~

이 방식은 Cloneable/clone 방식보다 나은 면이 많다. 

* 언어 모순적이고 위험천만한 객체 생성 매커니즘(생성자를 사용하지 않는 방식)을 사용하지 않는다
* 엉성하게 문서화된 규약에 기대지 않는다
* 정상적인 final 용법과도 충돌하지 않는다
* 불필요한 검사 예외를 던지지 않고 형변환도 필요치 않다.



##### 핵심 정리

> Cloneable이 몰고 온 모든 문제를 되짚어봤을 때, 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다. final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 드물게 허용해야 한다. 기본 원칙은 복제 기능은 생성자와 팩터리를 이용하는게 최고라는 것이다. 단, 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.



___



## 아이템 14: Comparable을 구현할 지 고려하라

compareTo는 두 가지만 빼면 Object의 equals와 같다. 단순 동치성 비교에 더해 순서까지 비교할 수 있고, 제네릭하다. Comparable을 구현했다는 것은 그 클래스의 인스턴스들에는 자연적인 순서가 있음을 뜻한다. Comparable을 구현한 객체들의 배열은 `Arrays.sort(a);` 같이 쉽게 정렬을 할 수 있다.

Comparable을 구현하여 이 인터페이스를 활용하는 수많은 제네릭 알고리즘과 컬렉션의 힘을 누릴 수 있다. 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면 반드시 Comparable 인터페이스를 구현하자.

##### comparable 메서드의 일반 규약

>이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을 크면 양의 정수를 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
>
> 다음 설명에서 sgn(표현식) 표기는 수학에서 말하는 부호 함수(signum function)를 뜻하며, 표현식의 값이 음수, 0, 양수일 때 -1, 0, 1을 반환하도록 정의했다.
>
>* Comparable을 구현한 모든 클래스는 모든 x,  y에 대해 sgn(x.compareTo(y)) == -sgn(y.compareTo(x)) 여야 한다.
>
>* Comparable을 구현한 모든 클래스는 추이성을 보장해야 한다. 
>  (x.compareTo(y) > 0 && y.compareTo(z) > 0) 이면 x.compareTo(z) > 0 이다.
>
>* Comparable을 구현한 모든 클래스는 모든 z에 대해 
>  x.compareTo(y) == 0 이면 sgn(x.compareTo(z)) == sgn(y.comparTo(z)) 다.
>
>* 이번 권고가 필수는 아니지만 꼭 지키는게 좋다. (x.compareTo(y) == 0) == (x.equals(y)) 여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. 다음과 같이 명시하면 적당할 것이다.
>
>  "주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다."

첫 번째 규약은 두 객체 참조 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.

두 번째 규약은 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 한다.

마지막 규약은 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.

 세 규약은 compareTo 메서드로 수행하는 동치성 검사도 equals 규약과 똑같이 반사성, 대칭성, 추이성을 충족해야 함을 뜻한다. compareTo의 마지막 규약은 필수는 아니지만 꼭 지키길 권한다. 마지막 규약을 간단히 말하면 compareTo 메서드로 수행한 동치성 테스트의 결과가 equals와 같아야 한다는 것이다. 이를 잘 지키면 compareTo로 줄지은 순서와 equals의 결과가 일관되게 된다.

 compareTo 메서드의 작성 요령은 equals와 비슷하다. 몇 가지 차이점만 주의하면 된다. Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 타입은 컴파일 타임에 정해진다. 입력 인수의 타입을 확인하거나 형변환할 필요가 없다. 인수 타입이 잘못됐다면 컴파일 자체가 되지않고 null을 인수로 넣으면 NullPointerException을 던져야 한다.

 compareTo 메서드는 각 필드가 동치인지를 비교하는 것이 아니라 그 순서를 비교한다. Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면 비교자(Comparator)를 사용한다. 비교자는 직접 만들거나 자바가 제공하는 것을 사용한다.

ex) CasInsensitiveString의 compareTo 메서드 - 자바 제공 비교자 사용

~~~java
// 자바가 제공하는 비교자를 사용해 클래스를 비교한다.
public int compareTo(CaseInsensitiveString cis) {
    return String.CASE_INSENSITIVE_ORDER.compare(s, cis.s);
}
~~~

##### compareTo 메서드에서 관계 연산자 <, >를 사용하는 이전 방식보다, 박싱된 기본 타입의 정적 메서드 compare을 이용하자.

ex) PhoneNumber의 compareTo 메서드

~~~java
// 기본 타입 필드가 여럿일 때의 비교자
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0)  {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0)
            result = Short.compare(lineNum, pn.lineNum);
    }
    return result;
}
~~~



##### 자바 8에서는 Comparator 인터페이스가 일련의 비교자 생성 메서드와 연쇄 방식으로 비교자를 생성할 수 있다.

단 약간의 성능 저하가 뒤따른다.

ex) PhoneNumber의 compareTo 메서드

~~~java
// 비교자 생성 메서드를 활용한 비교자
private static final Comparator<PhoneNumber> COMPARATOR =
        comparingInt((PhoneNumber pn) -> pn.areaCode)
                .thenComparingInt(pn -> pn.prefix)    // 자바의 타입 추론 능력이
                .thenComparingInt(pn -> pn.lineNum);  // 첫 번째 명시한 타입으로 추론을 한다. 

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
~~~

이 코드는 클래스를 초기화 할 때 비교자 생성 메서드 2개를 이용해 비교자를 생성한다. 첫 번째인 comparingInt는 객체 참조를 int 타입 키에 매핑하는 키 추출 함수를 인수로 받아 그 키를 기준으로 순서를 정하는 비교자를 반환하는 정적 메서드다. 여기에서는 areaCode를 키로 하여 순서를 정한다. 이후 다음 비교는 thenComparingInt가 수행하며 prefix와 lineNum 순서로 순서를 정한다. 

comparing이 1차, thenComparing이 2차, 3차, ... 순서를 정한다.

##### 값의 차를 기준으로 비교를 하는 compareTo나 compare 메서드

o1 < o2 이면 음수 반환, o1 > o2 이면 양수 반환, o1 == o2 이면 0을 반환

###### ex) 추이성을 위배하는 예제

~~~java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
		public int compare(Object o1, Object o2){
				return o1.hashCode() - o2.hashCode();
		}
}
~~~

이 방식은 사용하면 안된다. 정수 오버플로우나 부동소수점 계산 방식에 따른 오류를 낼 수 있다.

###### ex) 대신 사용할 방법들

~~~java
// 정적 compare 메서드를 활용한 비교자를 사용하자
static Comparator<Object> hashCodeOrder = new Comparator<>() {
		public int compare(Object o1, Object o2){
				return Integer.compare(o1.hashCode(), o2.hashCode());
		}
}
~~~

~~~java
// 비교자 생성 메서드를 활용한 비교자를 사용하자
static Comparator<Object> hashCodeOrder = 
				Comparator.comparingInt(o -> o.hashCode());
~~~



##### 핵심 정리

> 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다. compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자는 쓰지 말아야 한다. 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 메서드 compare 메서드나 Comaprator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.



___
3장. 모든 객체의 공통 메서드 끝.
