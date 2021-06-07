#  Chapter 16. Server-Side JavaScript with Node

- Node는 기본 운영체제에 바인딩된 javascript로, 파일 읽고 쓰기, 자식 프로세스 수행, 네트워크 커뮤니케이션 등이 가능한 javascript를 작성할 수 있게 한다.
- Node는 다음과 같은 부분에서 유용하다.
  - bash나 Unix shell의 난해한 구문에 시달리지 않는 셀 스크립트의 현대적 대안
  - 신뢰할 수 있는 프로그램을 실행하기 위한 범용 프로그래밍 언어이며, 신뢰할 수 없는 코드에 대해 웹 브라우저가 부과하는 보안 제약 조건에 영향을 받지 않는다.
  - 효율적이고 동시성이 높은 웹서버를 작성하기 위해 널리 사용되는 환경

- Node의 정의 기능은 비동기식 API에서 사용할 수 있는 **싱글 스레드 이벤트 기반의 동시성**이다.
- 이번 챕터에서는  Node 프로그래밍 모델에 대한 설명과 동시성, 스트리밍 데이터로 작동하는 Node API, binary 데이터로 작동하는 Node Buffer 타입을 설명한다.
- 초기 섹션에서는 파일, 네트워크, 프로세스, 스레드를 포함한 가장 중요한 Node API중 일부를 다루고 있다.

## 16.1 Node Programming Basics

> Node 프로그램의 구조와 운영체제와의 상호작용

### 16.1.1 Console Output

- 웹 브라우저 자바스크립트 프로그래밍을 할 때, Node의 `console.log()`는 디버깅 뿐 아니라 유저에게 메세지를 표시하거나 stdout 스트림의 결과를 보내는 일반적이고 가장 쉬운 방법이다.

```javascript
console.log("Hello World!");
```

- stdout을 작성할 수 있는 lower-level의 방법이 있지만 `console.log()`를 호출하는 것보다 더 공식적인 방법은 아니다. 

- 웹브라우저에서 `console.log()`, `console.warn()`, `console.error()`는 로그 메세지를 구분하기 위해 출력 옆에 작은 아이콘을 표시하고 있다. Node는 이렇지 않지만, `console.error()`의 결과는 stderr 스트림으로 작성되기 때문에 `console.log()`의 화면에 나타나는 결과와 구분된다. 
- Node를 사용해 stdout을 파일이나 pipe로 리다이렉트 하는 프로그램을 작성한다면 , `console.log()`로 출력하는 텍스트가 숨겨져있더라도 `console.error()`를 사용해 유저가 볼수있는 콘솔에 텍스트를 출력할 수 있다. 

### 16.1.2 Command-Line Arguments and Environment Variables

- 이전에 작성된 터미널이나 커맨드에서 호출하도록 설계된 Unix 스타일의 프로그램은 입력값을 주로 커맨드 라인 인자나 환경변수로부터  얻는다.
  Node도 이러한 Unix convention을 따른다.
- Node 프로그램은 `process.argv`라는 문자열의 배열로부터 커맨드라인 인자를 읽을 수 있다.
- 이 배열의 첫 번째 원소는 항상 node 실행파일의 경로이고, 두번째 인수는 Node가 실행중인 Javascript 코드 파일의 경로이다.
- 이 배열의 남은 원소는 Node를 호출할 때 커맨드라인으로 전달되는 공백으로 구분된 인수들이다.
- 짧은 Node 프로그램을 argv.js라는 파일에 저장했다고 가정하자.

```javascript
// argv.js
console.log(process.argv);
```

- 이 프로그램을 실행하면 아래와 같은 결과를 볼 수 있다.

```bash
$ node --trace-uncaught argv.js --arg1 --arg2 filename
[
  '/usr/local/bin/node',
  '/private/tmp/argv.js',
  '--arg1',
  '--arg2',
  'filename'
]
```

