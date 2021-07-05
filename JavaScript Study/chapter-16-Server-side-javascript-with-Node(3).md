# Chapter 16. Server-Side JavaScript with Node

## 16.5 Streams

-  데이터를 처리하는 알고리즘을 구현할 때, 메모리에서 데이터를 읽어 처리하고, 데이터를 쓰는 것이 가장 쉬운 방법이다.
- 아래는 파일을 복사하는 노드 함수이다.

```javascript
const fs = require("fs");
// 비동기 & 비스트림(즉, 비효율적) 함수
function copyFile(sourceFilename, destinationFilename, callback) {
  fs.readFile(sourceFilename, (err, buffer) => {
    if (err) {
      callback(err);
    } else {
      fs.writeFile(destinationFilename, buffer, callback);
    }
  });
}
```

- 이 `copyFile()` 함수는 비동기 함수와 콜백을 사용하기 때문에 block 하지 않으며, 서버같은 동시성 프로그램에서 사용하기에 적절하다.
- 그러나 파일의 전체 내용을 한 번에 메모리에 저장할 수 있는 충분한 메모리를 할당해야 한다.
  - 일부 사용 사례에서는 문제가 없을 수 있지만 복사할 파일이 매우 크거나 프로그램의 동시성이 높고 동시에 복사되는 파일이 많을 경우 오류가 발생하기 시작한다.
  - 이 `copyFile()` 구현의 또 다른 단점은 이전 파일의 읽기를 마칠 때까지 새 파일의 쓰기를 시작할 수 없다는 것이다.
- 이 문제점의 해결법은 데이터가 프로그램의 "흐름(flow)"으로 들어와 처리되고 프로그램의 밖으로 흘러나가는 "스트리밍 알고리즘"을 사용하는 것이다.
  - 알고리즘이 작은 청크로 데이터를 처리하여 전체 dataset을 한 번에 메모리에 보관하지 않는다는 것.
  - 스트리밍 솔루션이 가능해지면 메모리 효율성이 향상되고 속도도 빨라진다.
  - 노드의 네트워킹 API는 스트림 기반이며 노드의 파일 시스템 모듈은 파일을 읽고 쓰기 위한 스트리밍 API를 정의하므로, 사용자가 쓰는 많은 노드 프로그램에서 스트리밍 API를 사용할 수 있습니다.
  - 16.5.4 Reading Streams with Events의 "Flowing mode"에서 스트림 버전의  `copyFile()`을 볼수 있다.

- 노드는 4개의 기본 스트림 타입을 지원한다.
  - Readable
    - Readable 스트림은 데이터의 소스이다.
    - `fs.createReadStream()`이 반환하는 스트림은 지정된 파일의 내용을 읽을 수 있는 스트림이고, `process.stdin`은 표준 입력에서 데이터를 반환하는 Readable 스트림이다.
  - Writable
    - Writable 스트림은 데이터의 싱크 또는 목적이다.
    - `fs.createWriteStream()`의 반환 값은 Writable 스트림이다. 이 스트림은 데이터를 청크단위로 쓸 수 있도록 하며 모든 데이터를 지정된 파일에 출력한다.
  - Duplex
    - Duplex 스트림은 Readable 스트림과 Writable 스트림을 하나의 개체로 결합한다. 
    - net.connect() 및 기타 노드 네트워킹 API에서 반환되는 소켓 개체는 Duplex 스트림입니다. 소켓에 쓰면 데이터가 네트워크를 통해 소켓이 연결된 컴퓨터로 전송됩니다. 소켓에서 읽으면 다른 컴퓨터에서 작성한 데이터에 액세스할 수 있습니다.
  - Transform
    - Transform 스트림도 읽기 및 쓰기 가능하지만, Duplex 스트림과 구분되는 점은 Transform 스트림에서 쓴 데이터를 동일한 스트림에서 읽을 수 있다는 점이다.
    - zlib.createGzip() 함수는 기록된 데이터를 압축(gzip 알고리즘를 사용해)하는 Transform 스트림을 반환한다.
      crypto.createCiperiv() 함수는 기록된 데이터를 암호화하거나 암호를 해독하는 Transform 스트림을 반환한다.
