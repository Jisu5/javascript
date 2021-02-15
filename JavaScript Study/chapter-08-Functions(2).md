# Chapter 8 Functions (2)

## 8.5 Functions as Namespaces

- 함수 안에서 선언한 변수는 함수 바깥에서는 볼 수 없다.  
  => 전역 namescape를 어지럽히지 않고 임시 namespace에서 변수를 정의하는 것처럼 사용할 수 있다.
- 다수의 javascript프로그램에서 사용하고자 하는 임시 결과값을 저장하는 변수를 가지는 코드가 있을 때, 이 변수가 다양한 프로그램에서 사용되며 conflict를 일으킬 수 있다. 
  => 코드 덩어리를 함수로 정의해, 그 함수를 호출하는 방식으로 문제를 해결할 수 있다.

```javascript
function chunkNamespace() {
  // Chunk of code goes here
  // Any variables defined in the chunk are local to this function
  // instead of cluttering up the global namespace
}
chunkNamespace();
```

- 이 코드는 `chunkNamespace`라는 하나의 전역변수만 정의한다. 
  하나의 프로퍼티(함수)만 정의하는 것도 과하다면, 익명함수를 정의하고 호출하는 단일 표현식을 이용할 수도 있다.

```javascript
(function() {
  // Chunk of code goes here
}());	// End the function literal and invoke it now!
```

- 위 표현식은 관용적으로 자주 사용되는 표현식이며, '즉시실행함수(Immediately Invoked Function Expression)'이라고 불린다.
- 함수 앞의 시작괄호`(`는 반드시 필요하다. 만약 시작괄호가 없다면 javascript interpreter가 `function`키워드를 함수 선언문으로 해석하기 때문이다. 시작 괄호를 사용하면 interpreter가 함수 선언문이 아닌 함수 정의 표현식으로 올바르게 인식한다.
- Namespace로 사용되는 함수는 그 namespace의 변수를 사용하는 하나 이상의 함수를 정의해 사용할 때 유용하다. => `closures`



## 8.6 Closures

- Javascript는 `lexical scope`를 사용한다. => 함수가 호출되는 시점이 아닌 정의되는 시점에서의 variable scope에서 실행된다.
- `lexical scope`를 구현하기 위해, javascript의 함수 객체는 함수의 코드와 함수가 정의되는 시점의 scope의 참조도 포함하고 있다.
- 함수 객체와 variable binding된 scope를 합쳐 `closure`라고 부른다.
- 기술적으로 모든 javascript 함수는 clousure이다. 
- 대부분의 함수는 정의와 호출이 같은 scope에서 일어난다. 이 때는 closure라는 개념이 중요하게 작동하지 않는다. 
  하지만 호출과 정의가 다른 scope에서 일어날 때는 closure가 중요하다. (보통 중첩된 함수)

```javascript
// lexical scoping rules for nested function
let scope = "global scope";
function checkscope() {
  let scope = "local scope";
  function f() {return scope;}
  return f();
}
checkscope();	// => "local scope"
```



```javascript
let scope = "global scope";
function checkscope() {
  let scope = "local scope";
  function f() {return scope;}
  return f;
}
let s = checkscope()();	// => "local scape"
// ()이 checkscope 안에서 바깥으로 옮겨짐
```

- 8.4.1에서 나온 `uniqueInteger` 함수는 다음에 반환할 값을 유지하기 위해  함수 자신의 프로퍼티를 사용했다. 이 방식의 단점은 버그 혹은 악성 코드가 counter를 초기화하거나 non-integer값으로 바꿀 수 있어, 함수가 `unique`나 `integer`라는 자신의 규약을 위반할 수 있다는 점이다.
  => closure는 함수의 지역변수를 포착해 내부 상태로 사용할 수 있어 위의 단점을 보완할 수 있다.

```javascript
let uniqueInteger = (function() {
  let counter = 0;	// private state
  return funtion() { return counter++; }
}());	// define and invoke function => return value is assigned to uniqueInteger
uniqueInteger() // => 0
uniqueInteger() // => 1
```

- 위에서 counter같은 내부 변수는 여러 클로저가 공유할 수 있다. 

```javascript
function counter() {
  let n = 0;
  return {
    count: function() { return n++ },
    reset: function() { n = 0 }
  };
}
let c = counter(), d = counter();
c.count()	// 0
d.count()	// 0
c.reset()
c.count()	// 0
d.count()	// 1
```

