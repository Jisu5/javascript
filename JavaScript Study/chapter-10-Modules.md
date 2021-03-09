# Chapter 10. Modules
* 코드를 모듈화하는 목적은 다음과 같습니다. 처음에는 서로 다른 사람들이 다양한 목적을 가지고 코드를 썼다할지라도, 그 코드들을 용도에 맞게 조합할 수 있도록 하여, 큰 프로그램을 다른 코드들의 변수나 함수에 겹치지 않게하고, 보다 간결하게 작성할 수 있도록 해줍니다.
* 현실적으로, 모듈화란 캡슐화, Private한 Field의 구현을 숨기는 행위 등을 말합니다.
* 자바스크립트는 최근까지 모듈화에 관한 기능을 가지고 있지 않았습니다. 그래서 사람들은 클래스, 오브젝트, 클로저를 통해서 이를 구현하려고 하였습니다.
* 클로저를 바탕으로한 모듈화는 Node에서 채택한 `require()`로 이어지게 되었고, `require()`는 Node 프로그래밍에서 중요한 한자리를 잡게 되었습니다.
* ES6에서는 `import`와 `export` 키워드를 사용해서 모듈을 정의합니다. `import`와 `export`는 꾀 오랫동안 자바스크립트의 한 부분이였지만 Web에서만 주로 사용되었고, Node에서는 상대적으로 최근부터 사용되어지기 시작했습니다.
* 현실적으로 Javascript의 모듈화는 많은 부분 code-bundling 도구에 의존하고 있습니다.



## 10.1 Modules with Classes, Objects, and Closures
* 클래스의 가장 중요한 기능 중 하나는, 클래스가 가진 매서드를 모듈화 한다는 점입니다. 두 클래스가 같은 이름을 가진 메서드를 가진다고 해도, 클래스가 가진 특성때문에 우리는 두 메서드를 겹쳐 사용하지 않고 서로 독립적으로 사용할 수 있습니다.
* 자바스크립트에서 클래스와 오브젝트를 가지고 모듈화를 하는 것은 유용한 테크닉이지만 충분하지는 않습니다. 특히 클래스를 통한 모듈화는 내부 구현을 완벽히 숨길수가 없습니다.
* 하지만 Chapter8에서 확인했듯이, 지역변수와 함수안의 정의된 중첩함수는 모듈화의 Private한 특성과 잘 맞는 부분이 있습니다. 이를 통해서 우리는 이런 모듈화를 어느정도 이룰 수 있습니다. 
    ```
    const stats = (function() {
        const sum = (x,y) => x+y
        const square = x => x*x

        function mean(data) {
            return data.reduce(sum) / data.length
        }

        function stddev(data) {
            let m = mean(data)
            return Math.sqrt(
                data.map(x => x - m).map(square).reduce(sum)/(data.length - 1)
            )
        }

        return { mean , stddev }
    }())

    stats.mean([1, 3, 5, 7, 9])
    stats.stddev([1, 3, 5, 7, 9])
    ```

## 10.2 Modules in Node
* Node programming에서 각각의 파일은 자기 자신만의 네임스페이스를 지닌 독립적인 모듈입니다. 한 파일에서 정의된 상수, 변수, 함수, 그리고 클래스는  `export` 하지 않는 한 다른 파일에서 접근할 수 없습니다. 

### 10.2.1 Node Exports
* Node는 global object인 `exports`를 미리 정의 하고 있습니다. 여러 값을 `export`하는 노드 모듈을 작성하려고 한다면 `exports` object의 프로퍼티에 값을 할당함으로써 값을 `export`할 수 있습니다.
    ```
    const sum = (x, y) => x + y;
    const square = x => x * x;

    exports.mean = data => data.reduce(sum)/data.length;
    exports.stddev = function(d){
        let m = exports.mean(d)
        return Math.sqrt(d.map(x => x - m).map(square).reduce(sum)/(d.length-1))
    }
    ```
* 함수로 가득 찬 오브젝트나 클래스를 `export`하기 보다는 그냥 단지 하나의 함수 또는 클래스를 `export`하려고 한다면, `export`하려고 하는 하나의 값을 `module.exports`에 할당하면 됩니다.
    ```
    module.exports = class BitSet extends AbstractWritableSet {
        ...
    }
    ```
* `module.exports`의 default value는 `exports`의 오브젝트 입니다. 그래서 위의 `mean`함수와 `stddev`함수를 다음과 같이 `export`할 수 있습니다.
    ```
    const sum = (x, y) => x + y
    const square = x => x * x

    const mean = data => data.reduce(sum)/data.length
    const stddev = d => {
        let m = mean(d)
        return Math.sqrt(d.map(x => x - m).map(square).reduce(sum)/(d.length - 1))
    }

    module.exports = {mean, stddev};
    ```

### 10.2.2 Node Imports
* `require()`를 호출함으로써 다른 module을 `import`할 수 있습니다. `require()`의 인자로 import되는 어떠한 모듈이든 올수가 있고, exports된 함수, 클래스, 오브젝트 등 어느 값이든 반환될 수 있습니다.
* Node안에 있는 여러가지 모듈들을 import 하고 싶다면 `/`없이 인자에 적음으로써 import할 수 있습니다.
    ```
    const fs = require("fs");
    const http = require("http");

    const express = require("express");
    ```
* 자신이 쓴 모듈을 import하고 싶다면 파일이 위치한 path를 인자에 넣어줍니다. 
    ```
    const stats = require('./stats.js')
    const BitSet = require('./utils/bitset.js')
    ```
