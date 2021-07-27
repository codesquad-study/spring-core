## 1. DI, IoC + DI Container, ApplicationContext

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/e3ba5e8b-4aa8-45ee-ac24-6c7a9cbe1e33/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210726%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210726T150622Z&X-Amz-Expires=86400&X-Amz-Signature=170e309d0528b4ed40c2ffa82fb290e454ce010ddf249af804d7ec2c16f741ae&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

DI를 사용하는 이유는 무엇일까? 보통 어플리케이션을 개발할 때, `하나의 처리를 수행하기 위해 여러 개의 컴포넌트를 조합해 구현`해야 한다. 컴포넌트는 기능에 따라 분리한 요소들도 생각하면 쉽다. DB 접근 컴포넌트, 외부 시스템 접속 컴포넌트 등 다양한 컴포넌트들이 존재한다.

그렇다면 여러 개의 컴포넌트는 어떻게 조합할까? 스프링은 `DI(Dependency Injection, 의존 관계 주입)`을 통해서 컴포넌트를 조합하고 있다. DI란 어떤 클래스를 필요로 하는 컴포넌트를 외부에서 생성하여 필요한 위치에 주입하는 방식을 일컫는 말이다. DI는 IoC라고 하는 소프트웨어 디자인 패턴 중 하나이다.

`IoC(Inversion of Control)`는 ***인스턴스 제어 주도권이 역전됐다*** 는 의미이다. 그 이유는 기존에 객체를 생성하는 방식은 **소스 코드 내부에서 객체를 생성하는 방식**이었다. 그런데 IoC는 기존의 방식과 다르다. 객체를 생성하는 장소가 존재하고 객체를 생성한 다음, 필요한 곳에 해당 객체를 전달(주입)한다.  객체를 생성하는 장소를  `DI Container`라 부른다.

DI Container는 객체를 생성해서 해당 위치로 의존 관계를 주입하는 역할을 한다. 스프링에서는 DI Container 역할은 `ApplicationContext`담당한다. ApplicationContext는 일련의 과정을 통해 빈을 관리하고 원하는 위치에 빈을 주입하는 역할을 한다.

<br>

## 2. **DI 컨테이너의 간단한 빈 관리 및 주입과정**

1. **먼저 컴포넌트를 등록**한다. 등록한 컴포넌트는 ApplicationContext 인스턴스를 통해 가져올 수 있으며 스프링 프레임워크은 DI Container에 등록하는 컴포넌트를 `빈(Bean)`, 이 빈에 대한 설정 정보를 `빈 정의(Bean Definition)`, 빈을 꺼내오기 위해 DI Container에서 빈을 조회하는 작업을 `룩업(lookup)`이라 부른다.
2. **의존성을 주입한다**. 의존성을 주입하기 위해서는 해당 빈이 ApplicationContext에 등록되어 있어야 하며 존재하지 않은 경우, `NoUniqueBeanDefinitionException`을 발생한다. 의존성을 주입하는 법은 3가지 방법(setter, constructor, field)이 있으며, 빈을 주입하는 방식에는 수동 방식(@Bean, XML)과 자동 방식(@Autowired)가 있다
3. 의존성을 주입하는데 몇가지 예외 상황이 발생한다. 객체의 타입이 같은 경우, 객체 이름이 같은 경우 등등 겹치는 상황을 대비해 여러가지 방법을 통해 빈을 관리하고 의존성을 주입한다.

<br><br>

## 3. OCP와 Strategy Pattern

왜 OCP를 지켜야 할까? 만약 OCP를 지키지 않고 client가 다양한 방식으로 encoding하고 싶은 경우, 모든 경우에 대한 코드를 작성해야 한다. 만약 태권브이와 썬가드는 둘다 미사일로 공격한다고 가정해보자. 만약 아래와 같이 설계했을 경우, attack()메서드에 같은 로직을 중복해서 작성해야 한다. 이런 중복 설계가 많아지고 중복 코드가 많아지면 코드를 수정해야 하는 경우, 한군데만 수정해야 하는 것이 아니라 같은 로직의 코드를 일일이 코드를 수정해야 한다. 유지 보수가 어렵다.

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/a665391f-c28e-460e-aa6e-9af4eedc45b0/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210727%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210727T081955Z&X-Amz-Expires=86400&X-Amz-Signature=9163d019e166a6638ee40ad2e030108c19074b427b3d73ac57c6ce3416c20023&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="60%">



