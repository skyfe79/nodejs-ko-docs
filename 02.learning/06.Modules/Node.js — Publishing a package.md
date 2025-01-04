# Table of Contents

- [[패키지 배포하기](https://nodejs.org/en/learn/modules/publishing-a-package#publishing-a-package)](#패키지-배포하기httpsnodejsorgenlearnmodulespublishing-a-packagepublishing-a-package)
  - [[적용할 방법 선택](https://nodejs.org/en/learn/modules/publishing-a-package#pick-your-fix)](#적용할-방법-선택httpsnodejsorgenlearnmodulespublishing-a-packagepick-your-fix)
    - [[CJS 소스 및 배포](https://nodejs.org/en/learn/modules/publishing-a-package#cjs-source-and-distribution)](#cjs-소스-및-배포httpsnodejsorgenlearnmodulespublishing-a-packagecjs-source-and-distribution)
    - [[ESM 소스 및 배포](https://nodejs.org/en/learn/modules/publishing-a-package#esm-source-and-distribution)](#esm-소스-및-배포httpsnodejsorgenlearnmodulespublishing-a-packageesm-source-and-distribution)
    - [[CJS 소스와 ESM 배포만](https://nodejs.org/en/learn/modules/publishing-a-package#cjs-source-and-only-esm-distribution)](#cjs-소스와-esm-배포만httpsnodejsorgenlearnmodulespublishing-a-packagecjs-source-and-only-esm-distribution)
    - [[CJS 소스와 CJS & ESM 배포](https://nodejs.org/en/learn/modules/publishing-a-package#cjs-source-and-both-cjs--esm-distribution)](#cjs-소스와-cjs--esm-배포httpsnodejsorgenlearnmodulespublishing-a-packagecjs-source-and-both-cjs--esm-distribution)
      - [[명명된 내보내기를 `exports`에 직접 연결하기](https://nodejs.org/en/learn/modules/publishing-a-package#attach-named-exports-directly-onto-exports)](#명명된-내보내기를-exports에-직접-연결하기httpsnodejsorgenlearnmodulespublishing-a-packageattach-named-exports-directly-onto-exports)
      - [간단한 ESM 래퍼 사용하기](#간단한-esm-래퍼-사용하기)
      - [[두 가지 전체 배포 방식](https://nodejs.org/en/learn/modules/publishing-a-package#two-full-distributions)](#두-가지-전체-배포-방식httpsnodejsorgenlearnmodulespublishing-a-packagetwo-full-distributions)
    - [[ESM 소스와 CJS 배포만 사용하기](https://nodejs.org/en/learn/modules/publishing-a-package#esm-source-with-only-cjs-distribution)](#esm-소스와-cjs-배포만-사용하기httpsnodejsorgenlearnmodulespublishing-a-packageesm-source-with-only-cjs-distribution)
    - [[ESM 소스와 CJS 및 ESM 배포](https://nodejs.org/en/learn/modules/publishing-a-package#esm-source-and-both-cjs--esm-distribution)](#esm-소스와-cjs-및-esm-배포httpsnodejsorgenlearnmodulespublishing-a-packageesm-source-and-both-cjs--esm-distribution)
      - [[CJS 배포만 프로퍼티 exports로 게시하기](https://nodejs.org/en/learn/modules/publishing-a-package#publish-only-a-cjs-distribution-with-property-exports)](#cjs-배포만-프로퍼티-exports로-게시하기httpsnodejsorgenlearnmodulespublishing-a-packagepublish-only-a-cjs-distribution-with-property-exports)
      - [CJS 배포판을 ESM 래퍼로 게시하기](#cjs-배포판을-esm-래퍼로-게시하기)
      - [[CJS와 ESM 배포판 모두 게시하기](https://nodejs.org/en/learn/modules/publishing-a-package#publish-both-full-cjs--esm-distributions)](#cjs와-esm-배포판-모두-게시하기httpsnodejsorgenlearnmodulespublishing-a-packagepublish-both-full-cjs--esm-distributions)
        - [[전체 패키지를 ESM으로 표시하고 CJS 내보내기를 `.cjs` 파일 확장자로 명시적으로 표시](https://nodejs.org/en/learn/modules/publishing-a-package#mark-the-whole-package-as-esm-and-specifically-mark-the-cjs-exports-as-cjs-via-the-cjs-file-extension)](#전체-패키지를-esm으로-표시하고-cjs-내보내기를-cjs-파일-확장자로-명시적으로-표시httpsnodejsorgenlearnmodulespublishing-a-packagemark-the-whole-package-as-esm-and-specifically-mark-the-cjs-exports-as-cjs-via-the-cjs-file-extension)
        - [모든 소스 코드 파일에 `.mjs` (또는 동등한) 파일 확장자 사용](#모든-소스-코드-파일에-mjs-또는-동등한-파일-확장자-사용)
      - [[Node.js 12.22.x 이전 버전](https://nodejs.org/en/learn/modules/publishing-a-package#nodejs-before-1222x)](#nodejs-1222x-이전-버전httpsnodejsorgenlearnmodulespublishing-a-packagenodejs-before-1222x)
  - [일반적인 주의사항](#일반적인-주의사항)
  - [[토끼굴 속으로](https://nodejs.org/en/learn/modules/publishing-a-package#down-the-rabbit-hole)](#토끼굴-속으로httpsnodejsorgenlearnmodulespublishing-a-packagedown-the-rabbit-hole)
    - [[이중 패키지 위험](https://nodejs.org/en/learn/modules/publishing-a-package#the-dual-package-hazard)](#이중-패키지-위험httpsnodejsorgenlearnmodulespublishing-a-packagethe-dual-package-hazard)
    - [[어떻게 여기까지 왔을까](https://nodejs.org/en/learn/modules/publishing-a-package#how-did-we-get-here)](#어떻게-여기까지-왔을까httpsnodejsorgenlearnmodulespublishing-a-packagehow-did-we-get-here)
  - [주의사항](#주의사항)
  - [[각주](https://nodejs.org/en/learn/modules/publishing-a-package#footnote-label)](#각주httpsnodejsorgenlearnmodulespublishing-a-packagefootnote-label)

# [패키지 배포하기](https://nodejs.org/en/learn/modules/publishing-a-package#publishing-a-package)

제공된 모든 `package.json` 설정(특별히 "작동하지 않음"으로 표시되지 않은 것들)은 Node.js 12.22.x(v12 최신 버전, 지원되는 가장 오래된 버전)와 17.2.0(현재 최신 버전)에서 작동합니다<sup><a id="user-content-fnref-1" data-footnote-ref="true" aria-describedby="footnote-label" href="https://nodejs.org/en/learn/modules/publishing-a-package#user-content-fn-1">1</a></sup>. 또한, 웹팩 5.53.0과 5.63.0에서도 각각 작동합니다. 이 설정들은 [JakobJingleheimer/nodejs-module-config-examples](https://github.com/JakobJingleheimer/nodejs-module-config-examples)에서 확인할 수 있습니다.

더 깊은 배경과 설명이 궁금하다면, [어떻게 여기에 도달했는가](https://nodejs.org/en/learn/modules/publishing-a-package#how-did-we-get-here)와 [토끼굴 속으로](https://nodejs.org/en/learn/modules/publishing-a-package#down-the-rabbit-hole)를 참고하세요.


## [적용할 방법 선택](https://nodejs.org/en/learn/modules/publishing-a-package#pick-your-fix)

대부분의 사용 사례를 커버하는 두 가지 주요 옵션이 있습니다:

- **CJS로 소스 코드를 작성하고 배포** (`require()` 사용): CJS는 모든 Node 버전에서 CJS와 ESM 모두에서 사용 가능합니다. [CJS 소스 및 배포](https://nodejs.org/en/learn/modules/publishing-a-package#cjs-source-and-distribution)로 이동하세요.
- **ESM으로 소스 코드를 작성하고 배포** (`import` 사용, 최상위 `await` 미사용): ESM은 Node 22.x 및 23.x에서 ESM과 CJS 모두에서 사용 가능합니다. ([`require()`로 ES 모듈 로드](https://nodejs.org/api/modules.html#loading-ecmascript-modules-using-require) 참조). [ESM 소스 및 배포](https://nodejs.org/en/learn/modules/publishing-a-package#esm-source-and-distribution)로 이동하세요.

일반적으로 **CJS 또는 ESM 중 하나의 형식만 배포**하는 것이 좋습니다. 둘 다 배포하면 [이중 패키지 위험](https://nodejs.org/en/learn/modules/publishing-a-package#the-dual-package-hazard) 및 기타 단점이 발생할 수 있습니다.

역사적인 목적으로 사용할 수 있는 다른 옵션들도 있습니다.

| 패키지 작성자가 작성하는 코드 | 패키지 사용자가 작성하는 코드 | 선택 가능한 옵션 |
| --- | --- | --- |
| `require()`를 사용한 CJS 소스 코드 | ESM: 사용자가 `import`로 패키지 사용 | [CJS 소스 및 ESM 배포](https://nodejs.org/en/learn/modules/publishing-a-package#cjs-source-and-only-esm-distribution) |
| CJS & ESM: 사용자가 `require()` 또는 `import`로 패키지 사용 | [CJS 소스 및 CJS & ESM 배포](https://nodejs.org/en/learn/modules/publishing-a-package#cjs-source-and-both-cjs-amp-esm-distribution) |
| `import`를 사용한 ESM 소스 코드 | CJS: 사용자가 `require()`로 패키지 사용 (최상위 `await` 사용) | [ESM 소스 및 CJS 배포](https://nodejs.org/en/learn/modules/publishing-a-package#esm-source-with-only-cjs-distribution) |
| CJS & ESM: 사용자가 `require()` 또는 `import`로 패키지 사용 | [ESM 소스 및 CJS & ESM 배포](https://nodejs.org/en/learn/modules/publishing-a-package#esm-source-and-both-cjs-amp-esm-distribution) |


### [CJS 소스 및 배포](https://nodejs.org/en/learn/modules/publishing-a-package#cjs-source-and-distribution)

가장 기본적인 설정은 [`"name"`](https://nodejs.org/api/packages.html#name) 필드만 포함할 수 있습니다. 하지만 복잡하지 않을수록 좋습니다. 기본적으로 `"exports"` 필드를 통해 패키지의 내보내기를 선언하면 됩니다.

**작동 예제**: [cjs-with-cjs-distro](https://github.com/JakobJingleheimer/nodejs-module-config-examples/tree/main/packages/cjs/cjs-distro)

최소한의 package.json  
고급 (자세한) package.json

```json
{
  "name": "cjs-source-and-distribution"
  // "main": "./index.js"
}
```

`packageJson.exports["."] = filepath`는 `packageJson.exports["."].default = filepath`의 줄임 표현입니다.


### [ESM 소스 및 배포](https://nodejs.org/en/learn/modules/publishing-a-package#esm-source-and-distribution)

간단하고 검증된 방법입니다.

Node.js v23.0.0부터는 정적 ESM(최상위 `await`를 사용하지 않는 코드)을 `require`로 불러올 수 있습니다. 자세한 내용은 [ECMAScript 모듈을 `require()`로 불러오기](https://nodejs.org/api/modules.html#loading-ecmascript-modules-using-require)를 참고하세요.

이 방법은 위에서 설명한 CJS-CJS 구성과 거의 동일하지만, 한 가지 작은 차이점이 있습니다. 바로 [`"type"`](https://nodejs.org/api/packages.html#type) 필드입니다.

**작동 예제**: [esm-with-esm-distro](https://github.com/JakobJingleheimer/nodejs-module-config-examples/tree/main/packages/esm/esm-distro)

최소한의 package.json  
고급 (상세) package.json

```json
{
  "name": "esm-source-and-distribution",
  "type": "module"
  // "main": "./index.js"
}
```

JSON 복사

ESM은 이제 CJS와 "역호환"됩니다. CJS 모듈은 이제 플래그 없이도 [ES 모듈을 `require()`로 불러올 수 있습니다](https://nodejs.org/api/modules.html#loading-ecmascript-modules-using-require). 이 기능은 Node.js v23.0.0과 v22.12.0부터 사용 가능합니다.


### [CJS 소스와 ESM 배포만](https://nodejs.org/en/learn/modules/publishing-a-package#cjs-source-and-only-esm-distribution)

이 방법은 약간의 섬세함이 필요하지만 꽤 직관적입니다. 이는 새로운 표준을 목표로 하는 오래된 프로젝트나, 단순히 CJS를 선호하지만 다른 환경을 위해 배포하는 작성자에게 적합한 선택일 수 있습니다.

**작동 예제**: [cjs-with-esm-distro](https://github.com/JakobJingleheimer/nodejs-module-config-examples/tree/main/packages/cjs/esm-distro)

최소한의 package.json과 상세한 package.json 예제:

```json
{
  "name": "cjs-source-with-esm-distribution",
  "main": "./dist/index.mjs"
}
```

[`.mjs`](https://nodejs.org/api/esm.html#enabling) 파일 확장자는 **어떤** 다른 설정도 무시하고 파일을 ESM으로 처리합니다. 이 파일 확장자를 사용하는 것은 `packageJson.exports.import`가 파일이 ESM임을 나타내지 않기 때문에 필요합니다. (보편적인 오해와는 달리) 이는 단지 패키지가 임포트될 때 사용할 파일을 지정할 뿐입니다. (ESM은 CJS를 임포트할 수 있습니다. 아래 [주의사항](https://nodejs.org/en/learn/modules/publishing-a-package#gotchas)을 참고하세요.)


### [CJS 소스와 CJS & ESM 배포](https://nodejs.org/en/learn/modules/publishing-a-package#cjs-source-and-both-cjs--esm-distribution)

두 가지 환경(CommonJS와 ESM)에서 모두 "네이티브"로 동작하는 배포를 위해 직접적으로 제공할 수 있는 몇 가지 방법이 있습니다.


#### [명명된 내보내기를 `exports`에 직접 연결하기](https://nodejs.org/en/learn/modules/publishing-a-package#attach-named-exports-directly-onto-exports)

전통적인 방법이지만 약간의 세련된 기교가 필요합니다. 이 방법은 `module.exports` 전체를 재할당하는 대신, 기존 `module.exports`에 속성을 추가하는 것을 의미합니다.

**작동 예제**: [cjs-with-dual-distro (properties)](https://github.com/JakobJingleheimer/nodejs-module-config-examples/tree/main/packages/cjs/dual/property-distro)

최소한의 package.json과 상세한 package.json 예제:

```json
{
  "name": "cjs-source-with-esm-via-properties-distribution",
  "main": "./dist/cjs/index.js"
}
```

**장점**:

- 패키지 크기가 작음
- 간단하고 쉬움 (약간의 문법적 제약을 감수한다면 가장 적은 노력이 듬)
- [이중 패키지 위험](https://nodejs.org/en/learn/modules/publishing-a-package#the-dual-package-hazard)을 방지

**단점**:

- 매우 특정한 문법이 필요함 (소스 코드나 번들러에서의 기교가 필요할 수 있음)

때로는 CJS 모듈이 `module.exports`를 다른 것으로 재할당할 수 있습니다. 예를 들어 객체나 함수로 재할당하는 경우가 있습니다:

```javascript
const someObject = {
  foo() {},
  bar() {},
  qux() {},
};

module.exports = someObject;
```

Node.js는 CJS에서 명명된 내보내기를 [특정 패턴을 찾는 정적 분석](https://github.com/nodejs/cjs-module-lexer/tree/main?tab=readme-ov-file#parsing-examples)을 통해 감지합니다. 위 예제는 이를 회피합니다. 명명된 내보내기를 감지 가능하게 만들려면 다음과 같이 작성합니다:

```javascript
module.exports.foo = function foo() {};
module.exports.bar = function bar() {};
module.exports.qux = function qux() {};
```


#### 간단한 ESM 래퍼 사용하기

복잡한 설정과 적절한 균형을 맞추기 어려운 문제가 있습니다.

**작동 예제**: [cjs-with-dual-distro (래퍼)](https://github.com/JakobJingleheimer/nodejs-module-config-examples/tree/main/packages/cjs/dual/wrapper-distro)

최소한의 package.json과 상세한 package.json 예제입니다.

```json
{
  "name": "cjs-with-wrapper-dual-distro",
  "exports": {
    ".": {
      "import": "./dist/esm/wrapper.mjs",
      "require": "./dist/cjs/index.js",
      "default": "./dist/cjs/index.js"
    }
  }
}
```

**장점**:

- 패키지 크기가 작아짐

**단점**:

- 복잡한 번들러 작업이 필요할 수 있음 (Webpack에서 이를 자동화할 수 있는 기존 옵션을 찾지 못함)

번들러의 CJS 출력이 Node.js의 명명된 내보내기 감지를 피할 때, ESM 래퍼를 사용하여 ESM 소비자를 위해 명시적으로 명명된 내보내기를 다시 내보낼 수 있습니다.

CJS가 객체를 내보낼 때 (이 객체는 ESM의 `default`로 별칭이 지정됨), 래퍼에서 객체의 모든 멤버에 대한 참조를 로컬로 저장한 다음, 이를 다시 내보내어 ESM 소비자가 이름으로 모든 멤버에 접근할 수 있도록 할 수 있습니다.

```javascript
import cjs from '../cjs/index.js';

const { a, b, c /* … */ } = cjs;

export { a, b, c /* … */ };
```

**그러나**, 이 방법은 라이브 바인딩을 깨뜨립니다: `cjs.a`에 대한 재할당이 `esmWrapper.a`에 반영되지 않습니다.


#### [두 가지 전체 배포 방식](https://nodejs.org/en/learn/modules/publishing-a-package#two-full-distributions)

여러 파일을 한꺼번에 넣고 최선의 결과를 기대하는 방식입니다. 이 방법은 CJS에서 CJS와 ESM으로 전환하는 가장 일반적이고 쉬운 옵션 중 하나지만, 그만큼 비용이 따릅니다. 이 방법은 거의 좋은 아이디어가 아닙니다.

**작동 예제**: [cjs-with-dual-distro (double)](https://github.com/JakobJingleheimer/nodejs-module-config-examples/tree/main/packages/cjs/dual/double-distro)

최소한의 `package.json`과 상세한 `package.json` 예제입니다.

```json
{
  "name": "cjs-with-full-dual-distro",
  "exports": {
    ".": {
      "import": "./dist/esm/index.mjs",
      "require": "./dist/cjs/index.js",
      "default": "./dist/cjs/index.js"
    }
  }
}
```

**장점**:

-   간단한 번들러 설정

**단점**:

-   패키지 크기가 커짐 (기본적으로 두 배)
-   [이중 패키지 위험](https://nodejs.org/en/learn/modules/publishing-a-package#the-dual-package-hazard)에 취약

또는 `"default"`와 `"node"` 키를 사용할 수 있습니다. 이 방식은 덜 직관적이지만, Node.js는 항상 `"node"` 옵션을 선택하고 (이 옵션은 항상 작동함), Node.js가 아닌 도구는 `"default"`를 선택합니다. **이 방식은 이중 패키지 위험을 방지합니다.**

최소한의 `package.json`과 상세한 `package.json` 예제입니다.

```json
{
  "name": "cjs-with-alt-full-dual-distro",
  "exports": {
    ".": {
      "node": "./dist/cjs/index.js",
      "default": "./dist/esm/index.mjs"
    }
  }
}
```


### [ESM 소스와 CJS 배포만 사용하기](https://nodejs.org/en/learn/modules/publishing-a-package#esm-source-with-only-cjs-distribution)

이제는 예전과 상황이 달라졌습니다.

[ESM 소스와 CJS & ESM 배포](https://nodejs.org/en/learn/modules/publishing-a-package#esm-source-and-both-cjs-amp-esm-distribution)와 거의 동일한 설정(2가지 옵션이 있음)을 사용하되, `packageJson.exports.import`를 제외하면 됩니다.

💡 `"type": "module"`<sup><a id="user-content-fnref-2" data-footnote-ref="true" aria-describedby="footnote-label" href="https://nodejs.org/en/learn/modules/publishing-a-package#user-content-fn-2">2</a></sup>을 `.cjs` 파일 확장자(CommonJS 파일용)와 함께 사용하면 가장 좋은 결과를 얻을 수 있습니다. 자세한 이유는 [Down the rabbit-hole](https://nodejs.org/en/learn/modules/publishing-a-package#down-the-rabbit-hole)과 [Gotchas](https://nodejs.org/en/learn/modules/publishing-a-package#gotchas) 섹션을 참고하세요.

**작동 예제**: [esm-with-cjs-distro](https://github.com/JakobJingleheimer/nodejs-module-config-examples/tree/main/packages/esm/cjs-distro)


### [ESM 소스와 CJS 및 ESM 배포](https://nodejs.org/en/learn/modules/publishing-a-package#esm-source-and-both-cjs--esm-distribution)

소스 코드가 자바스크립트가 아닌 언어(예: TypeScript)로 작성된 경우, 해당 언어에 특화된 파일 확장자(예: `.ts`)를 사용해야 하기 때문에 옵션이 제한될 수 있습니다. 또한 `.mjs`에 해당하는 파일이 없을 수도 있습니다.

[CJS 소스와 CJS 및 ESM 배포](https://nodejs.org/en/learn/modules/publishing-a-package#cjs-source-and-both-cjs-amp-esm-distribution)와 마찬가지로, 여러분은 동일한 옵션을 사용할 수 있습니다.


#### [CJS 배포만 프로퍼티 exports로 게시하기](https://nodejs.org/en/learn/modules/publishing-a-package#publish-only-a-cjs-distribution-with-property-exports)

이 옵션은 [CJS 소스와 CJS & ESM 배포의 프로퍼티 exports](https://nodejs.org/en/learn/modules/publishing-a-package#attach-named-exports-directly-onto-raw-exports-endraw-)와 거의 동일합니다. 유일한 차이점은 `package.json`에서 `"type": "module"`을 사용한다는 점입니다.

이 출력을 생성하는 빌드 도구는 일부만 지원합니다. [Rollup](https://www.rollupjs.org/)은 commonjs를 타겟으로 할 때 기본적으로 호환되는 출력을 생성합니다. Webpack은 [v5.66.0+](https://github.com/webpack/webpack/releases/tag/v5.66.0)부터 새로운 [`commonjs-static`](https://webpack.js.org/configuration/output/#type-commonjs-static) 출력 타입을 통해 호환되는 출력을 생성합니다 (이전 버전에서는 호환되는 출력을 생성할 수 없었습니다). 현재 [esbuild](https://esbuild.github.io/)에서는 이 기능을 사용할 수 없습니다 (esbuild는 비정적 `exports`를 생성합니다).

아래의 동작 예제는 Webpack의 최근 릴리스 이전에 생성되었기 때문에 Rollup을 사용합니다 (Webpack 옵션도 추가할 예정입니다).

이 예제들은 `.js` 확장자를 사용하는 자바스크립트 파일을 가정합니다. `package.json`의 `"type"`은 이러한 파일들이 어떻게 해석될지 결정합니다:

`"type":"commonjs"` + `.js` → `cjs`  
`"type":"module"` + `.js` → `mjs`

만약 모든 파일이 명시적으로 `.cjs` 또는 `.mjs` 확장자를 사용한다면 (`.js`를 사용하지 않는다면), `"type"`은 불필요합니다.

**동작 예제**: [esm-with-cjs-distro](https://github.com/JakobJingleheimer/nodejs-module-config-examples/tree/main/packages/esm/dual/property-distro)

최소한의 `package.json`:

```json
{
  "name": "esm-with-cjs-distribution",
  "type": "module",
  "main": "./dist/index.cjs"
}
```

💡 `"type": "module"`<sup><a id="user-content-fnref-2-2" data-footnote-ref="true" aria-describedby="footnote-label" href="https://nodejs.org/en/learn/modules/publishing-a-package#user-content-fn-2">2</a></sup>을 `.cjs` 파일 확장자와 함께 사용하면 (commonjs 파일의 경우) 최상의 결과를 얻을 수 있습니다. 자세한 내용은 [Down the rabbit-hole](https://nodejs.org/en/learn/modules/publishing-a-package#down-the-rabbit-hole)과 [Gotchas](https://nodejs.org/en/learn/modules/publishing-a-package#gotchas)를 참고하세요.


#### CJS 배포판을 ESM 래퍼로 게시하기

이 방법은 복잡한 경우가 많고, 일반적으로 최선의 선택은 아닙니다.

이 방식은 [CJS 소스와 ESM 래퍼를 사용한 이중 배포](https://nodejs.org/en/learn/modules/publishing-a-package#use-a-simple-esm-wrapper)와 거의 동일하지만, `"type": "module"` 설정과 `package.json`에서 `.cjs` 파일 확장자를 사용하는 점에서 미묘한 차이가 있습니다.

**작동 예제**: [esm-with-dual-distro (wrapper)](https://github.com/JakobJingleheimer/nodejs-module-config-examples/tree/main/packages/esm/dual/wrapper-distro)

최소한의 `package.json`과 상세한 `package.json` 예제는 다음과 같습니다.

```json
{
  "name": "esm-with-cjs-and-esm-wrapper-distribution",
  "type": "module",
  "exports": {
    ".": {
      "import": "./dist/esm/wrapper.js",
      "require": "./dist/cjs/index.cjs",
      "default": "./dist/cjs/index.cjs"
    }
  }
}
```

💡 `"type": "module"`을 사용하고 `.cjs` 파일 확장자(CommonJS 파일용)를 함께 사용하면 최상의 결과를 얻을 수 있습니다. 자세한 이유는 [Down the rabbit-hole](https://nodejs.org/en/learn/modules/publishing-a-package#down-the-rabbit-hole)과 [Gotchas](https://nodejs.org/en/learn/modules/publishing-a-package#gotchas) 섹션을 참고하세요.


#### [CJS와 ESM 배포판 모두 게시하기](https://nodejs.org/en/learn/modules/publishing-a-package#publish-both-full-cjs--esm-distributions)

여러 가지 요소를 추가하고(예상치 못한 결과를 포함하여) 최선을 바라는 방식입니다. 이 방법은 ESM을 CJS와 ESM으로 변환하는 옵션 중 가장 일반적이고 쉬운 방법이지만, 그만큼 비용이 따릅니다. 이 방법은 거의 좋은 아이디어가 아닙니다.

패키지 설정과 관련해서는 주로 개인적인 선호에 따라 몇 가지 옵션이 있습니다.


##### [전체 패키지를 ESM으로 표시하고 CJS 내보내기를 `.cjs` 파일 확장자로 명시적으로 표시](https://nodejs.org/en/learn/modules/publishing-a-package#mark-the-whole-package-as-esm-and-specifically-mark-the-cjs-exports-as-cjs-via-the-cjs-file-extension)

이 방법은 개발자 경험에 가장 적은 부담을 줍니다.

이 방법은 빌드 도구가 `.cjs` 파일 확장자를 가진 배포 파일을 생성해야 함을 의미합니다. 이는 여러 빌드 도구를 연결하거나 파일을 이동/이름 변경하여 `.cjs` 확장자를 추가하는 단계가 필요할 수 있습니다 (예: `mv ./dist/index.js ./dist/index.cjs`). 이는 [Rollup](https://rollupjs.org/)이나 [간단한 쉘 스크립트](https://stackoverflow.com/q/21985492)를 사용하여 출력된 파일을 이동/이름 변경하는 추가 단계로 해결할 수 있습니다.

`.cjs` 파일 확장자 지원은 Node.js 12.0.0에서 추가되었으며, 이를 사용하면 ESM이 파일을 CommonJS로 올바르게 인식합니다 (`import { foo } from './foo.cjs'`가 작동함). 그러나 `require()`는 `.js` 파일처럼 `.cjs`를 자동으로 해석하지 않으므로, 일반적으로 CommonJS에서 생략하던 파일 확장자를 생략할 수 없습니다: `require('./foo')`는 실패하지만, `require('./foo.cjs')`는 작동합니다. 패키지의 내보내기에서 이를 사용하는 데는 단점이 없습니다: `packageJson.exports` (그리고 `packageJson.main`)는 파일 확장자를 요구하며, 사용자는 패키지의 `"name"` 필드를 통해 패키지를 참조하므로 (그들은 이에 대해 전혀 알지 못합니다).

**작동 예제**: [esm-with-dual-distro](https://github.com/JakobJingleheimer/nodejs-module-config-examples/tree/main/packages/esm/dual/double-distro)

간단한 import & require package.json  
상세한 import & require package.json

```json
{
  "type": "module",
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/index.cjs"
    }
  }
}
```

또는 `"default"`와 `"node"` 키를 사용할 수 있습니다. 이 방법은 덜 직관적이지만, Node.js는 항상 `"node"` 옵션을 선택하며 (이것은 항상 작동함), Node.js가 아닌 도구는 `"default"`를 선택합니다. **이 방법은 이중 패키지 위험을 방지합니다.**

간단한 default & node package.json  
상세한 default & node package.json

```json
{
  "type": "module",
  "exports": {
    ".": {
      "node": "./dist/index.cjs",
      "default": "./dist/esm/index.js"
    }
  }
}
```

💡 `"type": "module"`<sup><a id="user-content-fnref-2-4" data-footnote-ref="true" aria-describedby="footnote-label" href="https://nodejs.org/en/learn/modules/publishing-a-package#user-content-fn-2">2</a></sup>을 `.cjs` 파일 확장자 (CommonJS 파일용)와 함께 사용하면 최상의 결과를 얻을 수 있습니다. 자세한 내용은 [Down the rabbit-hole](https://nodejs.org/en/learn/modules/publishing-a-package#down-the-rabbit-hole)과 [Gotchas](https://nodejs.org/en/learn/modules/publishing-a-package#gotchas)를 참조하세요.


##### 모든 소스 코드 파일에 `.mjs` (또는 동등한) 파일 확장자 사용

이 설정은 [CJS 소스와 CJS & ESM 배포](https://nodejs.org/en/learn/modules/publishing-a-package#cjs-source-and-both-cjs-amp-esm-distribution)와 동일합니다.

**비자바스크립트 소스 코드**: 비자바스크립트 언어 자체의 설정에서 입력 파일이 ESM임을 인식하거나 지정해야 합니다.


#### [Node.js 12.22.x 이전 버전](https://nodejs.org/en/learn/modules/publishing-a-package#nodejs-before-1222x)

🛑 이렇게 하지 마세요: Node.js 12.x 이전 버전은 더 이상 지원이 종료되었으며, 심각한 보안 취약점에 노출될 수 있습니다.

만약 여러분이 보안 연구자로서 Node.js v12.22.x 이전 버전을 조사해야 한다면, 설정에 도움을 받기 위해 언제든지 문의해 주세요.


## 일반적인 주의사항

**구문 감지**는 올바른 패키지 설정을 대체할 수 없습니다. 구문 감지는 완벽하지 않으며, 상당한 성능 비용이 발생할 수 있습니다.

`package.json`에서 `"exports"`를 사용할 때는 일반적으로 `"./package.json": "./package.json"`을 포함하는 것이 좋습니다. 이렇게 하면 패키지를 가져올 수 있습니다. (`module.findPackageJSON`은 이 제한에 영향을 받지 않지만, `import`가 더 편리할 수 있습니다.)

`"exports"`는 `"main"`보다 권장될 수 있습니다. 왜냐하면 외부에서 내부 코드에 접근하는 것을 방지할 수 있기 때문입니다. 따라서 사용자가 의존해서는 안 되는 것에 의존하지 않는다는 것을 상대적으로 확신할 수 있습니다. 만약 그런 기능이 필요하지 않다면, `"main"`이 더 간단하고 더 나은 선택일 수 있습니다.

`"engines"` 필드는 패키지가 호환되는 Node.js 버전을 사람과 기계 모두에게 알려줍니다. 사용하는 패키지 매니저에 따라, 호환되지 않는 Node.js 버전을 사용할 때 예외가 발생하여 설치가 실패할 수 있습니다. 이는 사용자에게 매우 도움이 될 수 있습니다. 이 필드를 포함하면, 오래된 Node.js 버전을 사용하는 사용자가 패키지를 사용하지 못하는 문제를 크게 줄일 수 있습니다.


## [토끼굴 속으로](https://nodejs.org/en/learn/modules/publishing-a-package#down-the-rabbit-hole)

Node.js와 관련하여 해결해야 할 4가지 문제가 있습니다:

1. 소스 코드 파일의 형식 결정 (작성자가 자신의 코드를 실행할 때)
2. 배포 파일의 형식 결정 (코드 사용자가 받게 될 파일)
3. `require()`로 불러올 때 배포 코드 공개 (사용자는 CJS를 기대함)
4. `import`로 불러올 때 배포 코드 공개 (사용자는 ESM을 원할 가능성이 높음)

⚠️ 처음 두 문제는 마지막 두 문제와 **독립적**입니다.

파일을 불러오는 방식이 파일이 해석되는 형식을 결정하지 않습니다:

- **package.json의** **`exports.require`** **≠** **`CJS`**. `require()`는 파일을 무조건 CJS로 해석하지 않으며, 그럴 수도 없습니다. 예를 들어, `require('foo.json')`은 파일을 JSON으로 올바르게 해석하며, CJS로 해석하지 않습니다. `require()`를 호출하는 모듈은 당연히 CJS여야 하지만, 불러오는 파일이 반드시 CJS일 필요는 없습니다.
- **package.json의** **`exports.import`** **≠** **`ESM`**. `import`도 마찬가지로 파일을 무조건 ESM으로 해석하지 않으며, 그럴 수도 없습니다. `import`는 CJS, JSON, WASM, 그리고 ESM을 모두 불러올 수 있습니다. `import` 문을 포함하는 모듈은 당연히 ESM이어야 하지만, 불러오는 파일이 반드시 ESM일 필요는 없습니다.

따라서 `require`나 `import`를 언급하거나 이름에 포함한 설정 옵션을 볼 때, 그것들이 CJS와 ES 모듈을 *결정*하기 위한 것이라고 단정하지 마세요.

⚠️ 패키지 설정에 `"exports"` 필드나 필드 세트를 추가하면, 명시적으로 나열되지 않은 하위 경로에 대한 패키지 내부 깊은 경로 접근이 [차단됩니다](https://nodejs.org/api/packages.html#package-entry-points). 이는 호환성을 깨뜨리는 변경이 될 수 있습니다.

⚠️ CJS와 ESM을 모두 배포할지 신중히 고려하세요: 이는 [이중 패키지 위험](https://nodejs.org/en/learn/modules/publishing-a-package#the-dual-package-hazard)을 초래할 가능성이 있습니다 (특히 잘못 구성되고 사용자가 꼼수를 부리려고 할 때). 이는 사용 중인 프로젝트에서 매우 혼란스러운 버그를 일으킬 수 있으며, 특히 패키지가 완벽하게 구성되지 않았을 때 더욱 그렇습니다. 사용자는 중간 패키지가 "다른" 형식의 패키지를 사용함으로써 예기치 못한 문제에 직면할 수도 있습니다 (예: 사용자가 ESM 배포판을 사용하는데, 사용 중인 다른 패키지가 CJS 배포판을 사용하는 경우). 패키지가 상태를 유지하는 경우, CJS와 ESM 배포판을 모두 사용하면 병렬 상태가 발생할 수 있습니다 (이는 거의 확실히 의도하지 않은 결과입니다).


### [이중 패키지 위험](https://nodejs.org/en/learn/modules/publishing-a-package#the-dual-package-hazard)

애플리케이션이 CommonJS와 ES 모듈 소스를 모두 제공하는 패키지를 사용할 때, 두 인스턴스가 모두 로드되면 특정 버그가 발생할 위험이 있습니다. 이 위험은 `const pkgInstance = require('pkg')`로 생성된 `pkgInstance`와 `import pkgInstance from 'pkg'`(또는 `'pkg/module'`과 같은 대체 메인 경로)로 생성된 `pkgInstance`가 동일하지 않다는 사실에서 비롯됩니다. 이를 "이중 패키지 위험"이라고 부르며, 동일한 런타임 환경 내에서 동일한 패키지의 두 인스턴스가 로드될 수 있습니다. 애플리케이션이나 패키지가 의도적으로 두 인스턴스를 직접 로드하는 경우는 드물지만, 애플리케이션이 한 인스턴스를 로드하고 애플리케이션의 의존성이 다른 인스턴스를 로드하는 경우는 흔합니다. 이 위험은 Node.js가 CommonJS와 ES 모듈을 혼합하여 사용할 수 있기 때문에 발생하며, 예상치 못한 혼란스러운 동작을 초래할 수 있습니다.

패키지의 메인 익스포트가 생성자인 경우, 두 인스턴스로 생성된 객체를 `instanceof`로 비교하면 `false`가 반환됩니다. 또한 익스포트가 객체인 경우, 한 인스턴스에 추가된 속성(예: `pkgInstance.foo = 3`)은 다른 인스턴스에 존재하지 않습니다. 이는 모든 CommonJS 또는 모든 ES 모듈 환경에서 `import`와 `require` 문이 작동하는 방식과 다르며, 따라서 사용자에게 예상치 못한 결과를 초래합니다. 또한 [Babel](https://babeljs.io/)이나 [`esm`](https://github.com/standard-things/esm#readme)과 같은 도구를 통해 트랜스파일링을 사용할 때 사용자가 익숙한 동작과도 다릅니다.


### [어떻게 여기까지 왔을까](https://nodejs.org/en/learn/modules/publishing-a-package#how-did-we-get-here)

[CommonJS (CJS)](https://wiki.commonjs.org/wiki/Modules)는 ECMAScript Modules (ESM)보다 훨씬 이전에 만들어졌습니다. 당시 자바스크립트는 아직 성장 중이었고, CJS와 jQuery는 불과 3년 차이로 탄생했습니다. CJS는 공식적인 [TC39](https://tc39.es) 표준이 아니며, 제한된 플랫폼(특히 Node.js)에서만 지원됩니다. 반면 ESM은 표준으로서 몇 년 동안 도입되어 왔으며, 현재 모든 주요 플랫폼(브라우저, Deno, Node.js 등)에서 지원됩니다. 이는 ESM이 거의 모든 곳에서 실행될 수 있음을 의미합니다.

ESM이 CJS를 효과적으로 대체할 것이라는 점이 명확해지면서, 많은 개발자들이 ESM 스펙의 특정 부분이 확정되기 전에 이를 도입하려고 시도했습니다. 이로 인해 더 나은 정보가 제공되면서(주로 초기에 도입한 개발자들의 경험을 통해) 최선의 추측에서 스펙에 맞추는 방식으로 변화가 이루어졌습니다.

추가적인 복잡성은 번들러(bundler)에서 비롯됩니다. 역사적으로 번들러는 이 영역의 많은 부분을 관리해왔습니다. 그러나 이전에 번들러가 필요했던 많은 기능들이 이제는 네이티브로 제공됩니다. 그럼에도 번들러는 여전히(아마도 항상) 일부 작업에 필요합니다. 불행히도, 더 이상 필요하지 않은 기능들이 오래된 번들러 구현에 깊이 뿌리내려 있어, 때로는 지나치게 도움을 주거나 심지어 안티패턴(번들러 작성자들조차 라이브러리 번들링을 권장하지 않는 경우)이 되기도 합니다. 이에 대한 이유와 방법은 별도의 글로 다룰 만한 주제입니다.


## 주의사항

`package.json`의 `"type"` 필드는 `.js` 파일 확장자가 `commonjs` 또는 ES `module` 중 어느 것을 의미할지 결정합니다. CJS와 ESM을 모두 포함하는 이중/혼합 패키지에서 이 필드를 잘못 사용하는 경우가 매우 흔합니다.

```json
{
  "type": "module",
  "main": "./dist/CJS/index.js",
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/cjs/index.js",
      "default": "./dist/cjs/index.js"
    },
    "./package.json": "./package.json"
  }
}
```

이 설정은 작동하지 않습니다. `"type": "module"`로 인해 `packageJson.main`, `packageJson.exports["."].require`, 그리고 `packageJson.exports["."].default`가 ESM으로 해석되기 때문입니다. 하지만 실제로는 CJS입니다.

`"type": "module"`을 제외하면 반대의 문제가 발생합니다.

```json
{
  "main": "./dist/CJS/index.js",
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/cjs/index.js",
      "default": "./dist/cjs/index.js"
    },
    "./package.json": "./package.json"
  }
}
```

이 설정도 작동하지 않습니다. `packageJson.exports["."].import`가 CJS로 해석되기 때문입니다. 하지만 실제로는 ESM입니다.


## [각주](https://nodejs.org/en/learn/modules/publishing-a-package#footnote-label)

1. Node.js v13.0–13.6 버전에는 `packageJson.exports["."]`가 첫 번째 항목으로 상세 설정 옵션(객체 형태)을, 두 번째 항목으로 "default"(문자열 형태)를 포함한 배열이어야 하는 버그가 있었습니다. 자세한 내용은 [nodejs/modules#446](https://github.com/nodejs/modules/issues/446)을 참고하세요. [↩](https://nodejs.org/en/learn/modules/publishing-a-package#user-content-fnref-1)

2. package.json의 `"type"` 필드는 `.js` 파일 확장자의 의미를 변경합니다. 이는 [HTML script 엘리먼트의 type 속성](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#attr-type)과 유사합니다. [↩](https://nodejs.org/en/learn/modules/publishing-a-package#user-content-fnref-2) [↩<sup>2</sup>](https://nodejs.org/en/learn/modules/publishing-a-package#user-content-fnref-2-2) [↩<sup>3</sup>](https://nodejs.org/en/learn/modules/publishing-a-package#user-content-fnref-2-3) [↩<sup>4</sup>](https://nodejs.org/en/learn/modules/publishing-a-package#user-content-fnref-2-4)