- 기본적으로 스트림은 버퍼를 읽고 쓴다. 
  노드의 스트림 API는 스트림이 버퍼 및 문자열보다 복잡한 객체를 읽고 쓰는 "object mode"도 지원한다. 

- Readable 스트림은 데이터를 어디에선가로부터 읽어야 하며 Writable 스트림은 데이터를 어디엔가 써야 하므로 모든 스트림에는 '입력과 출력' 또는 '소스와 목적'의 두 끝점이 있다. 스트림 기반 API의 까다로운 점은 스트림의 양끝이 거의 항상 다른 속도로 흐른다는 것이다. 
- 스트림 구현에는 작성되었지만 아직 읽지 않은 데이터를 저장하는 내부 버퍼가 포함됩니다. 
  버퍼링은 요청된 데이터를 읽을 수 있고 데이터를 기록할 때 데이터를 보관할 수 있는 공간을 확보할 수 있도록 도와준다. 그러나 이 두 가지 모두 보장될 수 없으며, 스트림 기반 프로그래밍의 특성상 독자들은 데이터가 쓰이기를 기다리거나(스트림 버퍼가 비어있기 때문에), 작성자들은 데이터를 읽기를 기다려야 합니다(스트림 버퍼가 가득 찼기 때문에).

- 스레드 기반 동시성을 사용하는 프로그래밍 환경에서 스트림 API는  blocking 호출을 한다. =>  blocking 호출 : 데이터가 스트림에 도착할 때까지 데이터 읽기 호출이 반환되지 않으며 스트림의 내부 버퍼에 새 데이터를 수용할 공간이 충분할 때까지 데이터 블록을 쓰기 위한 호출이 반환되지 않습니다.
- 그러나 이벤트 기반 동시성 모델에서는 blocking 호출이 의미가 없고, 노드의 스트림 API는 이벤트 기반 및 콜백 기반이다. 
  다른 노드 API와 달리 이 장 뒷부분에서 설명하는 API 는"동기화" 버전의 메서드가 없다.

- 이벤트를 통해 스트림 가독성(버퍼 비어있지 않음)과 쓰기성(버퍼 가득 차지 않음)을 조정해야 하기 때문에 Node의 스트림 API는 다소 복잡하다. 

### 16.5.1 Pipes

- 스트림에서 데이터를 읽어서 동일한 데이터를 다른 스트림에 써야 할 때가 있다.
  예를 들어 정적 파일의 디렉터리를 제공하는 단순 HTTP 서버를 쓸때, 파일 입력 스트림에서 데이터를 읽고 네트워크 소켓에 기록해야 한다.
  읽기 및 쓰기를 처리하기 위해 코드를 직접 작성하는 대신 두 소켓을 "pipe"로 연결하기만 하면 복잡한 작업을 노드에서 처리할 수 있다.

```javascript
const fs = require("fs");
function pipeFileToSocket(filename, socket) {
  fs.createReadStream(filename).pipe(socket);
}
```

- 다음 유틸리티 함수는 한 스트림을 다른 스트림을 파이프로 연결하고 완료되거나 오류가 발생하면 콜백을 호출한다.

```javascript
function pipe(readable, writable, callback) {
  function handleError(err) {
    readable.close();
    writable.close();
    callback(err);
  }
  // Next define the pipe and handle the normal termination case
  readable
    .on("error", handleError)
    .pipe(writable)
    .on("error", handleError)
    .on("finish", callback);
}
```

- Transform 스트림은 파이프에서 특히 유용하며, 두개 이상의 스트림을 포함하는 파이프라인을 생성한다. 다음은 파일을 압축하는 예제 함수이다. 

```javascript
const fs = require("fs");
const zlib = require("zlib");
function gzip(filename, callback) {
  let source = fs.createReadStream(filename);
  let destination = fs.createWriteStream(filename + ".gz");
  let gzipper = zlib.createGzip();
  source
    .on("error", callback)
    .pipe(gzipper)
    .pipe(destination)
    .on("error", callback)
    .on("finish", callback);
}
```

- `pipe()`  메소드를 사용해 데이터를 Readable 스트림에서 Writable 스트림으로 복사하는 것은 쉽지만 실제로 데이터를 프로그냄 전체의 스트림에서 처리해야 하는 경우가 많다. 이를 위한 한 가지 방법은 자체적인 Transform 스트림을 구현하여 처리하는 것이다. 이 방법을 사용하면 스트림을 수동으로 읽고 쓸 필요가 없습니다. 

