---
title: effective java 3/E - 6장. 열거 타입과 애너테이션
tags: 
 - Java
key: 11
---

# 6장 열거 타입과 애너테이션

##### 6장에서 배울 내용

* 특수한 목적의 참조 타입 2가지
  * 클래스의 일종인 열거 타입(enum; 열거형)
  * 인터페이스의 일종인 애너테이션(annotation)
* 이 타입들을 올바르게 사용하는 방법을 알아본다.



___



## 아이템 34: int 상수 대신 열거 타입을 사용하라

열거 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다.

##### 정수 열거 패턴 - 상당히 취약하다!

~~~java
public static final int APPLE_FUJI          = 0;
public static final int APPLE_PIPPIN        = 1;
public static final int APPLE_GRANNY_SMITH  = 2;

public static final int ORANGE_NAVEL	= 0;
public static final int ORANGE_TEMPLE	= 1;
public static final int ORANGE_BLOOD	= 2;
~~~

정수형 열거 패턴(int enum pattern) 기법에는 단점이 많다. 타입 안전을 보장할 방법이 없으며 표현력이 좋지 않다.

오렌지를 건네야 할 메서드에 사과를 보내고 동등 연산자로 비교하더라도 컴파일러는 아무런 경고를 출력하지 않는다.

~~~java
// 향긋한 오렌지 향의 사과 소스
int i = (APPLE_FUJI - ORANGE_TEMPLE) / APPLE_PIPPIN;
~~~

* 사과용 상수의 이름은 모두 APPLE_ 로 시작하고, 오렌지용 상수는 ORANGE_로 시작한다. 
  자바가 정수 열거형 패턴을 위한 별도 이름공간(namespace)을 지원하지 않기 때문에 접두어를 써서 이름 충돌을 방지한 것이다.
* 정수형 열거 패턴을 사용한 프로그램은 깨지기 쉽다. 평범한 상수를 나열한 것 뿐이라 컴파일하면 그 값이 클라이언트 파일에 그대로 새겨진다. 따라서 상수의 값이 바뀌면 클라이언트도 반드시 다시 컴파일 해야 한다.
* 정수 상수는 문자열로 출력하기가 다소 까다롭다. 그 값을 출력하거나 디버거로 살펴보면 단지 숫자로만 보여서 썩 도움이 되지 않는다.
* 정수 대신 문자열 상수를 사용하는 변형 패턴도 있다. 문자열 열거 패턴(string enum pattern)이라 하는 이 변형은 더 나쁘다. 상수의 의미를 출력할 수 있다는 점은 좋지만, 경험이 부족한 프로그래머가 문자열 상수의 이름 대신 문자열 값을 그대로 하드코딩하게 만들기 때문이다. 하드코딩된 문자열에 오타가 있어도 컴파일러는 확인할 길이 없어니 자연스럽게 런타임 버그가 생길 수 있다.

##### 자바는 열거 패턴의 단점을 씻어주는 열거 타입(enum types)를 제공한다.

~~~java
public enum Apple {FUJI, PIPPIN, GRANNY_SMITH}
public enum Orange {NAVEL, TEMPLE, BLOOD}
~~~

자바의 열거 타입은 완전한 형태의 클래스라서 (단순한 정숫값일 뿐인) 다른 언어의 열거 타입보다 훨씬 강력하다.

* 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필도로 공개한다.
* 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final 이며, 열거 타입 인스턴트들은 단 하나의 인스턴스만 존재하는 것이 보장된다. (인스턴스 통제, 싱글턴)
* 열거 타입은 컴파일타임 타입 안정성을 제공한다. 위의 Apple 열거 타입을 매개변수로 받는 메서드를 선언했다면, 세 가지 값 중 하나임이 확실하다. 다른 타입의 값을 넘기면 컴파일 오류가 난다.
* 열거 타입에는 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.
* 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.

이처럼 열거 타입은 정수 열거 패턴의 단점을 해소해준다. 또 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다. Object 메서드, Comparable, Serializable 을 잘 구현해 놓았다.

##### 열거 타입에서 메서드, 필드를 추가는 언제 필요할까?

각 상수와 연관된 데이터를 해당 상수 자체에 내재시키고 싶을 경우

ex) 과일의 색을 알려주거나 과일 이미지를 반환하는 메서드를 추가하고 싶을 수 있다.

##### 태영계의 여덟 행성은 거대한 열거 타입을 설명하기 좋은 예이다.

~~~java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS  (4.869e+24, 6.052e6),
    EARTH  (5.975e+24, 6.378e6),
    MARS   (6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN (5.685e+26, 6.027e7),
    URANUS (8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);

    private final double mass;           // 질량(단위: 킬로그램)
    private final double radius;         // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)

    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;

    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }

    public double mass()           { return mass; }
    public double radius()         { return radius; }
    public double surfaceGravity() { return surfaceGravity; }

    public double surfaceWeight(double mass) {
        return mass * surfaceGravity;  // F = ma
    }
}
~~~

* 각 행성은 질량과 반지름이 있고, 이 두속성으로 표면중력을 계산할 수 있다. 
* 어떤 객체의 질량이 주어지면 그 객체가 행성 표면에 있을 때의 무게도 게산할 수 있다.

##### 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.

Planet 열거 타입에서 어떤 객체의 지구에서의 무게를 입력받아 여덟 행성에서의 무게를 출력하는 코드

~~~java
double earthWeight = Double.parseDouble("185.0");
double mass = earthWeight / Planet.EARTH.surfaceGravity();
for (Planet p : Planet.values())
    System.out.printf("%s에서의 무게는 %f이다.%n", p, p.surfaceWeight(mass));

--------------------------------
MERCURY에서의 무게는 69.912739이다.
VENUS에서의 무게는 167.434436이다.
EARTH에서의 무게는 185.000000이다.
MARS에서의 무게는 70.226739이다.
JUPITER에서의 무게는 467.990696이다.
SATURN에서의 무게는 197.120111이다.
URANUS에서의 무게는 167.398264이다.
NEPTUNE에서의 무게는 210.208751이다.
~~~

* 열거 타입은 자신 안에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 values를 제공한다. 
* 값들은 선언된 순서로 저장된다.
* 각 열거 타입 값의 toString 메서드는 상수 이름을 문자열로 반환하므로 println과 printf로 출력하기에 안성맞춤이다.
* 기본 toString이 제공하는 이름이 내키지 않으면 원하는 대로 재정의한다.
* 열거 타입에서 상수를 하나 제거해도 제거한 상수를 참조하지 않는 클라이언트에는 아무런 영향이 없다.
  참조를 하고 있는 클라이언트는 다시 컴파일하면 컴파일 오류가 발생한다.
