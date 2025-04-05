# 1장. 타입스크립트 알아보기

## 아이템1. 타입스크립트와 자바스크립트 관계 이해하기

- 타입스크립트는 자바스크립트의 상위집합(superset)
    - 모든 자바스크립트 코드는 이미 타입스크립트 → js를 ts로 마이그레이션 하는데에 이점
    - 반대로 타입스크립트 코드는 자바스크립트X
        - TS는 타입을 명시하는 추가적인 문법을 가지고 있기 때문 (ex-타입구문)
    - 타입스크립트의 타입 구문
        - 타입스크립트는 런타임에 오류를 발생시킬 코드를 미리 찾아냄
        - 명시적으로 타입을 알려주지 않아도, 초깃값으로부터 타입 추론 가능
        - interface를 사용해 의도를 명확히하여 타입스크립트가 잠재적 문제점을 찾을 수 있게 할수있다
            
            ```tsx
            interface State {
            	name: string;
            	capital: string;
            }
            
            const states: State[] = [
            	{name: 'Arizona', capitol: 'Pheonix'} // 'capitol' 오류 발견 가능
            ] 
            ```
            
- 타입스크립트는 자바스크립트 런타임 동작을 모델링함
    
    ```tsx
    // [case1]
    const x = 2 + '3' // js, ts 모두 문자열 "23" 출력	
    
    // [case2]
    const a = null + 7; 
    // js는 a값 7 출력
    // ts의 타입체커는 문제점 표시
    ```
    
    - 타입스크립트는 런타임 오류를 발생시키는 코드를 찾으려고 함
    - 자바스크립트에서 오류가 발생하지 않더라도, ts 타입체커가 의도치 않은 오류를 표시할 수 있음 (case2)
    - 타입체크를 통과하더라도 런타임 오류 발생 가능
    - 이러한 문법적 엄격함은 취향의 차이, 우열을 가릴 수 없다. 상황에 맞춰 쓰기

## 아이템2. 타입스크립트 설정 이해하기

- tsconfig.json
    - 타입스크립트 타입체커 설정파일
    - 타입스크립트를 어떻게 사용할 계획인지 알 수 있음
- noImplicityAny
    
    ```tsx
    function add(a, b) {return a+b;} // noImplicityAny가 설정되었다면 오류, 없다면 암시적 any
    function add(a:number, b:number) {return a+b;}
    ```
    
    - 변수들이 미리 정의된 타입을 가져야하는지 여부를 제어함
    - 첫번째 줄의 경우, noImplictyAny 설정X면 암시적any로 설정되지만, 설정되어있다면 오류가 됨
    - any를 자주 사용하면 타입체커는 무력해짐. TS 문제 발견하기가 수월해지고 코드가독성이 좋아지므로 가급적 NoImplictyAny를 설정하자
- strictNullChecks
    
    ```tsx
    const x: number = null; // strictNullChecks 해제면 유효, 설정되었다면 오류
    const x: number | null = null; // 명시적으로 null 허용
    ```
    
    - null과 undefined가 모든 타입에서 허용되는지 확인
    - null과 undefined관련 오류를 잡아내는데에 도움되지만, 코드작성을 어렵게 함
    - 해당 설정 없이 쓴다면 “undefined는 객체가 아닙니다” 런타임에러 주의해야함
    - ts에 strict 설정을 하면 대부분의 오류를 잡아냄

## 아이템3. 코드 생성과 타입이 관계 없음을 이해하기

<aside>
💡

- TS컴파일러의 역할
    1. 브라우저에서 동작 가능한 구버전의 자바스크립트로 트랜스파일
    2. 코드의 타입 오류 체크
    
    → 이 두가지는 서로 완벽히 독립적임
    
</aside>

1. 타입오류가 있는 코드도 컴파일이 가능함
    - 컴파일은 타입체크와 독립적으로 동작 → 타입오류가 있어도 컴파일 가능함
