<span id="main"></span>
# Nemouoload Nodejs

- Nodejs express 서버에서 간편하게 사용할 수 있습니다.

- chunk 단위로 올라오는 파일을 하나로 합쳐서 저장합니다.

<span id="ready"></span>
## 준비하기

-  Nodejs express 웹 서버가 있어야 합니다.

- 파일을 준비합니다. (원격 저장소를 지원하지 않습니다.)  
    파일은 app.js 와 package.json 으로 간단하게 구성되어 있습니다.

    ```
    nemoupload_module
        ├ app.js
        └ package.json
    ```

- npm 을 사용하여 모듈을 설치합니다.  
    ` install ` 명령어 뒤에 모듈이 들어있는 디렉토리 경로를 입력하면 local 에서 설치가 됩니다.

    ```
    [js]
    npm install [디렉토리 경로]
    ```

<span id="start"></span>
## 시작하기

- 소스에 미들웨어로 사용할 모듈을 불러옵니다.

    ```
    [js]
    var nemoupload = require('nemoupload');
    ```

- ` .use() ` 를 사용하여 미들웨어를 추가합니다. 추가할 때는 파일이 저장될 임시 경로를 인자로 넣어줍니다.

    ```
    [js]
    app.use(nemoupload('./fileDirPath'));
    ```

<span id="result"></span>
## 처리결과

- nemoupload 미들웨어는 파일이 임시 저장소에 다 올라오면, 파일 정보를 ` request.files ` 배열에 넣습니다.

    - fieldname: 파일 들어있던 필드의 이름 입니다.

    - fileName: 파일의 실제 이름 입니다. (저장된 데이터는 임의의 이름을 가지게 됩니다.)

    - path: 저장된 경로 입니다.

- 파일과 같이 넘어온 prameter 는 ` request.body ` 에 들어가게 됩니다.

<span id="attention"></span>
## 주의사항

- 미들웨어의 사용 위치에 주의해야 합니다. ` route ` 설정 이전에 추가되어야 합니다.

- ` POST ` 방식의 ` multipart/form-data ` 의 경우만 미들웨어가 동작합니다.

- 전달받은 저장 경로를 가지고 후처리를 통하여 파일을 분류, 관리하는 것이 좋습니다.

- chunk 를 사용하는 경우, 임시저장소에 찌꺼기 파일이 남을 수 있습니다.