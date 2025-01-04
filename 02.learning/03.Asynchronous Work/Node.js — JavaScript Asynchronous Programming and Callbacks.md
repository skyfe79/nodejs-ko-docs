# Table of Contents

- [[JavaScript Asynchronous Programming and Callbacks](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#javascript-asynchronous-programming-and-callbacks)](#javascript-asynchronous-programming-and-callbackshttpsnodejsorgenlearnasynchronous-workasynchronous-flow-controljavascript-asynchronous-programming-and-callbacks)
  - [프로그래밍 언어에서의 비동기성](#프로그래밍-언어에서의-비동기성)
  - [[JavaScript](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#javascript)](#javascripthttpsnodejsorgenlearnasynchronous-workasynchronous-flow-controljavascript)
  - [[콜백](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#callbacks)](#콜백httpsnodejsorgenlearnasynchronous-workasynchronous-flow-controlcallbacks)
    - [콜백에서 에러 처리하기](#콜백에서-에러-처리하기)
    - [콜백의 문제점](#콜백의-문제점)
    - [콜백 대체 방법](#콜백-대체-방법)

# [JavaScript Asynchronous Programming and Callbacks](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#javascript-asynchronous-programming-and-callbacks)





## 프로그래밍 언어에서의 비동기성

컴퓨터는 기본적으로 비동기적으로 동작하도록 설계되어 있습니다.

비동기성이란, 메인 프로그램의 흐름과 독립적으로 일어날 수 있는 것을 의미합니다.

현재의 일반적인 컴퓨터에서는 모든 프로그램이 특정 시간 동안 실행된 후, 다른 프로그램이 실행될 수 있도록 멈춥니다. 이 과정은 매우 빠르게 반복되기 때문에 눈치채기 어렵습니다. 우리는 컴퓨터가 여러 프로그램을 동시에 실행한다고 생각하지만, 이는 착시 현상입니다(다중 프로세서 머신은 제외).

프로그램은 내부적으로 *인터럽트*를 사용합니다. 인터럽트는 시스템의 주의를 끌기 위해 프로세서에 보내는 신호입니다.

지금은 이 내부 동작에 대해 깊이 들어가지는 않겠습니다. 다만, 프로그램이 비동기적으로 동작하고, 주의가 필요할 때까지 실행을 멈추는 것이 일반적이라는 점을 기억해 주세요. 이렇게 하면 컴퓨터는 그동안 다른 작업을 실행할 수 있습니다. 예를 들어, 프로그램이 네트워크 응답을 기다릴 때, 요청이 완료될 때까지 프로세서를 멈출 수는 없습니다.

일반적으로 프로그래밍 언어는 동기적으로 동작하며, 일부 언어는 비동기성을 관리할 수 있는 방법을 제공합니다. C, Java, C#, PHP, Go, Ruby, Swift, Python은 모두 기본적으로 동기적입니다. 이들 중 일부는 스레드를 사용하거나 새로운 프로세스를 생성하여 비동기 작업을 처리합니다.


## [JavaScript](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#javascript)

JavaScript는 기본적으로 **동기식**이며 단일 스레드로 동작합니다. 이는 코드가 새로운 스레드를 생성하거나 병렬로 실행할 수 없음을 의미합니다.

코드는 순차적으로 한 줄씩 실행됩니다. 예를 들어:

```javascript
const a = 1;
const b = 2;
const c = a * b;
console.log(c);
doSomething();
```

JavaScript는 브라우저 내부에서 탄생했습니다. 초기 주요 역할은 `onClick`, `onMouseOver`, `onChange`, `onSubmit`과 같은 사용자 액션에 반응하는 것이었습니다. 동기식 프로그래밍 모델로 이를 어떻게 처리할 수 있었을까요?

그 해답은 환경에 있었습니다. **브라우저**는 이러한 기능을 처리할 수 있는 API 세트를 제공함으로써 이를 가능하게 했습니다.

최근에는 Node.js가 논블로킹 I/O 환경을 도입하여 파일 접근, 네트워크 호출 등으로 이 개념을 확장했습니다.


## [콜백](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#callbacks)

사용자가 언제 버튼을 클릭할지 알 수 없습니다. 따라서 **클릭 이벤트에 대한 이벤트 핸들러를 정의**합니다. 이 이벤트 핸들러는 이벤트가 발생했을 때 호출될 함수를 인자로 받습니다:

```javascript
document.getElementById('button').addEventListener('click', () => {
  // 버튼이 클릭됨
});
```

이것이 바로 **콜백**입니다.

콜백은 다른 함수에 값으로 전달되는 간단한 함수로, 이벤트가 발생했을 때만 실행됩니다. 자바스크립트는 함수를 변수에 할당하거나 다른 함수에 전달할 수 있는 **퍼스트클래스 함수**를 지원하기 때문에 이런 방식이 가능합니다. 이러한 함수를 **고차 함수**라고 부릅니다.

클라이언트 코드 전체를 `window` 객체의 `load` 이벤트 리스너로 감싸는 것이 일반적입니다. 이렇게 하면 페이지가 준비되었을 때만 콜백 함수가 실행됩니다:

```javascript
window.addEventListener('load', () => {
  // 윈도우가 로드됨
  // 원하는 작업 수행
});
```

콜백은 DOM 이벤트뿐만 아니라 다양한 곳에서 사용됩니다.

타이머를 사용하는 것이 대표적인 예입니다:

```javascript
setTimeout(() => {
  // 2초 후에 실행
}, 2000);
```

XHR 요청도 콜백을 받습니다. 이 예제에서는 특정 이벤트(요청 상태 변경)가 발생했을 때 호출될 함수를 프로퍼티에 할당합니다:

```javascript
const xhr = new XMLHttpRequest();
xhr.onreadystatechange = () => {
  if (xhr.readyState === 4) {
    xhr.status === 200 ? console.log(xhr.responseText) : console.error('에러 발생');
  }
};
xhr.open('GET', 'https://yoursite.com');
xhr.send();
```


### 콜백에서 에러 처리하기

콜백에서 에러를 어떻게 처리할까요? Node.js에서 채택한 매우 일반적인 전략은 **에러 우선 콜백(error-first callbacks)**을 사용하는 것입니다. 이는 콜백 함수의 첫 번째 매개변수로 에러 객체를 전달하는 방식입니다.

에러가 없을 경우, 이 객체는 `null`입니다. 에러가 발생하면, 해당 객체는 에러에 대한 설명과 기타 정보를 포함합니다.

```javascript
const fs = require('node:fs');

fs.readFile('/file.json', (err, data) => {
  if (err) {
    // 에러 처리
    console.log(err);
    return;
  }

  // 에러 없음, 데이터 처리
  console.log(data);
});
```


### 콜백의 문제점

콜백은 간단한 경우에 유용합니다! 하지만 모든 콜백은 코드의 중첩 수준을 하나씩 증가시킵니다. 콜백이 많아지면 코드가 매우 빠르게 복잡해집니다:

```javascript
window.addEventListener('load', () => {
  document.getElementById('button').addEventListener('click', () => {
    setTimeout(() => {
      items.forEach(item => {
        // 여기에 코드를 작성
      });
    }, 2000);
  });
});
```

이 예제는 단순히 4단계의 중첩을 보여줍니다. 하지만 더 많은 중첩 수준을 가진 코드를 본 적이 있으며, 이는 전혀 즐겁지 않습니다.

이 문제를 어떻게 해결할 수 있을까요?


### 콜백 대체 방법

ES6부터 자바스크립트는 콜백을 사용하지 않고도 비동기 코드를 처리할 수 있는 여러 기능을 도입했습니다. 이는 **Promise(ES6)**와 **Async/Await(ES2017)**를 포함합니다.


