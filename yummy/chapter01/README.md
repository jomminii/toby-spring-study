\- 스프링은 객체지향적

\- 스프링은 객체지향의 설계, 전략, 검증된 best practice를 프레임워크 형태로 손쉽게 적용할 수 있도록 함

## 1.1 초난감 DAO

\- 자바빈의 규약을 따르는 오브젝트 생성

```
package springbook.user.domain;

public class User {
    String id;
    String name;
    String password;


    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

\- 사용자 정보를 DB에 넣고 관리할 수 있는 DAO클래스 생성

\- DAO란 db를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트이다.

\- JDBC를 이용하는 일반적인 작업 순서는 다음과 같다.

> 1\. DB연결을 위한 Connection을 가져온다.  
> 2\. SQL을 담은 Statement or PrepareStatement를 만듦  
> 3\. 만들어진 Statement를 실행한다.  
> 4\. 조회의 경우 SQL쿼리의 실행 결과를 ResultSet으로 받아서 정보를 저장할 오브젝트에 옮겨준다.  
> 5\. 생성된 Connection,Statement,ResultSet은 반드시 생성한 순서 반대로 닫아준다.

**UserDao**

```
package springbook.user.dao;

import springbook.user.domain.User;

import java.sql.*;

public class UserDao {

    public void add(User user) throws ClassNotFoundException, SQLException {
        // JDBC Driver클래스 로드하기 위함
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook","username","userpwd");
        PreparedStatement ps = c.prepareStatement("insert into users(id,name,password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook","username","userpwd");
        PreparedStatement ps = c.prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }
}
```

\- 해당 dao의 기능이 제대로 동작하는지 확인하기 위해서 웹 브라우저로 테스트를 하는 방법이 있는데, 이는 효율적이지 않다. 

\- 따라서 자주 사용하곤 하는 static 메서드 main()을 이용하여 코드가 잘 동작하는지 검증

```
public static void main(String[] args) throws SQLException, ClassNotFoundException {
        UserDao dao = new UserDao();

        User user = new User();
        user.setId("whiteship");
        user.setName("백기선");
        user.setPassword("married");

        dao.add(user);

        System.out.println(user.getId() + "등록 성공");
        User user2 = dao.get(user.getId());
        System.out.println(user2.getName());

        System.out.println(user2.getPassword());

        System.out.println(user2.getId() + "조회 성공");

    }
```

 - 결과는 성공적으로 잘 출력되나, 사실 위의 UserDao클래스는 객체지향적으로 설계되지 않음.

 - 스프링은 해당 dao코드를 객체지향적으로 짤 수 있게끔 설계되었으며, 이를 다음 chapter에서 설명. 

## 1.2 DAO의 분리

\- 개발자가 객체를 설계할 때 가장 염두에 두어야하는 사항은 '미래의 변화를 어떻게 대비할 것인가'임

\- 설계의 기준은 '분리와 확장'임

\- 분리란, 관심이 같은 것끼리는 하나의 객체 안으로 도는 친한 객체로 모이게 하는 것이고, 관심이 다른 것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 하는것.

```
package springbook.user.dao;

import springbook.user.domain.User;

import java.sql.*;

public class UserDao {

    public void add(User user) throws ClassNotFoundException, SQLException {
        // JDBC Driver클래스 로드하기 위함
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook","username","userpwd");
        PreparedStatement ps = c.prepareStatement("insert into users(id,name,password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook","username","userpwd");
        PreparedStatement ps = c.prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }
}
```

\- 위의 코드에서 문제가 되는 점은, get()과 add()메서드의 DB커넥션을 가져오는 부분의 중복임.

\- 이러한 중복이 여러 곳에서 일어나면, 접속정보를 변경하려 할때마다 모두 수정해주어야 함.

이를 해결하기 위해선, 중복 코드의 메소드를 다음과 같이 추출해주는 것이다.

```
   public void add(User user) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = getConnection();
        ...
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = getConnection();
			....
    }
   
   private Connection getConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook","username","userpwd");
        return c;
    }
```

\- 이런 식으로 메서드를 따로 분리해두면, DB연결과 관련된 부분에 변경이 일어났을 경우, getConnection()이라는 메서드만 수정하면 된다. 

\- 관심의 종류에 따라 코드를 '분리'하는 좋은 예. 관심이 독립적으로 존재하므로, 수정의 절차가 간소화됨.

\- 결과는 달라진 것이 없지만, 유지보수가 용이하게 구조를 변경하는 작업을 refactoring이라고 함

\- 위와 같이 메소드로 중복된 코드를 뽑아내는 것을 리팩토링에서는 '메소드 추출(extract method)'이라고 부름

---

\- 다만, 여기서 더 나아가 userDao 클래스의 종류가 여러가지이고, dao별로 각기 다른 종류의 DB를 사용하고 싶을때, 

다음과 같이 리팩토링 할 수 있다.

```
public abstract class UserDao {

