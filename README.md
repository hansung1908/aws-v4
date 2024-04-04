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