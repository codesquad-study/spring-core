# 스프링 컨테이너와 스프링 빈



## 스프링 컨테이너 생성

- ApplicationContext 가 스프링 컨테이너다.
  - 스프링 컨테이너에는 각 빈을 참조할 수 있는 리스트를 가지고 있는데, 스프링 컨테이너를 만들 때 파라미터로 넘긴 `AppConfig`와 같은 구성 클래스내의 `@Bean` 메소드를 저장한다.
  - 그후 각 객체간 의존관계를 맺어주고 의존관계 주입을 한다.
  - 이게 지금 코드는 빈을 등록하면서 의존관계를 생성하는 구조라고 하시는데, 이후에 어떤방식으로 될지 궁금하다.



## 컨테이너에 등록된 모든 빈 조회

- 스프링 컨테이너에 등록된 빈을 조회하는 코드를 작성하셨는데, 지금은 쓸일이 없는 것 같아서 그냥 편하게 봤다.



## 스프링 빈 조회 - 기본

- 이전 강의랑 크게 다를게 없었다.. 그냥 빈을 이름으로 조회하고 타입으로 조회하고 구체타입으로 조회하고...지금은 그냥 스프링 컨테이너에서 빈을 꺼내올 수 있다는 정도만 알고, 필요할 때 찾아보면 될 것 같다..



## 스프링 빈 조회 - 동일한 타입이 둘 이상

- 스프링 컨테이너에 동일한 타입의 빈이 있을때 조회하는 법에 대해 강의를 해주셨다.
- 일반적인 getBean(타입)으로 찾게 될 경우,  모호하므로 에러가 발생한다.
  - 해결책(택1)
    1. 메소드명도 명시해준다.
    2. getBeansOfType()을 통해 동일한 타입의 Bean을 Map으로 받는다.



## 스프링 빈 조회 -  상속관계

- 스프링 빈은 부모타입을 조회시 자식타입까지 모두 조회되므로, 중복 예외에 유의해야한다.



## BeanFactory와 ApplicationContext

- 빈팩토리와 ApplicationContext를 스프링 컨테이너라고 한다.
- ApplicationContext 인터페이스의 조상이 BeanFactory다.
  - 국제화를 위한 메시지 소스, 환경변수 등.. 추가 편의 기능이 추가됨.
- 아직은 어려우니까, 그냥 둘다 스프링 컨테이너다! 라고 이해하고 넘어가자.



## 다양한 설정 형식 지원 - 자바 코드. XML

- 스프링 컨테이너는 다양한 형식의 설정 정보를 받을 수 있다.

```java
// appConfig.java
...
@Bean
public MemberService memberService(){
    return new MemberServiceImpl(memberRepository);
}

@Bean
public MemberRepository memberRepository(){
    return new MemoryMemberRepository();
}
```



```xml
<!-- appConfig.xml -->
...
    
<bean id="memberService" class="hello.core.member.MemberServiceImpl">
	<constructor-arg name="memberRepository" ref="memberRepository" />
</bean>
    
<bean id="memberRepository" class="hello.core.member.MemoryMemberRepository" />

</beans>

```

- GenericXmlApplicationContext(파일명) 으로 스프링 컨테이너 등록
- 그외 이전 스프링 컨테이너 활용 예제와 같음



## 스프링 빈 설정 메타 정보 - BeanDefinition (복습 필요~!)



- 스프링은 다양한 설정 형식을 BeanDefinition으로 스프링 메타정보를 추상화해 구현했다.
- 스프링 빈 만드는 법
  - 직접적으로 스프링 빈 등록
  - 팩토리빈(ex) AppConfig)
- 스프링 컨테이너는 설정 파일이 자바 코드인지 XML인지 알필요가 없다.

- 아직 뭔지 잘 모르겠다 ㅋㅋ 그냥 이런게 있다정도로 ㅎㅎ



---

# 싱글톤 컨테이너



## 웹 애플리케이션과 싱글톤

- 현재 스프링 컨테이너, AppConfig 클래스를 확인해 보면 요청시 마다 새로운 객체를 생성하고 있는데, 이건 웹의 경우 동시에 여러 유저가 요청을 하게되는데, 그럼 동기화 문제라던가 객체를 계속 생성하는 비용이 발생하는 비효율적인 문제점이 있다.

-  그럼 어떻게 해야할까? 디자인 패턴 중 객체를 한번만 생성해서 공유하는 싱글톤 패턴을 사용해야한다.

## 

## 싱글톤 패턴

- 1개의 객체 인스턴스 보장

- 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있음.

- 문제점: 

  - 전역상태이어서 추적하기 어렵고, 

  - 상속할 수 없어 다형성과 같은 객체지향 특징에 맞지 않음. 

  - 꼭 1개만 생성한다고 완전히 보장하진 못한다. (reflection..)

  - 등..

    

## 싱글톤 컨테이너

- 스프링 컨테이너는 싱글톤 패턴의 `문제점`을 해결하면서, 객체 인스턴스를 싱글톤으로 관리
  - `싱글톤 레지스트리`
  - @Bean이 붙은 메소드는 싱글톤으로 관리됨.
  - 생성자가 private이 꼭 아니어도 됨. 등..

- 스프링은 기본적으로 빈 등록 방식은 싱글톤 방식이지만, 싱글톤만 지원하는 것은 아니다. (빈 스코프)



## 싱글톤 방식의 주의점

- 공유해서 사용하다보니, 스프링이던 싱글톤 패턴이던 항상 `무상태`로 설계 해야함.

  

## @Configuration과 싱글톤

- 흐름만 따라가면 MemberRepository가 복수개 호출, 생성됨 
- 싱글톤이 깨질 것 같은데 테스트 결과는 그렇지 않았음. (모두 동일한 인스턴스)

```java
@Bean
public MemberService memberService() {
return new MemberServiceImpl(memberRepository());
}
@Bean
public OrderService orderService() {
	return new OrderServiceImpl(memberRepository(),discountPolicy());
}
}
@Bean
public MemberRepository memberRepository() {
	return new MemoryMemberRepository();
}
	
```



## @Configuration과 바이트코드 조작의 마법

- 스프링이 `CGLIB`라는 바이트코드 조작 라이브러리로 **`*@Configuration`을 붙인 클래스***를 상속받아 `임의의 다른 클래스`를 만들어서 빈으로 등록해줌.
- `@Bean`이 붙은 메서드마다 동적으로 스프링 빈이 존재하면 존재한 빈을 반환하고, 없으면 새로 생성해서 빈으로 등록
  -  이런 메커니즘으로 싱글톤을 보장해주는 것
- @Configuration을 안붙이면 우려했던 대로 호출마다 객체가 생성됨.
  - 순수한 자바코드가 돼서, 싱글톤을 보장받지 못함.
  - 해결책으로, `@Autowired`를 통해 의존관계 주입을 해주면 싱글톤을 보장 받을 수 있다.

---

# 컴포넌트 스캔

- 아직 못들었어요 ㅎㅎ