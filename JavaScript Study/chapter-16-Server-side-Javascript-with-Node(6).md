# Chapter 16. Server side Javascript with Node (6)

## 16.11 Worker Threads
* 노드의 concurrency 모델은 단일 스레드와 이벤트 기반으로 이루어져 있습니다. 노드 10이후부터 노드는 멀티 스레드를 지원합니다. 
* 멀티스레드 프로그래밍은 스레드들을 통해 메모리를 공유하기 때문에 이해하기 어렵기로 유명하지만, 자바스크립트의 스레드들은 자동적으로 메모리를 공유하지 않습니다. 그래서 멀티스레드 프로그래밍이 어렵다는 것은 자바스크립트의 Worker들에게는 해당하지 않습니다.
* 공유된 메모리를 사용하기 보다는, 자바스크립트의 Worker Threads들은 메시지를 주고받습니다. 
    * 메인 스레드는 postMessage()를 호출해서 워커 스레드에 메시지를 전송할 수 있습니다.
    * 워커 스레드는 "message" 이벤트를 부모로부터 listening함으로써 메시지를 받습니다.
    * 워커 스레드는 워커스레드만의  postMessage()를 호출해서 메인 스레드에 메시지를 보낼 수 있습니다.
    * 부모 스레드는 "message"이벤트 핸들러를 통해 메시지를 받습니다.
* Node application에서 워커 스레드를 사용해야하는 3가지 이유가 있습니다.
    1. 어플리케이션이 1개의 CPU 코어가 다룰 수 있는 것보다 많은 연산이 필요할 때 
    2. 어플리케이션이 1개의 CPU를 완전히 다 사용하지 않더라도, 메인스레드가 빠른 반응을 유지하기 위해 사용.
    3. 워커 스레드를 사용하면, Blocking 동기 연산을 Non-blocking 비동기 연산으로 만들 수 있다.
        * 어쩔 수 없이, 동기적인 legacy 코드를 사용할 때, 워커들을 사용하여 legacy 코드의 blocking을 피할 수 있다. 

## 16.11.1 Creating Workers and Passing Messages
```
const threads = require("worker_threads");
```
* `workter_threads` 모듈은 워커 스레드를 나타내는 워커 클래스를 정의하고, `threads.Worker()` 생성자를 통해 새로운 스레드를 만들 수 있습니다. 

```
const threads = require("worker_threads");

// The worker_threads module은 boolean인 isMainThread 프로퍼티를 export합니다.
// 이 프로퍼티는 노드가 메인 스레드를 통해 실행될 때 true이고, 워커 스레드를 통해 노드가 /// 실행될 때 false입니다.

if(threads.isMainThread) {
    // 메인스레드에서 실행되면 함수를 EXPORT합니다.
    // 메인스레드에서 많은 연산이 필요로 할 때, 이 함수는 태스크를 워커 스레드에게 전달하고 워커 스레드가 작업이 끝났을 때 RESOLVE되는 프라미스를 반환합니다.

    module.exports = function reticulateSplines(splines) {
        return new Promise((resolve, reject) => {
            // 스레드 생성, 로드 및 같은 코드를 실행
            let reticulator = new threads.Worker(__filename);

            // spline array의 카피를 워커 스레드에게 전달
            reticulator.postMessage(splines);

            // 워커 스레드로부터 메시지 또는 에러를 받을 때 resolve 또는 reject를 한다.
            reticulator.on("message", resolve);
            reticulator.on("error", reject);
        })
    }
} else {
    // 이 블록이 실행된다는 것은 워커스레드에서 실행됨을 의미합니다.
    // 메인 스레드로부터 메시지를 받기 위해 핸들러를 등록합니다.
    // 워커스레드는 단 하나의 메시지만 받기로 디자인되었기 때문에, once() 매서드를 사용하여 이벤트 핸들러를 등록합니다.
    // 이를 통해 작업이 끝나면 워커스레드를 자연스럽게 종료시킬 수 있습니다.

    for (let spline of splines) {
        spline.reticulate ? spline.reticulate() : spline.reticulated = true;
    }

    // 모든 spline을 돌았을 때 메인 스레드에 카피를 보냅니다.
    threads.parentPort.postMessage(splines)
}
```
* `Worker()` 생성자의 첫 번째 인자는 자바스크립트 코드 파일의 PATH입니다.
* `Worker()` 생성자는 두 번째 인자로 오브젝트를 받을 수 있는데, 이 오브젝트의 프로퍼티들은 워커 스레드의 설정 옵션들을 나타냅니다.
* 노드는 postMessage()를 통해 오브젝트의 카피를 전달합니다. 이를 통해 워커 스레드와 메인스레드는 메모리를 공유하지 않아도 됩니다. 


