# Chapter 5. Statements

- 자바스크립트 프로그래밍은 쉽게 말해, Statement들의 나열이고, Javascript의 interpreter가 이 Statement들을 하나씩 실행할 뿐입니다.

## 5.1 Expression Statements

- 자바스크립트에서 가장 단순한 형태의 Statement는 side effect를 가진 expression입니다.

1.  Assignment Statement는 expression statement의 대표적인 Statement입니다.
    ++와 --와 같은 Increment, Decrement operators들 역시 side effect를 가진 expression이기 때문에 statement에 포함될 수 있다. 주로 다음과 같이 쓰여집니다.
    ```
    greeting = "Hello" + name;
    *= 3;
    counter++;
    ```
2.  Delete 연산자는 Object의 Property를 삭제한다는 side effect를 가지고 있기 때문에, Expression의 한 부분이라기 보다, 주로 Statement로 사용되어진다고 할 수 있습니다.

    ```
    delete o.x;
    ```

3.  함수의 호출역시 주요한 Expression Statement입니다.

    ```
    console.log(debugMessage);
    displaySpinner(); // web app에서 spinner를 보여주는 함수
    ```

    위와 같은 함수는 Expression입니다. 하지만, 프로그램의 상태에 영향을 미치는 side effect를 가지고 있기 때문에 statement로 사용되어집니다. 함수가 side effect를 가지고 있지 않다면 우리는 함수를 호출할 필요가 없습니다. 예를 들어서,

    ```
    Math.cos(x)
    ```

    우리는 위처럼 cosine을 계산하고 결과를 버리지 않습니다.

    ```
    cx = Math.cos(x)
    ```

    위처럼 값을 계산하고 할당하게 됩니다.

## 5.2 Compound and Empty Statements

- Comma operator가 복수의 Expression을 하나의 Expression로 합치듯이, 하나의 Statement block으로 여러개의 Statement를 하나로 합칠 수 있습니다.
- Statement block은 curly brace로 여러개의 statement들을 하나의 statement로 만듭니다.
  ```
  {
      x = Math.PI;
      cx = Math.cos(x);
      console.log("cos(n)=", + cx);
  }
  ```
- 공식적으로, 자바스크립트의 syntax는 하나의 substatement를 가질 수 있습니다. 예를 들어서 while loop syntax는 loop의 body를 구성하는 하나의 substatement만을 가져야 합니다. 우리는 Statement block을 사용해서 이러한 하나의 substatement에 여러개의 statement를 구성할 수 있습니다.
  ```
  while (expresssion)
      {
          statement1;
          statement2;
              ...
      }
  ```
- Empty Statement는 statement가 예상된 곳에 statement를 쓰지 않을수 있도록 해줍니다. Empty statement는 loop의 빈 body를 구성할 때 주로 사용되어집니다.
  ```
  // Initialize an array a
  for (let i=0; i < a.length; a[i++]=0);
  ```
- Empty Statement를 for, while, if문과 사용할 때 주의하여 사용하지 않으면 발견하기 어려운 버그가 되기 때문에, 주석을 사용하여 이를 방지하여야 합니다.

  ```
  if ((a===0) || (b===0));  //이 줄은 아무것도 하지 않습니다.
      o = null;             //항상 이 줄이 실행됩니다.


  // Initialize an array a
  for (let i=0; i < a.length; a[i++]=0) /* empty */;
  ```

## 5.3 Conditionals

### 5.3.1 if

- if 문은 자바스크립트가 조건에 따라 statement를 실행할지 넘어갈지를 정할 수 있게 해줍니다.

```
if(!username) address = 'John Doe'
```

- nested if문을 사용할 때 보통 우리가 하는 실수는 다음과 같습니다

```
i = j = 1;
k = 2;
if (i === j)
    if (j === k)
        console.log("i equals k")
else
    console.log("i doesn't equal j")
```

