![header](https://capsule-render.vercel.app/api?type=wave&color=C3E5AE&height=200&section=header&text=Spring&nbsp;Boot&nbsp;Study&fontSize=50&fontColor=000000)

## 6.4 하이버네이트

>  **하이버네이트란?**
- 하이버네이트는 자바의 **ORM 프레임워크**로, JPA 가 정의하는 인터페이스를 구현하고 있는 JPA 구현체 중 하나   
     
     

### &nbsp; 6.4.1 Spring Data JPA   

   - JPA 를 편리하게 사용할 수 있도록 지원하는 스프링 하위 프로젝트 중 하나
   - 하이버네이트에서 자주 사용되는 기능을 더 쉽게 사용할 수 있도록 구현한 라이브러리
   - CRUD 에 필요한 인터페이스를 제공
 
&nbsp; :exclamation: **작동 방식**  

1. 하이버네이트의 EntityManager 를 직접 다루지 않고 **Repository 를 정의해 사용함**으로써, 스프링이 적합한 쿼리를 동적으로 생성하는 방식으로 DB 를 조작 

![images_mmy789_post_01c17c3f-0efa-49d6-9dbf-111bd80ef130_image](https://user-images.githubusercontent.com/78422940/225841993-758932e4-22f5-48c1-b19d-43f512e49aea.png)

개발자가 Repoistory interface 만 선언해주면, Spring Data JPA 가 자동으로 interface 를 구현한 클래스를 생성해준다.

 ### :sparkles: 참고
 
 - Spring Data JPA 가 제공하는 CRUD 에 필요한 interface  


 ![images_mmy789_post_ac6c7f03-d002-42c3-85e2-77ac12627060_image](https://user-images.githubusercontent.com/78422940/225842665-351d5136-d99d-40e9-b1a6-5231b1661e19.png)  
 
 **주요 메서드**
 * save(S): 새로운 엔티티는 저장하고 이미 있는 엔티티는 병합한다.  
 * delete(T): 엔티티 하나를 삭제한다. 내부에서 EntityManager.remove()를 호출한다.
 * findById(ID): 엔티티 하나를 조회한다. 내부에서 EntityManager.find()를 호출한다.
 * getOne(ID): 엔티티를 프록시로 조회한다. 내부에서 EntityManager.getReference()를 호출한다.
 * findAll(...): 모든 엔티티를 조회한다. 정렬(Sort)이나 페이징(Pageable) 조건을 파라미터로 제공 가능하다.
