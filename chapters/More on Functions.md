# More on Functions

함수는 어느 애플리케이션이든 기본적 구성단위 이다, 로컬 함수이든, 임포트된 모듈로 부터 온 것이든, 클레스의 메소드 이든.
함수는 또한 값이며, 다른 값들과 마찬가지로 타입스크립트는 함수가 어떻게 호출되는지를 서술하는 많은 방법을 가지고 있다.
타입을 어떻게 작성하며 함수를 어떻게 기술하는지 살펴보자.

__toc__

## Function Type Expressions

함수를 기술하는 가장 간단한 방법은 *함수 타입 식*을 사용하는 것이다.
이러한 타입은 애로우 함수오 문법적으로 유사하다:

```ts
function greeter(fn: (a: string) => void) {
    fn("Hello, World");
}
function printToConsole(s: string) {
    console.log(s);
}
greeter(printToConsole);
```

`(a: string) => void` 문법은 "하나의 파라메터 `a` 를 갖고 그 타입은 스트링이며 반환값이 없는 함수" 를 의미한다.
함수의 선언처럼, 파라메터 타입이 정의되지 않으면 `any`로 암묵적으로 처리된다.

> 파라메터의 이름은 **required** 된다는 것을 유념하라, `(string) => void` 이라는 함수 타입은 "`string` 이라는 이름을 가진 `any` 타입의 파라메터를 가진 함수" 라는 뜻이 된다.

물론, 함수 타입에 타입 별칭을 사용할 수 있다:

```ts
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
    // ...
}
```

## Call Signatures

자바스크립트에서, 호출가능 하다는 특성 외에 함수는 프로퍼티를 가질 수 있다.
그러나, 함수 타입 문법은 프로퍼티를 선언하는 것을 허용하지 않는다.
만약 호출가능한 프러퍼티를 가진 무엇을 선언하고 싶다면, 객체 타입에 *call signature*를 작성할 수 있다.

```ts
type DescribableFunction = {
    description: string;
    (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
    console.log(fn.description + " returned " + fn(6));
}
```

이 문법은 함수 타입 식과 약간 다른 것을 유념하라 - 파라메터와 반환 타입 사이에 `=>` 대신에 `:`을 사용하였다.

## Construct Signatures

자바스크립트 함수는 `new` 연산자로도 활성화 될 수 있다.
타입스립트는 이것을 *constructors*로 사용하는데 왜냐하면 보통 새 객체를 생성하는데 사용되기 때문이다.
call signature 앞에 `new` 키워드를 추가함으로써 *construct signature*를 작성할 수 있다.

```ts
type SomeObject = any;
//cut
type SomeConstructor = {
    new(s: string): SomeObject;
}
function fn(ctor: SomeConstructor) {
    return new ctor("hello");
}
```

자바 스크립트의 `Date` 객체와 같은 것은, `new`와 함께, 또는 없이 호출될 수 있다.
함수의 호출과 construct signatures룰 합쳐서 동일 타입에 임의로 적용할 수 있다:

```ts
interface CallOrConstruct {
    new(s: string): Date;
    (n?: number): number;
}
```

## Generic Functions

함수에서 인풋 타입이 아웃풋 타입과 관련되어 지는 것은 흔한 일이며 또는 두개의 인풋 타입이 서로 연관되는 것도 흔한 일이다.

```ts
function firstElement(arr: any[]) {
  return arr[0];
}
```

이 함수는 자신의 일을 잘 수행한다, 그러나 불행히도 반환 타입은 `any`이다.
배열 요소의 타입을 반환하는 함수가 더 좋다.

타입스크립트에서, *generics*는 두 값 사이의 관계성을 기술하는데 사용된다.
함수 시그니쳐에서 *type parameter*를 선언함으로써 우리는 이런 일을 할 수 있다.

```ts
function firstElement<T>(arr: T[]): T {
  return arr[0];
}
```

타입 파라메터터 `T`를 이 함수에 추가하고, 이 것을 두 군데에서 사용함으로써, 함수의 인풋 타입과 아웃풋 타입간의 관계를 생성하였다.
이제 이것이 호출 될 때, 더욱 구체적인 타입이 나온다:

```ts
declare function firstElement<T>(arr: T[]): T;
//cut
// s is of type 'string'
const s = firstElement(["a", "b", "c"]);
// n is of type 'number'
const n = firstElement([1, 2, 3]);
```