* 다수의 프로퍼티를 가진 오브젝트가 export되었을 때, 전체 object를 import를 할 수 도 있고, 하나의 프로퍼티만 import할수도 있습니다.
    ```
    const stats = require('./stats.js')

    let average = stats.mean(data)

    const {stddev} = require('./stats.js')

    let sd = stddev(data)
    ```

## 10.3 Modules in ES6
* ES6부터 `import`와 `export`가 Javascript에 추가되었습니다.  ES6의 모듈화는 Node의 모듈화와 개념적으로 같습니다. 각각의 파일은 자기만의 모듈, 상수, 변수, 함수, 클래스를 지니고 있어 다른 모듈과는 독립적인 관계를 뛰고 있습니다.
* ES6의 모듈은 다음과 같은 점에서 일반적인 자바스크립트와 차이가 있습니다.
    * 일반적인 스크립트에서 가장 높은 단계에서의 변수 선언, 함수 클래스는 모든 스크립트에서 공유되는 global context를 가집니다. 하지만 ES6의 모듈에서는 각각의 파일이 private한 context를 가지고 있습니다. 그리고 `import`와 `export`를 통해서 서로 공유할 수 있습니다.
    * ES6 모듈안에서 코드는 자동으로 strict mode입니다. 즉, ES6 module을 시작할 때 `with`문이나 `arguments` object, 선언되지 않은 변수를 사용할 수 없습니다.
### 10.3.1 ES6 Exports
* ES6모듈에서 class, function, variable, constant를 `export`하기 위해서 선언 앞에 `export`키워드를 더해주면 됩니다.
    ```
    export const PI = Math.PI
    export function degreesToRadians(d) { return d * PI / 180 }
    export class Circle {
        constructor(r) { this.r = r}
        area() { return PI * this.r * this.r }
    }
    ```
* 또는 다음과 같이 한줄로 간결하게 쓸 수도 있습니다.
    ```
    export { Circle, degreesToRadians, PI }
    ```
* 1개의 값을 가진 모듈을 다음과 같이 `export` 할 수 있습니다.
    ```
    export default class BitSet {
        ...
    }
    ```
    * `export default`가 일반적인 `non-default export`보다 조금 import하기 쉽습니다. 그래서 export할 값이 하나만 존재한다면 `default export`를 사용하는것이 좋습니다.

* 일반적인 `export`는 반드시 이름을 가진 선언 앞에 사용되어져야 합니다. `export default`는 이름 없는 함수 expression이나 class expression에 사용되어질 수 있습니다. 즉, `export`와 달리 export default앞에 curly brace가 존재한다면 이것은 `object literal`을 export하는 것입니다.

* `export`는 class, function, loop, 조건문안에서 사용되어질 수 없습니다.


### 10.3.2 ES6 Imports
* 다른 모듈에서 export된 값을 `import` keyword를 통해서 import할 수 있습니다.
    ```
    import BitSet from './bitset.js'
    ```
* import된 값이 할당되는 identifier는 상수입니다. `const` keyword를 통해서 선언된 것과 비슷합니다.
* `export`처럼 `import` 역시 모듈의 가장 높은 level에 존재하여, class, function, loop, 조건문 안에서 사용되어질 수 없습니다. 하지만 function 선언문과 비슷하게, import는 'hoisted'되어집니다. 
* 현재 폴더에 있는 모듈을 import할 때 'util.js'는 되지 않습니다. 반드시 './util.js'와 같이 사용되어져야 합니다.
* 여러 값을 `export`한 module을 `import`할때는 curly brace를 사용합니다.
    ```
    import { mean, stddev } from './stats.js' 
    ```
* 많은 값을 `export`하는 module을 import할 때 다음과 같이 쓸 수 있습니다.
    ```
    import * as stats from "./stats.js"
    ```
    * 이렇게 사용된 import는 object를 생성하고 stats이라는 이름을 가진 상수에 할당합니다. 이렇게 import된 값들은 stats object의 프로퍼티가 됩니다. `Non-default` export들은 항상 이름을 가지고 있는데, 이렇게 정의된 이름은 오브젝트안의 프로퍼티 이름으로 사용되어집니다.
* `export`와 `export default`를 함께 import할때는 다음과 같이 쓸 수 있습니다.
    ```
    import Histogram, {mean, stddev} from "./histogram-stats.js"
    ```
### 10.3.3 Imports and Exports with Renaming
* 2개의 모듈이 2개의 다른 값을 똑같은 이름으로 export하고, 2개의 값을 모두 import하려고 한다면 rename을 한 후 import하여야 합니다. 비슷하게, 현재 모듈에서 사용되어지고 있는 이름을 import하려고 한다면 이 역시도 rename해야 합니다.
* 이 때 다음과 같이 `as` keyword를 사용해서 rename할 수 있습니다.
    ```
    import { render as renderImage } from './imageutils.js'
    import { render as renderUI } from './ui.js'
    ```
* `default export`와 `named export`를 함께 import할 때 다음과 같은 방법으로도 import할 수 있습니다.
    ```
    import { default as Histogram, mean, stddev } from "./histogram-stats.js"
    ```

### 10.3.4 Re-Exports
* 우리는 구현이 서로 다른곳에서 정의된 file들을 export하고 싶을 때가 있습니다. 이럴 때는 이렇게 할 수도 있습니다.
    ```
    import { mean } from './stats/mean.js';
    import { stddev } from './stats/stddev.js'
    export { mean , stddev }
    ```
    * 하지만 ES6는 이런 상황이 있을 거라고 예측하고, 이를 대처하기 위한 syntax를 가지고 있습니다.
    다음과 같이 export를 from과 사용하여 re-export를 더 간결하게 사용할 수 있습니다.
        ```
        export { mean } from './stats/mean.js'
        export { stddev } from './stats/stddev.js'
        ```
* 
