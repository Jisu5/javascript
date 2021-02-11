# Chapter 8 Functions

## 8.1.1 Function Declaration

- 함수선언은 가장 자바스크립트에서 쉽게 함수를 만들 수 있는 방법입니다. 크게 3가지로 구성되는데 함수 이름, 인자, 선언문들로 이루어지게 됩니다.
  ```
  function distance(x1, y1, x2, y2){
      let dx = x2 - x1;
      let dy = y2 - y1;
      return Math.sqrt(dx*dx + dy*dy);
  }
  ```

1. 자바스크립트 함수 선언(Function Declaration)이 가장 중요한 점은 다음과 같습니다.

   - 함수 이름 -> 변수
   - 함수 자체 -> 변수의 값
   - 위와 같은 함수 선언문은 호이스팅(hoisting)되서, 함수를 선언하기 전에 먼저 호출을 하는것이 가능합니다.

     ```
     catName("Chloe");

     function catName(name) {
         console.log("My cat's name is " + name);
     }
     ```

   - 이 뜻은 자바스크립트 Interpretor가 코드를 실행하기전에 모든 함수선언문이 먼저 호이스팅되서 정의되는것을 뜻합니다.

2. 함수 선언에서 `return`이 없을 때
   ```
   function printprops(o){
       for(let p in o){
           console.log(`${p}: ${o[p]}\n`)
       }
   ```
   - 함수선언에서 return이 없을 경우에 호출했을 때 항상 `undefined`를 반환합니다.

## 8.1.2 Function Expressions

1. Function Exprsesion은 함수명을 생략할 수 있습니다.

   ```
   const square function(x) {return x*x;};
   ```

2. Function Declaration은 변수를 선언하고 함수를 변수에 할당하지만, Function Expression은 호출할 것이 아니라면 변수를 선언하지 않지 않아도 됩니다.
   ```
   [3,2,1].sort(function(a,b){return a-b});
   ```
   하지만 Function Expression을 호출하려면 반드시 변수에 할당을 해야 합니다.
3. Function Declaration은 Hoisting을 하지만 Function Expression은 Hosting을 하지 않습니다.

   ```
   hoisted();

   function hoisted() {
       console.log('foo');
   }


   vs

   notHoisted(); // TypeError: notHoisted is not a function

   var notHoisted = function() {
       console.log('bar');
   };
   ```

## 8.1.3 Arrow Functions

1.  Arrow Function은 다음과 같은 방법으로 구성할 수 있습니다.
    1. ES6부터 사용할 수 있는 새로운 함수 정의 방법입니다. 먼저 괄호를 이용하여 파라미터를 만들고, `=>`로 함수의 몸체를 잇는 형태로 선언할 수 있습니다.
       ```
       const sum = (x,y) => {return x + y;};
       ```
    2. 만약에 함수를 `return`이 하나의 선언문으로 나타낼 수 있다면 `return`역시 생략할 수 있습니다.
       ```
       const sum = (x,y) => x+y;
       ```
    3. 만약에 함수의 인자가 하나만 존재한다면 괄호(paranthesis)또한 생략할 수 있습니다.
       ```
       const polynomial = x => x*x + 2*x + 3;
       ```
    4. 하지만 함수의 인자가 존재하지 않는다면 반드시 괄호를 포함해야 합니다.
       ```
       const constantFunc = () => 42;
       ```
    5. 함수가 return 단 하나의 statement를 object로 가질 때, object의 정의와 함수의 block을 나타내는 curly brace가 겹치는 것을 막기 위해 괄호(paranthesis)를 사용해 선언의 모호함을 없애줍니다.
       ```
       const g = x => ({value : x});
       ```
2.  Arrow Function은 함수를 다른 함수로 넘겨줄 때 가장 이상적으로 사용되어질 수 있습니다.
    ```
    let filtered = [1, null, 2, 3].filter(x => x !== null) // [1,2,3]
    let squares = [1,2,3,4].map(x=>x*x);
    ```

## 8.2.1 Function Invocation

1. 함수가 호출될 때 먼저, 각각의 인자들이 평가되어지고, 인자들이 파라미터에 할당됩니다. 그리고 함수의 리턴값은 invocation expression의 값이 되게 됩니다.
   ```
   let total = distance(0,0,2,1) + distance(2,1,3,5);
   let probability = factorial(5)/factorial(13);
   ```
