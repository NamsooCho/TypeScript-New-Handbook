# Types from Transformation

자바스크립트에서 매우 흔하게 발생하는 어떤 패턴이 있다, 새로운 객체를 만들기 위해 객체들의 키를 순회한다던지, 제공된 입력 데이터에 기반해서 다른 값을 반환한다든지 하는 패턴이다.
즉석에서 타입과 값들을 생성하는 생각은 타입이 있는 언어에서는 약간 전통적인 방식이 아니다, 그러나 타입스크립트는 타입시스템에 유용한 기본적 구조들을 제공해서 이러한 행동을 정확히 모델링하게 해주며, `keyof` 가 객체의 프로퍼티 이름을 토론하는데 사용되는 방식과 매우 비슷하듯이, 그리고 인덱스 접근 타입이 특정 프로퍼티 이름을 추출하는데 사용될 수 있다.

우리는 간단히 이러한 조그만 구조가 결합해서 놀라울 정도로 강력하고 자바스크립트 생태계에 있는 많은 패턴을 표한할 수 있다는 것을 보여줄 것이다.

## Conditional Types

대부분의 유용한 프로그램의 핵심에서, 우리는 입력에 기반해서 결정을 내려야만 한다.
자바스크립트 프로그램도 다르지 않다, 그러나 값이 쉽게 해석된다는 것이 사실이라면, 그러한 결정은 입력의 타입에도 또한 기반 한다.
*조건 타입*은 입력과 출력 타입 간의 관계를 기술하는데 도움을 준다.

```ts
interface Animal {
    live(): void;
}
interface Dog extends Animal {
    woof(): void;
}

type Foo = Dog extends Animal ? number : string;
     ^?

type Bar = RegExp extends Animal ? number : string;
     ^?
```

조건 타입은 자바스크립트에서의 전통적 식 표현(`조건 ? 참인 경우의 식 : 거짓인 경우의 식`)과 약간 유사한 형태를 가진다:

```ts
type SomeType = any;
type OtherType = any;
type TrueType = any;
type FalseType = any;
type Stuff =
//cut
SomeType extends OtherType ? TrueType : FalseType
```

`extends`의 좌측 타입이 오른쪽 타입으로 대입가능하다면, 당신은 첫번째 브랜치(참인 경우 브랜치)를 얻을 것이고, 그렇지 않다면 당신은 두번째 브랜치(거짓인 경우 브랜치)를 얻을 것이다.

위 예제로 부터, 전통적 타입에는 조건타입이 즉각적으로 유용하지 않을 수 있다 - 우리는 `개가 동물을 확장한다` 가 맞는지 틀리는지 알 수 있고, `number` 나 `string`을 쉽게 뽑아 낼 수 있다!
그러나 조건 타입의 힘은 제너릭과 함께 사용될 때 발휘된다.

예를 들어, 다음의 `createLabel` 함수를 보자:

```ts
interface IdLabel { id: number, /* some fields */ }
interface NameLabel { name: string, /* other fields */ }

function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
    throw "unimplemented";
}
```

이러한 오버로드된 createLabel은 하나의 자바스크립트 함수를 기술하는데 인풋에 기반하여 하나의 선택을 하는 것이다. 몇 가지 유의점을 보자:

1. 만약 어떤 라이브러리가 자신의 API에서 같은 종류의 선택을 계속 반복해야 한다면, 이것은 성가신 일이 될 것이다.
2. 우리는 세 개의 오버로드를 만들어야만 한다: 하나는 우리가 타입을 *확신*하는 경우(하나는 `string` 그리고 하나는 `number`), 또 하나는 가장 일반적 케이스(`string | number`을 받는 경우). `createLabel`이 처리할 수 있도록 새로운 타입이 추가되려면, 오버로드의 갯수는 기하급수적으로 증가할 것이다.

그대신, 우리는 조건 타입으로 로직을 구성할 수 있다:

```ts
interface IdLabel { id: number, /* some fields */ }
interface NameLabel { name: string, /* other fields */ }
//cut
type NameOrId<T extends number | string> =
    T extends number ? IdLabel : NameLabel;
```

그다음 조건 타입을 사용하여 로버로드들을 단순화해서 하나의 함수로 사용할 수 있게 된다.

```ts
interface IdLabel { id: number, /* some fields */ }
interface NameLabel { name: string, /* other fields */ }
type NameOrId<T extends number | string> =
    T extends number ? IdLabel : NameLabel;
//cut
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
    throw "unimplemented"
}

let a = createLabel("typescript");
    ^?

let b = createLabel(2.8);
    ^?

let c = createLabel(Math.random() ? "hello" : 42);
    ^?
```

### Conditional Type Constraints

때때로, 조건 타입의 검사조건은 우리에게 새로운 정보를 제공한다.
타입 가드의 narrowing이 우리에게 더욱 세밀한 타입을 주는 것 처럼, 조건 타입의 참 브랜치는 우리가 검사하는 타입에 대해 더 세밀한 조건의 제너릭으로 제약한다.

