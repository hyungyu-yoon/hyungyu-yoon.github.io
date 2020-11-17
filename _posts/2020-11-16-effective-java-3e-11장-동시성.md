---
title: effective java 3/E - 11장. 동시성
tags: 
 - Java
key: 16

---



# 11장 동시성

##### 11장에서 배울 내용

* 스레드는 여러 활동을 동시에 수행할 수 있게 해준다.

* 하지만 동시성 프로그래밍은 단일 스레드 프로그래밍보다 어렵다.

* 동시성 프로그램을 명확하고 정확하게 만들고 문서화하는 데 도움이 되는 조언들을 담았다.

  

___



## 아이템 78: 공유 중인 가변 데이터는 동기화해 사용하라

**synchronized 키워드**는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다.

##### 동기화의 용도

1. 배타적 실행, 즉 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도
   * 한 객체가 일관된 상태를 가지고 생성되고, 이 객체를 접근하는 메서드는 그 객체에 락을 건다.
   * 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정한다.
2. 동기화는 일관성이 깨진 상태를 볼 수 없게하는 것은 물론, 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.

##### 원자적(atomic)

* long과 double 이외 변수를 읽고 쓰는 동작은 원자적(atomic)이다.

* 여러 스레드가 같은 변수를 동기화 없이 수정하는 중이라도, 항상 어떤 스레드가 정상적으로 저장한 값을 온전히 읽어옴을 보장한다는 뜻이다.

##### 동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.

성능을 높이려면 원자적 데이터를 읽고 쓸 때는 동기화하지 말아야 겠다는 생각은 위험한 발상이다.

##### 공유 중인 가변 데이터를 비록 원자적으로 읽고 쓸 수 있을지라도 동기화에 실패하면 처참한 결과로 이어질 수 있다.

* 다른 스레드를 멈추는 작업을 할  Thread.stop은 사용하지 말자.

##### 다른 스레드를 멈추는 올바른 방법

* 첫 번째 스레드는 자신의 boolean 필드를 폴링하면서 그 값이 true가 되면 멈춘다.

* 이 필드를 false 초기화해놓고, 다른 스레드에서 이 스레드를 멈추고자 할 때 true로 변경하는 식이다.

* ##### 잘못된 코드 - 이 프로그램은 얼마나 오래 실행될까?

  ~~~java
  public class StopThread {
      private static boolean stopRequested;
  
      public static void main(String[] args)
              throws InterruptedException {
          Thread backgroundThread = new Thread(() -> {
              int i = 0;
              while (!stopRequested)
                  i++;
          });
          backgroundThread.start();
  
          TimeUnit.SECONDS.sleep(1);
          stopRequested = true;
      }
  }
  ~~~

  이 코드는 동기화하지 않아 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보증할 수 없다.

* ##### 적절히 동기화해 스레드가 정상 종료한다.

  ~~~java
  public class StopThread {
      private static boolean stopRequested;
  
      private static synchronized void requestStop() {
          stopRequested = true;
      }
  
      private static synchronized boolean stopRequested() {
          return stopRequested;
      }
  
      public static void main(String[] args)
              throws InterruptedException {
          Thread backgroundThread = new Thread(() -> {
              int i = 0;
              while (!stopRequested())
                  i++;
          });
          backgroundThread.start();
  
          TimeUnit.SECONDS.sleep(1);
          requestStop();
      }
  }  
  ~~~

  * 쓰기 메서드(requestStop)와 읽기 메서드(stopRequested) 모두를 동기화 했다.

  * ##### 쓰기와 읽기 모두가 동기화 되지 않으면 동작을 보장하지 않는다.

* 반복문에서 매번 동기화하는 비용이 크지 않지만 속도가 더 빠른 대안
  stopRequested 필드를 volatile 으로 선언하면 동기화를 생략해도 된다. volatile 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게됨을 보장한다.

  ##### volatile 필드를 사용해 스레드가 정상 종료한다.

  ~~~java
  public class StopThread {
      private static volatile boolean stopRequested;
  
      public static void main(String[] args)
              throws InterruptedException {
          Thread backgroundThread = new Thread(() -> {
              int i = 0;
              while (!stopRequested)
                  i++;
          });
          backgroundThread.start();
  
          TimeUnit.SECONDS.sleep(1);
          stopRequested = true;
      }
  }
  ~~~

  * 하지만 이 경우라도 동기화를 적용하지 않았을 때 동작이 제대로 하지 않을 수 있다.

* java.util.concurrent.atomic 패키지의 AtomicLong 을 사용하는 방법도 있다.
  이 패키지에는 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 있다.

  ~~~java
  private static final AtomicLong nextSerialNum = new AtomicLong();
  
  public static long generateSerialNumber() {
      return nextSerialNum.getAndIncrement();
  }
  ~~~

##### 문제들을 피하는 가장 좋은 방법은 물론 애초에 가변 데이터를 공유하지 않는 것이다.

* 불변 데이터만 공유하거나 아무것도 공유하지 말자.
* 가변 데이터는 단일 스레드에서만 쓰도록 하자.
* 이 정책을 받아들였다면 그 사실을 문서에 남겨 유지보수 과정에서도 정책이 계속 지켜지도록 하는 게 중요하다.
* 또한, 사용하려는 프레임워크와 라이브러리를 깊이 이해하는 것도 중요하다.

##### 한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다.

* 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 값을 읽어갈 수 있다.
* 다른 스레드에 이런 객체를 건네는 행위를 안전 발행(safe publication)이라 한다.
* 클래스를 초기화 하는 과정에서 객체를 정적 필드, volatile 필드, final 필드, 혹은 보통의 락을 통해 접근하는 필드에 저장해도 된다.



