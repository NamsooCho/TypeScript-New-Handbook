# Classes

>> [Background reading: Classes (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

타입스크립트는 ES2015에서 도입된 `class` 키워드를 완벽히 지원한다.
자바스크립트의 다른 특성과 같이, 타입스크립트는 타입명시와 여타 문법들을 추가해서 당신이 클래스와 다른 타입간의 관계를 명시할 수 있도록 허용한다.

__toc__

## Class Members

여기에 가장 기본적 클래스가 있다 - 빈 클래스:

```ts
class Point {

}
```

이 클래스는 아직 그다지 유용하지 않다, 따라서 몇몇 멤버를 추가해보자.

### Fields

필드의 선언은 클래스에 퍼블릭한 프로퍼티를 생성한다:

```ts
// @strictPropertyInitialization: false
class Point {
    x: number;
    y: number;
}

const pt = new Point();
pt.x = 0;
pt.y = 0;
```

다른 곳에서도 마찬가지지만, 타입명시는 여기서도 선택적이다, 그러나 명시 되지 않을 경우 암시적 `any` 타입이 된다.

필드는 *이니셜라이저*를 가질 수 있다; 이것들은 클래스가 생성될 때 자동으로 실행 된다.

```ts
class Point {
    x = 0;
    y = 0;
}

const pt = new Point();
// Prints 0, 0
console.log(`${pt.x}, ${pt.y}`);
```

`const`, `let`, 그리고 `var` 처럼, 클래스 프로퍼티의 이니셜라이져는 타입을 추정하는데 사용된다:

```ts
class Point {
    x = 0;
    y = 0;
}
//cut
const pt = new Point();
pt.x = "0";
```

#### `--strictPropertyInitialization`

`strictPropertyInitialization` 세팅은 클래스의 필드가 생성자에서 초기화되어야 하는지 여부를 설정한다.

```ts
class BadGreeter {
  name: string;
}
```

```ts
class GoodGreeter {
  name: string;

  constructor() {
    this.name = "hello";
  }
}
```

필드는 *생성자 내부에서* 초기화될 필요가 있다는 것을 유의하라.
타입스크립트는 컨스트럭터가 호출하는 메소드가 초기화 작업을 하는지 분석하지 않는다, 왜냐하면 상속된 클래스가 이러한 메소드를 오버라이드 하면 멤버의 초기화가 안되는 경우가 발생하기 때문이다.

만약 생성자가 아닌 방법을 통해서 필드를 확실히 초기화 할 의도가 있다면(예를 들어, 외부 라이브러리가 당신을 위해 클래스의 일부를 채우는 경우), *학실한 대입 확인 연산자*인 `!`를 사용하면 된다:

```ts
class OKGreeter {
  // 초기화 되지 않았지만 에러가 아님.
  name!: string;
}
```

### `readonly` {#readonly-class-properties}

필드들은 `readonly` 수정자를 앞에 둘 수 있다.
이것은 생성자 외부에서 필드에 값이 대입되는 것을 막아준다.

```ts
class Greeter {
  readonly name: string = "world";

  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }

  err() {
    this.name = "not ok";
  }
}
const g = new Greeter();
g.name = "also not ok";
```

### Constructors

[Background Reading: Constructor (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor)

클래스 생성자는 함수와 매우 유사하다.
파라메터를 타입명시와 함께 추가할 수도 있고, 디폰트 값을 줄수도 있고, 오버로딩도 추가할 수 있다:

```ts
class Point {
  x: number;
  y: number;

  // Normal signature with defaults
  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
}
```

```ts
class Point {
  // Overloads
  constructor(x: number, y: string);
  constructor(s: string);
  constructor(xs: any, y?: any) {
    // TBD
  }
}
```

함수 시그니쳐와 클래스 생성자 시그니쳐 사이에는 약간 차이가 있다:

* 생성자는 타입 파라메터를 가질 수 없다 - 이것들은 바깥의 클래스 선언에 포함된 것이다, 이것은 나중에 다룰 것이다.
* 생성자는 반환 값의 타입을 가질 수 없다 - 항상 클래스 객체가 반환 된다.

#### Super Calls

자바스크립트에서아 같이, 만약 베이스 클래스가 있다면, 생성자 내부에서 `this.` 멤버를 사용하기 전에 반드시 `super();`를 호출하여야 한다.

```ts
class Base {
  k = 4;
}

class Derived extends Base {
  constructor() {
    // ES5에서는 틀린 값을 출력하고; ES6에서는 익셉션을 던진다.
    console.log(this.k);
    super();
  }
}
```

자바스크립트에서 `super`를 호출하는 것을 잊어버리는 것은 쉬운 실수 이나, 타입스크립트에서는 이것이 필요하다고 알려준다.

### Methods

>> [Background Reading: Method definitions (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions)

클래스의 함수 프로퍼티는 *메소드*라고 불리운다.
메소드는 함수와 생성자 처럼 타입명시를 사용할 수 있다.

```ts
class Point {
  x = 10;
  y = 10;

  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
```

표준 타입명시와 외에 타입스크립트는ㄴ 메소드에 추가적인 어떤것도 더하지 않았다.

메소드 몸체 내부에서 다른 메소드를 접근하려면 여전히 `this.`를 사용해야 한다는 것을 유념하라.
메소드 몸체 내부에서 식별해내지 못한 이름은 외부의 둘러싼 범위내의 어떤 것으로 추정된다:

```ts
let x: number = 0;

class C {
  x: string = "hello";

  m() {
    // 이것은 1번 라인의 'x' 로 추정된다, 클래스 프로퍼티가 아니다.
    x = "world";
  }
}
```

### Getters / Setters

클래스는 또한 *접근자*를 가질 수 있다:

```ts
class C {
  _length = 0;
  get length() {
    return this._length;
  }
  set length(value) {
    this._length = value;
  }
}
```

> get/set 쌍을 가진 필드가 추가적인 로직을 가지지 않는 경우는 자바스크립트에서 매우 드믈게 유용하다는 점을 유념하라.
> get/set 연산 중에 추가적인 로직이 필요 없는 경우는 그냥 퍼블릭으로 오픈하는게 좋다.

타입스크립트는 접근자에 대해 특별한 추정규칙을 가지고 있다.

* 만약 `set` 이 없다면, 그 프로퍼티는 자동적으로 `readonly`가 된다.
* setter의 파라메터 타입은 getter의 반한 값의 타입으로 부터 추정된다.
* setter 의 파라메터에 타입명시가 주어진 경우, 이 것은 반드시 getter의 반한값의 타입과 일치해야 한다.
* getter와 setter의 [[Member Visibility]]는 반드시 일치해야 한다. 즉 하나는 public 이고 다른 것은 private와 같이 하면 안된다.

접근자가 getter와 setter가 서로 다른 타입을 가지는 것은 불가능 하다.

setter없이 getter만 있는 경우, 그 필드는 자동적으로 `readonly`가 된다.

### Index Signatures {#class-index-signatures}

클래스는 인덱스 시그니쳐를 선언할 수 있다; 이것은 다른 객체 타입에서의 [[Index Signatures]]와 동일한 일을 한다.

```ts
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);
  check(s: string) {
    return this[s] as boolean;
  }
}
```

인덱스 시그니쳐 타입이 메소드의 타입을 또한 캡쳐해아 하므로, 이러한 타입을 유용하게 사용하기는 쉽지 않다.
일반적으로 인덱스 데이터는 다른 곳에 저장하고 클래스 인스턴스 자체에 저장하지 않는 것이 좋다.

## Class Heritage

객체지향 특성을 갖는 다른 언어와 같이, 자바스크립트에서의 클래스는 베이스 클래스로부터 상속을 받을 수 있다.

### `implements` Clauses

사용자는 어떤 클래스가 특정 `interface`를 만족하는지 검사하기 위하여 `implements` 구문을 사용할 수 있다.
어떤 클래스가 정확히 그 인터페이스를 구현하지 않은 경우 에러가 출력된다.

```ts
interface Pingable {
  ping(): void;
}

class Sonar implements Pingable {
  ping() {
    console.log('ping!');
  }
}

class Ball implements Pingable {
  pong() {
    console.log('pong!');
  }
}
```

클래스는 `class C implements A, B {` 처럼 여러개의 인터페이스를 구현 할 수 있다.

#### Cautions

`implements` 구문이 단지 어떤 클래스가 인터페이스 타입인가 여부를 검사하는데 사용될 뿐이라는 것을 이해하는 것이 중요하다.
이것은 클래스의 타입이나 메소드를 *전혀* 변경하지 않는다.
에러의 일반적인 원인은 `implements` 구문이 클래스의 타입을 변경하는 경우이다 - 이것은 허용되지 않는다!

```ts
// @noImplicitAny: false
interface Checkable {
  check(name: string): boolean;
}
class NameChecker implements Checkable {
  check(s) {
    // 이 문장은 에러가 아님을 유의하라.
    return s.toLowercse() === "ok";
           ^?
  }
}
```

이 예제에서, 우리는 아마도 `s`의 타입이 `check`의 파라메터인 `name: string`의 영향을 받았을 거라고 기대할 것이다.
그렇지 않다 - `implements` 구문은 클래스 바디가 어떻게 검사되거나 타입추정 되는 가를 변경하지 않는다.

비슷하게, 생략가능한 프로퍼티를 가진 인터페이스를 구현하는 것은 그 프로퍼티를 생성하는 것이 아니다:

```ts
interface A {
  x: number;
  y?: number;
}
class C implements A {
  x = 0;
}
const c = new C();
c.y = 10;
```

### `extends` Clauses

>> [Background Reading: extends keyword (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends)

클래스는 베이스 클래스로부터 `extend` 될 수 있다.
상속받은 클래스는 베이스 클래스의 모든 프로퍼티와 메소드들을 상속 받는다, 또한 추가적인 멤버를 정의할 수 있다.

```ts
class Animal {
  move() {
    console.log("Moving along!");
  }
}

class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log("woof!");
    }
  }
}

const d = new Dog();
// Base class method
d.move();
// Derived class method
d.woof(3);
```

#### Overriding Methods

>> [Background reading: super keyword (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super)

상속 받는 클래스는 베이스 클래스의 필드와 프로퍼티를 오버라이드 할 수 있다.
당신은 `super.`를 사용하여 베이스 클래스의 메소드를 접근할 수 있다.
자바스크립트 클래스는 간단한 loopup 객체이므로 "super field" 라는 특별한 표기가 없다는 점을 유의하라.

타입스크립트는 후손 클래스가 항상 베이스 클래스의 서브타입이 되도록 강제한다.

예를 들어, 이 것은 메소드를 오버라이드 하는 적절한 방법이다:

```ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}

const d = new Derived();
d.greet();
d.greet("reader");
```

후손 클래스가 베이스 클래스의 제약 사항을 따르는 것은 매우 중요하다.
후손 클래스의 객체를 베이스 클래스 리퍼런스로 접근하는 방식이 매우 흔하고 적절한 방법이라는 것을 기억하라.

```ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}
declare const d: Base;
//cut
// Alias the derived instance through a base class reference
const b: Base = d;
// No problem
b.greet();
```

`Derived`가 `Base`'의 제약사항을 따르지 않으면 어떻게 될까?

```ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  // 파라메터가 필요하게 만들었다.
  greet(name: string) {
    console.log(`Hello, ${name.toUpperCase()}`);
  }
}
```

이러한 에러에도 불구하고 우리가 컴파일을 강행 한다면, 이 예제는 크래쉬 될 것이다:

```ts
declare class Base  { greet(): void };
declare class Derived extends Base { }
//cut
const b: Base = new Derived();
// "name" 이 정의 되지 않아서 크래쉬 된다.
b.greet();
```

#### Initialization Order

자바스크립트 클래스가 초기화 되는 순서가 때때로 놀랍게 한다.
다음의 코드를 고려해 보자:

```ts
class Base {
  name = "base";
  constructor() {
    console.log("My name is " + this.name);
  }
}

class Derived extends Base {
  name = "derived";
}

// "base"를 출력한다, "derived"를 출력하지 않는다.
const d = new Derived();
```

무슨 일이 벌어진 것일까?

자바스크립트에 정의된 클래스의 초기화 순서는:
 * 베이스 클래스 필드가 초기화 된다.
 * 베이스 클래스 생성자가 실행된다.
 * 후손 클래스 필드가 초기화 된다.
 * 후손 클래스 생성자가 실행 된다.

이것은 베이스 클래스의 생성자가 자신의 `name` 값을 생성하는 동안 바라본 것이다, 왜냐면 후손 객체는 아직 초기화가 시작되지 않았기 때문이다.

#### Inheriting Built-in Types

> 주의 : 기본 포함된 타입들 `Array`, `Error`, `Map` 와 같은 것을 상속받을 계획이 없다면 이 섹션을 건너 뛰어도 좋다.

ES2015에서, 생성자는 어떤 `super(...)`를 호출하건 `this` 값을 대치하는 객체를 암묵적으로 반환 한다.
이것은 생성자가 `super(...)`의 잠재적 반환값을 캡쳐하고 `this`을 대치하는데 필요하다.

결과적으로, `Error`, `Array`, 그리고 다른 몇 가지를 상속하는 것은 더이상 원하는 대로 동작하지 않는다.
이것은 `Error`, `Array` 의 생성자 함수는 ECMAScript 6's `new.target`을 사용하여 프로토타입 체인을 변경하려 하기 때문이다;
하지만, ECMAScript 5에서 생성자를 호출하는 경우 `new.target`의 값을 확정하는 방법이 없다.
다른 레벨다운 컴파일러들은 일반적으로 동일한 기본적 한계를 가지고 있다.

다음과 같은 상속을 보자:

```ts
class FooError extends Error {
    constructor(m: string) {
        super(m);
    }
    sayHello() {
        return "hello " + this.message;
    }
}
```

아마도 당신은 다음의 내용을 발견할 것이다:

* 이 상속된 클래스의 생성자가 반한될 때 메소드는 아마도 `undefined`가 될 것이다 따라서 `sayHello`를 호출하는 것은 에러로 끝날 것이다.
* 하위 클래스의 인스턴스와 하위 클래스의 인스턴스간의 `instanceof`는 깨질 것이다, 따라서 `(new FooError()) instanceof FooError` 는 `false`를 반환 할 것이다.

결과적으로, 당신은 수작업으로 `super(...)` 호출 후에 즉시 프로토타입을 수정할 수 있다.

```ts
class FooError extends Error {
    constructor(m: string) {
        super(m);

        // 명시적으로 프로토타입을 수정한다.
        Object.setPrototypeOf(this, FooError.prototype);
    }

    sayHello() {
        return "hello " + this.message;
    }
}
```

하지만, `FooError`의 상속 클래스는 또다시 수작업으로 프로토타입을 수정해야 한다.
[`Object.setPrototypeOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)를 지원하지 않는 런타임 환경에서는 그대신 [`__proto__`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)를 사용할 수 있다.

불행히도, [이러한 수정 작업은 Internet Explorer 10 그리고 이전버젼 에서는 동작하지 않는다](https://msdn.microsoft.com/en-us/library/s4esdbwz(v=vs.94).aspx).
사용자는 수작업으로 인스턴스 자체에 프로토타입으로부터 메소드를 복사할 수 있다(즉, `FooError.prototype` onto `this`), 그러나 프로토타입 체인 그 자체는 픽스할 수 없다.

## Member Visibility

사용자는 타입스크립트를 사용하여 클래스의 외부에서 특정 메소드나 프로퍼티들이 보이게(안 보이게) 콘트롤 할 수 있다.

### `public`

클래스 멤버의 기본 가시성은 `public` 이다.
`public` 멤버는 어디에서든 접근이 가능하다.

```ts
class Greeter {
  public greet() {
    console.log("hi!");
  }
}
const g = new Greeter();
g.greet();
```

`public`은 이미 기본 가시성 수정자 이므로 클래스 멤버에 멩시적으로 표기할 *필요*가 없다, 그러나 가독성과 스타일링을 때문에 표기하는 것이 좋다.

### `protected`

`protected` 멤버는 그것이 선언된 클래스의 하위 클레스에서 보인다.

```ts
class Greeter {
  public greet() {
    console.log("Hello, " + this.getName());
  }
  protected getName() {
    return "hi";
  }
}

class SpecialGreeter extends Greeter {
  public howdy() {
    // OK to access protected member here
    console.log("Howdy, " + this.getName());
                            ^^^^^^^^^^^^^^
  }
}
const g = new SpecialGreeter();
g.greet(); // OK
g.getName();
```

#### Exposure of `protected` members

후손 클래스는 베이스 클래스의 제약 조건을 따른다, 그러나 더욱 일반적인 타입과 능력들을 노출 할 수 있다.
이것은 `protected` 맴버를 `public`으로 변경하는 것도 포함한다.

```ts
class Base {
  protected m = 10;
}
class Derived extends Base {
  // No modifier, so default is 'public'
  m = 15;
}
const d = new Derived();
console.log(d.m); // OK
```

상속받은 클래스는 이미 `m`을 자유롭게 읽고 쓸수 있었다는 것을 유의 하라, 따라서 이것은 이 상황의 "보안성"을 의미있게 변경한 것은 아니다.
여기서 중요한 점은, 상속받은 클레스에서 `protected`를 무심코 사용하지 않도록 조심해야 한다는 것이다.

#### Cross-hierarchy `protected` access

다른 객체지향 언어에서는 `protected` 멤버에 대한 베이스 클래스 리퍼런스를 통한 접근이 적정한가에 대하여 의견이 일치하지 않고 있다:

```ts
class Base {
    protected x: number = 1;
}
class Derived1 extends Base {
    protected x: number = 5;
}
class Derived2 extends Base {
    f1(other: Derived2) {
        other.x = 10;
    }
    f2(other: Base) {
        other.x = 10;
    }
}
```

에를 들어 자바에서는 이러한 것을 적정하다고 본다.
반면에 C# 과 C++에서는 부적절 하다고 본다. 

이 부분에서 타입스크립트는 C# 과 C++ 을 편든다, `Derived2`에서의 `x`에 대한 접근은 `Derived2`의 후손 클래스로부터만 적정하고 `Derived1`는 `Derived1`의 후손 클래스가 아니기 때문이다.
더구나, `Derived2` 리퍼런스를 통한 `x`의 접근은 부적절하다(확실히 되어야 할거 같은데), 따라서 베이스 클래스의 리퍼런스를 통한 접근은 이상황을 절대로 개선할 수 없다.

[Why Can’t I Access A Protected Member From A Derived Class?](https://blogs.msdn.microsoft.com/ericlippert/2005/11/09/why-cant-i-access-a-protected-member-from-a-derived-class/) 은 C#이 택한 방식의 이유를 잘 설명해 주고 있다.

### `private`

`private` 는 `protected`와 비슷하다, 그러나 서브클래스에서 조차 멤버에 접근 할 수 없게 해준다.

```ts
class Base {
  private x = 0;
}
const b = new Base();
// Can't access from outside the class
console.log(b.x);
```

```ts
class Base {
  private x = 0;
}
//cut
class Derived extends Base {
  showX() {
    // Can't access in subclasses
    console.log(this.x);
  }
}
```

`private` 멤버는 상속된 클래스에서 보이지 않기 때문에 상속된 클래스에서 그 가시성을 변경하는 것도 불가능 하다.


```ts
class Base {
  private x = 0;
}
class Derived extends Base {
  x = 1;
}
```

#### Cross-instance `private` access

객체지향 언어들은 또한 같은 클래스의 인스턴스들 간의 `private` 멤버에 대한 접근을 허용하는 것에 대해서도 의견이 불일치 하고 있다.
Java, C#, C++, Swift, 그리고 PHP는 이것을 허용하고 있고, Ruby는 불허하고 있다.

타입스크립트는 이것을 허용한다:

```ts
class A {
  private x = 10;

  public sameAs(other: A) {
    // No error
    return other.x === this.x;
  }
}
```

#### Caveats {#private-and-runtime-privacy}

타입스크립트 타입시스템의 여러 특성들 처럼, `private` 과 `protected` 는 오직 체크시에만 의미가 있다.
이것은 무슨 의미이냐면 `in`과 같은 자바스크립트 런타임 구현체 또는 간단한 프로퍼티 살펴보기는 여전히 `private` 또는 `protected` 멤버들은 여전히 접근 가능하다.

```ts
class MySafe {
  private secretKey = 12345;
}
```

```js
// In a JavaScript file...
const s = new MySafe();
// Will print 12345
console.log(s.secretKey);
```

만약 클래스의 값등이 잘못된 행동을 하는 것을 방지하고자 한다면, 강력한 런타임 보호를 하는 메카니즘을 사용하여야 한다, 이것은 클로져나 weak maps, 또는 [[private fields]] 같은 것들 이다.

## Static Members

>> [Background Reading: Static Members (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static)

클래스는 `static` 멤버를 가질 수 있다.
이런 멤버는 클래스의 특정 인스턴스와만 연관될 수는 없다.
그것은 클래스 객체 그 차체에서 접근할 수 있다.

```ts
class MyClass {
  static x = 0;
  static printX() {
    console.log(MyClass.x);
  }
}
console.log(MyClass.x);
MyClass.printX();
```

스태틱 멤버는 또한 `public`, `protected`, 그리고 `private` 가시성 수정자를 가질 수 있다.

```ts
class MyClass {
  private static x = 0;
}
console.log(MyClass.x);
```

스태틱 멤버는 또한 상속 될 수 있다:

```ts
class Base {
  static getGreeting() {
    return "Hello world";
  }
}
class Derived extends Base {
  myGreeting = Derived.getGreeting();
}
```

### Special Static Names

`Function` 프로토타입으로 부터 프로퍼티를 덮어 쓰는 것은 안전하지도 가능하지도 않다.
클래스 자체가 함수이고 `new`로 실행가능하므로 특정 `static` 이름은 사용하면 안된다.
`name`, `length`, 그리고 `call`과 같은 함수 프로퍼티는 `static` 멤버로 정의되어서는 안된다.

```ts
class S {
  static name = "S!";
}
```

### Why No Static Classes?

타입스크립트(그리고 자바스크립트)는 C# 과 Java가 하는 것처럼 `static class` 요소를 갖고 있지 않다.

이러한 요소들은 *오직* 데이터와 함수가 모두 클래스 내부에만 있는 언어에서만 존재한다; 타입스크립트에서는 그러한 제약이 없기 때문에 그러한 요소를 가질 필요가 없다.
오직 하나의 인스턴스만 갖는 클래스는 일반적으로 자바스크립트/타입스크립트에서는 일반적 *객체*로 표현이 가능하다.

예를 들어서, 정식의 객체(또는 최상의 레벨 함수인 경우라도)는 "static class"의 역할을 이미 잘 하고 있기 때문에 타입스크립트에서는 이러한 요소가 필요가 없다:

```ts
// Unnecessary "static" class
class MyStaticClass {
    static doSomething() {

    }
}

// Preferred (alternative 1)
function doSomething() {

}

// Preferred (alternative 2)
const MyHelperObject = {
    dosomething() {

    }
}
```

## Generic Classes

인터페이스 처럼 클래스는 제너릭으로 사용 가능 하다.
제너릭 클래스가 `new`로 생성될 경우, 그것의 타입 파라메터는 함수 호출의 경우와 동일하게 추정된다.

```ts
class Box<T> {
  contents: T;
  constructor(value: T) {
    this.contents = value;
  }
}

const b = new Box("hello!");
      ^?
```

클래스는 인터페이스의 경우와 동일하게 제너릭의 제한과 기본값을 사용할 수 있다.

### Type Parameters in Static Members

이 코드는 적절하지도 않으며 명백하지도 않다. 왜?:

```ts
class Box<T> {
  static defaultValue: T;
}
```

타입들은 자바스크립트로 변환될때 모두 삭제 된다는 것을 기억하라!
런타임에는 오직 *하나*의 `Box.defaultValue` 프로퍼티 공간만 가지게 될 것이다.
이것의 의미는 `Box<string>.defaultValue`로 세팅하는 것은(만약 가능하다면) `Box<number>.defaultValue`로 세팅된 값도 바꾸게 된다 - 좋지 않다.
제너릭 클래스의 스태틱 멤버는 클래스의 타입 파라메터를 참조해서는 안된다.

## `this` at Runtime in Classes {#runtime-this-in-classes}

>> [Background Reading: `this` keyword (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)

타입스크립트는 자바스크립트의 실행시 행동을 변경하지 않는다는 것을 기억하는 것은 중요하다. 그리고 자바스크립트는 실행시 행동이 약간 특별한 것으로 유명하다.

자바스크립트의 `this`를 다루는 것은 실로 일반적이지 않다.

```ts
class MyClass {
  name = "MyClass";
  getName() {
    return this.name;
  }
}
const c = new MyClass();
const obj = {
  name: "obj",
  getName: c.getName
};

// Prints "obj", not "MyClass"
console.log(obj.getName());
```

짧게 이야기 하자면, 기본적으로, 함수내의 `this` 의 값은 그 함수가 *어떻게 호출되었는가*에 따라 다르다.
이 예제에서, 함수는 `obj` 리퍼런스를 통해서 호출되었으므로, `this` 는 `obj`가 되며, 클래스 객체가 되지 않는다.

이것은 당신이 원하던 결과를 내놓지 않는다.
타입스크립트는 이런 종류의 에러를 치유하고 방지하는 몇가지 방법을 제공한다.

### Arrow Functions

>> [Background Reading: Arrow functions (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

만약 자주 이러한 `this` 관련 문제를 겪는는 함수를 갖고 있다면, 애로우 함수를 사용하는 것이 의미가 있다:

```ts
class MyClass {
  name = "MyClass";
  getName = () => {
    return this.name;
  }
}
const c = new MyClass();
const g = c.getName;
// Prints "MyClass" instead of crashing
console.log(g());
```

이것은 몇가지 트레이트-오프가 있다:
 * `this` 값은 실행시 정확한 값을 갖는 것이 보장된다, 타입스크립트가 체크하지 않는 경우라 하더라도 그렇다.
 * 이 방법은 메모리를 더 사용할 것이다, 왜냐하면 이 방식으로 정의된 각 함수는 자신의 복사본을 각 클래스마다 갖게되기 때문이다.
 * 상속된 클래스에서 `super.getName`를 사용할 수 없다, 왜냐하면 베이스 클래스로부터 프로퍼티 체인을 가져오는 입구가 없기 때문이다.

### `this` parameters

메소드나 함수에서, 타압스크립트에서는 초기화 파라메터에 `this`라는 이름은 특별한 의미가 있다.
이러한 파라메터는 컴파일 하는 동안에 지워진다.

```ts
type SomeType = any
//cut
// TypeScript input with 'this' parameter
function fn(this: SomeType, x: number) {
  /* ... */
}
```
```js
// JavaScript output
function fn(x) {
  /* ... */
}
```

타입스크립트는 `this` 파라메터를 가진 함수 호출이 올바른 환경에서 호출되었는지 체크한다.
애로우 함수의 사용 대신에 우리는 `this` 파라메터를 메소드 정의에 추가하여 정적으로 그 메소드가 정확히 호출되었는지 확실히 할 수 있다:

```ts
class MyClass {
  name = "MyClass";
  getName(this: MyClass) {
    return this.name;
  }
}
const c = new MyClass();
// OK
c.getName();

// Error, would crash
const g = c.getName;
console.log(g());
```

이 방법은 애로우 함수와는 정반대의 트레이드-오프를 가진다:
 * 자바스크립트 호출자는 여전히 잘못된 호출을 깨닫지 못한 채 이 클래스 메소드를 사용할 수 있다
 * 클래스 정의당 오직 하나의 함수만 할당된다, 클래스 인스턴스당 하나가 아니다.
 * 베이스 메소드 정의는 여전히 `super.`를 통하여 접근이 가능하다.

## `this` Types

클레스에서, 특별한 타입인 `this`는 *동적으로* 현재 클래스의 타입을 지칭한다.
이것이 어떻게 유용한지 다음을 보자:

```ts
class Box {
  contents: string = "";
  set(value: string) {
   ^?
    this.contents = value;
    return this;
  }
}
```

여기서, 타입스크립트는 `set`의 반환 타입을 `Box`가 아닌 `this`가 되도록 한다.
이제 `Box`의 상속 클래스를 만들어 보자:

```ts
class Box {
  contents: string = "";
  set(value: string) {
    this.contents = value;
    return this;
  }
}
//cut
class ClearableBox extends Box {
  clear() {
    this.contents = "";
  }
}

const a = new ClearableBox();
const b = a.set("hello");
      ^?
```

`this`는 타입명시에서 파라메터 타입에도 사용 가능 하다:

```ts
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}
```

이것은 `other: Box`라고 쓴 것과는 다르다 -- 상속된 클래스를 가진 경우라면, `sameAs` 메소드는 동일한 상속 클래스의 객체만을 받아들일 것이다:

```ts
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}

class DerivedBox extends Box  {
  otherContent: string = "?";
}

const base = new Box();
const derived = new DerivedBox();
derived.sameAs(base);
```

## Parameter Properties

타입스크립트는 생성자 파라메터를 동일한 이름과 값으로 클래스 프로퍼티가 되도록 해주는 특별한 문법을 제공한다.
이것은 *파라메터 프로퍼티*라고 불리우며, `public`, `private`, `protected`, or `readonly`와 같은 가시성 수정자와 같이 생성자 아규먼트로 생성된다.
결과로 만들어지는 필드는 이러한 수정자의 특성을 가진다:

```ts
class A {
  constructor (public readonly x: number, protected y: number, private z: number) {
    // No body necessary
  }
}
const a = new A(1, 2, 3);
console.log(a.x);
              ^?
console.log(a.z);
```

## Class Expressions

>> [Background reading: Class expressions (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/class)

클래스 식은 클래스 선언과 매우 유사하다.
실질적인 차이는 클래스 식은 이름을 가질 필요가 없다는 것인데, 우리는 연관된 이름을 사용하여 그 클래스를 접근 하기 때문이다.

```ts
const someClass = class<T> {
  content: T;
  constructor(value: T) {
    this.content = value;
  }
}

const m = new someClass("Hello, world");
      ^?
```

## `abstract` Classes and Members

클래스, 메소드, 필드는 *추상적* 일 수가 있다.

어떤 *추상 메소드* 또는 *추상 필드*는 아직 구현이 제공되지 않은 것을 말한다.
이러한 멤버들은 *추상 클래스*안에 존재하여야 하며, 이 추상 클래스는 직접적으로 객체를 만드는데 사용되어서는 안된다.

추상 클래스의 목적은 베이스 클래스로서 존재하면서 상속되는 클래스가 구체적 구현을 하도록 하는데 있다.
어떤 클래스가 추상 멤버가 하나도 없다면 그것은 *견고*하다고 말한다.
Classes, methods, and fields in TypeScript may be *abstract*.

예제를 보자:

```ts
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}

const b = new Base();
```

우리는 `Base` 에다가 `new`를 사용하여 객체를 만들수 없다, 왜냐하면 추상 클래스 이기 때문이다.
그대신, 상속 클래스를 만들어서 추상 멤버를 정의할 필요가 있다.

```ts
abstract class Base {
  abstract getName(): string;
  printName() { }
}
//cut
class Derived extends Base {
  getName() {
    return "world";
  }
}

const d = new Derived();
d.printName();
```

만약 베이스 클래스의 추상 멤버를 구현하지 않을 경우, 에러가 출력된다는 것을 유의하라:

```ts
abstract class Base {
  abstract getName(): string;
  printName() { }
}
//cut
class Derived extends Base {
  // 구현 하는 것을 잊어 버림
}
```

### Abstract Construct Signatures

때때로 클래스 생성자가 추상 클래스로부터 상속받은 클래스를 사용하여 추상클래스의 인스턴스를 생성하기를 원하는 때가 있을 것이다.

예를 들어, 다음과 같은 코드를 작성하고 싶을 때가 있다:

```ts
abstract class Base {
  abstract getName(): string;
  printName() { }
}
class Derived extends Base { getName() { return "" } }
//cut
function greet(ctor: typeof Base) {
  const instance = new ctor();
  instance.printName();
}
```

타입스크립트는 정확히 이 코드는 추상 클래스를 인스턴스화 하고 있다고 알려줄 것이다.
결국, 주어진 `greet` 정의는 완전히 적절하며, 추상 클래스를 사용하여 인스턴스를 생성하는 것으로 귀결될 것이다.

```ts
declare const greet: any, Base: any;
//cut
// Bad!
greet(Base);
```

그대신, 생성자 시그니쳐를 가진 그 무엇을 받아들이는 함수를 작성하기 원한다면:

```ts
abstract class Base {
  abstract getName(): string;
  printName() { }
}
class Derived extends Base { getName() { return "" } }
//cut
function greet(ctor: new() => Base) {
  const instance = new ctor();
  instance.printName();
}
greet(Derived);
greet(Base);
```

이제 타입스크립트는 생성자 함수가 호출될 수 있음을 말해줄 것이다 - `Derived`는 콘크리트 이므로 가능하지만, `Base`는 불가능 하다.

## Relationships Between Classes

대부분의 경우, 타입스크립트에서 클래스는 구조적으로 비교된다, 이것은 다른 타입도 마찬가지 이다.

예를 들어, 아래의 두 클래스는 완전히 동일하게 간주되므로 서로 위치를 바꾸어서 사용될 수 있다.

```ts
class Point1 {
  x = 0;
  y = 0;
}

class Point2 {
  x = 0;
  y = 0;
}

// OK
const p: Point1 = new Point2();
```

비슷하게, 상속이 명확히 선언되지 않은 경우 클래스 간의 서브 타이핑 관계도 마찬가지 이다.

```ts
// @strict: false
class Person {
  name: string;
  age: number;
}

class Employee {
  name: string;
  age: number;
  salary: number;
}

// OK
const p: Person = new Employee();
```

이것은 매우 직관적으로 들리지만, 다른 것과 다르게 약간 이상하게 보이는 경우도 몇가지 있다.

빈 클래스는 멤버가 없는 것을 말한다.
구조적 타입 시스템에서 멤버가 없는 타입은 일반적으로 어느 타입의 서브타입도 될수가 있다.
따라서 빈 클래스를 사용한다면(하지말라!), 어떤 것도 그것을 대치할 수 있다:

```ts
class Empty { }

function fn(x: Empty) {
  // 'x'룰 가지고 그 무어도 할수가 없다, 따라서 나는 안쓴다.
}

// All OK!
fn(window);
fn({ });
fn(fn);
```
