# Consistent Hashing

consitent hashing(안정 해시)란, 일반적으로 요청 또는 데이터를 서버에 균등하게 나누기 위해 일반적으로 사용되는 기술이다. 분산처리 환경에서 사용하는 알고리즘으로, 서버가 여러대일 때 어느 서버로 보내주어야 할 지 정할 때 사용하는 알고리즘이며 특징은 해시함수와 원입니다.

### 일반적인 Modular 방식의 문제점

: 서버의 추가/삭제 시에 리밸런싱이 계속 일어난다. (sharding에서의 modular의 방식의 단점 참고.)

### consistent hashing의 특징

- 서버의 추가/삭제 시에 리밸런싱은 적게 일어난다.
- 같은 hash 를 사용하면 hash(key)는 항상 같은 결과를 보여준다.

![Untitled](Consistent%20Hashing%20e1a993a0c5514e39b6e053479fc5ff67/Untitled.png)

- Rule을 선택한다. : Rule은 구현을 선택을 하면 된다.

강의 예제의 룰

: Hash(Key)의 값보다 크면서 가장 가까운 hash값을 가진 서버에 자신의 데이터를 저장한다.

**그러다 삭제가 필요하면?**

![Untitled](Consistent%20Hashing%20e1a993a0c5514e39b6e053479fc5ff67/Untitled%201.png)

: 다시 B가 살아나면 1은 다시 B에 속하게 됨

: 다른 값들은 변화가 없고, 서버의 범위 안에 있는 노드만 변화가 일어난다는 특징

### Consistent hashing은 부하에 안정적인가?

: No! 서버 장애가 발생하면 해시 Ring은 다음 서버에 모든 부하가 넘어간다

⇒ virtual node 도입

- 서버의 가상 주소를 만들어낸다.
- Consistent hashing은 hash 값에 관심을 가진다는 것을 사용한다.

⇒ 가상 서버의 주소가 충분히 많아지면 부하는 적절히 나눠질 가능성이 높다.

### Consistent hashing을 구성하는데 어떤 값을 서버 key값은 Unique한 값이어야 한다.

ip와 같이 변경될 수 있는 값을 사용하면 Hash Ring이 바뀌어서 데이터의 위치가 변경될 수 있다.

**닉네임 처럼 계속 사용할 수 있는 이름을 사용해야 한다.**

단순히 cache로만 사용할 때는 문제가 되지 않지만, 데이터를 유지하려고 한다면, 꼭 Unique한 Nickname을 사용해야 한다. 

: 예를 들어 Redis 서버로 RDS 나 AOF등으로 데이터를 백업하고 있는 경우

### Consistent Hashing hashCode

- Ketema hash
    - libmemcached 에서 이용하는 hash 함수
    - md5를 이용한다.
    
    ![Untitled](Consistent%20Hashing%20e1a993a0c5514e39b6e053479fc5ff67/Untitled%202.png)
    

- 많은 대체제가 있다. 적절한 것을 사용하면 된다.
    - murmurhash3
    - jump hash

![Untitled](Consistent%20Hashing%20e1a993a0c5514e39b6e053479fc5ff67/Untitled%203.png)

큌소트 알고리즘이랑 비슷. 자기보다 크지만 가장 가까운것을 찾으려함

<실습>

1. cd study/the_red/apache-zookeeper-3.6.3-bin/bin : 왜 전의 실습과 주키퍼 위치가 달라질까?
2. ./zkServer.sh start ( 반대: ./zkServer.sh stop)
3. ./zkCli.sh
4. ls  /
5. deleteall /the_red :하위 디렉토리 전부 지워주기

그리고 zoomtl.exe를 실행시키는데 이게 뭔지? ⇒ 필요 없는 듯함!

<탭2>

1. cd /the_red/chapter_2/consistent_hashing
2. vi docker-compose.yml
3. docker-compose up -d ( 반대: docker-compose down)

<aside>
💡 [https://bluegreencity.github.io/docker/Docker-Compose-사용하기/](https://bluegreencity.github.io/docker/Docker-Compose-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0/)

</aside>

 

1. telnet 0 16379
2. vi zoo_setup.py : nodes에 redis등록되어있고, ip대신 닉네임이 사용됨
3. python zoo_setup.py

<탭1>

1. ls /the_red/cache/redis/scrap : 4개의 노드가 생성된걸 볼 수 있음.

<탭2>

1. vi consistent_hash.py : replica를 지정하면 virtual node를 만드는게, 그 만큼 지정됨 
2. vi [main.py](http://main.py) : 샤드레인지 함수가 바뀔 때마다 Consistent Hash ring 를 새로 만듬. : 같은 키에 대핸 값은 값이 나온다.
3. ./start.sh 172.30.1.25:7001
4. /api/v1/scrap?url=https://www.fastcampus.co.kr 로 접속 : 캐시를 저장하게 됨 
5. /demo 페이지를 통해, 키가 어떻게 들어간지 볼 수 있음

![Untitled](Consistent%20Hashing%20e1a993a0c5514e39b6e053479fc5ff67/Untitled%204.png)

server hash : 닉네임을 캐싱한 값

key hash:url을 캐싱한 값

1. redis4를 없애봄 (vi zoo_setup.py에서 4번 주석처리) : 값은 그대로 남아있다.
2. kakao.com을 호출하면 캐시가 not exist가 뜨고, 다시 redis3에 추가됨 

![Untitled](Consistent%20Hashing%20e1a993a0c5514e39b6e053479fc5ff67/Untitled%205.png)

1. 그리고 다시 redis4를 살리면 (강의에서는.. 사라질거라고 했는데....)
2. 두개가됨..?!??

![Untitled](Consistent%20Hashing%20e1a993a0c5514e39b6e053479fc5ff67/Untitled%206.png)