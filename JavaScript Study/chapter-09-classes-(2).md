# CHAPTER 9. Classes (2)

## 9.5 Subclasses

- 객체 지향 프로그래밍에서 class B는 다른 class A로 확장하거나 하위 클래스가 될 수 있다.

  => A는 superclass, B는 subclass라고 부른다.

- class B의 인스턴스들은 class A의 메소드를 상속한다.

- class B는 자기 자신만의 메소드를 정의할 수 있다. => class A에서 정의한 동일한 이름의 메소드를 override 할 수 있다.

- 대부분의 경우 B의 메소드가 A의 메소드를 override한다면, overriding 메소드는 A의 overridden 메소드를 호출해야 합니다.

  => subclass 생성자 `B()`가 인스턴스 초기화를 완료하기위해 superclass 생성자 `A()`를 호출하는 것과 비슷하다.

### 9.5.1 Subclasses and Prototypes

- Example 9.2에서의 Range 클래스의 subclass인 Span을 정의한다. Range와 같이 동작하지만 start와 end로 초기화 하는 것 대신, start와 span으로 초기화한다.
- span 인스턴스는 커스터마이징한 toString() 메소드를 Span.prototype부터 상속한다. 
  Range의 subclass가 되기위해 Range.prototype으로부터 메소드들도 상속받는다.

```javascript
// Example 9.2
function Range(from, to) {
  this.from = from;
  this.to = to;
}
Range.prototype = {
  includes: function(x) { return this.from <= x && x <= this.to; },
  [Symbol.iterator]: function*() {
    for (let x = Math.ceil(this.from); x <= this.to; x++) yield x;
  },
  toString: function() { return "(" + this.from + "..." + this.to + ")"; }
}
```

```javascript
// Constructor function
function Span(start, span) {
  if (span >= 0) {
    this.from = start;
    this.to = start + span;
  } else {
    this.to = start;
    this.from = start + span;
  }
}

// Span Prototype inherits from Range Prototype
Span.prototype = Object.create(Range.prototype);

// We don't want to inherits Range.prototype.constructor.
// So we define our own constructor property.
Span.prototype.constructor = Span;

// Span override toString() method
Span.prototype.toString = function() {
  return `(${this.from}...+${this.to - this.from})`;
}
```

- Span을 Range의 subclass로 만들기 위해 Range.prototype으로부터 상속받은 Span.prototype을 선언한다.
- Span 생성자로 만들어진 객체는 Span.prototype을 상속받고, Span.prototype은 Range.prototype으로부터 상속받는다. => Span객체는 Span.prototype과 Range.prototype 모두에게서 상속받는다.
- Span() 생성자는 Range() 생성자와 마찬가지로 from과 to 프로퍼티를 set하기 때문에 새로운 객체를 초기화 하기위해 Range() 생성자를 호출할 필요는 없다. 이와 비슷하게 `toString()`메소드도 Range의 `toString()`메소드를 호출할 필요없이 수행가능하다.



### 9.5.2 Subclass with extends and super

- ES6부터 `extends` 키워드를 통해 쉽게 superclass를 생성할 수 있다. 

```javascript
class EZArray extends Array {
  get first() { return this[0]; }
  get last() { return this[this.length-1]; }
}

let a = new EZArray();
a instanceof EZArray	// true
a instanceof Array	// true
a.push(1,2,3,4)	// a.length = 4
a.pop()	// => 4
a.first	// => 1
a.last	// => 3
a[1]	// => 2
Array.isArray(a)	// true
EZArray.isArray(a)	// true
```

- EZArray subclass는 간단한 두개의 getter 메소드를 가진다.  EZArray의 인스턴스는 기존의 Array와 같게 동작하고 push(), pop(), length와 같은 메소드와 프로퍼티를 상속받아 사용할 수 있다.
- EZArray에서 정의한 `first`, `last` getter도 사용할 수 있고, 인스턴스 메소드 뿐 아니라 `Array.isArray()`같은 static method도 상속된다. 



