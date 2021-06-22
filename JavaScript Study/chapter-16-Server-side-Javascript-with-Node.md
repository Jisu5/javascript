## 16.3 Buffers
* Node에서 파일이나 네트워크에서 데이터를 읽을때, 자주 사용하는 데이터타입은 `Buffer` 클래스입니다.
* `Buffer`는 많은 부분 스트링과 비슷합니다. 스트링이 character들의 연속이라면, 버퍼는 byte들의 연속입니다.

#### Buffers vs Uint8Array
* 과거 코어 자바스크립트에는 `unsigned bytes`의 배열을 나타내는 `Uint8Array`가 없었는데, Node에서 이러한 필요성을 충족하기 위해 `Buffer`클래스를 정의했습니다. `Uint8Array`가 자바스크립트에서 정의되기 시작하면서 No


de에서 `Buffer`클래스는 Uint8Array의 서브클래스가 되었습니다.

#### Buffer 부호화 / 복호화
* Buffer와 Uint8Arary 다른점은 Buffer는 자바스크립트 스트링과 내부에서 작동하기 위해 디자인되었다는 점입니다.
    * 버퍼안에 있는 바이트들은 문자 스트링으로 초기화될 수 있거나 변환될 수 있습니다. 
    * string과 Character encoding이 주어졌을 때, bytes의 sequence로 암호화 할 수있습니다.
    * bytes와 Character encoding이 주어졌을 때, charcter의 sequence로 복호화 할 수있습니다.
    * 결론적으로, Node의 `Buffer`클래스는 부호화와 복호화를 할 수 있는 매서드를 가지고 사용할 수있습니다.
* encoding의 종류는 다음과 같습니다.
    * "utf8"
    * "utf16le"
    * "latin1"
    * "ascii"
    * "hex"
    * "base64"
* string에서 부호화 / 보호화하는 예시 코드는 다음과 같습니다.
    ```
        let b = Buffer.from([0x41, 0x42, 0x43]);    // <Buffer 41 42 43>
        b.toString()                                // "ABC"; default "utf8"
        b.toString("hex")                           // "414243"

        let computer = Buffer.from("IBM3111", "ascii");     //Conver string to Buffer
        for (let i = 0; i < computer.length; i++) {         //Use Buffer as byte array
            computer[i]--;                                  //Buffers are mutable
        }                 
        computer.toString("ascii")                          // => "HAL2000"
        computer.subarray(0,3).map(x=>x+1).toString()       // => "IBM"

        // Create new "empty" buffers with Buffer.alloc()
        let zeros = Buffer.alloc(1024);                     // 1024 zeros
        let ones = Buffer.alloc(128, 1);                    // 128 ones
        let dead = Buffer.alloc(1024, "DEADBEEF", "hex")    // Repeating pattern of bytes

        // Buffers have methods for reading and writing multi-byte values
        // from and to a buffer at any specified offset.
        dead.readUInt32BE(0)                                // => 0xDEADBEEF
        dead.readUInt32BE(1)                                // => 0xEADBEEFE
        dead.readBigUInt64BE(6)                             // => 0xBEEFDEADBEEFDEADn
        dead.readUInt32LE(1020)                                // => 0xEFBEADDE

    ```

## 16.4 Events and EventEmitter
* Node의 API는 기본적으로 비동기입니다. 많은 API들 중에서 비동기 API는 요청 작업이 완료되었을 때 호출되는 2개의 error-first callbacks 인자를 취합니다.
* 하지만, 몇몇 API는 이벤트 기반으로 작동합니다. 이는 API가 함수보다 오브젝트를 두고 디자인이 되었을 때나, 콜백함수가 여러번 호출되어야 하는 경우나, 다양한 타입의 콜백함수를 필요로 할 때 입니다.
* net.Server클래스를 생각했을 때, 이 타입의 오브젝트는 클라이언트로부터 연결을 받을때 사용하는 서버 소켓입니다. 
    * 소켓은 연결에 대해 listening하기 시작할 때 "listening" 이벤트를 Emit합니다.
    * 소켓은 클라이언트가 연결을 시도할때마다 "connection" 이벤트를 Emit합니다.
    * 소켓은 연결이 끊어져서 더이상 listening 하지 않을 때 "close" 이벤트를 Emit합니다.
* Node에서 event를 emit하는 오브젝트는 `EventEmitter`의 인스턴스이거나 서브클래스입니다.
    ```
    const EventEmitter = require("events");     // Module name does not match class name
    const net = require("net");

    let server = new net.Server();              // create a Server object
    server instanceof EventEmitter;             // => true : Servers are EventEmitters
    ```
