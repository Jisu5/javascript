# Chapter 16. Server-Side JavaScript with Node (5)

## 16.8 HTTP Clients and Servers

- 노드의 “http,” “https,”,  “http2” 모듈은 모든 기능을 제공하지만, 상대적으로 저레벨 수준의 Http 프로토콜로 HTTP 클라이언트 및 서버를 구현하기 위한 포괄적인 API를 정의한다.

- HTTP GET 요청을 만드는 가장 간단한 방법은 http.get() 또는 https.get()을 사용하는 것이다. 첫 번째 인자는 URL, 두 번째 인자는 서버의 응답이 도착할 때 IncomingMessage 객체와 함께 호출되는 콜백이다.

- 콜백을 호출할 때 HTTP status 및 헤더를 사용할 수 있지만 body는 아직 준비되지 않았을 수 있다.
- 다음 postJSON() 함수는 https.request()를 사용하여 JSON request body을 포함하는 HTTPS POST 요청을 만드는 방법을 보여 준다.

```javascript
const https = require("https");
// body 객체를 JSON 문자열로 변화해서 HTTP POST
function postJSON(host, endpoint, body, port, username, password) {
  // 즉시 Promise 객체를 반환하고, https request가 성공 혹은 실패하면 resolve, reject를 호출.
  return new Promise((resolve, reject) => {
    let bodyText = JSON.stringify(body);
    let requestOptions = {
      method: "POST",
      host: host,
      path: endpoint,
      headers: {
        "Content-Type": "application/json",
        "Content-Length": Buffer.byteLength(bodyText)
      }
    };
    if (port) {
      requestOptions.port = port;
    }
    // credentials이 있다면, Authorization header에 추가
    if (username && password) {
      requestOptions.auth = `${username}:${password}`;
    }
    
    let request = https.request(requestOptions);
    
    request.write(bodyText);
    request.end();
    
    request.on("error", e => reject(e)); // 네트워크 연결 실패 등 에러
    
    request.on("response", response => {
      if (response.statusCode !== 200) {
        reject(new Error(`HTTP status ${response.statusCode}`));
        response.resume();
        return;
      }
      response.setEncoding("utf8");
      let body = "";
      response.on("data", chunk => { body += chunk; });
      response.on("end", () => {
        try {
          resolve(JSON.parse(body));
        } catch(e) {
          reject(e);
        }
      });
    });
  });
}
```

- "http" 및 "https" 모듈을 사용해 HTTP, HTTPS 요청에 응답하는 서버를 작성할 수 있다. 

  • 새 서버 객체를 생성

  • 지정된 포트에서 요청을 수신하려면 listen() 메서드를 호출

  • "request" 이벤트에 대한 이벤트 핸들러를 등록하고, 핸들러를 사용하여 클라이언트의 요청(특히 request.url)을 읽고, response를 작성합니다.

- 다음 예제는 로컬 파일 시스템에서 정적 파일을 처리하는 간단한 HTTP 서버를 만드는 예제이다.

```javascript
const http = require("http");
const url = require("url");
const path = require("path");
const fs = require("fs");

function serve(rootDirectory, port) {
  let server = new http.Server();
  server.listen(port);
  console.log("Listening on port", port);
  server.on("request", (request, response) => {
    let endpoint = url.parse(request.url).pathname;
    if (endpoint === "/test/mirror") {
      response.setHeader("Content-Type", "text/plain; charset=UTF-8");
      response.writeHead(200); // status code
      response.write(`${request.method} ${request.url} HTTP/${
                     request.httpVersion
                     }\r\n`);
      
      let headers = request.rawHeaders;
      for(let i = 0; i < headers.length; i += 2) {
        response.write(`${headers[i]}: ${headers[i+1]}\r\n`);
      }
      response.write("\r\n");
      // request body를 response body에 복사
      request.pipe(response);
    }
    else {
      // 로컬 파일 시스템에 엔드포인트 매핑
      let filename = endpoint.substring(1);
      filename = filename.replace(/\.\.\//g, ""); // "../" => ""
      filename = path.resolve(rootDirectory, filename);
      let type;
      switch(path.extname(filename)) {
        case ".html":
        case ".htm": type = "text/html"; break;
        case ".js": type = "text/javascript"; break;
        case ".css": type = "text/css"; break;
        case ".png": type = "image/png"; break;
        case ".txt": type = "text/plain"; break;
        default: type = "application/octet-stream"; break;
      }
      let stream = fs.createReadStream(filename);
      stream.once("readable", () => {
        response.setHeader("Content-Type", type);
        response.writeHead(200);
        stream.pipe(response);
      });
      stream.on("error", (err) => {
        response.setHeader("Content-Type", "text/plain; charset=UTF-8");
        response.writeHead(404);
        response.end(err.message);
      });
    }
  });
}

serve(process.argv[2] || "/tmp", parseInt(process.argv[3]) || 8000);
```



