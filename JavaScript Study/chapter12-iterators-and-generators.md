# Chapter 12. Iterators and Generators

- Iterable 객체와 이들과 연관된 iterator들은 ES6의 특징이다.

- 문자열이나, Set, Map 객체처럼 TypedArray를 포함한 배열은 순회가능하다.
  => 이런 자료구조의 컨텐츠들이 for/of 루프를 통해 순회가능하다는 의미이다. (5.4.4)

```javascript
let sum = 0;
for(let i of [1,2,3]) { // Loop once for each of these values
  sum += i;
}
sum // => 6
```

- Iterator는 iterable 객체를 배열 초기화 혹은 함수 호출로 확장 혹은 "spread" 하기위해 `...` 오퍼레이터와 함께 사용될 수 있다. (7.1.2)

```javascript
let chars = [..."abcd"]; // chars == ["a", "b", "c", "d"]
let data = [1, 2, 3, 4, 5];
Math.max(...data) // => 5
```

- destructuring 선언과 함께  iterator가 사용될 수 있다.

```javascript
let purpleHaze = Uint8Array.of(255, 0, 255, 128);
let [r, g, b, a] = purpleHaze; // a == 128
```

- Map객체를 순회할 때 반환되는 값은 [key, value] 쌍이다. => for/of loop에서 restructuring 선언과 함께 잘 사용된다.

```javascript
let m = new Map([["one", 1], ["two", 2]]);
for(let [k,v] of m) console.log(k, v); // Logs 'one 1' and 'two 2'
```

- 만약 쌍보다 key 혹은 value만을 순회하고자 할때는 `keys()`, `values()` 메소드를 사용할 수 있다.

```javascript
[...m] // => [["one", 1], ["two", 2]]: default iteration
[...m.entries()] // => [["one", 1], ["two", 2]]: entries() method is the same
[...m.keys()] // => ["one", "two"]: keys() method iterates just map keys
[...m.values()] // => [1, 2]: values() method iterates just map values
```

- 마지막으로 일반적으로 배열객체와 함께 사용되는 많은 built-in 함수와 생성자들은 실제로 임의의 iterator들을 대신 받아들이기 위해 쓰여졌다. (ES6 이후)

```javascript
// Strings are iterable, so the two sets are the same:
new Set("abc") // => new Set(["a", "b", "c"])
```

## 12.1 How Iterators Work

- for/of loop와 spread operator는 iterable 객체들과 함께 원활하게 동작하지만, 
  순회 동작이 어떻게 일어나고 있는지를 이해하는 것이 더 중요하다.
- Javascript에서 순회를 이해하기 위해 다음 세가지 타입을 이해해야한다.
  - 순회가능한 Array, Set, Map 같은 타입의 Iterable 객체
  - Iterable 객체 그자체 (순회를 수행하는)
  - Iteration result 객체 - 순회 각각의 스텝의 결과를 가지고 있다.

- Iterable 객체는 이터레이터 객체를 반환하는 특수한 iterator 메소드를 가지고 있는 객체이다.
- Iterator는 iteration 결과 객체를 반환하는  `next()`  메소드를 가지고 있다.

- Iteration result object는 value와 done이라는 이름의 프로퍼티를 가지고 있다.
- Iterable 객체를 순회하려면
  - 먼저 Iterator 객체를 갖기위해 iterator 메소드를 호출한다.
  - 그리고 Iterator 객체의 `next()` 메소드의 반환값의 done프로퍼티가 true일때까지 반복적으로 호출한다

- 까다로운 점은 Iterator object의 iterator 메소드는 정해전 이름이 있는 것이 아니라 Symbol `Symbol.iterator`을 이름으로 사용한다는 것이다.

- iterable object의 간단한 for/of loop는 아래처럼 어려운 방법으로도 작성되어 순회할 수 있다.

```javascript
let iterable = [99];
let iterator = iterable[Symbol.iterator]();
for(let result = iterator.next(); !result.done; result = iterator.next()) {
  console.log(result.value) // result.value == 99
}
```

- Iterator 객체의 데이터 타입은 iterable 자기 자신이다. (즉, 자기자신을 반환하는 ` Symbol.iterator`  메소드를 가진다)
- 이것은 아래처럼 부분적으로 사용하는 iterator를 원할 때 유용하다.

```javascript
let list = [1,2,3,4,5];
let iter = list[Symbol.iterator]();
let head = iter.next().value; // head == 1
let tail = [...iter]; // tail == [2,3,4,5]
```

## 12.2 Implementing Iterable Objects

