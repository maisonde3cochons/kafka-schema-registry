## [schema registry]
--------------------------------

#### [주의사항!!! docker-compose 파일을 WLS2에서 실행 시 주요 이슈가 발생 부분]

##### 1. docker-compose 실행 path가 windows directory일 경우 volume mount(mysql/kafka/zookeeper) 문제가 발생한다
> 반드시 avoid!! <b>/mnt/c/ </b> 나 <b>/mnt/d/ </b> 에서 실행하지 않는다

##### 2. docker-compose 실행 후 생성되는 kafka/zooker의 권한을 꼭 확인한다
> docker container와 mount되는 path directory가 root권한으로 생성 되는 경우가 많다.
> 꼭 사전에 kafka/logs, zookeeper/data 폴더 생성 후 실행 user 권한으로 변경해준다
```
mkdir -p kafka/logs 
mkdir -p zookeeper/data
chown -R user:user zookeeper kafka
```

##### 3. /etc/hosts를 확인해서 사용할 도메인이 등록되어 있는지 확인한다
```
127.0.0.1 kafka zookeeper
```

##### 4. docker-compose 실행 후 docker container에 mount된 volume 권한을 변경하지 않는다
> docker-compose 재실행 시 권한 문제가 발생한다

![image](https://user-images.githubusercontent.com/30817824/170620369-16000fab-b9e1-47af-b95b-93e1cebf4282.png)


-----------------------

Producer에서 필드(삭제 또는 추가)변경이 발생할 때 Consumer에서 이것을 참조하고 있다면 문제가 발생할 수 있음<br/>
<b>(Kafka Broker에서 데이터 유효성 검증을 따로 안 하기 때문)</b><br/>
사고를 사전에 방지하기 위해 message schemas 유효성에 대해서 상호 보증할 수 있는 필요가 생김<br/>
이런 필요를 충족시켜주는 것이 schema registry!!<br/>

<b>How? :</b> Schema Registry는 동일 스키마에 대한 호환성 체크를 하기 위해 버전을 유지

- backward : 필드삭제, optional 필드 추가 허용 - 컨슈머부터 업그레이드
- forward : 필드 추가, optional 필드 삭제 허용 - 프로듀서부터 업그레이드
- Full : Backward, Forward 모두 만족
- None : Backward, Forward 모두 만족하지 않음


* avro :  apahche hadoop에서 개발한 Data Serializing & RPC F/W 
        json으로 schema작성하고 Binary로 Serializing 시켜준다

#### schema registry는 kafka broker / zookeeper와 별도 서버로 구성됨 <br/>
producer/consumer는  매번 schema registry에 schema를 전송하는 것은 아니고, <br/>
로컬 cache를 가지고 있다. 자신이 로컬 캐시에 schema를 가지고 없을 때만 schema registry와 통신한다.<br/>
schema registry에 문제가 생기면 심각한 문제 발생!!

-------------------------------

> #### /etc/hosts
```
127.0.0.1       kafka1  schemaregistry1 restproxy1
```

> #### docker-compose-confluent-schema-registry-and-rest-proxy.yml 실행
```
docker-compose -f docker-compose-confluent-schema-registry-and-rest-proxy.yml up
```

<b>schema_registry_and_rest_proxy_examples.txt</b> 참고하여 REST API test 진행
