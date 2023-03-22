# 리포지토리 객체 테스트

개발자가 구현하는 레이어 중 가장 데이터베이스와 가까운 레이어

\- JpaRepository를 상속받아 기본적인 쿼리 메서드를 사용할 수 있기 때문에 findById(), save()와 같은 기본 메서드에 대한 테스트는 큰 의미가 없음 → 테스트 구현 목적에 대해 고민하고 작성해야 함

\- 외부 요인에 속하는 데이터베이스의 연동 여부는 테스트 목적에 따라 달라질 수 있고, 실제 운용 DB와 테스트용 DB를 다르게 지정하여 연동할 수도 있음

### 테스트 코드 작성

```java
@DataJpaTest
public class ProductRepositoryTestByH2 {
    @Autowired
    private ProductRepository productRepository;

    @Test
    void saveTest() {
        //given
        Product product = new Product();
        product.setName("pen");
        product.setPrice(1000);
        product.setStock(1000);

        //when
        Product savedProduct = productRepository.save(product);

        //then
        assertEquals(product.getName(), savedProduct.getName());
        assertEquals(product.getPrice(), savedProduct.getPrice());
        assertEquals(product.getStock(), savedProduct.getStock());
    }

    @Test
    void selectTest() {
        //given
        Product product = new Product();
        product.setName("pen");
        product.setPrice(1000);
        product.setStock(1000);

        Product savedProduct = productRepository.saveAndFlush(product);

        //when
        Product foundProduct = productRepository.findById(savedProduct.getNumber()).get();

        //then
        assertEquals(product.getName(), foundProduct.getName());
        assertEquals(product.getPrice(), foundProduct.getPrice());
        assertEquals(product.getStock(), foundProduct.getStock());
    }
}
```

given절에서 객체를 데이터베이스에 저장하고, 저장된 데이터를 조회하여 비교를 통해 검증 수행
<br><br>
**@DataJpaTest**

\- JPA와 관련된 설정만을 로드해서 테스트 진행

\- 기본적으로 @Transactional 애노테이션을 포함하고 있어 테스트 코드가 종료되면 자동으로 데이터베이스의 롤백 진행

\- 기본 값으로 임베디드 데이터베이스를 사용하고, @AutoConfigureTestDatabase 애노테이션을 이용해 별도로 설정을 하여 다른 데이터베이스를 이용할 수 있음(해당 예시에서는 H2 사용)
<br><br>
# @SpringBootTest를 이용한 CRUD 테스트

```java
@SpringBootTest
public class ProductRepositoryTest {
    @Autowired
    ProductRepository productRepository;

    @Test
    public void basicCRUDTest() {
        //create
        Product givenProduct = Product.builder()
                .name("pen")
                .price(5000)
                .stock(2000)
                .build();
        Product savedProduct = productRepository.save(givenProduct);

        assertThat(savedProduct.getNumber())
                .isEqualTo(givenProduct.getNumber());
        assertThat(savedProduct.getName())
                .isEqualTo(givenProduct.getName());
        assertThat(savedProduct.getPrice())
                .isEqualTo(givenProduct.getPrice());
        assertThat(savedProduct.getStock())
                .isEqualTo(givenProduct.getStock());


        //read
        Product selectedProduct = productRepository.findById(savedProduct.getNumber())
                .orElseThrow(RuntimeException::new);

        assertThat(selectedProduct.getNumber())
                .isEqualTo(givenProduct.getNumber());
        assertThat(selectedProduct.getName())
                .isEqualTo(givenProduct.getName());
        assertThat(selectedProduct.getPrice())
                .isEqualTo(givenProduct.getPrice());
        assertThat(selectedProduct.getStock())
                .isEqualTo(givenProduct.getStock());


        //update
        Product foundProduct = productRepository.findById(selectedProduct.getNumber())
                .orElseThrow(RuntimeException::new);
        foundProduct.setName("toy");

        Product updatedProduct = productRepository.save(foundProduct);
        assertEquals(updatedProduct.getName(), "toy");


        //delete
        productRepository.delete(updatedProduct);
        assertFalse(productRepository.findById(selectedProduct.getNumber()).isPresent());
    }
}
```

@SpringBootTest 애노테이션을 활용하면 스프링의 모든 설정을 가져오고, 전체적인 빈 스캔도 진행하기 때문에 의존성 주입에 대해 고민할 필요 없이 테스트가 가능하지만 속도가 느리다는 단점이 존재
