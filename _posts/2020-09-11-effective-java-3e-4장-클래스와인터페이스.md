---
title: effective java 3/E - 4장. 클래스와 인터페이스
tags: 
 - Java
key: 9
---

# 4장 클래스와 인터페이스

##### 4장에서 배울 내용

* 클래스와 인터페이스를 사용하여 쓰기 편하고, 견고하며, 유연하게 만드는 방법



## 아이템 15: 클래스와 멤버의 접근 권한을 최소화하라

잘 설계된 컴포넌트는 클래스 내부 데이터와 내부 구현 정보를 외부로 부터 얼마나 잘 숨겼느냐의 여부이다. 잘 설계된 컴포넌트는 모든 내부 구현 내부를 완벽히 숨겨, 구현과 API를 깔끔히 분리한다. 정보은닉, 캡슐화라고 하는 이 개념은 소프트웨어 설계의 근간이 되는 원리이다.

##### 정보 은닉의 장점

* **시스템 개발 속도를 높인다.** 여러 컴포넌트를 병렬로 개발할 수 있기 때문이다.
* **시스템 관리 비용을 낮춘다.** 각 컴포넌트를 더 빨리 파악하여 디버깅할 수 있고, 다른 컴포넌트로 교체하는 부담도 적기 때문이다.
* **정보 은닉 자체가 성능을 높여주지는 않지만, 성능 최적화에 도움을 준다.** 완성된 시스템을 프로파일링해 최적화할 컴포넌트를 정한 다음, 다른 컴포넌트에 영향을 주지 않고 해당 컴포넌트만 최적화할 수 있기 때문이다.
* **소프트웨어 재사용성을 높인다.** 외부에 거의 의존하지 않고 독자적으로 동작할 수 있는 컴포넌트라면 그 컴포넌트와 함께 개발되지 않은 낯선 환경에서도 유용하게 쓰일 가능성이 크기 때문이다.
* **큰 시스템을 제작하는 난이도를 낮춰준다.** 시스템 전체가 아직 완성되지 않은 상태에서도 개별 컴포넌트의 동작을 검증할 수 있기 때문이다.

##### 자바의 정보 은닉을 위한 다양한 장치

 접근 제어 메커니즘은 클래스, 인터페이스, 멤버의 접근성(접근 허용 범위)를 명시한다. 각 요소의 접근성은 그 요소가 선언된 위치와 접근 제한자(private, protected, public)으로 정해진다. 이 접근 제한자를 제대로 활용하는 것이 정보 은닉의 핵심이다.

기본 원칙은 **모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.**

##### 톱레벨 클래스나 인터페이스의 접근 수준은 public과 package-private(default)만 가능하다.

* public 선언: 공개 API
  * API가 되면 하위 호환을 위해 영원히 관리가 필요하다.
* package-private 선언: 해당 패키지 내에서만 사용 가능. 외부에서 쓸일이 없다면 사용하자.
  * 클라이언트에 아무런 피해 없이 다음 릴리스에서 수정, 교체, 제거 가능하다.
  * 클래스 내 private static으로 중첩 시켜보자.
  * public 일 필요가 없는 클래스에 선언



##### 멤버(필드, 메서드, 중첩 클래스, 중첩 인터페이스)에 부여할 수 있는 접근 수준

* private: 멤버를 선언한 톱레벨 클래스에서만 접근 가능
* package-private: 멤버가 소속된 패키지 안의 모든 클래스에서 접근 가능
* protected: package-private을 포함하고 선언한 클래스의 하위 클래스에서 접근 가능
* public: 모든 클래스에서 접근 가능



##### 클래스의 멤버 접근 관리

* 클래스의 공개 API를 세심히 설계한 후, 그 외의 모든 멤버는 private으로 만들자. 같은 패키지의 다른 클래스가 접근해야 할 경우 package-private으로 풀어준다.

* public 클래스의 경우 멤버를 package-private에서 protected로 바꾸면 공개 API가 되어 영원히 지원되어야 한다.

* 상위 클래스의 메서드를 재정의할 때는 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없다. 이 제약은 상위 클래스의 인스턴스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다는 규칙(리스코프 치환 원칙)을 지키기 위해 필요하다.

* public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.

  * final이 아닌 필드가 public이라면 값을 제한할 힘을 잃게 된다.
  * 불변식을 보장할 수 없게 된다.
  * public 가변 필드를 갖으면 일반적으로 스레드 안전(thread-safe)하지 않다.

* 상수의 경우 public static final 필드로 공개 가능하다.

  * 배열은 static final로 두거나 반환하는 접근자 메서드를 제공해서는 안된다. 내용을 수정할 수 있게 된다.

    해결 법: 

    1. private 배열을 만들로 만들고 public 불변 리스트를 추가하여 사용

    2. 배열을 private로 만들고 clone으로 복사본을 반환

