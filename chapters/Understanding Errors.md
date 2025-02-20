
# Understanding Errors

타입스크립트가 에러를 발견 할 때 마다, 무엇이 잘못 되었는지 가능한 자세히 설명하려고 한다.
타입 시스템이 구조적이기 때문에, 이 노력은 자주 문제를 발견한 곳에서 상당히 긴 설명을 제공하는 것을 의미한다.

## Terminology

에러 메시지에서 당신이 자주 발견하는 몇명 용어는 문제를 이해하는데 도움이 된다.

#### *assignable to*

타입스크립트는 어떤 타입이 다른 타입에 *대입* 가능하다고 간주하는데, 그 조건은 어떤 것이 다른 것을 대체하는 것이 가능하다는 것이다.
다른 말로 하면, `Cat`은 `Animal`에 *대입 가능* 하다고 하는데 그 이유는 `Cat`은 `Animal`을 대체 가능하기 때문이다.

그 이름이 암시하듯이, 이 관계는 `t = s;` 라는 대입문의 유효성을 검사하는데 사용되는데 `t` 와 `s`의 타입을 검사함으로써 이루어 진다.
두 개의 타입이 상호작용하는 다른 곳에서도 이러한 검사는 사용된다.
예를 들어, 함수를 호출할 때, 각 아규먼트의 타입은 선언된 파라메터의 타입에 *대입*가능한지 검사한다.

비 정형적으로, `T is not assignable to S`라는 것을 보게 되면, 당신은 타입스크립트가 *"`T` 와 `S` 는 호환성이 없다"*라고 말한다고 생각하면 된다.
그러나, 이것은 *방향성* 있는 관계임을 유의하라: `S`가 `T`에 대입가능 하다고 해서 `T` 가 `S`에 대입가능 하다는 것을 암시하는 것은 아니다.

## Examples

몇몇 예제 에러 메시지를 보고 무엇이 잘못되었나 이해해보자.

### Error Elaborations

각각의 에러는 리딩 메시지로 시작한다, 때때로 추가적인 서브메시지를 갖고 있다.
각각의 서브 메시지는 위의 메시지가 *왜?* 라는 질문에 답하는 것이라고 생각할 수 있다.
실제로 그들이 어떻게 일하는지 몇몇 예제로 살펴보자.

여기에 예제 그자체보다 더 긴 에러 메시지를 생산하는 예제가 있다:

```ts
let a: { m: number[] };
let b = { m: [""] };
a = b;
```

타입스크립트는 마지막 라인을 검사할 때 에러를 발견 한다.
에러를 내는 로직은 대입문이 정상인지 검사하는 다음의 로직을 따른다.

1. `b`의 타입이 `a`에 대입이 가능한가? 아니라면 왜?
2. 왜냐하면 `m` 프로퍼티의 타입이 호환성이 없기 때문이다. 왜?
3. 왜냐하면 `b`의 `m`프로퍼티(`string[]`)가 `a`의 `m` 프로퍼티 (`number[]`)에 대입가능하지 않기 때문이다. 왜?
4. 왜냐하면 배열의 요소타입 (`string`)이 다른 배열의 요소 타입 (`number`)에 대입 가능하지 않기 때문이다

### Extra Properties

```ts
type A = { m: number };
const a: A = { m: 10, n: "" };
```


### Union Assignments

```ts
type Thing = "none" | { name: string };

const a: Thing = { name: 0 };
```