예를 들어, 다음을 보자:

```ts
type MessageOf<T> = T["message"];
```

이 예제에서, 타입스크립트는 에러가 되는데 그 이유는 `T`가 `message`라는 프로퍼티를 가지는지 알 수 없기 때문이다.
우리는 `T`에 제약을 줄 수 있으며, 타입스크립트는 더 이상 에러로 처리하지 않을 것이다:

```ts
type MessageOf<T extends { message: unknown }> = T["message"];

interface Email {
    message: string;
}

interface Dog {
    bark(): void;
}

type EmailMessageContents = MessageOf<Email>;
     ^?
```

그러나, 우리가 `MessageOf` 가 어떤 타입이라도 받아주길 원하다면, 그리고 `message` 프로퍼티가 없다면 `never`와 같은 기본 값이 되기를 원한다면 어떨까?
우리는 이것을 제약을 밖으로 옮겨 주고 조건 타입을 도입함으로써 할 수 있다:

```ts
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

interface Email { message: string }

interface Dog { bark(): void }

type EmailMessageContents = MessageOf<Email>;
     ^?

type DogMessageContents = MessageOf<Dog>;
     ^?
```

참 브랜치 안에서 타입스크립트는 `T`가 `message` 프로퍼티를 갖게 *될* 것이라는 것을 알 수 있다.

다른 예제로서, `Flatten` 아라고 불리우는 타입을 작성할 수 있는데, 이것은 배열의 타입을 그것의 요소들의 타입으로 평탄화 하는 것이며, 배열이 아닌 경우 그대로 놔두는 것이다:

```ts
type Flatten<T> = T extends any[] ? T[number] : T

// Extracts out the element type.
type Str = Flatten<string[]>;
     ^?

// Leaves the type alone.
type Num = Flatten<number>;
     ^?
```

배열 타입에 대해 `Flatten`이 적용되면, `number`를 가지고 인덱스 접근을 하여 `string[]` 요소들 타입을 꺼내온다.
그게 아니면, 그것은 단순히 주어진 타입을 그대로 반환한다.

### Inferring Within Conditional Types

우리는 조건 타입을 사용하여 제약조건을 적용하고 타입을 꺼내오는 것을 행하였다.
이것은 조건 타입이 더 쉽게 공통적인 연산을 하도록 종결 짓는다.

조건 타입은 `infer` 키워드를 사용하여 참 브랜치에서 타입을 비교하는데 추정을 사용 하도록 하는 방법을 제공한다.
예를 들어, 인덱스 접근 타입을 아려고 *수작업*으로 요소를 꺼내오는 대신에 `Flatten`에서 요소의 타입을 추정할 수 있다:

```ts
type Flatten<T> = T extends Array<infer U> ? U : T;
```

여기서, 우리는 `infer` 키워드를 사용하여 `U`라는 새로운 제너릭 타입 변수를 도입하였는데, 이것은 참 브랜치에서 `T` 타입의 요소를 꺼내오도록 규정하는 것 대신에 사용되었다.
이것은 우리가 관심을 갖는 타입의 구조를 파고들어 분해하도록 생각해야 하는 수고를 덜어준다.

`infer` 키워드를 사용하여 우리는 유용한 헬퍼 타입 별칭을 작성할 수 있다.
예를 들어, 단순한 경우로, 함수 타입의 반환타입을 추출할 수 있다:

```ts
type GetReturnType<T> =
    T extends (...args: never[]) => infer U ? U : never;

type Foo = GetReturnType<() => number>;
     ^?

type Bar = GetReturnType<(x: string) => string>;
     ^?

type Baz = GetReturnType<(a: boolean, b: boolean) => boolean[]>;
     ^?
```

## Distributive Conditional Types

조건 타입이 제너릭 타입에서 동작하면, 주어진 유니언 타입에 대해 *distributive*가 된다.
예를 들어, 다음을 보라:

```ts
type Foo<T> = T extends any ? T[] : never;
```

우리가 유니언 타입을 `Foo`에 주입하면, 조건 타입은 그 유니언의 각 멤버에게 적용된다.

```ts
type Foo<T> = T extends any ? T[] : never;

type Bar = Foo<string | number>;
     ^?
```

`Foo`가 여기에서 distributes 가 다음에 대해 되면 무슨 일이 벌어지는가?

```ts
type Blah =
//cut
string | number
```

그리고 유니온의 각 멤버에 대해 매핑되면 실제로 무슨 효과인가?

```ts
type Foo<T> = T extends any ? T[] : never;
type Blah =
//cut
Foo<string> | Foo<number>
```

이것은 다음의 결과를 가져온다.

```ts
type Blah =
//cut
string[] | number[]
```

일반적으로, distributivity는 설계된 대로 동작한다.
그 동작을 피하려면, 당신은 `extends` 키워드 양옆 요소를 스퀘어 브라켓으로 감싸면 된다.

```ts
type Foo<T> = [T] extends [any] ? T[] : never;

// 'Bar' is no longer a union.
type Bar = Foo<string | number>;
     ^?
```