##### 핵심 정리

> 프로그램 요소의 접근성은 가능한 한 최소한으로 하라. 꼭 필요한 것만 골라 최소한의 public API를 설계하자. 그 외엔는 클래스, 인터페이스, 멤버가 의도치 않게 API로 공개 되는 일이 없도록 해야 한다. public 클래스는 상수용 public static final 필드 외에는 어떠한 public 필드도 가져서는 안된다. public static final 필드가 참조하는 객체가 불변인지 확인하라.



___



## 아이템 16: public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

##### 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공하라.

 필드에 public 접근자를 사용하면 직접 접근을 할 수 있으므로 캡슐화의 이점을 제공하지 못한다. 철저한 객체 지향 프로그래머는 이런 클래스를 상당히 싫어해서 필드를 모두 private으로 만들고 getter를 추가한다.

##### package-private, private 중첩 클래스라면 데이터를 노출해도 상관없다.

그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 된다. 클래스 선언 면에서는 접근자(getter)를 사용하는 것보다 코드면에서 훨씬 깔끔하다.

public 클래스의 필드가 final(불변)이라면 직접 노출할 때의 단점은 줄어들지만 좋은 생각은 아니다. API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행해야하는 단점이 있다.

##### 핵심 정리

> public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다. 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다. 하지만 package-private 클래스나 private 중첩 클래스에서는 종종(불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.



___



## 아이템 17: 변경 가능성을 최소화하라

불변 클래스란 인스턴스의 내부 값을 수정할 수 없는 클래스이다. 불변 인스턴스의 정보는 객체가 파괴되는 순간까지 절대 달라지지 않는다.  ex) String, BigInteger, BigDecimal...

불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안전하다.

##### 불변 클래스를 만드는 5가지 규칙

* 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
* 클래스를 확장할 수 없도록 한다.
* 모든 필드를 final로 선언한다.
* 모든 필드를 private으로 선언한다.
* 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

##### 불변 클래스 예제 - 복소수(Complex) 예제

~~~java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    //...
}
~~~

이 클래스는 복소수를 표현하는 클래스로 실수부와 허수부를 나타내는 필드는 final로 정의하였다. 그리고 실수부, 허수부 값을 얻기 위한 접근자 메서드를 정의하고 필드의 값을 바꿀 수 있는 메서드는 만들지 않았다. 이때 연산을 하는 메서드들은 인스턴트 자신은 수정하지 않고 새로운 객체를 만들어서 반환을 한다.

##### 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.

* 여러 스레드에서 객체를 사용하더라도 불변이기 때문에 값을 바꿀 수 없어 객체가 훼손될 걱정이 없다. 불변 객체는 안심하고 공유할 수 있다. 따라서 불변 객체는 한번 만든 인스턴스를 최대한 재활용하기를 권한다.

###### 자주 쓰이는 값들을 상수(public static final)로 정의

~~~java
public static final Complex ZERO = new Complex(0, 0);
public static final Complex ONE  = new Complex(1, 0);
public static final Complex I    = new Complex(0, 1);
~~~

* 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복해서 생성하지 않게 해주는 정적 팩터리를 제공할 수도 있다.

* 불변 객체는 방어적 복사도 필요가 없다. 불변이기 때문에 원본과 복사본은 똑같다.

##### 불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체 내부끼리는 내부 데이터를 공유할 수 있다.

* 내부 데이터를 변경할 수 없으므로 데이터를 공유해도 염려가 없다.

##### 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.

* Map의 키와 Set의 원소로 쓰이기에 안성맞춤이다. Map, Set은 안에 담긴 값이 바뀌면 불변식이 허물어지는데, 불변 객체는 그런 걱정이 필요없다.

##### 불변 객체는 그 자체로 실패 원자성을 제공한다.

* 상태가 절대 변하지 않으므로 잠깐이라도 불일치 상태에 빠질 가능성이 없다.

##### 불변 클래스의 단점은 값이 다르면 반드시 독립된 객체로 만들어야 한다는 것이다.

* 값의 가짓수를 많이 갖고 있는 불변 클래스라면 이들을 모두 만드는데 큰 값을 치뤄야 한다. 필드 하나의 값만 다르더라도 그것을 만들기 위해 나머지 동일한 필드와 다른 값인 하나 필드로 새로운 객체를 생성해야 한다.

* 원하는 객체를 완성하기까지 거쳐야할 중간 단계들이 있다면, 그 단계마다 새로운 객체를 생성하고 결국에는 필요가 없어져 버려진다면 성능 문제가 생길 수 있다.

  ##### 해결방안

  * 다단계 연산들을 예측하여 기본 기능으로 제공한다. 불변 객체는 내부적으로 다단계 연산 속도를 높혀주는 가변 동반 클래스를 package-private로 두고 있다. 복잡한 연산들을 정확히 예상할 수 있다면 package-private 가변 동반 클래스만으로 충분하다.
  * public으로 제공하는 케이스가 있다. String 클래스는 가변 동반 클래스로 StringBuilder를 public으로 제공한다.

