## 10.3 스프링 부트에서의 유효성 검사
___
![image](https://user-images.githubusercontent.com/105872347/231727305-75918989-2475-4405-9aab-ee13c15c1881.png)
**유효성 검사**는 각 **계층**으로 **데이터가 넘어오는 시점에 해당 데이터에 대한 검사**를 실시합니다.
<br>
**스프링 부트 프로젝트**에서는 계층 간 데이터 전송에 대체로 **DTO 객체**를 활용하고 있기 때문에 **유효성 검사**를 **DTO 객체**를 대상으로 수행하는 것이 
일반적입니다.

### 1. ValidRequestDto 클래스 생성
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class ValidRequestDto {

    @NotBlank
    String name;

    @Email
    String email;

    @Pattern(regexp = "01(?:0|1|[6-9])[.-]?(\\d{3}|\\d{4})[.-]?(\\d{4})$")
    String phoneNumber;

    @Min(value = 20) @Max(value = 40)
    int age;

    @Size(min = 0, max = 40)
    String description;

    @Positive
    int count;

    @AssertTrue
    boolean booleanCheck;
}
```
각 필드에 어노테이션이 선언된 것을 볼 수 있습니다. 각 어노테이션은 유효성 검사를 위한 조건을 설정하는 데 사용됩니다.

**유효성 검사를 위한 대표적인 어노테이션**
1. **문자열 검증** 
   * **@Null** : null 값만 허용합니다.
   * **@NotNull** : null을 허용하지 않습니다. "", " "는 허용합니다.
   * **@NotEmpty** : null, ""을 허용하지 않습니다. " "는 허용합니다.
   * **@NotBlank** : null, "", " "을 허용하지 않습니다.


2. **최댓값/최솟값 검증**
    * BigDecimal, BigInteger, int, long 등의 타입을 지원합니다.
    * **@DecimalMax(value = "$numberString")** : $numberString보다 **작은 값**을 허용합니다.
    * **@DecimalMin(value = "$numberString")** : $numberString보다 **큰 값**을 허용합니다.
    * **@Min(value=$number)** : $number **이상**의 값을 허용합니다.
    * **@Max(value=$number)** : $number **이하**의 값을 허용합니다.


3. **값의 범위 검증**
    * BigDecimal, BigInteger, int, long 등의 타입을 지원합니다.
    * **@Positive** : **양수**를 허용합니다.
    * **@PositiveOrZero** : **0를 포함**한 **양수**를 허용합니다.
    * **@Negative** : **음수**를 허용합니다.
    * **@NegativeOrZero** : **0을 포함**한 **음수**를 허용합니다.

4.  **시간에 대한 검증**
    * Date, LocalDate, LocalDateTime 등의 타입을 지원합니다.
    * **@Future** : 현재보다 **미래**의 날짜를 허용합니다.
    * **@FutureOrPresent** : 현재를 **포함**한 **미래**의 날짜를 허용합니다.
    * **@Past** : 현재보다 **과거**의 날짜를 허용합니다.
    * **@PastOrPresent** : 현재를 **포함**한 **과거**의 날짜를 허용합니다.


5. **이메일 검증**
   * **@Email** : **이메일 형식**을 검사합니다. ""는 허용합니다.


6. **자릿수 범위 검증**
   * BigDecimal, BigInteger, int ,long 등의 타입을 지원합니다.
   * **@Digits(integer = $number1, fraction = $number2)** : $number1의 정수 자릿수와 $number2의 소수 자릿수를 허용합니다.


7. **Boolean 검증**
    * **@AssertTrue** : **true**인지 체크합니다. null 값은 체크하지 않습니다.
    * **@AssertFalse** : **false**인지 체크합니다. null 값은 체크하지 않습니다.


8. **문자열 길이 검증**
    * **@Size(min = $number1, max = $number2)** : $number1 **이상** $number2 **이하**의 범위를 허용합니다.


9. **정규식 검증**
    * **@Pattern(regexp = "$expression)** : 정규식을 검사합니다. 정규식은 자바의 java.util.regex.Pattern 패키지의 컨벤션을 따릅니다.

___

### 2. ValidationController 생성
```java
@RestController
@RequestMapping("/validation")
public class ValidationController {

    private final Logger LOGGER = LoggerFactory.getLogger(ValidationController.class);

    @PostMapping("/valid")
    public ResponseEntity<String> checkValidationByValid(
            @Valid @RequestBody ValidRequestDto validRequestDto) {
        LOGGER.info(validRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validRequestDto.toString());
    }
}
```

**동작 확인**
![image](https://user-images.githubusercontent.com/105872347/231737786-d29d9bf5-ea10-491d-801d-9adab7338218.png)
설정한 값들이 유효성 검사를 통과할 수 있는 값들이므로 '**200 OK**'로 응답합니다.

**유효성 설정 규칙을 벗어나는 경우 ( age를 -1로 설정하는 경우 결과 )**
 - 응답 : **"400 Bad Request"** (400 에러 발생)
```text
[Field error in object 'validRequestDto' on field 'age': rejected value [-1]; codes [Min.validRequestDto.age,Min.age,Min.int,Min]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validRequestDto.age,age]; arguments []; default message [age],20]; default message [20 이상이어야 합니다]] ]
```

**2개 이상의 유효성 검사를 통과하지 못하는 경우 ( age = -1, count = 0 )**
```text
with 2 errors: [Field error in object 'validRequestDto' on field 'age': rejected value [-1]; codes [Min.validRequestDto.age,Min.age,Min.int,Min]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validRequestDto.age,age]; arguments []; default message [age],20]; default message [20 이상이어야 합니다]] [Field error in object 'validRequestDto' on field 'count': rejected value [0]; codes [Positive.validRequestDto.count,Positive.count,Positive.int,Positive]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validRequestDto.count,count]; arguments []; default message [count]]; default message [0보다 커야 합니다]] ]
```
___

### 3. @Validated 활용
>**@Valid**어노테이션은 자바에서 지원하는 어노테이션이며, **@Validated**어노테이션은 스프링에서 지원하는 유효성 검사 어노테이션입니다. 
<br> **@Validated**은 **@Valid**의 기능을 포함하고 있기 때문에 **@Validated**로 변경할 수 있으며, 유효성 검사를 **그룹으로 묶어 대상을 
> 특정**할 수 있는 기능이 있습니다.


### **검증 그룹 사용하기**
- 검증 그룹은 별다른 내용이 없는 **마커 인터페이스**를 생성해서 사용합니다.

**ValidationGroup1 인터페이스 생성**
```java
public interface ValidationGroup1 {
}
```
**ValidationGroup2 인터페이스 생성**
```java
public interface ValidationGroup2 {
}
```


**DTO 클래스에 검증 그룹 설정** - ValidatedRequestDto 클래스 생성
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class ValidatedRequestDto {

    @NotBlank
    String name;

    @Email
    String email;

    @Pattern(regexp = "01(?:0|1|[6-9])[.-]?(\\d{3}|\\d{4})[.-]?(\\d{4})$")
    String phoneNumber;

    @Min(value = 20, groups = ValidationGroup1.class)
    @Max(value = 40, groups = ValidationGroup1.class)
    int age;


    @Size(min = 0, max = 40)
    String description;

    @Positive(groups = ValidationGroup2.class)
    int count;

    @AssertTrue
    boolean booleanCheck;

}
```
   * **@Min**, **@Max** 어노테이션의 **groups** 속성을 사용해 **ValidationGroup1** 그룹 설정
   * **@Positive** 어노테이션의 **groups** 속성을 사용해 **ValidationGroup2** 그룹 설정