- closure 기법과 getter/setter 프로퍼티를 결합할 수 있다.

```javascript
function counter(n) {
  return {
    // getter method returns and increments private counter var
    get count() { return n++; }
    // property setter doesn't allow the calue of n to decrease
    set count(m) {
      if (m>n) n = m;
      else throw Error("count can only be set to a larger value")
    }
  };
}
let c = counter(1000);
c.count	// => 1000
c.count	// => 1001
c.count = 2000
c.count	// => 2000
c.count	// => !Error
```

- Closure 기법을 사용해 내부 상태를 공유하는 방법을 일반화 한 Example 8.2

```javascript
// Example 8.2
// 이 함수는 프로퍼티 접근 메서드를 객체 o의 프로퍼티에 특정 이름으로 추가한다.
// 메서드 이름은 get<name>과 set<name>이다.
// 만약 predicate 함수가 제공된다면, setter 메서드는 전달된 인자를 저장하기 전에 유효성 테스트에 predicate 함수를 사용한다.
// predicate가 false를 반환하면, setter 메서드는 예외를 발생시킨다.

// getter/setter로 제어하는 값이 객체 o에 저장되지 않는 점을 주의해야한다.
// 대신, 그 값은 함수의 지역변수에 저장된다.
// 또한, getter/setter 메서드는 지역적으로 정의되기 때문에 지역변수에 접근할 수 있다.

function addPrivateProperty(o, name, predicate) {
  let value;	// property value
  
  o[`get${name}`] = function() { return value; }
  o[`set${name}`] = function(v) {
    if (predicate && !predicate(v)) {
      throw new TypeError(`set${name}: invalid value ${v}`);
    } else {
      value = v;
    }
  }
}

let o = {};

addPrivateProperty(o, "Name", x => typeof x === "string");

o.setName("Frank")
o.getName()	// => Frank
o.setName(0)	// => !TypeError
```

- Closure들이 공유해서는 안되는 변수에 접근해 공유하는 실수를 발견하는 것도 중요하다.

```javascript
function constfunc(v) { return () => v; }

let funcs = [];
for (var i = 0; i < 10; i++) funcs[i] = constfunc(i);

funcs[5]()	// => return 5
```

- 위의 예제처럼 루프를 이용해 여러 클로저를 생성할 때, 클로저를 정의하는 함수 내에서 루프를 도는 것이 일반적인 실수이다.

```javascript
function constfuncs() {
  let funcs = [];
  for (var i = 0; i < 10; i++) {
    funcs[i] = () => i
  }
  return funcs;
}

let funcs = constfuncs();
funcs[5]()	// => 10
```

- `funcs`는 모두 `constfuncs` 내부 변수 i를 공유하고 있다. `constfuncs()`가 반환될 때, 변수 i의 값이 10이므로 10개의 클로저 모두가 이 값을 공유한다. => 클로저와 연관된 scope가 'live'인 것이 중요하다.
- `this`는 javascript 키워드이지 변수가 아니라는 점을 기억해야 한다. 모든 함수 호출에는 this 값이 있고, 바깥쪽 함수가 this 값을 별도의 변수로 저장하지 않으면 클로저는 바깥쪽 this 값에 접근 할 수 없다.

```javascript
const self = this;	// 중첩 함수에서 this값을 사용하기 위해 변수에 따로 저장
```





## 8.7 Function Properties, Methods, and Constructor

- 함수는 객체이기 때문에 프로퍼티, 메서드, 생성자 (`Function()`)를 가질 수 있다.

### 8.7.1 The Length Property

- 읽기전용이며, 함수를 정의할 때 명시한 인자 개수(arity)를 반환한다. 
- `rest parameter`를 가지고 있다면, 그것은 포함되지 않는다.

### 8.7.2 The name Property

- 읽기전용. 함수의 이름 (함수가 정의될때의 이름 혹은 변수의 이름, 이름이 없는 함수 표현식이 저장된 프로퍼티)을 반환한다.
- 디버깅이나 에러메세지를 작성할 때 유용하다.

### 8.7.3 The prototype Property

- `arrow function`을 제외한 모든 함수는 prototype 프로퍼티를 가진다. 이 프로퍼티는 prototype object를 참조한다.
- 모든 함수는 서로 다른 프로토타입 객체를 가지고 있다.
- 함수가 생성자로 사용될 때, 새로 생성된 객체는 prototype object를 상속받는다.