    public void add(User user) throws ClassNotFoundException, SQLException {
        // JDBC Driver클래스 로드하기 위함
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = getConnection();
...

        ps.close();
        c.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = getConnection();
...

        return user;
    }

    public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
```

```
public class DUserDao extends UserDao{
    @Override
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook","username","userpwd");
        return c;
    }
}
```

```
public class NUserDao extends UserDao{
    @Override
    public Connection getConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook","username","userpwd");
        return c;
    }
}
```

\- 위 코드에 대해 설명하자면,  큰 흐름의 데이터 등록 절차는 UserDao가, 어떤 DB를 Connect 할 것인지에 관해서는 NUserDao, DUserDao가 클래스 레벨로 독립적으로 구분하여 담당하면서 변경 작업이 용이해짐

\- 위와 같이 슈퍼클래스 기능의 일부를 떼내어서, 추상메서드나 오버라이딩이 가능한 protected메소드 등으로 만든 뒤, 서브클래스에서 이런 메소드를 필요에 맞게 구현하는 방법을 **템플릿 메소드 패턴**이라고 함.

\- 서브 클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것은 **팩토리 메서드 패턴**이라 함 

\- 단점 1. 상속의 단점을 물려받음. 예컨대 UserDao가 다른 목적을 위해 이미 상속을 사용하고 있다면 java class 특성상 다중상속 불가능

\-  단점 2.  확장된 기능인 DB커넥션을 생성하는 코드를 다른 DAO클래스에 적용할 수 없음. UserDao외에 다른 dao클래스들이 만들어진다면, getConnection의 구현코드가 매 dao클래스마다 중복됨

## 1.3 DAO의 확장

![제목 없는 다이어그램 drawio](https://user-images.githubusercontent.com/40001921/190834910-b1cffd1e-3bb7-4a54-aea4-b58be9319271.png)

\- DB 커넥션 하는 부분을 이전과 달리 상속관계가 아닌 독립된 클래스로 만들고, 이를 UserDao가 이용하게끔 분리함

```
public class SimpleConnectionMaker {

    public Connection makeNewConnection() throws ClassNotFoundException, SQLException{
        Class.forName("com.mysql.jdbc.Driver");
        Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook","userid","userpwd");
        return c;
    }
}
```

```
public class UserDao {

    private SimpleConnectionMaker simpleConnectionMaker;

