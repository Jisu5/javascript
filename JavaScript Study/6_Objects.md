# 6. Objects

> Object는 Javascript의 가장 기본적인/핵심적인 데이터 타입

## 6.1 Instruction of Objects

- Object는 복합체 : 여러 값(원시타입/다른 객체타입)을 묶어 이름으로 저장하고, 값을 가져올 수 있다.
- Object는 name-value로 구성된 프로퍼티들의 정렬되지 않은 집합이다
- 프로퍼티의 이름은 String이 대부분 (ES6부터 프로퍼티의 이름에 Symbol이 사용될 수 있음) 
  - => String을 value에 mapping 시키는 것으로 볼 수 있다.
- 'hash', 'hashtable', 'dictionary', 'associative array'등의 자료구조가 String-to-value mapping 구조이다.
  - Object는 단순한 string-to-value map이 아니다. 
    프로퍼티를 유지하는 것 외에도, javascript에서 object는 `"prototype"`이라는 또 다른 object의 프로퍼티를 상속 받는다.

- Object의 메소드들은 일반적으로 상속받은 프로퍼티이고, 이를 `프로토타입 상속(Prototypal Inheritance)`이라고 부른다.
  **(javascript의 핵심기능)**
- Javascript의 object는 동적으로 프로퍼티의 추가/삭제가 가능하지만, '정적 객체'를 흉내낼 수도 있고, 정적타입언어에서의 구조체를 가장할 수 도 있다.
- String-to-value mapping에서의 value를 무시한다면 문자열 집합으로 표현될 수 도 있다.



## 6.2 Creating Objects

>  `object literal`,  `new`키워드, `Object.create()`함수

### 6.2.1 Objects Literal

- 가장 쉬운 방법. 
- Object Literal은 curly brace로 감싼 colon-seperated name:value pairs의 comma-separated list로 표현
- primitive value나 object value가 프로퍼티의 값이 될 수 있다.

```javascript
let empty = {};
let point = { x: 0, y: 0 };
let p2 = { x: point.x, y: point.y+1 };
let book = {
  "main title": "JavaScript",							// 프로퍼티 이름이 공백, 하이폰을 포함하기 떄문에 string literal 사용
  "sub-title": "The Definitive Guide",
  for: "all audience",
  author: {
    firstname: "David",
    lastname: "Flanagan"
  }
}
```

- 마지막 프로퍼티에 따라오는 `trailing comma`를 사용할 수 있다. 몇몇 프로그래밍 스타일은 syntax error를 줄이기 위해 trailing comma사용을 권장한다.

### 6.2.2 Creating Objects with new

- `new` operator는 새로운 객체를 생성하고 초기화한다. 
- `new` 키워드 다음에는 함수 호출문(`function invocation`)이 따라와야 한다. 이 함수를 cunstructor라고 부르고 새로 생성된 object를 초기화 할 수 있다.

```javascript
let o = new Object();		// Create empty object: same as {}
let a = new Array();		// Create empty array: same as []
let d = new Date();			// Create a Date object representing the current time
let r = new Map();			// Create a Map object for key/value mapping
```

### 6.2.3 Prototypes

- 대부분의 javascript object는 또 다른 javascript object와 연관되어 있다. 이러한 또 다른 object를 `prototype`이라고 하고, object는 `prototype`의 프로퍼티를 상속받는다.
- `Object literal`로 생성된 모든 object는 같은 prototype object를 가지고, `Object.prototype()`이라는 코드로 prototype object를 참조 할 수 있다.
- `new`키워드와 cunstructor invocation으로 생성된 object는 cunstructor function의 prototype 값을 자신의 prototype으로 사용한다.
  - `new Object()`로 생성한 object는 `{}`로 생성된 object 처럼 `Object.prototype`를 상속받는다.
  - `new Array()`로 생성된 object는 `Array.prototype`, `new Date()`로 생성된 object는 `Date.prototype`을 상속받는다.
- **대부분의 object는 prototype을 가진다. 하지만 상대적으로 작은 수의 object는 `prototype 프로퍼티`를 가진다.**

#### Prototype Chain

