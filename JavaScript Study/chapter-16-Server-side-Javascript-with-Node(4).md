## 16.6  Process, CPU, and Operating System Details

* 전역 Process 오브젝트는 실행되고 있는 노드의 상태에 접근하고 이에 관한 유용한 프로퍼티와 함수를 가지고 있습니다.
    ```
    process.argv                    // An array of command-line arguments.
    process.cwd()                   // Returns the current working directory.
    process.env                     // An object of environment variables.
    process.exit()                  // Terminates the program
    process.kill()                  // Send a signal to another process.
    ```
* "os" module은 process 오브젝트와 달리 `require()`를 통해 불러와야하고 컴퓨터의 저수준 단계를 조정할 수 있습니다. 많이 사용하는 부분은 아니지만 몇몇 함수들은 다음과 같습니다.
    ```
    const os = require("os");
    os.cpus()                       // Data about system CPU cores, including usage times.
    os.homedir()                    // Returns the current user's home directory.
    ...
    ```
## 16.7 Working with Files.
* "fs" 모듈은 파일과 디렉토리에 관한 작업을 할 수 있는 API들을 정의하고 있습니다.
* 노드의 "fs" 모듈은 파일을 읽고, 쓰고 복사하는 등의 소수의 고수준 함수를 가지고 있습니다. 
* 하지만 "fs" 모듈의 대부분은 Unix 시스템과 관련된 저수준 함수 입니다.
    * 예를 들어 파일을 삭제할 때 `unlink()`라는 함수를 호출합니다. 
* "fs"모듈은 파일과 폴더에 관한 작업을 할 때 그 작업마다 다양한 형태의 variant를 정의하고 있습니다. 
    * `fs.readFile`()은 nonblocking, callback-based, asynchronous한 읽기 variant입니다.
    * `fs.readFileSync()`는 blocking, synchronous한 읽기 variant입니다.
    * `fs.promises.readFile()`은 promised-based, asynchronous한 읽기 variant입니다.
* "fs"모듈의 path를 나타내는 스트링을 인자로 받는 variant 함수, file descriptor를 나타내는 integer를 인자로 받는 variant 함수들이 있습니다.
    * `fs.truncate()`는 파일의 path를 나타내는 스트링을 인자로 받습니다.
    * `fs.ftruncate()`는 file descriptor를 나타내는 integer를 인자로 받습니다.
### 16.7.1 Paths, File Descriptors, and FileHandles.
* "fs"모듈을 사용하기 위해 파일의 위치를 나타내는 path를 정의할 수 있어야 합니다.
* path가 "absolute"라면 filesystem의 root까지 모두 명시되어야 합니다.
* path가 "relative"라면 다른 path들과 함께 정의되어야 하고 서로 관련성이 있습니다. 예를 들어 current working directory.
* 주요 path 모듈이 정의하는 것들은 다음과 같습니다.
    ```
    process.cwd()                  //absolute path of the current working directory.
    __filename()                   //absolute path of the file that holds current code.
    __dirname()                    //absolute path of the directory that holds __filename.
    os.homedir()                   // The user's hoem directory.

    const path = require("path");

    path.sep                      // Either "/" or "\" depending on your OS

    // The path module has simple parsing functions.
    let p = "src/pkg/test.js";
    path.basename(p)               // => "test.js"
    path.extname(p)                // => ".js"
    path.dirname(p)                // -> "src/pkg"
    

    // normalize() cleans up paths:
    path.normalize("a/b/c/../d/")  // => "a/b/d/"
    path.normalize("a/./b")        // => "a/b"
    path.normalize("//a//b//")     // => "/a/b/"

    
    // join() combines path segments, adding, separators(), then normalizes
    path.join("src", "pkg", "t.js") // => "src/pkg/t.js"


    // resolve() takes one or more path segments and returns an absolute
    // path. It starts with the last argument and works backward, stopping
    // when it has built an absolute path or resolving against process.cwd()
    path.resolve()                      // => process.cwd()
    path.resolve("t.js")                // => path.join(process.cwd(), "t.js")
    path.resolve("/tmp", "t.js")        // => "/tmp/t.js"
    path.resolve("/a", "/b", "t.js")    // => "/b/t.js"

    ```
    * `path.normalize()`는 스트링을 통해서 path를 리턴할 뿐, 실제 filesystem에는 영향을 미치지 않습니다.
    * `fs.realpath(), fs.realpathSync()`는 filesystem에 영향을 미치는 path를 받아 작업을 할 수 있습니다. relative path를 cwd로 해석할 수 있습니다.
    * `path.posix`를 사용해서 OS가 Window인 환경에서 Unix-style의 path를 정의할 수 있습니다.
    * `path.win32`를 사용해서 OS가 Unix인 환경에서 Windows-style의 path를 정의할 수 잇습니다.

* File Descriptor는 파일을 열기 위해, OS-level의 reference로 사용되는 정수입니다. 
* File Descriptor는 `fs.open()`, `fs.openSync()`함수를 호출해서 얻을 수 있습니다.
* Process들은 한번에 정해진 만큼의 파일들을 열어야 합니다. 그래서 `fs.close()`를 통해서 작업이 끝났다면 File descriptor를 닫는게 중요합니다.

