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
    - 구현체 교체의 용이성
        - Hibernate 외의 다른 구현체로 쉽게 교체하기 위해
    - 저장소 교체의 용이성
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
<br />

---
## 3.2. 프로젝트에 Spring Data Jpa 적용

### Spring Data Jpa 적용

1.  **[build.gradle]** 의존성 등록

    ```java
    dependencies {
        ...
        implementation('org.springframework.boot:spring-boot-starter-data-jpa')
        implementation('com.h2database:h2')
    }
    ```

2.  **[com.jojoldu.book.springboot/domain]** 패키지 생성

    - 도메인을 담을 패키지1. **[domain/posts/Posts]** 패키지, 클래스 생성
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
    - 삽입: **생성자** 혹은 **@Builder**를 통해 채운 후 삽입
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
<br />

---
## 3.3. Spring Data JPA 테스트 코드 작성하기

</br>

- test 디렉토리에 domain.posets 패키지 생성후, PostRepositoryTest 클래스 생성
- PostRepositoryTest 에서는 save, findAll기능을 테스트

```java
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PostsRepositoryTest {

    @Autowired
    PostsRepository postsRepository;

    @After
    public void cleanup() {
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기() {

        String title = "테스트 게시글";
        String content = "테스트 본문";

        postsRepository.save(Posts.builder()
                .title(title)
                .content(content)
                .author("jojoldu@gmail.com")
                .build());


        List<Posts> postsList = postsRepository.findAll();

        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }


}

```

- 어노테이션 정리

  `@After`

  Junit에서 단위 테스트가 끝날때마다 수행되는 메소드를 지장, 여기선 테스트를 위해 생성된 데이터를 비워주어 다른데이터에 영향을 주지 않기 위해 데이터를 지워주었음

- 코드 설명

  `postRepository.save` id값이 있으면 update, 없으면 insert 쿼리가 실행됨

  `postRepository.findAll` 테이블 컬럼, 변경할 옵션이 있으면 사용

별다른 설정 없이 @SpringBootTest를 사용할 경우 H2 데이터베이스를 자동으로 실행해줌

---

- 실행된 쿼리 로그로 확인하기

src/main/resources 디렉토리에 application.properties 파일을 생성 </br>

> spring.jpa.show_sql=true

를 추가하면 쿼리 로그 확인할 수 있음

 </br>
-  출력되는 쿼리로그를 MySQL 버전으로 변경하기

application.properties 파일에

> spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect

추가하기

## 3.4. 등록/수정/조회 API 만들기 

### API를 만들기 위해선 총 3개의 클래스가 필요하다.

  1. Request 데이터 받을 DTO 
  2. API 요청을 받을 Controller 
  3. 트랜잭션,도메인 기능 간의 순서를 보장하는 Service 
  ( Service는 비지니스 로직을 처리하지 않는다. 트랜잭션,도메인 간 순서 보장의 역할만 한다.)
  
  spring 웹 계층 
  
  Web Layer
 : 흔히 사용하는 컨트롤러와 jsp등의 뷰템플릿 영역
   필터,인터셉터,컨트롤러 어드바이스 등 외부 요청과 응답에 대한 전반적인 영역을 이야기함 <br>
   <br>
 Service Layer 
 : @Service에 사용되는 서비스 영역 
   일반적으로 Controller와 Dao의 중간 영역에서 사용된다.
   @Transactional이 사용되어야 하는 영역 <br>
 Repository Layer 
 : Database와 같이 데이터 저장소에 접근하는 영역 
   기존의 Dao 영역으로 이해하면 편하다.<br>
   <br>
 Dtos 
 : Dto는 계층 간에 데이터 교환을 위한 객체를 이야기한다.
   Dtos는 이들의 영역을 얘기함 <br>
   <br>
 Domain Model 
 : 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화 시킨 것을 도메인 모델이라 한다.
   @Entity도 도메인 모델이다.
   무조건 데이터베이스의 테이블과 연관이 있어야 하는 것은 아니다.<br>
   <br>
   Web,Service,Reposiroty,Dto,Domain 이 5가지 레이어에서 비지니스 처리를 담당해야 하는 곳은 Domain이다.
   
   기존에 서비스로 비지니스 처리하던 방식을 트랜잭션 스크립트라고 한다.<br>
   <br>
   트랜잭션 스크립트 
   : 모든 로직이 서비스 클래스 내부에서 처리됨 
     서비스 계층이 무의미하며, 객체란 단순히 데이터 덩어리 역할만 한다.
     
   하지만 도메인에서 비지니스 처리를 할 경우 
   서비스 메소드는 트랜잭션과 도메인 간의 순서만 보장해 준다.
   