### Inference

이 예제에서 우리는 `T`를 구체적으로 명시하지 않았음을 유의하라.
타입은 타입스크립트에 의해 *추정*되었고 - 자동으로 선택된다.

여러개의 타입 파라메터를 사용하는 것도 가능하다.
예를 들어, `map`의 독립적 함수 버젼은 이와 같을 것이다:

```ts
function map<E, O>(arr: E[], func: (arg: E) => O): O[] {
  return arr.map(func);
}

// Parameter 'n' is of type 'number'
// 'parsed' is of type 'number[]'
const parsed = map(["1", "2", "3"], n => parseInt(n));
```

이 예제에서 타입스크립트는 함수식의 반환 값에 근거해서 `O`의 타입을 추정하고 `E`의 타입을 주어진 스트링 베열로 부터 추정하고 있는 것을 유의하라.

### Constraints

지금까지 *어떤* 종류의 값에도 작동하는 제너릭 함수를 작성해 보았다.
때때로 두 값들을 연관시키고 그러나 특정 값들의 부분집합에 대해서만 동작하도록 하기 원할 때가 있다.
이런 경우, 파라메터가 받아들이는 타입의 종류에 *제한*을 걸면 된다.

두개의 값들 중 더 긴 것을 반환하는 함수를 작성해 보자.
이를 위해 우리는 숫자인 `length` 프로퍼티가 필요하다.
우리는 `extends` 구절을 작성함으로써 타입에 이러한 *제한*을 건다.

```ts
function longest<T extends { length: number }>(a: T, b: T) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}

// longerArray is of type 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString is of type 'string'
const longerString = longest("alice", "bob");
// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
```

이 예제에서 몇가지 흥미로운 것이 있다.
타입스크립트는 `longest`의 반환 타입을 추정한다.
반환 타입 추정은 제너릭 함수에서도 또한 작동한다.

`T`에 `{ length: number }`로 제한을 두었기 때문에, `a` 와`b` 파라메터의 `.length` 프로퍼티에 접근이 가능하였다.
이러한 타입 제약이 없이는, length 프로퍼티가 없는 타입이 넘어올 수 있고, 그렇다면 이 프로퍼티에 접근이 불가능하므로 위 함수는 동작할 수 없을 것이다.

`longerArray` 와 `longerString`의 타입은 아규먼트의 타입에 기반해 추정되었다.
제너릭은 두개 이상의 값들을 같은 타입으로 관계시키는 것이라는 것을 기억하라!

마지막으로, 위에서 우리가 했듯이, `longest(10, 100)`에 대한 호출은 `number` 타입이 `.length` 프로퍼티가 없으므로 거절되었다.

### Working with Constrained Values

여기에 제너릭 제한을 하면서 가장 흔한 에러가 있다.

```ts
function minimumLength<T extends { length: number }>(obj: T, minimum: number): T {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum };
  }
}
```

이 함수는 OK안 것처럼 보인다 - `T`는 `{ length: number }`로 제한되었고, 함수는 `T`를 반환하거나 제한을 충족하는 값을 반환한다.
문제는 이 함수가 인풋 타입과 *동일*한 타입을 반환 한다고 약속한 사실이다, 단지 제약을 충족하는 *어떤* 객체를 반환하는 것이 아니다.
만약 이 코드가 정당하다고 허용되면, 우리는 확실히 동작하지 않는 코드를 작성하게 된다.

```ts
declare function minimumLength<T extends { length: number }>(obj: T, minimum: number): T;
//cut
// 'arr' gets value { length: 6 }
const arr = minimumLength([1, 2, 3], 6);
// and crashes here because arrays have
// a 'slice' method, but not the returned object!
console.log(arr.slice(0));
```

### Specifying Type Arguments

타입스크립트는 제너릭 호출에서 의도된 타입을 추정하는데, 항상 그런 것은 아니다.
예를 들어, 두개의 배열을 합치는 함수를 작성한다고 해보자:

```ts
function combine<T>(arr1: T[], arr2: T[]): T[] {
  return arr1.concat(arr2);
}
```

보통 불일치하는 배열로 이 함수를 호출하면 에러가 된다:
Normally it would be an error to call this function with mismatched arrays:

