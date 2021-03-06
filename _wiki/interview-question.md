---
layout  : wiki
title   : 
summary : 
date    : 2020-05-11 21:46:27 +0900
updated : 2020-07-01 20:12:58 +0900
tags    : 
toc     : true
public  : true
parent  : 
latex   : false
---
* TOC
{:toc}

## Database

### 캐시
 
#### 인메모리를 쓰는 이유 

- 애플리케이션 레벨의 로컬캐시가 빠르지만 두 가지 문제
    - 데이터유실
    - 데이터동기화
        - 서버가 두 대만 되어도 하나가 바뀌면 다른 서버에 어떻게 알려주냐부터 이슈
- 그래서 인메모리 캐시를 외부에 놓고 참조한다
    - 네트워크를 통하므로 로컬보다 느리지만 위 두 문제가 없다

#### 캐시가 유실되면?

- 인메모리는 모든 데이터가 디스크가 아닌 메모리에 올라가 있어
    - 유실될 수도 있으므로 캐시
    - 유실안 되는 개념은 스토어
- 캐시가 날라가면 디비로 가는데 준비를 안 해 놓아서 디비가 못 버티는 경우가 실제 꽤 많이 일어난다
    - 썬더링 커즈 - 캐시 키가 하나만 사라져도 십만 리퀴스트 일어나면 디비로 간다
예) 트위터 - 하이라키 캐시구조 1, 2, 3차까지, 비싸므로 파레토 법칙 적용해서
- 캐시 key가 하나가 없어진 사이에 이거에 요청이 몰리는 경우, DB가 죽을 수 있다


#### DB에 두고 공유하는 거랑, Redis 등의 in memory DB 서버를 따로 두고 쓰는 거랑 어떤 차이가 있는지?

- 디스크에 접근하면 초당 1천번 정도라면(DB는 디스크를 쓰게 되어 있다), 메모리에 접근하면 10만번이 가능 (in memory DB는 메모리만 쓴다)

#### 캐시에 데이터를 불러올 때

- 해시구조 키 밸류 (멤캐시드는 이것만 지원)
- 그 안에 셋
- 아니면 소티드 셋 

#### Cache를 스토리지처럼 쓸 수 있는 Data grid

- 레디스도 클러스터 있지만 클러스터보다 하나의 서버 개념
- 캐시 스토리지처럼 쓸 수 있다는 데이터그리드로 넘어가는 것이 새로운 경향?
    - 해이즐캐스트, 인피니스팬
    - 데이터 리플리케이션을 잘 전보다 복잡하고 잘 구현하는 개념


#### 네트워크에서 상용으로 쓰는 레디스

- 기본설정으로 쓰면 장애로 이어질 것
    - 테스트 옵션으로 되어 있기 떄문
- 엘라스틱 캐시 관리 이슈
    - 싱글 스레드라 긴 작업이 들어가면 장애

#### 캐시에 올릴 데이터 기준

- 잘 안 바뀌는데 자주 접근하는 데이터
    - SNS는 거의 캐시로 돌아가는 서비스

#### 어떤 데이터가 많이 잡히는지 볼 수 있는 방법?

- 태그 히트률을 가져와서 따로 테이블에 저장
- key마다 태그를 달아서 어떤 데이터가 많이 생성되는지 볼 수 있다. 혹은 read마다 update

#### Redis를 서버 동기화 목적?

- Redis를 서버 동기화 목적으로 쓰는 경우? 메시지큐 pub/sub
- 가볍다. sub이 없으면 pub해도 그냥 없어지므로, 보통 list 자료구조를 많이 쓴다.
- 그외 용도로 Redis를 활용하는 방법?
    - 다양한 자료구조를 지원하므로
        - 소티드셋 - 랭킹에 쓰기 좋다.
        - API limitter - 초당 몇 개만 써라

#### 레디스 랭킹API에 쓰면

- 보통 랭킹은 디비에 유저 스코어 저장하고 오더 바이로 정렬 후 읽어오기
- 개수가 많아지면 속도문제 발생
- 레디스는 소티드셋을 쓰면 랭킹 구현 가능
    - 덤으로 리플리케이션도 가능
    - 다만 한계에 종속될 수도
        - 저장할 id가 1개당 100byte라고 하면
            - 10명 1k
            - 10000명 1M
            - 10조면 1TB

#### 레디스 장점

- 레디스 자료구조가 Atomic하기 때문에 해당 Race Condition을 피할 수 있다.
    - 그래도 잘못 짜면 발생함


#### 레디스 사용처

- 리모트 데이터 스토어
    - A서버, B서버, C서버에서 데이터를 공유하고 싶을 때
- 한 대에서만 필요하다면 전역 변수를 쓰면 되지 않을까?
    - 레디스 자체가 Atomic을 보장해준다.