- Iterable 객체는 언제나 순회가능함을 나타내는 자기자신만의 iterable 데이터 타입을 만들수 있어 ES6에서 유용하다. 
- 예제 9-2, 9-3에서 나왔던 Range 클래스는 iterable이다. 이 클래스들은 generator 함수를 사용해 그들 스스로를 iterable 하게 만들었다.

- Generator에 대한 의존 없이 iterable하게 만든 Range 클래스를 다시 한번 수행해보자.

- 클래스를 iterable하게 만들기 위해 `Symbol.iterator` 메소드를 수행해야 한다.
  - 이 메소드는 `next()` 메소드를 가진 iterator 객체를 반환한다.
  - `next()`메소드는 value 프로퍼티와 boolean done 프로퍼티를 가진 iteration result object를 반환한다.

```javascript
/*
Range는 from <= x <= to를 나타내고, has메소드를 통해 해당 값이 범위 안에 있는지를 반환한다.
Range는 iterable이고 범위안의 모든 정수를 순회한다.
*/
class Range {
  constructor (from, to) {
    this.from = from;
    this.to = to;
  }
  
  has(x) { return typeof x === "number" && this.from <= x && x <= this.to; }
  toString() { return `{ x | ${this.from} ≤ x ≤ ${this.to} }`; }
  // Range 클래스를 iterable하게 만들기
  // Note that the name of this method is a special symbol, not a string.
  [Symbol.iterator]() {
    // Each iterator instance must iterate the range independently of
    // others. So we need a state variable to track our location in the
    // iteration. We start at the first integer >= from.
    let next = Math.ceil(this.from); // next value
    let last = this.to;
    return {
      // This next() method is what makes this an iterator object.
      // It must return an iterator result object.
      next() {
        return (next <= last) // If we haven't returned last value yet
          ? { value: next++ } // return next value and increment it
        : { done: true }; // otherwise indicate that we're done.
      },
      // As a convenience, we make the iterator itself iterable.
      [Symbol.iterator]() { return this; }
    };
  }
}
for(let x of new Range(1,10)) console.log(x); // Logs numbers 1 to 10
[...new Range(-2,2)] // => [-2, -1, 0, 1, 2]
```

- 클래스를 iterable하게 만들기 위해, iterable values를 반환하는 함수를 정의하는 것이 매우 유용하다.

```javascript
// Return an iterable object that iterates the result of applying f()
// to each value from the source iterable
function map(iterable, f) {
  let iterator = iterable[Symbol.iterator]();
  return { // This object is both iterator and iterable
    [Symbol.iterator]() { return this; },
    next() {
      let v = iterator.next();
      if (v.done) {
        return v;
      } else {
        return { value: f(v.value) };
      }
    }
  };
}
// Map a range of integers to their squares and convert to an array
[...map(new Range(1,4), x => x*x)] // => [1, 4, 9, 16]
// Return an iterable object that filters the specified iterable,
// iterating only those elements for which the predicate returns true
function filter(iterable, predicate) {
  let iterator = iterable[Symbol.iterator]();
  return { // This object is both iterator and iterable
    [Symbol.iterator]() { return this; },
    next() {
      for(;;) {
        let v = iterator.next();
        if (v.done || predicate(v.value)) {
          return v;
        }
      }
    }
  };
}
// Filter a range so we're left with only even numbers
[...filter(new Range(1,10), x => x % 2 === 0)] // => [2,4,6,8,10]
```

- Iterable 객체와 iterator의 핵심 기능중 하나는 본질적으로 lazy하단 것입니다.
  => 다음 값을 구하기 위해 계산이 필요할 때, 그 값이 이 실제로 필요할 때까지 계산을 미룬다
- 예를 들어 매우 긴 문자열을 공백으로 구분된 단어들로 토큰화 하는 예제에서, 간단하게 split() 메소드를 사용해 수행할 수있다.
  하지만 이렇게 한다면 첫번째 단어를 사용하기도 전에 모든 문자열에 대한 작업이 완료되어야 한다. 결국 반환되는 모든 문자열이 포함된 배열을 위해 많은 메모리가 할당되어야 한다.
- 아래 예제는 모든 메모리를 한번에 차지하지 않고 lazy하게 문자들을 순회하도록 한다. (ES2020에서는 11.3.2에서 나왔던 matchAll() 메소드를 사용해 더 간단하게 수행할 수 있다.)

```javascript
function words(s) {
  var r = /\s+|$/g; // Match one or more spaces or end
  r.lastIndex = s.match(/[^ ]/).index; // Start matching at first nonspace
  return { // Return an iterable iterator object
    [Symbol.iterator]() { // This makes us iterable
      return this;
    },
    next() { // This makes us an iterator
      let start = r.lastIndex; // Resume where the last match ended
      if (start < s.length) { // If we're not done
        let match = r.exec(s); // Match the next word boundary
        if (match) { // If we found one, return the word
          return { value: s.substring(start, match.index) };
        }
      }
      return { done: true }; // Otherwise, say that we're done
    }
  };
}
[...words(" abc def ghi! ")] // => ["abc", "def", "ghi!"]
```

