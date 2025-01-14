카프카 커넥트

주로 토픽에 데이터를 넣거나 뺄때 템플릿 형식으로 파이프라인을 반복적인 생성이 가능하다. 

	         	카프카 
		         토 픽

카프카     소스 커넥터 싱크 커넥터   <=== 스레드 프로세스  (REST API 로 실행시키고 중단시킬수 있음)
커넥트

              소스                싱크 
        애플리케이션    애플리케이션

카프카 커텍트(kafka connect)는 카프카 오픈소스에 포함된 툴 중 하나로 데이터 파이프라인 생성 시 반복 작업을 줄이고 효율적인 전송을 이루기 위한
애플리케이션이다. 커넥트는 특정한 작업 형태를 템플릿으로 만들어놓은 커넥터(connector)를 실행함으로써 반복 작업을 줄일수있다. 


커넥트 내부 구조 

카프카 커넥트
 태스트 #0
 태스트 #1
  커텍터 (한개스레드로 태스크를 관리)

사용자가 커넥트에 커넥터 생성 명령을 내리면 커넥트는 내부에 컨넥터와 태스크를 생성한다. 커넥터는 태스크들을 관리한다.
태스크는 커넥터에 종속되는 개념으로 실질적인 데이터 처리를 한다. 그렇기 때문에 데이터 처리를 정상저긍로 하는 지 확인하기 위해서는 각 태스크의
상태를 확인해야 한다. 
커넥터가 태스크 상태를 확인하는등 VALIDATION 역할을 한다.

소스 커넥터, 싱크 커넥터 크게 2가지가 존재한다.
커넥터는 프로듀서 역할을 하는 '소스 커넥터(source connector)'와 컨슈머 역할을 하는 '싱크 커넥터(sink connector)' 2가지로 나뉜다.
예를 들어, 파일을 주고받는 용도로 파일 소스 커넥터(file source connector)와 파일 싱크 커넥터(file sink connector)가 있다고 가정하자.
파일 소스 커넥터는 파일의 데이터를 카프카 토픽으로 전송하는 프로듀서 역할을 한다. 그리고 파일 싱크 커넥터는 토픽의 데이터를 파일로 저장하는 
컨슈머 역할을 한다. 파일 외에도 일정한 프로토콜을 가진 소스 애플리케이션이나 싱크 애플리케이션이 있다면 커넥터를 통해 카프카로 데이터를 보내거나
카프카에서 데이터를 가져올 수 있다.
MySQL, S3, MongoDB 등과 같은 저장소를 대표적인 싱크 애플리케이션, 소스 애플리케이션이라 볼 수 있다. 즉, MySQL에서 카프카로 데이터를 보낼 때 
그리고 카프카에서 데이터를 MySQL로 저장할 때 JDBC 커넥터를 사용하여 파이프라인을 생성할 수 있다.

커넥트 플러그인
카프카 2.6에 포함된 커넥트를 실행할 경우 클러스터 간 토픽 미러링을 지원하는 미러메이커2 커넥터와 파일 싱크 커넥터, 파일 소스 커넥터를 
기본 플러그인으로 제공한다. 이외에 추가적인 커넥터를 구현하는 클래스를 빌드한 클래스 파일이 포함되어 있다.
커넥터 플러그인을 추가하고 싶다면 직접 커넥터 플러그인을 만들거나 이미 인터넷상에 존재하는 커넥터 플러그인을 가져다 쓸 수도 있다. 

