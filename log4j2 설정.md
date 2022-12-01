# SpringBoot Console Log Setting



## 목차
- [yum의 repository 설치](#yum의-repository-설치)



## Logging 툴의 종류

- ### commons-logging : Spring3에서 사용하던 logging툴

- ### log4j : 효율적인 메모리 관리로 가장 많이 사용되었다

- ### logback : log4j보다 10배 정도 빠르게 수행되도록 내부가 변경되었으며, 메모리 효율성도 좋아짐

- ### SLF4J : 로깅 프레임워크들의 인터페이스



## Log4j 취약점 이슈

- ### 악의적인 공격자가 서버를 제어하는 임의의 Java코드를 실행할 수 있게 하는 원격코드 실행 취약점

- ### 대응 또는 대처 방안

  - #### log4j ver 2.15.0 패치했으니 java8 버전에만 작동

  - #### 2.16버전 이상을 권장

  

## 사용 툴

- ### Log4j2

  - #### 상대적으로 최근에 등장한 로깅 프레임워크

  - #### log4j를 기반으로 하고 있어서 설정하는 방법이나 사용 방법 유사

  - #### logback의 아키텍처에서 발생하는 문제점을 수정하여 apache에서 log4j2를 권장하기도 함

  - #### logback과 동일하게 자동리로드 기능과 필터링 기능을 제공

    

## 설정 방법
- ### pom.xml에서 dependency 선언

  - #### 기존에 내장되어있던 log4j를 exclude 시키고 log4j2 dependency를 추가

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
      <exclusion>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-logging</artifactId>
      </exclusion>
    </exclusions>
</dependency>
<dependency>
	<groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.25</version>
</dependency>
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-api</artifactId>
  <version>2.19.0</version>
</dependency>
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-core</artifactId>
  <version>2.19.0</version>
</dependency>
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-slf4j-impl</artifactId>
  <version>2.19.0</version>
</dependency>
```

- ### log4j2.xml 생성

  - #### 경로는 /src/main/resources/config/log4j2.xml

    - ##### 로그 파일을 지정된 장소에 저장

    - ##### 지정된 로그 파일을 주기적으로 삭제

  - #### configration stsatus : configuration file이  로그레벨 몇으로 지정될 것인지를 알려주는 것

  - #### PatternLayout : 로그가 어떤 모양으로 출력 될 것인지를 알려줌

  - #### Loggers : 어떤 로그 레벨들을 어떻게 출력할 것인지 알려줌

    - ##### Root : 필수 지정이며, 대부분은 INFO로 지정

    - ##### AppenderRef : 해당 로그 레벨로 어느정도까지의 일을 수행할지를 정함

      - ###### ex) 현재는 Console 로그만 찍히도록 수행

      

- ### log4j2.xml 전체 소스

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Appenders>
        <Console name="STDOUT" target="SYSTEM_OUT">
            <PatternLayout disableAnsi="false" pattern="[%style{%d{yyyy-MM-dd HH:mm:ss.SSS}}] %highlight{[%-5level]}{FATAL=bg_red, ERROR=red, WARN=yellow bold, INFO=green, DEBUG=blue} %style{[%t]}{yellow} [%C] - %m%n" />
        </Console>
    </Appenders>
    <Loggers>
      	<!-- 스프링 logger 설정 -->
        <logger name="org.springframework.core">
            <level value="INFO" />
            <AppenderRef ref="STDOUT" />
        </logger>

        <logger name="org.springframework.beans">
            <level value="INFO" />
            <AppenderRef ref="STDOUT" />
        </logger>

        <logger name="org.springframework.context">
            <level value="INFO" />
            <AppenderRef ref="STDOUT" />
        </logger>

        <logger name="org.springframework.web">
            <level value="INFO" />
            <AppenderRef ref="STDOUT" />
        </logger>
      
        <!-- 기본(디폴트) loger 설정-->
        <Root level="DEBUG">
            <AppenderRef ref="STDOUT"/>
        </Root>
    </Loggers>
</Configuration>
```

- ### application.yml에 log4j2.xml 경로 추가

```yaml
# logging
logging:
    config: src/main/resources/log4j2.xml
```

- ### 결과 

  - #### 빨간색(날짜 및 시간) : %d{yyyy-MM-dd HH:mm:ss.SSS}

  - #### 파란색(스레드) : %thread

  - #### 노란색(loglevel) : %-5level

  - #### 하늘색(logger) : %logger{36}

  - #### 보라색(메세지) : %m

<img width="1327" alt="스크린샷 2022-12-01 오후 6 58 25" src="https://user-images.githubusercontent.com/51348720/205024098-5f2949cf-fd7a-4f46-9671-3d9160f3a755.png">

- ### LogLevel

  - #### TRACE < DEBUG < INFO < WARN < ERROR < FATAL

    - ##### TRACE가 가장 낮고 FATAL이 가장 높은 레벨

    - ##### 레벨 지정시 자신 포함 상위 레벨을 로깅

  - #### TRACE : 추적레벨은 DEBUG보다 좀 더 상세한 정보를 나타냄

  - #### DEBUG : 프로그램을 디버깅하기 위한 정보 지정

  - #### INFO : 상태변경과 같은 정보성 메시지를 나타냄

  - #### WARN : 처리 가능한 문제, 향후 시스템 에러의 원인이 될 수 있는 경고성 메시지를 나타냄

  - #### ERROR : 요청을 처리하는 중 문제가 발생한 경우

  - #### FATAL : 아주 심각한 에러가 발생한 상태, 시스템적으로 심각한 문제가 발생해서 어플리케이션 작동이 불가능

  

- ### 쿼리 테이블 결과 로그 추가

  - #### pom.xml에서 dependency 선언

  ```xml
  <dependency>
    	<groupId>org.bgee.log4jdbc-log4j2</groupId>
    	<artifactId>log4jdbc-log4j2-jdbc4.1</artifactId>
    	<version>1.16</version>
  </dependency>
  ```

  - #### application.yml에 log4j2.xml 경로 추가

  ```yaml
    datasource:
      url: jdbc:log4jdbc:mysql://db서버:포트/planE?characterEncoding=utf8
      username: 계정아이디
      password: 계정비밀번호
      driver-class-name: net.sf.log4jdbc.sql.jdbcapi.DriverSpy
  ```

  - #### log4jdbc.log4j2.properties 생성

  ```properties
  log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
  log4jdbc.dump.sql.maxlinelength=0
  ```

  - #### log4j2.xml에서 loger 추가

    - ##### sqltiming : SQL문과 수행된 시간을 로그로 남김

    - ##### resultsettable : SQL 결과 조회된 데이터를 table 형식으로 만들어줌

  ```xml
  <logger name="jdbc.sqltiming">
    	<level value="TRACE" />
    	<AppenderRef ref="STDOUT" />
  </logger>
  
  <logger name="jdbc.resultsettable">
    	<level value="INFO" />
    	<AppenderRef ref="STDOUT" />
  </logger>
  ```

  - #### 테이블 logger 결과

  <img width="909" alt="스크린샷 2022-12-01 오후 8 02 42" src="https://user-images.githubusercontent.com/51348720/205036717-82337598-36eb-4c6d-bbcf-5cce647dba5b.png">

  