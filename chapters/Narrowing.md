# Narrowing

`padLeft`라는 함수를 갖고 있다고 가정하자.

```ts
function padLeft(padding: number | string, input: string): string {
    throw new Error("Not implemented yet!");
}
```

만약 `padding`이 `number`이면, `input`에다가 추가할 공백의 갯수로 취급할 것이다.
만약 `padding`이 `string`이면, 단지 `input`에 추가할 문자열 `padding`으로 취급되어야 한다.
`padLeft`에 `padding`을 `number`로 넘겨주는 때를 위한 로직을 구현해 보자.

```ts
function padLeft(padding: number | string, input: string) {
    return new Array(padding + 1).join(" ") + input;
}
```

어허, `padding + 1`에 에러가 난다.
타입스크립트는 `number`를 `number | string`에다 더하기를 한다고 경고하며, 그것은 아마 우리가 원하는 결과를 가져 오지 않을 것이고, 이런 행동은 옳다.
다른 말로 하면, 우리는 명확히 처음부터 `padding`이 `number`인지 검사하지 않았다, 또한 그것이 `string`인 경우의 처리도 하지 않았다, 그러므로 이제 정확히 그것을 해보자.

```ts
function padLeft(padding: number | string, input: string) {
    if (typeof padding === "number") {
        return new Array(padding + 1).join(" ") + input;
    }
    return padding + input;
}
```

이것은 재미없는 자바스크립트 코드인 것처럼 보인다면, 그것은 요점의 하나이다.
우리가 타입 명시를 집어넣은 것 말고는, 이 타입스크립트 코드는 자바스크립트 처럼 보인다.
이러한 생각은, 타입스크립트의 타입 시스템이 타입 안정성을 위해 많은 것을 뒤돌아 보지 않게 하면서 일반적 자바스크립트를 작성하기에 쉽게 하도록 하려는 목표 때문이다.

이것은 많은 것인 것처럼 보이지 않을 수 있지만, 실제로는 수면 아래에서 많은 것들이 오고 간다.
정적타입을 사용해서 타입스크립트가 런타임 값들을 분석하는 방식과 매우 비슷하게, 자바스크립트의 런타임 흐름 제어 요소들(`if/else`, conditional ternaries, loops, truthiness checks, etc.)은 이러한 타입에 영황을 끼치고,
이것들은 타입/자바스크립트에서 많이 겹친다. 

우리의 `if` 절 안에서, 타입스크립트는 `typeof padding === "number"`을 바라보게 되고, *type guard*라고 불리우는 특별한 형태를 이해하게 된다.
타입스크립트는 가능한 실행 경로를 따르면서, 주어진 위치에서 프로그램이 가질수 있는 모든 특정 타입의 값을 갖는지 분석한다.
그것은 이러한 특별한 검사(*type guards*로 불리우는)와 대입문을 살펴보며, 타입들을 더욱 세부적인 타입으로 다듬는데 이것을 우리는 *narrowing*이라 부른다.
많은 에디터에서, 그들이 변해가는 많은 이러한 타입을 볼 수 있으며, 우리의 예제에서 우리 스스로 이것을 해볼 것이다.

```ts
function padLeft(padding: number | string, input: string) {
    if (typeof padding === "number") {
        return new Array(padding + 1).join(" ") + input;
                         ^?
    }
    return padding + input;
           ^?
}
```

타입스크립트가 narrowing을 위해 이해하는 몇몇의 다른 구조가 있다.

## `typeof` type guards

우리가 보아 왔듯이, 자바스크립트는 `typeof` 연산자를 지원하며, 런타임에 값의 타입에 관한 기본적 정보를 준다.
타입스크립트는 이 연산자가 다음의 스트링 집합을 반환하기를 기대한다:

* `"string"`
* `"number"`
* `"bigint"`
* `"boolean"`
* `"symbol"`
* `"undefined"`
* `"object"`
* `"function"`

`padLeft`에서 본 것처럼, 이 연산자는 많은 자바스크립트 라이브러리에서 매우 자주 보이며, 타입스크립트는 서로 다른 브랜치에서 타입을 narrow하는 것으로 이해할 수 있다.

