[![Docker Pulls](https://img.shields.io/docker/pulls/wurstmeister/kafka.svg)](https://hub.docker.com/r/wurstmeister/kafka/)
[![Docker Stars](https://img.shields.io/docker/stars/wurstmeister/kafka.svg)](https://hub.docker.com/r/wurstmeister/kafka/)
[![](https://images.microbadger.com/badges/version/wurstmeister/kafka.svg)](https://microbadger.com/images/wurstmeister/kafka "Get your own version badge on microbadger.com")
[![](https://images.microbadger.com/badges/image/wurstmeister/kafka.svg)](https://microbadger.com/images/wurstmeister/kafka "Get your own image badge on microbadger.com")
[![Build Status](https://travis-ci.org/wurstmeister/kafka-docker.svg?branch=master)](https://travis-ci.org/wurstmeister/kafka-docker)

kafka-docker(한국어 요약본)
============
[튜토리얼 문서]([http://wurstmeister.github.io/kafka-docker/](http://wurstmeister.github.io/kafka-docker/))를 따라하다보면 분명 안 되는 부분들이 존재한다. 
그 부분들을 보강하기 위하여 코멘트를 일부 추가한다.

##  Pull 받은 후 Dockefile 수정하자.
```
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
  kafka:
    build: .
    ports:
      - "9092"
    environment:
    // Kafka host로 사용할 장비의 IP. 개인 장비에서 돌릴거면 private IP, 그 외는 Public IP가 나와야 한다.
    // 해당 IP를 Kafka host ip라고 부르겠다.
      KAFKA_ADVERTISED_HOST_NAME: 10.64.52.218 
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

```

##  도커 내부에 접속
보통 다음과 같은 키워드로 접속을 할텐데
`docker -it wurstmeister/kafka-docker /bin/bash`

도큐먼트에서 제공하는 방향으로 접속을 하려면 다음과 같이 입력 해야한다.
`$ start-kafka-shell.sh <DOCKER_HOST_IP> <ZK_HOST:ZK_PORT>`
<br>
앞서 수정한 Dockerfile을 바탕으로 위의 스크립트를 사용 하려면 아래와 같이 입력하면 되겠다.
`./start-kafka-shell.sh 10.64.52.218 10.64.52.218:2181`

## 컨슈머 접속 안되는 문제. 
단순 도큐먼트만 본다면 다음 명령어를 입력할 것 같은데,
`$KAFKA_HOME/bin/kafka-console-consumer.sh --topic=topic --zookeeper=$ZK
zookeeper is not a recognized option`

그러면 정확히 아래의 에러를 보게된다.
```
$KAFKA_HOME/bin/kafka-console-consumer.sh --topic=topic --zookeeper=$ZK
zookeeper is not a recognized option
```

이는 [요 문서](https://stackoverflow.com/questions/53428903/zookeeper-is-not-a-recognized-option-when-executing-kafka-console-consumer-sh)를 확인해보면 알 수 있는데
`--zookeeper=10.64.52.218:2181`에서의 `--zookeeper` 옵션은 Deprecated 되었다. 
대신 아래의 명령어로 대체 되었다. 
`--bootstrap-server`

그럼 이런 명령어가 완성이 된다. 
```
$KAFKA_HOME/bin/kafka-console-consumer.sh --topic=topic --bootstrap-server=$ZK
```
위의 명령어를 실행하면 주키퍼에서는 아래와 같은 에러를 무한정으로 뱉는데,

```
zookeeper_1  | 2021-03-24 13:09:14,685 [myid:] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@215] - Accepted socket connection from /172.18.0.1:51072
zookeeper_1  | 2021-03-24 13:09:14,688 [myid:] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:ZooKeeperServer@949] - Client attempting to establish new session at /172.18.0.1:51072
zookeeper_1  | 2021-03-24 13:09:14,692 [myid:] - INFO  [SyncThread:0:ZooKeeperServer@694] - Established session 0x10000068f5b0004 with negotiated timeout 30000 for client /172.18.0.1:51072
zookeeper_1  | 2021-03-24 13:09:14,916 [myid:] - INFO  [ProcessThread(sid:0 cport:2181)::PrepRequestProcessor@487] - Processed session termination for sessionid: 0x10000068f5b0004
zookeeper_1  | 2021-03-24 13:09:14,922 [myid:] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@1056] - Closed socket connection for client /172.18.0.1:51072 which had sessionid 0x10000068f5b0004
zookeeper_1  | 2021-03-24 13:10:15,769 [myid:] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@215] - Accepted socket connection from /172.18.0.1:51090
zookeeper_1  | 2021-03-24 13:10:15,788 [myid:] - WARN  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@383] - Exception causing close of session 0x0: Unreasonable length = 1818570083
zookeeper_1  | 2021-03-24 13:10:15,788 [myid:] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@1056] - Closed socket connection for client /172.18.0.1:51090 (no session established for client)
zookeeper_1  | 2021-03-24 13:10:15,898 [myid:] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@215] - Accepted socket connection from /172.18.0.1:51096
zookeeper_1  | 2021-03-24 13:10:15,899 [myid:] - WARN  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@383] - Exception causing close of session 0x0: Unreasonable length = 1818570083
zookeeper_1  | 2021-03-24 13:10:15,899 [myid:] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@1056] - Closed socket connection for client /172.18.0.1:51096 (no session established for client)
zookeeper_1  | 2021-03-24 13:10:16,003 [myid:] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@215] - Accepted socket connection from /172.18.0.1:51102
zookeeper_1  | 2021-03-24 13:10:16,004 [myid:] - WARN  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@383] - Exception causing close of session 0x0: Unreasonable length = 1818570083
zookeeper_1  | 2021-03-24 13:10:16,005 [myid:] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@1056] - Closed socket connection for client /172.18.0.1:51102 (no session established for client)
zookeeper_1  | 2021-03-24 13:10:16,110 [myid:] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxnFactory@215] - Accepted socket connection from /172.18.0.1:51108
zookeeper_1  | 2021-03-24 13:10:16,112 [myid:] - WARN  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@383] - Exception causing close of session 0x0: Unreasonable length = 1818570083
zookeeper_1  | 2021-03-24 13:10:16,112 [myid:] - INFO  [NIOServerCxn.Factory:0.0.0.0/0.0.0.0:2181:NIOServerCnxn@1056] - Closed socket connection for client /172.18.0.1:51108 (no session established for client)
```

이는 기본 레포에 있는 스크립트를 실행하여 브로커의 정보를 가져오면 된다. 
```
$ ./broker-list.sh
:32772
```
가져온 정보를 바탕으로 다음과 같이 실행하자. 
```
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server 10.64.52.218:32772 --topic "topic" --from-beginning
```

정상 실행된다.


