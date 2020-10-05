---
title: effective java 3/E - 5장. 제네릭
tags: 
 - Java
key: 10
---

# 5장 클래스와 인터페이스

##### 5장에서 배울 내용

* 제네릭은 자바5 부터 사용할 수 있다.
* 제네릭의 이점을 최대로 살리고 단점을 최소화 하는 방법을 이야기한다.



___



## 아이템 26: 로 타입은 사용하지 말라

##### 용어 정리

클래스와 인터페이스 선언에 타입 매개변수(type-prameter)가 쓰이면, 이를 **제네릭 클래스** 혹은 **제네릭 인터페이스**라고 한다.

List 인터페이스는 매개변수 E를 받는다. 인터페이스의 완전한 이름은 List\<E\> 지만, 짧게 List로 쓰며 제네릭 클래스와 인터페이스를 통틀어 **제네릭 타입**(generic type)이라 한다.



##### 매개변수화 타입(parameterized type)

클래스 또는 인터페이스 이름이 나오고, 꺽쇠(<,>) 괄호 안에 실제 타입 매개변수들을 나열한다. 

List\<String\>은 원소의 타입이 String인 리스트를 뜻하는 매개변수화 타입이다. String이 정규 타입 매개변수 E에 해당하는 실제 타입이다.



##### 로 타입(raw type)

로 타입이란 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.

List\<E\>의 로 타입은 List 이다.

###### <span style="color:red;">제네릭을 지원하기전 컬렉션 선언 - 따라하지 말 것</span>

~~~java
private final Collection stamps = ...;
~~~

이 코드를 사용하면 도장(stamp) 대신 동전(Coin)을 넣어도 아무 오류 없이 컴파일되고 실행된다. (경고 메시지는 보여줄 것이다.)

~~~java
// 실수로 코인을 넣는다.
stamps.add(new Coin(...));
~~~

컬렉션에서 이 코인을 다시 꺼내기 전까지는 오류를 알아채지 못한다.

~~~java
for(Iterator i = stamps.iterator(); i.hasNext(); ) {
	Stamp stamp = (Stamp) i.next(); // ClassCastException을 던진다.
	stamp.cancel();
}
~~~

스탬프 컬렉션에서 실수로 넣은 코인을 사용할 때 오류가 발생한다. 

하지만 오류는 가능한 한 발생 즉시, 이상적으로는 컴파일할 때 발견하는 것이 가장 좋다. 이 예제에서는 오류가 발생하고 한참 뒤인 런타임에야 알아챌 수 있는데, 이렇게 되면 런타임에 문제를 겪는 코드와 원인을 제공한 코드가 물리적으로 상당히 떨어져 있을 가능성이 커진다. ClassCastException이 발생하면 stamps에 Coin을 넣은 지점을 찾기 위해 코드 전체를 훑어봐야 할 수도 있다.

###### <span style="color:blue;">매개변수화된 컬렉션 타입 - 타입 안정성 확보!</span>

~~~java
private final Collection<Stamp> stamps = ...;
~~~

컴파일러는 stamps에  Stamp의 인스턴스만 넣어야 함을 인지하게 된다. 아무런 경고 없이 컴파일 된다면 의도대로 동작할 것임을 보장한다.

##### 로 타입을 쓰면 제네릭이 안겨주는 안정성과 표현력을 모두 잃게 된다.

로 타입을 만든 이유는 호환성 때문이다. 제네릭 없이 짜여진 코드들이 많아 이를 새로운 코드와 맞물려 돌아가기 위해 로 타입이 필요 했다.

##### List\<Object\> 같은 매개변수화 타입을 사용할 때와 달리 List 같은 로 타입을 사용하면 안정성을 잃게 된다.

List 같은 로 타입은 사용해서는 안되나 List\<Object\> 처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다. 

List는 제네릭 타입에서 완전히 발을 뺀 것이고, List\<Object\>는 모든 타입을 허용한다는 의사를 컴파일러에 명확히 전달한다.

매개변수로 List를 받는 메서드에는 List\<String\>을 넘길 수 있지만, List\<Object\>를 받는 메서드에는 넘길 수 없다.

###### <span style="color:red;">런타임 실패 - unsafeAdd 메서드 로 타입 사용 예제</span>

~~~java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다.
}

private static void unsafeAdd(List list, Object o) {
    list.add(o);
}
~~~

이 코드는 컴파일은 되지만 로 타입을 사용하였기에 경고가 발생한다.

이 프로그램을 실행하면 strings.get(0) 의 결과를 형변환하려 할 때 ClassCastException을 던진다.

###### <span style="color:blue;">컴파일타임 실패 - unsafeAdd 메서드를 List\<Object\>로 변경</span>

~~~java
private static void unsafeAdd(List<Object> list, Object o) {
    list.add(o);
}
~~~

이 경우에는 컴파일 조차 되지 않는다.

##### 로 타입 대신 비한정적 와일드카드 타입(unbounded wildcard type)을 대신 사용하는게 좋다.

제네릭 타입을 쓰고 싶지만 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않다면 물음표(?)를 사용하자.

Set\<E\>의 비한정적 와일드카드 타입은 Set\<?\> 이다.

와일드카드 타입은 안전하고, 로 타입은 안전하지 않다. 로 타입 컬렉션에는 아무 원소나 넣을 수 있어 불변식을 훼손하기 쉽다. 

반면, Collection\<?\>에는 (null 외에) 어떤 원소도 넣을 수 없다.

##### 로 타입을 쓰지 말라는 규칙의 예외

1. class 리터럴에는 로 타입을 써야한다.
   List.class, String[].class, int.class는 허용하고 List\<String\>.class, List\<?\>.class는 허용하지 않는다.

2. instanceof 연산자를 사용할 때 로 타입을 쓰는 편이 깔끔하다.

   ~~~java
   if(o instanceof Set){ // 로타입
   	Set<?> s = (Set<?>) o; // 와일드카드 타입
   }
   ~~~

   o의 타입이 Set임을 확인하고 와일드카드 타입인 Set\<?\>로 형변환해야 한다.



##### 핵심 정리

