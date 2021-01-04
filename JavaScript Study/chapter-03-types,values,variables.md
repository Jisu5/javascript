## 3.1 Overview and Definitions

- 자바스크립트의 Type은 크게 2가지로 나눌 수 있습니다. Primitive Types and Objective Types.

  ![Screenshot from 2020-12-21 23-55-26](https://user-images.githubusercontent.com/30207544/102789867-0b8a0380-43e8-11eb-9ab4-a6bc7c32571c.png)

- 하나의 Number, string, boolean, symbol, null, undefined가 아닌 모든 값은 Object입니다.

## 3.2 Numbers

- Number는 integer와 real number를 나타내기 위해 사용된다.

- 자바스크립트는 Number를 나타내기 위해 IEEE 754표준에서 정의한 64-bit floating-point를 사용한다.
  (Java, C++의 double이 64bit부동소수점을 사용한다.)

- 정확히 자바스크립트는 크게는 -1.7976931348623157 _ 10^308 와 +-1.7976931348623157 _ 10^308 사이를 나타낼 수 있고, 작게는 -5 _ 10^-324와 +5 _ 10^-324사이를 나타낼 수 있습니다.

  ```
  const max = Number.MAX_VALUE
  const x = max + 10

  //max : 1.7976931348623157e+308
  //  x : 1.7976931348623157e+308

  const min = Number.MIN_VALUE
  const y = min / 10

  //min : 5e-324
  //  y : 0

  const max = Number.MAX_VALUE;
  const z = max * 2;
  //  z : infinity

  ```

## 3.2.1 Integer Literals

- 자바스크립트는 기본적으로 10진수와 16진수를 나타낼 수 있습니다.

- ES6이상부터는 정수를 2진수와 8진수를 나타낼 수 있습니다.
  ```
  const input = 7;
  const output = input.toString(2);
  // output : 111"
  ```

## 3.2.2 Floating-Point Literals

- Javascript의 부동소수점은 다음과 같이 표현될 수 있습니다.
  > _[digits][.digits][(E|e)[](+|-)digits]_
  - 3.14, 2345.6789, 6.02e23, 1.4738223E-32
- ES6 자바스크립트는 긴 숫자를 나눠서 설명하기 위해 다음과 같이 사용될 수 있습니다.
  ```
  let billion = 1_000_000_000;   //1000000000
  let bytes = 0x89_AB_CD_EF;     //2309737967
  let bits = 0b0001_1101_0111;   //471
  let fraction = 0.123_456_789   //0.123456789
  ```

## 3.2.3 Arithmetic in JavaScript

- 자바스크립트는 +, -, \*, /, %, 그리고 ES6에서 추가된 \*\*(exponentiation)을 산술연산으로 가지고 있습니다.
- 자바스크립트의 산술연산은 Overflow, Underflow, 0으로 나누기 등과 같은 경우에 Error를 발생시키지 않습니다.

  ```
  const max = Number.MAX_VALUE
  const x = max + 10

  //max : 1.7976931348623157e+308
  //  x : 1.7976931348623157e+308

  const min = Number.MIN_VALUE
  const y = min / 10

  //min : 5e-324
  //  y : 0

  const max = Number.MAX_VALUE;
  const z = max * 2;
  //  z : infinity

  ```

- Overflow가 발생했을 때, 즉 산술연산의 결과가 자바스크립트가 표현할 수 있는 수를 넘어섰을 때 결과는 infinity 혹은 -infinity입니다.
- Underflow가 발생했을 때, 즉 산술연산이 0에 너무나 가까워져서 자바스크립트가 표현할 수 없을 때, 자바스크립트는 0을 반환합니다.
- 만약에 Underflow가 -쪽에서 발생하면 자바스크립트는 -0을 반환하게 됩니다.
- Underflow가 발생했을 때의 0은 일반적인 0과는 거의 구별할 수 없습니다.

  ```
  const zero = 0;      //regular zero
  const negz = -0;     //Negative zero
  zero === ngez        //true       : zero와 negative zero는 같습니다.
  1/zero === 1/negz    //false      : Infinity와 -Infinity는 같지않습니다.

  ```

- 0으로 나누는 수는 자바스크립트에서 에러가 아니고, infinity와 -infinity를 반환합니다. 하지만 예외적으로, 0/0은 NaN을 반환합니다.
  ```
  const inf = 100 / 0               //infinity
  const neginf = -100 / 0           //-infinity
  const nan = 0 / 0                 //NaN
  const infdiv = Infinity/Infinity  //NaN
  ```
- not-a-number값은 어느 다른 값들과 equal 비교를 할 수 없습니다. 그래서 다음과 같은 if문에서

  ```
  const x = 0 / 0            //NaN

  if (x === NaN) {
  console.log('X == NaN')
  } else {
  console.log('X != NaN')    //오답출력
  }

  ```

  틀린값을 'X != NaN' 출력하게 됩니다.

  ```
  const x = 0 / 0            //NaN

  if (Number.isNaN(x)) {
  console.log('X is NaN')    //정답 출력
  } else {
  console.log('X isnt NaN')
  }
  ```

## 3.2.4 Binary Floating-Point and Rounding Errors

- 자바스크립트는 부동소수점을 사용하는 Java, C++처럼 실수를 표현할 때 Approximation을 한다. 이것은 IEEE-754 부동소수점을 표현할 때 이진수로 표현하기 때문이다. 그래서 1/2, 1/8, 1/1024와 같은 값은 정확히 나타낼 수 있지만, 1/10, 1/100과 같은 경우는 근사값입니다.
- 이런 경우 흔한 문제는 다음과 같습니다.
  ```
  let x = 0.3 - 0.2      //Expected Value : 0.1
  let y = 0.2 - 0.1      //Expected Value : 0.1
  x===y                  //false
  x===0.1                //false  (x : 0.99999999999999998)
  y===0.1                //true
  ```
  이와 같은 문제는 Javascript에 국한된 것이 아니라 binary 부동소수점을 사용하는 모든 언어들에서 공통적으로 생기는 문제입니다. 0.999999999999998과 1은 매우 가까운 수이기 때문에 같다고 봐도 무방하지만, 문제는 같은지 아닌지를 비교할 때 생깁니다.
- 부동소수점의 근사값이 문제가 된다면, 다음과 같이 Scaled Integer를 사용하는 것이 해결책이 될 수 있습니다.

  ```
  let x = 0.3
  x = 30

  let y = 0.2
  y = 20
  ```

## 3.3 Text

- String은 Immutable하며, 16-bit의 값들의 연속으로 이루어져 있습니다.
- 각각 16bit값들은 Unicode Character들을 나타냅니다.
- 자바스크립트는 1개의 원소를 가진 String에 대해 Special Type을 가지고 있지 않습니다.

## 3.3.2 Escape Sequences in String Literals

- backlash character(\)는 Character와 결합하여 특수한 목적을 가집니다.

  ```
  const x = '\n'   //new line
  const y = 'You \'re right, it can\'t be a quote'

  ```

- \0, \b, \t, \n, \v, \f, \r, \", \' \\, \xnn, \unnnn, \u{n} 을 제외하고 backlash(\)를 사용하면 backlash는 무시됩니다.

## 3.3.3 Working with Strings

- String에서 +연산은 두 개의 String을 이어 붙입니다.
  ```
  let msg = "Hello" + "world";
  ```
- String의 모든 비교연산 <,>,<=,>=은 단순히 16-bit값들의 비교 연산입니다.

- String은 Immutable이기 때문에 replace()와 toUpperCase()와 같은 연산은 새로운 String을 반환합니다. 호출한 스트링은 바뀌지 않습니다.

- Strings은 읽기전용(Read-only) array로 취급할수 있습니다. 그리고 다음과 같이 brackets을 사용하여 각각의 16bit값들에 접근할 수 있습니다.
  ```
  let s = 'hello, world';
  s[0]                       //"h"
  ```

## 3.4 Boolean Values

- 모든 자바스크립트 값은 전부 boolean value로 변환될 수 있습니다. 그리고 다음과 같은 값은 false로 작동합니다.
  ```
  undefined
  null
  0
  -0
  NaN
  ""      //empty string
  ```
- 이것을 제외한 모든 값, 오브젝트들은 전부 true로 작동합니다.

## 3.5 null and undefined

- null은 value의 absence를 나타내기 위해 사용되는 키워드입니다. null을 typeof를 사용하여 타입을 체크하면 object를 반환합니다. 'no object'를 나타내는 special object라고 생각하는 것이 편하다.

- undefined는 조금 더 깊은 value의 absence를 나타내는 값이다. 주로 Initialized되지 않은 변수나, 존재하지 않는 배열의 원소나 Object의 Property를 찾으려고 할 때 나타난다.

  ```
  const testVar = null;
  alert(testVar);          //shows null
  alert(typeof testVar);   //shows object
  ```

  ```
  const testVar;
  alert(testVar);        //shows undefined
  alert(typeof testVar); //shows undefined
  ```

- undefined는 명시적으로 값을 반환하지 않은 function의 반환값이다.

- undefined는 null(language keyword)과 달리 미리 정의된 Global Constant이다.

- undefined를 typeof를 사용하면 undefined를 반환한다.

- null과 undefinied는 모두 False를 리턴한다.

- 저자는 undefined를 시스템 단계, 예상치 못함, 에러와 관련된 value of absence로 보고, null은 프로그램 단계, 정상적임, 예상됨과 같은 value of absence로 본다고 합니다.
  ![Screenshot from 2020-12-22 09-46-16](https://user-images.githubusercontent.com/30207544/102835718-d8bd2b00-443a-11eb-86d1-83031fb22a3a.png)