```ts
declare function combine<T>(arr1: T[], arr2: T[]): T[];
//cut
const arr = combine([1, 2, 3], ["hello"]);
```

만약 위 코드가 의도한 것이라면 `T`를 명확히 해야 한다:

```ts
declare function combine<T>(arr1: T[], arr2: T[]): T[];
//cut
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```

### Guidelines for Writing Good Generic Functions

제너릭 함수를 작성하는 것은 재미 있으며, 타입 파라메터를 가져 감으로써 쉽기도 하다.
너무 많은 타입 파라메터를 가지거나 제약을 너무 많이 필요 이상으로 사용하면 타입 추정을 덜 성공적이게 만들고, 함수 호출자를 당황하게 만든다.

#### Push Type Parameters Down

여기에 비슷하게 보이는 함수를 작성하는 두 가지 방법이 있다:

```ts
function firstElement1<T>(arr: T[]) {
  return arr[0];
}

function firstElement2<T extends any[]>(arr: T) {
  return arr[0];
}

// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```

처음 보기에는 두 가지가 동일해 보인다, 그러나 `firstElement1`이 더 나은 방법이다.
반환 타입 `T`를 정확히 추정하는데 반해, `firstElement2`의 추정 반환 타입은 `any` 가 되는데 그 이유는 타입스크립트는 반환 타입을 추정하는데 있어서 제약 타입을 사용하지, 호출될 때를 기다려서 배열의 요소를 가려내지 않기 때문이다.

> **규칙**: 가능하다면, 제약을 걸기보다는 타입 파라메터 자체를 사용하라

#### Use Fewer Type Parameters

여기에 비슷한 두개의 함수가 또 있다:

```ts
function filter1<T>(arr: T[], func: (arg: T) => boolean): T[] {
    return arr.filter(func);
}

function filter2<T, F extends (arg: T) => boolean>(arr: T[], func: F): T[] {
    return arr.filter(func);
}
```

*두 값들의 관계를 만들지 않는* 타입 파라메터 `F`가 만들어졌다.
이러한 것은 항상 좋지않다, 왜냐하면 아무 이유 없이 이 함수의 호출자는 추가적인 타입 아규먼트를 정의해야 하기 때문이다.
`F`는 함수를 더 이해하고 읽기 어렵게 만들 뿐 아무런 일을 하지 않는다!

> **규칙**: 가능한 적은 수의 타입 파라메터를 사용하라

#### Type Parameters Should Appear Twice

때때로 우리는 함수가 제너릭이 될 필요가 없다는 것을 잊는다:

```ts
function greet<S extends string>(s: S) {
  console.log("Hello, " + s);
}

greet("world");
```

이것은 간단한 버젼으로 더 쉽게 작성이 가능하다:

```ts
function greet(s: string) {
  console.log("Hello, " + s);
}
```

타입 파라메터는 *여러 값들의 타입들을 연관짓는 것* 이라는 것을 기억하라.
타입 파라메터가 함수 시그니쳐에서 오직 한곳에서만 사용된다면, 그것은 아무것도 연관짓지 않는 것이다.

> **규칙**: 타입 파라메터가 오직 한군데 에서만 사용된다면, 그것이 실제로 필요한지 강하게 재검토 하라

## Optional Parameters

자바스크립트에서 함수는 자주 변하는 갯수의 아규먼트를 받는다.
예를 들어, `number` 의 `toFixed` 메소드는 선택적인 자리수를 받는다:

```ts
function f(n: number) {
  console.log(n.toFixed()); // 0 arguments
  console.log(n.toFixed(3)); // 1 argument
}
```

타입스크립트에서는 이 모델을 `?`을 사용하여 *선택적*인 파라메터로 표시한다:

```ts
function f(x?: number) {
  // ...
}
f(); // OK
f(10); // OK
```

파라메터의 타입이 `number`로 표시되어 있지만 `x` 파라메터는 사실상 `number | undefined` 의 타입이다, 왜냐하면 정의되지 않은 파라메터가 자바스크립트에서는 `undefined` 값을 갖기 때문이다.

또한 파라메터에 *default* 값을 줄 수도 있다:

```ts
function f(x = 10) {
  // ...
}
```

이제 함수 `f`의 몸체에서 `x`는 `number` 타입을 가질 것이다 왜냐하면 `undefined` 아규먼트는 `10`으로 대치될 것이기 때문이다.

