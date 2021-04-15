# Modules

자바스크립트에서의 개념으로서 모듈은 오래고 복잡한 역사를 가지고 있어서 어떠한 하나의 정의나 기술을 어렵게 하고 있다.
수많은 경쟁적인 구현체들이 발표되었고 다수의 상호 호환이 불가능한 시스템이 이 기반위에 개발되었다.
궁극적으로, 이 챕터는 다른 챕터보다 더 많은 배경지식을 필요로 한다.

먼저 모듈에 대한 개관을 설명한 후 우리는 타입스크립트에 특징적인 주요 특성에 촛점을 맞출 것이다.

__toc__

## A Brief History of Modules

자바스크립트의 역사 내내 존재해 왔던 모듈의 순차적 살펴보기를 간단히 함으로써 이 거대한 물건에 대해 우리가 어떻게 다가갈지 살펴보자.

### No Modules

처음에는, HTML 파일에 단순히 `<script>` 태그가 있었다.
모든 자바스크립트 코드는 페이지가 그려지기 전에 적재되고 실행되었고, 모든 파일들은 전역 범위에서 동작 했다.
프로그램의 서로 다른 부분은 전역 변수를 통하여 서로 이야기 했다.

이것은 오늘날 `module: "none"`으로 대표된다 - 모듈이 없는 코드이다.

이러한 방법은 간단한 프로그램들에게는 좋았으나 이러한 접근방법은 금방 한계를 맞이했다.
한꺼번에 모든 자바스크립트 파일을 적재하는 것은 페이지 적재 시간에 나쁜 영향을 주었고 서로 다른 파일들은 서로 전역 변수가 충돌하지 않도록 세심한 주의가 필요했다.
더 나쁜 것은, 하나의 자바스크립트 파일에 의존성을 기술하는 방법이 없었다.
개발자들은 어떠한 HTML 파일이든 그들이 적절한 선결 사항을 다 만족하고 있다는 것을 확신시켜야만 했다.

### AMD

```js
// AMD(비동기 모듈 정의) 모듈의 예제
define("my_module", ["dependency_1", "dependency_2"], function(dep1, dep2) {
  return {
    name: "My Awesome Module",
    greet: () => {
      alert("Hello, world!");
    }
  };
});
```