##### 불변 클래스를 만드는 또 다른 방법

* 모든 생성자를 private 혹은 package-private로 만들고 public 정적 팩터리를 제공한다. 패키지 바깥에서 바라볼 때 이 불변 객체는 사실상 final이다. public, protected 생성자가 없으니 다른 패키지에서 이 클래스를 확장할 수 없기 때문이다.

##### 정리

* getter가 있다고 무조건 setter를 만들지는 말자. 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다.
* 성능 때문에 어쩔 수 없다면 가변 동반 클래스를 public으로 제공하자.
* 모든 클래스를 불변으로 만들 수는 없지만 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
* 다른 합당한 이유가 없다면 모든 필드는 private final이어야 한다.
* 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.



___



## 아이템 18: 상속보다는 컴포지션을 사용하라

상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다. 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만들게 된다. 상위 클래스와 하위클래스를 모두 같은 프로그래머가 통제하는 패키지 안에서라면 상속도 안전한 방법이다. 확장할 목적으로 설계되었고 문서화도 잘된 클래스도 마찬가지로 안전하다. 하지만 일반적인 패키지 경계를 넘어 다른 패키지의 구체 클래스를 상속하는 일은 위험하다.

##### 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.

상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다. 직접 만들지 않은 상위 클래스라면 릴리스마다 내부 구현이 달라질 수 있어, 코드 한줄 바꾸지 않은 하위 클래스가 오동작 할 수 있다. 이러한 이유로 상위 클래스 설계자가 확장을 충분히 고려하지 않고 문서화도 제대로 하지 않는 다면 하위 클래스는 상위 클래스의 변화에 따라 수정돼야 한다.

##### 하위 클래스가 깨지는 경우 1 - HashSet 예제 

~~~java
public class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
~~~

HashSet을 상속받아 객체 생성 후 원소가 몇 개 추가 되었는지 알 수 있도록 구현된 코드이다. 여기에서 addAll 메서드로 원소를 여러 개 추가를 한다고 하면 addCount에는 3이 저장될 것이라고 기대할 것이다. 실제로는 6을 반환하게 된다. 그 이유는 addAll은 super.addAll을 사용하면 HashSet의 addAll은 add로 원소를 추가한다. 이 때 호출되는 add는 상위 클래스인 HashSet의 add가 아닌 하위 클래스에서 재정의된 add를 호출하여 addCount가 두번씩 계산된다. 하위 클래스의 addAll에서 addCount 계산을 안 하면 기대한 대로 동작하겠지만, HashSet의 addAll이 add 메서드를 이용하여 구현했음을 가정한 해법이라는 한계를 지닌다.

 위의 예제처럼 개발자 스스로가 구현한 클래스가 아닌 클래스를 상속 받으려고 할 때, 상위 클래스가 평생 동일하게 유지되는 것을 보장할 수가 없다.

##### 하위 클래스가 깨지는 경우 2 - 다음 릴리스에서 상위 클래스의 메서드 추가

보안 때문에 컬렉션에 추가된 모든 원소가 특정 조건을 만족해야 한다. 그 컬렉션을 상속하여 원소를 추가하는 모든 메서드를 재정의해 필요한 조건을 먼저 검사하게 하면 될 것 같다. 하지만 이것은 새로운 메서드가 추가되기 전까지만 가능하고 새로운 메서드에서 허용되지 않은 원소를 추가하게 된다면 하위 클래스는 깨지게 된다.

##### 하위 클래스가 깨지는 경우 3 - 재정의하지 않고 하위 클래스의 새로운 메서드 추가

이 방식은 앞선 방식들보다 안전할 수 있지만 위험이 전혀 없지는 않다. 하위 클래스에서 정의한 메서드와 시그니처가 같고 반환 타입이 다른 메서드가 상위 클래스에서 만들어 진다면 컴파일 조차 되지 않는다. 반환 타입마저 같다면 재정의 한 것과 동일하므로 다른 방식의 문제가 또 발생할 수 있다.

##### 문제를 피해가는 묘안 - 컴포지션을 사용하자 😃

기존 클래스를 확장하는 대신 private 필드로 기존 클래스의 인스턴스를 참조하게 하자. 기존 클래스가 새로 만드는 클래스의 구성요소로 쓰인다는 뜻에서 **컴포지션(composition: 구성)**이라고 한다. 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다. 이 방식을 전달이라 하며, 새 클래스의 메서드들을 전달 메서드라고 부른다.

###### 래퍼 클래스와 전달 클래스를 사용하여 구현한 예제

