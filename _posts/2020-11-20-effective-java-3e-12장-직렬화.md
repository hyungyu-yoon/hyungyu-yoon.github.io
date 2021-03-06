---
title: effective java 3/E - 12장. 직렬화
tags: 
 - Java
key: 17


---



# 12장 직렬화

##### 12장에서 배울 내용

* 객체 직렬화란 자바가 객체를 바이트 스트림으로 인코딩하고(직렬화) 그 바이트 스트림으로부터 다시 객체를 재구성하는(역직렬화) 메커니즘이다.
* 직렬화된 객체는 다른 VM에 전송하거나 디스크에 저장한 후 나중에 역 직렬화할 수 있다.
* 직렬화가 품고 있는 위험과 그 위험을 최소화하는 방법에 집중한다.



___



## 아이템 85: 자바 직렬화의 대안을 찾으라

자바에 직렬화가 처음 도입된 후 어렵지 않게 분산 객체를 만들 수 있다는 것은 매력적이었지만, 보이지 않는 생성자, API와 구현 사이의 모호해진 경계, 잠재적인 정확성 문제, 성능 보안, 유지보수성 등 문제에 대한 대가가 크다.

##### 직렬화의 근본적인 문제는 공격 범위가 너무 넓고 지속적으로 더 넓어져 방어하기 어렵다는 점이다.

* ObjectInputStream의 readObject 메서드를 호출하면 객체 그래프가 역직렬화되기 때문이다.
* readObject 메서드는 클래스패스 안의 거의 모든 타입의 객체를 만들어 낼 수 있다.
* 바이트 스트림을 역직렬화하는 과정에서 이 메서드는 그 타입들 안의 모든 코드를 수행할 수 있으며, 이는 곧 타입들의 코드 전체가 공격 범위에 들어간다는 뜻이다.

##### 자바의 표준 라이브러리나 아파치 커먼즈 컬렉션 같은 서드파티는 물론 애플리케이션 자신의 클래스들도 공격 범위에 속한다.

* 모든 직렬화 가능 클래스들을 공격에 대비하도록 작성하여도, 애플리케이션은 취약할 수 있다.

##### 역직렬화에 시간이 오래 걸리는 짧은 스트림을 역직렬화하는 폭탄이라고 한다.

###### 예제 코드) 역직렬화 폭탄 - 이 스트림의 역 직렬화는 영원히 계속된다.

~~~java
public class DeserializationBomb {
    public static void main(String[] args) throws Exception {
        System.out.println(bomb().length);
        deserialize(bomb());
    }

    static byte[] bomb() {
        Set<Object> root = new HashSet<>();
        Set<Object> s1 = root;
        Set<Object> s2 = new HashSet<>();
        for (int i = 0; i < 100; i++) {
            Set<Object> t1 = new HashSet<>();
            Set<Object> t2 = new HashSet<>();
            t1.add("foo"); // t1을 t2와 다르게 만든다.
            s1.add(t1);
            s1.add(t2);
            s2.add(t1);
            s2.add(t2);
            s1 = t1;
            s2 = t2;
        }
        return serialize(root); // 이 메서드는 effectivejava.chapter12.Util 클래스에 정의되어 있다.
    }
}
~~~

1. 이 객체 그래프는 201개의 HashSet 인스턴스로 구성되며, 각각은 3개 이하의 객체 참조를 갖는다.
2. 스트림의 전체 크기는  5744 바이트이지만, 역직렬화는 영원히 끝나지 않을 것이다.
3. 문제는 HashSet 인스턴스를 역직렬화하려면 그 원소들의 해시코드를 계산해야 한다는 데 있다.
4. 루트 HashSet에 담긴 두 원소는 각각 다른 HashSet 2개씩을 원소로 갖는 HashSet이다.
5. 반복문에 의해 이 구조가 깊이 100 단계 까지 만들어진다.
6. 이 HashSet을 역직렬화하려면 hashCode를 2¹⁰⁰ 번 넘게 호출해야 한다.
7. 역직렬화가 계속되는 것도 문제지만, 무언가 잘못되었다는 신호조차 주지 않으며 단 몇 개의 객체만 생성해도 스택 깊이 제한에 걸린다.

