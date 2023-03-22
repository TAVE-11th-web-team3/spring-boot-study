# 7.4.1 JUnit의 세부 모듈
> **JUnit ?** 
> <br>
> JUnit은 자바 언어에서 사용되는 대표적인 테스트 프레임워크로 단위 테스트와 통합 테스트를 할 수 있는 기능을 제공합니다. 가장 큰 특징은 어노테이션
> 기반의 테스트 방식을 지원한다는 것입니다.
> <br>
***
### JUnit 5의 모듈 구성
**Junit Platform**  
* JVM에서 테스트를 시작하기 위한 뼈대 역할
* 테스트를 발견하고 테스트 계획을 생성하는 테스트 엔진(TestEngine) 인터페이스를 가짐
* 각종 IDE와의 연동을 보조하는 역할
* TestEngin API, Console Launcher, JUnit 4 Based Runner 등을 포함

**JUnit Jupiter**
* 테스트 엔진 API의 구현체를 포함
* JUnit 5에서 제공하는 Jupiter 기반의 테스트를 실행하기 위한 테스트 엔진을 가짐
* 테스트의 실체 구현체가 별도의 모듈 역할을 하며, 그 중 하나인 Jupiter Engine은 Jupiter API를 활용해서 작성한 테스트 코드를 발견하고 실행하는 역할을 수행

**JUnit Vintage**
* JUnit 3,4에 대한 테스트에 대한 엔진 API를 구현하고 있음
* 기존에 작성된 JUnit 3, 4 버전의 테스트 코드를 실행할 때 사용 ( 하위 호환성 ) 
* Vintage Engine 