~~~java
// 래퍼 클래스 - 상속 대신 컴포지션을 사용했다.
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }
}
~~~

~~~java
// 재사용할 수 있는 전달 클래스
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s; // 상속을 받는 대신 private 필드로 정의
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
~~~

HashSet의 기능을 가지고 있는 Set 인터페이스를 활용해 설계되어 견고하고 유연하다. InstrumentedSet은 다른 클래스를 감싸고 있다고 해서 래퍼 클래스라고 하며 데코레이터 패턴이라고도 한다.

래퍼 클래스의 단점은 거의 없다. 한 가지, 래퍼 클래스는 콜백 프레임워크와는 어울리지 않는 다는 것만 주의하면 된다.

##### 상속은 반드시 하위 클래스가 상위 클래스의 진짜 하위 타입인 경우에만 해야한다.

클래스 B가 A와 is-a 관계 일 때만 클래스 A를 상속해야 한다.

is-a 관계:  ex) 학생(B)은 사람(A)이다. 트럭(B)은 자동차(A)이다. 처럼 포함 관계가 성립할 때

is-a 관계라고 확신할 수 없다면 상속 대신 컴포지션을 사용하자.

##### 상속을 사용하기전 자문해야 할 사항

* 확장하려는 클래스의 API에 아무런 결함이 없는가?
* 결함이 있다면 결함이 여러분 클래스의 API로 전파돼도 괜찮은가?



##### 핵심 정리

> 상속은 강력하지만 캡슐화를 해친다는 문제가 있다. 상속은 상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 써야한다. is-a 관계일 때도 안심할 수만은 없는게, 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계되지 않았다면 여전히 문제가 될 수 있다. 상속의 취약점을 피하려면 상속 대신 컴포지션과 전달을 사용하자. 특히 래퍼 클래스로 구현할 적당한 인터페이스가 있다면 더욱 그렇다. 래퍼 클래스는 하위 클래스보다 견고하고 강력하다.



___



## 아이템 19: 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

상속을 고려하지 않은 외부 클래스를 사용하는 것은 프로그래머의 통제권 밖에 있어 언제 변경될 지 모른다.

##### 상속을 고려한 설계와 문서화

* 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지 문서로 남겨야 한다. public과 protected 중에 final이 아닌 모든 재정의 가능 메서드를 호출할 수 있는 상황을 문서로 남기자.

* 좋은 API 문서란 어떻게가 아닌 무엇을 하는지 설명해야 하지만, 상속이 캡슐화를 해치기 클래스를 안전하게 상속할 수 있도록 내부 구현 방식을 설명해야 한다.

* 메서드 주석에 @implSpec 태그를 붙이면 내부 동작을 설명하는 곳으로 'Implementation Requirements'을 붙여준다. 
  이 태그를 활성화 하기 위해서는 javadoc 명령에서 -tag 옵션을 지정해야 한다.
  ~~~shell
  ~> Javadoc -tag "implSpec:a:Implementation Requirements:" *.java
  ~~~

* 효율적인 하위 클래스를 큰 어려움 없이 만들 수 있게 하려면 클래스의 내부 동작 과정 중간에 끼어들 수 있는 훅(hook)을 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.

* 어떤 메서드를 protected로 만들지 결정하는 것은 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어 보는 것이 유일하다. 

* 널리 쓰일 클래스를 상속용으로 설계한다면 내부 사용 패턴과 protected 메서드, 필드를 구현하면서 선택한 결정을 책임져야 한다. 그래서 상속용으로 설계한 클래스는 배포전에 반드시 하위 클래스를 만들어 검증해야 한다.

* 상속용 클래스의 생성자는 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안된다.
  상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로 하위 클래스에서 재정의 한 메서드가 하위 클래스의 생성자보다 먼저 호출된다.

  ~~~java
  // 재정의 가능 메서드를 호출하는 생성자 - 따라 하지 말 것!
  public class Super {
      // 잘못된 예 - 생성자가 재정의 가능 메서드를 호출한다.
      public Super() {
          overrideMe();
      }
  
      public void overrideMe() {
      }
  }
  ~~~

   ~~~java
  // 생성자에서 호출하는 메서드를 재정의했을 때의 문제를 보여준다.
  public final class Sub extends Super {
      // 초기화되지 않은 final 필드. 생성자에서 초기화한다.
      private final Instant instant;
  
      Sub() {
          instant = Instant.now();
      }
  
      // 재정의 가능 메서드. 상위 클래스의 생성자가 호출한다.
      @Override public void overrideMe() {
          System.out.println(instant);
      }
  
      public static void main(String[] args) {
          Sub sub = new Sub();
          sub.overrideMe();
      }
  }
   ~~~

  > private, final, static 메서드는 재정의가 불가능하니 생성자에서 안심하고 호출해도 된다.