##### 직렬화 위험을 회피하는 가장 좋은 방법은 아무것도 역직렬화하지 않는 것이다.

* 작성하는 새로운 시스템에서 자바 직렬화를 써야할 이유는 전혀 없다.

* 객체와 바이트 시퀀스를 변환해주는 다른 메커니즘이 많이 있다.

  이 방식들은 자바 직렬화의 위험을 회피하면서 다양한 플랫폼 지원, 우수한 성능, 풍부한 지원 도구, 활발한 커뮤니티와 전문가 집단 등 수많은 이점까지 제공한다.

##### 데이터 표현의 선두주자는 JSON과 프로토콜 버퍼다.

* JSON은 브라우저와 서버의 통신용으로 설계되었다.
* 프로토콜 버퍼는 서버 사이에 데이터를 교환하고 저장하기 위해 설계했다.

##### JSON과 프로토콜 버퍼의 차이

* JSON은 텍스트 기반으로 사람이 읽을 수 있고, 데이터를 표현하는 데만 쓰인다.
* 프로토콜 버퍼는 이진 표현이라 효율이 훨씬 높으며, 문서를 위한 스키마(타입)를 제공하고 올바로 쓰도록 강요한다.
* JSON은 텍스트 기반 표현에 아주 효과적이고, 프로토콜 버퍼는 사람이 읽을 수 있는 텍스트 표현(pbtxt)도 지원한다.

##### 레거시 시스템 때문에 자바 직렬화를 완전히 배제할 수 없을 때의 차선책

* 신뢰할 수 없는 데이터는 절대 역직렬화하지 않는 것이다.

* 자바의 공식 보안 코딩 지침에서는 "신뢰할 수 없는 데이터의 역직렬화는 본질적으로 위험하므로 절대로 피해야 한다"라고 조언한다.

* 직렬화를 피할 수 없고 역직렬화한 데이터가 안전한지 완전히 확신할 수 없다면 역직렬화 필터링(java.io.ObjectInputFilter)을 사용하자.

  * 객체 역직렬화 필터링은 데이터 스트림이 역직렬화되기 전에 필터를 설치하는 기능이다.

  * 클래스 단위로, 특정 클래스를 받아들이거나 거부할 수 있다.

  * '기본 수용' 모드에서 블랙리스트에 기록된 잠재적 위험한 클래스를 거부한다.

  * '기본 거부' 모드에서 화이트리스트에 기록된 안전한 클래스들만 수용한다.

  * ##### 화이트 방식을 추천한다.

##### 직렬화는 여전히 자바 생태계 곳곳에 쓰이고 있다.

* 시간과 노력을 들여서라도 JSON이나 프로토콜 버퍼 같은 대안으로 마이그레이션하는 것을 심각하게 고민해보길 바란다.
* 하지만 현실적인 이유로 직렬화 가능 클래스를 작성하거나 유지보수해야 할 수 있기에 직렬화 가능 클래스를 올바르고 안전하고 효율적으로 작성하려면 상당한 주의가 필요하다.



##### 핵심 정리

> 직렬화는 위험하니 피해야 한다. 시스템을 밑바닥부터 설계한다면 JSON이나 프로토콜 버퍼 같은 대안을 사용하자.  신뢰할 수 없는 데이터는 역직렬화하지 말자. 꼭 해야 한다면 객체 역직렬화 필터링을 사용하되, 이마저도 모든 공격을 막아줄 수는 없음을 기억하자. 클래스가 직렬화를 지원하도록 만들지 말고, 꼭 그렇게 만들어야 한다면 정말 신경써서 작성해야 한다.



___



## 아이템 86: Serializable을 구현할지는 신중히 결정하라

어떤 클래스의 인스턴스를 직렬화할 수 있게 하려면 Serializable을 구현하면 된다. 직렬화를 지원하기란 짧게 보면 쉬워보이지만, 길게 보면 아주 값비싼 일이다.

##### Serializable을 구현하면 릴리스한 뒤에는 수정하기 어렵다.

