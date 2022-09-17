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

[##_Image|kage@becHG4/btrLVYBH7qh/3dGey7PaWStBka9fhygVKK/img.png|CDM|1.3|{"originWidth":265,"originHeight":105,"style":"alignCenter"}_##]

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

<!--[if IE]><meta http-equiv="X-UA-Compatible" content="IE=5,IE=9" ><![endif]-->
<!DOCTYPE html>
<html>
<head>
<title>Dao File</title>
<meta charset="utf-8"/>
</head>
<body><div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers tags lightbox&quot;,&quot;edit&quot;:&quot;_blank&quot;,&quot;xml&quot;:&quot;&lt;mxfile host=\&quot;app.diagrams.net\&quot; modified=\&quot;2022-09-12T08:53:05.271Z\&quot; agent=\&quot;5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36\&quot; etag=\&quot;JkUr828wV7RBBGzWd8hm\&quot; version=\&quot;20.3.0\&quot; type=\&quot;device\&quot;&gt;&lt;diagram id=\&quot;LVzI97gPg4KVyFokNSft\&quot; name=\&quot;페이지-1\&quot;&gt;7Vnfb9s2EP5rDGwPDSTql/MY21n3kG7GgqHtIy3REhuKNCi6tvvX7yiSlmTZadraddAJMBLd8UiR933HO1KjYFpu30q8Kt6JjLAR8rLtKJiNEIriEP5qxc4okiQ2ilzSzKj8RvFIvxCr9Kx2TTNSdQyVEEzRVVeZCs5Jqjo6LKXYdM2WgnXfusI56SkeU8z62vc0U4XRjlHS6P8kNC/cm/341rSU2BnblVQFzsSmpQruR8FUCqHMU7mdEqZ95/xi+v1xonU/MUm4ekmHyd1s/iTKaHk/+Xu+KKtP6dP8jR3lM2Zru+B/KyJnWNg5q51zRLWhJcMcpMlScPVoWzyQ04Ky7AHvxFpPpFI4fXLSpBCSfgF7zKDJBwU0S2VxRrr3kjI2FUxIUHBRv6Dp9KgHs6+RpIJuc7dg/0D1Dm87hg+4Um6CgjG8quiinrLuWGKZUz4RSonSGllPEKnI9qSL/T1wQHgiSqLkDkwc2y3Ulut+YMm/aZjjh9amaLHmdmwJa8ma70du8IQHC+k3wIt68OIs++33EYIgDPxm1S2cYfWqhkmKJ3KAyxGoMKM5B5GRpe6m3Ucheu6sWomVHmyFU8rzh9pmFjaaf6wPtEpA3yWrI6SgWUa4RlIorPBiz7SVoFzVToom8ANXTr2baBTBxKcg+40MP20u1VRwWAumNYIEOLEhmhcvg/t0zPQ5YEF34f41zFF8IcyDHuY5UQPml8Q8jK+MeXgM86lJiVTwAf1Lop+gK6MfDUn8jEk8CMNOFk9euKFfLInHQxI/f0hHrzuJJ0MS/9mYXz2Jj4ckfj30r57Eb3vo9zB2KD3gBWFzUVFNC2ha2Ax4AsZClU3e1vRwFxlIawq80sOX21zf39yYOxNk/kN7fW/j3cT6UTs81HhyodLifBkXeQfn5mjcB8M7AoY7b58dDHf71ELjr59aU8W/Vk2Fkm5NheIXJtgIXQrh/s3XsNue88jkHSfEngBX3m79/pF5NkT490d4GHT38FcQ4f1j8RDh54zw8HVHuCugnquoCM/uTKEzSxmuKpp26yUp1jwjmXU72VL1oS6HQit91HY3PlDNyLNty3S2awlzIimsi0in47BGM1jkxI/ttmaoWnJjmSWQrPfV6iBQYZliLVPyjINskQWbEsTFd2zlLWSjZ4ozSRhW9HN3vsfQtm+Ya3K3Sodxd2MJ0UHRZ9Zpe6HW96+DgYKDHSpIDgYyjugNVJNvv+wf4GP/Yv58fAziDiO9JPmlGXli6/mfMBIognctM5sOvomzIDafno158/0+uP8P&lt;/diagram&gt;&lt;/mxfile&gt;&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/js/viewer-static.min.js"></script>
</body>
</html>

\* (발그림 죄송... 저 화살표 부분이 UserDao에 연결되어있다고 가정해주십시오)

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

<!--[if IE]><meta http-equiv="X-UA-Compatible" content="IE=5,IE=9" ><![endif]-->
<!DOCTYPE html>
<html>
<head>
<title>separate</title>
<meta charset="utf-8"/>
</head>
<body><div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers tags lightbox&quot;,&quot;edit&quot;:&quot;_blank&quot;,&quot;xml&quot;:&quot;&lt;mxfile host=\&quot;app.diagrams.net\&quot; modified=\&quot;2022-09-12T10:12:19.611Z\&quot; agent=\&quot;5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36\&quot; etag=\&quot;LbzV3Md-c6SCGXMNaxov\&quot; version=\&quot;20.3.0\&quot; type=\&quot;device\&quot;&gt;&lt;diagram id=\&quot;gjivJGeRaO1PcaMqxmMY\&quot; name=\&quot;페이지-1\&quot;&gt;7Zdta9swEMc/TWB70eGHPPVl47QbrIWybPRlUa2LLSrrjKw0ST/9zrZs13ZCva5hDAoh+P46naX7nWRp5AfJ7qtmaXyDHOTIc/hu5C9HnjeZjuk/F/alMHPPSyHSgpeS2wgr8QxWdKy6ERyylqNBlEakbTFEpSA0LY1pjdu22xpl+60pi6AnrEIm++qd4CYu1bk3a/RvIKK4erM7tfNLWOVsZ5LFjOP2heRfjvxAI5ryKdkFIPPcVXkp+10daa0HpkGZIR3wfuuN9yAfgnsuVi5+973Z2aSM8sTkxk74VwZ6ydCO2eyrRGRbkUimyFqsUZmVbXHJZlJEip5DGgloEp5AG0E5vLANBlNSw1hIfs32uMnHmxkWPlbWIkYtnikskzYmNWtjy8GbtjxWeU+SHVI1ZORzWyXB7Ug3bNdyvGaZsUKIUrI0Ew/1NBKmI6EWaAwm1slmh6YDu6Npd2uYtAgAEzB6Ty62w9Tit/Vf29ummtxKi1uV5NkqthUc1aEbyPRgOf8B82mPeSaSVEJQLiGB6oY9EsduBVAOTEFG4yM5SyTUS4VlSQgpO1JVFRLW5mhNZCkLhYquC5/luFF+2EzkElLftSzWTiw4B5XzRMMMK+HlpFIUyhSZmizoRwkNnC+T0YQGHpDtNjb9cndtaL40FyYKjkCVsYW8OoZBP76a+pVQox9GvvJ7d/CzHvgeYykKdiXjasNz3wQ4IVQSGqI/c+DLM7dH3e9T9w8QluwB5C1mIq9Q0nTp2yH/Gtx29QoVgxanhD7xhkGfn4j5vMeccf7p88fifmfOQ7f1ky3u8x7oCEwOml7ou83H7IP5uzGf/+sNvTokv4C+GvQpP3KYc/76lHagbP7Xg9vYefPJzTkV7wFfcFD8Ir/3kIVpsbJIuRKyQkaWBUYHTKJjkqpF40Zx4K0MAu9dj17N36H8aJDMiKd2rEP5seFu89XfsKgvUZaF73VSnOFGh2B7vbwEdQJ1ofYCUUHTvtkLVOCq53iIIJnNXa50by7E/uVv&lt;/diagram&gt;&lt;/mxfile&gt;&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/js/viewer-static.min.js"></script>
</body>
</html>

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

<!--[if IE]><meta http-equiv="X-UA-Compatible" content="IE=5,IE=9" ><![endif]-->
<!DOCTYPE html>
<html>
<head>
<title>interface</title>
<meta charset="utf-8"/>
</head>
<body><div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers tags lightbox&quot;,&quot;edit&quot;:&quot;_blank&quot;,&quot;xml&quot;:&quot;&lt;mxfile host=\&quot;app.diagrams.net\&quot; modified=\&quot;2022-09-12T10:40:47.532Z\&quot; agent=\&quot;5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36\&quot; etag=\&quot;fLOVGC3rNYdv49x5v-pu\&quot; version=\&quot;20.3.0\&quot; type=\&quot;device\&quot;&gt;&lt;diagram id=\&quot;bFoD5zHsZeTquyHUPzh6\&quot; name=\&quot;페이지-1\&quot;&gt;7Vltb5swEP41kbYPq3gJlHxsw16ktVXVrNr2afLABasGM+M0yX79zmCHgJOGtmTrpEhRxD2cL/Y995gzGbnTbPmRoyK9ZDGmI8eKlyM3HDmO54/hWwKrGji1JzWQcBLXkN0AM/IbK9BS6JzEuGw5CsaoIEUbjFie40i0MMQ5W7Td7hht/2qBEmwAswhRE/1KYpHWaOCcNvgnTJJU/7Ltq/VlSDurlZQpitliA3Lfj9wpZ0zUV9lyiqnMnc5LPe7DjrvriXGciz4Dbr1JPE4+/xI/Tv3w5jZ5uLqYvVNRHhCdqwXflpiHiKk5i5VORLkgGUU5WOd3LBczdccGG1GS5HAdwUwwB+ABc0Egh2fqhmAFoFFKaHyBVmwu51sKFN1r6zxlnPyGsIiqmHCbC1UOjt/ymMmRAFuAclyCz7VOgt2BLtGy5XiBSqGAiFGKipL8XC8jQzwh+TkTgmXKSWUHloOXO9Nur8kEEWCWYcFX4KIG+Ip+Vf9re9FUk62xtFVJjqpiVcHJOnRDMlwonp/AuWNwXpKsoHhaS4iw/BLdA4/dCoAciIoZzu7BmTKgOsxZXRKE0g6kq4LiO7GzJsoCRSRPLiqfcNwgNyoTEmIw9o5W2klJHONc8skEEqgmTzJVMJKLKlPeOXwgoVPrxBt5MPEp2HZjw0e6cwHrhbUgUvGIoTIWWFZHP9J3q8mshDX1/ZjXfoMT7xrEGxxTUnFXc6w3PPtZBGdAFcUNo18k4eE722DdNVl3tzBM0U9Mr1lJZIUCxmvfDvP7yG1XL8lTzMkhSfecfqQHB+J8bHCO4vjN26O4B+a577Z+MHF7BtEJFpJo+EHXbh5mR84H4zz41xu6b3C+7xm+o4uzXtyebamX/7VjG1vPbtmsAxEdGERn6H6jYQOdH/X9DH37gzRswfhAtJ/ub9hwHp/JYy5YrKgSDcgHQrVQwVIyhfMEaFJk+g5n8zzGcUs3ODZOw3tVsy07HFMkyEM71rb0qHDXshgaBa7PzIoI1+kIq2RzHmE1avPM2wnUlbIRCLYxeEwagSq21mt8PoETg8Cr4w798h3aCdq0jt1/vkPrwMcteuAtevK6t2jbfIMWHiX+col7zuuTuPni7CjxQU5Zw7w3O5zGn9aHRRSVJYke67YgH3z1TRonnuVr4HsFTIJAA+Fy0z9cbVrXmBNYnnzfXoNLIlREZX3fuNNEkoYO1Lfpg5VWDdf+Tqfupx7xC/Y2kd4War1h+krX37GpPLmvtDt9ZdCvr4QSQasNNyW//hO2rMfn1W2M2v5wUc9g0CbXMVufx+ShH0x9tNFSxh5V9C/mvUW6a0P6O1W6i8QnV6nXqR7XO0yV+tagVQdm869o7d78tey+/wM=&lt;/diagram&gt;&lt;/mxfile&gt;&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/js/viewer-static.min.js"></script>
</body>
</html>

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

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers tags lightbox&quot;,&quot;edit&quot;:&quot;_blank&quot;,&quot;xml&quot;:&quot;&lt;mxfile host=\&quot;app.diagrams.net\&quot; modified=\&quot;2022-09-12T11:49:18.994Z\&quot; agent=\&quot;5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36\&quot; etag=\&quot;1YUgR8da9pSV2tdp5Cj9\&quot; version=\&quot;20.3.0\&quot; type=\&quot;device\&quot;&gt;&lt;diagram id=\&quot;dIaMDhr30mNbuuR2birH\&quot; name=\&quot;페이지-1\&quot;&gt;7VhNc9owEP01PiZjWTbGR2yS9pDMZIa2NEcFKbamssXIIpj8+kpY/iYDpcHJIRdGu9pdafc9HjYWjNLim0Dr5J5jwizHxoUF55bjeBNXfWrHrnT4ICgdsaC4dIHGsaCvxDht491QTPJOoOScSbruOlc8y8hKdnxICL7thj1z1j11jWIycCxWiA29S4plUnqnjt/4vxMaJ9XJYGL6S1EVbDrJE4T5tuWCNxaMBOeyXKVFRJieXTWXMu/2jd36YoJk8pSE9TT89fC6jAq8XPrb+EpCP78CsCzzgtjGdPwzJ2KO+A+SS3NxuaumkW9pylCmrPCZZ3JhdmxlrxLK8B3a8Y2+TS7R6k9lhQkX9FXFI6a2gHKobSEN2M5EV6OMRZxxoRwZ3x/QJC10MXOMILlKe6i6Bj3XPSo6gXcol9UFOWNondOn/ZV1YopETLOQS8lTE2SmQYQkxZtzBjV6ivWEp0SKnQoxCRODtyE8tI29begDXONLWtTxHMNaw9i4rtyAqhYG13/B2P/C+J0x/oQgT79AfmeQXf/TgRx8gXxhkN0PB7nqoQXyAFmS4Zl+uFHWE+N63CFGeUKwmZXav6X62P2YlGXQA46GSqYVsoJvMrzPKuPU/X9r49qrzMe6ojLmRTtyvqusgspWmrIeWztNkjZ2HegIHjx99YBTbfONWJETvhWKpTGRRzVySIU21JWwC8KQpC/d2x3C2pR74FTdu6YVDHraMbW7JcquTFb7qa1XyPOOFCq7HhTaU6/u8T/Y6A7ZeBNZU8eaRXoRzKyZmv+EaeV4EmoVyxrew4zla5INWdnh75sUPZU2o4Ds2D1s3DNBHjxO9AtdGuTJYZChFcz3C9cKgT5VrWe2FSjkZ1YIrdAbD/lGnNrSZF873onqBNraZNJGkSd4qjz5ozEXOL1fvbOZOz1S6NLMHb7bfKQ8HebaeFQbj0EucLuS5Xiqz7M45PpHS12aRcOXp0P6N66wHdO103lzXHmCD1Me6J2pPLAvYeC9lEeZzb9yZXjz1ya8+Qs=&lt;/diagram&gt;&lt;/mxfile&gt;&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/js/viewer-static.min.js"></script>

\- 위와 같은 구조로, UserDao와 ConnectionMaker클래스를 분리하고, 서로 영향을 주지 않으면서 필요에 따라 확장할 수 있는 구조로 만들었음

\- 이제까지의 리팩터링 작업들은 다음의 객체지향 원칙을 따르며 개선한 작업임

>  **개방 폐쇄 원칙(OCP, Open-Closed Principle)  : 클래스나 모듈은 확장에는 열려있어야 하며, 변경에는 닫혀있어야 한다.** 

> **높은 응집도 : 하나의 모둘, 클래스가 하나의 책임 또는 관심사에만 집중되어있다는 것.**  
>   
> 예컨대 ConnectionMaker를 예로 들 수 있음. 인터페이스를 생성하여 DB연결기능을 독립시켜서, DB관련 기능을 추가하려는 경우 해당 인터페이스 하위 클래스만 수정 또는 추가해주면 되므로, 다른 관심사에 속하는 클래스를 굳이 수정해줄 필요가 없음. 즉 이와 같은 사례는 높은 응집도를 가졌다고 할 수 있다.

> **낮은 결합도: 책임과 관심사가 다른 오브젝트 또는 모듈과는 낮은 결합도, 즉 느슨하게 연결된 형태를 유지하여,연결에 필요한 최소한의 방법만 간접적인 형태로 제공하는 것. 하나의 오브젝트가 변경이 일어날 때에 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도.** 가령 UserDao와 ConnectionMaker의 관계가 그렇다. UserDao는 구체적인 ConnectionMaker의 구현 클래스를 알 필요도 없음. 왜냐하면 ConnectionMaker라는 인터페이스를 통해 낮은 결합도로 최소한으로 연결되어있기 때문. 이 낮은 결합도를 통해 UserDao나 기타 Dao에 영향을 주지 않음

> **전략 패턴: 자신의 기능맥락에서 필요에 따라 변경이 필요한 알고리즘 클래스를 필요에 따라 바꿔 사용할 수 있기 하는 디자인 패턴. 이때 알고리즘이란, 독립적인 책임으로 분리가 가능한 기능을 뜻함.** UserDao가 전략 패턴 컨텍스트에 해당함. DB연결방식이라는 컨텍스트를 ConnectionMaker라는 인터페이스로 정의하고, 이를 구현한 클래스, 즉 전략을 바꾸면서 사용했기에 전략패턴과 관련이 있다고 볼 수 있음.