> 로 타입을 사용하면 런타임에 예외가 일어날 수 있으나 사용하면 안된다. 로 타입은 제네릭이 도입되기 이전 코드와의 호환성을 위해 제공될 뿐이다. 빠르게 훑어보자면, Set\<Object\>는 어떤 타입의 객체도 저장할 수 있는 매개변수화 타입이고, Set\<?\>는 모종의 타입 객체만 저장할 수 있는 와일드카드 타입이다. 그리고 이들의 로 타입인 Set은 제네릭 타입 시스템에 속하지 않는다. Set\<Object\>와 Set\<?\>는 안전하지만, 로 타입인 Set은 안전하지 않다.



___



## 아이템 27: 비검사 경고를 제거하라

제네릭을 사용하기 시작하면 수많은 컴파일러 경고를 보게 될 것이다. 비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등이다.

대부분의 비검사 경고는 쉽게 제거할 수 있다.

~~~java
Set<Lark> exaltation = new HashSet();
~~~

이 코드를 실행하면 컴파일러는 무엇이 잘못되었는지 알려준다. 매개변수를 명시하지 않아 발생한 경고이다.

다음과 같이 수정한다.

~~~java
Set<Lark> exaltation = new HashSet<>();
~~~

다이아몬드 연산자(<>)만을 추가하여 해결할 수 있다. 컴파일러는 올바른 실제 타입 매개변수를 추론해준다.

##### 할 수 있는한 비검사 경고를 제거하라

비검사 경고를 모두 제거한다면 그 코드는 타입 안정성이 보장된다.

* **경고를 제거할 수 없지만 타입 안전하다고 확실할 수 있다면 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨기자.**
  단, 타입 안전함을 검증하지 않은 채 경고를 숨기면 스스로에게 잘못된 보안 인식을 심어주는 꼴이다.
  애너테이션이 달린 코드는 경고 없이 컴파일이 되겠지만, 런타임에는 여전히 ClassCastException을 던질 수 있다.
* @SuppressWarnings는 개별 지역변수 선언부터 클래스 전체까지 달 수 있다.
  **하지만 @SuppressWarnings 애너테이션은 항상 가능한 한 좁은 범위에 적용하자.**
  보통은 변수 선언, 아주 짧은 메서드 혹은 생성자가 될 것이다.
* **@SuppressWarnings("unchecked") 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.**
  다른 사람이 코드를이해 하는데 도움이 되며, 더 중요하게는, 다른 사람이 그 코드를 잘못 수정하여 타입 안정성을 잃는 상황을 줄여준다.



##### 핵심 정리

>비검사 경고는 중요하니 무시하지 말자. 모든 비검사 경고는 런타임에 ClassCastException을 일으킬 수 있는 잠재적 가능성을 뜻하니 최선을 다해 제거하라. 경고를 없앨방법을 찾지 못하겠다면, 그 코드가 타입 안전함을 증명하고 가능한 한 범위를 좁혀 @SuppressWarnings("unchecked") 애너테이션으로 경고를 숨겨라. 그런 다음 경고를 숨기기로 한 근거를 주석으로 남겨라.



___



## 아이템 28: 배열보다는 리스트를 사용하라

배열과 제네릭 타입에는 중요한 차이가 두 가지 있다.

##### 첫 번째, 제네릭과 배열

* 배열은 공변이다. 
  Sub가 Super의 하위 타입이라면 배열 Sub[]는 배열 Super[]의 하위 타입이 된다.

* 제네릭은 불공변이다.
  서로 다른 타입 Type1과 Type2가 있을 때, List\<Type1\> 은 List\<Type2\> 의 하위 타입도 아니고 상위 타입도 아니다.

##### 배열의 문제 - 런타임에 실패한다.

~~~java
Object[] objectArray = new Long[1];
objectArray[0] = "스트링 타입"; // ArrayStoreException을 던진다.
~~~

이 코드는 런타임에 실패를 발견할 수 있다.

##### 제네릭 - 컴파일이 되지 않는다.

~~~java
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이다.
ol.add("스트링 타입");
~~~

위 두 코드는 Long용 저장소에 String을 넣을 수는 없다. 다만 배열은 그 실수를 런타임에 알게 되지만, 리스트를 사용하면 컴파일할 때 바로 알 수 있어 대처를 빠르게 할 수 있다.



##### 두 번째, 배열은 실체화(reify)된다.

배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다. 그래서 위의 배열 예제에서 ArrayStoreException이 발생한다. 

반면, 제네릭은 타입 정보가 런타임에는 소거(erasure)된다. 원소 타입을 컴파일타임에만 검사하며 런타임에는 알 수조차 없다는 뜻이다.

배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수를 사용할 수 없다. 즉, 코드를 new List\<E\>[], new List\<String\>[], new E[] 식으로 작성하면 컴파일할 때 제네릭 배열 생성 오류를 일으킨다.

##### 제네릭 배열을 만들지 못하기 한 이유

타입 안전하지 않기 때문이다. 이를 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임 ClassCastException을 발생할 수 있다. 런타임에 ClassCastException이 발생하는 일을 막아주겠다는 제네릭 타입 시스템의 취지에 어긋나는 것이다.

###### 예제 

~~~java
List<String>[] stringLists = new ArrayList<String>[1];    //(1)
List<Integer> intList = List.of(42);                      //(2)
Object[] objects = stringLists;                           //(3)
objects[0] = intList;                                     //(4)
String s = stringLists[0].get(0);                         //(5)
~~~

1. 제네릭 배열을 생성하는 (1)이 허용된다고 가정하자.
2. (2)는 원소가 하나인 List\<Integer\> 를 생성한다.
3. (3)은 (1)에서 생성한 List\<String\> 의 배열을 Object 배열에 할당한다. 배열은 공변이니 아무 문제 없다.
4. (4)는 (2)에서 생성한 List\<Integer\> 의 인스턴스를 Object 배열의 첫 원소로 저장한다. 제네릭은 소거 방식으로 구현되어서 이 역시 성공한다.
5. 런타임에는 List\<Integer\> 인스턴스의 타입은 단순히 List가 되고, List\<Integer\>[] 인스턴스의 타입은 List[]가 된다.
6. 따라서 (4)에서도 ArrayStoreException을 일으키지 않는다.
7. 여기부터 문제다. List\<String\> 인스턴스만 담겠다고 선언한 stringLists 배열에는 List\<Integer\> 인스턴스가 저장되어 있다.
8. (5)는 이 배열의 처음 리스트에서 첫 원소를 꺼내려한다. 컴파일러는 꺼낸 원소를 자동으로 String으로 형변환 하는데, 이 원소는 Integer이므로 런타임에  ClassCastException이 발생한다.
9. 이런 일을 방지하려면 결국 제네릭 배열이 생성되지 않도록 (1)에서 컴파일 오류를 내야한다.