타입스크립트에서, `typeof` 연산자로 반환되는 값의 검사는 type guard 이다.
타입스크립트는 서로 다른 값에 대해서 `typeof` 연산자가 사용되는 것을 인코딩 하므로, 자바스크립트에서의 별난 것들에 대해 알고 있다.
예를 들어, 위 리스트에서 `typeof`는 `null`을 반환하지 않는다는 것을 유념하라.
다음의 예를 살펴보자:

```ts
function printAll(strs: string | string[] | null) {
    if (typeof strs === "object") {
        for (const s of strs) {
            console.log(s)
        }
    }
    else if (typeof strs === "string") {
        console.log(strs)
    }
    else {
        // do nothing
    }
}
```

`printAll` 함수에서 `strs`이 배열 타입인지 보기위해 객체인지 검사한다(자바스크립트에서 배열은 객체임을 다시 강조할 시점인듯).
그러나 자바스크립트에서 `typeof null`은 실제로 `"object"` 임이 드러난다.
이것은 역사의 가장 불행한 사고 중 하나이다.

충분한 경험을 가진 사용자들은 아마 놀라지 않을 것이지만, 모든 사람이 자바스크립트에서 이러한 상황에 빠지지는 않는다; 다행히도, 타입스크립트는 `strs` 가 단지 `string[]`으로가 아니라 `string[] | null`으로 좁혀 졋다고 알게 해줄 것이다.

이제 우리가 부르는 "truthiness" 검사라는 주제로 넘어가 보자.

# Truthiness narrowing

Truthiness는 사전에서 찾아보는 그런 단어가 아닐 수도 있지만, 자바스크립트에서 들어보았던 그 무엇과 매우 유사하다.
<!-- TODO: I'm on an airplane, is truthiness in the dictionary?? -->
자바스크립트에서, 조건식에서 우리는 `&&`, `||`, `if` 문, `!` 연산자, 기타 등등을 사용할 수 있다.
예를 들어, `if` 문장은 그것의 조건이 반드시 `boolean` 타입을 갖도록 기대하는 것은 아니다.

```ts
function getUsersOnlineMessage(numUsersOnline: number) {
    if (numUsersOnline) {
        return `There are ${numUsersOnline} online now!`;
    }
    return "Nobody's here. :(";
}
```

자바스크립트에서, `if`와 같은 구조는 먼저 조건을 `boolean`으로 형변환 하여 그들이 의미를 갖게 하고 그리고 나서 결과가 참이냐 거짓이냐에 따라 분기를 한다.
다음과 값은 값들은

* `0`
* `NaN`
* `""` (the empty string)
* `0n` (the `bigint` version of zero)
* `null`
* `undefined`

모두 `false`로 형변환 되며, 다른 값들은 모두 `true`로 형변환 된다.
`Boolean` 함수를 통해서 또는 더 짧은 이중 negation을 사용해서 우리는 언제나 값을 `boolean`으로 형변환 가능하다.

```ts
// both of these result in 'true'
Boolean("hello");
!!"world";
```

이러한 행동을 이용하는 것은 매우 흔한 일이고, 특히 `null` or `undefined` 와 같은 값들을 보호하기에 적합하다.
예를 들자면, `printAll` 함수에 이것을 사용해 보자.

```ts
function printAll(strs: string | string[] | null) {
    if (strs && typeof strs === "object") {
        for (const s of strs) {
            console.log(s)
        }
    }
    else if (typeof strs === "string") {
        console.log(strs)
    }
}
```

`strs`에 대한 truthy 검사를 함으로써 위에서의 에러를 제거한 것을 눈치 챘을 것이다.
최소한 이것은 코드를 실행했을 때 다음과 같은 염려되는 에러를 막아준다:

```txt
TypeError: null is not iterable
```

그러나 프리미티브 타입에 대한 truthiness 검사는 에러가 발생하기 쉽다는 것을 유념하라.
예를 들어, `printAll`를 작성하는 다른 접근 방식을 고려해 보자면:

```ts
function printAll(strs: string | string[] | null) {
    // !!!!!!!!!!!!!!!!
    //  DON'T DO THIS!
    //   KEEP READING
    // !!!!!!!!!!!!!!!!
    if (strs) {
        if (typeof strs === "object") {
            for (const s of strs) {
                console.log(s)
            }
        }
        else if (typeof strs === "string") {
            console.log(strs)
        }
    }
}
```

