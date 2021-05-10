# 13. Asynchronous Javascript

* 현재 많은 프로그램들은 `Asynchronous` 합니다. 즉, 어떤 연산을 하기 위해서 데이터가 도착하하거나 이벤트가 발생하기를 기다려야합니다.
* 자바스크립트를 사용하는 웹 브라우저의 프로그램들은 주로 `'event-driven'`입니다. 이 프로그램들은 어떤  행동을 실행하기 전까지 유저가 클릭하기를 기다리고 있습니다. 
* 자바스크립트의 서버들은 어떤 행동을 실행하기전까지 Client가 네트워크를 통해 요청을 보내기를 기다리고 있습니다. 
* ES6에서 `Promise` Asynchronous 연산의 아직 사용가능하지 않은 결과를 나타냅니다.
* ES2017에서 `async`, `await` 연산은 `Promise-based` 코드를 마치 synchronous 한것처럼 쓴 것처럼 만들 수 있게 함으로써 asynchrnous 연산을 단순화하도록 했습니다.
* ES2018에서는 asynchrnous iterators와 for/await loop를 연속적인 asynchronous 연산을 loop를 사용하여 synchronous처럼 보일 수 있도록 할 수 있습니다.

## 13.1 Asynchronous Programming with Callbacks
* 자바스크립트에서 Asynchronous programming은 callback으로 끝납니다.
* Callback은 하나의 함수로서, 다른 함수에게 전해집니다. 그리고 전달받은 함수가 어떤 조건이 만족되거나 이벤트가 발생할 때 callback을 실행하게 됩니다. 

## 13.1.1 Timers
* 가장 단순한 asynchrony형태는 어떤 일정한 시간이 지났을 때 코드를 실행하게 하고 싶을 때 입니다.
```
setTimeout(checkForUpdates, 60000);
```
* checkForUpdates()는 콜백함수이고, setTimeout()은 콜백함수와 asynchronous 조건을 인자로 갖게 됩니다.


## 13.1.2 Events
* Client-side 자바스크립트는 대부분 event-driven입니다. 미리 정의해놓은 연산을 실행시키는게 아니라 유저가 클릭하기를 기다리고 유저의 행동에 따라 반응하게 됩니다.
* 웹 브라우저는 유저가 클릭하거나 키보드를 누르고 버튼을 누르고, 마우스를 움직이고, 터치스크린을 터치할 때 이벤트를 만들어냅니다.
* Event-driven 자바스크립트는 콜백함수와 어떤 특정한 이벤트가 어떻게 발생할지에 대해 정의를 해놓았다가, 웹 브라우저는 정의된 이벤트가 발생할 때 콜백함수를 호출하게 됩니다.
* 이러한 콜백함수를 event handlers 또는 event listeners라고 불립니다. 
```
let okay = document.querySelector('#confirmUpdateDialog button.oaky');

okay.addEventListener('click', applyUpdate);
```
* 여기서 `applyUpdate()`는 가상의 콜백함수라고 가정했을 때, document.querySelect()를 호출 했을 때, 웹 페이지 내에서 특정 원소를 나타내는 오브젝트를 리턴하게 됩니다.
* `addEventListener()` 의 첫번째 인자는 string으로써 우리가 관심있는 event의 종류를 나타냅니다. (마우스 클릭, 탭 터치스크린)
* 만약 유저가 웹 페이지의 어느 부분을 클릭하거나 탭하게 되면 브라우저는 콜백함수 applyUpdate()를 호출하여, event와 관련된 오브젝트를 전달하게 됩니다.


