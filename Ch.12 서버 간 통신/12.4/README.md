## 12.4 WebClient 사용하기
___
###  WebClient 구현

* **WebClient**를 **생성하는 방법**은 다음과 같이 크게 **두 가지**가 있습니다.
  * **create()** 메서드를 이용한 생성
  * **builder()** 를 이용한 생성

### GET 요청 예제
```java
@Service
public class WebClientService {

    public String getName() {
        
        //  builder()를 활용해 WebClient 생성
        WebClient webClient = WebClient.builder()
            .baseUrl("http://localhost:9090") // 기본 URL 설정
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE) // 헤더의 값 설정
            .build();

        // 실제 요청 코드
        return webClient.get()          // HTTP 메서드 설정 : GET
            .uri("/api/v1/crud-api")    // URI 확장
            .retrieve()                 // 요청에 대한 응답을 받았을 때 그 값을 추출하는 방법 중 하나입니다.
            .bodyToMono(String.class)   // retrieve()는 bodyToMono() 메서드를 통해 리턴 타입을 설정합니다.
            .block();                   // WebClient는 기본적으로 논블로킹 방식으로 동작, 블로킹으로 변경하는 코드
    }
    
    public String getNameWithPathVariable() {
        
        // create()를 이용해 WebClient 생성
        WebClient webClient = WebClient.create("http://localhost:9090");

        ResponseEntity<String> responseEntity = webClient.get()
            .uri(uriBuilder -> uriBuilder.path("/api/v1/crud-api/{name}") // uriBuilder을 사용해 path를 설정
                    . build("Flature"))  // build() 메서드에 추가할 값을 넣습니다.
            .retrieve().toEntity(String.class).block(); 
        
        // 위 코드를 더 간략하게 쓰는 방법
        ResponseEntity<String> responseEntity1 = webClient.get()
            .uri("/api/v1/crud-api/{name}", "Flature")  // 
            .retrieve()
            .toEntity(String.class) // toEntity()를 사용하면 ResponseEntity 타입으로 응답을 전달받을 수 있습니다.
                .block();

        return responseEntity.getBody();
    }

    // 쿼리 파라미터를 함께 전달하는 방법
    public String getNameWithParameter() {

      // create()를 이용해 WebClient 생성
      WebClient webClient = WebClient.create("http://localhost:9090");
        
        return webClient.get().uri(uriBuilder -> uriBuilder.path("/api/v1/crud-api")  
                .queryParam("name", "Flature") // uriBuilder를 사용하여 queryParam()메서드로 전달하려는 값을 설정합니다.
                .build())
            .exchangeToMono(clientResponse -> { // retrieve() 대신 exchange() 사용
                // exchange()를 사용하면 결과값에 따라 분기를 만들어 상황에 따라 값을 다르게 전달할 수 있습니다.
                if (clientResponse.statusCode().equals(HttpStatus.OK)) { 
                    return clientResponse.bodyToMono(String.class);
                } else {
                    return clientResponse.createException().flatMap(Mono::error);
                }
            })
            .block();
    }

}

```
**builder()를 사용할 경우 확장할 수 있는 메서드**
  * **defaultHeader()** : WebClient의 기본 헤더 설정
  * **defaultCookie()** : WebClient의 기본 쿠키 설정
  * **defaultUriVariable()** : WebClient의 기본 URI 확장값 설정
  * **filter()** : WebClient에서 발생하는 요청에 대한 필터 설정

**WebClient 복제**
  * 일단 빌드된 WebClent는 변경할 수 없습니다. 다만 다음과 같이 복사해서 사용할 수 있습니다.
```java
public class webClientService {
  public void cloneWebClient() {
    WebClient webClient = WebClient.create("http://localhost:9090");
    WebClient clone = webClient.mutate().build(); // webClient 복제
  }
}
```
### POST 요청 예제
```java
@Service
public class WebClientService {
    
  public ResponseEntity<MemberDto> postWithParamAndBody() {
    WebClient webClient = WebClient.builder()
            .baseUrl("http://localhost:9090")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();

    // HTTP 바디에 담을 값 생성
    MemberDto memberDTO = new MemberDto();
    memberDTO.setName("flature!!");
    memberDTO.setEmail("flature@gmail.com");
    memberDTO.setOrganization("Around Hub Studio");

    // post() : POST 메서드 통신 정의 
    return webClient.post().uri(uriBuilder -> uriBuilder.path("/api/v1/crud-api")
                    .queryParam("name", "Flature")                  // 파라미터1 정의
                    .queryParam("email", "flature@wikibooks.co.kr") // 파라미터2 정의
                    .queryParam("organization", "Wikibooks")        // 파라미터3 정의
                    .build())
            .bodyValue(memberDTO)       // HTTP 바디 값을 설정 일반적으로 데이터 객체 (DTO, VO )를 전달합니다.
            .retrieve()
            .toEntity(MemberDto.class)
            .block();
  }

  // 커스텀 헤더를 추가하는 방법
  public ResponseEntity<MemberDto> postWithHeader() {
    WebClient webClient = WebClient.builder()
            .baseUrl("http://localhost:9090")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();

    MemberDto memberDTO = new MemberDto();
    memberDTO.setName("flature!!");
    memberDTO.setEmail("flature@gmail.com");
    memberDTO.setOrganization("Around Hub Studio");

    return webClient
            .post() // POST 메서드 통신 정의
            .uri(uriBuilder -> uriBuilder.path("/api/v1/crud-api/add-header")   // path 설정
                    .build())
            .bodyValue(memberDTO)   // HTTP 바디 값을 설정
            .header("my-header", "Wikibooks API")   // header() 메서드를 사용해 헤더에 값을 추가합니다.
            // 일반적으로 임의로 추가한 헤더에는 외부 API를 사용하기 위해 인증된 토큰값을 전달합니다. 
            .retrieve()
            .toEntity(MemberDto.class)
            .block();
  }
}    
```