2. 함수의 호출이 `non-strict` mode일때, 일반 함수의 context(`this` value)는 `global` object입니다. `Stric-mode`일땐 호출 context는 `undefined`입니다.

   ```
   //non-stric mode
   function f1() {
       return this;
   }

   // In a browser:
   f1() === window; // true

   // In Node:
   f1() === global; // true


   ```

   ```
   //stric mode
   function f2() {
   'use strict'; // see strict mode
       return this;
   }
   f2() === undefined; // true
   ```

## 8.2.2 Method Invocation

1. 매서드는 오브젝트의 프로퍼티에 있는 함수입니다.
2. 매서드의 호출 context는 object가 됩니다. 그래서 `this` keyword로 오브젝트에 접근할 수 있습니다.
   ```
   let calculator = {
       operand1 : 1,
       operand2 : 1,
       add() {
           this.result = this.operand1 + this.operand2;
       }
   }
   calculator.add()
   calculator.result   // => 2
   ```
3. 매서드 안에 Nested Function의 호출 context는 `object`가 아니라 `global`이나 `undefined`입니다.

   ```
   let o = {
       m : function(){
           let self = this;
           this === 0          // true
           f();


           function f() {
               this === o      // false
               self === o      // true
           }
       }
   }
   ```

4. 이와 관해서, nested function을 arrow function으로 바꾸면 `this`를 선언한 context에서 `inherit`할 수 있습니다.
   ```
   const f = () => {
       this === o    //true
   }
   ```

## 8.2.3 Implicit Function Invocation

- 자바스크립트에서 함수의 호출 같지 않지만, 암묵적으로 함수가 호출되는 경우가 있습니다.
  1. `object`가 `getter`와 `setter`를 정의했다면, 해당하는 프로퍼티를 설정하거나 가져올때 암묵적으로 이 함수들이 호출됩니다.
  2. `object`가 string 컨텍스트에 사용되어 질때 (`+`연산자를 통해 스트링과 서로 붙는) `object`의 `toString()`이 호출되게 됩니다.
  3. `iterable object`가 loop를 돌 때 이와 관련한 매서드들이 호출되게 됩니다. (Chapter 12)

## 8.3.1 Optional Parameters and Defaults

- 함수가 많은 파라미터로 정의되었지만 그 보다 적은 인수로 호출되었을때 Optional Parameter를 사용할 수 있습니다.
  ```
  function getPropertyNames(o, a){
      a = a || [];
      for (let property in o) a.push(property);
      return a
  }
  let o = {x : 1},  p = {y : 2, z : 3};
  let a = getPropertyNames(o);        // a == ["x"]
  getPropertyNames(p,a)               // a == ["x", "y", "z"]
  ```
- 함수에서 앞에 있는 인자를 가지고, 뒤에서 사용할 수도 있습니다.
  ```
  const rectangle = (width, height=width*2) => ({width, height})
  rectangle(1)        // => {width:1, height:2}
  ```

## 8.3.2 Rest Parameters and Variable-Length Argument Lists

1. *Rest Parameter*를 통해 파라미터보다 더 많은 임의의 인자들을 정의할 수 있습니다.

   ```
   function max(first=-Infinity, ...rest) {
       let maxValue = first;
       for (let n of rest) {
           if (n > maxValue) {
               maxValue = n;
           }
       }
       return maxValue;
   }

   max(1, 10, 100, 2, 3, 1000, 4, 5, 6)   // => 1000
   ```

   - *Rest parameter*는 3개의 연속된 점 (`...`)을 통해 만들 수 있고, 반드시 함수 선언의 마지막 인자가 되어야 합니다.
   - *Rest parameter*는 array에 저장되서 rest parameter에 값이 됩니다.

2. *Rest Parameter*는 말그대로 `Array`입니다. 그리고 빈 인자가 들어오면 빈 배열이 되게 됩니다.

   ```
   function fun1(...theArgs) {
       console.log(theArgs.length)
   }

   fun1()         // 0
   fun1(5)        // 1
   fun1(5, 6, 7)  // 3
   ```

## 8.3.3 The Spread Operator for Function calls