```java
abstract class Robot {
    private String name;

		...

    public abstract void attack();
    public abstract void move();
}

class Atom extends Robot {
		...

		//중복 발생1! (미사일로 공격한다!)
    @Override
    public void attack() {
        System.out.println("I have strong punch and can attack with it.");
    }

    @Override
    public void move() {
        System.out.println("I can fly");
    }
}

class TaekwonV extends Robot {
		...

    @Override
    public void attack() {
        System.out.println("I have Missile and can attack with it.");
    }

		//중복 발생2! (날으는 기능!)
    **@Override
    public void move() {
        System.out.println("I can only fly");
    }**
}

class TaekwonV extends Robot {
		...

    @Override
    public void attack() {
        System.out.println("I have Missile and can attack with it.");
    }

    **@Override
    public void move() {
        System.out.println("I can only walk");
    }**
}

class Sungard extends Robot {
		...
		
		//중복 발생1! (미사일로 공격한다!)
    **@Override
    public void attack() {
        System.out.println("I have strong punch and can attack with it.");
    }**

		//중복 발생2! (날으는 기능!)
    **@Override
    public void move() {
        System.out.println("I can fly");
    }**
}
```

OCP는 한마디로 말해서 `변경없이 기능 확장이 가능`해야 한다. 그렇기 위해 상위 interface로 추상화하고 하위 클래스로 구현체를 생성하는 구조로 설계를 해야한다.  이를 나누는 기준은 `변경 요소`이다. `변하는 것과 변하지 않은 것을 구분`해야 한다. 변하지 않는 것을 인터페이스로 정의하고 자주 변할만한 요소들은 구현체에 정의하는 방식으로 접근해야 한다.  그렇다면 앞서 로봇을 구현하는 코드에서 다양한 client의 요구사항을 반영할 수 있는 방법은 없을까?? 현재까지 학습한 내용을 기반으로 해결 가능한 방법은 `전략 패턴(stategy pattern)`을 적용하는 것이다.

<br>

### 2. Strategy Pattern : 스트래티지 패턴

스트래티지 패턴은 같은 문제를 해결하는 여러 알고리즘이 클래스별로 캡슐화되어 필요할 때 교체 가능한 패턴이다. 전략 패턴이라고 명명한 이유는 말그대로 ***전략을 쉽게 변경할 수 있도록 하는 디자인 패턴\*** 이기 때문이다. 여기서 전략은 비즈니스 규칙, 문제를 해결하는 알고리즘을 의미한다.

<br>

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/4a61a07a-9ec2-4d86-9dfe-7d092345f7aa/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210727%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210727T081911Z&X-Amz-Expires=86400&X-Amz-Signature=dd5e0b86c4b858e80e6712fe6e950c1d14da381efd69d998ae4f2558391126d5&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="80%">

로봇은 공격을 한다는 공통점을 가지고 있지만 공격을 하는 방식은 다를 수 있다. 어느날 갑자기 아톰의 공격 방식이 펀치에서 미사일을 사용하고 싶은 경우를 생각해보자. 공격의 본질은 그대로 이지만 공격 방식이 변경됐다. 이전의 코드의 경우, 아톰의 attack()메서드를 변경하면 아톰 클래스를 찾고 attack()메서드를 찾아 변경해야 하는 번거로움이 있다. 이를 해결하고자하는 것이 `전략패턴`이다. 편하게 펀치 공격 클래스를 미사일 공격 클래스로 변경만 해주면 된다. 공격을 인터페이스로 묶고 공격하는 세부 구현을 클래스로 작성한다. 이렇게 로직을 구현할 경우, 구체적인 이동과 공격 방식이 `캡슐화`되고 객체에 관계없이 이동과 공격 방식을 변경없이 `추가 기능 구현`이 가능하다.

