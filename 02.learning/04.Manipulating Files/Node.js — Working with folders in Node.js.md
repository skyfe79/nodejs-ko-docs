# Node.js에서 폴더 작업하기

Node.js의 `fs` 코어 모듈은 폴더 작업에 유용한 여러 메서드를 제공합니다.


## 폴더 존재 여부 확인하기

폴더가 존재하는지, 그리고 Node.js가 해당 폴더에 접근할 수 있는 권한이 있는지 확인하려면 `fs.access()` 또는 Promise 기반의 `fsPromises.access()`를 사용합니다.

```javascript
const fs = require('fs');
const fsPromises = require('fs').promises;

// 콜백 기반 방식
fs.access('/path/to/folder', fs.constants.F_OK, (err) => {
    if (err) {
        console.log('폴더가 존재하지 않거나 접근 권한이 없습니다.');
    } else {
        console.log('폴더가 존재하며 접근 가능합니다.');
    }
});

// Promise 기반 방식
async function checkFolderExists() {
    try {
        await fsPromises.access('/path/to/folder', fs.constants.F_OK);
        console.log('폴더가 존재하며 접근 가능합니다.');
    } catch (err) {
        console.log('폴더가 존재하지 않거나 접근 권한이 없습니다.');
    }
}

checkFolderExists();
```

`fs.constants.F_OK`는 파일이나 폴더의 존재 여부를 확인하는 플래그입니다. 추가로 `fs.constants.R_OK`, `fs.constants.W_OK`, `fs.constants.X_OK`를 사용해 읽기, 쓰기, 실행 권한을 확인할 수도 있습니다.


## 새로운 폴더 생성하기

새로운 폴더를 생성하려면 `fs.mkdir()`, `fs.mkdirSync()`, 또는 `fsPromises.mkdir()`을 사용할 수 있습니다.

```javascript
const fs = require('node:fs');

const folderName = '/Users/joe/test';

try {
  if (!fs.existsSync(folderName)) {
    fs.mkdirSync(folderName);
  }
} catch (err) {
  console.error(err);
}
```

이 코드는 지정된 경로에 폴더가 존재하지 않을 경우에만 폴더를 생성합니다. 만약 폴더 생성 중 오류가 발생하면, 해당 오류를 콘솔에 출력합니다.


## 디렉토리 내용 읽기

디렉토리의 내용을 읽기 위해 `fs.readdir()`, `fs.readdirSync()`, `fsPromises.readdir()`를 사용할 수 있습니다.

아래 코드는 폴더의 내용을 읽어 파일과 하위 폴더의 상대 경로를 반환합니다:

```javascript
const fs = require('node:fs');

const folderPath = '/Users/joe';

fs.readdirSync(folderPath);
```

전체 경로를 얻으려면 다음과 같이 작성합니다:

```javascript
fs.readdirSync(folderPath).map(fileName => {
  return path.join(folderPath, fileName);
});
```

폴더를 제외하고 파일만 반환하도록 결과를 필터링할 수도 있습니다:

```javascript
const fs = require('node:fs');

const isFile = fileName => {
  return fs.lstatSync(fileName).isFile();
};

fs.readdirSync(folderPath)
  .map(fileName => {
    return path.join(folderPath, fileName);
  })
  .filter(isFile);
```


## 폴더 이름 변경하기

폴더 이름을 변경하려면 `fs.rename()`, `fs.renameSync()`, `fsPromises.rename()`을 사용합니다. 첫 번째 인자는 현재 경로, 두 번째 인자는 새로운 경로입니다.

### 비동기 방식 (`fs.rename`)

```javascript
const fs = require('node:fs');

fs.rename('/Users/joe', '/Users/roger', err => {
  if (err) {
    console.error(err);
  }
  // 작업 완료
});
```

### 동기 방식 (`fs.renameSync`)

```javascript
const fs = require('node:fs');

try {
  fs.renameSync('/Users/joe', '/Users/roger');
} catch (err) {
  console.error(err);
}
```

### Promise 기반 방식 (`fsPromises.rename`)

```javascript
const fs = require('node:fs/promises');

async function example() {
  try {
    await fs.rename('/Users/joe', '/Users/roger');
  } catch (err) {
    console.log(err);
  }
}
example();
```


## 폴더 삭제하기

폴더를 삭제하려면 `fs.rmdir()`, `fs.rmdirSync()`, `fsPromises.rmdir()`을 사용합니다.

```javascript
const fs = require('node:fs');

fs.rmdir(dir, err => {
  if (err) {
    throw err;
  }

  console.log(`${dir}가 삭제되었습니다!`);
});
```

내용물이 있는 폴더를 삭제하려면 `fs.rm()`을 `{ recursive: true }` 옵션과 함께 사용하여 재귀적으로 내용물을 삭제합니다.

`{ recursive: true, force: true }`를 사용하면 폴더가 존재하지 않을 때 예외를 무시합니다.

```javascript
const fs = require('node:fs');

fs.rm(dir, { recursive: true, force: true }, err => {
  if (err) {
    throw err;
  }

  console.log(`${dir}가 삭제되었습니다!`);
});
```


