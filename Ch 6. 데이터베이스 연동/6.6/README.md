# 데이터베이스 연동
### 1. 프로젝트 생성
* project build : maven
* Language : java
* Spring Boot : 2.7.9에서 2.5.6로 버전 수정 (pom.xml)
```xml 
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.5.6</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
```
* Dependencies
  1. Developer Tools: Lombok, Spring Configuration Processor
  2. Web: Spring Web
  3. SQL: Spring Data JPA, MariaDB Driver
* swagger 의존성 추가 (pom.xml)
```xml
		<!--Swagger 의존성 추가-->
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger2</artifactId>
			<version>2.9.2</version>
		</dependency>

		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger-ui</artifactId>
			<version>2.9.2</version>
		</dependency>
```
* SwaggerConfiguration 파일 수정
```java
package com.springboot.jpa.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfiguration {

  @Bean
  public Docket api() {
    return new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(apiInfo())
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.springboot.jpa"))
            .paths(PathSelectors.any())
            .build();
  }

  private ApiInfo apiInfo() {
    return new ApiInfoBuilder()
            .title("Spring Boot Open API Test with Swagger")
            .description("설명 부분")
            .version("1.0.0")
            .build();
  }
}

```

### 2. 데이터베이스 관련 설정 추가  ( application.properties)
```properties
spring.datasource.driverClassName=org.mariadb.jdbc.Driver
spring.datasource.url=jdbc:mariadb://{DB IP주소}:3306/{데이터베이스이름}
spring.datasource.username={DB 사용자명: root}
spring.datasource.password={비밀번호}

spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```
* line1 : mariaDB 드라이버 정의
* line2 : 마리아 DB의 경로임을 명시하고 경로와 데이터베이스명 입력
* line3, line 4 : 데이터베이스 계정 정보 기입 ( 일반적으로는 보안상 암호화해서 사용 )
* line6 : ddl-auto( DDL 자동 생성 ) 전략
  1. create : 애플리케이션이 가동되고 SessionFactory가 실행될 때 **기존 테이블을 지우고 새로 생성**
  2. create-drop : create와 동일한 기능을 수행하나 **애플리케이션을 종료하는 시점에 테이블을 지움**
  3. update : SessionFactory가 실행될 때 **객체를 검사해서 변경된 스키마를 갱신함** ( 기존에 저장된 데이터는 유지됨 )
  4. validate : update처럼 객체를 검사하지만 **스키마는 건드리지 않음** (검사 과정에서 데이터베이스의 테이블 정보와 객체 정보가 다르면 에러 발생)
  5. none : ddl-auto 기능을 사용하지 않음
* line7 : show-sql 설정시 로그에 하이버네이트가 생성한 쿼리문 출력
* line8 : 7번에서 출력되는 쿼리문을 보기 좋게 포매팅

<br> 
* ddl-auto <br>
운영환경 : 대체로 validate, none 사용 <br>
개발환경 : 대체로 create, update 사용<br>
<br>
