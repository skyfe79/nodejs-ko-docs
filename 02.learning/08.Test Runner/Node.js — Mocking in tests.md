# Table of Contents

- [테스트에서의 목킹](#테스트에서의-목킹)
  - [목킹이 필요한 경우와 불필요한 경우](#목킹이-필요한-경우와-불필요한-경우)
    - [자체 코드](#자체-코드)
      - [목킹이 필요한 이유](#목킹이-필요한-이유)
      - [목킹이 불필요한 이유](#목킹이-불필요한-이유)
    - [외부 코드](#외부-코드)
      - [목킹이 필요한 이유](#목킹이-필요한-이유)
      - [목킹이 불필요한 이유](#목킹이-불필요한-이유)
    - [외부 시스템](#외부-시스템)
  - [목킹 대상](#목킹-대상)
    - [모듈과 단위](#모듈과-단위)
    - [API](#api)
    - [시간](#시간)

# 테스트에서의 목킹

목킹은 실제 객체를 모방하는 가짜 객체를 만드는 기법이다. 일반적으로 'A라는 입력이 들어오면 B라는 동작을 수행한다'는 방식으로 동작을 제어한다. 이를 통해 테스트에서 불필요한 요소를 제거하고 중요한 부분만 검증할 수 있다. '목(mock)'과 '스텁(stub)'은 엄밀히 말하면 서로 다른 '테스트 대역(test double)'이다. 스텁은 호출 여부만 추적하는 단순한 대체물이며, 목은 스텁에 가짜 구현을 추가한 것이다. 이 문서에서는 이러한 차이점은 중요하지 않으므로, 스텁도 목으로 통칭한다.

테스트는 결정론적이어야 한다. 즉, 실행 순서나 횟수에 관계없이 항상 동일한 결과가 나와야 한다. 적절한 설정과 목킹을 통해 이를 달성할 수 있다.

Node.js는 다양한 코드 조각을 목킹하는 여러 방법을 제공한다.

이 문서에서는 다음과 같은 테스트 유형을 다룬다:

| 유형 | 설명 | 예시 | 목킹 대상 |
| :--- | :--- | :--- | :--- |
| 단위 테스트 | 분리 가능한 가장 작은 코드 단위 | `const sum = (a, b) => a + b` | 자체 코드, 외부 코드, 외부 시스템 |
| 컴포넌트 테스트 | 단위 + 의존성 | `const arithmetic = (op = sum, a, b) => ops[op](a, b)` | 외부 코드, 외부 시스템 |
| 통합 테스트 | 여러 컴포넌트의 연동 | - | 외부 코드, 외부 시스템 |
| 엔드투엔드(e2e) 테스트 | 앱 + 외부 데이터 저장소, 전달 시스템 등 | 실제 외부 시스템과 연결된 앱을 Playwright 에이전트와 같은 가상 사용자가 직접 사용 | 없음 (목킹 하지 않음) |

목킹의 사용 시점과 비사용 시점에 대해서는 여러 관점이 있으며, 이에 대한 개요는 아래에서 설명한다.

## 목킹이 필요한 경우와 불필요한 경우

목킹의 주요 대상은 다음 세 가지다:

- 자체 코드
- 외부 코드
- 외부 시스템

### 자체 코드

프로젝트에서 직접 제어하는 코드를 의미한다.

```javascript
// your-project/main.mjs
import foo from './foo.mjs';

export function main() {
  const f = foo();
}
```

여기서 `foo`는 `main`의 "자체 코드" 의존성이다.

#### 목킹이 필요한 이유

`main`의 순수한 단위 테스트를 위해서는 `foo`를 목킹해야 한다. `main`의 동작만 테스트하는 것이 목적이지, `main`과 `foo`의 통합을 테스트하는 것이 아니기 때문이다.

#### 목킹이 불필요한 이유

`foo`가 단순하고, 잘 테스트되어 있으며, 자주 변경되지 않는 경우에는 목킹이 오히려 번거로울 수 있다.

`foo`를 목킹하지 않으면 더 실제적인 테스트가 가능하고 `foo`의 테스트 커버리지도 높일 수 있다. 하지만 이는 문제 발견을 어렵게 만들 수 있다. `foo`에 문제가 생기면 다른 많은 테스트도 실패하게 되어, 실제 문제의 원인을 찾기가 어려워진다.

### 외부 코드

프로젝트에서 제어할 수 없는 코드를 의미한다.

```javascript
// your-project/main.mjs
import bar from 'bar';

export function main() {
  const f = bar();
}
```

여기서 `bar`는 npm 의존성과 같은 외부 패키지다.

단위 테스트에서는 이를 항상 목킹해야 한다는 점에는 이견이 없다. 컴포넌트 테스트와 통합 테스트에서는 해당 코드의 성격에 따라 목킹 여부를 결정한다.

#### 목킹이 필요한 이유

단위 테스트의 목적은 프로젝트에서 관리하지 않는 코드의 동작을 검증하는 것이 아니다(해당 코드는 자체 테스트가 있어야 한다).

#### 목킹이 불필요한 이유

때로는 목킹이 현실적으로 불가능할 수 있다. 예를 들어, React나 Angular 같은 대형 프레임워크는 거의 목킹하지 않는다(목킹하는 것이 더 큰 문제를 일으킬 수 있다).

### 외부 시스템

데이터베이스, 환경(웹 앱의 Chromium이나 Firefox, 노드 앱의 운영체제 등), 파일 시스템, 메모리 저장소 등을 의미한다.

이상적으로는 목킹이 불필요해야 하지만, 각 테스트 케이스마다 격리된 복사본을 만드는 것은 비용과 실행 시간 등의 이유로 현실적이지 않다. 목킹 없이는 테스트들이 서로 간섭할 수 있다:

```javascript
// storage.mjs
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

```javascript
// storage.test.mjs
import assert from 'node:assert/strict';
import { describe, it } from 'node:test';

import { db } from 'db';

import { save } from './storage.mjs';

describe('storage', { concurrency: true }, () => {
  it('should retrieve the requested item', async () => {
    await db.upsert('good', 'item'); // 읽을 데이터 생성
    await db.upsert('erroneous', 'item'); // 실패 가능성 추가

    const results = await read('a', true);

    assert.equal(results.length, 1); // 잘못된 항목을 읽지 않았는지 확인

    assert.deepEqual(results[0], { key: 'good', val: 'item' });
  });

  it('should save the new item', async () => {
    const id = await save('good', 'item');

    assert.ok(id);

    const items = await db.getAll();

    assert.equal(items.length, 1); // 중복 생성되지 않았는지 확인

    assert.deepEqual(items[0], { key: 'good', val: 'item' });
  });
});
```

위 예제에서 첫 번째와 두 번째 케이스(`it()` 구문)는 동시에 실행되면서 동일한 저장소를 변경하기 때문에 서로 간섭할 수 있다(경쟁 조건). `save()`의 삽입으로 인해 정상적인 `read()`의 테스트가 항목 수 검증에서 실패할 수 있고, 그 반대의 경우도 마찬가지다.

## 목킹 대상

### 모듈과 단위

Node.js 테스트 러너의 [`mock`](https://nodejs.org/api/test.html#class-mocktracker)을 활용한다.

```javascript
import assert from 'node:assert/strict';
import { before, describe, it, mock } from 'node:test';

describe('foo', { concurrency: true }, () => {
  let barMock = mock.fn();
  let foo;

  before(async () => {
    const barNamedExports = await import('./bar.mjs')
      // 원본 기본 내보내기 제거
      .then(({ default, ...rest }) => rest);

    // 보통은 각 테스트 후 restore()나 모든 테스트 후 reset()을 수동으로 
    // 호출할 필요가 없다(node가 자동으로 처리).
    mock.module('./bar.mjs', {
      defaultExport: barMock
      // 목킹하지 않을 다른 내보내기는 유지
      namedExports: barNamedExports,
    });

    // 목 설정 후에 임포트가 시작되도록 동적 임포트를 사용해야 한다.
    ({ foo } = await import('./foo.mjs'));
  });

  it('should do the thing', () => {
    barMock.mockImplementationOnce(function bar_mock() {/* ... */});

    assert.equal(foo(), 42);
  });
});
```

### API

잘 알려지지 않은 사실이지만, `fetch`를 목킹하는 내장 방법이 있다. [`undici`](https://github.com/nodejs/undici)는 Node.js의 `fetch` 구현체다. Node.js에 포함되어 있지만 직접 노출되지는 않으므로 설치해야 한다(예: `npm install undici`).

```javascript
// endpoints.spec.mjs
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

### 시간

닥터 스트레인지처럼 시간을 제어할 수 있다. 보통 테스트 실행 시간을 인위적으로 늘리지 않기 위해 사용한다(`setTimeout()`이 3분 동안 실행되기를 기다리고 싶지는 않을 것이다). 시간 여행이 필요할 수도 있다. Node.js 테스트 러너의 [`mock.timers`](https://nodejs.org/api/test.html#class-mocktimers)를 활용한다.

시간대 설정에 주의해야 한다(타임스탬프의 'Z'). 일관된 시간대를 지정하지 않으면 예상치 못한 결과가 발생할 수 있다.

```javascript
// master-time.spec.mjs
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

이는 저장소에 저장된 정적 데이터와 비교할 때 특히 유용하다. 예를 들어 이는 [스냅샷 테스팅](https://nodejs.org/api/test.html#snapshot-testing)과 같이 저장소에 저장된 정적 데이터와 비교할 때 특히 유용하다. 예를 들어, 날짜와 시간에 의존하는 기능을 테스트할 때 실제 시간을 고정된 값으로 대체함으로써 안정적이고 예측 가능한 테스트를 작성할 수 있다. 이는 테스트의 신뢰성과 재현성을 크게 향상시킨다.