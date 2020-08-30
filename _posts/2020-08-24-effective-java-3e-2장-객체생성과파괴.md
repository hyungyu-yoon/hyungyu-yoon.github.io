---
title: effective java 3/E - 2장. 객체 생성과 파괴
tags: 
 - Java
key: 7
---

# 2장 객체 생성과 파괴

##### 2장에서 배울 내용

* 객체를 만들어야 할 때와 만들지 말아야 할 때를 구분하는 법
* 올바른 객체 생성 방법과 불필요한 생성을 피하는 방법
* 제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요령



## 아이템 1 : 생성자 대신 정적 팩터리 메서드를 고려하라

 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자이다. 하지만 생성자와 별도로 **정적 팩터리 메서드(static factory method)**를 제공할 수 있다.



#### 정적 팩터리 메서드가 생성자보다 좋은 장점 다섯 가지

##### 1. 이름을 가질 수 있다.

* 생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다. 반면, 정적 팩터리 메서드는 이름만 잘 짓는다면 반환될 객체의 특성을 쉽게 묘사할 수 있다.

  ex) 3 비트로 표현할 수 있는 소수를 반환하는 생성자와 정적 팩터리 메서드

  ~~~java
  BigInteger prime = new BigInteger(3, 1, new Random()); // 생성자
  
  BigInteger prime = BigInteger.probablePrime(3, new Random());  // 정적 팩터리 메서드
  ~~~

  '값이 소수인 BigInteger 값을 반환한다' 라는 의미를 가진 생성자와 정적 팩터리 메서드이다. 정적 팩터리 메서드의 이름으로 반환될 객체의 특성을 알 수 있다.

##### 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

* 불변 클래스는(imumutable)는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피한다.

  ex) **Boolean.valueOf(boolean)** 메서드는 객체를 아예 생성하지 않는다. 
        플라이 웨이트 패턴(Flyweight pattern)

  (특히 생성 비용이 큰) 같은 객체가  자주 요청되는 상황이라면 성능을 상당히 끌어올려 준다.

* 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제할 수 있다. 인스턴스 통제(instance-controlled) 클래스

  * 싱글턴(singleton)을 만들 수 있다.
  * 인스턴스화 불가(noninstantiable)로 만들 수 있다.
  * 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장한다.

##### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

* 이것은 엄청난 유연성으로 API를 만들 때 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다. 이는 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크를 만드는 핵심기술이다.
* 프로그래머는 명시한 인터페이스대로 동작하는 객체를 얻을 것을 알기에 별도 문서를 찾아가 실제 구현 클래스가 무엇인지 알아보지 않아도 된다. 정적 팩터리 메서드를 사용하는 클라이언트는 얻은 객체를 인터페이스 만으로 다루게 된다.
* 자바 8부터 인터페이스가 정적 팩터리 메서드를 가질 수 있다.

##### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

* 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다. 

  ex) EnumSet 클래스는 public 생성자 없이 오직 정적 팩터리 메서드만 제공한다.

  ~~~java
  public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
          Enum<?>[] universe = getUniverse(elementType);
          if (universe == null)
              throw new ClassCastException(elementType + " not an enum");
  
          if (universe.length <= 64)
              return new RegularEnumSet<>(elementType, universe);
          else
              return new JumboEnumSet<>(elementType, universe);
      }
  ~~~

  원소가 64개 이하이면 RegularEnumSet 인스턴스를 반환하고 이상이면 JumboEnumSet 인스턴스를 반환한다. 클라이언트는 이 두 클래스의 존재를 모르며,  단지 반환하는 객체가 EnumSet의 하위 클래스이기만 하면 되는 것이다.

##### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

* 이러한 유연함은 서비스 제공자 프레임워크(Service Provider Framework)를 만드는 근간이 된다. 

  ex) 대표적 서비스 제공자 프레임워크 **JDBC(Java Database Connectivity)**

* 서비스 제공자 프레임워크의 3가지 핵심 컴포넌트 (+ 서비스 제공자 인터페이스)

  1. 구현체 동작을 정의하는 서비스 인터페이스(service interface)

  2. 제공자가 구현체를 등록할 때 사용하는 제공자 등록 API(provider registration API)

  3. 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 서비스 접근 API(service access API)

     서비스 접근 API는 '유연한 정적 팩터리'의 실체다.

  4. 서비스 제공자 인터페이스(service provider interface): 인스턴스를 생성하는 팩터리 객체를 설명해준다.

* 서비스 제공자 프레임워크 패턴의 여러 변형

  * 브리지 패턴(Bridge pattern)
  * 의존 객체 주입 프레임워크(dependency injection, 의존성 주입)



