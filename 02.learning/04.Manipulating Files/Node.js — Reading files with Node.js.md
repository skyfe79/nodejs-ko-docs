# Node.js에서 파일 읽기

Node.js에서 파일을 읽는 가장 간단한 방법은 `fs.readFile()` 메서드를 사용하는 것입니다. 이 메서드에 파일 경로, 인코딩, 그리고 파일 데이터(와 에러)를 받을 콜백 함수를 전달합니다.

```javascript
const fs = require('node:fs');

fs.readFile('/Users/joe/test.txt', 'utf8', (err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(data);
});
```

또는 동기 버전인 `fs.readFileSync()`를 사용할 수도 있습니다.

```javascript
const fs = require('node:fs');

try {
  const data = fs.readFileSync('/Users/joe/test.txt', 'utf8');
  console.log(data);
} catch (err) {
  console.error(err);
}
```

`fs/promises` 모듈에서 제공하는 Promise 기반의 `fsPromises.readFile()` 메서드를 사용할 수도 있습니다.

```javascript
const fs = require('node:fs/promises');

async function example() {
  try {
    const data = await fs.readFile('/Users/joe/test.txt', { encoding: 'utf8' });
    console.log(data);
  } catch (err) {
    console.log(err);
  }
}
example();
```

`fs.readFile()`, `fs.readFileSync()`, `fsPromises.readFile()` 세 가지 방법 모두 파일의 전체 내용을 메모리에 읽어들인 후 데이터를 반환합니다.

이 말은 큰 파일의 경우 메모리 사용량과 프로그램 실행 속도에 큰 영향을 미칠 수 있다는 것을 의미합니다.

이런 경우에는 스트림을 사용해 파일 내용을 읽는 것이 더 나은 선택입니다.


