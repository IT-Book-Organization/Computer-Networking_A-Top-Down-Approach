# 2.7 소켓 프로그래밍: 네트워크 애플리케이션 생성

네트워크 애플리케이션을 생성할 때는 두 프로그램, 클라이언트와 서버 프로그램을 작성해야 한다.

두 프로그램을 실행하면 프로세스가 생성되고, 두 프로세스가 소켓으로부터 읽고 쓰기를 통해 서로 통신한다.

<br/>
<br/>

클라이언트 - 서버 애플리케이션에는 두 가지 형태가 있다.

<br/>

> 1️⃣ HTTP 등의 RFC에 정의된 표준 프로토콜을 구현하는 클라이언트-서버 애플리케이션

이 애플리케이션을 구현할 때 그 프로토콜과 연관된 port를 사용하여야 한다.

<br/>

> 2️⃣ 개인의 독점적인 네트워크 애플리케이션으로 RFC 또는 다른 곳에 공식적으로 출판되지 않은 애플리케이션 계층 프로토콜을 채택하여 구현하는 애플리케이션

다른 독립 개발자는 이 애플리케이션과 상호작용하는 코드를 개발할 수 없다. 이 애플리케이션을 구현할 때는 잘 알려진 포트번호를 사용하지 않도록 유의해야 한다.

개발자는 TCP, UDP 프로토콜 중 어떤 프로토콜을 사용해야 하는지 각 프로토콜의 특징을 고려하여 선택해야 한다.

<br/>
<br/>
<br/>

# 2.7.1 UDP 소켓 프로그래밍

UDP를 사용할 때에는 송신 프로세스가 데이터의 패킷을 소켓 문 밖으로 밀어내기 전에, 먼저 패킷에 목적지 주소를 붙여넣어야 한다.

이 패킷이 송신자의 소켓을 통과한 후 인터넷은 이 목적지 주소를 이용하여 그 패킷을 인터넷을 통해 수신 프로세스에 있는 소켓으로 라우트(route)할 것이다.

패킷이 수신 소켓에 도착하면 수신 프로세스는 소켓을 통해 그 패킷을 추출하고, 다음에 패킷의 콘텐츠를 조사하여 적절한 동작을 취한다.

<br/>
<br/>

## 목적지 주소

패킷에 목적지 주소를 포함함으로써 인터넷의 라우터는 목적지 호스트로 인터넷을 통해 패킷을 라우트할 수 있다.

호스트는 각자 `IP 주소`를 식별자로 갖는다.

<br/>

그러나 호스트는 하나 혹은 그 이상의 소켓을 갖는 많은 네트워크 애플리케이션 프로세스를 수행하고 있을 수 있기 때문에,  
**목적지 호스트 내의 특정한 소켓을 식별할 필요가 있다.**

소켓이 생성될 때 `포트 번호`라고 하는 프로세스 식별자가 소켓에 할당된다.

즉, 패킷에는 IP 주소와 포트번호로 구성된 목적지 주소가 붙게 되며, 송신자의 출발지 주소(IP와 port)도 붙게 된다.

<br/>

일반적으로 이렇게 주소를 붙이는 것은 하부 운영체제가 자동으로 실행한다.

<br/>
<br/>

## 소켓 프로그래밍

<p align="center"><img width="600" alt="UDP 애플리케이션" src="https://user-images.githubusercontent.com/76640167/210794628-43c23c47-34d8-4a5b-b5e4-d7d617cdcf90.png">

위 그림은 UDP 서비스 상에서 통신하는 클라이언트와 서버의 주요 소켓 관련 활동을 나타낸다.

1. 클라이언트는 키보드로부터 한 줄의 문자를 읽고 그 데이터를 서버로 보낸다.
2. 서버는 그 데이터를 수신하고 문자를 대문자로 변환한다.
3. 서버는 수정된 데이터를 클라이언트에게 보낸다.
4. 클라이언트는 수정된 데이터를 수신하고 그 줄을 화면에 나타낸다.

위 순서로 작동하는 간단한 클라이언트-서버 애플리케이션을 만들 예정이다.

<br/>

### UDPClient.py

