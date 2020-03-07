# HTTP Method



흔히 웹 어플리케이션을 개발할 때 사용하는(CRUD) HTTP 메서드를 간략하게 알아보자.



### GET

* GET 메서드는 가장 많이 사용되는 메서드다. 
* 흔히 리소스를 읽어오고자할 때 사용하는 메서드다.
* READ의 성격이 강하며 멱등성이 보장된다.
* 정적, 동적 구분없이 클라이언트가 원하는 리소스를 필요로할 때 사용된다.

```text
GET /articles/123
```

* 위 요청은 123번의 게시글을 달라는 요청으로 이해할 수 있다.
* URI로 클라이언트가 읽어오려는 값을 명시하면서 GET의 HTTP 요청을 보낸다.
* 다른 예시

<img width="276" alt="스크린샷 2020-03-07 오후 10 06 18" src="https://user-images.githubusercontent.com/30451129/76144064-f560c000-60bf-11ea-8d7a-5cd370a77f6e.png">

> 위 예시는 크롬 브라우저에서 네이버 홈페이지를 들어갈 때 일어난 요청 정보다. 저 요청으로 인해 네이버 서버는 html 타입의 응답 바디를 내려준다. 즉, 위 요청은 네이버 홈페이지를 브라우저에서 렌더링하기 위한 html 파일을 달라는 요청이다. 이 요청에 의해 리소스에 대한 어떠한 값도 변경되지 않았으며 서버는 그저 어딘가에 존재하는 html 파일을 내려줬다.



### POST

* 흔히 새로운 리소스를 생성하고자 할 때 사용되는 메서드다.
* 생성뿐만아니라 body에 담긴 데이터를 가지고 특정 행위(로그인)를 할 때 사용하는 메서드다.
* html에서 form 태그를 만들 때 "method=post"로 설정 많이 하는데 form 태그의 input 태그의 값을 서버에 새롭게 추가하겠다는 의미다.
* 이 메서드는 비멱등성을 가지고 있어 파이프라인 커넥션에 사용되면 안된다.

```text
POST /articles
```

* 위 요청은 요청 메세지 body에 있는 값으로 게시글을 생성해달라는 요청이다.
* CREATE 성격을 가지는 메서드다.
* 예시

<img width="364" alt="스크린샷 2020-03-07 오후 9 28 11" src="https://user-images.githubusercontent.com/30451129/76144085-20e3aa80-60c0-11ea-8121-59396ab564c0.png">
<img width="606" alt="스크린샷 2020-03-07 오후 9 28 22" src="https://user-images.githubusercontent.com/30451129/76144086-23460480-60c0-11ea-9525-0dc496702647.png">

> 위 요청은 github 로그인시 일어나는 요청 정보다. 해당 URL과 POST 방식으로 요청이 서버에 전달된다. 밑에 있는 스크린샷을 보면 Form Data로 요청 body에 실려 함께 전달된다. 서버는 POST 요청에 대해 body를 확인하여 원하는 값을 추출 후 요청에 맞는 처리를 진행한다. 지금 위 요청은 로그인이기 때문에 DB에 로그인을 시도하는 계정의 정보 확인을 위해 body에 아이디와 비밀번호가 함께 전달되고 있다.



### PUT

* 리소스를 수정(변경)할 때 사용되는 메서드다.
* 서버는 요청 body에 함께 전달되는 정보로 원하는 리소스를 수정한다.

```text
PUT /articles/123
```

* 위 요청은 요청 메세지 body의 값으로 123번 게시글을 수정하려는 메세지다.
* UPDATE 성격의 메서드다.
* 예시

<img width="388" alt="스크린샷 2020-03-07 오후 9 40 32" src="https://user-images.githubusercontent.com/30451129/76144092-33f67a80-60c0-11ea-8837-b859cea8283a.png">
<img width="214" alt="스크린샷 2020-03-07 오후 9 40 48" src="https://user-images.githubusercontent.com/30451129/76144095-36f16b00-60c0-11ea-8727-0360a65c2af7.png">

> 위 요청은 gaejangmo.com 이라는 아주 유명한 웹 페이지에서 로그인한 사용자의 별명을 수정하는 요청이다. 요청 body에 `value : "우아한"` 이라는 payload가 전달되어 현재 로그인된 사용자의 정보를 조회한 뒤 특정 값을 body로 넘어온 payload 값으로 수정된다. POST 예시에서는 Form Data 형식으로 데이터가 전달되었고 여기서는 payload로 전달되었다. 이는 요청 메세지의 Content-Type 헤더 값에 따라 전달되는 데이터의 형식이 다르기 때문이다. 전자는 `content-type: application/x-www-form-urlencoded` 이며 위 예시인 경우에는 `content-type: application/json;charset=UTF-8` 이다.



### DELETE

* 특정 리소스를 삭제할 때 사용되는 메서드다.

```text
DELETE /articles/123
```

* 위 요청은 123번 게시글을 삭제하겠다라는 요청이다.
* READ 적 성격의 GET 메서드를 제외한 나머지 3개의 메서드는 사용자 요청에 대한 올바른 사용자 검증이 필요하다.
  * 자신의 게시글도 아닌데 삭제하게되면 안되기 때문이다.
* DELETE 성격의 메서드다.
* 예시

<img width="435" alt="스크린샷 2020-03-07 오후 10 01 50" src="https://user-images.githubusercontent.com/30451129/76144101-41136980-60c0-11ea-861e-685cfd7e4e75.png">

> 위 요청도 매우 유명한 페이지인 gaejangmo.com에서 사용자가 올린 제품 정보를 지우는 요청이다. 114번 user/product를 지우겠다는 의미다. DELETE 요청은 별다른 body 값을 넣을 필요없다. 삭제하려는 리소스의 식별 값을 URL로 충분히 나태낼 수 있기 때문이다.