* 클래스가 Serializable을 구현하면 직렬화된 바이트 스트림 인코딩(직렬화된 형태)도 하나의 공개 API가 된다.
* 이 클래스가 널리 퍼진다면 그 직렬화 형태도 영원히 지원해야 하는 것이다.
* 커스텀 직렬화 형태를 설계하지 않고 기본 방식을 사용한다면 직렬화 형태는 최소 적용 당시의 내부 구현 방식에 영원히 묶여버린다.
* 기본 직렬화 형태에서는 클래스의 private과 package-private 인스턴스 필드들마저 API로 공개되는 꼴이 된다.
* 뒤늦게 수정하기도 어렵기 때문에, 직렬화 가능 클래스를 만들고자 한다면 고품질의 직렬화 형태도 주의해서 함께 설계해야 한다.

##### 직렬화가 클래스 개선을 방해하는 간단한 예

* ##### 대표적으로 스트림 고유 식별자, 직렬 버전 UID(serial version UID)를 들 수 있다.

  * 모든 직렬화 클래스는 고유 식별 번호를 부여받는다. **serialVersionUID**
  * 이 번호를 명시하지 않으면 시스템이 런타임에 암호 해시 함수를 적용해 자동으로 클래스 안에 생성해 넣는다.
  * 이 값을 생성하는 데는 클래스 이름, 구현한 인터페이스들, 컴파일러가 자동으로 생성해 넣은 것을 포함한 대부분의 클래스 멤버들이 고려된다.
  * 나중에 내부 수정을 한다면 UID 값도 변해 호환성이 깨져버려 InvalidClassException이 발생할 것이다.

* ##### 버그와 보안 구멍이 생길 위험이 높아진다.

  * 객체는 생성자를 사용해 만드는게 기본이다.
  * 직렬화는 언어의 기본 메커니즘을 우회하는 객체 생성 기법인 것이다.
  * 역직렬화는 일반 생성자의 문제가 그대로 적용되는 '숨은 생성자' 다.
  * 기본 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출된다.

* ##### 해당 클래스의 신버전을 릴리스 할 때 테스트할 것이 늘어난 다는 것이다.

  * 직렬화 가능 클래스가 수정되면 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화 할 수 있는지, 그리고 그 반대도 가능한지 검사해야 한다.
  * 양방향 직렬화/역직렬화가 모두 성공하고, 원래의 객체를 충실히 복제해내는지 반드시 확인해야 한다.
  * 클래스를 처음 제작할 때 커스텀 직렬화 형태를 잘 설계했다면 테스트 부담은 줄일 수 있다.

##### Serializable 구현 여부는 가볍게 결정할 사안이 아니다.

* 객체를 전송하거나 저장할 때 자바 직렬화를 이용하는 프레임워크용으로 만든 클래스라면 선택의 여지가 없다.
* Serializable을 반드시 구현해야 하는 다른 클래스의 컴포넌트로 쓰일 클래스도 마찬가지다.
* 역사적으로 BigInteger와 Instant 같은 값 클래스와 컬렉션 클래스들은 Serializable을 구현하고, 스레드 풀처럼 동작하는 객체를 표현하는 클래스들은 대부분 구현하지 않았다.

##### 상속용으로 설계된 클래스는 대부분 Serializable을 구현하면 안 되며, 인터페이스도 대부분 Serializable을 확장해서는 안 된다.

* 그런 클래스를 확장하거나 그런 인터페이스를 구현하는 이에게 커다란 부담을 지우게 된다.
* 예외로 Serializable을 구현한 클래스만 지원하는 프레임워크를 사용하는 상황이라면 다른 방도가 없을 것이다.

##### 작성하는 클래스의 인스턴스 필드가 직렬화와 확장이 모두 가능 할 때 주의할 점

* 인스턴스 필드 값 중 불변식을 보장해야 할 게 있다면 반드시 하위 클래스에서 finalize 메서드를 재정의하지 못하게 해야 한다. finalize 메서드를 자신이 재정의하면서 final로 선언한다.