1. *Spread Operator*는 값을 실제로 만들어 낼 수 없다는 점에서 연산자라기 보다는, 특수한 자바스크립트 *Syntax*로 볼 수 있습니다.
2. Rest Paramter와 Spread Operator를 함께 있을 때 유용하게 사용되어질 수 있습니다.

   ```
   function timed(f) {
       return function(...args) {
           console.log('Entering function ${f.name}`);
           let startTime = Date.now();
           try {
               return f(...args)
           }
           finally {
               console.log(`Exiting ${f.name} after ${Date.now()-startTime}ms`)
           }
       }
   }

   function benchmark(n){
       let sum = 0;
       for (let i=1; i<=n; i++) sum += i;
       return sum
   }

   timed(benchmark)(1000000)
   ```

## 8.3.5 Destructuing Function Arguments into Parameters

1. 함수가 호출될 때, 인자들은 선언된 파라미터에 할당되어집니다. 그래서, *Destructing 할당 테크닉*을 함수에서 사용할 수 있습니다.
   ```
   function vectorAdd(v1, v2) {
       return [v1[0] + v2[0], v1[1] + v2[1]];
   }
   vectorAdd([1,2], [3,4])    // => [4,6]
   ```
   *Destructuring*을 사용하면 보다 가독성이 높은 코드를 쓸 수 있습니다.
   ```
   function vectorAdd([x1, y1], [x2, y2]) {
       return [x1 + x2, y1 + y2];
   }
   vectorAdd([1,2], [3,4])    // => [4,6]
   ```
2. `object`에도 적용되어질 수 있습니다.
   ```
   function vectorMultiply({x, y}, 2) {
       return { x:x*scalar, y:y*scalar};
   }
   vectorMultiply({x : 1, y : 2}, 2)    // => {x:2, y:4}
   ```
3. `object`를 *destructure*할 때 `rest parameter`를 사용할 수 있습니다.
   ```
   function vectorMultiply({x, y, z=0, ...props}, scalar){
       return {x: x*sclar, y:y*scalar, z:z*scalar, ...props};
   }
   vectorMultiply({x:1, y:2, w:-1}, 2)
   ```

## 8.3.6 Argument Types

1. 자바스크립트는 `Type`들이 필요에 의해 자동으로 변환하므로, 파라미터들은 선언시 타입설정이나 타입체킹을 하지 않습니다.
2. 이때, 우리는 타입체킹을 하는 함수를 직접 써서 미래에 일어날 수 있는 문제를 막을 수 있습니다.

   ```
   function sum(a) {
       let total = 0;
       for(let element of a) {
           if (typeof element !== "number") {
               throw new TypeError("sum(): elements must be numbers")
           }
           total += element;
       }

       return total;
   }

   sum([1,2,3])
   sum(1,2,3);     // TypeError: 1 is not iterable
   sum([1,2,"3"])  // TypeError: element 2 is not a number
   ```

## 8.4 Function as Values

1. 자바스크립트의 함수는 값이기도 합니다. 그래서 변수에 할당되어질 수 있고 오브젝트의 프로퍼티에 저장될 수 있고, 배열의 원소가 될 수도 있고, 다른 함수의 인자로 넘겨질 수도 있습니다.
2. 함수의 이름은 변수의 이름일 뿐입니다. 함수자체는 값이되어 다른 변수들에 할당될 수 있습니다.
   ```
   function square(x) { return x*x};
   let s = square;
   square(4)
   s(4)
   ```
3. 함수는 변수뿐 아니라 `object`에도 할당될 수 있습니다. 이때 우리는 `method`라고 부릅니다.
   ```
   let o = {square : function(x) {return x*x}}
   let y = o.square(16)
   ```
4. 함수가 값처럼 사용될 수 있다는 점은 다음과 같이 강점으로 사용되어질 수 있습니다.

   ```
   function add(x,y) { return x+y}
   function subtract(x,y) { return x-y}
   function multiply(x,y) { return x*y}
   function divide(x,y) { return x/y}

   function operate(operator, operand1, operand2) {
       return operator(operand1, operand2)
   }

   let i = operate(add, operate(add,2,3), operate(multiply, 4,5))
   ```

## 8.4.1 Defining Your Own Function Properties

1. 자바스크립트에서 함수는 원시값(_primitive values_)는 아니지만 특수한 형태의 `object`입니다. 즉, function은 *프로퍼티*를 가질 수 있습니다. 우리가 어떤 정보들을 계속 저장을 하고 싶을 때 구지 `global variable`을 사용하지 않고 함수의 프로퍼티를 사용할 수 있습니다.

   ```
   function factorial(n) {
       if(Number.isInteger(n) && n>0) {
           if(!(n in factorial)) {
               factorial[n] = n * factorial(n-1)
           }
           return factorial[n]
       } else {
           return NaN
       }
   }

   factorial[1] = 1
   factorial(6)  // ==>720
   factorial[5]  // ==>120
   ```