우리는 함수의 몸체 전체에 truthy 검사를 둘러 쌌는데, 이것은 미묘한 단점이 있다; 우리는 더이상 빈 문자열을 정확히 처리할 수 없다.
타입스크립트는 여기서 전혀 해롭지 않다, 그러나 이것은 자바스크립트와 덜 친하다면 이러한 행동은 주목할 만한 가치가 있다.
타입스크립트는 때때로 당신이 일찍 버그를 잡아내도록 도와주기도 한다. 그러나 당신이 어떤 값으로 *nothing* 을 한다면(즉 아무일도 안한다면), 매우 규범적이 되지 않고는 타입스크립트가 할수 있는 일은 많지 않을 것이다.
원한다면 린터와 같은 것으로도 이러한 상황을 처리할수 있다고 확신할 수도 있다.

truthiness에 의한 narrowing에 대한 마지막 말은 `!`(불리언 니게이션 연산자)를 사용하여 negated 브랜치를 걸러낼 수 있다는 것이다.

```ts
function multiplyAll(values: number[] | undefined, factor: number): number[] | undefined {
    if (!values) {
        return values;
    }
    else {
        return values.map(x => x * factor);
    }
}
```

## Equality narrowing

타입스크립트는 또한 `switch` 문장과 `===`, `!==`, `==`, 그리고 `!=`와 같은 등가 연산자를 사용하여 타입 narrow를 한다.
예를 들자면:

```ts
function foo(x: string | number, y: string | boolean) {
    if (x === y) {
        // We can now call any 'string' method on 'x' or 'y'.
        x.toUpperCase();
        ^?
        y.toLowerCase();
        ^?
    }
    else {
        console.log(x);
                    ^?
        console.log(y);
                    ^?
    }
}
```

위 예제에서 `x` 와 `y`가 같다는 검사를 할 때, 타입스크립트는 그들의 타입 또한 같아야 한다는 것을 안다.
`string` 만이 공통의 타입으므로 타입스크립트는 `x` 와 `y`는 `string` 일 때에만 첫번째 분기를 탄다.

또한 특정 리터럴 값을 검사하는 것 또한 잘 동작 한다.
truthiness narrowing에 관한 섹션에서, 빈 문자열을 적절히 처리하지 못한 에러가 나기 쉬운 `printAll` 함수를 작성한 적이 있다.
그 대신에 `null`을 막아주는 특별한 검사를 할 수도 있는데, 타입스크립트는 `strs` 타입으로 부터 `null`을 정확히 제거한다.

```ts
function printAll(strs: string | string[] | null) {
    if (strs !== null) {
        if (typeof strs === "object") {
            for (const s of strs) {
                            ^?
                console.log(s);
            }
        }
        else if (typeof strs === "string") {
            console.log(strs);
                        ^?
        }
    }
}
```

자바스크립트의 더 느슨한 동등성 검사인 `==` 와 `!=` 또한 정확히 narrow 타입을 얻게 해준다.
당신이 만약 익숙하지 않다면, 무언가가 `== null` 인가를 검사하는 것은 단지 그 것이 `null` 값인가 만을 검사하는 것은 아니라는 것 - 그것은 또한 잠재적으로 `undefined`인가도 검사한다.
동일한 것이 `== undefined` 에도 적용된다: 그것은 어떠한 값이 `null` 또는 `undefined` 인가를 검사한다.

```ts
interface Container {
    value: number | null | undefined
}

function multiplyValue(container: Container, factor: number) {
    // Remove both 'null' and 'undefined' from the type.
    if (container.value != null) {
        console.log(container.value);
                              ^?

        // Now we can safely multiply 'container.value'.
        container.value *= factor;
    }
}
```

## `instanceof` narrowing

