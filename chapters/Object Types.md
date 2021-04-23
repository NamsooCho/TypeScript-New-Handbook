# Object Types

자바스크립트에서, 우리가 관계있는 데이터를 구룹짓고 전달하는 근본적 방법은 객체를 통해서 이다.
타입스크립트에서, 우리는 그러한 것들을 *object types* 를 통해서 표현한다.

우리가 보아 왔듯이, 그들은 익명성을 가질 수 있다.

```ts
function greet(person: { name: string, age: number }) {
                       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    return "Hello " + person.age;
}
```

또는 그들은 인터페이스를 사용하여 이름을 부여하거나

```ts
interface Person {
^^^^^^^^^^^^^^^^
    name: string;
    age: number;
}

function greet(person: Person) {
    return "Hello " + person.age;
}
```

타입 별칭을 통하여 이름을 부여한다.

```ts
type Person = {
^^^^^^^^^^^
    name: string;
    age: number;
}

function greet(person: Person) {
    return "Hello " + person.age;
}
```

위 3 가지 예제에서, 우리는 객체를 받아들이는 함수를 작성하였고, 그 객체는 `string` 타입인 `name`과 `number` 타입인 `age` 프로퍼티를 갖고 있다.

## Property Modifiers

객체 타입에서 각각의 프로퍼티는 한쌍의 무엇을 정의한다: 타입, 그 프로퍼티가 생략 가능한지, 그리고 그 프로퍼티가 갱신 가능한지.

### Optional Properties

대부분의 경우, 우리는 프로퍼티 집합을 갖는 객체를 다루게 될 것이다.
그런 경우, 우리는 프로퍼티의 끝에 물음표 (`?`) 를 붙임으로써 그 프로퍼티가 *생략가능* 한지를 표시할 것이다.

```ts
interface Shape {};
declare function getShape(): Shape;

//cut
interface PaintOptions {
    shape: Shape;
    xPos?: number;
        ^
    yPos?: number;
        ^
}

function paintShape(opts: PaintOptions) {
    // ...
}

const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```

이 예제에서, `xPos` 와 `yPos`는 둘 다 생략가능하다.
우리는 그 둘 다 값을 제공할지 말지 선택이 가능하고, 따라서 위의 모든 `paintShape` 호출은 옳다.
모든 생략 가능성은 실제로 말하는 것은 그 프로퍼티가 셋팅된다면 어떤 특정한 타입을 가지는 것이 좋다는 것이다.

```ts
interface Shape {};
declare function getShape(): Shape;

//cut
interface PaintOptions {
    shape: Shape;
    xPos?: number;
        ^
    yPos?: number;
        ^
}

function paintShape(opts: PaintOptions) {
    // ...
}

const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
```

우리는 그러한 프로퍼티를 읽을 수 있는데 - 만약 `strictNullChecks` 하에서 우리가 이렇게 하면, 타입스크립트는 거기에 잠재적인 `undefined` 가 있다고 알려준다.

```ts
interface Shape {};
declare function getShape(): Shape;

interface PaintOptions {
    shape: Shape;
    xPos?: number;
        ^
    yPos?: number;
        ^
}

//cut
function paintShape(opts: PaintOptions) {
    let xPos = opts.xPos;
                    ^?
    let yPos = opts.yPos;
                    ^?
    // ...
}
```

자바스크립트에서, 그 프로퍼티가 셋팅되지 않을지라도, 우리는 여전히 그것을 접근할 수 있다 - 그것은 단지 우리에게 `undefined` 값을 줄 뿐이다.
우리는 특별히 `undefined`를 처리하기만 하면 된다.

```ts
interface Shape {};
declare function getShape(): Shape;

interface PaintOptions {
    shape: Shape;
    xPos?: number;
    yPos?: number;
}

//cut
function paintShape(opts: PaintOptions) {
    let xPos = opts.xPos === undefined ? 0 : opts.xPos;
        ^?
    let yPos = opts.yPos === undefined ? 0 : opts.yPos;
        ^?
    // ...
}
```

