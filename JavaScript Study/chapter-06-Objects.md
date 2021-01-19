## 6.6 Enumerating Properties

오브젝트(Object)를 순회(Iterate)하면서 프로퍼티(Property)를 열거(Enumerate)하는 방법은 다음이 있습니다.

1. For / In loops 사용
   - Loop Variable에 오브젝트 프로퍼티의 이름을 할당하면서, 열거가능한(Enumerable)한 프로퍼티를 열거합니다.
   - 주의할 점
     1. 오브젝트가 상속받은 Built-in methods들은 열거가능하지(Enumerable) 않습니다.
     2. 개발자가 직접 쓴 오브젝트의 Property는 Default로 열거가능(Enumerable)합니다.
        ```
        let o = {x : 1, y : 2, z : 3}      //user defined property
        o.propertyIsEnumerable("toString") //bult-in method
        for(let p in o){
            console.log(p)
        }
        ```
2. Property의 이름이 담긴 배열(Array)을 반환(return)하는 *4가지 함수*를 사용
   1. Object.keys()
      - Object의 `Enumerable`한 프로퍼티들을 배열로 반환합니다.
      - Object의 `Non-Enumerable`한 프로퍼티들은 포함하지 않습니다.
        (Non-Enumerable : 상속받은 프로퍼티, Symbol)
   2. Object.getOwnPropertyNames()
      - Object의 `Enumerable`, `Non-Enumerable`한 프로퍼티 모두를 배열로 반환합니다.
      - Property이름은 String이여야 합니다.
   3. Object.getOwnPropertySymbols()
      - Object의 `Enumerable`, `Non-Enumerable`한 프로퍼티 모두를 배열로 반환합니다.
      - Property이름은 Symbol이여야 합니다.
   4. Reflect.ownKeys()
      - Object의 `Enumerable`, `Non-Enumerable`한 프로퍼티 모두를 배열로 반환합니다.
      - Property이름은 String, Symbol모두 가능합니다.

## 6.6.1 Property Enumeration Order

1. ES6는 오브젝트(Object)의 프로퍼티(Property)들이 어떤 순서로 열거(Enumerate)될지 정의를 합니다.
2. Object.keys(), Object.getOwnPropertyNames(), Object.getOwnPropertySymbols(), Reflect.ownKeys(), JSON.stringify() 모두 아래와 같은방법으로 순서를 정의합니다.
   1. 프로퍼티 이름이 음수가 아닌 정수의 이름으로 정의된 `String`이 가장 먼저 열거됩니다. (오름차순으로 정의됩니다.)
   2. 프로퍼티 이름이 음수, 실수인 나머지 `String`들이 열거됩니다. (이 프로퍼티들은 오브젝트들에 더해지는 순서로 열거됩니다.)
   3. 프로퍼티 이름이 `Symbol`인 오브젝트들이 열거 됩니다. (이 프로퍼티들은 오브젝트들에 더해지는 순서로 열거됩니다.)

## 6.7 Extending Objects

1. ES6부터, 오브젝트의 프로퍼티를 복사하는 Object.assign()이 생겼습니다.
   `Object.assign(target, ...sources)`
