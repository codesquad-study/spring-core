# [week02] Spring Bean, Container, ComponentScan

## 1. Spring Bean

### 1. 빈(Bean)이란?

```
- 스프링 빈이란? DI Container에 등록하는 컴포넌트이다.
- 빈 정의(Bean Definition) : 빈에 대한 설정 정보
- 룩업(lookup) : 빈을 꺼내오기 위해 DI Container에서 빈을 조회하는 작업
```

<br>

### 2. 빈의 생명 주기

```
1. 빈 초기화 단계 (Initialization)
2. 빈 사용 단계 (Activation)
3. 빈 종료 단계 (Destruction)
```

1. 빈 초기화 단계

   - 과정

     | 분류                      | 과정                                                         |
     | ------------------------- | ------------------------------------------------------------ |
     | 빈 설정 읽기 및 보완      | 빈 설정 정보 읽어오기 → 빈 설정 정보 보완 처리               |
     | 빈 생성 및 의존 관계 해결 | Constructor injection → Field Injection → Setter Injection   |
     | 빈 생성 후 초기화 작업    | Bean Post Processor 전처리 → Bean Post Processor 초기화 처리 → Bean Post  Processor 후처리 |

     <br>

   1. `빈 설정 정보 읽기 및 보완` : 빈을 생성하는 데 필요한 정보를 수집한다.

      - Java Configuration
      - XML 기반 설정 파일
      - 어노테이션 기반 컨포넌트 스캔

      <br>

   2. 빈 생성 및 의존 관계 해결

      - 생성자 기반 의존성 주입
      - 필드 기반 의존성 주입
      - 세터 기반의존성 주입

      <br>

   3. 빈 생성 후 초기화 작업

      - @PostConstruct (return type : void , parameter가 없어야 한다.)

        ```java
        @Component
        public class UserServiceImpl implements UserService {
          	@PostConstruct
          	void populateCache() {
              	//캐시 등록
            }
        }
        ```

      - InitializingBean interface

      - @Bean, initMethod 속성에 지정한 메서드 (외부 라이브러리를 사용하는 경우에 장점이 있다.)

        ```java
        @Bean(initMethod = "populateCache")
        USerService userService() {
          return new UserServiceImpl();
        }
        ```

        

        <br>

      - XML기반 설정, \<bean>요소의 init-method 속성에 지정한 메서드

        ```xml
        <bean id = "userService" class = "comexample.demo.UserServiceImpl"
          	init-method="popluateCache">
        ```

        <br>

   4. 종료 단계

      1. 빈이 파괴되기 전에 전처리 수행

         - @PreDestroy

           ```java
           @Component
           public class UserServiceImpl implements UserService {
             	@PreDestroy
             	void clearCache() {
                 //캐시 삭제
               }
           }
           ```

         - DisposableBean

           ```java
           @Component
           public class UserService implements UserService, DisposableBean {
             	
             @Override
             public void destory() {
               	// 캐시 삭제
             }
           }
           ```

           

         - @Bean의 destroyMethod

           ```java
           @Bean(destroy = "clearCache")
           UserService userService() {
             	return new UserServiceImpl();
           }
           ```

           

         - XML 기반 destory-method 속성 지정 메서드

           ```xml
           <bean id="userService" class"com.example.demo.UserSErviceImpl" destroy-method="clearCache"/>
           ```

      
      <br><br>
   
   ## 싱글톤 패턴(Singleton Pattern)
   
   - SpringContainer를 사용하면 객체를 모두 singleton으로 관리한다.
   - 이미 생성한 객체를 공유해서 재사용하므로 성능이 좋아진다.
   
   ```java
   @Test
       @DisplayName("싱글톤 패턴을 적용한 객체 사용")
       void singletonServiceTest() {
           SingletonService singletonService1 = SingletonService.getInstance();
           SingletonService singletonService2 = SingletonService.getInstance();
   
           System.out.println("singletonService1 = " + singletonService1);
           System.out.println("singletonService2 = " + singletonService2);
   
           //isSameAs : 객체의 인스턴스의 참조를 비교하는 메서드
           assertThat(singletonService1).isSameAs(singletonService2);
       }
   }
   isSameAs(); // 동등성을 비교(주소값 참조)
   isEqualsTo(); // 동치성 비교(값의 일치로 확인)
   ```
   
   <br>
   
   ### 싱글톤 패턴 문제점
   
   - 객체를 생성하는 코드의 중복 코드가 발생한다.
   
     ```java
     public class SingletonService {
         private static final SingletonService instance = new SingletonService();
         public static SingletonService getInstance() {
             return instance;
         }
     
         private SingletonService() {
         }
     }
     ```
   
   - 의존 관계상 클라이언트가 구체 클래스에 의존 → DIP를 위반
   
   - 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 위험이 있다.
   
   - 테스트하기 어렵다.
   
   - 내부 속성 변경, 초기화가 어렵다.
   
   - private 생성자로 자식 클래스를 만들기 어렵다
   
   - 결론적으로 유연성이 떨어진다.
   
   - 안티패턴으로 불리기도 한다.
   
   <br>
   
   **하지만, 스프링 컨테이너는 Singleton Pattern이 가지는 문제점을 제거하면서 Singleton 형태로 관리한다.**
   
   <br><br>

