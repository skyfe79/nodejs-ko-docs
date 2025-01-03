# Node-API 버전과 비 Node-API 버전 패키지를 함께 배포하는 방법

다음은 `iotivity-node` 패키지를 예시로 설명합니다.

## 비 Node-API 버전 배포
1. `package.json` 파일에서 버전을 업데이트합니다. `iotivity-node`의 경우 버전을 `1.2.0-2`로 변경합니다.
2. 릴리스 체크리스트를 확인합니다. (테스트, 데모, 문서가 정상인지 확인)
3. `npm publish` 명령어를 실행하여 배포합니다.

## Node-API 버전 배포
1. `package.json` 파일에서 버전을 업데이트합니다. `iotivity-node`의 경우 버전을 `1.2.0-3`로 변경합니다. 버전 관리 시 [semver.org](https://semver.org/#spec-item-9)에서 설명한 사전 릴리스 버전 스키마를 따르는 것을 권장합니다. 예를 들어 `1.2.0-napi`와 같은 형식을 사용합니다.
2. 릴리스 체크리스트를 확인합니다. (테스트, 데모, 문서가 정상인지 확인)
3. `npm publish --tag n-api` 명령어를 실행하여 배포합니다.

이 예제에서 `n-api` 태그를 사용해 릴리스하면, 비록 버전 `1.2.0-3`이 비 Node-API 버전(`1.2.0-2`)보다 나중에 배포되었더라도, 사용자가 `npm install iotivity-node`를 실행하면 기본적으로 비 Node-API 버전이 설치됩니다. Node-API 버전을 설치하려면 `npm install iotivity-node@n-api`를 실행해야 합니다. npm 태그 사용에 대한 자세한 내용은 ["Using dist-tags"](https://docs.npmjs.com/getting-started/using-tags)를 참고하세요.


## Node-API 버전 패키지 의존성 추가하기

`iotivity-node`의 Node-API 버전을 의존성으로 추가하려면 `package.json` 파일을 다음과 같이 작성합니다.

```json
"dependencies": {
  "iotivity-node": "n-api"
}
```

> ["dist-tags 사용하기"](https://docs.npmjs.com/getting-started/using-tags)에서 설명한 것처럼, 일반 버전과 달리 태그가 지정된 버전은 `package.json` 내에서 `"^2.0.0"`과 같은 버전 범위로 지정할 수 없습니다. 이는 태그가 정확히 하나의 버전을 가리키기 때문입니다. 따라서 패키지 관리자가 동일한 태그로 나중 버전을 태그하면, `npm update`를 통해 나중 버전을 받게 됩니다. 최신 버전이 아닌 특정 버전을 사용해야 한다면, `package.json` 의존성은 다음과 같이 정확한 버전을 참조해야 합니다.

```json
"dependencies": {
  "iotivity-node": "1.2.0-3"
}
```


