# confluent-kafka

### Install Kafka

Confluent Kafka platform 7 을 다운로드 후 설치 합니다.   
https://docs.confluent.io/platform/current/installation/installing_cp/zip-tar.html

Confluent Platfrom Zip download
`````
curl -O http://packages.confluent.io/archive/7.0/confluent-7.0.0.zip
`````

설치할 경로에 다운로드한 파일을 압축 해제 합니다.
`````
$ unzip confluent-7.0.0.zip
``````


### Start Kafka
bash 등에 설치 경로를 PATH 로 하여 줍니다.

````
export CONFLUENT_HOME=/data/confluent-7.0.0
export PATH=.:~/.local/bin:$CONFLUENT_HOME/bin:$PATH
````

Kafka 를 시작 합니다.
`````
$ confluent local services start
The local commands are intended for a single-node development environment only,
NOT for production usage. https://docs.confluent.io/current/cli/index.html

Using CONFLUENT_CURRENT: /tmp/confluent.503949
Starting ZooKeeper
ZooKeeper is [UP]
Starting Kafka
Kafka is [UP]
Starting Schema Registry
....

``````


Kafka 정지는 다음 명령어로 합니다.
`````
$ confluent local services stop
``````

### DynamoDB Source  & MongoDB Connector install
kafka-connect-dynamodb-1.0.0.jar 파일을 복사
`````
$ cp kafka-connect-dynamodb-1.0.0.jar $CONFLUENT_HOME/share/java/kafka/
````

MongoDB Connector 설치    
Confluent-hub 에서 제공하는 kafka connector 를 설치 합니다.

````
$ confluent-hub install mongodb/kafka-connect-mongodb:1.6.1
````

### DynamoDB 준비
Migration 대상 DynamoDB table 에 사전 준비 사항을 설정 합니다.
Stream enable (Exports & Stream > DynamoDB stream details)    

<img src="/image/image03.png" width="90%" height="90%"> 

Tag 정보에 migration 대상으로 설정 하기 위해 약속된 Tag 를 추가 하여 줍니다.
Environment = dev

### Connector 생성
설치된 Connector를 이용하여 Producer, Consumer 를 생성 합니다.   
Connector Setup 을 위한 JSON 파일을 다운로드 후 해당 정보를 이용하여 생성 하여 줍니다.    

JSON 파일내에 AWS DynamoDB 접근을 위한 Key 정보를 입력 한 후 진행 합니다.    
DynamoDB Source Producer 생성   
````
$ curl -X PUT -H 'Content-type: application/json' http://localhost:8083/connectors/<<DynamoDB Connect Name>>/config -d @myDynamodbConnector.json
````

JSON 내에 MongoDB 연결 정보를 맞게 수정 하여 준 후 connector 설치를 진행 합니다.    
<<Database>> : Atlas Database 이름    
<<collectionName>> : Atlas Collection 이름    
<<USER>>:<<PASSWORD>> : Atlas User & Password    
<<Internal_IP>> : localhost 혹은 private IP    

MongoDB Sink Consumer 생성
````
$ curl -X PUT -H 'Content-type: application/json' http://localhost:8083/connectors/<<MongoDB Connect Name>>/config -d@mongoDB-sink.json
````

설치가 완료된 Kafka connector 정보    
KAFKA의 관리 콘솔 주소는 다음과 같습니다.    
http://<<KAFKA IP>>:9021/     
콘솔내에서 Connector 정보를 확인 합니다.    
<img src="/image/image01.png" width="90%" height="90%"> 
각 Connector 를 조회 하면 Json 으로 입력한 정보로 생성된 connector를 볼 수 있습니다.

DynamoDB Source Connector는 Source connector로 DyanmoDB를 주기적으로 Polling 하고 읽어야 할 데이터를 읽은 후 Topic에 저장 하는 역할을 합니다. 등록이 정상적으로 진행 된 경우 자동으로 연관된 토픽을 생성 합니다.    

생성된 Connector의 정보는 Connect > connect-default > <<DynamoDB Connect Name>> > Settings 에서 정보를 확인 할 수 있습니다.    
Operation tag key name 이 init 인 것을 확인 하고 Tables whitelist에 Migration 대상 DynamoDB table 이 등록된 것을 확인 합니다.    
<img src="/image/image02.png" width="90%" height="90%">    

토픽은 Topic prefix + DyanmoDB table 이름으로 생성 됩니다.
<img src="/image/image04.png" width="90%" height="90%">    


생성된 MongoDB Connnector 의 정보를 Connect > connect-default > <<MongoDB Connect Name>> > Settings 에서 확인 합니다.    
topics 항목이 생성된 topic 으로 설정 되어 있는지 확인 합니다. (첨부된 Json 은 Temp로 설정 되어 있음으로 temp를 삭제하고 맞는 topic을 선택 합니다)


### DynamoDB Data migration
DynamoDB connector 가 정상 작동하면 주기적으로 대상 테이블을 검색하고 지정된 단위로 데이터를 가져와 토픽에 넣게 됩니다.
<img src="/image/image06.png" width="90%" height="90%">  
