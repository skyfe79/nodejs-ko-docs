# [테스트에서의 모킹](https://nodejs.org/en/learn/test-runner/introduction#mocking-in-tests)

모킹은 가짜 객체를 만드는 방법입니다. 일반적으로 `'a'일 때, 'b'를 실행한다`는 방식으로 제어합니다. 이 방법은 복잡한 부분을 줄이고 "중요하지 않은" 요소를 통제하기 위해 사용됩니다. "목(mock)"과 "스텁(stub)"은 기술적으로 다른 종류의 "테스트 더블(test doubles)"입니다. 궁금한 독자를 위해 설명하자면, 스텁은 아무런 동작을 하지 않지만 호출을 추적하는 대체물입니다. 목은 스텁에 가짜 구현(`'a'일 때, 'b'를 실행한다`)이 추가된 것입니다. 이 문서에서는 두 개념의 차이가 중요하지 않으므로 스텁도 목으로 통칭합니다.

테스트는 결정적(deterministic)이어야 합니다. 어떤 순서로든, 몇 번을 실행하든 항상 동일한 결과를 내야 합니다. 적절한 설정과 모킹이 이를 가능하게 합니다.

Node.js는 다양한 코드를 모킹할 수 있는 여러 방법을 제공합니다.

이 글에서는 다음과 같은 테스트 유형을 다룹니다:

| 타입 | 설명 | 예제 | 모킹 대상 |
| --- | --- | --- | --- |
| 유닛 테스트 | 격리할 수 있는 가장 작은 코드 단위 | `const sum = (a, b) => a + b` | 내부 코드, 외부 코드, 외부 시스템 |
| 컴포넌트 테스트 | 유닛 + 의존성 | `const arithmetic = (op = sum, a, b) => ops[op](a, b)` | 외부 코드, 외부 시스템 |
| 통합 테스트 | 컴포넌트 간의 조합 | \- | 외부 코드, 외부 시스템 |
| 종단 간 테스트(e2e) | 앱 + 외부 데이터 저장소, 전달 등 | 실제 외부 시스템에 연결된 앱을 사용하는 가짜 사용자(예: Playwright 에이전트) | 없음(모킹하지 않음) |

모킹을 언제 사용하고 언제 사용하지 않을지에 대한 다양한 의견이 있습니다. 주요 내용은 아래와 같습니다.


## [모의 객체 사용 시기와 사용하지 말아야 할 때](https://nodejs.org/en/learn/test-runner/introduction#when-and-not-to-mock)

모의 객체를 사용할 주요 대상은 크게 세 가지로 나눌 수 있습니다:

-   **자체 코드**
-   **외부 코드**
-   **외부 시스템**


### [자체 코드](https://nodejs.org/en/learn/test-runner/introduction#own-code)

이 부분은 여러분의 프로젝트가 직접 제어하는 코드입니다.

```javascript
import foo from './foo.mjs';

export function main() {
  const f = foo();
}
```

여기서 `foo`는 `main` 함수의 "자체 코드" 의존성입니다.


#### [왜 필요한가](https://nodejs.org/en/learn/test-runner/introduction#why)

`main` 함수에 대한 진정한 단위 테스트를 위해서는 `foo`를 모의(mock)해야 합니다. 여러분이 테스트하려는 것은 `main`이 제대로 동작하는지 확인하는 것이지, `main`과 `foo`가 함께 동작하는지 확인하는 것이 아닙니다. (그것은 다른 테스트입니다.)


#### [왜 하지 않는가](https://nodejs.org/en/learn/test-runner/introduction#why-not)

`foo`를 모킹(mocking)하는 것은 그럴 가치보다 더 많은 문제를 일으킬 수 있습니다. 특히 `foo`가 단순하고, 잘 테스트되었으며, 자주 업데이트되지 않는 경우에는 더욱 그렇습니다.

