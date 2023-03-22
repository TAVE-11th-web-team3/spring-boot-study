# 7.4.4 JUnit의 생명주기
***
### 테스트 순서에 관여하는 대표적인 어노테이션
* @Test: 테스트 코드를 포함한 메서드 정의 
* @BeforeAll: 테스트를 시작하기 전에 호출되는 메서드 정의
* @BeforeEach: 각 테스트 메서드가 실행되기 전에 동작하는 메서드 정의
* @AfterAll: 테스트를 종료하면서 호출되는 메서드 정의
* @AfterEach: 각 테스트 메서드가 종료되면서 호출되는 메서드 정의

***
### 어노테이션 동작 확인하기
TestLifeCycle 클래스 생성
```java
public class TestLifeCycle {

    @BeforeAll
    static void beforeAll(){
        System.out.println("## BeforeAll Annotation 호출 ##");
        System.out.println();
    }

    @AfterAll
    static void afterAll(){
        System.out.println("## AfterAll Annotation 호출 ##");
        System.out.println();
    }

    @BeforeEach
    void beforeEach(){
        System.out.println("## BeforeEach Annotation 호출 ##");
        System.out.println();
    }

    @AfterEach
    void afterEach(){
        System.out.println("## AfterEach Annotation 호출 ##");
        System.out.println();
    }

    @Test
    void test1(){
        System.out.println("## test1 시작 ##");
        System.out.println();
    }

    @Test
    @DisplayName("Test Case 2!!!")
    void test2(){
        System.out.println("## test2 시작 ##");
        System.out.println();
    }

    @Test
    @Disabled
    void test3(){
        System.out.println("## test3 시작 ##");
        System.out.println();
    }

}
```
**콘솔 로그 출력**

![image](https://user-images.githubusercontent.com/105872347/227004328-f2ae8a51-cdb0-4c1f-8a10-bc0fa73eb26d.png)
***
**결과 확인**
* **@BeforeAll**, **@AfterAll** : 전체 테스트 동작에서 처음과 마지막에만 각각 수행됨
* **@BeforeEach**, **@AfterEach** : 각 테스트가 실행될 때 @Test 어노테이션이 지정된 테스트 메서드를 기준으로 실행
* **@Disabled** : 지정된 테스트는 실행되지 않음, 그러나 테스트 메서드로는 인식되고 있어 메서드가 비활성화됐다는 로그는 출력됨 
( **@BeforeEach**와 **@AfterEach**도 실행되지 않음 )
    
