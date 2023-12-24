### User
- `UserDetails`를 상속하고 있고, `UserDetails`는 SpringSecurity에서 제공한다.
  - `org.springframework.security.core.userdetails`

### UserService
- 유저와 관리자 등록을 `authority`를 통해 구분하며 등록한다.
  - 유저 등록: `return userRepository.save(new User(username, passwordEncoder.encode(password), "ROLE_USER"));` 
  - 관리자 등록: `return userRepository.save(new User(username, passwordEncoder.encode(password), "ROLE_ADMIN"));`
  - `PassEncoder`: SpringSecurity에서 제공한다.
    - `encode()`: 암호화하는 기능이다.

### SignUpController
- DTO를 어떻게 쓰는지
  - DTO는 계층간에 넘길 때 쓰기 위한 Getter, Setter만 있는 데이터 전송 객체
  - 이 프로젝트에서 쓰인 방식 :
    `userService.signup(userDto.getUsername(), userDto.getPassword());`
  - DTO 자체를 전달하는 방식이 아니라 DTO를 쪼개면서 메시지를 전달하고자 하는 객체가 필요한 데이터만 깔끔하게 보내는 게 좋다.
    - 다만, 컨트롤러는 뷰로부터 DTO 자체를 전달받았다. -> 뷰에서 데이터를 보내는 방식이므로 특수한 경우다.