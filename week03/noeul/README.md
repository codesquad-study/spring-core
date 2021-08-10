# 의존관계 자동 주입



## 다양한 의존관계 주입 방법

- 생성자 주입
  - 불변, 필수 의존관계에 사용
  - 스프링 빈인 경우, 생성자가 1개라면`@Autowired를 생략해도 자동 주입`
- Setter 주입
  - 선택, 변경 가능성이 있는 의존관계에 사용
  - Setter메서드 마다 `@Autowired`를 명시해서 주어야하고, 주입할 대상이 없다면 오류가 발생한다.
- 필드 주입
  - 비추천
  - Autowired는 스프링 컨테이너에서 빈이 등록된 후 의존관계를 맺는 형태인데, `Bean` 클래스를 new()로 생성하면 컨테이너에 들어가지 않으므로, 자동주입이 될 수 없음.
    - 테스트에서 Mock을 만들 수 있는 방법이 없음.
    - 즉, DI 프레임워크가 없으면 아무것도 못함.
    - 주입할 스프링 빈이 없이 동작해야 하는 경우는 `옵션 처리`필요
  - 실제 코드와 관계없는 테스트코드나 @Configuration 같은 곳에서 특별한 용도로 사용
- 일반 메서드 주입
  - 사용하는 경우가 거의 없음
  - 한번에 여러 필드를 주입 받을 수 있음. (Setter)

> 좋은 개발습관은 제약을 두는 것 ex) final..

## 옵션 처리

- 주입할 스프링 빈이 없어도 동작해야 할 때 사용
  - `@Autowired(required=false)` : 주입할 대상이 없다면 메서드 호출 X
  - @Nullable : 주입할 대상이 없다면 null 입력
  - Optional<> :자동 주입할 대상이 없으면 Optional.empty 입력

## 생성자 주입을 선택해라!

- 대부분의 의존관계는 애플리케이션이 종료하기 전까지 변하면 안됨.
- 생성자 주입을 하게 될 경우
  - 의존관계를 개발자가 인지하기 쉬움.
  - final 키워드 사용: 생성자에서 불변값을 세팅 할 수 있음, 누락시 컴파일 시점에서 막아줌
- 생성자 주입 방식을 선택하는 이유는
  - 프레임워크에 의존하지 않고, 순수한 자바 언어의 특징을 잘 살리는 방법이기도 함.

## 롬복과 최신 트랜드

- 롬복 짱짱

## 조회 빈이 2개 이상 - 문제

- Autowired는 타입으로 조회하기 때문에, 자동으로 의존관계를 맺을 때 같은 타입의 Bean이 복수개 있다면 충돌이 생길 수 있음.
- 그렇다고 타입을 명확히 지정해서 의존관계를 맺으면 DIP를 위반함.

## @Autowired 필드 명,  @Qualifier, @Primary

- Autowired
  - 타입 매칭 시도 
  - 타입 매칭의 결과가 2개 이상일 대 필드 명, 파라미터 명으로 빈 이름 매칭

- Qulifier
  - 추가 구분자를 붙여주는 방법
  - 빈 이름을 변경하는 것은 아님.
  - 빈 등록시 @Qualifier를 붙여주고, 주입시에도 @Qualifier를 붙여주고 등록한 이름을 적어줌.
  - Qualifier의 네이밍을 못찾으면 해당 네이밍의 스프링 빈을 추가로 찾음
- Primary
  - @Autowired시 여러 빈이 매칭되면 우선권을 가짐

- Qualifier와 Primary의 우선권
  - 더 좁은 상세한 범위인 Qualifier가 우선순위가 더 높음

## 애노테이션 직접 만들기

- 웬만한건 스프링이 제공하는 애노테이션으로 해결이 되지만, Qualifier 애노테이션에서 사용하는 value의 문자열은 컴파일타임에 에러를 체크할 수 없으므로, 커스텀 애노테이션을 만들어서 컴파일 타임에도 체크 가능하게 하는 예제를 볼 수 있었다.