자바스크립트를 비롯해 많은 프로그래밍 언어에서 else절의 짝은 default로 가장 가까운 if절입니다. 그렇기 때문에 Interpretor는 다음과 같이 사용되게 됩니다.

```
if (i === j){
    if (j === k)
        console.log("i equals k")
    else
        console.log("I doesn't equal j")
}
```

필자 David Flanagan은 이런 실수를 방지하기 위해, 하나의 statement를 사용할때도 curly brace를 사용하는 습관을 들이는 것이 좋다고 말합니다.

```
if (i===j){
    if (j===k) {
        console.log("i equals k");
    }
} else {
    console.log("I doesn't equal j")
}
```

### 5.3.2 else if

- else if문은 if문을 더욱 많은 branch로 사용할 수 있도록 해줍니다.
  ```
  if (n===1) {
      // Execute code block #1
  } else if (n===2) {
      // Execute code block #2
  } else if (n===3) {
      // Execute code block #3
  } else {
      // If all else fails, excute block #4
  }
  ```

### 5.3.3 switch

- 우리는 switch statement를 사용하여 좀 더 간결하게 else if의 많은 branch들을 표현할 수 있습니다.
- switch문이 실행될 때 case의 expression은 === operator처럼 값들의 동일성을 평가합니다. ('==' operator가 아님에 주의)

```
switch(n){
    case 1:                // Start here if(n===1)
    // Execute code block #1
    break;
    case 2:                // Start here if (n===2)
    // Execute code block #2
    break;
    case 3:                // Start here if (n===3)
    // Execute code block #3
    break;
    default:
    // Execute code block #4
    break;
}
```

- switch문에서 case 절은 코드의 실행 시작점을 명시하지만, 종료점은 명시하지 않습니다. 그렇기 때문에 break문과 같은 Jump Statment가 없다면 조건에 맞는 case의 code block을 실행하고 계속해서 block의 끝에 다다를 때까지 Code block들을 실행 하게 됩니다.

- Switch문이 실행될때마다 모든 case expression이 평가되는 것이 아니기 때문에, 함수 호출이나 할당과 같은 side effect를 포함한 expression을 사용하는 것은 지양해야 합니다. 가장 안전한 방법은 case expression을 상수로 사용하는 것입니다.

- 모든 case expression이 일치하지 않는다면 switch문은 default를 실행합니다. 만약 default가 없다면 switch문은 전체 body를 skip합니다.

## 5.4 Loops

- 자바스크립트에는 총 5가지의 Loop statement를 가지고 있습니다.
  - while, do/while, for, for/of, for/in

### 5.4.1 while

```
while (expresssion)
    statement

let count=0;
while (count < 10) {
    console.log(count);
    count++;
}
```

- while문을 실행하기 위해서, javascript interpreter는 먼저 expression을 평가합니다. 만약 expression의 값이 false이면 interpreter는 statement를 넘어가게 됩니다. 반대로 값이 true라면 interpreter는 statement를 실행하고, 다시 위로 올라와서 expression을 평가하면서 loop를 돌게 됩니다.

### 5.4.2 do/while

```
    do
        statement
    while (expression);

    function printArray(a) {
        let len = a.length, i = 0;
        if (len === 0) {
            console.log("Empty Array");
        } else {
            do {
                console.log(a[i]);
            } while(++i < len);
        }
    }

```

- do/while문은 while문과 많이 비슷하지만 loop expression이 마지막에 평가됩니다. 그래서 loop body는 최소한 1번은 꼭 실행되게 됩니다.

- do/while문은 잘 사용되지 않는데, 그 이유는 '최소한 1번'이 실행을 해야하는 case는 드물기 때문입니다.

### 5.4.3 for

- for문은 while문보다 더 간편하게 사용될 수 있는 loop 구조를 가지고 있습니다.
- 대부분의 for문은 counter variable을 가지고 있고, 이 variable은 loop가 시작되기 전에 먼저 초기화되고, 반복이 시작되기전에 평가됩니다.

  ```
  for (initialize; test; increment)
      statement

      vs

  initialize;
  while(test) {
      statement
      increment;
  }
  ```

