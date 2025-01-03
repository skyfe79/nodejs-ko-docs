# [The Node.js Event Loop](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#the-nodejs-event-loop)





## 이벤트 루프란 무엇인가?

이벤트 루프는 Node.js가 기본적으로 단일 자바스크립트 스레드를 사용함에도 불구하고, 가능한 경우 작업을 시스템 커널로 넘겨서 **논블로킹 I/O 작업**을 수행할 수 있게 해주는 메커니즘입니다.

대부분의 현대 커널은 멀티스레드로 동작하기 때문에, 여러 작업을 백그라운드에서 동시에 처리할 수 있습니다. 이러한 작업 중 하나가 완료되면, 커널은 Node.js에게 알립니다. 그러면 적절한 콜백이 **poll 큐**에 추가되어 결국 실행됩니다. 이에 대한 자세한 설명은 뒤에서 다루겠습니다.


## 이벤트 루프 설명

Node.js가 시작되면 이벤트 루프를 초기화하고, 제공된 입력 스크립트를 처리합니다. 이 스크립트는 비동기 API 호출을 하거나, 타이머를 예약하거나, `process.nextTick()`을 호출할 수 있습니다. 그런 다음 이벤트 루프 처리를 시작합니다.

다음 다이어그램은 이벤트 루프의 작업 순서를 간략히 보여줍니다.

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

> 각 박스는 이벤트 루프의 "단계"라고 부릅니다.

각 단계에는 실행할 콜백의 FIFO 큐가 있습니다. 각 단계는 고유한 특성을 가지고 있지만, 일반적으로 이벤트 루프가 특정 단계에 진입하면 해당 단계에 특화된 작업을 수행한 후, 큐가 소진되거나 최대 콜백 실행 횟수에 도달할 때까지 큐의 콜백을 실행합니다. 큐가 소진되거나 콜백 제한에 도달하면 이벤트 루프는 다음 단계로 이동합니다.

