# Chapter 13. Asynchronous JavaScript (2)

## 13.3 async and await

- ES2017에서 소개한 `async, await` 키워드는 Promise사용을 드라마틱하게 단순화시키고, 
  네트워크 response 혹은 비동기 이벤트를 기다리는 동안 블락하는 동기적인 코드처럼 Promise-based의 비동기적 코드를 작성할 수 있게 했다.
- 어떻게 Promise가 동작하는지를 이해하는 것은 여전히 중요하지만, async와 await를 사용함으로써 Promise의 복잡성 대부분은 사라진다.

- 비동기적 코드는 동기적 코드처럼 return value나 throw an exception을 할 수 없다.
  완전히 수행된 Promise의 값은 동기적 함수의 반환값과 같고, rejected Promise의 값은 동기적 코드에서의 예외처리 값(.catch()메소드)과 같다.
- `async - await`는 Promise-based 코드의 효율성을 취하고 Promise를 숨김으로써,  비효율적이고 blocking된 동기코드만큼 비동기 코드를 읽기 쉽고, 추론하기 쉽게 할 수 있다.

### 13.3.1 await Expressions

- `await` 키워드는 Promise를  return value 나 thrown exception로 되돌린다. 
- Promise 객체 p가 주어질 때, await p 표현식은 p값이 정해질 때까지 기다린다. 
  만약 p가 완전히 수행되면, await p의 값은 p이 수행된 값이 된다. 반면에 만약 p가 reject 되면,  rejection 값을 throw 한다.
- 보통, Promise를 가지는 변수에 `await`를 사용하는 것이 아니라 Promise를 반환하는 함수 호출 전에 사용한다.

```javascript
let response = await fetch("/api/user/profile");
let profile = await response.json();
```

- 이것은 `await` 키워드가 프로그램자체를 block하고 특정 Promise가 완료되기 전까지 아무것도 하지 않는것을 야기하지 않는 옳은 방법을 이해하는 데 중요하다.
- 코드는 비동기적으로 유지되고, `await`를 사용하는 코드들 자체도 비동기적이다.

### 13.3.2 async Functions

- `await`를 사용하는 어떤 코드든 비동기적이기 때문에, `async`키워드로 선언된 함수 안에서만 `await`를 사용할 수 있다는 강력한 규칙이 있다.
- 앞서 나왔던 `getHighScore()` 함수를 `async - await`로 재 작성한 코드가 있다.

```javascript
async function getHighScore() {
  let response = await fetch("/api/user/profile");
  let profile = await response.json();
  return profile.highScore;
}
```

- 함수를 `async`로 선언하는 것은 반환 값이 함수 바디에 Promise 관련된 코드가 있던 없던 Promise일 것이라는 것을 의미한다.

- 만약 `async` 함수가 정상적으로 반환한다면, 함수로 반환되는 Promise 객체는 resolve된 return value이고,
  throw exception한다면 반환되는 Promise 객체가 그 exception과 함께 reject된 것이다.

- `getHighScore()`  함수가 async로 선언되었기에, Promise를 반환한다. 
  그리고 Promise를 반환하였기 때문에, await 키워드와 함께 사용할 수 있다.

```javascript
displayHighScore(await getHighScore());
```

- 하지만, 위의 코드는 **또 다른 `async`함수** 안에서만 작동한다는 것을 기억해야한다.
- `async`함수 안에서 원하는 만큼 `await` 표현식을 중첩시켜 사용할 수 있다.

- 하지만 만약 어떤 이유로 top level에 있거나 `async`가 아닌 함수안에 있다면, `await`를 사용할 수 없고 반환된 Promise를 정석정인 방법으로 다루어야 한다.

```javascript
getHighScore().then(displayHighScore).catch(console.error);
```

- `async` 키워드는 어떤 종류의 함수던 함께 사용할 수 있고, 함수 키워드와 함께 statement 혹은 expression 처럼 작동한다.

- arrow function 그리고 클래스, 객체 리터럴의 메소드와 함께 사용할 수 있다.

### 13.3.3 Awaiting Multiple Promises

- `async`를 사용한 getJSON 함수를 다음처럼 정의했다.

```javascript
async function getJSON(url) {
  let response = await fetch(url);
  let body = await response.json();
  return body;
}
```

- 그리고 이 함수로 두 개의 JSON 값을 fetch하는 것을 가정한다.

```javascript
let value1 = await getJSON(url1);
let value2 = await getJSON(url2);
```

- 이 코드의 문제는 불필요한 연속성이 있다는 것이다. => 첫번째 fetch가 완료될 때까지 두 번째 fetch가 시작하지 않는다.

