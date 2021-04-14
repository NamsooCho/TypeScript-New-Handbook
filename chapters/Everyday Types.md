<!-- Extremely WIP, do not review -->

# Everyday Types

이 챕터에서는 우리가 자바스크립트 코드에서 가장 공통적으로 발견하는 타입들에 대해서 다루며 그런 타입들이 타입스크립트에는 어떻게 해당되는지도 설명한다.
이것은 충분한 리스트가 아니라서 앞으로의 챕터에서는 다른 타입의 이름가 사용법을 추가로 설명할 것이다.

타입들은 단순히 타입명시 말고도 매우 여러 *장소*에서 발견된다.
타입 그 자체에 대해서 배워가면서, 새로운 구조물을 만들면서 우리가 그러한 타입을 참조하는 장소에 대해서 또한 배워나갈 것이다.

우리는 자바스크립트나 타입스크립트를 사용하여 프로그래밍 하면서 가장 흔하게 마주치는 공통적인 타입이면서 기장 기본적인 타입을 살펴보는 것으로 시작한다.
이것들은 나중에 보다 복잡한 타입인 "building blocks"의 핵심을 형성할 것이다.

__toc__

## Primitives `string`, `number`, and `boolean`

자바스크립트는 세개의 가장 기본적인 [primitive](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)인 값의 종류를 갖는다: 그것은 `string`, `number`, 그리고 `boolean` 이다.
각각의 것은 해당하는 타입스크립트 타입이 있다.
당신이 기대하듯이, 이것들은 자바스크립트에서 `typeof` 연산자를 타입의 값들에 적용하면 볼 수 있는 이름과 동일하다.

* `string` 은 `"Hello, world"` 와 같은 문자열을 표현한다
* `number` 는 `42`와 같은 숫자를 위한 것이다. 자바스크립트는 정수를 위한 특별한 런타임을 가지고 있지 않다, 따라서 `int` 또는 `float`에 해당하는 것이 없다 - 모든 것은 단순히 `number` 이다.
* `boolean` 은 `true` 와 `false`의 두 개 값을 갖는다.

> `String`, `Number`, 그리고 `Boolean` 타입 이름은 (대문자로 시작하는) 문법 오류가 아니다, 그러나 이 이름들은 특별한 빌트인 타입을 가리키며 이것들은 당신의 코드에 나타나면 안된다. *항상* `string`, `number`, 또는 `boolean` 로 사용하라.

## Arrays

`[1, 2, 3]`와 같은 배열 타입을 정의하기 위해서, `number[]` 와 같은 문법을 사용하라; 이러한 문법은 어떤 타입에도 적용된다.(에를들어 `string[]`은 문자열의 배열이다.,...).
당신은 아마도 `Array<number>`와 같은 문장도 볼 것인데, 이것은 앞의 것가 같은 의미이다.
우리가 *제너릭*을 다룰 때 `T<U>`문법에 대해서 좀 더 다루어 볼 것이다.

> `[number]`는 다른 것임을 유념하라; *tuple types* 섹션을 참조하라.

## `any`

타입스크립트는 또한 특별한 타입인 `any`를 갖고 있다, 이것은 타입 검사를 행해서 오류를 잡아내는 것을 원치 않을 때 특정 값에 사용할 수 있다.

어떤 값의 타입이 `any`인 경우, 그것의 어떤 프로퍼티에도 접근이 가능하다, 함수인 것처럼 호출도 가능하고, 어떤 타입의 값도 할당 할 수 있고, 문법적으로 합당하다면 그 밖의 어떤 일도 할 수 있다:

```ts
let obj: any = { x: 0 };
// 이 모든 라인들은 에러가 아니다.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```

`any` 타입은 만약 긴 이름을 가진 타입을 단지 특정 코드가 문제가 없다는 것을 확신하도록 하기 위해 쓰지 않기를 원할 때 유용하다.

### `noImplicitAny`

문맥으로 부터 타입을 추정할 수 없고 타입이 명시되지도 않은 경우, 타입스크립트는 디폴트로 `any`이라고 인식한다.
`any` 값은 타입 검사의 이익을 누릴 수 없기 때문에, 이러한 상항을 피하는 것이 요망된다.
`noImplicitAny` 컴파일러 플래그는 이러한 *암묵적*인 `any`를 에러로 표기하도록 해준다.

