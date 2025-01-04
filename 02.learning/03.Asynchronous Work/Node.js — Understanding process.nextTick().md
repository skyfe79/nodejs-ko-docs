# Table of Contents

- [process.nextTick() 이해하기](#processnexttick-이해하기)
      - [이벤트 순서 예제](#이벤트-순서-예제)
      - [[예제 출력:](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#example-output)](#예제-출력httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlexample-output)

# process.nextTick() 이해하기

Node.js 이벤트 루프를 이해하려면 `process.nextTick()`이 중요한 역할을 합니다. 런타임이 이벤트를 위해 자바스크립트로 콜백을 호출할 때마다 이를 틱(tick)이라고 부릅니다.

`process.nextTick()`에 함수를 전달하면, 현재 작업이 완료된 후 이벤트 루프의 다음 단계로 넘어가기 전에 해당 함수를 즉시 실행하도록 엔진에 지시합니다:

```javascript
process.nextTick(() => {
  // 어떤 작업을 수행
});
```

이벤트 루프는 현재 함수 코드를 처리하는 데 바쁩니다. 이 작업이 끝나면, JS 엔진은 해당 작업 중에 `nextTick` 호출로 전달된 모든 함수를 실행합니다.

이 방법을 통해 JS 엔진에 함수를 비동기적으로(현재 함수 이후에) 처리하도록 지시할 수 있지만, 가능한 한 빨리 실행되도록 큐에 넣지 않고 처리합니다.

`setTimeout(() => {}, 0)`을 호출하면 다음 틱의 끝에서 함수가 실행됩니다. 이는 `nextTick()`을 사용할 때보다 훨씬 나중에 실행됩니다. `nextTick()`은 호출을 우선시하여 다음 틱이 시작되기 직전에 실행합니다.

다음 이벤트 루프 반복에서 코드가 이미 실행되었는지 확인하고 싶을 때 `nextTick()`을 사용하세요.


#### 이벤트 순서 예제

```javascript
console.log('Hello => number 1');

setImmediate(() => {
  console.log('Running before the timeout => number 3');
});

setTimeout(() => {
  console.log('The timeout running last => number 4');
}, 0);

process.nextTick(() => {
  console.log('Running at next tick => number 2');
});
```

이 코드는 Node.js에서 비동기 작업의 실행 순서를 보여주는 예제입니다. 각 이벤트가 어떤 순서로 실행되는지 확인할 수 있습니다.


#### [예제 출력:](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#example-output)

```javascript
Hello => number 1
Running at next tick => number 2
Running before the timeout => number 3
The timeout running last => number 4
```

ShellCopy to clipboard

실행할 때마다 정확한 출력 결과가 다를 수 있습니다.