이러한 작업 중 어느 것이라도 *추가* 작업을 예약할 수 있고, **poll** 단계에서 처리되는 새로운 이벤트는 커널에 의해 큐에 추가됩니다. 따라서 폴링 이벤트가 처리되는 동안에도 폴 이벤트가 큐에 추가될 수 있습니다. 결과적으로, 오래 실행되는 콜백은 폴 단계가 타이머의 임계값보다 훨씬 더 오래 실행되도록 할 수 있습니다. 자세한 내용은 [**타이머**](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#timers)와 [**폴**](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#poll) 섹션을 참고하세요.

> 윈도우와 Unix/Linux 구현 간에 약간의 차이가 있지만, 이 설명에서는 중요하지 않습니다. 가장 중요한 부분은 여기에 있습니다. 실제로는 일곱 또는 여덟 단계가 있지만, Node.js에서 실제로 사용하는 단계는 위에 나온 것들입니다.


## [단계 개요](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#phases-overview)

- **타이머(timers)**: 이 단계에서는 `setTimeout()`과 `setInterval()`로 예약된 콜백을 실행합니다.
- **보류 중인 콜백(pending callbacks)**: 다음 루프 반복으로 연기된 I/O 콜백을 실행합니다.
- **대기(idle), 준비(prepare)**: 내부적으로만 사용됩니다.
- **폴링(poll)**: 새로운 I/O 이벤트를 가져오고, I/O 관련 콜백을 실행합니다. (닫기 콜백, 타이머로 예약된 콜백, `setImmediate()`는 제외) Node.js는 적절한 경우 여기서 블로킹됩니다.
- **체크(check)**: `setImmediate()` 콜백이 여기서 호출됩니다.
- **닫기 콜백(close callbacks)**: `socket.on('close', ...)`와 같은 일부 닫기 콜백이 실행됩니다.

이벤트 루프가 한 번 실행될 때마다, Node.js는 비동기 I/O나 타이머를 기다리고 있는지 확인합니다. 기다리는 작업이 없으면 깔끔하게 종료됩니다.


## [Phases in Detail](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#phases-in-detail)





### [타이머](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#timers)

타이머는 콜백이 실행될 **정확한 시간**이 아니라, 콜백이 **실행될 수 있는** **임계값**을 지정합니다. 타이머 콜백은 지정된 시간이 지난 후 가능한 한 빨리 실행되도록 스케줄링됩니다. 하지만 운영체제 스케줄링이나 다른 콜백의 실행으로 인해 지연될 수 있습니다.

> 기술적으로, [**poll** 단계](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#poll)가 타이머가 실행되는 시점을 제어합니다.

예를 들어, 100ms 임계값 이후에 실행되도록 타임아웃을 설정한 후, 95ms가 걸리는 파일 읽기 작업을 비동기적으로 시작한다고 가정해 봅시다:

```javascript
const fs = require('node:fs');

function someAsyncOperation(callback) {
  // 이 작업이 95ms 걸린다고 가정
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);

// 95ms가 걸리는 someAsyncOperation 실행
someAsyncOperation(() => {
  const startCallback = Date.now();

  // 10ms가 걸리는 작업 실행
  while (Date.now() - startCallback < 10) {
    // 아무 작업도 하지 않음
  }
});
```

이벤트 루프가 **poll** 단계에 진입할 때, 큐는 비어 있습니다 (`fs.readFile()`이 아직 완료되지 않음). 따라서 가장 빠른 타이머의 임계값에 도달할 때까지 남은 시간만큼 대기합니다. 95ms가 지나는 동안 `fs.readFile()`이 파일 읽기를 마치고, 10ms가 걸리는 콜백이 **poll** 큐에 추가되어 실행됩니다. 콜백이 완료되면 큐에 더 이상 콜백이 없으므로, 이벤트 루프는 가장 빠른 타이머의 임계값에 도달했음을 확인하고 **timers** 단계로 돌아가 타이머의 콜백을 실행합니다. 이 예제에서는 타이머가 스케줄링된 시점부터 콜백이 실행될 때까지 총 105ms의 지연이 발생합니다.

> **poll** 단계가 이벤트 루프를 방해하지 않도록, [libuv](https://libuv.org/) (Node.js의 이벤트 루프와 모든 비동기 동작을 구현한 C 라이브러리)는 더 많은 이벤트를 폴링하기 전에 시스템에 따라 정해진 최대 시간을 설정합니다.


### [pending callbacks](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#pending-callbacks)

이 단계에서는 TCP 오류와 같은 시스템 작업에 대한 콜백을 실행합니다. 예를 들어, TCP 소켓이 연결을 시도할 때 `ECONNREFUSED`를 받는 경우, 일부 \*nix 시스템은 오류를 보고하기 위해 대기합니다. 이 작업은 **pending callbacks** 단계에서 실행되도록 큐에 추가됩니다.


### [poll](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#poll)

**poll** 단계는 두 가지 주요 기능을 수행합니다:

1. I/O를 위해 얼마나 오래 블로킹하고 폴링할지 계산한 후,
2. **poll** 큐에 있는 이벤트를 처리합니다.

이벤트 루프가 **poll** 단계에 진입했을 때 *예약된 타이머가 없는 경우*, 다음 두 가지 중 하나가 발생합니다:

- ***poll** 큐가 비어 있지 않다면*, 이벤트 루프는 콜백 큐를 순회하며 동기적으로 실행합니다. 큐가 모두 소진되거나 시스템에 의존적인 하드 제한에 도달할 때까지 계속됩니다.
    
- ***poll** 큐가 비어 있다면*, 다음 두 가지 중 하나가 발생합니다:
    
    - `setImmediate()`로 스크립트가 예약된 경우, 이벤트 루프는 **poll** 단계를 종료하고 **check** 단계로 이동하여 예약된 스크립트를 실행합니다.
        
    - `setImmediate()`로 스크립트가 **예약되지 않은 경우**, 이벤트 루프는 큐에 콜백이 추가될 때까지 기다린 후 즉시 실행합니다.

**poll** 큐가 비어 있으면, 이벤트 루프는 *시간 임계값에 도달한 타이머*를 확인합니다. 하나 이상의 타이머가 준비된 경우, 이벤트 루프는 **timers** 단계로 돌아가 해당 타이머의 콜백을 실행합니다.


### [check](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#check)

이 단계는 **poll** 단계가 완료된 직후 콜백을 실행할 수 있게 해줍니다. **poll** 단계가 유휴 상태가 되고 `setImmediate()`를 통해 스크립트가 큐에 추가되었다면, 이벤트 루프는 **poll** 단계에서 대기하지 않고 **check** 단계로 넘어갈 수 있습니다.

`setImmediate()`는 사실 이벤트 루프의 별도 단계에서 실행되는 특별한 타이머입니다. 이는 **poll** 단계가 완료된 후 콜백을 실행하도록 예약하는 libuv API를 사용합니다.

일반적으로 코드가 실행되면, 이벤트 루프는 결국 **poll** 단계에 도달하게 됩니다. 이 단계에서는 들어오는 연결이나 요청 등을 기다립니다. 하지만 `setImmediate()`로 콜백이 예약되어 있고 **poll** 단계가 유휴 상태가 되면, **poll** 이벤트를 기다리지 않고 **check** 단계로 넘어갑니다.


### [close callbacks](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#close-callbacks)

소켓이나 핸들이 갑자기 닫히는 경우(예: `socket.destroy()`), `'close'` 이벤트는 이 단계에서 발생합니다. 그렇지 않으면 `process.nextTick()`을 통해 이벤트가 발생합니다.


## [`setImmediate()` vs `setTimeout()`](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#setimmediate-vs-settimeout)

`setImmediate()`와 `setTimeout()`은 비슷해 보이지만, 호출되는 시점에 따라 다르게 동작합니다.

- `setImmediate()`는 현재 **poll** 단계가 완료된 후 스크립트를 실행하도록 설계되었습니다.
- `setTimeout()`은 지정된 시간(밀리초)이 지난 후 스크립트를 실행하도록 예약합니다.

이 타이머들이 실행되는 순서는 호출되는 컨텍스트에 따라 달라집니다. 만약 두 함수가 메인 모듈 내에서 호출된다면, 실행 순서는 프로세스의 성능에 따라 결정됩니다. 이는 머신에서 실행 중인 다른 애플리케이션의 영향을 받을 수 있습니다.

예를 들어, I/O 사이클 밖에서(즉, 메인 모듈에서) 다음 스크립트를 실행하면, 두 타이머의 실행 순서는 프로세스의 성능에 따라 달라지기 때문에 비결정적입니다:

```javascript
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});
```

하지만, 두 호출을 I/O 사이클 내로 옮기면, `setImmediate()`의 콜백이 항상 먼저 실행됩니다:

```javascript
// timeout_vs_immediate.js
const fs = require('node:fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```

`setImmediate()`를 `setTimeout()`보다 사용할 때의 주요 장점은, I/O 사이클 내에서 예약된 경우 `setImmediate()`가 항상 타이머보다 먼저 실행된다는 점입니다. 이는 타이머의 개수와 상관없이 적용됩니다.


## [`process.nextTick()`](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#processnexttick)





### `process.nextTick()` 이해하기

여러분은 `process.nextTick()`이 비동기 API의 일부임에도 불구하고 다이어그램에 표시되지 않은 것을 눈치챘을 겁니다. 이는 `process.nextTick()`이 기술적으로 이벤트 루프의 일부가 아니기 때문입니다. 대신, `nextTickQueue`는 현재 이벤트 루프의 단계와 상관없이 현재 작업이 완료된 후에 처리됩니다. 여기서 *작업*은 기본 C/C++ 핸들러에서의 전환과 실행해야 할 자바스크립트를 처리하는 것을 의미합니다.

다이어그램을 다시 보면, 특정 단계에서 `process.nextTick()`을 호출할 때마다, `process.nextTick()`에 전달된 모든 콜백은 이벤트 루프가 계속되기 전에 해결됩니다. 이는 **재귀적인 `process.nextTick()` 호출을 통해 I/O를 "굶주리게" 할 수 있게 하여** 이벤트 루프가 **poll** 단계에 도달하지 못하게 할 수 있기 때문에 좋지 않은 상황을 만들 수 있습니다.


### 왜 이런 것이 허용될까?

왜 Node.js에 이런 기능이 포함되었을까? 그 이유 중 하나는 API가 비동기적으로 동작해야 한다는 설계 철학 때문이다. 심지어 꼭 그럴 필요가 없는 경우에도 말이다. 다음 코드를 살펴보자.

```javascript
function apiCall(arg, callback) {
  if (typeof arg !== 'string')
    return process.nextTick(
      callback,
      new TypeError('argument should be string')
    );
}
```

이 코드는 인자를 검사하고, 올바르지 않다면 에러를 콜백으로 전달한다. 최근에 업데이트된 API는 `process.nextTick()`에 인자를 전달할 수 있게 해서, 콜백 이후에 전달된 인자들이 콜백의 인자로 전파되도록 한다. 이렇게 하면 함수를 중첩하지 않아도 된다.

여기서 우리는 사용자에게 에러를 반환하지만, 사용자의 나머지 코드가 실행된 **이후**에 반환한다. `process.nextTick()`을 사용하면 `apiCall()`의 콜백이 항상 사용자의 나머지 코드가 실행된 **이후**에, 그리고 이벤트 루프가 진행되기 **전**에 실행되도록 보장한다. 이를 위해 JS 호출 스택이 풀린 후 즉시 제공된 콜백을 실행한다. 이렇게 하면 `RangeError: Maximum call stack size exceeded from v8`에 도달하지 않고도 `process.nextTick()`을 재귀적으로 호출할 수 있다.

이 철학은 잠재적으로 문제가 될 수 있는 상황을 만들기도 한다. 다음 예제를 보자.

```javascript
let bar;

// 비동기 시그니처를 가지고 있지만, 콜백을 동기적으로 호출한다.
function someAsyncApiCall(callback) {
  callback();
}

// 콜백은 `someAsyncApiCall`이 완료되기 전에 호출된다.
someAsyncApiCall(() => {
  // someAsyncApiCall이 완료되지 않았기 때문에, bar는 아직 값이 할당되지 않았다.
  console.log('bar', bar); // undefined
});

bar = 1;
```

사용자는 `someAsyncApiCall()`을 비동기 시그니처로 정의했지만, 실제로는 동기적으로 동작한다. 이 함수가 호출되면, `someAsyncApiCall()`은 실제로 비동기 작업을 하지 않기 때문에 콜백이 같은 이벤트 루프 단계에서 호출된다. 결과적으로 콜백은 `bar`를 참조하려고 하지만, 스크립트가 완료되지 않았기 때문에 해당 변수가 아직 스코프에 없을 수 있다.

콜백을 `process.nextTick()`에 넣으면, 스크립트가 완료될 때까지 실행될 수 있게 되어 모든 변수와 함수 등이 초기화된 후에 콜백이 호출된다. 또한 이벤트 루프가 계속 진행되지 않도록 하는 장점도 있다. 이벤트 루프가 계속되기 전에 사용자에게 에러를 알리는 것이 유용할 수 있다. 다음은 `process.nextTick()`을 사용한 이전 예제이다.

```javascript
let bar;

function someAsyncApiCall(callback) {
  process.nextTick(callback);
}

someAsyncApiCall(() => {
  console.log('bar', bar); // 1
});

bar = 1;
```

다음은 실제 예제이다.

```javascript
const server = net.createServer(() => {}).listen(8080);

server.on('listening', () => {});
```

포트만 전달되면 포트가 즉시 바인딩된다. 따라서 `'listening'` 콜백이 즉시 호출될 수 있다. 문제는 `.on('listening')` 콜백이 아직 설정되지 않았을 수 있다는 점이다.

이를 해결하기 위해 `'listening'` 이벤트를 `nextTick()`에 큐잉하여 스크립트가 완료될 때까지 실행되도록 한다. 이렇게 하면 사용자가 원하는 이벤트 핸들러를 설정할 수 있다.


## [`process.nextTick()` vs `setImmediate()`](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#processnexttick-vs-setimmediate)

사용자 입장에서는 비슷해 보이는 두 가지 호출 방식이 있지만, 이름이 혼란스럽습니다.

- `process.nextTick()`은 같은 단계에서 즉시 실행됩니다.
- `setImmediate()`는 이벤트 루프의 다음 반복 또는 '틱'에서 실행됩니다.

사실 이름이 바뀌어야 합니다. `process.nextTick()`이 `setImmediate()`보다 더 즉각적으로 실행되지만, 이는 과거의 유산으로 인해 변경되기 어려운 상황입니다. 이 이름을 바꾸면 npm에 있는 많은 패키지가 깨질 가능성이 큽니다. 매일 새로운 모듈이 추가되기 때문에, 기다릴수록 더 많은 문제가 발생할 수 있습니다. 이름이 혼란스럽지만, 변경되지는 않을 것입니다.

> 개발자들에게는 이해하기 쉬운 `setImmediate()`를 모든 경우에 사용할 것을 권장합니다.


## `process.nextTick()`을 사용하는 이유

`process.nextTick()`을 사용하는 주된 이유는 두 가지입니다.

1. **에러 처리와 리소스 정리**: 이벤트 루프가 계속 진행되기 전에 사용자가 에러를 처리하거나 더 이상 필요하지 않은 리소스를 정리하거나, 요청을 다시 시도할 수 있도록 합니다.
2. **콜백 실행 시점 조절**: 때로는 콜 스택이 풀린 후, 이벤트 루프가 계속되기 전에 콜백이 실행되도록 해야 할 필요가 있습니다.

### 사용자의 기대에 맞추는 예제

간단한 예제로 사용자의 기대에 맞추는 경우를 살펴보겠습니다.

```javascript
const server = net.createServer();
server.on('connection', conn => {});

server.listen(8080);
server.on('listening', () => {});
```

`listen()`이 이벤트 루프의 시작 부분에서 실행되지만, `listening` 콜백이 `setImmediate()`에 배치된다고 가정해봅시다. 호스트네임이 전달되지 않으면 포트 바인딩은 즉시 발생합니다. 이벤트 루프가 진행되려면 **poll** 단계에 도달해야 하는데, 이는 연결 이벤트가 `listening` 이벤트보다 먼저 발생할 가능성이 있다는 것을 의미합니다.

### `EventEmitter` 확장 예제

`EventEmitter`를 확장하고 생성자 내부에서 이벤트를 발생시키는 경우를 살펴보겠습니다.

```javascript
const EventEmitter = require('node:events');

class MyEmitter extends EventEmitter {
  constructor() {
    super();
    this.emit('event');
  }
}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```

생성자에서 즉시 이벤트를 발생시키면, 사용자가 해당 이벤트에 콜백을 할당하기 전에 스크립트가 처리되지 않아 예상한 결과를 얻을 수 없습니다. 따라서 생성자 내부에서 `process.nextTick()`을 사용하여 생성자가 완료된 후 이벤트를 발생시키도록 할 수 있습니다.

```javascript
const EventEmitter = require('node:events');

class MyEmitter extends EventEmitter {
  constructor() {
    super();

    // nextTick을 사용하여 핸들러가 할당된 후 이벤트를 발생시킴
    process.nextTick(() => {
      this.emit('event');
    });
  }
}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```

이렇게 하면 사용자가 이벤트 핸들러를 할당한 후에 이벤트가 발생하므로 예상한 결과를 얻을 수 있습니다.


