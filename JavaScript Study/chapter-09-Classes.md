# CHAPTER 9. Classes

- 자바스크립트에서 클래스는 `prototype-based` inheritance를 사용합니다. 간단히 말하자면, 여러개의 인스턴스 오브젝트가 같은 `prototype` 프로퍼티를 상속했다면, 그 오브젝트들은 그 클래스의 인스턴스가 됩니다.
  ![Screenshot from 2021-02-21 23-51-27](https://user-images.githubusercontent.com/30207544/108628772-0c025180-74a0-11eb-9c9b-4d079177a0f8.png)

- 자바스크립트는 항상 클래스를 정의할 수 있도록 두었지만, ES6부터는 `class`라는 키워드를 사용해서 보다 클래스를 쉽고 명확하게 정의할 수 있도록 하였습니다. 새로운 `class`키워드를 사용하여 만든 클래스의 작동방법은 ES6이전의 클래스가 작동하는 방법과 정확히 동일합니다.

- 그래서 ES6이전의 클래스는 어떻게 사용했는지 살펴보면서 `class` keyword 안에서 무슨 일이 일어나는지 살펴보도록 하겠습니다.

## 9.1 Classes and Prototypes

- 자바스크립트에서 클래스란, 같은 `prototype object`로부터 프로퍼티를 상속받은 오브젝트들의 집합이라고 할 수 있습니다. 그래서 `prototype object`는 클래스의 가장 중요한 특성 중 하나입니다.
- 자바스크립트의 class의 중요한 부분을 이루는 `prototype object`와 `constructor`를 ES6이전에는 어떻게 정의했는지 먼저 팩토리함수를 사용한 예로 알아보도록 하겠습니다.

  ```
  function range(from, to) {

      let r = Object.create(range.methods);

      //These are noninherited properties that are unique to this object
      //These are used only in property object by being refered with this keyword
      r.from = from;
      r.to = to;

      return r
  }

  range.methods {
      includes(x) { return this.from <= x && x <= this.to; }


      *[Symbol.iterator](){
          for(let x = Math.ceil(this.from); x <= this.to; x++)
          yield x;
      }

      toString() { return "(" + this.from + "..." + this.to + ")";   }

  }


  let r = range(1,3);
  r.includes(2)
  r.toString()
  [...r]
  ```

1. 하나씩 살펴보면, 먼저 `range()`는 factory function으로 정의되어서, class의 생성자와 비슷한 역할을 합니다. 저희는 6장에서 `Object.create()`에 배운적이 있었습니다. 이 함수는 인자에 명시한 `object`의 `prototype object`를 상속하여 새로운 object를 만듭니다. 그래서 Object.create(range.methods)를 통해 `property object`를 상속하였습니다.

2. `range.methods`라는 `prototype object`를 정의했습니다. `prototype object`에는 3개의 `property`가 있습니다. 이 3개의 method가 정의된`object`는 `range()` 팩토리함수로 생성되는 모든 object들이 상속하는 `prototype object`가 됩니다.

3. 그래서 우리는 이렇게 정의한 클래스로 range object를 생성하고, 인스턴스들은 상속받은 `prototype object`를 사용할 수 있습니다.

## 9.2 Classes and Constructors

- 자바스크립트에서 `constructor`는 `new` keyword를 사용함으로써 호출됩니다. 이 때, `new` keyword는 자동으로 새로운 object를 생성하게 되고 `constructor`는 새롭게 생성된 object의 property를 initialize하는 역할을 합니다.

* constructor는 `prototype property`를 가지고 있는 `function object`입니다. 그래서 constructor로 생성된 모든 object들은 이 'prototype property'를 상속받게 되는 것이고, 같은 클래스의 멤버가 되는 것 입니다.

* ES6의 `class` keyword가 있기 전까지 Javascript의 class 정의는 어떻게 했는지 알아봄으로서, ES6 `class`의 실제 안에서는 어떻게 일어나는지 알아보도록 합시다.

  ```
  // 생성자 정의

  function Range(from, to) {
    this.from = from;
    this.to = to;
  }

  // 모든 Range object들은 이 오브젝트를 상속합니다.
  Range.prototype = {
      includes(x) { return this.from <= x && x <= this.to; }


      *[Symbol.iterator](){
          for(let x = Math.ceil(this.from); x <= this.to; x++)
          yield x;
      }

      toString() { return "(" + this.from + "..." + this.to + ")";   }

  }

  let r = new Range(1,3);
  r.includes(2)
  r.toString()
  [...r]

  ```

  - 전의 팩토리함수와 하나씩 비교하면서 살펴보도록 하겠습니다.
    1. 이번 예시에서는 new keyword를 사용함으로써 다양한 이점이 있었습니다 . 새로운 object를 만들기 위해 Object.create()를 호출하지 않아도 되었습니다.
    2. 전 예시와 달리 생성자는 새로운 object를 `return`하지 않아도 됩니다.
       - 이는 생성자가 호출될때 자동으로 새로운 object를 만들고, 생성자 자신이 object의 매서드 처럼 호출되고, 새로운 object를 return하기 때문입니다.
    3. `Range.prototype`처럼 `prototye object`를 명시적으로 정의했습니다. 그래서 Range() 생성자가 호출 시, 자동으로 `Range.prototype`이 새로운 Range object의 prototype이 되도록 합니다.
  - 두 예시의 공통점이라면, 두 예시 생성자 모두 arrow function을 사용하지 않았습니다. 이는 저희가 8장에서 살펴봤듯이 arrow function은 일반함수와 달리 `prototype object`를 갖고 있지 않기 때문입니다. 그리고 arrow function은 arrow function이 정의된 context에서 this keyword를 inherit하기 때문에 생성자에서 `this` keyword를 사용할때는 올바르지 않는 context를 참조하게 됩니다.
  - ES6의 `class` syntax에서 method를 arrow function으로 정의할 수 없습니다.

## 9.2.1 Constructor, Class Identity, and instance of

- 생성자는 `prototype object` 만큼 중요하지는 않지만, 생성자 함수의 이름이 클래스의 이름으로 사용되어지기 때문에, 생성자는 클래스의 대표성을 갖습니다.
- 그래서 우리가 `instanceof` 연산자로 어떤 `object`가 class의 member인지 확인할때 우리는 생성자의 이름을 사용합니다.
  ```
  r instanceof Range    //=> true
  ```
- 하지만 `instanceof`를 좀 더 들여다보면, 위의 `instanceof`연산자의 우측항이 생성자임에도, r이 생성자로 초기화되었는지를 체크하는 것이 아니라, r이 Range.prototype을 상속하였는지를 체크합니다.
- 그래서, 아래와같이 새로운 `Strange.prototype`을 `Range.prototype`로 생성한 후 `instanceof`로 비교를 하면, `Strange object`는 Range()의 object가 될 수 있음을 알 수 있습니다.
  ```
  function Strange() {}
  Strange.prototype = Range.prototype;
  new Strange() instanceof Range // =>
  ```
- 결론적으로, 생성자 함수는 대표성만 갖을뿐, instance의 class를 결정하는 것은 `prototype`이 하게 되어있습니다.

## 9.2.2 The constructor Property

- 위에서 저희는 클래스의 매서드를 정의하기 위해 `Range.prototype`이라는 새로운 object를 만들었지만, 사실 이것이 꼭 필요한 일은 아닙니다.
- 왜냐하면, Arrow Function, Async Function, Generator Function을 제외한 Javascript의 모든 함수는 생성자로 사용되어질 수 있는데, 생성자 호출시 반드시 생성자는 `prototype property`를 가져야만 합니다. 그래서 결론적으로 Javascript의 모든 함수는 `prototype property`를 자동으로 가지고 있습니다.
- 모든 Javascript의 함수가 가지고 있는 이 `prototype property`의 값은 `constructor`라는 프로퍼티를 가진 object입니다.

  ```
  let F = function() {};  //모든 함수는 생성자가 될 수 있다.
                          //생성자를 호출할 땐 prototype property가 필요하다.
                          //그래서 모든 함수는 prototype property를 가지고 있다.
  let p = F.prototype;
  let c = p.constructor;
  c === F                 //=> true
  ```

* 모든 함수에 `constructor` property가 담긴 `prototype object`가 항상 정의되어 있다는 점은 오브젝트들이 오브젝트들의 생성자를 가리키는 `constructor property`를 상속한다는 점입니다.
  ```
  let o = new F()
  o.constructor === F   //true
  ```

![Screenshot from 2021-02-22 23-21-15](https://user-images.githubusercontent.com/30207544/108720922-bfcd1500-7564-11eb-97da-09f7af4af2e1.png)

- 아까전의 `Range.prototype`을 정의할때, 사실 이미 정의된 prototype위에 덮어서 정의를 했습니다. 그래서 미리 정의된 `constructor`를 살리기 위해 다음과 같이 method들을 정의를 할 수 있습니다.

  ```

  Range.prototype ={
    constructor : Range,
  }


  vs

  Range.prototype.includes = {
    return this.from <= x && x <= this.to;
  }

  Range.prototype.toString = function(){
    return "(" + this.from + "..." + this.to + ")"
  }
  ```

## 9.3 Classes with the class Keyword

- Class는 Javascript의 처음부터 항상 함께해왔습니다. 그리고 ES6부터 `class`라는 keyword를 사용해서 보다 명확하게 class를 정의할 수 있게 되었습니다. 앞으로는 `class` keyword를 사용해서 어떻게 클래스를 정의할 수 있는지 알아보도록 하겠습니다.

```
class Range {
  constructor(from, to) {
    this.from = from;
    this.to = to;
  }

  includes(x) {
    return this.from <= x && x <= this.to;
  }

  *[Symbol.iterator](){
      for(let x = Math.ceil(this.from); x <= this.to; x++)
      yield x;
  }

  toString = {
    return "(" + this.from + "..." + this.to + ")"
  }
}


let r = new Range(1,3);
r.includes(2)
r.toString()
[...r]
```

- 이번 예시와 9.2의 예시는 똑같이 작동합니다. 즉, `class` keyword를 사용했다고해서, ES6이전의 prototype-based 클래스가 작동하는 방법과 전혀 다르지 않습니다

- `class` keyword를 사용함으로써, 이제부터 method를 정의할 때 9.2의 예시처럼 comma로 매서드들을 나누지 않습니다. 즉 class의 몸통부분은 object literal처럼 key-value값으로 쓰여지지 않습니다.

- `constructor` keyword를 사용해서, 생성자를 정의했습니다. `constructor` keyword를 통해 만들어진 생성자 함수의 이름은 `constructor`가 아닙니다. 실제로는 `class` keyword로 정의 선언을 내릴 때 `Range`라는 새로운 변수를 정의하고 `constructor` 함수가 `Range`변수에 할당되어집니다.

* 함수를 선언할때처럼, class를 선언할때 statement와 expression을 활용할 수 있습니다.

  ```
  let square = function(x) { return x * x};
  square(3)

  vs

  let Square = class { constructor(x) {this.area = x * x}}
  new Square(3).area
  ```

* 함수 선언과 달리, 클래스의 선언은 hoisted되지 않습니다. 그래서 클래스를 선언하기 전에 인스턴스를 생성할 수 없습니다.

## 9.3.1 Static Methods

- `static` keyword와 함께 static method를 정의할 수 있습니다.
- static method는 `prototype object`의 프로퍼티로 정의되지 않고, 생성자 함수의 프로퍼티로 정의됩니다.

  ```
  class Range {
    constructor() {
        ... 생성자 초기화
    }

    static parse(s) {
      let matches = s.match(/^\((\d+)\.\.\.(\d+))$/);
      if (!matches) {
        throw new TypeError('Cannot parse Range from "${s}".')
      }
      return new Range(parseInt(matches[1]), parseInt(matches[2]))
    }

    include(){...}

    toString(){...}
  }
  ```

  - 이렇게 정의된 static method는 Range.prototype.parse()가 아니라, Range.parse()로 정의됩니다. 그래서 인스턴스가 아니라 생성자를 통해서만 static method를 호출할 수 있습니다.
    ```
    let r = Range.prase('(1...10)');
    r.parse('(1...10)')
    ```

- static method는, class/constructor의 이름을 사용해서 호출되기 때문에 class method라고도 불립니다. 어떤 인스턴스로부터 호출된다기보다는 생성자로부터 호출되기 됩니다.

## 9.3.2 Getters, Setters, and other Method Forms

- Object literal에서 getter와 setter를 사용했듯이, class에서도 사용할 수 있습니다. 다른점이라면, class에서 사용될때 comma를 사용하지 않는다는 것입니다.

## 9.3.3 Public, Private and Static Fields

- ES6 Class에서 여러 매서드(getter, setter, generators 등등)와 static method는 허용하지만 field를 정의하는 syntax를 포함하지는 않습니다.
- 클래스에 field를 꼭 정의를 해야한다면, 반드시 constructor 함수안에서 이루어져야 합니다. 그리고 static field를 정의해야한다면, class를 정의하고 나서, class의 바깥에서 정의해야합니다.

- public, private, static fields를 정의하는 syntax가 Javascript에서 곧 표준이 될 예정이지만, 아직까지는 표준이 아닙니다. 하지만 Chrome에서는 지원되고 있고, Firefox에서는 부분적으로 지원됩니다. 그리고 Babel에서는 활발히 사용되고 있습니다.

- 앞으로 새롭게 생길 field syntax에 대해 알아보도록 하겠습니다.

  ```
  class Buffer {
    constructor() {
      this.size = 0;
      this.capacity = 4096;
      this.buffer = new Uint8Array(this.capacity);
    }
  }
  ```

  ```
  class Buffer{
    size = 0;
    capacity = 4096;
    buffer = newUint8Array(this.capacity)
  }
  ```

  - 새롭게 생길 field syntax에서는 생성자 밖으로 나와서 정의됩니다. 즉, class의 몸통에 바로 정의할 수 있게 됩니다. 이런 방법으로 instance field를 정의하면, class의 바로 위에 정의함으로써, 가독성을 올릴 수 있고 어떤 field가 instance의 상태를 정의하는지 바로 알 수 있습니다.

- #(Hashtag)를 사용함으로써, private fields를 정의할 수 있습니다. 이렇게 정의된 field는 정의된 클래스안에서는 사용이 가능하지만, 클래스 바깥에서는 접근할수가 없습니다.
  ```
  class Buffer {
    #size = 0;
    get size() {return this.#size;}
  }
  ```

## 9.3.4 Example : A complex Number Class

```
class Complex {
  // private fields
  #r = 0;
  #i = 0;

  //이 생성자는 instance fields인 r과 i를 생성합니다.
  //모든 instance는 r과 i를 사용할 수 있습니다.
  constructor(real, imaginary){
    this.r = real;
    this.i = imaginary
  }

  //instance method
  plus(that){
    return new Complex(this.r + that.r, this.i + that.i)
  }
  times(that){
    return new Complex(this.r * that.r - this.i * that.i,
                      this.r * that.i + this.i * that.r)
  }

  //static method
  static sum(c,d) { return c.plus(d)}
  static product(c,d) { return c.times(d)}

  //instance method
  get real() {return this.r}
  get imaginary() {return this.i}
  get magnitude() {return Math.hypot(this.r, this.i)}

  toString() {return `{${this.r},${this.i}}`}

  equals(that) {
    return that instanceof Complex &&
      this.r === that.r &&
      this.i === that.i
  }

}

Complex.ZERO = new Complex(0,0);
Complex.ONE = new Complex(1,0);
Complex.I = new Complex(0,1);

let c = new Complex(2,3)         //생성자로 새로운 오브젝트 생성
let d = new Complex(c.i, c.r)    //instance fields of c 사용하기
c.plus(d).toString()             //instance methods 사용하기
c.magnitude                      //getter function 사용하기
Complex.product(c,d)             //static method 사용하기

```

## 9.4 Adding Methods to Existing Classes

- Javascript의 prototype-based 메커니즘은 동적입니다. 즉, prototype의 프로퍼티가 오브젝트가 생성된 후에 변경된다고 하더라도, 오브젝트는 이 prototype의 프로퍼티를 상속받습니다.

- 즉, 매서드를 계속해서 prototype에 더함으로써, class의 피쳐들을 계속해서 쉽게 더해나갈 수 있습니다.

- 만약, Javascript의 버전이 오래되서, String의 startsWith 매서드가 없다면 아래처럼 쉽게 기능을 추가할 수 있습니다.

```
if (!String.prototype.startsWith){
  String.prototype.startsWith = function(s) {
    return this.indexOf(s) === 0;
  }
}
```

- 이와 같은 방법으로 Number, strings과 같은 built-in type에 method를 더할 수도 있습니다. 하지만 이는 새로운 버전의 자바스크립트가 똑같은 이름으로 매서드를 정의한다면 호환성에 문제를 일으킬수도 있습니다.
