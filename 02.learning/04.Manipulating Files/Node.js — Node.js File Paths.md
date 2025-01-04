# Table of Contents

- [Node.js 파일 경로](#nodejs-파일-경로)
  - [경로에서 정보 추출하기](#경로에서-정보-추출하기)
    - [예제](#예제)
  - [경로 다루기](#경로-다루기)

# [Node.js 파일 경로](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#nodejs-file-paths)

시스템의 모든 파일은 경로를 가지고 있습니다. Linux와 macOS에서는 `/users/joe/file.txt`와 같은 형태의 경로를 사용합니다. 반면 Windows 컴퓨터는 다르게 `C:\users\joe\file.txt`와 같은 구조를 가집니다.

애플리케이션에서 경로를 사용할 때는 이러한 차이를 고려해야 하므로 주의가 필요합니다.

파일에서 이 모듈을 포함시키려면 `const path = require('node:path');`를 사용하면 됩니다. 이후 해당 모듈의 메서드를 사용할 수 있습니다.


## 경로에서 정보 추출하기

주어진 경로에서 다음과 같은 메서드를 사용해 정보를 추출할 수 있습니다:

- `dirname`: 파일의 상위 폴더를 가져옵니다.
- `basename`: 파일 이름 부분을 가져옵니다.
- `extname`: 파일 확장자를 가져옵니다.


### [예제](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#example)

CJSMJS

```javascript
const path = require('node:path');

const notes = '/users/joe/notes.txt';

path.dirname(notes); // /users/joe
path.basename(notes); // notes.txt
path.extname(notes); // .txt
```

JavaScriptCopy to clipboard

`basename` 함수에 두 번째 인자를 지정하면 확장자를 제외한 파일 이름을 얻을 수 있습니다:

```javascript
path.basename(notes, path.extname(notes)); // notes
```

JavaScriptCopy to clipboard


## 경로 다루기

`path.join()`을 사용하면 두 개 이상의 경로를 합칠 수 있습니다.

```javascript
const name = 'joe';
path.join('/', 'users', name, 'notes.txt'); // '/users/joe/notes.txt'
```

`path.resolve()`를 사용하면 상대 경로의 절대 경로를 계산할 수 있습니다.

```javascript
path.resolve('joe.txt'); // '/Users/joe/joe.txt' (홈 폴더에서 실행한 경우)
```

이 경우 Node.js는 현재 작업 디렉토리에 `/joe.txt`를 추가합니다. 두 번째 매개변수로 폴더를 지정하면, `resolve`는 첫 번째 매개변수를 두 번째 매개변수의 기준으로 사용합니다.

```javascript
path.resolve('tmp', 'joe.txt'); // '/Users/joe/tmp/joe.txt' (홈 폴더에서 실행한 경우)
```

첫 번째 매개변수가 슬래시(`/`)로 시작하면 절대 경로로 간주합니다.

```javascript
path.resolve('/etc', 'joe.txt'); // '/etc/joe.txt'
```

`path.normalize()`는 상대 경로 지정자(`.` 또는 `..`)나 이중 슬래시(`//`)가 포함된 경우 실제 경로를 계산하는 데 유용합니다.

```javascript
path.normalize('/users/joe/..//test.txt'); // '/users/test.txt'
```

**`resolve`와 `normalize`는 경로가 실제로 존재하는지 확인하지 않습니다.** 단지 주어진 정보를 바탕으로 경로를 계산할 뿐입니다.


