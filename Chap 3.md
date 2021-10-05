## 3.1. JPA 소개

### JPA란?

- JPA 이전에는 데이터베이스에 접근할 때 SQL를 사용했음
- 객체지향이라는 패러다임 불일치 + SQL 작업을 피하기 위해
    
    → `JPA(자바 표준 ORM)` 생성
    
- 객체지향적으로 프로그래밍을 하면 JPA가 관계형 데이터베이스에 맞게 **SQL을 대신 작업**
<br />

### Spring Data JPA

- 인터페이스인 JPA를 사용하기 위해서는 구현체가 필요
    - e.g., Hibernate, Eclipse Link 등
- Spring에서 JPA를 사용할 때는 구현체를 직접 다루지 x
- 구현체를 추상화시킨 **Spring Data JPA** 모듈을 이용해 JPA 다룸
    - JPA ← Hibernate ←Spring Data JPA
- Hibernate 대신 Spring Data JPA를 사용하는 이유
    1. 구현체 교체의 용이성
        - Hibernate 외의 다른 구현체로 쉽게 교체하기 위해
    2. 저장소 교체의 용이성
        - 관계형 데이터베이스 외에 다른 저장소로 쉽게 교체하기 위해
<br />

### 실무에서 JPA

- CRUD 쿼리 직접 작성 필요x
- 객체지향 프로그래밍 쉽게 가능
- 네이티브 쿼리만큼의 성능
<br />

### 요구사항 분석

- 게시판 기능
    - 게시글 조회
    - 게시글 등록
    - 게시글 수정
    - 게시글 삭제
- 회원 기능
    - 구글/네이버 로그인
    - 로그인한 사용자 글 작성 권한
    - 본인 작성 글에 대한 권한 관리


---


## 3.2. 프로젝트에 Spring Data Jpa 적용

### Spring Data Jpa 적용

1. **[build.gradle]** 의존성 등록
    
    ```java
    dependencies {
        ...
        implementation('org.springframework.boot:spring-boot-starter-data-jpa')
        implementation('com.h2database:h2')
    }
    ```
    

2. **[com.jojoldu.book.springboot/domain]** 패키지 생성
    - 도메인을 담을 패키지

3. **[domain/posts/Posts]** 패키지, 클래스 생성
    - code
        
        ```java
        package com.jojoldu.book.springboot.domain.posts;
        
        import lombok.Builder;
        import lombok.Getter;
        import lombok.NoArgsConstructor;
        
        import javax.persistence.Column;
        import javax.persistence.Entity;
        import javax.persistence.GeneratedValue;
        import javax.persistence.GenerationType;
        import javax.persistence.Id;
        
        @Getter
        @NoArgsConstructor
        @Entity
        public class Posts {
            @Id 
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private Long id;
        
            @Column(length=500, nullable = false)
            private String title;
        
            @Column(columnDefinition = "TEXT", nullable = false)
            private String content;
        
            private String author;
        
            @Builder
            public Posts(String title, String content, String author){
                this.title = title;
                this.content = content;
                this.author = author;
            }
        }
        ```
        
    - Posts 클래스
        - DB 테이블과 매칭될 클래스 = **Entity 클래스**
        - 쿼리 작성 대신 Entity 클래스를 수정하여 DB 작업
    - 어노테이션 정리
        
        `@Entity` 테이블과 링크될 클래스임을 나타냄
        
        `@Id` PK 필드
        
        `@Column` 테이블 컬럼, 변경할 옵션이 있으면 사용
        
    - 롬복 어노테이션은 코드 변경량을 최소화 → 적극 사용
    - PK는 **Long타입의 Auto_increment** 권장
<br />    


### Entity 클래스

**Entity 클래스는 Setter 메소드 만들지 않기**

→ 필드 값 변경 필요 시, 목적을 나타내는 **메소드 추가**

- e.g., 주문 취소
    
    ```java
    public class Order{
    	public void cancelOrder(){
    		this.status = false;
    	}
    }
    public void 주문서비스_취소이벤트(){
    	order.cancelOrder();
    }
    ```
    
- DB에 데이터 업데이트
    - 변경: 이벤트에 맞는 **public 메소드** 호출
    - 삽입: **생성자 혹은 @Builder**를 통해 채운 후 삽입
        - 빌더 패턴 권장
<br />


### 빌더 클래스

- 생성자와 빌더 클래스 모두 값을 채우는 역할
- **빌더 패턴 권장**
- 생성자는 채워야 할 필드 명확히 지정x
    - a와 b를 바꿔도 실행 전까지 문제 알 수 없음
    
    ```java
    public Example (String a, String b){
    	this.a = a;
    	this.b = b;
    }
    ```
    
- 빌더는 어느 필드에 어떤 값을 채울지 명확
    
    ```java
    Example.builer()
    	.a(a)
    	.b(b)
    	.build();
    ```
<br />    


### Database 접근: Jpa Repository 생성

- **[domain/postsPostsRepository]** 인터페이스 생성
    - JPA-Repository = ibatis, MyBatis-Dao
    - DB Layer 접근자
    - `JpaRepository<Entity class, PK type>` 상속
        
        → 기본 CRUD 자동 생성
        
    
    ```java
    import org.springframework.data.jpa.repository.JpaRepository;
    public interface PostsRepository extends JpaRepository<Posts, Long>{ }
    ```
    
- **Entity 클래스와 Entity Repository는 같은 위치에**
- 도메인별로 프로젝트를 분리한다면, 둘은 함께 움직여야 함
    
    → **도메인 패키지**에서 함께 관리

