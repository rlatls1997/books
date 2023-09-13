# 7.1 HTTP 세션 이용
스프링 MVC에서 javax.servlet.http.HttpSession 객체를 이용하는 방법 설명

여러 화면에 걸쳐서 입력한 데이터나 온라인 쇼핑몰의 장바구니처럼 담아둔 상품 데이터를 취급하는 등 여러 요청이 같은 데이터를 공유할 때 사용한다.

HTTP 세션에 데이터를 저장하면 새로 열리는 창이나 탭에도 같은 HTTP 세션을 사용하기 때문에 탭이나 창마다 서로 다른 데이터를 필요로 하는 경우에는 사용할 수 없다.

애플리케이션이나 시스템의 요구사항을 고려해서 HTTP 세션의 사용 여부를 결정해야 한다.

## 세션에서 데이터를 관리하는 방법

스프링 MVC는 컨트롤러의 핸들러 메서드 매개변수로 HttpSession 객체를 받을 수 있지만  가능한 HttpSession API를 직접 사용하지 않는 방법(스프링 MVC의 서블릿 API의 추상화 구조)을 이용하자.

### 세션 속성(@SessionAttributes) 이용

스프링 MVC의 Model에 추가한 객체를 HttpSession API를 직접 사용하지 않고 HTTP 세션에서 관리.

@org.springframework.web.bind.annotation.SessionAttributes는 하나의 컨트롤러에서 여러 요청 간에 데이터를 공유하는 ㄱㅇ우 효과적인 방법.

폼 객체를 세션에 저장하면 애플리케이션의 설계나 구현이 단순해질 수 있다. 입력화면, 확인화면, 완료화면을 별도의 페이지로 구성되는 단순한 흐름이라면 HTTP 세션 대신 HTML폼의 hidden요소에 데이터를 가지고 다니는 방법 고려

- **세션에 관리할 객체를 지정하는 방법**

@SessionAttributes에 Http 세션에서 관리할 대상 객체를 지정한다. 관리 대상 객체를 지정하는 방법은 두 가지가 있음

클래스명 지정 : 관리대상이 되는 클래스명을 types 속성에 지정한다.

```java
@Controller
@RequestMapping("/accounts")
// @ModelAttribute 애너테이션이 붙은 메서드나 Model의 addAttribute 메서드를 통해 Model에 
// 추가한 객체 중에 types 속성에서 지정한 클래스의 객체가 있다면 그 객체를 HTTP 세션에 저장
@SessionAttributes(types = AccountCreateForm.class)
public class AccountCreateController{
	..
}
```

속성명 지정 : 관리 대상이 되는 객체명을 names 속성에 지정한다.

```java
@Controller
@RequestMapping("/accounts")
@SessionAttributes(names = "accountCreateDto")
public class AccountCreateController{
	..
}
```

- **세션에 객체를 저장하고 저장된 객체를 이용하는 방법**

@SessionAttributes를 이용해서 특정 객체를 HTTP 세션 안에 관리하고 싶다면 그 객체를 Model에 저장한다. 이후 Model에 저장한 객체 중에서 HTTP 세션에 관리하겠다고 지정한 객체만 HTTP 세션에 저장(export)되고, 반대로 HTTP 세션에서 관리되는 객체 중 HTTP 세션에 관리하겠다고 지정한 객체가 Model에 저장(import) 된다.

Model이 객체를 저장하는 방법에는 @ModelAttribute 메서드를 이용하는 방법과 Model API를 이용하는 방법이 있다.

객체를 HTTP 세션에 저장하는 예

```java
@Controller
@RequestMapping("/accounts")
@SessionAttributes(types = AccountCreateDto.class)
public class AccountCreateController{
	
	// 스프링 MVC의 명명 규칙에 따라 AccountCreateForm의 객체를 accountCreateForm 이름으로 세션에 저장
	@ModelAttribute("accountCreateForm")
	public AccountCreateForm setUpAccountCreateForm(){
		return new AccountCreateForm();
	}
}
```

Model에서 객체를 꺼내올 때의 구현 예

```java
@RequestMapping(path = "create", method = RequestMethod.POST)
// Model 에서 객체를 받기 위한 인수를 선언한다. 
// 인수에는 클래스명 첫글자를 소문자로 바꾼 이름과 같은 이름의 객체가 설정됨
// accountCreateForm 이라는 이름의 객체가 인수에 설정된다.
public String create(@Validated AccountCreateForm from, 
		BindingResult result,
// Model에서 가져올 객체의 이름을 @ModelAttribute의 value 속성에 지정할 수 있다.
// 해당 객체가 존재하지 않을 경우에는 HttpSessionRequiredException 발생.
		@ModelAttribute("password") String password,
		RedirectAttributes redirectAttributes){
		...
		return "redirect:/accounts/create?complete";
}
```