## 16.9 Non-HTTP Network Servers and Clients

- 네트워크 소켓은 일종의 Duplex 스트림이기 때문에 네트워킹이 비교적 간단하다.
- net 모듈은 서버 및 소켓 클래스를 정의한다.
- 서버를 생성하려면 `net.createServer()`를 호출한 다음 결과 개체의 listen() 메서드를 호출하여 서버에 연결을 수신할 포트를 지정한다. 클라이언트가 해당 포트에 연결할 때 서버 객체가 "connection" 이벤트를 생성하며 이벤트 리스너에 전달되는 값은 소켓 객체이다. 소켓 객체는 duplex 스트림이며 이를 사용하여 클라이언트에서 데이터를 읽고 쓸 수 있다. 연결을 끊으려면 소켓의 end()를 호출한다.

- 클라이언트를 작성하는 것은 훨씬 쉽다. 포트 번호와 호스트 이름을 `net.createConnection()`에 전달하여 해당 호스트에서 실행 중인 모든 서버와 통신하는 소켓을 생성한 다음 이 소켓을 사용하여 서버 간에 데이터를 읽고 씁니다.

```javascript
// 6789 포트에서 knock-knock joke를 전달하는 TCP 서버
const net = require("net");
const readline = require("readline");
let server = net.createServer();
server.listen(6789, () => console.log("Delivering laughs on port 6789"));

server.on("connection", socket => {
  tellJoke(socket)
    .then(() => socket.end())
    .catch((err) => {
    console.error(err);
    socket.end();
  });
});

const jokes = {
  "Boo": "Don't cry...it's only a joke!",
  "Lettuce": "Let us in! It's freezing out here!",
  "A little old lady": "Wow, I didn't know you could yodel!"
};

async function tellJoke(socket) {
  let randomElement = a => a[Math.floor(Math.random() * a.length)];
  let who = randomElement(Object.keys(jokes));
  let punchline = jokes[who];
  // readline 모듈로 사용자의 입력을 한줄씩 받음
  let lineReader = readline.createInterface({
    input: socket,
    output: socket,
    prompt: ">> "
  });

  function output(text, prompt=true) {
    socket.write(`${text}\r\n`);
    if (prompt) lineReader.prompt();
  }

  let stage = 0;
  output("Knock knock!");
  for await (let inputLine of lineReader) {
    if (stage === 0) {
      if (inputLine.toLowerCase() === "who's there?") {
        output(who);
        stage = 1;
      } else {
        output('Please type "Who\'s there?".');
      }
    } else if (stage === 1) {
      if (inputLine.toLowerCase() === `${who.toLowerCase()} who?`) {
        output(`${punchline}`, false);
        return;
      } else {
        output(`Please type "${who} who?".`);
      }
    }
  }
}
```

- 위와 같은 간단한 텍스트 기반 서버에는 일반적으로 사용자 정의 클라이언트가 필요하지 않는다.
- 시스템에 nc("netcat") 유틸리티가 설치되어 있는 경우 다음과 같이 유틸리티를 사용하여 이 서버와 통신할 수 있다.