#### 정적 팩터리 메서드의 단점

##### 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

##### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

* 생성자처럼 API 설명에 명확히 드러나지 않아 사용자는 정적 팩터리 메서드 방식 크래스를 인스턴스화 할 방법을 찾아야 한다. 
* API 문서를 잘 써놓고 메서드 이름을 널리 알려진 규약에 따라 짓는다.



##### 핵심 정리

>정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다. 그렇다고 하더라도 정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있면 고치자.



___



## 아이템 2:  생성자에 매개변수가 많다면 빌더를 고려하라

##### 점층적 생성자 패턴

 정적 팩터리 메서드와 생성자는 선택적 매개변수가 많을 경우 적절히 대응하기가 어렵다. 다수의 매개변수가 있고 필요한 것만 받을 때 프로그래머들은 점층적 생성자 패턴(telescoping constructor pattern)을 사용하여 매개변수를 점층적으로 늘리는 방식을 사용했다.
 **점층적 생성자 패턴도 쓸 수는 있지만, 매개변수가 많아지면 코드를 작성하거나 읽는 것이 어려워진다.**

~~~java
public class NutritionFacts {
    private final int servingSize;  // (mL, 1회 제공량)     필수
    private final int servings;     // (회, 총 n회 제공량)  필수
    private final int calories;     // (1회 제공량당)       선택
    private final int fat;          // (g/1회 제공량)       선택
    private final int sodium;       // (mg/1회 제공량)      선택
    private final int carbohydrate; // (g/1회 제공량)       선택

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings,
                          int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
~~~



##### 자바빈즈 패턴

 매개 변수가 많을 때 활용할 수 있는 두 번째 대안인 자바빈즈 패턴(JavaBeans pattern)이 있다. 매개 변수가 없는 생성자를 만들고 세터(setter) 메서드로 원하는 값을 설정한다.
 **자바빈즈 패턴에서는 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성(consistency)이 무너진 상태에 놓이게 되어 클래스를 불변으로 만들 수 없다.**

~~~java
NutritionFacts cocaCola = new NutritionFacts(); // 디폴트 생성자 생성
cocaCola.setServingSize(240); // setter로 값을 채움
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27); 
~~~



#### 빌더 패턴💎

점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비한 **빌더 패턴(Builder pattern)**
클라이언트는 필수 매개변수만으로 생성자(or 정적 팩터리)를 호출해 빌더 객체를 얻고 빌더 객체가 제공하는 일종의 세터 메서드로 원하는 매개변수를 설정한다. 마지막으로 build 메서드를 호출해 필요한(보통은 불변인) 객체를 얻는다.

~~~java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        // 빌더를 사용하여 객체 생성
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
                .calories(100).sodium(35).carbohydrate(27).build();
    }
}
~~~

 이 객체의 유효성을 검사를 위해 잘못된 매개변수를 최대한 일찍 발견하도록 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고, bulid 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식(invariant)을 검사하자.

##### 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.

 각 계층의 클래스에 관련 빌더를 멤버로 정의하자. 추상 클래스는 추상 빌더를, 구체 클래스는 구체 빌더를 갖게 한다.

###### 피자 클래스

~~~java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }
    
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }
}
~~~

###### 뉴욕 피자

~~~java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
~~~

###### 칼초네 피자

~~~java
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // 기본값

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() { return this; }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
~~~

###### 사용

~~~java
NyPizza pizza = new NyPizza.Builder(SMALL)
                .addTopping(SAUSAGE).addTopping(ONION).build();
        Calzone calzone = new Calzone.Builder()
                .addTopping(HAM).sauceInside().build();
~~~

 빌더 패턴에 장점만 있는 것은 아니다. 객체를 만들기 위해 먼저 빌더 부터 만들어야 한다. 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에는 문제가 될 수 있다. 매개변수가 4개 이상은 되어야 값어치를 하므로 매개변수가 많다면 애초에 빌더 패턴으로 시작하는 편이 나을 때가 많다.



##### 핵심정리

>생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다. 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 편하고, 자바빈즈보다 훨씬 안전하다.



___



## 아이템 3: private 생성자나 열거 타입으로 싱글턴임을 보증하라

 **싱글턴(singleton)**이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 싱글턴을 만드는 방식은 세 방식이 있다.