- **세션에 저장된 객체 삭제**

HTTP 세션에 저장된 객체를 삭제할 때는 setComplete 메서드 호출.

```java
@RequestMapping(path = "create", params = "complete", method = RequestMethod.GET)
public String createComplete(SessionStatus sessionStatus){
	sessionStatus.setComplete();

	return "accoun/createComplete"
}
```

- **뷰에서 세션에 접근하는 방법**

JSP를 뷰로 사용할 때는 EL에서 ${속성명}으로 지정하면 됨.

뷰(JSP)에서 접근하는 예

```java
이메일 주소 : <c:out value="${accountCreateForm.email"/>
```

### 세션 스코프 빈 이용

HTTP 세션에 관리하고 싶은 객체를 DI 컨테이너에 세션 스코프 빈으로 등록해서 HttpSession API 를 직접 사용하지 않고 HTTP 세션에서 관리한다.

### HttpSession API 이용

HttpSession API(setAttribute, getAttribute, removeAttribute 등)를 직접 사용해서 대상 객체를 HTTP 세션에서 관리한다.


# 7.2 파일 업로드
스프링 MVC에서 파일 업로드를 할 때는 다음 방법 중 하나를 사용한다.

- 서블릿 표준 업로드 기능
  서블릿 3.0에서 지원하는 파일 업로드 기능과 스프링 웹에서 제공하는 컴포넌트를 이용해서 파일을 업로드한다. 서블릿 버전이 3.0 이상인 애플리케이션 서버를 사용할 수 있을 때 사용한다.
- Apache Commons FileUpload 업로드 기능
  파일 업로드용 라이브러리인 Apache Commons FileUpload와 스프링 웹에서 제공하는 컴포넌트를 이용해서 파일을 업로드한다. 이 방법은 서블리 버전이 3.0 미만인 경우나 서블릿 표준 파일 업로드 기능으로는 요청 파라미터나 파일명이 깨지는 경우에 사용한다.

## 파일 업로드 구조

1. 업로드할 파일을 선택하고 업로드를 실행한다.
2. DispatcherServlet은 org.springframework.web.multipart.MultipartResolver 인터페이스의 메서드를 호출해서 멀티파트 요청을 해석한다.
3. MultipartResolver 구현 클래스는 멀티파트 요청을 해석한 후 업로드 데이터를 담을 org.springframework.web.multipart.MultipartFile을 생성한다.
4. DispatcherServlet은 컨트롤로의 처리 메서드를 호출한다. 3번에서 생성한 MultipartFile 객체는 컨트롤러의 인수나 폼 객체에 바인드받는다.
5. 컨트롤러는 MultipartFile 객체의 메서드를 통해 업로드된 파일이나 메타 정보를 가져온다.

스프링이 제공하는 컴포넌트가 서블릿이나  Apache Commons FileUpload의 API를 감춰주기 때문에 내부적으로 서블릿 표준이나 Apache Commons FileUpload 중 어느 쪽을 이용해도 컨트롤러를 같은 방식으로 구현할 수 있다.

## 파일 업로드 기능 설정

서블릿 표준과 스프링이 제공하는 파일 업로드 기능을 이용하려면 파일 업로드와 관련된 설정과 스프링 MVC를 연계하기 위한 설정이 필요하다.

- **파일 업로드 기능을 이용하기 위한 설정**

서블릿 표준 파일 업로드 기능을 사용하는 서블릿<servlet> 요소에 <multipart-config>요소를 추가한다.

web.xml

```xml
<web-app
	xmlns="http....
	...
	>
	...
	<servlet>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<multipart-config />
	</servlet>
	...
</web-app>
```

서블릿 표준의 파일 업로드 기능을 별다른 설정 없이 사용하면 업로드할 수 있는 파일 크기가 무제한이 된다. 파일 크기를 제한하고 싶다면 파일 단위의 최대 크기, 업로드 요청 전체의 최대 크기, 임시 파일을 생성하는 입계치 크기와 같은 세 가지 설정을 추가해야 한다.

web.xml - multipart-config 설정

```xml
<multipart-config>
	<max-file-size>5242880</max-file-size>
	<max-request-size>27262976</max-request-size>
	<file-size-threshold>1048576</file-size-threshold>
</multipart-config>
```

- max-file-size : 파일 하나의 최대 바이트 수를 지정한다. 기본값은 -1(제한 없음) 이다.
- max-request-size : 멀티파트 요청 전체의 최대 바이트 수를 지정한다. 기본값은 -1 이다
- file-size-threshold : 전송된 파일의 크기가 이 크기를 넘어서면 메모리에 있던 파일 내용을 임시 파일로 만든다. 기본값은 0(항상 파일에 출력) 이다.

