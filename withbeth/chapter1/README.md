Note : 해당 문서는 WorkFlowy에 최적화 되어있습니다.
Link : https://workflowy.com/s/1/rYO30BucKhnb4JD0

1. 오브젝트와 의존관계

- What I learned
  - Summary
    - 스프링이 POJO Programming을 가능케 하는데 초점을 맞춘 만큼, 그 대상인 POJO의 유연한 확장가능케 하는 설계 방법
    - 스프링의 POJO Programming 가능기술인 IoC/DI가 필요한 이유와 동작원리
  - Detail
    - 관심의 분리를 통한 리팩토링과 상속을 통한 확장의 장단점과 패턴(Template Method Pattern, Factory Method Pattern)
    - DI를 통한 상속을 통한 기능 확장의 단점해결법
    - IoC/DI의 프로그래밍 개념
    - Spring ApplicationContext와 POJO Singleton의 차이점
- [Goal] 
  - 스프링이 POJO Programming을 가능케 하는데 초점을 맞춘 만큼, 그 대상인 POJO의 유연한 확장가능케 하는 설계 방법 이해하기
  - from scratch부터 만들어 보고, 그 코드의 문제점을 분석. 객체지향의 다양한 방법과 패턴, 원칙, IoC/DI 프레임워크까지 적용해서 개선해보기.
    - 이를 통해, 스프링의 POJO Programming 가능기술인 IoC/DI가 필요한 이유와 동작원리 이해
- [Exam] 초난감 DAO
  - 예제 : JDBC API이용해 DB read / write하는 DAO 만들어 보기
    - User
      - JavaBean 규약 따르는 도메인 객체 생성
        - JavaBean은 다음 두가지 관례를 따르는 오브젝트를 가르킨다
          - default constructor
          - 해당 오브젝트가 노출하는 필드들의 getter/setter
    - UserDao
      - TODO
        - Get or Create DB Connection
        - Create Object which contains SQL(Statement or PreparedStatement)
        - Execute SQL
        - Convert result object to Domain object (ResultSet)
        - Close all resources(connection, Statement, ResultSet)
        - Handle exceptions which JDBC API throws
    - Test DAO in main()
- [Problem] 초난감 DAO의 문제점?
  - 예측치 못한 변화에 유연하게 대처하기 위한 유연성과 확장성을 위해 리팩토링 필요.