1. 생성자는 private로 감추고, public static 멤버를 하나 마련한다.

   ~~~java
   public class Elvis{
   	public static final Elvis INSTANCE = new Elvis();
   	private Elvis(){ ... }
   	
   	public void leveTheBuilding { ... }
   }
   ~~~

   private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화 할 때 딱 한 번만 호출 된다. public, protected 생성자가 없으므로 Elvis 클래스가 단 하나뿐임이 보장된다.

   ##### 장점

   * 해당 클래스가 싱글턴임이 API에 명백히 드러난다. public static 필드가 final 이므로 다른 객체를 참조할 수 없다. 
   * 그리고 간결하다.

2. 생성자는 private로 감추고, 정적 팩터리 메서드를 public static 멤버로 제공한다.

   ~~~java
   public class Elvis{
   	private static final Elvis INSTANCE = new Elvis();
   	private Elvis(){ ... }
   	public static Elvis getInstance(){
   		return INSTANCE;
   	}
   	
   	public void leveTheBuilding { ... }
   }
   ~~~

   Elvis.getInstance는 항상 같은 객체의 참조를 반환한다.

   ##### 장점

   * API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다. 
   * 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
   * 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.

둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순히 Serializable를 선언하는 것으로는 부족하다. 모든 인스턴스 필드를 일시적(transient)라고 선언하고 readResolve 메서드를 제공해야 한다. 이렇게 하지 안으면 직렬화된 인스턴스를 역직렬화 할 때마다 새로운 인스턴스가 만들어진다.

~~~java
private Object readResolve() {
	return INSTANCE;
}
~~~

3. 원소가 하나인 열거 타입을 선언하는 것이다.

   ~~~java
   public enum Elvis {
   	INSTANCE;
   	
   	public void leaveTheBuilding() { ... }
   }
   ~~~

   더 간결하고, 직렬화할 수 있고, 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 다른 인스턴스가 생기는 것을 막아준다.

   ##### 대부분의 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

   단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.

   

___



## 아이템 4: 인스턴스화를 막으려거든 private 생성자를 사용하라

 이따금 단순히 정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때가 있을 것이다. 공통의 일을 하는 유틸리티 클래스를 만들 때 사용할 수 있다. 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계하지 않지만 생성자를 명시하지 않으면 기본 생성자를 자동으로 생성한다. 추상 클래스로 만드는 것으로도 인스턴스화를 막을 수 없다. **오직 명시된 생성자가 없도록 private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.**

~~~java
public class UtilityClass {
	// 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지)
	private UtilityClass(){
		throw new AssertionError();
	}
	
	public static ...
}
~~~

이 코드는 어떤 환경에서도 클래스의 인스턴스화를 막아준다. 생성자가 존재하지만 호출할 수 없어 그다지 직관적이지는 않다. 적절한 주석을 달아 이해할 수 있도록 한다.



___



## 아이템 5: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

 많은 클래스는 하나 이상의 자원에 의존한다. 사전에 의존하는 맞춤법 검사기 예를 보자.

##### 정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다

~~~java
public class SpellChecker {
	private static final Lexicon dictionary = ...;
	
	private SpellChecker(){} // 객체 생성 방지
	
	public static boolean isValid(String word) { ... }
	public static boolean List<String> suggestions(String typo) { ... }
}
~~~

##### 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다.

~~~java
public class SpellChecker {
	private final Lexicon dictionary = ...;
	
	private SpellChecker() {}
	public static SpellChecker INSTANCE = new SpellChecker();
	
	public boolean isValid(String word) { ... }
	public List<String> suggestions(String typo) { ... }
}
~~~

두 방식모두 사전을 단 하나만 사용한다고 가정하는 점에서 훌륭하지 않다. 실전에서는 여러 사전이 필요할 수 있다.

##### 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식 - 의존 객체 주입😀

의존 객체 주입은 유연성과 테스트 용이성을 높혀준다.

~~~java
public class SpellCheker {
	private final Lexicon dictionary;
	
	public SpellChecker(Lexicon dictionary){
		this.dictionary = Object.requireNonNull(dictionary);
	}
	
	public boolean isValid(String word) { ... }
	public List<String> suggestions(String typo) { ... }
}
~~~

자원이 몇 개든 상관 없이 필요한 의존 객체를 주입하여 사용할 수 있다.

이 패턴의 쓸만한 변형으로 생성자에 자원 팩터리를 넘겨주는 방식이 있다. 팩터리란 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말한다. **팩터리 메서드 패턴(factory method pattern)** 을 구현한 것.

의존 객체 주입이 유연성과 테스트 용이성을 개선해주지만, 의존성이 수 천개가 되는 큰 프로젝트에서는 코드를 어지럽게 만든다. 이를 돕기 위한 대거(Dagger), 주스(Guice), 스프링(Spring) 같은 의존 객체 주입 프레임워크가 있다.

