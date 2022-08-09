---
layout: post
title: 서비스로그 설계
---
### 로그는 왜 수집해야 하는가?
로그는 사용자의 피드백(클릭, 사용자동선)을 반영하여 서비스에 녹이기 위한 중요한 자료이다. 서비스는 만들어진 순간부터 서비스가 종료되는 그 순간까지 개선과 리펙토링의 연속이다. 사용자의 패턴에 맞게 UX를 변경하고 오류를 개선하는 것은 살아있는 서비스를 만드는 모든 과정 중에 하나이다.
또한 사용자가 서비스를 이용하는중 장애, 강제종료등의 의도하지 않은 오류를 모니터링하여 안정적인 시스템 구축 하고 시스템의 에러로그가 감지되면 slack, 카톡, SMS 통해서 관리자에게 빠르게 노티 하여 문제를 해결할수 있도록 해야 한다.  이런 운영은 최대한 자동화 되어 운영의 비용을 최소화 하고 신규 기능 개발에 전력을 다해야 더 좋은 서비스로 성장하고 개발자나 운영자 역시 운영 리소스를 줄여서 새벽에 일어나는 일이 없어야 할것이다.
로그는 데이터 사이즈가 크고 재수집이 불가능하기 때문에 저장되고 나면 재처리 과정이 쉽지 않다. 그래서 포멧을 잘 정의해야 신규 로그 추가 시 기존 로그 변경없이 분석이 가능하다.

1. 로그 발생
2. 로그 수집
3. 로그 저장
4. 로그 활용

### 서비스 관리, 운영에 필요한 로그

**트랜잭션로그**

엄격하게 관리 해야 하는 로그 비용, 고객의 정보와 직결되는 중요한 로그

**디버깅로그**

서비스에 문제 발생시 혹은 개발중에 상세한 로그를 확인하기 위한 용도인데 서비스중에는 off해야 하는 경우가 많다.

**요청로그**

API에서 받은 요청을 로깅. 요청이 제대로 왔는지 parameter 확인에도 필요

**응답로그**

API에서 응답이 나간 요청을 로깅.  응답코드를 로깅함으로 응답 성공률 들을 분석할수 있다.

**에러로그**

API에서 발생한 오류를 로깅

에러로그를 로깅할때는 발생시간, 오류가 발생된 원인, 소스위치 등을 필터링 해서 표현해야 오류의 원인을  로그에서 바로 찾아 빠르게 복구 가능하다.

**앱로그**

클릭로그, 노출로그, latency로그

클릭로그 : 앱에서 발생된 클릭 정보[ 탭, 섹션, 클릭인덱스, key, keyword ...] 로 구성되어 어떤 페이지에서 어느 정도의 클릭이 발생하는지 사용자가 자주 이용하는 메뉴는 무엇인지 등을 분석

노출로그: 클릭로그를 쓰면 노출로그가 중복되는 경우도 있는데 클릭이벤트가 없이 페이지뷰만 서비스 되는 경우 사용자에게 해당 페이지 얼마나 노출되었는지, 사용자가 해당  페이지에 머문 시간등을 측정

latency 로그 : 서버와 클라이언트 상의 응답시간을 측정. 서버 요청로그에서 API의 처리 시간은 측정이 가능한데 실제 사용자의 체감시간을 네트웍 시간까지 포함한 시간임으로 클라이언트에서 호출한 시간대비 응답시간을 로깅한다.

**JVM  상태 모니터링 로그** - actuator

주요 피쳐 : Process Count , DB connection , GC, Session count 등등

**시스템 모니터링** - **Metricbeat**

주요 피쳐 :  디스크 상태, 메모리상태, CPU 등등

### 로그 저장

logback 라이브러리를 이용하여 파일로 로깅하고 파일은 Logstash를 통해 ES로 저장한다.

ES에 파일을 저장할때는 템플릿을 정의하여 index를 자동으로 생성해줄수 있다.

템플릿을 저장할때는 저장타입을 명시하여 검색용 필드와 exact매칭 필드를 분리하여 사용한다.

### 로그 활용

Grafana Dashboard를 이용하여 로그 모니터링 구현

서비스 대시보드 : request count, response count, 메뉴별 집계, 클릭 발생 항목

시스템 대시보드 : CPU 로드 상태, 메모리 상태, 디스크 사용량

### 로그 레벨 동적 변경 1

logback-spring.xml에 scan 설정 넣으면 설정변경시 application restart됨 내가 원하는 동작이 아님

```java
<configuration scan="true">
```

어플리케이션  restart 없이 로그레벨을 변경하는 방법

```java
// restapi를 이용해서 변경
@GetMapping("change")
public Map<String, String> change(@RequestParam String loggerName, @RequestParam String level) {

    LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
    Logger logger = loggerContext.getLogger(loggerName);

    if (level.equals("info")) {
        logger.setLevel(Level.INFO);
    } else if (level.equals("debug")) {
        logger.setLevel(Level.DEBUG);
    } else if (level.equals("error")) {
        logger.setLevel(Level.ERROR);
    }
}
```

### 로그 레벨 동적 변경 2

actuator에 해당기능이 구현되어 있음

```java

// 현재 로그 레벨 출력
curl -f /actuator/loggers/RESPONSE_LOG

// 로그레벨 변경
curl -XPOST /actuator/loggers/RESPONSE_LOG
  -H "Content-Type: application/json"
  -d {"configuredLevel":"DEBUG"}
```