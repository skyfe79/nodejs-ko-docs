# setImmediate() 이해하기

코드를 가능한 한 빨리 비동기적으로 실행하고 싶을 때, Node.js에서 제공하는 `setImmediate()` 함수를 사용할 수 있습니다.

```javascript
setImmediate(() => {
  // 코드 실행
});
```

`setImmediate()`에 인자로 전달된 함수는 콜백으로, 이벤트 루프의 다음 반복에서 실행됩니다.

그렇다면 `setImmediate()`는 `setTimeout(() => {}, 0)`(0ms 타임아웃), `process.nextTick()`, 그리고 `Promise.then()`과 어떻게 다를까요?

`process.nextTick()`에 전달된 함수는 현재 이벤트 루프 반복에서 현재 작업이 끝난 후 실행됩니다. 이는 `setTimeout`과 `setImmediate`보다 항상 먼저 실행됨을 의미합니다.

0ms 지연 시간을 가진 `setTimeout()` 콜백은 `setImmediate()`와 매우 유사합니다. 실행 순서는 다양한 요소에 따라 달라지지만, 둘 다 이벤트 루프의 다음 반복에서 실행됩니다.

`process.nextTick` 콜백은 `process.nextTick queue`에 추가됩니다. `Promise.then()` 콜백은 `promises microtask queue`에 추가됩니다. `setTimeout`과 `setImmediate` 콜백은 `macrotask queue`에 추가됩니다.

이벤트 루프는 `process.nextTick queue`의 작업을 먼저 실행한 다음, `promises microtask queue`를 실행하고, 마지막으로 `macrotask queue`를 실행합니다.

다음은 `setImmediate()`, `process.nextTick()`, 그리고 `Promise.then()` 사이의 순서를 보여주는 예제입니다.

```javascript
const baz = () => console.log('baz');
const foo = () => console.log('foo');
const zoo = () => console.log('zoo');

const start = () => {
  console.log('start');
  setImmediate(baz);
  new Promise((resolve, reject) => {
    resolve('bar');
  }).then(resolve => {
    console.log(resolve);
    process.nextTick(zoo);
  });
  process.nextTick(foo);
};

start();

// 출력 순서: start foo bar zoo baz
```

이 코드는 먼저 `start()`를 호출한 다음, `process.nextTick queue`에서 `foo()`를 호출합니다. 그 후, `promises microtask queue`를 처리하며 `bar`를 출력하고 동시에 `process.nextTick queue`에 `zoo()`를 추가합니다. 그런 다음 방금 추가된 `zoo()`를 호출합니다. 마지막으로 `macrotask queue`에 있는 `baz()`가 호출됩니다.

위에서 설명한 원칙은 CommonJS 경우에 적용되지만, ES Modules(예: `mjs` 파일)에서는 실행 순서가 다르다는 점을 유의해야 합니다.

```javascript
// 출력 순서: start bar foo zoo baz
```

이는 ES Module이 비동기 작업으로 로드되기 때문에 전체 스크립트가 실제로 이미 `promises microtask queue`에 있기 때문입니다. 따라서 Promise가 즉시 해결되면 그 콜백이 `microtask` 큐에 추가됩니다. Node.js는 다른 큐로 이동하기 전에 큐를 비우려고 시도하므로, `bar`가 먼저 출력되는 것을 볼 수 있습니다.