##### 핵심 정리

>클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이 자원들을 클래스가 직접 만들게 해서도 안된다. 대신 필요한 자원을 생성자에 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해준다.



___



## 아이템 6: 불필요한 객체 생성을 피하라

 똑같은 기능의 객체를 매번 생성하기보다 객체 하나를 재사용하는 편이 나을 때가 많다. 재사용은 빠르고 세련되다. 특히 불변 객체는 언제든 재사용할 수 있다.

##### 불변 객체인 String

~~~java
String str = new String("apple");
~~~

이 코드는 실행될 때마다 매번 String 인스턴스를 새로 만든다.

~~~java
String str = "apple";
~~~

이 코드는 새로운 인스턴스를 매번 만들지 않고 단 하나의 String 인스턴스를 사용한다. 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장된다.



##### 생성자 대신 정적 팩터리 메서드를 제공하는 불변 클래스

정적 팩터리 메서드의 사용으로 불필요한 객체 생성을 피할 수 있다. 불변 객체만이 아니라 가변 객체라 하더라도 사용중에 변경 되지 않을 것을 안다면 사용할 수 있다.



##### 생성 비용이 아주 비싼 객체

 비싼 객체가 반복해서 필요하다면 캐싱하여 재사용하자.

~~~java
static boolean isRomanNumeralSlow(String s) {
	return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
             + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
~~~

 String.matches는 정규표현식으로 문자열 행태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서는 반복해 사용하기에 적합하지 않다. 이 메서드 내부의 Pattern 인스턴스는 한번 사용하고 버려진다.

 성능을 개선하려면 필요한 정규표현식을 표현하는 Pattern 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱하고 나중에 isRomanNumeral 메서드가 호출될 때마다 이 인스턴스를 재사용한다. 

~~~java
private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

static boolean isRomanNumeralFast(String s) {
	return ROMAN.matcher(s).matches();
}
~~~

성능만 좋아지는 것만 아니라 코드도 명확해졌다.



##### 불필요한 객체 생성 - 오토박싱(auto boxing)

 오토박싱은 프로그래머가 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다. 코드상으로 함께 사용했을 때 문제는 없을 수 있지만 성능상에서 큰 문제가 발생할 수 있다. 박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.



##### 적절한 객체 생성이 필요하다

객체 생성은 비싸니 피해야 한다는 것은 아니다. 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 일반적으로 좋은일이다. 

아주 무거운 객체가 아닌 이상 객체 생성을 피하기 위해 객체 pool을 만들지 말자. (데이터베이스 연결 같이 생성 비용이 큰 경우)



___



## 아이템 7: 다 쓴 객체 참조를 해제하라

자바는 가비지 컬렉터가 다 사용된 객체를 알아서 회수에 메모리 관리의 수고를 덜어준다. 하지만 메모리 관리에 더 이상 신경을 쓰지 않아도 되는 것은 아니다.

##### 스택 예제 - 메모리 누수가 일어나는 위치

~~~java
public class Stack {
    ...
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    } 
  	...
}
~~~

 이 예제에서 스택에서 꺼내진 객체들을 가비지 컬렉터는 회수하지 않는다. 프로그램에서 더 이상 사용되지 않더라도 이 스택이 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문이다. 가비지 컬렉션 언어에서는 의도치 않게 객체를 살려두는 메모리 누수를 찾기 아주 까다롭다. 그래서 단 몇 개의 객체가 매우 많은 객체를 회수하지 못하게 할 수 있고 이는 잠재적으로 성능에 악영향을 줄 수 있다.

##### 해법

해당 참조를 다 썼을 때 null 처리하면 된다.

~~~java
public class Stack {
    ...
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }
  	...
}
~~~

 다 쓴 참조를 null 처리하면 다른 이점도 있다. 만약 null 처리한 참조를 실수로 사용하려 하면 프로그램은 즉시 NullPointerException을 던지며 종료된다. 프로그램의 오류는 가능한 조기에 발견하는 것이 좋다.
 이 문제로 인하여 모든 객체를 다 쓰자마자 일일히 null 처리할 필요도 없고 바람직하지도 않다. **객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.**

##### null 처리를 해야하는 경우

 스택처럼 메모리를 자기가 관리하는 경우에 해야한다. 이 스택은 객체 참조를 담는 elements 배열로 저장소 풀을 만들어 원소를 관리한다. 배열의 활성 영역에 속한 원소들이 사용되고 비활성 영역은 사용되지 않는다. 가비지 컬렉터는 이것을 모르기 때문에 프로그래머가 직접 비활성 영역이 되는 순간 null 처리를 하여 더 이상 쓰이지 않는 다는 것을 알려야 한다.

