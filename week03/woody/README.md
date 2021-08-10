## 스프링 핵심 원리 - 빈 스코프



### 빈 스코프

- 빈이 존재할 수 있는 범위
- 스프링 빈 - 기본적으로 싱글톤 스코프로 생성

<br>

### 빈 스코프 종류

- **싱글톤**
  - 기본 스코프
  - 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
  - 싱글톤 빈은 스프링 컨테이너 생성 시점에 초기화 메서드가 실행
- 싱글톤 스코프의 빈을 조회하면 스프링 컨테이너는 항상 같은 인스턴스의 스프링 빈을 반환
- **프로토타입**
  - 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여
    - `preDestroy` 같은 종료 메소드가 자동으로 실행되지 않음
  - 매우 짧은 범위의 스코프
  - 프로토타입 스코프의 빈은 스프링 컨테이너에서 빈을 조회할 때 생성되고, 초기화 메서드도 실행
  - 프로토타입 스코프를 스프링 컨테이너에 조회하면 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환한다.
- 웹 관련 스코프
  - **request** : 웹 요청이 들어오고 나갈때 까지 유지되는 스코프이다.
  - **session** : 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프이다.
  - **application** : 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프이다.

<br>

**싱글톤 VS 프로토타입**

**싱글톤** 스코프의 빈을 조회하면 스프링 컨테이너는 **항상 같은 인스턴스의 스프링 빈을 반환**한다.

**프로토타입** 스코프를 스프링 컨테이너에 조회하면 스프링 컨테이너는 **항상 새로운 인스턴스를 생성**해서 반환한다.

<br>

### 빈 스코프 설정

**컴포넌트 자동 스캔 등록**

```java
@Scope("prototype")
@Component
public class HelloBean {}
```

**수동 등록**

```java
@Scope("prototype")
@Bean
PrototypeBean HelloBean() {
 return new HelloBean();
}
```

### 프로토타입 스코프 - 싱글톤 빈과 함께 사용할 때 문제 발생

스프링 컨테이너에 프로토타입 스코프의 빈을 요청하면 항상 새로운 객체 인스턴스를 생성해서 반환

**BUT**, 싱글톤 빈이 의존관계 주입을 통해서 프로토타입 빈을 주입받아서 사용할 경우

→ 싱글톤에서 해당 빈을 꺼내 사용 할 때마다 새로 생성이 되는 것은 아니다! 착각 금지

→ 내부에 가지고 있는 프로토타입 빈은 이미 과거에 주입이 끝난 빈이다

<br>

### ObjectProvider

- 지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공
- 스프링에 의존적인 클래스

```java
@Autowired
private ObjectProvider<PrototypeBean> prototypeBeanProvider;

public int logic() {
		 // getObject()를 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환(DL)
		 PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
}
```

싱글톤 빈과 프로토타입 빈을 함께 사용할 때, 새로운 프로토타입 빈을 생성하는 방법이 될 수 있다

→ 내부에서 프로토타입 빈 대신 ObjectProvider<프로토타입빈> 형태로 의존성을 주입받으면 됨

<br>

### Provider

- ObjectProvider와 마찬가지로 지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공
- `javax.inject.Provider` 라는 JSR-330 자바 표준을 사용

```java
@Autowired
private Provider<PrototypeBean> provider;
public int logic() {
     // get() 을 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환(DL)
		 PrototypeBean prototypeBean = provider.get();
```

<br>

### 웹 스코프의 특징

- 웹 환경에서만 동작
- 스프링이 해당 스코프의 종료시점까지 관리

### <br>

### 웹 스코프 종류

- request
  - HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리된다.
- session
  - HTTP Session과 동일한 생명주기를 가지는 스코프
- application
  - 서블릿 컨텍스트( ServletContext )와 동일한 생명주기를 가지는 스코프
- websocket
  - 웹 소켓과 동일한 생명주기를 가지는 스코프

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5555bfa1-2fff-4ab4-bb88-f978f080742e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5555bfa1-2fff-4ab4-bb88-f978f080742e/Untitled.png)

<br>

### MyLogger 클래스

- request 스코프 설정

```java
@Component
@Scope(value = "request")
public class MyLogger {
	  private String uuid;
	  private String requestURL;
	 
    // ...

		@PostConstruct
		public void init() {
				 uuid = UUID.randomUUID().toString();
				 System.out.println("[" + uuid + "] request scope bean create:" + this);
		}

		@PreDestroy
		public void close() {
				 System.out.println("[" + uuid + "] request scope bean close:" + this);
		}
	}
}
```

- `@Scope(value = "request")` 를 사용해서 request 스코프로 지정했다.
  - 빈은 HTTP 요청 당 하나씩 생성되고
  - HTTP 요청이 끝나는 시점에 소멸된다.

- 빈 생성 시점에 자동으로 `@PostConstruct` 초기화 메서드를 사용해서 uuid 생성
  - 빈은 HTTP 요청 당 하나씩 생성되므로, uuid로 각 HTTP 요청을 구분할 수 있다.
- 빈 소멸 시점에 `@PreDestroy` 를 호출

<br>

### LogDemoService

- MyLogger를 의존성으로 받음

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {

		 private final MyLogger myLogger;

		 public void logic(String id) {
				 myLogger.log("service id = " + id);
		 }
}
```

<br>

### 🚨 그대로 실행 시 에러 발생!

**스프링 애플리케이션을 실행하는 시점**에 싱글톤 빈은 생성해서 주입이 가능하지만, **request 스코프 빈은 아직 생성되지 않는다**. 이 빈은 실제 고객의 요청이 와야 생성할 수 있기 때문

**Solution 1 - Provider 사용**

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {
		 
		 private final ObjectProvider<MyLogger> myLoggerProvider; // 👈👈
	
		// ...

}
```

**Solution 2 - 프록시 사용**

- proxyMode = ScopedProxyMode.TARGET_CLASS 를 추가해주자.
  - 적용 대상이 클래스 → TARGET_CLASS 적용 대상이 인터페이스 →  INTERFACES
- MyLogger의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관 없이 가짜 프록시 클래스를 다른 빈에 미리 주입해 둘 수 있다.

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS) // 👈👈
public class MyLogger {
			// ...
}
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e4f5dccb-821d-46e5-a88f-55d3edbdf213/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e4f5dccb-821d-46e5-a88f-55d3edbdf213/Untitled.png)

<br>

**CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다.**

- 가짜 프록시 객체는 요청이 오면 그때 내부에서 진짜 빈을 요청하는 위임 로직이 들어있다
  - 클라이언트가 `myLogger.logic()` 을 호출하면 사실은 가짜 프록시 객체의 메서드를 호출
  - 가짜 프록시 객체가 request 스코프의 진짜 `myLogger.logic()` 호출
- 클라이언트 입장에서는 사실 원본인지 아닌지도 모르게, 동일하게 사용할 수 있다(다형성)
- POINT! 진짜 객체 조회를 꼭 필요한 시점까지 **지연처리** 한다는 점