* 열거 타입을 선언한 클래스 혹은 그 패키지에서만 유용한 기능은 private이나 package-private 메서드로 구현한다.
  이렇게 구현된 열거 타입 상수는 자신을 선언한 클래스 혹은 패키지에서만 사용할 수 있는 기능을 담게된다.
* 널리 쓰이는 열거 타입은 톱레벨 클래스로 만들고, 특정 톱레벨 클래스에서만 쓰인다면 해당 클래스의 멤버 클래스로 만든다.

##### 상수가 더 다양한 기능을 제공해주었으면 하는 경우

###### 값에 따라 분기하는 열거 타입 - 이대로 만족하는가?

~~~java
public enum Operation {
	PLUS, MINUS, TIMES, DIVIDE;
	
	public double apply(double x, double y){
		switch(this) {
			case PLUS:    return x + y;
			case MINUS:   return x - y;
			case TIMES:   return x * y;
			case DIVIDE:  return x / y;
		}
		throw new AssertionError("알 수 없는 연산: " + this);
	}
}
~~~

동작은 하지만 그리 예쁘지는 않다. 마지막의 throw 문은 실제로는 도달할 일이 없지만 생략하면 컴파일 조차 되지 않는다. 

더 나쁜점은 깨지기 쉬운 코드라는 사실이다. 새로운 상수를 추가하면  case 문도 추가해야 한다. 추가를 하지 않는다면, 컴파일은 되지만 새로 추가한 연산을 추가할 때 런타임 오류를 발생시킬 것이다.

##### 더 나은 수단 - 상수별 메서드 구현을 활용한 열거 타입

열거 타입에 apply라는 추상 메서드를 선언하고 각 상수별 클래스 몸체, 즉 각 상수에서 자신에 맞게 재정의 하는 방법. 

이를 상수별 메서드 구현이라고 한다(constant-specific method implementation)

~~~java
public enum Operation {
	PLUS {public double apply(double x, double y){return x + y;}},
	MINUS {public double apply(double x, double y){return x - y;}},
	TIMES {public double apply(double x, double y){return x * y;}},
	DIVIDE {public double apply(double x, double y){return x / y;}},
	
	public abstarct double apply(double x, double y);
}
~~~



##### 상수별 메서드 구현을 상수별 데이터와 결합 - Operation의 toString을 재정의해 해당 연산을 뜻하는 기호를 반환

###### 상수별 클래스 몸체(class body)와 데이터를 사용한 열거 타입

~~~java
public enum Operation {
	PLUS("+") {public double apply(double x, double y){return x + y;}},
	MINUS("-") {public double apply(double x, double y){return x - y;}},
	TIMES("*") {public double apply(double x, double y){return x * y;}},
	DIVIDE("/") {public double apply(double x, double y){return x / y;}},
	
	private final String symbol;
	
	Operation(String symbol) { this.symbol = symbol; }
	@Override public String toString() { return symbol; }
	public abstarct double apply(double x, double y);
}

public static void main(String[] args) {
    double x = Double.parseDouble("2");
    double y = Double.parseDouble("4");
    for (Operation op : Operation.values())
        System.out.printf("%f %s %f = %f%n", 
	            x, op, y, op.apply(x, y));
}

--------------------------- 
2.000000 + 4.000000 = 6.000000
2.000000 - 4.000000 = -2.000000
2.000000 * 4.000000 = 8.000000
2.000000 / 4.000000 = 0.500000
~~~

열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valueOf(String) 메서드가 자동 생성된다.

열거 타입의 toString 메서드를 재정의하려거든, toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공하는 것을 고려해보자.

##### 열거 타입용 fromString 메서드

~~~java
private static final Map<String, Operation> stringToEnum =
    Stream.of(values()).collect(toMap(Object::toString, e -> e));
					
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
~~~

*  Operation 상수가 stringToEnum 맵에 추가되는 시점은 열거 타입 상수 생성 후 정적 필드가 초기화될 때다.
*  열거 타입 상수는 생성자에서 자신의 인스턴스를 맵에 추가할 수 없다. 컴파일 오류 발생
*  열거 타입의 정적 필드 중 열거 타입의 생성자에서 접근할 수 있는 것은 상수 변수뿐이다.
*  열거 타입 생성자가 실행되는 시점에는 정적 필드들이 아직 초기화되기 전이라, 자기 자신을 추가하지 못하게 하는 제약이 꼭 필요하다.
*  열거 타입 생성자에서 같은 열거 타입의 다른 상수에도 접근할 수 없다.
*  fromString이 Optional\<Operation\> 을 반환하는 점도 주의하자.
   이는 주어진 문자열이 가리키는 연산이 존재하지 않을 수 있음을 클라이언트에 알리고, 그 상황을 클라이언트에서 대처하도록 한 것이다.



##### 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 고유하기 어렵다는 단점이 있다.

급여명세서에서 쓸 요일을 표현하는 열거 타입을 예로 들어보자.

이 열거 타입은 직원의 기본 임금과 그날 일한 시간이 주어지면 일당을 계산해주는 메서드를 갖고 있다. 주중에 오버타임이 발생하면 잔업수당이 주어지고, 주말에는 무조건 잔업수당이 주어진다. switch 문을 이용하면 case 문을 날짜별로 두어 이 계산을 쉽게 수행할 수 있다.

###### <span style="color:red;">값에 따라 분기하여 코드를 공유하는 열거 타입 - 좋은 방법인가?</span>

~~~java
enum PayrollDay {
	MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
	
	private static final int MINS_PER_SHIFT = 8 * 60;
	
	int pay(int minutesWorked, int payRate) {
		int basePay = minutesWorked * payRate;
		
		int overtimePay;
		switch(this) {
			case SATURDAY: case SUNDAY: // 주말
				overtimePay = basePay / 2;
				break;
			default: // 주중
				overtimePay = minutesWorked <= MINS_PER_SHIFT ?
					0 : (minutesWroked - MINS_PER_SHIFT) * payRate / 2;
		}
		
		return basePay + overtimePay;
	}
}
~~~

* 간결하지만, 관리 관점에서는 위험한 코드이다.
* 휴가와 같은 새로운 값을 열거 타입에 추가하려면 그 값을 처리하는 case 문을 잊지 말고 쌍으로 넣어야 한다.
* 깜빡하는 날에는 휴가 기간에 열심히 일해도 평일과 똑같은 임금을 받게 된다.



