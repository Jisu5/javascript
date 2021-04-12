## 11.4 Dates and Times
* **Date class**는 날짜와 시간을 다루는 Javascript의 API입니다.
* 인자없이 Date 생성자를 사용해서 object를 만든다면 현재의 날짜와 시간을 반환합니다.
    ```
    let now = new Date()   // The current time
    ```
* 하나의 number 인자를 넣는다면 `Date()` 생성자는 1970년부터 시작하여 millisecond로 나타냅니다.
    ```
    let epoch = new Date(1) // 1970-01-01T00:00:00.001Z
    ```
* 두 개 이상의 정수 인자를 넣는다면, local time으로 년도, 달, 일, 시, 분, 초, millisecond를 뜻합니다.
    ```
    let century = new Date(2100,   // Year 2100
                            0,     // Janauary
                            1,     // 1st
                            2,     // 02:03:04.005, local time
                            3,
                            4,
                            5) 
    ```
    * 달은 0부터 시작합니다. 그래서 1월달은 0으로 표기합니다.
    * 일은 1부터 시작합니다.
    * 시간부분을 제외하고 넣는다면 모두 default로 0으로 되며, 00:00:00.000 정각이 됩니다.
* time zone은 local computer에 설정된 시간으로 정해지며, UTC로 사용하고 싶다면. `Date.UTC()`를 사용할 수 있습니다. 이 Static Method는 Date()생성자와 같은 인자를 받으며, millisecond를 반환합니다.
    ```
    let century = new Date(Date.UTC(2100, 0, 1))  // 2100-01-01T00:00:00.000Z
    ```
* 만약 `console.log(century)`를 사용해서 날짜를 print를 하려한다면 default로 local timze이 print됩니다. 그래서 UTC로 print하고 싶다면 `toUTCString()` 또는 `toISOString()`으로 변환을 해야합니다.

* 정수 대신에 String을 `Date()`생성자에 넣으려한다면, 생성자는 시간과 날짜로 스트링을 Parsing합니다.
    ```
    let century = new Date("2100-01-01T00:00:00Z")  // An ISO format date
    ```

* Date object를 사용한다면, `get`과 `set`을 사용해서 년,월,일,시,분,초를 조작하기가 쉬워집니다. `getFullYear()`, `getUTCFullYear()`, `setFullYear(), setUTCFullYear()`를 사용해서 년도를 가져오고 원하는 만큼의 해를 더하고 빼고 저장할 수 있습니다.
    ```
    let d = new Date();
    d.setFullYear(d.getFullYear() + 1);
    ```
    * 년도말고, 월,일,시간을 다루고 싶다면 얼마든지 FullYear를 'Month', 'Date', 'Hours', 'Minutes', 'Seconds', 'Milliseconds'로 사용할 수 있습니다.

* 참고할점은, 요일은 (0:일요일,  6:토요일)은 read-only이기 때문에, setDay() method가 존재하지 않습니다.


## 11.4.1 Timestamps
* 자바스크립트는 날짜(date)를 나타낼 때, 내부적으로는 1970-01-01 00:00:00를 0으로 시작하여 millisecond를 나타내는 정수로 돌아가게 됩니다.
* 정수는 8,640,000,000,000,000까지 지원되서, milliseconds로 270,000년 까지 나타낼 수 있습니다.
* `getTime()과` `setTime()` 통해서 시간을 다룰 수 있습니다. 예를 들어서 30초를 더한다면 아래와 같이 할 수 있습니다.
    ```
    d.setTime(d.getTime() + 30000); //1618034153727
    ```
* 이러한 milliseconds들을 종종 timestamps라고 불리는데, timestamp로 작업하는 것이 Data object를 다루는 것보다 유용한 경우가 많습니다. `Date.now()`는 현재시각을 timestamp로 반환하는데 이것을 이용해서 코드의 실행시간이 얼마나 걸렸는지 재볼 수 있습니다.
    ```
    let startTime = Date.now();
    reticulateSplines();
    let endTime = Date.now();
    console.log('Spline reticulation took ${endTime-startTime}ms.)
    ```

