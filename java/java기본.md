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



### Object (모든 클래스에 부모)

~~~java
Object cat = new Cat();
//메모리에 Cat Object 둘다 올라간다. 여기서 cat은 Object클래스를 가리키고 있기에 다운 캐스팅 하여 가리키는 주소를 Object가 아니라 Cat을 바라보게 하면 Cat에 메서드를 사용 할 수 있다.
~~~

* 다운 캐스팅을 하는 이유 : 다른 클래스 타입 여러가지를 저장하고 싶을때 Object 타입으로 업캐스팅하여 저장하여 담고 다운 캐스팅하여 원래 클래스 주소를 가리켜 사용 할 수 있다.

~~~java
class 소나타 {
	String name ="소나타";
}
class K5 {
	String name ="K5";
}

public static void main (String args[]) {
	Object [] arrObj = new Object[2];
	arrObj[0] = new 소나타();
	arrObj[1] = new K5();
	
	소나타 sonata = (소나타)arrObj[0];
	k5 K5 = (K5)arrObj[1];
	// 이렇게 사용 가능
}
~~~



###  제네릭

* Object를 자료형을 사용햇을때에 다운캐스팅을 해야하는 단점을 보완할 수 있다.

~~~java
class 호랑이 {
	String name ="호랑이";
}
class 사자 {
	String name = "사자";
}
class 동물<T> {
	T data;
}

public static void main (String args[]){
	동물 animal= new 동물(); // <-x 동물은 제네릭 타입이라 타입을 지정해주어야 합니다.
	동물<호랑이> 호랑이 = new 동물<>();
  s1.data = new 호랑이();
	sysout(호랑이.data.name); // 호랑이 출력
}
	/*
	class 동물<T> { <- T 에 호랑이가 들어오고
	T data;  <- data 는 호랑이 타입
	}
	*/

~~~

* 와일드 카드 제네릭에 ? 어떤것을 리턴할지 몰라!

~~~java
<? extends Object> // Object를 상속하는 모든것중 하나를 리턴한다 라는 의미
? add () {} // <- extends Object 생략되어 잇습니다.
~~~

* 예제

~~~java
class 가방<T> {
	private T data;
	
	public T getDate (){
		return this.data;
	}
	
	public T setDate (data){
	this.data = data;
	}
}

static 가방<?> 꺼내기 (int time) { // ? 는 extneds Object 가 생략되어있다.
	if(time == 9) {
		축구공 s1 = new 축구공();
		가방<축구공> b1 = new 가방<>();
		가방.setData = s1;
		return s1;
	} else {
		농구공 s2 = new 농구공();
		가방<농구공> b2 = new 가방<>();
		가방.setDate(s2);
		return b2;
	}
}
~~~

* 와일드 카드를 사용할때는 리턴하는 object 상위에 추상클래스를 만들어 그 추상 클래스를 상속받은 object로 사용하는게 좋다
* -> object 로 상속받은 객체이기에 object에 대한 메서드만 사용할수 있기에 리턴하는 클래스에 메서드를 사용 할 수 없기에 추상 클래스에 메서드를 오버라이드 하여서 리턴할 object들이 메서드를 재정의하여 사용하면 다운 캐스팅 없이 메서드를 사용할 수 있다.