### 12.2.1 “Closing” an Iterator: The Return Method

- (server-side) javascript에서 인자로 문자열이 아닌 파일의 이름을 받아 파일을 열고 문장을 읽어 문장들로부터 단어를 순회하는 words() iterator의 변형을 가정한다.

- 파일을 열고 읽는 대부분의 운영체제와 프로그램은 읽기가 끝났을 때 그 파일을 close하는 것을 기억할 필요가 있다.
  즉, 이 words() iterator의 변형은 next() 메소드가 마지막 단어를 반환했을 때 파일을 닫아야 한다.

- 하지만 iterator들은 항상 끝까지 동작하지 않는다. : for/of 루프가 break나 return, exception으로 종료되는 것을 생각하자.
- 비슷하게, iterator가 restructuring 선언과 함께 사용될 때, next() 메소드는 특정 변수 각각의 값을 얻을 때까지만 호출된다.
  Iterator는 반환할 수 있는 더 많은 값을 가지고 있지만, 절대 요청되지는 않는다. (never be requested)

- 만약 가상의 파일안의 단어를 읽는 iterator가 항상 끝까지 동작하지 않는다면, 열린 파일을 닫아야만 하는데, 이런 이유로 iterator 객체는 next()메소드와 함께 가는 return() 메소드를 수행 할 수 있다.
- 만약 iteration이 next() 메소드가 done property가 true인 값을 반환하기 전에 멈춘다면, 인터프리터는 iterator 객체가 return() 메소드를 가지고 있는지 체크하고 만약 존재한다면 전달인자 없이 return() 메소드를 호출한다.
  -> iterator가 파일을 닫고 메모리를 release 하고 스스로를 초기하는 기회를 준다.
- return() 메소드는 iterator result object를 반환해야 한다. 이 객체의 프로퍼티들은 무시되지만, non-object 값이 반환된다면 error이다.
- for/of 루프나 spread 연산자는 Javascript의 유용한 기능이므로 API등을 만들 때 가능하다면 이것들을 사용하는 것이 좋다.
  하지만 iterable, iterator, iterator result 객체와 함께 작동한다면 프로세스를 복잡하게 만들 수 있다.
  이럴 때 generator가 커스텀 iterator를 만드는 것을 간단하게 할 수 있다.



## 12.3 Generators

- Generator는 iterator의 한 종류로 새로운 ES6 규칙에서 정의되었다. 
  순회할 값이 자료구조의 원소가 아니라 계산의 결과일 때에 유용하다.

- Generator를 만들 때, 먼저 generator 함수를 정의해야한다. Generator 함수는 문법적으로 기본 javascript 함수와 비슷하지만 키워드 함수 * 와 함께 정의된다. (function* functionName)
- Generator 함수를 호출하면, 실제로 함수 바디를 실행하는 것이 아니라 generator 객체를 반환하는 것이다.
  이 generator 객체는 iterator이다.
- next() 메소드를 호출하는 것이 generator 함수 바디가 처음부터 yield statement에 도달할 때까지 동작하게 한다. 
  yield는 ES6에서 새롭에 나온 것으로 return문과 비슷하다.
- yield문의 값은 next()로 반환되는 값이 된다.

```javascript
// A generator function that yields the set of one digit (base-10) primes.
function* oneDigitPrimes() { 
  yield 2; 
  yield 3; 
  yield 5; 
  yield 7;
}
// When we invoke the generator function, we get a generator
let primes = oneDigitPrimes();
primes.next().value // => 2
primes.next().value // => 3
primes.next().value // => 5
primes.next().value // => 7
primes.next().done // => true
// Generators have a Symbol.iterator method to make them iterable
primes[Symbol.iterator]() // => primes
// We can use generators like other iterable types
[...oneDigitPrimes()] // => [2,3,5,7]
let sum = 0;
for(let prime of oneDigitPrimes()) sum += prime;
sum // => 17
```

- 위 예제에서 function* 선언문으로 generator을 정의한다. 

```javascript
const seq = function*(from,to) {
  for(let i = from; i <= to; i++) yield i;
};
[...seq(3,5)] // => [3, 4, 5]
```

- 클래스와 객체 리터럴에서 메소드를 정의할 떄 function 키워드를 생략할 수 있다. 이런 문맥에서 generator를 정의할 때 간단하게 `*`를 메소드 ㅇ이름 앞에 표기해 사용할 수 있다.