## 11.4.2 Date Arithmetic
* Date object는 `<`, `>`, `<=`, `>=`로 비교를 할 수 있습니다.
* Date object는 다른 Data object에서 뺄셈 연산을 하여 milliseconds를 반환할 수 있습니다.
* Data class의 `valueOf()` method가 timestamp를 반환하기 때문에 연산이 가능할 수 있습니다.
* Date object에서 시간을 더하거나 뺄 때 milliseconds를 변환 후 연산 작업을 할 수도 있지만, 달이나 해를 더할때는 계산이 어려워 집니다.
* 일, 월, 년을 연산할때는 `setDate(), setMonth(), setYear()`을 사용할 수 있습니다. 
    ```
    // 3개월과 2주를 더합니다.
    let d = new Date()
    d.setMonth(d.getMonth() + 3, d.getDate() + 14);
    ```
* 만약에 달을 더했을 때 11(12월)을 넘어가더라도 Data class는 정확히 다음해로 넘어가도록 구현되어있습니다.

## 11.4.3 Formatting and Parsing Date Strings.
* Date class는 Date object를 string으로 변환하는 다양한 method를 가지고 있습니다.
    ```
    let d = new Date(2020, 0, 1, 17, 10, 30);
    d.toString()   // => "Wed Jan 01 2020 17:10:30 GMT-0808"
    d.toUTCString()   // => "Thu, 02 Jan 2020 01:10:30 GMT"
    d.toLocaleDateString()   // => "1/1/2020" : 'en-US' locale
    d.toLocaleTimeString()   // => "5:10:30 PM" : 'en-US' locale
    d.toISOString()   // => "2020-01-02T01:10:30.000Z"
    ```
* String을 date와 time으로 변환하는 method로는 Date.parse()가 있습니다.

## 11.5 Error Classes
* `throw` statement로 유저가 정의한 예외를 throw할 수 있습니다. 현재 실행중인 함수는 멈추게 되어 `throw` 다음 statement는 실행되지 않습니다. 그리고 call stack 안에 있는 `catch` block으로 전달되게 됩니다. (만약에 catch block이 존재하지 않는다면 프로그램은 종료됩니다.)
    ```
    function getRectArea(width, height) {
    if (isNaN(width) || isNaN(height)) {
        throw 'Parameter is not a number!';
    }
    }

    try {
    getRectArea(3, 'A');
    } catch (e) {
    console.error(e);
    // expected output: "Parameter is not a number!"
    }
    ```
* Javascript는 error 신호를 보내기 위한 exception type을 갖고 있지 않습니다.
* 대신, `Error` class를 정의하고 있어서 Error의 인스턴스나 서브클래스와 `throw`문을 함께 사용하여 에러 신호를 보낼 수 있습니다.
* `Error` Class를 사용하면 좋은 이유는 Error가 만들어질 때, Javascript stack의 상태를 포착해서 stack trace가 에러 메시지와 함께 보여지게 됩니다. 이는 문제가 생겼을 때 디버그를 하기 쉽게 해줍니다.
(Stack trace는 `throw statement`가 throw한 곳을 보여주는것이 아니라, Error object가 생성되어 진 곳을 보여줍니다. )
* Error object는 2개의 프로퍼티 `message`와 `name`과 1개의 `toString()` method를 가지고 있습니다. 
    * `message` property는 `Error()` 생성자의 인자로 전달되어집니다. 
    * `name` property는 언제나 "Error"입니다.
    * `toString()` method는 단순히 `name, message` 를 반환합니다.
* ECMAScript표준은 아니지만, Node와 현재 Modern browser들은 Error 오브젝트에 `stack`이라는 프로퍼티를 가지고 있습니다.
    * 이 프로퍼티의 값은 에러 오브젝트가 생성될 때 자바스크립트의 콜 스택 기록을 포함하고 있는 여러줄의 스트링으로 되어 있습니다.
    * 예측하기 어려운 에러를 잡아낼 때 이러한 로그 정보들은 매우 유용합니다.