크기 제한을 설정한 상태에서 크기 제한을 넘으면 org.springframework.web.multipart.MultipartException 이 발생한다. 크기제한 에러를 잡고 싶다면 MultipartException을 catch로 잡으면 된다.

크기 제한을 초과하면 MultipartException이 발생한다고 했지만 스프링 MVC의 DispatcherServlet 이전에 요청 파라미터에 접근하게 되면 MultipartException이 발생하지 않을 수 있다.

예시로 스프링 시큐리티의 CSRF 방지용 서블릿 필터를 적용한 후 톰캣에서 크기 제한을 초과하면 MultipartException이 아닌 CSRF 오류가 발생한다. 이런 현상은 톰캣이 요청 파라미터 값을 null로 처리하기 때문인데 이러한 내부 동작들은 서블릿 API의 사양으로 정해져 있는 것이 아니기 때문에 애플리케이션 서버에 따라 다르게 동작할 수 있다.

애플리케이션 서버와 상관없이 크기 제한을 초과했을 때 반드시 MultipartException을 발생시키려면 어떻게 해야할까? 이럴때는 스프링 웹에서 제공하는 org.springframework.web.multipart.support.MultipartFilter를 사용하면 된다. MultipartFilter를 사용하면 요청 파라미터에 접근하기 전에 멀티파트 요청을 해석할 수 있기 때문에 크기 제한을 초과했을 때 반드시 MultipartException을 발생하게 만들 수 있다.

단 MultipartFilter는 스프링 시큐리티의 FilterChain보다 이전에 정의해야 한다.

- **스프링 MVC와 연계하기 위한 설정**

서블릿 표준의 파일 업로드 기능과 스프링 MVC를 연계하려면 서블릿 표준용 MultipartResolver를 스프링의 DI 컨테이너에 정의하면 된다. 이때 빈 이름은 ‘multipartResolver’로 지정한다.

자바 기반 설정 방식을 이용한 빈 정의

```java
@Bean 
public MultipartResolver multipartResolver(){
	return new StandaartServletMultipartResolver();
}
```

XML 기반 설정 방식을 이용한 빈 정의

```java
<bean
	id="multipartResolver"
	class="org.springframework.web.multipart.support.StandartServletMultipartResolver">
</bean>
```

## 업로드 데이터의 취득

업로드한 파일의 데이터는 org.springframework.web.multipart.MultipartFile을 폼 객체로 바인드하면 접근 가능해진다. 스프링 MVC에서는 javax.servlet.http.Part를 핸들러 메서드의 매개변수로 받을 수 있지만 다음과 같은 이유로 MultipartFile을 폼 객체로 바인드받는 방법이 더 나을 수 있다.

- 특정 업로드 기능의 API(서블릿 API)에 의존하지 않는다.
- Bean Validation 기능으로 입력값 검사를 할 수 있다.

파일 업로드 예시

- **폼 클래스 작성**

업로드 데이터를 받기 위한 프로퍼티를 폼 클래스에 정의한다

```java
public class FileUploadForm implements Serializable {
	// MultipartFile 타입의 프로퍼티를 정의한다.
	private MultipartFile file;
}
```

- **뷰 구현**

뷰에서는 HTML폼에서 멀티파트 요청을 전송한다. 다음은 뷰로 JSP를 사용할 때 구현하는 예이다.

```html
//enctype 속성에 "multipart/form-data"를 지정한다.
<form:form modelAttribute="fileUploadForm" enctype="multipart/form-data">

	//<form:input>요소 type에 file을 지정하고 파일 업로드용 필드를 생성한다.
	파일 : <form:input path="file" type="file"/><br>
	<form:button>업로드</form:button>
</form:form>
```

- **컨트롤러 구현**

폼 객체에서 MultipartFile을 꺼내온 후 파일을 저장할 때 필요한 데이터를 가져온다.

```java
@RequestMapping("/file/upload")
@Controller
public class FileUploadController{
	...
	@RequestMapping(method = RequestMethod.POST)
	public String upload(FileUploadForm form){
		MultipartFile file = form.getFile();
	
		// 콘텐츠 타입 정보를 가져온다
		String contentType = file.getContentType();

		// 요청 파라미터의 이름을 가져온다.
		String parameterName = file.getName();

		// 파일 이름을 가져온다.
		String originalFilename = file.getOriginalFilename();

		// 파일 크기를 가져온다.
		long fileSize = file.getSize();

		// 파일 내용을 java.io.InputStream을 사용해서 가져온다.
		try (InputStream content = file.getInputStream()){
			//업로드된 데이터를 저장
			...
		}
		
		return "redirect:/file/upload?complete";
	}
```

