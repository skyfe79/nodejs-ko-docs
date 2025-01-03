# [비동기 흐름 제어](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#asynchronous-flow-control)

> 이 글의 내용은 [Mixu's Node.js Book](http://book.mixu.net/node/ch7.html)에서 많은 영감을 받았습니다.

JavaScript의 핵심은 "메인" 스레드에서 논블로킹(non-blocking)으로 동작하도록 설계되었다는 점입니다. 이 메인 스레드는 뷰가 렌더링되는 곳입니다. 브라우저에서 이 기능이 얼마나 중요한지 상상할 수 있을 겁니다. 메인 스레드가 블로킹되면 사용자가 두려워하는 "프리징" 현상이 발생하고, 다른 이벤트가 전달되지 않아 데이터 획득이 중단되는 등의 문제가 생깁니다.

이러한 문제는 함수형 프로그래밍 스타일만이 해결할 수 있는 독특한 제약을 만듭니다. 여기서 콜백이 등장합니다.

하지만 콜백은 더 복잡한 절차에서 다루기 어려워질 수 있습니다. 이는 종종 "콜백 지옥"을 초래하는데, 여러 중첩된 콜백 함수가 코드를 읽기, 디버깅, 조직화 등을 더 어렵게 만듭니다.

```javascript
async1(function (input, result1) {
  async2(function (result2) {
    async3(function (result3) {
      async4(function (result4) {
        async5(function (output) {
          // output으로 무언가를 수행
        });
      });
    });
  });
});
```

물론 실제로는 `result1`, `result2` 등을 처리하기 위한 추가 코드가 있을 가능성이 높기 때문에, 이 문제의 길이와 복잡성은 위 예제보다 훨씬 더 지저분한 코드를 만들어냅니다.

**이때 *함수*가 큰 역할을 합니다. 더 복잡한 작업은 여러 함수로 구성됩니다:**

1. 시작 스타일 / 입력
2. 미들웨어
3. 종료자

**"시작 스타일 / 입력"은 시퀀스의 첫 번째 함수입니다. 이 함수는 작업을 위한 원본 입력을 받습니다. 작업은 실행 가능한 일련의 함수이며, 원본 입력은 주로 다음과 같습니다:**

1. 전역 환경의 변수
2. 인자 유무와 관계없이 직접 호출
3. 파일 시스템 또는 네트워크 요청으로 얻은 값

네트워크 요청은 외부 네트워크, 동일 네트워크의 다른 애플리케이션, 또는 동일 또는 외부 네트워크의 애플리케이션 자체에 의해 시작된 요청일 수 있습니다.

미들웨어 함수는 다른 함수를 반환하고, 종료자 함수는 콜백을 호출합니다. 다음은 네트워크 또는 파일 시스템 요청의 흐름을 보여줍니다. 여기서는 모든 값이 메모리에 있기 때문에 지연 시간이 0입니다.

```javascript
function final(someInput, callback) {
  callback(`${someInput} 그리고 콜백 실행으로 종료`);
}

function middleware(someInput, callback) {
  return final(`${someInput} 미들웨어에 의해 처리됨`, callback);
}

function initiate() {
  const someInput = '안녕하세요, 이건 함수입니다 ';
  middleware(someInput, function (result) {
    console.log(result);
    // 결과를 반환하려면 콜백이 필요함
  });
}

initiate();
```


## [상태 관리](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control#state-management)

함수는 상태에 의존할 수도 있고, 그렇지 않을 수도 있습니다. 상태 의존성은 함수의 입력값이나 다른 변수가 외부 함수에 의존할 때 발생합니다.

**상태 관리를 위한 두 가지 주요 전략은 다음과 같습니다:**

1. 변수를 직접 함수에 전달하는 방법
2. 캐시, 세션, 파일, 데이터베이스, 네트워크 또는 기타 외부 소스에서 변수 값을 가져오는 방법

여기서 전역 변수는 언급하지 않았습니다. 전역 변수를 사용해 상태를 관리하는 것은 종종 지저분한 안티 패턴이며, 상태를 보장하기 어렵거나 불가능하게 만듭니다. 복잡한 프로그램에서는 가능한 한 전역 변수를 피하는 것이 좋습니다.


## 제어 흐름

메모리에 객체가 존재하면 반복이 가능하며, 제어 흐름에 변화가 없습니다.

```javascript
function getSong() {
  let _song = '';
  let i = 100;
  for (i; i > 0; i -= 1) {
    _song += `${i} beers on the wall, you take one down and pass it around, ${
      i - 1
    } bottles of beer on the wall\n`;
    if (i === 1) {
      _song += "Hey let's get some more beer";
    }
  }

  return _song;
}

function singSong(_song) {
  if (!_song) throw new Error("song is '' empty, FEED ME A SONG!");
  console.log(_song);
}

const song = getSong();
// 정상 동작
singSong(song);
```

그러나 데이터가 메모리 외부에 존재하면 반복이 작동하지 않습니다.

```javascript
function getSong() {
  let _song = '';
  let i = 100;
  for (i; i > 0; i -= 1) {
    /* eslint-disable no-loop-func */
    setTimeout(function () {
      _song += `${i} beers on the wall, you take one down and pass it around, ${
        i - 1
      } bottles of beer on the wall\n`;
      if (i === 1) {
        _song += "Hey let's get some more beer";
      }
    }, 0);
    /* eslint-enable no-loop-func */
  }

  return _song;
}

function singSong(_song) {
  if (!_song) throw new Error("song is '' empty, FEED ME A SONG!");
  console.log(_song);
}

const song = getSong('beer');
// 동작하지 않음
singSong(song);
// Uncaught Error: song is '' empty, FEED ME A SONG!
```

왜 이런 일이 발생했을까요? `setTimeout`은 CPU에게 명령을 버스의 다른 곳에 저장하고 나중에 데이터를 가져오도록 지시합니다. 수천 번의 CPU 사이클이 지난 후 0밀리초 시점에 함수가 다시 실행되면, CPU는 버스에서 명령을 가져와 실행합니다. 문제는 `song('')`이 수천 사이클 전에 반환되었다는 점입니다.

파일 시스템과 네트워크 요청을 다룰 때도 같은 상황이 발생합니다. 메인 스레드는 불확실한 시간 동안 블로킹될 수 없기 때문에, 콜백을 사용해 코드 실행을 제어된 방식으로 스케줄링합니다.

다음 세 가지 패턴으로 거의 모든 작업을 수행할 수 있습니다.

1. **순차 실행:** 함수가 엄격한 순서대로 실행됩니다. `for` 루프와 가장 유사합니다.

```javascript
// 다른 곳에서 정의된 작업들
const operations = [
  { func: function1, args: args1 },
  { func: function2, args: args2 },
  { func: function3, args: args3 },
];

function executeFunctionWithArgs(operation, callback) {
  // 함수 실행
  const { args, func } = operation;
  func(args, callback);
}

function serialProcedure(operation) {
  if (!operation) process.exit(0); // 종료
  executeFunctionWithArgs(operation, function (result) {
    // 콜백 후 계속
    serialProcedure(operations.shift());
  });
}

serialProcedure(operations.shift());
```

2. **완전 병렬:** 순서가 중요하지 않은 경우, 예를 들어 1,000,000명의 이메일 수신자에게 메일을 보낼 때 사용합니다.

```javascript
let count = 0;
let success = 0;
const failed = [];
const recipients = [
  { name: 'Bart', email: 'bart@tld' },
  { name: 'Marge', email: 'marge@tld' },
  { name: 'Homer', email: 'homer@tld' },
  { name: 'Lisa', email: 'lisa@tld' },
  { name: 'Maggie', email: 'maggie@tld' },
];

function dispatch(recipient, callback) {
  // `sendEmail`은 가상의 SMTP 클라이언트
  sendMail(
    {
      subject: 'Dinner tonight',
      message: 'We have lots of cabbage on the plate. You coming?',
      smtp: recipient.email,
    },
    callback
  );
}

function final(result) {
  console.log(`Result: ${result.count} attempts \
      & ${result.success} succeeded emails`);
  if (result.failed.length)
    console.log(`Failed to send to: \
        \n${result.failed.join('\n')}\n`);
}

recipients.forEach(function (recipient) {
  dispatch(recipient, function (err) {
    if (!err) {
      success += 1;
    } else {
      failed.push(recipient.name);
    }
    count += 1;

    if (count === recipients.length) {
      final({
        count,
        success,
        failed,
      });
    }
  });
});
```

3. **제한 병렬:** 병렬 실행에 제한을 두는 경우, 예를 들어 1천만 명의 사용자 중 1,000,000명에게 이메일을 보낼 때 사용합니다.

```javascript
let successCount = 0;

function final() {
  console.log(`dispatched ${successCount} emails`);
  console.log('finished');
}

function dispatch(recipient, callback) {
  // `sendEmail`은 가상의 SMTP 클라이언트
  sendMail(
    {
      subject: 'Dinner tonight',
      message: 'We have lots of cabbage on the plate. You coming?',
      smtp: recipient.email,
    },
    callback
  );
}

function sendOneMillionEmailsOnly() {
  getListOfTenMillionGreatEmails(function (err, bigList) {
    if (err) throw err;

    function serial(recipient) {
      if (!recipient || successCount >= 1000000) return final();
      dispatch(recipient, function (_err) {
        if (!_err) successCount += 1;
        serial(bigList.pop());
      });
    }

    serial(bigList.pop());
  });
}

sendOneMillionEmailsOnly();
```

각 패턴은 고유의 사용 사례와 장단점이 있습니다. 가장 중요한 것은 작업을 모듈화하고 콜백을 사용하는 것입니다. 의심이 든다면 모든 것을 미들웨어처럼 다루세요!