* 인스턴스 필드 중 기본값으로 초기화되면 위배되는 불변식이 있다면 클래스에 다음의 readObjectNoData 메서드를 반드시 추가해야 한다.

  ##### 상태가 있고, 확장 가능하고, 직렬화 가능한 클래스용 readObjectNoData 메서드

  ~~~java
  private void readObjectNoData() throw InvalidObjectException {
      throw new InvalidObjectException("스트림 데이터가 필요합니다.")
  }
  ~~~

  * 기존의 직렬화 가능 클래스에 직렬화 가능 상위클래스를 추가하는 두문 경우를 위한 메서드

##### Serializable을 구현하지 않기로 할 때는 한 가지만 주의하자.

* 상속용 클래스인데 직렬화를 지원하지 않으면 그 하위 클래스에서 직렬화를 지원하려할 때 부담이 늘어난다.
* 보통 이런 클래스를 역직렬화하려면 그 상위 클래스는 매개변수 없는 생성자를 제공해야 하는데, 이런 생성자를 제공하지 않으려면 하위 클래스에서 어쩔 수 없이 직렬화 프록시 패턴을 사용해야 한다.

##### 내부 클래스는 직렬화를 구현하지 말아야 한다.

* 내부 클래스에는 바깥 인스턴스의 참조와 유효 범위 안의 지역변수 값들을 저장하기 위해 컴파일러가 생성한 필드들이 자동으로 추가된다.
* 다시 말해, 내부 클래스에 대한 기본 젹렬화 형태는 분명하지 않다.



##### 핵심 정리

> Serializable은 구현한다고 선언하기는 아주 쉽지만, 그것은 눈속임일 뿐이다. 한 클래스의 여러 버전이 상호작용할 일이 없고 서버가 신뢰할 수 없는 데이터에 노출될 가능성이 없는 등, 보호된 환경에서만 쓰일 클래스가 아니라면 Serializable 구현은 아주 신중하게 이뤄져야 한다. 상속할 수 있는 클래스라면 주의사항이 더욱 많아진다.



___



## 아이템 87: 커스텀 직렬화 형태를 고려해보라

클래스가 Serializable을 구현하고 기본 직렬화 형태를 사용한다면 현재 구현에 영원히 발이 묶이게 된다.

##### 먼저 고민해보고 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하라.

* 기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 사용해야 한다.
* 직접 설계하더라도 기본 직렬화 형태와 거의 같은 결과가 나올 경우에만 기본 형태를 써야 한다.
* 어떤 객체의 직렬화 형태는 객체가 포함된 데이터들과 그 객체에서부터 시작해 접근할 수 있는 모든 객체를 담아내며, 심지어 이 객체들이 연결된 위상(topology)까지 기술한다.
* 그러나 이상적인 직렬화 형태라면 물리적인 모습과 독립된 논리적인 모습만 표현해야 한다.

##### 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.

###### 기본 직렬화 형태에 적합한 후보 - 사람의 성명을 간략히 표현한 예제

~~~java
public class Name implements Serializable {
    /**
     * 성. null이 아니어야 함.
     * @serial
     */
    private final String lastName;
    
    /**
     * 이름. null이 아니어야 함.
     * @serial
     */
    private final String firstName;
    
    /**
     * 중간이름. 중간이름이 없다면 null.
     * @serial
     */
    private final String middleName;
    
    ... // 나머지 코드는 생략
}
~~~

* 성명은 이름, 성, 중간이름의 3개의 문자열로 구성되며, 인스턴스 필드는 이 논리적 구성요소를 정확히 반영했다.

* ##### 기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.

  ex) Name 클래스의 readObject 메서드가 lastName과 firstName이 null이 아님을 보장해야 한다.

###### 기본 직렬화 형태에 적합하지 않은 클래스 - 문자열 리스트를 표현하는 예제

~~~java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;
    
    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    
    ...//
}
~~~

* 논리적으로는 이 클래스는 일련의 문자열을 표현한다.
* 물리적으로는 문자열들을 이중 연결 리스트로 연결했다.
* 이 클래스에 기본 직렬화 형태를 사용하면 각 노드의 양방향 연결 정보를 포함해 모든 엔트리를 철두철미하게 기록한다.

##### 객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 크게 네 가지 면에서 문제가 생긴다.