##### E, List\<E\>, List\<String\> 같은 타입을 실체화 불가 타입(non-reifiable type)이라 한다. 

실체화 되지 않아서 런타임에는 컴파일타임보다 타입 정보를 적게 가지는 타입이다. 소거 매커니즘 때문에 매개변수화 타입 가운데 실체화될 수 있는 타입은 List\<?\> 와 Map\<?,?\> 같은 비한정적 와일드카드 타입 뿐이다. 배열을 비한정적 와일드카드 타입으로 만들 수는 있지만, 유용하게 쓰일 일은 거의 없다.



##### 배열을 제네릭으로 만들 수 없어 귀찮을 수 있는 점

제네릭 컬렉션에서는 자신의 원소 타입을 담은 배열을 반환하는 게 보통은 불가능하다. (아이템 33)

제네릭 타입과 가변인수 메서드를 함께 쓰면 해석하기 어려운 경고 메세지를 받게 된다. 가변인수 메서드를 호출할 때마다 가변인수 매개변수를 담은 배열이 하나 만들어지는데, 이때 그 배열의 원소가 실체화 불가 타입이라면 경고가 발생하는 것이다. 

이 문제는 @SafeVarargs 애너테이션으로 대체할 수 있다. (아이템 32)

배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우 대부분은 배열인 E[] 대신 컬렉션 List\<E\>를 사용하면 해결된다. 코드가 조금 복잡해지고 성능이 나빠질 수 있지만, 타입 안정성과 상호운용성은 좋아진다.

##### 생성자에서 컬렉션을 받는 Chooser 클래스 예제

컬렉션 안의 원소 중 하나를 무작위로 선택해 반환하는 choose 메서드를 제공한다.

~~~java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        choiceList = new choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
    		return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
~~~

이 클래스를 사용하려면 choose 메서드를 호출할 때마다 반환된 Object를 원하는 타입으로 형변환해야 한다. 혹시나 타입이 다른 원소가 들어 있었다면 런타임에 형변환 오류가 날 것이다.

###### 제네릭으로 변경 - 타입 안정성 확보

~~~java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
~~~

코드 양이 늘고 조금 느릴 수 있지만, 런타임에 ClassCastException을 만난일이 없다.



##### 핵심 정리

> 배열과 제네릭에는 매우 다른 타입 규칙이 적용된다. 배열은 공변이고 실체화되는 반면, 제네릭은 불공변이고 타입 정보가 소거된다. 그 결과 배열은 런타임에는 타입 안전하지만 컴파일타임에는 그렇지 않다. 제네릭은 반대다. 그래서 둘을 섞어 쓰기란 쉽지 않다. 둘을 섞어 쓰다가 컴파일 오류나 경고를 만나면, 가장 먼저 배열을 리스트로 대체하는 방법을 적용해보자.



___



## 아이템 29: 이왕이면 제네릭 타입으로 만들라

JDK가 제공하는 제네릭 타입과 메서드를 사용하는 일은 일반적으로 쉬운 편이지만, 제네릭 타입을 새로 만드는 일은 조금 더 어렵다. 그래도 배워두면 그만한 값어치는 충분히 한다.

##### Object 기반 스택 - 제네릭이 절실한 강력 후보!

~~~java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
    	elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
    	ensureCapacity();
        elements[size ++] = e;
    }
    
    public Object pop() {
    	if (size == 0)
        	throw new EmptyStackException();
        return elements[--size];
    }
    
    /**
    * 원소를 위한 공간을 적어도 하나 이상 확보한다.
    * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
    */
    private void ensureCapacity() {
    	if(elements.length  == size)
        	elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
~~~

이 스택 코드는 제네릭 타입이어야 마땅하다. 지금 상태에서 클라이언트가 스택을 사용할 때 꺼낸 객체를 형변환해야 하는데, 이때 런타임 오류가 날 위험이 있다.

##### 일반 클래스를 제네릭 클래스로 만드는 첫 단계는 클래스 선언에 타입 매개변수를 추가하는 일이다.

타입 이름으로 보통 E를 사용하고 코드에 쓰인 Object를 적절한 타입 매개변수로 바꾼다.

~~~java
// 제네릭 스택으로 가는 첫 단계 - 컴파일 되지 않는다.
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
    	elements = new E[DEFAULT_INITIAL_CAPACITY]; // 오류 발생
    }
    
    public void push(E e) {
    	ensureCapacity();
        elements[size ++] = e;
    }
    
    public E pop() {
    	if (size == 0)
        	throw new EmptyStackException();
        return elements[--size];
    }
    ...
}
~~~

생성자에서 E와 같은 타입 실체화 불가 타입으로는 배열을 만들수 없어 오류가 발생한다.

###### 해결책 - 두 가지

1. 제네릭 배열 생성을 금지하는 제약을 대놓고 우회하는 방법이다.
   Object 배열을 생성한 다음 제네릭 배열로 형변환해보자. -> 오류 대신 경고를 내보낼 것이다.
   그렇지만 일반적으로 타입 안전하지 않다.
   컴파일러는 이 프로그램이 타입 안전한지 증명할 방법이 없지만 우리는 할 수 있다. 비검사 형변환이 프로그램의 타입 안정성을 해치지 않음을 스스로 확인해야 한다. 
   비검사 형변환이 안전함을 직접 증명했다면 범위를 최소로 좁혀 @SuppressWarnings 애너테이션으로 경고를 숨긴다.

   ~~~java
   // 배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
   // 따라서 타입 안정성은 보장하지만, 이 배열의 런타임 타입은 E[]가 아닌 Object[]다!
   @SuppressWarnings("unchecked")
   public Stack() {
       elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
   }
   ~~~