`foo`를 모킹하지 않는 것이 더 나을 수 있습니다. 이는 더 진정성 있는 테스트를 가능하게 하고, `foo`의 커버리지를 높이기 때문입니다(`main`의 테스트가 `foo`도 검증하기 때문입니다). 하지만 이는 잡음을 만들 수도 있습니다. `foo`가 고장 나면, 다른 많은 테스트들도 함께 실패하게 되어 문제를 추적하는 것이 더 번거로워집니다. 만약 문제의 근본 원인이 되는 항목에 대한 단 하나의 테스트만 실패한다면, 그 문제를 발견하기 매우 쉬울 것입니다. 반면에 100개의 테스트가 실패한다면, 진짜 문제를 찾는 것이 바늘을 찾는 것처럼 어려워질 수 있습니다.


### [외부 코드](https://nodejs.org/en/learn/test-runner/introduction#external-code)

이것은 여러분의 프로젝트에서 제어할 수 없는 코드입니다.

```javascript
import bar from 'bar';

export function main() {
  const f = bar();
}
```

여기서 `bar`는 외부 패키지입니다. 예를 들어 npm 의존성과 같은 것들이죠.

단위 테스트에서는 이 코드를 항상 모킹(mock)해야 합니다. 컴포넌트 테스트와 통합 테스트에서는 모킹 여부가 이 코드의 역할에 따라 달라집니다.


#### [왜](https://nodejs.org/en/learn/test-runner/introduction#why-1)

여러분의 프로젝트가 유지하지 않는 코드가 작동하는지 확인하는 것은 단위 테스트의 목적이 아닙니다. 그 코드는 자체 테스트를 가져야 합니다.


#### [왜 사용하지 않을까](https://nodejs.org/en/learn/test-runner/introduction#why-not-1)

모킹(mocking)이 현실적이지 않은 경우가 있습니다. 예를 들어, React나 Angular 같은 대형 프레임워크를 모킹하는 것은 거의 없습니다. (해결책이 문제보다 더 나쁠 수 있기 때문입니다.)


### [외부 시스템](https://nodejs.org/en/learn/test-runner/introduction#external-system)

외부 시스템은 데이터베이스, 환경(웹 앱의 경우 Chromium이나 Firefox, Node 앱의 경우 운영체제 등), 파일 시스템, 메모리 저장소 등을 말합니다.

이상적으로는 이러한 외부 시스템을 모킹(mocking)할 필요가 없어야 합니다. 각 테스트 케이스마다 격리된 복사본을 만드는 것은 비용이나 추가 실행 시간 등의 이유로 거의 불가능합니다. 따라서 다음으로 좋은 방법은 모킹을 사용하는 것입니다. 모킹 없이 테스트를 실행하면 테스트들이 서로 방해할 수 있습니다.

**storage.mjs**
```javascript
import { db } from 'db';

export function read(key, all = false) {
  validate(key, val);

  if (all) return db.getAll(key);

  return db.getOne(key);
}

export function save(key, val) {
  validate(key, val);

  return db.upsert(key, val);
}
```

위 코드에서 첫 번째와 두 번째 케이스(`it()` 구문)는 동시에 실행되며 동일한 저장소를 변경하기 때문에 서로 방해할 수 있습니다. 이는 경쟁 조건(race condition)을 유발합니다. `save()` 함수의 삽입 작업은 `read()` 함수의 테스트가 항목을 찾는 데 실패하게 만들 수 있고, 반대로 `read()` 함수도 `save()` 함수의 테스트에 영향을 줄 수 있습니다.


## [What to mock](https://nodejs.org/en/learn/test-runner/introduction#what-to-mock)





### [모듈 + 유닛](https://nodejs.org/en/learn/test-runner/introduction#modules--units)

