# Overview

스프링이 자바에서 가장 중요하게 가치를 두는 것은 객체지향 프로그래밍이 가능한 언어라느 점이다. 자바 엔터프라이즈 기술이 잃어버렸던 객체지향 기술의 진정한 가치를 회복시키고 객체지향 프로그래밍이 제공하는 혜택을 누릴 수 있도록 기본으로 돌아가자는 것이 스프링의 핵심 철학이다.

스프링을 이해하려면 오브젝트에 깊은 관심을 가져야 한다. 애플리케이션에서 오브젝트가 생성된고 다른 오브젝트와 관계를 맺고 사용되고 소멸하기까지의 전 과정을 깊게 생각해봐야 한다. 더 나아가서 어떻게 설계해야 하는지 어떤 단위로 만들어지며 어떤 과정을 통해 자신의 존재를 드러내고 등장해야 하는지에 대해서도 살펴봐야 한다.

객체지향 설계의 기초와 원칙을 비롯해서 다양한 목적을 위해 재활용 가능한 설계 방법인 디자인 패턴, 리팩토링, 단위 테스트와 같은 오브젝트 설계와 구현에 관한 여러가지 응용 기술과 지식이 요구된다.

스프링은 객체지향 설계와 구현에 관해 특정한 모델과 기법을 억지로 강요하지 않는다. 하지만 오브젝트를 어떻게 효과적으로 설계하고 구현하고 사용하고 개선해나갈 것인가에 대한 명쾌한 기준을 마련해준다. 동시에 객체지향 기술과 설계, 구현에 관한 실용적인 전략과 검증된 베스트 프랙티스를 평범한 개발자도 자연스럽게 적용할 수 있도록 프레임워크 형태로 제공한다.

# 1. 초난감 DAO

> DAO
DAO(Data Access Object)는 DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트를 말한다.

> 자바빈
자바빈(JavaBean)은 원래 비주얼 툴에서 조작 가능한 컴포넌트를 말한다. 자바의 주력 개발 플랫폼이 웹 기반의 엔터프라이즈 방식으로 바뀌면서 비주얼 컴포넌트로서 자바빈은 인기를 잃어갔지만 자바빈의 몇 가지 코딩 관례는 JSP 빈, EJB와 같은 표준 기술과 자바빈 스타일의 오브젝트를 사용하는 오픈소스 기술을 통해 계속 이어져 왔다. 이제는 자바빈이라고 말하면 비주얼 컴포넌트라기보다는 다음 두 가지 관례에 따라 만들어진 오브젝트를 가르킨다.
- 디폴트 생성자: 자바빈은 파라미터가 없는 디폴트 생성자를 가지고 있어야 한다. 툴이나 프레임워크에서 리플렉션을 이용해 오브젝트를 생성하기 때문에 필요하다.
- 프로퍼티: 자바빈이 노출하는 이름을 가진 속성을 프로퍼티라고 한다. 프로퍼티는 set으로 시작하는 수정자 메소드(setter)와 get으로 시작하는 접근자 메소드(getter)를 이용해 수정 또는 조회할 수 있다.

JDBC를 이용하는 작업의 일반적인 순서는 다음과 같다.

- DB 연결을 위한 Connection을 가져온다.
- SQL을 담은 Statement(혹은 PreparedStatement)를 만든다.
- 만들어진 Statement를 실행한다.
- 조회의 경우 SQL 쿼리의 실행 결과를 ResultSet으로 받아서 정보를 저장할 오브젝트에 옮겨준다.
- 작업 중 생성된 Connection, Statement, ResultSet 같은 리소스는 작업을 마친 후 반드시 닫아준다.
- JDBC API가 만들어내는 예외(exception)을 잡아서 직접 처리하거나 메소드에 throws를 선언해 메소드 밖으로 던지게한다.

# 2. DAO의 분리

## 관심사의 분리

- 오브젝트에 대한 설계와 구현 코드는 변한다.
- 개발자가 객체를 설계할 때 가장 염두에 둬야 할 사항은 미래의 변화를 어떻게 대비할까이다.
- 객체지향 기술은 변화에 효과적으로 대처할 수 있다는 기술적인 특징 때문에 사용한다.
- 그러면 어떻게 변경이 일어날 때 필요한 작업을 최소화하고 변경이 다른 곳에 문제를 일으키지 않게 할 수 있을까?
    - 분리와 확장을 고려한 설계가 있었기 때문이다.

### 분리

- 모든 변경과 발전은 한 번에 한 가지 관심사항에서 집중해서 일어난다.
- 변화는 대체로 집중된 한 가지 관심에 대해 일어나지만 그에 따른 작업은 한 곳에 집중되지 않는 경우가 많다.
- 우리가 할 일은 한 가지 관심이 한 군데에 집중되게 하는 것이다. 즉 관심이 같은 것끼리 모음고 관심이 다른 것은 떨어져있게 하는 것이다.
- 관심사의 분리(Separation of Concerns)를 객체지향에 적용하면 관심이 같은 것끼리는 하나의 객체 안으로 또는 친한 객체로 모이게 하고 관심이 다른 것은 가능한 따로 떨어져서 서로 영향을 주지 않도록 분리하는 것이다.
- 관심의 종류에 따라 코드를 구분해놓았기 때문에 한 가지 관심에 대한 변경이 일어날 경우 그 관심이 집중되는 부분의 코드만 수정하면 된다.

기능이 추가되거나 바뀐 건 없어도 미래의 변화에 좀 더 손쉽게 대응할 수 있는 코드가 되도록 변경하는 것이 **리팩토링(refactoring)**이다. 공통의 기능을 담당하는 메소드로 중복된 코드를 뽑아내는 것을 **메소드 추출(extract method)** 기법이라고 한다.

## 리팩토링

- 기존의 코드를 외부의 동작방식에 변화 없이 내부 구조를 변경해서 재구성하는 작업 또는 기술
- 리팩토링을 하면 코드 내부의 설계가 개선되어 코드를 이해하기가 편해지고 변화에 효율적으로 대응할 수 있다.
- 생산성은 올라가고 코드의 품질은 높아지며 유지보수하기 용이해지고 견고하면서도 유연한 제품을 개발할 수 있다
- 리팩토링이 절실히 필요한 코드의 특징을 나쁜 냄새라고 부르기도 한다.

- 슈퍼클래스에 기본적인 로직의 흐름을 만들고 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하도록 하는 방법을 템플릿 **메소드 패턴(template method pattern)**이라고 한다.
    - 스프링에서 애용하는 패턴이다.
   
