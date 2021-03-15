# Chapter 8. Functions (3)

### 8.7.4 The call() and apply() Methods

- 다른 객체의 메소드인 것처럼 `call()`과 `apply()`는 함수를 간접적으로 호출할 수 있게 한다.
- `call()`과 `apply()`의 첫 번째 인자는 호출되는 함수객체이다. 이 인자는 호출 컨텍스트이고 함수 바디의  `this`의 값이 된다.
- 객체 `o`의 메소드로서 함수 `f()`를 호출하기 위해 (전달되는 인자 없이), `call()`과 `apply()`를 사용할 수 있다.

```javascript
f.call(o);
f.apply(o);
```

- 위의 두 코드 라인은 아래의 코드와 비슷하다.

```javascript
o.m = f; // Make f a temporary method of o.
o.m(); // Invoke it, passing no arguments.
delete o.m; // Remove the temporary method.
```

- Arrow function은 정의된 컨텍스트의 `this` 값을 상속한다. 이 this는 `call()`과 `apply()`메소드로 overriden 될 수 없다. 만약 arrow function에서 이 메소드들을 부른다면, 첫번째 인자는 무시된다.
- `call()`의 첫 번째 호출 컨텍스트 인자 뒤의 인자들은 함수로 전달되는 값들이다.  (이 인자들은 arrow function에서 무시되지 않는다.)
- 예를 들어 함수 `f()`에 두개의 숫자를 전달하고 객체 o의 메소드처럼 호출하는 것은 아래처럼 쓸 수 있다.

```javascript
f.call(o, 1, 2);
```

- `apply()`메소드는 `call()`과 비슷한데 인자를 배열로 전달하는 것이 다르다.

```javascript
f.apply(o, [1,2]);
```

- 만약 임의의 숫자의 인자들을 허락하는 함수가 정의한다면,  `apply()` 메소드로 임의의 길이의 배열 컨텐츠를 전달하는 함수를 호출할 수 있다.
- ES6이후 spread operator(`...`)를 사용해 임의의 숫자를 전달할 수 있지만, ES5에서는 apply() 메소드를 대신 사용한다.

```javascript
// spread operator 사용하지 않고 배열의 가장 큰 숫자를 찾는 방법으로 
// Math.max() 함수에 apply()로 배열을 전달해 사용할 수 있다.
let biggest = Math.max.apply(Math, arrayOfNumbers);
```

- 아래 예제에서 정의한 trace() 함수는 8.3.4의 timed() 함수와 비슷하지만 함수 대신 메소드로 작동한다.
- 이 예제에서는 apply() 메소드를 spread operator대신 사용하고 있고, 같은 인자를 가진 wrapped method와 같은 this 값을 wrapper method로 호출할 수 있다. 

```javascript
// Replace the method named m of the object o with a version that logs
// messages before and after invoking the original method.
function trace(o, m) {
  let original = o[m]; // Remember original method in the closure.
  o[m] = function(...args) { // Now define the new method.
    console.log(new Date(), "Entering:", m); // Log message.
    let result = original.apply(this, args); // Invoke original.
    console.log(new Date(), "Exiting:", m); // Log message.
    return result; // Return result.
  };
}
```

```javascript
let o = {
    'test': function(...args) {
      console.log('method! m in object o')
      const mapped_args = args.map(x => x + 1)
      return mapped_args
    }
}
let i = trace(o, 'test')	
let i_closure = o['test'](1, 2, 3)
/* 
2021-03-14T06:06:05.077Z Entering: test
method! m in object o
2021-03-14T06:06:05.083Z Exiting: test
i_closure:  [ 2, 3, 4 ]
*/
```



### 8.7.5 The bind() Method

- `bind()`메소드는 함수와 객체를 묶는 목적으로 사용한다.
- 함수 f의 bind() 메소드에 객체 o를 전달하면, 이 메소드는 새로운 함수를 반환한다. 이 새로운 함수를 호출하면 객체 o의 메소드로서 함수 f를 호출한다. 

```javascript
function f(y) { return this.x + y; } // This function needs to be bound
let o = { x: 1 }; // An object we'll bind to
let g = f.bind(o); // Calling g(x) invokes f() on o
g(2) // => 3
let p = { x: 10, g }; // Invoke g() as a method of this object
p.g(2) // => 3: g is still bound to o, not p.
```

- Arrow function은 정의된 환경의 `this` 값을 상속하고 그 값은 bind()로 overriden 될 수 없다. 즉 위의 코드가 만약 arrow function으로 정의된다면 binding이 작동하지 않는다.
- `bind()` 메소드는 partial application을 구현한다.  => `bind()`에 전달하는 인자 중 첫번째 이후 모든 인자는 `this`값과 함께 바인딩 된다.
- `bind()`의 Partial application 기능은 arrow function과 함께 작동한다. 
- Partion application은 함수형 프로그래밍의 일반적인 테크닉으로 `currying`이라고 불린다.