1. ##### 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.

   앞의 예에서 StringList.Entry가 공개 API가 되어 버린다. 내부 표현 방식이 바뀌더라도 여전히 연결리스트로 표현된 입력도 처리할 수 있어야 하기 때문에 관련 코드를 절때 제거할 수 없다.

2. ##### 너무 많은 공간을 차지할 수 있다.

   앞의 예에서 직렬화 형태는 연결 리스트의 모든 엔트리 연결 정보까지 기록했다. 직렬화 형태가 너무 커져서 디스크에 저장하거나 네트워크로 전송하는 속도가 느려진다.

3. ##### 시간이 너무 많이 걸릴 수 있다.

   직렬화 로직은 객체 그래프의 위상에 관한 정보가 없으니 그래프를 직접 순회해볼 수밖에 없다.

4. ##### 스택 오버플로우를 일으킬 수 있다.

   기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 이 작업은 중간 정도의 크기 객체 그래프에서도 자칫 스택 오버플로우를 일으킬 수 있다.

##### StringList를 위한 합리적인 직렬화 형태는 무엇일까?

* 단순히 리스트가 포함한 문자열의 개수를 적은 다음, 그 뒤로 문자열들을 나열하는 수준이면 될 것이다.
* StringList의 물리적은 상세 표현은 배재한 채 논리적인 구성만 담는 것이다.

###### 합리적인 커스텀 직렬화 형태를 갖춘 StringList

~~~java
public final class StringList implements Serializable {
    private transient int size   = 0;
    private transient Entry head = null;

    // 이제는 직렬화되지 않는다.
    private static class Entry {
        String data;
        Entry  next;
        Entry  previous;
    }

    // 지정한 문자열을 이 리스트에 추가한다.
    public final void add(String s) {  }

    /**
     * 이 {@code StringList} 인스턴스를 직렬화한다.
     *
     * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
     * ({@code int}), 이어서 모든 원소를(각각은 {@code String})
     * 순서대로 기록한다.
     */
    private void writeObject(ObjectOutputStream s)
            throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);

        // 모든 원소를 올바른 순서로 기록한다.
        for (Entry e = head; e != null; e = e.next)
            s.writeObject(e.data);
    }

    private void readObject(ObjectInputStream s)
            throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();

        // 모든 원소를 읽어 이 리스트에 삽입한다.
        for (int i = 0; i < numElements; i++)
            add((String) s.readObject());
    }

    // 나머지 코드는 생략
}
~~~

* writeObject와 readObject가 직렬화 형태를 처리한다.
* transient는 해당 인스턴스 필드가 기본 직렬화 형태에 포함되지 않는 다는 표시이다.
* writeObject와 readObject는 각각 가장 먼저 defaultWriteObject와 defaultReadObject를 호출해야 한다.

##### transient로 선언해도 되는 인스턴스 필드는 모두 붙여야 한다.

* 캐시된 해시 값처럼 다른 필드에서 유도되는 필드도 여기 해당한다.

* JVM을 실행할 때마다 값이 달라지는 필드인 long 필드도 여기에 속한다.

* ##### 해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 생략해야 한다.

##### 기본 직렬화를 사용한다면 transient 필드들은 역직렬화될 때 기본값으로 초기화 됨을 잊지 말자.

* 기본값을 그대로 사용하면 안 된다면 readObject에서 defaultReadObject를 호출한 다음, 해당 필드를 원하는 값으로 복원하자.
* 혹은 그 값을 처음 사용할 때 초기화 하는 방법도 있다.

##### 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다.

* 메서드를 syncronized로 선언하여 스레드 안전하게 만든 객체에서 기본 직렬화를 사용한다면 writeObject도 syncronized를 선언해야 한다.

  ~~~java
  private syncronized void writeObject(ObjectOutputStream s) throws IOException {
      s.defaultWriteObject();
  }
  ~~~

##### 어떤 직렬화 형태를 택하든 직렬화 가능 클래스 클래스 모두에 직렬 버전 UID를 명시적으로 부여하자.

* 직렬 버전 UID가 일으키는 잠재적인 호환성 문제가 사라진다.
* 성능도 조금 빨라질 수 있다.
* 구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 직렬 버전 UID를 절대 수정하지 말자.