* 자바스크립트는 Error class뿐만 아니라 Error에 관한 다양한 subclass를 가지고 있습니다.
    * `EvalError, RangeError, ReferenceError, SyntaxError, TypeError, and URIError`.
    * 상황에 맞게 이 에러 서브클래스를 사용할 수 있으며, 기본 `Error` Class처럼 생성자의 인자로 message를 전달할여 인스턴스를 생성할 수 있습니다. 
* subclass를 직접 정의할 수 있습니다.
    ```
    class HTTPError extends Error {
        constructor(status, statusText, url) {
            super(`${status} ${statusText} : ${url};)
            this.status = status;
            this.statusText = statusText;
            this.url = url
        }

        get name() {
            return "HTTPERrror";
        }

        let error = new HTTPError(404, "Not Found", "http://javascript.com")
        error.status
        error.message
        error.name
    }
    ```


## 11.6 JSON Serialization and Parsing.
* 프로그램이 데이터를 저장하거나 네트워크를 사용해서 데이터를 전송할 때 반드시 in-memory 데이터 구조를 bytes 스트링으로 변환해야 합니다. 이러한 변환 과정을 serialization, marsahling,pickling이라고 표현합니다.
* Javascript에서 가장 쉬운 방법은 JSON형태를 사용하는 것입니다.
    * JSON은 "Javascript Object Notation"이며, 자바스크립트의 오브젝트와 어레이를 사용하여 데이터의 변환을 돕습니다.
* JSON은 `numbers, strings, true, false, null` 뿐만 아니라 `array, object`를 표현할 수 있습니다.
* JSON은 `Map, Set, RegExp, Date, typed arrays`를 지원하지 않습니다.
* Javascript는 2개의 함수를 이용하여, JSON의 serialization과 deserialization을 돕습니다.
* `JSON.stringify()`는 이름에서 알수도 있듯이 스트링을 반환합니다. 그리고 주어진 string을 다시 워본 데이터 구조로 파싱하려면 `JSON.parse()`를 사용할 수 있습니다.
    ```
    let o = {s: "", n: 0, a: [true, false, null]};
    let s = JSON.stringify(o); // s == `{"s":"", "n":0, "a":[true,false,null]}`
    let copy = JSON.parse(s);  // copy == `{s: "", n: 0, a: [true, false, null]}`
    ```
* 일반적으로 `JSON.stringify()`와 `JSON.parse()`에 한 개의 인자를 전달합니다. 하지만 2개의 함수 모두 JSON format을 확장할 수 있는 옵션 인자를 가지고 있습니다.
    * JSON-formatted string을 'human-readable'(설정 파일로 사용할 때)형태로 원할 때, 2번째 인자로 `null`, 3번째 인자는 indendation과 관련되어 있어서 `number`나 , `string`을 전달할 수 있습니다.
    ```
    let o = {s: "test", n: 0};
    JSON.stringify(o, null, 2);  // => `{\n  "s":  "test", \n  "n":  0\n}`
    ```
    * 만약 3번째 인자가 `\t`와 같은 `string`이라면 tab과 같은 효과를 볼 수 있습니다.

## 11.6.1. JSON Customizations
* JSON format을 지원하지 않는 값들을 serialize할 때 우리는 그 값이 `toJSON()` method를 갖고 있는지 확인해야 합니다. 만약 `toJSON()` method가 정의되어 있다면, method의 리턴 값이 원본 값을 대체하여 stringify하게 됩니다.
    ```
    const event = new Date('August 19, 1975 23:15:30 UTC');

    const jsonDate = event.toJSON();

    console.log(jsonDate);
    // expected output: 1975-08-19T23:15:30.000Z
    ```
    * Date object는 toJSON() method를 구현해놔서 toISOString() method와 똑같은 값을 반환합니다.
    * 그래서 Date 인스턴스가 포함된 object를 serialize할때 Date 인스턴스는 자동으로 toJSON() method를 사용하여 string으로 변환됩니다.
    * 또, 이 serialized string을 다시 parse한다면 맨 처음 오브젝트의 상태와는 다른 값을 가지게 됩니다. 왜냐하면 Date 인스턴스는 toJSON() method로 반환된 다른 스트링 형태를 보여주기 때문입니다.
* 그래서 parsed된 오브젝트를 다른 형태로 고치고 싶다면 `JSON.parse()`의 2번째 인자로 `reviver`함수를 전달할 수 있습니다.
* `reviver`함수가 전달되면, 이 `reviver`함수가 각각의 원시 value들을 인자로 사용하여 호출되게 됩니다.
* `reviver`함수의 첫 번째 인자는 오브젝트의 프로퍼티 네임, 어레이의 인덱스가 될 수 있습니다.
* `reviver`함수의 두 번째 인자는 오브젝트의 값이나 어레이의 원소입니다.
```
const myObj = new Object();
myObj.firstname = "mike";
myObj.lastname = "smith";