```javascript
let o = {
  x: 1, y: 2, z: 3,
  // A generator that yields each of the keys of this object
  *g() {
    for(let key of Object.keys(this)) {
      yield key;
    }
  }
};
[...o.g()] // => ["x", "y", "z", "g"]
```

- Arrow function으로 generator 함수를 작성할 수 있는 방법이 없다는 것을 알아두자.
- generator는 interable 클래스를 쉽게 정의할 수 있는 방법이다. 12.1예제에서 정의했던 iterable 클래스를 *를 사용해 훨씬 짧게 정의할 수 있다.

```javascript
*[Symbol.iterator]() {
  for(let x = Math.ceil(this.from); x <= this.to; x++) yield x;
}
```

### 12.3.1 Generator Examples

- Generator는 계산을 통해 값을 만들어낼 때 더 흥미롭다.
- 아래는 피보나치 수를 만들어내는 generator 함수이다.

```javascript
function* fibonacciSequence() {
  let x = 0, y = 1;
  for(;;) {
    yield y;
    [x, y] = [y, x+y]; // Note: destructuring assignment
  }
}
```

- fibonacciSequence() generator 함수는 무한루프를 돌며 return값 없이 값을 영원히 생산한다.
- 만약 이 generator가 ... spread 연산자와 함께 사용된다면, 메모리가 소진될 때까지 그리고 프로그램이 crash될때까지 루프를 돌 것이다. 

```javascript
// Return the nth Fibonacci number
function fibonacci(n) {
  for(let f of fibonacciSequence()) {
    if (n-- <= 0) return f;
  }
}
fibonacci(20) // => 10946
```

- 이러한 무한한 generator는 `take()`와 함께 더 유용해질 수 있다.

```javascript
// Yield the first n elements of the specified iterable object
function* take(n, iterable) {
  let it = iterable[Symbol.iterator](); // Get iterator for iterable object
  while(n-- > 0) { // Loop n times:
    let next = it.next();
    if (next.done) return;
    else yield next.value;
  }
}
// An array of the first 5 Fibonacci numbers
[...take(5, fibonacciSequence())] // => [1, 1, 2, 3, 5]
```

- 아래는 여러 원소를 인터리브하게 한 generator 함수이다.

```javascript
// Given an array of iterables, yield their elements in interleaved order.
function* zip(...iterables) {
  // Get an iterator for each iterable
  let iterators = iterables.map(i => i[Symbol.iterator]());
  let index = 0;
  while(iterators.length > 0) { 
    if (index >= iterators.length) {
      index = 0; // go back to the first one.
    }
    let item = iterators[index].next(); 
    if (item.done) { 
      iterators.splice(index, 1);
    }
    else {
      yield item.value;
      index++;
    }
  }
}
// Interleave three iterable objects
[...zip(oneDigitPrimes(),"ab",[0])] // => [2,"a",0,3,"b",5,7]
```

### 12.3.2 yield* and Recursive Generators

- 앞에서 정의했던 zip() generator는, 원소들을 인터리빙하는 것보다 interable 객체의 원소끼리 연속되게 하는 것이 더 유용할 수도 있다.

```javascript
function* sequence(...iterables) {
  for(let iterable of iterables) {
    for(let item of iterable) {
      yield item;
    }
  }
}
[...sequence("abc",oneDigitPrimes())] // => ["a","b","c",2,3,5,7]
```

- 다른 iterable 객체의 원소로 생성하는 프로세스는  generator 함수에서 일반적이므로 ES6는 이를 위한 특별한 문법을 가진다.
-  yield* 키워드는 하나의 값을 yielding 하는 것보다 이터러블 객체를 순회하며 yield 하기위해 사용된다.

```javascript
function* sequence(...iterables) {
  for(let iterable of iterables) {
    yield* iterable;
  }
}
[...sequence("abc",oneDigitPrimes())] // => ["a","b","c",2,3,5,7]
```

- 배열의 forEach() 메소드는 배열을 순회하는 더 우아한 방법으로 아래처럼 사용하려 시도할 수 있다.

```javascript
function* sequence(...iterables) {
  iterables.forEach(iterable => yield* iterable ); // Error
}
```

- 하지만 이 방법은 작동하지 않는다. yield와 yield*는 generator 함수 안에서만 사용할 수 있는데 위 코드의 중첩된 arrow function은 regular fucntion이기 때문에 yield가 허용되지 않는다.

- yield* generator와 함께 수행되는 iterable을 포함한 모든 iterable object와  같이 사용될 수 있다. 
- 이것은  yield* 가  recursive generator를 정의하는 것을 허용함을 의미한다. 이 기능을 사용해 간단한 비재귀 iterator를 넘어서 재귀적으로 정의된 트리 구조도 사용할 수 있다.