```javascript
let sum = (x,y) => x + y; // Return the sum of 2 args
let succ = sum.bind(null, 1); // Bind the first argument to 1
succ(2) // => 3: x is bound to 1, and we pass 2 for the y argument
function f(y,z) { return this.x + y + z; }
let g = f.bind({x: 1}, 2); // Bind this and y
g(3) // => 6: this.x is bound to 1, y is bound to 2 and z is 3
```

### 8.7.6 The toString() Method

- Javascript의 모든 객체들 처럼 함수도 `toString()`메소드를 가지고 있다. 
- ECMAScript 명세는 이 메서드가 함수 선언문 다음에 나오는 문자열을 반환하라고 요구하고 대부분의(not at all) `toString()`메소드는 함수의 전체 소스코드를 반환한다.

### 8.7.7 The Function() Constructor

- 함수는 객체이다. => `Function()` 생성자로 새로운 함수를 생성할 수 있다.

```javascript
const f = new Function("x", "y", "return x*y;");
// const f = function(x, y) { return x*y; };
```

- 함수 생성자는 임의 개수의 문자열 인자를 가질 수 있다. 
  마지막 인자는 함수 바디의 텍스트이고, 세미콜론으로 구분된 자바스크립트 구문을 포함할 수 있다.
  생성자의 다른 모든 인자들은 함수의 parameter 이름(문자열) 이다.
  만약 인자가 없는 함수를 정의할 때 생성자에 함수 바디만 전달하면 된다.

- 함수 생성자는 생성하는 함수의 이름을 지정하지 않고, 함수 리터럴과 마찬가지로 익명의 함수를 만든다.
- 함수 생성자 `Function()`을 이해하는데 중요한 포인트
  - `Function()` 생성자는 함수가 런타임동안 동적으로 생성되고 컴파일되는 것을 허락한다.
  - `Function()` 생성자는 호출 될 때마다 함수 바디를 파싱하고 함수 객체를 생성한다. 
    => 루프나 자주 호출되는 함수를 생성자로 만드는 것은 비효율적이다.
    이와 반대로 nested function이나 함수 정의 표현식은 루프내에 있더라도 매번 재컴파일 되지 않는다.
  - **생성자로 만들어진 함수는 `lexical scope`를 사용하지 않는다.** 함수 생성자가 생성한 함수는 항상 최상위 레벨의 함수로 컴파일 된다. 

```javascript
let scope = "global";
function constructFunction() {
  let scope = "local";
  return new Function("return scope"); // Doesn't capture local scope!
}
// This line returns "global" because the function returned by the
// Function() constructor does not use the local scope.
constructFunction()() // => "global"
```

- 함수 생성자는 global scope에서 동작하는 `eval()`과 같다. => `eval()`은 자신이 수행되는 유효범위에서 (own private scope)에서 새로운 변수와 함수를 정의한다.

## 8.8 Functional Programming

- 함수를 객체로 취급한다는 말은 함수형 프로그래밍 기법을 사용할 수 있다는 의미다.
- `map()`과 `reduce()`같은 배열 메소드는 함수형 프로그래밍에 적합한 구조를 가진다.

### 8.8.1 Processing Arrays with Functions

- 숫자 배열로 평균과 표준편차를 계산하는 것을 비함수형 프로그래밍에서는 아래와 같이 할 수 있다.

```javascript
let data = [1,1,3,5,5]; // This is our array of numbers
// The mean is the sum of the elements divided by the number of elements
let total = 0;
for(let i = 0; i < data.length; i++) total += data[i];
let mean = total/data.length; // mean == 3; The mean of our data is 3
// To compute the standard deviation, we first sum the squares of
// the deviation of each element from the mean.
total = 0;
for(let i = 0; i < data.length; i++) {
  let deviation = data[i] - mean;
  total += deviation * deviation;
}
let stddev = Math.sqrt(total/(data.length-1)); // stddev == 2
```

- 위와 같은 계산을 `map()`과 `reduce()`를 사용한 함수형 프로그래밍으로 다음처럼 작성한다.

```JavaScript
// First, define two simple functions
const sum = (x,y) => x+y;
const square = x => x*x;
// Then use those functions with Array methods to compute mean and stddev
let data = [1,1,3,5,5];
let mean = data.reduce(sum)/data.length; // mean == 3
let deviations = data.map(x => x-mean);
let stddev = Math.sqrt(deviations.map(square).reduce(sum)/(data.length-1));
stddev // => 2
```