- `Object.prototype`은 prototype를 가지지 않는 매우 드문 object중 하나다. (아무런 프로퍼티도 상속받지 않는다). 다른 prototype object들은 prototype을 가지는 보통의 object이다.
- `built-in constructor`와 사용자 정의 constructor는 Object.prototype으로부터 상속받은 prototype을 가진다.
- `Date.prototype`은 `Object.prototype`으로부터 상속을 받는다. 즉, `new Date()`로 생성한 Date object는 `Date.prototype`과 `Object.prototype` 을 상속 받는다.
- Prototype Chain: Prototype Object들이 연결된 것

### 6.2.4 Object.create()

- `Object.create()`로 생성한 object의 경우 첫 번째 인자가 `prototype object`이다.

```javascript
let o1 = Object.create({ x: 1, y: 2 });		// property x, y를 상속받는다.
o1.x + o1.y;															// 3
```

- Prototype을 갖지않는 object를 생성하려면 `null`을 전달하면 된다. 
  - 이 경우 아무것도 상속을 받지 않는다.
  - `toString()`과 같은 기본적인 메소드도 사용할 수 없다. (`+ `operator도 마찬가지로 사용할 수 없다.)

```javascript
let o2 = Object.create(null);
```

- 일반적인 빈 object(`{}`나 `new Object()`로 생성한 것과 같은 object)를 만들고 싶다면, `Object.prototype`을 전달하면 된다.

```javascript
let o3 = Object.create(Object.prototype);		// {}
```

- 임의의 prototype을 상속받는 object를 생성할 수 있다는 것은 매우 강력한 점이다. 
- 라이브러리 함수에 의한 객체의 의도하지 않은 수정을 막기위해서 `Object.create()`를 사용 할 수 있다. 객체를 직접 함수에 전달하는 것 대신, 객체의 프로퍼티를 상속받아 전달 하는 것이다. => 만약 함수가 프로퍼티를 수정한다면, 이는 기존의 객체가 아닌 새로 만들어진 객체에만 영향을 끼친다.

```javascript
let o = { x: "don't change this value" };
library_function(Object.create(o));
```



## 6.3 Querying and Setting Properties

- Property의 값을 얻기 위해서는 `.`, `[]`을 사용한다.
  - `.`을 사용하기 위해서는 identifier가 간단한 형태여야 한다.
  - `[]`안에는 문자열로 평가되는 표현식이 들어갈 수 있다.

```javascript
let author = book.author;
let name = author.surname;
let title = book["main title"];
```

- Property를 생성하거나 값을 설정할 때에도 `.`, `[]`을 사용한다. 이 떄 property는 할당 표현식의 왼쪽에 위치한다.

```javascript
book.edition = 7;
book["main title"] = "ECMAScript";
```

### 6.3.1 Objects As Associative Arrays

```javascript
Object.property
Object["property"]
```

- Property의 값을 얻기 위한 두가지 표현식 중 `[]`를 사용하는 표현식은 숫자가 아닌 문자열로 인덱싱된 array에 접근하는 것과 비슷하다. 
- 문자열로 인덱싱된 array를 `associative array`라고 한다. (Hash, map, dictionary ...)

- Javascript의 object는 `associative array`이다. 
  - C, C++, Java같이 타입의 제약이 엄격한 언어에서는 object는 정해진 수의 property만 가질 수 있고, property의 이름도 미리 정의되어 있다.
    Javascript는 타입의 제약이 느슨해, 어떤 object든 원하는 만큼의 property를 만들 수 있다.
  - `.`은 identifier로만 접근할 수 있지만, `[]`를 사용할 때는 property의 이름을 문자열로 표현하기 때문에, 프로그램 실행 중에 생성하고 조작할 수 있다.

```javascript
let addr = "";
for (let i = 0; i < 4; i++) {
  // address0, address1, address2, address3 프로퍼티를 읽어서 addr에 더한다.
	addr += customer[`address${i}`] + "\n"
}
```

- 생성하거나 접근할 객체 프로퍼티의 이름이 어떤 타입일지 미리 알지 못하는 경우에도 유용하다.