```javascript
const stream = require("stream");
class GrepStream extends stream.Transform {
  constructor(pattern) {
    super({decodeStrings: false});
    this.pattern = pattern;
    this.incompleteLine = "";
  }
  _transform(chunk, encoding, callback) {
    if (typeof chunk !== "string") {
      callback(new Error("Expected a string but got a buffer"));
      return;
    }
    let lines = (this.incompleteLine + chunk).split("\n");
    this.incompleteLine = lines.pop();
    let output = lines
    .filter(l => this.pattern.test(l))
    .join("\n");
    if (output) {
      output += "\n";
    }
    callback(null, output);
  }
  _flush(callback) {
    if (this.pattern.test(this.incompleteLine)) {
      callback(null, this.incompleteLine + "\n");
    }
  }
}
let pattern = new RegExp(process.argv[2]);
process.stdin
  .setEncoding("utf8")
  .pipe(new GrepStream(pattern))
  .pipe(process.stdout).
  .on("error", () => process.exit());
```

### 16.5.2 Asynchronous Iteration

- 노드 12 이상에서 Readable 스트림은 비동기식  iterator이다. 즉, 비동기 함수 내에서 for/await 루프를 사용하여 문자열이나 스트림의 버퍼를 읽을수 있다.
- 비동기  iterator를 사용하는 것은 `pipe()` 메소드를 사용하는 것만큼 쉽다. 비동기 기능과 for/wait 루프를 사용하여 이전 섹션의 grep 프로그램을 다시 작성하는 방법은 다음과 같다.

```javascript
// Read lines of text from the source stream, and write any lines
// that match the specified pattern to the destination stream.
async function grep(source, destination, pattern, encoding="utf8") {
  source.setEncoding(encoding); // 문자열을 읽기 위한 소스 스트림을 설정
  destination.on("error", err => process.exit());
  // The chunks we read are unlikely to end with a newline, so each will
  // probably have a partial line at the end. Track that here
  let incompleteLine = "";
  for await (let chunk of source) {
    let lines = (incompleteLine + chunk).split("\n");
    incompleteLine = lines.pop();
    // Now loop through the lines and write any matches to the destination
    for(let line of lines) {
      if (pattern.test(line)) {
        destination.write(line + "\n", encoding);
      }
    }
  }
  if (pattern.test(incompleteLine)) {
    destination.write(incompleteLine + "\n", encoding);
  }
}
let pattern = new RegExp(process.argv[2]);
grep(process.stdin, process.stdout, pattern)
  .catch(err => {
  console.error(err);
  process.exit();
});
```

### 16.5.3 Writing to Streams and Handling Backpressure

- 앞의 비동기 grep() 함수는 Readable 스트림을 비동기식 iterator로 사용하는 방법과 `write()` 메소드로 전달해 데이터를 Writable 스트림에 쓸 수 있다는 사실도 보여 주었다. 
-  `write()` 메소드는 버퍼 또는 문자열을 첫 번째 인수로 사용한다. 버퍼를 전달하면 해당 버퍼의 바이트가, 문자열을 전달하면 쓰기 전에 바이트 버퍼로 인코딩되어 작성한다. 
-  `write()` 은 선택적으로 콜백 함수를 세 번째 인수로 사용한다. 데이터가 실제로 기록되고 더 이상 쓰기 가능한 스트림의 내부 버퍼에 없는 경우 이 콜백이 호출된다. 에러가 발생하는 경우에도 호출될 수 있지만 보장되지는 않는다. 에러를 감지하려면 쓰기 가능한 스트림에 "error" 이벤트 핸들러를 등록해야 합니다.)
- 스트림에서 write()를 호출하면 스트림은 항상 사용자가 전달한 데이터 청크를 수신하고 버퍼링한다. 그러면 내부 버퍼가 아직 가득 차지 않으면 true,  버퍼가 이제 꽉 찼거나 너무 꽉 찬 경우 false를 반환한다. Writable 스트림은 write()를 계속 호출할 경우 내부 버퍼를 필요한 만큼 확장한다.