##### 핵심 정리

> **여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반디스 동기화 해야 한다.** 동기화하지 않으면 한 스레드가 수행한 변경을 다른 스레드가 보지 못할 수도 있다. 공유되는 가변 데이터를 동기화하는 데 실패하면 응답 불가 상태에 빠지거나 안전 실패로 이어질 수 있다. 이는 디버깅 난이도가 가장 높은 문제에 속한다. 간헐적이거나 특정 타이밍에만 발생할 수도 있고, VM에 따라 현상이 달라지기도 한다. 배타적 실행은 필요 없고 스레드끼리의 통신만 필요하다면  volatile 한정자만으로 동기화할 수 있다. 다만 올바로 사용하기가 까다롭다.



___



## 아이템 79: 과도한 동기화는 피하라

과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 심지어 예측할 수 없는 동작을 낳기도 한다.

##### 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안 된다.

* 동기화된 영역 안에서 재정의할 수 있는 메서드를 호출하면 안 되며, 클라이언트가 넘겨준 함수 객체를 호출해서도 안 된다.
* 그 메서드가 무슨일을 할지 알지 못하며 통제할 수도 없다. 이러한 메서드를 외계인 메서드라 한다.
* 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손할 수도 있다.

##### 잘못된 코드 예제

어떤 집합(Set)을 감싼 래퍼 클래스이고, 이 클래스의 클라이언트는 집합에 원소가 추가되면 알림을 받을 수 있다.(관찰자 패턴)

~~~java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }

    private final List<SetObserver<E>> observers
            = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for (SetObserver<E> observer : observers)
                observer.added(this, element);
        }
    }

    @Override public boolean add(E element) {
        boolean added = super.add(element);
        if (added)
            notifyElementAdded(element);
        return added;
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c)
            result |= add(element);  // notifyElementAdded를 호출한다.
        return result;
    }
}
~~~

* 관찰자들은 addObserver와 removeObserver 메서드를 호출해 구독을 신청하거나 해지한다.
  두 경우 모두 다음 콜백 인터페이스의 인스턴스를 메서드에 건넨다.

  ~~~java
  @FunctionalInterface public interface SetObserver<E> {
      void added(ObservableSet<E> set, E element);
  }
  ~~~

  * 이 인터페이스는 구조적으로 BiConsumer\<ObservableSet\<E\>, E\> 와 같다.

* 눈으로 보기에 ObservableSet은 잘 동작할 것 같다.

* ##### 0~ 99까지를 출력하는 프로그램

  ~~~java
  public class Test1 {
      public static void main(String[] args) {
          ObservableSet<Integer> set =
                  new ObservableSet<>(new HashSet<>());
  
          set.addObserver((s, e) -> System.out.println(e));
  
          for (int i = 0; i < 100; i++)
              set.add(i);
      }
  }
  ~~~

  ##### 평상시에는 앞서와 같이 집합에 추가된 정숫값을 출력하다가 값이 23이면 자기 자신을 제거하는 관찰자를 추가

  ~~~java
  set.addObserver(new SetObserver<>) {
      public void added(ObservableSet<Integer> s, Integer e) {
          System.out.println(e);
          if(e == 23)
              s.removeObserver(this);
      }
  }
  ~~~

  * 0부터 23까지 출력한 후 관찰자 자신을 구독해지한 다음 조용히 조용할 것이다. 그런데 실제로 실행하면 그렇게 되지 않는다.
  * 23까지 출력한 다음 ConcurrentModificationException을 던진다.
  * 관찰자의 added 메서드 호출이 일어난 시점이 notifyElementAdded가 관찰자들의 리스트를 순회하는 도중이기 때문이다.
  * added 메서드는 ObservableSet의 removeObserver를 호출하고, 이 메서드는 다시 observers.remove를 호출한다.
  * 여기서 문제가 발생하는데, 리스트에서 원소를 제거하려 하는데, 마침 지금은 이 리스트를 순회하는 도중이다. 즉 허용되지 않는 동작이다.
  * notifyElementAdded 메서드에서 수행하는 순회는 동기화 블록 안에 있으므로 동시 수정이 일어나지 않도록 보장하지만, 정작 자신이 콜백을 거쳐 되돌아와 수정하는 것까지 막지는 못한다.

  ##### 다른 이상한 시도 - 구독해지를 하는 관찰자를 작성하는데, removeObserver를 직접 호출하지 않고 실행자 서비스(ExecutorService)를 사용해 다른 스레드한테 부탁할 것이다.

  ~~~java
  // 쓸데없이 백그라운드 스레드를 사용하는 관찰자
  set.addObserver(new SetObserver<>() {
      public void added(ObservableSet<Integer> s, Integer e) {
          System.out.println(e);
          if (e == 23) {
              ExecutorService exec =
                      Executors.newSingleThreadExecutor();
              try {
                  exec.submit(() -> s.removeObserver(this)).get();
              } catch (ExecutionException | InterruptedException ex) {
                  throw new AssertionError(ex);
              } finally {
                  exec.shutdown();
              }
          }
      }
  });
  ~~~

  * 이 프로그램은 예외는 나지 않지만 교착상태에 빠진다.
  * 백그라운드 스레드가 s.removeObserver를 호출하면 관찰자를 잠그려 시도하지만 락을 얻을 수 없다.
  * 메인 스레드가 이미 락을 쥐고 있기 때문이다.
  * 그와 동시에 메인 스레드는 백그라운드 스레드가 관찰자를 제거하기만을 기다리는 중이다. 바로 교착상태이다.

  ##### 똑같은 상황이지만 불변식이 임시로 깨진 경우라면 어떻게 될까?

  * 자바 언어의 락은 재진입을 허용하므로 교착상태에 빠지지는 않는다.
  * 예외를 발생시킨 첫 번째 예에서라면 외계인 메서드를 호출하는 스레드는 이미 락을 쥐고 있으므로 다음번 락 획득도 성공한다.
  * 그 락이 보호하는 데이터에 대해 개념적으로 관련이 없는 다른 작업을 진행중인데도 말이다.
  * 문제의 원인은 락이 제 구실을 하지 못했기 때문이다.
  * 재진입 가능 락은 객체 지향 멀티스레드 프로그램을 쉽게 구현할 수 있도록 해주지만, 응답불가(교착상태)가 될 상황을 안전 실패(데이터 훼손)로 변모시킬 수도 있다.

  ##### 다행히 어렵지 않게 해결할 수 있다. - 외계인 메서드를 동기화 블록 바깥으로 옮겼다.

  ~~~java
  private void notifyElementAdded(E element) {
      List<SetObserver<E>> snapshot = null;
      synchronized(observers) {
          snapshot = new ArrayList<>(observers);
      }
      for (SetObserver<E> observer : snapshot)
          observer.added(this, element);
  }
  ~~~

  * 앞서의 두 예제에서 발생한 예외 발생과 교착상태 증상이 사라진다.

  ##### 외계인 메서드를 바깥으로 옮기는 것보다 더 나은 방법 

  자바의 동시성 컬렉션 라이브러리의 CopyOnWriteArrayList가 정확히 이 목적으로 특별히 설계된 것이다.
  내부를 변경하는 작업은 항상 깨끗한 복사본을 만들어 수행하도록 구현했다.

  ~~~java
  private final List<SetObserver<E>> observers =
          new CopyOnWriteArrayList<>();
  
  public void addObserver(SetObserver<E> observer) {
      observers.add(observer);
  }
  
  public boolean removeObserver(SetObserver<E> observer) {
      return observers.remove(observer);
  }
  
  private void notifyElementAdded(E element) {
      for (SetObserver<E> observer : observers)
          observer.added(this, element);
  }
  ~~~

  * 동기화 영역 바깥에서 호출되는 외계인 메서드를 열린 호출(open call)이라 한다. 열린 호출은 실패 방지 효과 외에도 동시성 효율을 크게 개선해준다.