2. elements 필드의 타입을 E[]에서 Object[]로 바꾸는 것이다. 이렇게 하면 첫 번째 방법과는 다른 오류가 발생한다.
   배열이 반환한 원소를 E로 형변환하면 오류대신 경고가 뜬다.
   E는 실체화 불가 타입이므로 컴파일러는 런타임에 이뤄지는 형변환이 안전한지 증명할 방법이 없다.
   직접 증명을 하고 경고를 숨기는 방법이 있다.

   ~~~java
   public E pop() {
       if (size == 0)
           throw new EmptyStackException();
       
       // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
       @SuppressWarnings("unchecked") E result = (E) elements[--size];
       
       elements[size] = null; // 다 쓴 참조 해제
       return result;
   }
   ~~~

첫 번째 방법은 가독성이 더 좋다. 배열의 타입을 E[]로 선언하여 오직 E 타입 인스턴스만 받음을 확실히 어필한다. 코드도 더 짧고 보통의 제네릭 클래스라면 이곳저곳에서 이 배열을 자주 사용할 것이다.

첫 번째 방법에서는 형변환을 배열 생성시 단 한번만 해주면 되지만, 두 번째 방식에서는 배열에서 원소를 읽을 때마다 해줘야 한다.

지금까지 설명한 Stack 예는 "배열보다 리스트를 우선하라" 는 아이템 28과 모순돼 보인다. 사실 제네릭 타입 안에서 리스트를 사용하는 게 항상 가능하지도, 꼭 더 좋은 것도 아니다. 자바가 리스트를 기본 타입으로 제공하지 않으므로 ArrayList 같은 제네릭 타입도 결국은 기본 타입인 배열을 사용해 구현해야 한다. HashMap 같은 제네릭 타입은 성능을 높일 목적으로 배열을 사용하기도 한다.

Stack의 예처럼 대다수의 제네릭 타입은 타입 매개변수에 아무런 제약을 두지 않는다. 어떤 타입의 참조 타입으로도 Stack을 만들 수 있다. 단, 기본 타입은 사용할 수 없다.

ex) Stack\<Object\>, Stack\<int[]\>, Stack\<List\<String\>\>, Stack 등은 가능
      Stack\<int\>, Stack\<double\> 등 기본 타입은 불가능

##### 핵심 정리

> 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다. 그러니 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하라. 그렇게 하려면 제네릭 타입으로 만들어야 할 경우가 많다. 기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하자. 기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자를 훨씬 편하게 해주는 길이다.



___



## 아이템 30: 이왕이면 제네릭 메서드로 만들라

클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다. 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다. Collection의 '알고리즘' 메서드 (binarySearch, sort 등)는 모두 제네릭이다.

##### 제네릭 메서드 작성법

로 타입 사용 예제

~~~java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
~~~

이 코드는 컴파일은 되지만 경고가 두 개 발생한다. 경고를 없애려면 이 메서드를 타입 안전하게 만들어야 한다.

메서드 선언에서의 세 집합(입력 2개, 반환 1개)의 원소 타입을 타입 매개변수로 명시하고 메서드 안에서도 이 타입 매개변수만 사용하게 수정하면 된다.

##### 타입 매개변수 목록은 메서드의 제한자와 반환 타입 사이에 온다

~~~java
// 제네릭 메서드
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
~~~

단순한 제네릭 메서드라면 이 정도면 충분하다. 경고 없이 컴파일되고, 타입 안전하고, 쓰기도 쉽다.

메서드는 입력, 반환 타입이 모두 같아야 한다.



##### 제네릭 싱글턴 팩터리

불변 객체를 여러 타입으로 활용할 수 있도록 만들어야 할 때가 있다. 제네릭은 런타임에 타입 정보가 소거되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다. 하지만 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다. 이것이 제네릭 싱글턴 팩터리이다.

ex) Collections.reverseOrder, Collections.emptySet



##### 제네릭 싱글턴 팩터리 패턴 - 항등 함수

~~~java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

@SuppressWarnings("unchecked")
public static <T> UnaryOperator<T> identityFunction() {
    return (UnaryOperator<T>) IDENTITY_FN;
}
~~~

IDENTY_FN을 UnaryOperator\<T\>로 형변환하면 비검사 형변환 경고가 발생한다. T가 어떤 타입이든 UnaryOperator\<Object\> 는 Unaray\<T\> 가 아니기 때문이다.

하지만, 항등함수란 입력 값이 수정 없이 그대로 반환하는 특별한 함수이므로, T가 어떤 타입이든 UnaryOperator\<T\> 를 사용해도 타입 안전하다. 그러므로 @SuppressWarnings 추가하여 오류나 경고를 숨긴다.



##### 재귀적 타입 한정(recursive type bound)

자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다. 재귀적 타입 한정은 주로 타입의 자연적 순서를 정하는 Comparable 인터페이스와 함께 쓰인다. 

###### 예제

~~~java
public interface Comparable<T> {
	int compareTo(T o);
}
~~~

타입 매개변수 T는 Comparable\<T\> 를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다. 실제로 거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있다. 

String은 Comparable\<String\> 을 구현하고, Integer는 Comparable\<Integer\> 를 구현한다.

Comparable을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그 원소들을 정렬 혹은 검색하거나, 최솟값, 최댓값을 구하는 식으로 사용한다.

##### 재귀적 타입 한정을 이용해 상호 비교함을 표현

~~~java
public static <E extends Comparable<E>> E max(Collection<E> c);
~~~

타입 한정인 \<E extends Comparable\<E\>\> 는 "모든 타입 E는 자신과 비교할 수 있다"라고 읽을 수 있다.

상호 비교가 가능하드는 뜻을 아주 정확하게 표현하였다.

##### 메서드 구현 - 재귀적 타입 한정을 사용한 최댓값 반환

~~~java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어 있습니다.");

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return result;
}
~~~

> 이 메서드에 빈 컬렉션을 건네면 IllegalArgumentException을 던지니, Optional\<E\> 를 반환하도록 고치는 편이 나을 것이다.



##### 핵심 정리

>제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다. 타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우 그렇게 하려면 제네릭 메서드가 되어야 한다. 역시 타입과 마찬가지로, 형변환을 해줘야 하는 기존 메서드는 제네릭하게 만들자. 기존 클라이언트는 그대로 둔 채 새로운 사용자의 삶을 훨씬 편하게 만들어 줄 것이다.