* Cloneable과 Serializable 인터페이스는 상속용 설계의 어려움을 한층 더 더해준다. 둘 중 하나라도 구현한 클래스를 상속할 수 있게 설계하는 것은 일반적으로 좋지 않은 생각이다.
  clone과 readObject 메서드는 모두 직접적으로든 간접적으로든 재정의 가능 메서드를 호출해서는 안 된다.

* Serializable을 구현한 상속용 클래스가 readObject나 writeReplace 메서드를 갖는다면 protected로 선언해야 한다.

클래스를 상속용으로 설계하려면 엄청난 노력이 들고 그 클래스에 안기는 제약도 상당하다.



##### 일반 구체 클래스의 상속용으로 설계하지 않은 클래스는 상속을 금지한다.

* final 클래스로 선언한다.
* 생성자를 private나 package-private로 선언하고 public 정적 팩터리를 만든다.

##### 핵심 정리

> 상속용 클래스를 설계하기란 결코 만만치 않다. 클래스 내부에서 스스로를 어떻게 사용하는지 모두 문서로 남겨야 하며, 문서화한 것은 그 클래스가 쓰이는 한 반드시 지켜야 한다. 그러지 않으면 내부 구현 방식을 믿고 활용하던 하위 클래스를 오동작하게 만들 수 있다. 다른 이가 효율 좋은 하위 클래스를 만들 수 있도록 일부 메서드를 protected로 제공해야 할 수도 있다. 그러니 클래스를 확장해야 할 명확한 이유가 떠오르지 않으면 상속을 금지하는 편이 나을 것이다. 상속을 금지하려면 클래스를 final로 선언하거나 생성자를 모두 외부에서 접근할 수 없도록 만들면 된다.



___



## 아이템 20: 추상 클래스보다는 인터페이스를 우선하라

자바가 제공하는 다중 구현 매커니즘은 인터페이스와 추상 클래스 두 가지가 있다. 

##### 둘의 가장 큰 차이

* 추상 클래스가 정의한 타입을 구현한 클래스는 반드시 추상 클래스의 하위 타입이 된다.
* 인터페이스가 선언한 메서드를 모두 정의하고 일반 규약을 잘 지킨 클래스라면 어떤 클래스를 상속했든 같은 타입으로 취급된다.



#### 인터페이스의 장점

* **기존 클래스에도 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.**

  인터페이스가 요구하는 메서드를 추가하고, 클래스 선언에 implements 구문을 추가하여 구현하면 된다.

  기존 클래스에 추상 클래스를 끼워 넣는 것은 일반적으로 어렵다.

* 인터페이스는 믹스인(mixin) 정의에 안성 맞춤이다.

  믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다.

  ex) Comparable 인터페이스



* **인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.** 

  타입을 계층적으로 정의하면 수많은 개념을 구조적으로 잘 표현할 수 있지만, 현실에서는 계층을 엄격히 구분하기 어려운 개념도 있다.

  ex) Singer, SongWriter 예제

~~~java
public interface Singer {
	AudioClip sing(Song s);
}
~~~

~~~java
public interface SongWriter {
	Song compose(int chartPosition);
}
~~~

​	가수이면서 작곡가인 경우처럼 interface로 구현하면 모두 구현해도 된다.

~~~java
public interface SingerSongWriter extends Singer, SongWriter{
  AudioClip strum();
  void actSensitive();
}
~~~



* **래퍼 클래스 관용구와 함께 사용하면 인터페이스는 기능을 향상 시키는 안전하고 강력한 수단이 된다.**

* 상속해서 만든 클래스는 래퍼클래스보다 활용도가 떨어지고 깨지기는 더 쉽다.

* 인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공할 수 있다. 
  equals와 hashCode는 디폴트 메서드로 제공해서는 안되고 인스턴스 필드를 가질 수 없다. public이 아닌 정적 멤버도 가질 수 없다. (private 정적 메서드는 가능)

* 인터페이스와 추상 골격 구현(skeletal implementation) 클래스를 함께 제공하는 식으로 인터페이스와 추상 클래스의 장점을 모두 취하는 방법도 있다. 

  인터페이스로 타입을 정의하고 필요하면 디폴트 메서드 몇 개도 함께 제공한다. 그리고 골격 구현 클래스는 나머지 메서드들 까지 구현한다. 템플릿 메서드 패턴
  관례상 인터페이스 이름이 Interface 이면 골격 구현 클래스는 AbstractInterface가 된다.

  ex) AbstractCollection, AbstractSet, AbstractList ...

  ##### 골격 구현을 사용해 완성한 구체 클래스

  ~~~java
  public class IntArrays {
      static List<Integer> intArrayAsList(int[] a) {
          Objects.requireNonNull(a);
  
          // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
          // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
          return new AbstractList<>() {
              @Override public Integer get(int i) {
                  return a[i];  // 오토박싱(아이템 6)
              }
  
              @Override public Integer set(int i, Integer val) {
                  int oldVal = a[i];
                  a[i] = val;     // 오토언박싱
                  return oldVal;  // 오토박싱
              }
  
              @Override public int size() {
                  return a.length;
              }
          };
      }
  }
  ~~~

  int 배열을 받아 Integer 인스턴스의 리스트 형태로 보여주는 어댑터

