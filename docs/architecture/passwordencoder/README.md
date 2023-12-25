## Password 관리 조건
1. 회원가입할 때 Password를 입력받으면 그 값을 암호화해서 저장해야 한다.
2. 로그인할 때 입력받은 Password와 회원가입할 때의 Password를 비교할 수 있어야 한다.

## 관리 과정
1. 회원가입할 때 Password를 해시함수로 암호화해서 저장한다.
2. 로그인할 때 Password가 들어오면 같은 해시함수로 암호화한다.
3. 저장된 값을 불러와서 2번의 암호화된 값과 비교한다.
4. 동일하면 같은 암호로 인지한다.

### 사용하는 알고리즘, 해시 함수
해시 함수는 암호화는 비교적 쉽지만, 복호화가 거의 불가능한 방식의 알고리즘이다.

## PassEncoder 코드
```java
public interface PasswordEncoder {

	/**
	 * Encode the raw password. Generally, a good encoding algorithm applies a SHA-1 or
	 * greater hash combined with an 8-byte or greater randomly generated salt.
	 */
	String encode(CharSequence rawPassword);

	/**
	 * Verify the encoded password obtained from storage matches the submitted raw
	 * password after it too is encoded. Returns true if the passwords match, false if
	 * they do not. The stored password itself is never decoded.
	 * @param rawPassword the raw password to encode and match
	 * @param encodedPassword the encoded password from storage to compare with
	 * @return true if the raw password, after encoding, matches the encoded password from
	 * storage
	 */
	boolean matches(CharSequence rawPassword, String encodedPassword);

	/**
	 * Returns true if the encoded password should be encoded again for better security,
	 * else false. The default implementation always returns false.
	 * @param encodedPassword the encoded password to check
	 * @return true if the encoded password should be encoded again for better security,
	 * else false.
	 */
	default boolean upgradeEncoding(String encodedPassword) {
		return false;
	}

}
```

## PassEncoder 전략
- NoOpPasswordEncoder
  - 암호화하지 않고 평문으로 사용한다.
  - password가 그대로 노출되기 때문에 현재는 deprecated 되었고 사용하지 않기를 권장.
- BcryptPasswordEncoder
  - 해시 함수를 사용한 PasswordEncoder 
  - 애초부터 패스워드 저장을 목적으로 설계되다.
  - Password를 무작위로 여러번 시도하여 맞추는 해킹을 방지하기 위해 암호를 확인할 때 의도적으로 느리게 설정되어있다.
  - BcryptPasswordEncoder는 강도를 설정할 수 있는데 **강도가 높을수록 오랜 시간이 걸린다.** 
  - DelegatingPasswordEncoder는 인코딩 전략으로 Bcrypt를 기본 Encoder로 사용한다.
  - 사용 예시: `{bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG`
- StandardPasswordEncoder
  - 사용 예시: `{sha256}97cde38028ad898ebc02e690819fa220e88c62e0699403e94fff291cfffaf8410849f27605abcbc0`
- Pbkdf2PasswordEncoder
  - Pbkdf2는 NIST(National Institute of Standards and Technology, 미국표준기술연구소)에 의해서 승인된 알고리즘이고, 미국 정부 시스템에 서도 사용한다.
  - 사용 예시: `{pbkdf2}5d923b44a6d129f3ddf3e3c8d29412723dcbde72445e8ef6bf3b508fbf17fa4ed4d6b99ca763d8dc`
- ScryptPasswordEncoder
  - Scrypt는 Pbkdf2와 유사하다.
  - 해커가 무작위로 password를 맞추려고 시도할 때 메모리 사용량을 늘리거나 반대로 메모리 사용량을 줄여서  
    느린 공격을 실행할 수밖에 없도록 의도적인 방식을 사용한다.
    - 따라서 공격이 매우 어렵고 Pbkdf2보다 안전하다고 평가받는다.
  - 보안에 아주 민감한 경우에 사용한다.
  - 사용 예시: `{scrypt}$e0801$8bWJaSu2IKSn9Z9kM+TPXfOc/ 9bdYSrN1oD9qfVThWEwdRTnO7re7Ei+fUZRJ68k9lTyuTeUp4of4g24hHnazw==$OAOec05+bXxvuu/ 1qZ6NUR+xQYvYv7BeL1QxwRpY5Pc=`