- for statement는 **the initiailization, the test, and the update**라는 중요한 3개의 expression을 가지고 있고, 이 expression을 명백하고 간략하게 표기합니다. 이러한 점은 while문 보다 한눈에 표현이 되어질 수 있고, 실수를 줄일 수 있다는 장점이 있습니다.

- initialize, increment와 같은 expression은 side effect를 가져야 유용하게 사용될 수 있습니다. 자바스크립트는 initialize expression이 변수를 선언할 수 있게해서, 선언과 초기화를 동시에 할 수 있게 해줍니다. 또 increment expression 역시 ++, -- 또는 변수할당과 같이 side effect를 가질 수 있기 때문에 매우 유용하게 사용할 수 있게 됩니다.

```
function tail(o){                          // <- Return the tail of linked list
    for(; o.next; o = o.next) /* empty */ ; //Traverse while o.next is truthy
    return o;
}
```

- initialize, test, increment 3개의 expression은 생략될 수 있지만, 2개의 세미콜론은 생략될 수 없습니다.

### 5.4.4 for/of

- for/of 문은 ES6부터 사용되어집니다.
- for/of 문은 array, string, set, map과 같은 iterable object에 사용되어 집니다.
- for/of 문이 도는 동안 Array는 "live"로 변화합니다. .. 순회하는 동안의 결과는 달라질 수 있습니다. 예를 들어 다음과 같이 data.push(sum)을 loop의 body안에 넣게 되면 loop가 무한으로 돌게 됩니다.

```
let data = [1,2,3,4,5,6,7,8,9], sum = 0;
for (let element of data) {
    sum += element;
}

    vs

let data = [1,2,3,4,5,6,7,8,9], sum = 0;
for (let element of data) {
    sum += element;
    data.push(sum)
}
```

#### for/of with objects

- objects는 iterable이 아닙니다. for/of를 보통의 object에 사용하게 되면 TypeError가 나게 됩니다.

```
let o = {x : 1 , y : 2, z : 3};
for (let element of o) {
    console.log(element)       //!!TypeError
}
```

- object를 순회하고 싶다면 for/in loop를 사용하거나, for/of와 object의 property name을 array로 리턴하는 Object.keys() method를 사용해야 합니다.

```
let o = {x : 1 , y : 2, z : 3};
let keys = "";
for (let k of Object.keys(o)){
    keys += k;
}
keys // => "xyz"
```

#### for/of with strings

- String은 iterable object입니다.
- 주의할 점은 string은 UTF-16 character로 순회하지 않고, Unicode codepoint로 순회하게 됩니다. "I❤️ 🐰"그래서 위의 emoji같은 경우 a.length는 5입니다(2개의 emoji가 각각 2개의 UTF-16 character를 나타내기 때문). 하지만 for/of를 사용하게 되면 3번을 순회하게 됩니다.

#### for/of with map

- Map은 iterable object입니다.
- 특이한 점은 Map Object의 iterator는 Map의 key나 value를 순회하지 않습니다. Key/value의 쌍을 순회합니다. 매 순회마다 iterator는 첫번째 값은 Key이고 두번째 값은 Value인 Array를 리턴합니다.

```
let m = new Map([[1, "one"]]);
for (let [key, value] of m) {   // <-destructure
    key // => 1
    value // => "one"
}
```

### 5.4.5 for/in

- for/of가 of 뒤에 iterable object만을 필요로 하지만, for/in은 in 뒤에 어느 object든 올 수있습니다. for/of가 ES6에서 새로 나왔지만, for/in은 javascript의 처음부터 함께했습니다.

```
for (variable in object) {
    statement
}
```

- Javascript의 Interpreter가 for/in statement를 실행하는 순서는 다음과 같습니다.

  1. object expression을 평가합니다. 만약 object expression이 null, undefined라면 interpreter는 loop를 skip합니다.
  2. variable expression을 평가합니다. variable을 property의 name에 할당합니다.
  3. loop body를 순회합니다.

