## [schema registry]
--------------------------------

Producer에서 필드(삭제 또는 추가)변경이 발생할 때 Consumer에서 이것을 참조하고 있다면 문제가 발생할 수 있음
(Kafka Broker에서 데이터 유효성 검증을 따로 안 하기 때문에)
사고를 사전에 방지하기 위해 message schemas 유효성에 대해서 상호 보증할 수 있는 필요가 생김!!
이런 필요를 충족시켜주는 것이 schema registry!!

How : Schema Registry는 동일 스키마에 대한 호환성 체크를 하기 위해 버전을 유지

backward : 필드삭제, optional 필드 추가 허용 - 컨슈머부터 업그레이드
forward : 필드 추가, optional 필드 삭제 허용 - 프로듀서부터 업그레이드
Full : Backward, Forward 모두 만족
None : Backward, Forward 모두 만족하지 않음


avro :  apahche hadoop에서 개발한 Data Serializing & RPC F/W 
        json으로 schema작성하고 Binary로 Serializing 시켜준다

schema registry는 kafka broker / zookeeper와 별도 서버로 구성됨
producer/consumer는  매번 schema registry에 schema를 전송하는 것은 아니고,
로컬 cache를 가지고 있다. 자신이 로컬 캐시에 schema를 가지고 없을 때만 schema registry와 통신한다.
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

> #### schema_registry_and_rest_proxy_examples.txt 참고하여 REST API test 진행