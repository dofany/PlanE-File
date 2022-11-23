---

Title: Was Server Build - Wildfly-20.0.1.final
Publish Date: 2022.11.20
---

## PlanE WAS 환경 구축 및 설정

------

### WAS 란?

-----

동적 컨텐츠를 제공하기 위해 만들어진 Web Application Server로써 웹 프로그램을 실행할 수 있는 환경을 제공한다.

Tomcat이 대부분 Servlet Container와 Web Server인 반면 Jboss는 embedded tomcat으로 Servlet Container와 Web server를 포함하는 JEE 스택으로 확장한다.



### 목차

--------

01 WAS 환경세팅

* OpenJDK 11 설치
* JAVA PATH 설정

02 wildfly 사용자 생성

* wildfly group 생성
* wildfly user 생성

03 Wildfly 설치

* Wildfly 아카이브 다운로드
* Wildfly 압축 해제
* /opt/wildfly 심볼릭 링크 생성
* /opt/wildfly 소유권 변경
* WILDFLY HOME 설정

04 Systemd 구성

* Wildfly 구성파일 디렉토리 생성
* Wildfly 구성파일 복사
* daemon 관리 및 service 실행

​	05 방화벽 설정

​	06 Wildfly 접속 확인

-----



### 01 WAS 환경세팅

----

####  - OpenJDK 11 설치

​	Java SE 8부터 13까지 호환하는 WildFly-21.0.1.Final 을 설치하기 위해

​	먼저, CentOS 7의 기본 Java 개발 및 런타임인 Java 플랫폼의 오픈 소스 구현인 OpenJDK를 설치할 것이다.

​	wildfly 호환 확인 : https://www.wildfly.org/news/2020/06/08/WildFly20-Final-Released/

![was1_1](/uploads/10851250b3776dc3e36cc9d716a30773/was1_1.png)

- yum install -y java-11-openjdk-devel.x86_64 

![was1_2](/uploads/7d22d1ec0b5eaff692dea5fc53b0a95a/was1_2.png)

* "java -version" 명령어로 openJDK 11 버전이 잘 설치되었는지 확인할 수 있다.



#### - JAVA PATH 설정

![was1_4](/uploads/460824c51a7c840c6c32efd77f779b23/was1_4.png)

* "readlink -f /bin/javac" 명령어 실행 후 나오는 경로에서 bin/javac 전까지만 복사한다.

![was1_5](/uploads/64211222c8b816021be65e2ffd42e4b0/was1_5.png)

* "vi /etc/profile" 맨 하단에 export 작성하고 저장한다.
  export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.17.0.8-2.el7_9.x86_64/ 

* "source /etc/profile" 적용 후 $JAVA_HOME 확인 가능



### 02 wildfly 사용자 생성

----

Wildfly를 root사용자로 실행하는 것은 보안상 취약해지므로 모범 사례로 간주되지 않는다.

따라서 새로운 wildfly용 그룹 및 사용자를 만들어줘야한다. (home directory: /opt/wildfly)

#### - group 생성

![was2_1](/uploads/c8a72b832c77088d176289c1c944f0d3/was2_1.png)

- groupadd -r wildfly

- [옵션]

  • -r : 시스템에 사용되는 gid를 부여한다. (500번 이하의 가장 빠른 gid를 생성)

#### - user 생성

![was2_2](/uploads/4f52840fee969f1147950564a15896cb/was2_2.png)

* useradd -r -g wildfly -d /opt/wildfly -s /sbin/nologin wildfly

* [옵션]

  • -r : 시스템 계정 지정

  • -g gid: gid 그룹을 지정

  • -d 홈디렉토리 : 홈디렉토리의 경로를 지정한다.

  • -s shell: 쉘을 지정한다.

*  [추가] nologin이란?

  Shell 로그인이 안되는 계정을 의미한다.

  보안상의 이유로 cmd, putty, hi-tam 등 서버 접근 제어 프로그램을 이용해서 ssh 로그인을 할 수 없다.

  root 계정 접근을 막기 위해 FTP 접근만 허용하며 주로 사용하지 않는 디폴트 데몬 계정에 적용한다.



### 03 Wildfly 설치

----

#### - Wildfly 아카이브 다운로드

​	wildfly 버전을 선언한 뒤, wget 명령을 사용하여 /tmp 디렉토리에 WildFly 아카이브를 다운로드한다.

![1](/Users/junheekim/Desktop/was/3/1.png)

- WILDFLY VERSION=20.0.1.Final

- wget https://download.jboss.org/wildfly/$WILDFLY_VERSION/wildfly-$WILDFLY_VERSION.tar.gz -P /tmp

- [옵션]

  • -P 디렉토리: 다운로드한 파일을 디렉토리에 저장함

- [추가]  /tmp 디렉토리란?

  임시파일 디렉토리

#### - Wildfly 압축 해제

