# SpringSecurity Filter
Spring Security의 동작은 사실상 Filter들로 동작한다.

## 쓰이는 필터들의 종류
- SecurityContextPersistenceFilter 
- BasicAuthenticationFilter
- UsernamePasswordAuthenticationFilter
- CsrfFilter
- RememberMeAuthenticationFilter
- AnonymousAuthenticationFilter
- FilterSecurityInterceptor
- ExceptionTranslationFilter

## FilterOrderRegistration
- Filter들의 순서를 정의해놓은 곳이다.
- code
```java
final class FilterOrderRegistration {

    private static final int INITIAL_ORDER = 100;

    private static final int ORDER_STEP = 100;

    private final Map<String, Integer> filterToOrder = new HashMap<>();

    FilterOrderRegistration() {
        Step order = new Step(INITIAL_ORDER, ORDER_STEP);
        put(ChannelProcessingFilter.class, order.next());
        order.next(); // gh-8105
        put(WebAsyncManagerIntegrationFilter.class, order.next());
        put(SecurityContextPersistenceFilter.class, order.next());
        put(HeaderWriterFilter.class, order.next());
        put(CorsFilter.class, order.next());
        put(CsrfFilter.class, order.next());
        put(LogoutFilter.class, order.next());
        //...
    }
    // ...
}
```
- `Filter`들의 `ORDER_STEP`이 100인 이유: 100이라는 공백 사이사이에 커스텀 필터를 넣을 수 있도록 한 것이다.

### `SecurityContextPersistenceFilter`
- 보통 두번째로 실행되는 필터. 여기에서는 세 번째로 적용
  1. `ChannelProcessingFilter`: 웹 요청이 어떤 프로토콜로 (http 또는 https) 전달되어야 하는지 처리한다.
  2. `WebAsyncManagerIntegrationFilter`: Async 요청에 대해서도 SecurityContext를 처리할수 있도록해준다.
- `SecurityContext`를 찾아와서 `SecurityContextHolder`에 넣어주는 역할을 한다.
  - 없으면 새로 하나 만들어준다.
- HttpSession
  - 보통은 `SecurityContext`를 HttpSession에서 가져온다.
  - JSESSIONID
    - Session ID를 쿠키로 가지고 있기 위한 key

### `BasicAuthenticationFilter`
- `SpringSecurityConfig`에 `configure(HttpSecurity http)`에서 `BasicAuthentication`이 비활성화되어있다.
  - `http.httpBasic().disable(); // basic authentication filter 비활성화`
- 이것을 활성화시키기 전후로 터미널에 다음의 명령어로 개인노트 페이지 요청을 해보자.
  - `curl -u user:user -L http://localhost:8080/note`
- `BasicAuthentication` 활성화 코드
  - `http.httpBasic()`
- 위 실험 결과 `BasicAuthentication`이 비활성화됐을 때는 개인노트 페이지 요청이 안 되고 로그인 페이지로 리다이렉트되지만, 활성화됐을 때는 요청한 페이지가 나온다.
  - 이처럼 로그인 데이터를 Base64로 인코딩해서 모든 요청에 포함해서 보내면 `BasicAuthenticationFilter`는 인증을 한다.
    - Base64란: Base64 is a group of similar binary-to-text encoding schemes that **represent binary data in an ASCII string format** by translating it into a radix-64 representation.
  - 세션이 필요없고 요청이 올 때마다 인증이 이루어진다.(stateless)
  - 보안에 취약하다.
  - 따라서 `BasicAuthenticationFilter`를 사용할 때는 반드시 https를 사용하도록 권장한다.
  - **`BasicAuthenticationFilter`를 사용하지 않을 것이라면 명시적으로 disable시켜주는 것이 좋다.**

### `UsernamePasswordAuthenticationFilter`
- Form 데이터로 username, password 기반의 인증을 담당하고 있는 필터다.
- `UsernamePasswordAuthenticationFilter` => `ProviderManager(AuthenticationManager)` => `AbstractUserDetailsAuthenticationProvider` => `DaoAuthenticationProvider` => `UserDetailsService`