## 13.1.3 Network Events
* 자바스크립트에서 다른 형태의 Asynchrony는 네트워크 요청입니다.
* 브라우저에서 실행되고 있는 자바스크립트는 웹 서버에서 데이터를 가져올 수 있습니다.
```
function getCurrentVersionNumber(versionCallback) {
    let request = new XMLHttpRequest();
    request.open("GET", "http://www.example.com/api/version");
    request.send();

    request.onload = function() {
        if(request.status === 200) {
            let currentVersion = parseFloat(request.responseText);
            versionCallback(null, currentVersion);
        } else {
            versionCallback(response.statusText, null);
        }
    }

    request.onerror = request.ontimeout = function(e) {
        versionCallback(e.type, null);
    }
}
```
* 클라이언트 자바스크립트에서 HTTP 요청을 하고, 서버의 response를 비동기적으로 처리하기 위해서 XMLHttpRequest 클래스와 콜백함수를 사용할 수 있습니다.
* `getCurrentVersionNumber()` 함수는 HTTP요청을 보내고 서버에서 response를 받을 때 호출될 event handlers를 정의합니다.
* 위 코드는 전의 예제 코드처럼 `addEventListner()`를 호출하지 않습니다, 
* 대부분의 웹 API들에서 event handlers들은 이벤트와 콜백함수를 만들어내는 오브젝트에 `addEventListener()`를 호출함으로써 정의됩니다.
* 오브젝트의 프로퍼티에 직접 하나의 이벤트 리스너에 할당함으로써 이벤트 리스너를 정의할 수 있습니다. 위의 onload , onerror 그리고 ontimeout과 같은 프로퍼티들이 그렇습니다.
* 보통 이러한 이벤트리스너의 프로퍼티들은 on으로 시작하는 이름들을 가지고 있습니다.
* `addEventListner()`는 여러개의 이벤트 핸들러를 다룰 수 있기 때문에 다양한 상황에서 사용될 수 있습니다. 하지만 단순히 몇개의 이벤트 핸들러를 사용한다면, 위의 예시처럼 프로퍼티를 사용해서 나타낼 수 있습니다. 
* `getCurrentVersionNumber()`는 비동기적 요청을 하기 때문에 호출자가 원하는 값을 동기적으로 받을 수 없습니다. 대신에 호출자는 결과가 준비되거나 에러가 발생했을 때 호출되는 콜백함수를 전달합니다. 위의 예에서 호출자는 두 가지 인자를 받는 콜백함수를 가지고 있는데 만약 `XMLHttpRequest()`가 작동한다면 `getCurrentVersionNumber()`는 `null`이라는 첫 번째인자를 갖고 현재 버전의 숫자를 가진 인자를 갖고 있는 콜백함수를 호출합니다. 만약 에러가 발생한다면, `getCurrentVersionNumber()`는 에러의 상세사항을 나타내느 첫번째인자와 null 두번째인자를 가진 콜백함수를 호출하게 됩니다.


## 13.1.4 Callbacks and Events in Node
* Node.js의 환경은 많은 부분이 비동기적이고 콜백과 이벤트를 사용하는 API들을 정의하고 있습니다. 
* 아래 예는, 한 파일을 읽어내리고 콜백함수를 호출하는 비동기적인 디폴트 API를 나타냅니다.
```
const fs = require("fs");  
let options = {
    // default options would go here
}

//
fs.readFile("config.json", "utf-8", (err, text) => {
    if (err) {
        console.warn("Could not read config file:", err);
    } else {
        Object.assign(options, JSON.parse(text));
    }

    startProgram(options)
})
```
* Node의 fs.readFile() 함수는 두 개의 인자를 가진 콜백함수를 마지막 인자로 가지고 있습니다. `config.json`이라는 파일을 비동기적으로 읽고, 콜백함수를 호출합니다.
* 만약 파일이 성공적으로 읽히면, 파일의 컨텐츠를 콜백함수의 두 번째 인자로 전달하게 됩니다. 
* 만약 에러가 발생하면 콜백함수의 첫번째 인자로 에러를 전달하게 됩니다.
* Node는 이벤트를 기반으로 하는 API도 정의하고 있습니다. 아래 함수는 HTTP 요청으로 어떤 URL의 내용을 확인하는 예입니다. 이벤트리스너를 다루는 두 개의 비동기적인 코드를 가지고 있습니다. 그리고 addEventListner()를 사용하는 대신에 on() method로 이벤트리스너를 등록합니다. 
```
const https = require("https");

function getText(url, callback) {
    // Start an HTTP GET request for the URL
    request = https.get(url);

    // Register a function to handle the "response" event.
    request.on("response", response => {
        let httpStatus = response.statusCode;

        // The body of the HTTP response has not been received yet.
        // So we register more event handlers to be called when it arrives.

        response.setEncoding("utf-8");
        let body = "";

        // This event handler is called when a chunk of the body is ready
        response.on("data", chunk => { body += chunk; })

        // This event handler is called when the response is complete
        response.on("end", () => {
            if (httpStatus === 200) {
                callback(null, body)
            } else {
                callback(httpStatus, null);
            }
        })
    })

    request.on("error", (err) => {
        callback(err, null);
    })
}
```