​	다운로드가 완료되면 tar.gz 파일의 압축을 풀고 /opt 디렉토리로 이동한다.

![was3_1](/uploads/2265e190c69968d9406e9e0346f822b7/was3_1.png)

- tar xf /tmp/wildfly-$WILDFLY_VERSION.tar.gz -C /opt/

- [옵션]

  • -f : 대상 tar 아카이브 지정(필수)

  • -x : tar 아카이브에서 파일 추출(unarchive)

  • -C 디렉토리 : 대상 디렉토리 지정

#### - /opt/wildfly 심볼릭 링크 생성

![was3_3](/uploads/b623c6c4c498b479ab187fc4287d0baf/was3_3.png)

- ln -s /opt/wildfly-$WILDFLY_VERSION /opt/wildfly

- [옵션]

  • -s A(참조파일) B(심볼릭링크) :  B -> A 로 링크 된다.

#### - /opt/wildfly 소유권 변경

​	WildFly는 <u>wildfly directory에 액세스 권한이 있는</u> wildfly 사용자 아래에서 실행됩니다.

​	디렉터리 소유권을 사용자 및 그룹 wildfly로 변경합니다.

![was3_5](/uploads/0e0046859ada5cdaec7f5b8724fc06f2/was3_5.png)

- chown -RH wildfly: /opt/wildfly

- [옵션]

  • -R (—recursive) : 지정한 파일 하위까지 권한 변경

  • -H : (-R 옵션과 함께 사용) 심볼릭링크의 참조파일(A)만 변경된다.

#### - WILDFLY HOME 설정

![was3_4](/uploads/9c63e1863a8d34d825c476a09123754d/was3_4.png)

* "vi /etc/profile" 하단에 "export WILDFLY_HOME=/opt/wildfly" 을 작성하고 저장한다.

![was3_6](/uploads/f96d8fe851374077e94f1e2a873850dc/was3_6.png)

* "source /etc/profile" 적용 후 $WILDFLY_HOME 확인 가능

  

### 04 Systemd 구성

-----

#### - Wildfly 구성파일 디렉토리 생성

​	WildFly 패키지에는 WildFly를 서비스로 **실행**하는 데 필요한 파일이 포함되어 있기때문에

​	먼저, 해당 구성 파일을 저장할 디렉토리를 생성한다.

![was4_1](/uploads/f17352a69b39a5abc3eabec06c509184/was4_1.png)

- mkdir -p /etc/wildfly

- [옵션]

  • -p : 여러 덱스의 하위 디렉토리를 생성시에 사용한다.

- [추가]  /etc 이란?

  환경설정 파일

#### - Wildfly 구성파일 복사

##### 		(1) wildfly.conf

![was4_2](/uploads/7b1020d2d4a09b7ac1c5ede5e27f9955/was4_2.png)

- cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.conf /etc/wildfly/

##### 		(2) launch.sh

![was4_3](/uploads/3c5207ec844fb84b33e2cd55422c3fff/was4_3.png)

- cp /opt/wildfly/docs/contrib/scripts/systemd/launch.sh /opt/wildfly/bin/
- /opt/wildfly/**bin** 디렉토리의 스크립트에는 실행 파일 플래그가 있기 때문에 실행 권한을 추가해준다.
  sh -c 'chmod +x /opt/wildfly/bin/*.sh'

##### 		(3) wildfly.service

![was4_4](/uploads/b58be3324e0abf1e029f49e57fa21dd4/was4_4.png)

- cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.service /etc/systemd/system/

#### - daemon 관리 및 service 실행

![was4_5](/uploads/f25ae28170b849c4a59cffcb5e68deba/was4_5.png)

- 새 service 파일 생성 알림
  systemctl daemon-reload
- wildfly 서비스 실행
  systemctl start wildfly
- 부팅시 자동 실행 설정
  systemctl enable wildfly

![was4_6](/uploads/0be0f223b6f565876cdec96a4802f174/was4_6.png)

- wildfly 실행 상태 확인
  systemctl status wildfly





### 05 방화벽 설정

------

서버는 방화벽에 의해 보호되고 있습니다.

따라서 로컬 네트워크 외부에서 WildFly 인스턴스에 액세스하려면 포트를 열어야 합니다.

추가적으로, 오라클 클라우드에서는 포트개방을(?) 해줘야합니다.

![was5_1](/uploads/7b8029939acea02ccd23619f8002f8be/was5_1.png)

- 8080 포트 개방
  firewall-cmd --zone=public --permanent --add-port=8080/tcp

![was5_2](/uploads/f7799d5bc52d730ce88f30e6ab2f341b/was5_2.png)

- 방화벽 리로드(적용)
  firewall-cmd --reload



### 06 Wildfly 접속 확인

------

![was6_1](/uploads/0d17f93120c079464a0d7e14caad318f/was6_1.png)

*  http://[IP주소]:8080