### 등록 코드
PostsApiController
```
@RequiredArgsConstructor
@RestController
public class PostsApiController {
    private final PostsService postsService;

    @PostMapping("api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto){
        return postsService.save(requestDto);
    }
}

PostsService
```
@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto){
        return postsRepository.save(requestDto.toEntity()).getId();
    }
}

PostsSaveRequestDto
```
@Getter
@NoArgsConstructor
public class PostsSaveRequestDto {
    private String title;
    private String content;
    private String author;

    @Builder
    public PostsSaveRequestDto(String title,String content,String author){
        this.title=title;
        this.content=content;
        this.author=author;
    }
    public Posts toEntity() {
        return Posts.builder()
                .title(title)
                .content(content)
                .author(author)
                .build();
    }
}
```


### 등록 테스트 코드

PostsApiControllerTest
```
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment =SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostsRepository postsRepository;

    @After
    public void tearDown() throws Exception{
        postsRepository.deleteAll();
    }

    @Test
    public void Posts_등록된다() throws Exception{
        //given
        String title ="title";
        String content ="content";
        PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                .title(title)
                .content(content)
                .author("author")
                .build();

        String url="http://localhost:"+port+"/api/v1/posts";

        //when
        ResponseEntity<Long> responseEntity=restTemplate.
                postForEntity(url,requestDto,Long.class);

        //then
        assertThat(responseEntity.getStatusCode()).
                isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).
                isGreaterThan(0L);
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }
}

```

### 수정/조회 코드 

PostsApiController
```
@RequiredArgsConstructor
@RestController
public class PostsApiController {
    private final PostsService postsService;

    ...

    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto){
        return postsService.update(id,requestDto);
    }

    @GetMapping("/api/v1/posts/{id}")
    public PostsResponseDto findByID (@PathVariable Long id)
    {
        return postsService.findById(id);
    }
}
```

PostsResponseDto

@Getter
public class PostsResponseDto {
    private Long id;
    private String title;
    private String content;
    private String author;

    public PostsResponseDto(Posts entity){
        this.id=entity.getId();
        this.title=entity.getTitle();
        this.content=entity.getContent();
        this.author=entity.getAuthor();
    }
}

PostsUpdateRequestDto

```
@Getter
@NoArgsConstructor
public class PostsUpdateRequestDto {
    private String title;
    private String content;

    @Builder
    public PostsUpdateRequestDto(String title,String content){
        this.title=title;
        this.content=content;
    }
}
```

Posts
```
@Getter
@NoArgsConstructor
@Entity
public class Posts {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500,nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT",nullable = false)
    private String content;

    private String author;

    @Builder
    public Posts(String title,String content,String author){
        this.title=title;
        this.content=content;
        this.author=author;
    }

    public void update(String title,String content){
        this.title=title;
        this.content=content;
    }


}
```