const jsonString = JSON.stringify(myObj);
const jsonObj = JSON.parse(jsonString, dataReviver);

function dataReviver(key, value)
{ 
    if(key == 'lastname')
    {
        let newLastname = "test";
        return newLastname;
    }

  return value;  // < here is where un-modified key/value pass though

}

JSON.stringify(jsonObj )// "{"firstname":"mike","lastname":"test"}" 

```
* `reviver` 함수의 반환되는 값은 프로퍼티의 새로운 값이 됩니다.
* `reviver` 함수가 두 번째 인자를 그대로 반환한다면 프로퍼티는 변하지 않습니다. 
* `reviver` 함수가 `undefined`를 반환한다면 그 프로퍼티는 오브젝트나 어레이에서 삭제됩니다.

```
let data = JSON.parse(text, function(key, value) {
    if (key[0] === "_") return undefined;

    if (typeof value === "string" &&
    /^\d\d\d\d-\d\d-\d\dT\d\d:\d\d:\d\d.\d\d\dZ$/.test((value)) {
        return new Date(value;)
    }

    return value
})
```



* `JSON.stringify()`의 두번째 인자로 어레이나 함수를 전달하여 사용자 정의된 JSON format을 만들 수 있습니다.
* 만약에 string으로 구성된 어레이가 두 번째 인자로 전달된다면, 이 어레이의 원소가 오브젝트의 프로퍼티가 되고, 어레이의 원소에 포함되지 않은 프로퍼티는 삭제됩니다.
* 그리고 어레이의 순서대로 오브젝트는 반환되게 됩니다.
```
let text = JSON.stringify(address, ["city", "state", "country"])
```
* 만약에 두번째 인자로 함수를 전달한다면 이 함수는 `replacer`함수로 사용되어집니다. 
* `replacer`함수는 `reviver`함수의 반대적인 효과를 반환합니다.
* `replacer`함수가 인자로 들어가게 되면, `reviver`함수 처럼 프로퍼티나 어레이의 값들을 각각 인자로 호출하여 stringify하게 만듭니다. 
* `replacer`함수도 `reviver`함수와 마찬가지로 첫번째, 두번째 인자가 각각 key와 value를 뜻합니다.
```
let json = JSON.stringify(o, (k,v) => v instanceof RegExp ? undefined : v);
```
* `replacer`함수로 toJSON() method가 했던 일을 대신할 수 있습니다. 즉 nonserializable한 type을 serializable한 type으로 변환하는 역할을 할 수 있습니다.
* 만약 `replacer`함수를 사용해서 nonserializable type을 serialize로 변환하였다면, 다시 이 값들을 parse해야 한다면 `reviver`함수를 정의해야 합니다. 

## 11.7 The Internationalization API
* 자바스크립트에는 `Intl.NumberFormat, Intl.DateTimeFormat, Intl.Collator`의 클래스로 구성된 Internationalization API를 가지고 있습니다.
* 이 클래스들은 날짜, 시간, 화폐, 퍼센티지 등등을 포맷팅 할 수 있게 해줍니다.
* internationalization의 가장 중요한 부분은 사용자의 언어로 text를 번역할 수 있게 해주는 것입니다.


## 11.7.1 Formatting Numbers
* `Intl.NumberFormat` 클래스는 `format()` method를 정의하여 포맷팅을 할 수 있도록 돕습니다.
* `NumberFormat()`의 생성자로 첫번째 인자는 숫자가 포맷되어야 하는 locale 이름을 전달 받습니다.
    * `en-us`, `fr`, `zh-Hans-CN`등이 전달될 수 있습니다.
* `NumberFormat()`의 생성자로 두번째 인자는 숫자가 어떤 방식으로 포맷되어야 하는가에 대한 디테일한 오브젝트가 전달됩니다.
    * `styles` : 숫자가 나타내져야 하는 종류에 대한 포맷팅 (`decimail` (default), `percent`, `currency`)
    * `currrency` : styles가 `currency`라면 이 프로퍼티는 ISO currency code (USD, KRW)형태로 나타내져야 합니다.
    * `currencyDisplay` : styles가 `currency`라면 이 프로퍼티는 currency가 어떻게 보여지는지 정합니다. 

    ```
    let euros = Intl.NumberFormat("es", {style : "currency", currency: "EUR"})
    euros.format(10)    // => "10,00 €"

    let pounds = Intl.NumberFormat("en", {style: "currencty", currency: "GBP"})
    pounds.format(1000)  // => "£1,000.00"
    ```

* `Intl.NumberFormat`의 `format()` method는 `NumberFormat` 오브젝트에 bind되어있습니다. 그래서 formatting 오브젝트를 가져오는 변수를 정의하기 보다, format() method를 그 위에 호출할 수 있습니다.
    ```
    let data = [0.05, .85, 1];
    let formatData = Intl.NumberFormat(undefined, {
        style: "percent",
        minimumFractionDigits: 1,
        maximumFractionDigits: 1
    }).format;

    data.map(formatData)     // => ["5.0%", "75.0%", "100.0%"]
    ```
## 11.7.2 Formatting Dates and Times
* `Intl.DateTimeFormat()` 클래스는 `Intl.NumberFormat()`클래스와 동일한 방식으로 인자를 받고, `format()` method를 활용합니다.
* `Date` 클래스는 `toLocaleDateString()` 과 `toLocaleTimeString()` method로 유저의 locale을 표현할 수 있도록 했지만, 이 매서드들은 시간과 날짜의 어느 필드를 집어서 보일지 안보일지 컨트롤 할 수는 없습니다.  `Intl.DateTimeFormat()` 클래스는 이러한 경우에도 유용하게 사용되어질 수 있습니다.
    ```
    let d = new Date("2020-01-02T13:14:15Z");

    Intl.DateTimeFormat("en-us").format(d)   // => "1/2/2020"
    Intl.DateTimeFormat("fr-FR").format(d)   // => "02/01/2020"

    let opts = { weekday: "long", month: "long", year: "numeric", day: "numeric" };
    Intl.DateTimeFormat("en-US", opts).format(d)  // => "Thursday, January 2, 2020"
    Intl.DateTimeFormat("es-ES", opts).format(d)  // => "jueves, 2 de enero de 2020"

    opts = { hour: "numeric", minute: "2-digit", timeZone: "America/New_York" }
    Intl.DateTimeFormat("fr-CA", opts).format(d)  // => "8 h 14"
    ```

## 11.8 The Console API
* Console API는 `console.log()` 말고도 유용한 함수들을 정의하고 있습니다.
* `console.debug()`, `console.info()`, `console.warn(), console.error()` :  이 함수들은 `console.log()`와  거의 비슷합니다. Node에서 console.error()는 output을 stderr stream으로 보냅니다. 다른 함수들은 console.log()의 alias와 같은 함수여서 stdout stream으로 output을 보냅니다.

* `console.assert()` : 첫 번째 인자가 truthy하다면, 이 함수는 아무것도 하지 않습니다. 하지만 첫번째 인자가 `false`이거나 다른 falsy한 값이라면 남은 인자들이 `console.error()`에 전달된 것처럼 출력됩니다.


* `console.clear()` : 이 함수는 console 내용을 깨끗하게 만들어줍니다. browser와 Node에서 작동합니다.

* `console.table()` : 이 함수는 인자를 tabular form 형태로 출력합니다. (만약 tabular form으로 출력할 수 없다면 console.log()의 형식으로 출력합니다.) 이 함수는 오브젝트 안의 어레이나 어레이 안의 오브젝트 같은 형태에서 유용하게 사용되어질 수 있습니다.

* `console.count()` : 이 함수는 string을 인자로 받아 이 스트링을 기록하여 이 스트링이 몇 번 호출되었는지 카운트 합니다. Event Handler를 디버깅할 때 유용하게 사용되어질 수 있습니다. Event Handler가 몇 번 작동했는지 기록할 수 있습니다. (console.countReset()으로 카운트를 리셋할 수 있습니다.)

* `console.time()` : 이 함수는 스트링을 인자로 받아서, console.timeLog()와 함꼐 사용되어 이 스트링이 호출되는데 얼마나 걸렸는지 시간을 잴 수 있습니다.

* `console.timeLog()` : 이 함수는 스트링을 인자로 받아서, console.time()이 호출되고 나서 얼마나 시간이 걸렸는지 잴 수 있습니다.

* `console.timeEnd()` : 이 함수는 스트링을 받아, console.time()이 다시 처음으로 호출되지 않고서는 console.timeLog()가 유효하지 않도록 합니다.

## 11.8.1 Formatted Output with Console

* console function은 `%s, %i, %d, %f, %o %0, %c`와 같은 format string을 사용할 수 있습니다. 
* `%s` : 인자는 스트링으로 변환됩니다.
* `%i`, `%d` : 인자는 `number`로 변환되고 정수가 됩니다.
* `%f` : 인자는 `number`로 변환됩니다.
* `%o`, `%0` : 인자는 오브젝트처럼 변환되고, 프로퍼티 이름과 밸류가 보여집니다. 
* `%c` : Web browser에서 인자는 CSS style의 string으로 변환됩니다.

## 11.9 URL APIs
* `URL` class는 ECMAScript 표준은 아니지만, Node와 internet browser에서 작동합니다.
* `URL()` 생성자로 URL string을 인자로 보낼 수 있습니다. URL object가 생성되면, URL object의 프로퍼티티로 여러 조작이 가능합니다.
    ```
    let url = new URL("https://example.com:8000/path/name?q=term#fragment");
    url.href     // => "https://example.com:8000/path/name?q=term#fragment"
    url.origin   // => "https://example.com:8000/"
    url.protocol // => "https:"
    url.host     // => "example.com:8000"
    url.hostname // => "example.com"
    url.port     // => "8000"
    url.pathname // => "/path/name"
    url.search   // => "?q\term"
    url.hash     // => "#fragment"
    ```
* URL class가 유용하게 쓰이는 이유 중 하나는 url path와 query parameter를 더할 수 있다는 점입니다.
    ```
    let url = new URL("https://example.com")
    url.pathname = "api/search";
    url.search = "q=test";
    url.toString()    //  => "https://example.com/api/search?q=test"
    ```
* URL.searchParams 프로퍼티와 search 프로퍼티를 통해 query parameter를 key/value pair처럼 다룰 수 있습니다.
    ```
    let url = new URL("https://example.com/search");
    url.search                            // => "" : no query yet
    url.searchParams.append("q", "term")  // Add a search parameter
    url.search                            // => "?q=term"
    url.searchParams.set("q", "x")        // Change the value of this parameter
    url.search                            // "?q=x"
    url.searchParams.get("q")             // => "x" : query the parameter value
    url.searchParams.has("q")             // => true : there is a q parameter
    url.searchParams.has("p")             // => false : there is no p parameter
    url.searchParams.append("opts", 1)    // Add another search parameter
    url.search                            // => "?q=x&opts=1"
    url.searchParams.append("opts", "&")  // Add another vale for same name
    url.search                            // => "?q=x&opts=1&opts=%26
    url.searchParams.sort();              // => Put params in alphabetical order
    [...url.searchParams]                 // => [["opts", "y"], ["q", "x"]]
    ```

## 11.10 Timers
* 일정 시간이 지난 후에 원하는 함수를 예약 실행(호출)할 수 있게 하는 것을 '호출 스케줄링(scheduling a call)'이라고 합니다.

* 호출 스케줄링을 구현하는 방법은 두 가지가 있습니다.

    * setTimeout을 이용해 일정 시간이 지난 후에 함수를 실행하는 방법
    * setInterval을 이용해 일정 시간 간격을 두고 함수를 실행하는 방법

* 자바스크립트 명세서엔 setTimeout과 setInterval가 명시되어있지 않습니다. 하지만 시중에 나와 있는 모든 브라우저, Node.js를 포함한 자바스크립트 호스트 환경 대부분이 이와 유사한 메서드와 내부 스케줄러를 지원합니다.
    ```
    function sayHi() {
        alert('안녕하세요.');
    }

    setTimeout(sayHi, 1000);
    ```

    ```
    function sayHi(who, phrase) {
        alert( who + ' 님, ' + phrase );
    }

    setTimeout(sayHi, 1000, "홍길동", "안녕하세요."); // 홍길동 님, 안녕하세요.
    ```
* setTimeout을 호출하면 '타이머 식별자(timer identifier)'가 반환됩니다. 스케줄링을 취소하고 싶을 땐 이 식별자(아래 예시에서 timerId)를 사용하면 됩니다.

* 스케줄링 취소하기:
    ```
    let timerId = setTimeout(...);
    clearTimeout(timerId);
    ```
* 아래 예시는 함수 실행을 계획해 놓았다가 중간에 마음이 바뀌어 계획해 놓았던 것을 취소한 상황을 코드로 표현하고 있습니다. 예시를 실행해도 스케줄링이 취소되었기 때문에 아무런 변화가 없는 것을 확인할 수 있습니다.

    ```
    let timerId = setTimeout(() => alert("아무런 일도 일어나지 않습니다."), 1000);
    alert(timerId); // 타이머 식별자

    clearTimeout(timerId);
    alert(timerId); // 위 타이머 식별자와 동일함 (취소 후에도 식별자의 값은 null이 되지 않습니다.)
    ```
* 예시를 실행하면 alert 창이 2개가 뜨는데, 이 얼럿 창을 통해 브라우저 환경에선 타이머 식별자가 숫자라는 걸 알 수 있습니다. 다른 호스트 환경에선 타이머 식별자가 숫자형 이외의 자료형일 수 있습니다. 참고로 Node.js에서 setTimeout을 실행하면 타이머 객체가 반환합니다.


* setInterval 메서드는 setTimeout과 동일한 문법을 사용합니다.
* 인수 역시 동일합니다. 다만, setTimeout이 함수를 단 한 번만 실행하는 것과 달리 setInterval은 함수를 주기적으로 실행하게 만듭니다.

* 함수 호출을 중단하려면 clearInterval(timerId)을 사용하면 됩니다.

* 다음 예시를 실행하면 메시지가 2초 간격으로 보이다가 5초 이후에는 더 이상 메시지가 보이지 않습니다.
    ```
    // 2초 간격으로 메시지를 보여줌
    let timerId = setInterval(() => alert('째깍'), 2000);

    // 5초 후에 정지
    setTimeout(() => { clearInterval(timerId); alert('정지'); }, 5000);
    ```