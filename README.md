### 개인 보안 노트 서비스 만들기
#### main branch 
* Spring Security 단독 Project
#### jwt branch
* Spring Security + JWT Project

## 배운점
### Basic Authentication Filter 비활성화
   `SpringSecurityConfig`에 `http.httpBasic().disable(); // basic authentication filter 비활성화` 부분이 존재한다.
   httpBasic 방식 대신 Jwt를 사용하기 때문에 disable로 설정하는 작업이 필요하다.

- BasicAuthenticationFilter의 특징
  - http에서는 header에 username:password 값이 base64로 인코딩(decode 하기가 쉽다.)되어  전달 되기 떄문에 보안에 매우 취약하다, 반드시 https 프로토콜에서 사용할 것을 권장한다.
  - 최초 로그인시에만 인증을 처리하고, 이후에는 session에 의존한다. 세션이 만료된 이후라도 브라우저 기반의 앱에서는 장시간 서비스를 로그인 페이지를 거치지 않고 이용할 수 있다.
  - 에러가 나면 401(UnAuthorized) 에러를 내려보낸다.

### DTO를 어떻게 쓰는지
- DTO는 계층간에 넘길 때 쓰기 위한 Getter, Setter만 있는 데이터 전송 객체
- 이 프로젝트에서 쓰인 방식 :
  `userService.signup(userDto.getUsername(), userDto.getPassword());`
- DTO 자체를 전달하는 방식이 아니라 DTO를 쪼개면서 메시지를 전달하고자 하는 객체가 필요한 데이터만 깔끔하게 보내는 게 좋다.
- 다만, 컨트롤러는 뷰로부터 DTO 자체를 전달받았다. -> 뷰에서 데이터를 보내는 방식이므로 특수한 경우다.
