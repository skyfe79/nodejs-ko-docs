# Node.js에서 환경 변수 읽는 방법

Node.js의 `process` 코어 모듈은 프로세스가 시작될 때 설정된 모든 환경 변수를 담고 있는 `env` 속성을 제공합니다.

아래 코드는 `app.js`를 실행하면서 `USER_ID`와 `USER_KEY`를 설정합니다.

```bash
USER_ID=239482 USER_KEY=foobar node app.js
```

이 명령어는 `USER_ID`를 **239482**로, `USER_KEY`를 **foobar**로 전달합니다. 이 방법은 테스트에 적합하지만, 프로덕션 환경에서는 일반적으로 bash 스크립트를 사용해 변수를 내보내는 방식을 사용합니다.

> 참고: `process`는 Node.js의 전역 객체이기 때문에 별도로 임포트할 필요가 없습니다.

위 코드에서 설정한 `USER_ID`와 `USER_KEY` 환경 변수에 접근하는 예제는 다음과 같습니다.

```javascript
process.env.USER_ID; // "239482"
process.env.USER_KEY; // "foobar"
```

이와 같은 방식으로 여러분이 설정한 커스텀 환경 변수에도 접근할 수 있습니다.

Node.js 20에서는 **실험적**으로 [.env 파일 지원](https://nodejs.org/dist/latest-v22.x/docs/api/cli.html#--env-fileconfig)이 추가되었습니다.

이제 Node.js 애플리케이션을 실행할 때 `--env-file` 플래그를 사용해 환경 파일을 지정할 수 있습니다. 다음은 `.env` 파일 예제와 `process.env`를 사용해 해당 변수에 접근하는 방법입니다.

```bash
# .env 파일
PORT=3000
```

js 파일에서는 다음과 같이 접근합니다.

```javascript
process.env.PORT; // "3000"
```

`.env` 파일에 설정된 환경 변수를 사용해 `app.js` 파일을 실행합니다.

```bash
node --env-file=.env app.js
```

이 명령어는 `.env` 파일에서 모든 환경 변수를 로드하여 `process.env`를 통해 애플리케이션에서 사용할 수 있게 합니다.

또한, 여러 개의 `--env-file` 인자를 전달할 수 있습니다. 이후에 지정된 파일은 이전 파일에서 정의된 변수를 덮어씁니다.

```bash
node --env-file=.env --env-file=.development.env app.js
```

> 참고: 환경 변수와 파일에 동일한 변수가 정의된 경우, 환경 변수의 값이 우선합니다.

`.env` 파일을 선택적으로 읽고 싶다면, 파일이 없을 때 오류를 발생시키지 않도록 `--env-file-if-exists` 플래그를 사용할 수 있습니다.

```bash
node --env-file-if-exists=.env app.js
```