##### <span style="color:red;">상수별 메서드 구현으로 급여를 정확히 계산하는 방법 두 가지 </span>

1. 잔업수당을 계산하는 코드를 모든 상수에 중복해서 넣으면 된다.
2. 계산 코드를 평일용과 주말용으로 나눠 각각을 도우미 메서드로 작성한 다음 상수가 자신에게 필요한 메서드를 적절히 호출하면 된다.

두 방식 모두 코드가 장황해져 가독성이 크게 떨어지고 오류 발생 가능성이 높아진다.

##### <span style="color:blue;">가장 깔끔한 방법</span>

* 새로운 상수를 추가할 때 잔업수당 '전략'을 선택하도록 하는 것이다. 
* 잔업수당 계산을 private 중첩 열거 타입(PayType)으로 옮기고 PayrollDay 열거 타입의 생성자에서 이중 적당한 것을 선택한다. 
* 그러면 PayrollDay 열거 타입은 잔업수당 계산을 그 전략 열거 타입에 위임하여, switch 문이나 상수별 메서드 구현이 필요없게 된다.
* 이 패턴은 switch 문보다 복잡하지만 더 안전하고 유연하다.

~~~java
enum PayrollDay {
    MONDAY(WEEKDAY), TUESDAY(WEEKDAY), WEDNESDAY(WEEKDAY),
    THURSDAY(WEEKDAY), FRIDAY(WEEKDAY),
    SATURDAY(WEEKEND), SUNDAY(WEEKEND);
    // (역자 노트) 원서 1~3쇄와 한국어판 1쇄에는 위의 3줄이 아래처럼 인쇄돼 있습니다.
    // 
    // MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY,
    // SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);
    //
    // 저자가 코드를 간결하게 하기 위해 매개변수 없는 기본 생성자를 추가했기 때문인데,
    // 열거 타입에 새로운 값을 추가할 때마다 적절한 전략 열거 타입을 선택하도록 프로그래머에게 강제하겠다는
    // 이 패턴의 의도를 잘못 전달할 수 있어서 원서 4쇄부터 코드를 수정할 계획입니다.

    private final PayType payType;

    PayrollDay(PayType payType) { this.payType = payType; }
    // PayrollDay() { this(PayType.WEEKDAY); } // (역자 노트) 원서 4쇄부터 삭제
    
    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked, payRate);
    }

    // 전략 열거 타입
    enum PayType {
        WEEKDAY {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked <= MINS_PER_SHIFT ? 0 :
                        (minsWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minsWorked, int payRate) {
                return minsWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int mins, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minsWorked, int payRate) {
            int basePay = minsWorked * payRate;
            return basePay + overtimePay(minsWorked, payRate);
        }
    }

    public static void main(String[] args) {
        for (PayrollDay day : values())
            System.out.printf("%-10s%d%n", day, day.pay(8 * 60, 1));
    }
}

~~~

switch 문은 열거 타입의 상수별 동작을 구현하는 데 적합하지 않다.

##### 하지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.

서드파티에서 가져온 Operation 열거 타입이 있는데, 각 연산의 반대 연산을 반환하는 메서드가 필요하다고 해보자.

~~~java
public class Inverse {
    public static Operation inverse(Operation op) {
        switch(op) {
            case PLUS:   return Operation.MINUS;
            case MINUS:  return Operation.PLUS;
            case TIMES:  return Operation.DIVIDE;
            case DIVIDE: return Operation.TIMES;

            default:  throw new AssertionError("Unknown op: " + op);
        }
    }

    public static void main(String[] args) {
        double x = Double.parseDouble(args[0]);
        double y = Double.parseDouble(args[1]);
        for (Operation op : Operation.values()) {
            Operation invOp = inverse(op);
            System.out.printf("%f %s %f %s %f = %f%n",
                    x, op, y, invOp, y, invOp.apply(op.apply(x, y), y));
        }
    }
}
~~~

추가하려는 메서드가 의미상 열거 타입에 속하지 않는다면 직접 만든 열거 타입이라도 이 방식을 적용하는게 좋다.



##### 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.

태양계 행성, 한 주의 요일, 체스 말, 메뉴 아이템, 연산 코드, 명령줄 플래그 등 

열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.



##### 핵심 정리

> 열거 타입은 확실히 정수 상수보다 뛰어나다. 더 읽기 쉽고 안전하고 강력하다. 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다. 드물게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있다. 이런 열거 타입에서는 switch 문 대신 상수별 메서드 구현을 사용하자. 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.



___



## 아이템 35: ordinal 메서드 대신 인스턴스 필드를 사용하라

대부분의 열거 타입 상수는 자연스럽게 하나의 정숫값에 대응된다. 그리고 모든 열거 타입은 해당 상수가 그 열거 타입에서 몇 번째 위치인지를 반환하는 ordinal 이라는 메서드를 제공한다. 이런 이유로 열거 타입 상수와 연결된 정숫값이 필요하면 ordinal 메서드를 이용하고 싶은 유혹에 빠진다.

##### ordinal을 잘못 사용한 예 - 따라하지 말 것!

~~~java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;
    
    public int numberOfMusicians() { return ordinal() + 1}
}
~~~

* 동작은 하지만 유지보수가 끔찍한 코드다. 
* 상수 선언 순서를 바꾸는 순간 numberOfMusicians 가 오동작하며, 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다.
* 값을 중간에 비워둘 수도 없다.



##### 해결책

열거 타입 상수에 연결된 값은 ordinal 메서드로 얻지말고, 인스턴스 필드에 저장하자.

~~~java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
~~~



Enum의 API 문서에는 ordinal에 대해 이렇게 쓰여있다.

"대부분의 프로그래머는 이 메서드를 쓸일이 없다. 이 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다."

따라서 이런 용도가 아니라면 ordinal 메서드는 절대 사용하지 말자.



___



## 아이템 36: 비트 대신 EnumSet을 사용하라

열거한 값들이 주로 (단독이 아닌) 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해 왔다.

##### 비트 필드 열거 상수 - 구닥다리 기법!

~~~java
public class Text {
    public static final int STYLE_BOLD = 1 << 0;
    public static final int STYLE_ITALIC = 1 << 1;
    public static final int STYLE_UNDERLINE = 1 << 2;
    public static final int STYLE_STRIKETHROUGH = 1 << 3;
    
    public void applyStyles(int styles) { ... }
}
~~~