소켓을 생성할 때는 따로 소켓의 포트 번호를 명시하지 않아도 되며, 운영체제가 이 작업을 대신 수행한다.

```
# socket module이다. 이 module을 통해 소켓을 생성할 수 있다.
from socket import *

#서버의 IP 혹은 서버의 호스트 이름을 할당한다.
serverName = ’hostname’

# 목적지 port 번호를 나타낸다.
serverPort = 12000

# 클라이언트 소켓을 생성한다. AF_INET은 IPv4를 사용하고 있음을 나타내고, SOCK_DGRAM은 UDP 소켓임을 의미한다.
clientSocket = socket(AF_INET, SOCK_DGRAM)

# 보낼 메시지를 입력 받는다.
message = Input(’Input lowercase sentence:’)

# 소켓으로 바이트 형태를 보내기 위해 먼저 encode()를 통해 바이트 타입으로 변환한다.
# sendTo() 메서드는 목적지 주소를 메시지에 붙이고 그 패킷을 프로세스 소켓인 clientSocket으로 보낸다.
# 클라이언트 주소도 같이 보내지는데 이는 자동으로 수행된다.
clientSocket.sendto(message.encode(),(serverName, serverPort))

# 패킷 데이터는 modifiedMessage에 저장되고, 패킷의 출발지 주소(IP, port)는 serverAddress에 할당된다.
# recvfrom() 메서드는 2048의 버퍼 크기로 받아들인다. 
modifiedMessage, serverAddress = clientSocket.recvfrom(2048)

# 출력
print(modifiedMessage.decode())

# 소켓 닫기
clientSocket.close()
```

<br/>

### UDPServer.py

while 문을 통하여 한 번의 통신 이후에도, 계속 다음 UDP 패킷이 도착하기를 기다린다.

```
from socket import *

# 포트 번호
serverPort = 12000

# UDP 소켓 생성
serverSocket = socket(AF_INET, SOCK_DGRAM)

# 12000 포트 번호를 소켓에 할당한다. 이를 통해 서버 IP 주소의 12000 포트로 패킷을 보내면 해당 소켓으로 패킷이 전달된다.
serverSocket.bind((’’, serverPort))

print(”The server is ready to receive”)

while True:
    # 패킷이 서버에 도착하면 데이터는 메세지에 할당되고 패킷의 출발지 주소는 clientAddress에 저장된다.
    # 해당 주소로 서버는 응답을 어디에 보내야할지 알 수 있다.
    message, clientAddress = serverSocket.recvfrom(2048)
    
    # 바이트 데이터를 decode()하고 대문자로 변환한다.
    modifiedMessage = message.decode().upper()
    
    # 클라이언트 주소를 대문자로 변환된 메시지에 붙이고, 그 결과로 만들어진 패킷을 서버에 보낸다.
    # 서버의 주소도 같이 보내지는데 이는 자동으로 수행된다.
    serverSocket.sendto(modifiedMessage.encode(), clientAddress) 
```

<br/>
<br/>
<br/>

# 2.7.2 TCP 소켓 프로그래밍

`TCP`는 **연결 지향 프로토콜**로, 서로 데이터를 보내기 전에 먼저 TCP 연결을 설정할 필요가 있다.

TCP 연결을 생성할 때 클라이언트 소켓 주소와 서버 소켓 주소를 연결과 연관시킨다.

연결이 설정된 후 소켓을 통해 데이터를 TCP 연결로 보내면 된다.

<br/>
<br/>

## 환영 소켓과 연결 소켓

<p align="center"><img width="450" alt="TCP" src="https://user-images.githubusercontent.com/76640167/210809355-8e26c0e0-561c-45bc-ba57-8d34487c68bc.png">

서버 프로세스가 실행되면 클라이언트 프로세스는 서버로의 TCP 연결을 시도하는데, 이는 **클라이언트 프로그램에서 TCP 소켓을 생성**함으로써 가능하다.