## 16.11.2 The Worker Execution Enivronment
* 많은 부분, 노드의 워커스레드에서 돌아가는 자바스크립트 코드들은 메인스레드에서 돌아가는것처럼 돌아갑니다.
* 아까 살펴봤던 `Worker()`생성자의 2번째 인자를 비롯해 몇 가지 차이점이 있는데, 다음과 같습니다. 
    * `threads.isMainThread`는 메인스레드에서 `true`, 워커스레드에서 `false`입니다.
    * 워커스레드에서, `threads.paraentPort.postMessage()`를 통해 부모 스레드에 메시지를 전송할 수 있습니다. 부모 스레드에서 오는 메시지를 받기 위해 `threads.parentPort`.on을 통해 이벤트 핸들러를 등록할 수 있습니다. 메인스레드에서 `threads.parentPort`는 항상 `null`입니다.
    * 워커스레드에서, threads.workerData는 워커 생성자의 두 번째 인자에 대한 카피입니다. 메인스레드에서 이 프로퍼티는 항상 `null`입니다.
    * 워커스레드에서 `process.env`는 부모스레드 `process.env`의 복사본입니다. 부모 스레드는 `env`프로퍼티를 `Worker()` 생성자의 2번째 인자로 전달하여 환경 변수를 지정할 수 있습니다. 
    * 워커 스레드가 process.exit()를 호출하면, 그 스레드만 종료됩니다.
    * 워커 스레드는 그들이 속한 프로세스의 상태를 변화시킬 수 없습니다. 그래서 `process.chdir()`나 `process.setuid()`가 워커 스레드에서 호출되면 예외가 throw됩니다.
    * SIGINT나 SIGTERM같은 OS signal은 메인스레드에만 전달되고, 워커스레드는 받지 못합니다.

## 16.11.3 Communication Channels and MessagePorts
* 새로운 스레드가 생성될 때, 워커스레드와 메인스레드가 메시지를 주고 받을 수 있는 Communication Channel이 만들어집니다.
    * 워커 스레드는 부모스레드와 `threads.parentPort`를 통해 메시지를 주고 받습니다.
    * 부모 스레드는 워커 오브젝트를 통해 워커스레드와 메시지를 주고 받습니다.
```
cosnt threads = require("worker_threads");
let channel = new threads.MessageChannel();
channel.port2.on("message", console.log);     // 받는 메시지를 기록
channel.port1.postMessage("hello");           // "hello를 프린트"
```
* `MessageChannel()` 생성자를 통해 새로운 메시지 채널을 만들 수 있습니다. 메시지 채널 오브젝트는 두 프로퍼티를 갖고 있는데, `port1`과 `port2`입니다. 
* 두 포트는 MessagePort 오브젝트의 한 쌍을 나타냅니다.
* 한 포트에서`postMessage()`를 호출하는 것은 "message"이벤트가 `Message`오브젝트의 복사본이 반대편에 생성됨을 뜻합니다.
* close()를 통해 포트의 커넥션을 끊을 수 있습니다. 둘 중 하나의 매서드에서 호출되면 두 포트다 close 이벤트가 호출됩니다.

## 16.11.4 Transfering MessagePorts and Typed Arrays
* `postMessage()` 매서드는 structred clone 알고리즘을 사용하는데 `SSockets`나 `Streams` 오브젝트를 카피할 수는 없습니다.
* `postMessage()` 매서드는 두 번째 인자로 transferList를 받는데, 스레드 간에 카피되기보다는 전송되는 오브젝트의 어레이를 뜻합니다.
* MessagePort 오브젝트는 structured clone 알고리즘에 의해 복사되어질 수 없는데, 전송될 수 는 있습니다. 
    * 만약 `postMessage()`가 하나 이상의 MessagePort를 첫 번째 인자로 포함한다면, 이 MessagePort 오브젝트들은 두 번째 인자로 어레이의 멤버로 있어야 합니다.
    * 이것으로 Node에게 MessagePort를 카피하지 말고 기존의 오브젝트를 반대편 스레드에 전달하라고 말할 수 있습니다.
    * 한 가지 기억할 점은, 스레드간 value들을 전송한다는 것은, value가 전달된 후 더이상 스레드에서 postMessage()를 호출할 수 없습니다.
```
const threads = require("worker_threads");
let channel = new threads.MessageChannel();

// 워커의 디폴트 채널을 사용해서 반대편 워커에 있는 채널로 전송하기.
// 워커가 이 메시지를 받자마자, 새로운 채널의 메시지를 listen하기 시작한다.
worker.postMessage({command:"changeChannel", data: channel.port1}, [channel.port1])

// 사용자 채널을 통해 워커에게 새로운 메시지를 전송
channel.port2.postMessage("Can you here me now?")

// 워커로부터 response를 Listen
channel.port2.on("message", handleMessagesFromWorker);
```

* messagePort 오브젝트 뿐만 아니라 typed array도 message로 전송될 수 있습니다.
```
let pixels = new Uint32Array(1024*1024);

worker.postMessage(pixels, [pixels.buffer]);
```
    * MessagePort처럼, typed array 역시 한번 전송되고나면 사용될 수 없습니다. 
    * Message Port나 typed array를 사용하려고 하면 예외가 throw되지 않습니다.

## 16.11.5 Sharing Typed Arrays Between Threads
* typed array를 스레드간 전송을 할 수 있지만, 스레드간 공유를 할 수도 있습니다.
* typed array가 `SharedArrayBuffer`위에 생성하고 `postMessage()`를 통해 전달되면 스레드간에 메모리에서 공유될 수 있습니다.
    * 이 때 shared buffer를 postMessage()의 2번째 인자로 사용해서는 안됩니다.
* 하지만, 공유하는 것은 정말로 사용하지 않는 것이 좋은데, 자바스크립트의 멀티스레드는 공유하는 것을 목표로 디자인하지 않았기 때문입니다. 
* 