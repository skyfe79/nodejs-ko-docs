# Table of Contents

- [[Don't Block the Event Loop (or the Worker Pool)](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#dont-block-the-event-loop-or-the-worker-pool)](#dont-block-the-event-loop-or-the-worker-poolhttpsnodejsorgenlearnasynchronous-workasynchronous-flow-controldont-block-the-event-loop-or-the-worker-pool)
  - [이 가이드를 읽어야 할까요?](#이-가이드를-읽어야-할까요)
  - [[요약](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#summary)](#요약httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlsummary)
  - [Event Loop과 Worker Pool을 블로킹하지 말아야 하는 이유](#event-loop과-worker-pool을-블로킹하지-말아야-하는-이유)
  - [[Node.js 간단 리뷰](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#a-quick-review-of-node)](#nodejs-간단-리뷰httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controla-quick-review-of-node)
    - [이벤트 루프에서 실행되는 코드는 무엇인가?](#이벤트-루프에서-실행되는-코드는-무엇인가)
    - [[Worker Pool에서 실행되는 코드는 무엇인가요?](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#what-code-runs-on-the-worker-pool)](#worker-pool에서-실행되는-코드는-무엇인가요httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlwhat-code-runs-on-the-worker-pool)
    - [Node.js는 다음에 실행할 코드를 어떻게 결정할까?](#nodejs는-다음에-실행할-코드를-어떻게-결정할까)
    - [애플리케이션 설계에 어떤 영향을 미칠까?](#애플리케이션-설계에-어떤-영향을-미칠까)
  - [이벤트 루프를 블로킹하지 마세요](#이벤트-루프를-블로킹하지-마세요)
    - [얼마나 조심해야 할까?](#얼마나-조심해야-할까)
    - [[이벤트 루프 블로킹: REDOS](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#blocking-the-event-loop-redos)](#이벤트-루프-블로킹-redoshttpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlblocking-the-event-loop-redos)
      - [[취약한 정규 표현식 피하기](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#avoiding-vulnerable-regular-expressions)](#취약한-정규-표현식-피하기httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlavoiding-vulnerable-regular-expressions)
      - [[REDOS 예제](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#a-redos-example)](#redos-예제httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controla-redos-example)
      - [[Anti-REDOS 리소스](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#anti-redos-resources)](#anti-redos-리소스httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlanti-redos-resources)
    - [[이벤트 루프 블로킹: Node.js 코어 모듈](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#blocking-the-event-loop-nodejs-core-modules)](#이벤트-루프-블로킹-nodejs-코어-모듈httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlblocking-the-event-loop-nodejs-core-modules)
    - [[이벤트 루프 블로킹: JSON DOS](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#blocking-the-event-loop-json-dos)](#이벤트-루프-블로킹-json-doshttpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlblocking-the-event-loop-json-dos)
    - [이벤트 루프를 블로킹하지 않고 복잡한 계산 수행하기](#이벤트-루프를-블로킹하지-않고-복잡한-계산-수행하기)
      - [파티셔닝](#파티셔닝)
      - [[오프로딩](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#offloading)](#오프로딩httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controloffloading)
        - [[작업 오프로딩 방법](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#how-to-offload)](#작업-오프로딩-방법httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlhow-to-offload)
        - [오프로딩의 단점](#오프로딩의-단점)
        - [[작업 분산을 위한 몇 가지 제안](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#some-suggestions-for-offloading)](#작업-분산을-위한-몇-가지-제안httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlsome-suggestions-for-offloading)
      - [[오프로딩: 결론](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#offloading-conclusions)](#오프로딩-결론httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controloffloading-conclusions)
  - [Worker Pool을 블로킹하지 마세요](#worker-pool을-블로킹하지-마세요)
    - [[작업 시간의 변동 최소화](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#minimizing-the-variation-in-task-times)](#작업-시간의-변동-최소화httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlminimizing-the-variation-in-task-times)
      - [[변형 예제: 오래 걸리는 파일 시스템 읽기](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#variation-example-long-running-file-system-reads)](#변형-예제-오래-걸리는-파일-시스템-읽기httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlvariation-example-long-running-file-system-reads)
      - [[변형 예제: 오래 걸리는 암호화 작업](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#variation-example-long-running-crypto-operations)](#변형-예제-오래-걸리는-암호화-작업httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlvariation-example-long-running-crypto-operations)
    - [[작업 분할](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#task-partitioning)](#작업-분할httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controltask-partitioning)
    - [[태스크 분할 피하기](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#avoiding-task-partitioning)](#태스크-분할-피하기httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlavoiding-task-partitioning)
    - [[Worker Pool: 결론](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#worker-pool-conclusions)](#worker-pool-결론httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlworker-pool-conclusions)
  - [npm 모듈의 위험성](#npm-모듈의-위험성)
  - [결론](#결론)

# [Don't Block the Event Loop (or the Worker Pool)](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#dont-block-the-event-loop-or-the-worker-pool)





## 이 가이드를 읽어야 할까요?

간단한 커맨드라인 스크립트보다 복잡한 프로그램을 작성한다면, 이 가이드를 통해 더 높은 성능과 보안을 갖춘 애플리케이션을 개발하는 데 도움을 얻을 수 있습니다.

이 문서는 Node.js 서버를 염두에 두고 작성되었지만, 복잡한 Node.js 애플리케이션에도 적용할 수 있는 개념들을 다룹니다. 운영체제별로 차이가 있는 부분은 리눅스를 중심으로 설명합니다.


## [요약](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#summary)

Node.js는 이벤트 루프(초기화 및 콜백)에서 자바스크립트 코드를 실행하며, 파일 I/O와 같은 무거운 작업을 처리하기 위해 워커 풀을 제공합니다. Node.js는 확장성이 뛰어나며, 때로는 Apache와 같은 더 무거운 접근 방식보다 더 나은 성능을 보입니다. Node.js의 확장성 비결은 적은 수의 스레드로 많은 클라이언트를 처리한다는 점입니다. Node.js가 더 적은 스레드로도 충분하다면, 시스템의 시간과 메모리를 스레드의 오버헤드(메모리, 컨텍스트 스위칭)에 소모하는 대신 클라이언트 작업에 더 많이 투자할 수 있습니다. 하지만 Node.js는 스레드가 적기 때문에, 여러분은 애플리케이션을 스레드를 효율적으로 사용하도록 구조화해야 합니다.

Node.js 서버를 빠르게 유지하기 위한 좋은 규칙은 다음과 같습니다: *Node.js는 주어진 시간에 각 클라이언트와 관련된 작업이 "작을 때" 빠릅니다.*

이 규칙은 이벤트 루프의 콜백과 워커 풀의 작업 모두에 적용됩니다.


## Event Loop과 Worker Pool을 블로킹하지 말아야 하는 이유

Node.js는 적은 수의 스레드를 사용해 많은 클라이언트를 처리합니다. Node.js에는 두 가지 종류의 스레드가 있습니다. 하나는 Event Loop(메인 루프, 메인 스레드, 이벤트 스레드 등으로도 불림)이고, 다른 하나는 Worker Pool(스레드풀)에 있는 `k`개의 Worker입니다.

스레드가 콜백(Event Loop)이나 작업(Worker)을 실행하는 데 오랜 시간이 걸리면 이를 "블로킹"되었다고 합니다. 스레드가 한 클라이언트를 위해 블로킹된 상태에서는 다른 클라이언트의 요청을 처리할 수 없습니다. 이 때문에 Event Loop와 Worker Pool을 블로킹하지 말아야 하는 두 가지 이유가 있습니다:

1. **성능**: 두 종류의 스레드 중 하나에서 무거운 작업을 정기적으로 수행하면 서버의 *처리량*(초당 요청 수)이 저하됩니다.
2. **보안**: 특정 입력에 대해 스레드가 블로킹될 가능성이 있다면, 악의적인 클라이언트가 이 "악성 입력"을 제출해 스레드를 블로킹시켜 다른 클라이언트를 처리하지 못하게 할 수 있습니다. 이는 [서비스 거부 공격](https://en.wikipedia.org/wiki/Denial-of-service_attack)이 됩니다.


## [Node.js 간단 리뷰](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#a-quick-review-of-node)

Node.js는 이벤트 기반 아키텍처를 사용합니다. 이벤트 루프가 오케스트레이션을 담당하고, 무거운 작업은 워커 풀에서 처리됩니다.


### 이벤트 루프에서 실행되는 코드는 무엇인가?

Node.js 애플리케이션은 시작할 때 초기화 단계를 먼저 완료합니다. 이 단계에서는 모듈을 `require`하고 이벤트에 대한 콜백을 등록합니다. 그런 다음 Node.js 애플리케이션은 이벤트 루프에 진입하여 들어오는 클라이언트 요청에 적절한 콜백을 실행하며 응답합니다. 이 콜백은 동기적으로 실행되며, 완료된 후 추가 처리를 위해 비동기 요청을 등록할 수 있습니다. 이러한 비동기 요청에 대한 콜백도 이벤트 루프에서 실행됩니다.

이벤트 루프는 콜백이 만든 논블로킹 비동기 요청(예: 네트워크 I/O)을 처리하는 역할도 담당합니다.

요약하면, 이벤트 루프는 이벤트에 등록된 자바스크립트 콜백을 실행하고, 네트워크 I/O와 같은 논블로킹 비동기 요청을 처리합니다.


### [Worker Pool에서 실행되는 코드는 무엇인가요?](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#what-code-runs-on-the-worker-pool)

Node.js의 Worker Pool은 libuv([문서](http://docs.libuv.org/en/v1.x/threadpool.html))로 구현되어 있으며, 일반적인 작업 제출 API를 제공합니다.

Node.js는 Worker Pool을 사용해 "비용이 많이 드는" 작업을 처리합니다. 여기에는 운영체제가 논블로킹 버전을 제공하지 않는 I/O 작업과 특히 CPU 집약적인 작업이 포함됩니다.

다음은 이 Worker Pool을 사용하는 Node.js 모듈 API들입니다:

1. **I/O 집약적 작업**
    1. [DNS](https://nodejs.org/api/dns.html): `dns.lookup()`, `dns.lookupService()`
    2. [파일 시스템](https://nodejs.org/api/fs.html#fs_threadpool_usage): `fs.FSWatcher()`와 명시적으로 동기화된 API를 제외한 모든 파일 시스템 API는 libuv의 스레드 풀을 사용합니다.
2. **CPU 집약적 작업**
    1. [Crypto](https://nodejs.org/api/crypto.html): `crypto.pbkdf2()`, `crypto.scrypt()`, `crypto.randomBytes()`, `crypto.randomFill()`, `crypto.generateKeyPair()`
    2. [Zlib](https://nodejs.org/api/zlib.html#zlib_threadpool_usage): 명시적으로 동기화된 API를 제외한 모든 zlib API는 libuv의 스레드 풀을 사용합니다.

많은 Node.js 애플리케이션에서, 이러한 API들이 Worker Pool에 작업을 제공하는 유일한 소스입니다. [C++ 애드온](https://nodejs.org/api/addons.html)을 사용하는 애플리케이션과 모듈은 Worker Pool에 다른 작업을 제출할 수 있습니다.

완전성을 위해, 이벤트 루프의 콜백에서 이러한 API 중 하나를 호출할 때, 이벤트 루프는 해당 API를 위한 Node.js C++ 바인딩에 진입하고 Worker Pool에 작업을 제출하는 데 약간의 설정 비용을 지불합니다. 이러한 비용은 작업의 전체 비용에 비해 미미하기 때문에, 이벤트 루프가 이를 오프로드하는 이유입니다. Worker Pool에 이러한 작업을 제출할 때, Node.js는 Node.js C++ 바인딩에서 해당 C++ 함수에 대한 포인터를 제공합니다.


### Node.js는 다음에 실행할 코드를 어떻게 결정할까?

추상적으로 보면, **이벤트 루프(Event Loop)**와 **워커 풀(Worker Pool)**은 각각 대기 중인 이벤트와 작업을 위한 큐를 유지합니다.

하지만 실제로 이벤트 루프는 큐를 직접 관리하지 않습니다. 대신, 이벤트 루프는 운영체제가 모니터링할 파일 디스크립터(file descriptor)의 모음을 가지고 있습니다. 이때 [epoll](http://man7.org/linux/man-pages/man7/epoll.7.html) (Linux), [kqueue](https://developer.apple.com/library/content/documentation/Darwin/Conceptual/FSEvents_ProgGuide/KernelQueues/KernelQueues.html) (OSX), event ports (Solaris), [IOCP](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365198.aspx) (Windows)와 같은 메커니즘을 사용합니다. 이 파일 디스크립터는 네트워크 소켓이나 감시 중인 파일 등에 해당합니다. 운영체제가 특정 파일 디스크립터가 준비되었다고 알려주면, 이벤트 루프는 이를 적절한 이벤트로 변환하고 해당 이벤트와 연결된 콜백 함수를 호출합니다. 이 과정에 대해 더 자세히 알고 싶다면 [여기](https://www.youtube.com/watch?v=P9csgxBgaZ8)를 참고하세요.

반면, 워커 풀은 실제 큐를 사용하며, 이 큐에는 처리할 작업들이 담겨 있습니다. 워커는 이 큐에서 작업을 꺼내 처리하고, 작업이 완료되면 "적어도 하나의 작업이 완료됨" 이벤트를 이벤트 루프에 전달합니다.


### 애플리케이션 설계에 어떤 영향을 미칠까?

Apache와 같은 클라이언트당 하나의 스레드를 사용하는 시스템에서는 각 대기 중인 클라이언트에 고유한 스레드가 할당됩니다. 만약 하나의 클라이언트를 처리하는 스레드가 블로킹되면, 운영체제가 이를 중단하고 다른 클라이언트에게 차례를 넘깁니다. 이렇게 하면 적은 작업이 필요한 클라이언트가 많은 작업이 필요한 클라이언트 때문에 불이익을 받지 않도록 보장됩니다.

반면 Node.js는 적은 수의 스레드로 많은 클라이언트를 처리합니다. 만약 하나의 스레드가 클라이언트 요청을 처리하다가 블로킹되면, 대기 중인 다른 클라이언트 요청은 해당 스레드가 콜백이나 작업을 마칠 때까지 차례를 받지 못할 수 있습니다. **따라서 클라이언트의 공정한 처리는 여러분의 애플리케이션 책임**이 됩니다. 이는 단일 콜백이나 작업에서 너무 많은 작업을 처리하지 않도록 해야 함을 의미합니다.

이것이 Node.js가 확장성을 잘 갖출 수 있는 이유 중 하나이지만, 동시에 공정한 스케줄링을 보장하는 것은 여러분의 책임이기도 합니다. 다음 섹션에서는 이벤트 루프와 워커 풀에서 공정한 스케줄링을 보장하는 방법에 대해 설명합니다.


## 이벤트 루프를 블로킹하지 마세요

이벤트 루프는 새로운 클라이언트 연결을 감지하고 응답 생성을 조율합니다. 모든 들어오는 요청과 나가는 응답은 이벤트 루프를 통과합니다. 이는 이벤트 루프가 어떤 지점에서 너무 오래 머무르면, 현재와 새로운 클라이언트 모두가 차례를 얻지 못한다는 것을 의미합니다.

여러분은 이벤트 루프를 절대 블로킹하지 않도록 해야 합니다. 다시 말해, 모든 자바스크립트 콜백은 빠르게 완료되어야 합니다. 이는 물론 `await`, `Promise.then` 등에도 적용됩니다.

이를 보장하는 좋은 방법은 콜백의 ["계산 복잡도"](https://en.wikipedia.org/wiki/Time_complexity)를 고려하는 것입니다. 콜백이 인자에 상관없이 일정한 수의 단계를 거친다면, 모든 대기 중인 클라이언트에게 공평한 차례를 줄 수 있습니다. 콜백이 인자에 따라 다른 수의 단계를 거친다면, 인자가 얼마나 길어질 수 있는지 생각해봐야 합니다.

예제 1: 상수 시간(constant-time) 콜백

```javascript
app.get('/constant-time', (req, res) => {
  res.sendStatus(200);
});
```

예제 2: `O(n)` 콜백. 이 콜백은 작은 `n`에 대해서는 빠르게 실행되지만, 큰 `n`에 대해서는 더 느리게 실행됩니다.

```javascript
app.get('/countToN', (req, res) => {
  let n = req.query.n;

  // 다른 클라이언트에게 차례를 주기 전에 n번 반복
  for (let i = 0; i < n; i++) {
    console.log(`Iter ${i}`);
  }

  res.sendStatus(200);
});
```

예제 3: `O(n^2)` 콜백. 이 콜백은 작은 `n`에 대해서는 여전히 빠르게 실행되지만, 큰 `n`에 대해서는 이전의 `O(n)` 예제보다 훨씬 더 느리게 실행됩니다.

```javascript
app.get('/countToN2', (req, res) => {
  let n = req.query.n;

  // 다른 클라이언트에게 차례를 주기 전에 n^2번 반복
  for (let i = 0; i < n; i++) {
    for (let j = 0; j < n; j++) {
      console.log(`Iter ${i}.${j}`);
    }
  }

  res.sendStatus(200);
});
```


### 얼마나 조심해야 할까?

Node.js는 자바스크립트를 위해 Google V8 엔진을 사용합니다. 이 엔진은 많은 일반적인 연산에서 매우 빠릅니다. 하지만 정규식(regexps)과 JSON 연산은 예외적으로 느릴 수 있습니다. 이에 대해서는 아래에서 더 자세히 다룹니다.

복잡한 작업을 처리할 때는 입력값을 제한하고 너무 긴 입력은 거부하는 것을 고려해야 합니다. 이렇게 하면 콜백이 아무리 복잡하더라도, 입력값을 제한함으로써 콜백이 최악의 경우에도 가장 긴 허용 가능한 입력에 대한 시간만큼만 소요되도록 보장할 수 있습니다. 그런 다음 이 콜백의 최악의 경우 비용을 평가하고, 해당 실행 시간이 여러분의 상황에서 허용 가능한지 판단할 수 있습니다.


### [이벤트 루프 블로킹: REDOS](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#blocking-the-event-loop-redos)

이벤트 루프를 심각하게 블로킹하는 일반적인 방법 중 하나는 "취약한" [정규 표현식](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)을 사용하는 것입니다.


#### [취약한 정규 표현식 피하기](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#avoiding-vulnerable-regular-expressions)

정규 표현식(regexp)은 입력 문자열을 패턴과 비교하여 매칭합니다. 일반적으로 정규 표현식 매칭은 입력 문자열을 한 번만 훑어보는 것으로 생각합니다. 즉, 입력 문자열의 길이 `n`에 대해 `O(n)` 시간이 걸린다고 가정합니다. 많은 경우 실제로 한 번 훑어보는 것만으로 충분합니다. 하지만 불행히도, 어떤 경우에는 정규 표현식 매칭이 입력 문자열을 지수적으로 많은 횟수만큼 훑어야 할 수도 있습니다. 이 경우 `O(2^n)` 시간이 소요됩니다. 지수적 횟수란, 엔진이 매칭을 결정하는 데 `x`번의 훑기가 필요하다면, 입력 문자열에 단 하나의 문자만 추가해도 `2*x`번의 훑기가 필요하다는 의미입니다. 훑는 횟수는 소요 시간과 선형적으로 연결되므로, 이러한 평가는 이벤트 루프를 블로킹하는 결과를 초래할 수 있습니다.

*취약한 정규 표현식*이란, 정규 표현식 엔진이 지수적 시간을 소요할 가능성이 있어 "악의적인 입력"에 대해 [REDOS](https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS)에 노출될 수 있는 정규 표현식을 말합니다. 여러분의 정규 표현식 패턴이 취약한지(즉, 정규 표현식 엔진이 지수적 시간을 소요할 가능성이 있는지) 여부는 실제로 답하기 어려운 질문이며, Perl, Python, Ruby, Java, JavaScript 등 사용하는 언어에 따라 다릅니다. 하지만 모든 언어에 공통적으로 적용할 수 있는 몇 가지 기본 규칙이 있습니다:

1. `(a+)*`와 같은 중첩된 수량자를 피하세요. V8의 정규 표현식 엔진은 일부를 빠르게 처리할 수 있지만, 다른 경우에는 취약할 수 있습니다.
2. `(a|a)*`와 같이 겹치는 절을 가진 OR 연산을 피하세요. 이 역시 때로는 빠르게 처리될 수 있습니다.
3. `(a.*) \1`과 같은 역참조 사용을 피하세요. 어떤 정규 표현식 엔진도 이를 선형 시간 내에 평가한다고 보장할 수 없습니다.
4. 단순한 문자열 매칭을 수행한다면, `indexOf`나 해당 언어의 동등한 기능을 사용하세요. 이는 더 저렴하고 `O(n)` 시간을 초과하지 않습니다.

여러분의 정규 표현식이 취약한지 확실하지 않다면, Node.js는 일반적으로 취약한 정규 표현식과 긴 입력 문자열에 대해서도 *매칭*을 보고하는 데 문제가 없다는 점을 기억하세요. 지수적 행동은 불일치가 발생했지만 Node.js가 입력 문자열을 통해 많은 경로를 시도해보기 전까지는 확신할 수 없는 경우에 트리거됩니다.


#### [REDOS 예제](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#a-redos-example)

다음은 서버를 REDOS에 노출시키는 취약한 정규 표현식 예제입니다:

```javascript
app.get('/redos-me', (req, res) => {
  let filePath = req.query.filePath;

  // REDOS
  if (filePath.match(/(\/.+)+$/)) {
    console.log('유효한 경로');
  } else {
    console.log('유효하지 않은 경로');
  }

  res.sendStatus(200);
});
```

이 예제의 취약한 정규 표현식은 Linux에서 유효한 경로를 확인하는 (나쁜!) 방법입니다. 이 정규식은 "/"로 구분된 이름 시퀀스(예: "/a/b/c")와 일치합니다. 이 정규식은 **첫 번째 규칙을 위반**하기 때문에 위험합니다. 즉, 중첩된 수량자를 가지고 있습니다.

클라이언트가 `///.../\n`(100개의 "/" 뒤에 정규식의 "."과 일치하지 않는 개행 문자)로 쿼리하면, 이벤트 루프가 사실상 영원히 걸리며 이벤트 루프를 블로킹합니다. 이 클라이언트의 REDOS 공격으로 인해 다른 모든 클라이언트는 정규식 매칭이 끝날 때까지 차례를 기다려야 합니다.

이러한 이유로, 사용자 입력을 검증하기 위해 복잡한 정규 표현식을 사용하는 것에 주의해야 합니다.


#### [Anti-REDOS 리소스](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#anti-redos-resources)

정규 표현식의 안전성을 확인할 수 있는 몇 가지 도구가 있습니다. 예를 들어:

-   [safe-regex](https://github.com/davisjam/safe-regex)
-   [rxxr2](https://github.com/superhuman/rxxr2)

하지만 이 도구들도 모든 취약한 정규 표현식을 잡아내지는 못합니다.

다른 접근 방식으로는 다른 정규 표현식 엔진을 사용하는 방법이 있습니다. Google의 빠른 [RE2](https://github.com/google/re2) 정규 표현식 엔진을 사용하는 [node-re2](https://github.com/uhop/node-re2) 모듈을 사용할 수 있습니다. 하지만 RE2는 V8의 정규 표현식과 100% 호환되지 않으므로, node-re2 모듈로 정규 표현식을 처리할 경우 회귀 테스트를 꼭 진행해야 합니다. 또한, 특히 복잡한 정규 표현식은 node-re2에서 지원되지 않습니다.

URL이나 파일 경로와 같이 "명확한" 것을 매칭하려는 경우, [regexp 라이브러리](http://www.regexlib.com)에서 예제를 찾거나 [ip-regex](https://www.npmjs.com/package/ip-regex)와 같은 npm 모듈을 사용하는 것이 좋습니다.


### [이벤트 루프 블로킹: Node.js 코어 모듈](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#blocking-the-event-loop-nodejs-core-modules)

Node.js의 여러 코어 모듈은 동기적으로 실행되는 비용이 큰 API를 제공합니다. 이들 API는 다음과 같습니다:

-   [암호화](https://nodejs.org/api/crypto.html)
-   [압축](https://nodejs.org/api/zlib.html)
-   [파일 시스템](https://nodejs.org/api/fs.html)
-   [자식 프로세스](https://nodejs.org/api/child_process.html)

이 API들은 상당한 계산(암호화, 압축)을 필요로 하거나, I/O(파일 입출력)를 요구하거나, 둘 다(자식 프로세스)를 포함하기 때문에 비용이 큽니다. 이 API들은 스크립팅 편의를 위해 제공되지만, 서버 환경에서 사용하기에는 적합하지 않습니다. 만약 이 API들을 이벤트 루프에서 실행하면, 일반적인 자바스크립트 명령어보다 훨씬 오래 걸려 이벤트 루프를 블로킹하게 됩니다.

서버에서는 *다음과 같은 동기 API를 사용하지 않아야 합니다*:

-   암호화:
    -   `crypto.randomBytes` (동기 버전)
    -   `crypto.randomFillSync`
    -   `crypto.pbkdf2Sync`
    -   또한 암호화 및 복호화 루틴에 큰 입력을 제공할 때 주의해야 합니다.
-   압축:
    -   `zlib.inflateSync`
    -   `zlib.deflateSync`
-   파일 시스템:
    -   동기 파일 시스템 API를 사용하지 마세요. 예를 들어, 접근하는 파일이 [NFS](https://en.wikipedia.org/wiki/Network_File_System)와 같은 [분산 파일 시스템](https://en.wikipedia.org/wiki/Clustered_file_system#Distributed_file_systems)에 있는 경우, 접근 시간이 크게 달라질 수 있습니다.
-   자식 프로세스:
    -   `child_process.spawnSync`
    -   `child_process.execSync`
    -   `child_process.execFileSync`

이 목록은 Node.js v9 기준으로 상당히 완전합니다.


### [이벤트 루프 블로킹: JSON DOS](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#blocking-the-event-loop-json-dos)

`JSON.parse`와 `JSON.stringify`는 잠재적으로 비용이 많이 드는 작업입니다. 이 작업들은 입력 길이에 대해 `O(n)`의 복잡도를 가지지만, 큰 `n` 값에 대해선 예상보다 오랜 시간이 걸릴 수 있습니다.

서버가 JSON 객체를 다루는 경우, 특히 클라이언트로부터 받은 JSON 객체를 처리할 때, 이벤트 루프에서 다루는 객체나 문자열의 크기에 주의해야 합니다.

예제: JSON 블로킹. 크기가 2^21인 객체 `obj`를 생성하고 `JSON.stringify`로 문자열로 변환한 후, 문자열에서 `indexOf`를 실행하고 다시 `JSON.parse`로 파싱합니다. `JSON.stringify`로 변환된 문자열은 50MB입니다. 객체를 문자열로 변환하는 데 0.7초가 걸리고, 50MB 문자열에서 `indexOf`를 실행하는 데 0.03초가 걸리며, 문자열을 파싱하는 데 1.3초가 소요됩니다.

```javascript
let obj = { a: 1 };
let niter = 20;

let before, str, pos, res, took;

for (let i = 0; i < niter; i++) {
  obj = { obj1: obj, obj2: obj }; // 매 반복마다 크기가 두 배로 증가
}

before = process.hrtime();
str = JSON.stringify(obj);
took = process.hrtime(before);
console.log('JSON.stringify took ' + took);

before = process.hrtime();
pos = str.indexOf('nomatch');
took = process.hrtime(before);
console.log('Pure indexof took ' + took);

before = process.hrtime();
res = JSON.parse(str);
took = process.hrtime(before);
console.log('JSON.parse took ' + took);
```

비동기 JSON API를 제공하는 npm 모듈도 있습니다. 예를 들어:

-   [JSONStream](https://www.npmjs.com/package/JSONStream): 스트림 API를 제공합니다.
-   [Big-Friendly JSON](https://www.npmjs.com/package/bfj): 스트림 API와 함께, 이벤트 루프에서 파티셔닝을 사용한 비동기 버전의 표준 JSON API를 제공합니다.


### 이벤트 루프를 블로킹하지 않고 복잡한 계산 수행하기

JavaScript에서 이벤트 루프를 블로킹하지 않고 복잡한 계산을 수행하려면 두 가지 방법을 고려할 수 있습니다: **분할(partitioning)** 또는 **오프로딩(offloading)**입니다.


#### 파티셔닝

여러분은 계산 작업을 *파티셔닝*하여 각 작업이 이벤트 루프에서 실행되지만 정기적으로 다른 대기 중인 이벤트에 차례를 양보하도록 할 수 있습니다. 자바스크립트에서는 클로저를 사용하여 진행 중인 작업의 상태를 쉽게 저장할 수 있습니다. 아래 예제 2에서 이를 확인할 수 있습니다.

간단한 예로, `1`부터 `n`까지의 숫자의 평균을 계산한다고 가정해 보겠습니다.

**예제 1: 파티셔닝되지 않은 평균 계산, `O(n)`의 비용 발생**

```javascript
for (let i = 0; i < n; i++) sum += i;
let avg = sum / n;
console.log('avg: ' + avg);
```

**예제 2: 파티셔닝된 평균 계산, 각 `n`개의 비동기 단계는 `O(1)`의 비용 발생**

```javascript
function asyncAvg(n, avgCB) {
  // 진행 중인 합계를 자바스크립트 클로저에 저장
  let sum = 0;
  function help(i, cb) {
    sum += i;
    if (i == n) {
      cb(sum);
      return;
    }

    // "비동기 재귀"
    // 다음 작업을 비동기적으로 스케줄링
    setImmediate(help.bind(null, i + 1, cb));
  }

  // 헬퍼 함수 시작, avgCB를 호출할 콜백 전달
  help(1, function (sum) {
    let avg = sum / n;
    avgCB(avg);
  });
}

asyncAvg(n, function (avg) {
  console.log('avg of 1-n: ' + avg);
});
```

이 원칙을 배열 반복 등에 적용할 수 있습니다.


#### [오프로딩](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#offloading)

더 복잡한 작업을 해야 한다면, 파티셔닝은 좋은 선택이 아닙니다. 파티셔닝은 이벤트 루프만 사용하기 때문에, 여러분의 머신에서 거의 확실하게 사용 가능한 멀티 코어의 이점을 누릴 수 없기 때문입니다. *이벤트 루프는 클라이언트 요청을 조율해야 하며, 직접 요청을 처리해서는 안 됩니다.* 복잡한 작업의 경우, 이벤트 루프에서 작업을 워커 풀로 옮기는 것이 좋습니다.


##### [작업 오프로딩 방법](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#how-to-offload)

작업을 오프로딩할 대상 Worker Pool로 두 가지 옵션이 있습니다.

1. **Node.js 내장 Worker Pool 사용**: [C++ 애드온](https://nodejs.org/api/addons.html)을 개발하여 Node.js의 내장 Worker Pool을 활용할 수 있습니다. 이전 버전의 Node에서는 [NAN](https://github.com/nodejs/nan)을 사용해 C++ 애드온을 빌드하고, 최신 버전에서는 [N-API](https://nodejs.org/api/n-api.html)를 사용합니다. [node-webworker-threads](https://www.npmjs.com/package/webworker-threads)는 JavaScript만으로 Node.js Worker Pool에 접근할 수 있는 방법을 제공합니다.
2. **커스텀 Worker Pool 생성 및 관리**: Node.js의 I/O 중심 Worker Pool 대신 계산 작업에 전념하는 별도의 Worker Pool을 만들고 관리할 수 있습니다. 가장 간단한 방법은 [Child Process](https://nodejs.org/api/child_process.html)나 [Cluster](https://nodejs.org/api/cluster.html)를 사용하는 것입니다.

**주의사항**: 모든 클라이언트마다 [Child Process](https://nodejs.org/api/child_process.html)를 생성하는 것은 피해야 합니다. 클라이언트 요청을 받는 속도가 Child Process를 생성하고 관리하는 속도보다 빠를 수 있으며, 이 경우 서버가 [포크 폭탄](https://en.wikipedia.org/wiki/Fork_bomb)이 될 위험이 있습니다.


##### 오프로딩의 단점

오프로딩 방식의 단점은 *통신 비용*이라는 오버헤드가 발생한다는 점입니다. 이벤트 루프만이 여러분의 애플리케이션의 "네임스페이스"(JavaScript 상태)를 볼 수 있습니다. 워커에서는 이벤트 루프의 네임스페이스에 있는 JavaScript 객체를 직접 조작할 수 없습니다. 대신, 공유하려는 객체를 직렬화하고 역직렬화해야 합니다. 그런 다음 워커는 이 객체의 복사본을 가지고 작업을 수행한 후, 수정된 객체(또는 "패치")를 이벤트 루프로 반환합니다.

직렬화와 관련된 자세한 내용은 JSON DOS 섹션을 참고하세요.


##### [작업 분산을 위한 몇 가지 제안](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#some-suggestions-for-offloading)

CPU 집약적 작업과 I/O 집약적 작업은 서로 다른 특성을 가지고 있기 때문에 이를 구분하는 것이 좋습니다.

**CPU 집약적 작업**은 해당 Worker가 스케줄링될 때만 진행됩니다. Worker는 머신의 [논리 코어](https://nodejs.org/api/os.html#os_os_cpus) 중 하나에 스케줄링되어야 합니다. 예를 들어, 4개의 논리 코어와 5개의 Worker가 있다면, 이 중 하나의 Worker는 진행할 수 없습니다. 결과적으로, 해당 Worker에 대한 오버헤드(메모리 및 스케줄링 비용)를 지불하면서도 아무런 이득을 얻지 못하게 됩니다.

**I/O 집약적 작업**은 외부 서비스 프로바이더(DNS, 파일 시스템 등)에 쿼리를 보내고 응답을 기다리는 작업입니다. I/O 집약적 작업을 수행하는 Worker는 응답을 기다리는 동안 다른 작업을 할 수 없으므로, 운영체제에 의해 스케줄링에서 제외될 수 있습니다. 이렇게 되면 다른 Worker가 요청을 제출할 기회를 얻습니다. 따라서 *I/O 집약적 작업은 해당 스레드가 실행되지 않는 동안에도 진행될 수 있습니다*. 데이터베이스나 파일 시스템과 같은 외부 서비스 프로바이더는 많은 대기 중인 요청을 동시에 처리하도록 고도로 최적화되어 있습니다. 예를 들어, 파일 시스템은 대기 중인 쓰기 및 읽기 요청을 검토하여 충돌하는 업데이트를 병합하고 최적의 순서로 파일을 검색합니다.

만약 Node.js Worker Pool과 같은 단일 Worker Pool에만 의존한다면, CPU 바운드 작업과 I/O 바운드 작업의 서로 다른 특성으로 인해 애플리케이션의 성능이 저하될 수 있습니다.

이러한 이유로, 별도의 **계산용 Worker Pool**을 유지하는 것을 고려해볼 수 있습니다.


#### [오프로딩: 결론](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#offloading-conclusions)

간단한 작업, 예를 들어 임의의 길이를 가진 배열의 엘리먼트를 반복하는 경우, 분할(partitioning)이 좋은 선택일 수 있습니다. 계산이 더 복잡하다면 오프로딩이 더 나은 접근 방식입니다. 이벤트 루프와 워커 풀 간에 직렬화된 객체를 전달하는 오버헤드와 같은 통신 비용은 여러 코어를 사용하는 이점으로 상쇄됩니다.

그러나 서버가 복잡한 계산에 크게 의존한다면, Node.js가 정말 적합한지 고민해봐야 합니다. Node.js는 I/O 중심 작업에서 뛰어나지만, 고비용 계산에는 최적의 선택이 아닐 수 있습니다.

오프로딩 방식을 선택한다면, 워커 풀을 블로킹하지 않는 방법에 대한 섹션을 참고하세요.


## Worker Pool을 블로킹하지 마세요

Node.js는 `k`개의 Worker로 구성된 Worker Pool을 가지고 있습니다. 앞서 설명한 오프로딩 패러다임을 사용한다면, 별도의 Computational Worker Pool을 가질 수 있습니다. 이 경우에도 동일한 원칙이 적용됩니다. 두 경우 모두 `k`가 동시에 처리해야 하는 클라이언트 수보다 훨씬 작다고 가정해 봅시다. 이는 Node.js의 "하나의 스레드로 많은 클라이언트 처리" 철학과 확장성의 비결에 부합합니다.

앞서 언급했듯이, 각 Worker는 Worker Pool 큐에 있는 다음 작업으로 넘어가기 전에 현재 작업을 완료합니다.

클라이언트 요청을 처리하는 데 필요한 작업의 비용은 다양합니다. 어떤 작업은 빠르게 완료될 수 있습니다(예: 짧거나 캐시된 파일 읽기, 적은 수의 랜덤 바이트 생성). 반면 다른 작업은 더 오래 걸릴 수 있습니다(예: 크거나 캐시되지 않은 파일 읽기, 더 많은 랜덤 바이트 생성). 여러분의 목표는 **작업 시간의 변동을 최소화**하는 것이며, 이를 위해 **작업 분할**을 사용해야 합니다.


### [작업 시간의 변동 최소화](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#minimizing-the-variation-in-task-times)

만약 워커의 현재 작업이 다른 작업들보다 훨씬 더 비용이 많이 든다면, 해당 워커는 다른 대기 중인 작업을 처리할 수 없게 됩니다. 즉, *상대적으로 긴 작업은 완료될 때까지 워커 풀의 크기를 효과적으로 하나씩 줄입니다*. 이는 바람직하지 않은데, 일정 범위 내에서 워커 풀에 있는 워커가 많을수록 워커 풀의 처리량(초당 작업 수)이 증가하고, 결과적으로 서버의 처리량(초당 클라이언트 요청 수)도 증가하기 때문입니다. 상대적으로 비용이 많이 드는 작업을 요청한 하나의 클라이언트는 워커 풀의 처리량을 감소시키고, 이는 서버의 처리량도 감소시킵니다.

이를 방지하려면 워커 풀에 제출하는 작업의 길이 변동을 최소화해야 합니다. I/O 요청(DB, FS 등)에 접근하는 외부 시스템을 블랙박스로 취급하는 것은 적절하지만, 이러한 I/O 요청의 상대적 비용을 인지하고 특히 오래 걸릴 것으로 예상되는 요청을 제출하지 않도록 주의해야 합니다.

작업 시간의 가능한 변동을 설명하기 위해 두 가지 예제를 살펴보겠습니다.


#### [변형 예제: 오래 걸리는 파일 시스템 읽기](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#variation-example-long-running-file-system-reads)

여러분의 서버가 클라이언트 요청을 처리하기 위해 파일을 읽어야 한다고 가정해 보겠습니다. Node.js [파일 시스템](https://nodejs.org/api/fs.html) API를 확인한 후, 간단함을 위해 `fs.readFile()`을 사용하기로 결정했습니다. 하지만 `fs.readFile()`은 (현재 [이슈](https://github.com/nodejs/node/pull/17054)로 인해) 분할되지 않습니다. 이 함수는 전체 파일을 대상으로 하는 단일 `fs.read()` 작업을 제출합니다. 일부 사용자에게는 짧은 파일을, 다른 사용자에게는 긴 파일을 읽게 되면 `fs.readFile()`은 작업 길이에 상당한 차이를 만들어내며, 이는 Worker Pool의 처리량에 악영향을 미칠 수 있습니다.

최악의 시나리오를 가정해 보겠습니다. 공격자가 여러분의 서버를 속여 *임의의* 파일을 읽도록 할 수 있다고 합시다 (이는 [디렉토리 순회 취약점](https://www.owasp.org/index.php/Path_Traversal)입니다). 서버가 리눅스에서 실행 중이라면, 공격자는 매우 느린 파일인 [`/dev/random`](http://man7.org/linux/man-pages/man4/random.4.html)을 지정할 수 있습니다. 실질적으로 `/dev/random`은 무한히 느리며, 이 파일을 읽도록 요청받은 모든 Worker는 해당 작업을 절대 완료하지 못합니다. 공격자는 각 Worker마다 하나씩 `k`개의 요청을 제출하고, Worker Pool을 사용하는 다른 클라이언트 요청은 전혀 진행되지 않게 됩니다.


#### [변형 예제: 오래 걸리는 암호화 작업](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#variation-example-long-running-crypto-operations)

여러분의 서버가 [`crypto.randomBytes()`](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#variation-example-long-running-crypto-operations)를 사용해 암호학적으로 안전한 랜덤 바이트를 생성한다고 가정해 보겠습니다. `crypto.randomBytes()`는 분할되지 않습니다. 이 함수는 요청한 만큼의 바이트를 생성하기 위해 단일 `randomBytes()` 작업을 생성합니다. 일부 사용자에게는 적은 수의 바이트를 생성하고, 다른 사용자에게는 많은 바이트를 생성한다면, `crypto.randomBytes()`는 작업 길이의 변형을 일으키는 또 다른 원인이 됩니다.


### [작업 분할](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#task-partitioning)

작업 시간이 일정하지 않으면 Worker Pool의 처리량에 악영향을 미칠 수 있습니다. 작업 시간의 변동을 최소화하려면 가능한 한 각 작업을 비슷한 비용의 하위 작업으로 **분할**해야 합니다. 각 하위 작업이 완료되면 다음 하위 작업을 제출하고, 마지막 하위 작업이 완료되면 제출자에게 알려야 합니다.

`fs.readFile()` 예제를 계속해서 살펴보면, `fs.read()`(수동 분할) 또는 `ReadStream`(자동 분할)을 대신 사용하는 것이 좋습니다.

이 원칙은 CPU 집약적인 작업에도 동일하게 적용됩니다. `asyncAvg` 예제는 이벤트 루프에는 적합하지 않을 수 있지만, Worker Pool에는 잘 맞습니다.

작업을 하위 작업으로 분할하면 짧은 작업은 적은 수의 하위 작업으로 확장되고, 긴 작업은 더 많은 수의 하위 작업으로 확장됩니다. 긴 작업의 각 하위 작업 사이에 할당된 Worker는 다른 짧은 작업의 하위 작업을 처리할 수 있으므로, Worker Pool의 전체 작업 처리량이 향상됩니다.

하위 작업의 완료 횟수는 Worker Pool의 처리량을 측정하는 데 유용한 지표가 아닙니다. 대신 **작업**의 완료 횟수에 주목해야 합니다.


### [태스크 분할 피하기](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#avoiding-task-partitioning)

태스크 분할의 목적은 태스크 실행 시간의 편차를 최소화하는 것입니다. 짧은 태스크와 긴 태스크를 구분할 수 있다면(예: 배열 합산 vs 배열 정렬), 각 태스크 유형별로 별도의 워커 풀을 생성할 수 있습니다. 짧은 태스크와 긴 태스크를 서로 다른 워커 풀로 라우팅하면 태스크 시간 편차를 줄이는 또 다른 방법이 됩니다.

이 접근 방식의 장점은, 태스크 분할은 오버헤드를 발생시킨다는 점입니다(워커 풀 태스크 표현을 생성하고 워커 풀 큐를 조작하는 비용). 분할을 피하면 워커 풀에 추가로 접근하는 비용을 절약할 수 있습니다. 또한 태스크를 분할하면서 발생할 수 있는 실수를 방지할 수 있습니다.

하지만 이 방식의 단점은 모든 워커 풀의 워커들이 공간과 시간 오버헤드를 발생시키고 CPU 시간을 서로 경쟁한다는 점입니다. CPU 바운드 태스크는 스케줄링 중에만 진행된다는 것을 기억해야 합니다. 따라서 이 접근 방식을 고려하기 전에 신중하게 분석해야 합니다.


### [Worker Pool: 결론](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#worker-pool-conclusions)

Node.js Worker Pool만 사용하든, 별도의 Worker Pool을 유지하든, 여러분은 Pool의 Task 처리량을 최적화해야 합니다.

이를 위해 Task 분할을 사용하여 Task 시간의 변동을 최소화하세요.


## npm 모듈의 위험성

Node.js 코어 모듈은 다양한 애플리케이션을 위한 기본 구성 요소를 제공하지만, 때로는 더 많은 기능이 필요할 수 있습니다. Node.js 개발자들은 [npm 생태계](https://www.npmjs.com/)에서 큰 혜택을 받습니다. 수십만 개의 모듈이 개발 프로세스를 가속화하는 기능을 제공합니다.

하지만 이러한 모듈 대부분은 제3자 개발자들이 작성했으며, 일반적으로 최선의 노력만을 보장하며 출시된다는 점을 기억해야 합니다. npm 모듈을 사용하는 개발자는 두 가지 사항을 고려해야 합니다. 두 번째 사항은 종종 잊히곤 합니다.

1. API가 제대로 동작하는가?
2. API가 이벤트 루프나 워커를 블로킹할 가능성이 있는가? 많은 모듈이 API의 비용을 명시하지 않아 커뮤니티에 해를 끼칠 수 있습니다.

간단한 API의 경우 비용을 추정할 수 있습니다. 문자열 조작의 비용은 이해하기 어렵지 않습니다. 하지만 많은 경우 API의 비용이 얼마나 될지 명확하지 않습니다.

*비용이 많이 드는 작업을 수행할 가능성이 있는 API를 호출한다면, 비용을 다시 확인하세요. 개발자들에게 문서화를 요청하거나, 직접 소스 코드를 확인하고 비용을 문서화하는 PR을 제출하세요.*

API가 비동기적이라고 해도, 각 파티션에서 워커나 이벤트 루프에 소요되는 시간을 알 수 없다는 점을 기억하세요. 예를 들어, 위에서 언급한 `asyncAvg` 예제에서 헬퍼 함수가 각 호출마다 숫자 하나가 아니라 *절반*을 더한다고 가정해 보겠습니다. 이 함수는 여전히 비동기적이지만, 각 파티션의 비용은 `O(1)`이 아니라 `O(n)`이 되어, 임의의 `n` 값에 대해 사용하기 훨씬 덜 안전해집니다.


## 결론

Node.js는 두 가지 종류의 스레드를 가지고 있습니다: 하나는 이벤트 루프이고, 다른 하나는 `k`개의 워커입니다. 이벤트 루프는 자바스크립트 콜백과 논블로킹 I/O를 담당합니다. 워커는 비동기 요청을 완료하는 C++ 코드에 해당하는 작업을 실행하며, 여기에는 블로킹 I/O와 CPU 집약적인 작업이 포함됩니다. 두 종류의 스레드 모두 한 번에 하나의 작업만 처리합니다. 만약 콜백이나 작업이 오래 걸린다면, 해당 스레드는 *블로킹* 상태가 됩니다. 애플리케이션이 블로킹 콜백이나 작업을 수행하면, 최악의 경우 처리량(초당 클라이언트 수)이 저하되거나 서비스가 완전히 중단될 수 있습니다.

높은 처리량을 유지하고 DoS 공격에 강한 웹 서버를 만들려면, 정상적인 입력과 악의적인 입력 모두에서 이벤트 루프와 워커가 블로킹되지 않도록 해야 합니다.


