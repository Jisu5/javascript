## Chapter 11. The JavaScript Standard Library

* 자바스크립트에는 number, string, object 그리고 array같은 중요한 data type도 존재하지만, 다양한 상황에서 유용하게 사용될 수 있는 standard library도 존재합니다. 이들은 class와 function으로 이루어진 javascript의 한 요소이며, web browser와 Node에서 모두 사용될 수 있습니다.

* 이 장에서는 다음과 같은 절로 구성되어 있습니다.
    1. Set and Map classes
    2. Array-like objects (Typed Arrays)
    3. Regular Expressions and the RegExp class
    4. The Date class
    5. The Error class
    6. The JSON object
    7. The Intl object and the classes
    8. The Console object
    9. The URL class
    10. setTimeout() and related function


## 11.1 Sets and Maps


## 11.1.1 The Set Class 
* Set은 Array처럼 값들의 집합이지만, set은 인덱스가 존재하지 않습니다. 그리고 **중복을 허용하지 않습니다.**
* 다음과 같이 Set을 생성할 수 있습니다.
    ```
    let s = new Set();
    let t = new Set([1, s]);
    ```
* Set 생성자안의 인자는 꼭 array일 필요는 없습니다. Iterable object이기만 (Set object를 포함한) 하면 됩니다.
    ```
    let t = new Set(s);
    let unique = new Set("Mississippi");
    ```
* set의 `size` 프로퍼티는 array의 `length`프로퍼티와 비슷합니다.
    ```
    unique.size   // => 4
    ```
* Set을 생성할 때 꼭 값을 초기화할 필요는 없습니다. 언제든지 `add()`, `delete()`, `clear()`를 통해 값을 더하고 지울 수 있습니다. 
    ```
    let s = new Set()
    s.add(1)
    s.add(true)
    s.delete(true)
    s.add([1,2,3])
    s.clear()
    ```
    * `add()`는 하나의 인자를 받습니다. 만약에 array가 인자로 들어온다면, array의 원소들을 나눠서 넣는게 아니라 array 전체 하나만 넣습니다. 만약에 여러 값을 set에 추가하고 싶다면, `s.add('a').add('b').add('c')`처럼 사용해야 합니다.
    * `delete()`는 하나의 인자를 받아 하나의 원소를 지웁니다. `add()`와 달리, boolean을 리턴합니다. 만약 지운 값이 set의 member라면 `true`를 반환하고, 그렇지 않다면 `false`를 반환합니다.
    * set은 `===`operator가 작동하듯이 강한 eqeuailty check를 합니다. 그래서 set은 number 1과  string "1"을 동시에 가질 수 있습니다. 그래서 아래의 코드에서 `[1,2,3]` 같은 array을 변수를 통해 참조하지 않는다면 `delete()`는 false를 반환합니다. 
        ```
        let s = new Set()
        s.add([1,2,3])       
        s.delete([1,2,3])    => false

        let t = new Set()
        const newItem = [1,2,3]
        s.add(newItem)
        s.delete(newItem)    => true
        ```
* 보통 Set은 element를 추가하거나 삭제하는데 사용하지 않고 특정 값이 set의 member인지 체크를 하는데 사용합니다. 이는 set이 membership testing을 하는데에 최적화가 되어있기 때문입니다. 아무리 많은 member들을 set이 갖고 있다하더라도 set의 `has()` method는 똑같이 빠르게 작동합니다. `includes()` method도 member의 유무를 testing하는데 사용되지만, array의 size에 따라 시간 복잡도는 비례합니다.
    ```
        let oneDigitPrimes = new Set([2,3,5,7])
        oneDigitPrimes.has(2)  => true
        oneDigitPrimes.has(3)  => true
        oneDigitPrimes.has(4)  => false
    ```
* Set은 Iterable이기 때문에 원소를 순회하기 위해서 `for/of` loop를 사용할 수 있습니다.
    ```
    let sum = 0;
    for (let p of oneDigitPrimes) {
        sum += p
    }
    sum
    ```
