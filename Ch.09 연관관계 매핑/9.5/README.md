# 다대다 매핑
다대다(M:N) 연관관계는 각 엔티티에서 서로를 리스트로 가지는 구조가 만들어지는데, 이러한 경우 교차 엔티티라고 부르는 중간 테이블을 생성해서 다대다 관계를 일대다 또는 다대일 관계로 해소한다.

## 다대다 단방향 매핑

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "producer")
public class Producer extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String code;

    private String name;

    @ManyToMany
    @ToString.Exclude
    private List<Product> products = new ArrayList<>();

    pubic void addProduct(Product product) {
        products.add(product);
    }
}
```

```java
public interface ProducerRepository extends JpaRepository<Producer, Long> {
}
```

생산업체에 매핑되는 도메인은 Producer라고 가정하고 위와 같은 리포지토리를 생성하면 생산업체에 대한 기본적인 데이터베이스 조작이 가능하다.

## 다대다 양방향 매핑

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

    @OneToOne(mappedBy = "product")
    @ToString.Exclude
    private ProductDetail productDetail;

    @ManyToOne
    @JoinColumn(name = "provider_id")
    @ToString.Exclude
    private Provider provider;

    @ManyToMany
    @ToString.Exclude
    private List<Producer> producers = new ArrayList<>();

    public void addProducer(Producer producer) {
        this.producers.add(producer);
    }
}
```

`@ManyToMany`를 통해 생산업체에 대한 다대다 연관관계를 설정하고 있다. 즉, 중간 테이블이 연관관계를 설정하고 있는 구조이기 때문에 데이터베이스의 테이블 구조는 변경되지 않는다.

다대다 연관관계를 설정하면 중간 테이블을 통해 연관된 엔티티의 값을 가져올 수 있지만 중간 테이블이 생성되기 때문에 예기치 못한 쿼리가 발생하는 등 관리하기 힘든 포인트가 발생할 수 있다는 한계가 존재한다. 이러한 다대다 연관관계의 한계를 극복하기 위해 중간 테이블을 생성하는 대신 일대다 다대일로 연관관계를 맺을 수 있는 중간 엔티티로 승격시켜 JPA에서 관리할 수 있도록 할 수 있다.
