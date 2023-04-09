## 9.3 일대일 매핑
___

**상품 테이블과 일대일로 매핑될 상품정보 테이블 생성**

![image](https://user-images.githubusercontent.com/105872347/230773859-160a50c3-4873-4696-ac36-0f749ae02b5b.png)

이처럼 하나의 상품에 하나의 상품정보만 매핑되는 구조는 일대일 관계라고 볼 수 있음.
___
### 9.3.1 일대일 단방향 매핑

```java
@Entity
@Table(name = "product_detail")
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class ProductDetail extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String description;

    @OneToOne
    @JoinColumn(name = "product_number")
    private Product product;

}
```
* **@OneToOne** : 다른 엔티티 객체를 필드로 정의했을 때 **일대일** 연관관계로 매핑하기 위해 사용
* **@JoinColumn** 
    * name : 매핑할 외래키의 이름 설정
    * referencedColumnName : 외래키가 참조할 상대 테이블의 칼럼명 지정
    * foreignKey : 외래키를 생성하면서 지정할 제약조건을 설정 (unique, nullable, insertable, updatable 등)

<br>- **@JoinColumn**은 기본값이 설정돼 있어 자동으로 이름을 매핑하지만 의도한 이름이 들어가지 않기 떄문에
   **name** 속성을 사용해 원하는 컬럼명을 지정하는 것이 좋습니다.


```java
// ProductDetailRepository 생성
    public interface ProductDetailRepository extends JpaRepository<ProductDetail, Long> {

}
```
**테스트 코드 작성**
```java
@SpringBootTest
class ProductDetailRepositoryTest {

    @Autowired
    ProductDetailRepository productDetailRepository;

    @Autowired
    ProductRepository productRepository;

    @Test
    public void saveAndReadTest1(){
        Product product = new Product();
        product.setName("스프링 부트 JPA");
        product.setPrice(5000);
        product.setStock(500);

        productRepository.save(product);

        ProductDetail productDetail = new ProductDetail();
        productDetail.setProduct(product);
        productDetail.setDescription("스프링 부트와 JPA를 함께 볼 수 있는 책");

        productDetailRepository.save(productDetail);

        // 생성한 데이터 조회
        // ProductDetail 객체에서 Product 객체를 일대일 단방향 연관관계를 설정했기 때문에 ProductDetailRepository에서 
        // ProductDetail 객체를 조회한 후 연관 매핑된 Product 객체를 조회할 수 있다.
        System.out.println("savedProduct: " + productDetailRepository.findById(productDetail.getId()).get().getProduct());
        
        System.out.println("savedProductDetail : " + productDetailRepository.findById(productDetail.getId()).get());
    }
}
```
**쿼리 로그 확인**
```text
Hibernate: 
    select
        productdet0_.id as id1_1_0_,
        productdet0_.created_at as created_2_1_0_,
        productdet0_.updated_at as updated_3_1_0_,
        productdet0_.description as descript4_1_0_,
        productdet0_.product_number as product_5_1_0_,
        product1_.number as number1_0_1_,
        product1_.created_at as created_2_0_1_,
        product1_.updated_at as updated_3_0_1_,
        product1_.name as name4_0_1_,
        product1_.price as price5_0_1_,
        product1_.stock as stock6_0_1_ 
    from
        product_detail productdet0_ 
    left outer join
        product product1_ 
            on productdet0_.product_number=product1_.number 
    where
        productdet0_.id=?
```
**select** 구문을 보면 **productDetail** 객체와 **Product** 객체가 함께 조회되는 것을 볼 수 있습니다. 이처럼 엔티티를 조회할 때 연관된 엔티티도 함께 
조회하는 것을 '**즉시 로딩**'이라고 합니다. 또한 **@OneToOne** 어노테이션으로 인해 **left outer join**이 생성되는 것을 볼 수 있습니다.

```java
// @OneToOne 어노테이션 인터페이스
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface OneToOne {
    Class targetEntity() default void.class;

    CascadeType[] cascade() default {};

    FetchType fetch() default FetchType.EAGER;

    boolean optional() default true;

    String mappedBy() default "";

    boolean orphanRemoval() default false;
}
```
* fetch() : 기본 전략으로 EAGER, 즉 즉시 로딩 전략이 채택되어 있음
* optional() : 기본값이 true (매핑되는 값이 nullable)

```java
public class ProductDetail extends BaseEntity {

@OneToOne(optional = false)
@JoinColumn(name = "product_number")
private Product product;
}
```
optional = false 속성 설정 시 테이블 생성 쿼리
```text
Hibernate: 
    
    create table product_detail (
       id bigint not null auto_increment,
        created_at datetime(6),
        updated_at datetime(6),
        description varchar(255),
        product_number bigint not null,  // NOT NULL이 설정됨
        primary key (id)
    ) engine=InnoDB
```
Test 실행 쿼리
```text
Hibernate: 
    select
        productdet0_.id as id1_1_0_,
        productdet0_.created_at as created_2_1_0_,
        productdet0_.updated_at as updated_3_1_0_,
        productdet0_.description as descript4_1_0_,
        productdet0_.product_number as product_5_1_0_,
        product1_.number as number1_0_1_,
        product1_.created_at as created_2_0_1_,
        product1_.updated_at as updated_3_0_1_,
        product1_.name as name4_0_1_,
        product1_.price as price5_0_1_,
        product1_.stock as stock6_0_1_ 
    from
        product_detail productdet0_ 
    inner join   // left outer join  ->  inner join
        product product1_ 
            on productdet0_.product_number=product1_.number 
    where
        productdet0_.id=?
```
___
### 9.3.2 일대일 양방향 매핑
양방향 매핑은 양쪽에서 단방향으로 서로를 매핑하는 것을 의미합니다.

**Product 엔티티**
```java
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "product")
public class Product extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    // 단방향 관계를 추가함
    @OneToOne
    private ProductDetail productDetail;
}
```
**다음과 같이 Product 테이블에 Product_detail_id 칼럼이 추가됩니다**


![image](https://user-images.githubusercontent.com/105872347/230780259-29fe39e2-dc8d-40de-bd7d-4a85707ab02f.png)

**쿼리 확인**
```text
Hibernate: 
    select
        productdet0_.id as id1_1_0_,
        productdet0_.created_at as created_2_1_0_,
        productdet0_.updated_at as updated_3_1_0_,
        productdet0_.description as descript4_1_0_,
        productdet0_.product_number as product_5_1_0_,
        product1_.number as number1_0_1_,
        product1_.created_at as created_2_0_1_,
        product1_.updated_at as updated_3_0_1_,
        product1_.name as name4_0_1_,
        product1_.price as price5_0_1_,
        product1_.product_detail_id as product_7_0_1_,
        product1_.stock as stock6_0_1_,
        productdet2_.id as id1_1_2_,
        productdet2_.created_at as created_2_1_2_,
        productdet2_.updated_at as updated_3_1_2_,
        productdet2_.description as descript4_1_2_,
        productdet2_.product_number as product_5_1_2_ 
    from
        product_detail productdet0_ 
    left outer join
        product product1_ 
            on productdet0_.product_number=product1_.number 
    left outer join
        product_detail productdet2_ 
            on product1_.product_detail_id=productdet2_.id 
    where
        productdet0_.id=?
```
쿼리를 보면 양쪽에서 외래키를 가지고 left outer join이 두 번이나 수행되어 효율성이 떨어집니다.
<br>실제 데이터베이스에서도 테이블 간 연관관계를 맺으면 한쪽 테이블이 외래키를 가지는 구조로 이뤄집니다 ( **'주인'** 개념 )
> JPA 에서도 실제 데이터베이스의 연관관계를 반영해서 한쪽의 테이블에서만 외래키를 바꿀 수 있도록 정하는 것이 좋습니다.

___
### JPA 양방향 연관관계의 주인을 지정하는 방법
* **mappedBy** 속성 사용
   : **엔티티는 양방향**으로 매핑하되 **한쪽에게만 외래키**를 줘야 하는데, 이때 사용되는 속성 값이 mappedBy 입니다. 
    **mappedBy** 속성은 어떤 객체가 주인인지 표시하는 속성입니다.


```java
public class Product extends BaseEntity {

    // @OneToOne 어노테이션에 mappedBy 속성값을 사용합니다.
    // mappedBy에 들어가는 값은 연관관계를 갖고 있는 상대 엔티티에 있는 연관관계 필드의 이름입니다.
    @OneToOne(mappedBy = "product")
    private ProductDetail productDetail;
    
}
```
이 설정으로 ProductDetail 엔티티가 Product 엔티티의 주인이 되고,
다음과 같이 Product 테이블에 있던 외래키가 사라진 것을 볼 수 있습니다.

![image](https://user-images.githubusercontent.com/105872347/230781612-e0046256-3dcb-4d0c-a667-efb3bb504de4.png)