* Set은 Iterable이기 때문에 `...`operator를 사용해서 array로 복사할 수 있습니다.
    [...oneDigitPrimes]    //  => [2,3,5,7]
    Math.max(...oneDigitPrimes)
* Set은 종종 "unordered collections"으로 생각하지만, 이는 사실이 아닙니다. Javascript의 set은 array처럼 index가 없을뿐입니다. Set은 정확히 element들이 들어간 순서를 기억하고 있기때문에, set의 element들을 순회한다면 가장 먼저 들어간 원소부터 시작하여, 마지막에 들어간 원소를 마지막으로해서 순회하게 됩니다.
* Set은 `forEach()` method를 갖고 있지만 array의 `forEach()`와는 조금 다릅니다. array의 `forEach()`는 2번째 인자로 인덱스를 받지만, Set은 인덱스가 없기 때문에 array처럼 사용할 수 없습니다.



## 11.1.2 The Map Class
* Map 오브젝트는 key와 그에 상응하는 value로 구성되어 있습니다. map은 array와 비슷하지만, array는 정수의 순열을 key로 사용하지만, map은 임의의 값들을 key로 사용할 수 있다는 점이 다릅니다. array만큼은 아니지만 map역시도 size가 아무리 크다고 하더라도 key를 사용해서 값들을 빠르게 찾을 수 있습니다. 
* 다음과 같이 `Map()` 생성자를 사용해서 새로운 instance를 생성할 수 있습니다.
    ```
    let m = new Map();
    let n = new Map([
        ["one", 1],
        ["two", 2]
    ])
    ```
* `Map()` 생성자의 인자는 [key, value] array로 구성된 iterable object이여야 합니다. `Map()`생성자를 사용해서 다른 map object를 copy할수도 있습니다. Object의 프로퍼티 이름과 값을 빌려서 copy를 할수도 있습니다.
    ```
    let copy = new Map(n)
    let o = { x:1, y:2 }
    let p = new Map(Object.entries(o))
    ```
* `get()`과 map의 key를 사용해서 값을 찾을수 있으며, `set()`을 통해서 새로운 key/value pair를 추가할 수 있습니다. `set()`을 사용해서 이미 존재한 key를 추가한다면, 새로운 key/value pair가 들어가는 것이 아니라 그 위에 덮어쓰는 결과를 보여줍니다.
* Set처럼 `has()`, `clear()`, `size` 프로퍼티들을 사용할 수 있습니다.
    ```
    let m = new Map();
    m.size
    m.set("one", 1)
    m.set("two", 2)
    m.size
    m.get("two")
    m.get("three")  // => undefined
    m.set("one", true)
    m.has("one")
    m.delete("three")
    m.clear()
    ```
* Set처럼 Map도 add() method를 chained형태로 사용할 수 있습니다.
    ```
    let m = new Map().set("one", 1).set("two", 2).set("three", 3)
    ```
* `...`operator를 사용해서 Map object를 순회하면, `Map()`생성자에 넣은것처럼 array의 array를 반환합니다.
    ```
    let m = new Map([
        ["x", 1], 
        ["y", 2]
    ])
    [...m]        // => [["x", 1], ["y", 2]]
    ```

* `for/of` loop를 통해서 map을 순회할 때, destructuring assignment를 사용하는 것이 관습입니다.
    ```
    for(let [key, value] of m){
        ...
        ...
    }
    ```
* Map class도 Set과 같이 insert한 순서를 가지고 있습니다.
* `keys()`, `values()` method를 통해 각각 key, value만 순회할 수 있습니다.
    ```
    [...m.keys()]
    [...m.values()]
    ```

## 11.1.3 WeakMap and WeakSet
* WeakMap class는 Map class의 subclass가 아니라 변형된 형태입니다. Map과 달리 garbage collection이 사용하지 않는 오브젝트의 메모리를 회수하도록 합니다.
* Weakmap과 Map의 차이점은 다음과 같습니다.
    * WeakMap의 key는 반드시 object나 array여야 합니다. primitive value는 garbage collection의 대상이 아니기 때문에 key로 사용될 수 없습니다.
    * WeakMap은 `get()`, `set()`, `has()`, `delete()` method만 구현되어 있습니다. 특히 WeakMap은 iterable이 아니기 때문에 keys(), values(), forEach() 매서드를 정의하지 않습니다.
    * WeakMap의 size는 garbage collection이 작동할때마다 언제든지 바뀔 수 있기 때문에 WeakMap은 size 프로퍼티를 가지고 있지 않습니다.
