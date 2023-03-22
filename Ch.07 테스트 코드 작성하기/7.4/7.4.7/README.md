# 서비스 객체 테스트

도메인 모델을 활용해 애플리케이션에서 제공하는 핵심 기능 제공

### 테스트 코드 작성

```java
class ProductServiceTest {

    private ProductRepository productRepository = Mockito.mock(ProductRepository.class);
    private ProductServiceImpl productService;

    @BeforeEach
    public void setUpTest() {
        productService = new ProductServiceImpl(productRepository);
    }
    @Test
    void saveProduct() {
        Mockito.when(productRepository.save(any(Product.class)))
                .then(returnsFirstArg());

        ProductResponseDto productResponseDto = productService.saveProduct(new ProductDto("pen", 1000, 1234));

        assertEquals(productResponseDto.getName(), "pen");
        assertEquals(productResponseDto.getPrice(), 1000);
        assertEquals(productResponseDto.getStock(), 1234);

        verify(productRepository).save(any());
    }
}
```

슬라이스 테스트를 적용한 컨트롤러 객체와 달리 서비스 객체는 단위 테스트를 적용하였기에 @SpringBootTest, @WebMvcTest와 같은 외부 요인을 모두 배제
<br><br>

**any()**

\- Mockito의 ArgumentMatchers에서 제공하는 메서드로, Mock 객체의 동작을 정의하거나 검증하는 단계에서 조건으로 특정 매개변수의 전달을 설정하지 않고 메서드의 실행만을 확인하거나 좀 더 큰 범위의 클래스 객체를 매개변수로 전달받는 등의 상황에 사용

\- 일반적으로 given()으로 정의된 Mock 객체의 메서드 동작 감지는 매개변수의 비교를 통해 이루어지지만, 레퍼런스 변수의 비교는 주솟값으로 이루어지기 때문에 any()를 사용해 클래스만 정의하는 경우도 존재