___



## 아이템 31: 한정적 와일드카드를 사용해 API 유연성을 높이라

매개변수화 타입은 불공변이다. 즉, 서로 다른 타입 Type1과 Type2가 있을 때 List\<Type1\> 은 List\<Type2\> 의 하위 타입도 상위 타입도 아니다.

##### List\<String\> 은 List\<Object\> 의 하위 타입이 아니다. 

* List\<Object\> 에는 어떤 객체든 넣을 수 있지만 List\<String\> 에는 문자열만 넣을 수 있다.

* List\<String\> 은 List\<Object\> 가 하는 일을 제대로 수행하지 못하니 하위 타입이 될 수 없다.

* 리스코프 치환 원칙에 어긋난다.

  ##### 리스코프 치환 원칙

  > 컴퓨터 프로그램에서 자료형 S가 자료형 T의 하위형이라면 필요한 프로그램의 속성(정확성, 수행하는 업무 등)의 변경 없이 자료형 T의 객체를 자료형 S의 객체로 교체(치환)할 수 있어야 한다는 원칙이다.



##### 스택의 예제 - Stack의 public API

~~~java
public class Stack<E> {
	public Stack();
	public void push(E e);
	public E pop();
	public boolean isEmpty();
}
~~~

여기에 일련의 원소를 스택에 넣는 메서드를 추가해 보자.

##### <span style="color:tomato">pushAll 메서드</span>

~~~java
public void pushAll(Iterable<E> src){
	for(E e : src)
		push(e);
}
~~~

컴파일은 깨끗히 되지만 완벽하진 않다. Iterable의 원소 타입이 스택의 원소 타입과 일치하면 잘 동작한다.

하지만 Stack\<Number\> 로 선언한 후 Integer 타입의 intVal로 pushAll(intVal)을 호출하면 오류가 발생한다. 

Integer는 Number의 하위 타입으로 잘 동작할 것 같지만 매개변수화 타입이 불공변이기 때문이다. 

##### 해결책 - E 생산자(producer) 매개변수에 한정적 와일드카드 타입 사용

pushAll의 입력 매개변수 타입은 'E의 Iterable' 이 아니라 'E의 하위 타입의 Iterable' 이어야 한다.
와일드카드 타입 Iterable\<? extends E\> 은 'E의 하위 타입의 Iterable' 이라는 뜻이다.

~~~java
public void pushAll(Iterable<? extends E> src){
		for(E e : src)
				push(e);
}
~~~

Stack과 클라이언트 모두 깔끔히 컴파일이 될 것이고 모든 것이 타입 안전하다.

##### <span style="color:tomato">popAll 메서드</span>

~~~java
public void popAll(Collection<E> dst) {
		while(!isEmpty())
				dst.add(pop());
}
~~~

이 메서드도 컬렉션의 원소 타입이 스택의 원소 타입과 일치한다면 말끔히 컴파일되고 문제없이 동작한다.

Stack\<Number\> 의 원소를 Object용 컬렉션으로 옮기려하면 오류가 발생한다.

Collection\<Object\> 는 Collection\<Number\> 의 하위 타입이 아니다.

##### 해결책 - E 소비자 매개변수에 와일드카드 타입 적용

popAll의 입력 매개변수의 타입이 'E의 Collection' 이 아니라 'E의 상위 타입의 Collection' 이어야 한다.

와일드카드 타입을 사용한 Collection\<? spuper E\> 가 'E의 상위 타입의 Collection' 이라는 뜻이다.

~~~java
public void popAll(Collection<? spuer E> dst) {
		while(!isEmpty())
				dst.add(pop());
}
~~~

Stack과 클라이언트 모두 말끔히 컴파일된다.

##### <span style="color:blue">유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라.</span>



##### 와일드카드 타입을 써야하는 경우 - 공식

펙스(PECS): producer-extends, consumer-super

매개변수화 타입 T가 생산자라면 \<? extends T\> 를 사용하고, 소비자라면 \<? super T\> 를 사용하라.



##### <span style="color:tomato">아이템 28의 Chooser 생성자- 생산자 매개변수에 와일드카드 타입 적용</span>

~~~java
public Chooser(Collection<T> choices) // 아이템 28의 Chooser - 수정 전
public Chooser(Collection<? extends T> choices) // 한정적 와일드카드 타입 적용 - 수정 후
~~~

Chooser\<Number\> 의 생성자에 List\<Integer\> 를 넘기려고 할 때 수정 전 생성자로는 컴파일 조차 되지 않겠지만, 한정적 와일드카드 타입으로 선언한 수정 후 생성자에서는 문제가 사라진다.

##### <span style="color:tomato">아이템 30의 union 메서드 - 생산자 매개변수에 와일드카드 타입 적용</span>

~~~java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) // 수정 전
public static <E> Set<E> union(Set<? extemds e> s1, Set<? extends E> s2) // 수정 후
~~~

> 반환 타입에는 한정적 와일드카드 타입을 사용하면 안된다.

수정한 선언을 사용하면 말끔히 컴파일 된다.

##### <span style="color:blue">클래스 사용자가 와일드카드 타입을 신경써야 한다면 그 API에는 무슨 문제가 있을 가능성이 크다.</span>



##### <span style="color:tomato">아이템 30의 max 메서드 - 생산자 매개변수에 와일드카드 타입 적용</span>

~~~java
public static <E extends Comparable<E>> E max(List<E> list) // 수정 전
public static <E extends Comparable<? super E>> E max(List<? extends E> list) // 수정 후
~~~

입력 매개변수에서는 E 인스턴스를 생산하므로 원래의 List\<E\> 를 List\<? extends E\> 로 수정했다.

타입 매개변수의 원래 선언은 E가 Comparable\<E\> 를 확장한다고 정의했는데, 이때 Comparable\<E\> 는 E 인스턴스를 소비한다. 그래서 매개변수화 타입 Comparable\<E\> 를 한정적 와일드카드 타입인 Comparable\<? super E\> 로 대체 했다. 

##### <span style="color:blue">Comparable 언제나 소비자이므로 Comparable\<? super E\> 를 사용하는 편이 낫다.</span>



##### 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카도르 대체하라