## 13.2 Promises
* Promise 오브젝트는 비동기적인 연산의 결과를 나타냅니다. 그 결과는 준비가 되어 있을수도, 준비가 되어있지 않을수도 있습니다. 동기적으로 Promise 오브젝트의 값을 가지고 올 수는 없습니다. 값이 준비가 되었을 때, Promise가 콜백함수를 호출하도록 해야만 합니다. 
* 단순하게 말해서 Promise는 콜백함수를 다루는 다른 방법일 뿐입니다. 콜백함수를 사용할 때 콜백안에 콜백함수가 들어가는 경우가 많게 되는데 이는 가독성을 해치게 됩니다. Promise는 선형적으로 콜백함수를 연결함으로써 더욱 가독성을 높입니다.
* 콜백함수의 또 다른 문제점은 에러를 핸들링하게 이럽게 만듭니다. 비동기적인 함수가 Exception을 던질 때, Exception을 다시 비동기적인 연산으로 다시 돌아오게 만들 방법이 없습니다. Promise는 에러를 표준화함으로써 더욱 쉽게 핸들링할 수 있게 만듭니다.
* Promise는 하나의 비동기적인 연산들의 미래 결과를 나타냅니다. 비동기적인 반복된 연산을 나타낼 수 없습니다. setInterval() 함수는 콜백함수를 반복적으로 호출하기 때문에 Promise를 사용할 수 없습니다. 

## 13.2.1 Using Promises
```
getJSON(url).then(jsonData => {
    // This is a callback function that will be asynchronously
    // invoked with the parsed JSON value when it becomes available.
})
```
* 여기서 `getJSON()`은 특정 URL로 HTTP요청을 비동기적으로 보내고, 요청이 진행될 때 동안 Promise 오브젝트를 반환합니다. 
* `getJSON()`에 콜백함수를 직접 전달하지 않고, 콜백함수를 then() 매서드에 전달합니다.
* HTTP Response가 도착했을 때, response는 JSON으로 파싱되고, 파싱된 결과값은 우리가 then() 매서드에 정의한 함수로 전달되게 됩니다.
* then() 매서드를, 이벤트 핸들러를 등록하는데 사용했던 addEventListner()매서드라고 생각하면 더 쉽습니다. 
* 다른 이벤트리스너들과 다른점이 있다면, Promise는 단 하나의 연산만을 나타냅니다. 그리고 `then()`에 등록된 각각의 함수들은 단 한번만 호출됩니다. `then()` 을 호출할 때 비동기적인 연산이 이미 완료되었다고 하더라도,  `then()`에 등록된 함수들은 비동기적으로 호출됩니다. 
* Promise를 반환하고 Promise의 결과를 사용하는 함수를 동사로 정의하는 것이 관습입니다. 이러한 방법은 코드의 가독성을 높입니다.
```
// Suppose you have a function like this to display a user profile
function displayUserProfile(profile) { /*  implementation omitted */  }

getJSON("/api/user/priofile").then(displayUserProfile)
```

##### Handling errors with Promises
* 네트워크와 관련된 부분에서 많은 경우 요청이나 리스폰스가 실패하곤 합니다. 이런 경우 에러를 핸들링 할 수 있는 코드를 쓸 수 있어야 합니다.
* 우리는 then() 매서드에 두 번째 함수를 전달함으로써 에러를 핸들링 할 수 있습니다.
```
getJSON("/api/user/profile").then(displayUserProfile, handleProfileError);
```
* Promise 오브젝트가 우리에게 반환된 후에 연산이 실행되기 때문에 값을 반환하거나 Exception을 던질방법이 없습니다. 이 때 우리는 then()의 다른 대안을 사용합니다. 
* 동기적인 연산이 완료되었을 때 연산은 호출자에게 결과를 반환합니다. Promise에 기반한 비동기적인 연산이 완료되었을 때, 연산은 결과를 then()의 첫번째 인자에 정의된 함수에게 전달합니다.
* 비동기적인 연산이 정상적으로 완료되지 못했을 때, 예외를 던지고 `catch` 절을 만날 때까지 콜스택을 따라 올라갑니다. 비동기적인 연산이 실행될 때 호출자는 더 이상 스택에 존재하지 않습니다. 그래서 어떤 연산이 잘못될 때, Exception을 던져 호출자로 돌아갈 수 없습니다.
* Promise에 기반한 비동기적인 연산은 예외를 then() 매서드의 두 번째 함수에 전달합니다. 그래서 getJSON()이 정상적으로 실행한다면 결과를 displayUserProfile()에 전달하고, 에러가 발생한다면 Error오브젝트를 handleProfileError()에 전달하게 됩니다.
* 사실 then() 매서드에 두 가지 함수를 인자로 잘 사용하지 않는데, 그 이유는 `getJSON()`이후에 `displayUserProfile()`에서 error가 생겼을 때 예외를 처리할 수가 없기 때문입니다. 
```
getJSON("/api/user/profile").then(displayUserProfile).catch(handleProfileError)
```
* 이 경우, 정상적으로 작동했을 때 `getJSON()`은 결과를 `displayUserProfile()`에 전달합니다.  `getJSON()` 또는 `displayUserProfile()`에 에러가 발생했을 때 `handleProfileError()`가 예외를 처리하게 됩니다. 
