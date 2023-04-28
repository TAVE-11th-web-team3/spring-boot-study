# 스프링 시큐리티 동작 구조
스프링 시큐리티는 서블릿 필터를 기반으로 동작하며, DispatcherServlet 앞에 필터가 배치되어 있음.

<img width="594" alt="image" src="https://user-images.githubusercontent.com/77332981/235070211-212a4dd5-8718-4bb7-929f-93f8410af5c8.png">

클라이언트에서 애플리케이션으로 요청을 보내면 서블릿 컨테이너는 URI를 확인해서 필터와 서블릿을 매핑

<img width="594" alt="image" src="https://user-images.githubusercontent.com/77332981/235070277-3db8a537-7e9d-4287-91d1-5c997694c482.png">

이때 스프링 시큐리티는 사용하고자 하는 필터체인을 서블릿 컨테이너의 필터 사이에서 동작시키기 위해 DelegatingFilterProxy를 사용하는데, 이는 **서블릿 컨테이너의 생명주기와 스프링 애플리케이션 컨텍스트 사이에서 다리 역할을 수행하는 필터 구현체**로 역할을 위임할 필터체인 프록시(스프링부트의 자동 설정에 의해 자동 생성됨)를 내부에 가지고 있음.

보안필터 체인에서 사용하는 여러 종류의 필터는 각 필터마다 실행되는 순서가 다르며, WebSecurityConfigurerAdapter 클래스를 상속받아 설정 가능

중요한 점은 WebSecurityConfigurerAdapter를 상속받는 클래스를 여러 개 생성하여 여러 개의 보안 필터체인을 만들 경우, @Order 애노테이션을 통해 순서를 정의하는 것이 중요한데 이는 WebSecurityConfigurerAdapter 클래스에 **이미 지정되어있는 우선순위가 같을 경우 예외가 발생**하기 때문

별도의 설정이 없다면 스프링 시큐리티에서는 UsernamePasswordAuthenticationFilter를 통해 인증 처리

<img width="593" alt="image" src="https://user-images.githubusercontent.com/77332981/235070346-9b23c9a2-090f-43cd-a65f-7640f51c5110.png">

**인증 수행 과정**

1. 클라이언트로부터 요청을 받으면 서블릿 필터에서 SecurityFilterChain으로 작업이 위임되고 그 중 UsernamePasswordAuthenticationFilter에서 인증 처리
2. AuthenticationFilter는 HttpServletRequest에서 username과 password를 추출해서 토큰 생성 후 AuthenticationManager에게 토큰 전달
3. AuthenticationManager의 구현체인 ProviderManager는 인증을 위해 AuthenticationProvider로 토큰 전달
4. AuthenticationProvider는 토큰 정보를 UserDetailService에 전달
5. UserDetailService는 전달받은 정보를 통해 데이터베이스에서 일치하는 사용자를 찾아 UserDetails 객체 생성
6. 생성된 객체는 AuthenticationProvider로 전달되며, 해당 Provider에서 인증을 수행하고 성공하게 되면 ProviderManager로 권한을 담은 토큰 전달
7. ProviderManager는 검증된 토큰을 AuthenticationFilter로 전달
8. AuthenticationFilter는 검증된 토큰을 SecurityContextHolder에 있는 SecurityContext에 저장
