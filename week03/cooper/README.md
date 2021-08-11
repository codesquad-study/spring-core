1. # [week3] Dependency Autowired, Bean Lifecycle Callback, Bean Scope

   ### index

   ------

   1. 의존 관계 주입 4가지
   2. 빈 충돌 시 해결법
   3. 빈 라이프 사이클
   4. 빈 스코프

   <br>

   ## 1. 의존 관계 주입 4가지

   ### 1. 생성자 주입

   > 생성자를 통해서 의존 관계를 주입받는 방법

   - 예시 코드

     ```java
     public class OrderServiceImpl implements OrderService {
     
         private final MemberRepository memberRepository;// final 선언하면 값이 무조건 있어야 한다!
         private final DiscountPolicy discountPolicy;
     
         @Autowired // 생성자 주입 방식 : 생성자 하나일 경우 생략 가능!
         public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
             this.memberRepository = memberRepository;
             this.discountPolicy = discountPolicy;
         }
     }
     ```

     1. 생성자가 하나일 경우, `@Autowired 생략` 가능!

     2. 생성자 주입을 선언하면 `final 키워드`를 사용할 수 있다.

        (Java에서 final은 불변보다는 필수 + 재할당 금지라는 말이 적합한 것 같다.)

     3. 생성자 주입의 장점 : `불변`, `누락`, `final 키워드`

     - `불변` : 생성자 주입을 객체 생성 시, `1회만 호출`하므로 이후 호출할 일이 없어 불변하게 설계 가능

       (대부분의 의존 관계 주입은 한번 일어나면 애플리케이션 종료 시점까지 의존관계가 변하면 안된다.)

     - `누락` : 생성자 주입을 사용할 시, 주입 데이터 누락일 경우 `컴파일 오류 발생` (final 선언 시)

     - `final 키워드` : 만약 값이 설정되지 않는 오류를 컴파일 시점에서 막아준다.

       ```java
       Component
       public class OrderServiceImpl implements OrderService {
           private final MemberRepository memberRepository;
           private final DiscountPolicy discountPolicy;
       
       		@Autowired
           public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy
           discountPolicy) {
               this.memberRepository = memberRepository;
           }
       }
       
       // 컴파일 에러 발생
       // java: variable discountPolicy might not have been initialized
       ```

   <br>

   ### 2. 수정자 주입(setter 주입)

   > 수정자(setter)를 통해서 의존관계를 주입하는 방식

   - 예시 코드

     ```java
     @Component
     public class OrderServiceImpl implements OrderService {
         private MemberRepository memberRepository;
         private DiscountPolicy discountPolicy;
     
         @Autowired
         public void setMemberRepository(MemberRepository memberRepository) {
             this.memberRepository = memberRepository;
         }
     
         @Autowired
         public void setDiscountPolicy(DiscountPolicy discountPolicy) {
             this.discountPolicy = discountPolicy;
         }
     }
     ```

     1. **`선택, 변경`** 가능성이 있는 의존관계에 사용 가능
     2. @Autowired는 주입할 대상이 없으면 오류 발생하므로 만약 오류 발생을 원하지 않는다면 `@Autowired(required = false)`를 지정한다.

     <br>

     **자동 주입 대상 옵션으로 처리 방법**

     > - `@Autowired(required = false)` : 자동 주입 대상이 없으면 수정자 메서드 자체 호출 안함.

     - `org.psringframework.lang.@Nullable` : 자동 주입 대상이 없으면 null 반환
     - `Optional<>` : 자동 주입 대상이 없으면 `Optional.empty`가 입력된다.

     ```java
     //호출 안됨
     @Autowired(required = false)
     public void setNoBean1(Member member) {
           System.out.println("setNoBean1 = " + member);
       }
     
     //null 호출
     @Autowired
     public void setNoBean2(@Nullable Member member) {
         System.out.println("setNoBean2 = " + member);
     }
     
     //Optional.empty 호출
     @Autowired(required = false)
     public void setNoBean3(Optional<Member> member) {
         System.out.println("setNoBean3 = " + member);
     }
     
     /* 출력 결과
      * 1. setNoBean1 : 호출이 안됨.
      * 2. setNoBean2 : null
      * 3, setNoBean3 : Optional.empty
      */
     ```

   <br>

   ### 3. 필드 주입

   > 필드에 바로 의존 관계를 주입하는 방식

   - 예시 코드

     ```java
     @Component
     public class OrderServiceImpl implements OrderService {
         @Autowired
         private MemberRepository memberRepository;
         @Autowired
         private DiscountPolicy discountPolicy;
     }
     ```

     1. 외부 변경이 불가능해 테스트하기 힘들다는 단점이 존재한다.
     2. 주로 메인 코드에서는 사용하지 말고, `테스트 코드` 혹은 `스프링 설정`을 목표로 하는 @Configuration 같은 곳에서만 사용하는 것을 추천

     <br>

   ### 4. 일반 메서드 주입

     > 일반 메서드를 통해서 주입할 수 있다.

     - 예시 코드

       ```java
       @Component
       public class OrderServiceImpl implements OrderService {
           private MemberRepository memberRepository;
           private DiscountPolicy discountPolicy;
       
       		@Autowired
           public void init(MemberRepository memberRepository, DiscountPolicy
       discountPolicy) {
               this.memberRepository = memberRepository;
               this.discountPolicy = discountPolicy;
           }
       }
       ```

       1. 한번에 여러 필드 주입이 가능하지만 일반적으로 `잘 사용하지 않는 방식`이다.

   <br><br>

   ## 2. 빈 충돌 시 해결법 : @Autowired 필드명, @Qualifier, @Primary

   ### 1. @Autowired 매칭 정리

   > @Autowired는 우선 `타입 매칭`으로 시도하고 타입 매칭 결과가 2개 이상 필드일 경우, `필드 명`, `파라미터 명`으로 빈 이름 매칭한다.

   - DiscountPolicy이 2개 이상 있을 경우, 필드명을 discountPolcy를  rateDiscountPolicy로 변경하면 매칭 가능!

   <br>

   ### 2. @Qualifier사용

   > @Qualifier라는 추가 구분자를 추가는 방법

   - 예시

     ```java
     @Component
     @Qualifier("mainDiscountPolicy")
     public class RateDiscountPolicy implements DiscountPolicy {}
     
     //주입 시, @Qualifier annotaion을 추가해서 맵핑하도록 하자!
     @Autowired
     public OrderServiceImpl(MemberRepository memberRepository,
     	 @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
     	this.memberRepository = memberRepository;
       this.discountPolicy = discountPolicy;
     }
     ```

     1. 만약 @Qulifier("mainDiscountPolicy")를 조회 실패할 경우, `mainDiscountPolicy라는 이름의 스프링 빈을 추가로 탐색`한다. (하지만, @Qualifier는 @Qualifier 용도로 사용하는 것 추천!)

     2. 그러 와중에도 발견하지 못한다면 `NoSuchBeanDefinitionException` 발생

     3. @Qualifier를 annotation을 만들면 `컴파일 시 타입 체크`를 할 수 있다!

        ```java
        @Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
        @Retention(RetentionPolicy.RUNTIME)
        @Documented           
        @Qualifier("mainDiscountPolicy")
        public @interface MainDiscountPolicy {
        }
        
        //사용 시
        @Component
        @MainDiscountPolicy
        public class RateDiscountPolicy implements DiscountPolicy {}
        ```

   <br>

   ### 3. @Primary 사용

   > @Primary annotation을 추가할 경우 여러 빈이 매칭될 때 `우선권`을 갖는다.

   - 예시

     ```java
     @Component
     @Primary
     public class RateDiscountPolicy implements DiscountPolicy {}
     
     @Component
     public class FixDiscountPolicy implements DiscountPolicy {}
     
     //생성자
     @Autowired
     public OrderServiceImpl(MemberRepository memberRepository,
     		DiscountPolicy discountPolicy) {
       this.memberRepository = memberRepository;
       this.discountPolicy = discountPolicy;
     }
     ```

   <br>

   ### 4. 활용 추천

   - `메인` 데이터 베이스의 커넥션을 획득하는 스프링 빈에서는 `@Primary` annotation을 적용하자!

   - `서브` 데이터베이스 커넥션 빈을 획득할 때는 `@Qualifier`를 지정해 명시적으로 획득하는 방식으로 사용하도록 하자.

   - 우선 순쉬 : `@Primary < @Qualifier`

     (좁은 범위의 선택권일수록 우선순위가 높다 → @Qualifier가 좁은 범위의 선택권)

   <br><br>

   ## 3. 빈 라이프 사이클

   ------

   ### 1. **스프링 빈의 이벤트 라이프 사이클**

   > 스프링 컨테이너 생성 → 스프링 빈 생성 → 의존 관계 주입 → `초기화 콜백` → 사용 → `소멸전 콜백` → 스프링 종료

   **객체의 생성과 초기화를 분리하자!**

   - 생성자 : `필수 정보를 받고`, 메모리를 할당해서 객체를 생성하는 책임.
   - 초기화 : `생성된 값들을 활용`해서 외부 커넥션을 연결하는 작업 수행.

   → 책임의 입장에서 생성자와 초기화 작업을 분리해서 가져가는 것이 `유지보수 관점`에서 좋다.

   <br>

   **스프링의 빈 생명주기 콜백**

   1. 인터페이스(InitializingBean, DisposableBean)
   2. 설정 정보에 초기화 메서드 종료 메서드 지정
   3. @PostConstruct, @PreDestroy 에노테이션 지원

   <br>

   ### 2. 인터페이스(InitializingBean, DisposableBean)

   ```java
   import org.springframework.beans.factory.DisposableBean;
   import org.springframework.beans.factory.InitializingBean;
   
   public class NetworkClient implements InitializingBean, DisposableBean {
   	
   	@Override
     public void afterPropertiesSet() throws Exception {
   		//초기화 콜백처리
   	}
   
   	@Override
     public void destroy() throws Exception {
   		//소멸전 콜백처리
     }
   }
   ```

   - 스프링 전용 인터페이스이므로, 해당 코드가 `스프링 전용 인터페이스에 의존`한다.
   - 초기화, 소멸 `메서드의 이름 변경 불가`.
   - `외부 라이브러리에 적용 불가`
   - 거의 사용하지 않는 방법.

   <br>

   ### 3. 빈 등록 초기화, 소멸 메서드 지정

   ```java
   public class NetworkClient {
   	public void init() {
   		System.out.println("NetworkClient.init");
   	}
   
   	public void close() {
   		System.out.println("NetworkClient.close");
   	}
   }
   
   @Configuration
   class LifeCycleConfig {
   	@Bean(initMethod = "init", destroyMethod = "close") // 초기화, 소멸 메서드 선언
       public NetworkClient networkClient() {
         NetworkClient networkClient = new NetworkClient();
         networkClient.setUrl("<http://hello-spring.dev>");
         return networkClient;
   	}
   }
   ```

   - `메서드 이름`을 자유롭게 설정 가능

   - 스프링 빈이 스프링 코드에 `의존하지 않는다.`

   - 설정 정보를 사용하기 때문에 `외부 라이브러리`에도 `초기화, 종료 메서드 적용 가능`

   - @Bean의destroyMethod 추론 기능 : close, shutdown 메서드를 자동으로 호출해준다.

     (대부분의 라이브러리의 종료 메서드 명이 close()와 shutdown()을 사용한다.)

     (추론 기능이 싫을 경우 `destroyMethod = ""` 으로 지정한다.)

   <br>

   ### 4. 에노테이션 @PostConstruct, @PreDestroy

   ```java
   import javax.annotation.PostConstruct;
   import javax.annotation.PreDestroy;
   
   public class NetworkClient {
   	@PostConstruct
     public void init() {
   		System.out.println("NetworkClient.init"); connect();
   		call("초기화 연결 메시지");
   	}
   
   	@PreDestroy
     public void close() {
         System.out.println("NetworkClient.close");
         disConnect();
     }
   }
   
   @Configuration
   class LifeCycleConfig {
   	
   	@Bean
     public NetworkClient networkClient() {
   		NetworkClient networkClient = new NetworkClient();
       networkClient.setUrl("<http://hello-spring.dev>");
   		return networkClient;
   	}
   }
   ```

   - 최신 스프링에서 권장하는 기술이며 에노테이션 하나만 붙여서 편리하다.
   - `JSR-250 자바 표준`이다. 스프링이 아닌 다른 컨테이너에서도 동작한다.
   - `컴포넌트 스캔`과 잘 어울린다.
   - `외부 라이브러리에 적용하지 못한다.`

   <br>

   ### 5. 결론

   - **@PostConstruct, @PreDestory 애노테이션을 사용하자**
   - 코드를 고칠 수 없는 `외부 라이브러리`를 초기화, 종료해야 하면 @Bean 의 `initMethod` , `destroyMethod`를 사용하자.

   <br><br>

   ## 4. 빈 스코프

   ### 1. 빈 스코프란?

   > 빈이 존재할 수 있는 범위를 나타낸다.

   - `싱글톤` : 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위.
   - `프로토타입` : 스프링 컨테이너는 프로토타입 빈의 생성과 의존 관계 주입까지만 관여하고 더는 관리하지 않은 짧은 범위 스코프
   - 웹 관련 스코프
     - `request` : 웹 요청이 들어오고 나갈때까지 유지되는 스코프
     - `session` : 웹 세션이 생성되고 종료될 때까지 유지되는 스코프
     - `application` : 웹의 서블릿 컨텍스와 같은 범위로 유지되는 스코프

   **컴포넌트 스캔 자동 등록**

   ```java
   @Scope("prototype")
   @Component
   public class HelloBean {}
   ```

   <br>

   **컴포넌트 스캔 수동 등록**

   ```java
   @Scope("prototype")
   @Bean
   PrototypeBean HelloBean() {
   	return new HelloBean();
   }
   ```

   <br>

   ### 2. 싱글톤 타입 + 프로토 타입

   1. `싱글톤`

      - 여러 클라이언트에게 `같은 빈`을 반환한다. 이후 스프링 컨테이너에 같은 요청이 와도 같은 객체 인스턴스의 스프링 빈을 반환한다.

      <br>

   2. `프로토 타입`

      - 각각의 클라이언트에게 `각각의 빈`을 반환한다. 빈을 스프링 컨테이너에 요청 시, 프로토 타입 빈을 생성하고 필요한 의존 관계를 주입한다.
      - 클라이언트에게 빈을 반환한 이후 생성된 프로토타입 빈을 관리하지 않는다. `@PreDestroy` 같은 종료 메서드를 호출하지 않는다. (`프로토 타입 빈 생성`, `의존관계 주입`, `초기화`까지만 처리하므로 @PostConstruct는 기능한다!)

      <br>

   3. `프로토 스코프 + 싱글톤 빈` 함께 사용시 문제점

      ![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/7565eb43-1e20-4137-8e49-52ea6762281b/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210810%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210810T103856Z&X-Amz-Expires=86400&X-Amz-Signature=a1649b125065ff2de43269b9b0971dc75cf7c13029806fab8db205b592643655&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

      - 싱글톤 빈에 프로토 타입 빈 의존관계를 주입할 경우, 싱글톤은 생성 시점에만 의존관계를 주입받기 때문에 프로토 타입 빈이 생성되지만, 싱글톤 빈의 의존성을 변경하지 않는다. (`사용할 때마다 생성되지 않는다!`)
      - 이 부분을 인지하고 있지 못한다면 객체 필드의 `일관성 결함을 야기`시킬 수 있다.

      <br>

   ## 3. 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 해결법

   ### 1. ObjectFactory, ObjectProvider

   - ***`provider` 사용 이유*** : 우리가 원하는 의도에 맞게 `객체를 생성 및 사용`하고 싶은 경우 사용한다.

   - Spring은 `ObjectProvider`라는 DL서비스를 제공한다.

     ```java
     @Autowired
     private ObjectProvider<PrototypeBean> prototypeBeanProvider;
     
     public int logic() {
       PrototypeBean prototypeBean = prototypeBeanProvider.getObject(); // 스프링 컨테이너를 통해 해당 빈을 찾아 반환한다.(DL)
       prototypeBean.addCount();
       int count = prototypeBean.getCount();
       return count;
     }
     ```

     - `prototypeBeanProvider.getObject()` : 호출 시, 내부에서 스프링 컨테이너를 통해 해당 빈을 찾아 반새로운 프로토타입 빈을 생성한다.
     - 기능이 단순해 단위테스트를 만들거나 `mock 코드를 작성하기 쉽다.`
     - `ObjectFactory`: 기능이 단순, 별도의 라이브러리 필요 없음, 스프링에 의존 (옛날 거)
     - `ObjectProvider`: ObjectFactory 상속, 옵션, 스트림 처리등 편의 기능이 많고, 별도의 라이브러리 필요 없음, 스프링에 의존

     <br>

   ### 2. JSR-330 Provider

   - `javax.inject.Provider`라는 JSR-330 자바 표준을 사용하는 방법
   - `javax.inject:javax.inject:1` 라이브러리를 gradle에 추가해야 한다.

   ```java
   @Autowired
   private Provider<PrototypeBean> provider;
   
   public int logic() {
     PrototypeBean prototypeBean = provider.get(); // 스프링 컨테이너를 통해 해당 빈을 찾아 반환한다.(DL)
     prototypeBean.addCount();
     int count = prototypeBean.getCount();
     return count;
   }
   ```

   - 자바 표준이므로 `다른 컨테이너에서도 사용이 가능`하다.

   <br>

   ## 4. 웹 스코프

   ### 1. 웹 스코프 종류

   - `request` : 하나의 요청 후, 응답까지의 스코프, HTTP 요청마다 별도의 빈 인스턴스가 생성 및 관리된다.
   - `session` : HTTP Session과 동일한 생명주기
   - `application` : 서블릿 컨텍스트와 동일한 생명주기
   - `websocket` : 웹 소켓과 동일한 생명주기

   <br>

   ### 2. 웹 스코프 예제

   - MyLogger 예제

     ```java
     import javax.annotation.PostConstruct;
     import javax.annotation.PreDestroy;
     import java.util.UUID;
     
     /* MyLogger class */
     @Component
     @Scope(value = "request")
     @Setter
     public class MyLogger {
       public void log(String message) {...}
     }
     
     /* Controller code */
     @Controller
     @RequiredArgsConstructor
     public class LogDemonController {
         private final LogDemonService logDemonService;
         private final MyLogger myLogger;
     		...
     }
     ```

   - request scope는 고객의 요청이 발생해야 생성 및 호출하는 객체이므로 아직 빈을 생성하지 않아 MyLogger부분에서 에러를 발생한다. 이를 해결하는 방법에는 `Provider`를 사용하거나 `Proxy`를 사용해야 한다.

   <br>

   ### 3. Provider를 도입한 Controller

   ```java
   @Controller
   @RequiredArgsConstructor
   public class LogDemonController {
   
       private final LogDemonService logDemonService;
       private final ObjectProvider<MyLogger> myLoggerProvider; //이렇게 변경하자!
   
       @RequestMapping("log-demo")
       @ResponseBody
       public String logDemo(HttpServletRequest request) {
         MyLogger myLogger = myLoggerProvider.getObject();
         String requestURL = request.getRequestURL().toString();
   			...
       }
   }
   ```

   <br>

   ### 4. proxy를 도입한 Controller

   ```java
   @Component
   @Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
   public class MyLogger {
     public void log(String message) {
       System.out.println("[" + uuid + "]" + "[" + requestURL + "] " + message);
     }
   }
   
   /*
    * - MyLogger logging 결과
    *   myLogger = class com.cooper.springcoreprinciple.common.MyLogger$$EnhancerBySpringCGLIB$$20f7e69e
    */
   ```

   - MyLogger에 `가짜 프록시를 생성`하고 HTTP Request에 관계 없이 가짜 프록시 클래스를 빈에 미리 주입할 수 있다!
   - 스프링 컨테이너 실행 시 `$EnhancerBySpringCGLIB`라는 가짜 프록시 객체를 생성해 의존 관계를 주입한다.
   - 그리고 요청을 받을 때, 객체는 내부에 `실제 빈에게 요청하는 위임 로직`이 들어있어 실제 요청이 들어오면 실제 빈의 메서드를 호출(myLogger.logic())한다.

   ![https://s3.us-west-2.amazonaws.com/secure.notion-static.com/ca9fe9c5-0ccd-4611-ac0d-89306ba34b62/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210810%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210810T071612Z&X-Amz-Expires=86400&X-Amz-Signature=cc6d4a7ec3c99ad1259c73f86dfecab477fea49f9af35fe2423d955b8692dccd&X-Amz-SignedHeaders=host&response-content-disposition=filename %3D"Untitled.png"](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/ca9fe9c5-0ccd-4611-ac0d-89306ba34b62/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210810%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210810T071612Z&X-Amz-Expires=86400&X-Amz-Signature=cc6d4a7ec3c99ad1259c73f86dfecab477fea49f9af35fe2423d955b8692dccd&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

   <br>

   ### 5. proxy의 특징

   1. 클라이언트는 동일하게 사용 가능하다. (원본이지 아닌지를 모른다.)
   2. 핵심 아이디어 : 진짜 개체 조회를 꼭 필요한 시점까지 `지연(lazy) 처리`한다는 점이다.
   3. ***어노테이션 설정 변경*** 만으로 원본 객체를 ***프록시 객체로 대체*** 할 수 있다.→ `다형성과 DI 컨테이너의 장점`