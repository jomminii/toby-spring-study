> Note : 해당 문서는 [WorkFlowy](https://workflowy.com/s/what-i-learned/u8g3pBjScYR1u2rO)에 최적화 되어있습니다.

# Chapter3 - Template

## What I learend?

### try/catch/finally 및 try-with-resource 이용한 예외처리로 안전한 리소스 반환 보장 코드 작성법

- 두 문법의 예외처리 차이점.
    - Situation : try block과 close()에서 동시 예외 발생
    - try/finally : close()에서 발생한 예외 반환
    - try-with : try block에서 발생한 예외 반환
  - close()에서의 명시적인 예외처리를 통해, 리소스 반환이 이루어지지 않고 메소드가 종료될 가능성 제거하기.
- 반복되는 작업 흐름중, 변하지 않는 부분은 context(template), 변하는 부분을 callback으로 만들어, callback을 분리 및 재사용하는 방법.

### Template method pattern vs Template callback pattern

#### Template method pattern
    - What?
      - 상속을 통한 기능 확장.
      - 변하지 않는 부분: SuperClass
      - 변하는 부분: SubClass에서 Override하여 정의
    - Cons?
      - 1. 매번 상속을 통해 새로운 클래스 생성이 필요해진다. 알다시피 자바는 다중상속이 안된다..
      - 2. 확장 구조가 클래스 설계 시점(컴파일 시점)에 고정되어 버리기에 유연성이 떨어진다.
#### Template callback pattern
    - What?
      - DI방식의 전략패턴
      - client는, template이 사용할 callback interface를 구현한 개체를 매번 새로만들어, template호출과 동시에 DI한다.
      - 이때 callback은, 일반적으로 단일 메소드 인터페이스를 이용한다.(특정 기능을 위해 한번 호출되는 경우가 일반적이기 때문에)
    - When?
      - 전략패턴으로 개선후, 바뀌는 부분(전략)이 여러종류가 만들어 질수 있을 때
    - Char?
      - template은, 매번 "메소드" 단위로 callback을 "새롭게" 전달 받는다
      - callback은, 자신을 생성한 client개체 메소드내의 정보를 캡처링한다 = client과 callback의 강하게 결합되어 있다.
    - Why?
      - template method pattern보다 유연하여 확장성이 뛰어나다 (OCP)
      - 런타임에도 유연하게 구현정보 변경 가능하다
    - How?
      - template: 
        - 반복되는 작업 흐름중, 변하지 않는 부분.
        - 한번에 하나 이상의 callback을 사용할수 있다.
        - 하나의 callback을 여러번 호출 할수 있다.
      - callback: 
        - 반복되는 작업 흐름중, 변하는 부분.
        - template내의 특정 흐름에서 로직을 실행할 목적으로 template에 전달된다.
        - callback에도 일정한 패턴이 반복된다면, callback도 template에 포함시켜 재활용한다.
        - 단일 전략 메소드를 갖는 전략패턴이며, 매번 전략을 새로 만들어 사용.
        - 일반적으론, 특정 메소드의 로직을 실행하는 것을 목적으로 다른 개체 메소드에 전달되는 개체를 말한다.
        - 자바에선, 메소드 자체를 파라미터로 전달할 방법이 없기에, 메소드가 담긴 개체를 전달한다.
      - client: 
        - Context(template)호출과 동시에, template안에서 사용될 callback을 DI한다
      - context
        - client에 사용되는 Context가 여러개라면, 클래스 분리를 통해 다양한 개체에서 공유되도록 한다
        - context는 Bean으로 등록해 사용하거나, client에서 직접 생성해서 사용한다
    - Key point : 
      - template과 callback사이에 각각 전달하는 내용이 무엇인지 파악하는 것이 가장 중요하다. 그에 따라 callback interface를 정의해야 하기 때문이다.
      - template과 callback은, interface를 통한 메시지 전달을 통해 OCP를 지킬 수 있도록 한다.
