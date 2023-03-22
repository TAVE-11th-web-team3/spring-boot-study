# 컨트롤러 객체 테스트
### 컨트롤러의 역할
클라이언트로부터 요청을 받아 요청에 맞는 서비스 컴포넌트로 요청을 전달하고 그 결괏값을 가공하여 응답; 웹에 가장 가까이 있는 모듈
<br><br>
### 테스트 코드 작성
```java
@RestController
@RequestMapping("/product")
@RequiredArgsConstructor
public class ProductController {
    private final ProductServiceImpl productService;

    @GetMapping()
    public ResponseEntity<ProductResponseDto> getProduct(Long number) {
        ProductResponseDto productResponseDto = productService.getProduct(number);
        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }

    @PostMapping()
    public ResponseEntity<ProductResponseDto> createProduct(@RequestBody ProductDto productDto) {
        ProductResponseDto productResponseDto = productService.saveProduct(productDto);
        return ResponseEntity.status(HttpStatus.OK).contentType(MediaType.APPLICATION_JSON).body(productResponseDto);
    }

    @PutMapping()
    public ResponseEntity<ProductResponseDto> changeProductName(
            @RequestBody ChangeProductNameDto changeProductNameDto) throws Exception {
        ProductResponseDto productResponseDto = productService.changeProductName(
                changeProductNameDto.getNumber(),
                changeProductNameDto.getName());
        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }

    @DeleteMapping()
    public ResponseEntity<String> deleteProduct(Long number) throws Exception {
        productService.deleteProduct(number);
        return ResponseEntity.status(HttpStatus.OK).body("정상적으로 삭제되었습니다.");
    }
}
```
<br>
위와 같은 컨트롤러에 대해 테스트 코드를 작성한다고 가정했을 때, ProductController는 ProductService의 객체를 의존성 주입받고 있기 때문에 컨트롤러만 테스트하고 싶다면 서비스 객체는 **외부 요인**에 해당
<br>→ 독립적인 테스트 코드를 작성하기 위해서 Mock 객체 활용

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    ProductServiceImpl productService;

    @Test
    @DisplayName("MockMvc를 통한 Product 데이터 가져오기 테스트")
    void getProduct() throws Exception {
        given(productService.getProduct(1L)).willReturn(
                new ProductResponseDto(1L, "pen", 1000, 2000));

        String productId = "1";

        mockMvc.perform(
                    get("/product?number=" + productId))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.number").exists())
                .andExpect(jsonPath("$.name").exists())
                .andExpect(jsonPath("$.price").exists())
                .andExpect(jsonPath("$.stock").exists())
                .andDo(print());

        verify(productService).getProduct(1L);
    }

    @Test
    @DisplayName("Product 데이터 생성 테스트")
    void createProduct() throws Exception {
        given(productService.saveProduct(new ProductDto("pen", 5000, 2000)))
                .willReturn(new ProductResponseDto(12315L, "pen", 5000, 2000));

        ProductDto productDto = ProductDto.builder()
                .name("pen")
                .price(5000)
                .stock(2000)
                .build();

        Gson gson = new Gson(); //JSON 파싱 라이브러리로 자바 객체와 JSON 문자열 간 변환 지원
        String content = gson.toJson(productDto);

        mockMvc.perform(
                post("/product")
                        .content(content)
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.number").exists())
                .andExpect(jsonPath("$.name").exists())
                .andExpect(jsonPath("$.price").exists())
                .andExpect(jsonPath("$.stock").exists())
                .andDo(print());

        verify(productService).saveProduct(new ProductDto("pen", 5000, 2000));
    }
}
```

**@WebMvcTest**<br>
웹에서 사용되는 요청과 응답에 대한 테스트 수행(슬라이스 테스트)
옵션으로 대상 클래스를 지정하면 해당 클래스만 로드, 지정하지 않으면 컨트롤러 관련 빈 객체 모두 로드
<br>
@SpringBootTest에 비해 테스트 부담이 적음
<br><br>
**@MockBean**<br>
실제 빈 객체가 아닌 가짜 객체를 생성해서 주입
해당 애노테이션이 선언된 객체는 실제 객체가 아니므로 결과 반환과 같은 실제 행위를 수행하지 않음. 따라서 Mockito의 given() 메서드를 통해 동작 정의 필요
<br><br>
**@Test**<br>
테스트 코드가 포함되어 있다고 선언하는 애노테이션
<br><br>
**@DisplayName**<br>
테스트 메서드명이 복잡해서 가독성이 떨어질 경우 해당 애노테이션을 통해 테스트에 대한 표현 정의 가능
<br><br>
**given()**<br>
어떤 메서드가 호출되고 어떤 파라미터를 주입받는지 가정
<br><br>
**content()**<br>
@RequestBody의 값을 넘겨주는 역할 수행
<br><br>
**willReturn()**<br>
어떤 결과를 리턴할 것인지 정의
<br><br>
**perform()**<br>
서버로 URL 요청을 보내는 것처럼 통신 테스트 코드 작성
<br><br>
**andExpect()**<br>
결괏값 검증 수행 역할
<br><br>
**jsonPath().exists()**<br>
도출된 결괏값에 대해 각 항목이 존재하는지 검증
<br><br>
**andDo()**<br>
요청과 응답의 전체 내용 확인
<br><br>
**verify()**<br>
지정된 메서드가 실행됐는지 검증
<br><br>
### 슬라이스 테스트

단위 테스트와 통합 테스트의 중간 개념으로, 레이어드 아키텍처를 기준으로 각 레이어별 테스트 진행

**필요성**

단위 테스트의 경우 모든 외부 요인을 차단하고 테스트를 진행해야 하지만 컨트롤러와 같이 개념상 외부와 맞닿은 레이어의 경우 외부 요인을 차단하고 테스트하면 의미가 없음.

**대표적인 애노테이션**

```
@DataJdbcTest
@DataJpaTest
@DataMongoTest
@DataRedisTest
@JdbcTest
@JooqTest
@JsonTest
@RestClientTest
@WebFluxTest
@WebMvcTest
@WebServiceClientTest
```
