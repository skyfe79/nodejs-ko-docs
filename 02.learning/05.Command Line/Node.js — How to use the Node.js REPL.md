# Table of Contents

- [How to use the Node.js REPL](#how-to-use-the-nodejs-repl)
  - [Node.js REPL이란 무엇인가?](#nodejs-repl이란-무엇인가)
  - [Node.js REPL 사용 방법](#nodejs-repl-사용-방법)
    - [`_` 특수 변수](#_-특수-변수)
    - [위쪽 화살표 키](#위쪽-화살표-키)
    - [Dot commands](#dot-commands)
    - [JavaScript 파일에서 REPL 실행하기](#javascript-파일에서-repl-실행하기)

# [How to use the Node.js REPL](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#how-to-use-the-nodejs-repl)





## Node.js REPL이란 무엇인가?

Node.js는 기본적으로 REPL(Read-Eval-Print Loop) 환경을 제공합니다. 이 환경을 통해 여러분은 자바스크립트 코드를 대화형으로 실행할 수 있습니다. REPL은 터미널을 통해 접근할 수 있으며, 작은 코드 조각을 테스트하기에 매우 유용합니다.


## Node.js REPL 사용 방법

`node` 명령어는 Node.js 스크립트를 실행할 때 사용합니다:

```shell
node script.js
```

스크립트 파일 없이 `node` 명령어만 실행하거나 인자를 전달하지 않으면 REPL 세션이 시작됩니다:

```shell
node
```

> **참고:** `REPL`은 Read Evaluate Print Loop의 약자입니다. 이는 사용자 입력으로 단일 표현식을 받아 실행한 후 결과를 콘솔에 반환하는 프로그래밍 언어 환경(기본적으로 콘솔 윈도우)입니다. REPL 세션은 간단한 JavaScript 코드를 빠르게 테스트할 수 있는 편리한 방법을 제공합니다.

터미널에서 시도해보면 다음과 같은 화면이 나타납니다:

```shell
❯ node
>
```

명령어는 대기 모드로 유지되며, 사용자가 무언가를 입력하기를 기다립니다.

> **팁:** 터미널을 여는 방법이 확실하지 않다면, "How to open terminal on your-operating-system"를 검색해보세요.

REPL은 정확히 말하면 JavaScript 코드를 입력하기를 기다리고 있습니다. 간단하게 시작해보세요:

```shell
> console.log('test')
test
undefined
>
```

첫 번째 값인 `test`는 콘솔에 출력하라고 지시한 결과입니다. 그 다음 `undefined`는 `console.log()`를 실행한 반환 값입니다. Node는 이 코드 줄을 읽고, 평가한 후 결과를 출력한 다음, 더 많은 코드 줄을 기다립니다. Node는 REPL에서 실행하는 모든 코드에 대해 이 세 단계를 반복하며, 세션을 종료할 때까지 계속됩니다. 이것이 REPL이라는 이름이 붙은 이유입니다.

Node는 JavaScript 코드의 결과를 자동으로 출력합니다. 예를 들어, 다음 줄을 입력하고 엔터를 누르세요:

```shell
> 5 === '5'
false
>
```

위 두 줄의 출력 결과 차이를 주목하세요. Node REPL은 `console.log()`를 실행한 후 `undefined`를 출력했지만, `5 === '5'`의 결과는 바로 출력했습니다. 전자는 단순히 JavaScript의 문장(statement)이고, 후자는 표현식(expression)이라는 점을 기억해야 합니다.

테스트하려는 코드가 여러 줄로 구성된 경우도 있습니다. 예를 들어, 무작위 숫자를 생성하는 함수를 정의하려면 REPL 세션에서 다음 줄을 입력하고 엔터를 누르세요:

```shell
function generateRandom() {
...
```

Node REPL은 여러분이 코드 작성을 아직 마치지 않았다는 것을 알아차리고, 여러 줄 입력 모드로 전환합니다. 이제 함수 정의를 마치고 엔터를 누르세요:

```shell
function generateRandom() {
...return Math.random()
}
undefined
```


### `_` 특수 변수

코드를 실행한 후 `_`를 입력하면, 마지막 연산의 결과가 출력됩니다.


### [위쪽 화살표 키](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#the-up-arrow-key)

`위쪽 화살표 키`를 누르면 현재 또는 이전 REPL 세션에서 실행했던 코드 라인들의 기록에 접근할 수 있습니다.


### [Dot commands](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#dot-commands)

REPL에는 점 `.`으로 시작하는 특별한 커맨드들이 있습니다. 이들은 다음과 같습니다.

-   `.help`: 점 커맨드 도움말을 표시합니다.
-   `.editor`: 편집기 모드를 활성화하여 여러 줄의 자바스크립트 코드를 쉽게 작성할 수 있습니다. 이 모드에서 ctrl-D를 누르면 작성한 코드를 실행합니다.
-   `.break`: 여러 줄의 표현식을 입력 중일 때, `.break` 커맨드를 입력하면 추가 입력을 중단합니다. ctrl-C를 누르는 것과 동일합니다.
-   `.clear`: REPL 컨텍스트를 빈 객체로 초기화하고 현재 입력 중인 여러 줄 표현식을 모두 지웁니다.
-   `.load`: 현재 작업 디렉토리를 기준으로 자바스크립트 파일을 불러옵니다.
-   `.save`: REPL 세션에서 입력한 모든 내용을 파일로 저장합니다. (파일명을 지정해야 합니다.)
-   `.exit`: REPL을 종료합니다. (ctrl-C를 두 번 누르는 것과 동일합니다.)

REPL은 `.editor`를 호출하지 않아도 여러 줄의 문장을 입력 중인지 자동으로 인식합니다.

예를 들어, 다음과 같이 반복문을 입력하기 시작하면:

```javascript
[1, 2, 3].forEach(num => {
```

Shell SessionCopy to clipboard

`enter`를 누르면, REPL은 3개의 점(`...`)으로 시작하는 새로운 줄로 이동합니다. 이는 해당 블록을 계속 작업할 수 있음을 나타냅니다.

```javascript
... console.log(num)
... })
```

Shell SessionCopy to clipboard

줄 끝에 `.break`를 입력하면, 여러 줄 모드가 중단되고 해당 문장은 실행되지 않습니다.


### [JavaScript 파일에서 REPL 실행하기](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#run-repl-from-javascript-file)

JavaScript 파일에서 `repl`을 사용해 REPL을 불러올 수 있습니다.

```javascript
const repl = require('node:repl');
```

`repl` 변수를 사용해 다양한 작업을 수행할 수 있습니다. REPL 커맨드 프롬프트를 시작하려면 다음 코드를 입력하세요.

```javascript
repl.start();
```

이 파일을 커맨드라인에서 실행합니다.

```bash
node repl.js
```

REPL이 시작될 때 표시되는 문자열을 전달할 수 있습니다. 기본값은 '> '(뒤에 공백이 있음)이지만, 커스텀 프롬프트를 정의할 수도 있습니다.

```javascript
// 유닉스 스타일 프롬프트
const local = repl.start('$ ');
```

REPL을 종료할 때 메시지를 표시할 수도 있습니다.

```javascript
local.on('exit', () => {
  console.log('REPL 종료');
  process.exit();
});
```

REPL 모듈에 대해 더 자세히 알아보려면 [REPL 문서](https://nodejs.org/api/repl.html)를 참고하세요.