```javascript
function computeValue(portfolio) {
  let total = 0.0;
  for (let stock in portfolio) {
    let shares = portfolio[stock];
    let price = getQuote(stock);
    total += shares * price 
  }
  return total;
}
```

### 6.3.2 Inheritance

- Object는 고유 property들과 prototype에서 상속받은 property를 가지고 있다.

- Object의 프로퍼티에 접근할 때에 고유 프로퍼티 중에 찾고자 하는 프로퍼티가 없다면, 상속받은 prototype의 프로퍼티에서 찾는다. 이 작업은 프로퍼티를 찾거나 prototype이 null인 경우를 만날 때까지 계속된다.

```javascript
let o = {};
o.x = 1;
let p = Object.create(o);
p.y = 2;
let q = Object.create(p);
let f = q.toString();			// Object.prototype에서 상속받은 메소드 toString
q.x + q.y;								// 3 => o와 p에서 상속받은 x, y 값
```

- **프로퍼티를 정의할 때에는 상속이 되지만, 값을 새로 설정할 떄는 그렇지 않다.**

```javascript
let unitcircle = { r: 1 };
let c = Object.create(unitcircle);
c.x = 1; c.y = 1;
c.r = 2;								// c overrides its inherited property
unitcircle.r;						// 1 => prototype is not affected
```

### 6.3.3 Property Access Error

- Property 접근 표현식이 항상 값을 반환하거나 설정하지는 않는다.
- 존재하지 않는 프로퍼티에 접근할 때에 에러가 발생하지는 않는다. `undefined`를 반환한다.

```javascript
book.subtitle; 		// undefined, property does not exist
```

- 존재하지 않는 object의 프로퍼티에 접근하려고 하면 에러가 발생한다.
   **null과 undefined는 프로퍼티를 가지지 않는다**. 이런 값들의 프로퍼티에 접근할 때 에러가 발생한다.

```javascript
len len = book.subtitle.length 	// TypeError!! undefined doesn't have length 
```

- 프로퍼티 접근 표현식은 `.`의 왼쪽이 null 혹은 undefined일떄 실패한다. 

  - 예를 들어 `book.author.surname`이라는 표현식을 쓸 때에 `book`, `book.author`일 실제 정의된 값인지 확실하지 않을때 주의해야한다.

  - 이러한 문제를 해결하기 위한 두가지 방법은 아래와 같다.

```javascript
// A vorbose and explict technique
let surname = undefined;
if (book) {
  if (book.author) {
    surname = book.author.surname;
  }
}

// A concise and idilmatic alternative to get surname or null or undefined
surname = book && book.author && book.author.surname
let surname = book?.author?.surname 
```

- `null`이나 `undefined`에 프로퍼티를 설정하려고 해도 `TypeError`가 발생한다. 
  다른 값들에 프로퍼티를 설정하는 것 역시 항상 성공하는 것은 아니다.
  - read-only 이거나 설정할 수 없는 프로피티일 경우가 있다. 그리고 몇몇 object는 새로운 프로퍼티의 추가를 허용하지 않는다.
  - Strict mode에서 프로퍼티 설정 실패는 TypeError를 발생시킨다. Strict mode가 아닐때는 프로퍼티 설정 실패는 조용히 일어난다.
- 아래의 환경에서 `object o` 에 `property p`를 설정하는 것이 실패한다.
  - `o`가 `read-only` 고유 프로퍼티 `p`를 가질 때
  - `o`가 가진 상속된 프로퍼티 `p`가 `read-only`일 때
  - `o`가 고유 프로퍼티 `p`를 가지지 않을 떄: 상속된 프로퍼티 `p`도 없고, `o`의 `extensible` 속성이 `false`일때. `extensible`이 `true`일 때만 새로운 프로퍼티를 정의할 수 있다.



## 6.4 Deleting Properties

> Delete operator는 프로퍼티의 값을 지우는 것이 아니라 프로퍼티 자체를 삭제한다.

```javascript
delete book.author;					// book object에 author 프로퍼티는 더이상 존재하지 않는다.
delete book["main title"];	// "main title" 프로퍼티도 이제 존재하지 않는다.
```