자바스크립트는 어떤 값이 다른 값의 "instance" 인지 아닌지 검사하는 연산자가 있다.
더 엄밀하게는, 자바스크립트에서 `x instanceof Foo`는 `x`의 *프로토타입 체인*이 `Foo.prototype`을 포함하는지 검사한다.
여기서 깊이 들어갈 것은 아니지만, 당신은 클래스에 들어갈 때 이것에 대해 더 많은 것을 보게 될 것이며, `new`로 만들어진 대부분의 값들에 대해 여전히 유용하다.
당신이 추측하듯이, `instanceof`는 또한 type guard 이며, 타입스크립트는 `instanceof`로 가드된 브랜치로 narrows 할 것이다.

```ts
function logValue(x: Date | string) {
    if (x instanceof Date) {
        console.log(x.toUTCString());
                    ^?
    }
    else {
        console.log(x.toUpperCase());
                    ^?
    }
}
```

## Assignments

앞에서 언급했듯이, 어떤 값을 대입하면, 타입스크립트는 대입문의 오른쪽을 보아서 적절히 왼쪽을 narrows 한다.

```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";
    ^?
x = 1;

console.log(x);
            ^?
x = "goodbye!";

console.log(x);
            ^?
```

위의 모든 문장이 옳다는 것을 유념하라.
첫 문장 이후로 `x`의 타입이 `number`로 관찰되어도, 우리는 여전히 `x`에 `string`을 대입할 수 있다.
이것은 `x`의 *선언된 타입* - `x`의 시작 타입 - 이 `string | number` 타입이고, 대입 가능성 검사는 항상 선언된 타입으로 이루어 지기 때문이다.

만약 우리가 `x`에 `boolean`을 대입했다면, 선언된 타입과 다른 타입이므로 에러가 나올 것이다.

```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";
    ^?
x = 1;

console.log(x);
            ^?
x = true;

console.log(x);
            ^?
```

## Control flow analysis

이 지점 까지, 타입스크립트가 어떻게 특정 브랜치에서 narrows를 하는지 기본적 예제를 가지고 왔다.
그러나 단지 모든 변수에 대해 살펴보고 `if`s, `while`s, 조건식, etc들을 살펴보는 것 이상의 것들이 있다.
예를 들자면

```ts
function padLeft(padding: number | string, input: string) {
    if (typeof padding === "number") {
        return new Array(padding + 1).join(" ") + input;
    }
    return padding + input;
}
```

`padLeft` 는 첫번째 `if` 블럭에서 리턴할 수 있다.
타입스크립트는 이 코드를 분석할 수 있으며 몸체의 나머지 부분(`return padding + input;`)이 만약 `padding`이 `number`이면 *unreachable* 하다는 것을 알아낸다.
결과적으로, 그것은 `padding`의 타입중에서 `number`를 제거하여(narrowing from `string | number` to `string`) 함수의 나머지에 넘겨주는 기능을 한다.

이러한 도달 가능성에 기반한 분석을 *제어 흐름 분석*이라고 하며, 타입스크립트는 타입가드와 대입문을 만나면 이 흐름분석을 행하여 타입 narrow를 한다.
변수가 분석될 때, 제어 흐름은 분리되거나 재겹합 되고 이것은 반복될 수 있다, 그리고 그 변수는 각 지점에서 다른 타입을 가지는 것을 보게 될 수 있다.

```ts
function foo() {
    let x: string | number | boolean;

    x = Math.random() < 0.5;

    console.log(x);
                ^?

    if (Math.random() < 0.5) {
        x = "hello";
        console.log(x);
                    ^?
    }
    else {
        x = 100;
        console.log(x);
                    ^?
    }

    return x;
           ^?
}
```

# Discriminated unions

지금까지 우리가 살펴본 대부분의 예제는 `string`, `boolean`, and `number`와 같은 간단한 타입에 하나의 변수를 narrowing 하는 것을 다루었다.
비록 이것이 일반적이긴 하지만, 자바스크립트에서 대부분의 경우는 약간 더 복잡한 구조를 다루는 것이다.

몇가지 동기로, 원이나 사각형 같은 모양을 인코딩 한다고 가정해 보자.
원은 반지름을 따르고, 사각형은 그들의 측면의 길이를 따른다.
우리는 `kind`라고 명명한 필드를 사용하여 우리가 다루는 것이 무슨 모양인지 구분할 것이다.
여기에 우리의 `Shape`를 정의하는 첫번째 시도가 있다.

