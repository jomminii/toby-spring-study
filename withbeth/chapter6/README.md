## What I learned

-   관심이 다른 서로 다른 성격의 코드(기술코드와 비즈니스로직코드)를 분리 하는 여러가지 방법중, Dynamic Proxy를 이용하여 분리하는 방법.
-   이때 IF + DI(Strategy + Decorator패턴) + Dynamic Proxy를 이용하여, 소스코드 변경없이 설정 변경만으로 부가기능 추가를 가능하여 OCP를 지키는 방법.
-   이때, 사용하는 Dynamic Proxy는, JDK Dynamic Proxy를 이용하여, 그 장단점과 한계, 그리고 Spring의 ProxyFactoryBean를 이용하여 그 한계를 어떻게 해결하는지에 대해 알수 있었다.

## TOC

### Tx경계설정 코드 분리

Problem :

Tx PSA를 통해, 특정 기술 환경에 종속되지 않게 되었지만, 아직 경계설정코드가 비즈니스 로직에 남아있다.

Solution :

1. Extract method
2. Extract Class using DI + Decorator Pattern

Merit :

1. 성격다른 코드의 SoC
2. 비즈니스 로직은 PSA조차 필요 없는 순수 POJO로 작성가능해졌기에, SRP하며, Testable해졌다

### 고립된 단위 테스트

#### Goal :

고립테스트의 장단점과 통합테스트와의 구분

#### BackGround :

- 가장 심플하며 테스트하기 좋은 코드는, 테스트 대상 개체를, 그 의외의 것들(외부 리소스, 테스트 대상개체의 의존객체들)과 고립시키는 것.
- 이를 통해 해당 테스트가 실패한 경우, 실패 원인 식별을 빠르게 수행 할 수 있다.

#### Case :

- UserService는, 사용자 정보를 관리하는 비즈 로직의 구현 코드이지만, 여러가지 의존관계를 가지고 있다.
- 해당 의존 개체의 동작 수행 여부에 따라, UserServiceTest자체가 실패할 가능성도, 실패시의 원인찾기도 힘들어질 가능성이 있다.
- 따라서, 테스트 대역을 이용해, 테스트하고자 하는 개체만 고립시킬 필요가 있다.
- 테스트 대역을 이용하여, 테스트 대상 개체가, 그 의존 개체와 메시지 전달을 통한 상호작용이 바르게 수행했는지만 검증 할 수 있다면, 해당 테스트 개체는 원했던 방식으로 작동한다고 결론 지을 수 있다.

#### 고립테스트 Pros :

- 다른 의존 대상에 영향받지 않는다.
- 테스트 실패시 실패 원인 식별을 빠르게 수행 가능하다.
- 외부 리소스와 고립되어 수행하기에, 테스트 수행 시간도 비약적으로 상승한다.

#### 고립테스트 Cons :

- 테스트 대역 Test double/stub/mock의 코드 구현이 필요하다 -> Mock Framework로 대체 가능

#### 고려사항 :

- 단위 테스트로 만들기 어렵거나 애매한 코드는 분명히 존재한다.
- 예) DAO
	- DAO자체는 로직을 담고 있기보다, 인터페이스와 같은 역할을 수행한다.
	- 또한 DAO가 가지고 있는 SQL은, 실제 DB와의 상호작용을 통해야만 수행여부를 파악할 수 있다.
	- 따라서, 기본적으로 DAO는 TestDB와 연동하여 테스트하는 편이 좋다.
	- 이러한 검증이 충분히 끝났다면, DAO를 이용하는 개체의 테스트는 Mock하여 테스트 수행시간을 줄이는 편이 낫다(예제의 JavaMailAPI또한 충분히 검증한 API이므로 Mock을 이용하여 상호작용만 검증한 것 처럼)

### 다이내믹 프록시와, 팩토리 빈

#### Term : Proxy, ProxyPattern and DecoratorPattern

##### What we did so far?

- 위 예제에서, Tx부가기능을, IF + DI + Decorator pattern을 이용하여, 비즈니스 로직과 분리 시켰다.
- 이때, 데코레이터는 핵심 비즈니스 로직과 같은 인터페이스를 구현하고, 클라이언트는 인터페이스로 상호작용하도록 하여, 클라이언트와 핵심기능(비지니스로직) 사이에 데코레이터를 끼워넣는 식으로 구현하였다.
- 클라이언트로부터 직접 불리게 되는 데코레이터는, 같은 API를 구현한 핵심 기능을 Composition하여, 부가기능이외의 모든 기능은, 핵심 기능에 위임하였다.