## 싱글톤 컨테이너

- 스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 하나만 관리한다.
- 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 `싱글톤 레지스트리`라 한다.
- 스프링 컨테이너의 이런 기능 덕분에 싱글턴의 모든 단점 해결
  - 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
  - DIP, OCP, 테스트, private 생성자로부터 자유롭게 싱글톤을 사용할 수 있다.

[Code]

- MemberService에 singleton pattern이 존재하지 않아도 singleton으로 관리할 수 있다

```java
@Test
@DisplayName("스프링 컨테이너를 적용한 객체 사용")
void springContainer() {
    //AppConfig appConfig = new AppConfig();

    AnnotationConfigApplicationContext ac
            = new AnnotationConfigApplicationContext(AppConfig.class);

    MemberService memberService1 = ac.getBean("memberService", MemberService.class);
    MemberService memberService2 = ac.getBean("memberService", MemberService.class);

    //참조값이 같은 것을 확인
    System.out.println("memberService1 = " + memberService1);
    System.out.println("memberService2 = " + memberService2);

    //memberService1 == memberService2
    assertThat(memberService1).isSameAs(memberService2);
}
```

<br><br>

### 1. 싱글톤의 주의점

- 객체 인스턴스 하나를 공유하므로 여러 클라이언트가 접근할 때 주의해야 한다.
- 왜냐하면, 다른 클라이언트의 상태를 유지할 경우, 다른 사용자의 결과가 달라질 수 있기 때문이다.
- 그러므로, 빈을 관리할 때는 무상태(stateless)로 관리해야 한다.

### 2. 상태유지(Stateful) 테스트

- ThreadA가 사용자 A 호출, ThreadB가 사용자 B 호출

- StatefultService의 price필드는 공유되는 필드 → order메서드로 특정 클라이언트의 값을 변경한다.

  ```java
   public void order(String name, int price) {
          System.out.println("name = " + name + ", price = " + price);
          this.price = price; //여기가 문제
      }
  ```

- 꼭 스프링 빈은 항상 `무상태(Stateful)`로 설계하자! (공유필드 정말 조심!!)

  ```java
  public int order(String name, int price) {
          System.out.println("name = " + name + ", price = " + price);
          return price;
      }
  ```

<br><br>

스프링 컨테이너는 `싱글톤 레지스트리`다. 따라서 스프링 빈 싱글톤을 보장해주어야 한다.

이전에 MemberRespository를 한번만 호출하는 것을 확인하기 위해 클래스의 바이트코드를 조작하는 라이브러리(`CGLIB`)를 한다. 모든 비밀은 `@Configuration`에 있다!!

[test code]

