
<img width="300" height="60" alt="image" src="https://github.com/user-attachments/assets/a28523ac-efb8-4eb1-9b2a-bc372633f678" />

> 타임리프는 서버 측(Server-Side)에서 실행되는 자바 템플릿 엔진(Java Template Engine)이다.  
> 주요 목적은 서버에서 관리되는 동적 데이터를 HTML 문서에 결합하여, 최종적인 정적 HTML을 생성(렌더링)하는 것이다.

<br>

### **1. Natural Templating**
> 타임리프 템플릿 파일은 그 자체로 완전한 HTML 문서 구조를 유지한다.

<br>

**동작 원리**
- 타임리프는 기존 HTML 태그에 `th:` 접두사가 붙은 네임스페이스 속성을 사용하여 동적 로직을 적용한다.
- 웹 브라우저는 표준 HTML 명세에 따라 자신이 이해하지 못하는 속성(예: `th:text`)을 무시하고 기존 HTML 태그와 속성만 렌더링한다.
- 이 원리 때문에 서버의 개입 없이도 원본 템플릿 파일의 정적인 디자인을 확인할 수 있다.

<br>
 
> 💡 **JSP와의 차이점**  
> JSP는 `<%...%>`, `<c:...>` 와 같이 HTML 표준이 아닌 문법을 사용하므로, 서버의 렌더링 없이는 웹 브라우저가 정상적인 HTML 구조로 해석할 수 없다.

<br>
<br>

### **2. Server-Side Rendering**  
> 서버 사이드 렌더링은 서버가 데이터까지 모두 적용하여 사용자가 즉시 볼 수 있는 완전한 HTML 페이지를 만들어 클라이언트에게 보내주는 방식이다.

<br>

**처리과정**

**1. 요청(Request):**
   - 사용자가 웹 브라우저에서 특정 페이지 링크를 클릭하거나 주소를 입력하면, 브라우저는 해당 URL의 서버로 페이지를 요청한다.
     
**2. 서버 처리(Server Processing)**
   - 컨트롤러 실행: 서버(예: 스프링 MVC)는 요청에 맞는 컨트롤러를 실행하여 비즈니스 로직을 처리한다.
   - 데이터 조회: 데이터베이스 등에서 화면에 필요한 데이터를 조회하여 Model 객체에 담는다.

**3. 렌더링 (Rendering):**
   - 템플릿 결합: 서버의 템플릿 엔진이 HTML 템플릿 파일과 Model에 담긴 데이터를 결합한다.
   - HTML 생성: 모든 동적 데이터가 실제 값으로 채워지고, 조건문/반복문 등이 처리된 완전한 형태의 순수 HTML 문서가 서버 메모리상에 생성된다.

**4. 응답 (Response):**
   - 서버는 생성된 HTML 문서를 HttpResponse에 담아 클라이언트(웹 브라우저)로 전송한다.

**5. 표시 (Display):**
   - 웹 브라우저는 전달받은 HTML 문서를 즉시 화면에 렌더링(표시)합니다. 사용자는 바로 페이지의 전체 내용을 볼 수 있다.

<br>
<br>
<br>

**참고 자료**
- [공식 사이트](https://www.thymeleaf.org/)  
- [공식 메뉴얼 - 기본 기능](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html)  
- [공식 메뉴얼 - 스프링 통합](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html)  