* ##### 기본 규칙은 동기화 영역에서는 가능한 한 일을 적게 하는 것이다.

  * 락을 얻고, 공유 데이터를 검사하고 필요하면 수정하고 락을 놓는다. 
  * 오래 걸리는 작업이라면 아이템 78의 지침을 어기지 않으면서 동기화 영역 바깥으로 옮기는 방법을 찾아보자.

##### 성능 측면

* 자바의 동기화 비용은 빠르게 낮아져 왔지만, 과도한 동기화를 피하는 일은 오히려 과거 어느 때보다 중요하다.

가변 클래스를 작성하려거든 다음 두 선택지 중 하나를 따르자.

1. 동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야 하는 클래스가 외부에서 알아서 동기화하게 하자.
2. 동기화를 내부에서 수행해 스레드 안전한 클래스를 만들자. 
   단, 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 두 번째 방법을 선택해야 한다.

java util은 첫 번째 방식을 취했고, java.util.concurrent는 두 번째 방식을 취했다.

* StringBuffer의 인스턴스는 거의 항상 단일 스레드로 쓰였음에도 내부적으로 동기화를 수행했다.

  뒤늦게 StringBuilder가 등장하였다.(동기화 되지 않은 StringBuffer다.)

* java.util.Random은 동기화 되지 않은 버전인 java.util.concurrent.ThreadLocalRandom으로 대체되었다.

##### 

##### 핵심 정리

> 교착상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지 말자. 일반화해 이야기하면, 동기화 영역 안에서의 작업은 최소한으로 줄이자. 가변 클래스를 설계할 때는 스스로 동기화해야 할지 고민하자. 멀티코어 세상인 지금은 과도한 동기화를 피하는 게 과거 어느 때보다 중요하다. 합당한 이유가 있을 때만 내부에서 동기화하고, 동기화했는지 여부를 문서에 명확히 밝히자.



___



## 아이템 80: 스레드보다는 실행자, 태스크, 스트림을 애용하라

java.util.concurrent 패키지는 실행자 프레임워크(Executor Framework)라고 하는 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다.

##### 작업 큐를 다음의 한 줄로 생성한다.

~~~java
ExecutorService exec = Executors.newSingleThreadExecutor();
~~~

실행자에 실행할 태스크(task; 작업)를 넘기는 방법

~~~java
exec.execute(runnable);
~~~

종료하는 방법

~~~java
exec.shutdown();
~~~

##### 실행자 서비스의 주요 기능

* 특정 태스크가 완료되기를 기다린다.
* 태스크 모음 중 아무것도 하나(invokeAny 메서드) 혹은 모든 태스크(invokeAll 메서드)가 완료되기를 기다린다.
* 실행자 서비스가 종료하기를 기다린다(awaitTermination 메서드).
* 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용).
* 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(ScheduledThreadPoolExecutor 이용).

##### 큐를 둘 이상의 스레드가 처리하게 하고 싶다면 간단히 다른 정적 팩터리를 이용하여 다른 종류의 실행자 서비스를 생성하면 된다.

* 스레드 개수는 고정할 수도 있고 필요에 따라 늘어나거나 줄어들게 설정할 수도 있다.
* java.util.concurrent.Executors의 정적 팩터리를 이용해 생성할 수 있다.
* 평범하지 않은 실행자를 원한다면 ThreadPoolExecutor 클래스를 직접 사용해도 된다.
  이 클래스로는 스레드 풀 동작을 결정하는 거의 모든 속성을 설정할 수 있다.

