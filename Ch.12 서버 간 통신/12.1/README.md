## RestTemplate

스프링에서 HTTP 통신 기능을 손쉽게 사용하도록 설계된 템플릿으로, 해당 템플릿을 이용하면 RESTful 원칙을 따르는 서비스를 편리하게 만들 수 있음.

**특징**

-   HTTP 프로토콜의 메서드에 맞는 여러 메서드 제공
-   RESTful 형식을 갖춘 템플릿
-   HTTP 요청 후 JSON, XML, 문자열 등의 다양한 형식으로 응답을 받을 수 있음
-   blocking I/O 기반의 동기 방식 사용
-   다른 API를 호출할 때 HTTP 헤더에 다양한 값 설정 가능

## RestTemplate의 동작 원리
<img width="933" alt="image" src="https://user-images.githubusercontent.com/77332981/232464952-70f9f4c0-ed70-4e81-a83f-85eb59dc129c.png">

-   애플리케이션에서 RestTemplate을 선언하고 URI와 HTTP 메서드, Body 등을 설정
-   외부 API로 요청을 보낼 경우 RestTemplate에서 HttpMessageConverter를 통해 RequestEntity를 요청 메시지로 변환
-   RestTemplate에서 변환된 요청 메시지를 ClientHttpRequestFactory를 통해 ClientHttpRequest로 가져온 후 외부 API로 요청 전송
-   외부에서 요청에 대한 응답을 받으면 RestTemplate은 ResponseErrorHandler로 오류를 확인하고, 오류가 있다면 ClientHttpResponse에서 응답 데이터 처리
-   받은 응답 데이터가 정상적이라면 다시 HttpMessageConverter를 거쳐 자바 객체로 변환해서 애플리케이션으로 반환

## RestTemplate의 대표적인 메서드

| 메서드 | HTTP 형태 | 설명 |
| --- | --- | --- |
| getForObject | GET | GET 형식으로 요청한 결과를 객체로 반환 |
| getForEntity | GET | GET 형식으로 요청한 결과를 ResponseEntity 형식으로 반환 |
| postForLocation | POST | POST 형식으로 요청한 결과를 헤더에 저장된 URI로 반환 |
| postForObject | POST | POST 형식으로 요청한 결과를 객체로 반환 |
| postForEntity | POST | POST 형식으로 요청한 결과를 ResponseEntity 형식으로 반환 |
| delete | DELETE | DELETE 형식으로 요청 |
| put | PUT | PUT 형식으로 요청 |
| patchForObject | PATCH | PATCH 형식으로 요청한 결과를 객체로 반환 |
