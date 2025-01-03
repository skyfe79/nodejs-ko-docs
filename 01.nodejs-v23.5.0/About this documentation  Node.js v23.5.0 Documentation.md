# 이 문서에 대해

Node.js 공식 API 참조 문서에 오신 것을 환영합니다!

Node.js는 [V8 JavaScript 엔진](https://v8.dev/)을 기반으로 구축된 JavaScript 런타임입니다.

### 기여하기[#](https://nodejs.org/docs/latest/api/documentation.html#contributing)

이 문서의 오류는 [이슈 트래커](https://github.com/nodejs/node/issues/new)에 보고할 수 있습니다. 풀 리퀘스트를 제출하는 방법은 [기여 가이드](https://github.com/nodejs/node/blob/HEAD/CONTRIBUTING.md)를 참고하세요.

### 안정성 지수[#](https://nodejs.org/docs/latest/api/documentation.html#stability-index)

문서 전체에는 각 섹션의 안정성을 나타내는 표시가 있습니다. 일부 API는 검증되었고 광범위하게 사용되어 변경될 가능성이 거의 없습니다. 반면, 다른 API는 새롭고 실험적이거나 위험할 수 있습니다.

안정성 지수는 다음과 같습니다:

- **안정성: 0 - 사용 중단됨**: 해당 기능은 경고를 발생시킬 수 있습니다. 하위 호환성이 보장되지 않습니다.
- **안정성: 1 - 실험적**: 이 기능은 [시맨틱 버저닝](https://semver.org/) 규칙을 따르지 않습니다. 향후 릴리스에서 호환되지 않는 변경이나 제거가 발생할 수 있습니다. 프로덕션 환경에서 사용하지 않는 것이 좋습니다.

실험적 기능은 다음과 같은 단계로 나뉩니다:

- **1.0 - 초기 개발**: 이 단계의 실험적 기능은 미완성이며 상당한 변경이 있을 수 있습니다.
- **1.1 - 활성 개발**: 이 단계의 실험적 기능은 최소한의 사용 가능성을 갖추고 있습니다.
- **1.2 - 릴리스 후보**: 이 단계의 실험적 기능은 안정화될 준비가 거의 되었습니다. 더 이상의 주요 변경은 예상되지 않지만 사용자 피드백에 따라 발생할 수 있습니다. 사용자 테스트와 피드백을 권장하여 이 기능이 안정화될 준비가 되었는지 확인할 수 있습니다.

실험적 기능은 일반적으로 안정화되거나 사용 중단 절차 없이 제거됩니다.

- **안정성: 2 - 안정적**: npm 생태계와의 호환성이 우선순위입니다.
- **안정성: 3 - 레거시**: 이 기능은 제거될 가능성이 낮고 시맨틱 버저닝 보장을 받지만, 더 이상 적극적으로 유지 관리되지 않으며 대안이 있습니다.

레거시 기능은 사용에 해가 없고 npm 생태계에서 널리 사용되는 경우 사용 중단 대신 레거시로 표시됩니다. 레거시 기능에서 발견된 버그는 수정되지 않을 가능성이 높습니다.

실험적 기능을 사용할 때는 특히 라이브러리를 작성할 때 주의하세요. 사용자들은 실험적 기능이 사용되고 있는지 알지 못할 수 있습니다. 실험적 API가 변경되면 버그나 동작 변화가 사용자에게 예기치 못한 영향을 미칠 수 있습니다. 이러한 문제를 방지하기 위해 실험적 기능을 사용할 때는 커맨드라인 플래그가 필요할 수 있습니다. 실험적 기능은 [경고](https://nodejs.org/docs/latest/api/process.html#event-warning)를 발생시킬 수도 있습니다.

### 안정성 개요[#](https://nodejs.org/docs/latest/api/documentation.html#stability-overview)

| API | 안정성 |
| --- | --- |
| [Assert](https://nodejs.org/docs/latest/api/assert.html) | (2) 안정적 |
| [Async hooks](https://nodejs.org/docs/latest/api/async_hooks.html) | (1) 실험적 |
| [Asynchronous context tracking](https://nodejs.org/docs/latest/api/async_context.html) | (2) 안정적 |
| [Buffer](https://nodejs.org/docs/latest/api/buffer.html) | (2) 안정적 |
| [Child process](https://nodejs.org/docs/latest/api/child_process.html) | (2) 안정적 |
| [Cluster](https://nodejs.org/docs/latest/api/cluster.html) | (2) 안정적 |
| [Console](https://nodejs.org/docs/latest/api/console.html) | (2) 안정적 |
| [Crypto](https://nodejs.org/docs/latest/api/crypto.html) | (2) 안정적 |
| [Diagnostics Channel](https://nodejs.org/docs/latest/api/diagnostics_channel.html) | (2) 안정적 |
| [DNS](https://nodejs.org/docs/latest/api/dns.html) | (2) 안정적 |
| [Domain](https://nodejs.org/docs/latest/api/domain.html) | (0) 사용 중단됨 |
| [File system](https://nodejs.org/docs/latest/api/fs.html) | (2) 안정적 |
| [HTTP](https://nodejs.org/docs/latest/api/http.html) | (2) 안정적 |
| [HTTP/2](https://nodejs.org/docs/latest/api/http2.html) | (2) 안정적 |
| [HTTPS](https://nodejs.org/docs/latest/api/https.html) | (2) 안정적 |
| [Inspector](https://nodejs.org/docs/latest/api/inspector.html) | (2) 안정적 |
| [Modules: `node:module` API](https://nodejs.org/docs/latest/api/module.html) | (1) .2 - 릴리스 후보 (비동기 버전) 안정성: 1.1 - 활성 개발 (동기 버전) |
| [Modules: CommonJS modules](https://nodejs.org/docs/latest/api/modules.html) | (2) 안정적 |
| [Modules: TypeScript](https://nodejs.org/docs/latest/api/typescript.html) | (1) .1 - 활성 개발 |
| [OS](https://nodejs.org/docs/latest/api/os.html) | (2) 안정적 |
| [Path](https://nodejs.org/docs/latest/api/path.html) | (2) 안정적 |
| [Performance measurement APIs](https://nodejs.org/docs/latest/api/perf_hooks.html) | (2) 안정적 |
| [Punycode](https://nodejs.org/docs/latest/api/punycode.html) | (0) 사용 중단됨 |
| [Query string](https://nodejs.org/docs/latest/api/querystring.html) | (2) 안정적 |
| [Readline](https://nodejs.org/docs/latest/api/readline.html) | (2) 안정적 |
| [REPL](https://nodejs.org/docs/latest/api/repl.html) | (2) 안정적 |
| [Single executable applications](https://nodejs.org/docs/latest/api/single-executable-applications.html) | (1) .1 - 활성 개발 |
| [SQLite](https://nodejs.org/docs/latest/api/sqlite.html) | (1) .1 - 활성 개발. |
| [Stream](https://nodejs.org/docs/latest/api/stream.html) | (2) 안정적 |
| [String decoder](https://nodejs.org/docs/latest/api/string_decoder.html) | (2) 안정적 |
| [Test runner](https://nodejs.org/docs/latest/api/test.html) | (2) 안정적 |
| [Timers](https://nodejs.org/docs/latest/api/timers.html) | (2) 안정적 |
| [TLS (SSL)](https://nodejs.org/docs/latest/api/tls.html) | (2) 안정적 |
| [Trace events](https://nodejs.org/docs/latest/api/tracing.html) | (1) 실험적 |
| [TTY](https://nodejs.org/docs/latest/api/tty.html) | (2) 안정적 |
| [UDP/datagram sockets](https://nodejs.org/docs/latest/api/dgram.html) | (2) 안정적 |
| [URL](https://nodejs.org/docs/latest/api/url.html) | (2) 안정적 |
| [Util](https://nodejs.org/docs/latest/api/util.html) | (2) 안정적 |
| [VM (executing JavaScript)](https://nodejs.org/docs/latest/api/vm.html) | (2) 안정적 |
| [Web Crypto API](https://nodejs.org/docs/latest/api/webcrypto.html) | (2) 안정적 |
| [Web Streams API](https://nodejs.org/docs/latest/api/webstreams.html) | (2) 안정적 |
| [WebAssembly System Interface (WASI)](https://nodejs.org/docs/latest/api/wasi.html) | (1) 실험적 |
| [Worker threads](https://nodejs.org/docs/latest/api/worker_threads.html) | (2) 안정적 |
| [Zlib](https://nodejs.org/docs/latest/api/zlib.html) | (2) 안정적 |

### JSON 출력[#](https://nodejs.org/docs/latest/api/documentation.html#json-output)

추가된 버전: v0.6.12

모든 `.html` 문서에는 해당 `.json` 문서가 있습니다. 이는 문서를 사용하는 IDE 및 기타 유틸리티를 위한 것입니다.

### 시스템 호출 및 매뉴얼 페이지[#](https://nodejs.org/docs/latest/api/documentation.html#system-calls-and-man-pages)

Node.js에서 시스템 호출을 감싸는 함수는 이를 문서화합니다. 문서는 해당 시스템 호출이 어떻게 동작하는지 설명하는 매뉴얼 페이지로 연결됩니다.

대부분의 Unix 시스템 호출은 Windows에서도 유사하게 동작합니다. 그러나 동작 차이는 피할 수 없을 수 있습니다.


