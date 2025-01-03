# [Writing files with Node.js](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#writing-files-with-nodejs)





## 파일 쓰기

Node.js에서 파일을 쓰는 가장 쉬운 방법은 `fs.writeFile()` API를 사용하는 것입니다.

```javascript
const fs = require('node:fs');

const content = 'Some content!';

fs.writeFile('/Users/joe/test.txt', content, err => {
  if (err) {
    console.error(err);
  } else {
    // 파일이 성공적으로 작성됨
  }
});
```


### 파일 동기적으로 쓰기

동기 방식으로 파일을 작성하려면 `fs.writeFileSync()`를 사용할 수 있습니다.

```javascript
const fs = require('node:fs');

const content = 'Some content!';

try {
  fs.writeFileSync('/Users/joe/test.txt', content);
  // 파일이 성공적으로 작성됨
} catch (err) {
  console.error(err);
}
```

또는 `fs/promises` 모듈에서 제공하는 Promise 기반의 `fsPromises.writeFile()` 메서드를 사용할 수도 있습니다.

```javascript
const fs = require('node:fs/promises');

async function example() {
  try {
    const content = 'Some content!';
    await fs.writeFile('/Users/joe/test.txt', content);
  } catch (err) {
    console.log(err);
  }
}

example();
```

기본적으로 이 API는 파일이 이미 존재할 경우 **파일 내용을 덮어씁니다**.

**플래그를 지정하여 기본 동작을 변경할 수 있습니다:**

```javascript
fs.writeFile('/Users/joe/test.txt', content, { flag: 'a+' }, err => {});
```


#### [자주 사용하는 플래그](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#the-flags-youll-likely-use-are)

| 플래그 | 설명 | 파일이 없을 때 생성 여부 |
| --- | --- | --- |
| `r+` | 파일을 **읽기**와 **쓰기**용으로 엽니다. | ❌ |
| `w+` | 파일을 **읽기**와 **쓰기**용으로 열고, 스트림을 파일의 **시작** 위치로 이동시킵니다. | ✅ |
| `a` | 파일을 **쓰기**용으로 열고, 스트림을 파일의 **끝** 위치로 이동시킵니다. | ✅ |
| `a+` | 파일을 **읽기**와 **쓰기**용으로 열고, 스트림을 파일의 **끝** 위치로 이동시킵니다. | ✅ |

-   더 자세한 정보는 [fs 문서](https://nodejs.org/api/fs.html#file-system-flags)에서 확인할 수 있습니다.


## 파일에 내용 추가하기

파일에 새로운 내용을 덮어쓰지 않고 추가하고 싶을 때, 파일에 내용을 추가하는 기능이 유용합니다.


파일 끝에 내용을 추가하는 유용한 방법으로 `fs.appendFile()`과 그 동기 버전인 `fs.appendFileSync()`가 있습니다.

```javascript
const fs = require('node:fs');

const content = 'Some content!';

fs.appendFile('file.log', content, err => {
  if (err) {
    console.error(err);
  } else {
    // 완료!
  }
});
```


#### [Promise를 사용한 예제](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#example-with-promises)

다음은 `fsPromises.appendFile()`을 사용한 예제입니다:

```javascript
const fs = require('node:fs/promises');

async function example() {
  try {
    const content = 'Some content!';
    await fs.appendFile('/Users/joe/test.txt', content);
  } catch (err) {
    console.log(err);
  }
}

example();
```