비트별 OR을 사용해 여러 상수를 하나의 집합으로 모을 수 있으며, 이렇게 만들어진 집합을 비트 필드라 한다.

~~~java
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
~~~

비트 필드를 사용하면 비트별 연산을 사용해 합집합과 교집합 같은 집합 연산을 효율적으로 수행할 수 있다.

하지만, 비트 필드는 정수 열거 상수의 단점을 그대로 지니며, 추가로 다음과 같은 문제까지 안고 있다.

비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기 훨씬 어렵다. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기도 까다롭다.



##### 더 나은 대안

java.util 패키지의 EnumSet 클래스는 열거 타입 상수의 값으로 구성된 집합을 효과적으로 표현한다.

Set 인터페이스를 완벽히 구현하며, 타입 안전하고, 다른 어떤 Set 구현체와도 함께 사용할 수 있다.

##### EnumSet - 비트 필드를 대처하는 현대적 기법

~~~java
public class Text {
    public enum Style {BOLD, ITALIC, UNDERLINE, STRIKETHROUGH}

    // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
    public void applyStyles(Set<Style> styles) {
        System.out.printf("Applying styles %s to text%n",
                Objects.requireNonNull(styles));
    }
}
~~~

applyStyles 메서드에 EnumSet을 건네는 클라이언트 코드

~~~
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
~~~

applyStyles 메서드가 EnumSet\<Style\> 이 아닌 Set\<Style\> 을 받는 이유를 생각해보자.

모든 클라이언트가 EnumSet을 건네리라 짐작되는 상황이라도 이왕이면 인터페이스로 받는 게 일반적으로 좋은 습관이다.

다른 Set 구현체를 넘기더라도 처리할 수 있다.

##### 핵심 정리

> **열거할 수 있는 타입을 한데 모아 집합 형태로 사용한다고 해도 비트 필드를 사용할 이유는 없다.** EnumSet 클래스가 비트 필드 수준의 명료함과 성능을 제공하고 아이템 34에서 설명한 열거 타입의 장점까지 선사하기 때문이다. EnumSet의 유일한 단점이라면 불변 EnumSet을 만들 수 없다는 것이다. 그래도 향후 릴리스에서는 수정되리라 본다. 그때까지는 Collections.unmodifiableSet으로 EnumSet을 감싸 사용할 수 있다.



___



## 아이템 37: ordinal 인데싱 대신 EnumMap을 사용하라

이따금 배열이나 리스트에서 원소를 꺼낼 때 ordinal 메서드로 인덱스를 얻는 코드가 있다.

##### 식물을 간단히 나타낸 클래스 예제

~~~java
class Plant {
    enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }

    final String name;
    final LifeCycle lifeCycle;

    Plant(String name, LifeCycle lifeCycle) {
        this.name = name;
        this.lifeCycle = lifeCycle;
    }

    @Override public String toString() {
        return name;
    }
}
~~~

* 정원에 심은 식물들을 배열 하나로 관리하고, 이들을 생애주기별로 묶어보자
* 생애주기별로 총 3개의 집합을 만들고 정원을 한 바퀴 돌며 각 식물을 해당 집합에 넣는다.
* 이때 어떤 프로그래머는 집합들을 배열 하나에 넣고 생애주기의 ordinal 값을 그 배열의 인덱스로 사용하려 할 것이다.

##### ordinal() 배열 인덱스로 사용 - 따라하지 말 것!

~~~java
Set<Plant>[] plantsByLifeCycleArr =
            (Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycleArr.length; i++)
    plantsByLifeCycleArr[i] = new HashSet<>();
for (Plant p : garden)
    plantsByLifeCycleArr[p.lifeCycle.ordinal()].add(p);
// 결과 출력
for (int i = 0; i < plantsByLifeCycleArr.length; i++) {
    System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycleArr[i]);
}
~~~

* 동작은 하지만 문제가 많다.
* 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않을 것이다.
* 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
* 가장 심각한 문제는 정확한 정숫값을 사용한다는 것을 직접 보증해야 한다.
* 정수는 열거 타입과 달리 타입 안전하지 않기 때문이다.
* 잘못된 값을 사용하면 잘못된 동작을 묵묵히 수행하거나 ArrayIndexOutOfBoundsException을 던질 것이다.

##### 해결책 - EnumMap을 사용해 데이터와 열거 타입을 매핑한다.

~~~java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle =
        new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
    plantsByLifeCycle.put(lc, new HashSet<>());
for (Plant p : garden)
    plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
~~~

* 더 짧고 명료하고 안전하고 성능도 괜찮다.
* 안전하지 않은 형변환은 쓰지 않고, 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하니 출력 결과에 직접 레이블을 달 필요가 없다.
* 배열 인덱스를 계산하는 과정에서 오류가 날 가능성도 원천봉쇄 된다.
* EnumMap은 내부에서 배열을 사용하지만 구현 방식을 숨겨서 Map의 타입 안정성과 성능을 모두 얻었다.
* EnumMap의 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰으로, 런타임 제네릭 타입 정보를 제공한다.

##### 해결책 - 스트림을 사용한 코드 1 - EnumMap을 사용하지 않는다.

~~~java
System.out.println(Arrays.stream(garden).collect(groupingBy(p -> p.lifeCycle)));
~~~

* EnumMap이 아닌 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다는 문제가 있다.

##### 해결책 - 스트림을 사용한 코드 2 - EnumMap을 이용해 데이터와 열거 타입을 매핑했다.

~~~java
System.out.println(Arrays.stream(garden)
        .collect(groupingBy(p -> p.lifeCycle, 
            () -> new EnumMap<>(LifeCycle.class), toSet())));
~~~

* 스트림을 사용하면 EnumMap만 사용했을 때와는 살짝 다르게 동작한다.
* EnumMap 버전은 언제나 식물의 생애주기당 하나씩 중첩 맵을 만들지만, 스트림 버전에서는 해당 생애주기에 속하는 식물이 있을 때만 만든다.
* 정원에 한해살이와 여러해살이 식물만 살고 두해살이는 없다면, EnumMap 버전에서는 맵을 3개를 만들고 스트림 버전에서는 2개를 만든다.



##### 두 열거 타입 값들을 매핑하느라 ordinal을 쓴 배열들의 배열

두 가지 상태(Phase)를 전이(Transition)와 매핑하도록 구현한 프로그램

액체(LIQUID)에서 고체(SOLID)로의 전이는 응고(FREEZE)가 되고, 액체에서 기체(GAS)로의 전이는 기화(BOIL)가 된다.

~~~java
public enum Phase {
    SOLID, LOQUID, GAS;
    
