### Static , Heap ,Stack 이란?

* 메모리는 스택처럼 쌓아져있고 각각 주소를 가지고 있다. 메모리에 있는 데이터를 가지고 연산을 하고 싶다면 cpu가 필요하다<br/>-> 메모리는 저장에 용도 / cpu는 연산에 용도 / 모니터 출력 용도
* 자바에서 메모리는 논리적으로 영역을 나누어 **static  heap stack** 으로 구분을 지어 사용한다.<br/>

#### static

* 프로그램에 시작과 끝 까지 존재하는 영역 : 부하가 제일 크다.

#### heap

* 동적으로 사용하고 필요없을때 사라져야하는 영역 : 동적 할당 영역

#### stack

* 잠깐 사용하는 영역 가장 짧게 사용하는 영역 변수가 저장되는 영역 호출되면 바로 사라집니다

~~~java
static int a = 10; <- static 영역
  public static void main(args[]){  <- static 영역
  int b = 10;
}
static 은 메인 메서드와 a 변수
stack 영역은 b
main 메서드 내부에 있는 b는 stack 영역이므로 main이 열리면 실행되고 호출시 사라진다s
~~~



### 클래스 자료형 (static)

~~~java
class beans {
	static int a = 10;
	static boolean b = false;
}
// beans 는 클래스 자료형 이며 개발자가 만든 커스텀 자료형 :: 여러가지 데이터를 가지고 있는 클래스를 Beans라고 한다.
~~~

* 데이터 값이 다른 데이터를 100개 필요하면 노트 클래스를 100개 만들어야한다.

* 만약 프로그램 실행 중 만든 클래스자료형 이상 필요하다면 줄 수 없다 -> static은 프로그램 실행할때 뜨고 끌때까지 유지 새로 생성x
  * 동적으로 생성 필요 -> heap

### 클래스자료형 (heap)

~~~java
class Note {
	int price =1000;
	int num = 1;
}
public class Var {
  public static void main(String[] args) {
    new Note(); // <- heap 영역에 note클래스가 가지고 있는 모든 데이터를 할당 static 제외 (동적할당s)
    new Note();
    new Note();
    Note note1 = new Note(); // <- 레퍼런스를 등록하여 각각에 데이터를 사용 할 수 있다.
    int a = 10; // <- heap이 아닌 main에 stack 공간에 쌓인다.
  }
}
//static 영역에 Note 와 Var가 있고 statid에 Var 영역에는 main이라는 스택이 있다
//Var에 stack영역이 열려 그 안에있는 main에서 New Note() <- new 연산자를 통해 static이라는 데이터를 제외하고 나머지 데이터를 heap 영역에 저장하게 된다.
//heap 영역에는 int char .... 가 아닌 note라는 영역에 price / num 이라는 데이터가 저장한다.
~~~

