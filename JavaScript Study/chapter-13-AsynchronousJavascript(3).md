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


## 13.2.2 Chaining Promises
* Promises를 사용하는 가장 큰 이점은 콜백안에 콜백이 들어가는 형태를 선형적인 체인형태로 쉽게 나타낼 수 있다는 점입니다.
    ```
    fetch(documentURL)                    //HTTP 요청
    .then(response => response.json())    //JSON Body 응답
    .then(document => {                   //JSON이 오게 되면
        return render(document)           //유저에게 보여주기
    })
    .then(rendered => {                   //랜더링된 데이터가 오면
        cacheInDatabase(rendered)         //데이터베이스에 caching
    })
    .catch(error => handle(error))        //Handling Error
    ```
* Chaining Promise가 어떤 순서대로 작동하는지 알아보자면
    ```
    fetch(theURL)           //task 1; returns promise 1
        .then(callback1)    //task 2, returns promise 2
        .tehn(callback2)    //task 3, returns promise 3
    ```
    1. fetch함수가 주어진 URL과 함께 호출되어 HTTP요청을 시작하고 Promise object를 반환합니다.( HTTP Request : task 1, the response : promise 1 )
    2. promise1의 then method를 호출합니다. Promise가 이행되었을 때 실행할 콜백 함수를 인자로 넘깁니다. 콜백 함수를 어딘가에 저장한 후, 새로운 Promise를 리턴합니다. 
    ( Callback1 : task2, the new return : promise2 )
    3. promise2의 then method를 호출합니다. Callback2 함수를 인자로 넘깁니다. 그리고 이 then method는 callback2함수를 기억해놨다가 또 다른 Promise를 리턴합니다. 
    ( Callback2 : task3, the other new return : promise3 )
    4. 위의 1,2,3순서는 동기적으로 일어납니다. 그다음에 1번 순서의 HTTP 요청이 시작되는 동안 비동기적인 중지가 일어납니다.
    5. 결국 HTTP 응답이 도착하게 되면, 비동기 fetch()함수가 HTTP Status와 Header를 Response 오브젝트로 묶어 promise1의 값으로 이행시키게 됩니다.
    6. Promise1이 이행되면(Response Object로), callback1 함수를 호출하여 task2가 실행됩니다. 
    7. task2가 완료되면, JSON object로 promise2를 이행합니다.
    8. 이행된 promise2는 task3의 input이 되어 callback2에게 전달됩니다. task3가 완료되면 promise3가 이행됩니다. 이후 비동기연산의 chain은 끝이납니다.
## 13.2.3 Resolving Promises
* 콜백함수 `c`를 `then()` 매서드의 인자로 보냈을 때, `then()` 매서드는 `c`를 저장하고 promise `p`를 반환합니다
* 이후 차례가 되어, 콜백함수가 연산을 하고 값 `v`를 반환했을 때 `p`는 값 `v`로 resolved되었다고 표현합니다.
* `p`가 Promise가 아닌 `v`로 resolved 되었을 때 `p`는 resolved되었고 `v`로 이행(fulfill)됩니다.
* `p`가 Promise인 `v`로 resolved 되었을 때 `p`는 resolved되었지만 아직 이행(fulfill)되지 않았다고 표현합니다.
* Resolved 상태의 Promise란 어떤 값과 associated되거나, 또 다른 Promise에 얹혀져 있는 상태를 말합니다. 우리는 p가 이행될지 거부될지 모르지만 콜백 c는 더이상 관여하지 않습니다. 
## 13.2.4 More on Promises and Errors
* 동기적인 코드에서, 에러 핸들링 코드를 제외하면 최소한 Exception과 Stack Trace는 볼 수 있습니다. 하지만, 비동기적인 코드에서 예외처리를 하지 않는다면 에러는 아무도 눈치채지 못하게 일어나게 되고 디버깅을 어렵게 만듭니다.
##### The catch and finally methods
```
p.then(null, c)
p.catch(c)
```
* 위의 코드는 똑같이 작동하지만, 2번째 코드가 더 심플하고 `try/catch`의 이름과 비슷해서 더욱 읽기 쉽습니다. 
* 동기적인 코드에서 뭔가 잘못되면, 우리는 Exception이 `catch` block을 만날때까지 콜 스택을 올라간다고 말합니다.
* 비동기적인 코드에서 뭔가 잘못되면, 우리는 `.catch`를 만날때까지 promise chain을 따라 내려간다고 표현합니다.
* 어떤 파일을 여는 코드나 네트워크 연결과 같은 코드를 닫을 때 우리는 `.finally()`를 사용할 수 있습니다. 
```
fetch("/api/user/profile")
.then(response => {
    if (!response.ok) {
        return null;
    }

    let type = response.headers.get("content-type");

    if (type !== "application/json") {
        throw new TypeError("Expected JSON, got ${type}")
    }


    return response.json()
})
.then(profile => {
    if (profile) {
        displayUserProfile(profile);
    } else {
        displayLoggedOutProfilePage();
    }
})
.catch(e => {
    if (e instanceof NetworkError) {
        displayErrorMessage("Check your internet connection.")
    }
    else if (e instanceof TypeError) {
        displayErrorMessage("Something is wrong with our server!")
    }
    else {
        console.error(e)
    }
})
```
* 위의 코드에서 fail할 수 있는 부분은 fetch()의 HTTP요청이 네트워크 불안정 때문에 실패하는 경우 입니다. 그러면 첫 Promise는 NetworkError object를 리턴하면서 Reject됩니다. 그리고 두 번째 .then()매서드에서 아무런 처리가 안되어있으므로 똑같이 NetworkError Object를 리턴하게 되고, 마지막 catch Promise에서도 Reject되어 c3에서 에러핸들링 콜백함수가 불려지게 됩니다.
* 또 fail할 수 있는 부분은 HTTP요청이 404 Not Found나 다른 HTTP error를 냈을 때 입니다. 이들은 유효한 HTTP 응답이기 때문에 `fetch()`호출에서 error로 여기지 않고, 404 Not Found를 Response object로 묶어 반환하게 됩니다. 그리고 이 값은 promise를 이행시켜 callback을 호출하게 만듭니다. 이에 관한 콜백함수에서 null값으로 HTTP error를 처리하도록 되어 있어 이대로 처리가 되고, 이 값이 첫번째 Promise가 이행하게 됩니다. 두번째 `.then()` 매서드는 첫번째에서 이행된 Promise를 받아서 null값을 유저에게 보이게 됩니다.
* 한 가지더 Fail할 수 있는 부분은 HTTP 응답의 Content-type header가 적절하게 오지 않은 경우입니다. 첫 callback함수가 이 부분을 체크하는데, 만약 header가 잘못 들어왔다면 TypeError를 `throw`하고 다음 `.then()`매서드에 Error가 전달되고 2번째 콜백함수가 불려지지 않고 세번째 콜백함수로 전달되어 error를 처리하게 됩니다.