[AMD, the asynchronous module definition](https://requirejs.org/docs/whyamd.html) 는 이러한 문제들의 다수를 해결 했다.
이것은 프로그램의 각각의 부분이 자신의 의존성을 선언하고, 모듈들이 비동기적으로 적재되게끔 허용했다.
각각의 모듈은 자신의 명시적으로 작성된 함수 몸체에서 실행되었고, 전역 공간의 충돌을 피할 수 있었다.
개발자는 원한다면 다수의 모듈들을 자바스크립트 파일에서와 같은 방식으로 작성하는 것 또한 가능했다.

이것이 [RequireJS](https://requirejs.org)를 사용한 표준 구현이었고 몇몇 모듈 번들러들이 지원하였다.

AMD는 주어진 파일에서 모듈 이름이 식별되는 방법에 있어서 매우 다양한 구성을 허용한다.
실제로, 하나의 파일이 다수의 모듈 정의를 제공할 수 있으며, 파일 찾기는 어떤 모듈 이름을 찾기위해서는 전혀 발생하지 않았다.

### CommonJS

```js
// An example CommonJS module
const fs = require("fs");

module.exports = function() {
  return fs.readFile("someFile.txt");
};
```

[Node.js](https://nodejs.org)는 다른 접근 방법을 취했는데 [CommonJS](https://nodejs.org/docs/latest/api/modules.html)라고 알려진 방식을 구현 했다.
여기서, 모듈들은 `require` 함수를 호출하여 동기적으로 적재되고 이것의 의미는 모듈의 의존성은 *declarative* 라기 보다는 *imperative* 하다는 것이다.
또한 파일과 모듈간의 일대일 관계가 더욱 명확 했다.

CommonJS 규격 그 자체가 모듈 이름과 파일 경로 간의 관계를 정의하지는 않았지만, 그것은 NodeJS의 모듈 이름 찾기 알고리즘이 암시하는 방식이 규격이라고 공통적으로 받아들여졌다(상대 경로가 아닌 `node_modules` 에서의 찾기도 포함).

### UMD

```js
// UMD 는 전역이거나 AMD환경을 포함하는 것이다
// 다음은 https://github.com/umdjs/umd/blob/master/templates/amdWebGlobal.js로 부터 인용된 것임
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        define(['b'], function (b) {
            return (root.amdWebGlobal = factory(b));
        });
    } else {
        root.amdWebGlobal = factory(root.b);
    }
}(typeof self !== 'undefined' ? self : this, function (b) {
    // b를 여러 용도로 활용하라.
    return {};
}));
```

이 시점에서, 많은 라이브러리들이 모듈이 아닌 환경이거나, AMD 환경이거나, CommonJS 환경 에서 사용되었다.
동일한 코드를 세가지 버젼을 출시하는 것 대신에, 많은 라이브러리들이 그들의 코드를 래핑하는 조그만 코드를 작성하기로 했고, 그 코드는 자신이 어떠한 환경인지 검출해서 그 환경에 대응하도록 하였다.
모듈이 아닌 환경에서 실행될 경우, 그들은 전역 변수를 제공하였고, 다른 경우는 AMD 또는 CommonJS와 호환되는 모듈을 노출하였다.
이러한 방식은 *UMD* 패턴이라고 알려졌다.

UMD 라이브러리로부터 의존성을 적재하는 것은 약간 심란한 일이어서 이 패턴은 그들 자신은 의존성을 갖지 않는 라이브러리들에서 주로 사용되었다.

몇몇 UMD 라이브러리들은 *항상* 전역 변수를 생성하는데, 반면 다른 것들은 모듈 적재기가 없는 경우에만 이런 일을 한다.

### ES6

```js
// An example ES6 module
import * as NS from "someModule";
import { prop1, prop2 } from "../anotherModule";

export const A = prop1 + prop2;
export function fn() {
  return NS.method();
}
```

TC39 위원회는 모듈에 대해 상황을 조사 했으며 CommonJS 와 AMD의 몇몇 특징들과 새로운 개념을 도입한 표준을 작성 하였다.
ES6 모듈은 임포트와 익스포트를 정적으로 선언하며 의존성은 동기적으로 적재된다.
나중에, 동적 `import`가 추가 되었는데, 이로써 비동기 non-static 의존성이 로딩 가능하게 되었다.

ES6 모듈은 `import` 문장에서 사용된 경로와 디스크상의 파일간의 관계를 정의하지 않았다.
일반적으로, 번들러들은 임포트 경로를 파일 경로로 바꾸는 기존 도구들의 정의를 사용하거나 사용자가 구성하거나, 또는 둘 다 사용하게 될 것이다.

## Modules in TypeScript

타입스크립트에서 모듈 기반의 코드를 작성할 때 고려해야할 것으로 중요한 것은 3가지 이다.

 * **Syntax**: 임포트와 익스포트를 사용할 때 어떤 문법을 사용할 것인가?
 * **Module Resolution**: 모듈 이름(또는 경로)와 디스크 상의 파일과의 관계는 무엇인가?
 * **Module Target**: 생성되는 자바스크립트의 모듈 포멧은 무언인가?

이 것들을 각각 더 자세히 살펴보자.

## Syntax

### ES6

>> **Background Reading**: [import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) and [export](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export) declarations (MDN)

타입스크립트는 `import` and `export` 의 다양한 문법을 완벽히 지원한다.
어떠한 모듈 타겟을 사용하던 개발자는 이러한 형식들을 사용할 수 있다.
ES6 모듈을 타겟으로 한다면, 이것들은 있는 그대로 이식될 것이다 (사용하지 않는 임포트를 제외하고; [[Unused and Type-Only Imports]]를 보라).

ES6문법을 사용하지만 CommonJS or AMD 룰 타겟으로 한다면, 타입스크립트는 Babel과 같은 상호작용 스킴을 따를 것이다.
여기에 ES6 임포트 와 익스포트 그리고 해당하는 CommonJS or AMD 출력물을 예제가 있다.

> **Warning!** 이러한 상호작용 규칙은 현재 노드와 TC39 (자바스크립트 지휘부) 간의 위원회와 워킹그룹이 논의 중이다,
> 또한 다른 위원회 멤버와도 논의 중이다. 이 글이 작성되는 시점에 *아무런* 이러한 상호작용 규칙이 표준화가 되지 않은 상태이고, 만약 ES6 문법을 사용하여 CommonJS를 임포트 한다면 
> 향후 당신의 코드는 동작하지 않을 수 있다.
> 가장 안전하게 할 수 있는 것은 CommonJS 코드를 작성할 때는 CommonJS-style 임포트를 사용 하는 것이다.

#### Namespace Imports

네임스페이스 임포트는 전체 모듈 객체를 임포트 하는 것으로 간주 된다:

```js
// Namespace import
import * as ns from "m";

// Becomes (CommonJS)
const ns = require("m");

// Becomes (AMD)
define(["m"], function (ns) {
})
```

##### Namespace Imports of Functions and Classes

함수를 임포트 하려는 임포트 문법에 ES6 네임스페이스를 사용하려고 하는 것은 흔한 에러이다:

```js
import * as express from "express";
// Error
express();
```

이 코드는 ES6 환경에서 실행되지 않는다.
임포트 하려는 것이 함수이면 디폴트 임포트를 사용하던지 CommonJS 스타일의 임포트를 사용해야 한다.
보다 자세한 것은 다음의 스택오버플로우 게시물을 보라 ["What does “… resolves to a non-module entity and cannot be imported using this construct” mean?](https://stackoverflow.com/questions/39415661/what-does-resolves-to-a-non-module-entity-and-cannot-be-imported-using-this).

#### Destructuring Imports

디스트럭쳐링 임포트는 모듈의 프로퍼티에 바인드 한다:

```js
// Destructured import
import { prop } from "m";
prop.greet();

// Becomes (CommonJS; AMD is similar)
var _m = require("m");
_m.prop.greet();
```

흔한 질문은 왜 `_m`이 나왔냐는 것이며 왜 `prop`이 지역 변수로서 나타나지 않았는 것이다.
그 답변은 ES6 모듈 바인딩이 *live*이기 때문이다: 그들이 읽힐 때 마다, 그들은 임포트된 모듈로 부터 현재의 프로퍼티 값을 가져온다.
예를 들어, 간단한 `counter` 모듈을 작성했다면:

```js
export let counter = 0;
export function increment() {
  counter++;
}
```

그리고 이것을 사용한다:

```js
import { counter, increment } from "./counter";
increment();
increment();
// Should print '2'
console.log(counter);
```

만약 타입스크립트가 `var counter = _m.counter`로 생성했다면, 이 코드는 `2` 대신에 `0`을 프린트 할 것이고 이것은 틀린 값이다.

#### Default Imports

디폴트는 모듈의 `.default` 멤버를 임포트 한다:

```js
import df from "m";
df.greet();

// Becomes (CommonJS; AMD is similar)
var _m = df;
_m.default.greet();
```

#### Synthetic Defaults and `esModuleInterop`

CommonJS 모듈이 `default`로 명명된 멤버를 실제로 익스포트 하는 것은 일반적이지 않다.
여기서의 일반적인 의도는, 예를 들어, `"m"`으로 표현된 전체 모듈을 `df`에 바인딩 하는 것이다.

만약 당신의 모듈 로더가 *자동적으로* CommonJS 모듈상의 `.default` 프로퍼티를 제공한다면, 그것은 모듈 그 자체를 가리키는 것이고, `--allowSyntheticDefaultImports` 컴파일러 옵션을 켜는 것이 가능하다.
만약 이것이 활성화 되면, 타입스크립트는 디폴트 임포트를 마치 모듈 자체를 임포트 하는 것으로 다룰 것이다.

**This does not change the emitted code!**

만약 당신의 모듈 로더가 자동적으로 CommonJS 모듈 상의 프로퍼티를 제공하지 *않는다면*, 디폴트 임포트 문법을 사용하여 이러한 모듈을 임포트하기 원할 경우, `--esModuleInterop` 플래그를 활성화 하면 된다.
이 것은 런타임에 non-ES6 모듈을 검출해 내는 추가적인 헬퍼를 산출해 낼 것이고 CommonJS 모듈이 기본 임포트를 통해서 적재되도록 허용할 것이다.

#### Export Forms

Export declarations follow the same pattern as imports -- when targeting AMD or CommonJS, they create corresponding named properties.
Note that if you're writing a CommonJS module using ES6 syntax, you usually don't want to create a `default` export, as CommonJS consumers won't be expecting to find a property with this name.

### CommonJS-style `import` and `export =`

만약 CommonJS 모듈을 작성 중이라면(즉, Node.js에서 실행될 것을 작성중) 또는 AMD 모듈을 작성 중이라면, ES6 문법 대신에 `require` 문법을 사용하기를 권한다.

#### `import ... = require(...)`

CommonJS-style `import`는 정확히 하나의 형식을 가진다:

```ts
// @noErrors
import fs = require("fs");

// Becomes (CommonJS)
var fs = require("fs");

// Becomes (AMD)
define(["fs"], function (fs) {

});
```



#### Unsupported Syntax

## Unused and Type-Only Imports

타입스크립트는 값들을 임포트 할 때에도 타입과 네임스페이스를 임포트 할때와 동일한 문법을 사용한다.

## Module Syntax in TypeScript

타입스크립트는 모듈 기반의 코드를 작성하도록 허용하며 당신의 선택에 따른 모듈 양식에 맞게 이식될 수 있다.

## Non-modules

어떤 `.ts` 파일이 `import` or `export` 선언을 가지고 있지 않다면, 그 파일은 자동적으로 모듈기반이 아닌 것으로 간주된다.
이 파일의 변수들은 전역 공간에서 선언된 것이며, `--outFile` 컴파일러 옵션을 사용하여 여러 입력 파일을 합치거나 여러 `<script>` 태그를 HTML 파일에 사용하여 이러한 파일을 정확한 순서로 적재하는 것으로 간주된다.

만약 파일이 아무런 임포트나 익스포트 선언이 현재 없더라도 모듈로 간주되기를 원한다면 아래의 문장을 추가하여

```ts
export { };
```

모듈로서 아무것도 익스포트 하지 않는 것으로 만들 수 있다.
이 문법은 모듈 타겟의 종류에 상관없이 잘 동작한다.

### ES6



### AMD



## Import Paths and Module Resolution

## Declaring Modules



"모듈"은 모던 자바스크립트에서 아주 특별한 단어이다.

"모듈들" 이라는 단어는 많은 변이를 포함한다.

ECMAScript 2015이 사작되면서, 자바스크립트는 모듈이라는 개념을 정립하였고
타입스크립트는 이러한 개념을 공유한다.

모듈들은 그들 자신의 범위 내에서 실행되며, 전역 범위에서 실행되는 것이 아니다.
이것의 의미는 변수, 함수, 클래스 등 모듈 내에서 선언된 것이 명확히 익스포트 되지 않으면 모듈 바깥에서 보이지 않는다는 것이다.
반대로, 다른 모듈에서 익스포트된 변수, 함수, 클래스, 인터페이스,등등을 사용하려면, 그것은 먼저 임포트 되어야 한다.

모듈은 선언적이다: 모듈간의 관계는 파일 레벨에서 임포트와 익스포트의 관점에서 규정되어 진다.

모듈들은 모듈 로더에 의해 하나씩 임포트 된다.
런타임에 모듈 로더는 모듈이 실행되기 전에 모든 의존성을 위치시키고 실행시키는 책임을 가진다.
자바스크립트에서 잘 알려진 모듈 로더는 Node.js를 위한 CommonJS가 있고, 웹 어플리케이션을 위한 require.js가 있다.

타입스크립트에서, ECMAScript 2015에서와 마찬가지로, 톱레벨 임포트나 익스포트를 가진 어떤 파일도 모듈로 간주된다.
반대로, 톱 레벨 임포트나 익스포트 선언이 없는 파일은 스크립트로 간주되고 그들의 내용은 전역 공간에서 얻을 수 있다.

### ES Modules

>> [Background Reading: ES Modules: A cartoon deep-dive](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)






## Import Forms










## Importing CommonJS modules with ES Syntax






  * Overview of Choices
    * ES6 (read MDN)
    * CommonJS
    * AMD
    * SystemJS
    * UMD
    * See the appendix because oh my god
  * Import forms
  * Paths and Module resolution
  * Synthetic defaults
  * Import ellision

