# 3. RDS 접속 테스트
## 3.2 EC2에서 CLI를 통한 접속 테스트

### MySQL 클라이언트 설치/접속
```
$ sudo apt update # 리눅스와 관련된 패키지 업데이트
$ sudo apt install mysql-client -y # mysql-client 설치
```

```
$ mysql -h dbmysql80.{엔드포인트}.ap-northeast-2.rds.amazonaws.com -P 3306 -u myroot -p
```
|  옵션  |  의미   | 설명 |
|--------|---------|-------------------------------------------|
| -h   | Host    |  접속할 MySQL 서버의 호스트 이름 또는 IP 주소(여기서는 AWS RD의 Endpoint) |
| -P  | Port  |  MySQL이 사용하는 포트 번호. 기본값: 3306 반드시 대문자로 입력해야 함. |
| -u |  User  |  접속할 사용자 이름 |
| -p | Password/Prompt | 비밀번호 입력 프롬프트를 띄움. |

### 기본 CRUD 작업
```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| shopping           |
| sys                |
+--------------------+
5 rows in set (0.02 sec)

mysql> use shopping;
mysql> show tables;
mysql> create table products( id BIGINT primary key, name varchar(100), price int, description text, image_url varchar(255));
mysql> insert into products (id, name, price, description, image_url) values
    -> (1, '카푸치노', 4500, '맛있는 카푸치노', '/images/cappuccino01.png'),
    -> (2, '크로아상', 5000, '빵은 역시 크로아상^^;', '/images/croissant_01.png'),
    -> (3, '우유', 3000, '따뜻한 우유 좋아욯ㅎ', '/images/milk01.jpg');
mysql> commit;
mysql> select * from products;
+----+--------------+-------+-------------------------------+--------------------------+
| id | name         | price | description                   | image_url                |
+----+--------------+-------+-------------------------------+--------------------------+
|  1 | 카푸치노     |  4500 | 맛있는 카푸치노               | /images/cappuccino01.png |
|  2 | 크로아상     |  5000 | 빵은 역시 크로아상^^;         | /images/croissant_01.png |
|  3 | 우유         |  3000 | 따뜻한 우유 좋아욯ㅎ          | /images/milk01.jpg       |
+----+--------------+-------+-------------------------------+--------------------------+
```

## 3.3 스프링 부트 서버와 RDS 연동
### GitHub에서 파일 다운로드
* https://github.com/***/shopping_02_yes_database

Public-2A EC2 인스턴스에서 작업 진행
```
$ cd
$ git clone https://github.com/***/shopping_02_yes_database.git
$ cd shopping_02_yes_database/src/main/resources/

```

### application.properties 파일
* 스프링 부트에서 설정을 정의하는 기본 속성 파일로, 데이터베이스 연결, 서버 포트, 로깅 등 다양한 값을 key=value 형식으로 관리한다.
* 이번 예시에서 **RDS 접속 URL과 관리자 계정 정보**를 이 파일에 적용한다.

```
spring.application.name=shopping
server.port=9000

spring.thymeleaf.cache=false
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html

spring.datasource.url=jdbc:mysql://{엔드포인트}/shopping?useSSL=false&serverTimezone=Asia/Seoul&allowPublicKeyRetrieval=true
spring.datasource.username={id}
spring.datasource.password={password}

spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

### 패키징 하기
Maven 설정 파일 pom.xml이 들어있는 디렉토리로 이동하여 패키징 작업을 수행한다. 다시 결과물 파일이 들어있는 대상 디렉터리로 이동하여, 해당 애플리케이션을 구동한다.
```
$ cd
$ cd shopping_02_yes_database/
$ mvn clean package -DskipTests
...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  11.086 s
[INFO] Finished at: 2026-08-31T14:35:17Z
[INFO] ------------------------------------------------------------------------
...
$ cd target/
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 9000
$ java -jar shopping-0.0.1-SNAPSHOT.jar
```

## 3.4 스프링 부트 서버 테스트
### 웹 브라우저에서 상품명 확인
실제 AWS RDS 인스턴스에 접속하여 데이터를 직접 변경해본다.

```
mysql> update shopping.products set name = '쭈쭈바' where id = 1;
mysql> commit;
```

## 3.5 Public-2C 인스턴스 테스트

```
$ git clone https://github.com/***/shopping_02_yes_database.git
$ cd shopping_02_yes_database/src/main/resources/
$ vi application.properties # Public-2A와 동일하게 수정
$ cd
$ cd shopping_02_yes_database/
$ mvn clean package -DskipTests
$ cd target/
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 9000
$ java -jar shopping-0.0.1-SNAPSHOT.jar
```

Public-2A 영역에서 다시 상품 이름을 원래대로 변경하고 잘 반영이 되는지 확인한다.

```
update products set name = '카푸치노' where id = 1;
commit;
```

