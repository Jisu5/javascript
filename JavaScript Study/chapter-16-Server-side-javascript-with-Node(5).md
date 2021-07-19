#

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
      // Now we need to copy any request body to the response body
      // Since they are both streams, we can use a pipe
      request.pipe(response);
    }
    else {
      // Map the endpoint to a file in the local filesystem
      let filename = endpoint.substring(1); // strip leading /
      // Don't allow "../" in the path because it would be a security
      // hole to serve anything outside the root directory.
      filename = filename.replace(/\.\.\//g, "");
      // Now convert from relative to absolute filename
      filename = path.resolve(rootDirectory, filename);
      // Now guess the type file's content type based on extension
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
        // If the stream becomes readable, then set the
        // Content-Type header and a 200 OK status. Then pipe the
        // file reader stream to the response. The pipe will
        // automatically call response.end() when the stream ends.
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
// A TCP server that delivers interactive knock-knock jokes on port 6789.
// (Why is six afraid of seven? Because seven ate nine!)
const net = require("net");
const readline = require("readline");
// Create a Server object and start listening for connections
let server = net.createServer();
server.listen(6789, () => console.log("Delivering laughs on port 6789"));
// When a client connects, tell them a knock-knock joke.
server.on("connection", socket => {
  tellJoke(socket)
    .then(() => socket.end()) // When the joke is done, close the socket.
    .catch((err) => {
    console.error(err); // Log any errors that occur,
    socket.end(); // but still close the socket!
  });
});
// These are all the jokes we know.
const jokes = {
  "Boo": "Don't cry...it's only a joke!",
  "Lettuce": "Let us in! It's freezing out here!",
  "A little old lady": "Wow, I didn't know you could yodel!"
};
// Interactively perform a knock-knock joke over this socket, without blocking.
async function tellJoke(socket) {
  // Pick one of the jokes at random
  let randomElement = a => a[Math.floor(Math.random() * a.length)];
  let who = randomElement(Object.keys(jokes));
  let punchline = jokes[who];
  // Use the readline module to read the user's input one line at a time.
  let lineReader = readline.createInterface({
    input: socket,
    output: socket,
    prompt: ">> "
  });
  // A utility function to output a line of text to the client
  // and then (by default) display a prompt.
  function output(text, prompt=true) {
    socket.write(`${text}\r\n`);
    if (prompt) lineReader.prompt();
  }
  // Knock-knock jokes have a call-and-response structure.
  // We expect different input from the user at different stages and
  // take different action when we get that input at different stages.
  let stage = 0;
  // Start the knock-knock joke off in the traditional way.
  output("Knock knock!");
  // Now read lines asynchronously from the client until the joke is done.
  for await (let inputLine of lineReader) {
    if (stage === 0) {
      if (inputLine.toLowerCase() === "who's there?") {
        // If the user gives the right response at stage 0
        // then tell the first part of the joke and go to stage 1.
        output(who);
        stage = 1;
      } else {
        // Otherwise teach the user how to do knock-knock jokes.
        output('Please type "Who\'s there?".');
      }
    } else if (stage === 1) {
      if (inputLine.toLowerCase() === `${who.toLowerCase()} who?`) {
        // If the user's response is correct at stage 1, then
        // deliver the punchline and return since the joke is done.
        output(`${punchline}`, false);
        return;
      } else {
        // Make the user play along.
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
socket.pipe(process.stdout); // Pipe data from the socket to stdout
process.stdin.pipe(socket); // Pipe data from stdin to the socket
socket.on("close", () => process.exit()); // Quit when the socket closes.
```

- TCP 기반 서버 지원 외에도 노드의 "net" 모듈은 포트 번호가 아닌 파일 시스템 경로로 식별되는 "Unix 도메인 소켓"을 통한 프로세스 간 통신도 지원한다다.  여기에 공간이 없는 다른 노드 기능에는 UDP 기반 클라이언트 및 서버용 "dgram" 모듈과 "https"로 "net"할 "tls" 모듈이 포함된다. 
- tls.Server 및 tls.TLS 소켓 클래스를 사용하면 HTTPS 서버처럼 SSL로 암호화된 연결을 사용하는 TCP 서버(knock knock joke server 등)를 생성할 수 있다.