PostsService
```
@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    ...
    
    @Transactional
    public Long update(Long id , PostsUpdateRequestDto requestDto){
        Posts posts=postsRepository.findById(id)
                .orElseThrow(()->new
                        IllegalArgumentException("해당 게시글이 없습니다.id="+ id));

        posts.update(requestDto.getTitle(),requestDto.getContent());

        return id;
    }

    public PostsResponseDto findById(Long id){
        Posts entity=postsRepository.findById(id)
                .orElseThrow(()->new
                        IllegalArgumentException("해당 게시글이 없습니다.id="+ id));

        return new PostsResponseDto(entity);
    }
}

```

### 수정/조회 테스트 코드 

```
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment =SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    ...

    @Test
    public void Posts_수정된다() throws Exception{
        //given
        Posts savedPosts=postsRepository.save(Posts.builder()
                        .title("title")
                        .content("content")
                        .author("author")
                        .build());

        Long updateId= savedPosts.getId();
        String expectedTitle="title2";
        String expectedContent="content2";

        PostsUpdateRequestDto requestDto=
                PostsUpdateRequestDto.builder()
                        .title(expectedTitle)
                        .content(expectedContent)
                        .build();
        String url = "http://localhost:" + port+"/api/v1/posts/"+updateId;

        HttpEntity<PostsUpdateRequestDto> requestEntity =new HttpEntity<>(requestDto);

        //when
        ResponseEntity<Long> responseEntity=restTemplate.exchange(url,HttpMethod.PUT,requestEntity,Long.class);

        //then
        assertThat(responseEntity.getStatusCode()).
                isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).
                isGreaterThan(0L);
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);


    }
}
```

H2를 사용하기 위해선 웹 콘솔을 사용해야 한다. 

application.properties 설정 

```
spring.h2.console.enabled=true
```
## 3.5. JPA Auditing으로 생성시간/수정시간 자동화하기
- entity: 생성, 수정시간 포함.
- 모든 테이블과 서비스 메소드에 날짜 데이터 등록, 수정하는 코드 포함하려면 힘듦 -> JPA Auditing 사용

### LocalDate
Java8부터는 LocalDate와 LocalDateTime이 등장. Java의 기본 날짜 타입인 Date의 문제점을 고친 타입이라서 Java8부터는 무조건 써야 함.

- domain 패키지에 BaseTimeEntity 클래스 생성 
```java
@Getter
@MappedSuperclass //jpa entity 클래스들이 BaseTimeEntity을 상속할 경우 필드들(createdData,modifiedData)도 칼럼으로 인식하도록 함
@EntityListeners(AuditingEntityListener.class)//BaseTimeEntity 클래스에 Auditing 기능을 포함시킨다.

// 모든 Entity의 상위 클라스 -> Entity들의 createdDate, modifiedDate를 자동으로 관리하는 역할
public abstract class BaseTimeEntity {

    @CreatedDate//엔티티가 생성되어 저장될 떄의 시간이 자동 저장
    private LocalDateTime createdData;

    @LastModifiedDate//조회한 엔티티의 값을 변경할 떄의 시간이 자동 저장된다.
    private LocalDateTime modifiedData;
}
```

- Board 클래스가 BaseTimeEntity를 상속받을 수 있도록 변경
```java
	...
public class Board extends BaseTimeEntity {
	...
}
```

- JPA Auditing 어노테이션들을 모두 활성화하기 위해 Application 클래스에 활성화 어노테이션을 추가
```java
@EnableJpaAuditing//JPA Auditing 활성화
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

### JPA Auditing 테스트 코드 작성하기
```java
@Test
    public void BaseTimeEntity_등록() {
        //given
        LocalDateTime now = LocalDateTime.of(2019, 6, 4, 0, 0, 0);
        postsRepository.save(Posts.builder()
                .title("title")
                .content("content")
                .author("author")
                .build());
        //when
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);

        System.out.println(">>>>>>>>> createDate=" + posts.getCreateDate() + ", modifiedDate=" + posts.getModifiedDate());

        assertThat(posts.getCreateDate()).isAfter(now);
        assertThat(posts.getModifiedDate()).isAfter(now);
    }
```