```javascript
class TypedMap extends Map {
  constructor(keyType, valueType, entries) {
    // If entries are specified, check their types
    if (entries) {
      for(let [k, v] of entries) {
        if (typeof k !== keyType || typeof v !== valueType) {
          throw new TypeError(`Wrong type for entry [${k}, ${v}]`);
        }
      }
    }
 		// Initialize the superclass with the (type-checked) initial entries
    super(entries);
    // And then initialize this subclass by storing the types
    this.keyType = keyType;
    this.valueType = valueType;
  }
  // Now redefine the set() method to add type checking for any
  // new entries added to the map.
  set(key, value) {
  // Throw an error if the key or value are of the wrong type
  if (this.keyType && typeof key !== this.keyType) {
    throw new TypeError(`${key} is not of type ${this.keyType}`);
  }
  if (this.valueType && typeof value !== this.valueType) {
    throw new TypeError(`${value} is not of type ${this.valueType}`);
  }
  // If the types are correct, we invoke the superclass's version of
  // the set() method, to actually add the entry to the map. And we
  // return whatever the superclass method returns.
  return super.set(key, value);
  }
}
```

- TypedMap() 생성자의 첫 두 인자는 key와 value의 타입이며, typeof 연산자로 반환받는 "number"이나 "boolean"같은 문자열이다.

  세번째 인자는 [key, value] 배열이다.

- 생성자는 먼저 type이 맞는지 유효성 검사를 하고, `super`키워드로 superclass 생성자를 호출한다. `Map()`생성자는 1개의 optional 인자 ([key, value] 배열)을 가지기 때문에, Map() 생성자를 호출할 때, TypedMap() 생성자의 세번째 인자를 넘겨준다.

- superclass 생성자로 superclass의 state를 초기화 한 다음, subclass의 state를 초기화 한다. (this.keyType & this.valueType)

- **`super()` 생성자를 사용할때 중요한 규칙이 있다.**
  - `extends`키워드로 클래스를 정의할 때, 그 클래스의 생성자는 superclass의 생성자를 호출하기 위해 `super()`를 사용해야 한다.
  - subclass에 생성자를 정의하지 않으면 자동으로 정의된다. 이렇게 생성된 생성자는 받은 값들을 전부 `super()`에 그대로 전달한다.
  - `super()`로 superclass 생성자를 호출하기 전에 `this` 키워드를 사용하지 않는다. => superclass를 초기화한 다음 subclass 초기화가 되게 한다.
  - 표현식 `new.target`은 함수가 `new`키워드를 통해 호출되지 않으면 undefined이다. 하지만 생성자 함수에서는 `new.target` 호출된 생성자를 참조한다. Subclass 생성자가 호출되고, `super()`를 통해 superclass 생성자를 호출할 때, superclass 생성자는 `new.target`값으로 subclass 생성자를 본다.
- Map class의 `set()`은 map에 새로운 entry를 추가하는 것으로 정의되어 있다. TypedMap의 `set()`메소드는 superclass의 `set()`을 override 하고 
- 생성자에서는 this는 super()전에 나올 수 없지만, 메소드 오버라이딩에서는 이러한 규칙이 있지는 않다. 오버라이딩 할 때 superclass 메소드를 호출하는것이 요구되지 않는다. 만약 `super`를 호출한다면 이 키워드는 overridding 메소드 어디에서나 호출될 수 있다.

### 9.5.3 Delegation Instead of Inheritance

- `extends` 키워드를 통해 subclass를 쉽게 만들 수 있지만, 여러 subclass들을 만들 수 있다는 것을 의미하지는 않는다. 
  다른 클래스의 행동을 공유하는 클래스를 작성하고자 할 때 subclass를 통해 그 행동을 상속할 수 있다. 하지만 그 행동을 필요한 인스턴스에게 위임하는 식으로 구현하는 것이 더 쉽고 유연하다.

- Subclass가 아닌 wrapping 이나 'composing'으로 클래스를 생성한다. 위임접근법은 composition으로 불리기도 한다.

- Javascript의 Set class와 비슷하게 작동하지만 값이 추가되었는지 아닌지가 아니라, 몇번 추가되었는지를 계산하는 Histogram 클래스를 만든다.

  Set과 비슷하게 작동하기 때문에 count 메소드만 추가한 Set의 subclass로 만들려고 할 수 있다. 하지만 count의 작동방식을 생각해보면 Map과 더 비슷하다.
  => Set의 서브 클래스가 아닌  Set-like API이면서, Map 객체로부터 위임받아 작동하는 메소드를 가진 클래스를 생성한다. 

