## 1. Redis 와 Memcached

가장 많이 사용되는 오픈소스 In-Memory Cache 솔루션

키-밸류 storage중에 redis가 1위, memcached 가 4위 기록 

| Redis | Memcached |
| --- | --- |
| 많은 자료구조(Collection)을 제공한다. | Redis에 비해 메모리 관리가 안정적이다. slab으로 관리해서 메모리 파편화가 덜 일어난다. |
| Replication 제공  | x |
| Cluster 모드 제공 | x |
| jemalloc으로 메모리 관리  | 자체적으로 메모리 관리 |
| 캐시 저장소로 주로 사용. 특수한 케이스는 디비 자체로 사용. sql 은 지원하지 않음 |  |

## 2. Key Value Store

데이터를 지칭하는 key와 이에 대한 데이터를 저장하는 Value 구조로 데이터를 저장하는 형태 

## 3. 레디스가 지원하는 자료구조

기본적으로 Hash Table 구조. 선형 리스트 크기가 커지면 메모리 할당에 문제가 생길 수 있다.

### 1. Strings

데이터의 사이즈가 너무 크면 느려질 수 있다.

1. Key/Value를 사용하는 구조
2. Key를 이용해서 Data를 저장하고 가져온다 
3. Key와 Value를 저장하는 set 명령어(Set<Key><Value)와 key로 value를 가져오는 get 명령(Get<Key>이 있다.
4. 기본적인 hash table을 이용한다.

```
set api:feeds:1234 "{'user_id':1234}"
get api:feeds:1234
//결과 : "{'user_id':1234}"

mset<key><value><key><value>...
mget<key><key>
```

주의점 

1. 레디스는 싱글 스레드이기 때문에 mget 사용 시 최대 50개 정도로 처리하는 것이 좋다.
2. 데이터의 ttl이 달라서 name은 없어졌는데 email은 남아있는다던가 하는 한번에 관리해야하는 데이터에 문제가 생길 수 있기 때문에 이런 경우는 hash 로 사용하는 것이 좋다.

### 2. List

기본으로 hash table이고 슬롯에 연결되는 데이터가 list

1. 중간에 추가/삭제가 느리고 head, tail에만 데이터를 추가/삭제할 때 유용하다
2. O(N) 이라 비용이 비쌈 
3. Queue 형태의 자료구조가 필요할 때 많이 사용 (SideKiq 나 Jesqueue 라는 레디스 기반의 Queue들이 존재한다)

```
LPush : 데이터를 왼쪽에 lpush<key><value>
RPush : 오른쪽으로 추가 rpush<key><value>
Lpop : 데이터를 왼쪽에서 가져옴 lpop<key>
RPop : 오른쪽에서 가져옴 rpop<key>
lrange : 리스트의 아이템들을 가져옴 lange<key><시작 인덱스><끝 인덱스>
```

### 3. Set

기본으로 hash table이고 슬롯에 연결되는 데이터가 set

1. 유일한 값들만 있는 집합을 유지하는 자료구조
2. 친구 리스트, 팔로워 리스트 등을 저장하는데 사용될 수 있다. 시큐리티나 oauth 에서 access token 을 저장하는 redis token store가 보통 set을 사용하고 있다.

```
SADD : set에 데이터 추가 
SISMEMBER: set에 해당 아이템이 있는지 확인하고, 데이터가 있으면 1, 없으면 0
SREM : Set에 해당 아이템이 있으면 삭제한다.
```

### 4. Hash

기본으로 hash table이고 슬롯에 연결되는 데이터가 hash

1. key 하위에 subkey를 이용해 추가적인 hash table을 제공하는 자료구조.

```
Hset: subkey에 value 추가
Hget: subkey로 value 가져옴 
Hmset: 여러 subkey 추가 
Hmget: 여러 subkey를 가져옴
Hgetall: 모든 subkey와 value를 가져옴
```

### 5. Sorted Set

기본으로 hash table이고 슬롯에 연결되는 데이터가 sorted set

1. 스코어를 가지는 set 구조 
2. 아이템들의 랭킹을 가지는데 사용할 수 있다.
3. 스코어는 double 형태이므로 특정 정수값을 사용할 수 없다.
4. skiplist 자료구조를 이용한다.
    1. Log(N)의 검색 속도를 가지는 리스트 자료구조 ([https://popcorntree.tistory.com/45](https://popcorntree.tistory.com/45))

```
ZADD : sorted set에 아이템과 score로 추가/변경 
ZRANGE: 요청한 Range의 아이템들을 가져온다.
ZREVRANGE : 요청한 Range의 아이템들을 score 역순으로 가져온다.
ZRANGEBYSCORE : 요청한 score range의 아이템들을 가져온다.
```

## 4. Redis Transaction = Multi/Exec

레디스는 싱글 스레드여서 아토믹 하다.

1. Multi : Exec이 나올 때까지 명령을 모아서 대기한다.
2. Exec : Multi로 모인 명령이 순서대로 실행되고, 수행되는 동안에는 다른 명령의 수행이 일어나지 않는다. 그래서 너무 많은 작업이 있으면 전체 Redis의 성능이 떨어진다.

트랜잭션을 보장하면서 다른 방법을 찾는다면 Lua script를 사용할 수도 있다.

## 5. Redis Pipeline

명령을 수행하고 응답을 기다리는 동안 명령 수행 간에 Time gap 이 존재하는데, 응답을 기다리지 않고 명령을 미리 보내는 방식.

파이프라인을 사용하지 않은 것과 비교해서 100만개를 테스트했을 때 10배 이상이 차이가 남.

실제로 레디스에서 제공하는건 아니고 라이브러리에서 제공한다.