**ValidationController 클래스 수정**
```java
@RestController
@RequestMapping("/validation")
public class ValidationController {

    private final Logger LOGGER = LoggerFactory.getLogger(ValidationController.class);

    @PostMapping("/valid")
    public ResponseEntity<String> checkValidationByValid(
            @Valid @RequestBody ValidRequestDto validRequestDto) {
        LOGGER.info(validRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validRequestDto.toString());
    }


    @PostMapping("/validated")
    public ResponseEntity<String> checkValidation(
            @Validated @RequestBody ValidatedRequestDto validatedRequestDto) {
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }

    @PostMapping("/validated/group1")
    public ResponseEntity<String> checkValidation1(
            @Validated(ValidationGroup1.class) @RequestBody ValidatedRequestDto validatedRequestDto) {
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }

    @PostMapping("/validated/group2")
    public ResponseEntity<String> checkValidation2(
            @Validated(ValidationGroup2.class) @RequestBody ValidatedRequestDto validatedRequestDto) {
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }


    @PostMapping("/validated/all-group")
    public ResponseEntity<String> checkValidation3(
            @Validated({ValidationGroup1.class, ValidationGroup2.class}) @RequestBody ValidatedRequestDto validatedRequestDto) {
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }


}
```
   * **checkValidation()** : group을 지정하지 않았습니다.
   * **checkValidation1()** : **ValidationGroup1**을 그룹으로 지정했습니다.
   * **checkValidation2()** : **ValidationGroup2**을 그룹으로 지정했습니다.
   * **checkValidation3()** : **ValidationGroup1**, **ValidationGroup2**을 그룹으로 지정했습니다.