##### 실행자 서비스를 사용하기에 까다로운 애플리케이션도 있다.

* 작은 프로그램이나 가벼운 서버라면 Executors.newCachedThreadPool이 일반적으로 좋은 선택일 것이다.

* CachedThreadPool 은 무거운 프로덕션 서버에는 좋지 못하다.

  * CachedThreadPool 에서는 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임 돼 실행된다.

  * 가용한 스레드가 없다면 새로 하나를 생성한다.
  * 서버가 아주 무겁다면 CPU 이용률이 100%에 치닫고, 새로운 태스크가 도착하는 족족 또 다른 스레드를 생성하며 상황을 더욱 악화시킨다.
  * Executors.newFixedThreadPool을 선택하거나 ThreadPoolExecutor를 직접 사용하는 편이 낫다.

##### 작업 큐를 손수 만드는 일은 삼가야 하고, 스레드를 직접 다루는 것도 일반적으로 삼가야 한다.

* 스레드를 직접 다루면 Thread가 작업 단위와 수행 메커니즘 역할을 모두 수행하게 된다.
* 반면 실행자 프레임워크에서는 작업 단위와 실행 메커니즘이 분리된다.
* 작업 단위를 나타내는 핵심 추상 개념은 태스크다.
* 태스크에는 두가지가 있다.
  * Runnable과 Callable이다.
* 태스크를 수행하는 일반적인 메커니즘이 바로 실행자 서비스다.
* 태스크 수행을 실행자 서비스에 맡기면 원하는 태스크 수행 정책을 선택할 수 있고, 생각이 바뀌면 언제든 변경할 수 있다.
* 핵심은 실행자 프레임워크가 작업 수행을 담당해준다는 것이다.

##### 자바 7이 되면서 실행자 프레임워크는 포크-조인(fork-join) 태스크를 지원하도록 확장되었다.

* 포크-조인 태스크는 포크-조인 풀이라는 특별한 실행자 서비스가 실행해준다.
* ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있고, ForkJoinTask를 구성하는 스레드들이 이 태스크들을 처리하며, 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있다.
* 이렇게 하여 모든 스레드가 바쁘게 움직여 CPU를 최대한 활용하면서 높은 처리량과 낮은 지연시간을 달성한다.



___



## 아이템 81: wait와 notify보다는 동시성 유틸리티를 애용하라

고수준의 동시성 유틸리티가 이전이라면 wait와 notify로 하드코딩해야 했던 전형적인 일들을 대신 처리해준다. 

##### wait와 notify는 올바르게 사용하기가 아주 까다로우니 고수준 동시성 유틸리티를 사용하자.

##### java.util.concurrent의 고수준 유틸리티는 세 가지 범주로 나눌 수 있다.

* 실행자 프레임워크,
* 동시성 컬렉션(concurrent collection)
* 동기화 장치(synchronizer)

##### 동시성 컬렉션은 List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다

* 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다.

* ##### 동시성 컬렉션에서 동시성을 무력화하는 건 불가능하며, 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.

* 동시성 컬렉션에서 동시성을 무력화하지 못하므로 여러 메서드를 원자적으로 묶어 호출하는 일 역시 불가능하다.
  그래서 여러 기본 동작을 하나의 원자적 동작으로 묶는 '상태 의존적 수정' 메서드들이 추가되었다.

  ex) Map의 PutIfAbsent(key, value) 메서드는 주어진 키에 매핑된 값이 아직 없을 때 새 값을 집어 넣는다.
  그리고 기존 값이 있었다면 그 값을 반환하고, 없었다면 null을 반환한다.

  ##### ConcurrentMap으로 구현한 동시성 정규화 맵 - 최적은 아니다.

  ~~~java
  public class Intern {
      private static final ConcurrentMap<String, String> map =
              new ConcurrentHashMap<>();
  
      public static String intern(String s) {
          String previousValue = map.putIfAbsent(s, s);
          return previousValue == null ? s : previousValue;
      }
  
  }
  ~~~

  * ConcurrentHashMap은 get 같은 검색 기능에 최적화되었다. 따라서 get을 먼저 호출하여 필요할 때만 putIfAbsent를 호출하면 더 빠르다.

  ##### 더 빠른 예제

  ~~~java
  public static String intern(String s) {
      String result = map.get(s);
      if (result == null) {
          result = map.putIfAbsent(s, s);
          if (result == null)
              result = s;
      }
      return result;
  }
  ~~~

  * ConcurrentMap은 동시성이 뛰어나며 속도도 무척 빠르다.

* 컬렉션 인터페이스 중 일부는 작업이 성공적으로 완료될 때까지 기다리도록 확장되었다.

  * Queue를 확장한 BlockingQueue에 추가된 메서드 중 take는 큐의 첫 원소를 꺼낸다.
  * 이때 만약 큐가 비었다면 새로운 원소가 추가될 때까지 기다린다.
  * 이런 특성 덕에 작업 큐(생산자-소비자 큐)로 쓰기에 적합하다.
  * 작업 큐는 하나 이상의 생산자(producer) 스레드가 작업(work)을 큐에 추가하고, 하나 이상의 소비자(consumer) 스레드가 큐에 있는 작업을 꺼내 처리하는 방식이다.

##### 동기화 장치는 스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다.

* 가장 자주 쓰이는 동기화 장치는 CountDownLatch와 Semaphore다.

* CyclicBarrier와 Exchanger는 그보다 덜 쓰인다.

* 가장 강력한 동기화 장치는 바로 Phaser다.

