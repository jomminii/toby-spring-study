### 3장 템플릿

- 템플릿 : 변경 거의 없음 + 일정 패턴으로 유지되는 부분을 독립시켜 활용
- 스프링에 적용된 템플릿/콜백 기법 - JdbcTemplate 기반
    - ex) DB 접근시 예외 처리 - JDBC try-catch-finally
    - **템플릿 메소드 패턴** : 상속을 활용한 확장 → 한계 존재
        - 변하지 않는 부분은 슈퍼 클래스에 두고, 변하는 부분은 추상 메소드로 정의해 서브클래스에서 오버라이드해 새롭게 정의해서 사용함
        
        ```java
        abstract protected PreparedStatement makeStatement(Connection c) throws 
        	SQLException;
        
        // 위 추상클래스를 상속 받아 구현하는 서브 클래스
        public class UserDaoDeleteAll extends UserDao {
        	protected PreparedStatement makeStatement(Connection c) throws SQLException {
        	  PreparedStatement ps = c.prepareStatement("delete from users");
        		return ps;
        	}
        }
        ```
        
        - 위와 같은 템플릿 메소드 패턴은 DAO 로직마다 상속을 통해 새로운 클래스를 만들어야 한다는 제한이 있다.
        - 확장구조가 이미 클래스 설계 시점에 고정된다는 단점이 하나 더 있다.
    - **전략 패턴** : 인터페이스 활용
        - Client -전략→ Context → Strategy → Strategy1 , Strategy2 …
        - Context 안에 contextMethod()에서 일정 구조를 갖고 동작하다가 특정 확장 기능은 Strategy 인터페이스를 통해 외부의 전략 클래스에 위임
        - deleteAll() 메서드에서 JDBC를 이용해 DB를 업데이트하는 작업이 변하지 않는 Context를 가지므로 contextMethod()에 해당함
        - PrepareStatement를 만들어줄 외부 기능을 호출하는게 Strategy에 해당
        
        ```java
        public interface StatementStrategy {
        	PreparedStatement makePreparedStatement(Connection c) throws SQLException;
        }
        
        // 위 인터페이스를 구현하는 전략 클래스
        public class DeleteAllStatement implements StatementStrategy {
        	public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        		PreparedStatement ps = c.preparedStatement("delete fron users");
        		 return ps;
        	}
        }
        
        // context 코드 - client가 컨텍스트를 호출할 때 전략을 매개변수로 넘김
        public void jdbcConnectionWithStatementStrategy(StatementStrategy stmt) throws ...
        
        // 클라이언트의 책임을 담당할 메소드
        public void deleteAll() throws SQLException {
        	StatementStrategy st = new DeleteAllStatement(); // 선택한 전략 오브젝트 생성
        	jdbcConnectionWithStatementStrategy(st); // context 호출하면서 전략 전달
        }
        ```
        
    - **템플릿 콜백 패턴** : 전략 패턴 기본 구조 + 익명 내부 클래스 활용
        - 컨텍스트 → 템플릿, 익명 내부 클래스로 만들어진 객체 → 콜백
        - Callback은 실행되는 것을 목적으로 다른 오브젝트의 메소드에 전달되는 오브젝트, 템플릿 안에서 호출되는 것이 목적
        - 보통 단일 메소드 인터페이스를 사용한다.
        - 템플릿/콜백 패턴의 일반적인 작업흐름
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4413e3d5-b25c-44c3-ac45-d196f82b0eea/Untitled.png)
        
        - Callback을 분리해 재활용
            
            ```java
            private void executeSql(final String query) throws SQLException {
            	this.jdbcContext.workWithStatementStrategy(
            		new StatementStrategy() {
            			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
            				return c.prepareStatement(query);
            			}
            		}
            	);
            }
            ```
            
- 드물지만 인터페이스를 사용하지 않는 클래스를 직접 의존하는 DI가 등장하는 경우도 있다. → 왜? 클래스끼리 이미 강한 응집도를 갖고 있어 테스트에서도 다른 구현으로 대체해 사용할 이유가 없다면 굳이 인터페이스를 두지 않고 결합 관계를 허용함
    - 단, 이 경우는 가장 마지막에 고려해볼 사항이다.
- 템플릿과 콜백 타입이 다양하게 변경될 수 있다면 제네릭 활용
- 템플릿/콜백 설계 시 그 사이 주고 받는 정보에 관심을 둬야 한다.