## Type Annotations on Variables

`const`, `var`, 또는 `let` 을 사용하여 변수를 선언할 때, 그 변수의 타입을 명시적으로 표현하기 위해 타입 명시를 추가할 수 있다:

```ts
let myName: string = "Alice";
          ^^^^^^^^ Type annotation
```

> 타입스크립트는 `int x = 0;` 와 같이 좌측 타입 선언을 사용하지 않는다.
> 타입 명시는 타이핑 되는 값의 *뒤에*에 항상 위치 한다.

대부분의 경우, 타입 명시는 불필요 하다.
가능할 때마다 타입스크립트는 자동으로 당신의 코드상의 타입을 *추정* 한다.
예를 들어, 변수의 타입은 그 변수의 초기화 값의 타입으로 *추정* 된다.

```ts
// No type annotation needed -- 'myName' inferred as type 'string'
let myName = "Alice";
```

대부분의 상항에서 당신은 추정 규칙을 명확히 공부할 필요가 없다.
만약 시작한다면, 생각하는 것 보다 더 적은 수의 타입 명시를 사용하려고 노력해보라 - 아마도 당신은 타입스크립트가 무엇이 어떻게 돌아가고 있는지 이해하도록 하기위한 타입명시가 얼마나 적게 필요한지 놀랄 것이다.

## Functions

자바스크립트에서 함수는 데이터를 전달 하는 주요 수단 이다.
타입스크립트는 함수의 입력과 출력 값의 타입을 정의하도록 허용한다.

### Parameter Type Annotations

함수를 선언할 때, 함수가 받아들이는 파라메터의 종류를 제한 하도록 각 파라메터의 타입 명시를 추가할 수 있다.
파라메터 타입 명시는 파라메터 이름 뒤에 온다:

```ts
// Parameter type annotation
function greet(name: string) {
                   ^^^^^^^^
    console.log("Hello, " + name.toUpperCase() + "!!");
}
```

파라메터가 타입 명시를 가지면, 그 함수를 호출할 때 타입 체크가 이루어진다:

```ts
declare function greet(name: string): void;
//cut
// Would be a runtime error if executed!
greet(42);
```

### Return Type Annotations

함수의 반한 값의 타입도 명시될 수 있다.
반환 값 타입 명시는 파라메터 리스트 뒤에 온다:

```ts
function getFavoriteNumber(): number {
                            ^^^^^^^^
    return 26;
}
```

변수의 타입 명시아 매우 비슷하게, 반환 값 타입 명시는 보통 불필요 한데 그 이유는 타입스크립트는 `return` 문장의 타입으로 부터 그 함수의 반환 값 타입을 추정하기 때문이다.
위의 타입 명시 예제는 실제로 아무것도 변경시키지 않는다.
어떤 코드베이스는 문서화 목적으로 반환값 타입 명시를 한다, 우연한 변경을 방지하고 또는 단지 개인적 참조용으로 말이다.

### Function Expressions

함수 식은 함수 선언과 약간 다르다.
함수 식이 타입스크립트가 어떻게 호출되는지 결정할수 있는 장소에서는 함수의 파라메터는 자동적으로 주어진 타입으로 결정된다.

여기 예제가 있다:

```ts
// 여기에는 타입 명시가 없지만, 타입 스크립트는 버그를 식별해 낼 수 있다.
const names = ["Alice", "Bob", "Eve"];
names.forEach(function (s) {
    console.log(s.toUppercase());
});
```

`s`에 아무런 타입 명시가 없지만, 타입스크립트는 `forEach` 함수에 배열로부터의 타입 추정으로 `s`가 가질 타입을 결정하였으며 그 타입으로 검사를 행한다.

이러한 것을 *문맥 타이핑* 이라 부르는데 그 이유는 *문맥*이 그 함수가 가져야 하는 타입을 말해주기 때문이다.
타입 추정 규칙과 비슷하게, 당신은 이런 일이 어떻게 일어나는지 명확히 배울 필요가 없지만, 타입 명시가 필요하지 않을 때에 이러한 일이 당신을 돕기 위해 벌어진다는 것을 이해하는 것은 필요하다.
나중에, 우리는 문맥이 값의 타입에 어떤 영향을 주는지 추가적인 예제를 살펴볼 것이다.

