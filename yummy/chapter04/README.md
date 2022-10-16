## 예외

- 예외란 정상적인 처리에서 벗어나는 경우에 이를 처리하기 위한 방법
- 예외는 반드시 적절하게 복구되든지, 작업을 중단시키고 운영자 또는 개발자에게 통보되어야 함
- 예외를 나쁘게 처리하는 두 가지 방법은 다음과 같다.
 1. 예외를 잡고 아무것도 하지 않음
 2. throws Exception을 남발하여 예외를 위로 던지기만 하고 아무것도 하지 않음.

## 예외의 종류

1. Error
- java.lang.Error클래스의 서브 클래스들
- 에러는 시스템에 뭔가 비정상적인 상황이 발생했을 경우 사용된다.
- ex) OutOfMemoryError, ThreadDeath 에러

2. Exception과 체크 예외
- CheckedException과 UncheckedException으로 나뉨
- CheckedException은 반드시 예외를 처리하는 코드를 함께 작성해야 함. (throws or catch)

3. RuntimeException과 언체크 / 런타임 예외
- 명시적인 예외처리를 강제하지 않기 때문에 언체크 예외라고 불림
- UncheckedException은 RuntimeException을 상속하는 예외
- 주로 프로그램의 오류가 있을 때 발생하도록 의도된 것.
- 개발자가 부주의해서 발생할 수 있는 경우에 발생하도록 만든 것
- 예상하지 못했던 예외상황에서 발생하는게 아니기 때문에, catch나 throws안써도 됨
- ex) NullPointerException, IllegalArgumentException

## 예외처리 방법
- 예외 복구: 예외상황을 파악하고, 문제를 해결해서 정상 상태로 돌려놓으려는 것
- 예외처리 회피 : 예외 처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것
``` 
public void add() throws SQLException {
      try{
        // JDBC API
      }
      catch(SQLException e) {
        throw e;
      }
     } 
```
- 예외 전환: 적절한 예외로 전환해서 예외를 메소드 밖으로 던지는 것 
   이렇게 조치하면 의미가 분명한 예외가 던져져서 서비스 계층 오브젝트에서 적절한 복구 작업을 시도할 수 있음
``` 
public void add(User user) throws DuplicateUserIdException, SQLException {
        try {
            // JDBC를 이용해 user정보를 DB에 추가하는 코드 또는
            // 그런 기능을 가진 다른 SQLExcpetion을 던지는 메소드를 호출하는 코드
        }
        catch(SQLException e){
            // error 코드가 mysql의 duplicate entry이면 예외 전환
            if(e.getErrorCode() == MysqlErrorNumbers.EP_DUP_ENTRY)
                throw DuplicateUserIdExcpetion();
            else
                throw e;
        }
    }
```

  보통 전환하는 예외에 원래 발생한 예외를 담아서 중첩 예외로 만든다.
  initCause()메서드를 이용해서 처음 발생한 예외가 무엇인지 확인할 수 있음
  
```
    catch(SQLException e){
        ...
        throw DuplicateUserIdException().initCause(e);
    }
```
  또는 예외를 처리하기 쉽고 단순하게 만들기 위해 포장하는 방법이 있다. 
  복구 가능한 예외가 아닐 경우 런타임 에러, 예컨대 RuntimeException을 상속 받는 EjbException으로 포장하여 던짐.

## 예외전환
 예외전환의 목적 2가지는 다음과 같다.
 1. 런타임 예외로 포장해서 굳이 필요하지 않은 catch/ throws를 줄여주는 것
 2. 로우레벨의 예외를 좀 더 의미있고 추상화된 예외로 바꿔서 던져주는 것

## JDBC의 한계
 jdbc는 자바를 이용해 DB에 접근하는 방법을 추상화된 api형태로 정의하고, 각 db업체가 jdbc표준을 따라 만들어진 드라이버를 제공하게 해준다. 
 표준화된 jdbc api가 db프로그램 개발 방법을 학습하는 부담은 확실히 줄여주지만, db를 자유롭게 변경해서 사용할 수 있는 유용한 코드를 보장하지는 못하는데 그 이유는 다음과 같다.
1. 대부분의 db에서 비표준 문법과 기능들도 제공함 => 이는 특정 db에 종속된 dao를 만들게 함.
2. DB마다 에러의 종류와 원인도 제각각이기 때문에, 에러코드도 제각각임

=> 스프링은 이를 해결하기 위해 수많은 런타임 예외 클래스(DataAccessException와 이를 상속하는 클래스들)를 정의하여 제공함. 
예컨대 스프링의 JdbcTemplate을 상요하면 JDBC에서 발생하는 DB관련 예외는 거의 신경쓰지 않아도 됨.

## DataAccessException
- 스프링의 DataAccessException은 자바의 주요 데이터 액세스 기술에서 발생할 수 있는 대부분의 예외를 추상화하고 있음.
- Data Access 에서 발생 가능한 대부분의 예외를 계층 구조로 분류해 놓았음.