##### 핵심 정리

> 클래스를 직렬화하기로 했다면 어떤 직렬화 형태를 사용할지 심사숙고하기 바란다. 자바의 기본 직렬화 형태는 객체를 직렬화한 결과가 해당 객체의 논리적 표현에 부합할 때만 사용하고, 그렇지 않으면 객체를 적절히 설명하는 커스텀 직렬화 형태를 고안하라. 직렬화 형태도 공개 메서드를 설계할 때에 준하는 시간을 들여 설계해야 한다. 한번 공개된 메서드는 향후 릴리스에서 제거할 수 없듯이, 직렬화 형태에 포함된 필드도 마음대로 제거할 수 없다. 직렬화 호환성을 유지하기 위해 영원히 지원해야 하는 것이다. 잘못된 직렬화 형태를 선택하면 해당 클래스의 복잡성과 성능에 영구히 부정적인 영향을 남긴다.



___



## 아이템 88: readObject 메서드는 방어적으로 작성하라

##### 방어적 복사를 사용하는 불변 클래스

~~~java
public final class Period {
    private final Date start;
    private final Date end;
    
    public Period(Date start, Date end){
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if(this.start.comparTo(this.end) > 0) {
            throw new IllegalArgumentException(
                    start + "가 " + end "보다 늦다.");
        }
    }
  
    public Date start() { return new Date(start.getTime());}
    public Date end() { return new Date(end.getTime());}
    public String toString() {return start + " - " + end;}
}
~~~

* 이 클래스에 Serializable을 구현하는 것으로 직렬화를 할 수 있을 것 같다.
* 하지만 이렇게 해서는 이 클래스의 주요한 불변식은 더는 보장하지 못한다.
* 문제는 readObject 메서드가 실질적으로 또 다른 public 생성자 역할을 하기 때문이다.
* readObject에서도 보통의 생성자 처럼 메서드가 유효한지 검사해야 하고, 필요하다면 방어적 복사해야 한다.
* 이 작업을 하지 않는 다면 공격자는 손쉽게 해당 클래스의 불변식을 깨뜨릴 수 있다.

##### Period readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야 한다.

* 유효성 검사에 실패하면 InvalidObjectException을 던진다.

  ##### 유효성 검사를 수행하는 readObject 메서드 - 아직 부족하다.

  ~~~java
  private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
      s.defaultReadObject();
      
      if(start.compareTo(end) > 0) {
          throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
      }
  }
  ~~~

  * 이 코드는 공격자가 허용되지 않는 Period 인스턴스를 생성하는 것은 막을 수 있지만, 미묘한 문제가 있다.

* 정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어 낼 수 있다.

##### 객체를 역직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 방어적으로 복사해야한다.

* readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다.

  ##### 방어적 복사와 유효성 검사를 수행하는 readObject 메서드

  ~~~java
  private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
      s.defaultReadObject();
      
      start = new Date(start.getTime());
      end = new Date(end.getTime());
      
      if(start.compareTo(end) > 0) {
          throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
      }
  }
  ~~~

  * final 필드는 방어적 복사가 불가능 하므로 readObject에서 사용하기 위해 필드의 final 을 제거해야 한다.

##### 기본 readObject 메서드를 써도 좋을지 판단하는 방법

* transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 좋은가?

  답이 "아니오" 라면 커스텀 readObject를 만들고 유효성 검사와 방어적 복사를 해야 한다.
  혹은 직렬화 프록시 패턴을 사용하는 방법도 있다.



##### 핵심 정리

> readObject 메서드를 작성할 때는 언제나 public 생성자를 작성하는 자세로 임해야 한다. readObject는 어떤 바이트 스트림이 넘어오더라도 유효한 인스턴스를 만들어내야 한다. 바이트 스트림이 진짜 직렬화된 인스턴스라고 가정해서는 안 된다. 이번 아이템에서는 기본 직렬화 형태를 사용한 클래스를 예로 들었지만 커스텀 직렬화를 사용하더라도 모든 문제가 그대로 발생할 수 있다. 이어서 안전한 readObject 메서드를 작성하는 지침을 요약해 보았다.
>
> * private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라. 불변 클래스 내의 가변 요소가 여기에 속한다.
> * 모든 불변식을 검사하여 어긋나는 게 발견되면 InvalidObjectException을 던진다. 방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 한다.
> * 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하라.
> * 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자.