## Object Types

프리미티브아 별개로, 가장 흔하게 당신이 마주치는 타입은 *객체 타입* 이다.
이것은 프로퍼티를 가진 모든 자바스크립트 값을 말하는데, 이것은 거의 모든 값들을 포함한다!
객체 타입을 정의하려면, 단순히 프로퍼티와 그 타입을 리스트로 나타내면 된다.

예를 들어, 여기에 좌표 비슷한 객체를 받아들이는 함수가 있다:

```ts
// 파라메터의 타입은 객체 타입이라고 명시되었다.
function printCoord(pt: { x: number, y: number }) {
                        ^^^^^^^^^^^^^^^^^^^^^^^^
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });
```

여기서, 둘 다 `number` 타입인 `x` 와 `y`를 프로퍼티로 가지는 객체 타입을 명시하였다.
`,` 또는 `;` 는 프로퍼티를 구분하는데 사용되며, 마지막 프로퍼티에 적용하는 것은 선택적이다.

각 프로퍼티의 타입 부분도 선택적인 부분이다.
만약 타입이 명시되지 않으면, 그 타입은 `any`로 가정된다.

### Optional Properties

객체 타입은 또한 자신의 프로퍼티의 일부 또는 전부를 *선택적*으로 정의할 수 있다.
이를 위해서는 프로퍼티 이름 뒤에 `?`를 추가하면 된다.

```ts
function printName(obj: { first: string, last?: string}) {
  // ...
}
// Both OK
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });
```

자바스크립트에서, 존재하지 않는 프로퍼티에 접근하려 하면, 런타임 에러 대신에 `undefined` 값을 얻게 될 것이다.
이 것 때문에, 선택적 프로퍼티를 *읽기* 할때, 반드시 그전에 `undefined` 검사를 해야 한다.

```ts
function printName(obj: { first: string, last?: string}) {
  // Error - 'obj.last' 가 제공되지 않은 경우 프로그램이 다운 될 수 있다.
  console.log(obj.last.toUpperCase());
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }
}
```

## Union Types

타입스크립트는 넒은 범위의 연산자를 사용하여 기존에 있던 타입들을 사용하여 새로운 타입을 만들수 있도록 해준다.
이제 우리는 몇가지 타입을 어떻게 쓰는지 알고 있으므로, 흥미로운 방법으로 이러한 타입들을 *조합*하는 것에 대해 다룰 때가 되었다.

### Defining a Union Type

타입들을 조합하는 첫번째 방법으로 *유니온* 타입을 보자.
두개 이상의 타입으로 구성된 것이 유니온 타입인데, 이러한 타입 중 *어떤 하나*인 값을 표현하는 타입이다.
우리는 이러한 개개의 타입을 유니온의 *멤버*라 부른다.

스트링 또는 숫자에 동작하는 함수를 하나 작성해 보자.

```ts
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId([1, 2]);
```

### Working with Union Types

유니온 타입에 매치되는 값을 *제공*하는 것은 쉽다 - 단순히 유니온 멤버의 하나와 일치하는 타입의 값을 제공하면 된다.
만약 유니온 타입의 값을 *소유*한 경우 그것을 가지고 어떻게 일할 것인가?

타입스크립트는 오직 유니온의 *모든* 멤버에게 적절한 일을 하는 것만 허용할 것이다.
예를 들어, 만약 `string | number` 유니온을 가지고 있다면, 오직 `string`에만 허용되는 메소드는 사용할 수 없을 것이다.

```ts
function printId(id: number | string) {
  console.log(id.toUpperCase());
}
```

이와 같은 문제의 해결책은 유니온을 코드를 사용하여 *좁히는* 것이다, 이것은 자바스크립트에서 타입 명시 없이 했던 방식과 동일하다.
*내로윙* 은 코드의 구조에 기반하여 타입스크립트가 어떤 값을 더 특정적인 타입으로 좁히는 과정에 일어난다.

예를 들어, 타입스크립트는 `"string"`에 대한 `typeof` 는 오직 `string`임을 알고 있다.

```ts
function printId(id: number | string) {
  if (typeof id === "string") {
    // In this branch, id is of type 'string'
    console.log(id.toUpperCase());
  } else {
    // Here, id is of type 'number'
    console.log(id);
  }
}
```