파라메터가 선택적이라면, 호출자는 항상 `undefined`를 넘겨줄 수 있는데 이는 단순히 "생략된" 아규먼트로 처리되기 때문이다:

```ts
declare function f(x?: number): void;
// cut
// All OK
f();
f(10);
f(undefined);
```

### Optional Parameters in Callbacks

이제 함수 타입 식과 선택적 파라메터에 대해 배웠으니, 콜백을 호출하는 함수를 작성하면서 다음과 같은 실수를 하는 것은 매우 흔한 일이 될 것이다:

```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```

`index?`를 선택적인 파라메터로 한 보동의 *의도*는 다음의 호출도 가능하게 하려는 것일 것이다:

```ts
declare function myForEach(arr: any[], callback: (arg: any, index?: number) => void): void
//cut
myForEach([1, 2, 3], a => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```

이것이 *실제로* 의미하는 것은 *`callback` 이 하나의 아규먼트로도 호출될 수 있다* 라는 것이다.
다른 말로 하면, 함수 정의는 구현이 다음과 같은 것일 거라고 이야기 하고 있다:

```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // I don't feel like providing the index today
    callback(arr[i]);
  }
}
```

결국, 타입스크립트는 다음과 같은 의미라고 강제할 것이고 실제로 가능하지 않다고 에러를 출력할 것이다:

```ts
declare function myForEach(arr: any[], callback: (arg: any, index?: number) => void): void
//cut
myForEach([1, 2, 3], (a, i) => {
  console.log(i.toFixed());
});
```

자바스크립트에서, 파라메터의 수 보다 많은 아규먼트를 가지고 호출이 가능하다, 추가적인 아규먼트는 단순히 무시된다.
타입스크립트도 같은 행동을 한다.
더 적은 파라메터를(같은 타입의) 가진 함수는 더 많은 파라메터를 가진 함수를 항상 대치할 수 있다.

> 콜백을 위해 함수 타입을 작성하는 경우, *절대로* 선택적 파라메터를 사용하지 말라 - 아규먼트를 전달하지 않고 함수를 *호출*하려는 의도가 아닌한.

## Function Overloads

몇몇의 자바스크립트 함수는 다양한 아규먼트 숫자와 타입으로 호출될 수 있다.
예를 들어, 타임스탬프(하나의 아규먼트) 또는 월/일/년도 정의(3개 아규먼트)를 받아서 `Date`를 만드는 함수는 작성할 수 있다.

타입스크립트에서는, *overload signatures* 를 사용하여 서로 다른 방식으로 호출되는 함수를 작성할 수 있다.
이를 위해서, 몇개의(두개 이상의) 함수 시그니쳐를 작성하고 함수의 몸체를 작성한다:

```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
```

이 예제에서, 두개의 오버로딩이 작성되었다: 하나는 한개의 아규먼트를 받는 것이고, 다른 하나는 세개의 아규먼트를 받는 것이다.
앞의 두개의 시그니쳐를 *overload signatures* 라고 부른다.

그리고 나서, 우리는 호환되는 시그니쳐를 가지고 함수 몸체를 구현했다.
함수는 하나의 *구현* 시그니쳐를 가지며, 이 시그니쳐는 직접 호출이 불가능하다.
하나의 필수 파라메터와 두개의 선택적 파라메터를 가진 함수를 작성 했다 하여도, 그것은 두개의 파라메터로 호출될 수 없다.

### Overload Signatures and the Implementation Signature

이것은 평범한 혼돈의 근원이다.
자주 사람들은 다음과 같은 코드를 작성하고 왜 에러가 나는지 이해하지 못한다:

```ts
function fn(x: string): void;
function fn() {
  // ...
}
// Expected to be able to call with zero arguments
fn();
```

다시 말하지만, 함수 몸체를 작성하는데 사용된 시그니쳐는 외부에서 보이지 않는다.

> *구현* 에 사용된 시그니쳐는 외부에서 보이지 않는다.
> 오버로딩된 함수를 작성할 때에는, 항상 *두개* 이상의 시그니쳐를 구현 함수위에 두어야 한다.

구현 시그니쳐는 반드시 오버로딩 시그니쳐와 *호환*되어야 한다.
예를 들어, 이러한 함수는 구현 시그니쳐가 오버로드 시그니쳐와 호황되지 않으므로 에러이다:

