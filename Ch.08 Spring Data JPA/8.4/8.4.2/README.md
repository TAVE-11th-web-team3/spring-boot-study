## **2) 페이징 처리하기**

**페이징이란?** 

페이징이란 데이터베이스의 레코드를 개수로 나눠 페이지를 구분하는 것


**Page와 Pageable** 

**Page** 인터페이스는 **Slice** 인터페이스를 확장하고 있으며 **페이징 처리된 결과**를 담고 있다. **전체 페이지의 총 개수**와 **총 데이터 개수**를 계산하고, 해당 페이지에 보여줄 데이터를 가져온다. 
>  Page 인터페이스는 전체 페이지의 총 개수와 총 데이터 수를 계산하기 위해서 Count 쿼리를 추가로 날린다.

**Pageable** 인터페이스는 **페이징 정보**를 담고 있으며, 페이지 번호, 페이지 크기, 장렬 방식 등을 설정할 수 있다. 일반적으로 컨트롤러에서 파라미터로 전달된다.

**Slice**

전체 페이지 개수와 총 데이터를 계산하지 않고 **다음 Slice 존재 여부만을 파악**하며 **현재 페이지의 데이터**만 가져온다. 

---

**페이징 처리를 위한 쿼리 메서드 예시**

```
Page<Product> findByName(String name, Pageable pageable);
```

```
Slice<Product> findSliceByName(String name, Pageable pageable);
```

**페이징 쿼리 메서드를 호출하는 방법**

```
Page<Product> page = productRepository.findByName("펜", PageRequest.of(0, 2));
```

- 리턴 타입으로 Page 객체를 받아야 하기 떄문에 Page<Product>로 타입을 선언하여 객체를 리턴받는다


- **PageRequest**는 **pageable**의 구현체로 일반적으로 **of** 메서드를 통해 **pageRequest 객체를 생성**한다.

![image](https://user-images.githubusercontent.com/105872347/228638711-cbda6f5a-6573-4afd-9615-b77cc0f669a9.png)


### **하이버네이트 로그**

**Page 사용**

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
        product0_.name=? limit ?
        
        
Hibernate: 
    select
        count(product0_.number) as col_0_0_ 
    from
        product product0_ 
    where
        product0_.name=?
```

```text
select product0_.number as number1_0_, product0_.created_at as created_2_0_, product0_.name as name3_0_, product0_.price as price4_0_, product0_.stock as stock5_0_, product0_.updated_at as updated_6_0_ from product product0_ where product0_.name='펜' limit 2;

select count(product0_.number) as col_0_0_ from product product0_ where product0_.name='펜';
```

**Slice 사용**

```text
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
        product0_.name=? limit ?
```

```text
select product0_.number as number1_0_, product0_.created_at as created_2_0_, product0_.name as name3_0_, product0_.price as price4_0_, product0_.stock as stock5_0_, product0_.updated_at as updated_6_0_ from product product0_ where product0_.name='펜' limit 3;
```

- 다음 **Slice가 존재하는지 파악**하기 위해 쿼리가 limit (**size + 1**)로 나가는 것을 확인할 수 있다.

**페이지를 구성하는 세부적인 값 확인하기**

```java
Page<Product> productPage = productRepository.findByName("펜", PageRequest.of(0, 2));
System.out.println(productPage);
System.out.println(productPage.getContent());
```

**출력**

```
Page 1 of 2 containing com.springboot.jpa.data.entity.Product instances

[Product(number=1, price=1000, stock=100, createdAt=2023-03-30T03:02:38.748945, updatedAt=2023-03-30T03:02:38.748945), Product(number=2, price=0, stock=300, createdAt=2023-03-30T03:02:38.748945, updatedAt=2023-03-30T03:02:38.748945)]
```

---

**Slice 사용 예시**

```java
Slice<Product> slice = productRepository.findSliceByName("펜", PageRequest.of(0, 2));
while (slice.hasNext()) {

    List<Product> products = slice.getContent();

	slice = productRepository.findSliceByName("펜", slice.nextPageable());

}
```

**조회한 데이터를 DTO로 변환하기** 

```java
Slice<ProductDto> productDtoSlice = slice.map(product -> new ProductDto(product.getName(), product.getPrice(), product.getStock()));
```