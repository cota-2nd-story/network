# Options Method

Options는 브라우저에서 cross origin ajax request가 있을 때 서버에 날려지는 initial request에 쓰인다.
이에 대해 더 알아보고자 한다.


### 교차 출처 리소스 공유(Cross-Origin Resource Sharing = CORS)

>  한 출처에서 실행 중인 웹 애플리케이션이 다른 출처의 선택한 자원에 접근할 수 있는 권한을 부여하도록 "추가 HTTP 헤더를 사용하여" 브라우저에 알려주는 체제

* XMLHttpRequest와 Fetch API, 웹 폰트(@font-face에서 교차 도메인 푠트 사용 시), WebGL 텍스쳐, drawImage()를 사용하여 캔버스에 그린 이미지/비디오 프레임, 이미지로부터 추출하는 css shape은 동일 출처 정책을 따른다.
* 이 API를 사용하는 웹 애플리케이션은 자신의 출처와 동일한 리소스만 불러올 수 있다.
* 다른 출처의 리소스를 불러오려면 그 출처에서 올바른 CORS 헤더(Access-Control-Allow-Origin)를 포함한 응답을 반환해야 한다.


### Options method & Preflight(사전 전달) 요청

> side effect를 일으킬 수 있는 HTTP 요청 메서드에 대해 브라우저가 Options 메서드에 preflight하여 접근 가능 여부를 요청하고, 서버의 허가가 떨어지면 실제 요청을 보낸다.


#### Simple request (단순 요청): side effect가 없을 때

아래 경우들에는 단순 요청을 한다.
* GET, HEAD, POST 요청
* 사용자가 임의로 지정한 header가 없음
* Content-Type이 application/x-www-form-urlencoded, multipart/form-data, text/plain 중 하나
* XMLHttpRequestUpload에 이벤트 리스너가 등록되어 있으면 안됨
* request에 ReadableStream이 있으면 안됨

단순 요청의 경우를 예를 들면 아래와 같다.

Requset from client
```text 
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example
```

Response from server
```text
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[…XML Data…]
```
위의 예시를 보면 requset header에 Origin이 포함되어있고, response header에 Access-Control-Allow-Origin이 포함되어 있다. 단순 요청의 경우 preflight request 없이도 위와 같이 request를 보내고 response를 받을 수 있다.


#### Preflight request (사전 전달 요청)

단순 요청을 할 수 없는 경우에는 preflight request를 한다.

<img width="276" alt="preflight request example" src="https://mdn.mozillademos.org/files/16753/preflight_correct.png">

위 예시는, POST에 X-PINGOTHER이라는 사용자 지정 header를 담아서 보내려 할 때, 일어난 prefight request이다. Preflight request의 과정은 아래와 같다.

1. client에서 Access-Control-Request-*, Origin 헤더를 포함한 OPTIONS 요청을 보낸다.
2. server에서 Access-Control-Allow-* 헤더를 포함하여 접근 가능 여부를 알려주는 response를 보낸다.
3. client에서 본래 보내려 했던 request를 보낸다.
4. server에서 response를 보낸다.

CORS 요청을 핸들링 하기 위해서는 서버에 OPTIONS 메서드 요청을 받아 컨트롤 할 수 있는 설정이 필요하다. (특히 특정 origin에서의 요청만을 받고 싶을 경우에는 필수)

참고자료
*  https://developer.mozilla.org/ko/docs/Web/HTTP/CORS