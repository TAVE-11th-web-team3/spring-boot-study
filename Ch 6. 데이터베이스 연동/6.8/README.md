# 리포지토리 인터페이스 설계
Spring Data JPA는 JpaRepository를 기반으로 더욱 쉽게 DB를 사용할 수 있는 아키텍처를 제공하기 때문에, 스프링 부트로 
JpaRepository를 상속받는 인터페이스를 생성하면 기존의 다양한 메서드를 손쉽게 활용할 수 있다.

JpaRepository를 상속받는 인터페이스를 생성하면 기존의 다양한 메서드를 손쉽게 활용할 수 있다.
## 리포지토리 인터페이스 생성
엔티티를 DB 테이블과 구조를 생성하는 데 사용했다면, Spring Data JPA가 제공하는 인터페이스인 리포지토리(Repository)는 엔티티가 생성한 DB에 접근하는 데 사용된다.

리포지토리를 생성하기 위해서는 접근하려는 테이블과 매핑되는 엔티티에 대한 인터페이스를 생성하고, JpaRepository를 상속받으면 된다.	
```java
public interface ProductRepository extends JpaRepository<Product, Long> {
}
```
JpaRepository를 상속받을 때는 대상 엔티티와 기본값 타입을 지정해야 하는데, 위와 같이 대상 엔티티를 Product로 설정하고 해당 엔티티의 `@Id`의 필드 타입인 Long을 설정하면 된다.

생성된 리포지토리는 JpaRepository를 상속받기 때문에 별도의 메서드 구현 없이도 많은 기능을 제공한다.
```java
@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {

    @Override
    List<T> findAll();

    @Override
    List<T> findAll(Sort sort);
 
    @Override
    List<T> findAllById(Iterable<ID> ids);

    @Override
    <S extends T> List<S> saveAll(Iterable<S> entities);

    void flush();
 
    <S extends T> S saveAndFlush(S entity);

    <S extends T> List<S> saveAllAndFlush(Iterable<S> entities);

    void deleteAllInBatch(Iterable<T> entities);

    void deleteAllByIdInBatch(Iterable<ID> ids);

    void deleteAllInBatch();

    T getReferenceById(ID id);

    @Override
    <S extends T> List<S> findAll(Example<S> example);

    @Override
    <S extends T> List<S> findAll(Example<S> example, Sort sort);
}
```


## 리포지토리 메서드 생성 규칙
리포지토리에서는 몇 가지 명명규칙에 따라 커스텀 메서드도 생성할 수 있다.

일반적으로 CRUD에서 따로 생성해서 사용하는 메서드는 대부분 Read에 해당하는 Select 쿼리인데, 이는 엔티티를 저장/갱신/삭제할 때는 일반적으로 별도의 규칙이 필요하지 않고 리포지토리에서 기본으로 제공하는 조회 메서드는 기본값으로 단일 조회하거나 전체 엔티티를 조회하는 것만 지원하고 있기 때문에 필요에 따라 다른 조회 메서드가 필요하기 때문이다.

메서드에 이름을 붙일 때는 첫 단어를 제외한 이후 단어들의 첫 글자를 대문자로 설정해야 JPA에서 정상적으로 인식하고 쿼리를 자동으로 만들어줄 수 있다.
조회 메서드에 조건으로 붙일 수 있는 몇 가지 기능들은 다음과 같다.(자세한 쿼리 메서드는 7장에서 다룸)

**FindBy**
- SQL문의 where 절 역할을 수행하는 구문으로, 해당 키워드 뒤에 엔티티의 필드값을 입력해서 사용

**AND, OR**
- 조건을 여러 개 설정하기 위해 사용

**Like/NotLike**
- SQL문의 like와 동일한 기능을 수행하며, 특정 문자를 포함하는지 여부를 조건으로 추가

**StartsWith/StartingWith**
- 특정 키워드로 시작하는 문자열 조건 설정

**EndsWith/EndingWith**
- 특정 키워드로 끝나는 문자열 조건 설정

**IsNull/IsNotNull**
- 레코드 값이 Null이거나 Null이 아닌 값 검색

**True/False**
- Boolean 타입의 레코드를 검색할 때 사용

**Before/After**
- 시간을 기준으로 값 검색

**LessThan/GreaterThan**
- 특정 숫자 값을 기준으로 대소 비교를 할 때 사용

**Between**
- 두 숫자 값 사이의 데이터 조회

**OrderBy**
- SQL문에서 order by와 동일한 기능을 수행

**CountBy**
- SQL문의 count와 동일한 기능 수행하며, 결괏값의 개수 추출