2. 런타임에는 타입체크가 불가능함
    
    ```tsx
    interface Square {width:number;}
    interface Rectangle extends Square {height:number;}
    
    function calcArea(s:Shape) {
    	if(s instanceof Rectangle) return s.width * s.height;
    	else return s.width * s.width;
    }
    ```
    
    - 타입스크립트의 타입은 제거가능(erasable) → 컴파일 과정에서 인터페이서, 타입, 타입구문은 제거됨
    - instanceof 체크는 런타임에 일어나지만, Rectangle은 타입이기때문에 런타임에 역할X
    - 런타임에 타입정보를 유지하려면,
        1. 태그기법 사용
            
            ```tsx
            // kind 추가
            interface Square {kind:'square', width:number;}
            interface Rectangle extends Square {kind:'rectangle', width:number; height:number;}
            
            function calcArea(s:Shape) {
            	if(s.kind==='rectangel') return s.width * s.height; // kind를 통한 비교
            	else return s.width * s.width;
            }
            ```
            
            - 태그된 유니온(tagged union)
            - 런타임에 타입정보를 유지
        2. 타입을 클래스로 만들기
            
            ```tsx
            
            class Square {
            	constructor(public width:number){} 
            }
            class Rectangle extends Square {
            	constructor(public width:number, public height:number){super(width)} 
            }
            type Shape = Square | Rectangle;
            
            function calcArea(s:Shape) {
            	if(s instanceof Rectangle) return s.width * s.height;
            	else return s.width * s.width;
            }
            ```
            
            - 인터페이스는 타입으로만 사용 가능하지만, 클래스는 타입과 값 모두 사용 가능함
3. 타입 연산은 런타임에 영향을 주지 않음
    - string or number를 항상 number로 정제하는 경우
        
        ```tsx
        function asNumber(val: number | string): number {
        	return val as number;
        }
        
        // 변환된 JS
        function asNumber(val) {return val;}
        ```
        
        - as number는 타입연산(타입 단언문) → 변환된 자바스크립트의 런타임에는 아무런 영향 X
        - 변환은 자바스크립트 연산을 통해서만 가능하다
4. 런타임 타입은 선언된 타입과 다를 수 있다
    
    ```tsx
    function setLightSwitch(value: boolean) {
    	switch (value) {
    		case true: 
    			turnLigntOn(); 
    			break;
    		case false:
    			turnLigntOff(); 
    			break;
    		default: console.log("???");
    	}
    }
    ```
    
    - :boolean은 타입선언문 → 런타임에 제거됨
    - 만약 해당 파람값이 실제로 문자열이었다면, 런타임에는 문자열이 전달됨
    - 런타임 타입과 선언된 타입이 맞지 않을 수 있다
5. 타입스크립트 타입으로는 함수를 오버로드 할 수 없다.
    
    ```tsx
    function add(a:number, b:number){return a + b;}
    function add(a:string, b:string){return a + b;}
    // 중복된 함수 구현
    
    // 구현체
    function add(a,b) {return a + b;}
    ```
    
    - 오버로딩: 동일한 이름에 매개변수만 다른 여러 버전의 함수 허용하는 것
    - ts가 js로 변환되며 구현체만 남게 됨 → 함수의 구현체는 오직 하나
6. 타입스크립트 타입은 런타임 성능에 영향을 주지 않음
    - 타입과 타입 연산자는 자바스크립트 변환 시점에 제거됨
        
        == 런타임 성능에 영향X 
        
        == 런타임 오버헤드X
        
    - 타입스크립트 컴파일러는 빌드타임 오버헤드 O

## 아이템4. 구조적 타이핑에 익숙해지기

- 자바스크립트는 덕 타이핑(duck typing)기반
    - 덕타이핑: 객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우, 해당 타입에 속하는 것으로 간주
    - 매개변수값이 요구사항을 만족한다면 타입이 무엇인지 신경쓰지 않음
