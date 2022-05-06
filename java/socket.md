# Socket

### port ( 통신의 입구 )

* 포트는 스트림으로 연결은 컴퓨터당 한개만 가능하기에 통신을 받을때 각각에 프로그램을 컴퓨터에 분산 해주기위해 있다 
* 포트의 개수는 0개  ~ 65535개  존재 0~1023 포트는 기본으로 정해진 포트 그외는 사용 가능



### 소켓통신

* 양 끝단에 포트를 연결하고 포트끼리 ByteStream으로 데이터를 주고 받는것을 말한다.

~~~java
ServerSocket serverSocket; // 연결을 받는 소켓
Socket socket;	// 실제 통신을 하는 소켓
~~~

* 소켓을 2개 두는 이유
* 클라이언트가 서버에 포트를 연결 -> ip : port번호로 연결 시도 -> 연결 성공시 서버소켓이 새로운 소켓을 만들어낸다.
  * 서버소켓에 역활은 연결만 받는거고 서버소켓에서 새로운 소켓을 만들어 클라이언트라 바이트스트림으로 연결하여 통신한다. 포트 값은 사용하지 않는 값을 랜덤으로 선정한다.
  * 실제 통신은 클라이언트 소켓이랑 서버에 소켓이 연결한다.

~~~java
public ServerFile() {
        System.out.println("서버 소켓 시작");
        try{
            serverSocket = new ServerSocket(10000);
            System.out.println("서버 소켓 생성 완료 : 클라이언트 접속 대기중...");
            socket = serverSocket.accept(); // 클라이언트 접속 대기...
            System.out.println("클라이언트 연결 완료");

            br = new BufferedReader(new InputStreamReader(socket.getInputStream())); 
          // <- System.in : 키보드와 연결이 아니라 // socket에 입력 스트림과 연결
        }catch (Exception e){
            System.out.println("서버소켓 에러 발생"+e.getMessage());
        }
    }

public ClientFile () {
        try {
            System.out.println("클라이언트 소켓 시작");
            socket = new Socket("localhost",10000); // <- socket 연결 서버에 accept() 메서드 호출
            bw = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
            br = new BufferedReader(new InputStreamReader(System.in));
            String keyboardMsg = br.readLine();
            // 메시지 끝을 알려줘야한다. \n <- 필수 통신 규칙
            bw.write(keyboardMsg+"\n");
            bw.flush();
        } catch (Exception e) {
            System.out.println("서버소켓 에러 발생"+e.getMessage());
        }
    }
~~~



* 양방향 통신을 위해 스레드를 통한 소켓통신

* 위 에 코드는 현재 메인 스레드에서 서버는 읽고 클라이언트는 메시지를 보내고 있기에 다른 일을 못해서 서버는 쓰고 클라이언트는 출력하는 기능을 서브 스레드를 활용한다.

* #### 서버 코드

~~~java
public class ServerFile {

    ServerSocket serverSocket;
    Socket socket;
    BufferedReader br;
    // 새로운 스레드
    BufferedWriter bw;
    BufferedReader keyboard;

    public ServerFile() {
        System.out.println("서버 소켓 시작");
        try{
            serverSocket = new ServerSocket(10000);
            socket = serverSocket.accept(); // 클라이언트 접속 대기...
            System.out.println("클라이언트 연결 완료");
            br = new BufferedReader(new InputStreamReader(socket.getInputStream())); // <- System.in : 키보드와 연결이 아니라
            keyboard = new BufferedReader(new InputStreamReader(System.in));
            bw = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
            WriteThread wt = new WriteThread();
            Thread t1 = new Thread(wt);
            t1.start();

            System.out.println("서버 소켓 생성 완료 : 클라이언트 접속 대기중...");
            // socket에 입력 스트림과 연결
            while(true){
                String msg = br.readLine();
                System.out.println("클라이언트로부터 받은 메시지 : "+msg);
            }
        }catch (Exception e){
            System.out.println("서버소켓 에러 발생"+e.getMessage());
        }
    }

    class WriteThread implements Runnable {
        @Override
        public void run() {
            while(true) {
                try {
                    String keyboardMsg = keyboard.readLine();
                    bw.write(keyboardMsg+"\n");
                    bw.flush();
                } catch (Exception e) {
                    System.out.println("서버 소켓쪽 키보드 입력 오류 : " + e.getMessage());
                }
            }
        }
    }

    public static void main(String[] args) {
        ServerFile serverFile = new ServerFile();
    }
}

~~~



* #### 클라이언트 코드

~~~java
public class ClientFile {

    Socket socket;
    BufferedWriter bw;
    BufferedReader keyboard;
    BufferedReader br;

    public ClientFile () {
        try {
            System.out.println("클라이언트 소켓 시작");
            socket = new Socket("localhost",10000); // <- socket 연결 서버에 accept() 메서드 호출
            bw = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream()));
            keyboard = new BufferedReader(new InputStreamReader(System.in));
            br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            ReadThread rt = new ReadThread();
            Thread t1 = new Thread(rt);
            t1.start();

            while(true) {
                String keyboardMsg = keyboard.readLine();
                // 메시지 끝을 알려줘야한다. \n <- 필수 통신 규칙
                bw.write(keyboardMsg+"\n");
                bw.flush();
            }
        } catch (Exception e) {
            System.out.println("서버소켓 에러 발생"+e.getMessage());
        }
    }

    class ReadThread implements Runnable {
        @Override
        public void run() {
            while (true) {
                try{
                    String msg = br.readLine();
                    System.out.println("서버로부터 받은 메시지 : "+msg);
                } catch (Exception e){
                    System.out.println("클라이언트 읽기 오류 " +e.getMessage());
                }

            }
        }
    }

    public static void main(String[] args) {
        ClientFile clientFile = new ClientFile();
    }
}
~~~

