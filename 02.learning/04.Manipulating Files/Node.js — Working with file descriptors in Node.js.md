# Node.js에서 파일 디스크립터 다루기

파일 시스템에 있는 파일과 상호작용하려면 먼저 파일 디스크립터를 얻어야 합니다.

파일 디스크립터는 열린 파일을 참조하는 숫자(fd)로, `fs` 모듈의 `open()` 메서드를 사용해 파일을 열 때 반환됩니다. 이 숫자(fd)는 운영체제에서 열린 파일을 고유하게 식별합니다.

```javascript
const fs = require('node:fs');

fs.open('/Users/joe/test.txt', 'r', (err, fd) => {
  // fd는 파일 디스크립터입니다.
});
```

`fs.open()` 호출 시 두 번째 인자로 사용한 `r`을 주목하세요. 이 플래그는 파일을 읽기 위해 열겠다는 의미입니다.

**자주 사용하는 다른 플래그는 다음과 같습니다:**

| 플래그 | 설명 | 파일이 없을 때 생성 여부 |
| --- | --- | --- |
| r+ | 파일을 읽고 쓰기 위해 엽니다. | ❌ |
| w+ | 파일을 읽고 쓰기 위해 열고, 스트림을 파일의 시작 위치로 이동합니다. | ✅ |
| a | 파일을 쓰기 위해 열고, 스트림을 파일의 끝으로 이동합니다. | ✅ |
| a+ | 파일을 읽고 쓰기 위해 열고, 스트림을 파일의 끝으로 이동합니다. | ✅ |

콜백 대신 파일 디스크립터를 반환하는 `fs.openSync` 메서드를 사용해 파일을 열 수도 있습니다.

```javascript
const fs = require('node:fs');

try {
  const fd = fs.openSync('/Users/joe/test.txt', 'r');
} catch (err) {
  console.error(err);
}
```

파일 디스크립터를 얻은 후에는 `fs.close()` 호출이나 파일 시스템과 상호작용하는 다른 작업들을 수행할 수 있습니다.

또한 `fs/promises` 모듈에서 제공하는 Promise 기반의 `fsPromises.open` 메서드를 사용해 파일을 열 수도 있습니다.

`fs/promises` 모듈은 Node.js v14부터 사용 가능합니다. v14 이전, v10 이후 버전에서는 `require('fs').promises`를 사용할 수 있습니다. v10 이전, v8 이후 버전에서는 `util.promisify`를 사용해 `fs` 메서드를 Promise 기반 메서드로 변환할 수 있습니다.

```javascript
const fs = require('node:fs/promises');
// v14 이전에는 const fs = require('fs').promises를 사용합니다.
async function example() {
  let filehandle;
  try {
    filehandle = await fs.open('/Users/joe/test.txt', 'r');
    console.log(filehandle.fd);
    console.log(await filehandle.readFile({ encoding: 'utf8' }));
  } finally {
    if (filehandle) await filehandle.close();
  }
}
example();
```

`util.promisify` 사용 예제는 다음과 같습니다.

```javascript
const fs = require('node:fs');
const util = require('node:util');

async function example() {
  const open = util.promisify(fs.open);
  const fd = await open('/Users/joe/test.txt', 'r');
}
example();
```

`fs/promises` 모듈에 대한 자세한 내용은 [fs/promises API](https://nodejs.org/api/fs.html#promise-example)를 참고하세요.