- 타입스크립트는 이를 모델링 하기 위해 구조적 타이핑을 사용함
    - 구조적타이핑: 타입 간의 관계를 선언하지 않고도 구조적으로 동일한 타입을 가지면 허용해주는 것
        
        ```tsx
        interface Vector2D {
          x: number;
          y: number;
        }
        function calculateLength(v: Vector2D) {
          return Math.sqrt(v.x * v.x + v.y * v.y);
        }
        
        interface NamedVector {
          name: string; // diff
          x: number;
          y: number;
        }
        const v: NamedVector = { x: 3, y: 4, name: 'Zee' };
        
        calculateLength(v); // return 5
        // Vector2D와 NamedVector의 관계를 선언하지 않았음에도 구조적으로 동일한 타입을 가짐으로 함수 호출 가능
        ```
        
    - 구조적 타이핑은 문제를 발생시킬 수 있다
        
        ```tsx
        // 위 코드 이어서
        interface Vector3D {
          x: number;
          y: number;
          z: number;
        }
        function normalize(v: Vector3D) {
          const length = calculateLength(v); // 2d기반 연산 함수, v=3d 기반
        
          return {
            x: v.x / length,
            y: v.y / length,
            z: v.z / length,
          };
        }
        
        normalize({ x: 3, y: 4, z: 5 }); // {x: 0.6, y: 0.8, z: 1}
        ```
        
        - Vector2d를 받는 calculateLength를 Vector3D로 호출하여도, 구조적 타이핑 관점에서 호환됨 → x, y가 있기 때문
        - 타입스크립트의 타입은 열려있기 때문에 (타입의 확장에 열려있음) 타입에 선언된 이외의 속성 가질 수 있음
    - 구조적 타이핑은 클래스 관련된 할당문에서도 이슈
        
        ```tsx
        class C {
        	foo: string;
        	constructor(foo: string) {this.foo = foo;}
        }
        
        const c = new C('instance of C');
        const d: C = {foo: 'object literal'}// d를 C타입에 할당 가능!
        ```
        
        - 구조적으로 필요한 1️⃣속성(string타입의 foo), 2️⃣하나의 매개변수로 호출되는 생성자(Object.prototype)가 존재하기 때문에 d를 C타입에 할당 가능

## 아이템5. any타입 지양하기

타입스크립트의 타입시스템은 점진적(코드에 타입을 조금씩 추가 가능), 선택적(타입체커 해제 가능) ⇒ any를 통해 가능. 하지만 any를 사용하면 TS의 장점들을 누릴 수 없음!

- any는 타입 안정성이 없다
- any는 약속된 타입을 받고, 약속된 타입을 반환하는 함수 시그니처(contract)를 무시한다
- any타입은 언어서비스(자동완성, 도움말, 타입-이름 포매팅 기능)가 적용되지 않는다
- any타입은 코드 리팩토링때 버그를 감춘다
- any타입은 설계를 감춘다
- any는 타입시스템의 신뢰도를 떨어뜨린다

# 2장. 타입스크립트의 타입 시스텝

## 아이템6. 편집기를 사용하여 타입시스템 탐색하기

- 타입 스크립트의 언어서비스
    - 편집기에서 언어서비스를 이용해 코드 자동완성, 명세, 검사, 검색, 리팩토링 등가
    - 타입스크립트는 타입을 추론함, 만약 이외의 경우라면 타입구문을 명시해야한다

## 아이템7. 타입이 값들의 집합이라고 생각하기

타입 == 할당 가능한 값들의 집합

| 타입스크립트 용어 | 집합 용어 |
| --- | --- |
| never | 공집합 |
| 리터럴타입 | 원소가 한개인 집합 |
| 값이 T에 할당 가능 | 값 ∈ T |
| T1이 T2에 할당 가능 | T1 ⊂ T2 |
| T1이 T2를 상속 | T1 ⊂ T2 |
| T1 & T2 | T1 ∩ T2 |
| T1 | T2 | T1 ∪ T2 |

### never

공집합 = 아무값도 포함하지 않음 = 아무 값 할당 X

### 리터럴타입 = 유닛타입

한가지 값만 포함하는 타입

```tsx
type A = 'A';
type B = 'B';
type twelve = 12;
```

### 유니온타입

2개 혹은 3개로 묶은 타입

```tsx
type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;

const a: AB = 'A' // 정상
const c: AB = 'C' // 'C'형식은 AB형식에 할당할 수 없음
```

### 타입체커

타입체커의 기능은 **하나의 집합이 다른집합의 부분집합인지 검사하는 것**

```tsx
interface Identified {
	id:string;
}
```

- 인터페이스란 타입 범위 내 값들에 대한 설명
    
    → 만약 어떤 객체가 string으로 할당 가능한 id속성을 가지고 있다면, 그 객체는 Identified 타입
    

