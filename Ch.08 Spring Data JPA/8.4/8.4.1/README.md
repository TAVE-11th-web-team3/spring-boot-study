## **1) 정렬 처리하기**

**Spring Data JPA에서 정렬을 사용할 때는 일반적인 쿼리문과 비슷하게 Order By 구문을 사용한다.**

```java
    // Asc: 오름차순, Desc: 내림차순
    List<Product> findByNameOrderByNumberAsc(String name);
    List<Product> findByNameOrderByNumberDesc(String name);
```

**하이버네이트 로그 ( findByNameOrderByNumberAsc(String name) 실행 결과 )** 

```xml
Hibernate: 
    select
        product0_.number as number1_0_,
        product0_.created_at as created_2_0_,
        product0_.name as name3_0_,
        product0_.price as price4_0_,
        product0_.stock as stock5_0_,
        product0_.updated_at as updated_6_0_ 
    from
        product product0_ 
    where
        product0_.name=? 
    order by
        product0_.number asc
```

---

**정렬 구문에서 여러 정렬 기준을 사용하기 위해서는 우선순위를 기준으로 차례대로 작성한다.**

```java
    List<Product> findByNameOrderByPriceAscStockDesc(String name);
```

**하이버네이트 로그**

```text
Hibernate: 
    select
        product0_.number as number1_0_,
        product0_.created_at as created_2_0_,
        product0_.name as name3_0_,
        product0_.price as price4_0_,
        product0_.stock as stock5_0_,
        product0_.updated_at as updated_6_0_ 
    from
        product product0_ 
    where
        product0_.name=? 
    order by
        product0_.price asc,
        product0_.stock desc
```

- **Price**를 기준으로 **오름차순 정렬**한 후 **Stock** 기준으로 **내림차순 정렬**을 하는 쿼리가 제대로 나갔음을 확인할 수 있다.

- 이처럼 쿼리 메서드의 이름에 **정렬 키워드를 삽입**해서 정렬을 수행하는 것도 가능하지만 메서드의 이름이 길어질수록
**가독성이 떨어지는 문제**가 발생한다.

---

**매개변수를 활용하는 방법**

```java
    List<Product> findByName(String name, Sort sort);
```

- 메서드 이름에 키워드를 넣지 않고 Sort 객체를 활용해 매개변수로 받아들인 정렬 기준을 가지고 쿼리문을 작성하는 방법

**테스트 코드 작성**

```java
@SpringBootTest
class ProductRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Test
    public void sortingAndPagingTest() throws Exception {

        Product product1 = new Product();
        product1.setName("펜");
        product1.setPrice(1000);
        product1.setStock(100);
        product1.setCreatedAt(LocalDateTime.now());
        product1.setUpdatedAt(LocalDateTime.now());


        Product product2 = new Product();
        product2.setName("펜");
        product2.setPrice(000);
        product2.setStock(300);
        product2.setCreatedAt(LocalDateTime.now());
        product2.setUpdatedAt(LocalDateTime.now());


        Product product3 = new Product();
        product3.setName("펜");
        product3.setPrice(3000);
        product3.setStock(4000);
        product3.setCreatedAt(LocalDateTime.now());
        product3.setUpdatedAt(LocalDateTime.now());

        productRepository.save(product1);
        productRepository.save(product2);
        productRepository.save(product3);

        productRepository.findByName("펜", Sort.by(Order.asc("price")));
        productRepository.findByName("펜", Sort.by(Order.asc("price"), Order.desc("stock")));
        
        }
```

- **Sort** 클래스는 **내부 클래스**로 정의돼 있는 **Order** 객체를 활용해 정렬 기준을 생성한다.

- **Order** 객체에는 **asc**, **desc** 메서드가 포함되어 있어 **정렬 방향**을 지정하며, 여러 정렬 기준을 사용할 경우 **콤마(,)** 사용함

**하이버네이트 로그**

```text
Hibernate: 
    select
        product0_.number as number1_0_,
        product0_.created_at as created_2_0_,
        product0_.name as name3_0_,
        product0_.price as price4_0_,
        product0_.stock as stock5_0_,
        product0_.updated_at as updated_6_0_ 
    from
        product product0_ 
    where
        product0_.name=? 
    order by
        product0_.price asc
        
        
Hibernate: 
    select
        product0_.number as number1_0_,
        product0_.created_at as created_2_0_,
        product0_.name as name3_0_,
        product0_.price as price4_0_,
        product0_.stock as stock5_0_,
        product0_.updated_at as updated_6_0_ 
    from
        product product0_ 
    where
        product0_.name=? 
    order by
        product0_.price asc,
        product0_.stock desc
```

- 쿼리 메서드를 정의하는 단계에서 코드가 줄어드는 장점이 있지만, 호출하는 위치에서 정렬 기준이 길어져 가독성이 떨어진다.

---

**Sort 부분을 하나의 메서드로 분리해서 쿼리 메서드를 호출하는 코드를 작성하는 방법**

```java
private static Sort getSort() {
    return Sort.by(
            Order.asc("price"),
            Order.desc("stock")
    );
```