```java
@Test
    void configurationDeep() {
        AnnotationConfigApplicationContext ac
                = new AnnotationConfigApplicationContext(AppConfig.class);
        AppConfig bean = ac.getBean(AppConfig.class);

        System.out.println("bean = " + bean);
    }
```

[결과]

```java
bean = com.cooper.springcoreprinciple.config.AppConfig$$EnhancerBySpringCGLIB$$111e82b7@2cf3d63b
```

MemberRepository가 3개를 호출할거라고 예상했지만,  예상과 다르게 클래스 명에 `xxxCGLIB`가 붙으면서 상당히 복잡해진 것을 확인할 수 있다.

이것은 스프링이 CGLIB라는 바이트 코드 조작 라이브러리를 사용해서 AppConfig를 상속받은 임의의 다른 클래스를 만들고, 다른 클래스를 스프링 빈으로 등록한 것이라고 한다. (아래는 예시 그림!)

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/5d2f22a8-843e-4e25-a0f9-04349894f49f/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210803%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210803T110400Z&X-Amz-Expires=86400&X-Amz-Signature=1c071232f0ff9c55b804b9d16d1ba5d84d4438ab1f79e367c1f408c455fcbbf5&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

그러면 만약 ***@Configuration을 처리하지 않고 @Bean을 사용할 경우\*** 어떻게 될까?

- AppConfig의 **`@Configuration`**을 제거하고 테스트를 실행해보면

  ```java
  memberRepository1 = com.cooper.springcoreprinciple.member.infra.MemberMemoryRepository@24fcf36f
  memberRepository2 = com.cooper.springcoreprinciple.member.infra.MemberMemoryRepository@10feca44
  memberRepository = com.cooper.springcoreprinciple.member.infra.MemberMemoryRepository@3fb1549b
  ```

싱글톤이 깨지는 것을 확인할 수 있다ㅠㅠ

[**결론]**

> ***@Configuratoin을 붙이면\***  바이트코드를 조작하는 CGLIB기술을 사용해 ***싱글톤을 보장\*** 한다!(***@Configuration없이*** ***@Bean\*** 만 사용하면, ***스프링 빈으로 등록\*** 되지만 ***싱글톤 보장하지 않는다\***!)

<br><br>

- 개발자는 반복을 싫어한다.. → 그래서 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.

- `@ComponentScan` : @Component annotation을 표시한 모든 클래스를 스프링 빈으로 등록한다.

  1. option : 예외할 클래스 설정을 추가할 수 있도록 한다.

     ```java
     @ComponentScan(
             excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
     )
     public class AutoAppConfig {
     }
     ```

  2. AppConfig의 @Configuration에도 `@Component` annotation이 추가되어 있기 때문에 해당 클래스의 스프링 빈 등록을 제외시킨다.

- 그렇다면 의존성 주입은 어떻게 해줄지??

  Ans - 생성자에 @Autowired를 추가해 parameter에 표시된 클래스를 스프링 빈에 등록되어 있는지 확인하고, 해당 클래스의 의존성을 주입한다.

(+ 내 프로젝트 진행할 때는 작업을 하는데 스프링빈을 못 불러오는 어려움이 있었다.)

<br><br>

### 1. ComponentScan에서 같은 빈 이름을 등록하면?

- caseA. 자동 빈 등록 vs 자동 빈 등록
- caseB. 수동 빈 등록 vs 자동 빈 등록

### 2. 케이스에 따른 충돌

1. **caseA. 자동 빈 등록 vs 자동 빈 등록**

   ```java
   @Component("service")
   class MemberServiceImpl {...}
   
   @Component("service")
   class OrderServiceImpl {...}
   ```

   - 컴포넌트 스캔으로 자동으로 스프링 빈을 등록할 경우, 이름이 같으면 스프링에서 오류를 발생시킨다.
     - 발생 예외 : `ConflictBeanDefinitionException`

