# Node.js 테스트 러너 사용하기

Node.js는 유연하고 강력한 내장 테스트 러너를 제공합니다. 이 가이드에서는 이를 설정하고 사용하는 방법을 설명합니다.

## 아키텍처 개요
## 의존성 설치
## package.json

```
example/
  ├ …
  ├ src/
    ├ app/…
    └ sw/…
  └ test/
    ├ globals/
      ├ …
      ├ IndexedDb.js
      └ ServiceWorkerGlobalScope.js
    ├ setup.mjs
    ├ setup.units.mjs
    └ setup.ui.mjs
```

> **참고**: glob 패턴은 Node.js v21 이상에서 사용 가능하며, glob 패턴 자체를 따옴표로 감싸야 합니다. 따옴표를 사용하지 않으면 예상과 다른 동작이 발생할 수 있습니다. 처음에는 작동하는 것처럼 보이지만 실제로는 그렇지 않을 수 있습니다.

항상 필요한 몇 가지 설정이 있다면, 다음과 같은 기본 설정 파일에 넣어두세요. 이 파일은 다른 더 구체적인 설정 파일에서 불러와 사용됩니다.


## 일반 설정

```javascript
import { register } from 'node:module';

register('some-typescript-loader');
// 이제부터 TypeScript를 사용할 수 있습니다.
// 하지만 다른 테스트/설정 파일(*.mjs)은 여전히 일반 JavaScript여야 합니다!
```