```javascript
/**
 * A Set-like class that keeps track of how many times a value has
 * been added. Call add() and remove() like you would for a Set, and
 * call count() to find out how many times a given value has been added.
 * The default iterator yields the values that have been added at least
 * once. Use entries() if you want to iterate [value, count] pairs.
 */
class Histogram {
  // To initialize, we just create a Map object to delegate to
  constructor() { this.map = new Map(); }
  // For any given key, the count is the value in the Map, or zero
  // if the key does not appear in the Map.
  count(key) { return this.map.get(key) || 0; }
  // The Set-like method has() returns true if the count is non-zero
  has(key) { return this.count(key) > 0; }
  // The size of the histogram is just the number of entries in the Map.
  get size() { return this.map.size; }
  // To add a key, just increment its count in the Map.
  add(key) { this.map.set(key, this.count(key) + 1); }
  // Deleting a key is a little trickier because we have to delete
  // the key from the Map if the count goes back down to zero.
  delete(key) {
    let count = this.count(key);
    if (count === 1) {
      this.map.delete(key);
    } else if (count > 1) {
      this.map.set(key, count - 1);
    }
  }
  // Iterating a Histogram just returns the keys stored in it
  [Symbol.iterator]() { return this.map.keys(); }
  // These other iterator methods just delegate to the Map object
  keys() { return this.map.keys(); }
  values() { return this.map.values(); }
  entries() { return this.map.entries(); }
}
```

- Histogram 생성자는 Map 객체를 생성하고, 대부분의 메소드는 map으로부터 위임받은 한 줄짜리 메소드로 정의한다.
- 상속대신 위임을 사용했기 때문에 Histogram 객체는 Set이나 Map의 인스턴스는 아니다.

### 9.5.4 Class Hierarchies and Abstract Classes

- 데이터를 캡슐화하고 모듈화하기 위해 클래스를 사용하는 것은 좋은 테크닉이고, class 키워드를 자주 사용하게 될 것이다. 

  하지만, 상속보다 composition을 더 선호하며, extends를 거의 사용하지 않는 것을 보게 될 것이다.

- Multiple 레벨의 subclass가 적절한 환경도 있다.

- 다음 예제에서 여러개의 subclass들을 정의하고 `abstract class`를 어떻게 정의하는지 증명한다. Abstract class는 모든 subclass들이 상속하고 공유하는 partial implementation으로 정의할 수 있다. 

