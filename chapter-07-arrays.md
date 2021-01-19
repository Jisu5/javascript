# Arrays

1. Javascript의 배열(Arrays)은 정렬된 값의 모임(Collection)입니다.
2. Javascript의 배열은 untyped입니다.
   - 배열의 엘리먼트(Element)는 어떤 타입이든지 될 수 있습니다.
   - 배열의 엘리먼트는 오브젝트(Object)가 올수도 있고 다른 배열이 올수도 있습니다.
3. Javascript의 배열은 0부터 시작(zero-based)하며, 32-bit 인덱스를 사용합니다.
   - 첫 엘리먼트의 인덱스는 0이며, 사용할 수 있는 가장 높은 인덱스는 4294967294(2^32-2)입니다.
   - 그래서 배열은 최대 4,294,967,295의 엘리먼트를 저장할 수 있습니다.
4. Javascript의 배열은 동적(Dynamic)입니다.
   - Javascript의 배열은 필요한 만큼 증가, 감소할 수 있습니다.
   - Javascript의 배열은 고정 크기를 선언할 필요도 없고, 크기가 변할 때 마다 재할당 할 필요도 없습니다.
5. Javascript의 배열은 희소성(Sparse)을 가지고 있습니다.
   - 즉, 인덱스들끼리 인접할 필요가 없고, 인덱스 사이에 빈 인덱스가 있을 수 있습니다.
     ```
     const sparseArray = [1,2,,,3]
     // 0 : 1
     // 1 : 2
     // 4 : 3
     ```
6. Javascript의 배열은 모두 **length** 프로퍼티(property)를 가지고 있습니다.
7. Javascript의 배열은 자바스크립트의 특별한 형태의 오브젝트입니다. 그리고 배열의 인덱스들은 단지 정수라는 프로퍼티보다는 더 특별한 의미를 가지고 있습니다.
   - 이러한 점은 배열이 인덱스를 통해 값을 접근하는데 최적화할 수 있도록 만들어 줍니다.
8. Javascript의 배열은 **Array.prototype**의 프로퍼티를 상속(Inherit)합니다.
   - 이 프로퍼티들은 주로 배열을 조작하는 매서드를 정의합니다.
   - 주로 이 Method들은 **generic**이기 때문에 일반적인 어레이(Array)뿐만 아니라 Array-like 오브젝트들에도 작동합니다.
9. Javascript의 스트링(String)은 케릭터(Character)들의 배열처럼 작동합니다.

## 7.1 Creating Arrays

- 자바스크립트의 배열은 다음과 같은 방법으로 배열을 생성할 수 있습니다.
  1.  Array Literals
  2.  ... Spread operator
  3.  The Array() 생성자(constructor)
  4.  The Array.of() and Array.from() 팩토리 매서드(factory methods)

## 7.1.1 Array Literals

1.  배열 리터럴 (Array Literals)을 가장 심플하게 배열을 생성하는 방법입니다.
    ```
    let empty = [];
    let primes = [2,3,5,7,11];
    let misc = [1.1, true, "a"];
    ```
2.  배열 리터럴이 (Array Literal) 상수일 필요는 없습니다. 어느 Expression이든 올 수 있습니다.
    ```
    let base = 1024;
    let table = [base, base+1, base+2, base+3];
                //1024,  1025,  1026,  1027
    ```
3.  배열 리터럴은 오브젝트와 다른 배열 리터럴을 원소로 쓸 수 있습니다
    ```
    let b = [[1, {x:1, y:2}], [2, {x:3, y:4}]]
    ```
4.  배열 리터럴에서 아무런 값 없이 콤마(comma)를 연속으로 넣으면 희소배열(Sparse Array)가 됩니다. 값이 없는 엘리먼트는 undefined가 됩니다.
    ```
    let count = [1,,3]
    // 0 : 1
    // 2 : 3
    let undefs = [,,]
    // No elements, length : 2
    ```
