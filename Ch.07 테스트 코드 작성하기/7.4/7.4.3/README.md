# 7.4.3 스프링 부트의 테스트 설정
***

### 의존성 추가

```xml
<!-- spring-boot-starter-test 의존성-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

**spring-boot-starter-test 라이브러리에서 제공하는 라이브러리**
* JUnit 5: 자바 애플리케이션의 단위 테스트 지원
* Spring Test & Spring Boot Test: 스프링 부트 애플리케이션에 대한 유틸리와 통합 테스트 지원
* AssertJ: 다양한 단정문(assert)을 지원하는 라이브러리
* Hamcrest: Matcher를 자원하는 라이브러리
* Mockito: 자바 Mock 객체를 지원하는 라이브러리
* JSONassert: JSON용 단정문 라이브러리
* JsonPath: JSON용 XPath를 지원

