# 간단한 서비스 추상화 모습



## 대규모 서비스를 위해 필요한 부분들

### 기본적인 부분

- 데이터의 샤딩
  - 큰 데이터를 위한
  - 스케일업 ~> 스케일아웃으로 대응
- 기본적인 캐싱
  - 디비 부하 낮춤 
  - 샤딩 외에 캐싱을 해야 더 많은 트래픽 버티기 가능
- 모니터링
  - CPU 사용량, 메모리 사용량, 네트워크 사용량, API 응답시간 Latency, 트래픽, 에러 로그, .. 

<br/>

### 추가적인 부분

- 비동기 작업을 이용한 부하의 조절
- Primary Write, Secondary Read

<br/>

### 추상도

(그림)

<br/>

### 실습

간단하게 글을 Write 하고 목록을 Read 하는 서비스

- 글 생성 시에 guid와 scrap 서버를 호출해서
- 결과를 비동기로 DB에 저장하고
- 서비스를 위해 Cache를 먼저 저장하는 
- Write-Back 방식을 사용한다. 

<br/>

```
1. docker-compose 로 아래 항목들 생성
- redis 1, 2, 3 캐시용
- redis_queue 큐용
- mysql 
iteshuicBookPro:my_service itesh$ docker-compose up -d

iteshuicBookPro:guid itesh$ docker ps
CONTAINER ID   IMAGE                      COMMAND                  CREATED              STATUS              PORTS                     NAMES
7de09efb4b07   the_red/guid               "/bin/sh -c './start…"   About a minute ago   Up About a minute                             focused_lichterman
02942200f6ed   the_red/guid               "/bin/sh -c './start…"   4 minutes ago        Up 4 minutes                                  elegant_nightingale
62801a5de3c7   the_red/guid               "/bin/sh -c './start…"   5 minutes ago        Up 5 minutes                                  goofy_pike
7b373185d51c   my_service_the_red_mysql   "docker-entrypoint.s…"   30 minutes ago       Up 30 minutes       0.0.0.0:13306->3306/tcp   the_red_mysql
b6d0ad135ff0   redis:6.0.6                "docker-entrypoint.s…"   30 minutes ago       Up 30 minutes       0.0.0.0:16379->6379/tcp   my_service_the_red_redis_queue_primary_1
639b007ee32d   ascensionit/sidekiq-web    "/bin/sh -c 'bundle …"   30 minutes ago       Up 30 minutes       8080/tcp                  the_red_sidekiq
1edc9a4c4996   redis:6.0.6                "docker-entrypoint.s…"   30 minutes ago       Up 30 minutes       0.0.0.0:16380->6379/tcp   my_service_the_red_redis_1_1
bfdb6119de45   redis:6.0.6                "docker-entrypoint.s…"   30 minutes ago       Up 30 minutes       0.0.0.0:16382->6379/tcp   my_service_the_red_redis_3_1
d5c1fb63f4dc   redis:6.0.6                "docker-entrypoint.s…"   30 minutes ago       Up 30 minutes       0.0.0.0:16381->6379/tcp   my_service_the_red_redis_2_1

2. Zookeeper를 통해서 서비스 디스커버리 실행
3. 캐시용 레디스 주소와 큐용 레디스 주소 등록
iteshuicBookPro:my_service itesh$ python3 zoo_setup.py 
Created: /the_red/my_service/cache/nodes/redis1:127.0.0.1:16380
Created: /the_red/my_service/cache/nodes/redis2:127.0.0.1:16381
Created: /the_red/my_service/cache/nodes/redis3:127.0.0.1:16382

참고) guid와 scrap 서버는 실행 시에 zookeeper에 자신을 등록한다.

4. guid 서버와 scrap 서버 여러대를 동시에 쉽게 띄우기 위해 docker 이용
iteshuicBookPro:guid itesh$ pwd
/Users/itesh/develp/the_red/chapter_3/my_service/guid
iteshuicBookPro:guid itesh$ docker build . -t the_red/guid

iteshuicBookPro:guid itesh$ docker run -e WORKER_ID=0 -e DATACENTER the_red/guid

악..
```



<br/>

# 어떻게 성장할 것인가?



"Hard Skill + Soft Skill"



#### 오픈소스 선정 조건

- 관심도 : 내가 관심이 있던 기술인가?
- 사용 빈도 : 내가 자주 접하는 오픈 소스인가?
- 언어 : 내가 충분히 익숙한 언어로 구현되었는가?
- 기반 지식 : 관련 기반 지식을 이해하고 있는가?

<br/>

#### 이상적인 오픈소스 참여 플로우

업무로 오픈소스 사용 -> 사용 중에 버그 발견 -> 버그 수정 -> upstream 반영