```ts
function fn(x: boolean): void;
// Argument type isn't right
function fn(x: string): void;
function fn(x: boolean) {

}
```

```ts
function fn(x: string): string;
// Return type isn't right
function fn(x: number): boolean;
function fn(x: string | number) {
  return "oops";
}
```

### Writing Good Overloads

제너릭 처럼, 함수 오버로딩에는 몇 가지 가이드라인이 있다.
이러한 원칙을 따르면 함수를 더 쉽게 호출하고, 더 쉽게 이해하고 더 쉽게 구현이 가능하다.

문자열이나 배열의 길이를 반환하는 함수를 고려해 보자:

```ts
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
  return x.length;
}
```

이 함수는 훌륭하다; 우리는 배열이나 문자열로 이 함수의 호출이 가능하다.
그러나, 이 함수는 스트링 *또는* 배열일 수 있는 값으로 호출이 불가능 하다, 왜냐하면 타입스크립트는 함수의 호출을 오직 하나의 오버로드로 해석하기 때문이다:

```ts
declare function len(s: string): number;
declare function len(arr: any[]): number;
//cut
len(""); // OK
len([0]); // OK
len(Math.random() > 0.5 ? "hello" : [0]);
```

두개의 오버로드가 같은 아규먼트 갯수와 반환 타입을 가지므로, 오버로드 대신 오버로드가 아닌 함수로 작성이 가능하다:

```ts
function len(x: any[] | string) {
  return x.length;
}
```

이것이 훨씬 낫다!
호출자는 두 종류의 값으로 호출이 가능하고, 추가적인 보너스로, 정확한 구현 시그니쳐를 찾아낼 필요도 없다.

> 가능하다면 오버로드 대신에 유니온 타입을 파라메터로 사용하라

## Other Types to Know About

함수 타입으로 작업하다보면 자주 나타나는 것으로서 인식되기를 원하는 추가적인 타입이 몇몇 있다.
모든 타입과 마찬가지로, 그들은 어디에나 사용될 수 있으나, 특별히 함수 문맥에서 유용하다.

### `void`

`void` 는 함수가 값을 반환하지 않는다는 것을 나타낸다.
그것은 함수가 `return` 문장을 가지고 있지 않은 경우 언제나 추정된다 또는 `return` 문장이 있더라도 명백히 아무런 값을 반환하지 않는 경우에도 마찬가지 이다:

```ts
// The inferred return type is void
function noop() {
  return;
}
```

자바스크립트에서는, 함수가 명확히 값을 반환하지 않는 경우 그 값은 `undefined`이다.
그러나, 타입스크립트에서 `void` 와 `undefined`는 같은 것이 아니다.
이것에 대한 기나긴 토론에 대해 보려면 레퍼런스페이지를 보라[[Why void is a special type]].

> `void` 는 `undefined`와 다른 것이다.

### `object`

특별한 타입인 `object`는 프리미티브인 (`string`, `number`, `boolean`, `symbol`, `null`, or `undefined`)를 제외한 어떤 값을 지칭한다.
이것은 *빈 객체 타입* `{ }`와도 다르고, 글로벌 타입인 `Object`와도 다르다.
`Object`와 관련된 정보를 레퍼런스 페이지 [[The global types]]에서 볼 수 있다 - 긴 이야기를 짧게 하자면, 절대로 `Object`를 사용하지 말라.

> `object` 는 `Object`가 아니다. **항상** `object`를 사용하라!

자바스크립트에서는, 함수 값은 objects이다: 그들은 프로퍼티를 가지고, 프로퍼티 체인에 `Object.prototype`를 갖는, 그 프로퍼티는 `instanceof Object`이고 그들에 대해 `Object.keys`를 호출할 수 있다, 기타 등등.
이러한 이유 때문에, 타입스크립트에서 함수 타입은 `object` 로 간주된다.

### `unknown`

`unknown`은 *어떠한* 값을 대표한다.
이것은 `any`와 비슷하지만, 더 안전한데 그 이유는 `unknown` 값에 대해 어떠한 행위도 에러이기 때문이다.

```ts
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b();
}
```

함수 몸체에 `any` 값들을 갖지 않고서도 어떤 값들이라도 받는 함수를 기술할 때 이것은 유용하다.

