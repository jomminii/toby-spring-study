# 토프링 3장 템플릿
[[토프링 3장] 전략 패턴과 템플릿 콜백 패턴](https://velog.io/@jomminii/toby-spring-chapter03)

## 전략 패턴

전략 패턴은 context 속에 사용하고자 하는 전략을 interface 의 관계로 맺어 구체적인 구현체를 사용하는 방식을 말한다.

코드로 살펴보겠다.

움직이는 행위에 대한 클래스가 있다고 해보자.
여기서 움직이는 행위에 대한 전략(걷는다, 뛴다)에 따라 움직임이 변하도록 구현하려고 한다.

여기서는 useStrategy 를 context 라고 할 수 있다.
일련의 움직이기 전과 후의 과정이 있고 전략에 따라 움직임을 어떻게 할지 정하게 된다.


```java
public class Move {

    void useStrategy(Strategy strategy) {
        System.out.println("start");
        System.out.println("움직이기 전 스트레칭");
        strategy.executeStrategy();
        System.out.println("움직이고 난 후 스트레칭");
        System.out.println("end");
    }
}
```

<br>

여기서 사용한 `Strategy` 는 아래의 인터페이스를 구현한 구현체가 자리할 것이다.

```java
public interface Strategy {

    void executeStrategy();
}

```
인터페이스에서는 전략을 실행하려는 부분을 구현하도록 기준 메서드를 마련해 놓고 있다.

<br>

그리고 각각의 구현체들은 구체적인 실행 내용을 구현하고 있다.

```java
public class StrategyOne implements Strategy {

    @Override
    public void executeStrategy() {
        System.out.println("Strategy One - walking");
    }
}

public class StrategyTwo implements Strategy {

    @Override
    public void executeStrategy() {
        System.out.println("StrategyTwo - running");
    }
}


```

<br>



이 구현체들을 활용해 클라이언트에서는 원하는 전략을 적용해 움직임을 실행하게 된다.

```java
public class Client {
    public static void main(String[] args) {
        Strategy strategyOne = new StrategyOne();
        Move moveOne = new Move();
        moveOne.useStrategy(strategyOne);

        Strategy strategyTwo = new StrategyTwo();
        Move moveTwo = new Move();
        moveTwo.useStrategy(strategyTwo);
    }
}
```

<br>

실행한 결과는 아래와 같다.

```bash
start
움직이기 전 스트레칭
Strategy One - walking
움직이고 난 후 스트레칭
end
start
움직이기 전 스트레칭
StrategyTwo - running
움직이고 난 후 스트레칭
end

```


이처럼 전략 패턴은 클라이언트가 사용하고자 하는 전략(구현체)을 context에
주입함으로써 사용한다.

<br>



## 템플릿 콜백 패턴

템플릿 콜백 패턴은 전략 패턴이 좀 다르게 변형된 형태라고 볼 수 있다. 전략 패턴과는 다르게 context 가 해당 전략을 실행할 때 전략이 정해진다고 볼 수 있다.

context 라는 공통 로직 안에 콜백이라는 바뀌는 부분을 따로 정의하여 파라미터로 넘겨 적용 시킨다.

DB에서 데이터를 가져와 엑셀을 만들어주는 모듈이 있다고 해보자.

```java
public class ExcelModule {
    void makeExcel(GetDataStrategy strategy) {

        Integer valueOne = 3;
        Integer valueTwo = 4;

        System.out.println("엑셀 만들기 시작");
        System.out.println("데이터 가져오기");
        Integer data = strategy.getData(valueOne, valueTwo);
        System.out.println("data = " + data);
        System.out.println("가져온 데이터로 엑셀로 만들기");
        System.out.println("결과 출력");
    }
}

```

엑셀을 만들어주는 여러 로직은 공통으로 사용하는 context 이고, 데이터를 가져오는 `strategy.getData()` 부분만 어떻게 데이터를 가져오는 전략을 취할지에 따라 달라지게 된다.


가져오는 전략은 이렇게 간략하게 짠다고 생각해보자. 간단히 파라미터로 a와 b 를 받고 Integer 를 반환하도록 정의했다. (functional interface 라고 보면 된다.)
```java
public interface GetDataStrategy {

    Integer getData(Integer a, Integer b);
}


-->
원래라면 이런식으로 구현될 수 있을 것 같다.
public interface GetDataStrategy {
	List<Order> orderList(OrderCondition condition);
}
```


이 엑셀 모듈을 사용할 클라이언트는 아래처럼 구현했다.

클라이언트에서 엑셀 모듈을 사용한다고 할 때 각각의 데이터 조회 전략을 다르게 가져갈 수 있다.


```java
public class ExcelClient {

    public static void main(String[] args) {
        System.out.println("start");
        ExcelModule excelModule = new ExcelModule();

        GetDataStrategy strategyOne = (valueOne, valueTwo) -> valueOne + valueTwo + 5 + 4;
        
        GetDataStrategy strategyTwo = (valueOne, valueTwo) -> valueOne + valueTwo + 100 + 400;

		--> 실제라면 아래와 같은 식이 될 것 같다.
		GetDataStrategy strategyThree = () -> OrderRepository.getOrderList(OrderCondition condition);
        GetDataStrategy strategyFour = () -> OrderRepository.getOrderDetailList(OrderCondition condition);


        excelModule.makeExcel(strategyOne);
        excelModule.makeExcel(strategyTwo);
        System.out.println("end");
    }
}

```

이 클라이언트를 돌려보면 각각의 전략에 맞게 엑셀 생성 로직이 작동하는 것을 볼 수 있다.

```bash
start
엑셀 만들기 시작
데이터 가져오기
data = 16
가져온 데이터로 엑셀로 만들기
결과 출력
엑셀 만들기 시작
데이터 가져오기
data = 507
가져온 데이터로 엑셀로 만들기
결과 출력
end
```

위와 같이 템플릿/콜백 패턴을 사용하게 되면 공통된 로직은 공통으로 사용하면서, 다르게 적용하고 싶은 부분만 따로 작성하여 파라미터로 넘겨주어 쓰면 된다.

전략 패턴의 경우에는 미리 구체적인 구현체들을 만들어놔야하지만, 템플릿/콜백 패턴은 사용할 때만 직접 작성해주면 돼서 보다 편리한 부분도 있다.

하지만 익명 함수 형태(혹은 람다)로 작성해야하다보니 긴 로직은 작성하기가 어렵다.

참고로 전략 설정에 사용했던 람다는 아래의 익명함수에서 비롯된 친구이다.
```java
GetDataStrategy strategyOne = new GetDataStrategy() {
    @Override
    public Integer getData(Integer a, Integer b) {
        return a + b + 5 + 4;
    }
};

->
GetDataStrategy strategyOne = (valueOne, valueTwo) -> valueOne + valueTwo + 5 + 4;
```

함수형 인터페이스를 바로 구현해서 Override 한 형태인데, 이렇게 작성하면 IDE 가 람다로 바꾸라고 알려준다.

아직은 이 형태가 익숙하지 않아서 IDE 기능으로 람다<->익명함수를 돌려보면서 익숙해지려고 하고 있다.

