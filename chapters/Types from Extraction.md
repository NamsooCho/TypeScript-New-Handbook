# Types from Extraction

타입스크립트의 타입 시스템은 타입을 다른 타입의 측면에서 표현할 수 있도록 허용하기 때문에 매우 강력하다.
이것의 가장 간단한 형태가 제너릭이지만 우리는 실젤 광범위한 *타입 연산자*를 사용할 수 있다.
또한 타입을 이미 우리가 갖고 있는 값의 측면에서 표현할 수도 있다.

다양한 타입 연산자를 결합해서, 우리는 간결하고 유지보수 가능한 방법으로 복잡한 연산이나 값을 표현할 수 있다.
이 챕터에서 우리는 기존 타입이나 값의 측면에서 타입를 표현하는 다양한 방법에 대해 다룰 것이다.

__toc__

## The `typeof` type operator  {#typeof}

자바스크립트는ㄴ 이미 `typeof` 연산자를 가지고 있는데 이 것을 우리는 *식* 문맥에서 사용할 수 있다:

```ts
// Prints "string"
console.log(typeof "Hello world");
```

타입스크립트는 `typeof` 연산자에다 *타입* 문맥에 사용하도록 추가되었는데, 이것은 변수나 프로퍼티의 타입을 의미한다.

```ts
let s = "hello";
let n: typeof s;
    ^?
```

이것은 기본 타입에 대하여는 별로 유용하지 않지만 다른 타입 연산자와 결합하여 당신은 `typeof` 연산자를 편리하게 다양한 패턴으로 사용이 가능하다.
예를 들어, 미리 정의된 타입 `ReturnType<T>`를 살펴보는 것으로 시작해 본다.
그것은 *함수 타입*을 받아서 그 리턴 타입을 생산한다:

```ts
type Predicate = (x: unknown) => boolean;
type K = ReturnType<Predicate>;
     ^?
```

만약 `ReturnType`을 함수 이름에 대하여 사용하면, 에러가 나타남을 볼 것이다:

```ts
function f() {
    return { x: 10, y: 3 };
}
type P = ReturnType<f>;
```

*값* 과 *타입*은 같은 것이 아니라는 것을 기억하라.
*값 `f`*의 *타입*을 갖기 원한다면 `typeof`를 사용해야 한다:

```ts
function f() {
    return { x: 10, y: 3 };
}
type P = ReturnType<typeof f>;
     ^?
```

### Limitations

타입스크립트는 의도적으로 `typeof`를 사용할 수 있는 식의 종류를 제한 한다.
특히, `typeof`를 변수 이름같은 identifiers나 그들의 프로퍼티에만 사용하는 것이 규칙에 맞는다.
이것은 당신 생각에 돌아갈 것 같은 실제로는 안 돌아가는 코드를 작성할 때의 혼돈을 피하게 해준다:

```ts
declare const msgbox: any;
type msgbox = any;
//cut
// Meant to use =
let x : msgbox("Are you sure you want to continue?");
```

## The `keyof` type operator {#keyof}

`keyof` 연산자는 객체 타입을 받아서, 그것의 키로 문자열이나 숫자 리터럴을 생산한다:

```ts
type Point = { x: number, y: number };
type P = keyof Point;
     ^?
```

만약 타입이 `string` 또는  `number` 인덱스 시그니쳐를 가지면, `keyof`는 그러한 타입을 그대신 반환한다:
If the type has a `string` or `number` index signature, `keyof` will return those types instead:

```ts
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;
     ^?

type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
     ^?
```

이 예제에서 `M` 은 `string | number`이 됨을 유념하라 -- 이것은 자바스크립트의 객체 키는 항상 스트링으로 강제 형변환 되기 때문이다, 따라서 `obj[0]`은 항상 `obj["0"]`과 같다.

`keyof`는 mapped types과 결합되어 사용될 때 특히 유용한데, 나중에 이에 대해 더 다룰 것이다.

## Indexed Access Types

`typeof` 연산자는 값의 프로퍼티의 타입을 참조하는데 사용할 수 있다.
그대신 타입의 프로퍼티의 타입을 참조하는 것을 원한다면?

우리는 *indexed access type* 을 사용하여 다른 타입의 특별한 프로퍼티를 읽어볼 수 있다:

```ts
type Person = { age: number, name: string, alive: boolean };
type A = Person["age"];
     ^?
```

인덱스된 타입 그 자체는 타입이다, 따라서 유니온, `keyof`, 또는 다른 타입들 전체를 사용 할 수 있다:

```ts
type Person = { age: number, name: string, alive: boolean };
//cut
type I1 = Person["age" | "name"];
     ^?

type I2 = Person[keyof Person];
     ^?

type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName];
     ^?
```

만약 존재하지 않는 프로퍼티 인덱스를 시도하면 에러를 얻을 것이다:

```ts
type Person = { age: number, name: string, alive: boolean };
//cut
type I1 = Person["alve"];
```

다른 임의의 타입을 인덱싱하는 예제는 `number`를 사용하여 배열 엘리먼트의 타입을 얻는 것이다.
우리는 `typeof` 연산자와 결합하여 배열 리터럴의 요소 타입을 간편하게 가져올 수 있다:

```ts
const MyArray = [
    { name: "Alice", age: 15 },
    { name: "Bob", age: 23 },
    { name: "Eve", age: 38 }
];

type T = (typeof MyArray)[number];
     ^?
```