비한정적 타입 매개변수라면 비한정적 와일드카드로 바꾸고, 한정적 타입 매개변수라면 한정적 와일드카드로 바꾼다.

###### 리스트에서 명시한 두 인덱스의 아이템을 교환하는 swap 두 가지 선언

~~~java
public static <E> void swap(List<E> list, int i, int j); // 첫 번째 swap
public static void swap(List<?> list, int i, int j);     // 두 번째 swap
~~~

두 번째 경우가 어떤 리스트든 명시한 인덱스의 원소를 교환해주어 더 낫다. 하지만 문제가 하나 있다.

~~~java
public static void swap(List<?> list, int i, int j) {
		list.set(i, list.set(j, list.get(i)));
}
~~~

두 번째를 직관적으로 구현한 코드이다. 하지만 컴파일 되지 않는다.

비한정적 와일드카드 List\<?\> 는  null 이외의 어떤 값도 넣을 수 없다는 것을 아이템 26에서 배웠다.

##### 해결 방법 - private 도우미 메서드를 활용한다.

~~~java
public static void swap(List<?> list, int i, int j) {
    swapHelper(list, i, j);
}

// 와일드카드 타입을 실제 타입으로 바꿔주는 private 도우미 메서드
private static <E> void swapHelper(List<E> list, int i, int j) {
    list.set(i, list.set(j, list.get(i)));
}
~~~

swapHelper 메서드는 리스트가 List\<E\> 임을 알고 있다. 그래서 이 리스트에서 꺼낸 값은 항상 E이고, E 타입의 값이라면 항상 이 리스트에 넣어도 안전하다.

> 도우미 메서드의 시그니처는 앞에서 "public API로 쓰기에는 너무 복잡하다"는 이유로 버렸던 첫 번째 swap 메서드의 시그니처와 완전히 똑같다.



##### 핵심 정리

> 조금 복잡하더라도 와일드카드 타입을 적용하면 API가 훨씬 유연해진다. 그러니 널리 쓰일 라이브러리를 작성한다면 반드시 와일드카드 타입을 적절히 사용해줘야 한다. PECS 공식을 기억하자. 즉, 생산자(producer)는 extends를 소비자(consumer)는 super를 사용한다. Comparable과 Comparator는 모두 소비자라는 사실도 잊지 말자.



___



## 아이템 32: 제네릭과 가변인수를 함께 쓸 때는 신중하라

* 제네릭과 가변인수는 자바 5에서 추가되었지만 잘 어울리지 않는다.

* 가변인수는 메서드에 넘기는 인수의 개수를 클라이언트가 조절할 수 있게 해주는데 구현 방식에 허점이 있다.

* 가변인수 메서드를 호출하면 가변인수를 담기 위한 배열이 자동으로 하나 생성된다.
* 생성된 배열은 내부로 감춰야했지만 클라이언트에 노출하는 문제가 있다.
* 그 결과 varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 어려운 컴파일 경고가 발생한다.
* 거의 모든 제네릭과 매개변수화 타입은 실체화되지 않아 메서드를 선언할 때 실체화 불가 타입으로 varargs 매개변수를 선언하면 경고를 보낸다.
* 가변인수 메서드를 호출할 때도 varargs 매개변수가 실체화 불가 타입으로 추론되면 경고를 낸다.
* 매개변수화 타입의 변수가 타입이 다른 객체를 참조하면 힙 오염이 발생한다.



##### 제네릭과 varargs를 혼용하면 타입 안정성이 깨진다.

~~~java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42);
    Object[] objects = stringLists;
    objects[0] = intList; // 힙 오염 발생
    String s = stringLists[0].get(0); // ClassCastException
}
~~~

이 메서드에서는 형변환하는 곳이 보이지 않아도 인수를 건네 호출하면 ClassCastException을 던진다.

마지막 줄에 컴파일러가 생성한 형변환이 숨어있고 이 때문에 타입 안정성이 깨져 **제네릭 varargs 배열 매개변수에 값을 저장하는 것은 안전하지 않다.**



##### 제네릭 varargs 매개변수를 받는 메서드 선언을 허용한 이유

제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 매우 유용하기 때문이다.

###### 자바 라이브러리에서 제공하는 메서드

* Arrays.asList(T... a)
* Collections.addAll(Collection\<? super T\> c, T... elements )
* EnumSet.of(E first, E... rest) 
* ...

이 메서드 들은 타입 안전하다.



##### @SafeVarargs 애너테이션

메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치. 컴파일러는 경고를 더 이상 하지 않는다.

단, 메서드가 안전한게 확실하지 않다면 절대 @SafeVarargs 를 달아선 안된다.

##### <span style="color:tomato">메서드가 안전한지 확신하는 방법</span>

* 가변인수 메서드를 호출할 때 varargs 매개변수를 담는 제네릭 배열이 만들어진다.

* 메서드가 이배열에 아무것도 저장하지 않고 그 배열의 참조가 밖으로 노출되지 않는다면 타입 안전하다.

* varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면 그 메서드는 안전하다.

