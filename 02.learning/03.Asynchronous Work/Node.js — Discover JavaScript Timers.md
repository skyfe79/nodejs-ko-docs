# Table of Contents

- [Discover JavaScript Timers](#discover-javascript-timers)
  - [`setTimeout()`](#settimeout)
    - [Zero delay](#zero-delay)
  - [`setInterval()`](#setinterval)
  - [재귀적 setTimeout](#재귀적-settimeout)

# [Discover JavaScript Timers](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#discover-javascript-timers)





## [`setTimeout()`](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#settimeout)

자바스크립트 코드를 작성하다 보면 특정 함수의 실행을 지연시키고 싶을 때가 있습니다. 이때 `setTimeout`을 사용할 수 있습니다. `setTimeout`은 나중에 실행할 콜백 함수와, 얼마나 지연시킬지 밀리초 단위로 지정할 수 있습니다.

```javascript
setTimeout(() => {
  // 2초 후에 실행
}, 2000);

setTimeout(() => {
  // 50밀리초 후에 실행
}, 50);
```

이 구문은 새로운 함수를 정의합니다. 콜백 함수 안에서 원하는 다른 함수를 호출하거나, 기존 함수 이름과 인자를 전달할 수도 있습니다.

```javascript
const myFunction = (firstParam, secondParam) => {
  // 어떤 작업 수행
};

// 2초 후에 실행
setTimeout(myFunction, 2000, firstParam, secondParam);
```

`setTimeout`은 타이머 ID를 반환합니다. 일반적으로 이 ID는 사용하지 않지만, 저장해두었다가 예정된 함수 실행을 취소하고 싶을 때 `clearTimeout`을 사용할 수 있습니다.

```javascript
const id = setTimeout(() => {
  // 2초 후에 실행될 예정
}, 2000);

// 마음이 바뀌어 취소
clearTimeout(id);
```


### [Zero delay](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#zero-delay)

타임아웃 지연 시간을 `0`으로 설정하면, 콜백 함수는 현재 함수 실행이 끝난 후 가능한 한 빨리 실행됩니다.

```javascript
setTimeout(() => {
  console.log('after ');
}, 0);

console.log(' before ');
```

이 코드는 다음과 같이 출력됩니다.

```
before
after
```

이 방법은 CPU를 블로킹하지 않고, 무거운 계산을 수행하는 동안 다른 함수가 실행될 수 있도록 스케줄러에 함수를 큐에 넣는 데 특히 유용합니다.

> 일부 브라우저(IE와 Edge)는 동일한 기능을 수행하는 `setImmediate()` 메서드를 구현했지만, 이는 표준이 아니며 [다른 브라우저에서는 사용할 수 없습니다](https://caniuse.com/#feat=setimmediate). 하지만 Node.js에서는 표준 함수로 제공됩니다.


## [`setInterval()`](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#setinterval)

`setInterval`은 `setTimeout`과 비슷한 함수입니다. 하지만 한 번만 실행하는 `setTimeout`과 달리, 지정한 시간 간격(밀리초 단위)마다 콜백 함수를 계속 실행합니다.

```javascript
setInterval(() => {
  // 2초마다 실행
}, 2000);
```

위 함수는 2초마다 실행됩니다. 이를 멈추려면 `setInterval`이 반환한 인터벌 ID를 `clearInterval`에 전달하면 됩니다.

```javascript
const id = setInterval(() => {
  // 2초마다 실행
}, 2000);

clearInterval(id);
```

`clearInterval`을 `setInterval` 콜백 함수 안에서 호출하는 경우도 많습니다. 이렇게 하면 다시 실행할지 멈출지를 자동으로 결정할 수 있습니다. 예를 들어, 아래 코드는 `App.somethingIWait` 값이 `arrived`가 될 때까지 특정 작업을 실행합니다.

```javascript
const interval = setInterval(() => {
  if (App.somethingIWait === 'arrived') {
    clearInterval(interval);
  }
  // 그렇지 않으면 작업 수행
}, 100);
```


## 재귀적 setTimeout

`setInterval`은 함수가 실행을 마치는 시점을 고려하지 않고, n 밀리초마다 함수를 시작합니다.

함수가 항상 동일한 시간 동안 실행된다면 문제가 없습니다:

![setInterval 정상 작동](https://nodejs.org/_next/image?url=%2Fstatic%2Fimages%2Flearn%2Fjavascript-timers%2Fsetinterval-ok.png&w=3840&q=75)

하지만 함수가 네트워크 상태 등에 따라 실행 시간이 달라질 수 있습니다:

![setInterval 실행 시간 변동](https://nodejs.org/_next/image?url=%2Fstatic%2Fimages%2Flearn%2Fjavascript-timers%2Fsetinterval-varying-duration.png&w=3840&q=75)

또한, 한 번의 긴 실행이 다음 실행과 겹칠 수도 있습니다:

![setInterval 실행 겹침](https://nodejs.org/_next/image?url=%2Fstatic%2Fimages%2Flearn%2Fjavascript-timers%2Fsetinterval-overlapping.png&w=3840&q=75)

이를 방지하기 위해, 콜백 함수가 끝날 때 재귀적으로 `setTimeout`을 호출하도록 스케줄링할 수 있습니다:

```javascript
const myFunction = () => {
  // 작업 수행

  setTimeout(myFunction, 1000);
};

setTimeout(myFunction, 1000);
```

이렇게 하면 다음과 같은 시나리오를 달성할 수 있습니다:

![재귀적 setTimeout](https://nodejs.org/_next/image?url=%2Fstatic%2Fimages%2Flearn%2Fjavascript-timers%2Frecursive-settimeout.png&w=3840&q=75)

`setTimeout`과 `setInterval`은 Node.js의 [Timers 모듈](https://nodejs.org/api/timers.html)을 통해 사용할 수 있습니다.

Node.js는 또한 `setImmediate()`를 제공하는데, 이는 `setTimeout(() => {}, 0)`과 동일하며 주로 Node.js 이벤트 루프와 함께 사용됩니다.