- 여기에는 기억해야할 몇가지가 있다.
  - `process.argv`의 첫 번째와 두 번째 원소는 Node 실행파일과 실행되는 javascript 파일에 대한 완전한 파일시스템 경로가 된다.
  - Node 실행파일로 해석되고 의도된 커맨드라인 인자는 Node 실행파일 의해 사용되므로 `process.argv`에서 나타나지 않는다.
    (`--trace-uncaught` 커맨드라인 인자는 앞에 있는 예제에서는 실제로 아무것도 하지 않고, 단지 output에 나타나지 않는 것을 보여주기 위해 존재한다. )
    --arg1, filename 같은 javascript 파일 이름 뒤에 나타나는 다른 인자들은 `process.argv`에서 나타난다.
- Node 프로그램은 Unix 스타일의 환경변수로부터 입력값을 받을 수 있다.
  - 이러한 변수들은 `process.env` 객체를 통해 사용할 수 있다.
  - 이 객체의 프로퍼티 이름은 환경변수 이름이고, 프로퍼티 값(항상 문자열)은 각 환경 변수의 값이다.
- 아래는 환경변수의 일부 리스트이다.

```bash
$ node -p -e 'process.env'
{
  SHELL: '/bin/bash',
  USER: 'david',
  PATH: '/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin',
  PWD: '/tmp',
  LANG: 'en_US.UTF-8',
  HOME: '/Users/david',
}
```

- `node -h` 혹은 `node --help`를 통해 `-p`와 `-e` 커맨드라인 인자가 무엇을 하는지 알수있다.
  - `-p` = `--print` , `-e` = `--eval`

### 16.1.3 Program Life Cycle

- Node커맨드는 커맨드라인의 인자로 특정하는 javascript 코드 파일을 실행시킨다.
- 초기 파일은 javascript 코드의 모듈을 import 하고, 자기자신만의 클래스와 함수를 정의한다.
- 근본적으로 Node는 지정한 파일의 javascript 코드를 위에서 아래로 실행하며, 몇몇 Node 프로그램들은 파일의 마지막 코드 라인을 수행하면 끝이 난다.
  하지만 종종 Node 프로그램은 초기 파일을 수행하고 나서도 계속 돌아가고 있다.
- 이 섹션에서 Node 프로그램이 비동기적이고 콜백과 이벤트 핸들러 기반임을 논의할 것이다. 
  Node 프로그램은 초기 파일을 수행하고 모든 콜백이 호출되고 보류중인 이벤트가 없을 때까지 종료되지 않는다.
  즉, 들어오는 네트워크 커넥션을 수하는 node-based 서버 프로그램은 항상 더 많은 이벤트를 기다리기 때문에 이론적으로 영원히 끝나지 않는다.
- 프로그램은 `process.exit()`호출을 통해 종료할 수 있고, 사용자는 터미널에서`Ctrl-C`를 입력해 실행 중인 Node 프로그램을 종료할 수있다.
  프로그램에 signal handler function을 등록해 `Ctrl-C `를 무시할 수도 있다. (`process.on("SIGINT", ()=>{})`)

- 프로그램의 코드가 예외를 던지고 그걸 처리하는 어떠한 catch 구절도 없다면, 프로그램은 stack trace를 출력하고 종료한다.
- node의 비동기적 특성 때문에 콜백이나 이벤트 핸들러에서 발생하는 예외는 로컬에서 처리되거나 전혀 처리되지 않아야 하며, 이는 프로그램의 비동기적인 부분에서 발생하는 예외를 처리하는 것이 어려운 문제가 될 수 있음을 의미한다.
  이러한 예외로 인해 프로그램이 완전히 중단되지 않도록 하려면 중단 대신 호출될 global handler function을 등록해야 한다.

```javascript
process.setUncaughtExceptionCaptureCallback(e => {
  console.error("Uncaught exception:", e);
});
```

- 만약 프로그램에서 만들어진 Promise가 reject 되고 이것을 처리할 `.catch()` 호출이 없다면 비슷한 상황이 발생한다.
  Node 13에서는 이것이 프로그램을 종료할 정도의 치명적이 에러는 아니지만, 상세한 에러 메세지를 콘솔에 출력한다.