* ##### <span style="color:red">varargs 매개변수 배열에 아무것도 저장하지 않고도 타입 안저엉을 깰 수 있는 경우</span>

  자신의 제네릭 매개변수 배열의 참조를 노출한다 - 안전하지 않다

  ~~~java
  static <T> T[] toArray(T... args){
  		return args;
  }
  ~~~

  이 메서드가 반환하는 배열의 타입은 이 메서드에 인수를 넘기는 컴파일 타임에 결정되는데, 그 시점에는 컴파일러에게 충분한 정보가 주어지지 않아 타입을 잘못 판단할 수 있다. 따라서 자신의 varargs 매개변수 배열을 그대로 반환하면 힙 오염을 이 메서드를 호출한 쪽의 콜스택으로까지 전이하는 결과를 낳을 수 있다.

  ##### 구체적인 예제

  ~~~java
  static <T> T[] pickTwo(T a, T b, T c) {
      switch(ThreadLocalRandom.current().nextInt(3)) {
          case 0: return toArray(a, b);
          case 1: return toArray(a, c);
          case 2: return toArray(b, c);
      }
      throw new AssertionError(); // 도달할 수 없다.
  }
  ~~~

  1. 이 메서드는 제네릭 가변인수를 받는 toArray 메서드를 호출한다는 점만 빼면 위험하지 않고 경고도 내지 않을 것이다.
  2. 컴파일러는 toArray에 넘길 T 인스턴스 2개를 담을 varargs 매개변수 배열을 만드는 코드를 생성한다.
  3. 이 코드가 만드는 배열의 타입은 Object[]이다.
  4. toArray 메서드가 돌려준 배열이 그대로 반환된다. 따라서 pickTwo는 항상 Object[] 타입을 반환한다.
  5. `String[] attributes = pickTwo("좋은","빠른", "저렴한");`을 실행하면 경고없이 컴파일 한다.
  6. 하지만 실행하면 ClassCastException을 던진다.
  7. 에러는 pickTwo의 반환 값을 String[] 으로 형변환하는 과정에서 발생한다.
  8. Object[] 는 String[] 의 하위 타입이 아니므로 형변환은 실패하기 때문이다.

  ##### 이 예제는 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다는 것을 보여준다.

  단 예외 두가지가 있다.

  1. @SafeVarargs가 붙은 다른 varargs 메서드에 넘기는 것은 안전하다.
  2. 배일 내용의 일부 함수만 호출하는 일반 메서드에 넘기는 것도 안전하다.



##### 제네릭 varargs 매개변수를 안전하게 사용하는 전형적인 예제

~~~java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
~~~

flattern 메서드는 임의 개수의 리스트를 인수로 받아, 받은 순서대로 그 안의 모든 원소를 하나의 리스트로 옮겨 담아 반환한다.

@SafeVarargs 애너테이션이 있으므로 선언하는 쪽과 사용하는 쪽 모두에서 경고를 내지 않는다.

##### @SafeVarargs를 사용할 때를 정하는 규칙 

제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달아라.

반드시 그 메서드가 진짜 안전한지 점검을 하라. 

1. varargs 매개변수 배열에 아무것도 저장하지 않았는 지.
2. 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않았는 지.

> @SafeVarargs 애너테이션은 재정의 할 수 없는 메서드에만 달아야 한다.



@SafeVarargs 애너테이션이 유일한 정답은 아니다.

##### varargs 매개변수를 List 매개변수로 바꿀 수도 있다.

~~~java
// 제네릭 varargs 매개변수를 List로 대체한 예 - 타입 안전하다.
static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists)
        result.addAll(list);
    return result;
}
~~~

정적 팩터리 메서드인 List.of 를 활용하면 다음과 같이 임의 개수의 인수를 넘길 수 있다.

~~~java
audience = flattern(List.of(friends, romans, countrymen));
~~~

이 방식의 장점은 컴파일러가 이 메서드의 타입 안정성을 검증할 수 있다는 데에 있다. 실수로 안전하다고 판단할 걱정이 없다.

##### <span style="color:tomato">toArray 예제에 적용</span>

~~~java
static <T> List<T> pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
        case 0: return List.of(a, b);
        case 1: return List.of(a, c);
        case 2: return List.of(b, c);
    }
    throw new AssertionError();
}
~~~

~~~java
List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
~~~

배열없이 제네릭만 사용하므로 타입 안전하다.



##### 핵심 정리

> 가변인수와 제네릭은 궁합이 좋지 않다. 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하고, 배열과 제네릭의 타입 규칙이 서로 다르기 때문이다. 제네릭 varargs 매개변수는 타입 안전하지는 않지만, 허용된다. 메서드에 제네릭(혹은 매개변수화된) varargs 매개변수를 사용하고자 한다면, 먼저 그 메서드가 타입 안전한지 확인한 다음 @SafeVarargs 애너테이션을 달아 사용하는 데 불편함이 없게끔 하자.



___



## 아이템 33: 타입 안전 이종 컨테이너를 고려하라

제네릭은 Set\<E\>, Map\<K,V\> 등의 컬렉션과 ThreadLocal\<T\>, AtomicReference\<T\> 등 단일원소 컨테이너에도 흔히 쓰인다. 이런 모든 쓰임에서 매개변수화되는 대상은 (원소가 아닌) 컨테이너 자신이다.

Set에서는 원소의 타입을 뜻하는 단 하나의 매개변수만 있으면 되며, Map에는 키와 값의 타입을 뜻하는 2개만 필요하다.



##### 타입 안전 이종 컨테이너 패턴(type safe heterogeneous container pattern)

컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면 된다. 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장한다.

##### 예제

타입별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorites 클래스

* 각 타입의 Class 객체를 매개변수화한 키 역할로 사용하면 되는데, 이 방식이 동작하는 이유는 class의 클래스가 제네릭이기 때문이다.
* class 리터럴의 타입은 Class가 아닌 Class\<T\>다. 
* String.class 의 타입은 Class\<String\> 이고 Integer.class 의 타입은 Class\<Integer\> 이다.
* 컴파일타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 타입 토큰(type token)이라고 한다.

~~~java
// 타입 안전 이종 컨테이너 패턴 - API
public class Favorites {
    public <T> void putFavorite(Class<T> type, T instance);
    public <T> T getFavorite(Class<T> type);
}
~~~

클라이언트 사용

~~~java
public static void main(String[] args) {
    Favorites f = new Favorites();
    
    f.putFavorite(String.class, "Java");
    f.putFavorite(Integer.class, 0xcafebabe);
    f.putFavorite(Class.class, Favorites.class);
       
    String favoriteString = f.getFavorite(String.class);
    int favoriteInteger = f.getFavorite(Integer.class);
    Class<?> favoriteClass = f.getFavorite(Class.class);
        
    System.out.printf("%s %x %s%n", favoriteString,
               favoriteInteger, favoriteClass.getName());
}
~~~

즐겨 찾는 String, Integer, Class 인스턴스를 저장, 검색, 출력한다.

Favorites 인스턴스는 타입 안전하다. String을 요청했는데 Integer를 반환하는 일은 절대 없다.

또한 모든 키의 타입이 제각각이라, 일반적인 맵과 달리 여러 가지 타입의 원소를 담을 수 있다.

따라서 Favorites는 타입 안전 이종 컨테이너라고 할만하다.

##### Favorites의 구현