- 주로 많이 쓰는 곳들
    - 인증 토큰 통을 저장
    - 랭킹 보드로 사용
    - 유저 API Limit
    - 잡큐
        - 리시트로 많이 쓴다

#### Redis Collections

- Strings(키, 밸류)
    - 기본사용법
        - Set <Key> <Value>
        - Get <Key>
        - mset <key1> <value1> <key2> <value2> ...
        - mget <key1 <key2> ...
- List(삽입 시 헤드와 테일은 빠르지만 중간은 느리다)
    - 기본사용법
        - Lpush <Key> <A>
            - Key: <A>
        - Rpush <Key> <B>
            - Key: <A, B>
        - Lpush <Key> <C>
            - Key: <C, A, B>
        - pop
            - 꺼내올 떄, LPOP, RPOP
- Set
    - 기본사용법
        - SADD <Key> <Value>
            - Value가 이미 Key에 있으면 추가되지 않는다.
        - SMEMBERS <Key>
            - 모든 Value를 돌려줌
                - '모든'이 들어간 명령은 주의해서 써야
        - SISMEMBER <Key> <Value>
            - value가 존재하면 1, 없으면 0
    - 보통 특정 유저를 Follow하는 목록을 저장할 때 많이 쓴다. 유니크하므로
- Sorted Set
    - Set은 순서가 없지만 Sorted Set은 Score를 줘서 순서를 보장할 수 있다
    - 기본사용법
        - ZADD <Key> <Score> <Value>
            - Value가 이미 Key에 있으면 해당 Score로 변경된다.
            - 스코어 값 순서대로 정렬되어서 저장된다.
        - ZRANGE <Key> <StartIndex> <EndIndex>
            - 해당 Index 범위 값을 모두 돌려줌
            - Zrange testkey 0 -1
                - 모든 범위를 가져옴
                - 번호가 인덱스, -1이 끝을 의미
    - 유저 랭킹보드로 사용할 수 있음
    - 소티드 셋의 스코어는 정수형이 아니라 더블 타입이므로 주의해야
        - 실수가 표현할 수 없는 정수값들이 존재
        - 간혹 발생하는 내가 왜 그랬을까의 대표적 사례
    - Sorted Sets - 정렬이 필요한 값이 필요하다면?
        - `select * from rank order by score limit 50, 20;`
            - zrange rank 50 70
        - `select * from rank order by score desc limit 50, 20;`
            - Zrevrange rank 50 70
                - 뒤에서부터 순서를 매긴다.
- Hash
    - Key 밑에 sub key가 존재
    - 키밸류 안에 다시 키밸류가 존재
    - 기본 사용법
        - Hmset <Key> <subkey1> <value1> <subkey2> <value2>
        - Hgetall <Key>
            - 해당 key의 모든 subkey와 value를 가져옴
        - Hget <Key> <subkey>
        - Hmget <Key> <subkey1> <subkey2> ...

#### Collection 주의사항

- 하나의 컬렉션에 너무 많은 아이템을 담으면 좋지 않음
    - 10000개 이하 몇천 개 수준으로 유지하는 게 좋음
- Expire는 Collection의 item 개별로 걸리지 않고 전체 Collection에 대해서만 걸림
    - Expire - 삭제
    - 즉 해당 10000개의 아이템을 가진 Collection에 expire가 걸려 있다면 그 시간 후에 10000개의 아이템이 모두 삭제

#### 레디스 운영

- 메모리 관리를 잘하자
    - In-memory이므로 피지컬 메모리 이상 사용하면 당연히 문제 발생
        - 바로 죽지 않고 Swap 일어난다
        - Swap - 메모리 꽉차면 디스크로 내리고 비면 다시 메모리로 올리고
        - Swap이 한 번이라도 있다면 해당 메모리 Page 접근 시마다 느려짐
    - Maxmemory를 설정하더라도 이보다 더 사용할 가능성이 큼
        - jemalloc? 때문에 레디스는 자기가 사용하는 메모리 크기를 정확하게 알 수 없기 때문에
    - RSS 값을 모니터링해야 함
    - 메모리 관리
        - 큰 모메로 사용하는 인스턴스 하나보다 적은 메모리 인스턴스 여러 개가 더 안전하다.
            - 라이트가 헤비한 레디스는 포크를 해서 최대 메모리를 2배까지 쓸 수 있다.
            - 24기가면 48기가를 소모, 8기가면 16기가를 소모
    - 메모리 부족할 때는?
        - 좀더 메모리 많은 장비로 Migration
        - 있는 데이터 줄이기
            - 다만 Swap 사용중이라면 프로세스 재시작해야 함
        - 메모리 줄이기 위한 설정
            - 기본적인 Collection 틀
                - Hash -> HashTable
                - Sorted Set -> Skeplist와 HashTable
                - Set -> HashTable
                - 해당 자료 구조들은 메모리를 많이 사용함
            - Ziplist를 이용하자
                - In-memory 특성상 적은 개수라면 선형 탐색을 하더라도 빠르다.
                - List, hash, sorted set 등을 ziplist로 대체해서 처리함는 설정이 존재
