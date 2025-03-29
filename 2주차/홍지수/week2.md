# 📘 Effective TypeScript - 2주차 정리 (Item 10~18)

> 📚 [원본 링크](https://github.com/danvk/effective-typescript?tab=readme-ov-file)

---

## 10. 객체 래퍼 타입 피하기

- `String`, `Number`, `Boolean`은 JS에서 제공하는 래퍼 객체 타입 → 타입스크립트에서는 지양
- 대신 `string`, `number`, `boolean` 등의 기본 타입을 사용할 것

```ts
// ✅
const title: string = "책 제목";

// ❌
const count: Number = 5;
```

- typeof 사용 시 차이 발생

```ts
typeof "hello"; // "string"
typeof new String("hello"); // "object"
```

- string은 String에 할당 가능하지만, 그 반대는 불가
- 런타임의 값은 객체가 아닌 기본형(primitive) 이다

---

## 11. 잉여 속성 체크의 한계 인지하기

- 객체 리터럴을 직접 할당할 경우, TypeScript는 잉여 속성 체크 수행

```ts
interface Config {
  width: number;
  height: number;
}

// ❌ 직접 할당 → 에러
const config: Config = {
  width: 100,
  height: 200,
  color: "red",
};

// ✅ 변수로 우회하면 통과
const options = { width: 100, height: 200, color: "red" };
const config2: Config = options;
```

- 잉여 속성 체크는 타입 선언 시 객체 리터럴에만 적용되며, 구조적 타이핑과는 별개

```ts
interface Options {
  title: string;
  darkMode?:boolean
}
const o1: Options = document;
const o2: Options = new HTMlAnchorElement;
})
```

- 예시인 Options 타입은 범위가 매우 넓기 때문에, 순수한 구조적 타입체커는 이런 종류의 오류를 찾아내지 못한다. 현재의 darkmode속성에 boolean타입이 아닌 다른 타입의 값이 지정된 경우를 제외하면, 또다른 어떤 속성을 가지는 모든 객체는 Options타입의 범위에 속한다.
- Options 인터페이스 타입의 변수에는 document나 HTMlAnchorElement의 인스턴스 모두 string 타입인 title 속성을 가지고 있기때문에 할당문은 정상이다.

---

## 12. 함수 표현식에 타입 적용하기

- 함수 선언보다 **타입을 먼저 선언한 후 표현식으로 구현**하는 방식이 더 안전

```ts
// ✅ 명확한 방식
const double: (x: number) => number = (x) => x * 2;

// ❌ 암묵적 any 발생 가능
function double(x) {
  return x * 2;
}
```

- 함수 표현식을 사용하게 되면 매개변수와 반환값의 타입을 한 번에 선언할 수 있고 재활용 가능
- 반복되는 함수 시그니처는 타입 별칭으로 추출해서 재사용

```ts
type BinaryFn = (a: number, b: number) => number;

const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

---

## 13. 타입과 인터페이스의 차이점 알기

```ts
type Tstate = {
  name: string;
  capital: string;
};

interface IState {
  name: string;
  capital: string;
}
```

### 공통점

- 타입의 기본 동작

```ts
type Tname = {
  name: string;
};

interface Iname {
  name: string;
}

const typeName: Tname = {
  name: "James",
};

const interfaceName: Iname = {
  name: "James",
};
```

- 인덱스 시그니처 사용가능

```ts
type TypeIndexSignature = {
  [key: string]: string;
};

interface InterfaceIndexSignature {
  [key: string]: string;
}
```

- 함수타입 정의 가능

```ts
type TypeFunction = {
  (x: number): number;
};

interface InterfaceFunction {
  (x: number): number;
}

const typeFunction: TypeFunction = (x) => 0;

const interfaceFunction: InterfaceFunction = (x) => 0;
```

- 제너릭 가능

```ts
type TypeGeneric<T> = {
  first: T;
};

interface InterfaceGeneric<T> {
  first: T;
}
```

- 타입 확장

```ts
type TypeExtendedInterfaceGeneric<T> = InterfaceGeneric<T> & { second: T };

interface InterfaceExtendedTypeGeneric<T> {
  second: T;
}
```

- 클래스 구현

```ts
class ClassTypeGeneric<T> implements TypeGeneric<T> {
  first: T;

  constructor() {
    this.first = Object();
  }
}

class ClassInterfaceGeneric<T> implements InterfaceGeneric<T> {
  first: T;

  constructor() {
    this.first = Object();
  }
}
```

### 차이점

- 인터페이스는 extends키워드를 통해 확장 가능

```ts
interface IStateWithPop extends TState {
  population: number;
}
type TStateWithPop = IState & { population: number };
```

- 유니온과 같은 복잡한 타입을 type은 확장할 수 있다.
  - type에는 유니온 타입이 있지만, interface에는 유니온 인터페이스가 없다

```ts
type TypeAorB = "a" | "b";

interface InterfaceAorB {
  // ...?
}
```

- 튜플과 배열 타입도 type 키워드를 통해 더 간결하게 표현 가능

```ts
type Tuple = [number, number];
type StringList = string[];

const tuple: Tuple = [0, 1];
// Tuple 타입은 Array 메서드들을 사용 가능
```

```ts
interface Triple {
  0: number;
  1: number;
  2: number;
  length: 3;
}

const triple: Triple = [0, 1, 2];
```

- 인터페이스로 튜플 구현시 concat 같은 메서드 사용 불가
- interface는 보강 기법 사용 가능 → 선언을 2번해서 속성 확장

```ts
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

const wyoming: IState = {
  name: "Wyoming",
  capital: "Cheyenne",
  population: 500,
};
// 정상
```

---

## 14. 타입 연산과 제너릭 사용으로 반복 줄이기

- DRY(Don't Repeat Yourself) 원칙을 타입에도 최대한 적용해야 한다.
- 타입에 이름을 붙이기
  - extends를 사용해서 인터페이스 필드의 반복을 피해야 한다.

```ts
// 타입에 이름 붙이기
function distance(a: { x: number; y: number }, b: { x: number; y: number }) {
  // do something...
}

type Point2D = {
  x: number;
  y: number;
};

function distance2(a: Point2D, b: Point2D) {
  // do something...
}
- 중복된 함수 타입을 시그니처를 명명한 타입으로 분리하기

// 시그니처를 명명된 타입으로 분리
type HTTPFunction = (url: string, opts: Options) => Promise<Response>;
const get: HTTPFunction = (url, opts) => {
  /* ... */
};
const post: HTTPFunction = (url, opts) => {
  /* ... */
};
//
```

- 인터섹션 연산자 사용

```ts
type PersonWithBirthDate = Person & { birth: Date };
```

- Pick, Partial과 같은 유틸리티 타입 사용

```ts
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

type TopNavState = {
  userId: State["userId"];
  pageTitle: State["pageTitle"];
  recentFiles: State["recentFiles"];
};

type TopNavState = {
  [k in "userId" | "pageTitle" | "recentFiles"]: State[k];
};

type CustomPick<T, K extends keyof T> = {
  [k in K]: T[k];
};

type TopNavState = CustomPick<State, "userId" | "pageTitle" | "recentFiles">;
```

- 값의 형태에 해당하는 타입: typeof
- 함수나 메서드의 반환값에 해당하는 타입: ReturnType<typeof 함수>

---

## 15. 동적 데이터에 인덱스 시그니처 사용하기

- 인덱스 시그니처

```ts
type IndexSignatureType = { [property: string]: string };
// [property: string]: string
// (키의 이름)  (키의 타입) (값의 타입)
```

- 런타임 때까지 객체의 속성을 알 수 없을 경우에만 사용
- 단점

  - 모든 키를 허용한다. : 객체에는 없는 키를 이용하더라도 타입 체크에서 에러가 나지 않는다.
  - 특정 키가 필요하지 않는다. : 빈 오브젝트{}도 할당이 된다.
  - 키마다 다른 타입을 가질 수 없다. : 값의 타입을 유니온 타입을 통해 타입을 확장 시켜야 한다.
  - 타입스크립트의 언어 서비스를 제공받을 수 없다.

- 키들이 있지만 얼마나 많은 키들이 있는지 모른다면 선택적 필드 혹은 유니온 타입으로 모델링하면 된다.

```ts
interface Row1 { [column: string]: number } // 너무 광범위
interface Row2 { a:number; b?: number; c?: number; d?: number }  // 최선
type Row 3=
	| { a: number; }
	| { a: number; b: number; }
	| { a: number; b: number; c: number; }
	| { a: number; b: number; c: number; d: number; } // 가장 정확하지만 번거로움
```

- 런타임 때가지 객체의 속성을 알 수 없을 경우에만(예를 들어 CSV 파일에서 로드하는 경우) 인덱스 시그니처를 사용하도록 한다.
- 안전한 접근을 위해 인덱스 시그니처의 값 타입에 undefined를 추가하는 것을 고려해야 한다.
- 가능하다면 인터페이스, Record, 매핑된 타입 같은 인덱스 시그니처보다 정확한 타입을 사용하는 것이 좋다.

---

## 16. number 인덱스 시그니처보다는 Array, 튜플, ArrayLike을 사용하기

자바스크립트에서 이상하게 동작하기로 유명한 부분중 제일 악명높은것은
암시적 타입 강제라고 불리는 `Type coercion`에 있다.

```ts
"0" == 0; // true
```

- 다행히도 암시적 타입 강제와 관련된 문제는 ===와 !==로 대부분이 해결이 가능하다.

자바스크립트에서 객체란 키/값 쌍의 모음이다.
자바스크립트 엔진에서의 object의 키는 string 혹은 symbol타입만이 가능하다.
따라서, 배열에서도 number 타입의 key로 접근하는것이 불가능하다.
하지만 자바스크립트 엔진에서 자동으로 형변환이 이루어지기 때문에 number 타입의 키로도 접근이 가능하다.
즉, arr[0]은 내부적으로 arr['0']으로 바뀜

```ts
const arr = [1, 2, 3];
console.log(Object.keys(arr)); // ['0','1','2']
```

인덱싱은 number를 사용하지만, 실제 배열의 key들은 string이다.

따라서, 엄격하게 하자면 배열에 접근할 때에도 string 타입을 통해서만 배열의 원소에 접근이 가능하다.

그러나, 우리는 자바스크립트에서 범용적으로 number 타입의 키를 통해 배열의 원소에 접근을 했었기 때문에 타입스크립트에서는 일관성을 위해 number 타입의 키를 허용한다.

배열 순회

- 인덱스에 신경을 쓰지 않는다면 for-of
- 인덱스의 타입이 중요하다면 number 타입을 제공해 줄 Array.prototype.forEach
- 루프 중간에 멈추어야 한다면 for(;;)

ArrayLike

```ts
function checkedAccess<T>(xs: ArrayLike<T>, i: number): T {
  if (i < xs.length) {
    return xs[i];
  }
  throw new Error(`배열의 끝을 지나서 ${i}에 접근하려고 했습니다`);
}

// 드물지만 길이와 숫자 인덱스 시그니처만 있는것.
// ArrayLike를 사용하더라도 키는 여전히 `문자열`
const tupleLike: ArrayLike<string> = {
  "0": "A",
  "1": "B",
  length: 2,
}; // 정상
```

---

## 17. 변경 관련된 오류 방지를 위해 readonly 사용하기

### 객체 타입 Property 앞에 붙는 readonly

자바스크립트에서 상수 키워드인 const는 재할당을 금지한다.
원시 타입의 변수에게 const를 이용해서 생성을 해보면 직관적으로 이해할 수 있지만 조금 헷갈릴만한 내용이 있다.

```ts
const bibiObject = {
  property: "bucky",
};
bibiObject.property = "barnes";
```

const로 선언된 객체의 속성을 바꿀수 있다는점이다.
const로 선언된 객체는 해당 식별자에 대해서 다른 객체를 할당할수는 없지만 해당 객체의 속성의 변경은 가능하다는 것이다.

객체 속성의 변경을 막으려면 Object.freeze 함수를 이용

```ts
Object.freeze(bibiObject);
bibiObject.property = "barnes"; // error
```

타입스크립트에서는 readonly를 이용해서 속성 변경을 막을 수 있다.

```ts
type ReadonlyType = {
  readonly prop: string;
};

const readonlyType: ReadonlyType = {
  prop: "a",
};

readonlyType.prop = "b"; // cannot assign to 'prop' because it's a read-only property
```

readonly 접근 제어자를 사용할 때 readonly 접근 제어자가 얕게(shallow) 동작하는 것을 주의하기

```ts
type ReadonlyType = {
  readonly prop: InnerType;
};

type InnerType = {
  innerProp: string;
};

const readonlyType: ReadonlyType = {
  prop: {
    innerProp: "a",
  },
};

readonlyType.prop.innerProp = "b"; // 정상

readonlyType.prop = { innerProp: "b" }; // 읽기 전용 속성이므로 'prop'에 할당할 수 없습니다.
```

### Array, 튜플 타입 앞에 붙는 readonly

number[]와 같은 Array 타입 혹은 튜플 타입 앞에도 readonly 키워드를 사용 가능하다
이것은 배열 혹은 튜플을 수정할 수 없음을 의미한다.

```ts
const readOnlyArr: readonly number[] = [1, 2, 3];
readOnlyArr.push(3); // Property 'push' does not exist on type 'readonly number[]'
```

불변성을 가진 배열로 선언하고자 할때에 사용한다
추가로 함수의 인자 타입에서 사용하여 인자의 불변성을 유지하는 목적으로도 사용이 가능하다

함수 내에서 해당 인자를 변경하지 않는다면 readonly 접근 제어자를 사용하는 것이 좋다
불변성을 중요시하는 함수형 프로그래밍의 관점에서 좋은 습관이 될것이다

```ts
function functionReadOnly(arg: readonly number[]) {
  arg[0] = 100;
  // 'readonly number[]' 형식의 인덱스 시그니처는 읽기만 허용됩니다.
}
```

함수 내에서 해당 인자를 변경하지 않는다면 readonly 접근 제어자를 사용하는 것이 좋다.

- readonly 접근 제어자가 있으면 배열의 요소를 읽을 수 있으나, 새롭게 추가하거나 변경할 수 없다.
- 배열을 변경하는 pop, push 와 같은 메서드를 사용할 수 없다.
- length 속성을 읽을 수는 있으나, 변경 불가능하다.

---

## 18. 매핑된 타입을 사용하여 값을 동기화 하기

리액트에서는 props가 변경될 때 해당 컴포넌트와 자식 컴포넌트가 리렌더링되는데, 이벤트 핸들러 같이 눈에 보이는 요소가 아니라면 해당 부분이 바뀌어도 다시 렌더링 될 필요가 없다.

```ts
interface ViewProps {
  // data
  xs: number[];
  ys: number[];
  // display
  xRange: [number, number];
  yRange: [number, number];
  color: string;
  // event
  onClick: (x: number, y: number, index: number) => void;
}
```

최적화를 두가지 방법으로 구현해보기

### '보수적(conservative) 접근법' / '실패에 닫힌(fail close) 접근법'

```ts
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== "onClick") return true;
    }
  }
  return false;
}
```

새로운 속성이 추가되면 shouldUpdate 함수는 값이 변경될 때마다 차트를 다시 그린다. 이 접근법을 이용하면 차트가 정확하지만 너무 자주 그려질 가능성이 있다.

### 실패에 열린 접근법

```ts
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
    // (no check for onClick)
  );
}
```

차트를 불필요하게 다시 그리는 단점을 해결했지만, 실제로 차트를 다시 그려야 할 경우에 누락되는 일이 생길 수 있다.

### 최고의 최적화 방법

```ts
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}
```

`[k in keyof ViewProps]` 는 타입 체커에게 ViewProps와 동일한 속성을 가져야 한다는 것을 알려준다.
`REQUIRES_UPDATE` 에 boolean 값을 가진 객체를 사용했다.
나중에 추가 속성이 더해질때 REQUIRES_UPDATE에서 에러가 발생할 것이다.