* 골격 구현 클래스의 아름다움은 추상 클래스처럼 구현을 도와주는 동시에, 추상 클래스로 타입을 정의할 때 따라오는 심각한 제약에서는 자유롭다는 점에 있다.

* 골격 구현 작성은 상대적으로 쉽다. 

  1. 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드를 선정한다.
  2. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 디폴트 메서드로 제공한다.
     (Object의 메서드들은 디폴트 메서드로 만들면 안됨!)
  3. 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아 있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드들을 작성해 넣는다.

  ex) Map.Entry 인터페이스 예제 getKey, getValue는 기반 메서드

  ~~~java
  // 골격 구현 클래스
  public abstract class AbstractMapEntry<K,V>
          implements Map.Entry<K,V> {
      // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야 한다.
      @Override public V setValue(V value) {
          throw new UnsupportedOperationException();
      }
      
      // Map.Entry.equals의 일반 규약을 구현한다.
      @Override public boolean equals(Object o) {
          if (o == this)
              return true;
          if (!(o instanceof Map.Entry))
              return false;
          Map.Entry<?,?> e = (Map.Entry) o;
          return Objects.equals(e.getKey(),   getKey())
                  && Objects.equals(e.getValue(), getValue());
      }
  
      // Map.Entry.hashCode의 일반 규약을 구현한다.
      @Override public int hashCode() {
          return Objects.hashCode(getKey())
                  ^ Objects.hashCode(getValue());
      }
  
      @Override public String toString() {
          return getKey() + "=" + getValue();
      }
  }
  
  ~~~

  > Map.Entry 인터페이스나 그 하위 인터페이스로는 이 골격 구현을 제공할 수 없다. 디폴트 메서드는 equals, hashCode, toString 같은 Object 메서드를 재정의할 수 없기 때문이다.

* **단순 구현은 골격 구현의 작은 변종으로 AbstractMap.SimpleEntry가 좋은 예이다.**



##### 핵심 정리

> 일반적으로 다중 구현용 타입으로는 인터페이스가 가장 적합하다. 복잡한 인터페이스라면 구현하는 수고를 덜어주는 골격 구현을 함께 제공하는 방법을 꼭 고려해보자. 골격 구현은 '가능한 한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다. '가능한 한'이라고 한 이유는, 인터페이스에 걸려 있는 구현상의 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하기 때문이다.



___



## 아이템 21: 인터페이스는 구현하는 쪽을 생각해 설계하라

자바 8 전에는 기존 구현체를 깨뜨리지 않고는 인터페이스에 메서드를 추가 할 방법이 없었다. 인터페이스에 메서드를 추가하면 컴파일 오류가 났다. 자바 8에 와서 디폴트 메서드를 추가 할 수 있게 되었다.

디폴트 메서드를 선언하면, 그 인터페이스를 구현한 후 디폴트 메서드를 재정의하지 않은 모든 클래스에서 디폴트 구현이 쓰인다. 하지만 기존 구현체들과 매끄럽게 연동되리라는 보장은 없다.

자바 8에서는 핵심 컬렉션 인터페이스들에 다수의 디폴트 메서드가 추가되었다. 주로 람다를 활용하기 위해서다. 자바 라이브러리의 디폴트 메서드는 코드 품질이 높고 범용적이라 대부분 상황에서 잘 작동한다. **하지만 생각할 수 있는 모든 상황에서 불변식을 해치지 않는 디폴트 메서드를 작성하기란 어려운 법이다.**

디폴트 메서드를 추가했을 때 기존에 인터페이스를 구현한 클래스에서 재정의를 하고 있지 않기 때문에 예기치 못한 문제나 결과로 이어질 수 있다.

##### 디폴트 메서드는 기존 구현체에 런타임 오류를 일으킬 수 있다.

기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니라면 피해야 한다. 반면 새로운 인터페이스를 만드는 경우라면 표준적인 메서드 구현을 제공하는데 아주 유용한 수단이며, 그 인터페이스를 더 쉽게 구현해 활용할 수 있게 해준다.

디폴트 메서드는 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도가 아님을 명시해야 한다.

##### 핵심은 인터페이스를 설계 할 때에는 여전히 세심한 주의를 기울여야 한다는 것이다.