다른 예제는 `Array.isArray` 와 같은 함수를 사용하는 것이다:

```ts
function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // Here: 'x' is 'string[]'
    console.log("Hello, " + x.join(" and "));
  } else {
    // Here: 'x' is 'string'
    console.log("Welcome lone traveler " + x);
  }
}
```

`else` 분기에서 특별한 무엇을 할 필요가 없음을 유의하라 - 만약 `x`가 `string[]`이 아니라면 그것은 반드시 `string`이기 때문이다.

때때로 모든 멤버가 공통의 무엇을 가지는 유니온을 사용할 것이다.
예를 들어, 배열과 스트링은 공통적으로 `slice` 메소드를 가진다.
유니온의 모든 멤버가 공통의 프로퍼티를 가지면 내로윙 없이 그 프로퍼티를 사용할 수 있다:

```ts
// Return type is inferred as number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}
```

> *유니온* 이라는 이름이 사실은 이러한 타입의 프로퍼티들의 *교집합*을 가지는 것으로 보이는 것은 혼란스러울 수 있다.
> 이것은 문제가 아니다 - 이론상으로도 *유니온*이 맞다.
> `number | string` *유니온*은 각 타입의 *값들의 합집합* 으로 구성된다.
> 두개의 집합이 각각의 집합에 해당하는 사실들로 구성되어 있으면, 오직 그들의 *교집합*인 사실들만 *유니온*인 행위로서 적용될 수 있음을 유의하라.
> 예를 들어, 우리가 모자를 쓴 키큰 사람들만 있는 방을 가지고 있다고, 다른 방은 모자를 쓴 스페인어를 쓰는 사람들의 방이 있다면, 이 두 방을 유니온 하면, *모든* 사람이 모자를 썼다는 사실만을 알게 된다.

## Type Aliases

우리는 객체 타입과 유니온 타입을 사용하여 타입 명시에 직접적으로 사용하였다.
이것은 편리하다, 그러나 같은 타입을 여러번 사용하고 이러한 타입을 하나의 이름으로 명명하고 싶은 것은 흔한 일이다.

*타입 별칭*은 정확히 그것이다 - 어떤 *타입*을 위한 어떤 *이름*이다.
타입 별칭의 문법은:

```ts
type Point = {
  x: number,
  y: number
};

// 이전 예제와 완전히 동일하다.
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

실제로 어떤 타입에도 타입 별칭으로 다른 이름을 줄 수 있으며 객체 타입에만 한정되는 것은 아니다.
예를 들어, 유니온 타입에도 타입 별칭을 줄 수 있다:

```ts
type ID = number | string;
```

별칭은 *단지* 별칭일 뿐이라는 것을 유의하라 - 동일한 타입을 별칭을 사용하여 다른/구분되는 버젼의 타입을 만들 수는 없다.
별칭을 사용하면, 그것은 정확히 원래의 타입을 사용하는 것과 같다.
다른 말로 설명하면, 코드는 *부적절*하게 보일지 몰라도, 타입스크립트는 두개의 별칭이 같은 타입이므로 OK로 판단한다:

```ts
type Age = number;
type Weight = number;

const myAge: Age = 73;
// *not* an error
const myWeight: Weight = myAge;
```

## Interfaces {#interfaces}

*인터페이스 선언*은 객체 타입을 명명하는 또 다른 방법이다:

```ts
interface Point {
  x: number;
  y: number;
}

function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

타입 별칭에서 우리가 했듯이, 이 예제는 익명 객체 타입을 사용했을 때 처럼 동작한다.
타입스크립트는 우리가 전달한 `printCoord`의 *구조*에만 관심을 갖는다 - 그것은 오직 기대되는 프로퍼티들이 있는가만 관심을 갖는다.
타입의 구조와 능력에만 관심을 갖는다는 것은 우리가 타입스크립트를 *구조적 타입* 타입 시슽템을 가진다고 부르는 이유이다.

### Differences Between Type Aliases and Interfaces {#interface-vs-alias}

타입 별칭과 인터페이스는 매우 유사하다, 그리고 많은 경우 둘 사이에서 자유롭게 선택해도 무방하다.
당신이 알아야 할 둘 사이의 가장 적절한 차이가 여기 있다.
나중에 당신은 이러한 개념에 대해 더 설명을 들을 것이므로 지금 당장 이것을 이해하지 못하더라도 걱정할 필요는 없다.