```ts
interface Shape {
    kind: "circle" | "square";
    radius?: number;
    sideLength?: number;
}
```

우리가 원을 다루는지 사각형을 다루는지 구분하기 위해 `"circle"` 과 `"square"`의 스트링 리터럴 타입을 사용한 것을 유의하라.
`string` 대신에 `"circle" | "square"` 을 사용함으로써 스펠링 오류를 피할 수 있다.

```ts
interface Shape {
    kind: "circle" | "square";
    radius?: number;
    sideLength?: number;
}

//cut
function handleShape(shape: Shape) {
    // oops!
    if (shape.kind === "rect") {
        // ...
    }
}
```

원이나 사각형에 따라서 넓이를 적절히 구하는 `getArea` 함수를 작성할 수 있다.
우리는 처음에 원에 대해 다루어 본다.

```ts
interface Shape {
    kind: "circle" | "square";
    radius?: number;
    sideLength?: number;
}

//cut
function getArea(shape: Shape) {
    return Math.PI * shape.radius ** 2;
}
```

<!-- TODO -->
`strictNullChecks`을 사용한 경우 이것은 에러가 된다 - `radius`가 정의되지 않을 수 있으므로 이것은 적절하다.
그러면 `kind` 프로퍼티에 대해 적절히 검사를 한다면?

```ts
interface Shape {
    kind: "circle" | "square";
    radius?: number;
    sideLength?: number;
}

//cut
function getArea(shape: Shape) {
    if (shape.kind === "circle") {
        return Math.PI * shape.radius ** 2;
    }
}
```

흠, 타입스크립트는 여전히 여기서 무얼 해야 할지 모른다.
이 지점은 우리가 타입 체커가 하는 것 이상으로 우리의 값에 대해 알고 있는 곳이다.
우리는 non-null assertion(`shape.radius` 다음에 `!`를 붙인 것)을 시도하여 `radius`가 확실히 존재한다고 알려줄 수 있다.

```ts
interface Shape {
    kind: "circle" | "square";
    radius?: number;
    sideLength?: number;
}

//cut
function getArea(shape: Shape) {
    if (shape.kind === "circle") {
        return Math.PI * shape.radius! ** 2;
    }
}
```

그러나 이것은 느낌상 이상적이지 않다.
우리는 타입 체커에게 이러한 non-null assertions (`!`)을 가지고 `shape.radius`가 정의되었다고 외쳐야 하지만, 이러한 것은 에러가 되기 쉬운데, 만약 우리가 코드를 주변에 옮기기 시작하면 그렇다.
추가적으로, `strictNullChecks`을 사용하지 않을 경우, 우리는 이러한 필드를 어떠한 값이 되었든 재앙적으로 접근하게 될 것이다(선택적 프로퍼티는 그 코드를 읽을 때 항상 값이 있다고 가정하기 때문이다).
우리는 확실하게 이보다 더 잘할 수 있다.

`Shape`를 이렇게 인코딩 하는 것의 문제는 타입체커가 `kind` 프로퍼티에 기반하여 `radius` 또는 `sideLength`가 존재하는지 알 길이 없다는 것이다.
우리는 우리가 알고 있는 것을 타입체커와 소통할 필요가 있다.
이것을 염두에 두고, `Shape`을 인코딩 하는 다른 접근을 해보자.

```ts
interface Circle {
    kind: "circle";
    radius: number;
}

interface Square {
    kind: "square";
    sideLength: number;
}

type Shape = Circle | Square;
```

여기서, 우리는 `Shape`를 `kind` 프로퍼티에 따라서 다른 값을 가지는 두 개의 타입으로 분리 했다, 그러면서 `radius` 와 `sideLength`는 자신의 타입에서 필요한 값으로 선언 했다.

우리가 `Shape`의 `radius`를 접근할 때 어떤 일이 벌어지는지 보자.

```ts
interface Circle {
    kind: "circle";
    radius: number;
}

interface Square {
    kind: "square";
    sideLength: number;
}

type Shape = Circle | Square;

//cut
function getArea(shape: Shape) {
    return Math.PI * shape.radius ** 2;
}
```

