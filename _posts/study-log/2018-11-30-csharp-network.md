---
title:  "c# 기본서 - 네트워크"
date:   2018-11-30 15:13:00
tags: [c#, bcl, network, tcp, udp, ip]
categories: study-log
---

### 비동기 호출
- 프로토콜: 어떤 절차를 거쳐서 통신을 주고받을것이냐에 대한 규칙
    - 현재 인터넷에서 가장 많이사용하는 규칙은 tcp/ip이다.
- IPv4: Internet Protocol의 4번째버전에 해당하는 기술
    - 이미 주소가 바닥났지만
    - 하나의 IP라우터를 통해 공용IP를 공유하고 사설 IP를 할당받는 등의 방식으로 큰 불편을 느끼지 못하고 있다.
    - 사설IP는 공식적인 인터넷 기관에 사용여부가 등록되지 않은 IP이다
- 포트
    - IP주소는 컴퓨터에 장착된 네트워크 어댑터는 식별해주지만 실행중인 프로그램은 구분할 수 없다
    - 이러한 구분을 위해 포트를 사용한다
- DNS: 도메인 하나에 여러개의 IP를 묶을 수 있다. 이를 통해 부하분산(load balance)을 할 수 있다.
{% highlight c# %}
string myComputer = Dns.GetHostName();
IPHostEntry entry = Dns.GetHostEntry(myComputer);
foreach (IPAddress ipAddress in entry.AddressList)
    Console.WriteLine(ipAddress.AddressFamily+": "+ipAddress)
    //ex) InterNetworkV6: fe80::bcae:bc0f:10d5:4ec4%12
    //ex) InterNetwork: 192.168.50.95
{% endhighlight %}
    - 클라이언트는 ping 명령어를 통해 dns의 어떤 IP와 바인딩이 됐는지 확인할 수 있으며
    - 바인딩 정보는 시스템에 저장된다
    - 관리자모드로 cmd를 실행하고 아래의 명령어를 통해 목록을 비울 수 있다.
{% highlight c# %}
$ ipconfig \flushdns
{% endhighlight %}
- 사설 IP의 경우로 많이 사용되는 대역은 아래와 같다 
    - 10.0.0.0 ~ 10.255.255.255
    - 172.16.0.0 ~ 172.31.255.255
    - 192.168.0.0 ~ 192.168.255.255


### System.Net.Sockets.Socket
- Socket 클래스는 IDisposable 상속을 받기 때문에 자원해제를 해줘야한다.
- TCP와 UDP의 서버소켓 모두 특정 IP와 바인딩된다. 바인딩된 IP는 접점(EndPoint)이라고 부른다
    - 이렇게 바인딩이 되고 나면 다른 소켓에서는 절대로 동일한 접점 정보로 바인딩 할 수 없다.
    - 소켓은 모든 IP에 대해 바인딩할 수 있는 방법을 제공하는데 이때 제공하는 주소가 "0.0.0.0"이다
- 프로그램을 실행중인 컴퓨터의 IP주소를 의미하는 용도로 "127.0.0.1"이 예약되어있다
    - loopback address라고 한다.


### System.Net.Sockets.Socket: UDP
- TCP와 UDP의 제일 큰 차이는 데이터 전송의 신뢰성확보
    - 때문에 TCP가 더 느리지만
    - UDP를 사용할때도 신뢰성 확보를 위한 코드를 추가하게되면 비슷해져서 TCP를 사용한다.
- UDP는 신뢰성이 결여되어있다
    - 이 의미는 즉, 중간에 거쳐가는 네트워크 장치가 많아질수록 상대방에게 데이터가 전달되지 않을수도 있다는 점이다.
    - 뿐만아니라 파편화되어 전달될경우 패킷이 유실될 확률이 높다.
    - 또한 순서가 확보되지 않으며
- UDP는 한번에 보낼 수 있는 데이터 한계가 있다
    - .Net의 UDP용 SendTo()메서드는 65535바이트를 넘을 수 없다. 또한 UDP 장비중에선 32KB정도만 허락하는 경우도 있으므로 많은 데이터를 보내는것을 권장하지 않는다.


### System.Net.Sockets.Socket: TCP
- TCP는 Listen(9)호출을 통해 클라이언트로부터 접속을 허용하며, 안에 인자로 받는 숫자는 허용하는 클라이언트 접속 큐의 갯수이다.
- Accpet()를 통해 클라이언트 연결을 받고 Send/Receive를 한다.
    - 소켓통신은 기본적으로 동기호출이므로 Send/Receive를 호출한 메서드는 블로킹된다
    - 때문에 서버가 Accept를 빠르게 처리할 수 없다.
- Accept를 ThreadPool을 통해 비동기로 구현할 수 있지만
    - 이 경우는 스레드 문맥 전환문제가 생긴다.
    - 이를 극복하기위해 비동기로 구현하는 Begin/End+Send/Receive가 있다.
        - 그러나 이 방법 또한 과다하게 코드가 복잡해지므로 고성능 TCP서버 구현이 아니면 스레드와 클라이언트 간의 1:1 대응 방식이 선호된다
- HTTP 통신은 TCP 서버/클라이언트의 한 사례이다
    - BCL은 Socket클래스를 사용하지 않고 HTTP통신을 쉽게 구현할 수 있는
        - System.Net.HttpWebRequest와
        - System.Net.WebClient를 제공한다