    public enum Transition {
        MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
        
        private static final Transition[][] TRANSITIONS = {
            {null, MELT, SUBLIME},
            {FREEZE, null, BOIL},
            {DEPOSIT, CONDENSE, null}
        }
        
        public static Transition from(Phase from, Phase to) {
            return TRANSITIONS[from.ordinal()][to.ordinal()];
        }
    }
}
~~~

* 컴파일러는 ordinal과 배열 인덱스의 관계를 알 도리가 없다.
* Phase나 Phase.Transition 열거 타입을 수정하면서 상전이 표 TRANSITIONS를 함께 수정하지 않거나 실수로 잘못 수정하면 런타임 오류가 날 것이다. (ArrayIndexOutOfBoundsException, NullPointerException 등)
* 상전이 표의 크기는 상태의 가짓수가 늘어나면 제곱해서 커지며 null로 채워지는 칸도 늘어난다.

##### 중첩 EnumMap으로 데이터와 열거 타입 쌍을 연결했다.

전이를 하나 얻으려면 이전 상태(from)와 이후 상태(to)가 필요하니, 맵 2개를 중첩하면 쉽게 해결할 수 있다.

~~~java
public enum Phase {
    SOLID, LIQUID, GAS;
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

        private final Phase from;
        private final Phase to;
        
        Transition(Phase from, Phase to) {
            this.from = from;
            this.to = to;
        }

