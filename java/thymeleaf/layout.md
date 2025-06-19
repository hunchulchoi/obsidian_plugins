Thymeleaf Layout은 웹 애플리케이션에서 공통된 레이아웃 요소(Header, Footer, Navigation 등)를 효율적으로 관리하기 위한 템플릿 구조화 방법입니다. 이를 통해 코드의 재사용성을 높이고 유지보수를 용이하게 할 수 있습니다.

## **기본 설정**

### **의존성 추가**

```xml
<!-- Maven -->
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>
```

```gradle
// Gradle
implementation 'nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect'
```

### **폴더 구조**

표준적인 Thymeleaf Layout 폴더 구조는 다음과 같습니다[1][2]:

```
templates/
├── common/
│   ├── fragment/          # 재사용 가능한 조각들
│   │   ├── config.html    # 공통 CSS, JS
│   │   ├── header.html    # 헤더 영역
│   │   └── footer.html    # 푸터 영역
│   └── layout/
│       └── defaultLayout.html  # 전체 레이아웃 틀
└── pages/                 # 실제 콘텐츠 페이지들
    └── test/
        └── test.html
```

## **Fragment 구성**

### **Header Fragment**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<header th:fragment="headerFragment">
    <div>
        <h1>사이트 헤더</h1>
        <nav>
            <!-- 네비게이션 메뉴 -->
        </nav>
    </div>
</header>
</html>
```

### **Footer Fragment**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<footer th:fragment="footerFragment">
    <div>
        <p>&copy; 2025 회사명. All rights reserved.</p>
    </div>
</footer>
</html>
```

### **Config Fragment**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<th:block th:fragment="configFragment">
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- CSS -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
    <link rel="stylesheet" type="text/css" th:href="@{/css/common.css}">
    
    <!-- JavaScript -->
    <script src="https://code.jquery.com/jquery-2.2.4.min.js"></script>
    <script th:src="@{/js/common.js}" defer></script>
</th:block>
</html>
```

## **Layout 템플릿**

### **기본 레이아웃 구성**

```html
<!DOCTYPE html>
<html lang="ko" 
      xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
<head>
    <title>기본 레이아웃</title>
    <!-- 공통 설정 불러오기 -->
    <th:block th:replace="common/fragment/config :: configFragment"></th:block>
    
    <!-- 페이지별 CSS -->
    <th:block layout:fragment="css"></th:block>
</head>
<body>
    <div class="main-container">
        <!-- 헤더 영역 -->
        <th:block th:replace="common/fragment/header :: headerFragment"></th:block>
        
        <!-- 메인 콘텐츠 영역 -->
        <main>
            <th:block layout:fragment="content"></th:block>
        </main>
        
        <!-- 푸터 영역 -->
        <th:block th:replace="common/fragment/footer :: footerFragment"></th:block>
    </div>
    
    <!-- 페이지별 JavaScript -->
    <th:block layout:fragment="script"></th:block>
</body>
</html>
```

## **실제 페이지 구현**

### **콘텐츠 페이지**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" 
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorate="~{common/layout/defaultLayout}">

<!-- 페이지별 CSS -->
<th:block layout:fragment="css">
    <link rel="stylesheet" th:href="@{/css/test.css}">
</th:block>

<!-- 페이지별 JavaScript -->
<th:block layout:fragment="script">
    <script th:src="@{/js/test.js}"></script>
</th:block>

<!-- 메인 콘텐츠 -->
<div layout:fragment="content">
    <div class="container">
        <h2>테스트 페이지</h2>
        <p th:text="${message}">기본 메시지</p>
        
        <form th:action="@{/test}" method="post">
            <input type="text" name="username" placeholder="사용자명">
            <button type="submit">제출</button>
        </form>
    </div>
</div>
</html>
```

## **핵심 문법**

### **Fragment 관련 속성**

- **`th:fragment="이름"`**: Fragment를 정의합니다[5]
- **`th:replace="경로::이름"`**: Fragment를 완전히 교체합니다[2][5]
- **`th:insert="경로::이름"`**: Fragment를 삽입합니다[5]

### **Layout 관련 속성**

- **`layout:decorate="~{레이아웃경로}"`**: 사용할 레이아웃을 지정합니다[2][5]
- **`layout:fragment="이름"`**: 레이아웃에서 교체될 영역을 정의합니다[2][5]

## **Controller 구성**

```java
@Controller
public class LayoutController {
    
    @GetMapping("/test")
    public String testPage(Model model) {
        model.addAttribute("message", "레이아웃 테스트 페이지입니다.");
        return "pages/test/test";  // templates/pages/test/test.html
    }
    
    @PostMapping("/test")
    public String processTest(@RequestParam String username, Model model) {
        model.addAttribute("message", "안녕하세요, " + username + "님!");
        return "pages/test/test";
    }
}
```

## **장점과 활용**

### **주요 장점**

- **코드 재사용성**: 공통 요소를 한 번 정의하여 여러 페이지에서 사용[1]
- **유지보수 용이성**: 공통 부분 수정 시 한 곳만 변경하면 전체 적용[1]
- **일관된 디자인**: 모든 페이지에서 동일한 레이아웃 구조 유지[2]
- **개발 효율성**: 페이지별로 콘텐츠 영역만 집중하여 개발 가능[3]

### **실무 활용 팁**

- **조건부 Fragment**: `th:if`를 사용하여 상황에 따라 다른 Fragment 적용 가능
- **매개변수 전달**: Fragment에 매개변수를 전달하여 동적 콘텐츠 구성 가능
- **중첩 Layout**: 복잡한 구조에서는 Layout을 중첩하여 사용 가능

이러한 Thymeleaf Layout 구조를 통해 효율적이고 유지보수가 용이한 웹 애플리케이션을 구축할 수 있습니다.

Sources
[1] [Java] Thymeleaf, Thymeleaf Layout 적용하기 - Contributor9 - 티스토리 https://adjh54.tistory.com/75
[2] [Thymeleaf] Thymeleaf template layout으로 초기 세팅하기2 https://yulfsong.tistory.com/98
[3] [spring] 2. spring boot + Thymeleaf 템플릿 구조 및 사용 - 개발은 개 발로 https://lims-dev.tistory.com/32
[4] Spring Boot Thymeleaf Layout Dialect 사용하기 - latemorning https://latemorning.github.io/2019-04-12/Thymeleaf-Layout-Dialect/
[5] thymeleaft-layout-dialect 설정 및 적용 - kiki - 티스토리 https://kiki-100.tistory.com/217
[6] [Thymeleaf] 템플릿 조각 및 템플릿 레이아웃 - velog https://velog.io/@coo9292/Thymeleaf-%ED%85%9C%ED%94%8C%EB%A6%BF-%EC%A1%B0%EA%B0%81-%EB%B0%8F-%ED%85%9C%ED%94%8C%EB%A6%BF-%EB%A0%88%EC%9D%B4%EC%95%84%EC%9B%83
[7] [Spring]Thymeleaf(타임리프) 기본 사용법 정리 - 네이버블로그 https://blog.naver.com/hj_kim97/223031615864
[8] Thymeleaf를 이용한 웹 개발 소개 및 기능 - 춍춍 블로그 https://chung-develop.tistory.com/144
[9] [Thymeleaf + Spring] 8. Template Layout - 개발을개발로할태니 https://delightpip.tistory.com/56
[10] Thymeleaf를 이용해서 레이아웃적용하기(feat.layout-dialect) - YouTube https://www.youtube.com/watch?v=sbbyX-tbP5Q