___



## 아이템 89: 인스턴스 수를 통제해야 한다면 readResolve 보다는 열거 타입을 사용하라

##### 아이템 3의 싱글턴 패턴 클래스

~~~java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    
    public void leaveTheBuilding() { ... }
}
~~~

* 이 클래스는 Serializable을 구현하면 더 이상 싱글턴이 아니게 된다.

* 기본 직렬화를 쓰지 않았고, 명시적인 readObject를 제공하더라도 초기화할 때 만들어진 인스턴스와 별개인 인스턴스를 반환한다.

* Serializable을 구현하고 readResolve 메서드를 추가하면 싱글턴 속성을 유지할 수 있다.

  ~~~java
  private Object readResolve() {
      return INSTANCE;
  }
  ~~~

  * 역직렬화된 객체는 무시하고 초기에 만들어진 인스턴스를 반환한다.

  * Elvis의 직렬화 데이터는 아무런 실 데이터를 가질 이유가 없으니 모든 필드를 transient로 선언해야 한다.

  * ##### readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다.

  * 그렇지 않으면 잘 조작된 스트림을 써서 해당 참조 필드의 내용이 역직렬화되는 시점에 그 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다.

##### 싱글턴 클래스를 원소 하나짜리 열거 타입으로 바꾸는 편이 더 나은 선택이다.

~~~java
public enun Elvis {
    INSTANE;
    ...
}
~~~

* 직렬화 가능한 인스턴스 통제 클래스를 열거 타입을 이용해 구현하면 선언한 상수 외에 다른 객체는 존재하지 않음을 자바가 보장해준다.
* 전통적인 싱글턴보다 우수하다.

##### readResolve 사용하는 방식이 쓸모없는 것은 아니다.

* 컴파일타임에 어떤 인스턴스들이 있는지 알 수 없는 상황이라면 열거 타입으로 표현하는 것이 불가능하다.

##### readResolve의 접근성은 매우 중요하다.

* final 클래스라면 private 이어야 한다.

  private로 선언하면 하위 클래스에서 사용할 수 없다.

* package-private로 선언하면 같은 패키지에 속한 하위 클래스에서만 사용할 수 있다.

* protected나 public으로 선언하면 이를 재정의하지 않은 모든 하위 클래스에서 사용할 수 있다.

  하위 클래스의 인스턴스를 역직렬화하면 상위 클래스의 인스턴스를 생성하여 ClassCastException을 일으킬 수 있다.



##### 핵심 정리

> 불변식을 지키기 위해 인스턴스를 통제해야 한다면 가능한 열거 타입을 사용하자. 여의치 않은 상황에서 직렬화와 인스턴스 통제가 모두 필요하다면 readResolve 메서드를 작성해 넣어야 하고, 그 클래스에서 모든 참조 타입 인스턴스 필드를 transient로 선언해야 한다.



___



##  아이템 90: 직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라

Serializable을 구현하기로 결정한 순간 언어 정상 메커니즘인 생성자 이외의 방법으로 인스턴스를 생성할 수 있게 된다.

버그와 보안 문제가 일어날 가능성이 커진다는 것을 의미하는데, 이 위험을 크게 줄여줄 기법이 하나 있다. 

##### 직렬화 프록시 패턴(serialization proxy pattern)

* 바깥 클래스의 논리적 상태를 정밀하게 표현하는 중첩 클래스를 설계해 private으로 선언한다.

  이 중첩 클래스가 바로 바깥 클래스의 직렬화 프록시다.

* 중첩 클래스의 생성자는 단 하나여야 하며, 바깥 클래스를 매개변수로 받아야 한다.

* 이 생성자는 단순히 인수로 넘어온 인스턴스의 데이터를 복사한다.

* 일관성 검사나 방어적 복사도 필요 없으며, 바깥 클래스, 직렬호 프록시 모두 Serializable을 구현한다고 선언하면 된다.

