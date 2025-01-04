# Table of Contents

- [사용법과 예제](#사용법과-예제)
    - [사용법[#](https://nodejs.org/docs/latest/api/synopsis.html#usage)](#사용법httpsnodejsorgdocslatestapisynopsishtmlusage)
    - [예제[#](https://nodejs.org/docs/latest/api/synopsis.html#example)](#예제httpsnodejsorgdocslatestapisynopsishtmlexample)

# 사용법과 예제

### 사용법[#](https://nodejs.org/docs/latest/api/synopsis.html#usage)

`node [options] [V8 options] [script.js | -e "script" | - ] [arguments]`

자세한 내용은 [커맨드라인 옵션](https://nodejs.org/docs/latest/api/cli.html#options) 문서를 참고하세요.

### 예제[#](https://nodejs.org/docs/latest/api/synopsis.html#example)

Node.js로 작성된 [웹 서버](https://nodejs.org/docs/latest/api/http.html) 예제입니다. 이 서버는 `'Hello, World!'`로 응답합니다.

이 문서의 커맨드는 `$` 또는 `>`로 시작합니다. 이는 사용자의 터미널에서 어떻게 보이는지 재현하기 위함입니다. 실제로 `$`와 `>` 문자를 입력하지 마세요. 각 커맨드의 시작을 나타내기 위해 사용되었습니다.

`$` 또는 `>`로 시작하지 않는 줄은 이전 커맨드의 출력 결과를 보여줍니다.

먼저, Node.js를 다운로드하고 설치했는지 확인하세요. 자세한 설치 방법은 [패키지 매니저를 통한 Node.js 설치](https://nodejs.org/en/download/package-manager/)를 참고하세요.

이제 `projects`라는 빈 프로젝트 폴더를 만들고, 해당 폴더로 이동합니다.

Linux와 Mac:

```bash
mkdir ~/projects
cd ~/projects
```

Windows CMD:

```powershell
mkdir %USERPROFILE%\projects
cd %USERPROFILE%\projects
```

Windows PowerShell:

```powershell
mkdir $env:USERPROFILE\projects
cd $env:USERPROFILE\projects
```

다음으로, `projects` 폴더 안에 `hello-world.js`라는 새로운 소스 파일을 생성합니다.

원하는 텍스트 편집기로 `hello-world.js` 파일을 열고 다음 내용을 붙여넣습니다:

```js
const http = require('node:http');
const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello, World!\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

파일을 저장한 후, 터미널 창에서 `hello-world.js` 파일을 실행하려면 다음을 입력합니다:

```bash
node hello-world.js
```

터미널에 다음과 같은 출력이 나타납니다:

```console
Server running at http://127.0.0.1:3000/
```

이제, 원하는 웹 브라우저를 열고 `http://127.0.0.1:3000`에 접속합니다.

브라우저에 `Hello, World!` 문자열이 표시된다면, 서버가 정상적으로 작동하고 있는 것입니다.