- 만약 두 번째 URL이 첫 번째 URL에서 얻는 값에 의존하지 않는다면, 두 값을 동시에(concurrency) fetch하는 것도 가능하다.
- 동시에 수행되는 async 함수들을 await 하기 위해, `Promise.all()`을 사용할 수 있다.

```javascript
let [value1, value2] = await Promise.all([getJSON(url1), getJSON(url2)]);
```

- concurrency는 동시에 실행되는 것처럼 보인다. (Parallelism과 다르다.)
- javascript 는 싱글스레드, 멀티스레드는 공유자원에 접근할때 문제가 생긴다.  race condition 그리고 heap, 공유자원의 문제는 해결 (멀티스레드에있는). 그렇다면 asyncrounous도 작동하게 하기위해 message queue를 사용해서. 원래 싱글스레드에서는 non-blocking이 안되니까 안되는데 되게했음. Non-blocking

### 13.3.4 Implementation Details

- Async 함수가 어떻게 작동하는지 이해하기 위해서 under the hood에서 무엇이 일어나는지에 대한 생각이 도움이 될것이다.

```javascript
async function f(x) { /* body */ }
```

- 위와 같은 async 함수의 바디를 감싸 Promise를 반환하는 것처럼 생각할 수 있다.

```javascript
function f(x) {
  return new Promise(function(resolve, reject) {
    try {
      resolve((function(x) { /* body */ })(x));
    }
    catch(e) {
      reject(e);
    }
  });
}
```

- await 키워드를 이 처럼 변환해 표현하는 것이 더 어렵지만 분리된 synchronous 덩어리로 함수의 바디를 분해하는 marker로 생각하자.

- ES2017 인터프리터는 함수의 바디를 연속되는 분리된 subfunction으로 분해할 수 있고, 각각에서 얻은 것을 앞서 있는 await-marked Promise 의 then() 메소드로 전달한다.



## 13.4 Asynchronous Iteration

- 이 챕터의 시작에서 callback과 event-based 비동기에 대해 설명했고, Promise를 소개할 때 single-shot asynchronous 계산들에 유용하지만, setInterval()이나 웹브라우저의 'click' 이벤트 같은 반복적인 비동기 이벤트나 Node stream의 'data' 이벤트에는 적합하지 않다고 이야기 했다.

- 이는 단일 Promise는 연속된 비동기 이벤트에서 작동하지 않기 때문이고, 일반적인 asyn함수와 await 문 역시 이러한 것들에서는 사용할 수 없다.
- 하지만 ES2018에서 해결책을 제공하고 있다. Asynchronous iterator는 chapter 12에서 소개한 iterator들과 비슷하지만 Promise-based이고 for/of 루프의 새로운 형태인 for/await를 사용한다.

### 13.4.1 The for/await Loop

- Node 12 읽을 수 있는 스트림들을 비동기적인 iterable로 만든다.
  => 이것은 스트림에서의 연속적인 데이터 덩어리를 for/await 루프를 사용해 읽을 수 있다는 의미이다.

```javascript
const fs = require("fs");

async function parseFile(filename) {
  let stream = fs.createReadStream(filename, { encoding: "utf-8"});
  for await (let chunk of stream) {
    parseChunk(chunk); // Assume parseChunk() is defined elsewhere
  }
}
```

- 일반적인 await 표현식에서처럼, for/await 루프도 Promise-based이다.
- asynchronous iterator는 Promise를 생성하고, for/await루프는 Promise가 완전히 수행되기를 기다리고, 수행된 값을 루프 변수에 할당한다.

- 그리고 나서 다시 시작해 iterator로부터  다른 Promise 만들고, 새로운 Promise가 수행되기를 기다린다.

```javascript
const urls = [url1, url2, url3];
```

- 위의 url 배열로부터 각각의 URL로 fetch()를 호출해 Promise의 배열을 얻을 수 있다.

```javascript
const promises = urls.map(url => fetch(url));
```

- 앞서 본 것처럼 `Promise.all()`을 사용해 배열의 모든 Promise가 완전히 수행되기를 기다릴 수 있다.
- 하지만, 만약 첫번째 fetch의 결과를 최대한 빨리 얻고, 모든 URL의 fetch를 기다리지 않기를 원한다고 가정해보자.
- 배열은 iterable이기 때문에 일반적인 for/of루프를 사용해 promises 배열을 순회할 수 있다.

```javascript
for(const promise of promises) {
  response = await promise;
  handle(response);
}
```

- 이 예제 코드는 일반적인 iterator와 함께 일반적인 for/of 루프를 사용한다. 
  하지만, 이 iterator가 Promise를 반환하기 때문에 for/await를 사용해 더 간단한 코드를 작성할 수도 있다.

```javascript
for await (const response of promises) {
  handle(response);
}
```