- 커스텀 애노테이션에는 Qualifier가 포함된 형태의 애노테이션이었는데, 스프링에서 애노테이션을 중첩하는 기능을 제공해준다고 한다.(순수 자바에서는 애노테이션의 상속이 불가능)

  

## 조회한 빈이 모두 필요할 때,  List,  Map

- 동적으로 빈을 생성해야 할 때, 모든 Bean을 Map으로 불러와서 생성하는 방법을 배움.

- Map의 Key(String)는 Bean 이름이 저장됨.

  ```java
  public DiscountService(Map<String, DiscountPolicy> policyMap,
  List<DiscountPolicy> policies) {
  ...
  }
  ```

  

## 자동, 수동의 올바른 실무 운영 기준

- 편리한 자동 기능(빈 등록..)을 기본으로 사용하자
- 직접 등록하는 기술 지원 객체, 공통 관심사(AOP)는 수동 등록
  - 데이터베이스 연결, 공통 로그 처리
- 다형성을 적극 활용하는 비즈니스 로직은 수동 등록 고민
  - ex) 동적 빈 생성..
    - Map 형태는 해당 빈을 구현하고 있는 객체를 바로 볼 수 없어서, 구조를 한 눈에 파악하기 어려움.



# 빈 생명주기 콜백

---

## 빈 생명주기 콜백 시작

- 데이터베이스 커넥션 풀이나, 네트워크 소켓처럼  애플리케이션 시작 시점에 필요한 연결을 미리 해두고, 애플리케이션 종료 시점에 연결을 모두 종료하는 작업을 진행하려면, 객체의 `초기화`와 `종료` 작업이 필요하다.
- 스프링 빈의 이벤트 라이프 사이클
  - 스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> `초기화` 콜백 -> 사용 -> 소멸전 콜백 -> 스프링 `종료`
- 객체의 생성과 초기화 분리
  - 단일 책임 관점

---

## 인터페이스 InitializingBean, DisposableBean

```java
public class NetworkClient implements InitializingBean, DisposableBean {
    
...
	@Override
    public void afterPropertiesSet() throws Exception {
        connect();
        call("초기화 연결 메시지");
    }	
    @Override
    public void destroy() throws Exception {
		disConnect();
	}
...
```

- `IntiailizingBean` 인터페이스는 `afterPRopertiesSet()` 메서드로 초기화를 지원
- `DisposableBean` 인터페이스는 `destroy()` 메서드로 종료(소멸)을 지원
- 단점
  - 초창기에 나온 방식으로 요새는 잘 안쓰임.
  - 스프링 전용 인터페이스로 코드가 스프링에 의존하게 된다.
  - 구현 메서드의 이름을 변경할 수 없다.
  - 내가 코드를 고칠 수 없는 외부 라이브러리에는 적용할 수 없다.

---

## 빈 등록 초기화, 소멸 메서드

