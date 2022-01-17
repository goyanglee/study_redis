## 0. FailOver란?

FailOver: Active한 시스템에 장애가 발생했을 때, 혹은 장비의 이전 등의 사유로 서버가 변경되어야 할 때 StandBy 서버가 Active로 전환돼서 서비스가 계속 운영되게 하는 것. 서비스 가용성(High Availability)를 제공하기 위해 사용된다.

간단한 시나리오 : 클라가 Primary 서버 설정을 Secondary 설정으로 바꾸고 재배포 

→ 보통은 디비나 캐시처럼 스토리지 서버가 이런 경우인데 누군가 모니터링을 계속 해야한다는 단점이 있다.

→ 자동화된 FailOver 가 필요하다.

## 1. Coordinator 를 이용한 방식으로 FailOver 자동화

1. Client 가 Coordinator에게 API 주소를 알려달라고 호출 
2. Coordinator가 헬스체크하거나 헬스체크 서비스를 따로 둬서 괜찮은 상태의 서버 API 주소를 반환

(Q. LB가 이런걸 하지않나?)

### 특징

1. Coordinator 와의 연동이 필요하다.
2. 이미 Coordinator 를 쓰고 있다면 적용이 쉽고, 아니면 적용이 복잡하다.

## 2. VIP 와 DNS를 이용한 FailOver 자동화

### 2.1 VIP(Virtual IP)를 이용하는 방법

1. 클라이언트는 VIP 만 호출한다.
2. VIP 는 Active를 가리키고 있다가 헬스체크 서비스가 장애를 감지하면 StandBy 서비스로 변경한다.
3. 이미 연결되어 있는 커넥션을 끊어준다.

### 2.2 DNS 를 이용하는 방법

VIP 대신에 DNS로 관리한다. 단, DNS 는 TTL 이 존재하는데 외부 DNS는 며칠 혹은 영구히 캐싱해서 호출과 부하를 줄인다. 내부 DNS는 호출이 적어서 TTL 을 짧게 가져가도 된다. 

#### 2.2.1 AWS에서 제공하는 FailOver 

1. DNS 방식을 이용한다.
2. 기본적으로 Multi-AZ를 사용해야한다.(Available Zone)

#### 2.2.2 주의점

1. TTL 이 존재한다. 짧게 설정해야하지만 부하를 많이 줄 수 있다. 외부는 VIP, 내부는 DNS를 보통 사용한다.
2. 영구 캐시하는솔루션도 있다.

---

## 실습

### 1. 레디스 구축 (Primary, Secondary)

프로그램이 레디스를 모니터링하고 있다가 서비스 디스커버리가 바꿔주는 예제 

api/v1/write/{key}?value={value}

apiv1/get/{key}

1. cd ~/the_red/redis
2. src/redis-server ../conf/redis_6379.conf
3. src/redis-server ../conf/redis_6380.conf
4. telnet 0 6379 (brew install telnet)
5. info
6. the_red/apache-zookeeper-~/bin/ ./zkServer.sh start
7. python zoo_setup.py
8. ./zkCli.sh 
9. 

[zk: localhost:2181(CONNECTED) 3] get /the_red/storage/posts
{"primary": "127.0.0.1:6379", "secondary": ["127.0.0.1:6380"]}

1. python [monitor.py](http://monitor.py/) ./monitor.ini
2. telnet 0 6379 > info 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b745d7f-6802-4648-adf6-8f2acaf8e32d/Untitled.png)

1. API 서버 띄우기. the_red/chapter_2/redis_failover ./start.sh 172.30.1.23:7001 
2. [http://172.30.1.23:7001/api/v1/write/100?value=user100](http://172.30.1.23:7001/api/v1/write/100?value=user100) 
3. [http://172.30.1.23:7001/api/v1/get/100](http://172.30.1.23:7001/api/v1/get/100)
4. kill 6379
5. monitor 에서 보면 아래와 같이 뜸

check: 127.0.0.1:6380 failure_count: 0
Primary Redis is 127.0.0.1:6380
Error 61 connecting to 127.0.0.1:6379. Connection refused.
Error 61 connecting to 127.0.0.1:6379. Connection refused.

1. 다시 src/redis-server ../conf/redis_6379.conf 

Error 61 connecting to 127.0.0.1:6379. Connection refused.
Error 61 connecting to 127.0.0.1:6379. Connection refused.
set 127.0.0.1:6379 as replica of 127.0.0.1:6380