* 카운트다운 래치는 일회성 장벽으로, 하나 잇강의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.

  CountDownLatch의 유일한 생성자는 int값을 받으며, 이 값이 래치의 countDown 메서드를 몇번 호출해야 대기 중인 스레드들을 깨우는지 결정한다.

  ##### 어떤 동작들을 동시에 시작해 모두 완료하기까지의 시간을 재는 간단한 프레임워크

  * 이 프레임워크는 메서드 하나로 구성되며, 이 메서드는 동작들을 실행할 실행자와 동작을 몇 개나 동시에 수행할 수 있는지를 뜻하는 동시성 수준(concurrency)을 매개변수로 받는다
  * 타이머 스레드가 시계를 시작하기 전에 모든 작업자 스레드는 동작을 수행할 준비를 마친다.
  * 마지막 작업자 스레드가 준비를 마치면 타이머 스레드가 시작 방아쇠를 당겨 작업자 스레드들이 일을 시작하게 한다.
  * 마지막 작업자 스레드가 동작을 마치자 마자 타이머 스레드는 시계를 멈춘다.

  ~~~java
  public class ConcurrentTimer {
      private ConcurrentTimer() { } // 인스턴스 생성 불가
  
      public static long time(Executor executor, int concurrency,
                              Runnable action) throws InterruptedException {
          CountDownLatch ready = new CountDownLatch(concurrency);
          CountDownLatch start = new CountDownLatch(1);
          CountDownLatch done  = new CountDownLatch(concurrency);
  
          for (int i = 0; i < concurrency; i++) {
              executor.execute(() -> {
                  ready.countDown(); // 타이머에게 준비를 마쳤음을 알린다.
                  try {
                      start.await(); // 모든 작업자 스레드가 준비될 때까지 기다린다.
                      action.run();
                  } catch (InterruptedException e) {
                      Thread.currentThread().interrupt();
                  } finally {
                      done.countDown();  // 타이머에게 작업을 마쳤음을 알린다.
                  }
              });
          }
  
          ready.await();     // 모든 작업자가 준비될 때까지 기다린다.
          long startNanos = System.nanoTime();
          start.countDown(); // 작업자들을 깨운다.
          done.await();      // 모든 작업자가 일을 끝마치기를 기다린다.
          return System.nanoTime() - startNanos;
      }
  }
  ~~~

  * 이 코드는 카운트다운 래치를 3개 사용한다.
  * ready 래치는 작업자 스레드들이 준비가 완료됐음을 타이머 스레드에 통지할 때 사용한다.
  * 통지를 끝낸 작업자 스레드들은 두 번째 래치인 start가 열리기를 기다린다
  * 마지막 작업자 스레드가 ready.countDown을 호출하면 타이머 스레드가 시작 시각을 기록하고 start.countDown을 호출하여 기다리던 작업자 스레드들을 깨운다.
  * 그 직후 타이머 스레드는 세 번째 래치인 done이 열리기를 기다린다.
  * done 래치는 마지막 남은 작업자 스레드가 동작을 마치고 done.countDown을 호출하면 열린다.
  * 타이머 스레드는 done 래치가 열리자마자 깨어나 종료 시각을 기록한다.
  * time 메서드에 넘겨진 실행자(executor)는 concurrency 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 한다. 그렇지 못하면 이 메서드는 결고 끝나지 않을 것이다.
  * 시간 간격을 잴 때는 항상 System.currentTimeMills가 아닌 System.nanoTime을 사용하자.
  * 이 예제의 코드는 작업에 충분한 시간이 걸리지 않는다면 정확한 시간을 측정할 수 없을 것이다.

##### 새로운 코드라면 wait와 notify가 아닌 동시성 유틸리티를 써야 할 것이다.

하지만 어쩔수 없이 레거시 코드를 다뤄야 할 때도 있을 것이다.

* wait 메서드는 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용한다.

  락 객체의 wait 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다.

  ##### wait 메서드를 사용하는 표준 방식

  ~~~java
  synchronized(obj) {
      while(<조건이 충족되지 않았다>)
          obj.wait();  // 락을 놓고, 깨어나면 다시 잡는다.
          
     ... // 조건이 충족됐을 때의 동작을 수행한다.
  }
  ~~~

  wait 메서드를 사용할 때는 반드시 대기 반복문(wait loop) 관용구를 사용하라. 반복문 밖에서는 절대로 호출하지 말자.

* 대기 전에 조건을 검사하여 이미 충족되었다면 wait를 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치다.

  만약 조건이 이미 충족되었는데 스레드가 notify 혹은 notifyAll 메서드를 먼저 호출한 후 대기 상태로 빠지면, 그 스레드를 다시 깨울 수 있다고 보장할 수 없다.

##### 조건이 만족되지 않아도 스레드가 깨어날 수 있는 상황

* 스레드가 notify를 호출한 다음 대기 중이었던 스레드가 깨어나는 사이에 다른 스레드가 락을 얻어 그 락이 보호하는 상태를 변경한다.
* 조건이 만족되지 않았음에도 다른 스레드가 실수로 혹은 악의적으로 notify를 호출한다. 공개된 객체를 락으로 사용해 대기하는 클래스는 이런 위험에 노출된다. 외부에 노출된 객체의 동기화된 메서드 안에서 호출하는 wait는 모두 이 문제에 영향을 받는다.
* 깨우는 스레드는 지나치게 관대해서, 대기 중인 스레드 중 일부만 조건이 충족되어도 notifyAll을 호출해 모든 스레드를 깨울 수도 있다.
* 대기 중인 스레드가 (드물게) notify 없이도 깨어나는 경우가 있다. 허위 각성(spurious wakeup) 이라는 현상이다.

##### notify와 notifyAll 중 무엇을 선택하느냐 하는 문제

