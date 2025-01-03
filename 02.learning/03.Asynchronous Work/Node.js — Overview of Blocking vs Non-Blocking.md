# 블로킹 vs 논블로킹 개요

이 문서는 Node.js에서 **블로킹**과 **논블로킹** 호출의 차이를 설명합니다. 이벤트 루프와 libuv에 대해 언급하지만, 해당 주제에 대한 사전 지식은 필요하지 않습니다. 독자는 JavaScript 언어와 Node.js의 [콜백 패턴](https://nodejs.org/en/learn/asynchronous-work/javascript-asynchronous-programming-and-callbacks)에 대한 기본적인 이해가 있다고 가정합니다.

> "I/O"는 주로 [libuv](https://libuv.org/)가 지원하는 시스템의 디스크 및 네트워크와의 상호작용을 의미합니다.


## 블로킹(Blocking)

**블로킹**은 Node.js 프로세스에서 추가적인 JavaScript 코드의 실행이 비 JavaScript 작업이 완료될 때까지 기다려야 하는 상황을 말합니다. 이는 **블로킹** 작업이 진행 중일 때 이벤트 루프가 JavaScript를 계속 실행할 수 없기 때문에 발생합니다.

Node.js에서 I/O와 같은 비 JavaScript 작업을 기다리는 대신 CPU 집약적인 작업으로 인해 성능이 저하되는 JavaScript는 일반적으로 **블로킹**이라고 부르지 않습니다. Node.js 표준 라이브러리에서 libuv를 사용하는 동기 메서드가 가장 일반적으로 사용되는 **블로킹** 작업입니다. 네이티브 모듈도 **블로킹** 메서드를 포함할 수 있습니다.

Node.js 표준 라이브러리의 모든 I/O 메서드는 **논블로킹**인 비동기 버전을 제공하며, 이들은 콜백 함수를 인자로 받습니다. 일부 메서드는 `Sync`로 끝나는 이름을 가진 **블로킹** 버전도 함께 제공합니다.


## 코드 비교

**블로킹** 메서드는 **동기적으로** 실행되고, **논블로킹** 메서드는 **비동기적으로** 실행됩니다.

파일 시스템 모듈을 예로 들어, 다음은 **동기적** 파일 읽기입니다:

```javascript
const fs = require('node:fs');

const data = fs.readFileSync('/file.md'); // 파일을 읽을 때까지 여기서 블로킹됨
```

다음은 동일한 작업을 수행하는 **비동기적** 예제입니다:

```javascript
const fs = require('node:fs');

fs.readFile('/file.md', (err, data) => {
  if (err) throw err;
});
```

첫 번째 예제가 두 번째보다 간단해 보이지만, 두 번째 줄에서 파일을 모두 읽을 때까지 추가적인 JavaScript 실행이 **블로킹**된다는 단점이 있습니다. 동기 버전에서는 오류가 발생하면 이를 잡아내지 않으면 프로세스가 중단됩니다. 비동기 버전에서는 오류를 던질지 여부를 작성자가 결정할 수 있습니다.

예제를 조금 더 확장해 보겠습니다:

```javascript
const fs = require('node:fs');

const data = fs.readFileSync('/file.md'); // 파일을 읽을 때까지 여기서 블로킹됨
console.log(data);
moreWork(); // console.log 이후에 실행됨
```

다음은 비슷하지만 동등하지 않은 비동기 예제입니다:

```javascript
const fs = require('node:fs');

fs.readFile('/file.md', (err, data) => {
  if (err) throw err;
  console.log(data);
});
moreWork(); // console.log 이전에 실행됨
```

첫 번째 예제에서는 `console.log`가 `moreWork()`보다 먼저 호출됩니다. 두 번째 예제에서는 `fs.readFile()`이 **논블로킹**이므로 JavaScript 실행이 계속되고 `moreWork()`가 먼저 호출됩니다. 파일 읽기가 완료될 때까지 기다리지 않고 `moreWork()`를 실행할 수 있는 능력은 더 높은 처리량을 가능하게 하는 핵심 설계 선택입니다.


## 동시성과 처리량

Node.js에서 자바스크립트 실행은 단일 스레드로 동작합니다. 따라서 동시성은 이벤트 루프가 다른 작업을 완료한 후 자바스크립트 콜백 함수를 실행할 수 있는 능력을 의미합니다. 동시적으로 실행될 것으로 예상되는 모든 코드는 I/O와 같은 비자바스크립트 작업이 진행되는 동안 이벤트 루프가 계속 실행될 수 있도록 해야 합니다.

예를 들어, 웹 서버에 대한 각 요청이 완료되는 데 50ms가 걸리고, 그 중 45ms가 비동기적으로 처리될 수 있는 데이터베이스 I/O 작업이라고 가정해 보겠습니다. **논블로킹** 비동기 작업을 선택하면 요청당 45ms를 다른 요청을 처리하는 데 사용할 수 있습니다. **블로킹** 메서드 대신 **논블로킹** 메서드를 사용하기만 해도 처리 능력에 큰 차이가 생깁니다.

이벤트 루프는 다른 많은 언어에서 동시 작업을 처리하기 위해 추가 스레드를 생성할 수 있는 모델과는 다릅니다.


## 블로킹과 논블로킹 코드를 혼용할 때의 위험성

I/O 작업을 다룰 때 피해야 할 몇 가지 패턴이 있습니다. 예제를 통해 살펴보겠습니다.

```javascript
const fs = require('node:fs');

fs.readFile('/file.md', (err, data) => {
  if (err) throw err;
  console.log(data);
});
fs.unlinkSync('/file.md');
```

위 예제에서 `fs.unlinkSync()`는 `fs.readFile()`보다 먼저 실행될 가능성이 높습니다. 이 경우 `file.md` 파일이 실제로 읽히기 전에 삭제될 수 있습니다. 이를 완전히 **논블로킹** 방식으로 작성하고, 작업 순서를 보장하는 더 나은 방법은 다음과 같습니다.

```javascript
const fs = require('node:fs');

fs.readFile('/file.md', (readFileErr, data) => {
  if (readFileErr) throw readFileErr;
  console.log(data);
  fs.unlink('/file.md', unlinkErr => {
    if (unlinkErr) throw unlinkErr;
  });
});
```

위 코드는 `fs.readFile()`의 콜백 내에서 **논블로킹** 방식으로 `fs.unlink()`를 호출하여 작업 순서를 보장합니다.


## [추가 리소스](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#additional-resources)

- [libuv](https://libuv.org/)