##### Proxy?

**What?**

- 자신이 클라이언트가 사용하려고 하는 실제 대상처럼 위장해서 클라이언트의 요청을 대리하는 것.
- 이때 Proxy를 통해 최종적으로 요청을 위임받아 실제로 요청을 처리하는 개체를 TARGET이라 한다.

**특징?**

- Target과 같은 인터페이스를 구현해야 하며,
- Proxy가 Target을 제어 할 수 있다.

**When to use?**

- 클라이언트가, Target에 접근하는 방법 제어 => Proxy Pattern
- Target에 부가기능 추가 => Decorator Pattern

**고려사항?**

- Callback Framework에서의 SELF problem 주의

##### Decorator Pattern?

**What?**

- Target에 부가기능을, Runtime에 Dynamic하게 추가하기 위해 사용하는 Proxy.
- Compile시점에는 어떤 방법과 순서로 Proxy와 Target이 연결되는지 모른다.

**특징?**

- 다수개의 Proxy 를 이용하여 부가기능을 여러개 추가할수 있으며,
- Proxy가 직접 Target을 사용토록 고정시킬 필요 없다.

**Pros?**

- Target과 클라이언트 변경없이, 부가기능 추가 가능

**Exam?**

- Java IO Stream
```java
InputStream is = new BufferedInputStream(new FileInputStream("test.txt"));
```

##### Proxy Pattern?

**What?**

- Target의 접근제어를, Runtime에 Dynamic하게 수행하는 Proxy

**When to use?**

- 1. LazyLoading (+ Cache)
- 2. RMI와 같은 원격객체 이용
- 3. Target접근제한 
	- 특정 레이어에서는 READONLY로 사용하고 싶은 경우등(Collections.unmotifiableList())

**How?**

- Same as decorator pattern
- 하지만, Target의 구체적인 정보를 알아야 할 경우도 있다(LazyLoading케이스)

#### Term : Dynamic Proxy

##### 수동 Proxy 문제점 :

- 1. Target인터페이스의 구현 및 위임의 번거로움
	- mock이나 stub을 수동으로 만들기 번거로웠던 것처럼.
- 2. 부가기능의 중복
	- 예제의 Tx부가기능은, 전체 서비스 레이어에서 필요하다.
	- 각 서비스마다 Tx부가기능 Proxy를 만들기 시작하면..끝이 없다.

##### Solution : JDK Dynamic Proxy(or CGLib)

##### Dynamic Proxy

**What?**

- ProxyFactory를 이용해(Proxy클래스의 newProxyInstance() 정적 팩토리메서드), Runtime시 Dynamic하게 만들수 있는 Proxy.
- 내부에서, class literal과 같은 type token을 대상으로 리플렉션하여 프록시 작성.


**How?**

- Proxy생성시, Target의 인터페이스 타입을 제공하여, 해당 인터페이스 구현한 Dynamic Proxy개체를 생성받는다.
- 또한, 이때 부가기능를 담당하는 InvocationHanlder인터페이스 구현 개체를 제공하면, Dynamic Proxy개체는 모든 요청을 해당 개체의 invoke() 메서드로 보내준다.


**Pros?**

* 1. Target 개체 위임하는 코드량을 줄여준다.
	* (자동으로 Target인터페이스를 구현한 Dynamic Proxy생성되므로)
* 2. 부가기능핸들러(InvocationHanlder)는 Target 인터페이스 타입과 상관 없이 적용 가능하다. 
	* 이때, 부가기능을 적용하고 싶은 메서드의 return type & method name을 특정해야 한다.

#### Case : Dynamic Proxy를 Spring Factory Bean Interface로 구현

##### Goal :

JDK DynamicProxy이용하여 만든 DynamicProxy개체를, Spring의 DI를 이용하여 사용하고 싶다.

##### Problem :

Dynamic Proxy는 보통방법으로는 Spring Bean으로 등록 불가능하다.

**Why?**

- 스프링은 기본적으로 Class.forName({classname}).newInstance() 리플렉션을 이용하여, 디폴트 생성자로 해당 클래스 개체를 만든다. 
- 하지만,
	- Dynamic Proxy 개체는 이런식으로 생성되지 않으며, (private 생성자와 정적팩토리메서드 이용해 개체 생성)
	- Class자체도 내부적으로 다이내믹하게 새로 정의해서 사용하기에, 스프링 빈생성시에, 클래스정보를 알 길이 없다.

