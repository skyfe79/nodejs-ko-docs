# Node.js에서 커맨드라인 입력 받기

Node.js CLI 프로그램을 어떻게 대화형으로 만들 수 있을까요?

Node.js 버전 7부터는 [`readline` 모듈](https://nodejs.org/docs/latest-v22.x/api/readline.html)을 제공합니다. 이 모듈은 `process.stdin` 스트림과 같은 읽기 가능한 스트림에서 입력을 받아올 수 있습니다. Node.js 프로그램 실행 중에는 이 스트림이 터미널 입력으로 사용되며, 한 번에 한 줄씩 입력을 받습니다.

```javascript
const readline = require('node:readline');

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

rl.question(`이름이 뭐예요?`, name => {
  console.log(`안녕하세요, ${name}님!`);
  rl.close();
});
```

이 코드는 사용자의 **이름**을 묻고, 텍스트를 입력한 후 엔터를 누르면 인사말을 출력합니다.

`question()` 메서드는 첫 번째 인자로 받은 질문을 표시하고 사용자 입력을 기다립니다. 엔터를 누르면 콜백 함수를 호출합니다.

이 콜백 함수에서는 readline 인터페이스를 닫습니다.

`readline`은 여러 가지 다른 메서드도 제공합니다. 위에 링크된 패키지 문서에서 확인해 보세요.

비밀번호를 요구해야 한다면, 입력된 내용을 그대로 표시하지 않고 `*` 기호로 표시하는 것이 좋습니다.


