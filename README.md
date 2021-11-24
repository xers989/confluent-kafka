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
````
$ confluent-hub install mongodb/kafka-connect-mongodb:1.6.1
````

### Connector 생성
Connector Setup 을 위한 JSON 파일을 다운로드 후 해당 정보를 이용하여 생성 하여 줍니다.    
JSON 내에 MongoDB 연결 정보를 맞게 수정 하여 준 후 connector 설치를 진행 합니다.    
<<Database>> : Atlas Database 이름    
<<collectionName>> : Atlas Collection 이름    
<<USER>>:<<PASSWORD>> : Atlas User & Password    
<<Internal_IP>> : localhost 혹은 private IP    

MongoDB Connector 설치
````
$ curl -X PUT -H 'Content-type: application/json' http://localhost:8083/connectors/mongoDB-sink/config -d@mongoDB-sink.json
````

JSON 파일내에 AWS DynamoDB 접근을 위한 Key 정보를 입력 한 후 진행 합니다.    
DynamoDB 설치
````
$ curl -X PUT -H 'Content-type: application/json' http://localhost:8083/connectors/myDynamodbConnector/config -d @myDynamodbConnector.json
````

DynamoDB 설치 후 동기화 대상 DynamoDB 테이블 이름을 WhiteList 에 입력 하여 줍니다.    
(사전에 DynamoDB 에 Stream 옵션을 enable 하여 주고 Environment tag 를 추가 하여 줍니다. - dev 값 입력)
DynamoDB connector 가 정상 등록 되면 관련된 Topic 이 생성 되고 초기 동기화를 위한 데이터 로드가 진행 됩니다.   

