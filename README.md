# aws-v4

### 엘라스틱 빈스톡 종료
- 종료시, ec2도 함께 종료
- 빈스톡 어플리케이션과 ec2 보안 규칙은 남아 있으므로 따로 삭제

### 엘라스틱 빈스톡 생성
- 이름은 aws-v4-beanstalk, 키페어는 기존에 사용하던 키페어 (없으면 생성)
- 5단계의 환경 속성에서 RDS와 관련된 환경 속성을 추가
```shell
RDS_HOSTNAME = '해당 데이터베이스 ip 주소 (mysql 주소)'
RDS_DB_NAME = '임의의 db 이름'
RDS_PORT = '해당 데이터베이스 포트번호 (3306)'
RDS_USERNAME = '임의의 로그인 이름'
RDS_PASSWORD = '임의의 비밀번호'
```

### vpc
- virtual private cloud (가상 개인용 클라우드), 고유 id가 존재
- vpc 안에 ec2 서버 + rds (db)가 존재, 서버는 퍼블릭, 프라이빗 ip를 부여받음
- 따로 보안그룹도 존재, 고유 이름과 id를 부여받음
- 보안 규칙으론 22 포트는 전체 개방, 3306 포트는 로컬 컴퓨터 ip + ec2 프라이빗 ip 설정으로 두 ip만 접근 가능
- 3306 포트 설정으로 같은 보안 그룹으로 묶어 ec2와 rds간 접근 가능하게 하면서 rds의 허용되지 않은 외부 접근은 차단
- 기본적으론 vpc는 둘로 나눠 외부 엑세스 허용 유무를 결정, 허용한 쪽에 ec2를 두고 금지한 쪽에 rds를 두고 운용
- 클라이언트는 허용된 vpc, 즉 ec2 서버로만 접근 가능하고 rds는 접근 불가
- ec2 서버는 특정 포트 (예: mysql - 3306)로 접근 가능

### rds
- 데이터베이스 생성 클릭 후, mysql 선택
- 마스터 이름과 암호는 엘라스틱 빈스틱 환경 변수 설정과 동일하게 설정
- 로컬 컴퓨터로 db에 접근하기 위해 퍼블릭 엑세스는 허용
- vpc와 보안 그룹은 사용할 걸로 선택 후 데이터베이스 생성
- 보안 설정을 통해 다른 접속은 차단하기 위해 기존 보안 그룹에 인바운드 규칙 수정
- 3306 포트로 로컬 컴퓨터 ip와 보안 그룹을 추가하여 두 경로로만 접근 허용
- 이후 보안 그룹에 다른 ec2를 추가하기만 하면 접속 가능

### db 생성
- mysql workbench를 통해 rds에 접속
- connection 생성시, 호스트 이름은 rds 엔드포인트 + 포트는 3306 + 유저 이름과 비밀번호는 임의로 설정
- rds에서 테스트할 db 생성
```shell
# db 생성
create database testdb;

# db 선택
use testdb;

# 테이블 생성
create table Book (
	id bigint auto_increment primary key,
    title varchar(255),
    content varchar(255),
    author varchar(255)
);

# 테이블 확인  
select * from Book;
```

- 한글 깨짐 문제를 해결하기 위한 설정
```shell
# character_set_database, character_set_server, collation_database 등 c로 시작하는 설정 확인
show variables like 'c%';

# 한글 깨짐 방지를 위해 utf8mb4로 설정
alter database testdb character set = 'utf8mb4' collate = 'utf8mb4_general_ci';
```

### 프로젝트 배포 및 테스트
- yml 설정시 엘라스틱 빈스톡 환경 속성을 사용하기 위해 하드코딩이 아닌 환경변수 설정
```shell
  datasource:
    url: jdbc:mysql://${rds.hostname}:${rds.port}/${rds.db.name}?serverTimezone=Asia/Seoul
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: ${rds.username}
    password: ${rds.password}
```

- 테스트할 프로젝트를 jar 파일로 생성
```shell
./gradlew build
```

- 엘라스틱 빈스톡 환경에 업로드 및 배포
- 테스트 진행시, insomnia 같은 테스트툴을 사용해 테스트