- `write()` 메소드에서 false의 반환 값은 backpressure의 한 형태이다. 즉 스트림에서 데이터를 처리할 수 있는 것보다 더 빨리 작성했다는 뜻이다.
- 이러한  backpressure에 대한 적절한 대응은 스트림이 다시 한 번 버퍼에 공간이 있음을 알리는 "drain" 이벤트를 방출할 때까지 write() 호출을 중지하는 것이다.

```javascript
function write(stream, chunk, callback) {
  let hasMoreRoom = stream.write(chunk);
  if (hasMoreRoom) {
    setImmediate(callback);
  } else {
    stream.once("drain", callback);
  }
}
```

- 때로는 write()를 연속으로 여러 번 호출하는 것이 괜찮고, 때로는 쓰기 사이에 이벤트를 기다려야 하기 때문에 알고리즘이 어색해진다. 
  pipe() 메소드를 사용하는 것이 매력적인 이유 중 하나로, pipe()를 사용할 때 노드가 자동으로 backpressure를 처리한다.
- 프로그램 안에서 async - await를 사용하고 Readable 스트림을 비동기 iterator로  다루고 있다면, backpressure를 적절히 처리하기 위해 위의 write() 유틸리티 함수의 Promise 기반 버전을 구현하는 것은 간단하다.
- 앞서 살펴본 비동기 grep() 기능에서는 backpressure을 처리하지 않았다. 다음 예제의 async copy() 함수는 올바르게 수행할 수 있는 방법을 보여 줍니다. 이 함수는 소스 스트림에서 목적 스트림으로 청크를 복사하기만 하며 copy(source, destination)은 source.pipe(destination)와 매우 유사하다.

```javascript
function write(stream, chunk) {
  let hasMoreRoom = stream.write(chunk);
  if (hasMoreRoom) {
    return Promise.resolve(null); 
  } else {
    return new Promise(resolve => {
      stream.once("drain", resolve); 
    });
  }
}
async function copy(source, destination) {
  destination.on("error", err => process.exit());
  for await (let chunk of source) {
    await write(destination, chunk);
  }
}
copy(process.stdin, process.stdout);
```

- 스트림에 대한 쓰기에 대한 논의를 마치기 전에 backpressure에 대응하지 못하면 Writable 스트림의 내부 버퍼가 오버플로우되어 점점 더 커질 때 프로그램이 사용해야 하는 메모리보다 더 많은 메모리를 사용하게 될 수 있고, 네트워크 서버를 쓰는 경우 이는 원격으로 악용할 수 있는 보안 문제가 될수 있다.

### 16.5.4 Reading Streams with Events

- 노드의 Readable 스트림은 각각 읽기 위한 API가 있는 두 가지 모드를 가진다. 프로그램에서 파이프나 비동기 반복을 사용할 수 없는 경우 스트림을 처리하기 위해 이 두 개의 이벤트 기반 API 중 하나를 선택해야 하고, 둘 중 하나만 사용하고 두 API를 함께 사용하지 않는 것이 중요하다.

#### Flowing mode

- Flowing mode에서는 읽을 수 있는 데이터가 도착하면 즉시 "데이터" 이벤트의 형태로 방출된다. 
- 이 모드의 스트림에서 읽으려면 이벤트 핸들러를 등록하기만 하면 스트림이 데이터 청크(버퍼 또는 문자열)를 사용할 수 있게 되는 즉시 사용자에게 푸시한다.
- Flowing mode에서는 read() 메서드를 호출할 필요가 없으며 "데이터" 이벤트만 처리하면 된다. 
- 새로 생성된 스트림은 Flowing mode에서 시작되지 않고, 데이터 이벤트 핸들러를 등록하면 스트림이 Flowing mode로 전환된다. 이것은 첫 번째 "데이터" 이벤트 핸들러를 등록할 때까지 스트림이 "데이터" 이벤트를 내보내지 않는다는 것을 의미한다.
- Flowing mode를 사용하여 readable 스트림에서 데이터를 읽고 처리한 다음 writable 스트림에 쓰려면 쓰기 가능한 스트림의 backpressure을 처리해야 할 수 있다.
  -  write() 메서드가 false를 반환하여 쓰기 버퍼가 가득 찼음을 나타낼 경우 Readable 스트림에서 pause()를 호출하여 데이터 이벤트를 일시적으로 중지할 수 있다. 그런 다음 쓰기 가능한 스트림에서 "drain" 이벤트를 수신하면 읽기 스트림에서 resume()를 호출하여 "데이터" 이벤트 흐름을 다시 시작할 수 있다.