- 설정 정보 `@Configuration` 에서  @Bean 애노테이션에 옵션을 통해 메서드 지정 가능

  ```java
  @Bean(initMethod = "init", destroyMethod = "close")
  public NetworkClient networkClient() {
  ```

  - 특징
    - destroyMethod는 추론((inferred)이 가능해서, 기본값으로 둘 경우 close, shutdown 네이밍에 해당 하는 메소드를 자동 탐색
    - 메서드 이름 커스텀 가능
    - 스프링 코드 의존 X
    - 외부 라이브러리에도 적용 가능

---

## 애노테이션  @PostConstruct, @PreDestory

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@PostConstruct
public void init() {
}
@PreDestroy
public void close() {
}
```

- 최신 스프링에서 가장 권고하는 방법
- 설정 정보가 아닌, 원하는 빈 클래스 메서드에 애노테이션을 붙여주면 된다. (간단)
- 스프링 종속적인 기술X, 자바 표준
- 단점: 수정이 불가능한 외부 라이브러리에는 적용이 불가능

---

# 빈 스코프

---

## 빈 스코프란?

- 빈의 생명주기

- 종류

  - singleton : 기본 스코프, 스프링 컨테이너와 생명주기가 같은 가장 넓은 범위의 스코프

  - prototype : 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하는 짧은 범위 스코프
  - request : 웹 요청이 들어오고 나갈때까지 유지되는 스코프

- 사용

  ```java
  @Scope("prototype")
  @Component
  ```

---

## 프로토타입 스코프

- 싱글톤 스코프와 다르게, 스프링 컨테이너에 프로토타입 스코프의 빈을 요청하면 항상 새로운 객체 인스턴스를 생성해서 반환한다.
- 프로토타입 스코프는 초기화 이후 관리를 안해주기 때문에, 종료 메서드(@PreDestory)가 작동하지 않는다.

---

## 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 문제점

- 싱글톤 빈에서 의존관계 주입을 통해서 프로토 타입 빈을 주입 받아서 사용할 경우, 항상 새로운 프로토타입을 생성할 수 있을까?
  - 싱글톤 빈이 생성되고, 의존관계 자동 주입을 사용한다. 
  - 이후 스프링 컨테이너에게 프로토타입 빈이 `요청`되면, 프로토타입 빈을 `새로 생성`한다.
  - 이제 싱글톤 빈 내부 필드에 프로토타입 빈이 보관된다.
  - 싱글톤 빈 내부의 프로토타입 빈은 과거 의존 관계 주입으로 생성이 끝난 빈으로, 프로토타입 빈을 **사용할 때마다 새로운 빈이 생성되지는 않는다.** (의도와 다른결과)

---

## 프로토타입 스코프 - 싱글톤 빈과 사용시 Provider로 문제 해결

> 프로토타입 빈은 실무에서 직접적으로 사용하는 일은 매우 드물다고 한다.

1. 가장 간단한 방법은 싱글톤 빈이 프로토타입을 사용할 때 마다 스프링 컨테이너에 새로 요청하는 것

```java
@Autowired private ApplicationContext ac;

public int logic() {
	PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
	prototypeBean.addCount();
    
	int count = prototypeBean.getCount();
	return count;
}
```

- DL(=Depandency Lookup) 의존관계를 외부에서 주입받지 않고, 직접 필요한 의존관계 찾기

- 스프링 애플리케이션 컨텍스트 전체를 주입받게 되면, 스프링 컨테이너에 종속적인 코드가 되고, 단위 테스트가 어렵다??

  

### 지정한 프로토타입 빈을 컨테이너에서 대신 찾아주는 DL 기능만 제공하는게 필요.

2. ObjectProvider ( extends ObjectFactory ) 
   - 기능이 단순해 단위 테스트를 만들거나 mock 코드 만들기 쉬움
   - 스프링에 의존적이며, 별도의 라이브러리 필요 없음.
   - `getObject()`  - DL

3. JSR-330 Provider 
   - javax.inject.Provider
   - 자바 표준이지만 외부 라이브러리 등록 필요.
   - 단위 테스트나 mock 만들기 쉬움
   - `get()` - DL

---

## 웹 스코프

- 웹 환경에서만 동작
- 프로토타입 스코프와 다르게 스프링이 해당 스코프 종료시점까지 관리
  - 종류
    - request, session, application, websocket
- request 
  - 요청이 오면 요청이 끝날 때까지 해당 클라이언트의 전용 빈을 생성하고 관리한다.

---

## request 스코프 예제 만들기

```java
@Component
@Scope(value = "request")
public class MyLogger {
```

```
Error creating bean with name 'myLogger': Scope 'request' is not active for the
current thread; consider defining a scoped proxy for this bean if you intend to
refer to it from a singleton;
```

- 스프링 애플리케이션이 실행하는 시점에 의존관계를 자동으로 맺으려는데 request 스코프 빈이 아직 생성되지 않아서 생긴 오류
  - request scope는 고객 요청이 와야 생성이 된다.
  - Provider 필요

---

## 스코프와 Provider

- 앞서서 배운 Provider 를 통해 요청이 왔을 때, DL 하는 방식

---

## 스코프와 프록시 



---