- `delete` operator는 고유 프로퍼티만 삭제할 수 있고, 상속된 프로퍼티의 삭제는 불가능하다. 
  상속된 프로퍼티를 삭제하고 싶다면, prototype object에서 해당 프로퍼티를 삭제해야 한다. 이 동작은 prototype을 상속받는 모든 object에 영향을 준다.
- `delete` operator는 삭제가 성공했을 때, 삭제 동작이 아무런 효과가 없을 때 (property가 존재하지 않는 경우) 모두 `true`로 평가된다.
- `delete` operator는 뒤에 오는 표현식이 프로퍼티 접근 표현식이 아니여도 `true`로 평가된다.

```javascript
let o = {x: 1};
delete o.x						// true	=> delete property x
delete o.x						// true => does nothing. property x does not exist.
delete o.toString			// true => toString is not own property
delete 1							// true => nonsense, but true
```

- `delete` operator는 `configurable` 속성이 `false`인 프로퍼티를 삭제하지 않는다.
- 변수선언이나 함수 선언으로 만들어진 전역 객체 프로퍼티와 같이 내장 객체의 특정 프로퍼티들은 값을 변경할 수 없다. (`non-configurable`).

- Strict mode에서 non-configurable 프로퍼티를 삭제하려고 할 때 `TypeError`를 발생시킨다. Non-Strict mode에서는 `false`로 평가된다.

```javascript
// Strict Mode에서는 false를 반환하는 대신 TypeError를 발생시킨다.
delete Object.prototype		// false => property is non-configurable
var x = 1;								// global variable x
delete globalThis.x				// false => can't delete
function f() {};					// global function
delete globalThis.f				// false
```

- non-strict mode에서는 전역 객체의 프로퍼티를 삭제할 때 `globalThis`를 생략할 수 있다. strict mode 에서는 생략하면 syntaxError 발생한다.

```javascript
let globalThis.x = 1;
delete x					// true in non-strict mode. SyntaxError in strict mode
delete globalThis.x		// strict mode에서는 globalThis를 써줘야한다.
```



## 6.5 Testing Properties

- 객체에 해당 이름의 프로퍼티가 존재하는지 확인이 필요한 경우가 있다.
- `in` operator나 `hasOwnProperty()`, `propertyIsEnumerable()`메서드를 사용하거나, 단순히 프로퍼티에 접근하는 방법으로 확인할 수 있다.

- `in` operator사용은 연산자 왼쪽에 문자열로 된 프로퍼티의 이름이 들어간다.

```javascript
let o = { x: 1 };
"x" in o		// true
"y" in o		// false
"toString" in o		// true : 상속받은 프로퍼티
```

- `hasOwnProperty()`메서드는 주어진 프로퍼티가 해당 객체에 존재하는지를 검사한다. **상속받은 프로퍼티의 경우 `false`를 반환한다**

```javascript
let o = { x: 1 };
o.hasOwnProperty("x");				// true
o.hasOwnProperty("y");				// false
o.hasOwnProperty("toString");	// false : 상속받은 프로퍼티는 false 반환
```

- `propertyIsEnumerable()` 메서드는 고유 프로퍼티이며, 열거할 수 있는 프로퍼티(`enumerable` 속성이 true)인 경우에 true를 반환한다.

```javascript
let o = { x: 1 };
o.propertyIsEnumerable("x");				// true
o.propertyIsEnumerable("toString");	// false : 상속받은 프로퍼티
Object.prototype.propertyIsEnumerable("toString");	// false : 열거할 수 없는 프로퍼티
```

- 프로퍼티에 직접 접근하는 방법

```javascript
let o = { x: 1 };
o.x !== undefined;				// true
o.y !== undefined;				// false
o.toString !== undefined;		// true
```

- 프로퍼티가 존재하지만 값이 undefined인 경우

```javascript
let o = { x: undefined };
o.x !== undefined;				// false : o.x는 존재하지만 값이 undefined일 경우
o.y !== undefined;				// false
"x" in o		// true
"y" in o		// false
```