- 앞으로의 Node 버전에서는, 처리되지 않은 Promise rejection이 치명적이 오류가 될 것으로 예상된다.
  처리되지 않은 Rejection을 원하지 않는다면, global handler function을 등록해 에러메세지를 출력하거나  프로그램을 종료해야 한다.

```javascript
process.on("unhandledRejection", (reason, promise) => {
  // reason은 .catch() function으로 전달되는 값이다.
  // promise는 reject된 Promise 이다.
});
```

### 16.1.4 Node Modules

- Chapter 10에서는 Node 모듈과 ES6 모듈 모두를 다루는 자바스크립트 모듈 시스템에 대한 설명이 있었다.
  - 자바스크립트가 모듈 시스템을 갖추기 전에 Node가 만들어 졌기 때문에, Node는 자기자신만의 모듈 시스템을 생성해야 했다. 
  - Node 모듈 시스템은 `require()` 함수를 통해 모듈로 값을 가져오고, export object 혹은 `module.exports` 프로프티로 모듈에서 값을 내보낸다. (chapter 10.2)

- Node13은 표준 ES6 모듈과 require-based 모듈 ("CommonJS modules") 에 대한 지원을 추가했다 
  - 두 모듈 시스템이 완전히 호환되지 않기 때문에 이 작업은 다소 까다롭다. 
  - Node는 모듈을 로드하기 전에 `require()`과 `module.exports`를 사용할 건지 `import`와 `export`를 사용할 건지 알아야 한다.
  - Node가 Javascript 코드파일을 `CommonJS module`로서 로드할 때, 식별자 `export`, `module`과 함께 `require()` 함수를 자동으로 정의하며, `import`, `export`키워드를 활성화하지 않는다.
  - 반면에 Node가 코드 파일을 ES6 모듈로 로드할 때, `import` 및 `export` 선언을 활성화해야 하며,  `require()`, `module`및 `export` 같은 추가 식별자를 정의해서는 안 된다.
- 로드 중인 모듈의 종류를 node에게 알려주는 가장 간단한 방법은 파일 확장자에서 이 정보를 인코딩하는 것이다.
  - `.mjs`로 끝나는 파일에 JavaScript 코드를 저장하면 node는 항상 이 코드를 ES6 모듈로 로드하고 `import` 및 `export`를 사용할 것으로 예상하며 `require()` 기능을 제공하지 않는다.
  - `.cjs`로 끝나는 파일에 코드를 저장하면 node가 항상 CommonJS module로 간주하며, `require()` 함수를 제공하고 `import` 또는 `export`선언을 사용하는 경우 SyntaxError를 발생시킵니다.

- 명시적 .mjs 또는 .cjs 확장자가 없는 파일의 경우, node는 파일과 동일한 디렉토리에서 `package.json`이라는 이름을 가진 파일을 찾는다.
  - 가장 가까운 `package.json` 파일이 발견되면 Node는 JSON 객체에서 최상위 type 프로퍼티를 확인한다.
    형식 속성 값이 `module`이면 node는 파일을 ES6 모듈로 로드하고, 해당 속성의 값이 `commonjs`이면 node는 파일을 `CommonJS module`로 로드한다.
  - Node 프로그램을 실행하는데  `package.json`을 꼭 가지고 있어야 하는 것은 아니다.
    만약 `package.json`파일이 없거나 파일이 있더라도 type 프로퍼티가 없을때, Node는 `CommonJS module`을 기본적으로 사용하게 된다.
  - `package.json` 은 node에서 ES6 모듈을 사용하고 .mjs 파일 확장자를 사용하지 않으려는 경우에만 필요하다. 
  - `CommonJS module`을 사용하여 작성된 기존 node 코드가 어마어마하기 때문에, node는 ES6 모듈이 `import`  키워드를 사용하여  `CommonJS module`를 로드하도록 허용한다.
    그러나 반대는 허용되지 않는다.  `CommonJS module`는 ES6 모듈을 로드하는 데 `require()`를 사용할 수 없다.

### 16.1.5 The Node Package Manager