- 비함수형으로 작성한 코드와 함수형으로 작성한 코드가 다르게 보이지만, 여전히 객체의 메소드를 호출하고 있기 때문에 객체 지향 컨벤션이 남아있다고 할 수 있다.
- 함수형 버전의 `map()`과 `reduce()`메소드를 작성해보자.

```javascript
const map = function(a, ...args) { return a.map(...args); };
const reduce = function(a, ...args) { return a.reduce(...args); };
```

- 이렇게 정의한 `map()`과 `reduce()`를 사용하여 평균과 표준현차를 계산하는 코드를 다음과 같이 작성할 수 있다.

```javascript
const sum = (x,y) => x+y;
const square = x => x*x;
let data = [1,1,3,5,5];
let mean = reduce(data, sum)/data.length;
let deviations = map(data, x => x-mean);
let stddev = Math.sqrt(reduce(map(deviations, square), sum)/(data.length-1));
stddev // => 2
```



### 8.8.2 Higher-Order Functions

- `Higher-order function`은 하나 이상의 함수를 인자로 받아서 새로운 함수를 반환하는 함수이다.

```javascript
// This higher-order function returns a new function that passes its
// arguments to f and returns the logical negation of f's return value;
function not(f) {
  return function(...args) { // Return a new function
    let result = f.apply(this, args); // that calls f
    return !result; // and negates its result.
  };
}
const even = x => x % 2 === 0; // A function to determine if a number is even
const odd = not(even); // A new function that does the opposite
[1,1,3,5,5].every(odd) // => true: every element of the array is odd
```

- 이 `not()` 함수는 함수인자를 가지고 새로운 함수를 반환하기 때문에  `higher-order function`이다.

- 다른 예로 `mapper()`함수를 살펴보면, 함수 인자를 가지고 하나의 배열을 다른 배열에 매핑하는 함수를 반환하는 함수이다. 
  이 함수는 앞에서 정의한 `map()`함수를 사용하는데 두 함수가 어떻게 다른지 이해하는 것이 중요하다.

```javascript
// Return a function that expects an array argument and applies f to
// each element, returning the array of return values.
function mapper(f) {
	return a => map(a, f);
}
const increment = x => x+1;
const incrementAll = mapper(increment);
incrementAll([1,2,3]) // => [2,3,4]
```

- 다음 예는 더 일반적인 예제로, 두개의 함수 `f`, `g`를 가지고 `f(g())`를 계산하는 새로운 함수를 반환하는 예제이다.

```javascript
// Return a new function that computes f(g(...)).
// The returned function h passes all of its arguments to g, then passes
// the return value of g to f, then returns the return value of f.
// Both f and g are invoked with the same this value as h was invoked with.
function compose(f, g) {
  return function(...args) {
    // We use call for f because we're passing a single value and
    // apply for g because we're passing an array of values.
    return f.call(this, g.apply(this, args));
  };
}
const sum = (x,y) => x+y;
const square = x => x*x;
compose(square, sum)(2,3) // => 25; the square of the sum
```

### 8.8.3 Partial Application of Functions

- 앞에서 살펴본 `bind()`메소드는 지정된 컨텍스트와 인지 집합을 가지고 함수 f를 호출하는 새로운 함수를 반환한다.

  => 함수를 객체에 바인당하며 부분적으로 인자를 적용한다는 것이다.

- `bind()` 메소드는 부분적으로 인자들을 왼쪽에 적용하는데, `bind()`를 통해 전달한 인자는 전달받은 함수의 인자 리스트의 첫번째에 위치한다는 뜻이다.
  이는 부분적으로 인자를 오른쪽으로 적용하는 것 역시 가능하다.

```javascript
// The arguments to this function are passed on the left
function partialLeft(f, ...outerArgs) {
  return function(...innerArgs) { // Return this function
    let args = [...outerArgs, ...innerArgs]; // Build the argument list
    return f.apply(this, args); // Then invoke f with it
  };
}
// The arguments to this function are passed on the right
function partialRight(f, ...outerArgs) {
  return function(...innerArgs) { // Return this function
    let args = [...innerArgs, ...outerArgs]; // Build the argu0]ment list
    return f.apply(this, args); // Then invoke f with it
  };
}
// The arguments to this function serve as a template. Undefined values
// in the argument list are filled in with values from the inner set.
function partial(f, ...outerArgs) {
  return function(...innerArgs) {
    let args = [...outerArgs]; // local copy of outer args template
    let innerIndex=0; // which inner arg is next
    // Loop through the args, filling in undefined values from inner args
    for(let i = 0; i < args.length; i++) {
    	if (args[i] === undefined) args[i] = innerArgs[innerIndex++];
    }
    // Now append any remaining inner arguments
    args.push(...innerArgs.slice(innerIndex));
    return f.apply(this, args);
  };
}
// Here is a function with three arguments
const f = function(x,y,z) { return x * (y - z); };
// Notice how these three partial applications differ
partialLeft(f, 2)(3,4) // => -2: Bind first argument: 2 * (3 - 4)
partialRight(f, 2)(3,4) // => 6: Bind last argument: 3 * (4 - 2)
partial(f, undefined, 2)(3,4) // => -6: Bind middle argument: 3 * (2 - 4)
```