> 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.

 캐시 역시 메모리 누수를 일으키는 주범이다. 객체 참조를 캐시에 넣고 그 객체를 다 쓴 뒤로 한참을 두는 경우가 있다. 엔트리가 살아있는 캐시가 필요한 상황이라면 WeakHashMap을 사용해 캐시를 만들자. 다 쓴 엔트리는 즉시 자동으로 제거 될 것이다.

 리스너 혹은 콜백도 메모리 누수가 있다. 콜백을 등록만 하고 명확히 해지하지 않는다면, 뭔가 조치를 취하지 않는 이상 콜백은 쌓일 것이다. WeakHashMap에 키로 저장하자.

##### 핵심정리

> 메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있다. 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 한다. 그래서 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요하다.



___



## 아이템 8: finalizer와 cleaner 사용을 피하라

finalizer (java 8), cleaner (java 9)

##### 사용하지 말아야 할 이유

* finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
* cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.
* 즉시 수행된다는 보장이 없다. 즉, 제때 실행되어야 하는 작업은 절대 할 수 없다.
* 자바 언어 명세는 finalizer나 cleaner의 수행 시점뿐 아니라 수행 여부조차 보장하지 않는다. 상태를 영구적으로 수정하는 작업에서는 finalizer나 cleaner에 의존해서는 안된다.
* 심각한 성능 문제도 동반한다.
* finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.

##### 

##### AutoCloseable을 사용하자.

 AutoCloseable을 구현하고, 클라이언트에서 인스턴스를 다 쓰고나면 close를 호출하면 된다. (일반적으로 예외가 발생해도 제대로 종료되도록 try-with-resources를 사용해야한다.)
 각 인스턴스는 자신이 닫혔는지 추적하는 것이 좋다. close 메서드에서 이 객체는 더 이상 유효하지 않음을 필드에 기록하고 다른 메서드에서 이 필드를 검사하여 객체가 닫힌후 불렀다면 IllegalStateException을 던진다.



##### 핵심정리

> cleaner(자바 8까지는 finalizer)는 안전망 역할이나 중요하지 않는 네이티브 자원 회수 용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.



___



## 아이템 9: try-finally보다는 try-with-resources를 사용하라

 자바 라이브러리에는 close 메서드를 호출해 직접 닫아 줘야하는 자원이 많다. InputStream, OutputStream, java.sql.Connection 등이 좋은 예다. 

##### 전통적인 자원이 제대로 닫힘을 보장하는 try-finally - 더 이상 자원을 회수하는 최선의 방책이 아니다.

~~~java
static String firstLineOfFile(String path) throws IOException {
  	BufferedReader br = new BufferedReader(new FileReader(path));
  	try {
    	return br.readLine();
  	} finally {
  		br.close();
		}
}
~~~

나쁘지 않지만 자원을 하나 더 사용한다면?

~~~java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
   	try {
            OutputStream out = new FileOutputStream(dst);
    		try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
        		out.write(buf, 0, n);
        } finally {
        		out.close();
    		}
    } finally {
    		in.close();
		}
}
~~~

 자원이 둘 이상이면 try-finally 방식은 너무 지저분하다.



##### 해결책 try-with-resources를 사용하자

 이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다.

~~~java
public class AutoCloseableTest implements AutoCloseable {
    @Override
    public void close() throws Exception { // try-with-resources에서 자동 호출 된다.
        System.out.println("close test");
    }
}
~~~

~~~java
// try-with-resources - 자원을 회수하는 최선책! (48쪽)
static String firstLineOfFile(String path) throws IOException {
		try (BufferedReader br = new BufferedReader(
    		new FileReader(path))) {
        return br.readLine();
    }
}
~~~

 try-with-resources 버전이 짧고 읽기 수월할 뿐만 아니라 문제를 진단하기도 훨씬 좋다. 예외를 처리하기 위해 catch 절을 사용할 수 있다.

~~~java
static String firstLineOfFile(String path, String defaultVal) {
		try (BufferedReader br = new BufferedReader(
    		new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) { // catch 절로 예외를 처리할 수 있다.
        return defaultVal;
    }
}
~~~

##### 핵심정리

> 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자. 예외는 없다. 코드는 더 짧아지고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 쉽고 정확하게 자원을 회수할 수 있다.



___

2장 객채 생성과 파괴 끝

