## 4.1 서버 템플릿 엔진과 머스테치 소개

---

### 템플릿 엔진이란?

- 지정된 **템플릿 양식과 데이터** 가 합쳐져 HTML 문서를 출력하는 소프트웨어
- **서버 템플릿 엔진**
    - JSP, Freemarker
- **클라이언트 템플릿 엔진**
    - React, Vue 의 View 파일들


### 서버 템플릿 엔진

- 서버에서 구동됨
- 화면 생성 과정
    - **서버에서 Java 코드로 문자열 생성 → 문자열을 HTML로 변환 → 브라우저로 전달**


### 클라이언트 템플릿 엔진

- 화면 생성 과정
    - 브라우저에서 화면 생성
    - **서버에서는 Json이나 Xml 형식의 데이터만 전달 → 클라이언트에서 조립**
- 서버 사이드 렌더링
    - React, Vue 와 같은 자바스크립트 프레임워크의 화면 생성 방식을 서버에서 실행


### 머스테치란?

- Ruby, JavaScript, Python, PHP, Java, Pearl, Go, ASP 등 **다양한 언어를 지원하는 템플릿 엔진**
    - Java에서 사용될 때는 서버 템플릿 엔진으로, JavaScript에서 사용될 때는 클라이언트 템플릿 엔진으로 사용
- 장점
    - 심플한 문법
    - 로직 코드 사용 불가 → View와 서버의 명확한 역할 분리
    - 하나의 문법으로 서버/클라이언트 템플릿 모두 사용 가능
    


## 4.2 기본 페이지 만들기

---

1. ***build.gradle*** 에 의존성 등록

```java
implementation('org.springframework.boot:spring-boot-starter-mustache')
```

2. ***src/main/resources/templates*** 에 ***index.mustache*** 생성

```html
<!DOCTYPE HTML>
<html>
	<head>
		<title>스프링 부트 웹서비스</title>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
	</head>

	<body>
		<h1>스프링 부트로 시작하는 웹 서비스</h1>
	</body>
</html>
```

3. ***web*** 패키지 안에 ***IndexController*** 생성 후 URL Mapping

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class IndexController {
	@GetMapping("/")
	public String index(){
		return "index";
	}
}
```

- 머스테치 스타터에 의해 컨트롤러에서 문자열 반환 시 앞의 경로와 뒤의 파일 확장자는 자동으로 지정
    - "index" → src/main/resources/templates/index.mustache
4. ***test*** 패키지에 ***IndexControllerTes***t 클래스 생성해 테스트코드 검증

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;
import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.boot.test.context.SpringBootTest.WebEnvironment.RANDOM_PORT;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = RANDOM_PORT)
public class IndexControllerTest {
    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void 메인페이지_로딩(){
        //when
        String body=this.restTemplate.getForObject("/",String.class);

        //then
        assertThat(body).contains("스프링 부트로 시작하는 웹서비스");
    }
}
```

## 4.3 게시글 등록 화면 만들기

---

### 프론트엔드 라이브러리 사용

- 외부 CDN 사용
    - 외부 서비스에 의존하게 되어 실제 서비스에서는 자주 사용 X
- 직접 라이브러리 받아서 사용

### 레이아웃 방식이란?

- **공통 영역을 별도의 파일로 분리**하여 필요한 곳에서 가져다 쓰는 방식
1. ***src/main/resources/templates/layout*** 디렉토리에 ***header.mustache*** 생성

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
</head>
<body>
```

2. ***src/main/resources/templates/layout*** 디렉토리에 ***footer.mustache*** 생성

```html
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>

</body>
</html>
```

3. ***index.mustache*** 코드 수정

```html
{{>layout/header}}

<h1>스프링 부트로 시작하는 웹서비스 Ver.2</h1>

{{>layout/footer}}
```

### 글 등록 기능

1. ***index.mustache*** 에 글 등록 버튼 추가

```html
{{>layout/header}}

<h1>스프링 부트로 시작하는 웹서비스 Ver.2</h1>

<div class="col-md-12">
    <div class="row">
        <div class="col-md-6">
            <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
        </div>
    </div>
</div>

{{>layout/footer}}
```

2. ***IndexController*** 에서 /posts/save 에 대한 URL Mapping

```java
@RequiredArgsConstructor
@Controller
public class IndexController {
	...
  @GetMapping("/posts/save")
  public String postsSave(){
    return "posts-save";
  }
}
```

3. ***posts-save.mustache*** 파일 생성

```html
{{>layout/header}}

<h1>게시글 등록</h1>

<div class="col-md-12">
    <div class="col-md-4">
        <form>
            <div class="form-group">
                <label for="title">제목</label>
                <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
            </div>
            <div class="form-group">
                <label for="author">작성자</label>
                <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요">
            </div>
            <div class="form-group">
                <label for="content">내용</label>
                <textarea class="form-control" id="content" placeholder="내용을 입력하세요"></textarea>
            </div>
        </form>
        <a href="/" role="button" class="btn btn-secondary">취소</a>
        <button type="button" class="btn btn-primary" id="btn-save">등록</button>
    </div>
</div>

{{>layout/footer}}
```

4. ***src/main/resources*** 에 ***static/js/app*** 디렉토리 생성
5. ***static/js/app*** 디렉토리에 ***index.js*** 생성

```jsx
var main={
    init:function(){
        var _this=this;
        $('#btn-save').on('click',function(){
            _this.save();
        });
    },
    save:function(){
        var data={
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };

        $.ajax({
            type: 'POST',
            url: '/api/v1/posts',
            dataType: 'json',
            contentType: 'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function(){
            alert('글이 등록되었습니다.');
            window.location.href="/";
        }).fail(function(error){
            alert(JSON.stringify(error));
        });
    }
};

main.init();
```

- ajax를 이용해 서버와 데이터 주고받음
- window.location.href='/'
    - 글 등록이 성공하면 메인 페이지로 이동
- index.mustache 에 다른 .js 파일이 추가되어 중복된 함수 이름이 생기더라도, index.js 만의 유효 범위를 가지도록 var main 사용

6. ***index.js*** 를 ***footer.mustache*** 에 추가

```html
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>

<!--index.js 추가-->
<script src="/js/app/index.js"></script>
</body>
</html>
```

- index.js 호출 코드에서 절대 경로(/) 로 시작
    - 스프링부트에서 기본적으로 /src/main/resources/static 에 위치한 정적 파일들은 URL에서 /로 설정됨