![image](https://user-images.githubusercontent.com/66561524/190833498-d1edce80-6a43-4968-9834-ed6d35e53a51.png)

서브클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것을 팩토리 메소드 패턴(factory method pattern)이라고 한다.


![image](https://user-images.githubusercontent.com/66561524/190833517-87359467-3e08-46c8-bb06-3d5ce150dc1c.png)

- 상속구조를 통해 성격이 다른 관심사항을 분리한 코드를 만들어내고 서로 영향이 덜 주도록하는 것이다.

## 디자인 패턴

- 소프트웨어 설계 시 특정 상황에서 자주 만나는 문제를 해결하기 위해 사용할 수 있는 재사용 가능한 솔루션
- 모든 패턴에는 간결한 이름이 있어 적용하고 자 할 때 간단히 패턴 이름을 언급하는 것만으로도 설계의 의도와 해결책을 함께 설명할 수 있다.
- 디자인 패턴은 주로 객체지향 설계에 관한 것이고 대부분 객체지향적인 설계로부터 문제를 해결하기 위해 적용할 수 있는 확장성 추구 방법이 대부분 두 가지 구조로 정돈된다.
    - 하나는 클래스 상속이고 다른 하나는 오브젝트 합성이다.
    - 따라서 패턴의 결과로 나온 코드나 설계 구조는 대부분 비슷
- 패턴에서 가장 중요한 것은 패턴의 핵심이 담긴 목적 또는 의도다. 패턴을 적용할 상황 해결해야 할 문제, 솔루션의 구조와 각 요소의 역할과 함께 핵심 의도가 무엇인지를 기억해둬야 한다.

## 템플릿 메소드 패턴

- 상속을 통해 슈퍼클래스의 기능을 확장할 때 사용하는 대표적인 방법. 변하지 않는 기능은 슈퍼클래스에 만들어두고 자주 변경되며 확장할 기능은 서브클래스에서 만들도록 한다.
- 슈퍼클래스에서는 미리 추상 메소드 또는 오버라이드 가능한 메소드를 정의해두고 이를 활용해 코드의 기본 알고리즘을 담고 있는 템플릿 메소드를 만든다.
- 슈퍼클래스에서 디폴트 기능을 정의해두거나 비워뒀다가 서브클래스에서 선택적으로 오버라이드할 수 있도록 만들어둔 메서드를 훅(hook) 메소드라고 한다.
- 서브클래스에서는 추상 메소드를 구현하거나 훅 메소드를 오버라이드하는 방법을 이용해 기능의 일부를 확장한다.

- 예제 코드
    
    ```kotlin
    public class TemplateMethod {
    
        public static void main(String[] args) {
            Child1 child1 = new Child1();
            child1.templateMethod();
            /*
            default method
            hook method
            child1 method
             */
    
            Child2 child2 = new Child2();
            child2.templateMethod();
            /*
            default method
            hook method
            child2 method
             */
        }
    
        public static abstract class Super {
            public void templateMethod() {
                // 기본 알고리즘 코드
                defaultMethod();
                hookMethod();
                abstractMethod();
            }
    
            private void defaultMethod() {
                System.out.println("default method");
            }
    
            // 선택적으로 오버라이드 가능한 훅 메소드
            protected void hookMethod() {
                System.out.println("hook method");
            }
    
            public abstract void abstractMethod();
        }
    
        public static class Child1 extends Super {
    
            @Override
            public void abstractMethod() {
                System.out.println("child1 method");
            }
        }
    
        public static class Child2 extends Super {
    
            @Override
            public void abstractMethod() {
                System.out.println("child2 method");
            }
        }
    }
    ```
    

## 팩토리 메서드 패턴

- 상속을 통해 기능을 확장하게 하는 패턴
- 슈퍼 클래스 코드에서는 서브클래스에서 구현할 메소드를 호출해서 필요한 타입의 오브젝트를 가져와 사용
- 이 메소드는 주로 인터페이스 타입으로 오브젝트를 리턴하므로 서브클래스에서 정확히 어떤 클래스의 오브젝트를 만들어 리턴할지는 슈퍼클래스에서는 알지 못한다. 사실 관심도 없다.
- 이렇게 서브클래스에서 오브젝트 생성 방법과 결정할 수 있도록 미리 정의해둔 메소드를 팩토리 메소드라고 하고 이 방식을 통해 오브젝트 생성 방법을 슈퍼 클래스의 기본 코드에서 독립시키는 방법을 팩토리 메소드 패턴이라고 한다.
- 자바에서는 종종 오브젝트를 생성하는 기능을 가진 메소드를 팩토리 메소드라고 부르는데 팩토리 메소드와 팩토리 메소드 패턴의 팩토리 메소드는 의믜가 다르므로 혼동하지 말아야 한다.

- 예제 코드
    
    ```kotlin
    public class FactoryMethodPattern {
        public static void main(String[] args) {
            Child1 child1 = new Child1();
            child1.conversation();
            /*
            안녕
            냐옹
            잘가
             */
    
            Child2 child2 = new Child2();
            child2.conversation();
            /*
            안녕
            왈
            잘가
             */
        }
    
        public static abstract class Super {
    
            public void conversation() {
                hello();
                Animal animal = getAnimal();
                animal.speak();
                bye();
            }
    
            public void hello() {
                System.out.println("안녕");
            }
    
            public void bye() {
                System.out.println("잘가");
            }
    
            protected abstract Animal getAnimal();
    
        }
    
        interface Animal {
            void speak();
        }
    
        public static class Child1 extends Super{
    
            @Override
            protected Animal getAnimal() {
                return new Cat();
            }
    
            public static class Cat implements Animal {
    
                @Override
                public void speak() {
                    System.out.println("냐옹");
                }
            }
        }
    
        public static class Child2 extends Super{
    
            @Override
            protected Animal getAnimal() {
                return new Dog();
            }
    
            public static class Dog implements Animal {
    
                @Override
                public void speak() {
                    System.out.println("왈");
                }
            }
        }
    }
    ```
    

# 3. DAO의 확장

- 관심사에 따라 분리한 오브젝트들은 제각기 독특한 변화의 특징이 있다.
- 변화의 성격이 다르낟는 건 변화의 이유와 시기, 주기 등이 다르다는 것이다.
- 추상 클래스를 만들고 이를 상속한 서브클래스에서 변화가 필요한 부분을 바꿔서 쓸 수 있게 만든 이유는 변화의 서엮을 분리해서 서로 영향을 주지 않은 채로 각각 필요한 시점에 독립적으로 변경할 수 있게 하기 위해서다.
- 그러다 단점이 많은 상속이라는 방법을 사용했다는 사실이 불편한다.

## 클래스의 분리

- 두 개의 관심사를 본격적으로 독립시키면서 동시에 손쉽게 확장할 수 있는 방법
- 성격이 다른 다르게 변할 수 있는 관심사를 분리하는 작업
- 클래스의 분리는 자유롭게 확장하기 어려울 수 있다.

## 인터페이스의 도입

- 클래스를 분리하면서도 두 개의 클래스가 서로 긴밀하기 연결되어 있지 않도록 중간에 추상적인 느슨한 연결을고리를 만들어주면 확장에 자유로워 진다.
- 추상화란 어떤 것들의 공통적인 성격을 뽑아내어 이를 따로 분리하는 작업이다.
- 자바가 추상화를 위해 제공하는 가장 유용한 도구는 인터페이스다.
- 오브젝트를 만드려면 구체적인 클래스를 하나 선태해야하지만 인터페이스로 추상화해놓은 최소한의 통로를 통해 접근하는 쪽에서 오브젝트를 만들 때 사용할 클래스가 무엇인지 몰라도 된다.

## 관계설정 책임의 분리

- 오브젝트가 다른 오브젝트의 기능을 사용한다면 사용하는 오브젝트를 클라이언트, 사용되는 오브젝트를 서비스라고 할 수 있다.
- 클래스 사이에 관계를 설정하는 것이 아니라 오브젝트 사이에 다이나믹한 관계를 만들어야 한다.
- 코드에서 특정 클래스를 전혀 알지 못하더라도 해당 클래스가 구현한 인터페이스를 사용했다면 클래스의 오브젝트를 인터페이스 타입으로 받아서 사용할 수 있다. 다형성이라는 특징 덕분이다.

## 원칙과 패턴

### 개방 폐쇄 원칙(OCP, Open-Closed Principle)

- 클래스나 모듈은 확장에 열려 있어야 하고 변경에는 닫혀 있어야 한다.
- 잘 설계된 객체지향 클래스의 구조를 살펴보면 이 개방 폐쇄 원칙을 아주 잘 지키고 있다.
- 인터페이스를 사용해 확장 기능을 정의한 대부분의 API는 이 개방 폐쇄 원칙을 따른다.

### 높은 응집도와 낮은 결합도

- 개방 폐쇄 원칙은 높은 응집도와 낮은 결합도(high coherence and low coupling)라는 소프트웨어 개발의 원리이다.
- 응집도가 높다는 건 하나의 모듈, 클래스가 하나의 책임 또는 관심사에만 집중되어 있다는 뜻이다.
- 불필요하거나 직접 관련이 없는 외부의 관심과 책임이 얽혀있지 않으며 하나의 공통 관심사는 한 클래스에 모여있다.
- 높은 응집도는 클래스 레벨뿐 아니라 패키지, 컴포넌트, 모듈에 이르기까지 대상의 크기가 달라도 동일한 원리로 적용될 수 있다.
- **높은 응집도**
    - 변화가 일어날 때 해당 모듈에서 변하는 부분이 큰 것이다.
    - 변경이 일어날 때 모듈의 많은 부분이 함께 바뀐다면 응집도가 낮은 것이다.
    - 모듈의 일부분에만 변경이 일어나도 된다면 모듈 전체에서 어떤 부분이 바뀌어야 하는지 파악해야 하고 그 변경으로 인해 바뀌지 않는 부분에는 다른 영향을 미치지는 않은지 확인해야 하는 이중의 부담이 생긴다.
        - 반면에 응집도를 높이면 해당 클래스를 직접 테스트하는 것으로만해도 충분하다.
- **낮은 결합도**
    - 높은 응집도보다 더 민감한 원칙이다.
    - 책임과 관심사가 다른 오브젝트 또는 모듈과는 낮은 결합도 즉, 느슨하게 연결된 형태를 유지하는 것이 바람직하다.
        - 결합도란 ‘하나의 오브젝트가 변경이 일어날 때 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도’
        - 낮은 결합도란 하나의 변경이 발생할 때 여타 모듈과 객체로 변경에 대한 요구가 전파되지 않는 상태

### 객체지향 설계 원칙(SOLID)

- 로버트 마틴이 정리한 SOLID 원칙
    - SRP(Single Responsibility Principle): 단일 책임 원칙
    - OCP(Open-Closed Principle): 개방 폐쇄 원칙
    - LSP(Thre Liskov Substituion Principle): 리스코프 치환 원칙
    - ISP(The Interface Segregation Principle): 인터페이스 분리 원칙
    - DIP(The Dependency Inversion Principle): 의존관계 역전 원칙
    

### 전략 패턴(Strategy Pattern)

- 전략 패턴은 자신의 기능 맥락(context)에서 필요에 따라 변경이 필요한 알고리즘을 인터페이스를 통해 통째로 외부로 분리시키고 이를 구현한 구체적인 알고리즘 클래스를 필요에 따라 바꿔서 사용할 수 있게 하는 디자인 패턴이다.

# 4. 제어의 역전(IoC)

IoC라는 약자로 사용되는 제어의 역전(Inversion of Control)이라는 용어가 있다. 

## 오브젝트 팩토리

- 두 개의 오브젝트가 연결돼서 사용할 수 있도록 관계를 맺어주는 역할

### 팩토리

- 객체의 생성 방법을 결정하고 그렇게 만들어진 오브젝트를 돌려주는 역할을 한다.
- 추상 팩토리 패턴, 팩토리 메소드 패턴과는 다르다.
- 팩토리는 컴포넌트의 구조와 관계를 정의한 설계도 같은 역할을 한다.

![image](https://user-images.githubusercontent.com/66561524/190833533-3916efc7-44a0-4b86-92b8-a2dac3a7c0fc.png)

- 애플리케이션의 컴포넌트 역할을 하는 오브젝트와 애플리케이션의 구조를 결정하는 오브젝트가 분리되어있다.

## 제어권의 이전을 통한 제어관계 역전

- 제어의 역전이라는 것은 간단히 프로그램의 제어 흐름 구조가 뒤바뀌는 것이다.
- 일반적인 프로그램의 흐름
    - 프로그램이 시작되는 지점에서 다음 사용할 오브젝트를 결정하고 결정한 오브젝트를 생성하고 만들어진 오브젝트에 있는 메소드를 호출하고 오브젝트 메소드 안에서 다음에 사용할 것을 결정하고 호출하는 식의 작업이 반복된다.
    - 각 오브젝트는 프로그램 흐름을 결정하거나 사용할 오브젝트를 구성하는 작업에 능동적으로 참여한다.
- 제어의 역전이란 이런 일반적인 제어 흐름의 개념을 거꾸로 뒤집는다.
    - 오브젝트가 자신이 사용할 오브젝트를 스스로 선택하지 않는다. 생성하지도 않는다. 자신이 어떻게 만들어지고 사용되는지를 모른다.
    - 모든 제어 권한을 자신이 아닌 다른 대상에게 위임하기 때문이다.
- 제어의 역전 개념은 여러 디자인 패턴에서 확인할 수 있다.
    - 템플릿 메소드 패턴
        - 슈퍼클래스에서 메서드를 구현해놓고 서브 클래스에서는 특정 추상 메서드를 구현해야 한다. 그런데 그 추상 메서드가 어떻게 사용이 될지 모른다.
        - 제어권을 상위 템플릿 메소드에 넘기고 자신은 필요할 때 호출되어 사용되도록 한다는 제어의 역전 개념이다.
        - 제어의 역전이라는 개념을 활용해 문제를 해결하는 디자인 패턴이라고 할 수 있다.
    - 프레임워크
        - 라이브러리를 사용하는 애플리케이션 코드는 애플리케이션 흐름을 직접 제어한다.
        - 애플리케이션 코드가 프레임워크에 의해 사용된다.
            - 프레임워크 위에 개발한 클래스를 등록해두고 흐름을 주도하는  중에 개발자가 만든 애플리케이션 코드를 사용하도록 만드는 방식이다.
            - 프레임워크는 제어의 역전 개념이 적용되어 애플리케이션 코드가 수동적으로 작용하는 틀을 의미한다.
- 제어의 역전에서는 프레임워크 또는 컨테이너와 같이 애플리케이션 컴포넌트의 생성과 관계설정, 사용, 생명주기 관리 등을 관장하는 존재가 필요하다.

# 5. 스프링의 IoC

- 스프링의 핵심을 담당하는 건 빈 팩토리 또는 애플리케이션 컨텍스트이다.

## 오브젝트 팩토리를 이용한 스프링 IoC

### 애플리케이션 컨텍스트와 설정정보

- 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트를 빈(bean)이라고 부른다.
    - 빈(bean)은 오브젝트 단위의 애플리케이션 컴포넌트를 말한다.
    - 스프링 빈은 스프링 컨테이너가 생성과 관계설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트이다.
- 빈의 생성과 관계설정 같은 제어를 담당하는 IoC 오브젝트를 빈 팩토리(bean factory)라고 부른다. 보통은 이를 확장한 **애플리케이션 컨텍스트(application context)**를 주로 사용한다.
    - 애플리케이션 컨텍스트는 IoC 방식을 따라 만들어진 일종의 빈 팩토리이다. 두가지가 동일하다고 보면 된다.
- 애플리케이션 컨텍스트는 별도의 정보를 참고해서 빈(오브젝트)의 생성, 관계설정 등의 제어 작업을 총괄한다. 애플리케이션 콘텍스트는 별도로 설정 정보를 들고 있지 않고 어디선가 가져와서 활용하는 범용적인 IoC 엔진이다.
    - 설정정보를 만드는 방법은 여러가지가 있다.

## 애플리케이션 컨텍스트의 동작방식

- 애플리케이션 컨텍스트는 IoC 컨테이너라 부르기도 하고 스프링 컨테이너라고 부르기도 한다.
- ApplicationContext는 BeanFactory를 상속 받기 때문에 일종의 빈 팩토리이다.
    - 스프링의 가장 대표적인 오브젝트이다.
- `@Configuration`이 붙은 객체는 애플리케이션 컨텍스트가 활용하는 IoC 설정정보다.

```kotlin
// DaoFactory 를 설정정보로 사용하는 애플리케이션 컨텍스트 ApplicationContext 타입
// @Configuration 이 붙은 자바 코드를 설정정보로 사용하기 위해서 AnnotationConfigApplicationContext 사용
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(DaoFactory.class);
UserDao userDao = applicationContext.getBean("userDao", UserDao.class);
```

![image](https://user-images.githubusercontent.com/66561524/190833556-c08e4f53-910d-47d8-89eb-906a4e7e5337.png)

- 클라이언트가 애플리케이션 컨텍스트의 getBean() 메소드를 호출하면 빈 목록에서 요청한 이름이 있는지 찾고 있다면 빈을 생성하는 메소드를 호출해서 오브젝트를 생성시킨 후 클라이언트에 돌려준다.

## 애플리케이션 컨텍스트 사용시 장점

### 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다.

- 애플리케이션이 발전하면 IoC를 적용한 오브젝트가 계속 추가될 것이다.
- 클라이언트가 필요한 오브젝트를 가져오려면 어떤 팩토리 클래스를 사용해야 할지 알아야 하고 필요할 때마다 팩토리 오브젝트를 생성해야 하는 번거로움이 있다.
- 애플리케이션 컨텍스트를 사용하면 오브젝트 팩토리가 많아져도 이를 알아야하거나 직접 사용할 필요가 없다.
- XML처럼 단순한 방법을 사용해 IoC 설정 정보를 만들 수도 있다.

### 애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해준다

- 애플리케이션 컨텍스트는 오브젝트가 만들어지는 방식, 시점과 전략을 다르게 가져갈 수 있고 부가적으로 자동생성, 오브젝트에 대한 후처리, 정보의 조합, 설정 방식의 다변화, 인터셉팅 등 오브젝트를 효과적으로 활용할 수 있는 다양한 기능을 제공한다.
- 빈이 사용할 수 있는 기반 기술 서비스나 외부 시스템과의 연동 등을 컨테이너 차원에서 제공해주기도 한다.

### 애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다

- getBean() 메소드는 이름을 이용해 빈을 찾는다.
- 타입만으로 빈을 검새갛거나 특별한 애노테이션 설정이 되어있는 빈을 찾을 수도 있다.

## 스프링 IoC의 용어 정리

- 빈(bean)
    - 빈 또는 빈 오브젝트는 스프링이 IoC 방식으로 관리하는 오브젝트
    - 관리 되는 오브젝트(managed object)라고 부르기도 한다.
    - 스프링을 사용하는 애플리케이션에서 만들어지는 모든 오브젝트가 빈은 아니다.
    - 스프링이 생성과 제어를 담당하는 오브젝트만을 빈이라고 부른다.
- 빈  팩토리(bean factory)
    - 스프링의 IoC를 담당하는 핵심 컨테이너
    - 빈을 등록하고 생성하고 조회해서 돌려주고 그 외 부가적인 빈을 관리하는 기능을 담당
    - 이 빈 팩토리를 바로 사용하지 않고 이를 확장한 애플리케이션 컨텍스트를 이용
    - BeanFactory라고 붙여쓰면 팩토리가 구현하고 있는 가장 기본적인 인터페이스의 이름이 된다. 이 인터페이스에 getBean()과 같은 메소드가 정의되어 있다.
- 애플리케이션 컨텍스트(application context)
    - 빈 팩토리를 확장한 IoC 컨테이너
    - 빈을 등록하고 관리하는 기본적인 기능은 빈 팩토리와 동일
    - 스프링이 제공하는 각종 부가 서비스를 추가로 제공
    - ApplicationContext는 BeanFactory를 상속
- 설정정보/설정 메타정보(configuration metadata)
    - IoC를 적용하기 위한 메타정보
    - 구성정보 내지는 형상정보라는 의미
- 컨테이너(container) 또는 IoC 컨테이너
    - IoC 방식으로 빈을 관리. 애플리케이션 컨텍스트나 빈 팩토리를 컨테이너 혹은 IoC 컨테이너라고 부른다.
    - 컨테이너라는 말 자체가 IoC의 개념을 담고 있다. 스프링 컨테이너라고 부르는 걸 선호하는 사람들도 있다.
    - ApplicationContext 인터페이스를 구현한 오브젝트를 가르키기도한다. 애플리케이션 컨텍스트 오브젝트는 하나의 애플리케이션에서 보통 여러개가 만들어져서 사용된다. 이를 통틀어서 스프링 컨테이너라고 부른다.
- 스프링 프레임워크
    - IoC 컨테이너, 애플리케이션 컨텍스트를 포함해서 스프링이 모든 기능을 통틀어 말할 때 주로 사용

# 6. 싱글톤 레지스트리와 오브젝트 스코프

## 오브젝트의 동일성과 동등성

- 두 개의 오브젝트가 같은가라는 말은 주의해서 사용해야 한다.
- 두 개의 오브젝트가 완전이 같은 동일한(idential) 오브젝트라고 말하는 것과 동일한 정보를 담고 있는(equivalent) 오브젝트라고 말하는 것은 차이가 있다. 전자는 동일성(identity) 비교이며 후자는 동등성(equality) 비교라고 한다. 동일성은 == 연산자로, 동등성은 equals() 메소드를 이용해 비교한다.
- 두개의 오브젝트가 동일하다면 하나의 오브젝트만 존재하며 두 개의 오브젝트 레퍼런스 변수를 갖고 있을 뿐이다. 두 개의 오브젝트가 동일하지는 않지만 동등한 경우에는 두 개의 각기 다른 오브젝트가 메모리상에 존재하는 것인데 오브젝트의 동등성 기준에 따라 두 오브젝트의 정보가 동등하다고 판단하다는 것일 뿐이다. 물론 동일한 오브젝트는 동등하기도 할 것이다.
- 자바 클래스 equals() 메서드를 따로 구현하지 않았다면 최상위 클래스인 Object 클래스에 구현되어 있는 equals() 메소드가 사용된다. Object의 equals() 메소드는 두 오브젝트의 동일성을 비교해서 그 결과를 돌려준다.(메모리 참조값을 비교) 따라서 동일한 오브젝트여양지 동등한 오브젝트라고 여겨질 것이다.

```kotlin
@Configuration
public class DaoFactory {
    @Bean
    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }
}
```

```kotlin
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(DaoFactory.class);
UserDao userDao = applicationContext.getBean("userDao", UserDao.class);
UserDao userDao2 = applicationContext.getBean("userDao", UserDao.class);
System.out.println(userDao);
System.out.println(userDao2);
System.out.println(userDao == userDao2);
/*
springbook.user.dao.UserDao@74e52303
springbook.user.dao.UserDao@74e52303
true
 */
```

위의 코드처럼 userDao 빈을 호출하면 동일한 객체가 반환된다. 처음에 우리가 알아봤던 오브젝트 팩토리와 스프링 애플리케이션 컨텍스트에는 차이가 있다. 스프링은 여러 번에 빈을 요청하더라도 매번 동일한 오브젝트를 돌려주는 것이다.

## 싱글톤 레지스트리로서의 애플리케이션 컨텍스트

- 애플리케이션 컨텍스트는 IoC 컨테이너이면서 싱글톤을 저장하고 관리하는 **싱글톤 레지스트리(singletone registry)**이기도 하다.
- 별다른 설정을 하지 않으면 내부에서 생성하는 빈 오브젝트를 모두 싱글톤으로 만든다. 디자인 패턴 싱글톤 패턴과는 비슷하지만 구현 방법은 다르다.

### 서버 애플리케이션과 싱글톤

- 스프링은 자바 엔터프라이즈 기술을 사용하는 서버환경에 주로 적용된다.
- 스프링이 처음 설계되었을 때는 높은 성능이 요구되는 서버 환경이었다. 또한 하나의 요청을 처리하기 위해 다양한 기능을 담당하는 오브젝트들이 참여하는 계층형 구조이다.
- 자바의 오브젝트 생성과 가비지 컬렉션의 성능이 좋아졌다고 한들 매번 요청에 다양한 오브젝트를 생성하면 부하가 걸려 서버가 감당하기 어렵다.
- 서블릿은 자바 엔터프라이즈 기술의 가장 기본이 되는 서비스 오브젝트이다. 서블릿은 대부분 멀티스레드 환경에서 싱글톤으로 동작한다. 서블릿 클래스당 하나의 오브젝트만 만들어두고 요청을 담당하는 여러 스레드에서 하나의 오브젝트를 공유해 동시에 사용한다.
- 애플리케이션 안에 제한된 수, 대개 한 개의 오브젝트만 만들어서 사용하는 것이 싱글톤 패턴의 원리이다.
- 하지만 디자인 패턴에 소개된 싱글톤 패턴은 사용하기 까다롭고 여러가지 문제점이 있다. 싱글톤 패턴을 안티패턴(anti pattern)이라고 부르는 사람도 있다.

## 싱글톤 패턴

- Gof가 소개한 디자인 패턴 중의 하나이다.
- 가장 자주 활용되지만 가장 많은 비판을 받는 패턴이기도 하다.
    - GoF 멤버조차도 매우 조심해서 사용해야 하거나 피해야한다고 말할정도
- 어떤 클래스를 애플리케이션 내에서 제한된 인스턴스 개수, 이름처럼 주로 하나만 존재하도록 강제하는 패턴
- 하나만 만들어지는 클래스의 오브젝트는 애플리케이션 내에서 전역적으로 접근이 가능
- 단일 오브젝트만 존재해야 하고 애플리케이션의 여러 곳에서 공유하는 경우에 주로 사용

### 싱글톤의 한계

**자바에서 싱글톤을 구현하는 방법**

- 클래스 밖에서는 오브젝트를 생성하지 못하도록 생성자를 private으로 만든다.
- 생성된 싱글톤 오브젝트를 저장할 수 있는 자신과 같은 타입의 스태틱 필드를 정의한다.
- 스태틱 팩토리 메소드인 getInstance()를 만들고 메소드가 최초로 호출되는 시점에서 한번만 오브젝트가 만들어지게 한다. 생성된 오브젝트는 스태틱 필드에 저장된다. 또는 스태틱 필드의 초기값으로 오브젝트를 미리 만들어둘 수도 있다.
- 한번 오브젝트(싱글톤)가 만들어지고 난 후에는 getInstance() 메소드를 통해 이미 만들어져 스태틱 필드에 저장해둔 오브젝트를 넘겨준다.

**싱글톤 패턴 구현 방식 문제**

- private 생성자를 갖고 있기 때문에 상속할 수 없다
    - 싱글톤 클래스 자신만이 자기 오브젝트를 만들도록 제한한다.
    - 다른 생성자가 없다면 상속이 불가능하다.
    - 상속과 다형성을 적용할 수 없다.
        - 객체지향적인 설계의 장점을 적용하기 어렵다.
    - static 필드와 메소드를 사용하는 것도 객체지향의 특징이 적용되지 않아 문제가 발생한다.
- 테스트하기 힘들다.
    - 만들어지는 방식이 제한적이기 때문에 목 오브젝트 등으로 대체하기 어렵다.
    - 초기화 과정에서 생성자 등을 통해 사용할 오브젝트를 다이나믹하게 주입하기 힘들기 때문에 필요한 오브젝트는 직접 오브젝트를 만들어서 사용해야 한다.
    - 테스트용 오브젝트로 대체하기 어렵다. 테스트는 엔터프라이즈 개발의 핵심이다.
- 서버환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못한다.
    - 서버에서 클래스 로더를 어떻게 구성하고 있느냐에 따라서 싱글톤 클래스임에도 하나 이상의 오브젝트가 만들어질 수 있다. 따라서 자바 언어를 이용한 싱글톤 패턴 기법은 서버환경에서는 싱글톤이 꼭 보장된다고 볼 수 없다.
    - 여러개의 JVM에 분산돼서 설치가 되는 경우에도 각각 독립적으로 오브젝트가 생기기 때문에 싱글톤으로서의 가치가 떨어진다.
- 싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 바람직하지 못하다
    - 싱글톤은 사용하는 클라이언트가 정해져있지 않다.
    - 스태틱 메소드를 이용해 언제든지 싱글톤에 쉽게 접근할 수 있기 때문에 애플리케이션 어디서든지 사용될 수 있고 전역 상태(global state)로 사용되기 쉽다.
    - 아무 객체나 자유롭게 접근하고 수정하고 공유할 수 있는 전역 상태를 갖는 것은 객체지향 프로그래밍에서 권장하지 않는 방법이다.
    - 그럴 바에는 스태틱 필드와 메소드로만 구성된 클래스를 사용하는 편이 낫다.

## 싱글톤 레지스트리

- 스프링은 서버환경에서 싱글톤이 만들어져서 서비스 오브젝트 방식으로 사용하는 것을 적극 지지한다.
- 하지만 위에서 봤듯이 자바의 싱글톤 패턴 구현 방식은 단점이 많다.
- 스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공한다.
- 그것이 바로 싱글톤 레지스트리(singleton registry)이다.
- 스프링 컨테이너는 싱글톤을 생성하고 관리하고 공급하는 싱글톤 관리 컨테이너이기도 하다.
- 싱글톤 레지스트리의 장점
    - 스태틱 메소드와 private 생성자를 사용해야 하는 비정상적인 클래스가 아니라 평범한 자바 클래스를 싱글톤으로 활용하게 해준다.
    - IoC 방식의 컨테이너를 사용해서 생성과 관계설정, 사용 등에 대한 제어권을 컨테이너에게 넘기면 손쉽게 싱글톤 방식으로 만들어져 관리되게 할 수 있다. 오브젝트 생성에 관한 모든 권한은 IoC 기능을 제공하는 애플리케이션 컨텍스트에게 있기 때문이다.wwwww
- 싱글톤 레지스트리 덕분에 애플리케이션 클래스라도 public 생성자를 가질 수 있다.
- 테스트 환경에서도 자유롭게 오브젝트를 만들 수 있고 Mock 오브젝트로 대체하는 것도 간단하다.
- 싱글톤 패턴과 달리 스프링이 지지하는 객체지향적인 설계 방식과 원칙, 디자인 패턴(싱글톤 패턴은 제외) 등을 적용하는데 아무런 제약이 없다.
    - 스프링이 빈을 싱글톤으로 만드는 것은 오브젝트의 생성 방법을 제어하는 IoC 컨테이너로서의 역할이다.
    - 고전적인 싱글톤 패턴을 대신해서 싱글톤을 만들고 관리하는 것은 싱글톤 레지스트리이다.

## 싱글톤과 오브젝트의 상태

- 멀티스레드 환경이라면 여러 스레드가 동시에 접근해서 사용할 수 있다.
- 싱글톤이 멀티스레드 환경에서 서비스 형태의 오브젝트로 사용되는 경우에 상태정보를 내부에 갖고 있지 않은 무상태(sateless) 방식으로 만들어야 한다.
    - 싱글톤 오브젝트의 인스턴스 변수를 수정하는 것은 매우 위험하다.
    - 따라서 인스턴스 필드의 값을 변경하고 유지하는 상태유지(stateful)방식으로 만들지 않는다.
- 상태가 없는 방식으로 클래스를 만드는 경우에 각 요청에 대한 정보나 DB나 서버의 리소스로부터 생성한 정보는 어떻게 다뤄야할까?
    - 파라미터와 로컬 변수, 리턴 값 등을 이용해야 한다.
- 싱글톤 오브젝트가 다른 싱글톤 빈을 사용하는 인스턴스 변수로는 사용해도 괜찮다. 해당 객체도 싱글톤이기 때문이다.
- 읽기전용 속성을 가진 정보도 싱글톤에서 인스턴스 변수로 사용해도 좋다. 읽기전용 값은 static final로 선언하는 편이 낫다.

## 스프링 빈의 스코프

- 스프링이 관리하는 오브젝트, 빈이 생성되고 존재하고 적용되는 범위를 빈의 스코프(scope)라고 한다.
- 스프링 빈의 기본 스코프는 싱글톤이다.
- 싱글톤 스코프는 컨테이너 내에 한 개의 오브젝트만 만들어져서 가엦로 제거하지 않는 한 스프링 컨테이너가 존재하는 동안 계속 유지된다.
- 경우에 따라서 싱글톤 외의 스코프를 가질 수 있다. 대표적으로 프로토 타입(prototype) 스코프가 있다. 싱글톤과 달리 컨테이너에 빈을 요청할 때마다 매번 새로운 오브젝트를 만들어준다. 그 외에도 웹을 통해 새로운 HTTP 요청이 생길때마다 생성되는 요청(request) 스코프와 세션(session) 스코프가 있다.

# 7. 의존관계 주입(DI)

## 제어의 역전(IoC)과 의존관계 주입

- 객체지향 설계나 디자인 패턴, 컨테이너에서 동작하는 서버 기술을 사용한다면 자연스럽게 IoC를 적용하거나 그 원리로 동작하는 기술로 사용하게 된다.
- 스프링이 제공하는 IoC 방식을 핵심으로 짚어주는 의존관계 주입(Dependency Injection)이라는 의도가 명확히 드러나는 이름을 사용한다. 그래서 DI 컨테이너라고도 많이 불린다.

## 의존관계 주입, 의존성 주입, 의존 오브젝트 주입?

- 의존성이라는 말은 DI의 의미가 무엇인지 잘 드러내주지 못한다.
- 오브젝트는 다른 오브젝트에 주입할 수 있는 게 아니라 오브젝트의 레퍼런스가 전달될 뿐이다.
- DI는 오브젝트 레퍼런스를 외부로부터 제공(주입) 받고 이를 통해 여타 오브젝트와 다이나믹하게 의존관계가 만들어지는 것이 핵심이다.
- 용어는 동작방식(매커니즘)보다는 의도를 가지고 이름짓는 것이 좋다. 그런 면에서 의존관계 주입이라는 번역이 적절하다.

## 런타임 의존관계 설정

### 의존관계

- 의존한다는 것은 의존대상이 변하면 그 객체를 의존하고 있는 개체에 영향을 끼치는 것이다. 의존 대상에 새로운 메소드가 추가되거나 기존 메소드가 바뀌면 그 객체의 코드도 변경해야 하는 경우다. 결과적으로 영향을 미치게 되므로 의존관계가  있다고 볼 수 있다.
- 의존 관계는 방향성이 있다. A가 B에 의존해도 B는 A에 의존하지 않을 수 있다. 의존하지 않는다는 것은 A가 변해도 B가 영향을 받지 않는 다는 것이다.

- UML에서 말하는 의존관계란 설계모델의 관점에서 말한다. 그런데 모델이나 코드에서 클래스와 인터페이스를 통해 드러나는 의존관계말고 런타임시에 오브젝트 사이에서 만들어지는 의존관계도 있다.
- 런타임 의존관계 또는 오브젝트 의존관계인데 설계 시점의 의존관계과 실체화된 것이라고 볼 수 있다. 런타임 의존관계와 모델링 시점의 의존관계는 성격이 다르다.
- 인터페이스를 통해 설계 시점에 의존관계를 갖는 경우에는 런타임에 어느 오브젝트가 구현체로 들어올 지 미리 알 수 없다. 설계와 코드 속에서는 구현체가 드러나지 않는 것이다. 프로그램이 시작되고 오브젝트가 만들어지고 나서 런타임시에 의존관계를 맺는 대상, 즉 실제 사용대상인 오브젝트를 의존 오브젝트(dependent object)라고 한다. 의존관계 주입은 클라이언트라고 부르는 오브젝트를 런타임 시에 연결해주는 작업을 말한다.

**의존관계 주입이란 다음 세가지를 만족하는 작업을 말한다.**

- 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스에만 의존하고 있어야 한다.
- 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.
- 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.

의존관계 주입의 핵심은 설계 시점에 알지 못했던 두 오브젝트의 관계를 맺도록 도와주는 제 3의 존재가 있다는 것이다. DI에서 말하는 제 3의 존재는 바로 관계설정 책임을 가진 코드를 분리해서 만들어진 오브젝트이다. 애플리케이션 컨텍스트, 빈 팩토리, IoC 컨테이너등이 이에 해당한다.

![image](https://user-images.githubusercontent.com/66561524/190833575-cc82281c-3c50-4aff-95fe-a8fabc18a0d9.png)

DI는 자신이 사용할 오브젝트에 대한 선택과 생성 제어권을 외부로 넘기고 자신은 수동적으로 주입받은 오브젝트를 사용한다는 점에서 IoC의 개념에 잘 들어맞는다. 스프링 컨테이너의 IoC는 주로 의존관계 주입 또는 DI라는 데 초점이 맞춰져있다.

## 의존관계 검색과 주입

- 스프링이 제공하는 IoC 방법에는 의존관계 주입만 있는 것이 아니다. 구체적인 클래스에 의존하지 않고 런타임시에 의존관계를 결정한다는 점에서 의존관계 주입과 비슷하지만 의존관계를 맺는 방법이 외부로부터 주입이 아니라 스스로 검색을 이용하기 떄문에 **의존관계 검색(dependency lookup)**이라고 불리는 것이 있다.
- 의존관계 검색은 자신이 필요로 하는 의존 오브젝트를 능동적으로 찾는다.
- 의존관계 검색은 런타임시 의존관계를 맺을 오브젝트를 결정하는 것과 오브젝트의 생성 작업은 외부 컨테이너에게 IoC로 맡기지만 이를 가져올 때는 메소드나 생성자를 통한 주입 대신 스스로 컨테이너에게 요청하는 방법을 사용한다.
- 의존관계 검색은 기존 의존관계 주입의 거의 모든 장점을 가지고 있다.
- 하지만 컴포넌트가 다른 오브젝트에 의존하게 되므로 바람직하지 않다.

### 의존관계 검색과 의존관계 주입 차이점

- 의존관계 검색 방식에서는 검색하는 오브젝트는 자신이 스프링의 빈일 필요가 ㅇ벗다.
- 의존관계 주입에서는 DI가 적용되려면 의존을 받는 객체는 스프링 빈 오브젝트여야 한다.
    - 의존을 받는 객체에 대해 생성과 초기화 권한을 스프링 컨테이너가 갖고 있어야 하기 때문이다.
- 이런 점에서 DI와 DL은 적용 방법에 차이가 있다.

> **DI 받는다**
외부에서 파라미터로 오브젝트를 넘겨줬다고 해서 모두 DI가 아니다. 주입받는 메소드 파라미터가 이미 특정 클래스 타입으로 고정되어 있으면 DI가 일어날 수 없다. DI에서 말하는 주입은 다이나믹하게 구현 클래스를 결정해서 제공받을 수 있도록 인터페이스의 파라미터를 통해서 이뤄줘야 한다.

## 의존관계 주입의 응용

- DI 기술의 장점
    - 코드에는 런타임 클래스에 대한 의존관계가 나타나지 않고 인터페이스를 통해 결합도가 낮은 코드를 만드므로 다른 책임을 가진 사용 의존관계 있는 대상이 바뀌거나 변경되더라도 자신은 영향 받지 않고 변경을 통한 다양한 확장 방법에는 자유롭다.
    

### 기능 구현의 교환

- 환경별(Profile) 설정을 바꿀 수 있다.

### 부가기능 추가

- 스프링은 DI를 편하게 사용할 수 있도록 도와주는 도구이면서 그 자체로 DI를 적극 활용한 프레임워크이다.

## 메소드를 이용한 의존관계 주입

- 의존관계 주입시 반드시 생성자를 사용할 필요는 없다. 일반 메소드를 사용할 수 있을 뿐만 아니라 생성자를 사용하는 방법보다 더 자주 사용된다.

### 수정자(setter) 메소드를 이용한 주입

- setter 메소드는 외부에서 오브젝트 내부의 attribute 값을 변경하려는 용도로 주로 사용된다.
- setter는 외부로부터 제공받은 오브젝트 레퍼런스를 저장해뒀다가 내부의 메소드에서 사용하게 하는 DI 방식에서 활용하기에 적당하다.
    - 부가적으로 validation 및 다른 작업을 추가할 수 있다.

### 일반 메소드를 이용한 주입

- setter는 set으로 시작해야하고 한 번에 한 개의 파라미터만 가질 수 있다.
- 한 번에 한 개의 파라미터만 가질 수 있다는 제약이 싫다면 여러 개의 파라미터를 갖는 일반 메소드를 DI용으로 사용할 수 있다.
- 생성자가 setter 보다 나은 점은 한 번에 여러 개의  파라미터를 받을 수 있다는 점이다.
    - 하지만 파라미터의 개수가 많아지고 비슷한 타입이 늘어나면 실수하기 쉽다.
- XML을 사용하는 경우 자바빈 규약을 따르는 setter 주입이 편하다.
    - 이름을 잘 지어야 한다.
    

# 8. XML을 이용한 설정

생략

# 9. 정리

- 책임이 다른 코드를 분리해서 두 개의 클래스로 만든다. **(관심사의 분리, 리팩토링)**
- 그 중에서 바뀔 수 있는 쪽의 클래스는 인터페이스를 구현하도록 하고 다른 클래스에서 인터페이스를 통해서만 접근하도록 만들었다. 인터페이스를 정의한 쪽의 구현 방법이 달라져 클래스가 바뀌더라도 그 기능을 사용하는 클래스의 코드는 같이 수정할 필요가 없도록 만들었다. **(전략패턴)**
- 자신의 책임 자체가 변경되는 경우 외에는 불필요한 변화가 발생하지 않도록 막아주고 자신이 사용하는 외부 오브젝트의 기능은 자유롭게 확장하거나 변경할 수 있게 만들었다. **(개방 폐쇄 원칙)**
- 한쪽 기능의 변화가 다른 쪽의 변화를 요구하지 않아도 되게 했고**(낮은 결합도)**, 자신의 책임과 관심사에만 순수하게 집중하는**(높은 응집도)** 깔끔한 코드를 만들 수 있었다.
- 오브젝트가 생성되고 여타 오브젝트와 관계를 맺는 작업의 제어권을 별도의 오브젝트 팩토리를 만들어 넘겼다. 오브젝트 팩토리의 기능을 일반화한 IoC 컨테이너로 넘겨서 오브젝트가 자신이 사용할 대상의 생성이나 선택에 관한 책임으로부터 자유롭게 만들어줬다. **(제어의 역전/IoC)**
- 전통적인 싱글톤 패턴 구현 방식의 단점과 서버에서 사용되는 서비스 오브젝트로서의 장점을 살릴 수 있는 싱글톤을 사용하면서 싱글톤 패턴의 단점을 극복할 수 있도록 설계된 컨테이너를 활용하는 방법을 알아봤다. **(싱글톤 레지스트리)**
- 설계 시점과 코드에 클래스와 인터페이스 사이의 느슨한 의존관계만 만들어놓고 런타임시에 실제 사용할 구체적인 의존 오브젝트를 제 3자(DI 컨테이너)의 도움으로 주입받아서 다이나믹한 의존관계를 갖게 해주는 IoC의 특별한 케이스를 알아봤다. **(의존관계 주입/DI)**
- 의존오브젝트를 주입할 때 생성자를 이용하는 방법과 수정자 메소드를 이용하는 방법을 알아봤다. **(생성자 주입과 수정자 주입)**

스프링의 관심은 오브젝트와 그 관계다. 오브젝트를 어떻게 설계하고 분리하고 개선하고 의존관계를 가질지 결정하는 일은 개발자의 역할과 책임이다.

# 10. 읽기모임 정리

[0904 ****hunch****](https://www.notion.so/0904-hunch-4b12b8f3ebbe49f6a17aaf407eeb96fa)