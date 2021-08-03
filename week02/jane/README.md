# Spring Bean, Container, ComponentScan

## Application Context

- 스프링 컨테이너
- BeanFactory 인터페이스를 상속하고 있는 인터페이스
  - BeanFactory 외 MessageSource, EnvironmentCapable, ApplicationEventPubilsher, ResourceLoader 상속
- 구현 객체: AnnotationConfig, ApplicationContext
- 빈 이름 = 메서드 이름
  - `@Bean(name="jane")`을 통해 이름 지정 가능
  - 빈의 이름이 중복되면 한 빈이 무시되거나 오류가 발생한다.

- 빈의 생성과 의존관계 주입의 단계가 나누어져 있다.



#### 빈 조회

- 타입으로 조회 시 같은 타입의 빈이 두 개 이상이면 `NoUniqueBeanDefinitionException`가 발생한다.
- 그럴 땐 빈 이름 + 타입으로 조회하면 된다.
- 부모 타입으로 조회하면 자식 타입도 조회된다.

```java
ac.getBean(빈이름, 타입)
ac.getBean(타입)
```



#### 빈 출력

```java
if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
 Object bean = ac.getBean(beanDefinitionName);
 System.out.println("name=" + beanDefinitionName + " object=" +
bean);
```

- ROLE_APPLICATION: 사용자가 직접 등록한 애플리케이션 빈
- ROLE_INFRASTRUCTURE: 스프링이 내부에서 사용하는 빈



## Bean Definition

- 빈 설정 메타정보

```java
beanDefinitionName = appConfig beanDefinition = Generic bean: class [hello.spring.core.AppConfig$$EnhancerBySpringCGLIB$$dd5ce34f]; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
```

- 역할과 구현을 개념적으로 분리
- `AnnotationConfigApplicationContext`는 `AnnotatedBeanDefinitionReader` 를 사용해서 `AppConfig.class`를 읽고 `BeanDefinition`을 생성
  - `ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);`와 같이 설정해주면 `AppConfig.class`가 스프링 빈으로 등록된다.
  - 스프링은 CGLIB(바이트 코드 조작 라이브러리)를 사용하여 `@Configuration` 애너테이션이 붙은 클래스를 상속받은 다른 클래스를 생성한 뒤 해당 클래스를 스프링 빈으로 등록한다. (싱글톤 보장)
    - `bean = class hello.spring.core.AppConfig$$EnhancerBySpringCGLIB$$d6c3b312`

- [destroyMethod]: https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html#destroyMethod--

  를 명시하지 않을 경우 public & no-args로 선언된 `close()`  또는 `shutdown()` 메서드가 있나 확인하고 자동으로 해당 메서드를 destroyMethod로 등록한다. 이 설정을 비활성화 하고 싶은 경우 `@Bean(destroyMethod="")`와 같이 선언해주면 된다.



## Singleton Container

- 싱글톤 패턴의 문제점을 해결하면서 객체 인스턴스를 싱글톤으로 관리하는 컨테이너
  - 싱글톤 패턴의 문제점
    - 클라이언트가 구체 클래스에 의존 (구체클래스.getInstance() 메서드를 통해 구현체를 호출해야 한다)
    - 테스트하기 어렵다.
    - private 생성자로 자식 클래스를 만들기 어렵다.

- 싱글톤 레지스트리: 싱글톤 객체를 생성 및 관리
- 싱글톤 객체는 stateless해야 한다.
  - 클라이언트에 의존적인 필드, 클라이언트가 값을 변경할 수 있는 필드 금지
  - read-only
  - 필드 대신 지역변수, 파라미터, ThreadLocal 사용



## Component Scan

- 설정 정보 없이 자동으로 스프링 빈을 등록하는 기능
- `@Component` 애너테이션이 붙은 클래스를 스캔하여 스프링 빈으로 등록한다.
  - 빈 이름 직접 지정: `@Component("labelService")`

- 탐색 위치 지정

  ```java
  @ComponentScan(
   basePackages = {<패키지 이름1>, <패키지 이름2>}
  }
  ```

- 프로젝트 시작 루트에 `@ComponentScan`을 붙인 파일을 두면 자동으로 설정 파일이 속한 패키지 아래에 있는 파일들이 컴포넌트 스캔의 대상이 된다.
  - `@SpringBootApplication`
- `@Component` 애너테이션을 붙이면 컴포넌트 스캔의 대상이 된다.