반대로, 알 수 없는 타입의 값을 반환하는 함수를 사용할 때도 유용하다.

```ts
declare const someRandomString: string;
//cut
function safeParse(s: string): unknown {
  return JSON.parse(s);
}

// Need to be careful with 'obj'!
const obj = safeParse(someRandomString);
```

### `never`

어떤 함수들은 절대로 값을 반환하지 않는다(*never*):

```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```

`never` 타입은 절대로 나타날 수 없는 값을 대표한다.
반환 타입으로 사용되면, 그 의미는 함수가 예외를 던지거나 프로그램의 실행을 종료한다는 것이다.

`never` 는 또한 타입스크립트에서 유니온에 아무 것도 남아있지 않다는 것을 의미한다.

```ts
function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    x; // has type 'never'!
  }
}
```

### `Function` {#the-global-function-type}

전역 타입인 `Function` 은 `bind`, `call`, `apply`와 같은 프로퍼티를 기술하며, 자바스크립트에서는 제공된 모든 함수 값을 기술한다.
`Function` 타입의 값을 갖는 특별한 프로퍼티를 갖는데 이 프로퍼티는 언제나 호출 가능하고 `any`를 반환한다:

```ts
function doSomething(f: Function) {
  f(1, 2, 3);
}
```

이것은 *untyped function call*이며, 안전하지 않은 `any` 타입을 반환하므로 사용을 피하는 것이 제일 좋은 전략이다.

임의의 함수를 받아야할 필요가 있고, 호출할 의도가 없다면 `() => void` 타입이 일반적으로 더 안전하다.

## Rest Parameters and Arguments

**Background reading**: [Rest Parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters) and [Spread Syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)

### Rest Parameters

고정된 숫자의 아규먼트를 다양하게 받으려고 오버로드나 선택적 파라메터를 사용하는 것 외에, *rest parameters*를 사용하여 *unbounded* 숫자의 아규먼트를 받아들이는 함수를 정의하는 것이 가능하다.

레스트 파라메터는 다른 모든 파라메터 이후에 나타나며 `...` 문법을 사용한다:

```ts
function multiply(n: number, ...m: number[]) {
  return m.map(x => n * x);
}
// 'a' gets value [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```

타입스크립트에서, 이러한 파라메터에 대한 타입의 묵시적으로 `any` 가 아니고 `any[]` 이며, 어떠한 타입 명시도 `Array<T> `또는  `T[]`, 또는 튜플 타입이어야 한다(튜플 타입은 나중에 다룰 것이다).

### Rest Arguments

반대로, 우리는 변하는 숫자의 아규먼트를 배열로부터 스프레드 문법을 사용하여 *제공*할 수 있다.
예를 들어, 배열의 `push` 메소드는 어떠한 개수의 아규먼트도 받을 수 있다:

```ts
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```

일반적으로 타입스크립트는 배열이 변경 불가능하다고 가정하지 않는다.
이것은 약간 놀라운 행동으로 나타나는데:

```ts
// Inferred type is number[] -- "an array with zero or more numbers",
// not specfically two numbers
const args = [8, 5];
const angle = Math.atan2(...args);
```

이것을 정정하기 위한 가장 좋은 방법은 약간 당신의 코드에 의존하는데, `const`를 사용하는 것이 일반적으로 가장 직관적인 해결책이다:

```ts
// Inferred as 2-length tuple
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```

<!-- TODO link to downlevel iteration -->

## Parameter Destructuring

>> **Background reading**: [Destructuring Assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

함수 몸체의 있는 지역 변수에다가 바로 아규먼트를 언팩해서 편리하게 제공할 수 있는 방법으로 파라메터 디스트럭쳐링을 사용할 수 있다.
자바스크립트에서, 그것은 다음과 비슷할 것이다:

```js
function sum({ a, b, c }) {
  console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```

객체의 타입 명시는 디스트럭칭 문법 다음에 나타난다: 

```ts
function sum({ a, b, c }: { a: number, b: number, c: number }) {
  console.log(a + b + c);
}
```

이것은 약간 거창하게 보인다 그래서 다음과 같은 타입명 방식도 사용가능 하다:

```ts
// Same as prior example
type ABC = { a: number, b: number, c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

## Assignability of Functions
