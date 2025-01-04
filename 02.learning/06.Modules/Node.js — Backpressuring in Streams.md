# Table of Contents

- [스트림에서의 백프레셔](#스트림에서의-백프레셔)
  - [데이터 처리의 문제점](#데이터-처리의-문제점)
  - [데이터가 너무 빠르게 들어올 때](#데이터가-너무-빠르게-들어올-때)
  - [가비지 컬렉션의 과도한 부하](#가비지-컬렉션의-과도한-부하)
  - [메모리 소모](#메모리-소모)
  - [백프레셔가 이러한 문제를 어떻게 해결하나요?](#백프레셔가-이러한-문제를-어떻게-해결하나요)
  - [`.pipe()`의 생명주기](#pipe의-생명주기)
  - [백프레셔 가이드라인](#백프레셔-가이드라인)
  - [커스텀 스트림 구현 시 지켜야 할 규칙](#커스텀-스트림-구현-시-지켜야-할-규칙)
  - [Readable Streams에 특화된 규칙](#readable-streams에-특화된-규칙)
  - [Writable Streams에 특화된 규칙](#writable-streams에-특화된-규칙)
  - [결론](#결론)

# [스트림에서의 백프레셔](https://nodejs.org/en/learn/modules/publishing-a-package#backpressuring-in-streams)

데이터 처리 중 발생하는 일반적인 문제 중 하나는 [`백프레셔`](https://en.wikipedia.org/wiki/Backpressure_routing)입니다. 이는 데이터 전송 중 버퍼 뒤에 데이터가 쌓이는 현상을 설명합니다. 데이터를 받는 쪽에서 복잡한 작업을 수행하거나 어떤 이유로 느려지면, 들어오는 데이터가 마치 막힌 것처럼 쌓이게 됩니다.

이 문제를 해결하려면, 한 소스에서 다른 소스로 데이터가 원활하게 흐르도록 하는 위임 시스템이 필요합니다. 다양한 커뮤니티는 이 문제를 각자의 프로그램에 맞게 독창적으로 해결했는데, Unix 파이프와 TCP 소켓이 좋은 예시이며, 이를 흔히 *흐름 제어*라고 부릅니다. Node.js에서는 스트림이 이 문제를 해결하는 방법으로 채택되었습니다.

이 가이드의 목적은 백프레셔가 무엇인지, 그리고 Node.js의 소스 코드에서 스트림이 이를 어떻게 해결하는지 자세히 설명하는 것입니다. 가이드의 두 번째 부분에서는 스트림을 구현할 때 애플리케이션 코드가 안전하고 최적화되도록 권장하는 모범 사례를 소개합니다.

이 가이드를 읽기 전에 [`백프레셔`](https://en.wikipedia.org/wiki/Backpressure_routing), [`Buffer`](https://nodejs.org/api/buffer.html), 그리고 Node.js의 [`EventEmitters`](https://nodejs.org/api/events.html)에 대한 기본적인 이해가 있다고 가정합니다. 또한 [`Stream`](https://nodejs.org/api/stream.html)에 대한 경험이 어느 정도 있다고 가정합니다. 해당 문서를 아직 읽어보지 않았다면, API 문서를 먼저 살펴보는 것이 좋습니다. 이 가이드를 읽는 동안 이해를 넓히는 데 도움이 될 것입니다.


## 데이터 처리의 문제점

컴퓨터 시스템에서 데이터는 파이프, 소켓, 시그널 등을 통해 한 프로세스에서 다른 프로세스로 전달됩니다. Node.js에서는 이와 유사한 메커니즘인 [`Stream`](https://nodejs.org/api/stream.html)을 제공합니다. 스트림은 Node.js에서 매우 유용하며, 내부 코드베이스의 거의 모든 부분에서 이 모듈을 활용합니다. 개발자로서 여러분도 스트림을 적극적으로 사용하는 것을 권장합니다!

```javascript
const readline = require('node:readline');

// process.stdin과 process.stdout은 모두 Stream의 인스턴스입니다.
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

rl.question('스트림을 사용해야 하는 이유는 무엇인가요? ', answer => {
  console.log(`아마도 ${answer} 때문일 수도 있고, 스트림이 정말 멋지기 때문일 수도 있죠! :)`);

  rl.close();
});
```

스트림을 통해 구현된 백프레셔(backpressure) 메커니즘이 얼마나 훌륭한 최적화인지 보여주는 좋은 예제는 Node.js의 [`Stream`](https://nodejs.org/api/stream.html) 구현과 내부 시스템 도구를 비교해보면 알 수 있습니다.

한 가지 시나리오에서, 우리는 대용량 파일(약 9GB)을 가져와서 익숙한 [`zip(1)`](https://linux.die.net/man/1/zip) 도구를 사용해 압축합니다.

```bash
zip The.Matrix.1080p.mkv
```

이 작업이 완료되기까지 몇 분이 걸리는 동안, 다른 쉘에서 Node.js 모듈인 [`zlib`](https://nodejs.org/api/zlib.html)을 사용하는 스크립트를 실행할 수 있습니다. 이 모듈은 또 다른 압축 도구인 [`gzip(1)`](https://linux.die.net/man/1/gzip)을 감싸고 있습니다.

```javascript
const gzip = require('node:zlib').createGzip();
const fs = require('node:fs');

const inp = fs.createReadStream('The.Matrix.1080p.mkv');
const out = fs.createWriteStream('The.Matrix.1080p.mkv.gz');

inp.pipe(gzip).pipe(out);
```

결과를 테스트하기 위해 각각 압축된 파일을 열어보세요. [`zip(1)`](https://linux.die.net/man/1/zip) 도구로 압축된 파일은 파일이 손상되었다고 알려줄 것입니다. 반면에 [`Stream`](https://nodejs.org/api/stream.html)을 통해 완료된 압축 파일은 오류 없이 압축이 해제됩니다.

> 이 예제에서는 `.pipe()`를 사용해 데이터 소스에서 목적지까지 데이터를 전달합니다. 하지만 적절한 에러 핸들러가 연결되어 있지 않다는 점에 주의하세요. 만약 데이터 청크가 제대로 수신되지 않으면, `Readable` 소스나 `gzip` 스트림이 파괴되지 않습니다. [`pump`](https://github.com/mafintosh/pump)는 파이프라인 중 하나가 실패하거나 닫힐 때 모든 스트림을 적절히 파괴하는 유틸리티 도구로, 이 경우 필수입니다!

[`pump`](https://github.com/mafintosh/pump)는 Node.js 8.x 이하 버전에서만 필요합니다. Node.js 10.x 이상 버전에서는 [`pipeline`](https://nodejs.org/api/stream.html#stream_stream_pipeline_streams_callback)이 도입되어 [`pump`](https://github.com/mafintosh/pump)를 대체합니다. 이 모듈 메서드는 스트림 간에 파이프를 연결하고, 오류를 전달하며, 파이프라인이 완료되면 콜백을 제공합니다.

다음은 `pipeline`을 사용한 예제입니다:

```javascript
const { pipeline } = require('node:stream');
const fs = require('node:fs');
const zlib = require('node:zlib');

// pipeline API를 사용해 일련의 스트림을 쉽게 연결하고
// 파이프라인이 완전히 완료되면 알림을 받습니다.
// 대용량 비디오 파일을 효율적으로 gzip으로 압축하는 파이프라인:

pipeline(
  fs.createReadStream('The.Matrix.1080p.mkv'),
  zlib.createGzip(),
  fs.createWriteStream('The.Matrix.1080p.mkv.gz'),
  err => {
    if (err) {
      console.error('파이프라인 실패', err);
    } else {
      console.log('파이프라인 성공');
    }
  }
);
```

또한 [`stream/promises`](https://nodejs.org/api/stream.html#streampipelinesource-transforms-destination-options) 모듈을 사용해 `async` / `await`와 함께 `pipeline`을 사용할 수도 있습니다:

```javascript
const { pipeline } = require('node:stream/promises');
const fs = require('node:fs');
const zlib = require('node:zlib');

async function run() {
  try {
    await pipeline(
      fs.createReadStream('The.Matrix.1080p.mkv'),
      zlib.createGzip(),
      fs.createWriteStream('The.Matrix.1080p.mkv.gz')
    );
    console.log('파이프라인 성공');
  } catch (err) {
    console.error('파이프라인 실패', err);
  }
}
```


## 데이터가 너무 빠르게 들어올 때

[`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams) 스트림이 [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams) 스트림에 데이터를 너무 빠르게 전달하는 경우가 있습니다. 이는 소비자가 처리할 수 있는 속도보다 훨씬 빠른 경우죠.

이런 상황이 발생하면, 소비자는 나중에 처리하기 위해 모든 데이터 청크를 큐에 쌓기 시작합니다. 쓰기 큐는 점점 길어지고, 이로 인해 전체 프로세스가 완료될 때까지 더 많은 데이터를 메모리에 보관해야 합니다.

디스크에 쓰는 작업은 디스크에서 읽는 작업보다 훨씬 느립니다. 따라서 파일을 압축하여 하드 디스크에 쓰려고 할 때, 쓰기 디스크가 읽기 속도를 따라가지 못해 백프레셔(backpressure)가 발생합니다.

```javascript
// 스트림은 속으로 이렇게 말하고 있습니다: "워, 워! 잠깐만, 이건 너무 많아!"
// `write`가 들어오는 데이터 흐름을 따라가려고 하면서
// 데이터 버퍼의 읽기 측면에 데이터가 쌓이기 시작합니다.
inp.pipe(gzip).pipe(outputFile);
```

이 때문에 백프레셔 메커니즘이 중요합니다. 백프레셔 시스템이 없으면, 프로세스가 시스템의 메모리를 모두 사용하게 되어 다른 프로세스가 느려지고, 시스템의 큰 부분을 독점하게 됩니다.

이로 인해 다음과 같은 문제가 발생합니다:

-   현재 실행 중인 다른 모든 프로세스가 느려짐
-   가비지 컬렉터가 과도하게 동작함
-   메모리 고갈

다음 예제에서는 `.write()` 함수의 [반환 값](https://github.com/nodejs/node/blob/55c42bc6e5602e5a47fb774009cfe9289cb88e71/lib/_stream_writable.js#L239)을 제거하고 `true`로 변경하여 Node.js 코어에서 백프레셔 지원을 비활성화합니다. '수정된' 바이너리를 언급할 때는 `return ret;` 줄을 제거하고 `return true;`로 대체한 `node` 바이너리를 실행하는 것을 의미합니다.


## 가비지 컬렉션의 과도한 부하

간단한 벤치마크를 살펴보겠습니다. 위의 예제를 사용해 두 가지 바이너리의 중간값 시간을 측정했습니다.

```
   시행 (#)  | `node` 바이너리 (ms) | 수정된 `node` 바이너리 (ms)
=================================================================
      1       |      56924         |           55011
      2       |      52686         |           55869
      3       |      59479         |           54043
      4       |      54473         |           55229
      5       |      52933         |           59723
=================================================================
평균 시간: |      55299         |           55975
```

두 바이너리 모두 실행에 약 1분 정도 걸리므로 큰 차이는 없습니다. 하지만 의심을 확인하기 위해 자세히 살펴보겠습니다. 리눅스 도구인 [`dtrace`](https://dtrace.org/about/)를 사용해 V8 가비지 컬렉터의 동작을 평가했습니다.

가비지 컬렉터(GC)의 측정 시간은 가비지 컬렉터가 한 번의 전체 사이클을 수행하는 간격을 나타냅니다:

```
대략 시간 (ms) | GC (ms) | 수정된 GC (ms)
=================================================
          0       |    0    |      0
          1       |    0    |      0
         40       |    0    |      2
        170       |    3    |      1
        300       |    3    |      1

         *             *           *
         *             *           *
         *             *           *

      39000       |    6    |     26
      42000       |    6    |     21
      47000       |    5    |     32
      50000       |    8    |     28
      54000       |    6    |     35
```

두 프로세스는 동일하게 시작되고 GC를 같은 속도로 처리하는 것처럼 보입니다. 하지만 몇 초 후, 백프레셔 시스템이 제대로 작동하면 GC 부하를 4-8밀리초 간격으로 일정하게 분산시킵니다. 데이터 전송이 끝날 때까지 이 패턴이 유지됩니다.

반면, 백프레셔 시스템이 없으면 V8 가비지 컬렉션이 점점 늘어납니다. 일반 바이너리는 1분 동안 약 **75**번 GC를 호출하지만, 수정된 바이너리는 **36**번만 호출합니다.

이는 메모리 사용량이 증가하면서 발생하는 느리고 점진적인 부채입니다. 백프레셔 시스템이 없으면 데이터가 전송될 때마다 더 많은 메모리가 사용됩니다.

할당된 메모리가 많을수록 GC는 한 번의 작업에서 더 많은 부분을 처리해야 합니다. 작업 범위가 커질수록 GC는 어떤 메모리를 해제할지 결정해야 하며, 더 큰 메모리 공간에서 분리된 포인터를 스캔하는 데 더 많은 컴퓨팅 파워가 소모됩니다.


## 메모리 소모

각 바이너리의 메모리 소모량을 측정하기 위해 `/usr/bin/time -lp sudo ./node ./backpressure-example/zlib.js` 명령어로 각 프로세스를 개별적으로 실행했습니다.

일반 바이너리의 출력 결과는 다음과 같습니다:

```bash
.write()의 반환 값을 존중한 경우
=============================================
real        58.88
user        56.79
sys          8.79
  87810048  maximum resident set size
         0  average shared memory size
         0  average unshared data size
         0  average unshared stack size
     19427  page reclaims
      3134  page faults
         0  swaps
         5  block input operations
       194  block output operations
         0  messages sent
         0  messages received
         1  signals received
        12  voluntary context switches
    666037  involuntary context switches
```

가상 메모리가 차지하는 최대 바이트 크기는 약 87.81MB입니다.

이제 [`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback) 함수의 [반환 값](https://github.com/nodejs/node/blob/55c42bc6e5602e5a47fb774009cfe9289cb88e71/lib/_stream_writable.js#L239)을 변경하면 다음과 같은 결과를 얻습니다:

```bash
.write()의 반환 값을 존중하지 않은 경우
==================================================
real        54.48
user        53.15
sys          7.43
1524965376  maximum resident set size
         0  average shared memory size
         0  average unshared data size
         0  average unshared stack size
    373617  page reclaims
      3139  page faults
         0  swaps
        18  block input operations
       199  block output operations
         0  messages sent
         0  messages received
         1  signals received
        25  voluntary context switches
    629566  involuntary context switches
```

가상 메모리가 차지하는 최대 바이트 크기는 약 1.52GB입니다.

백프레셔를 위임할 스트림이 없으면 메모리 공간이 훨씬 더 많이 할당됩니다. 동일한 프로세스임에도 엄청난 차이가 발생합니다.

이 실험은 Node.js의 백프레셔 메커니즘이 컴퓨팅 시스템에 얼마나 최적화되고 비용 효율적인지 보여줍니다. 이제 이 메커니즘이 어떻게 동작하는지 자세히 살펴보겠습니다!


## 백프레셔가 이러한 문제를 어떻게 해결하나요?

데이터를 한 프로세스에서 다른 프로세스로 전송하는 다양한 함수가 있습니다. Node.js에는 내장 함수인 [`.pipe()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_pipe_destination_options)가 있습니다. 또한 사용할 수 있는 [다른 패키지들](https://github.com/sindresorhus/awesome-nodejs#streams)도 있습니다! 하지만 이 프로세스의 기본적인 수준에서는 두 가지 별도의 컴포넌트가 있습니다: 데이터의 *소스*와 *소비자*입니다.

소스에서 [`.pipe()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_pipe_destination_options)가 호출되면, 소비자에게 전송할 데이터가 있다는 신호를 보냅니다. `pipe` 함수는 이벤트 트리거를 위한 적절한 백프레셔 클로저를 설정하는 데 도움을 줍니다.

Node.js에서 소스는 [`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams) 스트림이고, 소비자는 [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams) 스트림입니다 (이 두 가지는 [`Duplex`](https://nodejs.org/api/stream.html#stream_duplex_and_transform_streams)나 [`Transform`](https://nodejs.org/api/stream.html#stream_duplex_and_transform_streams) 스트림으로 교체될 수 있지만, 이 가이드의 범위를 벗어납니다).

백프레셔가 트리거되는 시점은 [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams)의 [`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback) 함수의 반환 값으로 정확히 좁힐 수 있습니다. 이 반환 값은 몇 가지 조건에 따라 결정됩니다.

데이터 버퍼가 [`highWaterMark`](https://nodejs.org/api/stream.html#stream_buffering)를 초과하거나 쓰기 큐가 현재 바쁜 경우, [`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback)는 `false`를 반환합니다.

`false` 값이 반환되면 백프레셔 시스템이 작동합니다. 이 시스템은 들어오는 [`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams) 스트림이 데이터를 전송하는 것을 일시 중지하고, 소비자가 다시 준비될 때까지 기다립니다. 데이터 버퍼가 비워지면 [`'drain'`](https://nodejs.org/docs/latest/api/stream.html#stream_event_drain) 이벤트가 발생하고, 들어오는 데이터 흐름이 재개됩니다.

큐가 완료되면 백프레셔는 데이터를 다시 전송할 수 있게 합니다. 사용 중이던 메모리 공간은 해제되고, 다음 데이터 배치를 위해 준비됩니다.

이를 통해 [`.pipe()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_pipe_destination_options) 함수가 사용하는 메모리 양을 고정할 수 있습니다. 메모리 누수나 무한 버퍼링이 발생하지 않으며, 가비지 컬렉터는 메모리의 한 영역만 처리하면 됩니다!

그렇다면 백프레셔가 이렇게 중요한데, 왜 여러분은 (아마도) 들어본 적이 없을까요? 그 이유는 간단합니다: Node.js가 이 모든 것을 자동으로 처리해 주기 때문입니다.

이것은 정말 좋은 일이지만, 커스텀 스트림을 구현하는 방법을 이해하려고 할 때는 그다지 좋지 않을 수 있습니다.

> 대부분의 머신에서는 버퍼가 가득 찼을 때를 결정하는 바이트 크기가 있습니다 (이는 머신마다 다를 수 있습니다). Node.js는 커스텀 [`highWaterMark`](https://nodejs.org/api/stream.html#stream_buffering)를 설정할 수 있게 하지만, 일반적으로 기본값은 16KB (16384, 또는 객체 모드 스트림의 경우 16)로 설정되어 있습니다. 이 값을 높이고 싶은 경우에는 조심스럽게 진행하세요!


## `.pipe()`의 생명주기

백프레셔(backpressure)를 더 잘 이해하기 위해, [`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams) 스트림이 [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams) 스트림으로 [파이핑](https://nodejs.org/docs/latest/api/stream.html#stream_readable_pipe_destination_options)될 때의 생명주기를 나타낸 플로우 차트를 살펴보겠습니다.

```
                                                     +===================+
                         x-->  파이핑 함수들이    +-->   src.pipe(dest)  |
                         x     .pipe 메서드 실행 중 |===================|
                         x     설정된다.           |  이벤트 콜백들     |
  +===============+      x                        |-------------------|
  |   여러분의 데이터   |      x     이들은 데이터 흐름 외부에 존재하지만, | .on('close', cb)  |
  +=======+=======+      x     중요한 이벤트를 연결하고, | .on('data', cb)   |
          |              x     각각의 콜백을 설정한다.  | .on('drain', cb)  |
          |              x                          | .on('unpipe', cb) |
+---------v---------+    x                          | .on('error', cb)  |
|  Readable Stream  +----+                          | .on('finish', cb) |
+-^-------^-------^-+    |                          | .on('end', cb)    |
  ^       |       ^      |                          +-------------------+
  |       |       |      |
  |       ^       |      |
  ^       ^       ^      |    +-------------------+         +=================+
  ^       |       ^      +---->  Writable Stream  +--------->  .write(chunk)  |
  |       |       |           +-------------------+         +=======+=========+
  |       |       |                                                 |
  |       ^       |                              +------------------v---------+
  ^       |       +-> if (!chunk)                |    이 청크가 너무 큰가?    |
  ^       |       |     emit .end();             |    큐가 바쁜가?            |
  |       |       +-> else                       +-------+----------------+---+
  |       ^       |     emit .write();                   |                |
  |       ^       ^                                   +--v---+        +---v---+
  |       |       ^-----------------------------------<  No  |        |  Yes  |
  ^       |                                           +------+        +---v---+
  ^       |                                                               |
  |       ^               emit .pause();          +=================+     |
  |       ^---------------^-----------------------+  return false;  <-----+---+
  |                                               +=================+         |
  |                                                                           |
  ^            큐가 비었을 때     +============+                         |
  ^------------^-----------------------<  버퍼링 |                         |
               |                       |============|                         |
               +> emit .drain();       |  ^Buffer^  |                         |
               +> emit .resume();      +------------+                         |
                                       |  ^Buffer^  |                         |
                                       +------------+   큐에 청크 추가        |
                                       |            <---^---------------------<
                                       +============+
```

> 여러분이 데이터를 조작하기 위해 몇 개의 스트림을 연결하는 파이프라인을 설정한다면, 대부분 [`Transform`](https://nodejs.org/api/stream.html#stream_duplex_and_transform_streams) 스트림을 구현하게 될 것입니다.

이 경우, [`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams) 스트림의 출력이 [`Transform`](https://nodejs.org/api/stream.html#stream_duplex_and_transform_streams) 스트림으로 들어가고, 다시 [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams) 스트림으로 파이핑됩니다.

```javascript
Readable.pipe(Transformable).pipe(Writable);
```

백프레셔는 자동으로 적용되지만, [`Transform`](https://nodejs.org/api/stream.html#stream_duplex_and_transform_streams) 스트림의 들어오고 나가는 `highWaterMark`가 조작될 수 있으며, 이는 백프레셔 시스템에 영향을 미칠 수 있습니다.


## [백프레셔 가이드라인](https://nodejs.org/en/learn/modules/publishing-a-package#backpressure-guidelines)

[Node.js v0.10](https://nodejs.org/docs/v0.10.0/)부터 [`Stream`](https://nodejs.org/api/stream.html) 클래스는 [`.read()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_read_size)와 [`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback)의 동작을 수정할 수 있는 기능을 제공합니다. 이는 각각의 함수의 언더스코어 버전([`._read()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_read_size_1)와 [`._write()`](https://nodejs.org/docs/latest/api/stream.html#stream_writable_write_chunk_encoding_callback_1))을 사용하여 가능합니다.

[Readable 스트림 구현](https://nodejs.org/docs/latest/api/stream.html#stream_implementing_a_readable_stream)과 [Writable 스트림 구현](https://nodejs.org/docs/latest/api/stream.html#stream_implementing_a_writable_stream)에 대한 가이드라인이 문서화되어 있습니다. 여러분이 이를 이미 읽었다고 가정하고, 다음 섹션에서는 좀 더 깊이 있게 다룰 예정입니다.


## 커스텀 스트림 구현 시 지켜야 할 규칙

스트림의 **황금 규칙은 항상 백프레셔를 존중하는 것**입니다. 가장 좋은 실천 방법은 모순되지 않는 실천입니다. 내부 백프레셔 지원과 충돌하는 행동을 피하는 데 주의한다면, 여러분은 좋은 실천을 따르고 있다고 확신할 수 있습니다.

일반적으로 다음 사항을 지켜야 합니다.

1. 요청받지 않은 상태에서는 절대 `.push()`를 사용하지 않는다.
2. `.write()`가 false를 반환한 후에는 호출하지 않고, 'drain' 이벤트를 기다린다.
3. 스트림은 Node.js 버전과 사용하는 라이브러리에 따라 달라질 수 있다. 주의 깊게 테스트해야 한다.

> 세 번째 항목과 관련해, 브라우저 스트림을 구축하는 데 매우 유용한 패키지로 [`readable-stream`](https://github.com/nodejs/readable-stream)이 있습니다. Rodd Vagg는 이 라이브러리의 유용성을 설명하는 [훌륭한 블로그 글](https://r.va.gg/2014/06/why-i-dont-use-nodes-core-stream-module.html)을 작성했습니다. 간단히 말해, 이 라이브러리는 [`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams) 스트림에 대한 자동화된 우아한 기능 저하를 제공하며, 오래된 버전의 브라우저와 Node.js를 지원합니다.


## Readable Streams에 특화된 규칙

지금까지는 [`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback)가 백프레셔에 미치는 영향과 [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams) 스트림에 대해 집중적으로 살펴봤습니다. Node.js의 기능상 데이터는 기술적으로 [`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams)에서 [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams)로 흐릅니다. 하지만 데이터, 물질, 에너지의 전송에서 볼 수 있듯이, 소스는 목적지만큼 중요하며, [`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams) 스트림은 백프레셔 처리 방식에 있어 핵심적입니다.

이 두 프로세스는 효과적으로 소통하기 위해 서로 의존합니다. 만약 [`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams)이 [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams) 스트림이 데이터 전송 중단을 요청할 때 이를 무시한다면, [`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback)의 반환 값이 잘못된 경우와 마찬가지로 문제가 될 수 있습니다.

따라서 [`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback)의 반환 값을 존중하는 것뿐만 아니라, [`._read()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_read_size_1) 메서드에서 사용되는 [`.push()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_push_chunk_encoding)의 반환 값도 존중해야 합니다. 만약 [`.push()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_push_chunk_encoding)가 `false`를 반환하면, 스트림은 소스에서 읽기를 중단합니다. 그렇지 않으면, 스트림은 멈추지 않고 계속 진행됩니다.

다음은 [`.push()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_push_chunk_encoding)를 잘못 사용한 예제입니다:

```javascript
// 이 코드는 push의 반환 값을 완전히 무시하며,
// 이는 목적지 스트림의 백프레셔 신호일 수 있습니다!
class MyReadable extends Readable {
  _read(size) {
    let chunk;
    while (null !== (chunk = getNextChunk())) {
      this.push(chunk);
    }
  }
}
```

또한, 커스텀 스트림 외부에서도 백프레셔를 무시하는 것은 문제가 될 수 있습니다. 다음은 좋은 예제의 반대 사례로, 애플리케이션 코드가 데이터가 사용 가능할 때마다 ([`'data'` 이벤트](https://nodejs.org/api/stream.html#stream_event_data)로 신호를 받음) 데이터를 강제로 전송하는 경우입니다:

```javascript
// 이 코드는 Node.js가 설정한 백프레셔 메커니즘을 무시하고,
// 목적지 스트림이 준비되었는지 여부와 상관없이
// 데이터를 무조건 전송합니다.
readable.on('data', data => writable.write(data));
```

다음은 Readable 스트림에서 [`.push()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_push_chunk_encoding)를 사용하는 예제입니다.

```javascript
const { Readable } = require('node:stream');

// 커스텀 Readable 스트림 생성
const myReadableStream = new Readable({
  objectMode: true,
  read(size) {
    // 스트림에 데이터를 푸시
    this.push({ message: 'Hello, world!' });
    this.push(null); // 스트림의 끝을 표시
  },
});

// 스트림 소비
myReadableStream.on('data', chunk => {
  console.log(chunk);
});

// 출력:
// { message: 'Hello, world!' }
```

이 예제에서는 [`.push()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_push_chunk_encoding)를 사용해 스트림에 단일 객체를 푸시하는 커스텀 Readable 스트림을 생성합니다. [`._read()`](https://nodejs.org/docs/latest/api/stream.html#stream_readable_read_size_1) 메서드는 스트림이 데이터를 소비할 준비가 되었을 때 호출되며, 이 경우 즉시 데이터를 스트림에 푸시하고 `null`을 푸시해 스트림의 끝을 표시합니다.

그런 다음 'data' 이벤트를 리스닝하여 스트림에 푸시된 각 데이터 청크를 로깅합니다. 이 예제에서는 단일 데이터 청크만 푸시했기 때문에 하나의 로그 메시지만 출력됩니다.


## Writable Streams에 특화된 규칙

[`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback) 메서드는 특정 조건에 따라 `true` 또는 `false`를 반환할 수 있습니다. 다행히도, 우리가 직접 [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams) 스트림을 만들 때, [`스트림 상태 머신`](https://en.wikipedia.org/wiki/Finite-state_machine)이 콜백을 처리하고 백프레셔를 관리하며 데이터 흐름을 최적화해 줍니다.

하지만 [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams)을 직접 사용할 때는 [`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback)의 반환 값을 존중하고 다음 조건에 주의해야 합니다:

-   쓰기 큐가 바쁜 경우, [`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback)는 `false`를 반환합니다.
-   데이터 청크가 너무 큰 경우, [`.write()`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback)는 `false`를 반환합니다 (이 한계는 [`highWaterMark`](https://nodejs.org/api/stream.html#stream_buffering) 변수로 표시됩니다).

```javascript
// 이 Writable은 JavaScript 콜백의 비동기 특성 때문에 유효하지 않습니다.
// 마지막 콜백 이전에 각 콜백에 대한 return 문이 없으면,
// 여러 콜백이 호출될 가능성이 큽니다.
class MyWritable extends Writable {
  _write(chunk, encoding, callback) {
    if (chunk.toString().indexOf('a') >= 0) callback();
    else if (chunk.toString().indexOf('b') >= 0) callback();
    callback();
  }
}

// 올바르게 작성하는 방법은 다음과 같습니다:
if (chunk.contains('a')) return callback();
if (chunk.contains('b')) return callback();
callback();
```

[`._writev()`](https://nodejs.org/api/stream.html#stream_writable_writev_chunks_callback)를 구현할 때도 주의할 점이 있습니다. 이 함수는 [`.cork()`](https://nodejs.org/api/stream.html#writablecork)와 함께 사용되지만, 작성 시 흔히 하는 실수가 있습니다:

```javascript
// 여기서 .uncork()를 두 번 호출하면 C++ 레이어에서 두 번 호출되어
// cork/uncork 기법이 무의미해집니다.
ws.cork();
ws.write('hello ');
ws.write('world ');
ws.uncork();

ws.cork();
ws.write('from ');
ws.write('Matteo');
ws.uncork();

// 올바른 방법은 process.nextTick()을 사용하여 다음 이벤트 루프에서 실행되도록 하는 것입니다.
ws.cork();
ws.write('hello ');
ws.write('world ');
process.nextTick(doUncork, ws);

ws.cork();
ws.write('from ');
ws.write('Matteo');
process.nextTick(doUncork, ws);

// 전역 함수로 정의합니다.
function doUncork(stream) {
  stream.uncork();
}
```

[`.cork()`](https://nodejs.org/api/stream.html#writablecork)는 원하는 만큼 호출할 수 있지만, 다시 흐르게 하려면 [`.uncork()`](https://nodejs.org/api/stream.html#stream_writable_uncork)를 같은 횟수만큼 호출해야 합니다.


## 결론

스트림은 Node.js에서 자주 사용되는 모듈입니다. Node.js의 내부 구조에서 중요한 역할을 하며, 개발자들이 Node.js 모듈 생태계를 확장하고 연결하는 데 필수적입니다.

이제 여러분은 백프레셔를 고려하여 자신만의 [`Writable`](https://nodejs.org/api/stream.html#stream_writable_streams)과 [`Readable`](https://nodejs.org/api/stream.html#stream_readable_streams) 스트림을 안전하게 코딩하고 문제를 해결할 수 있을 것입니다. 또한, 이 지식을 동료나 친구들과 공유할 수 있게 되었기를 바랍니다.

Node.js로 애플리케이션을 구축할 때 스트리밍 능력을 향상시키고 활용하기 위해 [`Stream`](https://nodejs.org/api/stream.html)의 다른 API 함수들에 대해 더 알아보는 것을 잊지 마세요.