이 예제는 Node.js 테스트 러너의 [`mock`](https://nodejs.org/api/test.html#class-mocktracker) 기능을 활용합니다.

```javascript
import assert from 'node:assert/strict';
import { before, describe, it, mock } from 'node:test';

describe('foo', { concurrency: true }, () => {
  let barMock = mock.fn();
  let foo;

  before(async () => {
    const barNamedExports = await import('./bar.mjs')
      // 원본 default export는 제외
      .then(({ default, ...rest }) => rest);

    // 각 테스트 후에 수동으로 restore()를 호출하거나
    // 모든 테스트 후에 reset()을 호출할 필요는 없습니다.
    // (Node.js가 자동으로 처리합니다.)
    mock.module('./bar.mjs', {
      defaultExport: barMock
      // 모킹하지 않을 다른 exports는 유지
      namedExports: barNamedExports,
    });

    // 모킹이 설정된 후에만 import가 시작되도록
    // 반드시 동적 import를 사용해야 합니다.
    ({ foo } = await import('./foo.mjs'));
  });

  it('should do the thing', () => {
    barMock.mockImplementationOnce(function bar_mock() {/* … */});

    assert.equal(foo(), 42);
  });
});
```


### [APIs](https://nodejs.org/en/learn/test-runner/introduction#apis)

많은 사람들이 잘 모르는 사실이지만, `fetch`를 모킹(mock)할 수 있는 내장된 방법이 있습니다. [`undici`](https://github.com/nodejs/undici)는 Node.js에서 `fetch`를 구현한 라이브러리입니다. 이 라이브러리는 `node`와 함께 제공되지만, 현재는 `node` 자체에서 노출되지 않기 때문에 별도로 설치해야 합니다 (예: `npm install undici`).

```javascript
import assert from 'node:assert/strict';
import { beforeEach, describe, it } from 'node:test';
import { MockAgent, setGlobalDispatcher } from 'undici';

import endpoints from './endpoints.mjs';

describe('endpoints', { concurrency: true }, () => {
  let agent;
  beforeEach(() => {
    agent = new MockAgent();
    setGlobalDispatcher(agent);
  });

  it('should retrieve data', async () => {
    const endpoint = 'foo';
    const code = 200;
    const data = {
      key: 'good',
      val: 'item',
    };

    agent
      .get('example.com')
      .intercept({
        path: endpoint,
        method: 'GET',
      })
      .reply(code, data);

    assert.deepEqual(await endpoints.get(endpoint), {
      code,
      data,
    });
  });

  it('should save data', async () => {
    const endpoint = 'foo/1';
    const code = 201;
    const data = {
      key: 'good',
      val: 'item',
    };

    agent
      .get('example.com')
      .intercept({
        path: endpoint,
        method: 'PUT',
      })
      .reply(code, data);

    assert.deepEqual(await endpoints.save(endpoint), {
      code,
      data,
    });
  });
});
```


### [시간](https://nodejs.org/en/learn/test-runner/introduction#time)

닥터 스트레인지처럼 여러분도 시간을 조절할 수 있습니다. 보통 이 기능은 테스트 실행 시간을 인위적으로 늘리지 않기 위해 사용합니다. 예를 들어, `setTimeout()`이 3분 동안 기다리게 하는 것을 원하지 않을 때 유용합니다. 또한 시간 여행을 하고 싶을 때도 활용할 수 있습니다. 이 기능은 Node.js 테스트 러너의 [`mock.timers`](https://nodejs.org/api/test.html#class-mocktimers)를 사용합니다.

여기서 시간대(`Z` 타임스탬프) 사용에 주의하세요. 일관된 시간대를 포함하지 않으면 예상치 못한 결과가 발생할 수 있습니다.

```javascript
import assert from 'node:assert/strict';
import { describe, it, mock } from 'node:test';

import ago from './ago.mjs';

describe('whatever', { concurrency: true }, () => {
  it('should choose "minutes" when that\'s the closet unit', () => {
    mock.timers.enable({ now: new Date('2000-01-01T00:02:02Z') });

    const t = ago('1999-12-01T23:59:59Z');

    assert.equal(t, '2 minutes ago');
  });
});
```

이 기능은 저장소에 체크인된 정적 픽스처와 비교할 때 특히 유용합니다. 예를 들어 [스냅샷 테스트](https://nodejs.org/api/test.html#snapshot-testing)에서 사용할 수 있습니다.