    public UserDao() {
        simpleConnectionMaker = new SimpleConnectionMaker();
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        // JDBC Driver클래스 로드하기 위함
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = simpleConnectionMaker.makeNewConnection();
        PreparedStatement ps = c.prepareStatement("insert into users(id,name,password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c =  simpleConnectionMaker.makeNewConnection();
        PreparedStatement ps = c.prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }
```

\- UserDao의 코드가 SimpleConnectionMaker라는 특정 클래스에 종속되어있어서, N사와 D사의 dao를 분리한 것 처럼 디비 접속정보를 확장함이 불가능해졌음

---

\- 이러한 단점을 인터페이스 도입을 통해 리팩터링하여 해소할 수 있다.

\- 추상화(공통적인 성격을 뽑아내어 따로 분리해주는 것) 작업을 진행하면 됨.

\- 자바에서 추상화를 위해 제공하는 가장 유용한 도구는 인터페이스

\- 인터페이스를 통해, 어떤 일만 하겠다고 정의를 하면 되고, 해당 인터페이스를 구현하는 클래스에서 필요한 내용을 구현하면 됨.

![제목 없는 다이어그램 drawio (1)](https://user-images.githubusercontent.com/40001921/190834936-ccac2a56-478b-425c-9268-5945c3cb8bff.png)


\- ConnectionMaker를 정의하고 이를 각자 구현함

\- UserDao입장에서는 ConnectionMaker인터페이스 타입의 오브젝트라면 어떤 클래스로 만들어졌든지 상관없이, makeConnection()메서드를 호출하기만 하면 Connection타입의 오브젝트를 만들어서 돌려줄 것이라고 기대할 수 있음

```
import java.sql.Connection;
import java.sql.SQLException;

public interface ConnectionMaker {
    public Connection makeConnection() throws ClassNotFoundException, SQLException;
}
```

```
public class DConnectionMaker implements ConnectionMaker{
    @Override
    public Connection makeConnection() throws ClassNotFoundException, SQLException {
        // D사의 독자적 방법으로 Conncecion을 생성하는 코드...
        return null;
    }
}
```

```
public class UserDao {

    private ConnectionMaker connectionMaker;

    public UserDao() {
        connectionMaker = new DConnectionMaker();
    }

    public void add(User user) throws ClassNotFoundException, SQLException {
        // JDBC Driver클래스 로드하기 위함
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c = connectionMaker.makeConnection();
        PreparedStatement ps = c.prepareStatement("insert into users(id,name,password) values(?,?,?)");
        ps.setString(1, user.getId());
        ps.setString(2, user.getName());
        ps.setString(3, user.getPassword());

        ps.executeUpdate();

        ps.close();
        c.close();
    }

    public User get(String id) throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.cj.jdbc.Driver");
        Connection c =  connectionMaker.makeConnection();
        PreparedStatement ps = c.prepareStatement("select * from users where id = ?");
        ps.setString(1, id);

        ResultSet rs = ps.executeQuery();
        rs.next();
        User user = new User();
        user.setId(rs.getString("id"));
        user.setName(rs.getString("name"));
        user.setPassword(rs.getString("password"));

        rs.close();
        ps.close();
        c.close();

        return user;
    }
```

\-  다만, 초기에 한 번 어떤 클래스의 오브젝트를 사용할지를 결정하는 생성자의 코드는 제거되지 않고 남아 있음

\- 위의 코드로는 UserDao의 생성자 메서드를 직접 수정하지 않고 고객에게 자유로운 DB커넥션 확장 기능을 가진 UserDao를 제공할 수 없다.

---

\- 아래 코드는, 위의 결함을 개선한 코드임.

```
public class UserDao {

    private ConnectionMaker connectionMaker;

    public UserDao(ConnectionMaker connectionMaker) {
        this.connectionMaker = connectionMaker;
    }
```

```
public class UserDaoTest {
    public static void main(String[] args) {
        // userDao가 사용할 ConnectionMaker 구현 클래스를 결정하고 오브젝트를 만든다.
        ConnectionMaker connectionMaker = new DConnectionMaker();
        
        UserDao dao = new UserDao(connectionMaker);
    }
}
```

\- 생성자 부분에 범용적으로 ConnectionMaker 오브젝트를 받을 수 있도록 수정하여, 어떤 ConnectionMaker를 사용할지에 대한 결정 책임을 클라이언트에 떠넘기면 해결됨. 

\- 즉 UserDaoTest는 UserDao와 ConnectionMaker구현 클래스와의 런타임 오브젝트 의존 관계를 설정하는 책임을 담당하고 있음 

![제목 없는 다이어그램 drawio (2)](https://user-images.githubusercontent.com/40001921/190834971-f4066622-9b3b-41db-ab85-86700101c3ac.png)

\- 위와 같은 구조로, UserDao와 ConnectionMaker클래스를 분리하고, 서로 영향을 주지 않으면서 필요에 따라 확장할 수 있는 구조로 만들었음

\- 이제까지의 리팩터링 작업들은 다음의 객체지향 원칙을 따르며 개선한 작업임

>  **개방 폐쇄 원칙(OCP, Open-Closed Principle)  : 클래스나 모듈은 확장에는 열려있어야 하며, 변경에는 닫혀있어야 한다.** 

> **높은 응집도 : 하나의 모둘, 클래스가 하나의 책임 또는 관심사에만 집중되어있다는 것.**  
>   
> 예컨대 ConnectionMaker를 예로 들 수 있음. 인터페이스를 생성하여 DB연결기능을 독립시켜서, DB관련 기능을 추가하려는 경우 해당 인터페이스 하위 클래스만 수정 또는 추가해주면 되므로, 다른 관심사에 속하는 클래스를 굳이 수정해줄 필요가 없음. 즉 이와 같은 사례는 높은 응집도를 가졌다고 할 수 있다.

> **낮은 결합도: 책임과 관심사가 다른 오브젝트 또는 모듈과는 낮은 결합도, 즉 느슨하게 연결된 형태를 유지하여,연결에 필요한 최소한의 방법만 간접적인 형태로 제공하는 것. 하나의 오브젝트가 변경이 일어날 때에 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도.** 가령 UserDao와 ConnectionMaker의 관계가 그렇다. UserDao는 구체적인 ConnectionMaker의 구현 클래스를 알 필요도 없음. 왜냐하면 ConnectionMaker라는 인터페이스를 통해 낮은 결합도로 최소한으로 연결되어있기 때문. 이 낮은 결합도를 통해 UserDao나 기타 Dao에 영향을 주지 않음

> **전략 패턴: 자신의 기능맥락에서 필요에 따라 변경이 필요한 알고리즘 클래스를 필요에 따라 바꿔 사용할 수 있기 하는 디자인 패턴. 이때 알고리즘이란, 독립적인 책임으로 분리가 가능한 기능을 뜻함.** UserDao가 전략 패턴 컨텍스트에 해당함. DB연결방식이라는 컨텍스트를 ConnectionMaker라는 인터페이스로 정의하고, 이를 구현한 클래스, 즉 전략을 바꾸면서 사용했기에 전략패턴과 관련이 있다고 볼 수 있음.


## 1.4 제어의 역전

\- Inversion Of Control : 제어의 역전 

\- 팩토리: 객체의 생성 방법을 결정하고 그렇게 만들어주는 오브젝트를 돌려주는 것

\- 아래와 같이 Factory클래스를 분리하여, userDao 와 connectionMaker를 연결짓는 부분을 만든다.

```
package springbook.user.dao;


public class DaoFactory {
    public UserDao userDao(){
        return new UserDao(connctionMaker());
    }

    /*public AccountDao accountDao(){
        return new AccountDao(connctionMaker());
    }

    public MessageDao messageDao(){
        return new MessageDao(connctionMaker());
    }*/

    public ConnectionMaker connctionMaker() {
        return new DConnectionMaker();
    }
}
```

## 1.5 스프링 IoC

\- **Bean** : 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트. 제어의 역전이 적용되었음

\- **Bean Factory** : 빈의 생성과 관계 설정 같은 제어를 담당하는 IoC오브젝트. 빈 생성과 제어에 초점이 맞추어져있음. 

\- **ApplicationContext** : IoC방식을 따라 만들어진 일종의 빈 팩토리로, 애플리케이션 전반에 걸쳐 모든 구성요소의 제어 작업을 담당하는 IoC엔진. 스프링 컨테이너를 이야기함. BeanFactory를 상속한다.

\- **Configuration MetaData**  : IoC를 적용하기 위해 사용하는 메타정보. 

\- **Spring FrameWork** : IoC컨테이너 ( 빈팩토리 초점),ApplicationContext를 포함해서 스프링이 제공하는 모든 기능

\- 별도의 정보를 참고해서 빈의 생성, 관계설정 등의 제어 작업을 총괄한다. 

```
@Configuration
public class DaoFactory {
    @Bean
    public UserDao userDao(){
        return new UserDao(connctionMaker());
    }

    /*public AccountDao accountDao(){
        return new AccountDao(connctionMaker());
    }

    public MessageDao messageDao(){
        return new MessageDao(connctionMaker());
    }*/

    @Bean
    public ConnectionMaker connctionMaker() {
        return new DConnectionMaker();
    }
}
```

\- @Configuration : 애플리케이션 컨텍스트 도는 빈 팩토리가 사용할 정보라는 것을 명시하는 스프링 어노테이션

\- @Bean : 오브젝트 생성을 담당하는 IoC용 메서드에 붙임

```
public class UserDaoTest {
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao dao = context.getBean("userDao", UserDao.class);

    }
}
```

\- @Configuration이 붙은 자바 코드를 설정정보로 사용하려면 AnnotationConfigApplicationContext를 사용하여, 생성자 파라미터에 해당 클래스를 넣고 사용한다.

\- getBean()은 ApplicationConext가 관리하는 오브젝트를 요청하는 메서드. userDao라는 빈은 생성자 메서드 이름과 동일함. 

\- Application context를 사용했을 때의 장점.

1\. 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다. => 일관된 방식으로 오브젝트를 가져올 수 있기 때문.

2\. 애플리케이션 컨텍스트는 종합 IoC서비스를 제공해준다. => aop,자동 생성등과 같은 다양한 기능 제공.

3\. 애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다.  => getBean() 이외에도 타입만으로 빈을 검색하거나 애노테이션이 설정이 되어있는 빈을 찾을 수 있음.

## 1.6 싱글톤 레지스트리

\- 스프링은 서버 환경에서 싱글톤이 만들어져서 서비스 오브젝트 방식으로 사용되는 것은 적극 지지함.

\- 자바의 싱글톤 패턴의 구현 방식은 여러가지 단점이 있기에, 스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공함 => 싱글톤 레지스트리

\- 싱글톤 레지스티리는 자바의 일반적인 싱글톤 패턴과 달리, private이나 스테틱 메서드를 사용하지 않고도 평범한 자바클래스를 싱글톤으로 활용할 수 있게 해줌 

\- 빈 스코프 : 빈의 생성 주기 및 적용되는 범위 => 스프링은 싱글톤 스코프로, 컨테이너 내에 한 개의 오브젝트만 마들어짐.