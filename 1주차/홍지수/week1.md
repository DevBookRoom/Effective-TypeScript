# 📘 Effective TypeScript - 1주차 정리 (Item 1~9)

> 📚 [원본 링크](https://github.com/danvk/effective-typescript?tab=readme-ov-file)

---

## 1. 자바스크립트와 타입스크립트 관계 이해하기

- 모든 자바스크립트는 타입스크립트이다.
- 하지만, 일부 자바스크립트(그리고 타입스크립트)만이 타입 체크를 통과한다.

---

## 2. 타입스크립트 설정 이해하기

- 설정 파일은 `tsc --init`으로 간단히 생성 가능

#### 🔹 `noImplicitAny`

- 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어

```ts
// false
function add(a: any, b: any): any {
  return a + b;
}

// true
function add(a: number, b: number): number {
  return a + b;
}
```

#### 🔹 `strictNullChecks`

- `null`과 `undefined`가 모든 타입에서 허용되는지 확인

```ts
const x: number = null; // 유효한 값인지 판단
```

---

## 3. 코드 생성과 타입이 관계없음을 이해하기

- 타입 오류가 있는 코드도 컴파일 가능
- 런타임 시 타입체크는 불가능
- 타입 연산은 런타임에 영향을 주지 않음
- 런타임 타입은 선언된 타입과 다를 수 있음
- 타입스크립트 타입으로는 함수를 오버로드할 수 없음
- 타입스크립트 타입은 런타임 성능에 영향을 주지 않음
  - → **타입과 타입 연산자는 자바스크립트 변환 시점에 제거됨**

---

## 4. 구조적 타이핑에 익숙해지기

- 자바스크립트는 본질적으로 덕 타이핑 기반

### 🦆 덕 타이핑

> 객체의 실제 타입보다는 그 객체가 무엇을 할 수 있는지가 중요하다는 방식

```ts
function quack(duck) {
  if (typeof duck.quack === "function") {
    duck.quack();
  } else {
    console.log("이건 오리가 아니야!");
  }
}

const realDuck = {
  quack: function () {
    console.log("꽥꽥!");
  },
};

const toyDuck = {
  quack: () => console.log("삑삑!"),
};

quack(realDuck); // 꽥꽥!
quack(toyDuck); // 삑삑!
```

- 자바스크립트는 는 동적 타입 언어로, 변수나 함수의 인자 타입을 명시하지 않아도 되고 런타임에 객체 구조만 맞으면 정상 동작
- 구조적 타이핑은 이런 자바스크립트를 모델링하기 위한 방식
- 클래스도 구조적 타이핑을 따른다. 클래스 인스턴스가 예상과 다를 수 있음
- 구조적 타이핑은 유닛 테스트에서 mock을 만들기 쉽게 해줌

---

## 5. any 타입 지양하기

- `any` 타입은 다음과 같은 문제를 유발한다:
  - 타입 안정성이 없음
  - 함수 시그니처를 무시함
  - 언어 서비스가 적용되지 않음
  - 코드 리팩터링 시 버그를 감춤
  - 타입 설계를 감춤
  - 타입 시스템 신뢰도를 떨어뜨림

---

## 6. 편집기를 사용하여 타입 시스템 탐색하기

- 타입스크립트의 강력한 도구 중 하나는 편집기 지원
  - 마우스 오버, 자동완성, 타입 추론 등을 통해 타입을 탐색 가능

---

## 7. 타입이 값들의 집합이라고 생각하기

- `never` = 공집합: 아무 값도 포함하지 않음
- 리터럴 타입: 단 하나의 값만 포함
- 타입은 엄격한 상속 구조가 아닌 벤 다이어그램으로 이해하는 것이 더 적절

```ts
interface Vector1D {
  x: number;
}
interface Vector2D {
  x: number;
  y: number;
}
interface Vector3D {
  x: number;
  y: number;
  z: number;
}
```

- 한 객체의 추가적인 속성이 타입 선언에 언급되지 않더라도 그 타입에 속할 수 있다.

---

## 8. 타입 공간과 값 공간의 심벌 구분하기

### 타입 공간 vs 값 공간

| 구분      | 설명                                                                   |
| --------- | ---------------------------------------------------------------------- |
| 타입 공간 | 타입만 존재하는 영역 (`type`, `interface`, 타입용 `typeof`)            |
| 값 공간   | 런타임에 존재하는 영역 (`const`, `let`, `function`, 런타임용 `typeof`) |

- 같은 이름의 심벌이 타입과 값 공간에 따로 존재할 수 있음(권장되는 방식은 아님)

```ts
type Visualcamp = "SeeSo" | "RnD" | "Marketing" | "";
const Visualcamp = "Eye tracking";
```

- `typeof`는 문맥에 따라 다르게 작동

```ts
// 값 공간
const jamesType = typeof james;
console.log(jamesType); // "object"

// 타입 공간
type James = typeof james; // james의 타입을 추론
```

### 클래스는 예외적으로 타입/값 공간에 모두 존재

```ts
class Dog {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

type DogConstructor = typeof Dog; // 생성자 함수의 타입
type DogInstance = Dog; // 클래스 인스턴스의 타입
```

---

## 9. 타입 단언보다는 타입 선언을 사용하기

- 단언문은 컴파일 시 제거되므로 조심해야 함
- `!`는 접미사로 쓰이면 해당 값이 null이 아님을 단언하는 문법

```ts
const el = document.getElementById("myDiv")!;
el.innerHTML = "Hello"; // null이면 런타임 에러 발생
```

- 보다 안전한 방식:

```ts
const el = document.getElementById("myDiv");
if (el) {
  el.innerHTML = "Hello";
}
```

---

## 🎯 요약

- 타입스크립트는 정적 타입 언어지만, 자바스크립트의 유연함을 고려한 설계
- 타입 공간/값 공간, 구조적 타이핑, any 사용 지양 등 기본 개념을 정확히 이해하면 훨씬 안전하고 강력한 코드를 작성할 수 있음