- [How] 어떻게 리팩토링 하죠?
  - DAO 분리
    - SoC를 통한 관심사 분리
      - Create/Close Connection
      - Prepare/Execute SQL
    - 중복부분의 추출
      - 중복 부분 : Create/Close Connection
      - How : Extract Method
    - 중복부분의 분리
    - 상속을 통한 중복부분의 기능확장
      - 다양한 DB선택을 위한 상속을 통한 확장 (Template Method Pattern / Factory Method Pattern)
        - Template Method Pattern
          - Purpose? 
            - 자주 사용하지만 고정적인 작업 흐름과, 자주 변화하는 부분을 SoC
          - How? 
            - 고정적인 작업 흐름은 SuperClass, 변화하는 부분은 SubClass에게 위임 by overriding hook or abstract method
        - Factory Method Pattern
          - Purpose?
            - 객체 생성 부분과 이용부분을 SoC
          - How?
            - 객체 생성부분은 SubClass에게 위임하여, SuperClass에선 단순히 객체 이용
  - DAO 확장
    - 상속을 통한 확장의 문제점
      - 1. 상속은, SuperClass가 변한다면, 캡슐화를 깰 가능성이 많다.
        - When?
          - 메소드 재정의 하는 경우
          - 메소드 추가 되는 경우
      - 2. 낮은 확장성 : 자바는 1개의 상속만 지원하기에, 기존 클래스가 이미 상속하고 있다면 확장하기 힘들다
    - 상속대신 클래스 분리하여 Composition
    - 결합도를 낮추기 위한 IF 도입
    - 관계설정 책임의 분리
      - 제3자를 통한, 런타임 의존관계 설정
    - 원칙과 패턴
      - OCP로 지금까지 리팩토링한 코드의 이점 설명가능하다
  - IoC 적용하기 
    - Object Factory를 통한 객체생성 및 관계설정 책임의 분리
      - 이를 통해, 비지니스 로직과 개체 생성 및 의존관계 설정의 책임을 분리
    - 제어권 이전을 통한 제어관계 역전
      - Object Factory에서 의존 구현 개체 선택, 생성, 의존관계설정, 사용제어권을 갖는다 = IoCFactory
  - Spring IoC 이용하기 
    - What?
      - Spring의 Core는, IoC Container
        - BeanFactory
        - ApplicationContext
        - IoC Container
        - DI Container
      - DI Container needs 설정정보
    - How?
      - DaoFactory를 설정정보 저장하는 클래스로 변환
    - Why?
      - Client는 구체적 팩토리를 몰라도 된다
      - DI Container는 종합적인 IoC서비스 제공해준다
      - 다양한 DL(Dependency LookUp) 방법 존재
    - Term
      - Spring Bean?
        - 스프링이 IoC방식으로 관리하는 개체
        - IoC방식?
          - 스프링이 제어권을 가지고 개체 생성, 의존관계설정, 생명주기 관리
      - Bean Factory?
        - 스프링 IoC담당 컨테이너. 빈 등록생성 제어 관리. <<BeanFactory>>
      - Application Context?
        - 빈팩토리 확장한 IoC컨테이너, 추가적으로 앱 전반에 걸쳐 구성 요소 제어담당 및 부가 기능 제공
      - 설정정보?
        - IoC적용하기위해 사용되는 메타정보.
        - 객체 생성 정보 및 의존관계 정보
  - DaoFactory와 AppContext의 차이점
    - 개체 동일성 불일치 문제
      - AppContext이 관리하는 Bean의 기본Scope는 Singleton
      - 웹 환경에서 Service개체방식으로 활용을 위해 당연히 Singleton이 효율이 좋다
    - 기존 자바 싱글톤의 문제점?
      - 개체에 많은 제약이 가해지기에 추천하지 않는다.
      - (-) private 생성자이기에, 객체지향설계의 장점을 적용하기 어렵다 (상속 및 다형성)
      - (-) 전역에서 개체 접근 가능하다
      - (-) 테스트 Mocking하기 어렵다
      - (-) 서버 환경에서 정말로 개체 하나만 보장하기가 어렵다
    - 스프링에서는 어떻게 해결하였는가?
      - 개체의 생성 시점, 관계 설정 제어를 DI　Container에서 관리하기에, 기존 싱글톤의 문제점을 어느정도 해결한 싱글톤 Bean을 생성가능하다
      - 핵심은, 평범한 POJO를 싱글톤으로 활용가능하다는 점
      - = 객체지향설계의 장점 적용 가능, No private 생성자, 전역 객체 접근 불가, Mocking 가능
      - Singleton Bean은 State-less가 기본이어야 한다.
  - DI
    - IoC and DI
      - IoC 용어가 너무 일반이고 포괄적인 개념이기에, 스프링이 제공하는 IoC방식을 명확하게 표현하기 위해 DI라는 용어 사용
        - IoC?
          - What?
            - 개체가 능동적으로 관계를 맺고 의존관계 설정에 참여하는 것이 아닌, 제3자가 제어를 하는 것
          - Why?
            - OCP (유연성있는 확장)
          - How?
            - 예제에서 본 DaoFactory or Use Spring DI Container
        - DI?
          - What?
            - 의존관계 주입
            - 개체 레퍼런스를 외부로부터 제공받고, 이를 통해 다이나믹하게 의존관계가 만들어 지는것.
            - 자신이 사용할 개체 선택 및 생성 제어권은 외부에 넘기고, 자신이 주입받은 객체를 수동적으로 사용하는 점에서 IoC개념에 들어맞는다.
          - Why?
            - OCP (유연성있는 확장)
          - How?
            - Constructor Injection (Mainly Use)
              - (+) POJO로 작성 가능
              - (+) 의존관계 투명성
              - (+) 불변성 - final 붙일 수 있다
              - (+) Testability - DI Container의존성 없다
              - (-) 순환 참조 문제
                - CodeSmell. 애초에 디자인이 잘못된 신호 일수 있다
                - Use Lazy Annotation
              - (-) 같은 타입 개체 2개 이상 존재할때, 순서뒤바뀐 경우 체크가 어렵다
                - Setter이용한 빌더패턴으로 해결?
            - Field Injection
              - (-) DI Container 의존적
              - (-) 의존관계를 숨긴다
              - (-) final한 개체 생성 불가능
              - (-) Testability - Reflaction없이는 테스트가 힘들다
            - Setter Injection
              - 의존 객체가 Optional일 경우 주로 사용
          - 응용?
            - 기능 교체(전략 패턴)
            - 기능의 Dynamic교체(다이내믹 프록시 라우팅) - RunTime에 한번 DI받고도 동적으로 변경
            - 부가 기능 추가(데코레이터 패턴)
            - IF 변경 (어댑터패턴)
            - Remote Proxy
            - Lazy Loading
            - Templated and callback pattern
    - RunTime 의존관계 설정
      - 의존관계?
        - 의존하고 있는 개체가 변하면, 그 영향이 의존받는 개체에게 미치는 것 
        - 의존관계에는 방향성이 존재한다
        - IF를 통해 의존관계를 제한하면, 어느 정도 변경에서 자유로워 진다
      - 런타임 의존관계?
        - 구현개체와 그 클라이언트를 런타임시 연결해 주는 방식.
        - 런타임 전까진 어떤 구현 개체가 주입될 것인지 알 수 없다.
        - 런타임 의존관계 설정을 위해서는 다음 3가지 조건 충족 필요.
          - 1. 클래스모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않아야 한다 (IF의존 필요)
          - 2. 제3자가 런타임시의 의존관계를 결정한다 (DI Container)
          - 3. 의존관계는 개체 레퍼런스를 외부에서 주입함으로써 만들어져야 한다
    - DL(Dependency Lookup)
      - What?
        - 런타임 의존관계를 맺을 개체 선택과 생성제어권은 외부에 맡기고, 
        - 그 개체는 제3자로부터 주입받는것이 아닌, 스스로 찾는것 (getBean()등을 이용하여)
      - When?
        - 주로 앱 기동시점 (DI를 이용해 주입받을 방법이 없기에)
      - Char?
        - Spring Bean일 필요가 없다 (의존 클래스만 Bean이라면)
        - 반면, DI는 해당 개체 생성 및 초기화 권한을 가지고 있어야 하므로 SpringBean이어야 한다