오픈소스 커넥터 
오프손스 커넥터는 직접 커넥터를 만들 필요가 없으며 커넥터 jar파일을 다운로드하여 사용할 수 있다는 장점이 있다. 
HDFS 커넥터, AWS S3 커넥터, JDBC 커넥터, 엘라스틱서치 커넥터 등 100개가 넘는 커넥터들이 이미 공개되어 있다.
필요한 커넥터를 검색하고 아키텍처에 알맞는 커넥터를 찾아서 다운받아 사용하면 된다.
오픈소스 커넥터의 종류는 컨플루언트 허브(https:///confluent.io/hub/)에서 검색할 수 있다. 다만, 오픈 소스라고 모두 무료로 제한없이 사용할 수 있는
것은 아니기 때문에 라이선스를 참고하여 사용 범위를 확인한 이후에 사용해야 한다. 

컨버터, 트랜스폼 (추가적인 플러그인)
사용자가 커넥터를 사용하여 파이프라인을 생성 할 때 컨버터(converter)와 트랜스폼(transform) 기능을 옵션으로 추가할 수 있다.
커넥터를 운영할 때 반드시 필요한 설정을 아니지만 데이터 처리를 더욱 풍부하게 도와주는 역할을 한다.
컨버터는 데이터 처리를 하기 전에 스키마를 변경하도록 도와준다.
JsonConverter, String Converter, ByteArrayConverter를 지원하고 필요하다면 커스텀 컨버터를 작성하여 사용할 수도 있다.
트랜스폼은 데이터 처리 시 각 메시지 단위로 데이터를 간단하게 변환하기 위한 용도로 사용 된다.
예를 들어, JSON 데이터를 커넥터에서 사용할 때 트랜스폼을 사용하면 특정 키를 삭제하거나 추가할 수 있다.
기본 제공 트랜스폼으로 Cast, Drop, ExtractField 등이 있다.

커넥트를 실행하는 방법
커넥트를 실행하는 방법은 크게 두가지가 있다. 단일 모드 커넥트 (standalone mode kafka connect) 이고 두 번째는 분산 모드 커넥트(distributed mode kafka connect)이다. 단일 모드 커넥트는 단일 애플리에키엿느로 샐행된다. 커넥터를 정의하는 파일을 작성하고 해당 파일을 참조하는 단일 모드 커넥트를 실행함으로써 파이프라인을 생성 할 수 있다. 

단일 모드 커넥트 
단일 모드 커넥트는 1개 프로세스만 실행되는 점이 특징인데, 단일 프로세스로 실행되기 때문에 고가용성 구성이 되지 않아서 단일 장애점 failover가 되지 않는다. 그러므로 단일 모드 커넥트 파이프라인은 주로 개발환경이나 중요도가 낮은 파이프라인을 구성할때 활용

분산 모드 커넥트 (2~100개 이상 커넥트 운영)
분산 모드 커넥트는 2대 이상의 서버에서 클러스터 형태로 운영함으로써 단일 모드 커넥트 대비 안전하게 운영할 수 있다는 장점이 있다.
2개 이상의 커넥트가 클러스터로 묶이면 1개의 커넥트가 이슈 발생으로 중단되더라도 남은 1개의 커넥트가 파이프라인을 지속적으로 처리할 수 있다.
분산 모드 커넥트는 데이터 처리량의 변화에도 유연하게 대응할 수 있따. 커넥트가 실행되는 서버 개수를 늘림으로써 무중단으로 스케일 아웃하여 
처리량을 늘릴 수 있기 때문이다. 이러한 장점이 있기 때문에 상용환경에서 커넥트를 운영한다면 분산 모드 커넥트를 2대 이상으로 구성하고 설정하는 
것이 좋다.

커넥트 REST API 인터페이스 
REST API를 사용하면 현재 실행 중인 커넥트의 커넥터 플로그인 종류, 태스크 상태, 커넥터 상태 등을 조회할 수 있다.
커넥트는 8083 포트로 호출할 수 있으며 HTTP 메서드 기반 API를 제공한다.

GET        /            						실행 중인 커넥트 정복 확인
GET       /connectors 						실행 중인 커넥터 이름 확인
POST     /connectors/						새로운 커넥터 생성 요청
GET      /connectors/{커넥터 이름}				실행 중인 커넥터 정보 확인
GET      /connectors/{커넥터 이름}/config		실행 중인 커넥터의 설정값 확인
PUT      /connectors/{커넥터 이름}/config                실행 중인 커넥터 설정값 변경 요청
GET      /connectors/{커넥터 이름}/status			실행 중인 커넥터 상태 확인
POST    /connectors/{커넥터 이름}/restrat		실행 중인 커넥터 재시작 요청


단일 모드 커넥트 설정 
단일 모드 커넥트를 실행하기 위해서는 단일 모드 커넥트를 참조하는 설정 파일인 connect-standalone.properties 파일을 수정해야 한다 .

단일 모드 커넥트의 실행 (파일 단위로 실행하게 된다 connector.class 옵션에 커넥터 클래스를 지정)
단일 모드 커넥트를 실행 시에 파라미터로 커넥트 설정파일과 커넥터 설정파일을 차례로 넣어 실행 한다.


*** 분산 모드 커넥트
분산 모드 커넥트는 단일 모드 커넥트와 다르게 2개 이상의프로세스가 1개의 그룹으로 묶여서 운영된다.
이를 통해 1개의 커넥트 프로세스에 이슈가 발생하여 종료되더라도 살아있는 나머지 1개 커넥트 프로세스가 커넥터를 이어받아서 파이프라인을 
지속적으로 실핼 할 수 있다는 특징이 있다. 이제 분산 모드 커넥트를 묶어서 운옇하기 위해 어떤 설정을 해야 하는지 분산 모드 설정 파일인 
connect-distributed.properties를 살펴보자. 해당 파일은 카프카 바이너리 디렉토리의 config 디렉토리에 있다.

옵션값중 topic에 대해서 다른 그룹이라면 connect- ~ 시리즈의 토픽이름이 겹치지 않도록 한다. 

커스텀 소스 커넥터 

소스 커넥터는 소스 애플리케이션 또는 소스 파일로부터 데이터를 가져와 토픽으로 넣는 역할을 한다.
오픈소스 소스 커넥터를 사용해도 되지만 라이선스 문제나 로직이 원하는 요구사항과 맞지 않아서 직접 개발 해야 하는 경우도 있는데
이때는 카프카 커넥트 라이브러에서 제공하는 SourceConnector와 SorcurceTask 클래스를 사용하여 직접 소스 커넥터를 구현하면 된다. 
직접 구현한 소스 커넥터를 빌드하여 jar파일로 만들고 커넥트를 실행 시 플러그인으로 추가하여 사용할 수 있다. 
토픽에 저장하는 프로듀서 역할과 동일한 역할을 한다. 실질적인 역할을 하는 소스 태스크를 개발 한다. 

커스텀 소스 커넥터 디펜던시 
소스 커넥터를 만들 떄는 connect-api 라이브러리를 추가해야 한다. connect-api 라이브러리에는 커넥터를 개발하기 위한 클래스들이 포함되어 있다. 
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>connect-api</artifactId>
        <version>2.5.0</version>
    </dependency>

SourceConnector, SourceTask 총 2가지를 개발하여야 한다.
 SocurceConnector (기본적으로 한개의 스레드에서 실행)
 태스크를 실행하기 전 커넥터 설정파일을 초기 화 하고 어떤 태스크 클래스를 사용할 것인지 정의하는 데에 사용
 실질적인 데이터를 다루는 부분은 들어가지 않는다.

 SourceTask
 소스 애플리케이션 또는 소스 파일로부터 데이터를 가져와서 토픽으로 데이터를 보내는 역할을 수행한다.
 SourceTask만의 특징은 토픽에서 사용하는 오프셋이 아닌 자체적으로 사용하는 오프셋을 사용한다는 점이 있다.
 SourceTask에서 사용하는 오프셋은 소스 애플리케이션 또는 소스 파일을 어디까지 읽었는지 저장하는 역할을 한다.
 이 오프셋을 통해 데이터를 중복해서 토픽으로 보내는 것을 방지할 수 있다. 예를 들어, 파일의 데이터를 한 줄씩 읽어서 토픽으로
 데이터를 보낸다면 토픽으로 데이터를 보낸 줄 번호(line)를 오프셋에 저장할 수 있다.

  커넥터 옵션값 설정시 중요도(Importance) 지정 기준
 커넥터를 개발할 때 옵션값의 중요도를 Importance enum 클래스로 지정할 수 있다. Importance enum 클래스는 HIGH, MEDIUM, LOW 3가지
 종류로 나위어 있다. 결로부터 말하자면 옵션의 Importance를 HIGH, MEDIUM, LOW로 정하는 명확한 기준은 없다. 단지 사용자로 하여금 
 이 옵션이 중요하다는 것을 명시적으로 표시하기 위한 문서로 사용할 뿐이다. 그러므로 커넥터에서 반드시 사용자가 입력한 설정이 필요한 값은
 HIGH,  사용자의 입력값이 없더라도 상관없고 기본값이 있는 옵션을 MEDIUM, 사용자의 입력값이 없어도 되는 옵션을 LOW 정도로 구분하여 지정하면
 된다. importance enum 클래스에 대한 정리 작업은 카프카 내부에서도 논의하고 있다.

 커스텀 싱크 커넥터 (컨슈머 동작과 같음)
 싱크 커넥터는 토픽의 데이터를 타깃 애플리케이션 또는 타깃 파일로 저장하는 역할을 한다.
 카프카 커넥트 라이브러이에서 제공하는 SinkConnector와 SinkTask 클래스를 사용하면 직접 싱크 커넥터를 구현 할 수 있다.
 직접 구현한 싱크 커넥트는 빌드하여 jar로 만들고 커넥트의 플러그인으로 추가하여 사용 할 수 있다. 

 
 분산모드 카프카 커넥트 설정은 config/connect-distributed.properties 파일에서 변경 할 수 있따. 

분산 모드 커넥트 실행 및 플러그인 확인 
./bin/windows/connect-distributed.bat config/connect-distributed.properties

postman에서 get으로 http://localhost:8083/connector-plugins 

FileStreamSinkConnector 테스트 
post 방시긍로 http://localhost:8083/connectors
json 방식으로 값을 넘김

{
	"name" : "file-sink-tesk",
        "config" : {
          "topics" : "test",
          "connector.class": "org.apache.kafka.connect.file.FileStreamSinConnector",
          "tasks.max": 1,
          "file": "/tmp/connect-test.txt"
        }
}


1. config/connect-distributed.properties 설정 변경 
2. ./bin/windows/connect-distributed.bat config/connect-distributed.properties
http://localhost:8083/connectors/file-sink-test/status
 
# 커넥터 플러그인 조회
$ curl -X GET http://localhost:8083/connector-plugins

# FileStreamSinkConnector 생성

$ curl -X POST \
  http://localhost:8083/connectors \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "file-sink-test",
    "config":
    {
	    "topics":"test",
	    "connector.class":"org.apache.kafka.connect.file.FileStreamSinkConnector",
	    "tasks.max":1,
	    "file":"/tmp/connect-test.txt"
    }
  }'