2. **caseB. 수동 빈 등록 vs 자동 빈 등록**

   ```java
   @Component //자동 빈 등록
   public class MemoryMemberRepository {...}
   
   @Configuration
   class AutoAppConfig {
   
   	@Bean(name = "memmoryMemberRepository")
   	MemberRepository memberRepository() {
   		return new MemoryMemberRepository();
   	}
   }
   ```

   - ***수동 빈이 우선권*** 을 갖는다. = ***수동 빈으로 재정의(overriding)*** 한다.

   - 수동빈이 우선권을 가지는 것이 좋지만 실제로는 여러 설정들이 꼬여서 발생하는 경우가 많다.

     이 경우, 정말 잡기 어려운 ***애매한 버그\*** 이다.

   - 최근 스프링 부트에서는 ***수동 빈 등록과 자동 빈 등록이 충돌나면 오류가 발생\*** 하도록 기본 값을 변경했다.

     (그래서, 스프링 빈이 중복된 상태로 실행시키면, 에러가 발생하고 override를 원할 경우, 옵션을 추가하도록 안내해준다.)

     ```
     //충돌 시, 에러 메세지
     Consider renaming one of the beans or enabling overriding by setting
     spring.main.allow-bean-definition-overriding=true
     ```

     ```java
     spring.main.allow-bean-definition-overriding=true # overrding 허용
     ```



## 생성자 주입하라.

- 불변

  1. 의존관계 주입은 한번 일어나면 애플리케이션 종료 시점까지 의존 관계를 변경할 일이 없다. 대부분 의존 관계는 애플리케이션 종료 전까지 변하면 안된다.

  2. 다른 사람이 실수로 변경할 위험이 있다.

  3. 생성자 주입은 객체를 생성 시, ***1번만 호출하고 이후에 호출하지 않는다\***. 고로 불변하게 설계할 수 있다.

  4. 필드에 final keyword를 추가해 객체를 변경할 수 없고, 실수로 생성자에 값이 설정하지 않을 경우 컴파일 시점에서 막아준다.

     ```java
     private final MemberRepository memberRepository;
         private final DiscountPolicy discountPolicy;
     
         @Autowired
         public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
             this.memberRepository = memberRepository;
             //this.discountPolicy = discountPolicy; //없으면 컴파일에러
         }
     ```

     > 수정자 주입을 포함한 나머지 주입 방식은 모두 생성자 이후에 호출되므로, 필드에 `final`키워드를 사용할 수 없다.

- 누락

  1. 프레임워크 없이 순수한 자바 코드를 단위 테스트하는 경우

  - 생성자 주입을 하는 경우, NullPointException을 반환한다.

    ```java
    class OrderServiceImpl {
    //내부 필드를 꼭 주입해주어야 한다.
    	private final MemberRepository memberRepository;
    	private final DiscountPolicy discountPolicy;
    }
    
    @Test
    void orderServiceTest() {
    		OrderService orderService = new OrderServiceImpl(new MemberReposotiry(), new FixDiscountPolicy());
    }
    ```

- 결론!

  - 생성자 주입 방식은 여러가지가 있지만, 프레임워크에 의존하지 않고, 순수한 자바 언어의 특징을 잘 살리는 방법이기도 하다.
  - 항상 생성자 주입을 선택해라! 옵션이 필요한 경우 수정자 주입을 선택해라. 필드 주입은 절대 사용 금지.

<br><br>

## 조회 빈이 2개 이상 - 문제

### 1. 빈 충돌 발생할 경우!

- 만약 같은 하위 타입이 Component로 빈 등록이 될 경우, `NoUniqueBeanDefinitionException`이 발생한다.

(= 하나의 빈을 기대했는데 fixDiscountPolicy와 rateDiscountPolicy 2개가 발견되었다고 알려준다.)

```java
@Component
public class FixDiscountPolicy implements DiscountPolicy {...}

@Component
public class RateDiscountPolicy implements DiscountPolicy {...}

//이렇게 의존관계 자동 주입을 실행할 경우,
@RequiredArgsConstructor //생성자 주입에서 discountPolicy를 자동 주입할 때, 충돌이 발생한다.
@Component
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
		
}
```

