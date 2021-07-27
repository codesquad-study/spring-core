# IoC, DI, Container

## Spring의 핵심
- 자바 언어 기반 프레임워크
- EJB 종속적으로 개발할 경우 자바 언어의 특징인 객체 지향이 주는 장점을 살리기 힘들었고, POJO로 돌아가자는 흐름이 생겨났었다. 그러나 스프링은 자바가 가진 객체 지향적 특징을 살려 좋은 객체 지향 애플리케이션을 개발할 수 있도록 도와준다.


## 역할과 구현의 분리
- 역할과 구현을 분리하여 설계하면 자유롭게 구현 객체(저장소, 할인 정책 등)를 변경할 수 있다. 
- DIP를 지키려면(= 추상화에만 의존하려면 = 인터페이스에만 의존하도록 설계를 하려면) 클라이언트에 인터페이스의 구현 객체를 대신 생성하고 주입해줄 대상이 필요하다.
- config 클래스를 애플리케이션의 전체 동작 및 구성을 책임지는 기획자라고 생각해보자. config 클래스에서 구현 객체를 생성하고 연결하면, 정책 변경 등에 사용 영역은 영향을 받지 않고 config가 속한 구성 영역만 영향을 받는다.
- 스프링 컨테이너가 `@Configuration` 애너테이션이 붙는 클래스를 구성 정보로 사용하여 객체를 생성하고 DI까지 대신 해주며, 컨테이너에 등록된 객체를 스프링 빈이라고 부른다.

## IoC (Inversion of Control)
- 프로그램의 제어 흐름 구조가 역전되는 것(직업 제어 &rarr; 외부에서 관리)
- 원래는 오브젝트가 자신이 사용할 클래스를 직접 결정하고 언제 & 어떻게 오브젝트를 생성할지를 스스로 제어하지만, 제어 흐름이 역전된 상황에서는 오브젝트를 사용하는 쪽에서 제어 권한을 위임받아 오브젝트를 제어하게 된다.
- e.g. 서블릿, 디자인 패턴, 프레임워크
  - 프레임워크: 애플리케이션의 코드는 프레임워크가 정의한 틀 안에서 수동적으로 동작한다. 

## DI
- 런타임에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결 되는 것
- 정적인 클래스 의존관계 변경 없이, 동적인 객체 인스턴스 의존관계를 변경할 수 있다.

## IoC / DI 컨테이너
- 오브젝트를 생성하고 관리하면서 의존관계를 연결해 주는 것
- 빈: 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트
- 빈 팩토리: 빈의 생성과 관계 설정 등의 제어를 담당하는 IoC 오브젝트
- 애플리케이션 컨텍스트: IoC 방식을 따라 만들어준 빈 팩토리
    - 직접 설정 정보를 담고있다기 보다는 외부에서 설정 정보(오브젝트 생성, 연결 등에 관련된 정보)를 가져와 활용하는 범용 IoC 엔진이라고 생각하면 된다.

### 애플리케이션 컨텍스트
- 싱글톤을 저장하고 관리하는 싱글톤 레지스트리
- 싱글톤으로 만드는 이유?
  - 스프링의 적용 대상이 자바 엔터프라이즈 기술을 사용하는 서버 환경이기 때문
  - 클라이언트 요청마다 새로운 오브젝트를 생성한다면 GC의 성능이 좋더라도 서버에 부하가 걸린다.
  - 따라서 서블릿 클래스 당 하나의 오브젝트만 생성하고 여러 스레드에서 하나의 오브젝트를 공유하여 사용하는 것이 좋다.

### 1주차 스터디 내용 정리
- ApiResponse는 상태 코드 200 안에 에러를 wrapping해서 보내는 방식인 반면, ResponseEntity를 사용할 경우 실제 상태 코드로 응답을 할 수 있다.
- [Spring HATEOAS - Reference Documentation](https://docs.spring.io/spring-hateoas/docs/current/reference/html/)
- [스프링 HATEOAS 개념 및 적용](https://engkimbs.tistory.com/866)
- [Value vs Environment](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/env/Environment.html)
  - 공식 문서를 보면 애플리케이션 레벨의 빈은 Environment랑 직접적으로 소통하기보다는 `@Value` 등을 통해 property value를 주입받는 것이 좋다고 한다. 
`In most cases, however, application-level beans should not need to interact with the Environment directly but instead may have to have ${...} property values replaced by a property placeholder configurer such as PropertySourcesPlaceholderConfigurer, which itself is EnvironmentAware and as of Spring 3.1 is registered by default when using <context:property-placeholder/>.
`
- [생성자 주입을 해야 하는 이유](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)
   - 객체의 불변성 확보
      - 필드 객체에 final 키워드를 선언하여 컴파일 시점에 누락된 의존성을 확인할 수 있다. 
   - 순수한 자바 코드를 이용한 단위 테스트
      - 필드 주입의 경우 스프링 컨테이너를 통해 DI를 하기 때문에, 순수 자바 코드를 이용한 단위테스트에서 DI가 되지 않아 NPE가 발생한다.
      - 그러나 생성자 주입을 이용하면 컴파일 시점에 객체를 주입받아 테스트코드를 작성할 수 있다.
   - 생성자 주입을 사용하면 객체의 생성 시점(애플리케이션 구동 시점)에 순환 참조 에러를 잡을 수 있다.
       - 필드 주입의 경우 먼저 빈을 생성한 뒤에 `@Autowired` 애너테이션이 붙은 필드에 해당하는 빈을 찾아서 주입한다. 
       - 생성자 주입의 경우 생성자로 객체를 생성하는 시점에 빈을 주입한다. 


