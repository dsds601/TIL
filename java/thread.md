# Thread

* 하나의 CPU 가 두가지 이상의 일을 동시에 하는 것 처럼 보이게한다. (멀티 스레드)
* 자바는 기본적으로 메인스레드를 가지고 있다.

* 스레드 예제

~~~java
class SubThread implements Runnable {

    // 자바의 서브 스레드
    @Override
    public void run() {
        for (int i = 1; i<6; i++){
            try {System.out.print("메인 스레드 : "+i);
                Thread.sleep(1000);
            } catch (InterruptedException e){
                e.printStackTrace();
            }

        }
    }
}

public class ThreadEx01 {
    // 자바 메인 스레드
    public static void main(String[] args) {
      //서브 스레드 객체를 생성하면 heap 공간에 SubThread 라는 객체가 생성이되고 Runnable이라는 부모도 같이 생성이 된다.
      //SubThread가 Runnable에 run이라는 메서드를 오버라이드하여 부모에 메서드를 무효화되고 자식에 run메서드를 사용한다.
        SubThread st = new SubThread();
      // 스레드가 새로 생성이된다.
      // api를 보면 스레드는 생성자로 오버로딩 되어 잇는데 그중 매개변수로 Runaable 타입을 받는 생성자가 있다.
      // 그러므로 방금 생성한 Runaable을 구현한 SubThread를 매개변수로 새로운 스레드 생성이 가능하다. <- 타겟은 구현한 run
//        Thread t1 = new Thread();
				Thread t1 = new Thread(st);
	      t1.start(); // <- run메서드 실행
        for (int i = 1; i<6; i++){
            try {System.out.print("메인 스레드 : "+i);
                Thread.sleep(1000); //<- 타겟이 메인
            } catch (InterruptedException e){
                e.printStackTrace();
            }

        }
    }
}

~~~

* 스레드를 여러가지 만들어 실행할경우 한 스레드가 끝날때까지 기다리는게 아니라 서로 독립적으로 왓다갓다 하면서 실행된다.
* 문맥교환 -> **Context Switching** 이 일어난다.

* 스레드를 만들고 -> run 메서드를 오버라이딩하여 타겟이 될 기능을 지정 -> 스레드 생성자를 생성하며 매개변수에 Runnable를 구현한 객체를 넣어 새로운 스레드를 생성 -> 만든 스레드 statr
* 스레드에 핵심은 새로운 스레드를 만들고 run 메서드를 통해 타겟을 지정하여 실행한다. run에는 실행할 기능을 작성



### String Contant Pool

* String 은 클래스이다. <- 기본 자료형이 아니다. 주소를 가지고 있다.

~~~java
String s1 = new String();
String s2 = new String();
s1 == s2 // false;
String s3 = "a";
String s4 = "b";
s3 == s4 // true;
s3 = s3+"c"; // 
~~~

* String은 String에 상수 풀이 있어서 새로운 String에 값을 넣어 만들면 같은 공간(같은 주소)에 데이터가 생성이 된다.

* 기존에 생성된 값에 데이터를 추가하면 데이터가 변경되고 그 주소를 가리키는게 아니라 다른 데이터가 새로 생성되어 다른 주소를 가리키게된다.
* String은 데이터를 계속 생성하므로 buffer를 이용하여 메모리를 절약 할 수 있다.
* 데이터가 변경되어 사용하지않는 데이터(주소를 가리키지 않을 경우) -> GC 대상이된다.
* 장점 : 같은 문자열 (같은 공간 공유) -> 메모리 효율 높다.
* 단점 : 문자열을 변경하게 될때마다 새로운 공간이 할당 메모리를 많이 사용하게 된다. StringReader 사용