### Intersection & Union 연산자

```tsx
interface Person {
	name: string;
}
interface LifeSpan {
	birth: Date;
	death?: Date;
}
```

1. Intersection ( & )
    
    ```tsx
    type PersonSpan = Person & LifeSpan;
    
    const ps: PersonSpan = {
    	name: 'ABC',
    	birth: new Date('1888/08/18');
    	death: new Date('1999/09/19');
    }
    ```
    
    - 두 타입의 교집합 = 두 타입의 범위의 교집합 = Person과 Lifespan을 둘다 가지는 값
    - 앞의 세가지보다 더 많은 속성을 가지는 값도 PersonSpan에 속함 (구조적 타이핑)
2. Union ( | )
    
    ```tsx
    type K = keyof (Person | LifeSpan); // never (공집합)
    ```
    
    - 두 타입의 합집합에 속하는 값은 어떠한 키 값도 없음 == never

```tsx
keyof (A&B) = (keyof A) | (keyof B)
keyof (A|B) = (keyof A) & (keyof B)
```

### extends

타입은 할당 가능한 집합이라는 관점에서, extends는 **‘~에 할당가능한’, ‘~의 부분집합’** 이라는 의미로 받아들일 수 있다

```tsx
interface Vector1D { x: number; }
interface Vector2D extends Vector1D { y: number; }
interface Vector3D extends Vector2D { z: number; }

// 동일
interface Vector1D { x: number; }
interface Vector2D { x: number, y: number }
interface Vector3D { x: number, y: number, z: number }
```

- Vector1D ⊇ Vector2D ⊇ Vector3D
- Vector3D는 Vector2D의 서브타입, Vector2D는 Vector1D의 서브타입
- extends를 사용하지 않고 인터페이스로 코드를 재작성해도, 부분집합, 서브타입, 할당 가능성의 관계가 바뀌지 않는다

## 아이템8. 타입 공간과 값 공간의 심벌 구분하기

TS의 symbol은 1️⃣타입공간 이나 2️⃣값공간 중 한곳에 존재한다.

```tsx
interface Cylinder {
	radius: number;
	height: number;
}

const Cylinder = (radius:number, height:number) => ({ radius, height });
```

- interface의 Cylinder는 타입
- const Cylinder는 값

→ 상황에 따라 타입으로 쓰일 수도, 값으로 쓰일 수도 있다

```tsx
function calculateVolume(shape: unknown) {
	if(shape instanceof Cylinder) {
		shape.radius // ERROR - {}형식에 radius 속성이 없습니다.
	}
}
```

- instanceof는 자바스크립트의 런타임 연산자 이므로, 값에대해 연산한다.
- 따라서 instanceof Cylinder는 타입이 아니라 함수를 참조함

### type & const

```tsx
type T1 = 'string literal';
const v1 = 'string literal';

interface Person = { first: string; last: string; }
const p: Person = {first:'JANE', last:'SMITH'};
```

- 타입은 일반적으로 type이나 Interface 다음에 온고, 값은 const나 let 다음에 온다
- 일반적으로 타입선언(:), 타입단언문(as) 다음에 오는 심벌은 타입, =다음에 오는 것은 값

### class & enum

class와 enum은 타입과 값 두가지 모두 가능하다

```tsx
class Cylinder { radius=1; height=1; }

function calculateVolume(shape: unknown) {
	if(shape instanceof Cylinder) {
		shape      // 타입은 Cylinder
		shape.radius // 타입은 number
	}
}
```

- 클래스가 타입으로 쓰일때는 형태(속성 & method)가 사용됨
- 클래스가 값으로 쓰일 때는 생성자가 사용됨

### typeof

typeof는 연산자 중에 타입에서 쓰일 때와 값에서 쓰일 때 다른기능을 한다

```tsx
type T1 = typeof p; // 타입 - PERSON
const v1 = typeof p; // 값 - "object"
```

- type관점 : typeof는 값을 읽어서 TS타입을 반환함
- 값의 관점 : tpyeof는 자바스크립트 런타임의 typeof 연산자
    - 자바스크립트엔 단 6개의 런타임타입만 존재함 (string, number, boolean, undefined, object, function)