##### 새로운 인터페이스를 릴리스 전에 반드시 테스트를 거쳐야 한다. 인터페이스를 릴리스한 후라도 결함을 수정하는게 가능한 경우도 있겠지만, 절대 그 가능성에 기대서는 안된다.



___



## 아이템 22: 인터페이스는 타입을 정의하는 용도로만 사용하라

인터페이스는 자신을 구현한 클래스의 인스턴스를 참조할 수 있는 타입 역할을 한다. 인터페이스는 오직 이 용도로만 사용해야 한다.

##### 안티 패턴 - 상수 인터페이스 사용 금지

~~~java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
~~~

상수 인터페이스 안티 패턴은 인터페이스를 잘못 사용한 예다. 클래스 내부에서 사용하는 상수는 외부 인터페이스가 아니라 내부 구현에 해당한다. 상수 인터페이스를 구현하는 것은 내부 구현을 클래스의 API로 노출하는 행위다. 

##### 상수를 공개할 목적이라면 더 합당한 선택지

* 특정 클래스나 인터페이스와 강하게 연결된 상수라면 그 클래스나 인터페이스 자체에 추가한다.
  ex) Interger.MIN_VALUE, Integer.MAX_VALUE

* 열거 타입으로 나타내기 적합한 상수라면 열거타입으로 만들어 공개한다.

* 인스턴스화 할 수 없는 유틸리티 클래스에 담아 공개한다.

  ~~~java
  public class PhysicalConstants {
    private PhysicalConstants() { }  // 인스턴스화 방지
  
    // 아보가드로 수 (1/몰)
    public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
  
    // 볼츠만 상수 (J/K)
    public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;
  
    // 전자 질량 (kg)
    public static final double ELECTRON_MASS    = 9.109_383_56e-31;
  }
  ~~~

  유틸리티 클래스의 상수를 사용하려면 **클래스명.상수명** 으로 호출해야 한다. `PhysicalConstants.ELECTRON_MASS`

  상수를 자주 사용한다면 정적 임포트를 통해 상수명만으로 사용할 수 있다

  ~~~java
  import static chapter4.item22.PhysicalConstants.*;
  ~~~

  

##### 핵심 정리

> 인터페이스는 타입을 정의하는 용도로만 사용해야 한다. 상수 공개용 수단으로 사용하지 말자.



___



## 아이템 23: 태그 달린 클래스보다는 클래스 계층구조를 활용하라

태그 달린 클래스란 한 개의 클래스에서 두 가지 이상의 의미를 표현하고 그중 현재 표현하는 의미를 태그 값으로 알려주어 클래스의 상태를 표현하는 클래스이다.

##### 예제 태그 달린 클래스, 클래스 계층 구조보다 훨씬 나쁘다!

~~~java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
~~~

##### 태그 달린 클래스에는 단점이 가득하다. 

* 열거 타입, 태그 필드, switch 문 등 쓸데 없는 코드가 많다.
* 여러 구현이 한 클래스에 혼합되어 있어 가독성이 나쁘다.
* 다른 의미를 위한 코드도 함께하니 메모리도 많이 사용한다.
* 필드를 final로 선언하려면 해당 의미에서 쓰이지 않는 필드까지 초기화 해야한다.
* 또 다른 의미를 추가하려면 코드를 수정해야 한다. ex) switch 문에서 다른 의미로 분기하는 처리
* 인스턴스 타입만으로 현재 나타내는 의미를 알 수 없다.
* **태그 달린 클래스는 장황하고 오류 내기 쉽고, 비효율적이다.**



##### 태그 달린 클래스보다는 계층구조를 사용하자.

~~~java
abstract class Figure {
    abstract double area();
}
~~~

~~~java
class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}
~~~

~~~java
class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    @Override double area() { return length * width; }
}
~~~

태그 달린 클래스의 단점을 모두 날린 계층구조 클래스이다. 간결하고 명확하며 쓸데 없는 코드가 모두 사라졌다.

루트 클래스의 코드를 건드리지 않고 다른 프로그래머들이 독립적으로 계층구조를 확장하고 함께 사용할 수 있다.

##### 정사각형을 지원하도록하는 클래스 생성 - Rectangle 클래스 상속

~~~java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
~~~



##### 핵심 정리

> 태그 달린 클래스를 써야 하는 상황은 거의 없다. 새로운 클래스를 작성하는 데 태그 필드가 등장한다면 태그를 없애고 계층구조로 대체하는 방법을 생각해보자. 기존 클래스가 태그 필드를 사용하고 있다면 계층구조로 리팩터링하는 걸 고민해보자.



___



## 아이템 24: 멤버 클래스는 되도록 static으로 만들라

중첩 클래스(nested class)란 다른 클래스안에 정의된 클래스를 말한다. 중첩 클래스는 자신을 감싼 클래스에서만 쓰여야 하며, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

##### 중첩 클래스 종류