- O(N) 관련 명령어는 주의하자
    - 싱글스레드라 한 번에 한 명령만 처리
        - 긴 시간이 필요한 명령을 수행하면? 망한다.
        - 초당 10만TPS 처리하는데 1개만 1초 걸려도 끝장
    - 대표적인 O(N) 명령어들
        - KEYS
            - 한 번에 모든 데이터 가져오는 명령
            - scan 명령으로 하나의 긴 명령을 짧은 여러 번 명령으로 바꿀 수 있다
        - FLUSHALL, FLUSHDB
        - Delete Collections
        - Get All Collections
            - Collection의 모든 item을 가져와야 할 때?
                - Collection의 일부만 거져오거나
                    - Sorted Set
                - 큰 Collection을 작은 여러 개의 Collection으로 나눠서 저장
                    - Userranks -> Userrank1, Userrank2, Userrank3
                    - 하나당 몇천 개 안쪽으로 저장하는 게 좋음

#### Redis Replication

- Async Replication
    - Replication Lag이 발생할 수 있다.
    - A 데이터 바뀌면 B 데이터에게 쏴주는데 그 틈에 생긴 변화
    - 많이 벌어지면 슬레이브가 마스터와 연결을 끊어버린 후 일정 기간 이후 다시 연결되면서 부하
- `'Replicaof'(>=5.0.0) or 'slaveof'` 명령으로 설정 가능(마스터, 슬레이브가 바뀐 용어)
    - Replicaof hostname port
- DBMS로 보면 statement replication이 유사
- Replication 설정 과정
    - Secondary에 replicaof or slaveof 명령을 전달
        - 마스터의 호스트 IP, 도메인 어드레스, 포트
    - Secondary는 Primary에 sync 명령 전달
    - Primary는 현재 메모리 상태를 저장하기 위해
        - Fork
    - Fork한 프로세서는 현재 메모리 정보를 디스크에 덤프
    - 해당 정보를 secondary에 전달
    - Fork 이후의 데이터를 Secondary에 계속 전달
- Redis Replication 시 주의할 점
    - Replication 과정에서 포크가 발생하므로 메모리 부족이 발생할 수 있다
    - Redis-cli --rdb 명령은 현재 상태의 메모리 스냅샷을 가져오므로 같은 문제를 발생시킴
    - AWS나 클라우드의 Redis는 좀 다르게 구현되어서 해당 부분이 좀더 안정적
    - 많은 대수의 레디스 서버가 Replica를 두고 있다면
        - 네트워크 이슈나 사람의 작업으로 동시에 replication이 재시도되도록 하면 문제가 발생할 수 있다
        - ex) 같은 네트웍 안에 30GB를 쓰는 Redis Master 100대 정도가 리클리케이션을 동시에 재시작하면 어떤 일이 벌어질 수 있을까?

#### redis.conf 권장 설정 Tip

- Maxclient 설정 50000
- RDB/AOF 설정 off
- 특정 커맨드 disable
    - Keys
    - AWS의 ElasticCache는 이미 하고 있음
- 전체 장애의 90% 이상이 KEYS와 SAVE 설정을 사용해서 발생

#### Redis 데이터 분산

- 데이터의 특성에 따라 선택할 수 있는 방법이 달라진다.
    - Cache일 때는 우아한 Redis?
    - Persistent 해야만 안 우아한 Redis!
        - Open the hellgate
- 데이터 분산 방법
    - Application
        - Consistent Hashing
            - twemproxy를 사용하는 방법으로 쉽게 사용 가능
        - Sharding
    - Redis Cluster

#### 참고할만한 sample 설정?

