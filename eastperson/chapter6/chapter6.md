# Overview

---

- AOP는 IoC/DI, 서비스 추상화와 더불어 스프링의 3대 기반기술의 하나다.
- AOP를 바르게 이용하려면 OOP를 대체하려고 하는 것처럼 보이는 AOP의 필연적인 등장배경과 스프링이 그것을 도입한 이유 그 적용을 통해 얻을 수 있는 장점이 무엇인지에 대한 이해가 필요하다.
- 스프링에 적용된 가장 인기있는 AOP의 적용 대상은 선언적 트랜잭션(`@Transactional`)이다. 서비스 추상화를 통해 근본적인 문제를 해결했던 트랜잭션 경계설정 기능을 AOP를 이용해 세련되고 깔끔하게 변경할 수 있다.

# 1. 트랜잭션 코드의 분리

---

## 메소드 분리

![image](https://user-images.githubusercontent.com/66561524/198752230-b6bffc03-b271-464b-b954-fd1496a0f054.png)

- 트랜잭션 경계설정 코드와 비즈니스로 로직 코드가 복잡하게 얽혀 있는 듯이 보이지만 코드가 구분되어 있다.
- 비즈니스 로직 코드를 사이에 두고 트랜잭션 시작과 종료를 담당하는 코드가 있다.
- 비즈니스 로직과 트랜잭션 코드는 서로 주고받는 정보가 없다.
- 완벽하게 독립적인 코드다.

![image](https://user-images.githubusercontent.com/66561524/198752238-980243bc-ab49-4e02-948b-b1f48aaaa986.png)

- 이렇게 비즈니스 로직 코드와 트랜잭션 코드를 분리할 수 있다.
- 실수로 트랜잭션 코드를 건드릴 염려도 없다.

## DI를 이용한 클래스의 분리

- 메서드를 분리했지만 트랜잭션 담당 기술이 버젓이 `UserService`안에 있다.
- 서로 직접적으로 정보를 주고받지 없으니 트랜잭션 코드를 클래스 밖으로 뽑아내고 싶다.

### DI 적용을 이용한 트랜잭션 분리

![image](https://user-images.githubusercontent.com/66561524/198752251-1c4feecb-ccaa-44e0-a820-c863f77f3b77.png)

- 트랜잭션 코드를 어떻게든 UserService에서 빼버리면 클라이언트 코드에서는 트랜잭션이 빠진다.
- DI의 기본 아이디어는 실제 사용할 오브젝트의 클래스 정체는 감춘 채 인터페이스를 통해 간접으로 접근하는 것

![image](https://user-images.githubusercontent.com/66561524/198752261-a575d2fd-73aa-44b2-b951-05a0da13a336.png)

- `UserService`를 인터페이스로 만들고 구현 클래스에서 코드를 만들면 유연하게 만들 수 있다.
- 인터페이스를 이용해 구현 클래스를 클라이언트에 노출하지 않고 런타임시에 DI를 통해 적용하는 이유는 일반적으로 구현 클래스를 바꿔가면서 사용하기 때문이다.
    - 테스트 클래스, 정규 구현 클래스를 넣어줄 수 있다.
- `UserService`에는 순수하게 비즈니스 로직을 담고 있는 코드만 놔두고 트랜잭션 경계설정을 담당하는 코드를 외부로 빼내고 싶다. 클라이언트가 `UserService` 기능을 이용하려면 트랜잭션이 적용되어야 한다.

![image](https://user-images.githubusercontent.com/66561524/198752276-9a23a3f0-761b-454a-adfc-284be2c95467.png)

- 비즈니스 로직은 `UserServiceImpl`에 `UserServiceTx`는 트랜잭셔 코드를 가지고 있는다.

## UserService 인터페이스 도입

![image](https://user-images.githubusercontent.com/66561524/198752303-723e315b-120d-41b1-aa59-c384ffe67e7c.png)
![image](https://user-images.githubusercontent.com/66561524/198752311-ee1c271c-3528-4b2f-bf06-f9bd6cce6984.png)

- UserServiceImpl에는 비즈니스 로직만 있다.
- 트랜잭션, 시스템 환경, 스프링에 관련된 코드 등 아무것도 없다.

## 분리된 트랜잭션 기능

![image](https://user-images.githubusercontent.com/66561524/198752322-16b61b76-31fe-46f9-a155-005843b8d675.png)

- UserService 같은 인터페이스를 구현한 다른 오브젝트에게 작업을 위임하면 된다.
    - Delegate Pattern
- UserServiceTx는 비즈니스 로직에 대해서 아무런 관여도 하지 않는다.

![image](https://user-images.githubusercontent.com/66561524/198752329-d9723b4d-66c2-46e7-a837-47382528c281.png)

- 구체적인 트랜잭션 경계설정 작업을 추가하면 위와 같이 된다.
- 트랜잭션 처리 메소드와 비즈니스 로직 메서드를 분리했을 때 트랜잭션을 담당한 메소드와 한 메소드가 됐다. 추상화된 트랜잭션 구현 오브젝트를 DI 받을 수 있도록 PlatformTransactionManager 타입의 프로퍼티도 추가됐다.

## 트랜잭션 적용을 위한 DI 설정

- 클라이언트가 UserService라는 인터페이스를 통해 사용자 관리 로직을 이용하려고 할 때 먼저 트랜잭션을 담당하는 오브젝트가 사용돼서 트랜잭션에 관련된 작업을 진행해주고 실제 사용자 관리 로직을 담은 오브젝트가 이후에 호출돼서 비즈니스 로직에 관련된 작업을 수행하도록 만든다.

![image](https://user-images.githubusercontent.com/66561524/198752350-e5dd3236-5abe-47fc-90e6-9e09645a11aa.png)

- transactionManager는 UserServiceTx의 빈이 들어간다.

## 트랜잭션 경계설정 코드 분리의 장점

**트랜잭션 경계설정 코드의 분리와 DI를 통한 연결로 인한 장점**

1. 비즈니스 로직을 담당하고 있는 `UserServiceImpl`의 코드를 작성할 때는 트랜잭션과 같은 기술적인 내용에는 신경쓰지 않아도 된다.
    1. 트랜잭션의 적용이 필요한지도 신경 쓰지 않아도 된다. 스프링의 JDBC나 JTA 같은 로우레벨의 트랜잭션 API는 물론이고 스프링의 트랜잭션 추상화 API 조차 필요없다.
    2. 트랜잭션은 DI를 이용해 `UserServiceTx`와 같은 트랜잭션 기능을 가진 오브젝트가 먼저 실행되도록 만들기만 하면 된다.
    3. 스프링이나 트랜잭션의 로우레벨의 기술적인 지식은 부족한 개발자라고 하더라도 비즈니스 로직을 잘 이해하고 자바 언어의 기초에 충실하면 복잡한 비즈니스로 로직을 담은 `UserService` 클래스를 개발할 수 있다.
2. 비즈니스 로직에 대한 테스트를 손쉽게 만들 수 있다.

# 2. 고립된 단위 테스트

---

- 가장 편하고 좋은 테스트 방법은 가능한 작은 단위로 쪼개서 테스트하는 것이다.
    - 테스트가 실패했을 때 원인을 찾기 쉽다.
        - 오류가 발견 됐을 때 진행되는 동안 실행된 코드의 양이 많다면 원인을 찾기가 힘들다.
    - 테스트의 의도나 내용이 분명해지고 만들기 쉽다.
- 처음부터 작은 단위로 테스트하면서 진행해왔다면 덩치가 커져도 어렵지 않게 오류를 찾을 수 있다.
- 테스트로 검증한 부분은 제외하고 접근할 수 있기 때문이다.
- 작은 단위로 테스트하고 싶어도 그럴 수 없는 경우가 많다.

## 복잡한 의존관계 속의 테스트

- `UserService`는 세 가지 타입의 의존 오브젝트가 필요하다. `UserDao`, `MailSender`, `PlaforTransactionManager`

![image](https://user-images.githubusercontent.com/66561524/198752365-df5122f9-d5dc-490e-b2bc-dcb0ae38b9de.png)

- `UserService`의 코드를 테스트할 때 이 세가지 의존관계의 오브젝트가 같이 실행된다. 따라서 그 세 가지 의존관계를 갖는 오브젝트들이 테스트가 진행되는 동안에 같이 실행된다.
- 더 큰 문제는 세가지 오브젝트가 갖고 있는 의존관계다. DataSource의 구현 클래스와 DB 드라이버, DB 서버까지의 네트워크 통신, DB 서버 자체, 안에 정의된 테이블 모두 의존한다. JTA 였다면 JTA 구현 오브젝트와 WAS 서버의 트랜잭션 서비스에까지 의존하고 있을 것이다. 메일도 마찬가지다.
- 따라서 `UserService` 테스트가 아니라 오브젝트와 환경, 서비스, 서버, 네트워크까지 테스트를 하는 셈이다.
- `UserService`는 문제가 없는데도 누군가 `UserDao`의 코드를 잘못 수정해서 그 오류 때문에 `UserService`의 테스트가 실패한다면 그 원인을 찾느라고 불필요한 시간을 낭비해야 할 수도 있다.
- 따라서 테스트를 준비하기도 힘들고 환경이 달라지면 동일한 테스트 결과를 내지 못할 수 있으며 수행속도는 느리고 테스트를 작성하고 실행하는 빈도가 떨어지게 된다. 그 오류 때문에 테스트가 실패하면 원인을 찾느라고 불필요한 시간을 낭비해야 할수도 있다.
- DB가 함께 동작해야 하는 테스트는 작성하기 힘든 경우도 많다.

## 테스트 대상 오브젝트 고립시키기

- 테스트의 대상이 환경, 외부 서버, 다른 클래스 코드에 종속되고 영향을 받지 않도록 고립시킬 필요가 있다.
- 테스트를 의존 대상으로부터 분리해서 고립 시키는 방법은 테스트를 위한 대역을 사용하는 것이다.

### 테스트를 위한 UserServiceImpl 고립

- 고립된 테스트가 가능하도록 UserSerivce를 재구성해보면 다음과 같은 구조가 된다.

![image](https://user-images.githubusercontent.com/66561524/198752379-2e2b9e23-b7ec-4f11-9928-251823e6e275.png)

- 두 개의 목 오브젝트에만 의존하는 고립된 테스트 대상으로 만들 수 있다.
- `UserDao`는 테스트 대상의 코드가 정상적으로 수행되도록 도와주기만 하는 스텁이 아니라 부가적인 검증 기능까지 가진 목 오브젝트로 만들었다.
- 테스트의 검증에 사용할 수 있게 하는 목 오브젝트를 만들 필요가 없다.

### 고립된 단위 테스트 활용

![image](https://user-images.githubusercontent.com/66561524/198752400-7f5c158e-d9a2-4f33-bd81-c3e40190250b.png)

1. 테스트 실행 중에 `UserDao`를 통해 가져올 테스트용 정보를 DB에 넣는다. `UserDao`는 결국 DB를 이용해 정보를 가져오기 때문에 최후의 의존 대상인 DB에 직접 정보를 넣어줘야 한다.
2. 메일 발송 여부를 확인하기 위해 `MailSender` 목 오브젝트를 DI 해준다.
3. 실제 테스트 대상인 `userService`의 메소드를 실행한다.
4. 결과가 DB에 반영됐는지 확인하기 위해서 `UserDao`를 이용해 DB에서 데이터를 가져와 결과를 확인한다.
5. 목 오브젝트를 통해 UserService에 의한 메일 발송이 있었는지를 확인하면 된다.

## 테스트 수행 성능의 향상

- DB까지 연동하는 의존 대상이 모두 포함된 테스트다.
- DB를 이용하는 테스트와목 오브젝트만을 이용하는 테스트의 수행시간을 비교하면 몇백배의 차이가 난다.
- 고립된 테스트를 하면 테스트가 다른 의존 대상에 영향을 받을 경우를 대비해 복잡하게 준비할 필요가 없을 뿐만 아니라 테스트 수행 성능도 크게 향상된다.
- 고립된 테스트를 만들려면 목 으브젝트 작성과 같은 수고가 더 필요할지 모르겠지만 보상은 충분히 기대할만 하다.

## 단위 테스트와 통합 테스트

- 단위 테스트의 단위는 정하기 나름이다.
    - 사용자 관리 기능을 단위로 볼 수 있다.
    - 하나의 클래스나 하나의 메소드를 단위로 볼 수 있다.
    - **중요한 것은 하나의 단위에 초점을 맞춘 테스트라는 점**이다.
- 책에서는 ‘테스트 대상 클래스를 목 오브젝트 등의 테스트 대역을 이용해 의존 오브젝트나 외부의 리소스를 사용하지 않도록 고립시켜서 테스트 하는 것’을 ***단위테스트***라고 부르려고 한다.
- 책에서는 ‘두 개 이상의 성격이나 계층이 다른 오브젝트가 연동하도록 만들어 테스트하거나 또는 외부의 DB나 파일, 서비스 등의 리소스가 참여하는 테스트는 ***통합 테스트***라고 부르려고 한다.

### 단위 테스트와 통합 테스트 중 어떤 방법을 쓸지 결정하는 방법 (가이드 라인)

- 항상 단위 테스트를 먼저 고려한다.
- 하나의 클래스나 성격과 목적이 같은 긴밀한 클래스 몇 개를 모아서 외부와의 의존관계를 모두 차단하고 필요에 따라 스텁이나 목 오브젝트 등의 테스트 대역을 이용하도록 테스트를 만든다. 단위 테스트는 테스트 작성도 간단하고 실행 속도도 빠르며 테스트 대상 외의 드나 환경으로부터 테스트 결과에 영향을 받지도 않기 때문에 가장 빠른 시간에 효과적인 테스트를 작성하기에 유리하다.
- 외부 리소스를 사용해야만 가능한 테스트는 통합 테스트로 만든다.
- 단위 테스트로 만들기가 어려운 코드도 있다. 대표적인 게 DAO다. DAO는 그 자체로 로직을 담고 있기보다는 DB를 통해 실행하는 코드만으로는 고립된 테스트를 작성하기가 힘들다. 작성한다고 해도 가치가 없는 경우가 대부분이다. 따라서 DAO는 DB까지 연동하는 테스트로 만드는 편이 효과적이다. DB를 사용하는 테스트는 DB에 테스트 데이터를 준비하고 DB에 직접 확인을 하는 등의 부가적인 작업이 필요하다.
    - DAO 테스트는 DB라는 외부 리소스를 사용하기 때문에 통합 테스트로 분류된다. 하지만 코드에서 보자면 하나의 기능 단위를 테스트하는 것이기도 하다. DAO를 테스트를 통해 충분히 검증해두면 DAO를 이용하는 코드는 DAO 역할을 스텁이나 목 오브젝트로 대체해서 테스트할 수 있다. 이후에 실제 DAO와 연동했을 때도 바르게 동작하리라고 확실할 수 있다. 물론 각각의 단위 테스트가 성공했더라도 여러 개의 단위를 연결해서 테스트하면 오류가 발생할 수도 있다. 하지만 충분한 단위 테스트를 거친다면 통합 테스트에서 오류가 발생할 확률도 줄어들고 발생한다고 하더라도 쉽게 처리할 수 있다.
- 여러 개의 단위가 의존관계를 가지고 동작할 때를 위한 통합 테스트는 필요하다. 다만 단위 테스트를 충분히 거쳤다면 통합 테스트의 부담은 상대적으로 줄어든다.
- 단위 테스트를 만들기가 너무 복잡하다고 판단되는 코드는 처음부터 통합 테스트를 고려해본다. 이때도 통합 테스트에 참여하는 코드 중에서 가능한 많은 부분을 미리 단위 테스트로 검증하는 게 유리하다.
- 스프링 테스트 컨텍스트 프레임워크를 이용하는 테스트는 통합 테스트다. 가능하면 스프링의 지원 없이 직접 코드 레벨의 DI를 사용하면서 단위 테스트를 하는 게 좋겠지만 스프링의 설정 자체도 테스트 대상이고 스프링을 이용해 좀 더 추상적인 레벨에서 테스트해야 할 경우도 종종 있다. 리럴 때 스프링 테스트 컨텍스트 프레임워크를 이용해 통합 테스트를 작성한다.

단위 테스트와 통합 테스트는 개발자가 스스로 자신이 만든 코드를 테스트하기 위해 만드는 개발자 테스트다.

- QA나 고객에 의해 진행되는 테스트와 다른 관점이다.
- 모든 코드가 작성되고 빠르게 진행되는 편이 좋다.
- TDD는 코드를 만들자마자 바로 테스트가 가능하다는 장점이 있다.
- 코드를 작성하면서 테스트를 어떻게 작성할지를 생각하는 것은 좋은 습관이다.

- 스프링이 지지하고 권장하는 깔끔하고 유연한 코드를 만들다보면 테스트도 그만큼 만들기 쉬워지고 테스트는 다시 코드의 품질을 높여주고 리팩토링과 개선에 대한 용기를 주기도 할 것이다.
- 반대로 좋은 코드를 만들려는 노력을 게을리하면 테스트 작성이 불편해지고 테스트를 잘 만들지 않게 될 가능성이 높아진다.
- 테스트가 없으니 과감하게 리팩토링할 엄두를 내지 못하고 코드의 품질은 점점 떨어지고 유연성과 확장성을 잃어갈지 모른다.

## 목 프레임워크

- 단위 테스트를 만들기 위해서는 스텁이나 목 오브젝트의 사용이 필수적이다.
    - 의존관계가 없는 단순한 클래스나 세부 로직을 검증하기 위해 메소드 단위로 테스트할 때가 아니라면 대부분 의존 오브젝트를 필요로 하는 코드를 테스트하게 된다.
- 단위 테스트의 작성은 장점이 많지만 번거롭다.
    - 목 오브젝트를 만드는 일은 가장 큰 짐이다.
        - 테스트에서는 사용하지 않는 인터페이스도 모두 일일히 구현해줘야 한다.
    - 테스트 메소드별로 다른 검증 기능이 필요하다면 같은 의존 인터페이스를 구혀한 여러 개의 목 클래스를 선언해줘야 한다.
    - 다행이도 목 오브젝트를 편리하게 작성하도록 도와주는 다양한 목 오브젝트 지원 프레임워크가 있다.

### Mockito 프레임워크

- Mockito 프레임워크는 사용하기 편리하고 코드도 직관적이라 인기가 많다.
- Mockito와 같은 목 프레임워크의 특징은 목 클래스를 일일이 준비해둘 필요가 없다는 점이다.
- 인터페이스를 구현한 클래스를 만들 필요도 없고 리턴 값을 생성자를 통해 넣어줬다가 메소드 호출시 리턴하도록 코드를 만들 필요가 없다.

Mockitio 목 오브젝트는 다음 네 단계를 거쳐서 사용하면 된다.

- 인터페이스를 이용해 목 오브젝트를 만든다.
- 목 오브젝트가 리턴할 값이 있으면 이를 지정해준다. 메소드가 호출되면 예외를 강제로 던지게 만들 수 있다.
- 테스트 대상 오브젝트에 DI 해서 목 오브젝트가 테스트 중에 사용되도록 만든다.
- 테스트 대상 오브젝트를 사용한 후에 목 오브젝트의 특정 메소드가 호출됐는지 어떤 값을 가지고 몇 번 호출됐는지를 검증한다.

# 3. 다이내믹 프록시와 팩토리 빈

---

## 프록시와 프록시 패턴, 데코레이터 패턴

- 트랜잭션 기능을 분리하면 전형적인 전략 패턴을 사용하면 된다.
    - 하지만 전략 패턴으로는 트랜잭션 기능의 구현을 분리해냈을 뿐이다.

![image](https://user-images.githubusercontent.com/66561524/198752410-1a4fe3bb-ab14-4ee7-9227-b2a6e91da73b.png)

- 트랜잭션과 같은 부가적인 기능을 위임을 통해 외부로 분리했을 때의 결과이다.
- 구체적인 구현 코드는 제거했을지라도 위임을 통해 기능을 사용하는 코드는 핵심 코드와 함께 남아있다.
- 트랜잭션이라는 기능은 사용자 관리 비즈니스 로직과는 성격이 다르기 때문에 아예 그 적용 사실 자체를 밖으로 분리할 수 있다.

![image](https://user-images.githubusercontent.com/66561524/198752424-56da3135-fa96-4364-aff3-623741a147a4.png)

- 이렇게 분리된 부가기능을 담은 클래스는 중요한 특징이 있다.
    - 부가기능 외의 나머지 모든 기능은 원래 핵심기능을 가진 클래스로 위임해줘야 한다.
    - 핵심기능은 부가기능을 가진 클래스의 존재 자체를 모른다. 따라서 부가기능이 핵심기능을 사용하는 구조가 되는 것이다.
- 문제는 클라이언트가 핵심기능을 가진 클래스를 직접 사용해버리면 부가기능이 적용될 기회가 없다.
    - 부가기능이 마치 자신이 핵심 기능을 가진 클래스인 것 처럼 꾸며서 클라이언트가 자신을 거쳐서 핵심기능을 사용하도록 만들어야 한다.
    - 클라이언트는 인터페이스를 통해서만 핵심기능을 사용하게 하고 부가기능 자신도 같은 인터페이스를 구현한 뒤에 자신이 그 사이에 끼어들어야 한다.
    - 부가기능 자신도 같은 인터페이스를 구현한 뒤에 자신이 그 사이에 끼어들어야 한다.

![image](https://user-images.githubusercontent.com/66561524/198752446-5beba73b-c399-4c68-9f71-fe01d32926c8.png)

- 클라이언트는 인터페이스만 보고 사용하기 때문에 핵심기능을 가진 클래스를 사용할거라 기대하지만 부가기능을 통해 핵심 기능을 이용하게 되는 것이다.
- 부가기능 코드에서는 핵심기능으로 요청을 위임해주는 과정에서 자신이 가진 부가적인 기능을 적용해줄 수 있다.
- 이렇게 마치 자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것을 대리자, 대리인과 같은 역할을 한다고 해서 **프록시(proxy)**라고 부른다.
- 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트를 타겟(target) 또는 실체(real subject)라고 부른다. 클라이언트가 프록시를 통해 타깃을 사용하는 구조를 보여준다.

![image](https://user-images.githubusercontent.com/66561524/198752457-5b832681-4298-4415-87c4-f7d92a24dc47.png)

- 프록시의 특징은 타깃과 같은 인터페이스를 구현했다는 것과 프록시가 타깃을 제어할 수 있는 위치에 있다는 것이다.
- 프록시는 사용 목적에 따라 두 가지로 구분할 수 있다.
    1. 클라이언트가 타깃에 접근하는 방법을 제어하기 위해서다.
    2. 타깃에 부가적인 기능을 부여해주기 위해서이다.
- 두 가지 모두 대리 오브젝트라는 개념의 프록시를 두고 사용한다는 점은 동일하지만 목적에 따라서 디자인패턴에서는 다른 패턴으로 구분한다.

### 데코레이터 패턴

- 타깃에 부가적인 기능을 런타임시 다이나믹하게 부여해주기 위해 프록시를 사용하는 패턴
    - 컴파일 시점, 코드상에서는 어떤 방법과 순서로 프록시와 타깃이 연결되어 사용되는지 정해져있지 않다는 점이다.
    - 코드상에서는 어떤 방법과 순서로 프록시와 타깃이 연결되어 사용되는지 정해져있지 않다.
- 이 패턴의 이름이 데코레이터라고 불리는 이유는 마치 제품이나 케익 등을 여러 겹으로 포장하고 그 위에 장식을 붙이는 것처럼 실제 내용물은 동일하지만 부가적인 효과를 부여해줄 수 있기 때문이다.
    - 데코레이터 패턴에서는 프록시가 꼭 한 개로 제한되지 않는다.
    - 프록시가 직접 타깃을 사용하도록 고정시킬 필요도 없다.
    - 데코레이터 패턴에서는 같은 인터페이스를 구현한 타겟과 여러 개의 프록시를 사용할 수 있다. 프록시가 여러 개인 만큼 순서를 정해서 단계적으로 위임하는 구조로 만들면 된다.
![image](https://user-images.githubusercontent.com/66561524/198752466-0587f76c-4c22-43b2-8a20-d1bfdef32b68.png)

- 프록시로 동작하는 각 데코레이터는 위임하는 대상에 인터페이스로 접근하기 때문에 자신이 최종 타깃으로 위임하는지 아니면 다음 단계의 데코레이터 프록시로 위임하는지 알지 못한다.
- 데코레이터의 다음 위임 대상은 인터페이스로 선언하고 생성자나 setter를 통해 위임 대상을 외부에서 런타임시에 주입받을 수 있도록 만들어야 한다.
- 인터페이스를 통한 데코레이터 정의와 런타임 시의 다이나믹한 구성 방법은 스프링의 DI를 이용하면 편리하다. 데코레이터 빈의 프로퍼티로 같은 인터페이스를 구현한 다른 데코레이터 또는 타깃 빈을 설정하면 된다.
- 데코레이터 패턴은 인터페이스를 통해 위임하는 방식이기 때문에 어느 데코레이터에서 타깃으로 연결될지 코드레벨에서는 알 수 없다.
- 데코레이터 패턴은 타깃의 코드를 손대지 않고 클라이언트가 호출하는 방법도 변경하지 않은 채로 새로운 기능을 추가할 때 유용한 방법이다.

### 프록시 패턴

- 일반적으로 사용하는 프록시라는 용어와 디자인 패턴의 프록시 패턴은 구분할 필요가 있다.
    - 프록시 용어는 클라이언트와 사용 대상 사이에 대리 역할을 맡은 오브젝트를 두는 방법을 총칭한다.
    - 프록시 패턴은 프록시를 사용하는 방법 중에서 타깃에 대한 접근 방식을 제어하려는 목적을 가진 경우다.
        - 프록시 패턴의 프록시는 타깃의 기능을 확장하거나 추가하지 않는다.
        - 타깃 오브젝트를 생성하기가 복잡하거나 당장 필요하지 않은 경우에 꼭 필요한 시점까지 오브젝트를 생성하지 않는 편이 좋다. 그런데 타깃 오브젝트에 대한 레퍼런스가 미리 필요할 때 프록시 패턴을 적용한다.
        - 프록시의 메소드를 통해 타깃을 사용하려고 시도할 때 프록시가 타깃 오브젝트를 생성하고 요청을 위임해주는 식이다.
        - 레퍼런스는 갖고 있지만 끝까지 사용하지 않거나 많은 작업이 진행된 후에 사용되는 경우라면 프록시를 통해 생성을 최대한 늦춤으로써 얻는 장점이 많다.
- 원격 오브젝트를 이용하는 경우에 프록시를 사용하면 편리하다.
    - 리모팅 기술을 이용해 다른 서버에 존재하는 오브젝트를 사용해야 한다면 원격 오브젝트에 대한 프록시를 만들어두고 클라이언트는 마치 로컬에 존재하는 오브젝트를 쓰는 것처럼 프록시를 사용하게 할 수 있다.
    - 프록시는 클라이언트 요청을 받으면 네트워크를 통해 원격 오브젝트를 실행하고 결과를 받아서 클라이언트에게 돌려준다.
    - 클라이언트로 하여금 원격 오브젝트에 대한 접근 방법을 제공해주는 프록시 패턴의 예라고 할 수 있다.
- 특별한 상황에서 타깃에 대한 접근 권한을 제어하기 위해 프록시 패턴을 사용할 수 있다.
    - 수정 가능한 오브젝트가 있는데 특정 레이어에서는 읽기전용으로만 동작하게 강제해야한다고 할 때 오브젝트의 프록시를 만들어서 프록시의 특정 메소드를 사용하려고 하면 접근이 불가능하다고 예외를 발생시킬 수 있다.
- 프록시 패턴은 타깃의 기능 자체에는 관여하지 않으면서 접근하는 방법을 제어해주는 프록시를 이용하는 것이다.
    - 구조적으로 보자면 프록시와 데코레이터는 유사하다.
    - 다만 프록시는 코드에서 자신이 만들거나 접근할 타깃 클래스 정보를 알고 있는 경우가 많다.
    - 생성을 지연하는 프록시라면 구체적인 생성 방법을 알아야 하기 때문에 타깃 클래스에 대한 직접적인 정보를 알아야 한다.
    - 프록시 패턴이라고 하더라도 인터페이스를 통해 위임할 수 있도록 만들어야 한다.
    - 인터페이스를 통해 다음 호출 대상으로 접근하게 하면 그 사이에 다른 프록시나 데코레이터가 계속 추가될 수 있기 때문이다.

![image](https://user-images.githubusercontent.com/66561524/198752483-7c1b6c15-8706-4946-baec-6c618ac00605.png)

- 접근 제어를 위한 프록시를 두는 프록시 패턴과 컬러, 페이징 기능을 추가하기 위한 프록시를 두는 데코레이터 패턴을 함께 적용한 예다. 두 가지 모두 프록시의 기본 원리대로 타깃과 같은 인터페이스를 구현해두고 위임하는 방식으로 만들어져있다.

## 다이나믹 프록시

- 프록시는 기존 코드에 영향을 주지 않으면서 타깃의 기능을 확장하거나 접근 방법을 제어할 수 잇는 유용한 방법이다.
- 프록시를 만드는 일은 상당히 번거롭게 느껴진다. 매번 새로운 클래스를 정의해야 하고 인터페이스의 구현해야 할 메소드는 많으면 메소드를 일일이 구현해서 위임하는 코드를 넣어야 하기 때문이다.
- 단위 테스트를 위해 목이나 스텁을 일일이 클래스로 정의하고 인터페이스의 모의 메소드를 구현하는 일이 불편했던 것과 마찬가지다.
- 프록시도 일일이 모든 인터페이스를 구현해서 클래스로 만들어야할까?

자바에서는 `java.lang.reflect` 패키지 안에 프록시를 손쉽게 만들 수 있도록 지원해주는 클래스들이 있다. 일일이 프록시 클래스를 정의하지 않고도 몇 가지 API를 이용해 프록시처럼 동작하는 오브젝트를 다이나믹하게 생성하는 것이다.

### 프록시의 구성과 프록시 작성의 문제점

프록시는 다음의 두 가지 기능으로 구성된다.

- 타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 오브젝트로 위임한다.
- 지정된 요청에 대해서는 부가기능을 수행한다.

프록시의 기능 구분

```java
public class UserServiceTx implements USerService {
	// 타깃 오브젝트
	UserService userService;

	...

	// 메소드 구현과 위임
	public void add(User user) {
		this.userService.add(user);
	}

	// 메소드 구현
	public void upgradeLevels() {
		// 부가기능 수행
		TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
			try {
				// 위임
				userService.upgradeLevels();

				// 부가기능 수행
				this.transactionManager.commit(status);
			} catch (RuntimeException e) {
					this.transactionManager.rollback(status);
					throw e;
			}

	}

}
```

- `UserServviceTx` 코드는 `UserService` 인터페이스를 구현하고 타깃으로 요청을 위임하는 트랜잭션 부가기능을 수행하는 코드로 구분할 수 있다.
- 프록시의 역할은 위임과 부가작업 두 가지로 구분할 수 있다.
    1. 타깃의 인터페이스를 구현하고 위임하는 코드를 작성하기가 번거롭다. 부가기능이 필요 없는 메소드도 구현해서 타깃으로 위임하는 코드를 일일이 만들어줘야 한다. 복잡하진 않지만 인터페이스의 메소드가 많아지고 다양해지면 상당히 부담스러운 작업이 될 것이다. 또 타깃 인터페이스의 메소드가 추가되거나 변경될 때마다 함께 수정해줘야 한다는 부담도 없다.
    2. 부가기능 코드가 중복될 가능성이 많다. 트랜잭션은 DB를 사용하는 대부분의 로직에 적용될 필요가 있다. 아직까지 `add()` 메소드에는 트랜잭션 부가기능을 적용하지 않았지만 사용자를 추가하는 과정에서 다른 작업이 함께 진행돼야 한다면 `add()` 메소드에도 트랜잭션 경계설정 부가기능이 적용돼야 한다. 메소드가 많아지고 트랜잭션 적용의 비율이 높아지면 트랜잭션 기능을 제공하는 유사한 코드가 여러 메소드에 중복돼서 나타날 것이다.
- 사용자 관리 로직 외에도 다양한 비즈니스 로직을 담은 클래스가 만들어진다.
- 트랜잭션 외의 프록시를 활용할만한 부가기능, 접근제어 기능은 일반적인 성격을 띤 것들이 많다.
    - 다양한 타깃 클래스와 메소드에 중복돼서 나타날 가능성이 높다.
- 두 번째 문제인 부가기능의 중복 문제는 중복되는 코드를 어떻게든 해결해보면 될 것 같지만 첫 번째 문제인 인터페이스 메소드의 구현과 위임 기능 문제는 간단해 보이지 않는다. 이런 문제를 유용한 것이 바로 JDK의 다이내믹 프록시다.

### 리플렉션

- 다이나믹 프록시는 리플렉션 기능을 이용해서 프록시를 만든다.
- 자바의 모든 클래스는 그 클래스 자체의 구성정보를 담은 Class 타입의 오브젝트를 하나씩 갖고 있다.
    - ‘클래스이름.class’ 라고 하거나 오브젝트의 getClass() 메소드를 호출하면 클래스 정보를 담은 Class 타입의 오브젝트를 가져올 수 있다.
    - 클래스 오브젝트를 이용하면 클래스 코드에 대한 메타정보를 가져오거나 오브젝트를 조작할 수 있다.
        - 클래스 이름
        - 클래스 상속관계
        - 인터페이스 구현관계
        - 필드
        - 타입
        - 메소드
        - 메소드의 파라미터 및 리턴 타입
    - 오브젝트 필드의 값을 읽고 수정하거나 원하는 파라미터 값을 이용해 메소드를 호출할 수도 있다.
    - [java.lang.reflect](https://docs.oracle.com/javase/tutorial/reflect/index.html)
- 리플렉션 API 중에서 메소드에 대한 정의를 담은 Method라는 인터페이스를 이용해 메소드 호출하는 방법을 알아보자
    - String 클래스의 정보를 담은 Class 타입의 정보는 String.class로 가져올 수 있다.
    - 클래스 정보에서 특정 이름을 가진 메소드 정보를 가져올 수 있다.
        
        ```java
        Method lengthMethod = String.class.getMethod("length");
        ```
        
    - Method 인터페이스에 정의된 `invoke()` 메소드를 사용해서 실행시킬 수 있다. invoke() 메소드는 메소드를 실행시킬 대상 오브젝트(obj)와 파라미터 목록(args)을 받아 메소드를 호출한 뒤에 그 결과를 Object 타입으로 돌려준다.
        
        ```java
        public Object invoke(Object obj, Object... args)
        ```
        
        ```java
        int length = lengthMethod.invoke(name); // int length = name.length();
        ```
        

### 프록시 클래스

- 다이나믹 프록시를 이용한 프록시를 만들어보자.
- 프록시를 적용할 간단한 타깃 클래스와 인터페이스를 아래와 같이 정의한다.

**Hello 인터페이스**

```java
interface Hello {
	String sayHello(String name);
	String sayHi(String name);
	String sayThankYou(String name);
}
```

**타깃 클래스**

```java
public class HelloTarget implements Hello {

	public String sayHello(String name) {
		return "Hello " + name;
	}

	public String sayHi(String name) {
		return "Hi " + name;
	}

	public String sayThankYou(String name) {
		return "Thank You " + name;
	}

}
```

**클라이언트 역할의 테스트**

```java
@Test
public void simpleProxy() {
	// 타깃은 인터페이스를 통해 접근하는 습관을 들이자.
	Hello hello = new HelloTarget(); 

	assertThat(hello.sayHello("Toby"), is("Hello Toby"));
	assertThat(hello.sayHi("Toby"), is("Hi Toby"));
	assertThat(hello.saayThankYou("Toby"), is("Thank You Toby"));
}
```

- 프록시에는 데코레이터 패턴을 적용해서 타깃인 `HelloTarget`에 부가기능을 추가한다..
    - 프록시의 이름은 `HelloUppercase`이다.
    - 추가할 기능은 리턴하는 문자를 모두 대문자로 바꿔주는 것이다.
    - `HelloUppercase` 프록시는 `Hello` 인터페이스를 구현하고 `Hello` 타입의 타깃 오브젝트를 받아서 저장한다.
    - 타깃 오브젝트의 메소드를 호출한 뒤에 결과를 대문자로 바꿔주는 부가기능을 적용하고 리턴한다.
- 위임과 기능 부가라는 두 가지의 프록시의 기능을 처리하는 전형적인 프록시 클래스다.

프록시 클래스

```java
public class HelloUppercase implements Hello {
	// 위임할 타깃 오브젝트 여기서는 타깃 클래스의 오브젝트인 것은 알지만 다른 프록시를 추가할 수도 있으므로 인터페이스로 접근한다.
	public HelloUppercase(Hello hello) {
		this.hello = hello;
	}

	public String sayHello(String name) {
		// 위임과 부가기능 적용
		return hello.sayHello(name).toUppperCase();
	}

	public String sayHi(String name) {
		return hello.sayHi(name).toUpperCase();
	}

	public String sayThankYou(String name) {
		return hello.sayThankYou(name).toUpperCase();
	}
}
```

`HelloUppercase` 프록시 테스트

```java
// 프록시를 통해 타깃 오브젝트에 접근하도록 구성한다.
Hello proxiedHello = new HelloUppercase(new HelloTarget();
assertThat(hello.sayHello("Toby"), is("HELLO TOBY"));
assertThat(hello.sayHi("Toby"), is("HI TOBY"));
assertThat(hello.saayThankYou("Toby"), is("THANK YOU TOBY"));
```

- 이 프록시는 일반적인 문제점 두 가지를 모두 갖고 있다.
1. 인터페이스의 모든 메소드를 구현해 위임하도록 코드를 만들어야 한다.
2. 부가기능인 리턴 값을 대문자로 바꾸는 기능이 모든 메소드에 중복돼서 나타난다.

### 다이나믹 프록시 적용

![image](https://user-images.githubusercontent.com/66561524/198752501-b811ef77-0e06-4229-ac38-1479afd5757a.png)

- 다이나믹 프록시는 프록시 팩토리에 의해 런타임 시 다이나믹하게 만들어지는 오브젝트다.
    - 다이나믹 프록시 오브젝트는 타깃의 인터페이스와 같은 타입으로 만들어진다.
    - 클라이언트는 다이나믹 프록시 오브젝트를 타깃 인터페이스를 통해 사용할 수 있다.
    - 프록시 팩토리에게 인터페이스 정보만 제공해주면 해당 인터페이스를 구현한 클래스의 오브젝트를 자동으로 만들어준다.
- 다이나믹 프록시가 인터페이스 구현 클래스의 오브젝트는 만들어주지만 프록시로서 필요한 부가기능 제공 코드는 직접 작성해야 한다.
    - 부가기능은 프록시 오브젝트와 독립적으로 `InvocationHandler`를 구현한 오브젝트에 담는다.
    - `InvocationHandler` 인터페이스는 메소드 한 개만 가진 간단한 인터페이스이다.
    
    ```java
    public Object invoke(Object proxy, Method method, Object[] args)
    ```
    
    - `invoke()` 메소드는 리플렉션의 `Method` 인터페이스를 파라미터로 받는다.
    - 메소드를 호출할 때 전달되는 파라미터도 args로 받는다.
    - 다이나믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환해서 `InvocationHandler` 구현 오브젝트의 `invoke()` 메소드로 넘기는 것이다.
    - 타깃 인터페이스의 모든 메소드 요청이 하나의 메소드로 집중되기 때문에 중복되는 기능을 효과적으로 제공할 수 있다.
- 리플렉션으로 메소드와 파라미터 정보를 모두 갖고 있으므로 타깃 오브젝트의 메소드를 호출하게 할 수도 있다.
    - 리플렉션 학습 테스트를 만들어 `Method`와 파라미터 정보가 있으면 특정 오브젝트의 메소드를 호출하게도 할 수 있다.
    - 리플렉션 학습 테스트를 만들어 `Method`와 파라미터 정보가 있으면 특정 오브젝트의 메소드를 실행할 수 있음을 확인했다.
    - `InvocationHandler` 구현 오브젝트가 타깃 오브젝트 레퍼런스를 갖고 있다면 리플렉션을 이용해 간단히 위임 코드를 만들어 낼 수 있다.
- 프록시 팩토리에게 다이나믹 프록시를 만들어달라고 요청하면 Hello 인터페이스의 모든 메소드를 구현한 오브젝트를 생성해준다.
    - `InvocationHandler` 인터페이스를 구현한 오브젝트를 제공해주면 다이나믹 프록시가 받는 모든 요청을 `InvocationHandler`의 `invoke()` 메소드 하나로 처리할 수 있다.
    - `Hello` 인터페이스의 메소드가 아무리 많더라도 `invoke()` 메소드 하나로 처리할 수 있다.
    - 다이나믹 프록시 오브젝트와 `InvocationHandler` 오브젝트, 타깃 오브젝트 사이의 메소드 호출이 일어나는 과정을 나타낸다.

![image](https://user-images.githubusercontent.com/66561524/198752530-c68b6401-bb8b-4fad-9114-1871761ce03d.png)

- 다이나믹 프록시부터 메소드 호출 정보를 받아서 처리하는 InvocationHandler를 만들어보자

```java
public class UppercaseHandler implements InvocationHandler {
	Hello target;

	// 다이나믹 프록시부터 전달받은 요청을 다시 타깃 오브젝트에 위임해야 하기 때문에 타깃 오브젝트를 주입받아야 한다.
	public UppercaseHandler(Hello target) {
		this.target = target;
	}

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		// 타깃으로 위임 인터페이스의 메소드 호출에 모두 적용된다.
		String ret = (String) method.invoke(target, args);

		// 부가기능 제공
		return ret.toUpperCase(); 
	}
}
```

- 다이나믹 프록시가 클라이언트로부터 받은 모든 요청은 invoke() 하나뿐이다.
- 다이나믹 프록시를 통해 요청이 전달되면 리플렉션 API를 이용해 타깃 오브젝트의 메소드를 호출한다.
- Hello 인터페이스의 모든 메소드는 결과가 String 타입이므로 메소드 호출의 결과를 String 타입으로 변환해도 안전하다.
- 타깃 오브젝트의 메소드 호출이 끝났으면 프록시가 제공하려는 부가기능인 리턴 값을 대문자로 바꾸는 작업을 수행하고 결과를 리턴한다.
- 리턴된 값은 다이나믹 프록시가 받아서 최종적으로 클라이언트에 전달된다.

```java
// 생성된 다이나믹 프록시 오브젝트는 Hello 인터페이스를 구현하고 있으므로 Hello 타입으로 캐스팅해도 안전
Hello proxiedHello = (Hello) Proxy.newProxyInstance(
	// 동적으로 생성되는 다이나믹 프록시 클래스의 로딩에 사용할 클래스 로더
	getClass().getClassLoader(),
	// 구현할 인터페이스
	new Class[] { Hello.class },
	// 부가기능과 위임 코드를 담은 InvocationHandler
	new UppercaseHandler(new HelloTarget())
);
```

- 다이나믹 프록시의 생성은 `Proxy` 클래스의 `newProxyInstance()` 스태틱 팩토리 메소드를 이용하면 된다.
- 사용 방법
    - 첫 번째 파라미터는 클래스 로더를 제공해야 한다.
        - 다이나믹 프록시가 정의되는 클래스 로더를 정의하는 것이다.
    - 두 번째 파라미터는 다이나믹 프록시가 구현해야 할 인터페이스다.
        - 다이나믹 프록시는 한 번에 하나 이상의 인터페이스를 구현할 수도 있다.
    - 세 번째 파라미터는 부가기능과 위임 관련 코드를 담고 있는 `InvocationHandler` 구현 오브젝트를 제공해야 한다.
    - Hello 타입의 타깃 오브젝트를 생성자로 받고 모든 메소드 호출의 리턴 값을 대문자로 바꿔주는 `UppercaseHandler` 오브젝트를 전달했다.
- `newProxyInstance()` 에 의해 만들어지는 다이나믹 프록시 오브젝트는 파라미터로 제공한 Hello 인터페이스를 구현한 클래스의 오브젝트이기 때문에 Hello 타입으로 캐스팅이 가능하다.
- 리플렉션 API를 적용하고 복잡한 다이나믹 프록시 생성 방법을 적용했는데 원래 만들었던 HelloUppercase 프록시 클래스보다 코드 양도 줄어들지 않은 것 같다. 장점은 무엇일까?

### 다이나믹 프록시의 확장

- Hello 인터페이스의 메소드가 3개가 아니라 30개라면 `HelloUppercase` 처럼 클래스로 직접 구현한 프록시는 매번 코드를 추가해야 한다.
    - 하지만`UppercaseHandler`와 다이나믹 프록시를 생성해서 사용하는 코드는 손 댈 것이 없다.
    - 다이나믹 프록시가 만들어질 때 추가된 메소드가 자동으로 포함되고 부가기능은 `invoke()` 메소드에서 처리된다.
    - `UppercaseHandler` 는 모든 메소드의 리턴 타입이 스트링이라고 가정한다. 스트링 외의 리턴 타입이면 캐스팅 오류가 발생한다. 리플렉션은 막강한 기능 대신 주의 깊게 사용할 필요가 있다.
    - Method를 이용한 타깃 오브젝트의 메소드 호출 후 리턴 타입을 확인해서 스트링인 경우만 대문자로 바꿔주도록 수정하자.

```java
public class UppercaseHandler implements InvocationHandler {

	// 어떤 종류의 인터페이스를 구현한 타깃에도 적용 가능하도록 Object 타입으로 수정
	Object target;
	public UppercaseHandler(Object target) {
		this.target = target;
	}

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		// 호출한 메소드의 리턴 타입이 String인 경우만 대문자 변경 기능을 적용하도록 수정
		Object ret = method.invoke(target, args);
		if (ret instanceof String) {
				return ret.toUpperCase(); 
		} else {
			return ret;
		}
	}
}
```

- 어떤 종류의 인터페이스를 구현한 타깃이든 상관없이 재사용할 수 있다.
- 메소드의 리턴 타입이 스트링인 경우만 대문자로 결과를 바꿔주도록 `UppercaseHandler`를 만들 수 있다.
- `InvocationHandler` 는 단일 메소드에서 모든 요청을 처리하기 때문에 어떤 메소드에 어떤 기능을 적용할지를 선택하는 과정이 필요할 수 있다.
- 호출하는 메소드의 이름, 파라미터의 개수와 타입, 리턴 타입 등의 정보를 가지고 부가적인 기능을 적용할 메소드를 선택할 수 있다.
- 리턴 타입뿐 아니라 메소드의 이름도 조건으로 걸 수 있다. 메소드의 이름이 say로 시작하는 경우에만 대문자로 바꾸는 기능을 적용하고 싶다면 Method 파라미터에서 메소드 이름을 가져와 확인하는 방법을 사용하면 된다.

```java
public class UppercaseHandler implements InvocationHandler {
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		// 호출한 메소드의 리턴 타입이 String인 경우만 대문자 변경 기능을 적용하도록 수정
		Object ret = method.invoke(target, args);
		// 리턴 타입과 메소드 이름이 일치하는 경우에만 부가기능을 적용한다.
		if (ret instanceof String && method.getName().startsWith("say")) {
				return ((String) ret).toUpperCase(); 
		} else {
			// 조건이 일치하지 않으면 타깃 오브젝트의 호출 결과를 그대로 리턴한다.
			return ret;
		}
	}
}
```

## 다이나믹 프록시를 이용한 트랜잭션 부가 기능

- UserServiceTx를 다이나믹 프록시 방식으로 바꾸자.
- 트랜잭션 부가기능을 제공하는 다이나믹 프록시를 만들어 적용하는 방법은 효율적이다.

### 트랜잭션 InvocationHandler

```java
public class TransactionHandler implements InvocationHandler {

	// 부가 기능을 제공할 타깃 오브젝트. 어떤 타입의 오브젝트에도 적용 가능하다.
  private Object target;
	// 트랜잭션 기능을 제공하는 데 필요한 트랜잭션 매니저
  private PlatformTransactionManager platformTransactionManager;
	// 트랜잭션을 적용할 메소드 이름 패턴
  private String pattern;

  /*
  요청을 위임할 타깃을 DI로 제공 받도록 함.
  어떠한 타깃 오브젝트 에도 적용 할 수 있다.
   */
  public void setTarget(Object target) {
    this.target = target;
  }

  public void setPlatformTransactionManager(PlatformTransactionManager platformTransactionManager) {
    this.platformTransactionManager = platformTransactionManager;
  }

  /*
  모든 메소드에 적용되지 않도록 패턴을 DI 받는다.
  "get" 으로 주면 get으로 시작하는 모든 메소드에 트랜잭션 적용
   */
  public void setPattern(String pattern) {
    this.pattern = pattern;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		// 트랜잭션 적용 대상 메소드를 선별해서 트랜잭션 경계설정 기능을 부여한다.
    if (method.getName().startsWith(pattern)) { // 패턴에 맞는 메소드 인지 확인
      return invokeInTransaction(method, args); // 패턴과 일치하면 호출
    } else {
      return method.invoke(target, args); // 아니라면 부가기능 없이 호출
    }
  }

  private Object invokeInTransaction(Method method, Object[] args) throws Throwable {
    TransactionStatus status = this.platformTransactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
			// 트래잭션을 시작하고 타깃 오브젝트의 메소드를 호출한다.
			// 예외가 발생하지 않았다면 커밋한다.
      Object ret = method.invoke(target, args);
      this.platformTransactionManager.commit(status);
      return ret;
    } catch (InvocationTargetException e) {
			// 예외가 발생하면 트랜잭션을 롤백한다.
      this.platformTransactionManager.rollback(status);
      throw e.getTargetException();
    }
  }
}
```

- `UserServiceTx`보다 코드는 복잡하지 않으면서 `UserService`뿐 아니라 모든 트랜잭션이 필요한 오브젝트에 적용 가능한 트랜잭션 프록시 핸들러가 만들어졌다.

### TransactionHanlder와 다이나믹 프록시를 이용하는 테스트

```java
@Test
public void upgradeAllorNothing() throws Exception {
	UserServiceImpl testUserService = new TestUserService(users.get(3).getId());
	testUserService.setUserDao(this.userDao); 
	testUserService.setMailSender(mailSender);
	
	// InovcationHandler 구현 객체 생성 
	TransactionHandler txHandler = new TransactionHandler();

	// 트랜잭션 핸들러가 필요한 정보와 오브젝트를 DI 해준다.
	txHandler.setTarget(testUserService);
	txHandler.setTransactionManager(transactionManager);
	txHandler.setPattern("upgradeLevels");
	
	// UserService 인터페이스 타입의 다이나믹 프록시 객체 생성 
	UserService txUserService = (UserService) Proxy.newProxyInstance(
			getClass().getClassLoader(),
			new Class[] {UserService.class},
			txHandler);
	//...
 
}
```

## 다이나믹 프록시를 위한 팩토리 빈

- `TransactionHandler`와 다이나믹 프록시를 스프링의 DI를 통해 사용할 수 있도록 만들어야 한다.
- 문제는 DI의 대상이 되는 다이나믹 프록시 오브젝트는 일반적인 스프링의 빈으로 등록할 방법이 없다.
- 스프링의 빈은 기본적으로 클래스 이름과 프로퍼티로 정의된다. 스프링은 지정된 클래스 이름을 가지고 리플렉션을 이용해서 해당 클래스의 오브젝트를 만든다. 클래스의 이름을 갖고 있다면 새로운 오브젝트를 Class의 `newInstance()` 메소드로 만들 수 있다. `newInstance()`는 해당 클래스의 파라미터가 없는 생성자를 호출하고 그 결과 생성되는 오브젝트를 돌려주는 리플렉션 API다.
- 스프링은 내부적으로 리플렉션 API를 이용해서 빈 정의에 나오는 클래스 이름을 가지고 빈 오브젝트를 생성한다. 문제는 다이나믹 프록시 오브젝트는 이런식으로 프록시 오브젝트가 생성되지 않는다. 클래스 자체도 내부적으로 다이나믹하게 새로 정의해서 사용ㅎ나다.
- 사전에 프록시 오브젝트의 클래스 정보를 미리 알아내서 스프링 빈에 정의할 방법이 없다. 다이나믹 프록시는 Proxy 클래스의 `newProxyInstance()`라는 스태틱 팩토리 메소드를 통해서만 만들 수 있다.

### 팩토리 빈

- 클래스 정보를 가지고 디폴트 생성자를 통해 오브젝트를 만드는 방법 외에도 빈을 만들 수 있는 여러가지 방법을 제공한다.
- 대표적으로 팩토리 빈을 이용한 빈 생성 방법이 있다.
- 팩토리 빈은 스프링을 대신해서 오브젝트의 생성로직을 담당하도록 만들어진 빈이다.
- 팩토리 빈을 만드는 방법에는 여러 가지가 있는데 가장 간단한 방법은 스프링의 `FactoryBean`이라는 인터페이스를 구현하는 것이다.

- `FactoryBean` 인터페이스는 세 가지 메소드로 구현되어 있다.

```java
public interface FactoryBean<T> {

	String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

	// 빈 오브젝트를 생성해서 돌려준다.
	@Nullable
	T getObject() throws Exception;

	// 생성되는 오브젝트의 타입을 알려준다.
	@Nullable
	Class<?> getObjectType();

	// getObject()가 돌려주는 오브젝트가 항상 같은 싱글톤 오브젝트인지 알려준다.
	default boolean isSingleton() {
		return true;
	}

}
```

```java
package springbook.learningtest.jdk;
 
public class Message {
	
	String text;
 
	// 외부에서 생성자를 통해서 오브젝트를 만들 수 없다. 
	private Message(String text) {
		this.text = text;
	}
	
	public String getText() {
		return this.text;
	}
	
	// 생성자 대신에 사용할 수 있는 스태택 팩토리 메소드를 제공한다.
	public static Message newMessage(String text) {
		return new Message(text);
	}
}
```

- Message 클래스는 생성자를 통해 오브젝트를 만들 수 없다.
- 사실 스프링은 private 생성자를 가진 클래스도 빈으로 등록해주면 리플렉션을 이용해 오브젝트를 만들어준다.
- 리플렉션은 private으로 선언된 접근 규약을 위반할 수 있는 강력한 기능이 있다.
- 하지만 생성자를 private으로 만들었다는 것은 스태틱 메소드로 오브젝트가 만들어져야하는 중요한 이유가 있어서 이를 무시하면 위험하다.

```java
package springbook.learningtest.spring.factorybean;
 
import org.springframework.beans.factory.FactoryBean;
 
public class MessageFactoryBean implements FactoryBean<Message>{
 
	String text; 
	
	// 오브젝트를 생성할 때 필요한 정보를 팩토리 빈의 프로퍼티로 설정해서 
	// 대신 DI 받게 한다. 주입된 정보는 오브젝트 생성 중에 사용된다. 
	public void setText(String text) {
		this.text = text;
	}
	
	// 실제 빈으로 사용될 오브젝트를 직접생성한다.
	@Override
	public Message getObject() throws Exception {
		return Message.newMessage(this.text);
	}
 
	@Override
	public Class<? extends Message> getObjectType() {
		return Message.class;
	}
	
	// 싱글톤 여부를 알려 주는 메소드이다. 
	// 이 팩토리빈은 매번 요청할 때마다 새로운 오브젝트를 만들므로 false로 설정한다.
	@Override
	public boolean isSingleton() {
		return false;
	}
}
```

### 다이나믹 프록시를 만들어주는 팩토리 빈

- Proxy의 `newProxyInstance()` 메소드를 통해서만 생성이 가능한 다이나믹 프록시 오브젝트는 일반적인 방법으로 스프링의 빈으로 등록할 수 없다.
- 대신 팩토리 빈을 사용하면 다이나믹 프록시 오브젝트를 스프링의 빈으로 만들어 줄 수 있다.
- 팩토리 빈의 `getObject()` 메소드에 다이나믹 프록시 오브젝트를 만들어주는 코드를 넣으면 된다.

![image](https://user-images.githubusercontent.com/66561524/198752537-b6d6fae6-2fbe-4ab7-a045-042792b565a1.png)

- 스프링 빈에는 팩토리 빈과 `USerServiceImpl`만 빈으로 등록한다.
- 팩토리 빈은 다이나믹 프록시가 위임할 타깃 오브젝트인 `UserServiceImpl`에 대한 레퍼런스를 프로퍼티를 통해 DI 받아야 한다.
- 다이나믹 프록시와 함께 생성할 `TransactionHandler`에게 타깃 오브젝트를 전달해줘야 하기 때문이다.

### 트랜잭션 프록시 팩토리 빈

- `TransactionHandler`를 이용하는 다이나믹 프록시를 생성하는 팩토리 빈 클래스다.

```java
package springbook.user.service;
 
import java.lang.reflect.Proxy;
 
import org.springframework.beans.factory.FactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
 
import springbook.user.transaction.TransactionHandler;
 
//생성할 오브젝트 타입을 지정할 수도 있지만 범용적으로 사용하기 위해 Object로 지정했다. 
public class TxProxyFactoryBean implements FactoryBean<Object>{
 
	Object target; 
	
	PlatformTransactionManager transactionManager;
	
	String pattern;
	
	// 다이내믹 프록시를 생성할 때 필요하다.
	// UserService외의 인터페이스를 가진 타깃에도 적용할 수 있다.
	Class<?> serviceInterface;
	
	
	public void setTarget(Object target) {
		this.target = target;
	}

	public void setTransactionManager(PlatformTransactionManager transactionManager) {
		this.transactionManager = transactionManager;
	}
 
	public void setPattern(String pattern) {
		this.pattern = pattern;
	}
 
	public void setServiceInterface(Class<?> serviceInterface) {
		this.serviceInterface = serviceInterface;
	}
 
	// DI 받은 정보를 이용해서 TransactionHanlder를 사용하는 다이내믹 프록시를 생성한다.
	@Override
	public Object getObject() throws Exception {
		TransactionHandler txHandler = new TransactionHandler();
		txHandler.setTarget(target);
		txHandler.setTransactionManager(transactionManager);
		txHandler.setPattern(pattern);
		Object proxiedObj = Proxy.newProxyInstance(
				getClass().getClassLoader(), 
				new Class[] {serviceInterface},
				txHandler);
		return proxiedObj;
	}
 
	// 팩토리 빈이 생성하는 오브젝트 타입은 DI 받은 인터페이스 타입에 따라 달라진다.
	// 다양한 타입의 프록시 오브젝트 생성에 재사용할 수 있다. 
	@Override
	public Class<?> getObjectType() {
		return serviceInterface;
	}
	
	@Override
	public boolean isSingleton() {
		return false;
	}
}
```

- 팩토리 빈이 만드는 다이나믹 프록시는 구현 인터페이스나 타깃의 종류 제한이 없다.
- 따라서 `UserService` 외에도 트랜잭션 부가기능이 필요한 오브젝트를 위한 프록시를 만들 때 얼마든지 재사용 가능하다.
- 설정이 다른 여러개의 `TxProxyFactoryBean` 빈을 등록하면 된다.

### 트랜잭션 프록시 팩토리 빈 테스트

```java
public class UserServiceTest {
//...
 
	// 팩토리 빈을 가져오려면 애플리케이션 컨텍스트가 필요하다. 
	@Autowired
	ApplicationContext context;
 
//...
    
	@Test
	@DirtiesContext 
	public void upgradeAllorNothing() throws Exception {
		
		UserServiceImpl testUserService = new TestUserService(users.get(3).getId());
		testUserService.setUserDao(this.userDao); 
		testUserService.setMailSender(mailSender);
		
		// 팩토리 빈 자체를 가져와야 하므로 빈 이름에 &를 반드시 넣어야 한다. 
		TxProxyFactoryBean txProxyFactoryBean = 
				context.getBean("&userService", TxProxyFactoryBean.class);
		txProxyFactoryBean.setTarget(testUserService);
		// 변경된 타깃 설정을 이용해서 트랜잭션 다이내믹 프록시 오브젝트를 생성한다. 
		UserService txUserService = (UserService) txProxyFactoryBean.getObject();
		
		userDao.deleteAll();
		for(User user : users) userDao.add(user);
		
		try {
			txUserService.upgradeLevels();
			fail("TestUserServiceException exptected");
		} catch (TestUserServiceException e) {
			// TODO: handle exception
		}
			
		checkLevelUpgrade(users.get(1), false);
	}
    
}
```

## 프록시 팩토리 빈 방식의 장점과 한계

- 다이나믹 프록시를 생성해주는 팩토리 빈을 사용하는 방법은 여러가지 장점이 있다.
- 한 번 부가기능을 가진 프록시를 생성하는 팩토리 빈을 만들어두면 타깃의 타입에 상관없이 재사용할 수 있기 때문이다.

### 프록시 팩토리 빈의 재사용

- `TransactionHandler`를 이용하는 다이나믹 프록시를 생성해주는 `TxProxyFactoryBean`은 코드의 수정 없이도 다양한 클래스에 적용할 수 있다.
- 타깃 오브젝트에 맞는 프로퍼티 정보를 설정해서 빈으로 등록해주기만 하면 된다.
- 하나 이상의 `TxProxyFactoryBean`을 동시에 빈으로 등록해도 상관없다.
- 팩토리 빈이기 때문에 각 빈의 타입은 타깃 인터페이스와 일치한다.
    - `UserService` 외에 트랜잭션 경계설정 기능을 부여해줄 필요가 있는 필요한 클래스가 있다고 해보자.
- target 프로퍼티를 coreServiceTarget 빈으로 설정해주고 serviceInterface에는 프록시가 구현할 인터페이스인 CoreService를 넣어주면 모든 준비가 끝난다.

![image](https://user-images.githubusercontent.com/66561524/198752556-f9e51c67-fde2-497c-9860-df90229f0120.png)

- 프록시 팩토리 빈을 이용하면 프록시 기법을 아주 빠르고 효과적으로 적용할 수 있다.

### 프록시 팩토리 빈 방식의 장점

- 데코레이터 패턴이 적용된 프록시를 사용하면 많은 장점이 있지만 두가지 문제가 있다.
    1. 프록시를 적용할 대상이 구현하고 있는 인터페이스를 구현하는 프록시 클래스를 일일이 만들어야 한다는 번거로움이다.
    2. 부가적인 기능이 여러 메소드에 반복적으로 나타나게 되서 코드 중복의 문제가 발생한다는 점이다.
- 프록시 팩토리 빈은 이 문제를 해결해준다.
- 다이나믹 프록시를 이용하면 타깃 인터페이스를 구현하는 클래스를 일일이 만드는 번거로움을 제거할 수 있다. 하나의 핸들러 메소드를 구현하는 것만으로도 수많은 메소드에 부가기능을 부여해줄 수 있어서 부가기능 코드의 중복문제도 사라진다.
- 다이나믹 프록시에 팩토리 빈을 이용한 DI까지 더해주면 번거로운 다이나믹 프록시 생성 코드도 제거할 수 있다.
    - DI 설정만으로 다양한 타깃 오브젝트에 적용도 가능하다.
    - 프록시를 도입하려고 했을 때 고민했던 문제점을 거의 완벽하게 해결한 듯하다.
- 스프링 DI는 중요한 역할을 했다. 프록시를 사용하려면 DI가 필요한 것은 물론이고 효율적인 프록시 생성을 위한 다이나믹 프록시를 사용하려고 할 때도 팩토리 빈을 통한 DI는 필수다.

### 프록시 팩토리 빈의 한계

- 더 욕심을 내서 중복 없는 최적화된 코드와 설정만을 이용해 일너 기능을 적용하려고 하면 문제가 생긴다.
- 프록시를 통해 타깃에 부가기능을 제공하는 것은 메소드 단위로 일어나는 일이다. 하나의 클래스 안에 존재하는 여러 개의 메소드에 부가기능을 한 번에 제공하는 건 어렵지 않게 가능했다.
- 한 번에 여러 개의 클래스에 공통적인 부가기능을 제공하는 일은 지금까지 살펴본 방법으로 불가능하다.
    - 하나의 타깃오브젝트에만 부여되는 부가기능은 상관없지만 트랜잭션과 같이 비즈니스 로직을 담은 많은 클래스의 메소드에 적용할 필요가 없다면 비슷한 프록시 팩토리 빈의 설정이 중복되는 것을 막을 수 없다.
- 하나의 타깃에 여러 개의 부가기능을 적용하려고 할 때 문제가 생긴다.
    - 같은 타깃 오브젝트에 대해 트랜잭션 프록시뿐 아니라 보안 기능을 제공하는 프록시도 적용하고 싶고 기능 검사를 위해 주고받는 메소드 정보를 저장해두는 부가기능을 담은 프록시도 추가하고 싶다면?
    - 빈 설정 코드가 상당히 많아진다.
- `TransactionHandler`오브젝트가 프록시 팩토리 빈 개수만큼 만들어진다.
    - `TransactionHandler` 는 타깃 오브젝트를 프로퍼티로 갖고 있다.
    - `TransactionHandler` 는 다이나믹 프록시처럼 굳이 팩토리 빈에서 만들지 않아도 된다.
        - 스스로 빈으로 등록될 수 있다.
    - 하지만 타깃 오브젝트가 다르기 때문에 타깃 오브젝트 개수만큼 다른 빈으로 등록해야 하고 그만큼 많은 오브젝트가 생겨날 것이다.
    - `TransactionHandler` 의 중복을 없애고 모든 타깃에 적용 가능한 싱글톤 빈으로 만들어서 적용할 수 없을까?

# 4. 스프링의 프록시 팩토리 빈

---

## ProxyFactoryBean

- 트랜잭션 기술과 메일 발송 기술에 적용했던 서비스 추상화를 프록시 기술에도 동일하게 적용한다.
- 자바에서는 JDK에서 제공하는 다이나믹 프록시 외에도 편리하게 프록시를 만들 수 있는 다양한 기술이 있다.
- 따라서 스프링은 일관된 방법으로 프록시를 만들 수 있게 도와주는 추상 레이어를 제공한다.
- 생성된 프록시는 스프링의 빈으로 등록돼야 한다. 스프링은 프록시 오브젝트를 생성해주는 기술을 추상화한 팩토리 빈을 제공한다.
- 스프링의 ProxyFactoryBean은 프록시를 생성해서 빈 오브젝트로 등록하게 해주는 팩토리 빈이다. 순수하게 프록시를 생성하는 작업만을 담당하고 프록시를 통해 제공해줄 부가기능은 별도의 빈에 둘 수 있다.
- `ProxyFactoryBean`에서 사용할 부가기능은 `MethodInterceptor` 인터페이스를 구현해서 만든다. `MethodInterceptor`는 `InvocationHandler`와 비슷하지만 한 가지 다른 점이 있다. `InvocationHandler` 의 `invoke()` 메소드는 타깃 오브젝트에 대한 정보를 제공하지 않는다. 타깃은 `InvocationHandler`를 구현한 클래스가 직접 알고 있어야 한다. 반면에 `MethodInterceptor` 의 `invoke()` 메소드는 `ProxyFactoryBean`으로부터 타깃 오브젝트에 대한 정보까지도 함께 제공받는다. 그 차이 덕분에 `MethodInterceptor`는 타깃 오브젝트에 상관없이 독립적으로 만들어질 수 있다. 따라서 `MethodInterceptor` 오브젝트는 타깃이 다른 여러 프록시에서 함께 사용할 수 있고 싱글톤 빈으로 등록 가능하다.

추가할 라이브러리

```java
com.springsource.org.aopalliance-1.0.0.jar
org.springframework.aop-3.0.7.RELEASE.jar
```

```java
package com.david.learningtest.jdk.proxy;

public class DynamicProxyTest {
    @Test
    public void proxyFactoryBean() {
        ProxyFactoryBean pfBean = new ProxyFactoryBean();
        // 타깃 설정
        pfBean.setTarget(new HelloTarget());
        // 부가기능을 담은 어드바이스 추가
        // 여러개 추가도 가능
        pfBean.addAdvice(new UppercaseAdvice());

        // FactoryBean 이므로 getObject()로 생성된 프록시를 가져온다.
        Hello proxiedHello = (Hello) pfBean.getObject();
        assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
        assertThat(proxiedHello.sayHello("Toby"), is("HI TOBY"));
        assertThat(proxiedHello.sayThankYou("Toby"), is("THANK YOU TOBY"));
    }

    static class UppercaseAdvice implements MethodInterceptor {
        @Override
        public Object invoke(MethodInvocation invocation) throws Throwable {
            // 타깃 오브젝트를 전달할 필요 없음. 메소드 정보와 타깃 오브젝트를 이미 알고 있음
            String ret = (String) invocation.proceed();
            // 부가 기능 적용
            return ret.toUpperCase();
        }
    }

    // 타깃과 프록시가 구현할 인터페이스
    static interface Hello {
        String sayHello(String name);
        String sayHi(String name);
        String sayThankYou(String name);
    }

    // 타깃 클래스
    static class HelloTarget implements Hello {
        @Override
        public String sayHello(String name) {
            return "Hello " + name;
        }

        @Override
        public String sayHi(String name) {
            return "Hi " + name;
        }

        @Override
        public String sayThankYou(String name) {
            return "Thank You " + name;
        }
    }
}
```

### 어드바이스: 타깃이 필요 없는 순수한 부가기능

- `ProxyFactoryBean`을 적용한 코드를 기존의 JDK 다이나믹 프록시를 사용했던 코드와 비교해보면 몇 가지 눈에 띄는 차이점이 있다.
- `InvocationHandler` 를 구현했을 때와 달리 `MethodInterceptor` 를 구현한 `UppercaseAdvice`에는 타깃 오브젝트가 등장하지 않는다.
- `MethodInterceptor` 로는 메소드 정보와 함께 타깃 오브젝트가 담긴 `MethodInvocation` 오브젝트가 전달된다. `MethodInterceptor`  은 타깃 오브젝트의 메소드를 실행할 수 있는 기능이 있기 때문에 `MethodInterceptor` 는 부가기능을 제공하는 데만 집중할 수 있다.
- `MethodInterceptor` 은 일종의 콜백 오브젝트로 `proceed()` 메소드를 실행하면 타깃 오브젝트의 메소드를 내부적으로 실행해주는 기능이 있다.
- `MethodInvocation` 구현 클래스는 이롲ㅇ의 공유 가능한 템플릿처럼 동작한다. 바로 이 점이 JDK의 다이나믹 프록시를 직접 사용하는 코드와 스프링이 제공해주는 프록시 추상화 기능인 `ProxyFactoryBean`을 사용하는 코드의 큰 차이점이자 `ProxyFactoryBean` 의 장점이다.
