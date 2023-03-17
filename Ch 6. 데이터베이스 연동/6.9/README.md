# DAO 설계

> DAO(Data Access Object) : 데이터베이스에 접근하기 위한 로직을 관리하는 객체 
> <br> 비즈니스 로직의 동작 과정에서 데이터를 조작하는 기능은 DAO 객체가 수행함
> <br> 스프링 데이터 JPA에서 DAO의 개념은 리포지토리가 대체하고 있음

###  DAO 클래스 생성
* DAO 클래스는 일반적으로 '인터페이스 - 구현체' 구성으로 생성
* DAO 클래스는 의존성 결합을 낮추기 위한 디자인 패턴으로, 서비스 레이어에서 DAO 객체를 주입받을 때 인터페이스를 선언하는 방식으로 
구성할 수 있음

  1) DAO 인터페이스 생성 ( ProductDAO 인터페이스 )
  ```java
  public interface ProductDAO {
    // 제품 삽입 메서드
    Product insertProduct(Product product);
    // 단일 제품 조회 메서드
    Product selectProduct(Long number);
    // 제품명 수정 메서드
    Product updateProductName(Long number, String name) throws Exception;
    // 제품 삭제 메서드
    void deleteProduct(Long number) throws Exception;
  }
  ``` 
  2) DAO 구현체 생성 ( ProductDAOImpl 클래스 )

  ```java
  // 클래스를 스프링 빈으로 등록 ( @Componet 또는 @Service 사용)
  @Component
  public class ProductDAOImpl implements ProductDAO {
  private final ProductRepository productRepository;

    // 생성자를 이용한 의존성 주입
    @Autowired
    public ProductDAOImpl(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Override
    public Product insertProduct(Product product) {
        return null;
    }

    @Override
    public Product selectProduct(Long number) {
        return null;
    }

    @Override
    public Product updateProductName(Long number, String name) throws Exception {
        return null;
    }

    @Override
    public void deleteProduct(Long number) throws Exception {
    }
  }
  ```
  3) 인터페이스 메소드 구현
  * 저장 메서드 : insertProduct()
  ```java
    @Override
    public Product insertProduct(Product product) {
        // spring data JPA가 제공하는 기본 메소드 save() 
        Product savedProduct = productRepository.save(product);
        
        return savedProduct;
    }
  ```
  * 조회 메서드 : selectProduct() 
  <br>- **getById()**: 내부적으로 EntityManager의 getReference() 메서드를 호출함. 해당 메서드는 proxy 객체를 리턴하기 때문에 실제 쿼리는
  proxy 객체를 통해 최초로 데이터에 접근하는 시점에 실행됨.
    <br><br>- **findById()**: 내부적으로 EntityManager의 find() 메서드를 호출함. 해당 메서드는 먼저 영속성 컨텍스트의 캐시에서 값을 조회하고,
  영속성 컨텍스트에 값이 존재하지 않는다면 실제 데이터베이스에서 조회함.
  ```java
    @Override
    public Product selectProduct(Long number) {
        Product selectedProduct = productRepository.getById(number);
        return selectedProduct;
    }
  ```

  * 업데이트 메서드 : updateProductName()
  ```java
    // 상품명 업데이트 메서드
    @Override
    public Product updateProductName(Long number, String name) throws Exception {
        Optional<Product> selectedProduct = productRepository.findById(number);

        Product updatedProduct;
        if (selectedProduct.isPresent()) {
            Product product = selectedProduct.get();

            // 별도의 update() 메서드가 없고, save() 호출 시 JPA의 더티 체크(Dirty Check)를 통해 변경 감지를 수행
            product.setName(name);
            product.setUpdatedAt(LocalDateTime.now());

            updatedProduct = productRepository.save(product);
        } else { // 데이터가 존재하지 않는 경우 예외처리
            throw new Exception();
        }
        return updatedProduct;
    }
  ```
  
  * 삭제 메서드 : deleteProduct() 
  ```java
    @Override
    public void deleteProduct(Long number) throws Exception {
        // 객체를 가져오는 작업
        Optional<Product> selectedProduct = productRepository.findById(number);

        if (selectedProduct.isPresent()) {
            Product product = selectedProduct.get();

            productRepository.delete(product);
        } else {
            throw new Exception();
        }
    }  
  ```
  <br>- SimpleJpaRepository의 delete()메서드는 delete() 메서드로 전달 받은 엔티티가 영속성 컨텍스트에 있는지 파악하고,
  해당 엔티티를 영속성 컨텍스트에 영속화하는 작업을 거쳐 데이터베이스의 레코드와 매핑한 후, 삭제 요청을
  수행하는 메서드를 실행해 작업을 마치고 커밋(Commit) 단계에서 삭제를 진행함.
  <br><br> 삭제 과정 정리<br>1) em.find()를 실행하여 엔티티가 내부 영속 컨텍스트 내부 캐시에 등록되지 않은 경우, DB를 조회해 객체를 
  영속성 컨텍스트에 저장함
  <br>2) em.remove() 실행시, 캐시에 등록되어 있던 엔티티를 삭제하고, delete sql문을 쓰기 지연 SQL 저장소에 저장
  <br>3) 커밋 단계에서 delete sql문 실행