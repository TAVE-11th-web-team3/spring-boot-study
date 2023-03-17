![header](https://capsule-render.vercel.app/api?type=wave&color=C3E5AE&height=200&section=header&text=Spring&nbsp;Boot&nbsp;Study&fontSize=50&fontColor=000000)

## 6.10 &nbsp; DAO 연동을 위한 컨트롤러와 서비스 설계  

  ### 6.10.1 서비스 클래스 만들기
  #### 1. DTO 클래스 생성 
  
  :star2: ProductDto 클래스
  
  ``` java
   @Getter
   @Setter
   public class ProductDto {
     
     private String name;
     private int price;
     private int stock;
     
     public ProductDto(String name, int price, int stock) {
       this.name = name;
       this.price = price;
       this.stock = stock;
     }
   }  

  ```   
  
  
   :star2: ProductResponseDto 클래스
  
  ``` java
   @Getter
   @Setter
   public class ProductResponseDto {
     
     private Long number;
     private String name;
     private int price;
     private int stock;
     
     public ProductResponseDto() {}
     
     public ProductDto(Long number, String name, int price, int stock) {
       this.number = number;
       this.name = name;
       this.price = price;
       this.stock = stock;
     }
   }  

  ```  
  #### 2. Service interface 생성
  
   :star2: ProductService interface  
   
   ```java
      public interface ProductService {
          ProductResponseDto getProduct(Long number);
          ProductResponseDto saveProduct(ProductDto productDto);
          ProductResponseDto changeProductName(Long number,String name) throws Exception;
          void deleteProduct(Long number) throws Exception;
      }
   ```    
   
   
   * DAO 에서 구현한 기능을 서비스 인터페이스에서 호출해 결괏값을 가져오는 작업을 수행하도록 설계   
   * 서비스 레이어에서 DTO 객체와 엔티티 객체를 **각 레이어에 변환해서 전달하는 역할**도 수행
   
   :exclamation: 서비스는 클라이언트가 요청한 데이터를 **적절하게 가공**해서 **컨트롤러에게 넘기는 역할**
   
   #### 3. Spring Boot Application 의 구조 
   
   ![image](https://user-images.githubusercontent.com/78422940/225923750-690e630e-973a-462a-9b76-58b3b71e1b41.png)
   
   1. 데이터베이스와 밀접한 관련이 있는 **데이터 액세스 레이어 까지는 엔티티 객체를 사용**
   2. **클라이언트와 가까워지는 다른 레이어에서는 데이터를 교환하는 데에 DTO 객체를 사용**


  #### 4. Service 레이어 구현
  
  ```java
    @Service
    public class ProductServiceImpl implements ProductService {
        private final ProductDAO productDAO;
        
        @AutoWired
        public ProudctServiceImpl (ProductDAO productDAO) {
          this.productDAO = productDAO;
        }
      
        @Override
        public ProductResponseDto saveProduct(ProductDto productDto) {
            //1. 엔티티 객체 생성
            Product product= new Product();
            
            //2. 엔티티 객체 초기화
            product.setName(productDto.getName());
            product.setPrice (productDto.getPrice());
            product.setStock(productDto.getStock());
            product.setCreatedAt(LocalDateTime.now());
            product.setUpdatedAt(LocalDateTime.now());
            
            //3. DAO 객체로 전달
            Product saveProduct = productDAO.insertProduct(product);
          
            //4. 데이터베이스에 저장한 데이터의 인덱스를 DTO 에 담아 클라이언트에 전달
            ProductResponseDto productResponseDto = new ProductResponseDto();
            productResponseDto.setNumber(saveProduct.getNumber());
            productResponseDto.setName(saveProduct.getName());
            productResponseDto.setPrice(saveProduct.getPrice());
            productResponseDto.setStock(saveProduct.getStock());
          
            return productResponseDto;
        }
        
       @override
       public ProductResponseDto changeProductName(Long number, String name) throws Exception {
         //1. DAO 에 클라이언트로 부터 전달 받은 number 와 name을 넘겨 실제 레코드 값을 변경하는 작업 수행
         Product changeProduct = productDAO. updateProductName(number,name);
         
         //2. 데이터베이스 수정한 데이터의 인덱스를 DTO 에 담아 클라이언트에 전달
         ProductResponseDto productResponseDto = new ProductResponseDto();
         
         ProductResonseDto.setNumber(changeProduct.getNumber());
         ProductResonseDto.setName(changeProduct.getName());
         ProductResonseDto.setPrice(changeProduct.getPrice());
         ProductResonseDto.setStock(changeProduct.getStock());
         
         return productResponseDto;
       }  
      
       @Override
       public void deleteProduct(Long number) throws Exception {
         //DAO 로 인덱스 를 넘겨 삭제 로직 수행
         productDAO.deleteProduct(number);
       }
    }
  ```  
 ------------------------------------------------------------------------------- 
   ### 6.10.2 컨트롤러 생성  
   
   :bulb: 컨트롤러 : **비지니스 로직과 클라이언트의 요청을 연결** 하는 역할
   * 컨트롤러는 요청과 응답을 전달하는 역할만 맡는 것이 좋다.  

  #### 1. Controller 클래스 생성  
  
  
   ```java
      @RestController
      @RequestMapping("/product")
      public class ProductController {
        
        private final ProductService productService;
        
        @AutoWired
        public ProductController(ProductService productService) {
            this.productService = productService;
        }
        
        @GetMapping()
        public ResponseEntity<ProductResponseDto> getProduct(Long number) {
            ProductResponseDto productResponseDto = productService.getProduct(number);
          
            return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
        }
        
        @PostMapping()
        public ResponseEntity<ProductResponseDto> createProduct(@RequestBody ProductDto productDto) {
          ProductResponseDto productResponseDto = productService.saveProduct(productDto);
          
          return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
        }
        
        @PutMapping()
        public ResponseEntity<ProductResponseDto> changeProductName(
            @RequestBody ChangeProductNameDto changeProductNameDto) throws Exception {
            ProductResponseDto productResponseDto = productService.changeProductName(
              changeProductNameDto.getNumber();
              changeProductNameDto.getNmae());
            
            return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
        }
        
        @DeleteMapping()
        public ResponseEntity<String> deleteProduct(Long number) throws Excpetion {
          productService.deleteProduct(number);
          
          return ResponseEntity.status(HttpStatus.OK).body("정상적으로 삭제되었습니다.");
        }
      
      }
  ```  
--------------------------------------------------
### 6.10.3 Swagger API를 통한 동작 확인

Swagger 사진을 기반으로 진행되기 때문에 책으로 대체....