* 인터페이스는 `extend`(확장)될 수 있다. 우리는 이것을 나중에 토의 할 것이지만 이 말의 의미는 인터페이스가 다른 타입들로 부터 새로운 타입을 생성할때 더 많은 보장을 제공한다는 의미이다.
* 타입 별칭은 선언 병합에 참여할 수 없지만 인터페이스는 가능하다.
* 인터페이스는 오직 객체 선언에서만 사용이 가능하다.
* 인터페이스 이름은 에러 메시지에서 그들의 원래 형태로 *항상* 나타나지만 *오직* 그 이름으로 사용되었을 때만 이다.
* 타입 별칭은 *아마도* 에러 메시지에 나타날 수 있다, 때때로 동일한 익명 타입을 대신해서 나타난다(원하는 것이든 원하지 않는 것이든).

대부분의 경우, 개인적 취향에 따라 선택하여 사용하면 되며 타입스크립트는 다른 선언이 필요하면 친절히 알려줄 것이다.

## Type Assertions

때때로 타입스크립트가 알아낼 수 없는 어떤 값의 타입에 관한 정보를 개발자는 갖고 있는 경우가 있다.

예를 들어, `document.getElementById`를 쓰고 있다면, 타입스크립트는 단지 *어떤* 종류의 `HTMLElement`를 갖는다는 것 만을 알수 있으나 당신은 웹 페이지가 항상 주어진 id 에는 오직 `HTMLCanvasElement` 만이 있다는 것을 알고 있는 경우가 있다.

이런 상황에서는, 타입의 더욱 특정적인 타입으로 규정하는 *타입 확신"을 사용할 수 있다.

```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

타입 명시와 비슷하게, 타입 확정은 컴파일러에 의해 제거되며 코드의 행동에 어떤한 영향도 끼치지 않는다.

또한 앵글 브라켓 문법은 이와 동일한(`.tsx` 파일 안에서는 제외하고) 의미이다.

```ts
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

> 기억할 점: 이들은 컴파일 타임에 제거되므로, 타입 확신은 실행시 어떠한 체크도 일어나지 않는다.
> 타입 확신이 틀렸다 하더라도 어떠한 예외나 `null`이 생성되지는 않는다.

타입스크립트는 타인 확신을 *더 세밀한* 또는 *덜 세밀한* 타입의 버젼으로 변경하게 해준다.
이러한 규칙은 불가능한 강제 형변환을 방지한다:

```ts
const x = "hello" as number;
```

때때로 이러한 규칙은 너무 보수적이어서 더 복잡하지만 적절한 형변환 조차 허락하지 않는다.
만약 이러한 일이 일어나면, 두개의 타입 확신을 사용하여 원하는 타입으로 지정이 가능하다 - 먼저 `any` (또는 `unknown`으로, 나중에 설명할 것이다)로 바꾸고 그다음 원하는 타입으로 변경.

```ts
declare const expr: any;
type T = any;
//cut
const a = expr as any as T;
```

## Literal Types

`string` 과 `number` 같은 범용 타입 외에, 타입 명시 위치에 *특정적인* 스트링과 숫자를 지정할 수 있다.

그 자체로는, 리터럴 타입들은 매유 유용하지는 않다:

```ts
let x: "hello" = "hello";
// OK
x = "hello";
// OK

x = "hello";
// ...
x = "howdy";
```

변수가 오직 하나의 값만 가지는 것은 많이 사용되지는 않는다!

그러나 유니온으로 *합성* 되어서는, 더욱 유용한 것을 표현 할 수 있다 - 예를 들어, 알고있는 값들의 집합 만을 받아들이는 함수를 만들 수 있다:

```ts
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
```

숫자 리터럴도 같은 방식으로 동작한다:

