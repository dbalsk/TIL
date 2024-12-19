# **서버와 비동기로 상호작용 (AJAX)**
> JSON 데이터 형식과 AJAX 요청에 대해서 이해하고, JSON을 주고받는 AJAX 코드로 즐겨찾기(북마크) 서비스를 구현해보자.   
이를 통하여 동기/비동기/블로킹/논블로킹에 대해서도 이해해보자.
  
## JSON (JavaScript Object Notation)
**데이터를 저장하거나 전송하기 위한 데이터 형식** 중 하나이다. (자바스크립트, 웹, 도큐먼트 방식의 데이터베이스 등 여러 곳에서 표준처럼 사용)   
자바스크립트에서 **객체(Object)를 표기**하는 방법으로 사용된다. (JSON이 자바스크립트의 객체인 것은 아니다.)

**✔ JSON의 형태**  

    {
        "name": "홍길동",
        "age": 22
    }    

데이터 쌍의 왼쪽은 **키**(key), 오른쪽은 **값**(value)이고 각 데이터 쌍은 쉼표(,)로 구분된다.   
키에는 문자열만 올 수 있기에 큰따옴표가 필수이며 값은 문자열, 정수, 실수, 배열, 다른 JSON 데이터, null, true/false가 가능하다.

>+JSON Formatter    
>JSON을 정해진 형식에 맞게 정렬하거나 실제로 맞게 작성되었는지 검증. (주로 웹페이지에서 사용)

**✔ JSON vs XML**   
* **데이터 크기**: JSON이 XML보다 작아 데이터 전송에 소요되는 시간/비용이 절약된다.
* **데이터 파싱(구문분석)**: JSON이 자바스크립트 엔진을 통해 더 빠르게 수행된다. (서버와 서버 간 통신에서는 XML이 유리할 수도 있다.)
* **추가적인 정보**: XML이 메타데이터를 함께 전송하거나, 속성을 추가할 수 있기에 편리하다.

</br>

## AJAX (Asynchronous JavaScript And XML)
자바스크립트를 통해 서버에 비동기로 요청하는 것을 의미한다. (명칭에 XML이 있으나, JSON을 사용해도 무관)

>웹 페이지를 새로고침하지 않고 서버로 정보를 보내거나 서버의 정보를 가져올 때 비동기적인 방식이 사용된다. 

</br>

## 즐겨찾기(북마크) 서비스
>리스트를 활용한 웹 브라우저의 **즐겨찾기(북마크)** 서비스를 JSON을 주고받는 AJAX 코드로 작성하여 구현해보자.    

**✔ 서비스 구상**  
* 즐겨찾기는 이름과 URL로 구성한다. 
* 즐겨찾기 등록 / 조회 기능을 구현한다.
* 등록 / 조회 기능은 AJAX로 동작하여 새로고침하지 않도록 구현한다.

**✔ 구현**
1. **JSON** 정의
    ```
        {"name": "구글", "URL": "http://www.google.com/"}
    ```
2.  Bookmark 자바 클래스 구현
    ```ruby
    public class Bookmark {
        public String name;
        public String url;
    } 
    ```
>+JSON의 키와 이름이 같은 경우 서로 매핑(대응)됨.

3. 컨트롤러에 **즐겨찾기를 등록하는 API**와 **즐겨찾기 목록을 조회하는 API**를 추가.
    * 리스트 선언 및 할당   
        ```ruby
            private List<Bookmark> bookmarks = new ArrayList<>();
        ```    
    * 즐쳐찾기 등록 API
        - HTTP 메소드: POST 메소드 
        - API 경로: /bookmark
        ```ruby
            @RequestMapping(method = RequestMethod.POST, path="/bookmark")
            public String registerBookmark(@RequestBody Bookmark bookmark){
                bookmarks.add(bookmark);
                return "registered";
            }
        ```
    * 즐쳐찾기 목록 조회 API
        - HTTP 메소드: GET 메소드 
        - API 경로: /bookmarks (이해를 위해 복수형으로 구분)
        ```ruby
            @RequestMapping(method = RequestMethod.GET, path="/bookmarks")
            public List<Bookmark> getBookmarks(){
                return bookmarks;
            }
        ```
4. 프런트엔드 페이지 작성 (버튼구현)
    *  즐겨찾기 등록 버튼 구현 (**form 태그**)
        - 입력 박스: input 태그 "text" 타입 ("name"타입은 "name"과 "url"로) 
        - 제출 버튼: input 태그 "submit" 타입
        - onsubmit 이벤트: 제출 버튼 클릭 시, 즐겨찾기 등록 요청 함수 호출

    * 즐겨찾기 목룍 조회 버튼 구현
        - 조회 버튼: button 태그 
        - 출력: ol 태그 (순서태그)에 추가되도록 구현.
<br>

5. 프런트엔드 페이지 작성 (자바스크립트 작성) 
    * 즐겨찾기 등록 요청 함수 
        - **XHR 객체** 생성 
        - **이벤트 핸들러** 함수 선언 및 등록 
        - **open()** 함수: XHR 객체 초기화(준비)
        - **send()** 함수: XHR 객체가 AJAX 요청(POST) 전송 (응답과는 무관. -> 비동기)
        - AJAX 요청 시 readyState의 상태가 바뀌며 이벤트 핸들러가 실행되고, **DONE(요청과 응답이 모두 완료된 상태)** 일 경우 HTTP 상태 코드가 **200(요청이 성공)** 인지 체크하여 맞다면 로그 출력.   
                  
    * 즐겨찾기 목록 조회 요청 함수
        - 즐겨찾기 등록 요청 함수와 유사 (POST요청이 아닌 GET 요청)
        - 요청이 성공(200)일 경우 forEach() 메서드로 배열(즐겨찾기 목록)을 순회하며 HTML 태그 추가.

>+이벤트 핸들러   
>이벤트 발생 시, 호출되는 함수. (위 코드에서는 XHR 객체의 readyState가 변경되는 것을 이벤트로 설정)

>+JSON.stringify()  
>자바스크립트 객체를 JSON 문자열로 변환해주는 함수. (JSON.parse는 반대로 수행.)

>+직렬화/역질렬화   
>(추후 추가 예정)


<br>

## 참고자료
[이것이 백엔드 개발이다](https://product.kyobobook.co.kr/detail/S000211834105)   



