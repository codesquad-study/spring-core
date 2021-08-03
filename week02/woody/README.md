## 스프링 컨테이너
### IoC 컨테이너 (Inversion of Control)

-   제어권을 개발자가 아닌 제 3자(프레임워크)가 가지게 하는 것
    
-   The container then injects those dependencies when it creates the bean
    
    This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.
    
    -   빈이 자신의 의존성을 직접적으로 관리하는 것이 아니라 IoC 컨테이너가 빈이 생성될 때 해당 빈의 의존성을 주입 및 관리해주는 역할을 한다.

### 빈 (bean)

-   In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans [[참고]](https://docs.spring.io/spring-framework/docs/5.3.2/reference/html/core.html#beans)
    -   어플리케이션의 중추가 되는 오브젝트
    -   스프링이 loC 방식으로 관리하는 오브젝트
-   A bean is an object that is instantiated, assembled, and managed by a Spring IoC container
    -   빈은 스프링 IoC 컨테이너에 의해서 생성되고, 조합되거나 관리되는 오브젝트

### 빈 팩토리 (bean factory)

-   빈의 등록, 생성, 조회 등 빈을 관리하는 기능을 담당하는
-   getBean() 메소드가 정의되어 있다

### 어플리케이션 컨텍스트 (application context)

-   빈 팩토리를 확장한 IoC 컨테이너 (BeanFactory를 상속받아 구현)

## 빈 설정 방식

### 자바 기반 설정 (java based config)

자바 클래스에 `@Configuration`, 자바 메소드에 `@Bean`을 추가하여 빈을 정의하는 방식

### 어노테이션 기반 설정 (annotation based config)

마커 어노테이션 `@Component` 이 부여된 클래스를 스캔(Component Scan)해서 DI 컨테이너에 자동으로 빈을 등록

### 컴포넌트 스캔(Component Scan)

-   클래스 로더를 스캔하면서 특정 클래스를 찾은 다음 DI 컨테이너에 등록하는 방법
-   설정 정보 없이 자동으로 스프링 빈을 등록할 수 있다

### 컴포넌트 스캔 기본 대상

-   `@Component` 어노테이션이 붙은 클래스를 스캔해서 스프링 빈으로 등록

아래 어노테이션도 @Component을 포함하고 있기 때문에 컴포넌트 스캔 대상이 된다.

-   `@Service` : 스프링 비지니스 로직에서 사용
-   `@Repository` : 스프링 데이터 접근 계층에서 사용
-   `@Controller` : 스프링 MVC 컨트롤러에서 사용
-   `@Configuration` : 스프링 설정 정보에서 사용

✨ 기본적으로 어노테이션에는 상속 관계가 없기 때문에 위의 **어노테이션들이 컴포넌트 어노테이션을 들고 있는걸 인식할 수 없다**.

→ 자바의 기본 기능이 아니라 **스프링이 부가적으로 지원하는 기능**

## 싱글톤
## 싱글톤 패턴

-   클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴
-   주로 스레드 풀, 설정 정보, 로그 기록용 객체 등 관리용 객체처럼 반드시 하나만 존재해야하는 경우에 많이 사용됨

### 고전적인 싱글턴 패턴 구현법

🚨 멀티 쓰레드에 적절하지 못함

```java
public class Singleton {
		private static Singleton uniqueInstance;
		
		private Singleton() { }
		
		public static Singleton getInstance() {
				if (uniqueInstance == null) {
						uniqueInstance = new Singleton();
				}
				return uniqueInstance;
		}
}

```

1번 스레드가 `if (uniqueInstance == null)`을 호출해서 true → if statement 내부로 진입

2번 스레드로 context switching

2번 스레드가 `if (uniqueInstance == null)`을 호출해서 true → if statement 내부로 진입

⇒ 두 개의 인스턴스가 생성

### 멀티 스레딩 문제 해결 방법

🚨 불필요한 오버헤드 발생

```java
public class Singleton {
		private static Singleton uniqueInstance;
		
		private Singleton() { }
		
		public static synchronized Singleton getInstance() { // ✨✨ synchronized 추가
				if (uniqueInstance == null) {
						uniqueInstance = new Singleton();
				}
				return uniqueInstance;
		}
}

```

한 스레드가 메소드 사용을 끝내기 전가지 다른 스레드는 기다려야 한다.

BUT! 일단 uniqueInstnace 변수에 Singleton 인스턴스가 대입된 이후에는 굳이 메소드를 동기화 상태로 유지할 필요가 없다!!

### DCL(Double-Checked-Locking) Singleton 패턴

🚨 jdk 1.5 이상에서만 사용 가능

```java
public class Singleton {
		private volatile static Singleton uniqueInstance; // ✨✨ volatile 추가
		
		private Singleton() { }
		
		public static Singleton getInstance() {
				if (uniqueInstance == null) {
						uniqueInstance = new Singleton();
				}
				return uniqueInstance;
		}
}

```

-   멀티 스레드 환경에서 스레드가 변수 값을 읽어올 때 각각의 CPU Cache 에 저장된 값이 다르기 때문에 변수 값 불일치 문제가 발생하게 된다.
-   인스턴스가 CPU 캐시에서 변수를 참조하지 않고 메인 메모리에서 변수를 참조한다 → 변수 값 불일치가 생기지 않는다.

???? 그런데 이렇게 해도 서로 다른 쓰레드가 간발의 차이로 둘다 if statement로 들어갈 수 있는 가능성이 있는거 아닌가.

### LazyHolder Singleton 패턴

```java
public class Singleton { 
		private Singleton() { } 
		public static Singleton getInstance() { 
					return LazyHolder.INSTANCE; 
		}
	 
		private static class LazyHolder { 
					private static final Singleton INSTANCE = new Singleton(); 
		} 
}


```

-   static 영역에 초기화를 하지만, 객체가 필요한 시점까지 초기화를 미루는 방식
-   LazyHolder 클래스의 변수가 없기 때문에 Singleton 클래스 로딩 시 LazyHolder 클래스를 초기화하지 않는다.
-   Class를 로딩하고 초기화하는 시점은 thread-safe를 보장하기 때문에 동시성 문제가 발생하지 않는다.

## 싱글톤 컨테이너

스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하면서 객체 인스턴스를 싱글톤으로 관리한다.

👉 스프링 빈은 싱글톤으로 관리된다!

👉 이미 만들어진 객체를 공유해서 효율적으로 재사용 가능

### @Configuration

-   @Configuration 사용 O
    
    → CGLIB 라이브러리를 사용해서 빈의 싱글톤을 보장한다
    
-   @Configuration 사용 X (@Bean만 적용)
    
    -   싱글톤이 보장되지 않는다.
    -   호출한 횟수 만큼 인스턴스가 생성

### 무상태성

-   객체 인스턴스를 하나만 생성해서 공유하는 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하므로 주의해야 한다
-   따라서 싱글톤 객체는 상태를 유지하도록 설계하면 안된다 → 무상태(stateless) 설계 필요

### 무상태 설계

-   특정 클라이언트에 의존적인 필드 X
-   특정 클라이언트가 값을 변경할 수 있는 필드 X
-   가급적이면 읽기만 허용
-   필드 대신 자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.

## Reference

-   [자바 공작소](https://javaplant.tistory.com/21) - DCL
-   Head First Design Patterns
-   [Multi Thread 환경에서의 올바른 Singleton](https://medium.com/@joongwon/multi-thread-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-singleton-578d9511fd42) - Lazy Loading