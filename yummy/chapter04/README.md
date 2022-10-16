# 예외


## 예외의 종류

1. Error
- java.lang.Error클래스의 서브 클래스들
- 에러는 시스템에 뭔가 비정상적인 상황이 발생했을 경우 사용된다.
- ex) OutOfMemoryError, ThreadDeath 에러

2. Exception과 체크 예외
- CheckedException과 UncheckedException으로 나뉨
- UncheckedException은 RuntimeException을 상속하는 예외
- CheckedException은 반드시 예외를 처리하는 코드를 함께 작성해야 함
- 
