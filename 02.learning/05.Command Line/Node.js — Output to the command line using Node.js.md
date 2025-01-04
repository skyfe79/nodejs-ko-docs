# Table of Contents

- [[Output to the command line using Node.js](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#output-to-the-command-line-using-nodejs)](#output-to-the-command-line-using-nodejshttpsnodejsorgenlearncommand-linerun-nodejs-scripts-from-the-command-lineoutput-to-the-command-line-using-nodejs)
    - [콘솔 모듈을 사용한 기본 출력](#콘솔-모듈을-사용한-기본-출력)
    - [[콘솔 지우기](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#clear-the-console)](#콘솔-지우기httpsnodejsorgenlearncommand-linerun-nodejs-scripts-from-the-command-lineclear-the-console)
    - [엘리먼트 개수 세기](#엘리먼트-개수-세기)
    - [[카운트 초기화](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#reset-counting)](#카운트-초기화httpsnodejsorgenlearncommand-linerun-nodejs-scripts-from-the-command-linereset-counting)
    - [스택 트레이스 출력하기](#스택-트레이스-출력하기)
    - [[소요 시간 계산하기](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#calculate-the-time-spent)](#소요-시간-계산하기httpsnodejsorgenlearncommand-linerun-nodejs-scripts-from-the-command-linecalculate-the-time-spent)
    - [[stdout와 stderr](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#stdout-and-stderr)](#stdout와-stderrhttpsnodejsorgenlearncommand-linerun-nodejs-scripts-from-the-command-linestdout-and-stderr)
    - [[터미널 출력에 색상 입히기](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#color-the-output)](#터미널-출력에-색상-입히기httpsnodejsorgenlearncommand-linerun-nodejs-scripts-from-the-command-linecolor-the-output)

# [Output to the command line using Node.js](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#output-to-the-command-line-using-nodejs)





### 콘솔 모듈을 사용한 기본 출력

Node.js는 커맨드라인과 상호작용할 수 있는 매우 유용한 방법을 제공하는 [`console` 모듈](https://nodejs.org/docs/latest-v22.x/api/console.html)을 제공합니다.

이 모듈은 브라우저에서 볼 수 있는 `console` 객체와 기본적으로 동일합니다.

가장 기본적이고 자주 사용되는 메서드는 `console.log()`입니다. 이 메서드는 전달된 문자열을 콘솔에 출력합니다.

객체를 전달하면 문자열로 렌더링합니다.

`console.log`에 여러 변수를 전달할 수도 있습니다. 예를 들어:

```javascript
const x = 'x';
const y = 'y';

console.log(x, y);
```

Node.js는 두 변수를 모두 출력합니다.

또한 변수와 형식 지정자를 전달하여 문장을 예쁘게 포맷할 수도 있습니다. 예를 들어:

```javascript
console.log('My %s has %d ears', 'cat', 2);
```

- `%s`: 변수를 문자열로 포맷
- `%d`: 변수를 숫자로 포맷
- `%i`: 변수의 정수 부분만 포맷
- `%o`: 변수를 객체로 포맷

예제:

```javascript
console.log('%o', Number);
```


### [콘솔 지우기](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#clear-the-console)

`console.clear()`는 콘솔을 지웁니다. (사용하는 콘솔에 따라 동작이 다를 수 있습니다.)


### 엘리먼트 개수 세기

`console.count()`는 유용한 메서드입니다.

다음 코드를 살펴보세요:

```javascript
const x = 1;
const y = 2;
const z = 3;

console.count(
  'x의 값은 ' + x + '이고, 몇 번 확인되었을까요?'
);

console.count(
  'x의 값은 ' + x + '이고, 몇 번 확인되었을까요?'
);

console.count(
  'y의 값은 ' + y + '이고, 몇 번 확인되었을까요?'
);
```

`console.count()`는 특정 문자열이 몇 번 출력되었는지 세고, 그 횟수를 문자열 옆에 출력합니다.

이 메서드를 사용해 사과와 오렌지의 개수를 세는 것도 가능합니다:

```javascript
const oranges = ['orange', 'orange'];
const apples = ['just one apple'];

oranges.forEach(fruit => {
  console.count(fruit);
});
apples.forEach(fruit => {
  console.count(fruit);
});
```


### [카운트 초기화](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#reset-counting)

`console.countReset()` 메서드는 `console.count()`와 함께 사용된 카운터를 초기화합니다.

여기서는 사과와 오렌지 예제를 통해 이를 설명하겠습니다.

```javascript
const oranges = ['orange', 'orange'];
const apples = ['just one apple'];

oranges.forEach(fruit => {
  console.count(fruit);
});
apples.forEach(fruit => {
  console.count(fruit);
});

console.countReset('orange');

oranges.forEach(fruit => {
  console.count(fruit);
});
```

`console.countReset('orange')` 호출이 카운터 값을 0으로 초기화하는 것을 확인할 수 있습니다.


### 스택 트레이스 출력하기

코드의 특정 부분에 어떻게 도달했는지 궁금할 때, 함수의 호출 스택 트레이스를 출력하면 유용할 수 있습니다.

`console.trace()`를 사용하면 이를 확인할 수 있습니다.

```javascript
const function2 = () => console.trace();
const function1 = () => function2();
function1();
```

이 코드를 실행하면 스택 트레이스가 출력됩니다. Node.js REPL에서 실행했을 때의 출력 예시는 다음과 같습니다.

```
Trace
    at function2 (repl:1:33)
    at function1 (repl:1:25)
    at repl:1:1
    at ContextifyScript.Script.runInThisContext (vm.js:44:33)
    at REPLServer.defaultEval (repl.js:239:29)
    at bound (domain.js:301:14)
    at REPLServer.runBound [as eval] (domain.js:314:12)
    at REPLServer.onLine (repl.js:440:10)
    at emitOne (events.js:120:20)
    at REPLServer.emit (events.js:210:7)
```


### [소요 시간 계산하기](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#calculate-the-time-spent)

`time()`과 `timeEnd()`를 사용하면 함수 실행에 걸리는 시간을 쉽게 측정할 수 있습니다.

```javascript
const doSomething = () => console.log('test');
const measureDoingSomething = () => {
  console.time('doSomething()');
  // 작업을 수행하고 소요 시간을 측정
  doSomething();
  console.timeEnd('doSomething()');
};
measureDoingSomething();
```


### [stdout와 stderr](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#stdout-and-stderr)

`console.log`는 콘솔에 메시지를 출력하는 데 유용합니다. 이를 **표준 출력(stdout)**이라고 부릅니다.

반면 `console.error`는 **표준 에러(stderr)** 스트림에 출력합니다. 

이 메시지는 콘솔에는 나타나지 않지만, 에러 로그에는 기록됩니다.


### [터미널 출력에 색상 입히기](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#color-the-output)

> **참고** 이 부분은 버전 22.11을 기준으로 작성되었으며, `styleText`가 '개발 중'으로 표시되어 있습니다.

터미널에서 멋진 출력을 얻기 위해 특정 텍스트를 붙여넣고 싶은 경우가 많을 것입니다.

`node:util` 모듈에서 제공하는 `styleText` 함수를 사용하면 이를 쉽게 할 수 있습니다. 이 함수를 어떻게 사용하는지 알아보겠습니다.

먼저, `node:util` 모듈에서 `styleText` 함수를 불러와야 합니다:

```javascript
import { styleText } from 'node:util';
```

그런 다음, 이 함수를 사용해 텍스트에 스타일을 적용할 수 있습니다:

```javascript
console.log(
  styleText(['red'], '이 텍스트는 빨간색입니다 ') +
    styleText(['green, bold'], '이 텍스트는 초록색이고 굵게 표시됩니다 ') +
    '이 텍스트는 일반 텍스트입니다'
);
```

첫 번째 인자는 스타일 배열이고, 두 번째 인자는 스타일을 적용할 텍스트입니다. 더 자세한 내용은 [공식 문서](https://nodejs.org/docs/latest-v22.x/api/util.html#utilstyletextformat-text-options)를 참고하세요.