**`ProviderManager(AuthenticationManager)`**
- 인자로 받은 `authentication`이 유효한지 확인하고 `authentication`을 반환
- 인증하면서 계정에 문제가 있는 것이 발견되면 `AuthenticationException` throw
- `AuthenticationManager`를 구현한 Class가 `ProviderManager`
  - Password가 일치하는지, 계정이 활성화되어있는지 확인 후 authentication 반환

**`DaoAuthenticationProvider(AbstractUserDetailsAuthenticationProvider)`**
- 유저정보를 가져오는 `Provider`
- `AbstractUserDetailsAuthenticationProvider`를 `DaoAuthenticationProvider`가 구현한다.
- 사실상 우리가 구현한 `UserDetailService`를 불러오는게 거의 전부다.

### `CsrfFilter`
- Csrf Attack을 방어하는 Filter
- [csrf 관련 간단하게 기록한 링크](https://github.com/kyeoungchan/spring-security-practice/tree/main/src/main/java/me/benny/practice/spring/security/config)
- CsrfFilter는 Csrf Token을 사용하여 위조된 페이지의 악의적인 공격을 방어한다.
  - Thymeleaf는 페이지를 만들 때 자동으로 Csrf Token을 hidden으로 넣어준다.
    - `<input type="hidden" name="_csrf" value="594af42a-63e9-4ef9-aeb2-3687f12cdf43"/>`
- 자동으로 활성화되는 Filter지만, 명시적으로 On하기 위해 `http.csrf()` 코드를 추가한다.(위 링크 참고)
  - Off하고 싶다면 `http.csrf().disable();`

### `RememberMeAuthenticationFilter`
- `RememberMeAuthenticationFilter`는 일반적인 세션보다 훨씬 오랫동안 로그인 사실을 기억할 수 있도록 해준다.
- `Session`의 세션 만료 시간은 기본 설정이 30분이지만 `RememberMeAuthenticationFilter`의 기본 설정은 2주
- `http.rememberMe();`를 통해 `RememberMeAuthenticationFilter` On
- thymeleaf 코드
  ```html
  <div>
    <span>로그인 유지하기</span>
    <input type="checkbox" id="remember-me" name="remember-me" class="form-check-input mt-0" autocomplete="off">
  </div>
  ```

### `AnonymousAuthenticationFilter`
- 인증이 안된 유저가 요청을 하면 Anonymous(익명) 유저로 만들어 `Authentication`에 넣어주는 필터
  - 인증이 안 되어도 무조건 `null`이 아니라 기본 `Authentication`을 만들어주는 개념이다.
- 활성화 코드
  - `http.anonymous().principal("anonymousUser)`

### `FilterSecurityInterceptor`
- 이름은 `Interceptor`로 끝나지만 `Filter` 종류다.
- 앞에서 여러 필터들에서 넘어온 `authentication`의 내용을 기반으로 최종 인가 판단을 내린다.
  - 그렇기 때문에 대부분의 경우 필터 중 제일 뒤쪽에 위치한다.
- 전달 받은 `authentication` 기반 판단
  - 인증에 문제가 없으면 해당 인증으로 인가를 판단 => 정상적으로 필터가 종료
  - 인증에 문제가 있으면 => `AuthenticationException` 발생
  - 인가가 거절된다면 => `AccessDeniedException` 발생
- 동작 순서
  - `FilterSecurityInterceptor.doFilter()`
  - => `AbstractSecurityInterceptor.beforeInvocation()`
  - => `AbstractSecurityInterceptor.authenticateIfRequired()`
    - 인증에 문제가 있으면 `AuthenticationException`
  - => `AbstractSecurityInterceptor.attemptAuthorization()`
    - 인가에 문제가 있으면 `AccessDeniedException`

### `ExceptionTranslationFilter`
- `FilterSecurityInterceptor`에서 발생할 수 있는 두 가지 `Exception`을 처리해주는 필터
- 현재 이 프로젝트의 경우
  - `AuthenticationException` 또는 `Anonymous`의 `AccessDeniedException` 발생 => 로그인 페이지로 리다이렉트
  - `AccessDeniedException` 발생 => 403 Forbidden Whitelabel Error Page로 이동