- Flowing mode의 스트림은 스트림 끝에 도달하면 "end" 이벤트를 발생시킨다. 이 이벤트는 더 이상 "데이터" 이벤트가 발생하지 않음을 나타냅니다. 모든 스트림과 마찬가지로 에러가 발생하면 "error" 이벤트가 발생한다.

```javascript
const fs = require("fs");
function copyFile(sourceFilename, destinationFilename, callback) {
  let input = fs.createReadStream(sourceFilename);
  let output = fs.createWriteStream(destinationFilename);
  input.on("data", (chunk) => {
    let hasRoom = output.write(chunk);
    if (!hasRoom) { 
      input.pause();
    }
  });
  input.on("end", () => {
    output.end();
  });
  input.on("error", err => { 
    callback(err); 
    process.exit();
  });
  output.on("drain", () => {
    input.resume();
  });
  output.on("error", err => {
    callback(err);
    process.exit();
  });
  output.on("finish", () => {
    callback(null);
  });
}

let from = process.argv[2], to = process.argv[3];
console.log(`Copying file ${from} to ${to}...`);
copyFile(from, to, err => {
  if (err) {
    console.error(err);
  } else {
    console.log("done.");
  }
});
```

#### Paused mode

- 읽기 스트림의 다른 모드는 "Paused mode"로. 이것이 스트림이 시작되는 모드입니다.
- 데이터 이벤트 핸들러를 등록하지 않고 파이프() 메서드를 호출하지 않는 경우 읽기 스트림이 Paused mode로 유지된다.
- Paused mode에서는 스트림이 "데이터" 이벤트 형태로 데이터를 푸시하지 않는다. 대신 스트림의 read() 메서드를 명시적으로 호출하여 스트림에서 데이터를 가져온다. 이것은 blocking 호출이 아니며 스트림에서 읽을 수 있는 데이터가 없으면 null을 반환한다.
- 데이터를 대기할 동기 API가 없기 때문에 paused mode API도 이벤트 기반이다. 
- Paused mode의 읽기 스트림은 스트림에서 데이터를 읽을 수 있게 되면 "읽을 수 있는" 이벤트를 발생시키고, 이에 대해 코드가 read() 메서드를 호출하여 해당 데이터를 읽어야 한다다. null이 반환될 때까지 read()를 반복적으로 호출하여 루프에서 이 작업을 수행해야 한다.
- 미래에 새로운 "읽을 수 있는" 이벤트를 트리거하려면 이와 같은 스트림 버퍼를 완전히 드레인해야 한다. 읽을 수 있는 데이터가 있는 동안 read() 호출을 중지하면 "읽을 수 있는" 이벤트가 추가로 발생하지 않으며 프로그램이 중단될 가능성이 높다.

- Paused mode의 스트림은 flowing mode 스트림과 마찬가지로 "end" 및 "error" 이벤트를 발생시킨다. 
- Readable 스트림에서 데이터를 읽어 쓰기 가능한 스트림에 쓰는 프로그램을 작성하는 경우 Paused mode를 선택하는 것이 좋지 않을 수 있다. 
- Backpressure을 적절하게 처리하기 위해 입력 스트림을 읽을 수 있고 출력 스트림이 백업되지 않은 경우에만 읽기를 원할 수 있다.Paused mode서는 read()가 null을 반환하거나 write()가 false를 반환한 다음 읽기 또는 쓰기 이벤트를 다시 시작한다는 의미인데, 이 경우 flowing mode(또는 파이프)가 더 쉽다는 것을 알 수 있다

```javascript
const fs = require("fs");
const crypto = require("crypto");
function sha256(filename, callback) {
  let input = fs.createReadStream(filename);
  let hasher = crypto.createHash("sha256");
  input.on("readable", () => {
    let chunk;
    while(chunk = input.read()) {
      hasher.update(chunk);
    }
  });
  input.on("end", () => {
    let hash = hasher.digest("hex");
    callback(null, hash);
  });
  input.on("error", callback);
}

sha256(process.argv[2], (err, hash) => {
  if (err) {
    console.error(err.toString());
  } else { 
    console.log(hash); 
  }
});
```

