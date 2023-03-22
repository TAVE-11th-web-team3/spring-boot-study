# 7.4.2 스프링 부트 프로젝트 생성
***

**Dependencies**
* Developer Tools: ****Lombok, Spring Configuration Processor
* Web: Spring Web
* SQL: Spring Data JPA, MariaDB Driver

***

**소스코드 수정** 
    : ProductServiceImpl, Entity 클래스

**ProductServiceImpl 클래스**
    
``` java
@Service
public class ProductServiceImplV2 implements ProductService {

    // ch7 변경사항
    private final Logger LOGGER = LoggerFactory.getLogger(ProductServiceImplV2.class);
    private final ProductRepository productRepository;

    // ch7 변경사항
    @Autowired
    public ProductServiceImplV2(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Override
    public ProductResponseDto getProduct(Long number) {
        // ch7 변경사항
        LOGGER.info("[getProduct] input number: {}", number);
        Product product = productRepository.findById(number).get();

        // ch7 변경사항
        LOGGER.info("[getProduct] product number: {}, name : {}", product.getNumber(), product.getName());

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(product.getNumber());
        productResponseDto.setName(product.getName());
        productResponseDto.setPrice(product.getPrice());
        productResponseDto.setStock(product.getStock());

        return productResponseDto;
    }

    @Override
    public ProductResponseDto saveProduct(ProductDto productDto) {

        // ch7 변경사항
        LOGGER.info("[saveProduct] productDto: {}", productDto.toString());
        Product product = new Product();
        product.setName(productDto.getName());
        product.setPrice(productDto.getPrice());
        product.setStock(productDto.getStock());
        product.setCreatedAt(LocalDateTime.now());
        product.setUpdatedAt(LocalDateTime.now());


        // ch7 변경사항
        Product savedProduct = productRepository.save(product);
        LOGGER.info("[saveProduct] savedProduct: {}", savedProduct);

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(savedProduct.getNumber());
        productResponseDto.setName(savedProduct.getName());
        productResponseDto.setPrice(savedProduct.getPrice());
        productResponseDto.setStock(savedProduct.getStock());

        return productResponseDto;
    }

    @Override
    public ProductResponseDto changeProductName(Long number, String name) throws Exception {

        // ch7 변경사항
        Product foundProduct = productRepository.findById(number).get();
        foundProduct.setName(name);
        Product changedProduct = productRepository.save(foundProduct);

        ProductResponseDto productResponseDto = new ProductResponseDto();

        productResponseDto.setNumber(changedProduct.getNumber());
        productResponseDto.setName(changedProduct.getName());
        productResponseDto.setPrice(changedProduct.getPrice());
        productResponseDto.setStock(changedProduct.getStock());

        return productResponseDto;
    }

    @Override
    public void deleteProduct(Long number) throws Exception {
        // ch7 변경사항
        productRepository.deleteById(number);
    }
}
```

**Entity 클래스**
```java
// ch7 변경사항
@Entity
@Builder
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode
@ToString(exclude = "name")
@Table(name = "product")
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

}
```