이런 식으로 미확정된 값들을 디폴트 값으로 셋팅하는 패턴은 자바스크립트에서 매우 흔한 문법적 지원을 갖고 있다.
Note that this pattern of setting defaults for unspecified values is so common that JavaScript has syntax to support it.

```ts
interface Shape {};
declare function getShape(): Shape;

interface PaintOptions {
    shape: Shape;
    xPos?: number;
        ^
    yPos?: number;
        ^
}

//cut
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
    console.log("x coordinate at", xPos);
                                   ^?
    console.log("y coordinate at", yPos);
                                   ^?
    // ...
}
```

여기서 우리가 사용한 [a destructuring pattern](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)은 `paintShape` 파라메터에 기본 값 부여 [default values](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Default_values) 이다.
이제 `xPos` 와 `yPos` 둘다 확정적인 값을 `paintShape` 몸체 내에서 갖게 되었으나, 여전히 `paintShape`를 호출하는 측에서는 생략가능하다.

<aside>
현재 디스트럭쳐링 패턴에서 타입 명시를 할 방법이 없다는 것을 유념하라.
그 이유는 자바스크립트에서 다른 무언가를 의미하는 문법이 이미 있기 때문이다.

```ts
// @noImplicitAny: false
interface Shape {}
declare function doSomething(x: unknown);
//cut
function foo({ shape: Shape, xPos: number = 100, /*...*/ }) {
    doSomething(shape);
    doSomething(xPos);
}
```

객체 디스트럭쳐링 패턴에서, `shape: Shape`의 의미는 `shape`를 받아서 `Shape`라는 이름의 로컬 변수로 재정의 한다는 것이다.
비스하게 `xPos: number`는 `number`라는 이름의 로컬 변수를 생성하며 그 값은 `xPos` 파라메터의 값에 근거 한다.
</aside>

### `readonly` Properties

프로퍼티는 타입스크립트에서 `readonly`로 표시 될 수 있다.
런타임에 그것은 어떠한 행동도 변경할 수 없게 되지만, `readonly`로 표시된 프로퍼티는 타입 체킹 동안에도 값이 변경될 수 없다.

```ts
interface SomeType {
    readonly prop: string;
}

function doSomething(obj: SomeType) {
    // We can read from 'obj.prop'.
    console.log(`prop has the value '${obj.prop}'.`);

    // But we can't re-assign it.
    obj.prop = "hello";
}
```

`readonly` 수정자를 사용하는 것이 반드시 그 값이 완전히 갱신 불가능하다는 것을 암시하지는 않는다 - 다른 말로 - 그 내부의 내용이 변경 불가능 하다는 것이 아니다.
그것은 단지 프로퍼티 자체가 다시 기록될수 없다는 것을 의미한다.

```ts
interface Home {
    readonly resident: { name: string, age: number };
}

function visitForBirthday(home: Home) {
    // We can read and update properties from 'home.resident'.
    console.log(`Happy birthday ${home.resident.name}!`);
    home.resident.age++;
}

function evict(home: Home) {
    // But we can't write to the 'resident' property itself on a 'Home'.
    home.resident = {
        name: "Victor the Evictor",
        age: 42,
    };
}
```

`readonly` 가 암시하는 것이 무엇인지 기대하는 것을 잘 관리하는 것은 중요하다.
개발 기간 동안에 타입스크립트에서 그 객체가 어떻게 사용되는지를 신호를 주는 용도로 유용하다.
타입스크립트는 두 타입이 호환성이 있는지 검사할 때 그들이 `readonly`인지 고려하지 않는다, 따라서 `readonly` 프로퍼티는 타입 별칭 하는 과정에서 변화가 가능하다.

```ts
interface Person {
    name: string;
    age: number;
}

interface ReadonlyPerson {
    readonly name: string;
    readonly age: number;
}

let writablePerson: Person = {
    name: "Person McPersonface",
    age: 42,
};

// works
let readonlyPerson: ReadonlyPerson = writablePerson;

console.log(readonlyPerson.age); // prints '42'
writablePerson.age++;
console.log(readonlyPerson.age); // prints '43'
```

## Extending Types

