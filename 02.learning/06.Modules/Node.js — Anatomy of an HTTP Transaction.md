# HTTP 트랜잭션의 구조

이 가이드는 Node.js에서 HTTP 요청을 처리하는 과정을 명확히 이해하는 데 목적을 둡니다. 여러분이 일반적으로 HTTP 요청이 어떻게 동작하는지 알고 있다고 가정하겠습니다. 또한 Node.js의 [`EventEmitters`](https://nodejs.org/api/events.html)와 [`Streams`](https://nodejs.org/api/stream.html)에 어느 정도 익숙하다고 가정합니다. 만약 이 개념들이 익숙하지 않다면, 각 API 문서를 빠르게 읽어보는 것이 좋습니다.


## 서버 생성하기

모든 Node.js 웹 서버 애플리케이션은 어느 시점에서 웹 서버 객체를 생성해야 합니다. 이는 [`createServer`](https://nodejs.org/api/http.html#http_http_createserver_requestlistener)를 사용하여 수행됩니다.

```javascript
const http = require('node:http');

const server = http.createServer((request, response) => {
  // 여기서 마법이 일어납니다!
});
```

[`createServer`](https://nodejs.org/api/http.html#http_http_createserver_requestlistener)에 전달된 함수는 서버에 대한 모든 HTTP 요청마다 한 번씩 호출되며, 이를 **요청 핸들러**라고 합니다. 사실, [`createServer`](https://nodejs.org/api/http.html#http_http_createserver_requestlistener)가 반환하는 [`Server`](https://nodejs.org/api/http.html#http_class_http_server) 객체는 [`EventEmitter`](https://nodejs.org/api/events.html#events_class_eventemitter)이며, 여기서 사용한 방식은 `server` 객체를 생성한 후 나중에 리스너를 추가하는 것의 축약형입니다.

```javascript
const server = http.createServer();
server.on('request', (request, response) => {
  // 같은 종류의 마법이 여기서도 일어납니다!
});
```

HTTP 요청이 서버에 도달하면, Node.js는 해당 요청을 처리하기 위한 유용한 객체인 `request`와 `response`를 사용하여 요청 핸들러 함수를 호출합니다. 이에 대해서는 곧 자세히 알아보겠습니다.

실제로 요청을 처리하려면, `server` 객체에서 [`listen`](https://nodejs.org/api/http.html#http_server_listen_port_hostname_backlog_callback) 메서드를 호출해야 합니다. 대부분의 경우, `listen`에 전달해야 하는 것은 서버가 리스닝할 포트 번호뿐입니다. 다른 옵션들도 있으니, [API 참조](https://nodejs.org/api/http.html)를 확인해보세요.


## [메서드, URL 및 헤더](https://nodejs.org/en/learn/modules/publishing-a-package#method-url-and-headers)

요청을 처리할 때 가장 먼저 확인해야 할 것은 메서드와 URL입니다. 이를 통해 적절한 작업을 수행할 수 있습니다. Node.js는 `request` 객체에 편리한 속성을 제공하여 이 작업을 비교적 쉽게 만듭니다.

```javascript
const { method, url } = request;
```

> `request` 객체는 [`IncomingMessage`](https://nodejs.org/api/http.html#http_class_http_incomingmessage)의 인스턴스입니다.

여기서 `method`는 항상 일반적인 HTTP 메서드/동사입니다. `url`은 서버, 프로토콜 또는 포트를 제외한 전체 URL입니다. 일반적인 URL의 경우, 이는 세 번째 슬래시 이후의 모든 것을 의미합니다.

헤더도 쉽게 접근할 수 있습니다. 헤더는 `request` 객체 내 `headers`라는 별도의 객체에 있습니다.

```javascript
const { headers } = request;
const userAgent = headers['user-agent'];
```

여기서 주목할 점은 클라이언트가 실제로 헤더를 어떻게 보냈는지와 상관없이 모든 헤더가 소문자로 표현된다는 것입니다. 이는 헤더를 파싱하는 작업을 단순화합니다.

일부 헤더가 반복되는 경우, 헤더에 따라 값이 덮어쓰이거나 쉼표로 구분된 문자열로 결합됩니다. 경우에 따라 이는 문제가 될 수 있으므로 [`rawHeaders`](https://nodejs.org/api/http.html#http_message_rawheaders)도 사용할 수 있습니다.


## [Request Body](https://nodejs.org/en/learn/modules/publishing-a-package#request-body)

`POST`나 `PUT` 요청을 받을 때, 요청 본문은 여러분의 애플리케이션에서 중요한 역할을 할 수 있습니다. 요청 헤더에 접근하는 것보다 요청 본문 데이터를 얻는 것은 조금 더 복잡합니다. 핸들러에 전달된 `request` 객체는 [`ReadableStream`](https://nodejs.org/api/stream.html#stream_class_stream_readable) 인터페이스를 구현합니다. 이 스트림은 다른 스트림과 마찬가지로 이벤트를 수신하거나 다른 곳으로 파이프할 수 있습니다. 스트림의 `'data'`와 `'end'` 이벤트를 수신하여 데이터를 직접 가져올 수 있습니다.

각 `'data'` 이벤트에서 방출되는 청크는 [`Buffer`](https://nodejs.org/api/buffer.html)입니다. 만약 문자열 데이터가 될 것임을 알고 있다면, 데이터를 배열에 수집한 후 `'end'` 이벤트에서 이를 연결하고 문자열로 변환하는 것이 가장 좋습니다.

```javascript
let body = [];
request
  .on('data', chunk => {
    body.push(chunk);
  })
  .on('end', () => {
    body = Buffer.concat(body).toString();
    // 이 시점에서 `body`는 전체 요청 본문을 문자열로 저장하고 있습니다
  });
```

이 과정이 다소 번거롭게 느껴질 수 있고, 실제로 많은 경우 그렇습니다. 다행히도, [`npm`](https://www.npmjs.com)에는 [`concat-stream`](https://www.npmjs.com/package/concat-stream)이나 [`body`](https://www.npmjs.com/package/body)와 같은 모듈이 있어 이 로직을 숨기는 데 도움을 줄 수 있습니다. 하지만 이러한 모듈을 사용하기 전에 내부적으로 어떤 일이 일어나는지 잘 이해하는 것이 중요합니다. 그래서 여러분이 여기 있는 거죠!


## [에러에 대한 간단한 설명](https://nodejs.org/en/learn/modules/publishing-a-package#a-quick-thing-about-errors)

`request` 객체는 [`ReadableStream`](https://nodejs.org/api/stream.html#stream_class_stream_readable)이기 때문에, 동시에 [`EventEmitter`](https://nodejs.org/api/events.html#events_class_eventemitter)이기도 합니다. 따라서 에러가 발생할 때 `EventEmitter`처럼 동작합니다.

`request` 스트림에서 에러가 발생하면, 스트림에서 `'error'` 이벤트를 발생시킵니다. **이 이벤트에 대한 리스너가 없으면 에러가 *던져지며*, 이로 인해 Node.js 프로그램이 중단될 수 있습니다.** 따라서 `request` 스트림에 `'error'` 리스너를 추가하는 것이 좋습니다. 단순히 에러를 로그로 기록하고 계속 진행하는 것도 가능하지만, (아마도 HTTP 에러 응답을 보내는 것이 더 나을 것입니다. 이에 대해서는 나중에 더 설명하겠습니다.)

```javascript
request.on('error', err => {
  // 이 코드는 에러 메시지와 스택 트레이스를 `stderr`에 출력합니다.
  console.error(err.stack);
});
```

이러한 에러를 처리하는 다른 방법들도 있습니다. 예를 들어 다른 추상화 도구나 유틸리티를 사용할 수 있지만, 항상 에러가 발생할 수 있다는 점을 인지하고 이를 처리할 준비가 되어 있어야 합니다.


## [지금까지의 내용](https://nodejs.org/en/learn/modules/publishing-a-package#what-weve-got-so-far)

지금까지 서버를 생성하고, 요청에서 메서드, URL, 헤더, 본문을 추출하는 방법을 다뤘습니다. 이 모든 것을 합치면 다음과 같은 코드가 나올 수 있습니다.

```javascript
const http = require('node:http');

http
  .createServer((request, response) => {
    const { headers, method, url } = request;
    let body = [];
    request
      .on('error', err => {
        console.error(err);
      })
      .on('data', chunk => {
        body.push(chunk);
      })
      .on('end', () => {
        body = Buffer.concat(body).toString();
        // 이 시점에서 헤더, 메서드, URL, 본문을 모두 가지고 있으며,
        // 이제 요청에 응답하기 위해 필요한 작업을 수행할 수 있습니다.
      });
  })
  .listen(8080); // 이 서버를 활성화하고, 8080 포트에서 요청을 기다립니다.
```

이 예제를 실행하면 요청을 *받을* 수는 있지만, *응답*을 보내지는 못합니다. 실제로 웹 브라우저에서 이 예제에 접속하면, 클라이언트에게 아무것도 보내지 않기 때문에 요청이 타임아웃될 것입니다.

지금까지는 `response` 객체를 전혀 다루지 않았습니다. 이 객체는 [`ServerResponse`](https://nodejs.org/api/http.html#http_class_http_serverresponse)의 인스턴스이며, [`WritableStream`](https://nodejs.org/api/stream.html#stream_class_stream_writable)입니다. 이 객체는 클라이언트에게 데이터를 보내기 위한 여러 유용한 메서드를 포함하고 있습니다. 다음에 이에 대해 다루겠습니다.


## [HTTP 상태 코드](https://nodejs.org/en/learn/modules/publishing-a-package#http-status-code)

HTTP 응답의 상태 코드를 따로 설정하지 않으면 기본값으로 200이 반환됩니다. 하지만 모든 HTTP 응답이 이 상태 코드를 사용하는 것은 아니며, 상황에 따라 다른 상태 코드를 보내야 할 때가 있습니다. 이때 `statusCode` 속성을 설정하면 됩니다.

```javascript
response.statusCode = 404; // 클라이언트에게 리소스를 찾을 수 없음을 알립니다.
```

이 외에도 상태 코드를 설정하는 다른 방법들이 있는데, 이에 대해서는 곧 알아보겠습니다.


## 응답 헤더 설정하기

헤더는 [`setHeader`](https://nodejs.org/api/http.html#http_response_setheader_name_value)라는 편리한 메서드를 통해 설정할 수 있습니다.

```javascript
response.setHeader('Content-Type', 'application/json');
response.setHeader('X-Powered-By', 'bacon');
```

응답에 헤더를 설정할 때, 헤더 이름은 대소문자를 구분하지 않습니다. 만약 같은 헤더를 여러 번 설정한다면, 마지막에 설정한 값이 전송됩니다.


## 헤더 데이터 명시적으로 보내기

지금까지 다룬 헤더와 상태 코드 설정 방법은 "암시적 헤더"를 사용한다고 가정합니다. 이는 여러분이 본문 데이터를 보내기 전에 Node가 적절한 시점에 헤더를 보내주기를 기대한다는 의미입니다.

원한다면, 헤더를 *명시적으로* 응답 스트림에 작성할 수 있습니다. 이를 위해 [`writeHead`](https://nodejs.org/api/http.html#http_response_writehead_statuscode_statusmessage_headers)라는 메서드를 사용할 수 있습니다. 이 메서드는 상태 코드와 헤더를 스트림에 작성합니다.

```javascript
response.writeHead(200, {
  'Content-Type': 'application/json',
  'X-Powered-By': 'bacon',
});
```

헤더를 설정한 후(암시적이든 명시적이든), 응답 데이터를 보낼 준비가 된 것입니다.


## 응답 본문 전송하기

`response` 객체는 [`WritableStream`](https://nodejs.org/api/stream.html#stream_class_stream_writable)이기 때문에, 클라이언트에게 응답 본문을 보내는 것은 일반적인 스트림 메서드를 사용하는 것과 같습니다.

```javascript
response.write('<html>');
response.write('<body>');
response.write('<h1>Hello, World!</h1>');
response.write('</body>');
response.write('</html>');
response.end();
```

스트림의 `end` 함수는 스트림의 마지막 데이터로 보낼 선택적 데이터를 받을 수 있습니다. 따라서 위의 예제를 다음과 같이 간단하게 만들 수 있습니다.

```javascript
response.end('<html><body><h1>Hello, World!</h1></body></html>');
```

> 응답 본문에 데이터를 쓰기 전에 상태 코드와 헤더를 먼저 설정하는 것이 중요합니다. 이는 HTTP 응답에서 헤더가 본문보다 앞에 오기 때문에 당연한 일입니다.


## 에러에 대한 또 다른 간단한 설명

`response` 스트림도 `'error'` 이벤트를 발생시킬 수 있습니다. 언젠가는 이 이벤트도 처리해야 할 때가 옵니다. `request` 스트림 에러에 대한 모든 조언이 여기서도 동일하게 적용됩니다.


## [모든 것을 합쳐보기](https://nodejs.org/en/learn/modules/publishing-a-package#put-it-all-together)

이제 HTTP 응답을 만드는 방법을 배웠으니, 모든 것을 합쳐서 실습해보겠습니다. 앞서 다룬 예제를 기반으로, 사용자가 보낸 모든 데이터를 다시 보내는 서버를 만들어볼 것입니다. 이 데이터는 `JSON.stringify`를 사용해 JSON 형식으로 변환할 것입니다.

```javascript
const http = require('node:http');

http
  .createServer((request, response) => {
    const { headers, method, url } = request;
    let body = [];
    request
      .on('error', err => {
        console.error(err);
      })
      .on('data', chunk => {
        body.push(chunk);
      })
      .on('end', () => {
        body = Buffer.concat(body).toString();
        // 새로운 코드 시작

        response.on('error', err => {
          console.error(err);
        });

        response.statusCode = 200;
        response.setHeader('Content-Type', 'application/json');
        // 참고: 위의 두 줄은 다음 한 줄로 대체할 수 있습니다.
        // response.writeHead(200, {'Content-Type': 'application/json'})

        const responseBody = { headers, method, url, body };

        response.write(JSON.stringify(responseBody));
        response.end();
        // 참고: 위의 두 줄은 다음 한 줄로 대체할 수 있습니다.
        // response.end(JSON.stringify(responseBody))

        // 새로운 코드 끝
      });
  })
  .listen(8080);
```


## [에코 서버 예제](https://nodejs.org/en/learn/modules/publishing-a-package#echo-server-example)

이전 예제를 간단히 만들어서 요청으로 받은 데이터를 그대로 응답으로 보내는 간단한 에코 서버를 만들어 보겠습니다. 요청 스트림에서 데이터를 가져와 응답 스트림에 쓰는 작업만 하면 됩니다. 이전에 했던 것과 비슷합니다.

```javascript
const http = require('node:http');

http
  .createServer((request, response) => {
    let body = [];
    request
      .on('data', chunk => {
        body.push(chunk);
      })
      .on('end', () => {
        body = Buffer.concat(body).toString();
        response.end(body);
      });
  })
  .listen(8080);
```

이제 이 코드를 조금 수정해 보겠습니다. 다음과 같은 조건에서만 에코를 보내려고 합니다:

- 요청 메서드가 POST인 경우
- URL이 `/echo`인 경우

그 외의 경우에는 404로 응답합니다.

```javascript
const http = require('node:http');

http
  .createServer((request, response) => {
    if (request.method === 'POST' && request.url === '/echo') {
      let body = [];
      request
        .on('data', chunk => {
          body.push(chunk);
        })
        .on('end', () => {
          body = Buffer.concat(body).toString();
          response.end(body);
        });
    } else {
      response.statusCode = 404;
      response.end();
    }
  })
  .listen(8080);
```

> URL을 이렇게 확인하는 방식은 일종의 "라우팅"입니다. 다른 형태의 라우팅은 `switch` 문처럼 간단할 수도 있고, [`express`](https://www.npmjs.com/package/express)와 같은 전체 프레임워크처럼 복잡할 수도 있습니다. 라우팅만 필요한 경우 [`router`](https://www.npmjs.com/package/router)를 사용해 보세요.

좋습니다! 이제 이 코드를 더 간단히 만들어 보겠습니다. `request` 객체는 [`ReadableStream`](https://nodejs.org/api/stream.html#stream_class_stream_readable)이고, `response` 객체는 [`WritableStream`](https://nodejs.org/api/stream.html#stream_class_stream_writable)입니다. 즉, [`pipe`](https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options)를 사용해 데이터를 한 스트림에서 다른 스트림으로 직접 전달할 수 있습니다. 에코 서버에 딱 맞는 방법입니다!

```javascript
const http = require('node:http');

http
  .createServer((request, response) => {
    if (request.method === 'POST' && request.url === '/echo') {
      request.pipe(response);
    } else {
      response.statusCode = 404;
      response.end();
    }
  })
  .listen(8080);
```

스트림 만세!

하지만 아직 끝나지 않았습니다. 이 가이드에서 여러 번 언급했듯이, 오류는 발생할 수 있고 우리는 이를 처리해야 합니다.

요청 스트림에서 오류가 발생하면 `stderr`에 오류를 기록하고 `Bad Request`를 나타내는 400 상태 코드를 보냅니다. 실제 애플리케이션에서는 오류를 검사해 적절한 상태 코드와 메시지를 결정해야 합니다. 오류와 관련해서는 항상 [`Error` 문서](https://nodejs.org/api/errors.html)를 참조하세요.

응답에서는 오류를 `stderr`에 기록합니다.

```javascript
const http = require('node:http');

http
  .createServer((request, response) => {
    request.on('error', err => {
      console.error(err);
      response.statusCode = 400;
      response.end();
    });
    response.on('error', err => {
      console.error(err);
    });
    if (request.method === 'POST' && request.url === '/echo') {
      request.pipe(response);
    } else {
      response.statusCode = 404;
      response.end();
    }
  })
  .listen(8080);
```

이제 HTTP 요청 처리의 기본을 대부분 다뤘습니다. 이 시점에서 여러분은 다음을 할 수 있어야 합니다:

- 요청 핸들러 함수로 HTTP 서버를 인스턴스화하고 포트에서 대기시킬 수 있다.
- `request` 객체에서 헤더, URL, 메서드, 본문 데이터를 가져올 수 있다.
- URL 및 `request` 객체의 다른 데이터를 기반으로 라우팅 결정을 내릴 수 있다.
- `response` 객체를 통해 헤더, HTTP 상태 코드, 본문 데이터를 보낼 수 있다.
- `request` 객체에서 `response` 객체로 데이터를 파이프할 수 있다.
- `request` 및 `response` 스트림에서 스트림 오류를 처리할 수 있다.

이 기본 사항을 바탕으로 많은 일반적인 사용 사례에 대한 Node.js HTTP 서버를 구축할 수 있습니다. 이 API들이 제공하는 다른 기능들도 많으니, [`EventEmitters`](https://nodejs.org/api/events.html), [`Streams`](https://nodejs.org/api/stream.html), [`HTTP`](https://nodejs.org/api/http.html) 문서를 꼭 읽어보세요.