- 이 partial application function을 사용해 이미 정의했던 함수를 가지고 다른 함수를 쉽게 정의할 수 있다.

```javascript
const increment = partialLeft(sum, 1);
const cuberoot = partialRight(Math.pow, 1/3);
cuberoot(increment(26)) // => 3
```

- Partial application function은 higher-order function과 함께 결합할 때 더 흥미롭니다.
- 앞에서 정의했던 `not()`함수를 composition과 partial application function를 사용해 정의하는 방법을 보여준 예제이다.

```javascript
const not = partialLeft(compose, x => !x);
const even = x => x % 2 === 0;
const odd = not(even);
const isNumber = not(isNaN);
odd(3) && isNumber(2) // => true
```

- 또한 composition과 partial application function를 사용해 평균과 표준편차를 구하는 함수를 함수형 스타일로 다음처럼 사용할 수 있다.

```javascript
const neg = partial(product, -1);
const sqrt = partial(Math.pow, undefined, .5);
const reciprocal = partial(Math.pow, undefined, neg(1));
// Now compute the mean and standard deviation.
let data = [1,1,3,5,5]; // Our data
let mean = product(reduce(data, sum), reciprocal(data.length));
let stddev = sqrt(product(reduce(map(data,
                                     compose(square,
                                             partial(sum, neg(mean)))),
                                 sum),
reciprocal(sum(data.length,neg(1)))));
[mean, stddev] // => [3, 2]
```

- 평균과 표준편차를 계산하는 이 코드는 함수 호출이다. =>  operator가 사용되지 않았고, 여러 괄호들로  Lisp code처럼 보이는 자바 스크립트 코드를 짤 수 있다. 
  자바스크립트 코드 스타일이 아니지만 함수형 프로그래밍 코드를 어떻게 할 수 있는지 보여주는 흥미로운 예제이다.

### 8.8.4 Memoization

- 8.4.1에서 계산결과를 캐싱하는 팩토리얼 함수를 정의했었다.
  이러란 캐싱을 `memoization`이라고 한다.
- 다음 예제는 higher-order function을 사용한 `memoize()`함수로, 인자로 함수를 받아들이고 memoization을 적용한 새로운 함수를 반환한다.

```javascript
// Return a memoized version of f.
// It only works if arguments to f all have distinct string representations.
function memoize(f) {
  const cache = new Map(); // Value cache stored in the closure.
  return function(...args) {
    // Create a string version of the arguments to use as a cache key.
    let key = args.length + args.join("+");
    if (cache.has(key)) {
      return cache.get(key);
    } else {
      let result = f.apply(this, args);
      cache.set(key, result);
      return result;
    }
  };
}
```

- `memoize()` 함수는 캐시에 사용할 새로운 객체를 생성하고 지역변수로 이 객체를 선언한다.
  => 반환되는 함수의 내부(클로저 영역)에서만 존재하는 객체이다.
- 반환되는 함수는 인자 배열을 문자열로 변환하고 그 문자열을 cache 객체의 프로퍼티 이름으로 사용한다.
- 만약 값이 캐시에 존재한다면 바로 반환하고 아니면 특정 함수를 불러 인자를 가지고 값을 계산하고 캐시하고 리턴한다.

```javascript
// Return the Greatest Common Divisor of two integers using the Euclidian
// algorithm: http://en.wikipedia.org/wiki/Euclidean_algorithm
function gcd(a,b) { // Type checking for a and b has been omitted
  if (a < b) { // Ensure that a >= b when we start
    [a, b] = [b, a]; // Destructuring assignment to swap variables
  }
  while(b !== 0) { // This is Euclid's algorithm for GCD
    [a, b] = [b, a%b];
  }
  return a;
}
const gcdmemo = memoize(gcd);
gcdmemo(85, 187) // => 17
// Note that when we write a recursive function that we will be memoizing,
// we typically want to recurse to the memoized version, not the original.
const factorial = memoize(function(n) {
  return (n <= 1) ? 1 : n * factorial(n-1);
});
factorial(5) // => 120: also caches values for 4, 3, 2 and 1
```