**유효성 검사 확인**

```text 
{
    "age": -1,
    "booleanCheck" : true,
    "count" : -1,
    "description" : "Validation 실습 데이터입니다.",
    "email" : "flature@wikibooks.co.kr",
    "name" : "Falture",
    "phoneNumber" : "010-1234-5678"
}
```
- 호출할 때 전달하는 데이터는 위와 같습니다.
- age와 count 변수에 대한 유효성 검사를 통과하지 못하는 데이터입니다.

1. **@Validated 어노테이션에 특정 그룹을 지정하지 않은 경우** : checkValidation()
   * 정상 통과
   * @Validated 어노테이션에 **특정 그룹을 지정하지 않는 경우** **groups 속성을 설정하지 않은** 필드에 대해서만 유효성 검사를 실시합니다.
2. **@Validated 어노테이션에 ValidationGroup1을 그룹으로 지정한 경우** : checkValidation1()
   * 검사 오류가 발생할 수 있는 두 변수 중에서 **ValidationGroup1**을 그룹으로 설정한 **age**에 대한 에러가 발생
3. **@Validated 어노테이션에 ValidationGroup2을 그룹으로 지정한 경우** : checkValidation2()
   * **ValidationGroup2**을 그룹으로 설정한 **count**에 대한 에러만 발생
   
**유효성 검사 확인 데이터 변경**
```text
{
    "age": 30,
    "booleanCheck" : false,
    "count" : 30,
    "description" : "Validation ì¤ìµ ë°ì´í°ìëë¤.",
    "email" : "flature@wikibooks.co.kr",
    "name" : "Falture",
    "phoneNumber" : "010-1234-5678"
}
```
   * 위 데이터는 age와 count는 검사를 통과하고 booleanCheck 변수에서 검사를 실패하는 데이터입니다.
   * **CheckValidation3()** 호출 결과 정상적으로 응답
   
**@Validated 정리**
* @Validated 어노테이션에 **특정 그룹을 설정하지 않은 경우**에는 **groups가 설정되지 않은 필드**에 대해 유효성 검사 수행
* @Validated 어노테이션에 **특정 그룹을 설정하는 경우**에는 **지정된 그룹으로 설정된 필드**에 대해서만 유효성 검사를 수행

___
### 4. 커스텀 Validation 추가
- 실무에서는 유효성 검사를 실시할 때 자바 또는 스프링의 유효 검사 어노테이션에서 제공하지 않는 기능을 써야할 때도 있습니다.
- 이러한 경우 **ConstraintValidator**와 **커스텀 어노테이션**을 조합하여 별도의 유효성 검사 어노테이션을 생성할 수 있습니다.
- 동일한 정규식을 계속 쓰는 **@Pattern** 어노테이션의 경우가 가장 흔한 사례입니다.