* 정적 멤버 클래스
* 비정적 멤버 클래스
* 익명 클래스
* 지역 클래스



##### 정적 멤버 클래스

* 정적 멤버 클래스는 다른 클래스 안에 선언되고, 바깥 클래스의 private 멤버에도 접근할 수 있다는 점만 제외하고 일반 클래스와 똑같다. 
* private로 선언을 하면 바깥 클래스에서만 접근할 수 있다. 

* 바깥 클래스와 함께 쓰일 때만 유용한 public 도우미로 쓰인다.
* private 정적 멤버 클래스는 흔히 바깥 클래스가 표현하는 객체의 한 부분을 나타낼 때 쓴다.



##### 비정적 멤버 클래스

* 정적 멤버 클래스와 비정적 멤버 클래스는 static 유무 차이지만 의미상 차이는 꽤 크다. 비정적 멤버 클래스의 인스턴스는 바깥 클래스와 암묵적으로 연결되어 비정적 멤버 클래스의 메서드에서는 바깥클래스이름.this 으로 메서드 호출이나 참조를 가져올 수 있다. 
* 개념상 중첩 클래스의 인스턴스가 바깥 인스턴스와 독립적으로 존재할 수 있다면 정적 멤버 클래스로 만들어야 한다. 비정적 메서드는 바깥 인스턴스 없이는 생성할 수 없기 때문이다.
* 바깥 클래스가 비정적 멤버 클래스의 생성자를 호출할 때 자동으로 만들어지는게 보통이지만, 드물게 직접 
   `바깥인스턴스의클래스.new 비정적멤버클래스()` 를 호출해 수동으로 만들기도 한다. (메모리 공간 차지, 생성 시간 더 걸림)
* 어댑터를 정의할 때 자주 쓰인다.
* **멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자.**
* static이 없다면 바깥 인스턴스의 숨은 외부 참조를 갖게 되어 가비지 컬렉터가 수거하지 못해 메모리 누수가 일어날 수 있다.



##### 익명 클래스

* 익명 클래스는 당연히 이름이 없다.
* 바깥 클래스의 멤버도 아니다.
* 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
* 비정적 문맥에서만 바깥 클래스의 인스턴스 참조 가능하다.
* 정적 문맥에서 상수 변수 이외의 정적 멤버는 가질 수 없다.
* 선언한 지점에서만 인스턴스를 만들 수 있고 instanceof 검사나 클래스 이름이 필요한 작업은 수행할 수 없다.
* 작은 함수나 처리 객체를 만드는데 사용했지만 람다에게 그 자리를 대신한다.
* 주 쓰임은 정적 팩터리 메서드를 구현할 때다.



##### 지역 클래스

* 지역 변수를 선언할 수 있는 곳이라면 선언 가능하고, 유효 범위는 지역 변수와 같다.
* 멤버 클래스 처럼 이름이 있고 반복해서 사용할 수 있다.
* 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있으며 정적 멤버는 가질 수 없다.
* 가독성을 위해 짧게 작성한다.



##### 핵심 정리

> 중첩 클래스에는 네 가지가 있으며, 각각의 쓰임이 다르다. 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기엔 너무 길다면 멤버 클래스로 만든다. 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적으로, 그렇지 않으면 정적으로 만들자. 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스로 만들고, 그렇지 않으면 지역 클래스로 만들자.



___



## 아이템 25: 톱레벨 클래스에는 한 파일에 하나만 담으라

소스 파일 하나에 톱레벨 클래스 여러 개를 선언하더라도 자바 컴파일러는 불평하지 않는다. 

하지만 아무런 득이 없을 뿐더러 심각한 위험을 감수해야 하는 행위이다.

##### 두 클래스가 한 소스 파일에 존재할 때 문제

###### Utensil.java

~~~java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
~~~

###### Desert.java

~~~java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
~~~

###### Main.java

~~~java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
~~~

 javac Main.java, javac Main.java Utensil.java 로 실행하면 pancake 을 출력하지만 

javac Desert.java Main.java 명령은 potpie 를 출력한다.

컴파일러에 어느 소스를 먼저 건네느냐에 따라 동작이 달라지므로 반드시 바로 잡아야 한다.

##### 해결책

단순히 톱레벨 클래스를 서로 다른 소스 파일로 분리하면 된다.

만약 한 소스 파일에 같이 담고 싶다면 private 정적 멤버 클래스로 만들어 관리한다.



##### 핵심 정리

> 교훈은 명확하다. 소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자. 이 규칙만 따른다면 컴파일러가 한 클래스에 대한 정의를 여러 개 만들어 내는 일은 사라진다. 소스 파일을 어떤 순서로 컴파일하든 바이너리 파일이나 프로그램의 동작이 달라지는 일은 결코 일어나지 않을 것이다.



___

4장 클래스와 인터페이스 끝