~~~java
public class Favorites {
    // 코드 33-3 타입 안전 이종 컨테이너 패턴 - 구현 (200쪽)
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
~~~

* Favorites가 사용하는 private 맵 변수인 favorites의 타입은 Map\<Class\<?\>, Object\> 이다. 
* 비한정적 와일드카드 타입이라 맵 안에 아무것도 넣을 수 없다고 생각할 수 있지만, 사실은 그 반대다.
* 와일드카드 타입이 중첩(nested) 되었다는 점을 깨달아야 한다.
* 맵이 아니라 키가 와일드카드 타입인 것이다
* 모든 키가 서로 다른 매개변수화 타입일 수 있다는 뜻으로 첫 번째는 Class\<String\>, 두 번째는 Class\<Integer\> 식으로 될 수 있다. 다양한 타입을 지원하는 힘이 여기서 나온다.
* favorites 맵의 값의 타입은 단순히 Object 이다.
* 이 맵은 키와 값 사이의 타입 관계를 보증하지 않는다는 것이다. 
* 하지만 우리는 이 관계가 성립함을 알고 있고, 즐겨찾기를 검색할 때 그 이점을 누리게 된다.

###### putFavorite의 구현

주어진 Class 객체와 즐겨찾기 인스턴스를 favorites에 추가해 관계를 지으면 끝이다. 

###### getFavorite의 구현

주어진 Class 객체에 해당하는 값을 favorites에서 꺼낸다. 이 객체가 바로 반환해야 할 객체가 맞지만, 잘못된 컴파일타임 타입을 가지고 있다. 이 객체의 타입은 Object이나, 이를 T 로 바꿔 반환해야 한다.

따라서, getFavorite 구현은 Class의 cast  메서드를 사용해 이 객체 참조를 Class 객체가 가리키는 타입으로 동적 형변환한다.

cast 메서드는 형변환 연산자의 동적 버전이다. 이 메서드는 단순히 주어진 인수를 그대로 반환하고, 아니면 ClassCastException을 던진다.

getFavorite이 호출하는 cast는 ClassCastException을 던지지 않을 것을 우리는 알고 있다. 다시 말하면 favorites 맵 안의 값은 해당 키의 타입과 항상 일치함을 알고 있다.

cast 메서드를 사용하는 이유는 Class 클래스가 제네릭이라는 이점을 완벽히 활용하기 때문이다. cast 의 반환 타입은 Class 객체의 타입 매개변수와 같다.

~~~java
public Class<T> {
		T cast(Object obj);
}
~~~

getFavorite 메서드에 필요한 기능으로, T로 비검사 형변환하는 손실 없이도 Favorites를 타입 안전하게 만드는 비결이다.



##### Favorites 클래스에 알아두어야할 제약 - 두 가지

1. 악의적인 클라이언트가 Class 객체를 (제네릭이 아닌) 로 타입으로 넘기면 Favorites의 인스턴스의 타입 안정성이 쉽게 깨진다. 
   Favorites가 타입 불변식을 어기는 일이 없도록 보장하려면 putFavorite 메서드에 인수로 주어진 instance의 타입이 type으로 명시한 타입과 같은지 확인하면 된다. 동적 형변환을 쓴다.

   ~~~java
   // 동적 형변환으로 런타임 타입 안정성 확보
   public <T> void putFavorite(Class<T> type, T instance) {
       favorites.put(Objects.requireNonNull(type), type.cast(instance));
   }
   ~~~

   java.util.Collections의 checkedSet, checkedList, checkedMap 같은 메서드들...

2. 실체화 불가 타입에는 사용할 수 없다는 것이다. 
   즐겨찾는 String이나 String[]은 저장할 수 있어도 즐겨 찾는 List\<String\>은 저장할 수 없다.

   List\<String\> 을 저장하려는 코드는 컴파일 되지 않을 것이다. List\<String\> 용 Class 객체를 얻을 수 없기 때문이다.



##### Favorites가 사용하는 타입 토큰은 비한정적이다.

getFavorite과 putFavorite은 어떤 Class 객체든 받아들인다. 때로는 이 메서드들이 허용하는 타입을 제한하고 싶을 수 있는데, 한정적 타입 토큰을 활용하면 가능하다. 
한정적 타입 토큰이란 단순히 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한하는 타입이다.



##### 애너테이션 API는 한정적 타입 토큰을 적극적으로 사용한다.

~~~java
public <T extends Annotation> T getAnnotation(Class<T> annotationType);
~~~

annotationType 인수는 애너테이션 타입을 뜻하는 한정적 타입 토큰이다. 이 메서드는 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달려 있다면 그 애너테이션을 반환하고 없다면 null을 반환한다. 즉, 애너테이션된 요소는 그 키가 타입인, 타입 안전 이종 컨테이너 인 것이다.

##### Class\<T\> 타입의 객체가 있고 한정적 타입 토큰을 받는 메서드에 넘기려면 어떻게 할까?

Class 클래스가 이런 형변환을 안전하게 수행해주는 인스턴스 메서드를 제공한다.asSubClass 메서드로 호출된 인스턴스 자신의Class 객체를 인수가 명시한 클래스로 형변환한다.

형변환에 성공하면 인수로 받은 클래스 객체를 반환하고, 실패하면 ClassCastException을 던진다.

~~~java
static Annotation getAnnotation(AnnotatedElement element,
                                    String annotationTypeName) {
    Class<?> annotationType = null; // 비한정적 타입 토큰
    try {
        annotationType = Class.forName(annotationTypeName);
    } catch (Exception ex) {
        throw new IllegalArgumentException(ex);
    }
    return element.getAnnotation(
            annotationType.asSubclass(Annotation.class));
}
~~~

타입을 알 수 없는 애너테이션을 asSubClass 메서드를 사용해 런타임을 읽어내는 예이다. 이 메서드는 오류나 경고 없이 컴파일 된다.



##### 핵심 정리

> 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다. 하지만 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다. 타입 안전 이종 컨테이너는 Class의 키로 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 한다. 또한, 직접 구현한 키 타입도 쓸 수 있다. 예컨대 데이터베이스의 행(컨테이너)을 표현한 DatabaseRow 타입에는 제네릭 타입인 Column\<T\>를 키로 사용할 수 있다.



___

5장 제네릭 끝...