- For/await 루프는 루프안에 await 호출을 빌드해 코드를 더 컴팩트하게 만들 수 있지만, 두 코드는 정확하게 같다.
- 중요한 점은 두 예제 모두 함수가 async로 선언되어야 작동한다는 점이다. 

### 13.4.2 Asynchronous Iterators

- Chapter 12에서 나왔던 Iterable 객체는 for/of 루프와 사용할 수 있는 것 중 하나이다.
  - `Symbol.iterator` 메소드로 정의하고, 이 메소드는 iterator 객체를 반환한다.
  - Iterator 객체는 `next()` 메소드를 가지고 있어 반복적으로 호출되며 iterable 객체의 값들을 얻을수 있다. 반환 값은 iteration result object이다.
  - Iteration result object는 value 프로퍼티와 done 프로퍼티를 가진다

- 비동기 iterator는 일반 iterator와 거의 비슷하지만 두가지 다른점이 있다.
  - 첫째, 비동기적인 iterator 객체는 `Symbol.iterator` 대신 `Symbol.asyncIterator` 메소드로 실행한다.
  - 두번째, `next()`메소드는 iterator result object를 직접 반환하는 것 대신 resolve에서 iterator result object를 가지는 Promise를 반환한다.

### 13.4.3 Asynchronous Generators

- Chapter 12에서 봤듯이, iterator를 시행하는 가장 쉬운 방법은 생성자를 사용하는 것이다. 
  비동기 iterator역시 마찬가지며, async로 선언한 생성자 함수로 시행할 수 있다.
- Aync 생성자는 async함수의 기능과 생성자의 기능을 가지고 있다.
  - 일반적인 async 함수에서처럼 await를 사용할 수있고, 일반적인 생성자에서 사용했던 것처럼 yield를 사용할 수 있다.

- yield로 가지는 값은 자동적으로 Promise로 감싸져있다. 또한 async 생성자의 문법은 `async함수`와 `function*`의 조합이다. 
  => `async function*`

- 다음은 async 생성자와 for/await 루프로 setInterval() 콜백 함수 대신 루프문법으로 interval을 가지는 코드예제이다.

```javascript
// etTimeout()을 감싸는 Promise-base 함수
function elapsedTime(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
// count가 증가하며 yield되는 async 생성자 함수 
async function* clock(interval, max=Infinity) {
  for(let count = 1; count <= max; count++) {
    await elapsedTime(interval); // wait for time to pass
    yield count;
  }
}
// async 생성자와 for/await를 사용하는 테스트함수
async function test() { // Async가 있어 for/await 사용할 수 있다.
  for await (let tick of clock(300, 100)) { // Loop 100 times every 300ms
    console.log(tick);
  }
}
```

### 13.4.4 Implementing Asynchronous Iterators

- async 생성자로 비동기 iterator를 시행하는 것 대신에  `Symbol.asyncIterator()` 메소드로 직접 객체를 정의해 시행하는 것도 가능하다.
- 아래 코드에서는 앞서 있던 예제에서의 생성자 대신 단순히 비동기적인 iterator 객체를 반환하는 clock() 함수를 재시행하며, `next()` 메소드는 Promise를 반환하지 않고 next()를 async로 선언했다.

```javascript
function clock(interval, max=Infinity) {
  // interval 대신 정해진 시간을 가지는 setTimeOut을 Promise화
  function until(time) {
    return new Promise(resolve => setTimeout(resolve, time - Date.now()));
  }
  // 비동기적인 iterable object를 반환
  return {
    startTime: Date.now(),
    count: 1,
    async next() { // next() 메소드가 이것을 iterator로 만든다.
      if (this.count > max) {
        return { done: true };
      }
      let targetTime = this.startTime + this.count * interval;
      await until(targetTime);
      return { value: this.count++ };
    },
    // 이 메소드는 this iterator 객체가 iterable임을 의미한다.
    [Symbol.asyncIterator]() { return this; }
  };
}
```

- 이 iterator-based 버전의 clock()함수는 generatorbased의 버전의 결함을 해결했다.
- 이 코드는 각각의 순회의 (현재시간 - 시작시간)으로 setTimeOut()으로 전달하는 interval을 계산한다.
- 만약 clock()을 for/await 루프와 함께 사용한다면, 이 버전은 루프의 바디를 돌면서 요구되는 시간을 계산하기 때문에 특정 interval을 더 정확하게 루프 순회를 할 것이다. 하지만 이 버전은 timing 정확도에 대해서만 해결한다.
- for/await 루프는 항상 다음 순회를 시작하기 전에 Promise가 완전히 수행되는 것을 기다린다. 
  하지만 만약 비동기 순회를 for/await루프 없이 사용한다면, 원할 때에 next() 메소드를 호출할수 있는 방법이 없다.
