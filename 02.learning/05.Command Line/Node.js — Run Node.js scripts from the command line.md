# [커맨드라인에서 Node.js 스크립트 실행하기](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#run-nodejs-scripts-from-the-command-line)

Node.js 프로그램을 실행하는 일반적인 방법은 Node.js를 설치한 후 전역적으로 사용 가능한 `node` 커맨드를 실행하고 실행할 파일 이름을 전달하는 것입니다.

여러분의 주요 Node.js 애플리케이션 파일이 `app.js`라면, 다음과 같이 입력하여 실행할 수 있습니다:

```bash
node app.js
```

위에서 여러분은 명시적으로 쉘에게 `node`를 사용해 스크립트를 실행하라고 지시하고 있습니다. 이 정보를 JavaScript 파일에 ["shebang"](https://en.wikipedia.org/wiki/Shebang_(Unix)) 라인으로 포함시킬 수도 있습니다. "shebang"은 파일의 첫 번째 줄에 위치하며, OS에게 스크립트를 실행할 때 사용할 인터프리터를 알려줍니다. 아래는 JavaScript 파일의 첫 번째 줄 예제입니다:

```javascript
#!/usr/bin/node
```

위에서는 인터프리터의 절대 경로를 명시적으로 지정하고 있습니다. 모든 운영체제가 `node`를 bin 폴더에 가지고 있는 것은 아니지만, 모든 운영체제는 `env`를 가지고 있습니다. OS에게 `env`를 실행하고 `node`를 인자로 전달하도록 지시할 수 있습니다:

```javascript
#!/usr/bin/env node

// 여러분의 JavaScript 코드
```

shebang을 사용하려면 파일에 실행 권한이 있어야 합니다. `app.js`에 실행 권한을 부여하려면 다음 커맨드를 실행하세요:

```bash
chmod u+x app.js
```

커맨드를 실행할 때, `app.js` 파일이 있는 디렉토리에 있는지 확인하세요.


## 문자열을 파일 경로 대신 `node` 인자로 전달하기

문자열을 인자로 실행하려면 `-e` 또는 `--eval "script"`를 사용할 수 있습니다. 이 옵션은 뒤따르는 인자를 자바스크립트 코드로 평가합니다. REPL에 미리 정의된 모듈들도 스크립트에서 사용할 수 있습니다.

Windows의 cmd.exe에서는 작은따옴표(`'`)가 제대로 동작하지 않습니다. cmd.exe는 큰따옴표(`"`)만 인식합니다. 반면 Powershell이나 Git bash에서는 작은따옴표와 큰따옴표 모두 사용 가능합니다.

```javascript
node -e "console.log(123)"
```


## [애플리케이션 자동 재시작](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#restart-the-application-automatically)

Node.js V16부터 파일이 변경될 때 애플리케이션을 자동으로 재시작하는 내장 옵션이 추가되었습니다. 이 기능은 개발 과정에서 유용하게 사용할 수 있습니다. 이 기능을 사용하려면 Node.js에 `--watch` 플래그를 전달하면 됩니다.

```bash
node --watch app.js
```

파일을 변경하면 애플리케이션이 자동으로 재시작됩니다. [`--watch` 플래그 문서](https://nodejs.org/docs/latest-v22.x/api/cli.html#--watch)를 참고하세요.


## Node.js로 작업 실행하기

Node.js는 `package.json` 파일에 정의된 특정 명령어를 실행할 수 있는 내장 작업 실행기를 제공합니다. 이 기능은 테스트 실행, 프로젝트 빌드, 코드 린팅과 같은 반복적인 작업을 자동화하는 데 특히 유용합니다.


### `--run` 플래그 사용하기

[`--run`](https://nodejs.org/docs/latest-v22.x/api/cli.html#--run) 플래그를 사용하면 `package.json` 파일의 `scripts` 섹션에 정의된 명령어를 실행할 수 있습니다. 예를 들어, 다음과 같은 `package.json` 파일이 있다고 가정해 보겠습니다.

```json
{
  "type": "module",
  "scripts": {
    "start": "node app.js",
    "dev": "node --run -- --watch",
    "test": "node --test"
  }
}
```

이제 `--run` 플래그를 사용하여 `test` 스크립트를 실행할 수 있습니다.

```shell
node --run test
```


### 커맨드에 인자 전달하기

`package.json` 파일의 `scripts` 객체에 있는 `dev` 키를 설명해 보겠습니다.

`-- --another-argument` 구문은 커맨드에 인자를 전달할 때 사용됩니다. 이 경우 `--watch` 인자가 `dev` 스크립트로 전달됩니다.

```bash
node --run dev
```


### [환경 변수](https://nodejs.org/en/learn/command-line/run-nodejs-scripts-from-the-command-line#environment-variables)

`--run` 플래그는 여러분의 스크립트에 유용한 특정 환경 변수를 설정합니다:

-   `NODE_RUN_SCRIPT_NAME`: 실행 중인 스크립트의 이름입니다.
-   `NODE_RUN_PACKAGE_JSON_PATH`: 처리 중인 `package.json` 파일의 경로입니다.


### 의도적인 제한 사항

Node.js 태스크 러너는 `npm run`이나 `yarn run` 같은 다른 태스크 러너에 비해 의도적으로 더 제한적입니다. 이는 성능과 단순성에 초점을 맞추어 `pre`나 `post` 스크립트 실행 같은 기능을 생략했습니다. 이로 인해 간단한 작업에는 적합하지만 모든 사용 사례를 커버하지는 못할 수 있습니다.