어떤 타입이 다른 타입의 보다 세부적인 타입을 갖는 일은 매우 흔한 일이다.
예를 들어, `BasicAddress` 타입이 있고 미국 내에서 우편과 소포를 보낼 수 있는 필드를 갖고 있다면.

```ts
interface BasicAddress {
    name?: string;
    street: string;
    city: string;
    country: string;
    postalCode: string;
}
```

몇몇 상황에서 이것 만으로 충분하다, 그러나 주소들은 때때로 주소상의 빌딩이 여러개의 유닛으로 구성된 경우 유닛 번호도 갖는다.
우리는 그러면 `AddressWithUnit`을 기술해야 한다.

```ts
interface AddressWithUnit {
    name?: string;
    unit: string;
    ^^^^^^^^^^^^^
    street: string;
    city: string;
    country: string;
    postalCode: string;
}
```

이것은 제 역할을 한다, 그러나 여기서의 단점은 `BasicAddress` 가 원래의 것으로도 충분히 동작 할 때에도 모든 주소에서 이 필드를 반복해야 한다는 것이다.
그 대신, `BasicAddress` 타입을 확장해서, 새로운 필드를 추가하고 독립적인 `AddressWithUnit` 타입을 만드는 것이다.

```ts
interface BasicAddress {
    name?: string;
    street: string;
    city: string;
    country: string;
    postalCode: string;
}

interface AddressWithUnit extends BasicAddress {
    unit: string;
}
```

`interface`에 사용하는 `extends` 키워드는 다른 이름의 타입에 정의된 멤버들을 효과적으로 복사하고 원하는 새로운 멤버를 추가하도록 해준다.
이것은 타입 선언의 보일러플레이트의 양을 삭감해주고, 동일한 프로퍼티의 몇개의 다른 선언이 관련되어 있다는 신호를 주려고 할 때 유용하다.
예를 들어, `AddressWithUnit` 는 `street`를 다시 반복할 필요가 없으며, 이것은 `street`이 `BasicAddress`에 있기 때문이다, 코드를 읽는 사람은 두 개의 타입이 몇몇 부분에서 서로 연관되어 있다는 것을 알게 될 것이다.

`interface`는 또한 다수의 타입들로 부터 확장하는데 사용될 수 있다.

```ts
interface Colorful {
    color: string;
}

interface Circle {
    radius: number;
}

interface ColorfulCircle extends Colorful, Circle {}

const cc: ColorfulCircle = {
    color: "red",
    radius: 42,
}
```

## Intersection Types

`interface`는 어떤 타입을 확장함으로써 새로운 타입을 만들도록 해준다.
타입스크립트는 *intersection types*라는 구조를 제공해서 기존에 존재하는 타입들을 결합하는데 사용되도록 해준다.

intersection 타입은 `&` 연산자를 사용해서 정의 된다.

```ts
interface Colorful { color: string; }
interface Circle { radius: number; }

type ColorfulCircle = Colorful & Circle;
```

여기서, `Colorful` 와 `Circle`를 교차시켜서 두 `Colorful` *그리고* `Circle`의 모든 멤버를 갖는 새로운 타입을 만들었다.

```ts
interface Colorful { color: string; }
interface Circle { radius: number; }
//cut
function draw(circle: Colorful & Circle) {
    console.log(`Color was ${circle.color}`);
    console.log(`Radius was ${circle.radius}`);
}

// okay
draw({ color: "blue", radius: 42});

// oops
draw({ color: "red", raidus: 42});
```

## Interfaces vs. Intersections

우리는 타입을 결합하는 비슷한 두가지 방법을 살펴보았다, 그러나 두 방법은 실제로는 미묘하게 다른 점이 있다.
인터페이스를 가지고는, `extends` 절을 사용하여 다른 타입을 확장하였고, 인터섹션으로 비슷한 일을 하여 타입 별칭으로 이름을 줄 수 있었다.
두 방법의 주요한 차이점은 충돌이 어떻게 해결되는 가에 달려있으며, 그 차이는 두 방법 중 어느 것을 선택하는가의 주요한 이유가 된다.