typeof의 클래스에서의 사용

```tsx
const v = typeof Cylinder; // 값 - "function"

type T = typeof Cylinder; // 타입 - typeof Cylinder (생성자함수)
type C = InstanceType<typeof Cylinder>; // 타입 - Cylinder (인스턴스 타입)
```

- 클래스는 JS에서 실제로 함수로 구현됨 → “function” (클래스가 값으로 쓰일때는 생성자!)
- typeof Cylinder는 인스턴스의 타입이 아님 → new키워드를 사용할 때 볼 수 있는 생성자 함수
- InstanceType 제너릭을 사용해서 생성자타입과 인스턴스 타입을 전환할 수 있다

### 속성접근자 []

속성접근자 []는 타입으로 쓰일 때도 동일하게 동작하지만, 하지만 접근자(.)는 값이 동일하더라도 타입이 다를 수 있다. 따라서 타입의 속성을 얻을 때에는 반드시 [] 접근자를 사용해야 한다.

```tsx
const first: Person['first'] = p['first'];
```

- Person['first'] → 타입맥락(:)에서 쓰였기 때문에 타입

### 두 공간 사이에서 다른 의미를 가지는 코드패턴

|  | 값 | 타입 |
| --- | --- | --- |
| this | js의 this키워드 | ‘다형성(polymorphic) this’라고 불리는 this의 ts타입 |
| &, | | AND, OR 비트연산 | 인터섹션, 유니온 |
| const | 새 변수 선언 | as const - 리터럴, 리터럴 표현식의 추론된 타입을 바꿈 |
| extends | 클래스 상속 | - 서브클래스 : class A extends B
-서브타입: interface A extends B
-제네릭 타입의 한정자 : Generic<T esxtends number> |
| in | 루프 (for key in obj) | 매핑된 타입 |

## 아이템9. 타입단언보다는 타입선언을 사용하기

TS에서 변수에 값을 할당하고 타입을 부여하는 방법에는 1️⃣타입선언, 2️⃣타입단언 이 있다

```tsx
interface Person {name: string};

const alice:Person = {name: 'Alice'}; // 타입 선언 (:)
const bob = {name: 'Bob'} as Person; // 타입 단언 (as)

const alice:Person = {}; // 타입선언 - 에러발생
const bob = {} as Person; // 타입단언 - 에러 미발생
```

- 타입선언은 할당되는 값이 해당 인터페이스를 만족하는지 검사함 → TS가 오류 표기
- 타입 단언을 수행한다면, 타입스크립트가 추론한 타입이 있더라도 Person타입으로 간주됨 → TS가 오류 무시

### 화살표함수의 타입단언

```tsx
const people = ['alice', 'bob', 'jan'].map(name => ({} as Person)); // 타입단언 - 에러 미발생
const people: Person[] = ['alice', 'bob', 'jan'].map((name):Person => ({name}));
```

- 타입단언을 사용하면 런타임에 문제가 발생함
- 단언문을 쓰지 않고, 화살표함수 안에서 반환타입을 명시하면 오류가 발생하지 않는다

### 타입단언이 꼭 필요한 경우

타입단언은 타입체커가 추론한 타입보다, 스스로 판단하는 타입이 더 정확할 때 의미가 있다.

1. DOM ELEMENT
    
    ```tsx
    document.querySelector('#abc').addEventListener('click', e => {
    	e.currentTarget // 타입은 EventTarget
    	const button = e.currentTarget as HTMLButtonElement; // 타입은 HTMLButtonElement
    })
    ```
    
    - TS는 DOM에 접근할 수 없기때문에, 버튼엘리먼트인 것을 알 수 없음
    - TS가 알지 못하는 정보를 우리가 알고있으니 타입단언문을 쓰는것이 타당
2. ! 로 null이 아님을 단언하기
    
    ```tsx
    const elNull = document.getElementById('foo'); // 타입은 HTMLElement | null
    const elNull = document.getElementById('foo')!; // 타입은 HTMLElement
    ```
    
    - !가 접미사로 쓰이면 null이 아니라는 단언문으로 해석됨
    - 값이 null이 아닐때만 사용해야한다