### 2. 해결 노력??

- 하위 타입으로 @Component를 지정할 수 있지만 하위 타입으로 지정하는 것은 DIP를 위배하고 유연성이 떨어진다.
- 스프링 빈을 수동 등록해도 되지만, 의존 관계 자동 주입에서 해결하는 여러 방법이 있다.

<br><br>

## @Autowired 필드 명, @Qulifier, @Primary   

### 조회 대상 빈이 2개 이상일 때, 해결 방법

1. @Autowired 필드 명 매칭
2. @Qulifier → @Qualifier끼리 매칭 → 빈 이름 매칭
3. @Primary 사용

### 1. @Autowired 필드 명 매칭

- @Autowired는 우선 `타입 매칭을 시도`하고, 만약 여러 빈이 있으면 `필드 이름, 파라미터 이름`으로 빈 이름을 추가 매칭한다.

```java
//기존 코드
@Autowired
private discountPolicy discountPolicy

//필드 명을 빈 이름으로 변경
private DiscountPolicy rateDiscountPolicy
```

이렇게 코드를 작성할 경우, 필드명이 rateDiscountPolicy인 class가 정상 주입된다.

**@Autowired 매칭 정리**

1. 타입 매칭 결과가 2개 이상일 때, 필드 명으로 빈 이름 매칭

### 2. @Qualifier 사용

- @Qualifier : 추가 구분자 역할

- 사용방법 : 빈 등록시 @Qualifier을 붙여준다.

  ```java
  //1. 빈 등록 시, @Qualifier을 붙여 준다.
  
  @Component
  @Qualifier("rateDiscountPolicy")
  public class RateDiscountPolicy implements DiscountPolicy {...}
  
  @Component
  @Qualifier("fixDiscountPolicy")
  public class FixDiscountPolicy implements DiscountPolicy {...}
  
  //2. 주입시에 @Qualifier를 붙여주고 등록한 이름을 적어준다.
  @Component
  public class OrderServiceImpl implements OrderService {
  
      private final MemberRepository memberRepository;
      private final DiscountPolicy discountPolicy;
  
      public OrderServiceImpl(MemberRepository memberRepository, 
                              @Qualifier("rateDiscountPolicy") DiscountPolicy discountPolicy) {
          this.memberRepository = memberRepository;
          this.discountPolicy = discountPolicy;
      }		
  		...
  }
  ```

- 만약 lombok @RequiredArgsConstructor와 같이 사용하고 싶다면?

  1. field명을 @Qualifier에 명시한 이름과 일치시키거나,

     ```java
     @RequiredArgsConstructor
     @Component
     public class OrderServiceImpl implements OrderService {
     
         private final MemberRepository memberRepository;
         private final DiscountPolicy rateDiscountPolicy;
     ```

  2. 프로젝트 최상단에 `lombok.config`를 생성해서 다음 코드를 추가한다.

     ```java
     lombok.copyableAnnotations += org.springframework.beans.factory.annotation.Qualifier
     ```

     - 혹시 모르니 build clean 실행해서 기존 class 파일을 삭제하기

  **@Qualifier 정리**

  1. @Qualifier끼리 매칭

  2. 빈 이름 매칭

  3. NoSuchBeanDefinitionException 예외 발생

### 3. @Primary 사용하기

- `@Primary` : 우선 순위를 정하는 방법, @Autowired 시에 여러 빈 매칭되면 @Primay가 우선권을 가진다.

  ```java
  @Component
  @Primary
  public class RateDiscountPolicy implements DiscountPolicy {...}
  
  @Component
  public class FixDiscountPolicy implements DiscountPolicy {...}
  ```

- 데이터 베이스와 커넥션과 연관된 정보를 사용할 때는 @Primary를 사용하고 보조 데이터 베이스 커넥션 빈을 획득할 때는 @Qualifier를 사용하면 코드를 깔끔하게 구현할 수 있다.

### 4. 우선순위