# file-sink-test 커넥터 실행 상태 확인
$ curl http://localhost:8083/connectors/file-sink-test/status

# file-sink-test 커넥터의 태스크 확인
$ curl http://localhost:8083/connectors/file-sink-test/tasks

# file-sink-test 커넥터 특정 태스크 상태 확인
$ curl http://localhost:8083/connectors/file-sink-test/tasks/0/status

# file-sink-test 커넥터 특정 태스크 재시작
$ curl -X POST http://localhost:8083/connectors/file-sink-test/tasks/0/restart

# file-sink-test 커넥터 수정
$ curl -X PUT http://localhost:8083/connectors/file-sink-test/config \
  -H 'Content-Type: application/json' \
  -d '{
	    "topics":"test",
	    "connector.class":"org.apache.kafka.connect.file.FileStreamSinkConnector",
	    "tasks.max":1,
	    "file":"/tmp/connect-test2.txt"
	}'

# file-sink-test 커넥터 중지
$ curl -X PUT http://localhost:8083/connectors/file-sink-test/pause

# file-sink-test 커넥터 시작
$ curl -X PUT http://localhost:8083/connectors/file-sink-test/resume

# file-sink-test 커넥터 재시작
$ curl -X POST http://localhost:8083/connectors/file-sink-test/restart

# file-sink-test 커넥터 삭제
$ curl -X DELETE http://localhost:8083/connectors/file-sink-test

 카프카 커넥트를 운영하기 위한 웹페이지 
 카프카 커넥트는 편리한 rest api 인터페이스를 제공하지만 지속적으로 파이프라인(싱크 커넥터, 소스커넥터)를 운영하는데는 한계가 있어보인다.
 그렇기 때문에 지속적으로 싱크 커넥터, 소스 커넥터를 조회하고 수정 삭제하기 위해서는 웹페이지가 빈드시 필요하다. 커넥트를 운영하기 윈한 웹페이지는
 오픈소스를 사용해도 좋고 직접 만들어서 운영하는 것도 좋다 .
 https://github/com/kakao/kafka-connect-web

커넥트는 프로세스이고 WORKER의 단위


 

 

