# 무선조종차와 PC간 통신구현

![](../.gitbook/assets/image%20%289%29.png)



각 디바이스의 IP를 알고 사용자가 정한 PORT 번호가 client와 server가 같으면 같은 와이파이 범위 내에서 통신할 수 있다. 라즈베리 파이 Ubuntu MATE를 설치할 때 와이파이 설정을 해주었다면 별도의 설정 없이 바로 와이파이를 사용할 수 있다. PC는 iptime n100mini dongle을 사용하여 와이파이에 접속한다.

      자신의 컴퓨터 와이파이 IP 검색

Window라면 윈도우키 + R을 눌러 실행 탭에 cmd 입력

우분투라면 터미널 창을 실행

| \(윈도우\) ipconfig  \(우분투\) hostname -I |
| --- |


      socket 통신

공통

-        소켓 생성하기

socket.socket\(\) 함수를 이용해서 소켓 객체를 생성할 수 있다. 서버든 클라이언트 등 동일하게 소켓을 이용한 네트워킹을 하기 위해서는 소켓을 먼저 생성할 필요가 있다. 이 함수는 두 가지 인자를 받는데, 하나는 패밀리이고 다른 하나는 타입이다.

1\)     패밀리: 첫 번째 인자는 패밀리이다. 소켓의 패밀리란, “택배 상자에 쓰는 주소 체계가 어떻게 되어 있느냐”에 관한 것으로 흔히 AF\_INET, AF\_INET6를 많이 쓴다. 전자는 IP4v에 후자는 IP6v에 사용된다. 각각 socket.AF\_INET, socket.AF\_INET6로 정의되어 있다.

2\)     타입: 소켓 타입이다. raw 소켓, 스트림 소켓, 데이터그램 소켓등이 있는데, 보통 많이 쓰는 것은 socket.SOCK\_STREAM 혹은 socket.SOCK\_DGRAM이다. 가장 흔히 쓰이는 socket.AF\_INET, socket.SOCK\_STREAM 조합은 socket.socket\(\)의 인자 중에서 family=, type=에 대한 기본 인자 값이다. 따라서 이 타입의 소켓을 생성하고자 하면 인자 없이 socket.socket\(\)만 써도 무방하다.

-        정보 주고받기  
  
 소켓으로부터 데이터를 읽을 때는 sock.recv\(\)를, 정보를 보낼 때는 sock.sendall\(\)을 사용한다. sock.recv\(bufsize\)는 읽어들일 데이터의 크기를 정해서 그만큼을 읽어온다. 단 파일을 읽을 때처럼 반복해서 읽을 수는 없다. 소켓은 한 번은 읽고 한 번은 보내는 ‘턴 바이 턴’식으로 통신하기 때문이다.  sock.sendall\(data\)은 주어진 데이터를 보낸다.

-        닫기  
  
 소켓 역시 외부 리소스를 열어서 사용하는 것이므로 닫는 것이 매우 중요하다. 연결을 종료할 때에는 서버와 클라이언트 모두 소켓을 닫아야 하며, 이미 닫혀있는 소켓에서 데이터를 받으려 하거나 데이터를 보내려 하는 동작은 모두 에러가 된다. 소켓을 닫을 때에는 sock.close\(\) 메소드를 사용한다. 단, 소켓 객체는 컨텍스트매니저 프로토콜을 지원하기 때문에 with 문으로 사용하면 명시적으로 닫을 필요가 없다.

서버

-        바인드 : 서버 쪽의 맵핑 방식

서버가 특정한 포트를 열고 입력을 기다리기 위해서는 소켓을 포트에 바인드하는 과정이 선행되어야 한다.  이는 생성된 소켓 객체에 대해서 sock.bind\(\) 메소드를 이용해서 실행한다. bind\(\) 호출 시에는 호스트이름과 포트 번호가 필요한데, 이들을 각각의 인자로 넘기는 것이 아니라 튜플로 감싸서 전달한다.

바인드는 서버 쪽에서만 필요하다. 이 작업은 프로그램 인터페이스인 소켓과 네트워크 시스템이 자원을 구분하는 IP와 포트 번호를 연결한다. 즉 프로그램/프로그래머는 자신이 사용하는 포트가 명시적으로 몇 번인지, 자신의 IP가 무엇인지 알고 있어야 한다. \(알고 있다는 말은 즉 자신이 능동적으로 정해준다는 말이다.\) 그래야 이 정보를 교신상대, 클라이언트에게 알려 클라이언트가 접속할 수 있게 한다.

-        포트 듣기\(Listen\)와 열기

바인드가 완료되면 포트를 듣는다. sock.listen\(\) 메소드를 사용한다. 이 메소드는 호출되면 클라이언트가 해당 포트에 접속하는 것을 기다린다. 접속이 들어오면 \(그것이 원하는 클라이언트인지, 임의의 접속 요청인지는 알 수 없다.\) 리턴된다. 이는 ‘듣기’만 하는 것이다. 실제로 접속을 수락하는 것은 다음 차례이다.  listen\(\)으로 접속 시도를 알아챘다면 이쪽\(서버\)에서도 그 요청을 받아서 접속을 시작한다. 접속의 개시는 sock.accept\(\)를 사용한다.

이 메소드는 \(소켓, 주소정보\)로 구성되는 튜플을 리턴한다. 여기서 소켓은 실제 클라이언트와 접속이 이루어져 교신 가능한 소켓이다. 서버는 최초 생성되어 듣는 소켓이 아닌 accept\(\)의 리턴으로 제공되는 소켓을 사용해서 클라이언트와 정보를 주고받을 수 있다. \(왜냐하면, 소켓이라는 모델 자체가 1:N 통신을 상정하고 있기 때문이다.\)

클라이언트

-        연결하기 – connect

클라이언트가 소켓을 생성하는 방법은 서버 측과 같다. 서버는 \(생성-&gt;바인드-&gt;듣기-&gt;수락-&gt;읽기-&gt;쓰기-&gt;닫기\)의 사이클대로 동작한다면, 클라이언트는 조금 단순하다. 바인드 과정이 없으며, 그 자신이 접속을 능동적으로 수행하기 때문에 생성-&gt;연결-&gt;쓰기-&gt;읽기의 사이클이 적용된다.

연결은 sock.connect\(\)를 사용하며 이때 사용하는 인자는 bind\(\)와 같다.