```bash
$ nc localhost 6789
Knock knock!
>> Who's there?
A little old lady
>> A little old lady who?
Wow, I didn't know you could yodel!
```

- 노드에서 joke 서버에 대한 사용자 지정 클라이언트를 작성하는 것은 쉽다. 서버에 연결한 다음 서버의 출력을 stdout에 연결하고 stdin을 서버의 입력에 연결하기만 하면 된다.

```javascript
// Connect to the joke port (6789) on the server named on the command line
let socket = require("net").createConnection(6789, process.argv[2]);
socket.pipe(process.stdout);
process.stdin.pipe(socket);
socket.on("close", () => process.exit());
```

- TCP 기반 서버 지원 외에도 노드의 "net" 모듈은 포트 번호가 아닌 파일 시스템 경로로 식별되는 "Unix 도메인 소켓"을 통한 프로세스 간 통신도 지원한다다.  여기에 공간이 없는 다른 노드 기능에는 UDP 기반 클라이언트 및 서버용 "dgram" 모듈과 "https"로 "net"할 "tls" 모듈이 포함된다. 
- tls.Server 및 tls.TLS 소켓 클래스를 사용하면 HTTPS 서버처럼 SSL로 암호화된 연결을 사용하는 TCP 서버(knock knock joke server 등)를 생성할 수 있다.



## 16.10 Working with Child Processes

- Node는 동시성이 높은 서버로 다른 프로그램을 실행하는 스크립트 작성에도 적합하다. 
- 노드에서 `child_process` 모듈은 다른 프로그램을 하위 프로세스로 실행하기 위한 함수를 정의한다. 

### 16.10.1 execSync() and execFileSync()

- 다른 프로그램을 실행하기 위한 가장 쉬운 방법은 `child_process.execSync()`을 사용하는 것이다.
  - 이 함수는 실행을 위한 command를 첫 인자로 받는다.
  - 하위 프로세스를 생성하고, 그 프로세스는 셸을 실행해 전달한 command를 수행한다.
  - 그런 다음 명령(및 셸)이 종료될 때까지 blocking된다.
  - 명령이 오류와 함께 종료되면 execSync()가 예외를 발생시키고, 그렇지 않으면 execSync()는 명령이 stdout 스트림에 쓰는 모든 출력을 반환한다.
  - 기본적으로 이 반환 값은 버퍼이지만 선택적 두 번째 인자에 인코딩을 지정하여 문자열을 가져올 수 있다
  -  명령이 출력을 stderr에 쓰는 경우 해당 출력은 상위 프로세스의 stderr 스트림으로 전달된다.

```javascript
const child_process = require("child_process");
let listing = child_process.execSync("ls -l web/*.html", {encoding: "utf8"});
```

- `execSync()`가 Unix shell을 호출한다는 것은 전달하는 문자열에 세미콜론으로 구분된 명령이 여러 개 포함될 수 있으며 파일 이름 와일드카드, 파이프 및 출력 리디렉션과 같은 shell 기능을 활용할 수 있음을 의미한다.
  - 사용자 입력 등에서 가져온 명령을 execSync()에 전달하지 않도록 주의해야 한다. 공격자가 임의 코드를 실행할 수 있도록 셸 명령의 복잡한 구문을 쉽게 변환할 수 있기 때문이다.

- 셸 기능이 필요하지 않은 경우 `child_process.execFileSync()`를 사용하여 셸을 시작하는 오버헤드를 방지할 수 있다. 
  - 이 함수는 셸을 호출하지 않고 프로그램을 직접 실행한다. 
  - 셸이 포함되어 있지 않으므로 command line을 pasing 할 수 없고, 실행어를 첫 번째 인자로 전달하고 command line 인자 배열을 두 번째 인자로 전달해야 한다

```javascript
let listing = child_process.execFileSync("ls", ["-l", "web/"],
                                         {encoding: "utf8"});
```