- for/in loop는 object의 모든 property를 enumerate하지는 않습니다. 대표적인 경우는 다음과 같습니다.

  1. Property의 이름이 symbol인 경우 enumerate하지 않습니다.
  2. core Javascript에 의해 정의된 다양한 built-in method들은 enumerable하지 않습니다. 모든 object는 toString() method를 갖고 있지만, for/in loop는 이 toString property를 enumerate하지 않습니다.
  3. built-in method뿐만 아니라, 다른 많은 built-in object의 property도 enumerable하지 않습니다.

- 상속받은 특성들도(Inherited properties) enumerable합니다. 그래서 만약 for/in loop를 사용하면서, 모든 오브젝트들로부터 상속받은 특성을 정의하는 코드를 사용하게 되면 loop가 예상한대로 작동하지 않을 수 있습니다. 이러한 이유로 for/in loop를 사용하기 보다는 Object.keys()와 for/of loop를 사용하는 것이 좋습니다.
- for/in loop가 아직 enumerate하지 않은 property를 삭제하게 되면, 그 property는 실행되지 않습니다.
- for/in loop가 새로운 property를 정의하게 되면, 그 property는 실행될수도, 실행되지 않을 수도 있습니다.

## 5.5 Jumps

- Jump라는 이름이 말하듯이 Jump statement들은 코드의 다른 위치로 jump를 하게 해줍니다.

### 5.5.1 break, continue

- break는 loop의 마지막으로 가거나, 다른 statement로 이동하고 싶을때 사용합니다.
- break문은 loop, switch statement에만 사용할 수 있습니다. (or syntax error)

```
for (let i=0; i<a.length; i++) {
    if (a[i] === target) break;
}
```

- 가장 가까운 loop나 switch문이 아닌 다른 statement로부터 break하고 싶을 때, labeled 형태의 break를 사용할 수 있습니다.
  ```
  computeSum: if(matrix){
      for (let x = 0; x < matrix.length; x++) {
          let row = matrix[x];
          if(!row) break computeSum;
          for (let y=0; y < row.length; y++ ) {
              let cell = row[y];
              if (isNaN(cell)) break computeSum;
              sum += cell;
          }
      }
      success = true
  }
  ```
- continue문은 loop의 나머지 부분을 skip하고, 새로운 iteration을 시작하기 위해 loop의 가장 윗부분 부터 다시 시작합니다.
- continue문은 loop statement에서만 사용가능합니다. (or syntax error)
  - while loop
    1. expression이 다시 evaluate됩니다.
    2. true라면 loop를 시작합니다.
  - for loop
    1. increment expression이 다시 evaluate됩니다.
    2. test expression이 다시 test됩니다.
    3. true라면 loop를 다시 시작합니다.
  - for/of or for/in loop
    1. loop는 다음 순회 value나 property 부터 시작합니다.
  - 주의할 점은, while문은 바로 조건 expression을 평가하지만, for문은 increment부터 평가하고 조건 expression을 평가한다는 점입니다.
- continue문도 break문과 같이 labeled statement를 사용할 수 있습니다.
- 주의할 점은 labeled statement를 사용할 때 continue문과 break문은 반드시 아래와 같이 사용해야하며, line break는 여기서 허용되지 않습니다.

```
continue labelname;
```

### 5.5.2 throw

- Throw는 에러나 예외적인 상황에 대해 신호를 보내는 것이고, Catch는 예외를 다루는 것입니다.
- 자바스크립트에서 예외는 런타임에러가 발생할때나, throw statement가 실행될 때 발생합니다.

```
throw expression;
```

- throw의 expression은 에러코드가 담긴 number, 에러메시지가 담긴 string이 될수도 있습니다. 또한 Javascript의 Error 클래스를 사용할수도 있습니다.