        // 상전이 맵을 초기화한다.
        private static final Map<Phase, Map<Phase, Transition>>
                m = Stream.of(values()).collect(groupingBy(t -> t.from,
                () -> new EnumMap<>(Phase.class),
                toMap(t -> t.to, t -> t,
                        (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
        public static Transition from(Phase from, Phase to) {
            return m.get(from).get(to);
        }
    }
}
~~~

* 상전이 맵을 초기화하는 코드는 복잡하다.
* 이 맵의 타입인 Map\<Phase, Map\<Phase, Transition\>\> 은 "이전 상태에서 이후 상태에서 전이로의 맵에 대응시키는 맵" 이라는 뜻이다.
* 이러한 맵의 맵을 초기화하기 위해 수집기(java.util.stream.Collector) 2개를 차례로 사용했다.
  * 첫 번째 수집기인 groupingBy에서는 전이를 이전 상태를 기준으로 묶는다.
  * 두 번째 수집기의 병합 함수인 (x, y) -> y 는 선언만 하고 실제로는 쓰이지 않는데 이는 단지 EnumMap을 얻으려면 맵 팩터리가 필요하고 수집기들은 점층적 팩터리를 제공하기 때문이다.



##### 새로운 상태인 플라스마(PLASMA)를 추가해본다.

이 상태와 연결된 전이는 2개다. 

* 첫 번째는 기체에서 플라스마로 변하는 이온화(IONIZE)이고,
* 두 번째는 플라스마에서 기체로 변하는 탈 이혼화(DEIONIZE)이다.

~~~java
public enum Phase {
    // EnumMap 버전에 새로운 상태 추가하기
    SOLID, LIQUID, GAS, PLASMA;
        
    public enum Transition {
        MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
        BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
        SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID),
        IONIZE(GAS, PLASMA), DEIONIZE(PLASMA, GAS);
      
      ... // 나머지 코드는 동일
    }
}
~~~

기존 로직에서 잘 처리해주어 잘못 수정할 가능성이 극히 작다. 실제 내부에서는 맵들의 맵이 배열들의 배열로 구현되니 낭비되는 공간과 시간도 거의 없이 명확하고 안전하고 유지보수하기 쉽다.



##### 핵심 정리

> 배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라. 다차원 관계는 EnumMap\<..., EmumMap\<...\>\> 으로 표현하라. "애플리케이션 프로그래머는 Enum.ordinal을 (웬만해서는) 사용하지 말아야 한다"는 일반 원칙의 특수한 사례다.



___



## 아이템 38: 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

열거 타입은 거의 모든 상황에서 타입 안전 열거 패턴(typesafe enum pattern) 보다 우수하다. 단, 타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그럴수 없다. 타입 안전 열거 패턴은 열거한 값들을 그대로 가져온 다음 값을 추가하여 다른 목적으로 쓸 수 있는 반면, 열거 타입은 그렇게 할 수 없다.

확장할 수 있는 열거 타입이 어울리는 쓰임이 최소한 하나는 있다. 바로 연산 코드(operation code 혹은 opcode)다.
연산 코드의 각 원소는 특정 기계가 수행하는 연산을 뜻한다. 이따금 API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가 할 수 있도록 열여줘야 할 때가 있다.

##### 열거 타입으로 이 효과를 내는 멋진 방법

기본 아이디어는 열거 타입이 임의의 인터페이스를 구현할 수 있다는 사실을 이용하는 것이다.
연산 코드용 인터페이스를 정의하고 열거 타입이 이 인터페이스를 구현하게 하면 된다.
이때 열거 타입이 그 인터페이스의 표준 구현체 역할을 한다.

~~~java
// 인터페이스를 이용해 확장 가능 열거 타입을 흉내 냈다.
public interface Operation {
  double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
~~~

* 열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다.
* Operation을 구현한 또 다른 열거 타입을 정의해 기본 타입인 BasicOperation을 대체할 수 있다.

##### 확장 가능 열거 타입 - 지수 연산과 나머지 연산을 추가해보자

~~~java
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;
    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
    @Override public String toString() {
        return symbol;
    }
}
~~~

* 새로 작성한 연산은 기존 연산을 쓰던 곳이면 어디든 쓸 수 있다.
* Operation 인터페이스를 사용하도록 작성되어 있기만 하면된다.

##### 기본 열거 타입 대신 확장된 열거 타입을 넘겨 확장된 열거 타입의 원소 모두를 사용하는 코드

~~~java
// 열거 타입의 Class 객체를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);
}
    
private static <T extends Enum<T> & Operation> void test(
        Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
~~~

* main 메서드는 test 메서드에 ExtendedOperation의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다.
* class 리터럴은 한정적 타입 토큰 역할을 한다.
* opEnumType 매개변수의 선언\<T extends Enum\<T\> & Operation\> Class\<T\> 의 뜻은
  "Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다" 는 뜻이다.
  열거 타입이어야 원소를 순회할 수 있고, Operation이어야 원소가 뜻하는 연산을 수행할 수 있기 때문이다.



##### Class 객체 대신 한정적 와일드 카드 타입인 Collection\<? extends Operation\> 을 넘기는 방법

~~~java
// 컬렉션 인스턴스를 이용해 확장된 열거 타입의 모든 원소를 사용하는 예
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet,
                         double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
~~~

* 조금 더 덜 복잡하고 test 메서드가 살작 더 유연해졌다.
* 여러 구현 타입의 연산을 조합해 호출할 수 있게 되었다.
* 반면, 특정 연산에서 EnumSet과 EnumMap을 사용하지 못한다.



##### 인터페이스를 이용해 확장 가능한 열거 타입을 흉내 내는 방식에도 한 가지 문제가 있다.

열거 타입끼리 구현을 상속할 수 없다는 점이다. 아무  상태에도 의존하지 않는 경우 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있다.

반면, Operation 예는 연산 기호를 저장하고 찾는 로직이 BasicOperation과 ExtendedOperation 모두에 들어가야 한다. 이 경우에는 중복량이 적으니 문제되진 않지만, 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하는 방식으로 코드 방식을 없앨 수 있다.



##### 핵심 정리

> 열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다. 이렇게 하면 클라이언트는 이 인터페이스를 구현해 자신만의 열거 타입(혹은 다른 타입)을 만들 수 있다. 그리고 API가 (기본 열거 타입을 직접 명시하지 않고) 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.



___



## 아이템 39: 명명 패턴보다 애너테이션을 사용하라

전통적으로 도구나 프레임워크가 특별히 다뤄야 할 프로그램 요소에는 딱 구분되는 명명 패턴을 적용해왔다.

 Junit 3 에서 테스트 메서드 이름을 test로 시작해야 했다.

효과적인 방법이지만 단점도 크다.

첫 번째, 오타가 나면 안된다. test가 아닌 실수로 tset로 치면 이 메서드를 무시하고 지나가 통과했다고 오해할 수 있다.

두 번째, 올바른 프로그램 요소에서만 사용하리라 보증할 방법이 없다는 것이다. 메서드가 아닌 클래스 이름을 TestSafetyMechanisms라고 지어도 테스트 메서드들은 동작하지 않는다.

세 번째, 프로그램 요소를 매개변수로 전달할 마땅한 방법이 없다는 것이다. 특정 예외를 던져야 성공하는 테스트가 있다. 기대하는 예외 타입을 테스트에 매개변수로 전달해야 하는 상황이다. 예외의 이름을 테스트 메서드 이름에 덧붙이는 방법도 있지만, 깨지기 쉽다. 컴파일러는 메서드 이름에 덧붙인 문자열이 예외를 가리키는지 알 도리가 없다.

##### 애너테이션은 위의 문제들을 해결한다!

Test 라는 이름의 애너테이션을 정의한다.

~~~java
/**
 * 테스트 메서드임을 선언하는 애너테이션이다.
 * 매개변수 없는 정적 메서드 전용이다.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
~~~

* @Test 애너테이션 타입 선언 자체에도 두 가지의 다른 애너테이션이 달려있다. 바로 @Retention과 @Target 이다. 

* 이 처럼 다른 애너테이션 선언에 다는 애너테이션을 메타애너테이션(meta-annotation) 이라 한다.

* @Retention(RetentionPolicy.RUNTIME) 은 @Test가 런타임에도 유지되어야 한다는 표시이다.

* @Target(ElementType.METHOD) 은 @Test가 반드시 메서드 선언에만 사용돼야 함을 알려준다.

* 코드의 주석에는 "매개변수 없는 정적 메서드 전용이다" 라고 쓰여있다. 이 제약을 컴파일러가 강제할 수 있으면 좋겠지만 그렇게 하려면 적절한 애너테이션 처리기를 직접 구현해야 한다. 
  애너테이션 처리기 없이 인스턴스 메서드나 매개변수가 있는 메서드에 달면 컴파일은 잘 되겠지만, 테스트 도구를 실행할 때 문제가 된다.

##### @Test 애너테이션을 실제 적용한 모습

이와 같은 애너테이션을 "아무 매개변수 없이 단순히 대상에 마킹한다"는 뜻에서 마커(marker) 애너테이션이라 한다.

~~~java
public class Sample {
    @Test
    public static void m1() { }        // 성공해야 한다.
    public static void m2() { }
    @Test public static void m3() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m4() { }  // 테스트가 아니다.
    @Test public void m5() { }   // 잘못 사용한 예: 정적 메서드가 아니다.
    public static void m6() { }
    @Test public static void m7() {    // 실패해야 한다.
        throw new RuntimeException("실패");
    }
    public static void m8() { }
}
~~~

* Sample 클래스에는 정적 메서드가 7개이고, 4개에 @Test를 달았다.
* m3, m7은 예외를 던지고 m1, m5는 그렇지 않다.
* m5는 인스턴스 메서드이므로 @Test를 잘못 사용한 경우이다.
* 요약하면 1개는 성공, 2개는 실패, 1개는 잘못 사용되었다. @Test를 붙이지 않은 메서드는 무시한다.
* @Test 애너테이션은 Sample 클래스의 의미에 직접적인 영향을 주지는 않고 프로그램에게 추가 정보를 제공할 뿐이다.

##### 마커 애너테이션을 처리하는 프로그램

~~~java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
~~~

* 이 테스트 러너는 명령줄로부터 완전 정규화된 클래스 이름을 받아, 그 클래스에서 @Test 애너테이션이 달린 메서드를 차례로 호출한다.
* isAnnotationPresent가 실행할 메서드를 찾아주는 메서드다.
* 테스트 메서드가 예외를 던지면 리플렉션 메커니즘이 InvocationTargetException으로 감싸서 다시 던진다.
* 그래서 이 프로그램은 InvocationTargetException을 잡아 원래 예외에 담긴 실패 정보를 추출해 출력한다.
* InvocationTargetException 외의 예외가 발생한다면 @Test 애너테이션을 잘못 사용한 것이다.
* 두 번째 catch 블록은 잘못 사용해서 발생한 예외를 붙잡아 적절한 오류 메시지를 출력한다.



##### 특정 예외를 던져야만 성공하는 테스트 - 매개변수 하나를 받는 애너테이션 타입

~~~java
import java.lang.annotation.*;

/**
 * 명시한 예외를 던져야만 성공하는 테스트 메서드용 애너테이션
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
~~~

* 이 애너테이션의 매개변수 타입은 Class\<? extends Throwable\> 이다. 
* "Throwable을 확장한 클래스의 Class 객체" 라는 뜻이며, 모든 예외 타입을 다 수용한다.

##### 매개변수 하나짜리 애너테이션을 사용한 프로그램

~~~java
public class Sample2 {
    @ExceptionTest(ArithmeticException.class)
    public static void m1() {  // 성공해야 한다.
        int i = 0;
        i = i / i;
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m2() {  // 실패해야 한다. (다른 예외 발생)
        int[] a = new int[0];
        int i = a[1];
    }
    @ExceptionTest(ArithmeticException.class)
    public static void m3() { }  // 실패해야 한다. (예외가 발생하지 않음)
}
~~~

앞의 RunTests 클래스를 수정한다.

~~~java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }

            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (InvocationTargetException wrappedEx) {
                    Throwable exc = wrappedEx.getCause();
                    Class<? extends Throwable> excType =
                            m.getAnnotation(ExceptionTest.class).value();
                    if (excType.isInstance(exc)) {
                        passed++;
                    } else {
                        System.out.printf(
                                "테스트 %s 실패: 기대한 예외 %s, 발생한 예외 %s%n",
                                m, excType.getName(), exc);
                    }
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @ExceptionTest: " + m);
                }
            }
        }

        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
~~~

* 기존 코드와 한 가지 차이라면, 이 코드는 애너테이션 매개변수의 값을 추출하여 테스트 메서드가 올바른 예외를 던지는지 확인하는데 사용한다.
* 형변환 코드가 없으니 ClassCastException 걱정은 없다.
* 테스트 프로그램이 문제없이 컴파일되면 애너테이션 매개변수가 가리키는 예외가 올바른 타입이라는 뜻이다.
* 단, 해당 예외의 클래스 파일이 컴파일타임에는 존재했으나 런타임에는 존재하지 않을 수 있다. 이런 경우라면 테스트 러너가 TypeNotPresentException을 던질 것이다.

예외를 여러 개 명시하고 그중 하나가 발생하면 성공하게 만들 수도 있다. 애너테이션 메커니즘에는 이런 쓰임에 아주 유용한 기능이 기본으로 있다. @ExceptionTest 애너테이션의 매개변수 타입을 Class 객체의 배열로 수정해보자.

##### 배열 매개변수를 받는 애너테이션 타입

~~~java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Exception>[] value();
}
~~~

* 배열 매개변수를 받는 애너테이션용 문법은 아주 유연하다.
* 단일 원소 배열에 최적화했지만, 앞서의 @ExceptionTest들도 모두 수정없이 수용한다.
* 원소가 여럿인 배열을 지정할 때는 다음과 같이 원소들을 중괄호로 감싸고 쉼표로 구분해주기만 하면 된다.

~~~java
@ExceptionTest({ IndexOutOfBoundsException.class,
                 NullPointerException.class })
public static void doublyBad() {   // 성공해야 한다.
    List<String> list = new ArrayList<>();

    // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
    // NullPointerException을 던질 수 있다.
    list.addAll(5, null);
}
~~~

##### 새로운 @ExceptionTest를 지원하도록 테스트 러너를 수정

~~~java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }

            // 배열 매개변수를 받는 애너테이션을 처리하는 코드 (243쪽)
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    Class<? extends Throwable>[] excTypes =
                            m.getAnnotation(ExceptionTest.class).value();
                    for (Class<? extends Throwable> excType : excTypes) {
                        if (excType.isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
~~~



자바 8에서는 여러 개의 값을 받는 애너테이션을 다른 방식으로도 만들 수 있다. 

* 배열 매개변수를 사용하는 대신 애너테이션에 @Repeatable 메타애너테이션을 다는 방식이다 
* @Repeatable을 단 애너테이션은 하나의 프로그램 요소에 여러번 달 수 있다.
* 단 주의할 점이 있다.
  * 첫 번째, @Repeatable을 단 애너테이션을 반환하는 컨테이너 애너테이션을 더 정의하고 @Repeatable에 이 컨테이너 애너테이션의 class 객체를 매개변수로 전달해야 한다.
  * 두 번째, 컨테이너 애너테이션은 내부 애너테이션 타입의 배열을 반환하는 value 메서드를 정의해야 한다.
  * 마지막으로, 컨테이너 애너테이션 타입에는 적절한 보존 정책(@Retention)과 적용 대상(@Target)을 명시해야 한다.

~~~java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

// 반복 가능한 애너테이션의 컨테이너 애너테이션
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
~~~

##### 배열 방식 대신 반복 가능 애너테이션 적용

~~~java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {
    List<String> list = new ArrayList<>();

    // 자바 API 명세에 따르면 다음 메서드는 IndexOutOfBoundsException이나
    // NullPointerException을 던질 수 있다.
    list.addAll(5, null);
}
~~~

* 반복 가능 애너테이션을 여러 개 달면 하나만 달랐을 때와 구분하기 위해 해당 컨테이너 애너테이션 타입이 적용된다.
* getAnnotationType 메서드는 이 둘을 구분하지 않아서 반복 가능 애너테이션과 그 컨테이너 애너테이션을 모두 가져오지만 isAnnotationPresent 메서드는 둘을 명확히 구분한다.
* 반복 가능 애너테이션을 여러 번 단 다음 isAnnotationPresent로 반복 가능 애너테이션이 달렸는지 검사한다면 "그렇지 않다" 라고 알려준다.
* 그 결과 애너테이션을 여러 번 단 메서드들을 무시하고 지나친다.

##### 반복 가능 애너테이션 다루기

~~~java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class testClass = Class.forName(args[0]);
        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (InvocationTargetException wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    System.out.println(m + " 실패: " + exc);
                } catch (Exception exc) {
                    System.out.println("잘못 사용한 @Test: " + m);
                }
            }

            // 반복 가능 애너테이션 다루기
            if (m.isAnnotationPresent(ExceptionTest.class)
                    || m.isAnnotationPresent(ExceptionTestContainer.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 %s 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable wrappedExc) {
                    Throwable exc = wrappedExc.getCause();
                    int oldPassed = passed;
                    ExceptionTest[] excTests =
                            m.getAnnotationsByType(ExceptionTest.class);
                    for (ExceptionTest excTest : excTests) {
                        if (excTest.value().isInstance(exc)) {
                            passed++;
                            break;
                        }
                    }
                    if (passed == oldPassed)
                        System.out.printf("테스트 %s 실패: %s %n", m, exc);
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
~~~

* 반복 가능 애너테이션을 사용해 하나의 프로그램 요소에 같은 애너테이션을 여러 번 달 때의 코드 가독성을 높여보았다.
* 가독성을 개선할 수 있다면 이 방식을 사용한다.
* 하지만 애너테이션을 선언하고 처리하는 부분에서 코드 양이 늘어나며, 특히 처리 코드가 복잡해져 오류가 날 가능성이 커짐을 명심하자.



##### 애너테이션으로 할 수 있는 일을 명명 패턴으로 처리할 이유는 없다.

일반 프로그래머가 애너테이션 타입을 직접 정의할 일은 거의 없다. 

자바 프로그래머라면 예외 없이 자바가 제공하는 애너테이션 타입들은 사용해야한다.



___



## 아이템 40: @Override 애너테이션을 일관되게 사용하라

자바가 기본으로 제공하는 애너테이션 중 보통의 프로그래머에게 가장 중요한 것은 @Override 일 것이다. @Override는 메서드 선언에만 달 수 있으면, 이 애너테이션이 달렸다는 것은 상위 타입의 메서드를 재정의 했음을 뜻한다.

이 애너테이션을 일관되게 사용하면 여러 가지 악명 높은 버그들을 예방해준다.

##### 예제 - 영어 알파벳 2개로 구성된 문자열을 표현하는 클래스, 버그를 찾아보자

~~~java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
~~~

* main 메서드를 보면 똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 추가한 다음, 그 집합의 크기를 출력한다.
* Set은 중복을 허용하지 않으니 26이 출력될 것 같지만, 실제로는 260이 출력된다.
* 확실히 Bigram 작성자는 equals 메서드를 재정의한 것으로 보이고 hashCode도 함께 재정의 했다.
* 그러나 equals를 **재정의(overriding)** 한 것이 아니라 **다중 정의(overloading)** 해버렸다.
* Object의 equals는 매개변수 타입을 Object로 해야하는데 그러지 않았다.
* 이 결과로 Object의 equals를 재정의 한 것이 아니라 별개의 equals를 새로 정의한 꼴이 되었다.

##### 재정의 한다는 의도를 명시하라 - @Override

~~~java
@Override public boolean equals(Object o) {
    if (!(o instanceof Bigram2))
        return false;
    Bigram2 b = (Bigram2) o;
    return b.first == first && b.second == second;
}
~~~

@Override 애너테이션을 붙이고 매개변수를 Object로 수정한다.



#### <span style="color:blue">상위 클래스의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자.</span>

예외는 단 한가지로 구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 때는 굳이 달지 않아도 된다.



##### 핵심 정리

> 재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 여러분이 실수했을 때 컴파일러가 바로 알려줄 것이다. 예외는 한 가지뿐이다. 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우에는 이 애너테이션을 달지 않아도 된다(단다고 해서 해로울 것도 없다).



___



## 아이템 41: 정의하려는 것이 타입이라면 마커 인터페이스를 사용하라

아무 메서드도 담고 있지 않고, 단지 자신을 구현하는 클래스가 특정 속성을 가짐을 표시해주는 인터페이스를 마커 인터페이스(marker interface)라 한다.

ex) Serializable 은 자신을 구현한 클래스의 인스턴스는 ObjectOutputStream을 통해 쓸(Write) 수 있다고, 즉 직렬화 할 수 있다고 알려준다.

##### 마커 인터페이스가 마커 애너테이션보다 나은점 두 가지

1. 마커 인터페이스는 이를 구현한 클래스의 인스턴트들을 구분하는 타입으로 쓸 수 있으나, 마커 애너테이션은 그렇지 않다.

   마커 인터페이스는 어엿한 타입이기 때문에, 마커 애너테이션을 사용했다면 런타임에 발견될 오류를 컴파일 타임에 잡을 수 있다.

   ex) 자바의 직렬화는 Serializable 마커 인터페이스를 보고 대상이 직렬화 할 수 있는 대상인지 확인한다.

2. 적용 대상을 더 정밀하게 지정할 수 있다.

   적용 대상(@Target)을 ElementType.TYPE으로 선언한 애너테이션은 모든 타입에 달 수 있다. 부착을 할 수 있는 타입을 더 세밀하게 제한하지는 못한다는 뜻이다.

   ex) 특정 인터페이스를 구현한 클래스에만 적용하고 싶은 마커가 있다고 해보자. 이 마커를 인터페이스로 정의했다면 그냥 마킹하고 싶은 클래스에서만 그 인터페이스를 구현하면 된다. 그러면 마킹된 타입은 자동으로 그 인터페이스의 하위 타입임이 보장되는 것이다.



##### 마커 애너테이션이 마커 인터페이스보다 나은 점으로는 거대한 애너테이션 시스템의 지원을 받는다는 점이다.

애너테이션을 적극 활용하는 프레임워크에서는 마커 애너테이션을 쓰는 쪽이 일관성을 지키는데 유리할 것이다.



##### 마커 애너테이션과 마커 인터페이스를 써야할 때를 구분하는 법

확실한 것은 클래스와 인터페이스 외의 프로그램 요소(모듈, 패키지, 필드, 지역변수)에 마킹해야 할 때 애너테이션을 쓸 수 밖에 없다. 클래스와 인터페이스만이 인터페이스를 구현하거나 확장할 수 있기 때문이다.

마커를 클래스나 인터페이스에 적용해야 한다면 "이 마킹이 된 객체를 매개변수로 받는 메서드를 작성할 일이 있을까?" 자문해보자. "그렇다" 이면 마커 인터페이스를 써야한다. 이렇게 하면 그 마커 인터페이스를 해당 메서드의 매개변수 타입으로 사용하여 컴파일타임에 오류를 잡아낼 수 있다.

추가로, 애너테이션을 활발히 활용하는 프레임워크에서 사용하려는 마커라면 마커 애너테이션을 사용하는 편이 좋을 것이다.



##### 핵심 정리

> 마커 인터페이스와 마커 애너테이션은 각자의 쓰임이 있다. 새로 추가하는 메서드 없이 단지 타입 정의가 목적이라면 마커 인터페이스를 선택하자. 클래스나 인터페이스 외의 프로그램 요소에 마킹해야 하거나, 애너테이션을 적극 활용하는 프레임워크의 일부로 그 마커를 편입시키고자 한다면 마커 애너테이션이 올바른 선택이다. **적용 대상이 ElementType.TYPE인 마커 애너테이션을 작성하고 있다면, 잠시 여유를 갖고 정말 애너테이션으로 구현하는 게 옳은지, 혹은 마커 인터페이스가 낫지 않을지 곰곰히 생각해보자.**



___

6장 열거 타입과 애너테이션 끝...