예를 들어, 인터페이스에서 두 타입은 같은 프로퍼티를 선언할 수 있다.

TODO

## Generic Object Types

무슨 값이든 담을 수 있는 타입인 `Box`를 상상해 보자.

```ts
interface Box {
    contents: any;
}
```

당장은, `contents` 프로퍼티가 `any` 타입이 되었고, 이것은 잘 동작한다, 그러나 그 라인 아래로 사고가 날 수 있다.

그대신 우리는 `unknown`을 사용할 수 있지만, 그것은 우리가 `contents`의 타입을 이미 알고 있는 경우를 의미하며, 우리는 예방적 검사나 에러가 나기 쉬운 타입 assertions을 사용할 필요로 한다.

```ts
interface Box {
    contents: unknown;
}

let x: Box = {
    contents: "hello world",
}

// we could check 'x.contents'
if (typeof x.contents === "string") {
    console.log(x.contents.toLowerCase());
}

// or we could use a type assertion
console.log((x.contents as string).toLowerCase());
```

우리가 진정으로 타입 안정성에 대해 주의를 기울인다면, `contents`의 모든 타입을 위한 다른 `Box` 타입을 발판으로 사용할 수 있다.

```ts
interface NumberBox {
    contents: number;
}

interface StringBox {
    contents: string;
}

interface BooleanBox {
    contents: boolean;
}
```

그러나 그것은 다른 함수들 다 각각 만들거나, 함수를 오버로드 하여 그 각각의 타입에 적용하여야 한다는 것을 의미한다.

```ts
interface NumberBox {
    contents: number;
}

interface StringBox {
    contents: string;
}

interface BooleanBox {
    contents: boolean;
}
//cut
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
    box.contents = newContents;
}
```

이것은 좀 큰 보일러플레이트이며, 기술적으로 우리는 나중에 추가적인 타입과 오버로드가 필요해질 것이다.
이것은 매우 당황스러운 일인데, 우리의 box 타입과 오버로드들은 모두 실제로 같은 효과를 가지기 때문이다.

그대신, 우리는 *제너릭* `Box` 타입을 *타입 파라메터*를 선언하여 만들 수 있다.

```ts
interface Box<T> {
    contents: T;
}
```

우리는 이것을 "`T` 의 `Box`가 `contents`를 갖는데 그 타입이 `T`이다" 라고 읽을 수 있다.
나중에, `Box`를 참조할 때, 우리는 *타입 아규먼트*로서 `T`의 위치에 무언가를 주어야 한다.

```ts
interface Box<T> {
    contents: T;
}
//cut
let box: Box<string>;
```

실제 타입의 템플레이트로서 `Box`를 생각해 보라, 여기서 `T`는 나중에 다른 무슨 타입으로 대치되는 플레이스홀더 이다.
타입스크립트가 `Box<string>`을 보았을 때, `Box<T>`안의 모든 `T`들에 대하여 `string`으로 치환이 일어나고, 결국 `{ contents: string }` 와 비슷한 것이 되어 동작한다.
다른 말로 하면, `Box<string>`과 앞에 나온 `StringBox`는 완전히 동일하게 동작한다.

```ts
interface Box<T> {
    contents: T;
}
interface StringBox {
    contents: string;
}

let boxA: Box<string> = { contents: "hello" };
let boxB: StringBox = { contents: "world" };
```

`T`가 무엇으로든지 치환 가능하기 때문에 `Box` 는 재사용 가능하며, 이것이 의미하는 것은 새로운 타입으로 box가 필요할 경우 우리는 새로운 box 타입을 선언할 필요가 전혀 없다는 것이다(우리가 원한다면 새로 선언도 가능하다).

```ts
interface Box<T> {
    contents: T;
}

interface Apple {
    // ....
}

// Same as '{ contents: Apple }'.
type AppleBox = Box<Apple>;
```