```
function factorial(x){
    // If the input argument is invalid, throw an exception!
    if (x<0) throw new Error("X must not be negative!")
}
```

- exception이 thrown되었을 때 Javascript interpreter는 프로그램 실행을 멈추고 가장 가까운 exception handler로 이동합니다. 만약에 catch절을 사용한 exception handler가 없다면 그 다음 높은 순위를 가진 코드 block으로 가서 exception handler가 있는지 확인합니다. 이렇게해서 handler를 찾을 때 까지 javascript의 lexical 구조를 따라 올라가게 됩니다. 만약 handler를 찾지 못하게 되면 exception을 에러로 판단하고 유저에게 알립니다.

### 5.5.3 try/catch/finally

- try/catch/finally statement는 exception을 다룹니다.
- try절은 단순히 예외가 어떻게 처리될지를 정의합니다.
- catch절은 try절에 뒤이어서 정의하며, try block에서 예외가 일어날때 실행하게 됩니다.
- finally절은 catch절에 뒤이어서 정의하며, finally절의 코드 블록은 실행을 보장하고 주로 clean up code로 사용됩니다.

```
try {

 //

} catch(e) {

    //

} finally {

    //
}
```

- try/catch/finally 모두 curly brace를 무조건 사용해야합니다. curly brace는 try/catch/finally에 뒤이어오는 statement가 하나여도 생략할 수 없습니다.
- 보통 Javascript interpreter가 try block의 끝에 다다르면, finally block을 실행합니다. 만약 try block이 return, continue, break와 같은 statement를 통해 다른 statement로 이동할 때, finally block은 이동하기전에 실행되어집니다.
- 만약 try block에서 exception이 일어나고 catch block이 존재한다면, javascript interpreter는 catch를 실행하고 finally를 실행합니다. 만약에 catch block이 존재하지 않는다면 finally block을 실행한 다음에 가장 가까운 catch절로 이동합니다.
- finally block이 return, continue, break, exception을 일으키는 함수를 호출, throw등과 같은 statement에 의해서 jump가 일어난다면, Javascript interpreter는 그 전까지 대기중인 jump들은 모두 버리고 새로운 jump를 실행합니다. 예를 들어서, finally절에서 exception이 일어난다면, finally 이전에 일어났던 예외들을 모두 대체합니다.

## 5.6 Summary of Javascript Statements

| Statement         | Purpose                                                                   |
| ----------------- | ------------------------------------------------------------------------- |
| break             | Exit from the innermost loop or switch or fromo named enclosing statement |
| case              | Label a stement within a switch                                           |
| class             | Declare a class                                                           |
| const             | Declare and initialize one or more constants                              |
| continue          | Begin next iteration of the innermost loop or the named loop              |
| debugger          | Debugger breakpoint                                                       |
| default           | Label the default statement within a switch                               |
| do/while          | An alternative to the while loop                                          |
| export            | Declare values that can be imported into other modules                    |
| for               | An easy-to-use loop                                                       |
| for/await         | Asynchronously iterate the values of an async iterator                    |
| for/in            | Enumerate the propety names of an object                                  |
| for/of            | Enumerate the values of an iterable object such as an array               |
| function          | Declare a function                                                        |
| if/else           | Execute one statement or another depending on a condition                 |
| import            | Declare naems for values defined in other modules                         |
| label             | give statement a name for use with break and continue                     |
| let               | Declare and initialize one or more block-scoped variabls                  |
| return            | Return a value from a function                                            |
| switch            | Multiway branch to case or default : labels                               |
| throw             | Throw an exception                                                        |
| try/catch/finally | Handle exceptions and code cleanup                                        |
| "use strict"      | Apply strict mode restrction to script or function                        |
| var               | Declare and Initialize one or more variables (old syntax)                 |
| while             | A basic loop construct                                                    |
| with              | Extend the scope chain                                                    |
| yield             | Provide a value to be iterated; only used in generator functions          |