TCP 소켓을 생성할 때, 서버에 있는 `환영(welcome) 소켓`의 주소(IP, port #)를 명시한다.

소켓을 생성한 후 클라이언트는 3-way handshake를 하고 서버와 TCP 연결을 설정한다. (핸드셰이킹은 프로그램에서 전혀 인지 못한다.)

<br/>

핸드셰이킹 동안 서버는 **해당 클라이언트에게 지정되는 새로운 소켓**을 생성한다. 이를 `연결 소켓`이라고 한다.

애플리케이션 관점에서 볼 때 클라이언트의 소켓과 서버의 연결 소켓은 파이프에 의해 직접 연결된다.

파이프를 통해 클라이언트는 자신의 소켓으로 임의의 바이트를 보낼 수 있으며, 서버 프로세스가 그것을 수신하는 것을 TCP가 보장한다. 이는 서버 입장에서도 마찬가지이다.

<br/>
<br/>

## 소켓 프로그래밍

<p align="center"><img width="600" alt="TCP 애플리케이션 구조" src="https://user-images.githubusercontent.com/76640167/210813409-0e49cc1d-bab4-4bfb-8ba2-1d039f09f3c1.png">

위 그림은 전형적인 클라이언트-서버 TCP 연결 구조이다.

UDP 프로그램과 똑같은 기능을 하는 프로그램을 작성해보자.

<br/>

### TCPClient.py

주석은 UDP와 다른 부분을 위주로 작성하였다.

```
from socket import *

serverName = ’servername’
serverPort = 12000

# 클라이언트 소켓을 의미한다. SOCK_STREAM으로 TCP 소켓임을 명시했다.
# UDP 때와 마찬가지로 따로 출발지 주소를 명시하지 않는다. (운영체제가 대신 해준다.) 
clientSocket = socket(AF_INET, SOCK_STREAM)

# 클라이언트가 TCP 소켓을 이용하여 서버로 데이터를 보내기 전에 TCP 연결이 먼저 클라이언트와 서버 사이에 설정되어야 한다.
# 해당 라인으로 TCP 연결을 시작하고, connect() 메서드의 파라미터는 연결의 서버 쪽 주소이다.
# 이 라인이 수행된 후에 3-way handshake가 수행되고 클라이언트와 서버 간에 TCP 연결이 설정된다.
clientSocket.connect((serverName, serverPort))

sentence = raw_input(’Input lowercase sentence:’)

# 클라이언트 소켓을 통해 TCP 연결로 보낸다. UDP 소켓처럼 패킷을 명시적으로 생성하지 않으며 패킷에 목적지 주소를 붙이지 않는다.
# 대신 클라이언트 프로그램은 단순히 문자열에 있는 바이트를 TCP 연결에 제공한다.
clientSocket.send(sentence.encode())

# 서버로부터 바이트를 수신하기를 기다린다.
modifiedSentence = clientSocket.recv(1024)
print(’From Server: ’, modifiedSentence.decode())

# 연결을 닫는다. 이는 클라이언트 TCP가 서버의 TCP에게 TCP 메시지를 보내게 한다.
clientSocket.close()
```

<br/>

### TCPServer.py

```
from socket import *

serverPort = 12000

# TCP 소켓 생성
serverSocket = socket(AF_INET, SOCK_STREAM)

# 서버의 포트 번호를 소켓과 연관시킨다.
serverSocket.bind((’’, serverPort))

# 연관시킨 소켓은 대기하며 클라이언트가 문을 두드리기를 기다린다.
# 큐잉되는 연결의 최대 수를 나타낸다.
serverSocket.listen(1)
print(’The server is ready to receive’)

while True:
    # 클라이언트가 TCP 연결 요청을 하면 accept() 메소드를 시작해서 클라이언트를 위한 연결 소켓을 서버에 생성한다.
    # 그 뒤 클라이언트와 서버는 핸드셰이킹을 완료해서 클라이언트의 소켓과 연결 소켓 사이의 TCP 연결을 생성한다.
    connectionSocket, addr = serverSocket.accept()
    sentence = connectionSocket.recv(1024).decode()
    capitalizedSentence = sentence.upper()
    connectionSocket.send(capitalizedSentence.encode())
    
    # 응답을 보내고 연결 소켓을 닫는다. 그러나 환영소켓인 serverSocket이 열려있어 다른 클라이언트가 서버에 연결을 요청할 수 있다. 
    connectionSocket.close()
```
