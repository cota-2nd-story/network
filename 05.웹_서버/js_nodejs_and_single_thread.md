# Javascript와 Nodejs, 그리고 싱글 스레드

 자바스크립트가 싱글 스레드 언어라는 것은 익히 들어본 이야기이다. 이 글에서는 자바스크립트가 왜 싱글스레드 언어라고 불리는지 내부 동작 원리를 통해 알아본다. 나아가 Nodejs는 내부적으로 어떻게 구성되어 있으며 Nodejs 또한 싱글스레드 환경에서 실행되는지에 대해 알아보고자 한다.

## 1. 싱글 스레드와 JS

### A. 싱글 스레드 언어인 Javascript

 자바스크립트가 싱글 스레드 언어라는 것은 당신이 작성한 자바스크립트 코드가 단일한 스레드에서 실행된다는 의미이다. 단일한 스레드는 자바스크립트의 동작을 구성하는 구조 상의 명령문을 순차적으로 실행시키는 공간이 하나 밖에 없다는 의미이다. 결론부터 말하자면 자바스크립트는 '콜 스택'이라는 자료 구조를 이용해서 명령문을 실행하는데, 하나의 스레드에서 하나의 콜 스택으로 당신이 작성한 모든 명령문을 순차적으로 실행하게 되는 것이 자바스크립트가 싱글 스레드 언어라고 불리는 이유이다.

**Javascript의 동작 원리**

 자바스크립트의 동작 원리에 대해서 알아보자.