```ts
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

물론, 이러한 타입을 리터럴이 아닌 타입과 합성하는 것도 가능하다:

```ts
interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
configure({ width: 100 });
configure("auto");
configure("automatic");
```

리터럴 타입이 하나 더 존재한다: 불리언 리터럴이다.
불리언 리터럴 타입은 오직 두 개만 존재한다, 여러분이 추측하듯이, `true` 와 `false` 두 개의 타입 뿐이다.
`boolean` 타입 그 자체는 사실 `true | false` 유니온 타입의 별칭일 뿐이다.

### Literal Inference

객체를 가지고 변수를 초기화 한 경우, 타입스크립트는 나중에 그 객체의 프로퍼티가 변경될 것이라고 가정한다.
예를 들어, 이와 같은 코드를 작성했다면:

```ts
declare const someCondition: boolean;
//cut
const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
```

타입스크립트는 `1` 을 이전에 `0` 이었던 필드에 대입하는 연산이 에러가 되리라고 가정하지 않는다.
다른 말로 하면 `obj.counter` 의 타입은 `0`이 아니라 `number`가 된다, 왜냐하면 타입은 *읽기* 와 *쓰기* 행동을 결정하는데 사용되기 때문이다.

스트링에게도 동일한 규칙이 적용된다:

```ts
declare function handleRequest(url: string, method: "GET" | "POST"): void;
//cut
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
```

<!-- TODO: Use and explain const contexts -->

`req.method`애 `"GUESS"`와 같은 문자열을 대입하는 것이 문법적으로 틀리지 않기 때문에, 타입스크립트는 이 코드는 에러를 가지고 있다고 판단한다.
이러한 타입 추정을 피하려면 양쪽에 타입 확신을 추가해주면 된다.

```ts
declare function handleRequest(url: string, method: "GET" | "POST"): void;
//cut
const req = { url: "https://example.com", method: "GET" as "GET" };
/* or */
handleRequest(req.url, req.method as "GET");
```

처음 변경은 "나는 `req.method`가 *리터럴 타입*인 `"GET"`을 항상 갖도록 하겠다"를 의미하고, 이것은 `"GUESS"`와 같은 대입 연산을 막아준다.
두번째 변경은 "나는 `req.method`가 `"GET"` 값을 갖는 다른 이유를 안다" 라는 의미 이다.

## `null` and `undefined`

자바스크립트는 `null` 과 `undefined` 의 두개의 프리미티브 값을 가진다, 두개 다 부재와 초기화 되지 않았음을 나타내는 데 사용 된다.

타입스크립트 같은 이름으로 해당하는 *타입*을 갖는다. 이러한 타입들이 어떻게 행동하느냐는 `strictNullChecks` 옵션에 따라 달라진다.

### `strictNullChecks` off

`strictNullChecks`가 *off* 이면, `null` 또는 `undefined`가 되는 값들도 여전히 평범하게 접근되고, 어떤 타입에도 `null` 또는 `undefined` 값이 대입 될 수 있다.
이것은 널 체크가 없는 언어(예: C#, Java)의 행동과 비슷하다.
이러한 값들의 검사가 없는 것은 버그의 주요 원인이 되는 경향이 있다; `strictNullChecks`을 항상 켜서 사용하도록 사람들에게 권한다 - 만약 이것이 그들의 코드베이스에 적용 가능 하다면.

### `strictNullChecks` on

`strictNullChecks` 가 *on* 이면, 어떤 값이 `null` 또는 `undefined` 이면, 그 값에 대한 프로퍼티나 메소드를 사용하기 전에 반드시 검사해야할 필요가 있다.
옵셔널 프로퍼티에서 `undefined`를 검사했던 것 처럼, *narrowing* 을 사용하여 `null`이 될 수도 있는 값에 대해서 검사를 수행 할 수 있다.

```ts
function doSomething(x: string | null) {
  if (x === null) {
    // do nothing
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
```

### Non-null Assertion Operator (Postfix `!`) {#non-null-operator}

타입스크립트는 아무런 명시적 검사가 없이 `null` 과 `undefined` 을 타입으로부터 제거할 수 있는 특별한 문법이 있다.
연산식 다음에 `!`을 붙입으로써 값이 `null` 또는 `undefined`이 아니라는 확신을 효과적으로 할수 있다.

```ts
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```

다른 타입 확신과 마찬가지로, 이것은 코드의 런타임 행동을 변경하지는 않는다, 따라서 만약 값이 *절대로* `null` 또는 `undefined` 되지 않는 다고 확신 할 때에만 사용 가능 하다는 것은 중요하다.