- save 끄기
    - 기본값: 5분에 1만번 이상 업데이트하면 모든걸 디스크로 내림.
    - 테스트는 괜찮은데 실서비스에서 장애날 가능성 농후
    - elasticache에서는 아예 못 쓰게 막았다
    - [troubleshooting redis 참고](https://www.slideshare.net/charsyam2/troubleshooting-redis)

#### Cache에 대한 새로운 접근법/논문?

- Cache를 썼을 때 생기는 문제점. Look-Aside caching
- read/write 발생빈도에 따라 range 조절가능> 좀더 적은 수의 cache 서버로 같은 트래픽을 처리할 수 있다. adaptive c???
- hot data는 memory에, cold data는 ssd에.


#### 15년차 개발자로서 다른 분들에게 말하고 싶은 것

- 오픈소스를 쓴다면 코드를 꼭 봐라.
- 알고 넘어가는 것/ 에러를 근본적으로 해결하는 태도가 중요하다

### Memcached와 Redis의 차이

- Memcached
    - 명료함을 위해
    - 멀티스레드 지원, 스케일업을 통한 작업 처리 가능
    
- Redis
    - 다양한 용도
    - 싱글 스레드
    - 다양한 데이터 구조 활용 가능
    - 스냅샷 - 특정 시점 저장 보관 가능

### 스케일 아웃, 스케일 업 

#### 스케일 아웃

- 서버를 여러 대 추가해 확장하는 방법
- 각 서버에 걸리는 부하를 균등하게 해주는 '로드밸런싱' 필요
    - 한 대가 다운되더라도 다른 서버로 제공 가능하지만
    - 모든 서버가 동일한 데이터를 가지고 있어야 하므로 데이터 변화가 적은 '웹 서비스'에 적합

#### 스케일 업

- 서버에 고성능 부품을 교환하는 방법
- CPU나 RAM을 추가하려면 여유 슬롯이 있어야 하며 그렇지 않은 경우 서버 자체를 교체해야 함
    - 서버 한 대에 부하가 집중되므로 장애 시 영향을 크게 받을 수 있음
    - 한 대 서버에서 모든 데이터를 처리하므로 데이터 갱신이 빈번하게 일어나는 '데이터베이스 서버'에 적합한 방식


### 정규화 기준

- 애트리뷰트(필드)를 나눠서 작은 릴레이션으로 분해하는 작업

### NoSQL을 사용하는 이유 및 사례, 개념

#### 개념

'Not Only SQL'
- 스키마 없이 동작하며 대부분 클러스터(서버들)에서 실행할 목적으로 만들어졌기에 관계형 모델을 사용하지 않는다.

#### 등장 배경

- 트래픽이 늘면서 스케일 업에 따른 비용부담이 커졌다. 이에 분산 저장을 목표로 스케일 아웃에 사용할 NoSQL 등장

#### 장점

- NOSQL 수평확장, 유연한 추가저장, 데이터 형식 - 예)페이스북, 트위터 게시물 저장, 실시간으로 쌓이는 로그 데이터

#### 종류