```java
//Robot abstract class
abstract class Robot {
    private String name;

    private MovingStrategy movingStrategy;
    private AttackStrategy attackStrategy;

    public Robot(String name) {
        this.name = name;
    }

		// movingStrategy, attackStretegy setter

    void attack() {
        attackStrategy.attack();
    }

    void move() {
        movingStrategy.move();
    }

    void introduceMyself() {
        System.out.println("My name is: " + this.getName());
        this.move();
        this.attack();
    }
}

//Atom class
class Atom extends Robot {
    public Atom(String name) {
        super(name);
    }
}

//Sungard class
class Sungard extends Robot {
    public Sungard(String name) {
        super(name);
    }
}

//TaekwonV class
class TaekwonV extends Robot {
    public TaekwonV(String name) {
        super(name);
    }
}

//MovingStrategy interface
interface MovingStrategy {
    void move();
}

//WalkingStrategy class
class WalkingStrategy implements MovingStrategy {
    @Override
    public void move() {
        System.out.println("I can walk");
    }
}

//FlyingStrategy class
class FlyingStrategy implements MovingStrategy {
    @Override
    public void move() {
        System.out.println("I can fly");
    }
}

//AttackStrategy interface
interface AttackStrategy {
    void attack();
}

//PunchStrategy class
class PunchStrategy implements AttackStrategy {
    @Override
    public void attack() {
        System.out.println("I have strong punch and can attack with it.");
    }
}

//MisilleStrategy class
class MisilleStrategy implements AttackStrategy {
    @Override
    public void attack() {
        System.out.println("I have Misille and can attack with it");
    }
}

//Main class
class Main {
    public static void main(String[] args) {
        Robot taekwonV = new TaekwonV("taekwonV");
        Robot atom = new Atom("atom");
        Robot sungard = new Sungard("sungard");

        taekwonV.setAttackStrategy(new MisilleStrategy());
        taekwonV.setMovingStrategy(new FlyingStrategy());

        atom.setAttackStrategy(new PunchStrategy());
        atom.setMovingStrategy(new WalkingStrategy());

        sungard.setAttackStrategy(new PunchStrategy());
        sungard.setMovingStrategy(new FlyingStrategy());

        taekwonV.introduceMyself();
        atom.introduceMyself();
        sungard.introduceMyself();
    }
}

/**
 * 결과
 *
 * My name is: taekwonV
 * I can fly
 * I have Misille and can attack with it
 * My name is: atom
 * I can walk
 * I have strong punch and can attack with it.
 * My name is: sungard
 * I can fly
 * I have strong punch and can attack with it.
 *
 */
```

스트래티지 패턴에서 중요하게 생각해야 하는 부분은 `변화하지 않는 것과 변화하는 것을 구분` 지어야 한다.  변화하지 않는 것은 추상화(interface)로 정의하고 변화하는 것은 구현체(class)로 정의해야 한다. 스트래티지 패턴을 구현하면 다양한 상황에 따라서 원하는 기능을 변경하기 용이하고 중복코드의 발생이 줄어드는 효과가 있다.

앞서 이야기한 OCP의 정의를 생각해보면 `변경없이 기능을 추가할 수 있어야 한다.`는 이야기를 떠올려보자. 만약 이동에 대한 기능을 추가하고 싶다면 MovingStrategy interface의 구현체 클래스를 작성하여 구현하면 된다. 위와 같이 로직을 구현할 경우, 별도의 변경 로직없이 기능 추가를 할 수 있어 OCP를 만족할 수 있다!

<br>

**[스트래티지 패턴 정리]**

<img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/f5478cbe-d2d5-4e0f-954f-6f893e7bba69/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210727%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210727T081352Z&X-Amz-Expires=86400&X-Amz-Signature=26351da23f9eefb13240ae87aa88474c54736c02c378a7963229fe83fdc6f79e&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22" width="80%">

- **Strategy** : 인터페이스나 추상 클래스로 동일한 방식으로 알고리즘을 호출하는 방법을 명시한다.
- **ConcreteStrategy1, ConcreteStrategy2, ConcreteStrategy3** : 스트래티지 패턴에 명시한 알고리즘을 실제로 구현한 클래스이다.
- **Context** : 스트래티지 패턴을 이용하는 역할을 수행한다. 필요에 따라 구체적인 전략을 바꿀 수 있도록 setter 메서드를 제공한다.

