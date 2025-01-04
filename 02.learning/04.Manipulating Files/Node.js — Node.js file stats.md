# Table of Contents

- [Node.js 파일 상태 정보](#nodejs-파일-상태-정보)

# [Node.js 파일 상태 정보](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#nodejs-file-stats)

모든 파일은 Node.js를 사용해 확인할 수 있는 세부 정보를 가지고 있습니다. 특히, [`fs` 모듈](https://nodejs.org/api/fs.html)이 제공하는 `stat()` 메서드를 사용하면 됩니다.

파일 경로를 인자로 전달하여 호출하면, Node.js가 파일 정보를 가져온 후 콜백 함수를 호출합니다. 이때 콜백 함수는 두 개의 매개변수를 받습니다: 에러 메시지와 파일 상태 정보입니다.

```javascript
const fs = require('node:fs');

fs.stat('/Users/joe/test.txt', (err, stats) => {
  if (err) {
    console.error(err);
  }
  // `stats`를 통해 파일 상태 정보에 접근할 수 있습니다.
});
```

Node.js는 또한 동기식 메서드를 제공합니다. 이 메서드는 파일 상태 정보가 준비될 때까지 스레드를 블로킹합니다.

```javascript
const fs = require('node:fs');

try {
  const stats = fs.statSync('/Users/joe/test.txt');
} catch (err) {
  console.error(err);
}
```

파일 정보는 `stats` 변수에 포함됩니다. `stats`를 사용해 어떤 종류의 정보를 추출할 수 있을까요?

**다음과 같은 정보를 얻을 수 있습니다:**

- `stats.isFile()`과 `stats.isDirectory()`를 사용해 파일이 디렉토리인지 파일인지 확인
- `stats.isSymbolicLink()`를 사용해 파일이 심볼릭 링크인지 확인
- `stats.size`를 사용해 파일 크기를 바이트 단위로 확인

다른 고급 메서드들도 있지만, 일상적인 프로그래밍에서 주로 사용하는 것은 이 정도입니다.

```javascript
const fs = require('node:fs');

fs.stat('/Users/joe/test.txt', (err, stats) => {
  if (err) {
    console.error(err);
    return;
  }

  stats.isFile(); // true
  stats.isDirectory(); // false
  stats.isSymbolicLink(); // false
  stats.size; // 1024000 //= 1MB
});
```

원한다면 `fs/promises` 모듈이 제공하는 Promise 기반의 `fsPromises.stat()` 메서드를 사용할 수도 있습니다.

```javascript
const fs = require('node:fs/promises');

async function example() {
  try {
    const stats = await fs.stat('/Users/joe/test.txt');
    stats.isFile(); // true
    stats.isDirectory(); // false
    stats.isSymbolicLink(); // false
    stats.size; // 1024000 //= 1MB
  } catch (err) {
    console.log(err);
  }
}
example();
```

`fs` 모듈에 대해 더 자세히 알아보려면 [공식 문서](https://nodejs.org/api/fs.html)를 참고하세요.