2. Object.assign()은 다음과 같은 특성을 가지고 있습니다.

   ```
   const target = { a: 1, b: 2 };
   const source = { b: 4, c: 5 };

   const returnedTarget = Object.assign(target, source);

   console.log(target);
   // expected output: Object { a: 1, b: 4, c: 5 }

   console.log(returnedTarget);
   // expected output: Object { a: 1, b: 4, c: 5 }
   ```

   1. target 오브젝트를 수정하고 반환합니다.
   2. source 오브젝트는 수정하지 않습니다.
   3. source objec들은 `enumerable`한 프로퍼티들을 모두 target object에 복사합니다.
   4. 첫 번째 source 오브젝트는 target object에서 중복된 프로퍼티들을 덮어쓰게 되고, 두 번째 source 오브젝트는 첫 번째 source object의 중복된 프로퍼티들을 덮어씁니다.
   5. Object.assign은 getter와 setter같은 property를 복사하지 않는 대신 copy도중 getter와 setter의 호출된 값을 복사합니다.

      > https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
      > It uses [[Get]] on the source and [[Set]] on the target, so it will invoke getters and setters. Therefore it assigns properties, versus copying or defining new properties.
      > -- In Mozila Documents..

      - 즉, Object.assign()은 copy할 때 source object의 getter를 호출하고, target object의 setter를 호출하여 프로퍼티를 할당하는 작업을 합니다.

        ```
        let count = 0
        const one = {}
        const two = {
        get count () { return count },
        set count (value) { count = value }
        }
        const three = Object.assign({}, one, two)

        console.log('two:', two)
        console.log('three:', three)
        ```

        ![accessors-not-copied](https://user-images.githubusercontent.com/30207544/104829214-0750e900-58b5-11eb-8766-3c5c662ee964.png)

   6. Spread Operator를 사용해서 보다 간결하게 Object.assign()과 같은 효과를 얻을수도 있습니다.

      ```
      o = Object.assign({}, defualuts, o);

      o = {...defaults, ...o};
      ```

## 6.8 Serializing Objects

1. Object Serialization은 Object의 상태를 복원 가능한 String으로 변환하는 과정입니다. 즉 네트워크를 통해서 데이터 구조나 오브젝트를 전송하기 위해서 적합한 형태로 변형하는 과정을 뜻합니다.
   > **Serialization **
   > The process whereby an object or data structure is translated into a format suitable for transferral over a network, or storage
2. JSON.stringify()와 JSON.parse() Javascript의 Object를 serialize하고 restore를 합니다.
   - JSON은 "Javascript Object Notation"의 약자로서 Javascript의 Object, Array 리터럴과 매우 비슷한 형태를 가지고 있습니다.
   ```
   let o = {x : 1 y : {z : [false, null, ""]}};
   let s = JSON.stringify(o); s == '{"x":1, "y":{"z":[false, null, ""]}}'
   let p = JSON.parse(s); p == {x : 1 y : {z : [false, null, ""]}}
   ```
3. JSON은 Javascript Syntax의 Subset이고, Javascript의 모든 값들을 나타내지는 못합니다.
   1. Serialize, Restore 가능한 오브젝트
      - Objects
      - Arrays
      - Finite numbers
      - True, False
      - NULL
   2. Serialize, Restore 불가능한 오브젝트
      - Function
      - RegExp
        ```
        JSON.stringify(/^[0-9]+$/) // "{}"
        ```
      - Error Objects
      - undefined
   3. Null로 Serialize됨
      - NaN
      - Inifinity
      - -Infinity
        ```
        JSON.stringify([NaN, null, Infinity]); // '[null,null,null]'
        ```
   4. 다른 형태로 Serialize되고, 복원되지 못하는 오브젝트
      - Date(ISO-formatted date strings로 Serialize되지만, 다시 Date object로 복원되지 못함)
        ```
        JSON.stringify(new Date) // "2014-07-03T13:42:47.905Z"
        ```
4. JSON.stringify(), JSON.parse() 모두 2번째 argument로 property를 Customizing할 수 있습니다.
   ```
   let foo = {foundation: 'Mozilla', model: 'box', week: 45, transport: 'car', month: 7};
   JSON.stringify(foo, ['week', 'month']);
   // '{"week":45,"month":7}'
   ```

## 6.9 Object Methods

- 자바스크립트의 모든 오브젝트들은 Object.prototype에서 프로퍼티들을 상속받습니다.
- 상속받은 프로퍼티들은 어디서나 사용가능합니다.
- 4가지 주요 오브젝트 매서드들에 대해서 알아봅니다.

### 6.9.1 The toString() Method

1. toString() 매서드는 오브젝트가 텍스트로 나타내져야할 때 혹은 스트링이 예상되는 곳에 호출되게 됩니다.
   - 예를 들어, string과 어떤 오브젝트가 +연산자로 합쳐질 때 toString()이 호출됩니다.
2. 디폴트로 toString() 매서드가 사용자가 정의되지 않았다면, toString()은 object의 타입(type)이 정의된"[object type]"을 반환합니다.
   ```
   const o = new Object()
   o.toString(); // [object Object]
   ```
3. 정의한 toString()을 매서드는 어떠한 값이 될 수 있고, 오브젝트에 관한 대표적인 정보를 나타내는것이 중요합니다.

   ```
   function Dog(name, breed, color, sex) {
      this.name = name;
      this.breed = breed;
      this.color = color;
      this.sex = sex;
   }

   theDog = new Dog('Gabby', 'Lab', 'chocolate', 'female');
   ```

   여기서 정의한 오브젝트에 대해 toString()매서드를 호출하게 된다면, Object로 상속받은 default value를 반환하게 됩니다.

   ```
   theDog.toString() // returns [object Object]
   ```

   그래서 default toString() 매서드위에 덮어쓰기 위해 dogToString()을 만들어서, Dog object에 관한 정보를 담을 수 있도록 합니다.

   ```
   Dog.prototype.toString = function dogToString() {
      return `Dog ${this.name} is a ${this.sex} ${this.color} ${this.breed}`;
   }
   ```

   그래서 theDog가 string이 예상되는 곳에 사용되어질 때 마다 자바스크립트는 자동으로 dogToString()매서드를 호출하게 됩니다.

   ```
   "Dog Gabby is a female chocolate Lab"
   ```

### 6.9.2 The toLocaleString() Method

1. toLocaleString() 매서드의 목적은 어떤 오브젝트의 여러 특색에 맞는(localized) 스트링형태를 표현하기 위함입니다.

2. The Date and Number 클래스는 숫자와 날짜, 시간을 여러 특색에 맞는 형태의 스트링으로 표현하기 위해 toLocaleString()을 정의해놨습니다.

   ```
   const event = new Date(Date.UTC(2012, 11, 20, 3, 0, 0));

   // 영국의 일반적인 시간 표현
   console.log(event.toLocaleString('en-GB', { timeZone: 'UTC' }));
   // 20/12/2012, 03:00:00

   // 한국의 일반적인 시간 표현
   console.log(event.toLocaleString('ko-KR', { timeZone: 'UTC' }));
   // 2012. 12. 20. 오전 3:00:00
   ```

   ```
   const number = 123456.789;

   // toLocaleString()을 통한 일본의 화폐 표현
   console.log(number.toLocaleString('ja-JP', { style: 'currency', currency: 'JPY' }))
   // ￥123,457

   // toLocaleString()을 통한 독일의 화폐 표현
   console.log(number.toLocaleString('de-DE', { style: 'currency', currency: 'EUR' }));
   // 123.456,79 €

   ```

3. 배열 또한, toLocaleString()을 정의하여 배열의 원소들이 특색에 맞는(localized) 형식을 나타낼 수 있도록 하였습니다.

   ```
   const prices = ['￥7', 500, 8123, 12];
   prices.toLocaleString('ja-JP', { style: 'currency', currency: 'JPY' });

   // "￥7,￥500,￥8,123,￥12"

   ```

4. Object에 의해 정의된 Default toLocaleString() 매서드는 어떠한 특색에 맞는 스트링을 반환하지 않고, toString()을 반환합니다.

### 6.9.3 The valueOf() Method

1. 자바스크립트는 오브젝트를 원시형태의 값으로 변환하기 위해 valueOf() 매서드를 호출합니다.
2. 자바스크립트가 어떤 오브젝트의 원시형태의 값이 필요할 때 자동으로 호출하기 때문에, 프로그래머가 직접 호출하는 경우는 잘 없습니다.
   ```
   console.log(+"5")
   // 5 (string to number)
   console.log(+"")
   // 0 (string to number)
   console.log(+new Date())
   // same as (new Date()).getTime()
   ```

### 6.9.4 The toJSON() Method

1. toJSON() 매서드는 Object.prototype에 정의되어 있지 않습니다.
2. toJSON() 매서드는 JSON.stringify() 매서드가 오브젝트에 대해 Serialize를 할 때마다 찾습니다.
3. toJSON() 매서드가 serialized될 오브젝트에 존재한다면 오브젝트가 serialized될 때 이 매서드를 호출하여 serialized된 값을 반환합니다.
4. 대표적으로 Date 클래스가 toJSON() 매서드를 정의하여 날짜값의 serialized된 스트링 표현을 하게 됩니다.

   ```
   const event = new Date('August 19, 1975 23:15:30 UTC');

   const jsonDate = event.toJSON();

   console.log(jsonDate);
   // expected output: 1975-08-19T23:15:30.000Z
   ```

## 6.10 Extended Object Literal Syntax

- 최근 자바스크립트 버전에는 오브젝트 리터럴을 초기화하고 정의하는 새로운 문법을 가지고 있습니다. 이 문법들에 대해 알아보겠습니다.

### 6.10.1 Shorthand Properties

```
let x = 1, y = 2
let o = {
   x : x,
   y : y
}
```

- 어떤 값이 저장된 변수가 존재하고, 오브젝트의 프로퍼티를 이 변수들이 저장된 값으로 나타내고 싶다면 우리는 다음과 같이 더 간결하게 코드를 쓸 수 있습니다.

```
let x = 1, y = 2
let o = {x, y}
```

### 6.10.2 Computed Property Names

1. 이 Syntax는 Expression을 Brackets `[]`안에서 사용할 수 있도록하여, 프로퍼티 네임으로 값이 계산되고 사용되어질 수 있게 하였습니다.

   ```
   let i = 0
   let a = {
      ['foo' + ++i]: i,
      ['foo' + ++i]: i,
      ['foo' + ++i]: i
   }

   console.log(a.foo1) // 1
   console.log(a.foo2) // 2
   console.log(a.foo3) // 3

   //ES2015이전 CPN Syntax를 사용하기 전
   function objectify (key, value) {
      let obj = {}
      obj[key] = value
      return obj
   }

   objectify('name', 'Tyler') // { name: 'Tyler' }

   //ES2015이후 CPN Syntax를 사용한 후
   function objectify (key, value) {
      return {
         [key]: value
      }
   }

   objectify('name', 'Tyler') // { name: 'Tyler' }

   ```

### 6.10.4 Spread Operator

1. ES2018부터 `...`(spread operator)을 사용하여 오브젝트의 프로퍼티를 복사할 수 있습니다. 전에 봤던 Object.assign()보다 더 간결한 Syntax를 사용할 수 있습니다.

   ```
   let obj1 = { foo: 'bar', x: 42 };
   let obj2 = { foo: 'baz', y: 13 };

   let clonedObj = { ...obj1 };
   // Object { foo: "bar", x: 42 }

   let mergedObj = { ...obj1, ...obj2 };
   // Object { foo: "baz", x: 42, y: 13 }
   ```

2. Spread Operator는 상속받은 오브젝트의 프로퍼티는 반환하지 않습니다.
   ```
   let o = Object.create({x:1});
   let p = { ...o};
   p.x // undefined
   ```
3. Spread Operator는 꾀나 상당한 양의 워크로드를 가지고 있습니다.
   - Object가 n개의 프로퍼티를 가지고 있다면, spread operator가 이 오브젝트를 모두 펼치려면 `O(n)`만큼의 복잡도를 가지고 있습니다.
   - 이 뜻은 loop나 재귀함수안에서 spread operator를 사용한다면 `O(n^2)` 이상의 비효율적인 알고리즘을 쓸 수 있음을 의미합니다.

### 6.10.5 Shorthand Methods

1. 6.10.1에서 봤듯이 오브젝트의 매서드 역시 더욱 간결하게 쓸 수 있습니다.

   ```
   // ES6 이전 매서드 정의
   let squrae = {
      area : function() { return this.side * this.side},
      side : 10
   }
   square.area() => 100

   // ES6 이후 매서드 정의
   let squrae = {
      area() { return this.side * this.side},
      side : 10
   }
   square.area() => 100
   ```

### 6.10.6 Property Getters and Setters

1. 자바스크립트는 getter와 setter라는 accessor 프로퍼티를 지원합니다.
2. `get` syntax는 오브젝트의 프로퍼티를 함수에 bind하여 함수가 호출될 때 프로퍼티를 가져오도록 합니다(get).
3. `set` syntax는 오브젝트의 프로퍼티를 함수에 bind하여 함수가 호출될 때 프로퍼티를 설정하도록 합니다(set).
4. 프로퍼티가 `getter`, `setter` 2가지 매서드를 모두 가지고 있다면 read/write 특성을 가지고 있는 것입니다.
5. 프로퍼티가 `getter`만 가지고 있따면 read-only특성을 가지고 있는 것입니다.
6. 액세서 매서드(Accessor Method)는 다양하게 계산되어 정의하는 프로퍼티를 설정 또는 정의할 때 주로 사용됩니다.

   ```
   let p = {
      x : 1.0,
      y : 1.0,

      // R is read-write accessor
      get r() {return Math.hypot(this.x, this.y);}
      set r(newvalue) {
         let oldvalue = Math.hypot(this.x, this.y)
         let ratio = newvalue/oldvalue;
         this.x *= ratio;
         this.y *= ratio;
      }

      //Read-only accessor
      get theta() { return Math.atan2(this.y, this.x)}
   }

   p.r => Math.SQRT2
   p.theta  => Math.PI/4
   ```

7. 액세서 프로퍼티는 상속받을 수 있습니다. 그래서 6번에서 정의한 액세서 프로퍼티를 그대로 상속받아 다음과 같이 사용할 수 있습니다.
   ```
   let q = Object.create(p);
   q.x=3;
   q.y=4;
   q.r
   q.theta
   ```