* WeakMap의 주된 사용목적은 메모리 leak 없이 object와 value를 사용할 수 있게 하는 것입니다. 가령 오브젝트를 인자로 받는 함수를 쓰고, 그 오브젝트에 대해 시간이 많이 걸리는 연산을 사용한다고 했을 때, 연산된 결과를 재사용하기 위해 cache를 하고 싶을 것입니다. 만약에 Map object로 cache를 구현했다면 object가 회수되지 않을 것입니다. 하지만 WeakMap을 사용함으로써 이러한 문제를 해결할 수 있습니다.

## 11.2 Typed Arrays and Binary Data
* Javascript의 array는 원소로 어느 type이나 가질 수 있고, 동적으로 size를 늘리고 줄일 수 있습니다. Javascript의 여러 자료구조는 최적화되어 구현되었기 때문에 빠르게 동작하지만, 그럼에도 C나 Java같은 low-level 언어의 구현보다는 부족한면이 있습니다.
* ES6의 Typed array는 C나 Java의 그것들과 많이 비슷한 점이 있습니다. Typed array는 기존 array와 다른점이 많습니다.
    * typed array의 모든 원소는 `number`입니다. typed array는 signed, unsigned, floating point같은 type을 지정할 수 있게 하고, 8bits 부터 64bits까지 array의 size를 지정할 수 있습니다.
    * typed array를 생성할 때 반드시 length를 지정해야하고, 이 length는 절대 바뀔 수 없습니다.
    * typed array는 언제나 원소를 0으로 초기화됩니다.

## 11.2.1 Typed Array Types
* TypedArray라는 class가 정의된것은 아니고, 대신에 11개의 서로 다른 원소 타입과 생성자를 가진 array가 있습니다.
    | Constructor      | Numeric type |
    | ----------- | ----------- |
    | `Int8Array()`      | signed bytes       |
    | `Uint8Array()`   | unsgined bytes        |
    | `Uint8ClampedArray()`   | unsgined bytes without rollover        |
    | `Int16Array()`   | signed 16bit short integers        |
    | `Uint16Array()`   | unsigned 16-bit short integers        |
    | `Int32Array()`   | signed 32-bit integers        |
    | `Uint32Array()`   | unsigned 32-bit integers        |
    | `BigInt64Array()`   | signed 64-bit BigInt values (ES2020)        |
    | `BigUint64Array()`   | unsigned 64-bit BigInt values (ES2020)        |
    | `Float32Array()`   | 32-bit floating-point value        |
    | `Float64Array()`   | 64-bit floating-point value : a regular JavaScript number        |
* Int로 시작하는 type들은 1,2,4bytes의 signed integer들을 가질 수 있습니다.
* Uint로 시작하는 type들은 1,2,4bytes의 unsigned integer들을 가질 수 있습니다.
* BigInt, BigUint type들은 64-bit의 integer를 가질 수 있습니다.
* Float으로 시작하는 type은 floating point number를 가질 수 있습니다.
    * `Float64Array`의 원소들은 Javascript의 number와 같은 type입니다.
    * `Float32Array`의 원소들은 주로 C와 Java에서 float에 해당되며, `Float62Array`보다 낮은 precision을 갖고 있지만 더 적은 memory를 가지고 구성할 수 있습니다.

* Uint8ClampedArray는 Uint8Array의 특수한 형태입니다. 두 type모두 unsigned bytes를 갖고 0~255까지 나타낼 수 있지만 Uint8Array에 0보다 작은 원소나 255보다 큰 원소를 넣는다면 값이 다시 0부터 시작하는 값을 갖게 됩니다(Wrap around). 이렇게 low-level에서 computer memory가 작동하는 방법입니다. 반면에 UintClapedArray는 0보다 작은 원소나 255보다 큰 원소를 넣는다면 0 또는 255만 반환합니다. 

