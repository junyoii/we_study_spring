# 스프링 웹 개발 기초

### **1. 정적 컨텐츠**

정적리소스

- 고정된 HTML 파일, CSS, JS, 이미지, 영상 등을 제공
- 주로 웹 브라우저

<img width="1023" alt="_2021-05-05__7 32 44" src="https://user-images.githubusercontent.com/67837091/117134667-ce477080-ade0-11eb-9ef5-33400d92a4d2.png">

⇒ hello와 관련된 컨트롤러가 있는지 살펴본다. 여기서는 해당 컨트롤러가 없기 때문에 정적 컨텐츠(*`hello.html`*)을 찾아서 서빙한다.

### 2. MVC 패턴

<img width="643" alt="_2021-05-05__7 49 50" src="https://user-images.githubusercontent.com/67837091/117134691-d7d0d880-ade0-11eb-9403-309f36fe5a87.png">
<img width="649" alt="_2021-05-05__7 50 32" src="https://user-images.githubusercontent.com/67837091/117134699-db645f80-ade0-11eb-87bf-dc94a9c4b699.png">
<img width="640" alt="_2021-05-05__7 50 50" src="https://user-images.githubusercontent.com/67837091/117134703-dd2e2300-ade0-11eb-8b2c-0bc428fa731e.png">

```java
@Controller
public class HelloController {

	@GetMapping("hello-mvc")
	public String helloMvc(@RequestParam("name") String name, Model model) {
		model.addAttribute("name", name);
		return "hello-template";
	}
}
```

```html
<html xmlns:th="http://www.thymeleaf.org">
	<body>
	<p th:text="'hello ' + ${name}">hello! empty</p>
	</body>
</html>
```

- 먼저 [localhost:8000/hello-mvc로](http://localhost:8000/hello-mvc로) GET 요청이 들어온다.
- 스프링 부트는 먼저 *hello-mvc* 컨트롤러를 찾는다.
- 여기서는 `HelloController`가 존재하기 때문에 `hello-template.html` 파일을 서빙한다. 이때 model에 name:spring  형식으로 데이터를 담아서 전달.
- Thymeleaf 템플릿 엔진 처리 후 웹 브라우저에 띄운다.

### 3. API

```java
@Controller
public class HelloController {

	@GetMapping("hello-string")
	@ResponseBody
	public String helloString(@RequestParam("name") String name) {
		return "hello " + name;
	}
}
```

- **HTTP 응답 body부**에 "hello "+name을 전달 (ex. "hello spring")

```java
hello spring
```

- 따라서 *localhost:8080/hello-string?name=spring*을 들어가서 소스코드를 까보면 HTML 태그 없이 무식하게 "hello spring"이 그대로 들어오게 된다.

```java
@Controller
public class HelloController {

	@GetMapping("hello-api")
	@ResponseBody
	public Hello helloApi(@RequestParam("name") String name) {
		Hello hello = new Hello();
		hello.setName(name);
		return hello;
	}
	
	static class Hello {
		private String name;
		
		public String getName() {
			return name;
		}
		
		public void setName(String name) {
			this.name = name;
		}
	}
}
```

- **객체가 반환된다면 JSON 방식으로 데이터를 만들어서 HTTP 응답에 반환**하는 게 default 정책.

```java
{name: name}
```