첫번째 시도와 마찬가지로 이것은 여전히 에러이다.
`radius`가 선택적 이었다면 `strictNullChecks` 상황에서 우리는 에러를 얻는다 왜냐하면 타입스크립트는 그 프로퍼티가 제공 되었는지 모르기 때문이다.
지금은 `Shape`가 유니언 이므로 타입스크립트는 `shape`가 `Square` 일 수 있고 `Square`는 `radius`가 정의되어 있지 않다는 것을 말해 준다!
두개의 해석이 모두 옳다, 그러나 우리의 새로운 `Shape` 인코딩은 여전히 `strictNullChecks` 없이는 에러를 일으킬 수 있다.

그러나 만약 우리가 다시 `kind` 프로퍼티를 검사한다면?

```ts
interface Circle {
    kind: "circle";
    radius: number;
}

interface Square {
    kind: "square";
    sideLength: number;
}

type Shape = Circle | Square;

//cut
function getArea(shape: Shape) {
    if (shape.kind === "circle") {
        return Math.PI * shape.radius ** 2;
                         ^?
    }
}
```

이것은 에러를 제거한다!
유니온의 모든 타입이 공통의 리터럴 타입을 갖는다면, 타입스크립트는 이것을 *차별화된 유니언*으로 간주하고 유니언의 멤버를 narrow 할 수 있다.

이 경우에는, `kind`가 공통의 프로퍼티이다(`Shape`의 *차별화* 프로퍼티로 간주된다).
`kind` 프로퍼티가 `"circle"` 인가를 검사함으로써 `kind`가 `"circle"` 이 갖고 있지 않는 프로퍼티를 갖는 모든 `Shape`의 타입을 제거 할 수 있다.
narrowed된 `shape`는 `Circle` 타입으로 좁혀진다.

같은 검사가 `switch` 문장에도 역시 동작한다.
이제 우리는 `!` non-null assertions 없이 우리의 `getArea` 를 완성할 수 있다.

```ts
interface Circle {
    kind: "circle";
    radius: number;
}

interface Square {
    kind: "square";
    sideLength: number;
}

type Shape = Circle | Square;

//cut
function getArea(shape: Shape) {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
                             ^?
        case "square":
            return shape.sideLength ** 2;
                   ^?
    }
}
```

여기서 중요한 것은 `Shape`의 인코딩이다.
타입스크립트와 정확한 정보로 소통하는 것 - `Circle` 과 `Square`는 진실로 별개의 타입이고 `kind` 필드로 구분이 가능하다는 점 - 이 매우 중요하다.
그렇게 함으로써 우리는 타입 안전한 타입스크립트 코드를 작성할 수 있고 그 코드는 자바스크립트로 작성되었을 경우와도 다르게 보이지 않을 것이다.
거기서, 타입 시스템은 *옳은* 일을 해낼 수 있었고, `switch` 문장의 각 브랜치에서 무슨 타입인지 판별해 낼 수 있었다.

> 별도로, 위 예제를 가지고 여러 시도를 해보고 몇몇 반환 키워드를 제거해보자.
> 당신은 타입 검사가 `switch` 문장의 다른 구절에서 실수로 진입하는 버그를 피하도록 도와주는 것을 보게 될 것이다.

차별화된 유니온은 단순히 원과 사각형에 대해 말하는 것보다 유용하다.
그들은 자바스크립트에서 어떤 메시징 종류도 표현이 가능한 좋은 것이다, 네트웍을 통해서 메시지들을 전송하는 것처럼(클라이언트/서버 통신), 또는 상태 관리 프레임워크에서 쓰기 허용을 인코딩 하는 것처럼.

# The `never` type

```ts
interface Circle {
    kind: "circle";
    radius: number;
}

interface Square {
    kind: "square";
    sideLength: number;
}

type Shape = Circle | Square;

//cut
function getArea(shape: Shape) {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "square":
            return shape.sideLength ** 2;
    }
}
```

<!-- TODO -->

# Exhaustiveness checking

<!-- TODO -->

<!--
As another example, consider a `setVisible` function, that takes an `HTMLElement` and either takes a `boolean` to set whether or not the element is visible on the page, or a `number` to adjust the element's opacity (i.e. how non-transparent it is).

```ts

```
-->