![https://joshua1988.github.io/images/posts/web/translation/how-js-works/js-engine-runtime.png](https://joshua1988.github.io/images/posts/web/translation/how-js-works/js-engine-runtime.png)
<!--<figcaption>그림1. 자바스크립트의 동작 원리</figcaption>-->

- Call Stack: 콜 스택. 명령문이 쌓이며 하나씩 순차적으로 명령문을 실행하며 해당 명령문을 제거하는 자료 구조.
- Web APIs: 자바스크립트 혹은 더 넓은 범위로 웹 관련 기술을 지원하는 인터페이스의 모음.
- Event Loop: 이벤트 루프, Call Stack과 Callback Queue를 관찰하며 Call Stack이 비어있을때 Callback Queue에서 callback을 Call Stack에 쌓는 역할을 하는 일종의 Observer.
- Callback Queue(Task Queue): 콜백 큐, Web API에서 처리가 끝난 후 실행 되어야 하는 Callback(Task)이 순차적으로 쌓이는 자료 구조.

 위 그림의 구조는 싱글 스레드에서 이루어진다. 그런데 하나의 콜 스택에서 명령문이 순차적으로 실행된다 것은 지연(pending)을 일으키는 명령문이 있는 경우 프로그램 전체를 멈추게 한다는 의미이다. 그래서 자바스크립트는 지연을 일으키는 명령문들이 Web APIs에서 처리되도록 한다. 그때 명령문이 모두 처리된 후에 호출 될 함수를 하나 함께 넘기는 데 이를 콜백 함수(callback Function)라고 한다. Web APIs에서 처리된 명령문의 콜백 함수는 Callback Queue로 넘겨져 순서를 기다린다. 이때 이벤트 루프(Event Loop)는 실시간으로 두가지 일을 하는데, 하나는 콜 스택이 비워져있는 지 확인하는 것이고 다른 하나는 콜백 큐에 콜백이 있는 지이다. 이벤트 루프는 콜 스택이 비워져 있고 콜백 큐에 콜백이 있으면 그 콜백을 콜 스택으로 옮겨 실행되도록 한다.

 하나의 스레드에 하나의 콜 스택에서 지연이 걸리는 작업이 프로그램 전체를 멈추게 하지 않도록 지연이 걸리는 작업을 Web APIs에서 처리하고 이벤트 루프라는 일종의 옵저버로 실행 순서를 제어한다. 이러한 형태는 싱글 스레드 언어인 자바스크립트의 비동기 프로그래밍과 관련된다.

### B. setTimeout 0의 의미

    /** 일반적인 setTimeout의 형태 */
    let timeoutId = window.setTimeout(function [, delay, param1, param2, ...]);

 일반적으로 자바스크립트에서 일정 지연 시간 후에 명령문을 실행시키기 위해서 setTimeout을 사용한다. 그렇다면 만약 setTimeout의 두번째 인자인 delay에 매개 변수로 0을 넣으면 어떻게 될까? 0 밀리초 후에 실행하는 것이므로 즉시 실행되는 것과 동일한 의미일까?

  

    console.log(1);
    setTimeout(function() { console.log(2); }, 0);
    console.log(3);

위 코드를 실행하면 결과는 1 → 3 → 2 이다. 자바스크립트의 동작 원리를 다시 보면 setTimeout은 Web APIs에서 처리 되므로 콜백 함수는 0 밀리세컨드 간의 지연 후에 콜백 큐로 넘겨지게 된다. 그 사이 `console.log(3);`이 콜 스택에 먼저 쌓이게 되면서 이벤트 루프는 콜 스택이 비워질 때까지 기다렸다가 콜백 함수를 콜 스택에 넘길 수 있게 된다.

### C. Web Worker

 

 위에서 언급 한 것과 같이 자바스크립트는 싱글 스레드에서 동작하는 형태로 설계되어 있으며 이로 인해 발생하는 문제를 비동기 프로그래밍으로 극복하고자 한다. 물론 거의 대부분의 상황에서 별문제가 없기 때문에 자바스크립트는 많은 개발자들에게 사랑받으며 성장하고 있다. 하지만 비동기로 생각하며 문제를 해결해나가는 방식은 동기의 방식보다는 익숙해지기 전까지 어색하고 복잡한 경향이 있다. 또한 canvas에서 이뤄지는 무거운 이미지 처리와 예측하기 어려운 사용자 입력에 의한 이벤트 처리 그리고 화면에 나타나는 고프레임의 애니메이션이 모두 막힘없이 아름답게 비동기적으로 처리되도록 하는 것은 꽤 고민이 필요할 수 있다. 그래서 어떤 사람들은 이렇게 생각한 것 같다. '그냥 스레드를 더 늘려버리자.'

 웹 워커(Web Worker)는 Mozilla 재단과 구글이 협력해 만든 새로운 Web API이다. 브라우저는 탭 마다 하나의 스레드를 생성한다. 자바스크립트 프로그램은 이 메인 스레드(싱글 스레드)에서 작동하게 되는데, 웹 워커는 이 메인 스레드와는 별개로 브라우저 단의 새로운 스레드에서 실행된다. 그리고 메인 스레드와 웹 워커가 돌아가는 스레드는 서로 이벤트를 발생 시켜 커뮤니케이션한다. 웹 워커를 통해서 자바스크립트 개발자는 지연이 걸리는 다양한 작업을  메인 스레드의 동작과 별개로 실행할 수 있게 되었다.

## 2. 그러면, nodejs도 싱글스레드인가?

### A. nodejs를 자세히 들여다보기

 nodejs가 싱글스레드에서 돌아가는지에 앞서 nodejs이 어떻게 구성되어있는 지 알아보자.

**nodejs의 구성**

> nodejs는 브라우저가 아닌 외부에서 서버 사이드 스크립팅을 위해 자바스크립트를 실행하는 크로스 플랫폼 자바스크립트 런타임 환경이다.

 위 인용 문구는 참조한 자료에서 가져왔다. 읽었을 때 Nodejs가 대충 무엇을 하는 지는 알겠다. Nodejs를 처음 접했을 때 책에서 혹은 블로그에서 많이 봤던 내용인 것 같다. 하지만 그래서 어떻게 저런 일을 할 수 있는지가 궁금하다.

 

 

<img src="https://miro.medium.com/max/380/0*rBydgpa9yBemQswc"/>
<!--<figcaption>그림2. Nodejs 애플리케이션의 세가지 영역 - language</figcaption>-->

 Nodejs 애플리케이션은 세가지 영역으로 되어있다.

1. 우리가 작성하는 **자바스크립트 코드**.
2. 자바스크립트 코드를 C++ 코드로 트랜스파일링하는 **V8** 과 파일 시스템, 네트워킹 및 동시성과 같은 비동기 입출력(I/O) 기반의 연산을 지원하는 멀티 플랫폼 C++ 라이브러리인 **libuv**.
3. 우리가 작성한 자바스크립트 코드를 실행하는 역할과 libuv 라이브러리와 V8 사이의 인터페이스를 역할을 하는 역할을 하는 **Nodejs**.

 

 우리가 사용하는 Nodejs는 2, 3번이 합해진 형태로 구성 되어있다고 볼 수 있다. 그러면 2, 3번에서 언급된 V8, libuv, Nodejs에 대해서 좀 더 자세히 알아보자.

**Node**

 github에서 Nodejs 프로젝트의 여러 파일 확장자들을 보면 Javascript와 C++ 파일이 모두 포함된 것을 알 수 있다. /lib 하위의 파일 이름을 보면 nodejs의 내장 객체들(http, fs, crypto, path, ...)의 이름이 많이 보인다. 이 자바스크립트로 작성된 파일들은 우리가 Nodejs 앱을 만들 때 우리에게 필요한 인터페이스와 우리의 작성한 코드를 실행하는 역할을 한다. 그리고 /src 하위의 C++로 작성된 파일들은 libuv와 v8 라이브러리를 이용하기 위해 작성된 코드이다. 

**V8**

 V8 엔진은 구글에서 만들어진 Javascript 엔진이다. Javascript 엔진이란 Javascript를 컴파일 혹은 인터프리팅하는 프로그램으로 볼 수 있다. V8  이외에도 많은 엔진들이 존재한다. Nodejs는 많은 엔진들 중 V8을 품고 있다. Nodejs에서 V8 엔진의 역할은 주로 Javasciript 코드를 C++ 코드로 컨버팅하는 것이다.

**libuv**

 libuv는 비동기 입, 출력에 초점을 둔 멀티 플랫폼 지원 라이브러리이다. 다양한 네트워크, OS에 관련된 기능을 포함하고 있으며 주로 Nodejs에서 사용하기 위해 개발되었다. 기존에 브라우저에서만 구동 되던 자바스크립트가 브라우저를 벗어난 환경에서 구동될 수 있도록 새로운 장을 열어주기 위해 개발된 라이브러리로 볼 수 있다. Nodejs 내부에서는 V8이 컨버팅한 C++코드를 이해하고 실행한다.

아래 다이어그램은 Node 애플리케이션 내부에서 어떤 일이 일어나는지 알 수 있다.

 

<img src="https://miro.medium.com/max/352/0*aToI5oI5HYZLQ172"/>
<!--<figcaption>그림 3. Node 애플리케이션 내부에서 어떤 일이 일어나는가?</figcaption>-->

 우리가 작성한 자바스크립트 코드는 Nodejs의 lib 폴더 하위의 자바스크립트 부에서 실행된다. `Process.binding` 은 C++와 JS을 연결하는 역할을 수행한다. V8은 JS 코드를 C++ 코드로 트랜스파일링하는 작업을 수행하고 Nodejs의 src 폴더 하위의 C++ 부에서 V8로부터 컨버팅 된 코드를 libuv 라이브러리를 이용해 실행한다.

 

### B. nodejs와 싱글 스레드

 결론부터 말하면 nodejs는 기본적으로 이벤트루프가 작동하는 싱글 스레드에서 실행된다. 하지만 예외적으로 Nodejs가 포함하는 일부 라이브러리는 싱글 스레드에서 실행되지 않는 경우가 있다. 참조한 자료에서는 nodejs의 내장 객체 중 하나인 `crypto` 를 예시로 들고 있다. crypto를 이용해 암호화할때 사용하는 `crypto.pbkdf2` 함수를 이용해 해시 알고리즘의 반복 횟수를 10만번 정도로 지정하면 보통 500 ~ 1000 밀리초 정도가 소요된다. 그래서 순차적으로 이  함수를 4번 호출 했다고 가정할 때, 각각 호출 되는 시간의 간격을 console에 찍어 보면 500 ~ 1000 밀리초 간격이 나타나야하지만 결과는 그렇지 않다. 아래는 그 결과이다.

<img src="https://miro.medium.com/max/558/1*GOXPsN29oco0TCgncXFVkg.png"/>
<!--<figcaption>그림 4. crypto.pbkdf2를 동기적으로 4회 호출했을때 소요된 시간 로그</figcaption>-->

 함수 호출 간의 시간 간격이 10 밀리초 이하인 것을 알 수 있다. 어떻게 가능 한 것일까? Nodejs의 구성을 다시 생각해보면 Nodejs는 네트워크와 관련된 기능과 OS 관련 작업을 처리하는 libuv 라이브러리를 사용해 JS코드로부터 컨버팅 된 C++ 코드를 실행한다. 그런데, libuv는 OS 관련 처리를 수행하기 위해 4개의 스레드 풀을 세팅한다. 싱글 스레드 언어인 자바스크립트는 할 수 없지만 libuv의 C++ 코드로는 OS 스레드 스케줄러에 관여하여 멀티 스레드로 Nodejs가 실행될 수 있는 환경을 만들 수 있는 것이다. 아래는 이에 대한 내용을 다이어그램으로 간략히 보여준다.

<img src="https://miro.medium.com/max/427/1*OWlBzRwRk3lC_ikVErv4cw.png"/>
<!--<figcaption>그림 5. Nodejs 애플리케이션에서  pbkdf2 모듈이 작동되는 흐름</figcaption>-->

 자바스크립트는 싱글 스레드 언어이지만 Nodejs는 자바스크립트 만으로 이루어지지 않았다. 자바스크립트가 브라우저를 벗어나 작동할 수 있게 하기 위해 libuv 라이브러리를 이용하게 되었고, 자연스럽게 C++ 언어로 이루어진 이 라이브러리는 OS 단의 작업을 수행하여 스레드의 개수를 느릴 수 있었다. Nodejs는 기본적으로 싱글스레드로 작동하지만 일부 모듈은 멀티 스레드로 작동한다고 볼 수 있다.

## Reference


- [Parallel programming in JavaScript using Web Workers](https://itnext.io/achieving-parallelism-in-javascript-using-web-workers-8f921f2d26db)
- [Node.Js Under the Hood](https://medium.com/better-programming/learn-node-js-under-the-hood-37966a20e127)
- [Is Node.js Really Single-Threaded?](https://medium.com/better-programming/is-node-js-really-single-threaded-7ea59bcc8d64)
- [What the heck is the event loop anyway?](https://www.youtube.com/watch?time_continue=1&v=8aGhZQkoFbQ&feature=emb_logo)
- [자바스크립트의 동작원리: 엔진, 런타임, 호출 스택](https://joshua1988.github.io/web-development/translation/javascript/how-js-works-inside-engine/#%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%97%94%EC%A7%84)