이것은 또한 [generic functions](./More-on-Functions.md#Generic-Functions)을 사용함으로써 오버로딩도 완전히 피할 수 있는 것을 의미한다.

```ts
interface Box<T> {
    contents: T;
}

//cut
function setContents<T>(box: Box<T>, newContents: T) {
    box.contents = newContents;
}
```

이 시점에서, 타입 별칭도 제너릭이 될 수 있다는 것을 상기하는 것이 유익하며 우리는 새로운 `Box<T>` 인터페이스를 다음과 같이 정의하는 것이 가능하다.

```ts
interface Box<T> {
    contents: T;
}
```

이거 대신에 타입 별칭을 사용해서:

```ts
type Box<T> = {
    contents: T;
}
```

실제로, 단순히 객체 타입보다 타입 별칭이 더 많은 것을 기술할 수 있기 때문에, 우리는 때때로 또한 제너릭 헬퍼 타입을 작성할 수 있다.

```ts
type OrNull<T> = T | null;

type OneOrMany<T> = T | T[];

type OneOrManyOrNull<T> = OrNull<OneOrMany<T>>;
     ^?

type Foo = OneOrManyOrNull<string>;
     ^?
```

우리는 빙 돌아서 타입 별칭으로 조금 있다가 돌아올 것이다.

### The `Array` Type


제너릭 객체 타입은 자주 일종의 컨테이너 타입으로 동작하는데 그 이유는 그것이 담고 있는 요소의 타입과 독립적으로 동작하기 때문이다.
자료구조를 위해서는 이런식으로 동작하는 것이 이상적인데 서로 다른 데이터 타입에 대해 재사용이 가능하기 때문이다.

지금까지 우리는 이책 전체를 통해서 하나의 타입과 같은 비슷한 것을 다룬 것이 드러났다: 그것은 `Array`이다.
우리가 `number[]` 나 `string[]` 같은 타입을 쓸 때마다, 그것은 단지 `Array<number>` 나 `Array<string>`을 줄여서 쓴 것에 불과하다.

```ts
function doSomething(value: Array<string>) {
    // ...
}

let myArray: string[] = ["hello", "world"];

// either of these work!
doSomething(myArray);
doSomething(new Array("hello", "world"));
```

앞에서의 `Box` 타입과 매우 비스하게, `Array` 그 자체는 제너릭 타입이다.

```ts
// @noLib: true
interface Number {}
interface String {}
interface Boolean {}
interface Symbol {}
//cut
interface Array<T> {
    /**
     * Gets or sets the length of the array.
     */
    length: number;

    /**
     * Removes the last element from an array and returns it.
     */
    pop(): T | undefined;

    /**
     * Appends new elements to an array, and returns the new length of the array.
     */
    push(...items: T[]): number;

    // ...
}
```

모던 자바스크립트는 또한 제너릭인 `Map<K, V>`, `Set<T>`, 그리고 `Promise<T>`와 같은 자료구조를 제공한다.
이것들이 진실로 의미하는 것은 `Map`, `Set`, 의 `Promise` 어떻게 행동하는 가 이지 그 타입이 아니다, 그들은 어떤 종류의 타입과도 잘 동작한다.

### The `ReadonlyArray` Type

`ReadonlyArray`는 특별한 타입으로서, 값이 바뀌지 않는 배열을 기술한다.

```ts
function doStuff(values: ReadonlyArray<string>) {
    // We can read from 'values'...
    const copy = values.slice();
    console.log(`The first value is ${values[0]}`);

    // ...but we can't mutate 'values'.
    values.push("hello!");
}
```

프로퍼티를 위한 `readonly` 수정자와 같이, 그것은 우리의 의도를 위해 사용 가능한 도구이다.
`ReadonlyArray`를 반환하는 함수를 보면, 그것은 우리가 컨텐츠의 변경을 의미하지 않는다고 말해주고, `ReadonlyArray`를 소비하는 함수를 보면, 그것은 함수가 우리가 넘겨준 배열의 내용을 변경할까봐 걱정할 필요가 없다고 말해준다.

`Array`와 달리, 우리가 사용할 수 있는 `ReadonlyArray` 생성자가 없다.

```ts
new ReadonlyArray("red", "green", "blue");
```

그대신, 일반적 `Array`를 `ReadonlyArray`에 대입할 수 있다.

```ts
let roArray: ReadonlyArray<string> = ["red", "green", "blue"];
```

타입스크립트가 `Foo[]`로써 `Array<Foo>`의 축약 문법을 제공하듯이, `readonly Foo[]`는 `ReadonlyArray<Foo>`의 축약 문법이다.

```ts
function doStuff(values: readonly string[]) {
                         ^^^^^^^^^^^^^^^^^
    // We can read from 'values'...
    const copy = values.slice();
    console.log(`The first value is ${values[0]}`);

    // ...but we can't mutate 'values'.
    values.push("hello!");
}
```

유의해야 할 마지막 이야기는 `readonly` 프로퍼티 수정자와 달리, 정규 `Array`와 `ReadonlyArray`는 양방향 대입이 불가능 하다는 점이다.

```ts
let x: readonly string[] = [];
let y: string[] = [];

x = y;
y = x;
```

### Tuple Types

*튜플 타입*은 `Array` 타입의 또 다른 한 종류 로서, 엘리먼트가 몇개인지, 특정 위치에 무슨 타입을 가지고 있는지 정확히 아는 경우 이다.

```ts
type StringNumberPair = [string, number];
                        ^^^^^^^^^^^^^^^^
```

여기서, `StringNumberPair`는 `string` 과 `number`의 튜플 타입이다.
`ReadonlyArray` 처럼, 런타임에 그것은 아무련 대표성이 없지만, 타입스크립트에게는 의미가 크다.
타입 시스템에게, `StringNumberPair`는 `0`번 인덱스에 `string`을 `1`번 인덱스에 `number`를 담고 있는 배열을 기술하고 있다.

```ts
function doSomething(pair: [string, number]) {
    const a = pair[0];
          ^?
    const b = pair[1];
          ^?
    // ...
}

doSomething(["hello", 42]);
```

우리가 엘리먼트의 숫자를 넘어서서 인덱스를 접근하려고 하면 우리는 에러를 얻게 될 것이다.

```ts
function doSomething(pair: [string, number]) {
    // ...

    const c = pair[2];
}
```

우리는 또한 [destructure tuples](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring)를 할 수 있는데 이것은 자바스크립트의 배열 디스트럭쳐링을 사용한 것이다.

```ts
function doSomething(stringHash: [string, number]) {
    const [inputString, hash] = stringHash;

    console.log(inputString);
                ^?

    console.log(hash);
                ^?
}
```

<aside>
튜플 타입은 심하게 컨벤션기반인 API에 유용한데 각 엘리먼트의 의미가 명백한 경우이다.
이것은 디스트럭쳐 할때 우리가 원하는 무슨 이름이든 부여할 수 있는 융툥성을 준다.
위 예제에서, `0` 과 `1`엘리먼트의 이름을 우리가 원하는 것으로 부여할 수 있었다.

하지만, 모든 사용자가 명백한 것이 무엇인지 동일한 생각을 가지고 있지 않으므로, 당신의 API에 서술적인 프로퍼티 이름을 주는 것이 더 나은지 재고해 보는 것은 가치가 있다.
</aside>

길이 검사외에, 간단한 튜플 타입은 특정 인덱스에 프로퍼티를 선언하고 숫자 리터럴 타입으로 길이를 선언하는 `Array`의 한 버젼과 같다.

```ts
interface StringNumberPair {
    // specialized properties
    length: 2;
    0: string;
    1: number;

    // Other 'Array<string | number>' members...
    slice(start?: number, end?: number): Array<string | number>;
}
```

당신이 관심을 가질만한 다른 것은 튜플은 물음표(`?`를 엘리먼트 타입뒤에 추가)를 써줌으로써, 생략가능한 프로퍼티를 가질 수 있다는 것이다.
생략 가능한 튜플 엘리먼트는 오직 끝에만 올 수 있고, `length`의 타입에 영향을 준다.

```ts
type Either2dOr3d = [number, number, number?];

function setCoordinate(coord: Either2dOr3d) {
    const [x, y, z] = coord;
                 ^?

    console.log(`Provided coordinates had ${coord.length} dimensions`);
                                                  ^?
}
```

튜플은 나머지 엘리먼트를 가질 수도 있는데, 나머지 엘리먼트는 배역/튜플 타입인 경우 이다.

```ts
type StringNumberBooleans = [string, number, ...boolean[]];
```

`StringNumberBooleans`는 처음의 두 엘리먼트가 `string` 과 `number`이고, 다수의 `boolean`들을 가지고 있다고 기술하고 있다.
나머지 엘리먼트를 가진 튜플은 `length`가 설정되지 않는데 - 그것은 단지 처음의 알려진 엘리먼트만 가지고 있다.

```ts
type StringNumberBooleans = [string, number, ...boolean[]];
//cut
const a: StringNumberBooleans = ["hello", 1];
const b: StringNumberBooleans = ["beautiful", 2, true];
const c: StringNumberBooleans = ["world", 3, true, false, true, false, true];
```

왜 생략가능 또는 나머지 엘리먼트가 유용한가?
그것은 타입스크립트가 해당하는 튜플 파라메터 리스트를 허용하기 때문이다.
튜플은 [rest parameters and arguments](./More-on-Functions.md#rest-parameters-and-arguments)에서 사용될 수 있고, 다음을 보자:

```ts
function foo(...args: [string, number, ...boolean[]]) {
    var [x, y, ...z] = args;
    // ...
}
```

는 기본적으로 다음과 동일 하다

```ts
function foo(x: string, y: number, ...z: boolean[]) {
    // ...
}
```

나머지 파라메터로 변하는 수의 아규먼트를 받을 때 이것은 편리하며, 당신은 최소 엘리먼트의 숫자만 필요하며 중간 변수를 도입하지 않아도 되는 장점이 있다.

<!--
TODO do we need this example?

For example, imagine we need to write a function that adds up `number`s based on arguments that get passed in.

```ts
function sum(...args: number[]) {
    // ...
}
```

We might feel like it makes little sense to take any fewer than 2 elements, so we want to require callers to provide at least 2 arguments.
A first attempt might be

```ts
function foo(a: number, b: number, ...args: number[]) {
    args.unshift(a, b);

    let result = 0;
    for (const value of args) {
        result += value;
    }
    return result;
}
```

-->

### `readonly` Tuple Types

튜플 타입에 대한 마지막 주의점은 - 튜플 타입은 `readonly` 변이를 가진다, 그리고 `readonly` 수정자를 튜플앞에 고정적으로 정의 할 수도 있다 - 배열 축약 표현 처럼. 

```ts
function doSomething(pair: readonly [string, number]) {
                           ^^^^^^^^^^^^^^^^^^^^^^^^^
    // ...
}
```

당신이 기대하는 것처럼, 타입스크립트에서는 `readonly` 튜플의 어떤 엘리먼트도 갱신 될 수 없다.

```ts
function doSomething(pair: readonly [string, number]) {
    pair[0] = "hello!";
}
```

튜플은 대부분의 코드에서 생성된 후 갱신되지 않는 경향이 있다, 따라서 `readonly` 튜플을 사용하는 것은 가능한 좋은 기본 스타일 이다.
`const` assertions을 가진 배열 리터럴은 `readonly` 튜플 타입으로 추정된다는 것은 중요하다.

```ts
let point = [3, 4] as const;

function distanceFromOrigin([x, y]: [number, number]) {
    return Math.sqrt(x ** 2 + y ** 2);
}

distanceFromOrigin(point);
```

여기서, `distanceFromOrigin`는 절대로 튜플의 엘리먼트를 변경하지 않으나 갱신가능한 튜플을 원하고 있다.
`point`의 타입은 `readonly [3, 4]`로 추정되었으므로, `[number, number]`와 호환성이 없는데 그 이유는 `point`의 엘리먼트가 갱신되지 않을 거라는 것을 보장할 수 없기 때문이다.

## Other Kinds of Object Members

Most of the declarations in object types

### Method Syntax

### Call Signatures

### Construct Signatures

### Index Signatures
