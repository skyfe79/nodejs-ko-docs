# Table of Contents

- [스트림 사용 방법](#스트림-사용-방법)
  - [Node.js 스트림이란?](#nodejs-스트림이란)
    - [이벤트 기반 아키텍처(Event-Driven Architecture)](#이벤트-기반-아키텍처event-driven-architecture)
  - [[스트림을 사용하는 이유](https://nodejs.org/en/learn/modules/publishing-a-package#why-use-streams)](#스트림을-사용하는-이유httpsnodejsorgenlearnmodulespublishing-a-packagewhy-use-streams)
    - [[성능 관련 참고 사항](https://nodejs.org/en/learn/modules/publishing-a-package#note-on-performance)](#성능-관련-참고-사항httpsnodejsorgenlearnmodulespublishing-a-packagenote-on-performance)
  - [[스트림 역사](https://nodejs.org/en/learn/modules/publishing-a-package#stream-history)](#스트림-역사httpsnodejsorgenlearnmodulespublishing-a-packagestream-history)
    - [[Streams 0](https://nodejs.org/en/learn/modules/publishing-a-package#streams-0)](#streams-0httpsnodejsorgenlearnmodulespublishing-a-packagestreams-0)
    - [[Streams 1 (클래식)](https://nodejs.org/en/learn/modules/publishing-a-package#streams-1-classic)](#streams-1-클래식httpsnodejsorgenlearnmodulespublishing-a-packagestreams-1-classic)
    - [[Streams 2](https://nodejs.org/en/learn/modules/publishing-a-package#streams-2)](#streams-2httpsnodejsorgenlearnmodulespublishing-a-packagestreams-2)
    - [[Streams 3](https://nodejs.org/en/learn/modules/publishing-a-package#streams-3)](#streams-3httpsnodejsorgenlearnmodulespublishing-a-packagestreams-3)
  - [[Stream types](https://nodejs.org/en/learn/modules/publishing-a-package#stream-types)](#stream-typeshttpsnodejsorgenlearnmodulespublishing-a-packagestream-types)
    - [[Readable](https://nodejs.org/en/learn/modules/publishing-a-package#readable)](#readablehttpsnodejsorgenlearnmodulespublishing-a-packagereadable)
      - [주요 메서드와 이벤트](#주요-메서드와-이벤트)
      - [[기본적인 읽기 가능한 스트림](https://nodejs.org/en/learn/modules/publishing-a-package#basic-readable-stream)](#기본적인-읽기-가능한-스트림httpsnodejsorgenlearnmodulespublishing-a-packagebasic-readable-stream)
      - [[readable 이벤트를 활용한 고급 제어](https://nodejs.org/en/learn/modules/publishing-a-package#advanced-control-with-the-readable-event)](#readable-이벤트를-활용한-고급-제어httpsnodejsorgenlearnmodulespublishing-a-packageadvanced-control-with-the-readable-event)
    - [[Writable](https://nodejs.org/en/learn/modules/publishing-a-package#writable)](#writablehttpsnodejsorgenlearnmodulespublishing-a-packagewritable)
      - [Writable Streams의 주요 메서드와 이벤트](#writable-streams의-주요-메서드와-이벤트)
      - [Writable 생성하기](#writable-생성하기)
    - [[Duplex](https://nodejs.org/en/learn/modules/publishing-a-package#duplex)](#duplexhttpsnodejsorgenlearnmodulespublishing-a-packageduplex)
      - [듀플렉스 스트림의 주요 메서드와 이벤트](#듀플렉스-스트림의-주요-메서드와-이벤트)
    - [[Transform](https://nodejs.org/en/learn/modules/publishing-a-package#transform)](#transformhttpsnodejsorgenlearnmodulespublishing-a-packagetransform)
      - [Transform Streams의 주요 메서드와 이벤트](#transform-streams의-주요-메서드와-이벤트)
      - [[Transform Stream 생성하기](https://nodejs.org/en/learn/modules/publishing-a-package#creating-a-transform-stream)](#transform-stream-생성하기httpsnodejsorgenlearnmodulespublishing-a-packagecreating-a-transform-stream)
  - [스트림을 다루는 방법](#스트림을-다루는-방법)
    - [`.pipe()` 메서드](#pipe-메서드)
    - [[`pipeline()`](https://nodejs.org/en/learn/modules/publishing-a-package#pipeline)](#pipelinehttpsnodejsorgenlearnmodulespublishing-a-packagepipeline)
    - [[Async Iterators](https://nodejs.org/en/learn/modules/publishing-a-package#async-iterators)](#async-iteratorshttpsnodejsorgenlearnmodulespublishing-a-packageasync-iterators)
      - [스트림과 함께 비동기 이터레이터를 사용하는 장점](#스트림과-함께-비동기-이터레이터를-사용하는-장점)
    - [[Object 모드](https://nodejs.org/en/learn/modules/publishing-a-package#object-mode)](#object-모드httpsnodejsorgenlearnmodulespublishing-a-packageobject-mode)
    - [[백프레셔(Backpressure)](https://nodejs.org/en/learn/modules/publishing-a-package#backpressure)](#백프레셔backpressurehttpsnodejsorgenlearnmodulespublishing-a-packagebackpressure)
  - [스트림 vs 웹 스트림](#스트림-vs-웹-스트림)
    - [스트림과 웹 스트림의 상호 운용성](#스트림과-웹-스트림의-상호-운용성)

# 스트림 사용 방법

Node.js 애플리케이션에서 대량의 데이터를 다루는 것은 양날의 검과 같습니다. 방대한 양의 데이터를 처리할 수 있는 능력은 매우 유용하지만, 성능 병목 현상과 메모리 고갈을 초래할 수도 있습니다. 전통적으로 개발자들은 이 문제를 해결하기 위해 전체 데이터셋을 한 번에 메모리로 읽어들이는 방식을 사용했습니다. 이 방법은 작은 데이터셋에서는 직관적이지만, 대용량 데이터(예: 파일, 네트워크 요청 등)에서는 비효율적이고 리소스를 많이 소모합니다.

이때 Node.js의 **스트림**이 등장합니다. 스트림은 근본적으로 다른 접근 방식을 제공하며, 데이터를 점진적으로 처리하고 메모리 사용을 최적화할 수 있게 해줍니다. 데이터를 관리 가능한 청크 단위로 처리함으로써, 스트림은 가장 어려운 데이터셋도 효율적으로 다룰 수 있는 확장 가능한 애플리케이션을 구축할 수 있게 합니다. 흔히 말하듯, "스트림은 시간에 따른 배열"입니다.

이 가이드에서는 스트림의 개념, 역사, API를 간략히 살펴보고, 스트림을 사용하고 운영하는 방법에 대한 몇 가지 권장 사항을 제공합니다.


## Node.js 스트림이란?

Node.js 스트림은 애플리케이션에서 데이터 흐름을 관리하기 위한 강력한 추상화 도구입니다. 파일 읽기/쓰기나 네트워크 요청과 같이 대용량 데이터를 처리할 때 성능 저하 없이 효율적으로 작업할 수 있습니다.

스트림은 전체 데이터를 한 번에 메모리에 로드하는 방식과 다릅니다. 데이터를 작은 단위(chunk)로 나누어 처리하기 때문에 메모리 사용량을 크게 줄일 수 있습니다. Node.js의 모든 스트림은 [`EventEmitter`](https://nodejs.org/api/events.html#class-eventemitter) 클래스를 상속받아, 데이터 처리 과정에서 다양한 이벤트를 발생시킬 수 있습니다. 스트림은 읽기 전용, 쓰기 전용, 또는 둘 다 가능한 형태로 제공되어 다양한 데이터 처리 시나리오에 유연하게 대응할 수 있습니다.


### 이벤트 기반 아키텍처(Event-Driven Architecture)

Node.js는 이벤트 기반 아키텍처를 기반으로 동작하며, 이는 실시간 I/O 처리에 이상적입니다. 이는 입력이 가능한 즉시 소비하고, 애플리케이션이 출력을 생성하는 즉시 전송한다는 의미입니다. 스트림은 이러한 접근 방식과 자연스럽게 통합되어 지속적인 데이터 처리를 가능하게 합니다.

스트림은 주요 단계에서 이벤트를 발생시켜 이를 달성합니다. 이러한 이벤트에는 데이터 수신 신호([`data`](https://nodejs.org/api/stream.html#stream_event_data) 이벤트)와 스트림 완료 신호([`end`](https://nodejs.org/api/stream.html#event-end) 이벤트)가 포함됩니다. 개발자는 이러한 이벤트를 수신하고, 그에 맞춰 커스텀 로직을 실행할 수 있습니다. 이러한 이벤트 기반 특성은 외부 소스로부터의 데이터 처리에 스트림을 매우 효율적으로 만듭니다.


## [스트림을 사용하는 이유](https://nodejs.org/en/learn/modules/publishing-a-package#why-use-streams)

스트림은 다른 데이터 처리 방법에 비해 세 가지 주요 장점을 제공합니다:

- **메모리 효율성**: 스트림은 데이터를 점진적으로 처리하며, 전체 데이터셋을 메모리에 로드하지 않고 데이터를 청크 단위로 처리합니다. 이는 대용량 데이터를 다룰 때 메모리 사용량을 크게 줄이고 메모리 관련 성능 문제를 방지하는 데 큰 장점입니다.
- **응답 시간 개선**: 스트림은 데이터가 도착하는 즉시 처리할 수 있습니다. 데이터의 일부가 도착하면 전체 데이터셋을 기다리지 않고 바로 처리할 수 있습니다. 이는 지연 시간을 줄이고 애플리케이션의 전반적인 반응성을 향상시킵니다.
- **실시간 처리 확장성**: 데이터를 청크 단위로 처리함으로써 Node.js 스트림은 제한된 리소스로도 대량의 데이터를 효율적으로 처리할 수 있습니다. 이러한 확장성은 실시간으로 대량의 데이터를 처리해야 하는 애플리케이션에 이상적입니다.

이러한 장점들로 인해 스트림은 대용량 데이터셋이나 실시간 데이터 처리를 다루는 고성능, 확장 가능한 Node.js 애플리케이션을 구축하는 데 강력한 도구로 사용됩니다.


### [성능 관련 참고 사항](https://nodejs.org/en/learn/modules/publishing-a-package#note-on-performance)

여러분의 애플리케이션이 이미 모든 데이터를 메모리에 준비해 두었다면, 스트림을 사용하는 것은 불필요한 오버헤드와 복잡성을 추가하고 애플리케이션을 느리게 만들 수 있습니다.


## [스트림 역사](https://nodejs.org/en/learn/modules/publishing-a-package#stream-history)

이 섹션은 Node.js에서 스트림의 역사를 참조하기 위한 내용입니다. 여러분이 0.11.5 이전 버전(2013년)의 Node.js로 작성된 코드베이스를 다루지 않는 한, 이전 버전의 스트림 API를 접할 일은 거의 없습니다. 하지만 관련 용어들은 여전히 사용될 수 있습니다.


### [Streams 0](https://nodejs.org/en/learn/modules/publishing-a-package#streams-0)

Node.js가 처음 출시될 때 Streams의 첫 번째 버전도 함께 공개되었습니다. 당시에는 아직 Stream 클래스가 존재하지 않았지만, 다양한 모듈에서 스트림 개념을 활용하며 `read`와 `write` 함수를 구현했습니다. 데이터 흐름을 제어하기 위해 `util.pump()` 함수를 사용할 수 있었습니다.


### [Streams 1 (클래식)](https://nodejs.org/en/learn/modules/publishing-a-package#streams-1-classic)

2011년 Node v0.4.0이 출시되면서 Stream 클래스와 `pipe()` 메서드가 도입되었습니다.


### [Streams 2](https://nodejs.org/en/learn/modules/publishing-a-package#streams-2)

2012년 Node v0.10.0이 출시되면서 Streams 2가 공개되었습니다. 이 업데이트는 Readable, Writable, Duplex, Transform과 같은 새로운 스트림 서브클래스를 도입했습니다. 또한 `readable` 이벤트가 추가되었습니다. 이전 버전과의 호환성을 유지하기 위해, `data` 이벤트 리스너를 추가하거나 `pause()`, `resume()` 메서드를 호출하면 이전 모드로 전환할 수 있었습니다.


### [Streams 3](https://nodejs.org/en/learn/modules/publishing-a-package#streams-3)

2013년, Node v0.11.5에서 Streams 3가 출시되었습니다. 이 버전은 스트림이 `data`와 `readable` 이벤트 핸들러를 동시에 가질 때 발생하는 문제를 해결하기 위해 만들어졌습니다. 이를 통해 '현재' 모드와 '이전' 모드 중 하나를 선택할 필요가 없어졌습니다. Streams 3는 현재 Node.js에서 사용되는 스트림의 최신 버전입니다.


## [Stream types](https://nodejs.org/en/learn/modules/publishing-a-package#stream-types)





### [Readable](https://nodejs.org/en/learn/modules/publishing-a-package#readable)

[`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams)은 데이터 소스를 순차적으로 읽기 위해 사용하는 클래스입니다. Node.js API에서 `Readable` 스트림의 대표적인 예로는 파일을 읽을 때 사용하는 [`fs.ReadStream`](https://nodejs.org/api/fs.html#class-fsreadstream), HTTP 요청을 읽을 때 사용하는 [`http.IncomingMessage`](https://nodejs.org/api/http.html#class-httpincomingmessage), 그리고 표준 입력을 읽을 때 사용하는 [`process.stdin`](https://nodejs.org/api/process.html#processstdin)이 있습니다.


#### 주요 메서드와 이벤트

읽기 가능한 스트림은 데이터 처리에 대한 세밀한 제어를 가능하게 하는 여러 핵심 메서드와 이벤트를 사용하여 동작합니다:

-   **[`on('data')`](https://nodejs.org/api/stream.html#stream_event_data)**: 이 이벤트는 스트림에서 데이터를 사용할 수 있을 때마다 트리거됩니다. 스트림이 처리할 수 있는 한 빠르게 데이터를 밀어넣기 때문에 매우 빠르며, 높은 처리량이 필요한 시나리오에 적합합니다.
-   **[`on('end')`](https://nodejs.org/api/stream.html#event-end)**: 스트림에서 더 이상 읽을 데이터가 없을 때 발생합니다. 이 이벤트는 데이터 전달이 완료되었음을 나타냅니다. 스트림의 모든 데이터가 소비된 후에만 이 이벤트가 발생합니다.
-   **[`on('readable')`](https://nodejs.org/api/stream.html#event-readable)**: 이 이벤트는 스트림에서 읽을 데이터가 있거나 스트림의 끝에 도달했을 때 트리거됩니다. 필요할 때 더 제어된 데이터 읽기를 가능하게 합니다.
-   **[`on('close')`](https://nodejs.org/api/stream.html#event-close_1)**: 이 이벤트는 스트림과 그 하위 리소스가 닫혔을 때 발생하며, 더 이상 이벤트가 발생하지 않음을 나타냅니다.
-   **[`on('error')`](https://nodejs.org/api/stream.html#event-error_1)**: 이 이벤트는 처리 중 오류가 발생했을 때 언제든지 발생할 수 있습니다. 이 이벤트에 대한 핸들러를 사용하여 잡히지 않은 예외를 방지할 수 있습니다.

이러한 이벤트의 사용 예는 다음 섹션에서 확인할 수 있습니다.


#### [기본적인 읽기 가능한 스트림](https://nodejs.org/en/learn/modules/publishing-a-package#basic-readable-stream)

다음은 데이터를 동적으로 생성하는 간단한 읽기 가능한 스트림 구현 예제입니다.

```javascript
const { Readable } = require('node:stream');

class MyStream extends Readable {
  #count = 0;
  _read(size) {
    this.push(':-)');
    if (++this.#count === 5) {
      this.push(null);
    }
  }
}

const stream = new MyStream();

stream.on('data', chunk => {
  console.log(chunk.toString());
});
```

이 코드에서 `MyStream` 클래스는 `Readable`을 상속받고, `_read` 메서드를 오버라이드하여 내부 버퍼에 문자열 ":-)"을 추가합니다. 문자열을 다섯 번 추가한 후, `null`을 추가하여 스트림의 끝을 알립니다. `on('data')` 이벤트 핸들러는 받은 각 청크를 콘솔에 출력합니다.


#### [readable 이벤트를 활용한 고급 제어](https://nodejs.org/en/learn/modules/publishing-a-package#advanced-control-with-the-readable-event)

데이터 흐름을 더 세밀하게 제어하기 위해 `readable` 이벤트를 사용할 수 있습니다. 이 이벤트는 복잡하지만, 스트림에서 데이터를 읽는 시점을 명시적으로 제어할 수 있어 특정 애플리케이션에서 더 나은 성능을 제공합니다.

```javascript
const stream = new MyStream({
  highWaterMark: 1,
});

stream.on('readable', () => {
  console.count('>> readable event');
  let chunk;
  while ((chunk = stream.read()) !== null) {
    console.log(chunk.toString()); // 데이터 처리
  }
});
stream.on('end', () => console.log('>> end event'));
```

여기서 `readable` 이벤트는 필요할 때마다 스트림에서 데이터를 수동으로 가져오는 데 사용됩니다. `readable` 이벤트 핸들러 내부의 루프는 스트림 버퍼에서 데이터를 계속 읽다가 `null`을 반환하면 멈춥니다. 이는 버퍼가 일시적으로 비어 있거나 스트림이 종료되었음을 의미합니다. `highWaterMark`를 1로 설정하면 버퍼 크기가 작아져 `readable` 이벤트가 더 자주 발생하며, 데이터 흐름을 더 세밀하게 제어할 수 있습니다.

위 코드를 실행하면 다음과 같은 출력을 얻을 수 있습니다.

```
>> readable event: 1
:-):-)
:-)
:-)
:-)
>> readable event: 2
>> readable event: 3
>> readable event: 4
>> end event
```

이를 자세히 살펴보겠습니다. `on('readable')` 이벤트를 연결하면, `read()`를 처음 호출하여 `readable` 이벤트가 발생할 수 있도록 합니다. 이벤트가 발생한 후, `while` 루프의 첫 번째 반복에서 `read`를 호출합니다. 그래서 처음 두 개의 스마일리가 한 줄에 출력됩니다. 그 후, `null`이 반환될 때까지 `read`를 계속 호출합니다. 각 `read` 호출은 새로운 `readable` 이벤트를 발생시키지만, "flow" 모드(즉, `readable` 이벤트 사용)에서는 이벤트가 `nextTick`에 예약됩니다. 따라서 루프의 동기 코드가 끝난 후에 모든 이벤트가 발생합니다.

참고: `NODE_DEBUG=stream`을 사용하여 코드를 실행하면 각 `push` 후에 `emitReadable`이 트리거되는 것을 확인할 수 있습니다.

각 스마일리 전에 `readable` 이벤트가 호출되도록 하려면, `push`를 `setImmediate` 또는 `process.nextTick`으로 감싸면 됩니다.

```javascript
class MyStream extends Readable {
  #count = 0;
  _read(size) {
    setImmediate(() => {
      this.push(':-)');
      if (++this.#count === 5) {
        return this.push(null);
      }
    });
  }
}
```

이렇게 하면 다음과 같은 출력을 얻을 수 있습니다.

```
>> readable event: 1
:-)
>> readable event: 2
:-)
>> readable event: 3
:-)
>> readable event: 4
:-)
>> readable event: 5
:-)
>> readable event: 6
>> end event
```


### [Writable](https://nodejs.org/en/learn/modules/publishing-a-package#writable)

[`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams) 스트림은 파일 생성, 데이터 업로드, 또는 순차적으로 데이터를 출력하는 작업에 유용합니다. 읽기 가능한 스트림이 데이터의 출처를 제공한다면, Node.js의 쓰기 가능한 스트림은 데이터의 목적지 역할을 합니다. Node.js API에서 쓰기 가능한 스트림의 대표적인 예로는 [`fs.WriteStream`](https://nodejs.org/api/fs.html#class-fswritestream), [`process.stdout`](https://nodejs.org/api/process.html#processstdout), 그리고 [`process.stderr`](https://nodejs.org/api/process.html#processstderr)가 있습니다.


#### Writable Streams의 주요 메서드와 이벤트

- **[`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback)**: 이 메서드는 스트림에 데이터 청크를 쓰는 데 사용됩니다. 데이터를 정의된 한계(highWaterMark)까지 버퍼링하고, 더 많은 데이터를 즉시 쓸 수 있는지 여부를 나타내는 불리언 값을 반환합니다.
- **[`.end()`](https://nodejs.org/api/stream.html#writableendchunk-encoding-callback)**: 이 메서드는 데이터 쓰기 과정의 끝을 알립니다. 스트림에 쓰기 작업을 완료하고 필요한 경우 정리 작업을 수행하도록 신호를 보냅니다.


#### Writable 생성하기

다음은 들어오는 모든 데이터를 대문자로 변환한 후 표준 출력에 쓰는 Writable 스트림을 만드는 예제입니다.

```javascript
const { Writable } = require('node:stream');
const { once } = require('node:events');

class MyStream extends Writable {
  constructor() {
    super({ highWaterMark: 10 /* 10 bytes */ });
  }
  _write(data, encode, cb) {
    process.stdout.write(data.toString().toUpperCase() + '\n', cb);
  }
}
const stream = new MyStream();

for (let i = 0; i < 10; i++) {
  const waitDrain = !stream.write('hello');

  if (waitDrain) {
    console.log('>> wait drain');
    await once(stream, 'drain');
  }
}

stream.end('world');
```

이 코드에서 `MyStream`은 버퍼 용량(`highWaterMark`)이 10바이트인 커스텀 `Writable` 스트림입니다. 이 스트림은 `_write` 메서드를 오버라이드하여 데이터를 대문자로 변환한 후 출력합니다.

루프는 'hello'를 스트림에 10번 쓰려고 시도합니다. 버퍼가 가득 차면(`waitDrain`이 `true`가 됨), `drain` 이벤트를 기다린 후 계속 진행합니다. 이렇게 하면 스트림의 버퍼가 넘치지 않도록 보장합니다.

출력 결과는 다음과 같습니다.

```
HELLO
>> wait drain
HELLO
HELLO
>> wait drain
HELLO
HELLO
>> wait drain
HELLO
HELLO
>> wait drain
HELLO
HELLO
>> wait drain
HELLO
WORLD
```


### [Duplex](https://nodejs.org/en/learn/modules/publishing-a-package#duplex)

[`Duplex`](https://nodejs.org/api/stream.html#stream_duplex_and_transform_streams) 스트림은 읽기와 쓰기 인터페이스를 모두 구현합니다.


#### 듀플렉스 스트림의 주요 메서드와 이벤트

듀플렉스 스트림은 Readable과 Writable 스트림에 설명된 모든 메서드와 이벤트를 구현합니다.

듀플렉스 스트림의 좋은 예로는 `net` 모듈의 `Socket` 클래스가 있습니다:

```javascript
const net = require('node:net');

// TCP 서버 생성
const server = net.createServer(socket => {
  socket.write('서버에서 보내는 메시지!\n');

  socket.on('data', data => {
    console.log(`클라이언트가 보낸 메시지: ${data.toString()}`);
  });

  // 클라이언트 연결 종료 처리
  socket.on('end', () => {
    console.log('클라이언트 연결이 종료되었습니다.');
  });
});

// 8080 포트에서 서버 시작
server.listen(8080, () => {
  console.log('서버가 8080 포트에서 실행 중입니다.');
});
```

위 코드는 8080 포트에서 TCP 소켓을 열고, 연결된 클라이언트에게 `서버에서 보내는 메시지!`를 전송하며, 받은 데이터를 로그로 기록합니다.

```javascript
const net = require('node:net');

// localhost:8080 서버에 연결
const client = net.createConnection({ port: 8080 }, () => {
  client.write('클라이언트에서 보내는 메시지!\n');
});

client.on('data', data => {
  console.log(`서버가 보낸 메시지: ${data.toString()}`);
});

// 서버 연결 종료 처리
client.on('end', () => {
  console.log('서버와의 연결이 종료되었습니다.');
});
```

위 코드는 TCP 소켓에 연결하고, `클라이언트에서 보내는 메시지`를 전송하며, 받은 데이터를 로그로 기록합니다.


### [Transform](https://nodejs.org/en/learn/modules/publishing-a-package#transform)

[`Transform`](https://nodejs.org/api/stream.html#stream_duplex_and_transform_streams) 스트림은 **듀플렉스 스트림**으로, 입력을 기반으로 출력이 계산됩니다. 이름에서 알 수 있듯이, 이 스트림은 일반적으로 **읽기 가능한 스트림**과 **쓰기 가능한 스트림** 사이에 위치하여 데이터가 전달되는 동안 변환 작업을 수행합니다.


#### Transform Streams의 주요 메서드와 이벤트

Duplex Streams의 모든 메서드와 이벤트 외에도 다음과 같은 것이 있습니다:

-   **[`_transform`](https://nodejs.org/api/stream.html#transform_transformchunk-encoding-callback)**: 이 함수는 내부적으로 호출되어 읽기 가능한 부분과 쓰기 가능한 부분 사이의 데이터 흐름을 처리합니다. 이 함수는 애플리케이션 코드에서 호출해서는 안 됩니다.


#### [Transform Stream 생성하기](https://nodejs.org/en/learn/modules/publishing-a-package#creating-a-transform-stream)

새로운 Transform Stream을 생성하려면, `Transform` 생성자에 `options` 객체를 전달하면 됩니다. 이 객체에는 `transform` 함수가 포함되어 있으며, 이 함수는 `push` 메서드를 사용해 입력 데이터로부터 출력 데이터를 계산하는 방식을 처리합니다.

```javascript
const { Transform } = require('node:stream');

const upper = new Transform({
  transform: function (data, enc, cb) {
    this.push(data.toString().toUpperCase());
    cb();
  },
});
```

이 스트림은 모든 입력을 받아 대문자로 변환하여 출력합니다.


## 스트림을 다루는 방법

스트림을 다룰 때는 일반적으로 소스에서 데이터를 읽고, 목적지에 데이터를 쓰는 작업을 수행합니다. 이 과정에서 데이터를 변환해야 할 수도 있습니다. 다음 섹션에서는 이를 수행하는 다양한 방법을 살펴보겠습니다.


### `.pipe()` 메서드

[`.pipe()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_pipe_destination_options) 메서드는 읽기 가능한 스트림을 쓰기 가능한(또는 변환 가능한) 스트림에 연결합니다. 이 방법은 목표를 달성하는 간단한 방법처럼 보이지만, 모든 오류 처리를 프로그래머에게 위임하기 때문에 올바르게 처리하기 어렵습니다.

다음 예제는 현재 파일을 대문자로 변환하여 콘솔에 출력하려는 `pipe`를 보여줍니다.

```javascript
const fs = require('node:fs');
const { Transform } = require('node:stream');

let errorCount = 0;
const upper = new Transform({
  transform: function (data, enc, cb) {
    if (errorCount === 10) {
      return cb(new Error('BOOM!'));
    }
    errorCount++;
    this.push(data.toString().toUpperCase());
    cb();
  },
});

const readStream = fs.createReadStream(__filename, { highWaterMark: 1 });
const writeStream = process.stdout;

readStream.pipe(upper).pipe(writeStream);

readStream.on('close', () => {
  console.log('Readable stream closed');
});

upper.on('close', () => {
  console.log('Transform stream closed');
});

upper.on('error', err => {
  console.error('\nError in transform stream:', err.message);
});

writeStream.on('close', () => {
  console.log('Writable stream closed');
});
```

10개의 문자를 쓰고 나면, `upper`는 콜백에서 오류를 반환하며, 이로 인해 스트림이 닫힙니다. 하지만 다른 스트림들은 이 사실을 알지 못해 메모리 누수가 발생할 수 있습니다. 출력 결과는 다음과 같습니다.

```
CONST FS =
Error in transform stream: BOOM!
Transform stream closed
```


### [`pipeline()`](https://nodejs.org/en/learn/modules/publishing-a-package#pipeline)

`.pipe()` 메서드의 복잡성과 잠재적인 문제를 피하기 위해 대부분의 경우 [`pipeline()`](https://nodejs.org/api/stream.html#stream_stream_pipeline_streams_callback) 메서드를 사용하는 것이 좋습니다. 이 메서드는 스트림을 연결하는 더 안전하고 견고한 방법으로, 오류 처리와 정리 작업을 자동으로 수행합니다.

다음 예제는 `pipeline()`을 사용해 이전 예제의 문제를 방지하는 방법을 보여줍니다:

```javascript
const fs = require('node:fs');
const { Transform, pipeline } = require('node:stream');

let errorCount = 0;
const upper = new Transform({
  transform: function (data, enc, cb) {
    if (errorCount === 10) {
      return cb(new Error('BOOM!'));
    }
    errorCount++;
    this.push(data.toString().toUpperCase());
    cb();
  },
});

const readStream = fs.createReadStream(__filename, { highWaterMark: 1 });
const writeStream = process.stdout;

readStream.on('close', () => {
  console.log('Readable stream closed');
});

upper.on('close', () => {
  console.log('\nTransform stream closed');
});

writeStream.on('close', () => {
  console.log('Writable stream closed');
});

pipeline(readStream, upper, writeStream, err => {
  if (err) {
    return console.error('Pipeline error:', err.message);
  }
  console.log('Pipeline succeeded');
});
```

이 경우, 모든 스트림은 다음과 같은 출력과 함께 닫힙니다:

```
CONST FS =
Transform stream closed
Writable stream closed
Pipeline error: BOOM!
Readable stream closed
```

[`pipeline()`](https://nodejs.org/api/stream.html#stream_stream_pipeline_streams_callback) 메서드는 또한 [`async pipeline()`](https://nodejs.org/api/stream.html#streampipelinesource-transforms-destination-options) 버전도 제공합니다. 이 버전은 콜백을 받지 않고 대신 파이프라인이 실패할 경우 거부되는 Promise를 반환합니다.


### [Async Iterators](https://nodejs.org/en/learn/modules/publishing-a-package#async-iterators)

Async iterators는 Streams API와 상호작용하는 표준 방식으로 권장됩니다. 웹과 Node.js의 모든 스트림 기본 요소와 비교했을 때, async iterators는 이해하기 쉽고 사용하기 편리하며, 버그를 줄이고 유지보수하기 좋은 코드를 작성하는 데 도움을 줍니다. 최신 버전의 Node.js에서는 async iterators가 스트림과 상호작용하는 더 우아하고 가독성 높은 방식으로 등장했습니다. 이벤트 기반의 토대 위에 구축된 async iterators는 스트림 소비를 단순화하는 더 높은 수준의 추상화를 제공합니다.

Node.js에서 모든 읽기 가능한 스트림은 비동기 이터러블입니다. 이는 `for await...of` 구문을 사용해 스트림의 데이터가 사용 가능해질 때마다 반복할 수 있음을 의미합니다. 각 데이터 조각을 비동기 코드의 효율성과 간결함으로 처리할 수 있습니다.


#### 스트림과 함께 비동기 이터레이터를 사용하는 장점

스트림과 함께 비동기 이터레이터를 사용하면 비동기 데이터 흐름을 처리하는 것이 여러 가지 면에서 간편해집니다:

- **가독성 향상**: 코드 구조가 더 깔끔하고 읽기 쉬워지며, 특히 여러 비동기 데이터 소스를 다룰 때 유용합니다.
- **에러 처리**: 비동기 이터레이터는 일반 비동기 함수와 마찬가지로 `try/catch` 블록을 사용해 간단하게 에러를 처리할 수 있습니다.
- **흐름 제어**: 소비자가 다음 데이터를 기다리면서 흐름을 제어하기 때문에, 백프레셔가 자동으로 관리됩니다. 이는 메모리 사용과 처리를 더 효율적으로 만듭니다.

비동기 이터레이터는 읽기 가능한 스트림을 다룰 때 더 현대적이고 가독성이 높은 방법을 제공합니다. 특히 비동기 데이터 소스를 다루거나 데이터 처리를 순차적이고 루프 기반으로 선호할 때 유용합니다.

다음은 읽기 가능한 스트림과 함께 비동기 이터레이터를 사용하는 예제입니다:

```javascript
const fs = require('node:fs');
const { pipeline } = require('node:stream/promises');

await pipeline(
  fs.createReadStream(import.meta.filename),
  async function* (source) {
    for await (let chunk of source) {
      yield chunk.toString().toUpperCase();
    }
  },
  process.stdout
);
```

이 코드는 이전 예제와 동일한 결과를 달성하지만, 새로운 변환 스트림을 정의할 필요가 없습니다. 간결함을 위해 이전 예제의 에러는 제거되었습니다. 비동기 버전의 `pipeline`이 사용되었으며, 가능한 에러를 처리하기 위해 `try...catch` 블록으로 감싸는 것이 좋습니다.


### [Object 모드](https://nodejs.org/en/learn/modules/publishing-a-package#object-mode)

기본적으로 스트림은 문자열, [`Buffer`](https://nodejs.org/api/buffer.html), [`TypedArray`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray), 또는 [`DataView`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView)와 함께 동작합니다. 만약 이러한 타입과 다른 임의의 값(예: 객체)이 스트림에 푸시되면 `TypeError`가 발생합니다. 하지만 `objectMode` 옵션을 `true`로 설정하면 객체와 함께 동작할 수 있습니다. 이렇게 하면 스트림이 `null`을 제외한 모든 JavaScript 값을 처리할 수 있게 됩니다. 여기서 `null`은 스트림의 끝을 나타내는 신호로 사용됩니다. 즉, 읽기 가능한 스트림에서는 어떤 값이든 `push`하고 `read`할 수 있으며, 쓰기 가능한 스트림에서는 어떤 값이든 `write`할 수 있습니다.

```javascript
const { Readable } = require('node:stream');

const readable = Readable({
  objectMode: true,
  read() {
    this.push({ hello: 'world' });
    this.push(null);
  },
});
```

Object 모드에서 작업할 때는 `highWaterMark` 옵션이 바이트가 아닌 객체의 수를 나타낸다는 점을 기억해야 합니다.


### [백프레셔(Backpressure)](https://nodejs.org/en/learn/modules/publishing-a-package#backpressure)

스트림을 사용할 때, 데이터를 생산하는 쪽이 소비하는 쪽을 압도하지 않도록 주의해야 합니다. 이를 위해 Node.js API의 모든 스트림에는 백프레셔 메커니즘이 적용되어 있으며, 구현자는 이 동작을 유지할 책임이 있습니다.

데이터 버퍼가 [`highWaterMark`](https://nodejs.org/api/stream.html#stream_buffering)를 초과하거나 쓰기 큐가 현재 바쁜 상황에서는 [`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback)가 `false`를 반환합니다.

`false` 값이 반환되면 백프레셔 시스템이 작동합니다. 이 시스템은 들어오는 [`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams) 스트림이 데이터를 보내는 것을 일시 중지하고, 소비자가 다시 준비될 때까지 기다립니다. 데이터 버퍼가 비워지면 [`'drain'`](https://nodejs.org/api/stream.html#stream_event_drain) 이벤트가 발생하여 데이터 흐름이 재개됩니다.

백프레셔에 대해 더 깊이 이해하고 싶다면 [`백프레셔 가이드`](https://nodejs.org/en/learn/modules/backpressuring-in-streams)를 참고하세요.


## 스트림 vs 웹 스트림

스트림 개념은 Node.js에만 국한되지 않습니다. 사실 Node.js는 [`Web Streams`](https://nodejs.org/api/webstreams.html)라는 스트림 개념의 다른 구현을 가지고 있습니다. 이는 [`WHATWG Streams Standard`](https://streams.spec.whatwg.org/)를 구현한 것입니다. 두 개념은 비슷하지만, API가 다르고 직접 호환되지 않는다는 점을 알아두는 것이 중요합니다.

[`Web Streams`](https://nodejs.org/api/webstreams.html)는 [`ReadableStream`](https://nodejs.org/api/webstreams.html#class-readablestream), [`WritableStream`](https://nodejs.org/api/webstreams.html#class-writablestream), [`TransformStream`](https://nodejs.org/api/webstreams.html#class-transformstream) 클래스를 구현합니다. 이 클래스들은 Node.js의 [`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams), [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams), [`Transform`](https://nodejs.org/api/stream.html#stream_duplex_and_transform_streams) 스트림과 유사합니다.


### 스트림과 웹 스트림의 상호 운용성

Node.js는 웹 스트림과 Node.js 스트림 간 변환을 위한 유틸리티 함수를 제공합니다. 이 함수들은 각 스트림 클래스의 `toWeb`과 `fromWeb` 메서드로 구현되어 있습니다.

다음 예제는 [`Duplex`](https://nodejs.org/api/stream.html#stream_duplex_and_transform_streams) 클래스에서 읽기와 쓰기 스트림을 웹 스트림으로 변환하여 작업하는 방법을 보여줍니다:

```javascript
const { Duplex } = require('node:stream');

const duplex = Duplex({
  read() {
    this.push('world');
    this.push(null);
  },
  write(chunk, encoding, callback) {
    console.log('writable', chunk);
    callback();
  },
});

const { readable, writable } = Duplex.toWeb(duplex);
writable.getWriter().write('hello');

readable
  .getReader()
  .read()
  .then(result => {
    console.log('readable', result.value);
  });
```

이 헬퍼 함수들은 Node.js 모듈에서 웹 스트림을 반환하거나 그 반대의 경우에 유용합니다. 스트림을 일반적으로 사용할 때는 async 이터레이터를 통해 Node.js와 웹 스트림 모두와 원활하게 상호작용할 수 있습니다.

```javascript
const { pipeline } = require('node:stream/promises');

const { body } = await fetch('https://nodejs.org/api/stream.html');

await pipeline(
  body,
  new TextDecoderStream(),
  async function* (source) {
    for await (const chunk of source) {
      yield chunk.toString().toUpperCase();
    }
  },
  process.stdout
);
```

fetch의 body는 `ReadableStream<Uint8Array>`이므로, 문자열로 작업하기 위해 [`TextDecoderStream`](https://developer.mozilla.org/en-US/docs/Web/API/TextDecoderStream)이 필요합니다.

이 작업은 [Matteo Collina](https://github.com/mcollina)가 [Platformatic's Blog](https://blog.platformatic.dev/a-guide-to-reading-and-writing-nodejs-streams)에 게시한 내용을 기반으로 합니다.


