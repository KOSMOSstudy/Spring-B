

## 5.1. 스프링 시큐리티와 스프링 시큐리티 Oauth2 클라이언트

- id/password방식보다 소셜 로그인 기능을 선호
    - id/pw를 직접 구현하려면 너무 많은 기능을 구현해야 함
        - 보안/ 회원가입 시 이메일 혹은 전화번호 인증/ 비밀번호 찾기/ 비밀번호 변경/ 회원정보 변경 등
- OAuth 로그인 구현 시 구글, 페이스북, 네이버 등에 맡기면 됨

### 스프링 시큐리티

- 막강한 인증과 인가 기능을 가진 프레임워크
- 스프링 기반의 애플리케이션에서 보안 표준
<br>

## 5.2. 구글 서비스 등록

1. 구글 서비스에 신규 서비스 생성
2. 동의 화면 구성
3. 사용자 인증 정보 생성 후 아이디/비밀번호 획득
4. application-oauth 등록

<br>

### 1. 구글 서비스에 신규 서비스 생성

- 발급된 인증 정보(clientId, clientSecrete)를 통해 로그인 + 소셜 서비스 기능 사용 가능
1. 구글 클라우드 플랫폼 주소 [https://console.cloud.google.com](https://console.cloud.google.com) 으로 이동
2. [프로젝트 선택] 탭 클릭
3. [새 프로젝트] 탭 클릭
4. 등록될 서비스 이름 입력

<br>

### 2. 동의 화면 구성

[메뉴] > [API 및 서비스] >[동의 화면 구성]

- OAuth 동의화면
    - 앱 이름: freelec-springboot2-webservice
    - 사용자 지원 이메일:
- 범위
    - 이메일 주소 확인/ 개인정보 보기 클릭
<br>

### 3. 사용자 인증 정보 생성

[사용자 인증 정보 만들기] > [OAuth 클라이언트 ID]

- 애플리케이션 유형: 웹 애플리케이션
- 이름: freelec-springboot2-webservice
- 개발자 이메일:
- 승인된 리디렉션 url: http://localhost:8080/login/oauth2/code/google

 

⇒ 클라이언트 아이디/비밀번호 획득

<br>

### 4. application-oauth 등록

- ***src/man/resources*** 에 ***application-oauth.properties*** 파일 생성

```java
spring.security.oauth2.client.registration.google.client-id=구글클라이언트ID
spring.security.oauth2.client.registration.google.client-secret=구글클라이언트시크릿
spring.security.oauth2.client.registration.google.scope=profile,email
```
<br>

### properties 파일

- 스프링부트에서 application-xxx.properties로 파일을 생성하면 xxx라는 이름의 profile이 생성되어 이를 통해 관리할 수 있음
- **profile=xxx**로 호출 → 해당 **properties 설정을 가져옴**
- application.propreties에서 application-aouth.properties를 포함하도록 구성
    - ***application.properties***
    
    ```java
    spring.profiles.include=oauth
    ```
<br>  

### .gitignore

- 보안을 위해 클라이언트 아이디/비밀번호가 포함된 파일은 깃허브에 올리지 않아야 함
- application-oauth.properties 파일을 .gitignore에 추가