## 11.2.2 Creating Typed Arrayrs
* 원하는 원소의 수(length)를 인자로 넘겨서 typed array를 생성할 수 있습니다.
    ```
    let bytes = new Uint8Array(1024)
    let matrix = new Float64Array(9)
    let point = new Int16Array(3)
    let rgba = new Uint8ClampedArray(4)
    let sudoku = new Int8Array(81)
    ```
* typed array를 생성할 때, 모든 원소들은 `0`, `0n`, `0.0`으로 초기화되는 것이 보장됩니다. 
* typed array는 `Array.from()`, `Array.of()`처럼 작동하는 `from()`과 `of()`라는 factory method를 가지고 있습니다. 
    ```
        let white = Uint8ClampedArray.of(255,255,255,0)
    ```
* 생성자, `from()` 매서드 모두 기존 typed array를 복사할 수 있습니다. 그리고 이를 통해 type을 바꿀 수 있습니다.
    ```
        let ints = Uint32Array.from(white)
    ```
* 기존 array, iterable, array-like 오브젝트로부터 새로운 typed array를 만들 때, type의 constraint를 만족하기 위해 원소들이 truncated 될 수 있습니다. 이때 error나 warning이 나타나지 않음에 주의해주세요.
    ```
    // Floats 값은 int로 truncated됩니다.  
    Uint8Array.of(1.23, 2.99, 450000) => new Uint8Array([1,2,200])
    ```
* ArrayBuffer type을 사용해서 typed array를 생성할 수 있습니다. ArrayBuffer는 단순히 chunk of memory입니다. 원하는 만큼의 memory의 byte를 인자로 넘겨 할당할 수 있습니다.
    ```
    let buffer = new ArrayBuffer(1024*1024)
    buffer.byteLength // 1024 * 1024
    ```
    * ArrayBuffer 클래스는 할당한 byte를 직접 읽거나 쓰도록 허용하지 않습니다. 하지만 이를 이용하여 buffer의 메모리를 사용하는 typed array를 생성할 수 있습니다. typed array와 ArrayBuffer 통해 memory를 읽고 쓸 수 있습니다.

    * 이렇게 하기 위해 ArrayBuffer를 첫번째 인자로 해서 typed array 생성자를 호출합니다. 
    * 2번째 인자로는 array buffer의 byte offset을 넣고, 3번째 인자로 array의 length를 넣어줍니다.
    * 2,3번째 인자는 optional이지만, 두 인자를 넣지 않는다면, array는 array buffer에 있는 memory를 모두 사용합니다.
    * 3번째 인자를 제외한다면, array는 buffer의 시작위치부터 사용가능한 모든 메모리를 사용합니다.