##### Solution :

스프링이 제공하는 FactoryBean 인터페이스를 구현하여 빈으로 등록하기.

**What?**

- FactoryBean은, 스프링을 대신해서, 빈 생성로직만을 담당하는 스페셜 빈.
- 빈 생성과정에서만 이용된다.

**How?**

- FactoryBean 인터페이스를 구현한 클래스를 스프링 빈으로 등록하면,
- 스프링은 FactoryBean의 getObject()이용하여 개체를 가져와 빈으로 등록한다.

**Pros?**

- 수동프록시 작성시의 2가지 문제점의 해결
	- 1) Target 인터페이스 구현 및 위임의 번거로움
		- => Target 인터페이스 구현하는 Dynamic Proxy 자동생성으로 해결.
	- 2) 부가 기능의 중복
		- => 팩토리 빈을 한번 생성해두면, Target 타입에 상관없이, 설정변경만으로 재사용 가능하다.

**Limitation?**

- 1. 하나의 클래스의 여러 메서드에 부가기능을 간단히 추가 할 수 있지만, 여러개 클래스에 부가기능을 추가할시, 설정 중복이 일어난다.
- 2. 한번 스프링 빈으로 등록된 Proxy는 해당 Target에 의존되게 된다.
- 3. 하나의 클래스에 여러 부가기능을 추가하려면, 부가기능 수만큼의 프록시 생성이 필요하다.


### 스프링의 프록시 팩토리 빈

#### Goal : 
스프링이, FactoryBean인터페이스를 구현한 ProxyFactoryBean의 한계를 어떻게 해결하는지.

#### What?

스프링이 제공하는 프록시 팩토리 빈은,
* Proxy 개체 생성 기술의 PSA를, Tmpl/Callback패턴 + DI를 통해 제공한다.
- 더 쉽게 말하면, 기술 환경에 종속되지 않고, 일관된 방법으로 Proxy생성할 수 있는 추상 레이어(PSA)를 제공한다.
- 더 쉽게 말하면, Proxy생성 기술을 추상화한, FactoryBean을 제공한다.
- 이 FactoryBean은, Proxy생성 및 빈 등록만을 책임지며, 다음과 같은 기능도 제공한다
	- 부가기능 개체에 Target정보 제공.
	- Target 인터페이스 자동검출 기능.
- 따라서, Proxy를 통해 제공할 부가기능은 별도의 빈에 둘 수 있으며, MethodInterceptor인터페이스를 구현하여 작성한다.

#### JDK Dynamic Proxy와의 차이점 :

##### JDK Dynamic Proxy 동작방식
```java

class SomeInvocationHandlerImpl implements InvocationHanlder {

	Object target; // Proxy클래스에서 Target정보를 가지고 있다.

	Object invoke(Object proxy, Method method, Object[] args) {
		Object ret = method.invoke(target, args) 
		// method 선정 방법전략을 가지고 있다
		if (ret instanceof String && method.getName().startWith("say*")) { 
				// 부가기능 전략을 가지고 있다
				return (String) ret.toUpperCase();
			}
		return ret;
	}

}
```

##### Spring Proxy Factory Bean 동작방식

```java
class UpperCase implements MethodInterceptor {
	Object invoke(Method invocation invocation) throws Throwable {
		// Method invocation 개체가 Target정보를 가지고 있다.
		String ret = (String) invocation.proceed();
		return ret.toUpperCase();
	}
}
```

##### 차이점 1)

Tmpl/Callback패턴 + DI 이용하여, 부가기능 개체로부터 Target 의존을 분리.

- 이를 통해, 부가기능 개체는 순수하게 부가기능 전략(Advice)만 담아, 싱글톤빈으로 등록하여 재사용가능하게 되었다.
- 또한 하나의 Proxy에 여러 종류의 부가기능을 제공가능하게 되었다.


##### 차이점 2)

DI를 이용하여, 부가기능 개체로부터 메서드 선정 방법(Pointcut) 을 분리.
해당 전략들은 Advisor(Advice + Pointcut)로서, Dynamic Proxy에 DI할수 있다. 
따라서, 해당 전략은 모두 싱글톤빈으로 등록하여 재사용가능하게 한다.


##### 차이점 3)

프록시 생성시, Proxy가 구현해야 하는 인터페이스 타입을 제공하지 않아도 된다.
- 스프링의 프록시 팩토리빈에서 인터페이스 자동검출가능


