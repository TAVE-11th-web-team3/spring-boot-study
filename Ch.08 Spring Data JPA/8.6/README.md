**@Query의 한계점**

메서드의 이름을 기반으로 생성하는 JPQL의 한계를 @Query 애노테이션을 통해 대부분 해소할 수 있지만, 해당 과정에서 직접 문자열을 입력하기 때문에 잘못된 입력을 받은 경우 컴파일 시점에 에러를 잡지 못하고 애플리케이션이 실행된 이후에 런타임 에러가 발생할 가능성 존재
→ QueryDSL을 통해 문자열이 아닌 코드로 쿼리를 작성하는 방법을 적용하여 문제를 해결할 수 있음.


## QueryDSL이란?
- 정적 타입을 이용해 SQL과 같은 쿼리를 생성할 수 있도록 지원하는 프레임워크
- 문자열이나 XML 파일을 통해 쿼리를 작성하는 대신 QueryDSL이 제공하는 플루언트 API를 활용해 쿼리 생성 가능

## QueryDSL의 장점
- IDE가 제공하는 코드 자동 완성 기능 사용 가능
- 문법적으로 잘못된 쿼리를 활용하지 않기 때문에 정상적으로 활용된 경우 문법 오류를 발생시키지 않음
- 고정된 SQL 쿼리를 작성하지 않으므로 동적으로 쿼리 생성이 가능
- 코드를 통해 작성하므로 가독성 및 생산성 향상
- 도메인 타입과 프로퍼티를 안전하게 참조 가능


## 기본적인 QueryDSL 사용하기
```java
class ProductRepositoryTest {
    @PersistenceContext
    EntityManager entityManager;
    
    @Autowired
    JPAQueryFactory jpaQueryFactoryForTest4;
    
    @Test
    void queryDslTest() {
        JPAQuery<Product> query = new JPAQuery(entityManager);
        QProduct qProduct = QProduct.product;
        
        //빌더 형식으로 쿼리 작성
        List<Product> productList = query
            .from(qProduct)
            .where(qProduct.name.eq("펜"))
            .orderBy(qProduct.price.asc())
            .fetch();
            
        for (Product product : productList) {
            System.out.println("-----");
            System.out.println();
            System.out.println("Product Number : " + product.getNumber());
            System.out.println("Product Name : " + product.getName());
            System.out.println("Product Price : " + product.getPrice());
            System.out.println("Product Stock : " + product.getStock());
            System.out.println();
            System.out.println("-----");
        }
    }
    
    @Test
    void queryDslTest2() {
        JPAQueryFactory jpaQueryFactory = new JPAQueryFactory(entityManager);
        QProduct qProduct = QProduct.product;
        
        List<Product> productList = jpaQueryFactory.selectFrom(qProduct) //전체 칼럼 조회
            .where(qProduct.name.eq("펜"))
            .orderBy(qProduct.price.asc())
            .fetch();
            
        for (Product product : productList) {
            System.out.println("-----");     
            System.out.println();
            System.out.println("Product Number : " + product.getNumber());
            System.out.println("Product Name : " + product.getName());
            System.out.println("Product Price : " + product.getPrice());
            System.out.println("Product Stock : " + product.getStock());
            System.out.println();
            System.out.println("-----");
        }
    }
    
    @Test
    void queryDslTest3() {
        JPAQueryFactory jpaQueryFactory = new JPAQueryFactory(entityManager);
        QProduct qProduct = QProduct.product;
        
        //select 대상이 하나인 경우
        List<String> productList = jpaQueryFactory
                .select(qProduct.name) //일부 칼럼 조회
                .from(qProduct)
                .where(qProduct.name.eq("펜"))
                .orderBy(qProduct.price.asc())
                .fetch();
                
        for (String product : productList) {
            System.out.println("-----");     
            System.out.println("Product Name : " + product);
            System.out.println("-----");     
        }
        
        //select 대상이 여러 개 경우
        List<Tuple> tupleList = jpaQueryFactory
            .select(qProduct.name, qProduct.price)
            .from(qProduct)
            .where(qProduct.name.eq("펜"))
            .orderBy(qProduct.price.asc())
            .fetch();
            
        for (Tuple product : tupleList) {
            System.out.println("-----");
            System.out.println("Product Name : " + product.get(qProduct.name));
            System.out.println("Product Price : " + product.get(qProduct.price));
            System.out.println("-----");
        }
    }
    
    @Test
    void queryDslTest4() {
        QProduct qProduct = QProduct.product;
        
        List<String> productList = jpaQueryFactoryForTest4
                .select(qProduct.name)
                .from(qProduct)
                .where(qProduct.name.eq("펜"))
                .orderBy(qProduct.price.asc())
                .fetch();
                
        for (String product : productList) {
            System.out.println("-----");     
            System.out.println("Product Name : " + product);
            System.out.println("-----");     
        }
    }
}
```
아래와 같이 JPAQueryFactory 객체를 빈 객체로 등록해두면 매번 초기화 할 필요 없이 스프링 컨테이너에서 가져다 쓸 수 있다.
```java
@Configuration
public class QueryDSLConfiguration {
    @PersistenceContext
    EntityManager entityManager;
    
    @Bean
    public JPAQueryFactory jpaQueryFactory() {
        return new JPAQueryFactory(entityManager);
    }
}
```


## QuerydslPredicateExecutor, QuerydslRepositorySupport 활용

**QuerydslPredicateExecutor 인터페이스**

- QuerydslPredicateExecutor는 JpaRepository와 함께 QueryDSL을 사용할 수 있도록 인터페이스 제공
- 인터페이스의 메서드는 대부분 Predicate 타입을 매개변수로 받으며, 다양한 종류의 메서드가 존재
- 해당 인터페이스를 활용하면 더욱 편하게 QueryDSL을 사용할 수 있지만 join이나 fetch 기능은 사용할 수 없다는 단점 존재
<br><br><br><br>

**QuerydslRepositorySupport 추상 클래스 사용하기**

- QuerydslRepositorySupport 클래스 또한 QueryDSL을 사용하는 데 유용한 기능을 제공
<br>

<일반적으로 사용하는 방식>
1. JpaRepository를 상속받는 리포지토리 A 생성
2. 직접 구현한 쿼리를 사용하기 위해 JpaRepository를 상속받지 않는 커스텀 리포지토리 B를 생성 후, 메서드를 정의
3. 리포지토리 B에서 정의한 메서드를 사용하기 위해 리포지토리 A에서 리포지토리 B를 상속
4. 리포지토리 B의 구현체 클래스 C 작성
5. 클래스 C에서 다양한 방식으로 쿼리를 구성할 수 있지만, QueryDSL을 사용하기 위해 QuerydslRepositorySupport 상속
6. DAO나 서비스에서 리포지토리에 접근하기 위해 리포지토리 A를 사용하고, 이에 따라 QueryDSL의 기능도 사용이 가능