![스크린샷, 2021-03-23 08-25-46](https://user-images.githubusercontent.com/30207544/112070642-71398780-8bb1-11eb-8ddc-1a13cc336014.png)

    
    ```
    let asbytes = new Uint8Array(buffer);
    let asints = new Int32Array(buffer)
    let lastK = new Uint8Array(buffer, 1024*1024)
    let ints2 = new Int32Array(buffer, 1024, 256)
    ```
    * 4개의 typed array 모두, ArrayBuffer에 의해서 메모리가 나타내어지는 서로 다른 관점을 보여줍니다.

## 11.2.3 Using Typed Arrays
* typed array를 생성하면, object처럼 square-bracket을 사용해서 읽고 쓸 수 있습니다. 
    ```
    function sieve(n) {
        let a = new Uint8Array(n+1)       
        let max = Math.floor(Math.sqrt(n));
        let p = 2;
        while(p <= max){
            for(let i=2*p; i<=n; i+=p)
                a[i] = 1
            while(a[++p])
        }
        while(a[n]) n--;
        return n;
    }
    ```
    * 이 함수는 가장 큰 prime number를 계산합니다. 일반적인 array를 사용하는것과 똑같이 작동하지만 이 코드에서는 Uint8Array를 사용해서 보통 Array보다 4배 빠르고, 8배 적은 메모리를 사용합니다.
    * typed array는 true arrayr는 아니지만, 대부분의 array의 method를 재구현 했습니다. 그래서 일반적인 array의 method를 사용하듯이 사용할 수 있습니다.
    * typed array는 고정된 length를 가지기 때문에, `length` 프로퍼티는 read-only이고 push(), pop(), unshift()와 같은 array의 length를 바꾸는 method들은 구현되어 있지 않습니다.
    * sort(), reverse(), fill()과 같이 length는 바꾸지 않으면서 array안의 content들을 바꾸는 method들은 구현되어 있습니다.
## 11.2.4 Typed Array Methods and Properties
* typed array도 자신들만의 method를 가지고 있습니다. `set()` method는 typed array나 일반 array를 복사해서 여러 원소를 typed array에 넣을 수 있습니다.
    ```
    let bytes = new Uint8Array(1024)
    let pattern = new Uint8Array([0,1,2,3])

    bytes.set(pattern)
    bytes.set(pattern, 4)
    bytes.set([0,1,2,3], 8)
    bytes.slice(0,12)
    ```
    * `set()`method는 array 또는 typed array를 인자로 받고, 2번째 인자로 원소의 offset을 받습니다. (2번째 인자의 default는 0)

* `subarray()`method는 array의 어느 부분만 return할 수 있습니다.
    ```
    let ints = new Int16Array([0,1,2,3,4,5,6,7,8,9])
    let last3 = ints.subarray(ints.length-3, ints.length)   //마지막 3개 원소
    last3[0]         // => 7
    ```

## 11.2.5 DataView and Endianness
* Typed array는 8, 16, 32, 64bits chunks들의 순서대로 보여줍니다. 이것은 Endianness(엔디언)과도 연관이 있습니다. 엔디언은 byte가 정렬되는 순서를 뜻합니다.
* 메모리의 저장 공간은 인덱스, 또는 주소를 가지고 있습니다. 각각의 바이트는 8비트 숫자(0x00 이상, 0xff 이하)를 저장할 수 있으므로, 그보다 큰 숫자에 대해서는 두 개 이상의 바이트가 필요합니다. 여러 개의 바이트를 정렬하는, 지금까지 가장 많이 쓰이는 방법은 모든 Intel 프로세서가 사용하는 리틀 엔디언입니다.  
* 리틀 엔디언은 작은 단위부터 정렬하는 방식으로, 가장 작은 단위의 바이트가 맨 앞 혹은 앞쪽 주소를 차지합니다. 이 방식은 유럽식 날짜 표기(31-12-2050)에 대입할 수 있습니다.     
* 자연스럽게, 빅 엔디언은 그 반대 순서를 나타내며 ISO 날짜 표기(2050-12-31)와 같습니다. 빅 엔디언은 "네트워크 바이트 순서"라고도 부르는데, 대부분의 인터넷 표준은 데이터의 저장 방식에 빅 엔디언을 요구하기 때문입니다. 이는 표준 UNIX 소켓 단계부터, 표준화 웹 이진 데이터 구조까지 올라갑니다.
* 다음은 숫자 0x12345678 (10진수 305 419 896)으로 나타낸 예제입니다.
    * 리틀 엔디언: 0x78 0x56 0x34 0x12
    * 빅 엔디언: 0x12 0x34 0x56 0x78
* 일반적으로 외부 데이터와 관련되어 작업을 할 때, data를 개별적인 bytes의 array로 보기 위해  Int8Array와 Uint8Array를 사용할 수 있습니다. 이 때 다른 typed array를 사용할 수 없습니다. 대신에 DataView class를 사용할 수 있습니다. DataView Class는 ArrayBuffer에서 사용되는 value를 읽고 쓰는 매서드를 정의합니다.
    ```
    let view = new DataView(bytes.buffer,
                        bytes.byteOffset,
                        bytes.byteLength)
    let int = view.gietInt32(0)
    int = view.getInt32(4,false)
    int = view.getUint32(8,true)
    view.setUint32(8, int, false)
    ```