##### 직렬화한 period 클래스의 직렬화 프록시 예제

~~~java
// 방어적 복사를 사용하는 불변 클래스
public final class Period implements Serializable {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각; 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발행한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end   = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(
                    start + "가 " + end + "보다 늦다.");
    }

    public Date start () { return new Date(start.getTime()); }

    public Date end () { return new Date(end.getTime()); }

    public String toString() { return start + " - " + end; }


    // Period 클래스용 직렬화 프록시
    private static class SerializationProxy implements Serializable {
        private final Date start;
        private final Date end;

        SerializationProxy(Period p) {
            this.start = p.start;
            this.end = p.end;
        }

        private static final long serialVersionUID =
                234098243823485285L; // 아무 값이나 상관없다.
      
      private Object readResolve() {
          return new Period(start, end);
      }
    }

    // 직렬화 프록시 패턴용 writeReplace 메서드
    private Object writeReplace() {
        return new SerializationProxy(this);
    }

    // 직렬화 프록시 패턴용 readObject 메서드
    private void readObject(ObjectInputStream stream)
            throws InvalidObjectException {
        throw new InvalidObjectException("프록시가 필요합니다.");
    }
}
~~~

* 직렬화 프록시 클래스는 Period와 완전히 같은 필드로 구성되었다.
* 바깥 클래스에는 writeReplace 메서드를 추가한다.
  * 이 메서드는 자바의 직렬화 시스템이 바깥 클래스 대신 SerializationProxy의 인스턴스를 반환하게 하는 역할을 한다.
  * 직렬화가 이뤄지기 전에 바깥 클래스의 인스턴스를 직렬화 프록시로 변환해준다.ㄴ
* readObject를 추가하면 불변식을 훼손하는 공격을 막을 수 있다.
* 바깥 클래스와 논리적으로 동일한 인스턴스를 반환하는 readResolve 메서드를 직렬화 프록시 클래스에 추가한다.
  * 이 메서드는 역직렬화 시에 직렬화 시스템이 직렬화 프록시를 다시 바깥 클래스의 인스턴스로 변환하게 해준다.
* 방어적 복사처럼, 직렬화 프록시 패턴은 가짜 바이트 스트림 공격과 내부 필드 탈취 공격을 프록시 수준에서 차단해준다.
* 직렬화 프록시는 Period의 필드를 final로 선언해도 되므로 진정한 불변으로 만들 수 있다.

##### 직렬화 프록시 패턴이 readObject에서의 방어적 복사보다 강력한 경우가 하나 더 있다.

* 직렬화 프록시 패턴은 역직렬화한 인스턴스와 원래의 직렬화된 인스턴스의 클래스가 달라도 정상 동작한다.

##### 직렬화 프록시 패턴의 한계

1. 클라이언트가 멋대로 확장할 수 있는 클래스에는 적용할 수 없다.
2. 객체 그래프에 순환이 있는 클래스에도 적용할 수 없다.
3. 직렬화 프록시 패턴이 주는 강력함과 안정성에도 대가가 따르는데, 방어적 복사보다 느리다.



##### 핵심 정리

> 제3자가 확장할 수 없는 클래스라면 가능한 한 직렬화 프록시 패턴을 사용하자. 이 패턴이 아마도 중요한 불변식을 안정적으로 직렬화해주는 가장 쉬운 방법일 것이다.



___

12장 직렬화 끝...

### 😎이펙티브 자바를 마치며...

이펙티브 자바를 완독하며 정리하였다. 주니어 개발자로서 전부 이해가 된 것은 아니지만(사실 많이 어려운 편이었다 ㅠㅠ) 이런 상황도 있구나하는 정도로 일단 이해했다. 나중에 이펙티브 자바에서 본 내용들을 직접 다루게 된다면 도움이 될 것이다. 이후에는 책을 학습하며 자바 실력이 아직 많이 부족하다고 느꼈기에... 기본에 충실하여 자바 공부좀 해야겠다.

다음은 모던 자바 인 액션, 스프링 퀵 스타트, 스프링 부트, 테스트 코드 등 볼 책도 많이 있다...

