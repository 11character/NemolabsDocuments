<span id="main"></span>
# Nemoupload Servlet

- java의 Servlet을 사용합니다.

- chunk 단위로 올라오는 파일을 하나로 합쳐서 저장합니다.

<span id="ready"></span>
## 준비하기

- java 1.6 이상의 환경이 필요합니다.

- 해당 소스는 Apache Tomcat 8 을 사용하여 테스트 되었습니다.

- Maven pom.xml 파일에 다음 의존성을 추가합니다. (Maven을 사용하지 않으면 해당 라이브러리를 직접 추가하세요)  
    ```
    [xml]
    ...
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.0-b01</version>
    </dependency>
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.2.2</version>
    </dependency>
    <dependency>
        <groupId>com.googlecode.json-simple</groupId>
        <artifactId>json-simple</artifactId>
        <version>1.1</version>
    </dependency>
    ...
    ```

- nemoupload.jar 파일을 라이브러리에 추가합니다. (/WEB-INF/lib/)  
    추가한 다음에 web.xml에 Servlet 설정을 합니다.  
    ```
    [xml]
    ...
    <servlet>
        <servlet-name>upload</servlet-name>
        <servlet-class>kr.co.nemolabs.nemoupload.NemouploadServlet</servlet-class>
        <init-param>
            <param-name>dirPath</param-name>
            <param-value>/uploadTempFile</param-value>
        </init-param>
        <init-param>
            <param-name>forwardURL</param-name>
            <param-value>/forward</param-value>
        </init-param>
        <init-param>
            <param-name>byteBuffer</param-name>
            <param-value>1048576</param-value>
        </init-param>
        <init-param>
            <param-name>maxFileSize</param-name>
            <param-value>5242880</param-value>
        </init-param>
        <init-param>
            <param-name>maxRequestSize</param-name>
            <param-value>1048576</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>upload</servlet-name>
        <url-pattern>/upload</url-pattern>
    </servlet-mapping>
    ...
    ```

    ### Prameter

    - #### dirPath `(필수)`

        저장되는 위치. (해당 경로는 미리 만들어져 있어야 합니다)

    ---------------------------------------------------------------------------------
    - #### forwardURL `(필수)`

        파일을 저장하면 요청을 넘겨주는 경로.

    ---------------------------------------------------------------------------------

    - #### byteBuffer `(생략가능)`

        파일을 저장하면서 사용할 stream buffer의 크기.  
        기본값은 1048576(1MB) 입니다.

    ---------------------------------------------------------------------------------

    - #### maxFileSize `(생략가능)`

        파일의 최대 크기.  
        단일 파일의 최대 크기를 체크합니다.  
        chunk단위로 올라오는 파일을 이어쓰면서 해당 파일의 크기를 체크합니다.  
        단위는 **byte** 입니다.  
        초과시 쓰여진 파일을 지우고 오류코드 413 을 반환합니다.
        기본값은 무제한 입니다.

    ---------------------------------------------------------------------------------

    - #### maxRequestSize `(생략가능)`

        요청의 최대 크기.  
        Request의 크기를 체크합니다. (담겨진 데이터의 크기보다 큽니다.)  
        단위는 **byte** 입니다.  
        파일을 쓰기전에 크기를 체크합니다.  
        초과시 오류코드 413 을 반환합니다.  
        chunk단위로 올리는 경우 chunk가 하나의 요청으로 처리됩니다.
        기본값은 무제한 입니다.

    ---------------------------------------------------------------------------------

<span id="result"></span>
## 처리결과

- 업로드된 파일의 정보는 `HttpServletRequest.getAttribute("files")`로 받을 수 있습니다.  
    값의 타입은 Map<String, String> 배열 입니다.  
    ```
    [java]
    Map[] files = (Map[]) request.getAttribute("files");
    
    files[0].get("chunkCount");
    files[0].get("chunkNumber");
    files[0].get("fieldName");
    files[0].get("dirPath");
    files[0].get("fileID");
    files[0].get("filePath");
    files[0].get("fileName");
    ```
    - chunkCount: 조각의 개수 입니다.
    - chunkNumber: 1 부터 시작하는 현재 조각의 번호 입니다.
    - fieldName: 파일 들어있던 필드의 이름 입니다.
    - dirPath: 저장된 디렉터리 경로 입니다.
    - fileID: 저장시 사용되는 이름 입니다.
    - filePath: 저장된 경로 입니다. (dirPath + fileID)
    - fileName: 해당 파일의 실제 이름입니다.

<span id="attention"></span>
## 주의사항

- `POST` 방식의 `multipart/form-data` 전송일 경우만 처리가 가능합니다.

- 파일의 용량은 사용자 화면에서 한 번 체크하고 요청을 하도록 설정하는 것이 좋습니다. 

- 전달받은 저장 경로를 가지고 후처리를 통하여 파일을 분류, 관리하는 것이 좋습니다.

- chunk 를 사용하는 경우, 저장소에 찌꺼기 파일이 남을 수 있습니다. (전송하는 도중에 페이지를 강제로 닫는경우)