* 일반적으로 언제나 notifyAll을 사용하라는 게 합리적이고 안전한 조언이 될 것이다.

* 깨어나야 하는 모든 스레드가 깨어남을 보장하니 항상 정확한 결과를 얻을 것이다.

* 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 때마다 단 하나의 스레드만 혜택을 받을 수 있다면 notify를 사용해 최적화할 수 있다.

* 하지만 이상의 전제조건들이 만족될지라도 notify 대신 notifyAll을 사용해야 하는 이유가 있다.

  외부에 공개된 객체에 대해 실수 혹은 악의적으로 notify를 호출하는 상황에 대비하기 위해 wait를 반복문 안에서 호출했듯, notify 대신 notifyAll을 사용하면 관련 없는 스레드가 실수 혹은 악의적으로 wait를 호출하는 공격으로 보호할 수 있다.



##### 핵심 정리

> wait와 notify를 직접 사용하는 것을 동시성 '어셈블리 언어'로 프로그래밍하는 것에 비유할 수 있다. 반면 java.util.concurrent는 고수준 언어에 비유할 수 있다. **코드를 새로 작성한다면 wait와 notify를 쓸 이유가 거의(어쩌면 전혀)없다.** 이들을 사용하는 레거시 코드를 유지보수해야 한다면 wait는 항상 표준 관용구에 따라 while문 안에서 호출하도록 하자. 일반적으로 notify보다는 notifyAll을 사용해야 한다. 혹시라도 notify를 사용한다면 응답 불가 상태에 빠지지 않도록 각별히 주의하자.



____



## 아이템 82: 스레드 안정성 수준을 문서화하라

한 메서드를 여러 스레드가 동시에 호출할 때 그 메서드가 어떻게 동작하느냐는 해당 클래스와 이를 사용하는 클라이언트 사이의 중요한 계약과 같다. 만약 API 문서에 아무런 언급도 없다면 클래스 사용자는 나름의 가정을 해야하고, 가정이 틀리면 클라이언트 프로그램은 동기화를 충분히 하지 못하거나 지나치게 한 상태 일 것이며, 두 경우 모두 심각한 오류로 이어질 수 있다.

##### API 문서에 synchronized 한정자가 보이는 메서드라도 스레드 안전하다고 믿기 어렵다.

* 메서드 선언에 syncronized 한정자를 선언할지는 구현 이슈일 뿐 API에 속하지 않는다.

* syncronized 유무로 스레드 안정성을 알 수 없다.
* 멀티스레드 환경에서도 API를 안전하게 사용하려면 클래스가 지원하는 스레드 안전성 수준을 정확히 명시해야 한다.

##### 스레드 안정성이 높은 순

* **불변(immutable):** 이 클래스의 인스턴스는 마치 상수와 같아서 외부 동기화도 필요 없다. String, Long, BigInteger가 대표적이다.
* **무조건적 스레드 안전(unconditionally thread-safe):** 이 클래스의 인스턴스는 수정될 수 있으나, 내부에서 충실히 동기화하여 별도의 외부 동기화 없이 동시에 서용해도 안전하다. AtomicLong, ConcurrentHashMap이 여기에 속한다.
* **조건부 스레드 안전(conditionally thread-safe):** 무조건적 스레드 안전과 같으나, 일부 메서드는 동시에 사용하려면 외부 동기화가 필요하다. Collections.syncronized 래퍼 메서드가 반환한 컬렉션들이 여기에 속한다.
* **스레드 안전하지 않음(not thread-safe):** 이 클래스의 인스턴스는 수정될 수 있다. 동시에 사용하려면 각각의 (혹은 일련의) 메서드 호출을 클라이언트가 선택한 외부 동기화 메커니즘으로 감싸야 한다. ArrayList, HashMap 같은 기본 컬렉션이 여기에 속한다.
* **스레드 적대적(thread-hostile):** 이 클래스는 모든 메서드 호출을 외부 동기화로 감싸더라도 멀티스레드 환경에서 안전하지 않다. 이 수준의 클래스는 일반적으로 정적 데이터를 아무 동기화 없이 수정한다. 이런 클래스를 고의로 만드는 사람은 없겠지만, 동시성을 고려하지 않고 작성하다 보면 우연히 만들어질 수 있다. 스레드 적대적으로 밝혀진 클래스나 메서드는 일반적으로 문제를 고쳐 재배포하거나 사용자제(deprecated) API로 지정한다. 

이 분류는 스레드 안정성 애너테이션 @Immutable, @ThreadSafe, @NotThreadSafe 와 대략 일치한다.

##### 조건부 스레드 안전한 클래스는 주의해서 문서화해야 한다.

* 어떤 순서로 호출할 때 외부 동기화가 필요한지
* 그리고 그 순서로 호출하려면 어떤 락 혹은 락들을 얻어야 하는지 알려줘야 한다.
* 일반적으로 인스턴스 자체를 락으로 얻지만 예외도 있다.
* 클래스의 스레드 안정성은 보통 클래스의 문서화 주석에 기재하지만, 독특한 특성의 메서드라면 해당 메서드의 주석에 기재하도록 하자.
* 열거 타입은 굳이 불변이라고 쓰지 않아도 된다.
* 반환 타입만으로는 명확히 알 수 없는 정적 팩터리라면 자신이 반환하는 객체의 스레드 안정성을 반드시 문서화해야 한다.
  Collections.sychronizedMap이 좋은 예다.

##### 클래스가 외부에서 사용할 수 있는 락을 제공하면 클라이언트에서 일련의 메서드 호출을 원자적으로 수행할 수 있다.