### 16.7.2 Reading Files
* 노드는 파일을 *한번에 읽기*, *스트림을 통해 읽기*, *저수준 API읽기*를 지원합니다.
* 파일이 작거나 메모리나 퍼포먼스가 우선순위가 아니라면 한번의 호출로 한번에 전체 파일을 읽을 수 있습니다.
    * 한 번에 파일 읽기는 콜백, 프라미스, 동기적으로 등 다양한 방법으로 가능합니다.
    * 한 번에 파일 읽기는 default로 버퍼형태의 byte로 받습니다. 다른 형태로 encoding또한 가능합니다.
    ```
    const fs = require("fs");
    let buffer = fs.readFileSync("test.data")           // Synchronous, return buffer
    let text = fs.readFileSync("data.csv", "utf8")      // Synchronous, return string

    fs.readFile("test.data", (err, buffer) => {
        if (err) {
            // Handle the error here
        } else {
            // The bytes of the file are in buffer
        }
    })

    // Promised-based asynchronous read
    fs.promises
        .readFile("data.csv", "utf8")
        .then(processFileText)
        .catch(""handleReadError);

    ```
* 파일의 전부를 메모리에 저장하지 않고 또는 파일을 한꺼번에 읽지않고, 순차적으로 읽어내고 싶다면 stream형태로 읽을 수 있습니다.
    ```
        function printFile(filename, encoding="utf8"){
            fs.createReadStream(filename, encoding).pipe(process.stdout)
        }
    ```
* 마지막으로, 저수준 API로 파일을 읽을 수 있습니다. 파일을 읽을 때 몇 bytes까지 읽을지를 정하여 읽을 수 있습니다.
    * 파일을 열어(`fs.open()`, `fs.openSync()`, `fs.promises.open()`) file descriptor를 얻어낸 후 fs.read(), fs.readSync(), fs.promises.read()로 읽어낼 수 있습니다.
    ```
        fs.open("data", (err, fd) => {
            if (err) {
                return ;
            }
            try {
                fs.read(fd, Buffer.alloc(400), 0, 400, 20, (err, n, b) => {
                    // err is the error, if any.
                    // n is the number of bytes actually read
                    // b is the buffer that they bytes were read into.
                })
            }
            finally {
                fs.close(fd)
            }
        })
    ```

### 16.7.3 Writing Files
* 노드에서 쓰기는 읽기와 매우 비슷합니다. 다른 점은 쓸 파일의 filename 명시할 뿐입니다.
* 파일을 한번에 쓸 때는 `fs.writeFile()`, `fs.writeFileSync()`, `fs.promises.writeFile()`을 사용합니다.
    ```
    fs.writeFileSync(path.resolve(__dirname, "settings.json"),
                                    JSON.stringify(settings));
    ```
* 파일을 스트림 형태로 쓸 때는 다음과 같이 쓸 수 있습니다.
    ```
    const fs = require("fs");
    let output = fs.createWriteStream("numbers.txt");
    for(let i=0; i<100; i++) {
        output.write(`${i}\n`);
    }
    output.end();
    ```
* 파일을 저수준 API로 쓸 수 있습니다. 파일을 쓸 때 몇 bytes까지 쓸지 정하여 쓸 수 있습니다.
    * 파일을 열어(`fs.open()`, `fs.openSync()`, `fs.promises.open()`) file descriptor를 얻어낸 후 fs.write(), fs.writeSync()를 사용할 수 있습니다.
    * 쓸 파일의 타입이 string, buffer에 따라 다양한 쓰기 variant를 사용할 수 있습니다.
* 파일을 쓸 때 다음과 같은 flag를 표시해야합니다.
    ```
    fs.writeFileSync("message.log", "hello", {flag : "a"});

    fs.createWriteStream("messages.log", { flags : "wx"});
    ```
    * "w" : Open the file for writing
    * "w+" : Open for writing and reading
    * "wx" : Open for creating a new file; fails if the named file already exists.
    * "wx+" : Open for creation, and also allow reading; fails if the named file already exists.
    * "a" : Open the file for appending; existing content won't be overwritten
    * "a+" : Open for appending, but also allow reading.

* 노드에서 파일을 쓸때 주의할 점은 다음과 같습니다.
    * 위에서 설명한 파일을 쓰는 다양한 함수들은 데이터가 쓰여질 때(즉 다시 말해, Node가 OS에 파일을 넘겨줄 때) `return` 하거나, 전달받은 callback함수를 호출하거나, 전달받은 promise를 resolve했습니다.
    * 하지만 이 때 반드시 데이터가 디스크에 쓰여진 것은 아닙니다. 특히 데이터가 OS 어딘가의 버퍼에 아직 남아 있을 수 있습니다. 
        * fs.writeSync()를 사용해서 동기적으로 파일을 쓸 때 만약 정전이 된다면 데이터를 잃을 수 있습니다.
        * 이 때 데이터가 확실하게 저장되었는지 확인할 수 있는 방법은, `fs.fsync()`, `fs.fsyncSync()`와 같은 함수를 사용하는 것 입니다. 이 함수들은 file descriptor로만 작동하고 path-based version이 아닙니다.
