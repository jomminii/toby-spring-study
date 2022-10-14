> Note : 해당 문서는 [WorkFlowy](https://workflowy.com/s/what-i-learned/ucw29v6kAHxyhL4F)에 최적화 되어있습니다.

# Chapter4 - Exception

## What I learend?

### 예외 종류

  - Error
    - What?
      - 시스템레벨에서, 비정상적인 상황 발생시 사용.
      - 따라서, JVM에서 발생시키며, 어플리케이션코드에서 핸들링할 이유도 필요도 없다.
      - java.lang.Error Class & SubClasses
    - Exam?
      - OOME, ThreadDeath
  - Exception
    - What?
      - 애플리케이션 코드레벨에서, 예외상황 발생시 사용
      - java.lang.Exception Class & SubClasses
    - Checked (비관적 처리기법)
      - What?
        - 복구할 가능성이 있는 예외 구분
        - 명시적인 예외처리 강제
      - Exam?
        - Java: IOException 
        - JDBC: SQLException
        - 비지니스 예외(꼭 비지니스 예외를 Checked로 처리할 필요는 없지만 안정성을 위해 Checked로 구분하는 케이스도 존재)
      - When to use?
        - 예전엔 복구할 가능성이 조금이라도 보이는 예외를 Checked로 구분했지만,
        - 현재로선 복잡성, 생산성을 이유로 복구할 가능성이 확실한 예외만을 Checked로 구분하는 추세.
      - Pros?
        - 명시적인 예외처리 강제로 인한 상대적 안전성
      - Cons?
        - 불필요한 예외처리 강제
        - 실제로는 복구 불가능한 경우가 많아(특히 WebApp), Runtime예외로 래핑해서 전파하는 경우가 굉장히 많다...
    - Un-checked(Runtime 상속. 낙관적 처리 기법)
      - What?
        - 시스템 장애 or 휴먼 에러등으로 인한 프로그램상 오류 발생시 의도적으로 발생
        - 명시적인 예외처리 강제 하지 않는다
      - Exam?
        - NPE, IAE, ISE, etc.
      - When to use?
        - 복구 불가능한 예외상황 발생시
        - 복구 불가능한 Checked예외 래핑
      - Pros?
        - 명시적인 예외처리 강제하지 않으므로, 필요할 경우에만 throws선언, catch가능
      - Cons?
        - 안정성
        - 일어날 수 있는 예외상황을 미리 파악하고, 예방하려는 더 많은 주의가 필요하다.
#### 에외 처리 방법

  - 0. 원칙 : Failing Loudly Fast (빠르게, 크게 짖기)
    - Failing Fast : 오류 발생시점에서 최대한 가까운 곳에서 실패해야 한다
    - Failing Loudly : 오류는 무시되지 않고, 상위계층으로 전파하며, 적절히 기록되어 알아 차릴수 있어야 한다.
  - 1. 복구
  - 2. 회피(전파)
  - 3. 에러전환후 회피
    - Purpose
      - 1. 보다 meaningful한 예외로 전환
        - e.g, 중복 키 존재할경우, JDBC SQLException => DuplicatedUserIdException으로 전환
      - 2. 개발자가 사용하기 쉬운 예외로 전환
        - 복구 불가 Checked예외 -> UnChecked예외로 전환
      - 이 때 origin error(cause error)도 포함해 중첩예외로 만드는 것이 당연히 좋다

#### 에외 처리 전략

  - 복구 불가능 Checked예외는, ASAP Un-checked로 래핑해서 전파하기
  - 비지니스 예외는 Checked 예외로 만들어 적절한 복구 또는 대응 강제시키기
  - 하지만 케이스에 따라 Un-checked로 두고,필요에 따라 throws 선언 하여, 불필요한 catch/throws 제거를 하는 것이 더 편할 때도 있다.

#### 스프링이 제공하는 DataAccess기술의 예외처리 전략 및 원칙

1. 사용자 편의를 위해, 복구불가능 Checked예외를, Un-checked로 래핑해서 예외 발생시킨다
    - 기존 JDBC예외보다 의미가 명확하며, 사용하기 쉬운 예외로 전환
2. DB 및 DataAccess기술에 의존적이지 않은 추상화 계층 예외를 발생시킨다.

#### JDBC예외의 한계와, 스프링이 어떻게 그 한계를 넘어 추상화 계층 예외를 제공하는지 (Before JDBC v4.0) 

  - JDBC는, 어댑터패턴 이용한 PSA를 통해, 각DB벤더 API에 의존치 않고 일관적인 데이터 접근 API를 제공한다.
  - 하지만 세상에 완벽한 것은 없듯이, 다음과 같은 한계가 있다.
  - 1. 비표준SQL작성시, 결국 DB벤더에 종속적이게 된다 (DB마다 비표준SQL이 다르기에)
        - 치명적인 것은, 성능 향상을 위해 비표준SQL사용은 필수적이라는 것이다.
  - 2. JDBC의 Checked SQLException가 호환적이지 않다 (DB마다 에러종류와 원인이 다르기에)
        - DB마다 에러종류와 원인이 다름에도 불구하고, JDBC는 SQLException 하나로 소위 말하는 퉁쳤다..
        - 따라서, 해당 예외에 담긴 에러코드는, 각DB벤더가 정의한 코드가 "그대로" 들어가있다.
        - 이걸 해결하기 위해, SQL상태정보를 부가정보로 제공하지만 (getSQLState()),
        - 각 DB벤더들의 JDBC 드라이버들마다 제공하는 정보에 편차가 존재하기에 문제가 해결 되진 않았다
 - 이걸 스프링은 어떤 식으로 해결하였는가?
    - 데이터 접근 기술에 종속적이지 않은 DataAccessException 추상화 계층 예외 제공.
    - DB별 에러코드를 분류하여, 해당하는 추상화 계층 예외클래스에 매핑하는 방법으로 해결.
    - 뿐만 아니라, 자바 표준 데이터 접근 기술들의 예외까지 추상화시켜, 진정한 PSA를 제공하였다.
    - 따라서, DataAccessException 추상화 계층 예외를 이용하여, 특정 DB사용 및 데이터 접근 기술에 종속적이지 않은 DAO를 작성할 수 있게 되었다.
        - What & Why & How ???
            - DAO 계층을 따로 분리하여 사용하는 이유는 다음과 같다.
                - 1.성격다른 코드 SoC
                - 2.전략패턴이용하여 유연하게 런타임 의존성만 변경 가능키 위해
            - 이를 통해, 인터페이스를 이용하여, DAO의 "데이터 접근 사용 기술과 구현 코드"를, 전략 패턴과 DI를 통해 자유롭게 변경가능할 수 있다.
            - 하지만, 예외 정보는, 
                - JDBC를 사용하게 되면 SQLException이라는 기술 종속적인 예외정보를 클라이언트 쪽에서 처리하여야 하고,
                - 다른 표준 데이터 접근 기술로 바꾼다고 하여도, 해당 기술 종속적인 예외정보를 클라이언트에서 처리하여야 한다.
            - 따라서, DAO의 예외 처리 하나때문에, 클라이언트도 변경이 필요하게 된다.
            - 이를 해결하기 위해, 스프링이 제공하는 DataAccessException 추상화 계층 예외를 사용함으로써, 예외처리도 추상화가 가능해지게 된 것이다.
            - 결론적으로, IF사용 + Checked예외의 Un-checked예외 전환 + DataAccessException추상화, 를 통해 데이터 접근 기술과 구현방법에 독립한 DAO를 사용할 수 있게 되었다.