- [나무위키](https://namu.wiki/w/NoSQL)

### Database가 물리적으로 저장되는 과정

- [데이터베이스의 기본 구조](http://egloos.zum.com/sweeper/v/3003333)


### DB Indexing? 왜 사용하는가?

- 인덱스는 칼럼 값과 레코드 주소를 키와 값 쌍의 인덱스로 만드는 것.
- 정렬 상태를 유지하므로 값 탐색은 빠르지만 값 추가, 삭제 등의 수정 속도는 느리다.
    - 저장 성능을 희생하고 읽기 속도를 높이는 기능
- DBMS의 인덱스도 인덱스가 많은 테이블은 INSERT, UPDATE, DELETE 처리가 느리다. 하지만 SELECT는 빠르게 처리


#### DB replication? 사용하는 이유?

- Replication이란 두 개 이상의 DBMS 시스템을 Master와 Slave로 나눠 동일한 데이터를 저장하는 방식
- 그냥 복붙 아닌가? SPOF를 방지하기 위함이 아닐까?

#### sharding

- 같은 테이블 스키마를 가진 데이터를 다수의 데이터베이스에 분산하여 저장하는 방법을 의미합니다.
- Horizontal Partitioning이라고 볼 수 있습니다.
- 샤딩(Sharding)을 적용한다는 것은?
    - 프로그래밍, 운영적인 복잡도는 더 높아지는 단점이 있습니다.
- 가능하면 Sharding을 피하거나 지연시킬 수 있는 방법을 찾는 것이 우선되어야 합니다.
    - Read 부하가 크다면?
        - Cache나 Database의 Replication을 적용하는 것도 하나의 방법입니다.
    - Table의 일부 컬럼만 자주 사용한다면?
        - Vertically Partition도 하나의 방법입니다.

#### 모델링 절차

- 업무파악 -> 개념적 모델링(ERD) -> 논리적 모델링(ERD) -> 물리적 모델링(코딩)

## Deploy

### 제가 만든 서버가 동시접속자를 몇명이나 견딜 수 있는지 어떻게 아나요?

- 현업에선 품질팀에서 성능검증을 통해 품질 검증 후 상용망에 적용합니다.
- Jmeter같은 simulator 활용하여 부하테스트
- [AWS 기반 부하 테스트](https://aws.amazon.com/ko/blogs/korea/how-to-loading-test-based-on-aws/)

### 웹서비스를 실제 운영하다 보면 크로스스크립트, 클릭 인젝션 같은 공격이 들어올 때가 있다던데 AWS에서 막을 방법이 있을까요?

- 어플리케이션 단의 공격 이전 인프라 단에서 막는 장비들이 있습니다.
- AWS shield(DDoS), WAF(Web Application Firewall), CloudFront(CDN)
- Market place에서 UTM솔루션 구매(Sophos, Fortinet 등)
- DNS 제공해주는 Gabia, Cloudflare 등을 이용하면 WAF 제공

### 질문들

- 대규모 트래픽 처리한 경험?
- Docker, 사용이유? Container, Image
- AWS Lambda?
- 람다 함수와 파라미터 관련 event, context
- Kubernetes
- Serverless
- 쓰로틀
- Firebase
- Reverse proxy
- load balancer
- MSA
- 배포 절차

## DevOps

### Elastic Load Balancing으로 트래픽 분산해보기

- EC2 -> Scale UP -> 시스템 중단 문제 -> Scale Out
- Elastic Load Balancing
- Classic Load Balancer
- Application Load Balancer
    - HTTP, HTTPS
- Network Load Balancer
    - TCP, UDP

### Instance Schedular

- scheduler-cli
- hibernate
- cross-account

### 소프트웨어 생명 주기 관련 데브옵스의 역할, 하는 일

- 개발과 운영(서버, DB, 네트워크)이 하나로
    - 소규모 업데이트 자주, 마이크로 서비스
- 개발 라이프사이클을 이해하고 개발과 운영 양쪽에 충분한 지식과 경험을 보유한 사람?

#### CI/CD

- 지속적 통합 지속적 전달

    - 지속적 통합은 자동화된 빌드 및 테스트가 수행된 후, 개발자가 코드 변경 사항을 중앙 리포지토리에 정기적으로 병합하는 데브옵스 소프트웨어 개발 방식
    - 지속적 전달(Continuous Delivery)은 프로덕션에 릴리스하기 위한 코드 변경이 자동으로 준비되는 소프트웨어 개발 방식
        - 좀더 자주, 작은 단위까지 배포

## 암호화

### JWT 구성요소 세가지

- 헤더.페이로드.서명
    - 헤더- 토큰타입, 암호화 알고리즘
    `{
        "typ": "JWT",
        "alg": "HS256"
    }`
        - base64로 인코딩
    - 페이로드 - 클레임 정보
        - 1. 등록된 클레임
            - `iss`: 토큰 발급자(issuer)
            - `sub`: 토큰 제목(subject)
            - `aud`: 토큰 대상자(audience)
            - `exp`: 토큰 만료기간(expiration), 시간은 NumericDate형식(예:1480849147370)
            - `nbf`: Not Before를 의미, 토큰 활성날짜
            - `idt`: 토큰이 발급된 시간(issued at), 이 값을 통해 토큰의 age가 얼마나 되었는지 판단 가능
            - `jti`: JWT 고유 식별자로 중복처리 방지를 위해 사용, 1회용 토큰에 유용
        - 2. 공개(Public) 클레임
            - 충돌 방지를 위해 URI 형식
            `{
            "http://" : true
            }
            `
        - 3. 비공개(private) 클레임
            - 클라이언트와 서버 협의하에 사용되는 클레임 네임, 공개 클레임과 달리 중복 충돌 가능성이 있으므로 유의
            `{
            "username": "yongjun"
            }
            `
                - base64로 인코딩
    - 서명 - 헤더의 인코딩값과 정보의 인코딩값을 합친 후 시크릿키로 해시 적용
        - 문자열 인코딩이 아닌 hex -> base64 인코딩

- base64 인코딩 시 = 문자(패딩)가 붙으면 url로 쓸 수 없으므로 지워야 한다.
- [JWT 디버거](https://jwt.io/) 에서 토큰을 디코딩 테스

### Access Token, Refresh Token

액세스 토큰의 짧은 유효기간은 잦은 로그인을 야기하고 이는 곧 보안 취약점이다. 이에 액세스 토큰 만료 시 리프레시 토큰을 발행해 액세스토큰을 보완한다.
라고 하지만 아니라고 하는 사람도 있다.

- 액세스 토큰은 보통 세션에 저장
    - 액세스 토큰과 달리 리프레시토큰은 일반적으로 회원 DB에 저장
- 보통 SPA에서는 리프레시 토큰을 제공하지 않는다. 리프레시 토큰으로 액세스 토큰을 영원히 갱신할 수 있으므로

### 페북/인스타는 로그인 만료기간이 엄청 긴데, 어떻게 보안 관리를 하는지 아는가?

- 리프레시 토큰의 만료기간이 일정 기간 남았을 때(카카오- 리프레시토큰 유지기간 60일, 갱신기간 1개월) 갱신

### [JWT 변조 공격 어떻게 대처 하겠는지?](https://code-machina.github.io/2019/09/01/Security-On-JSON-Web-Token.html)

- JWT는 비밀키를 사용하여 서명되는데 이 때 HMAC 알고리즘 사용
    - 또한 RSA를 이용한 공개키/비밀키 쌍을 이용할 수 있다.

#### None 해싱 알고리즘

- 공격자가 해싱 알고리즘을 none으로 변경
- 몇몇 알고리즘은 none 알고리즘으로 서명한 토큰을 유효한 토큰으로 인식
- 공격자가 RSA 서명을 HMAC 알고리즘으로 변환해 퍼블릭키를 통해 복호화, 암호화하면 재서명이 가능
- 예방책
    - 아래 라이브러리 주의?
        - node-jsonwebtoken
        - pyjwt
        - namshi/jose
        - php-jwt
        - jsjwt

#### 토큰 하이재킹

- 토큰 가로채기
- 예방책
    - 사용자 컨텍스트를 토큰에 생성
        - 무작위 문자열의 SHA256해시 토큰 내 포함
        - HttpOnly, Secure, SameSite, Cooke prefixes와 같은 보안 설정
            - `HttpOnly` 쿠키는 자바스크립트의 `Document.cookie` API를 통해 접근이 불가능하다. 이들은 오로지 서버로만 전송된다. 예를 들어 서버사이드 세션은 자바스크립트가 접근할 필요가 없다. 이에 `HttpOnly` 플래그가 반드시 설정되어야 한다.
            - `Secure` 쿠키는 HTTPS 프로토콜을 통한 암호 요청만을 통해 전송할 수 있따. 이를테면 Http 통신을 하는 안전하지 않은 사이트는 Secure 디렉티브로 설정된 쿠키를 구울 수 없다.
            - `SameSite` 쿠키는 서버로 하여금 쿠키가 교차 도메인 전송을 차단하게 만든다. CSRF 보안을 제공한다.
            - 쿠키 접두사는 브라우저에 특정 속성이 필요하다고 전달한다. `__Secure-`가 대표적인 예이며 `__Host-`가 prefix 설정된 경우 `Path=/`와 `Secure` 속ㅈ성이 모두 필요하다고 브라우저에게 알려준다.

#### 사용자에 의한 명시적 토큰 철회/취소

#### 토큰 정보 노출

## Bcrypt 작동원리? 왜 사용? 단점?

- SHA2, SHA3는 너무 빨라서 레인보우 테이블(해시값 검색에 최적화된 테이블)도 빠르게 만들 수 있지만 Bcrpyt는 속도 조절 가능

### JWT 인증 방식 vs 세션 인증 방식

- 애플리케이션을 완전히 Stateless한 상태로 유지할 필요가 없다면 세션 시스템을 사용

### 암호화 Tool(Bcrypt, Argon2) 를 사용하는 이유?

### JWT를 활용하여 이상적인 Scalable한 인증서버를 만들려고 하고자 할때 어떻게 구성을 할 것인가?

## Django

### ORM과 쿼리 중 무엇을 선호? 차이점?

### Why Django?

- 배터리팩, Build Fast
- Simple syntax, MVC Core 아키텍처, ORM, 미들웨어 서포트, 유닛 테스트
- 확장성 있는 프로젝트 적합, 큰 트래픽과 데이터 핸들링 가능
- 보안 - 클릭 인젝션, SQL 인젝션 등 방지 용이

- MVC 패턴에 대하여(Django MVT 방식대로 설명)
- Django vs Flask
- Django vs Java Spring
- 같은 API에 Get요청을 여러 번 보내서 똑같은 요청을 할 경우, 효율성을 증대시키는 방법이 있는데 장고는 이를 자동으로 해준다. 이게 뭔지 알고있는지?(정답은 캐싱)
- DRF
- APP 어떤 기준으로 분리했는지?
- 클래스 인스턴스 활용
- 일대다, 다대다 관계
- request 요청시 동작 순서
- Django 장단점
- test.py, 유닛테스트, 사용한 모듈
- 로컬 DB와 서버 DB의 구조 차이로 인한 migrate가 conflict 날 경우 해결법?

## Python

- 데코레이터, 이터레이터, 제너레이터

### Decorator 사용시 주의점

- 데코레이터는 함수를 빠르게 변경할 때 사용 가능

### 파이썬 메모리 관리

- 개별적인 힙을 사용해서 메모리를 유지, 내부에 힙 매니저 구현
- 인터프리터가 접근, 프로그래머는 사용할 수 없음
- 빌트인 가비지 컬렉터를 통해 사용되지 않는 메모리의 힙공간을 관리

### Call by value, Call by reference

- Python은 passed by assignment
    - 불변 타입 -> Call by value
    - 가변 타입 -> Call by reference

- Call by value - 인자로 받은 값을 복사해서 처리, 복사하므로 메모리 증가
- Call by Reference - 인자로 받은 값의 주소를 참조해 직접 값에 영향을 준다. 값에 영향을 주는 대신 빠르다.

### Iterators는 무엇?

- 배열과 같은 객체로서 다음 요소로 이동하는 것이 가능하다. `next()` 메소드로 데이터를 순차적으로 호출한다.

### Iterators와 Iterable의 차이는?

- iterable은 하나씩 차례로 반환 가능한 객체. __iter__, __getitem__ 메소드로 정의된 클래스들이 iterable한 것.
- iterable을 iterator로 변환하고 싶다면 iter() 함수를 사용

### Closures?

- 함수 객체로서 다른 함수를 반환
- 일반 함수와 다르게 자신의 영역 밖에서 호출된 함수의 변수값과 레퍼런스를 복사, 저장, 접근 가능하게 한다

### 질문

- Why Python?
- 파이썬 가비지콜렉터 동작방식, 그리고 집적 구현한다면 어떤식으로 하겠는가?
- GCC(GNU Compiler Collection)이란 (파이썬 엔진과 연관지어서)
- 파이썬 인터프리터가 기존 컴파일러 언어에 비해서 퍼포먼스가 낮은데도 불구하고 파이썬이 주언어인 이유?

- 플랫폼 독립적 언어, 개발 기간 단축에 초점을 맞춘 언어 
예- 윈도우에서 개발한 코드로 맥과 리눅스에서 실행 가능 - OS에 인터르리터가 설치돼 있으므로, C는 컴파일 다시 진행해야

- 스크립트 언어 vs 컴파일 언어
- 디스크럽터
- 정규표현식
- 클래스 상속 시 메서드 실행 방식
- PyPy가 CPython보다 빠른 이유
- (JS와 비교하여) python의 장단점
- 비동기/동기 설명, 왜 쓰는지?
- 코루틴 설명, 왜 쓰는지?
- PEP8이 무엇인지?
- Python2와 Python3의 차이
- 프로젝트에서 이터레이터, 제너레이터 사용했는지
- 데코레이터 프로젝트 어디서 사용했는지, 왜 썼는지?
- 디스크립터가 무엇?
- Private Class, Public Class?

## API

### 서로 다른 API로 요청이 동시에 들어오면서 해당 API들이 내 DB의 같은테이블 하나를조회한다고 가정했을때, 병목현상이 생길텐데 어떻게 처리할 것인가?

### 이미지 업로드 어떻게 하는지?

### 자동 문서 도구 툴

### Graphql과 RestfulAPI를 비교했을 때 Graphql의 장단점은?

## Network

### HTTP/3

- TCP
    - 번거로운 3 Way Handshake
        - HTTP/1 -> HTTP/2 핸드쉐이크 횟수 최소화
    -  HOLB(Head of line Blocking)
        - 주고받은 시퀀스 번호를 참고해 정확한 순서대로 패킷을 재조립해야 한다.
        - 손실은 그냥 넘어가지 않고 다시 보내야 다음 단계 진행 가능
        - 이러한 이유로 발생하는 병목현상
- HTTP/3 - QUIC(Quick UDP Internet Connection)
    - TCP 문제해결하고 레이턴시 한계를 뛰어넘고자 구글이 개발
    - 순서가 존재히자 않는 독립적 패킷 사용
    - 목적지만 정해져 있다면 중간에 어딜가도 상관 없음, 핸드쉐이크 과정 필요 없음
        - RTT(Round Trip Time) - 3RTT(TCP) -> 1RTT(QUIC)
            - 첫번쨰 핸드쉐이크에 필요한 정보와 데이터까지 전송
    - 커스터마이징 용이
        - 꽉 들어찬 TCP의 헤더와 달리 아무것도 없는 UDP의 헤더
    - 재전송 모호성을 해결하기 위해 패킷의 헤에 별도 패킷 번호 공간 부여
    - 클라이언트 IP가 바뀌어도 연결이 유지됨
        - TCP 연결 끊어지면 다시 3 Way Hanshake거쳐야
        - 예) wi-fi에서 셀룰러로 전환
        - QUIC은 Connection ID를 사용해 서버와 연. 이는 랜덤한 값을 뿐 클라이언트 IP와는 무관


### 질문들

- web server(Apache, Nginx)? 장점?
- 
- HTTP 1.0/1.1/2.0/3.0의 특징 (Keepalive -> TCP/IP -> Header Compression -> UDP)
- 웹브라우저 동작원리(프론트 ->브라우저 -> 백엔드 -> DB -> OS 순으로)
- HTTP vs HTTPS
- HTTP 요청/응답 헤더
- TCP/IP와 UDP의 특징과 차이에 대해. 각각의 header
- localStorage, sessionStorage, Cookies / 그 외에 브라우저에 저장하는 방법?
- url에 [google.com](http://google.com) 주소 치면 어떤 과정 생기나?
- 단순히 글을 조회하지만, 글을 조회할때 조회수가 올라간다 이럴때 http 메소드는 뭘써야 하겠는가?
- HTTP request 구성
- Response code
- 500대 코드 에러 처리
- 서버가 죽으면 어떻게 500 에러를 내는 것인가
- www
- protocol
- DNS
- 크로스 브라우징
- 웹 성능과 관련된 이슈
- 서버 사이드 렌더링, 클라이언트 사이드 렌더링
- HTTP 인증 방식
- 데이터 주고받을때 json형식밖에 없나요? 왜 제이슨을 쓰나요
- 이미지나 데이터 로딩이 많을때 로딩시간 최적화 방법은?
- Socket.io vs WebSocket
- 동기 vs 비동기
- 비동기 처리는 왜 하나?
- CORS 통신
- GET, POST 메서드
- Frame, Packet, Segment, Datagram

## Git

- git, github
- 협업 시 git 관리 어떻게 했는가?
- 코드 리뷰를 해본 적이 있는지?
- Git Flow / branch 관리
- git rebase / squash
- git branch 어떻게 따는가?
- 이미 서비스가 런칭된 상태인데 기획자가 바뀐 기획의도를 가지고 웹서비스를 다시 만들라고 한다면 git을 어떻게 사용할 것인가?
- Git Conflict 상황에서 해결 방법

## 자료구조

- FIFO
- FIFO 반대 개념?
- Set / Set을 쓰면 좋은 예시?
- 숫자가 정리되지 않은 리스트가 있고, 해당 리스트에 특정 값이 있는지 확인해야한다. 효율적인 방법을 제시해 보아라 (정답은 binary tree)
- List의 resizing이 언제 일어나는가? 크기가 어떻게 변하는가? 두배로 늘린다고 하면 문제점? 얼마만큼의 분량을 재할당 하는게 효율적인가?
- Tree vs Graph. 깊이 탐색과 너비 탐색
- Array vs Linked list
- HashTable
- Stack
- Queue
- Binary Heap
- Red-Black Tree
- B+ Tree

## 운영체제

### 프로세스와 스레드 차이

- 프로세스 : 자신만의 고유 공간과 자원을 할당받아 사용
- 스레드 : 다른 스레드와 공간과 자원을 공유하면서 사용

- 프로세스 메모리상에서 실행 중인 프로그램의 인스턴스를 말하며, 스레드는 이 프로세스 안세서 실행되는 흐름의 단위다.
    - 프로세스마다 최소 하나의 스레드를 가지며 주소공간을 할당받는다(code, data, heap, stack).
    - 스레드는 이중에 stack만 할당받고 나머지 영역은 스레드끼리 공유한다.


### 멀티 프로세스로 처리 가능한데 멀티 스레드로 처리하는 이유?

- 프로세스를 생성해 자원을 할당하는 시스템 콜이 감소하면서 자원의 효율적 관리가 가능하다.
- 프로세스간 통신(IPC)보다 스레드 사이 통신 비용이 적다.
- 하지만 멀티스레드는 공유자원으로 발생하는 문제 해결을 위해 동기화에 신경써야 한다.


## 기타

- CDN 써보았는지? 개념은?
- 정렬에 대해서 아는가?
- 총 4대의 서버가 존재하고 동시에 크론탭을 돌릴 때 어떻게 처리?
- Scrum 방식
- 레거시 코드
- Scale up / Scale out
- 인증/인가
- 시멘틱 마크업? 왜 준수해야 하나?
- 객체 지향 프로그래밍
- 함수형 프로그래밍
- 레이지 로딩
- 클린아키텍쳐
- 부하테스트
- TDD
- Agile
- 0.2 + 0.1의 결과값?
- Reference Error
- Garbage collector
- Serializer
- Linux, 명령어
- CRUD
- IDE
- Compiler VS Interpreter
- Thread process
- PIG란(Apache PIG인듯)
- Hash, Hash 충돌, 해결책
- 추상화란?
- Call by Reference Vs. Call by Value
- PWA
- GC 작동 방식
- 메모리 누수가 발생할 수 있는 경우
- Duck Typing
- 딥러닝
- ML
- AI
- IoT
- 분산락
- 64bit CPU와 32bit CPU 차이
- CVS, SVN, Git
- 웹 서버(Web Server)와 웹 어플리케이션 서버(WAS)의 차이
- Servlet과 JSP
- Maven과 Gradle의 차이 
  
  
## Link

- [gabia 라이브러리](http://library.gabia.com/contents/infrahosting/1222)
- [Database의 샤딩이란?](https://nesoy.github.io/articles/2018-05/Database-Shard)
- [JSON Web Token이 뭘까?](https://velopert.com/2389)
- [Cache Redis vs. Memcached](https://medium.com/@chrisjune_13837/redis-vs-memcached-10e796ddd717)
- [Python 은 call-by-value 일까 call-by-reference 일까](https://www.pymoon.com/entry/Python-%EC%9D%80-callbyvalue-%EC%9D%BC%EA%B9%8C-callbyreference-%EC%9D%BC%EA%B9%8C)
- [Python 기술 면접 대비](https://shelling203.tistory.com/31)
- [JSON 웹 토큰 보안](https://code-machina.github.io/2019/09/01/Security-On-JSON-Web-Token.html)
- [강대명님 강의](https://www.slideshare.net/charsyam2/presentations)
- [HTTP/3는 왜 UDP를 선택한 것일까?](https://evan-moon.github.io/2019/10/08/what-is-http3/)