- generator-based 버전의 clock()에서 만약 next() 메소드를 연속적으로 세번 호출한다면, 원하지 않았던 거의 동시에 끝이나는 Promise 세개를 얻을 것이다.
  iterator-based 버전은 이러한 문제 없이 시행될 수 있다.
- 비동기 iterator의 이점은 비동기 이벤트나 데이터의 스트림을 표현하는 것을 허용한다는 것이다.
- clock()은 만들어낸 비동기 소스가 setTimeOut() 호출이기 때문에 작성하기에 아주 간단한 함수이다.
- 하지만 이벤트 핸들러의 트리거 같은 다른 비동기 소스들과 동작하게 할때에는 비동기 iterator 시행이 상당히 힘들다.
  - 보통 이벤트에 응답하는 하나의 이벤트 핸들러 함수 가지지만, iterator의 next() 메소드 호출은 각기 다른 Promise 객체를 반환해야하고, 첫번째의 Promise가 resolve되기 전에 여러번의 next() 호출이 일어나기 때문에
- 이것은 어떤 비동기 iterator 메소드든 비동기 이벤트가 발생하는 순서대로 resolve 되는 Promise의 내부 queue를 유지할 수 있어야 한다는 의미이다.
  => Promise-Queuing 동작을 `AsyncQueue` 클래스로 캡슐화하면 AsyncQueue를 기반으로 비동기 iterator를 쓰는 것이 훨씬 쉬워진다.

- `AsyncQueue` 클래스는 `enqueue()`와 `dequeue()`메소드를 가지고 있다.
  - `dequeue()` 메소드는 실제 값보다는 Promise를 반환하며, 이것은 `enqueue()` 호출 전에 `dequeue()`를 호출하는 것이 괜찮다는 의미이다.
- `AsyncQueue` 클래스 역시 비동기 iterator이고, 새로운 값이 비동기적으로 enqueue 될 때 바디가 각각 한번씩 실행되는 for/await 루프와 함께 사용되도록 의도되어있다.
  - `AsyncQueue` 는 `close()`메소드를 가지고, 값이 더이상 enqueue되어 있지 않을 때 한번 호출된다.
    닫힌 queue가 비어있다면, for/await 루프 역시 순회를 멈춘다.

- `AsyncQueue` 의 시행은 async await와 함께 사용되지 않고 Promise와 직접적으로 동작하는 것을 주의하자.

```javascript
class AsyncQueue {
  constructor() {
    this.values = []; // queue에 있지만 아직 dequeue되지 않은 값이 여기에 저장된다.
    this.resolvers = []; // Promise가 queue에 들어가기 전에 dequeue될 때, resolve 메소드가 여기에 저장된다.
    this.closed = false; // close가 실행되면 enqueue가 더이상 되지않고, 더이상 수행완료된 Promise가 반환되지 않는다.
  }
  enqueue(value) {
    if (this.closed) {
      throw new Error("AsyncQueue closed");
    }
    if (this.resolvers.length > 0) {
      const resolve = this.resolvers.shift();
      resolve(value);
    }
    else {
      this.values.push(value);
    }
  }
  dequeue() {
    if (this.values.length > 0) {
      const value = this.values.shift();
      return Promise.resolve(value);
    }
    else if (this.closed) {
      return Promise.resolve(AsyncQueue.EOS);
    }
    else {
      return new Promise((resolve) => { this.resolvers.push(resolve); });
    }
  }
  close() {
    // queue가 닫히고 나면 더이상 enqueue되지 않는다. 
    // 그렇기 때문에 모든 보류중인 Promise들을 end-of-stream를 이용해 resolve한다.
    while(this.resolvers.length > 0) {
      this.resolvers.shift()(AsyncQueue.EOS);
    }
    this.closed = true;
  }
  [Symbol.asyncIterator]() { return this; }
  next() {
    return this.dequeue().then(value => (value === AsyncQueue.EOS)
                               ? { value: undefined, done: true }
                               : { value: value, done: false });
  }
}
AsyncQueue.EOS = Symbol("end-of-stream");
```

- `AsyncQueue` 클래스가 비동기적 interation 기반으로 정의되었기 때문에, 자기자신만의 비동기 iterator를 비동기 대기열에 들어간 값으로 쉽게 만들수 있다.
- 아래는 AsyncQueue를 사용해 for/await 루프로 웹브라우저 이벤트 스트림을 다루는 것을 보여준다.

```javascript
function eventStream(elt, type) {
  const q = new AsyncQueue(); // Create a queue
  elt.addEventListener(type, e=>q.enqueue(e)); // Enqueue events
  return q;
}
async function handleKeys() {
  for await (const event of eventStream(document, "keypress")) {
    console.log(event.key);
  }
}
```