```javascript
/**
 * The AbstractSet class defines a single abstract method, has().
 */
class AbstractSet {
  // Throw an error here so that subclasses are forced
  // to define their own working version of this method.
  has(x) { throw new Error("Abstract method"); }
}
/**
 * NotSet is a concrete subclass of AbstractSet.
 * The members of this set are all values that are not members of some
 * other set. Because it is defined in terms of another set it is not
 * writable, and because it has infinite members, it is not enumerable.
 * All we can do with it is test for membership and convert it to a
 * string using mathematical notation.
 */
class NotSet extends AbstractSet {
  constructor(set) {
    super();
    this.set = set;
  }
  // Our implementation of the abstract method we inherited
  has(x) { return !this.set.has(x); }
  // And we also override this Object method
  toString() { return `{ x| x ∉ ${this.set.toString()} }`; }
}
/**
 * Range set is a concrete subclass of AbstractSet. Its members are
 * all values that are between the from and to bounds, inclusive.
 * Since its members can be floating point numbers, it is not
 * enumerable and does not have a meaningful size.
 */
class RangeSet extends AbstractSet {
  constructor(from, to) {
    super();
    this.from = from;
    this.to = to;
  }
  has(x) { return x >= this.from && x <= this.to; }
  toString() { return `{ x| ${this.from} ≤ x ≤ ${this.to} }`; }
}
/*
 * AbstractEnumerableSet is an abstract subclass of AbstractSet. It defines
 * an abstract getter that returns the size of the set and also defines an
 * abstract iterator. And it then implements concrete isEmpty(), toString(),
 * and equals() methods on top of those. Subclasses that implement the
 * iterator, the size getter, and the has() method get these concrete
 * methods for free.
 */
class AbstractEnumerableSet extends AbstractSet {
  get size() { throw new Error("Abstract method"); }
  [Symbol.iterator]() { throw new Error("Abstract method"); }
  isEmpty() { return this.size === 0; }
  toString() { return `{${Array.from(this).join(", ")}}`; }
  equals(set) {
    // If the other set is not also Enumerable, it isn't equal to this one
    if (!(set instanceof AbstractEnumerableSet)) return false;
    // If they don't have the same size, they're not equal
    if (this.size !== set.size) return false;
    // Loop through the elements of this set
    for (let element of this) {
      // If an element isn't in the other set, they aren't equal
      if (!set.has(element)) return false;
    }
    // The elements matched, so the sets are equal
    return true;
  }
}
/*
 * SingletonSet is a concrete subclass of AbstractEnumerableSet.
 * A singleton set is a read-only set with a single member.
 */
class SingletonSet extends AbstractEnumerableSet {
  constructor(member) {
    super();
    this.member = member;
  }
  // We implement these three methods, and inherit isEmpty, equals()
  // and toString() implementations based on these methods.
  has(x) { return x === this.member; }
  get size() { return 1; }
  *[Symbol.iterator]() { yield this.member; }
}
/*
 * AbstractWritableSet is an abstract subclass of AbstractEnumerableSet.
 * It defines the abstract methods insert() and remove() that insert and
 * remove individual elements from the set, and then implements concrete
 * add(), subtract(), and intersect() methods on top of those. Note that
 * our API diverges here from the standard JavaScript Set class.
 */
class AbstractWritableSet extends AbstractEnumerableSet {
  insert(x) { throw new Error("Abstract method"); }
  remove(x) { throw new Error("Abstract method"); }
  add(set) {
    for (let element of set) {
      this.insert(element);
    }
  }
  subtract(set) {
    for (let element of set) {
      this.remove(element);
    }
  }
  intersect(set) {
    for (let element of this) {
      if (!set.has(element)) {
        this.remove(element);
      }
    }
  }
}
/**
 * A BitSet is a concrete subclass of AbstractWritableSet with a
 * very efficient fixed-size set implementation for sets whose
 * elements are non-negative integers less than some maximum size.
 */
class BitSet extends AbstractWritableSet {
  constructor(max) {
    super();
    this.max = max; // The maximum integer we can store.
    this.n = 0; // How many integers are in the set
    this.numBytes = Math.floor(max / 8) + 1; // How many bytes we need
    this.data = new Uint8Array(this.numBytes); // The bytes
  }
  // Internal method to check if a value is a legal member of this set
  _valid(x) { return Number.isInteger(x) && x >= 0 && x <= this.max; }
  // Tests whether the specified bit of the specified byte of our
  // data array is set or not. Returns true or false.
  _has(byte, bit) { return (this.data[byte] & BitSet.bits[bit]) !== 0; }
  // Is the value x in this BitSet?
  has(x) {
    if (this._valid(x)) {
      let byte = Math.floor(x / 8);
      let bit = x % 8;
      return this._has(byte, bit);
    } else {
      return false;
    }
  }
  // Insert the value x into the BitSet
  insert(x) {
    if (this._valid(x)) { // If the value is valid
      let byte = Math.floor(x / 8); // convert to byte and bit
      let bit = x % 8;
      if (!this._has(byte, bit)) { // If that bit is not set yet
        this.data[byte] |= BitSet.bits[bit]; // then set it
        this.n++; // and increment set size
      }
    } else {
      throw new TypeError("Invalid set element: " + x);
    }
  }
  remove(x) {
    if (this._valid(x)) { // If the value is valid
      let byte = Math.floor(x / 8); // compute the byte and bit
      let bit = x % 8;
      if (this._has(byte, bit)) { // If that bit is already set
        this.data[byte] &= BitSet.masks[bit]; // then unset it
        this.n--; // and decrement size
      }
    } else {
      throw new TypeError("Invalid set element: " + x);
    }
  }
  // A getter to return the size of the set
  get size() { return this.n; }
  // Iterate the set by just checking each bit in turn.
  // (We could be a lot more clever and optimize this substantially)
  *[Symbol.iterator]() {
    for (let i = 0; i <= this.max; i++) {
      if (this.has(i)) {
        yield i;
      }
    }
  }
}
// Some pre-computed values used by the has(), insert() and remove() methods
BitSet.bits = new Uint8Array([1, 2, 4, 8, 16, 32, 64, 128]);
BitSet.masks = new Uint8Array([~1, ~2, ~4, ~8, ~16, ~32, ~64, ~128]);
```

