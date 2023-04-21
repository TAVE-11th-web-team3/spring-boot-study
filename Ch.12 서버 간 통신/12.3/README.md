## 12.3 WebClient란?
###  WebClient
>* **spring WebFlux는 HTTP 요청을 수행하는 클라이언트로 WebClient를 제공합니다.**
>* **WebClient는 리액터(Reactor) 기반으로 동작하는 API입니다.**
>* **리액터 기반이므로 스레드와 동시성 문제를 벗어나 비동기 형식으로 사용할 수 있습니다.**

### 특징
 * **논블로킹(Non-Blocking)** I/O를 지원합니다.
 * **리액티브 스트림**(Reactive Streams)의 **백 프레셔**(Back Pressure)를 지원합니다.
 * 적은 하드웨어 리소스로 **동시성**을 지원합니다.
 * **함수형 API**를 지원합니다.
 * **동기**, **비동기** 상호작용을 지원합니다.
 * **스트리밍**을 지원합니다.
