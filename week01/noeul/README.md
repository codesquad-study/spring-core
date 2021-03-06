## IoC(제어의 역전)
- 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것
- 컴포넌트의 의존관계 결정, 생명주기, 설정을 해결하기 위한 디자인 패턴

## DI(의존관계 주입)

> - 스프링 프레임워크가 처음 등장했을 때, 자신들의 강점으로 내세웠던 것 중의 하나가 `IoC`였음
> - 하지만, IoC라는 용어는 너무 폭넓게 사용하고 있어 모호한 측면이 있다는 이유가 있었음
- 이에 마틴 파울러가 `DI`라는 용어로 정리함.
- https://martinfowler.com/articles/injection.html

- 객체 간의 _**의존관계**_를 어떻게 해결(**제거**)하느냐에 따른 접근방식
  - 🛑 유일한 방법이 아님.
- 인스턴스를 생성하는 책임과 사용하는 책임을 **분리**하자는 것


> 의존관계란?
- 객체 혼자 모든 일을 처리하기 힘들기 때문에 내가 해야 할 작업(책임, 역할)을 다른 객체에게 위임하면서 발생


## 왜 DI가 필요한가?
 - 의존관계가 강한 코드는 유연한 개발을 하는데 한계가 있다.
   - 아래 소스코드는 오전, 오후를 체크하는 두개의 테스틀 동시에 통과할 수 없음.
   - 소스코드를 컴파일 하는 시점에 `Calendar` 인스턴스가 확정됨.

```java
public class DateMessageProvider{
	public String getDateMessage(){
    	Calendar now = Calendar.getInsatance(); // 의존관계
        int hour = now.get(Calendar.HOUR_OF_DAY);
        
        if(hour < 12){
        	return "오전";
        }
        return "오후";
    }
}

```

## 해결하려면?
> 외부에서 인스턴스를 생성한 후 전달하는 구조로 바꿔야 한다.(DI)

1. 생성자 주입
2. setter() 메소드를 이용한 의존성 삽입
3. 초기화 인터페이스를 이용한 의존성 삽입
...

---


## 문제는..
- 무엇인가 얻는 것이 있으면 잃는 것이 있다.
- 역할을 분리하다 보니 코드량이 더 많아지고, 고려해야할 부분이 생긴다.

> DI가 무엇이고, 왜 필요한 것인지에 대해 쉽게 공감하기 힘들 수 있다. 이에 대한 깨달음을 얻을 수 있는 가장 좋은 방법은 하나의 서비스를 지속적으로 운영해 보는 경험이 가장 좋다고 생각한다. 하지만 많은 경험과 시간이 필요하다. - 자바 웹프로그래밍 Next Step -


---

## IoC 컨테이너

- 객체에 대한 생성 및 생명주기를 관리할 수 있는 기능 제공

## Spring DI 컨테이너
- Spirng DI 컨테이너가 관리하는 객체를 빈(bean)이라고 한다.
- 이 빈을 관리한다는 의미로 컨테이너를 `빈 팩토리`라고 부름.
- 스프링의 DI 컨테이너는 단순한 DI 작업보다 더 많은 일을 함.
- `빈 팩토리`에 여러 가지 컨테이너 기능을 추가하여 `ApplicationContext`라고도 부름.


---



### Refrence.

- 자바 웹프로그래밍 Next Step
- https://dog-developers.tistory.com/12
- https://www.nextree.co.kr/p11247/