각 설정을 위해 전용 `setup` 파일을 만듭니다. 이때 기본 `setup.mjs` 파일이 각 파일 내에서 임포트되도록 해야 합니다. 설정을 분리하는 데는 여러 이유가 있지만, 가장 명확한 이유는 [YAGNI](https://en.wikipedia.org/wiki/You_aren't_gonna_need_it)와 성능 때문입니다. 여러분이 설정하는 많은 부분이 환경별 모의 객체(mocks)나 스텁(stubs)일 수 있으며, 이는 상당히 비용이 많이 들고 테스트 실행 속도를 늦출 수 있습니다. 이러한 비용(CI에 지불하는 실제 비용, 테스트가 끝나기를 기다리는 시간 등)을 필요하지 않을 때 피하고 싶을 것입니다.

아래 예제들은 실제 프로젝트에서 가져온 것입니다. 여러분의 프로젝트에 적합하지 않을 수 있지만, 각각은 널리 적용 가능한 일반적인 개념을 보여줍니다.


## ServiceWorker 테스트

[`ServiceWorkerGlobalScope`](https://developer.mozilla.org/docs/Web/API/ServiceWorkerGlobalScope)는 다른 환경에서는 존재하지 않는 매우 특정한 API를 포함하고 있습니다. 일부 API는 다른 것들과 유사해 보이지만(예: `fetch`) 확장된 동작을 가지고 있습니다. 이러한 API가 관련 없는 테스트에 영향을 미치지 않도록 주의해야 합니다.

```javascript
import { beforeEach } from 'node:test';
import { ServiceWorkerGlobalScope } from './globals/ServiceWorkerGlobalScope.js';
import './setup.mjs'; // 💡

beforeEach(globalSWBeforeEach);
function globalSWBeforeEach() {
  globalThis.self = new ServiceWorkerGlobalScope();
}
```

```javascript
import assert from 'node:assert/strict';
import { describe, mock, it } from 'node:test';
import { onActivate } from './onActivate.js';

describe('ServiceWorker::onActivate()', () => {
  const globalSelf = globalThis.self;
  const claim = mock.fn(async function mock__claim() {});
  const matchAll = mock.fn(async function mock__matchAll() {});

  class ActivateEvent extends Event {
    constructor(...args) {
      super('activate', ...args);
    }
  }

  before(() => {
    globalThis.self = {
      clients: { claim, matchAll },
    };
  });
  after(() => {
    global.self = globalSelf;
  });

  it('should claim all clients', async () => {
    await onActivate(new ActivateEvent());

    assert.equal(claim.mock.callCount(), 1);
    assert.equal(matchAll.mock.callCount(), 1);
  });
});
```


## 스냅샷 테스트

스냅샷 테스트는 Jest에서 처음 도입되었고, 이제는 Node.js v22.3.0을 포함한 많은 라이브러리에서 이 기능을 구현하고 있습니다. 이 테스트는 컴포넌트 렌더링 출력을 검증하거나 [Infrastructure as Code](https://en.wikipedia.org/wiki/Infrastructure_as_code) 설정을 확인하는 등 다양한 용도로 사용됩니다. 사용 사례에 관계없이 개념은 동일합니다.

`--experimental-test-snapshots`를 통해 기능을 활성화하는 것 외에 특별한 설정은 필요하지 않습니다. 하지만 선택적 설정을 보여주기 위해 기존 테스트 설정 파일에 다음과 같은 내용을 추가할 수 있습니다.

기본적으로 Node.js는 문법 강조 감지와 호환되지 않는 파일명을 생성합니다: `.js.snapshot`. 생성된 파일은 실제로 CJS 파일이므로, 더 적절한 파일명은 `.snapshot.cjs`로 끝나야 합니다 (또는 아래 예제처럼 더 간결하게 `.snap.cjs`). 이는 ESM 프로젝트에서도 더 잘 동작합니다.

```javascript
import { basename, dirname, extname, join } from 'node:path';
import { snapshot } from 'node:test';

snapshot.setResolveSnapshotPath(generateSnapshotPath);
/**
 * @param {string} testFilePath '/tmp/foo.test.js'
 * @returns {string} '/tmp/foo.test.snap.cjs'
 */
function generateSnapshotPath(testFilePath) {
  const ext = extname(testFilePath);
  const filename = basename(testFilePath, ext);
  const base = dirname(testFilePath);

  return join(base, `${filename}.snap.cjs`);
}
```

아래 예제는 UI 컴포넌트를 위한 [testing library](https://testing-library.com/)를 사용한 스냅샷 테스트를 보여줍니다. `assert.snapshot`에 접근하는 두 가지 다른 방법을 확인하세요:

```javascript
import { describe, it } from 'node:test';

import { prettyDOM } from '@testing-library/dom';
import { render } from '@testing-library/react'; // 어떤 프레임워크든 가능 (예: svelte)

import { SomeComponent } from './SomeComponent.jsx';

describe('<SomeComponent>', () => {
  // "fat-arrow" 구문을 선호하는 사람들에게는 다음 방식이 일관성을 유지하기에 더 나을 수 있습니다.
  it('should render defaults when no props are provided', (t) => {
    const component = render(<SomeComponent />).container.firstChild;

    t.assert.snapshot(prettyDOM(component));
  });

  it('should consume `foo` when provided', function() {
    const component = render(<SomeComponent foo="bar" />).container.firstChild;

    this.assert.snapshot(prettyDOM(component));
    // `this`는 `function`을 사용할 때만 동작합니다 ("fat arrow"에서는 안 됨).
  });
});
```

> ⚠️ `assert.snapshot`은 테스트 컨텍스트(`t` 또는 `this`)에서 제공되며, `node:assert`에서 제공되지 않습니다. 이는 테스트 컨텍스트가 `node:assert`가 접근할 수 없는 스코프에 접근할 수 있기 때문에 필요합니다 (`assert.snapshot`을 사용할 때마다 `snapshot(this, value)`처럼 수동으로 제공해야 한다면 매우 번거로울 것입니다).


## [유닛 테스트](https://nodejs.org/en/learn/test-runner/introduction#unit-tests)

유닛 테스트는 가장 간단한 테스트로, 특별한 설정이 거의 필요하지 않습니다. 대부분의 테스트는 유닛 테스트일 가능성이 높기 때문에, 설정을 최소한으로 유지하는 것이 중요합니다. 설정 성능이 조금만 떨어져도 그 영향이 크게 확대되기 때문입니다.

```javascript
import { register } from 'node:module';

import './setup.mjs'; // 💡

register('some-plaintext-loader');
// 이제 graphql 같은 일반 텍스트 파일을 불러올 수 있습니다:
// import GET_ME from 'get-me.gql'; GET_ME = '
```

```javascript
import assert from 'node:assert/strict';
import { describe, it } from 'node:test';

import { Cat } from './Cat.js';
import { Fish } from './Fish.js';
import { Plastic } from './Plastic.js';

describe('Cat', () => {
  it('should eat fish', () => {
    const cat = new Cat();
    const fish = new Fish();

    assert.doesNotThrow(() => cat.eat(fish));
  });

  it('should NOT eat plastic', () => {
    const cat = new Cat();
    const plastic = new Plastic();

    assert.throws(() => cat.eat(plastic));
  });
});
```


## [사용자 인터페이스 테스트](https://nodejs.org/en/learn/test-runner/introduction#user-interface-tests)

UI 테스트는 일반적으로 DOM과, 경우에 따라 브라우저 전용 API(예: 아래에서 사용된 [`IndexedDb`](https://developer.mozilla.org/docs/Web/API/IndexedDB_API))가 필요합니다. 이러한 테스트는 설정이 매우 복잡하고 비용이 많이 듭니다.

`IndexedDb`와 같은 API를 사용하지만 매우 독립적인 경우, 아래와 같은 전역 모킹(mocking)은 적합하지 않을 수 있습니다. 대신, `IndexedDb`에 접근하는 특정 테스트 내부로 `beforeEach`를 이동시키는 것이 좋습니다. 만약 `IndexedDb`에 접근하는 모듈이 널리 사용된다면, 해당 모듈을 모킹하거나(이것이 더 나은 선택일 수 있음) 여기에 유지하는 것을 고려하세요.

```javascript
import { register } from 'node:module';

// ⚠️ JSDom의 인스턴스가 하나만 생성되도록 주의하세요. 여러 개가 생성되면 문제가 발생할 수 있습니다.
import jsdom from 'global-jsdom';

import './setup.units.mjs'; // 💡

import { IndexedDb } from './globals/IndexedDb.js';

register('some-css-modules-loader');

jsdom(undefined, {
  url: 'https://test.example.com', // ⚠️ 이 값을 지정하지 않으면 문제가 발생할 수 있습니다.
});

// 전역 객체를 데코레이션하는 예시입니다.
// JSDOM의 `history`는 네비게이션을 처리하지 않습니다. 다음 코드는 대부분의 경우를 처리합니다.
const pushState = globalThis.history.pushState.bind(globalThis.history);
globalThis.history.pushState = function mock_pushState(data, unused, url) {
  pushState(data, unused, url);
  globalThis.location.assign(url);
};

beforeEach(globalUIBeforeEach);
function globalUIBeforeEach() {
  globalThis.indexedDb = new IndexedDb();
}
```

UI 테스트는 두 가지 수준으로 나눌 수 있습니다: 유닛 테스트와 유사한 방식(외부 요소와 의존성을 모킹)과 더 엔드투엔드에 가까운 방식(IndexedDb와 같은 외부 요소만 모킹하고 나머지는 실제로 사용). 전자가 일반적으로 더 순수한 옵션이며, 후자는 [Playwright](https://playwright.dev/)나 [Puppeteer](https://pptr.dev/)와 같은 도구를 사용한 완전한 엔드투엔드 자동화 테스트로 미뤄지는 경우가 많습니다. 아래는 전자의 예시입니다.

```javascript
import { before, describe, mock, it } from 'node:test';

import { screen } from '@testing-library/dom';
import { render } from '@testing-library/react'; // 어떤 프레임워크든 가능 (예: svelte)

// ⚠️ SomeOtherComponent는 정적 임포트가 아닙니다.
// 이는 해당 컴포넌트의 임포트를 모킹하기 위해 필요합니다.

describe('<SomeOtherComponent>', () => {
  let SomeOtherComponent;
  let calcSomeValue;

  before(async () => {
    // ⚠️ 순서가 중요합니다: 모킹은 해당 모듈을 사용하는 코드가 임포트되기 전에 설정되어야 합니다.

    // `--experimental-test-module-mocks` 플래그가 설정되어 있어야 합니다.
    calcSomeValue = mock.module('./calcSomeValue.js', { calcSomeValue: mock.fn() });

    ({ SomeOtherComponent } = await import('./SomeOtherComponent.jsx'));
  });

  describe('when calcSomeValue fails', () => {
    // 이 경우 스냅샷 테스트를 사용하지 않는 것이 좋습니다.
    // 스냅샷 테스트는 오류 메시지에 사소한 변경이 생기면 잘못된 실패를 일으킬 수 있으며,
    // 스냅샷을 업데이트해야 하는데 실제로는 아무런 가치가 없기 때문입니다.

    it('should fail gracefully by displaying a pretty error', () => {
      calcSomeValue.mockImplementation(function mock__calcSomeValue() { return null });

      render(<SomeOtherComponent>);

      const errorMessage = screen.queryByText('unable');

      assert.ok(errorMessage);
    });
  });
});
```


