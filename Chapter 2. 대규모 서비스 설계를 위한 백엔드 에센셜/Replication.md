## 0. Replication 이란?

1. Primary 의 데이터를 Secondary 에 복사해서 데이터의 sync를 맞추는 작업 
    1. 전통적인 RDB는 Primary, Secondary 형태로 Replication 지원 
    2. 많은 NoSQL 도 Replication 지원 
2. Replication을 하더라도 100% 데이터가 보장되지 않을 수 있다.
    1. Redis의 경우 Primary가 async로 데이터를 Secondary로 전달만 하기 때문에 데이터가 유실될 수도 있다.

### Repliacation 방식

1. Primary가 Secondary에게 직접 데이터를 보내는 방식 
2. Primary 가 Secondary 에게 데이터 변경 내역에 대한 명령어를 보내는 방식 
    1. 이 과정에서 실시간 데이터(현재 시간이라던가..)의 경우 데이터가 달라질 수 있다.

### 0.2 FailOver (Active, StandBy 패턴)

- Active, StandBy 패턴
    - Active: Primary는 읽기/쓰기를 모두 담당한다.
    - StandBy : Secondary로, Primary 의 데이터를 Replication 받다가 Primary 장애 시에 Primary 다 되어서 읽기/쓰기를 담당한다.
        - 이 때, 이전 Primary 에서 전달받은 데이터가 처리 전일 경우 이를 전부 처리하고 승격시켜준다.
            - MySQL - MHA / MMM
            - AWS - RDS

### Secondary 개수

1. 초기 데이터 복제 비용이 발생한다. 레디스는 포크 비용 발생 
2. 보통 2대를 추천.

### 읽기 분배 (Query Off)

1. StandBy는 평소에 백업용. Primary가 Replication  
2. 보통 Primary 에서 write를 하고 Secondary 에서 Read를 수행한다. → 부하가 줄어서 더 좋은 성능을 낸다.

#### 단점 

- Replication Lag
    - 데이터를 복제할 때 데이터의 시간차가 발생한다. → 이를 Eventual COnsistency 라 한다. SNS 처럼 바로 보이지 않을 경우엔 상관없지만 예약이라거나 현재 데이터가 중요한 경우에는 사용하기 어려울 수 있다.
    - 데이터를 캐시할 경우 문제가 더 지속될 수 있다.
    
    ---
    

## 실습

### 트러블 슈팅

1. db 만든다음에 2003 커넥션 에러 날떄 → 시큐리티 그룹 인바운드 규칙 수정. 수정못한다고 에러날텐데 하나 추가하고 이전거 삭제하면 됨 

[https://vitalflux.com/aws-error-2003-cant-connect-rds-mysql-server/](https://vitalflux.com/aws-error-2003-cant-connect-rds-mysql-server/)

2. the_Red 디비 없다고 하면 

mysql -h {end point} -p{password} -u{user}

create database the_red;

1. 인스턴스가 하나일 때 → 읽기 추가 
2. ro로 해도 write가 되는 이슈..
3. ro, write 도 ip 가 같음.. → 다를수도 있다고 하는거니까.. 같을 수도 잇지!
4. failover → 장애조치 