- 스프링은 넓은 범위의 선택권보다 좁은 범위의 선택권이 우선 순위가 높다.
- 결론 : `@Qualifier > @Primary`

<br><br>

## 자동, 수동, 올바른 실무 운영 기준

### 1. 편리한 `자동 기능`을 기본으로 사용하자.

- 어떤 경우에 컴포넌트 스캔과 자동 주입을 사용하고, 어떤 경우에 설정 정보를 통해서 수동으로 빈을 등록하고, 의존관계도 수동으로 주입해야 할까?

이유1. **`간편함`**

1. 개발자 입장에서는 스프링 빈을 하나 등록할 때,
   - 자동 주입 단계
     1. @Component만 넣어주면 끝
   - 수동 주입 단계
     1. @Configuration 설정 정보가서
     2. @Bean을 적고 객체 생성하고
     3. 주입할 대상을 적는 과정을 일일이 적는다.

이유2. `SOLID원칙`

- 자동 주입으로 설정해도 SOLID원칙을 지킬 수 있다.

### 2. 수동 빈 등록은 어제 사용할지?

- `업무 로직 빈`

  1. 컨트롤러, 서비스, 레포지토리 등에 해당하는 로직이다.
     - 컨트롤러 : 웹을 지원
     - 서비스 : 핵심 비즈니스 로직
     - 레포지토리 : 데이터 계층의 로직을 처리
  2. 보통 요구사항에 따라서 추가 혹은 변경되는 요소이다.
  3. 업무 로직은 숫자도 많고, 한번 개발해야 하면 컨트롤러, 서비스, 레포지토리처럼 유사한 패턴이 존재하므로 `자동 주입`을 사용하는 것이 좋다.

- `기술 지원 빈`

  1. 기술적인 공통 관심사(AOP)를 처리할 때 주로 사용된다.

     - ex. DB 연결, 공통 로그 처리

  2. 업무 로직을 지원하기 위한 하부 기술이나 공통 기술.

  3. 기술 지원 로직은 업무 로직에 비해 그 수가 매우 적고, 애플리케이션 전반에 광범위하게 영향을 미친다.

     또한, 기술 지원 로직은 적용이 잘되고 있는지 아닌지 파악하기 어려운 경우가 많아서 가급적 `수동 빈`으로 관리하는 것이 좋다.

  ***→ 애플리케이션에 광범위하게 영향을 미치는 기술 지원 객체는 수동 빈으로 등록해서 설정 정보에 나타나게 하는 것이 유지보수에 좋다.***

## 3. 업무로직에 수동 빈을 사용해야할 경우에는?

- 빈을 List, Map으로 관리해야할 경우에 사용하는 것이 좋다. Map<String, DiscountPolicy> 형태로 관리할 때, 빈들의 상태(주입, 이름)을 한번에 파악하기 쉽게하기 위함이다.

  ```java
  @Configuration
  public class DiscountPolicyConfig {
  		@Bean
  		public DiscountPolicy rateDiscountPolicy () {
  				return new RateDiscountPolicy();
  		}
  	
  		@Bean
  		pubilc DiscountPolicy fixDiscountPolicy () {
  				return new FixDiscountPolicy();
  		}
  }
  ```

- ***수동 빈 등록 또는 자동으로하면 `특정 팩키지에 같이 묶어 관리`하는 것이 중요하다! 핵심은 딱보고 이해가 되어야 한다!***

<br><br>

## 4. 스프링과 스프링 부트가 자동으로 등록하는 수 많은 빈들은 예외

- 예시로 `DataSource`와 같은 DB 연결 지원 로직까지 내부에서 자동으로 등록하는데, 이런 부분은 메뉴얼을 잘 참고해서 ***`스프링 부트가 의도한 대로 편리하게 사용`\*** 한다.
- 바면 ***`내가 직접 기술 지원 객체`\*** 를 스프링 빈으로 등록한다면 ***`수동으로 등록`\*** 해서 명확하게 드러내는 것이 좋다!