* 하지만 이 유연성에는 대가가 따른다.
* 내부에서 처리하는 고성능 동시성 제어 메커니즘과 혼용할 수 없게 되는 것이다.
* ConcurrentHashMap 같은 동시성 컬렉션과는 함께 사용하지 못한다.
* 클라이언트가 공개된 락을 오래 쥐고 놓지 않는 서비스 거부 공격(denial-of-servifce attack)을 수행할 수도 있다.

##### 서비스 거부 공격을 막는 방법 - 비공개 락 객체 관용구

~~~java
private final Object lock = new Object();

public void foo() {
    syncronized(lock) {
        ...
    }
}
~~~

* 비공개 락 객체는 클래스 바깥에서는 볼 수 없으니 클라이언트가 그 객체의 동기화에 관여할 수 없다.
* 비공개 락 객체 관용구는 무조건적 스레드 안전 클래스에서만 사용할 수 있다.
* 조건부 스레드 안전 클래스에서는 특정 호출 순서에 필요한 락이 무엇인지를 클라이언트에게 알려줘야 하므로 이 관용구를 사용할 수 없다.
* 비공개 락 객체 관용구는 상속용으로 설계한 클래스에 특히 잘 맞는다.
  상속용 클래스에서 자신의 인스턴스를 락으로 사용한다면, 하위 클래스는 아주 쉽게, 그리고 의도치 않게 기반 클래스의 동작을 방해할 수 있다.



##### 핵심 정리

> 모든 클래스가 자신의 스레드 안정성 정보를 명확히 문서화해야 한다. 정확한 언어로 명확히 설명하거나 스레드 안정성 애너테이션을 사용할 수 있다. syncronized 한정자는 문서화와 관련이 없다. 조건부 스레드 안전 클래스는 메서드를 어떤 순서로 호출할 때 외부 동기화가 요구되고, 그때 어떤 락을 얻어야 하는지도 알려줘야 한다. 무조건적 스레드 안전 클래스를 작성할 때는 syncronized 메서드가 아닌 비공개 락 객체를 사용하자. 이렇게 해야 클라이언트나 하위 클래스에서 동기화 메커니즘을 깨드리는 걸 예방할 수 있고, 필요하다면 다음에 더 정교한 동시성 제어 메커니즘으로 재구현할 여지가 생긴다.



___



## 아이템 83: 지연 초기화는 신중히 사용하라

**지연 초기화(lazy initialization)** 는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다. 그래서 값이 전혀 쓰이지 않으면 초기화도 결코 일어나지 않는다. 이 기법은 정적필드와 인스턴스 필드 모두에 사용할 수 있다.
지연 초기화는 주로 최적화 용도로 쓰이지만, 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결하는 효과도 있다.

##### 지연 초기화에 대한 최선의 조언 "필요할 때까지는 하지 말라"

* 지연 초기화는 양날의 검이다.
* 클래스 혹은 인스턴스 생성 시의 초기화 비용은 줄지만 그 대신 지연 초기화하는 필드에 접근하는 비용은 커진다.
* 지연 초기화하려는 필드들 중 결국 초기화가 이뤄지는 비율에 따라, 실제 초기화에 드는 비용에 따라, 초기화된 각 필드를 얼마나 빈번히 호출하느냐에 따라 지연 초기화가 실제로는 성능을 느려지게 할 수도 있다.

##### 지연 초기화가 필요할 때

* 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화하는 비용이 크다면 지연 초기화가 제 역할을 해줄 것이다.
* 하지만 정말 그런지 알 수 있는 방법은 적용 전후의 성능 측정을 해보는 것이다.

##### 멀티스레드 환경에서는 지연 초기화를 하기가 까다롭다.

* 지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 반드시 동기화해야 한다.

##### 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.

~~~java
// 인스턴스 필드를 초기화하는 일반적인 방법
private final FieldType field = computeFieldValue();
~~~

##### 지연 초기화가 초기화 순환성을 깨뜨릴 것 같으면 syncronized를 단 접근자를 사용하자.

~~~java
private FieldType field;

private syncronized FieldType getfield() {
    if (field == null)
        field = computeFieldValue();
    return field;
}
~~~

* 이상의 두 관용구는 정적 필드에도 똑같이 적용된다. static 한정자 추가

##### 성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스(lazy initialization holder class) 관용구를 사용하자.

~~~java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() {
    return FieldHolder.field;
}
~~~

* getField 메서드가 처음 호출되는 순간 FieldHolder.field 가 처음 읽히면서, 비로소 FieldHolder 클래스 초기화를 촉발한다.
* 동기화를 전혀 사용하지 않으니 성능이 느려질 거리가 전혀 없다.

##### 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사(double-check) 관용구를 사용하라.

* 이 관용구는 초기화된 필드에 접근할 때의 동기화 비용을 없애준다.
* 필드의 값을 두 번 검사하는 방식으로, 한 번은 동기화 없이 검사하고, 두 번째는 동기화하여 검사한다.
* 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시  volatile로 선언해야 한다.

~~~java
// 인스턴스 필드 지연 초기화용 이중검사 관용구
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result != null)    // 첫 번째 검사 (락 사용 안 함)
        return result;

    synchronized(this) {
        if (field == null) // 두 번째 검사 (락 사용)
            field = computeFieldValue();
        return field;
    }
}
~~~

* result는  필드가 이미 초기화된 상황에서는 그 필드를 딱 한 번만 읽도록 보장하는 역할을 한다.
* 반드시 필요하지는 않지만 성능을 높여주고, 저수준 동시성 프로그래밍에 표준적으로 적용되는 더 우아한 방법이다.

##### 이중검사의 두 가지 변종