### 16.7.4 File Operations
* 파일을 복사할 때 `fs.copyFile()`, `fs.copyFileSync()`, `fs.promises.copyFile()`을 사용할 수 있습니다.
* 이 함수들은 복사할 파일, 복사될 파일이름 2개 인자를 받습니다. 이 인자들은 string, URL, Buffer Object가 될 수 있습니다. 세 번째 함수는 optional로 복사의 세부사항 flag를 정의합니다.
```
// Basic synchronous file copy.
fs.copyFileSync("ch15.txt", "ch15.bak");


// The COPYFILE_EXCL argument copies only if the new file does not already exist.
// It prevents copies from overwrting existing files.
fs.copyFile("ch15.txt", "ch16.txt", fs.constants.COPYFILE_EXCL, err => {
    // This callback will be called when done. On error, err will be non-null
})

fs.promises.copyFile("Important data", "Important data ${new Date().toISOString()}"
                    fs.constants.COPYFILE_EXCL | fs.constants.COPYFILE_FICLONE)
                    .then(() => {
                        console.log("Backup complete");
                    });
                    .catch(() => {
                        console.error("Backup failed", err);
                    })
```
* 파일의 이름을 다시 짓거나 이동시킬 때 `fs.rename()`함수를 사용할 수 있습니다. flag 인자는 없습니다.
    ```
    fs.renameSync("ch15.bak", "backups/ch15.bak")
    ```
    * flag가 없으므로 기존 파일에 덮어쓰여질 수 있음에 주의해야 합니다.
* 파일을 삭제할 때, `fs.unlink()`함수를 사용할 수 있습니다. string, buffer, URL path을 인자로 넣어 삭제할 수 있습니다.

### 16.7.5 File Metadata
* 파일의 metadata를 얻어야 할 때 `fs.stat()`, `fs.statSync()`, `fs.promises.stat()` 함수를 사용할 수 있습니다.
    ```
    const fs = require("fs");
    let stats = fs.statSync("book/ch15.md");
    stats.isFile()                              // => true: this is an ordinary file
    stats.isDirectory()                         // => false
    stats.size                                  // file size in bytes
    stats.atime                                 // access time : Date when it was last read
    stats.mtime                                 // modification time: Date when it was last written
    stats.uid                                   // the user id of the file's owner
    stats.gid                                   // the group id of the file's owner
    stats.mode.toString(8)                      // => the file's permission, as an octal string
    ```
* 파일이 file descrptor나 FileHandleObject같은 열려있는 파일이라면, `fs.fstat()`을 사용할 수 있습니다.
* `fs.chmod()`, `fs.lchmod()`, `fs.fchmod()`를 사용하여 permission 설정을 할 수 있습니다.
    ```
    fs.chmodSync("ch15.md", 0o400)              // read-only to its owner
    ```
### 16.7.6 Working with Directories
* 새로운 폴더를 만들 때 `fs.mkdir()`를 사용할 수 있습니다.
    ```
    fs.mkdirSync("dist/lib", {recursive : true})
    ```
* 폴더를 삭제할 때 `fs.rmdir()`를 사용할 수 있습니다.
    ```
    let tempDirPath;
    try {
        tempDirPath = fs.mkdtempSync(path.join(os.tmpdir(), "d"));
    } finally {
        // Delete the temporary directory when we're done with it
        fs.rmdirSync(tempDirPath);
    }
    ```
* 폴더 안에 있는 파일 또는 디렉터리들을 나열할 때 `fs.readdir()`를 사용할 수 있습니다.
    ```
    let tempFiles = fs.readdirSync("/tmp");

    fs.promises.readdir("/tmp", {withFileTypes: true})
        .then(entries => {
            entries.filter(entry => entry.isDirectory())
                .map(entry => entry.name)
                .forEach(name => console.log(path.join("/tmp/", name)));
        })
        .catch(console.error);
    ```
* 폴더 안에 있는 파일 또는 디렉터리가 수천개라면 `fs.opendir()`라는 스트림 형태를 사용할 수 있습니다.
* Dir object를 사용하는 가장 쉬운 방법은 for/await loop와 async iterator를 함께 사용하는 방법입니다. straming API를 사용하여 디렉토리를 나열하고, stat()을 호출하여 파일의 이름과 용량을 출력합니다.
    ```
    const fs = require("fs");
    const path = require("path");

    async function listDirectory(dirpath) {
        let dir = await fs.promises.opendir(dirpath);
        for await (let entry of dir) {
            let name = entry.name;
            if (entry.isDirectory()){
                name += "/";
            }
            let stats = await fs.promises.stat(path.join(dirpath, name));
            let size = stats.size;
            console.log(String(size).padStart(10), name)
        }
    }
    ```