- Node를 설치할 때 일반적으로 npm이라는 프로그램도 제공된다.
  - 이것은 Node Package Manager이며, 프로그램이 의존하는 라이브러리를 다운로드하고 관리하는 데 도움이 된다.
  - npm은 프로젝트의 루트 디렉터리에 있는 `package.json` 파일에 있는 의존성(프로그램에 대한 다른 정보)을 추적합니다.
  - npm에서 만든 이 `package.json` 파일은 프로젝트에 ES6 모듈을 사용하려는 경우 `"type":"module"`을 추가하는 곳이다.

- 예를 들어 웹 서버를 개발하고 Express 프레임워크(https://expressjs.com)를 사용하여 작업을 단순화할 계획이라고 가정해보자.
  - 시작하려면 프로젝트에 대한 디렉토리를 생성한 다음 해당 디렉토리에 `npm init`을 입력한다.
  - npm은 프로젝트 이름, 버전 번호 등을 물어본 다음 응답을 기반으로 초기  `package.json` 파일을 만든다.
  - 이제 Express를 사용하려면 `npm install Express`를 입력한다. 이것은 npm에게 모든 의존성과 함께 Express 라이브러리를 다운로드하고 모든 패키지를 로컬 "node_modules/" 디렉토리에 설치하라고 지시한다.

```bash
$ npm install express
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN my-server@1.0.0 No description
npm WARN my-server@1.0.0 No repository field.

+ express@4.17.1
added 50 packages from 37 contributors and audited 126 packages in 3.058s
found 0 vulnerabilities
```

- 패키지를 npm으로 설치할 때, npm은 프로젝트가 Express에 종속된다는 이 종속성을 `package.json` 파일에 기록한다.
- 이 종속성을  `package.json` 에 기록하면, 다른 프로그래머에게 코드 복사본과  `package.json` 을 줄 수 있으며, 그들은 단순히 npm install을 입력하여 프로그램을 실행하기 위해 필요한 모든 라이브러리를 자동으로 다운로드하고 설치할 수 있다.

## 16.2 Node Is Asynchronous by Default

- 자바스크립트는 범용 프로그래밍 언어이므로 큰 행렬을 곱하거나 복잡한 통계 분석을 수행하는 CPU 집약적 프로그램을 작성하는 것이 완벽하게 가능하다.
- 그러나 Node는 I/O 집약적인 네트워크 서버와 같은 프로그램에 맞게 설계 및 최적화되어 있다. 특히 많은 요청을 동시에 처리할 수 있는 고도로 동시적인 서버(concurrent server) 를 쉽게 구현할 수 있도록 설계되었다.

- 그러나 많은 프로그래밍 언어와 달리 node는 멀티스레드로 동시성을 얻지 않는다.
  - 멀티스레드 프로그래밍은 정확하게 수행하기가 어렵고 디버그하기도 어려운 것으로 악명이 높다.
  - 또한 스레드는 상대적으로 무거운 추상화이므로 수백 개의 동시 요청을 처리할 수 있는 서버를 작성하려면 수백 개의 스레드를 사용하는 데 엄청난 양의 메모리가 필요할 수 있다.
  - 그래서 node에서는 웹에서 사용하는 단일 스레드 자바스크립트 프로그래밍 모델을 채택하고 있으며, 이는 네트워크 서버 생성을 난해한 기술에서 굉장히 단순화한 기술로 바꾸었다.

```html
True Parallelism with Node
- Node 프로그램은 여러 운영 체제 프로세스를 실행할 수 있으며, 
  node 10 이상에서는 웹 브라우저에서 빌린 스레드의 일종인 Worker 객체(§ 16.11)를 지원한다.
- 여러 프로세스를 사용하거나 하나 이상의 Worker 스레드를 만들고 둘 이상의 CPU가 있는 시스템에서 프로그램을 실행하면,
  프로그램이 더 이상 단일 스레드가 되지 않고 프로그램이 여러 코드 스트림을 동시에 실행합니다.
- 이러한 기술은 CPU 집약적인 작업에는 유용하지만 서버 같은 I/O 집약적인 프로그램에는 일반적으로 사용되지 않습니다.
- 그러나 node 프로세스 및 Worker 간 통신은 메시지 전달을 통해 이루어지며,
  서로 쉽게 메모리를 공유할 수 없기 때문에
  node 프로세스와 Worker가 다중 스레드 프로그래밍의 전형적인 복잡성을 피한다는 점은 주목할 필요가 있다.
```

- Node는 디폴트로 API를 비동기화 및 `non-blocking` 함으로써 단일 스레드 프로그래밍 모델을 유지하면서 높은 수준의 동시성을 달성한다.
- 네트워크에서 읽고 쓰는 함수가 비동기화될 것으로 예상했겠지만, Node는 더 나아가 로컬 파일 시스템에서 파일을 읽고 쓰기위해 `non-blocking asyncronous`  함수를 정의한다.
  - 이것은 Node API가 spinning hard driver가 표준이었던 시기에 설계되었던 것을 생각하면 이해할 수 있다. Spinning hard driver는 실제로 파일 작업을 시작하기 전 디스크 회전을 기다리는 동안 탐색시간을 blocking 하는데 밀리초가 걸렸었다.
  - 또한 현대의 데이터센터에서는 "로컬" 파일 시스템이 네트워크 전반에 걸쳐 있을 수 있으며, 네트워크 대기 시간이 드라이브 대기 시간보다 더 오래 걸릴 수 있다.
  - 하지만 파일을 비동기적으로 읽는 것이 평범하게 보일지라도, node는 더 나아가 네트워크 커넥션을 시작하거나 파일 수정시간을 조회하는 디폴트 함수조차도 `non-blocking` 이다.

- Node API의 몇몇 함수는 동기적이지만 `nonbloking`이다. Block되지 않고도 완료되고 반환된다.
- 그러나 대부분의 흥미로운 함수들은 어떤 종류의 입력이나 출력을 수행하며, 이것들은 비동기 함수여서 아주 작은 양의 block도 피할 수 있다. 

- node가 JavaScript에 Promise 클래스가 있기 전에 생성되었으므로 비동기 node API는 콜백 기반이다.
- 일반적으로 비동기 node 함수에 전달하는 마지막 인수는 콜백이다.
  - node에서는 일반적으로 두 개의 인수로 호출되는 error-first 콜백을 사용한다.
  - error-first 콜백에 대한 첫 번째 인수는 오류가 발생하지 않은 경우 일반적으로 null이며, 두 번째 인수는 호출한 원래 비동기 함수에 의해 생성된 데이터 또는 응답이다.
  - Error 인수를 우선하는 이유는 생략할 수 없도록 만들기 위한 것이며, 이 인수에 Null이 아닌 값이 있는지 항상 확인해야 한다.
  - 오류 개체이거나 정수 오류 코드 또는 문자열 오류 메시지인 경우 문제가 발생한 것이고, 이 경우 콜백 함수에 대한 두 번째 인수가 null일 수 있다.

- 다음 코드는 nonblocking readFile() 함수를 사용하여 configuration 파일을 읽고 JSON으로 파싱한 다음, 파싱된 configuration 객체를 다른 콜백에 전달하는 방법을 보여준다.

```javascript
const fs = require("fs"); // filesystem module
function readConfigFile(path, callback) {
  fs.readFile(path, "utf8", (err, text) => {
    if (err) {
      console.error(err);
      callback(null);
      return;
    }
    let data = null;
    try {
      data = JSON.parse(text);
    } catch(e) {
      console.error(e);
    }
    callback(data);
  });
}
```

- node는 표준화된 Promise보다 앞서지만 error-first 콜백에 대해 상당히 일관적이기 때문에 util.promisify() 래퍼를 사용하여 콜백 기반 API의 Promise 기반 변형을 쉽게 만들 수 있다. readConfigFile()을 다시 작성하여 을 Promise를 반환하는 방법은 다음과 같다.

```javascript
const util = require("util");
const fs = require("fs");
const pfs = {
  readFile: util.promisify(fs.readFile)
};
function readConfigFile(path) {
  return pfs.readFile(path, "utf-8").then(text => {
    return JSON.parse(text);
  });
}
```

- 또한 async await를 사용해 위의 코드를 단순화 시킬수 있다.

```javascript
async function readConfigFile(path) {
  let text = await pfs.readFile(path, "utf-8");
  return JSON.parse(text);
}
```

- util.promisify() 래퍼는 많은 node 함수의 Promise 기반 버전을 생성할 수 있다. 
  - node 10 이상에서 `fs.promise` 객체에는 파일 시스템 작업을 위한 미리 정의된 Promise 기반 함수가 많이 있다.
  - 이 장의 뒷부분에서 이 문제에 대해 논의하겠지만, 이전 코드에서는 `pfs.readFile()`을 `fs.promises.readFile()`로 대체할 수 있다는 점에 유의해야 한다.
- Node의 프로그래밍 모델이 기본적으로 비동기이지만, 프로그래머의 편의를 위해 Node에서는 (특히 filesystem 모듈에서) 많은 함수의 blocking, synchronous 변형을 정의한다. 일반적으로 이러한 함수에는 끝에 Sync(동기화)라는 분명한 라벨이 붙는다.
- 서버가 처음 시작되고 configuration 파일을 읽을 때, 아직 네트워크 요청을 처리하지 않으며, 실제로 거의 또는 전혀 동시성이 가능하지 않는다.
  - 따라서 이 상황에서는 blocking을 피할 필요가 없으며 fs.readFileSync()와 같은 blocking function을 안전하게 사용할 수 있다.
  - async await 코드를 삭제하고 readConfigFile() 함수의 순수 동기 버전을 쓸 수 있습니다. 이 함수는 콜백을 호출하거나 promise을 반환하는 대신 v파싱된 JSON 값을 반환하거나 예외를 발생시킨다.

```javascript
const fs = require("fs");
function readConfigFileSync(path) {
  let text = fs.readFileSync(path, "utf-8");
  return JSON.parse(text);
}
```

- error-first two-argument 콜백 외에도 Node는 많은 이벤트 기반 비동기 API를 가지고 있다. (보통 streaming data 처리)

  - Node에 내장된 `nonblocking` 함수는 운영 체제 버전의 콜백 및 이벤트 핸들러를 사용하여 작동한다.
  - 이러한 함수 중 하나를 호출하면 node가 작업을 시작하기 위한 행위를 한한 다음 작업이 완료되면 알림을 받도록 이벤트 핸들러를 운영 체제에 등록한다. 
  - node 함수에 전달한 콜백은 내부적으로 저장되므로 운영 체제가 해당 이벤트를 node에 전송할 때 node가 콜백을 호출할 수 있다. 

- 이러한 종류의 동시성을 이벤트 기반 동시성이라고 한다. 

  - 이벤트 기반 동시성의 핵심으로 "이벤트 루프"를 실행하는 단일 스레드를 node가 가지고 있다.

  - Node 프로그램이 시작되면 실행하라고 지시한 모든 코드를 실행한다. 

  - 이 코드는 적어도 하나의 `nonblocking` 함수를 호출하여 콜백 또는 이벤트 핸들러가 운영 체제에 등록되도록 한다. 
    (그렇지 않으면 동기적 노드 프로그램을 작성한 것이되어, 노드가 코드 끝에 도달하면 종료하게 된다 )

  - 노드가 프로그램 끝에 도달하면 이벤트가 발생할 때까지 block하고, 이벤트가 발생하면 OS에서 실행을 다시 시작한다. 

  - 노드는 OS 이벤트를 등록한 JavaScript 콜백에 매핑한 다음 해당 함수를 호출한다.

  - 콜백 함수가 더 많은 nonblocking 노드 함수를 호출하면, 더 많은 OS 이벤트 핸들러가 등록될 수 있다. 콜백 함수가 완료되면 노드는 다시 절전 모드로 전환되고 이 사이클이 반복된다.

    