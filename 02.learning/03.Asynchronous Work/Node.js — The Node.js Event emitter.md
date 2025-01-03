# Node.js 이벤트 에미터

브라우저에서 자바스크립트를 사용해 본 적이 있다면, 사용자 상호작용이 이벤트를 통해 얼마나 많이 처리되는지 알 것입니다. 마우스 클릭, 키보드 버튼 누르기, 마우스 이동에 반응하기 등이 그 예입니다.

백엔드에서는 Node.js가 [`events` 모듈](https://nodejs.org/api/events.html)을 사용해 비슷한 시스템을 구축할 수 있는 옵션을 제공합니다. 특히 이 모듈은 `EventEmitter` 클래스를 제공하며, 이를 사용해 이벤트를 처리할 수 있습니다.

다음과 같이 초기화합니다:

```javascript
const EventEmitter = require('node:events');
const eventEmitter = new EventEmitter();
```

이 객체는 여러 메서드 중에서도 `on`과 `emit` 메서드를 제공합니다.

- `emit`: 이벤트를 발생시키는 데 사용
- `on`: 이벤트가 발생했을 때 실행될 콜백 함수를 추가하는 데 사용

예를 들어, `start` 이벤트를 만들고, 이벤트가 발생했을 때 콘솔에 로그를 출력하는 예제를 살펴보겠습니다:

```javascript
eventEmitter.on('start', () => {
  console.log('started');
});
```

이제 다음 코드를 실행하면:

```javascript
eventEmitter.emit('start');
```

이벤트 핸들러 함수가 실행되고, 콘솔에 로그가 출력됩니다.

이벤트 핸들러에 인자를 전달하려면 `emit()`에 추가 인자로 전달하면 됩니다:

```javascript
eventEmitter.on('start', number => {
  console.log(`started ${number}`);
});

eventEmitter.emit('start', 23);
```

여러 개의 인자를 전달할 수도 있습니다:

```javascript
eventEmitter.on('start', (start, end) => {
  console.log(`started from ${start} to ${end}`);
});

eventEmitter.emit('start', 1, 100);
```

`EventEmitter` 객체는 이벤트와 상호작용하기 위한 여러 메서드도 제공합니다. 예를 들어:

- `once()`: 한 번만 실행되는 리스너 추가
- `removeListener()` / `off()`: 이벤트 리스너 제거
- `removeAllListeners()`: 특정 이벤트의 모든 리스너 제거

이 메서드들에 대해 더 자세히 알아보려면 [공식 문서](https://nodejs.org/api/events.html)를 참고하세요.


