
<img width="300" height="60" alt="image" src="https://github.com/user-attachments/assets/a28523ac-efb8-4eb1-9b2a-bc372633f678" />

> 타임리프는 서버 측(Server-Side)에서 실행되는 자바 템플릿 엔진(Java Template Engine)입니다.  
> 주요 목적은 서버에서 관리되는 동적 데이터를 HTML 문서에 결합하여, 최종적인 정적 HTML을 생성(렌더링)하는 것입니다.

<br>

### **1. Natural Templating**
타임리프 템플릿 파일은 그 자체로 완전한 HTML 문서 구조를 유지합니다.

**동작 원리**
- 타임리프는 기존 HTML 태그에 `th:` 접두사가 붙은 네임스페이스 속성을 사용하여 동적 로직을 적용
- 웹 브라우저는 표준 HTML 명세에 따라 자신이 이해하지 못하는 속성(예: `th:text`)을 무시하고 기존 HTML 태그와 속성만 렌더링
- 이 원리 때문에 서버의 개입 없이도 원본 템플릿 파일의 정적인 디자인을 확인 가능

<br>
 
> ✅ **JSP와의 차이점**  
> : JSP는 `<%...%>`, `<c:...>` 와 같이 HTML 표준이 아닌 문법을 사용하므로, 서버의 렌더링 없이는 웹 브라우저가 정상적인 HTML 구조로 해석할 수 없습니다.

<br>

### **2. Server-Side Rendering**


<br>
<br>

### **3. Spring Integration**

<br>
<br>


 












<br>

**참고 자료**
- [공식 사이트](https://www.thymeleaf.org/)  
- [공식 메뉴얼 - 기본 기능](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html)  
- [공식 메뉴얼 - 스프링 통합](https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html)  
