![header](https://capsule-render.vercel.app/api?type=wave&color=C3E5AE&height=200&section=header&text=Spring&nbsp;Boot&nbsp;Study&fontSize=50&fontColor=000000)

## 7.5 JaCoCo를 활용한 테스트 커버리지 확인 

> 코드 커버리지(code coverage)  

소프트웨어의 **테스트 수준이 충분한지를 표현하는 지표** 중 하나  
##
>JaCoCo 란?
* 커버리지 도구 중 가장 보편적으로 사용되는 도구, **Java Code Coverage 의 약자.**
* JUnit 테스트를 통해 어플리케이션의 코드가 얼마나 테스트됐는지 Line 과 Branch를 기준으로 한 커버리지로 리포트
* **런타임으로 테스트 케이스를 실행하고 커버리지를 체크하는 방식으로 동작**
* 리포트는 HTML, XML, CSV 같은 다양한 형식으로 확인 가능
##
### 7.5.1 JaCoCo 플러그인 설정
* JaCoCo의 플러그인 설정은 pom.xml 파일에서 **의존성 추가** 

```
  <dependency>
      <groupId>org.jacoco</groupId>
      <artifactId>jacoco-maven-plugin</artifactId>
      <version>0.8.7</version>
  </dependency>
```

* pom.xml 파일의 <build> 태그 안에 JaCoCo 플러그인 설정 추가
 ```
        <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.7</version>
                <configuration>
                    <excludes>
                        <exclude>**/ProductServiceImpl.class</exclude>
                    </excludes>
                </configuration>
                
                <executions>
                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>jacoco-report</id>
                        <phase>test</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>jacoco-check</id>
                        <goals>
                            <goal>check</goal>
                        </goals>
                        <!-- 예제 7.23 -->
                        <configuration>
                            <rules>
                                <rule>
                                    <element>BUNDLE</element>
                                    <limits>
                                        <limit>
                                            <counter>INSTRUCTION</counter>
                                            <value>COVEREDRATIO</value>
                                            <minimum>0.80</minimum>
                                        </limit>
                                    </limits>
                                    <element>METHOD</element>
                                    <limits>
                                        <limit>
                                            <counter>LINE</counter>
                                            <value>TOTALCOUNT</value>
                                            <maximum>50</maximum>
                                        </limit>
                                    </limits>
                                </rule>
                            </rules>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
 
 ```
 ##
 #### 1. configuration 태그
  ```
  <configuration>
      <excludes>
         <exclude>**/ProductServiceImpl.class</exclude>
      </excludes>
  </configuration>
                
  ```
  * 일부 클래스를 **커버리지 측정 대상에서 제외**
  - 해당 예제에서는 ProductServiceImpl.class 를 커버리지 측정 대상에서 제외하도록 설정   
  
  #### 2. executions 설정
  
  ```
          <executions>
                    <execution>
                        <goals>
                            <goal>prepare-agent</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>jacoco-report</id>
                        <phase>test</phase>
                        <goals>
                            <goal>report</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>jacoco-check</id>
                        <goals>
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
  
  ```

  * <execution> 은 기본적으로 <goal> 을 포함하며, 설정한 값에 따라 추가 설정이 필요한 내용을 <configuration> 과 <rule>을 통해 작성
  > **goal의 속성 값**
  
  * help: jacoco-maven-plugin에 대한 도움말을 보여줌.
  * prepare-agent: 테스트 중인 애플리케이션에 VM 인수를 전달하는 JaCoCo 런타임 에이전트의 속성을 준비, 에이전트는 maven-surefire-plugin 을 통해 테스트한 결과를 가져오는 역할 수행
  * prepare-agent-integration: prepare-agent와 유사하지만 통합 테스트에 적합한 기본값을 제공
  * merge: 실행 데이터 파일 세트(.exec)를 단일 파일로 병합
  * report: 단일 프로젝트 테스트를 마치면 생성되는 코드 검사 보고서를 다양한 형식(HTML, XML, CSV) 중에서 선택할 수 있게 함
  * report-integration: report 와 유사하나 통합 테스트에 적합한 기본값을 제공
  * report-aggregate: Reactor 내의 여러 프로젝트에서 구조화된 보고서(HTML, XML, CSV)를 생성. 보고서는 해당 프로젝트가 의존하는 모듈에서 생성됨.
  * check: 코드 커버리지의 매트릭 충족 여부를 검사함. **메트릭은 테스트 커버리지를 측정하는데 필요한 지표를 의미.** 메트릭은 check 가 설정된 <execution> 태그 내에서 <rule>을 통해 설정
  * dump: TCP 서버 모드에서 실행 중인 JaCoCo 에이전트에서 TCP/IP 를 통한 덤프를 생성
  * instrument: **오프라인 측정을 수행하는 명령** 테스트를 실행한 후 restore-instrumented-classes Goal로 원본 클래스 파일들을 저장해야 함.
  * restore-instrumented-class: 오프라인 측정 전 원본 파일을 저장하는 기능을 수행
  
  #### 3. Rule
  - JaCoCo 에서 설정할 수 있는 Rule. **configuration 태그 안에 설정** 하며, 다양한 속성을 활용할 수 있음.
  :mag: Element : 코드 커버리지를 체크하는 데 필요한 범위 기준을 설정
  * BUNDLE(기본값): 패키지 번들(프로젝트 내 모든 파일)
  * PACKAGE: 패키지
  * CLASS : 클래스
  * GROUP : 논리적 번들 그룹
  * SOURCEFILE: 소스 파일
  * METHOD: 메서드  
  
  
  :bulb: 값을 지정하지 않는 상태의 **기본값은 BUNDLE** , BUNDLE 은 Element를 기준으로 ```<limits>``` 태그 내 ```<counter>``` 와 ```<value>``` 를 활용해 커버리지 측정 단위와 방식을 설정  
  
  :mag: Counter: **커버리지를 측정하는 데 사용하는 지표**  
  
 :pushpin: 커버리지 측정 단위  
  * LINE: 빈줄을 제외한 실제 코드의 라인 수
  * BRANCH: 조건문 등의 분기 수
  * CLASS: 클래스 수
  * METHOD: 메서드 수
  * INSTRUCTION(기본값): 자바의 바이트코드 명령 수
  * COMPLECITY: 복잡도. 복잡도는 맥케이브 순환 복잡도 정의를 따름  
  
 :mag: Value: **커버리지 지표 설정**, 측정한 커버리지를 어떤 방식으로 보여주는지 설정
  * TOTALCOUNT: 전체 개수
  * MISSEDCOUNT: 커버되지 않는 개수
  * COVEREDCOUNT: 커버된 개수
  * MISSEDRATIO: 커버되지 않은 비율
  * COVEREDRATIO(기본값) : 커버된 비율  
  
   :bulb: value의 속성을 지정하지 않는 경우의 **기본값은 COVERDRATIO**
  
  ```
                        <configuration>
                            <rules>
                                <rule>
                                    <element>BUNDLE</element>
                                    <limits>
                                        <limit>
                                            <counter>INSTRUCTION</counter>
                                            <value>COVEREDRATIO</value>
                                            <minimum>0.80</minimum>
                                        </limit>
                                    </limits>
                                  <!-- 전체 라인 수를 최대 50줄로 설정 --!>
                                    <element>METHOD</element>
                                    <limits>
                                        <limit>
                                            <counter>LINE</counter>
                                            <value>TOTALCOUNT</value>
                                            <maximum>50</maximum>
                                        </limit>
                                    </limits>
                                </rule>
                            </rules>
                        </configuration>
  ```
  * counter : INSTRUCTION, 패키지 번들 단위로 바이트코드 명령 수를 기준
  * value : COVERDRATIO , 커버리지가 최소한 80% 달성하는 것을 limit 으로 설정
##
### 7.5.2 JaCoCo 테스트 커버리지 확인
![image](https://user-images.githubusercontent.com/78422940/227562268-eb500ae4-b9e1-4a8c-82e0-ac78d3a658ca.png)

#### :sparkler: 리포트 페이지 구성 칼럼
1. **Element**: 우측 테스트 커버리지를 측정한 단위를 표현. 링크를 따라 들어가면 세부 사항을 볼 수 있음.
2. **Missed Instructions - Cov.(Coverage)**: 테스트를 수행한 후 바이트코드의 커버리지를 퍼센티지 바 형식으로 제공
3. **Missed Branches - Cov.(Coverage)**: 분기에 대한 테스트 커버리지를 퍼센티지와 바 형식으로 제공
4. **Missed - Cxty(Complexity)**: 복잡도에 대한 커버리지 대상 개수와 커버되지 않는 수를 제공
5. **Missed - Lines**: 테스트 대상 라인 수와 커버되지 않는 라인수를 제공
6. **Missed- Methods**: 테스트 대상 메서드 수와 커버되지 않는 메서드 수를 제공
7. **Missed - Classes**: 테스트 대상 클래스 수와 커버되지 않은 메서드 수를 제공  

#### :file_folder: 코드 레벨에서의 테스트 커버리지 확인
![image](https://user-images.githubusercontent.com/78422940/227564719-3a4c9bc3-3fca-4631-942c-f0fc6668736f.png)

* 초록색은 **테스트에서 실행됐다**는 의미
* 빨간색은 테스트 코드에서 **실행되지 않는 라인**을 의미
* 노란색은 코드가 분기되는 지점이며, ture 와 false 에 대한 모든 케이스가 테스트됐다면 초록색으로, 둘 중 하나만 테스트됐다면 노란색으로 표시
