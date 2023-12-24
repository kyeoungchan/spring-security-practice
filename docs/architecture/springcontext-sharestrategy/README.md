## MODE_THREADLOCAL
- `ThreadLocalSecurityContextHolderStrategy`를 사용합니다.
- `ThreadLocal`을 사용하여 같은 Thread 안에서 1SecurityContext`를 공유합니다.
- **기본 설정 모드**입니다.
```java
final class ThreadLocalSecurityContextHolderStrategy implements SecurityContextHolderStrategy {

	private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<>();

	@Override
	public void clearContext() {
		contextHolder.remove();
	}

	@Override
	public SecurityContext getContext() {
		SecurityContext ctx = contextHolder.get();
		if (ctx == null) {
			ctx = createEmptyContext();
			contextHolder.set(ctx);
		}
		return ctx;
	}

	@Override
	public void setContext(SecurityContext context) {
		Assert.notNull(context, "Only non-null SecurityContext instances are permitted");
		contextHolder.set(context);
	}

	@Override
	public SecurityContext createEmptyContext() {
		return new SecurityContextImpl();
	}
}
```

## MODE_INHERITABLETHREADLOCAL
- `InheritableThreadLocalSecurityContextHolderStrategy`를 사용합니다.
- `InheritableThreadLocal`을 사용하여 **자식 Thread까지도** `SecurityContext`를 공유합니다.

## MODE_GLOBAL
- `GlobalSecurityContextHolderStrategy`를 사용합니다.
- `Global`로 설정되어 **애플리케이션 전체**에서 `SecurityContext`를 공유합니다.