* EventEmitter의 주요 feature는 이벤트 핸들러 함수를 `on()` method와 함께 등록할 수 있다는 점입니다.
* EventEmitter는 다양한 타입의 이벤트를 emit할 수 있고, 이벤트 타입은 이름으로 구별할 수 있습니다.
* EventEmitter가 `on()` method를 사용해서 이벤트 핸들러 함수를 등록할 때, event type의 이름과 이벤트 타입이 발생했을 때 호출할 함수를 인자로 사용합니다.

```
const net = require("net");
let server = new net.Server();              // Create a Server object
server.on("connection", socket => {         // Listen for "connection" events
    // Server "connection" events are passed a socket object
    // for the client that just connected. Here we send some data
    // to the client and disconnect
    socket.end("Hellor World", "utf-8")
})
```
* 그 외에 이벤트 핸들러를 등록,삭제하는 매서드들
    * `addListener()` : 이벤트 리스너를 등록하기 위해서 좀 더 명확한 이름의 매서드
    * `off()`, `removeListener()` : 등록된 이벤트 리스너를 지울때 사용하는 매서드
    * `once()` : 이벤트 리스너가 작동한 후 자동적으로 삭제할 수 있도록 하는 매서드

* EventEmitter 오브젝트의 특정 이벤트 타입이 발생했을 때, 현재 등록된 이벤트 타입의 모든 핸들러 함수를 호출합니다.
* 이벤트 핸들러들은 처음 등록된 핸들러가 가장 먼저 호출되고, 이 순서대로 끝까지 호출되게 됩니다. 
* 이벤트 핸들러 함수가 하나 이상 존재한다면 한개의 스레드에서 순서대로 호출되게 됩니다.
    * Node에서 paralleism은 존재하지 않습니다.
    * 이벤트 핸들러 함수는 비동기가 아니라, 동기적으로 호출됩니다. 
        * emit() 매서드는 호출될 이벤트 핸들러들을 대기시켜 놓지 않습니다.
        * emit() 매서드는 등록된 모든 핸들러 순서대로 하나씩 호출하고 마지막 이벤트 핸들러가 return할때까지 return하지 않습니다.
            * 이 뜻은 Node의 API가 event를 emit할 때 이 API가 이벤트 핸들러를 blocking합니다.
            * 만약 fs.readFileSync()와 같은 blocking function을 호출하는 이벤트 핸들러를 쓴다면, 동기적인 파일을 전부 읽을때까지 이벤트 핸들링은 일어나지 않습니다.

* 빠른 응답이 필요한 프로그램을 코드할 때, 이벤트 핸들링 함수가 blocking되면 안됩니다.
    * 이벤트가 발생했을 때, 많은 연산을 필요로 한다면 이러한 연산을 비동기적으로 스케줄링 할 수 있는 핸들러(`setTimeout()`)를 사용하는 것이 좋습니다.
#### emit()매서드
* EventEmitter 클래스는 등록된 이벤트 핸들러 함수를 호출하는 `emit()` 매서드를 정의합니다. 
* `emit()` 매서드는 반드시 event type의 이름을 첫 인자로 해서 호출되어야 합니다.
* `emit()` 매서드의 그 외 인자들은 등록된 이벤트 핸들러 함수의 인자가 됩니다.
* 핸들러 함수는 EventEmitter 오브젝트에 정의된 `this` value와 호출될 수도 있습니다. 
#### emit(), exception
* 이벤트 핸들러 함수에 의해서 리턴된 값은 ignored됩니다.
* 하지만 이벤트 핸들러함수가 `exception`을 `throw`하면 `emit()` 호출이 throw한 함수를 제외시키고 등록된 다른 이벤트 함수의 실행을 막습니다.

#### 콜백기반, 이벤트기반 API와 에러 핸들링
* 콜백기반의 API는 error-first 콜백을 사용합니다. 
    * 이때, 에러가 발생하면 콜백함수의 첫 인자를 확인하는 것이 중요합니다.
    * 이벤트기반의 API는 "error" event를 확인해야 합니다.

* 이벤트기반의 API는 네트워킹이나 I/O Streaming 형태에서 많이 사용되는데, 이때 이벤트 기반의 API는 예측할 수 없는 비동기적 에러에 취약합니다. 그리고 대부분의 EventEmitter는 에러가 발생할 때 emit하는 "error"이벤트를 정의합니다.
    * 이벤트 기반의 API를 사용할때, "error" 이벤트를 정의하는 습관을 들여야합니다. 
    * "Error" 이벤트는 EventEmitter에서 특별하게 취급받는데, 만약 `emit()`매서드가 "error" 이벤트를 emit하기 위해 호출되고, 이벤트타입에 핸들러가 정의되어 있지 않다면 exception이 throw됩니다. 이것은 비동기적으로 발생하기 때문에, `catch`블락에서 예외를 핸들링할 수 없습니다. 그래서 이러한 에러는 프로그램을 종료시키게 만듭니다.