#### @Telephone어노테이션 생성하기
**1.  **TelephoneValidator 클래스 생성****
   ```java
// ConstraintValidator 인터페이스를 구현하는 클래스 생성
// 어떤 어노테이션 인터페이스인지 타입 지정
public class TelephoneValidator implements ConstraintValidator<Telephone, String> {


   // 이 메서드를 구현하려면 직접 유효성 검사 로직을 작성해야 합니다.
   @Override
   public boolean isValid(String value, ConstraintValidatorContext context) {
       // null에 대한 허용 여부 로직 추가
      if (value == null) {
         return false;
      }
      // 지정한 정규식과 비교하는 로직 추가
      return value.matches("01(?:0:1:[6-9])[.-]?(\\d{3}|\\d{4})[.-]?(\\d{4})$");
   }
}
```
* 이 로직에서 false가 리턴되면 **MethodArgumentNotValidException** 예외가 발생합니다.

**2. **Telephone 어노테이션 인터페이스 생성****
```java
@Target(ElementType.FIELD)  // 필드에 선언 가능한 어노테이션
@Retention(RetentionPolicy.RUNTIME) // 컴파일 이후에도 JVM에 의해 계속 참조
@Constraint(validatedBy = TelephoneValidator.class) // TelephoneValidator와 매핑
public @interface Telephone {
    String message() default "전화번호 형식이 일치하지 않습니다.";
    Class[] groups() default {};
    Class[] payload() default {};

}
```
**@Target** : 어노테이션을 어디서 선언할 수 있는지 정의
   * ElementType.PACKAGE
   * ElementType.TYPE
   * ElementType.COSTRUCTOR
   * ElementType.FIELD
   * ElementType.METHOD
   * ElementType.ANNOTATION_TYPE
   * ElementType.LOCAL_VARIABLE
   * ElementType.PARAMETER
   * ElementType.TYPE_PARAMETER
   * ElementType.TYPE_USE

**@Retention** : 어노테이션이 실제로 적용되고 유지되는 범위
   * RetentionPolicy.RUNTIME : **컴파일 이후에도 JVM에 의해 계속 참조**합니다. 리플렉션이나 로깅에 많이 사용되는 정책입니다.
   * RetentionPolicy.CLASS : **컴파일러가 클래스를 참조할 때까지** 유지합니다.
   * RetentionPolicy.SOURCE : **컴파일 전까지**만 유지됩니다. 컴파일 이후에는 사라집니다.

**인터페이스 내부 요소**
   * **message()** : 유효성 검사가 실패할 경우 **반환되는 메시지**
   * **groups()** : 유효성 검사를 사용하는 **그룹으로 설정**
   * **payload()** : 사용자가 **추가 정보**를 위해 전달하는 값

**3. ValidationRequestDto 수정**
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class ValidatedRequestDto {

    @NotBlank
    String name;

    @Email
    String email;

    // @Pattern 어노테이션을 @Telephone 어노테이션으로 변경
    @Telephone
    String phoneNumber;

    @Min(value = 20, groups = ValidationGroup1.class)
    @Max(value = 40, groups = ValidationGroup1.class)
    int age;


    @Size(min = 0, max = 40)
    String description;

    @Positive(groups = ValidationGroup2.class)
    int count;

    @AssertTrue
    boolean booleanCheck;

}
```
**유효성 검사 확인**
```text
{
    "age": 30,
    "booleanCheck" : false,
    "count" : 30,
    "description" : "Validation 실습 데이터입니다.",
    "email" : "flature@wikibooks.co.kr",
    "name" : "Falture",
    "phoneNumber" : "12345678"
}
```

별도 그룹을 지정하지 않았기 때문에 checkValidation() 메서드를 호출했을 때 오류가 발생합니다.
```text
[Field error in object 'validatedRequestDto' on field 'phoneNumber': rejected value [12345678]; codes [Telephone.validatedRequestDto.phoneNumber,Telephone.phoneNumber,Telephone.java.lang.String,Telephone]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.phoneNumber,phoneNumber]; arguments []; default message [phoneNumber]]; default message [전화번호 형식이 일치하지 않습니다.]] ]
```