```
startAsyncOperation()
.then(doStageTwo)
.catch(recoverFromStageTwoError)
.then(doStageThree)
.then(doStageFour)
.catch(logStageThreeAndFourErrors)
```
* `catch()`매서드의 콜백함수는, 전 단계에서의 콜백함수가 error를 throw할때 호출됩니다. 만약 콜백함수가 정상적으로 리턴한다면, `.catch()`콜백은 지나가게 되고, 정상적인 값이 다음 `.then()`의 콜백으로 전달되게 됩니다.
* 만약 error가 발생해서 `.catch()` callback으로 전해졌을 때는 Promise chain이 멈추게 되어지고 이 때 `.catch()` callback에서 에러를 처리하여 복원시킬 수도 있습니다. 


## 13.2.5 Promises in Parallel
* 다수의 비동기적인 작업을 같이 실행 할 때 `Promise.all()`을 사용할 수 있습니다. 
* `Promise.all()`은 Promise object로 구성된 array를 인자로 받아 Promise를 리턴합니다. 반환된 Promise는 input Promise중 하나라도 reject하게 되면 reject됩니다. Input Promise들이 전부 이행(fulfill)되면 return promise도 이행(fulfill)됩니다.
```
const urls = [ /* zero or more URLs here]

promises = urls.map(url => fetch(url).then(r => r.text()));

Promise.all(promises)
    .then(bodies => {   /* do sth with the array of strings  })
    .catch( e => console.error(e))
```
* Promise.all()에서 input으로 Promise object뿐만 아니라 Non-Promise object값을 받을 수도 있습니다. 만약 Promise object가 아니라면, 이미 이행(fulfill)된 것처럼 여겨지게 됩니다.


## 13.2.6 Making Promises
##### Promised based on other Promises
* Promise를 리턴하는 다른 함수를 사용하면 Promise를 반환하는 함수를 쉽게 쓸 수 있습니다.
```
fucntion getJSON(url) {
    return fetch(url).then(response => response.json())
}
```
##### Promises based on synchronous values
* 기존 Promised를 기반으로한 API를 구현해서 Promise를 반환하는데, 이 때 연산이 비동기적인 작업은 필요로 하지 않을 때가 있습니다. 이 경우 `Promised.resolve()`, `Promise.rejct()`를 사용할 수 있습니다. 
* `Promise.resolve()`는 하나의 인자를 받아, 이 값으로 이행(fulfill)된 Promise를 반환합니다.
* `Promise.reject()`는 하나의 인자를 받아, 이 값으로 reject된 Promise를 반환합니다.

##### Promises from scratch
* 위의 `GetJson()`의 예처럼 Promise를 반환하는 function을 사용해서 다른 Promise를 반환하는 함수를 쓰고 싶지만, 그렇지 못할때는 `Promise()` 생성자를 사용할 수 있다.
* `Promise()` 생성자를 호출하고 한 함수를 인자로 전달하면 된다. 이 함수는 두 개의 인자를 필요로 하는데 `resolve`와 `reject`가 된다. 
* 생성자는 새롭게 생긴 Promise Object를 생성한다. 이 Promise는 값이 이행(fulfill)될지, reject될지에 따라 resolve(), reject()함수를 호출하게 된다.
```
function wait(duration) {
    return new Promise((resolve, reject) => {
        if (duration < 0) {
            reject(new Error("Time travel not yet implemented))
        }
        setTimeout(resolve, duration)
    })
}
```