* 반복해서 초기화해도 상관없는 인스턴스 필드를 지연 초기화해야 할 때가 있는데, 이런 경우라면 이중검사에서 두 번째 검사를 생략할 수 있다. 이 변종의 이름은 자연히 단일검사 관용구가 된다.

  ~~~java
  // 단일검사 관용구 - 초기화가 중복해서 일어날 수 있다.
  private volatile FieldType field;
  
  private Field getField() {
      FieldType result = field;
      if (result == null)
          field = result = computeFieldValue();
      return result;
  }
  ~~~

  * 모든 초기화 기법은 기본 타입 필드와 객체 참조 필드에 모두 적용할 수 있다.
  * 이중검사와 단일검사 관용구를 수치 기본 타입 필드에 적용한다면 필드의 값을 null 대신 0과 비교하면 된다.

* 모든 스레드가 필드의 값을 다시 계산해도 상관없고 필드의 타입이 long과 double을 제외한 다른 기본 타입이라면, 단일검사의 필드 선언에서 volatile 한정자를 없애도 된다.
  이 변종은 짜릿한 단일검사(racy single-check) 관용구라 불린다.
  어떤 환경에서는 필드 접근 속도를 높여주지만, 초기화가 스레드당 최대 한 번 더 이뤄질 수 있다.



##### 핵심 정리

> 대부분의 필드는 지연시키지 말고 곧바로 초기화해야 한다. 성능 때문에 혹은 위험한 초기화 순환을 막기 위해 꼭 지연 초기화를 써야 한다면 올바른 지연 초기화 기법을 사용하자. 인스턴스 필드에는 이중검사 관용구를, 정적 필드에는 지연 초기화 홀더 클래스 관용구를 사용하자. 반복해 초기화해도 괜찮은 인스턴스 필드에는 단일검사 관용구도 고려대상이다.



___



## 아이템 84: 프로그램의 동작을 스레드 스케줄러에 기대지 말라

여러 스레드가 실행 중이면 운영체제의 스레드 스케줄러가 어떤 스레드를 얼마나 오래 실행할지 정한다. 정상적인 운영체제라면 이 작업을 공정하게 수행하지만 구체적인 스케줄링 정책은 운영체제마다 다를 수 있다.

##### 정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램이라면 다른 플랫폼에 이식하기 어렵다.

* 견고하고 빠릿하고 이식성 좋은 프로그램을 작성하는 가장 좋은 방법은 실행 가능한 스레드의 평균적인 수를 프로세서 수보다  지나체가 많아지지 않도록 하는 것이다.
* 실행 준비가 된 스레드들은 맡은 작업을 완료할 때까지 계속 실행되도록 만들자.
* 실행 가능한 스레드의 수와 전체 스레드 수는 구분해야 한다.

##### 스레드는 당장 처리해야 할 작업이 없다면 실행돼서는 안 된다.

* 실행자 프레임워크를 예로 들면, 스레드 풀 크기를 적절히 설정하고 작업은 짧게 유지하면 된다.
  단, 너무 짧으면 작업을 분배하는 부담이 오히려 성능을 떨어뜨릴 수도 있다.

##### 스레드는 절대 바쁜 대기(busy waiting) 상태가 되면 안 된다.

* 공유 객체의 상태가 바뀔 때까지 쉬지 않고 검사해서는 안 된다는 뜻이다.

* 바쁜 대기는 스레드 스케줄러의 변덕에 취약할 뿐 아니라, 프로세서에 큰 부담을 주어 다른 유용한 작업이 실행될 기회를 박탈한다. 

  ##### 피해야할 CountDownLatch 예제 - 바쁜 대기 버전

  ~~~java
  public class SlowCountDownLatch {
      private int count;
  
      public SlowCountDownLatch(int count) {
          if (count < 0)
              throw new IllegalArgumentException(count + " < 0");
          this.count = count;
      }
  
      public void await() {
          while (true) {
              synchronized(this) {
                  if (count == 0)
                      return;
              }
          }
      }
      public synchronized void countDown() {
          if (count != 0)
              count--;
      }
  }
  ~~~

  * 자바의 CountDownLatch와 비교했을 때 10배가 느렸다.
  * 하나 이상의 스레드가 필요도 없이 실행 가능한 상태인 시스템은 흔하게 볼 수 있다. 이런 시스템은 성능과 이식성이 떨어질 수 있다.

##### 특정 스레드가 다른 스레드들과 비교해 CPU 시간을 충분히 얻지 못해서 간신히 돌아가는 프로그램을 보더라도 Thread.yield 를 써서  문제를 고쳐보려는 유혹을 떨쳐내자.

* 처음 JVM에서는 성능을 높여준 yield가 두 번째 JVM에서는 아무 효과가 없고, 세 번째에서는 오히려 느려지게 할 수도 있다.
* Thread.yield는 테스트할 수단도 없다.
* 차라리 애플리케이션 구조를 바꿔 동시에 실행 가능한 스레드 수가 적어지도록 조치해주자.



##### 핵심 정리

> 프로그램의 동작을 스레드 스케줄러에 기대지 말자. 견고성과 이식성을 모두 해치는 행위다. 같은 이유로, Thread.yield와 스레드 우선순위에 의존해서도 안 된다. 이 기능들은 스레드 스케줄러에 제공하는 힌트일 뿐이다. 스레드 우선순위는 이미 잘 동작하는 프로그램의 서비스 품질을 높이기 위해 드물게 쓰일 수는 있지만, 간신히 동작하는 프로그램을 '고치는 용도'로 사용해서는 절대 안 된다.



___

11장 동시성 끝...



##### 이펙티브 자바를 보면서 가장 이해가 안되는 장이다... 일단은 있는 그대로 이해를 해보려 노력하고 이후에 집중해서 볼 수 있도록 노력하자.
