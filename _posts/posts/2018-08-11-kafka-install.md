---
title:  "kafka 설치하기"
date:   2018-08-11 06:46:00
categories: posts
---

팀에서 관리하는 웹서버의 로깅시스템을 구축하면서 [kafka](https://kafka.apache.org/)를 사용하기로 했다. kafka는 서버간 데이터 전달을 위한 브로커 어플리케이션으로, 급격하게 증가하는 데이터를 장애없이 빠르게 보낼수 있는 messaging queue 역할을 한다. kafka를 사용하기로 한 데 있어서 결정적인 요소는 두가지였는데,
1. 게임 사이트, 런처를 관리하는 우리팀의 경우 서비스는 오픈이나 트래픽을 공격과 같이 급격하게 데이터가 밀려들어오는 이벤트가 자주 발생하고,
2. 기존 로깅시스템에서 사용되던 [fluntd](https://www.fluentd.org/)를 밀어내면서 fluntd의 대용량 처리기능을 보완하는 시스템이 필요했다.

아직 테스트 단계라 얼만큼 더 효과적으로 사용되고 있는지 확인하긴 어렵지만, 카프카의 취지와 기능을 고려했을때 현재 로깅 시스템 환경에 적합할 것이라고 판단했다.


### **kafka와 zookeeper**

kafka는 클러스터링 구성하는데 있어서 분산 시스템을 관리하는(coordination) 기능이 내장되어있지 않고 따로 주키퍼를 사용한다(Apache에서 개발리소스를 생각해서 함께 개발하지 않았다고 한다).

주키퍼는 카프카 application의 정보를 중앙에 집중하고 구성관리, 그룹관리네이밍, 동기화 등의 서비스를 제공한다. 아파치 카프카 공식 사이트에서 작성시점 현재 최신버전인 1.1.1버전을 받으면 함께 인스톨이 된다.

주키퍼는 다수가 되는 서버가 살아있으면 죽은 서비스를 살려내는 구조이므로, 홀수단위로 운용한다. 하지만 카프카 브로커를 1개만 운용한다고 해서 주키퍼 실행하지 않아도 되는 것은 아니므로, <u>반드시 카프카와의 연결 설정파일을 작성하고 먼저 실행을 시켜주도록하자.</u>

### **설치방법**

- JDK기반으로 돌아가기때문에 실행환경에서는 꼭 자바를 먼저 설치해놓도록 하자.
    (나의경우 카프카를 도커 이미지로 빌드했기 때문에 카프카는 볼륨으로 연결해서 실행하고, JDK는 이미지에 직접 설치했다)


- https://kafka.apache.org/downloads 에서 apache에서 권장하는 미러사이트로 tar파일 설치 
{% highlight bash %}
$ wget http://mirror.navercorp.com/apache/kafka/1.1.1/kafka_2.11-1.1.1.tgz
{% endhighlight %}

- 압축풀기
{% highlight bash %}
$ tar -xvf kafka_2.11-1.1.1.tgz
{% endhighlight %}

- 압축푼 폴더명 명령어 입력하기 편하도록 변경하기
{% highlight bash %}
$ mv kafka_2.11-1.1.1 kafka
{% endhighlight %}

### **zookeeper 설정파일 변경하기**
- 하단에 server.id로 입력된 주키퍼 서버 아이디의 넘버값은 dataDir경로에 myid이라는 파일명으로 작성되어있어야한다.

{% highlight bash %}
$ cat kafka/zkdata/myid # id가 1인 zookeeper데이터의 id파일
1
$ cd kafka/config
$ vi zookeeper.properties
{% endhighlight %}

{% highlight yaml %}
dataDir=/home/user/kafka/zkdata # 주키퍼의 트랜젝션 로그와 스냅샷이 저장되는 저장경로, 직접 편한 경로에 만들어주면 된다.
clientPort=2181 # client가 연결하는 TCP 포트
maxClientCnxns=0 # disable the per-ip limit on the number of connections since this is a non-production config
initLimit=5 # 팔로워가 리더가 초기에 연결하는 시간에 대한 타임아웃 tick 수
syncLimit=2 # 팔로워가 리더와 동기화하는 시간에 대한 타임아웃 tick수 (주키퍼에 저장된 데이터가 크면 늘려야함)

#서버.서버id=서버ip:리더인 경우 팔로워에 연결할 때 사용하는포트:리더 선출 시점에 사용하는 포트
#주키퍼 앙상블 전체의 id값과 서버, 포트를 입력해주면 된다. 나의 경우 같은 서버에 3개의 서비스를 올리려고했기 때문에 포트를 다 다르게 넣어줬다. 한 서버 내에서 리더팔로워 포트만 구분되면 된다.
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890
{% endhighlight %}


### **kafka 설정파일 변경하기**
{% highlight bash %}
$ vi server.properties
{% endhighlight %}

{% highlight yaml %}

broker.id=1 # The id of the broker. This must be set to a unique integer for each broker.
port=9092
delete.topic.enable=true #없으면 토픽삭제시 삭제가 안되므로 반드시 넣자

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
# listeners=PLAINTEXT://:9092

# Hostname and port the broker will advertise to producers and consumers. If not set,
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
# 이걸 설정을 안해주면 다른 서버에서 카프카로 통신이 되지 않는다ㅠㅠ
# 심지어 로컬로 다 포트바인딩을 해줬는데도 나는 도커 네트워크를 한번 타서 그랬는지 되지않았음.
advertised.listeners=PLAINTEXT://카프카서버IP:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600

############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/home/doyeon/kafka/kfdata

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended for to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# zookeeper #############################

# zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=localhost:2181,localhost:2182,localhost:2183

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=6000

############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0

{% endhighlight %}
	
	
