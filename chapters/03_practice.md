# 3.5. Nginx 설치 및 실행
Nginx(Enginx-X)는 성능이 뛰어나고 리소스 사용이 적은 웹 서버로, 정적 파일 서비스뿐 아니라 리버스 프록시, 로드 밸런서 등 다양한 기능을 지원한다.
이벤트 기반 아키텍처로 동시 접속 처리에 효율적이며, Node.js, Django, 스프링 부트 등 백엔드 애플리케이션과 리버스 프론시로 자주 연동된다.

## 3.5.1 Nginx 개요 및 장단점
Nginx는 적은 자원으로도 많은 트래픽을 처리 할 수 있어, 클라우드 환경이나 가상 인스턴스에서 매우 효율적이다.

## 3.5.2. Nginx 설치
```
$ sudo apt install -y nginx # nginx 설치
$ sudo systemctl start nginx # nginx 실행
$ sudo systemctl status nginx # nginx 상태 확인
```

* 참고
  - sudo -i : 루트 권한으로 로그인
  - apt info nginx : nginx 패키지 정보 확인
> ```
> root@ip-***.***.**.***:~# sudo apt install -y nginx
> ...
> ...
> Error: Failed to fetch http://security.ubuntu.com/ubuntu/pool/main/n/nginx/nginx-common_1.28.3-2ubuntu1.2_all.deb  404  Not Found
> Error: Failed to fetch http://security.ubuntu.com/ubuntu/pool/main/n/nginx/nginx_1.28.3-2ubuntu1.2_amd64v3.deb  404  Not Found
> Error: Unable to fetch some archives, maybe run apt update or try with --fix-missing?
> ```
> -> `sudo apt update` 로 해결


```
$ sudo systemctl status nginx
...
...
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-08-29 14:34:55 UTC; 30s ago
```

* Public-2C 인스턴스에도 적용

## 3.5.3 웹 서버 접속 테스트(퍼블릭 IP)
- 웹브라우저에서 인스턴스의 Public IP 주소로 접속하여 Welcome to nginx! 타이틀 메시지 확인

# 3.6. 내가 만든 홈페이지
## 3.6.1 Github에서 다운로드
https://github.com/***/ec2_homepage.git 접속

## 3.6.2 나만의 홈페이지 파일(index.html) 교체
### Public-2A 영역
```
# exit # 관리자 로그인 시 사용자 모드로 변경
$ git clone https://github.com/***/ec2_homepage.git
$ ls # 목록 확인
$ cd ec2_homepage/Public-2A # Public-2C 인스턴스에서는 Public-2C로 입력
$ ls
$ sudo cp -r ./* /var/www/html/
$ sudo systemctl restart nginx
```

# 3.7. 스프링 부트 연동
* 구현 프로세스: JDK 설치 -> Maven 설치 -> 소스 컴파일 -> 소스 패키징 -> 서버 테스트

## 3.7.1. JDK 및 Maven 설치
* JDK 설치 시 스프링 부트에서 코딩했을 시점의 JDK 버전과 동일한 버전을 설치해야 함.
```
$ sudo systemctl stop nginx # 이전 구동 웹 서버 중지
$ cd
$ sudo apt update # 관련 패키지 업데이트
$ sudo apt install openjdk-17-jdk -y
$ sudo apt install -y maven
$ java -version
openjdk version "17.0.20" 2026-07-21
OpenJDK Runtime Environment (build 17.0.20+8-1-26.04-Ubuntu)
OpenJDK 64-Bit Server VM (build 17.0.20+8-1-26.04-Ubuntu, mixed mode, sharing)
$ mvn -v
Apache Maven 3.9.12
Maven home: /usr/share/maven
Java version: 17.0.20, vendor: Ubuntu, runtime: /usr/lib/jvm/java-17-openjdk-amd64
Default locale: en, platform encoding: UTF-8
OS name: "linux", version: "7.0.0-1006-aws", arch: "amd64", family: "unix"
```

## 3.7.2 스프링 부트 프로젝트
### github에서 가져오기
```
$ git clone https://github.com/***/shopping_01_no_database.git
```

### 패키징
다운로드한 **소스를 패키징**하기 위해 해당 디렉터리로 이동 후, **mvn package 명령어**를 실행한다. 
이 명령어는 프로젝트를 배포 가능한 .jar 또는 .war 파일로 패키징하며, 테스트 통과 시
BUILD SUCCESS가 출력되고, 마지막에 shopping-0.0.1-SNAPSHOT.jar 파일이 생성된다. 이 파일이 스프링 부트 실행용 파일이다.

```
$ cd shopping_01_no_database/
$ mvn clean package -DskipTests # 전체 패키징
```

### 스프링 부트 서버 테스트
패키징 완료 후, 작업할 디렉터리로 이동한다.
1024번 이하 포트는 관리자 권한이 필요하므로, 포트 포워딩을 사용하는 것이 권장된다. **저자의 스프링 부트 애플리케이션은 기본 포트 9000번을 사용**하며, 
**java -jar** 명령어로 실행 시 내장 Tomcat 서버와 함께 **웹 애플리케이션이 시작**된다.

> 포트 포워딩(Port Forwarding)
>> 포트 포워딩은 외부에서 들어오는 네트워크 요청을 내부의 특정 컴퓨터(서버)의 포트로 전달해 주는 기술이다.

```
$ cd target
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 9000
$ java -jar shopping-0.0.1-SNAPSHOT.jar
```

* http://{인스턴스 IP}/products 입력 시 페이지가 로드됨

### 스프링 부트 서버 종료
스프링 부트 서버 종료를 위해서 해당 프로세스 ID(PID)를 우선 확인해야 한다.
명령어를 사용하여 시스템에서 9000번 포트를 사용 중인 프로세스(프로그램)을 우선 확인한다. 해당 프로세스의 ID 번호를 확인 후 종료 명령어를 사용한다.

```
$ sudo lsof -i:9000 # lsof = List Open Files
COMMAND  PID   USER  FD ...
java    3978 ...

$ sudo kill 3978
```

### Public-2C에서 html 문서 수정하기
Public-2A와 Public-2C 페이지를 구분하기 위해 html 문서 수정 필요.
다음 명령어를 Public-2C에서만 실행한다.
```
$ sudo vi /home/ubuntu/shopping_01_no_database/src/main/resources/templates/productList.html
:wq
```
