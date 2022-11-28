# AOP

## 스프링 AOP

 분리한 트랜잭션 코드는 투명한 부가기능 형태로 제공돼야 한다.

### 자동 프록시 생성

**프록시 팩토리 빈 방식의 접근 방법 한계**

1. 부가기능이 타깃 오브젝트마다 새로 만들어진다. → `ProxyFactoryBean` 의 어드바이스를 통해 해결
2. 부가기능 적용이 필요한 타깃 오브젝트마다 거의 비슷한 내용의 `ProxyFactoryBean` 빈 설정 정보를 추가해줘야 한다. → 여전히 문제

**빈 후처리기를 이용한 자동 프록시 생성기**

빈 후처리기는,

- `BeanPostProcessor` 인터페이스를 구현해서 만든다.
- 스프링 빈 오브젝트로 만들어지고 난 후에 빈 오브젝트를 다시 가공할 수 있게 한다.

`**DefaultAdvisorAutoProxyCreator`** 

어드바이저를 이용한 자동 프록시 생성기

**자동 프록시 생성 빈 후처리기:**

- 빈 후처리기 자체를 빈으로 등록해 빈 오브젝트를 생성할 때마다 후처리기에게 보낸다.
- 빈 오브젝트 일부를 프록시로 포장하고, 프록시를 빈으로 대신 등록한다.
- 이 때 적용할 빈을 선정하는 포인트컷을 어드바이저에 담아 빈으로 등록한다.

**확장된 포인트컷 테스트**

**두 가지 기능을 정의한 Pointcut 인터페이스**

```java
public interface Pointcut {
	ClassFilter getClassFilter();
	MethodMatcher getMethodMatcher();
}
```

**테스트**

```java
@Test
public void classNamePointcutAdvisor {
	// 포인트컷 준비
	NameMatchMethodPointcut classMethodPointcut = new NameMatchMethodPointcut() {
		public ClassFilter getClassFilter() {
			return new ClassFilter() {
				public boolean matches(Class<?> clazz) {
					return clazz.getSimpleName().startsWith("HelloT");
				}
			};
		}
	};
	classMethodPointcut.setMappedName("sayH*");

	// 테스트
	checkAdvice(new HelloTarget(), classMethodPointcut, true);
	checkAdvice(new HelloWorld(), classMethodPointcut, false);
}

private void checkAdviced(Object target, Pointcut pointcut, boolean adviced) {
	ProxyFactoryBean pfBean = new ProxyFactoryBean();
	pfBean.setTarget(target);
	pfBean.addAdvisor(new DefaultPointcutAdvisor(pointcut, new UppercaseAdvice()));
	Hello proxiedHello = (Hello) pfBean.getObject();

	if (adviced) {
		assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
		assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
		...
	} else {
		assertThat(proxiedHello.sayHello("Toby"), is("Hello Toby"));
		assertThat(proxiedHello.sayHi("Toby"), is("Hi Toby"));
		...
	}
}
```

### `DefaultAdvisorAutoProxyCreator` 의 적용

만든 포인트컷을 실제로 적용해보자.

메소드 이름만 비교하던 포인트컷인 `NameMatchMethodPointcut` 을 상속해서 프로퍼티로 주어진 이름 패턴을 가지고 클래스 이름을 비교하는 `ClassFilter` 를 추가할 것이다.

```java
public class NameMatchClassMethodPointcut extends NameMatchMethodPointcut {
	public void setMappedClassName(String mappedClassName) {
		this.setClassFilter(new SimpleClassFilter(mappedClassName));
	}

	static class SimpleClassFilter implements ClassFilter {
		String mappedName;

		private SimpleClassFilter(String mappedName) {
			this.mappedName = mappedName;
		}

		public boolean matches(Class<?> clazz) {
			return PatternMatchUtils.simpleMatch(mappedName, clazz.getSimpleName());
		}
	}
}
```

자동 프록시 생성기 `DefaultAdvisorAutoProxyCreator` 를 등록한다.

→ 빈 중에서 `Advisor` 인터페이스를 구현한 것을 모두 찾는다.

```java
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" />
```

새로 만든 클래스 필터 지원 포인트 컷을 빈으로 등록한다.

```java
<bean id="transactionPointcut"
			class="springbook.service.NameMatchClassMethodPointcut">
	<property name="mappedClassName" value="*ServiceImpl"/> => 클래스 이름 패턴
	<property name="mappedName" value="upgrade*" /> => 메소드 이름 패턴
</bean>
```

이제 `transactionAdvisor` 를 명시적으로 DI하는 빈은 존재하지 않는다.

대신 자동 프록시 생성기에 의해 자동 수집되고, 다이내믹하게 DI 되어 동작하는 어드바이저가 된다.

**자동 프록시 생성기를 사용하는 테스트**

`@Autowired` 를 통해 컨텍스트에서 가져오는 `UserService` 타입 오브젝트는 `UserServiceImpl` 오브젝트가 아니라 트랜잭션이 적용된 프록시여야 한다.

`UserServiceTest` 내부에 정의된 `TestUserServiceImpl` 클래스

```java
static class TestUsrServiceImpl extends UserServiceImpl {
	private String id = "madnite1"; // 테스트 픽스처 users(3)의 id 값 고정

	protected void upgradeLevel(User user) {
		if (user.getId().equals(this.id)) throw new TestUserServiceException();
		super.upgradeLevel(user);
	}
}
```

`TestUserServiceImpl` 클래스를 빈으로 등록한다.

```java
<bean id="testUserService"
	class="springbook.user.service.UserServiceTest$TestUserServiceImpl"
	parent="userService" />
```

- $: 스태틱 멤버 클래스 지정 시 사용
- parent: 다른 빈 설정의 내용을 상속받는다.

`upgradeAllOrNothing()` 이 `testUserService`를 사용하도록 수정

```java
public class UserServiceTest {
	@Autowired UserService userService;
	@Autowired UserService testUserService;
	...

	@Test
	public void upgradeAllOrNothing() {
		userDao.deleteAll();
		for(User user : users) userDao.add(user);

		try {
			this.testUserService.upgradeLevels();
			fail("TestUserServiceException expected");
		}
		catch (TestUserServiceException e) {}

		checkLevelUpgraded(users.get(1), false);
	}
}
	
```

**자동생성 프록시 확인**

1. 트랜잭션이 필요한 빈에 트랜잭션 부가기능이 적용됐는가?
    
     ⇒ upgradeAllOrNothing() 테스트로 검증했다.
    
2. 아무 빈에나 트랜잭션 부가기능이 적용된 것은 아닌가?
    
    ⇒ 클래스 필터가 제대로 동작해 프록시 생성 대상을 선별하고 있는지 여부가 궁금하다.
    

클래스 이름 패턴을 변경해 `testUsrService` 빈에 트랜잭션이 적용되지 않게 해보자.

⇒ upgradeAllOrNothing()만 실패하면 성공!

### 포인트컷 표현식을 이용한 포인트컷

메소드 이름보다 더 복잡하고 세밀한 기준으로 선정하고 싶다면? ⇒ **포인트컷 표현식**

`**AspectJExpressionPointcut**`

포인트컷 표현식을 지원하는 포인트컷 클래스

**포인트컷 테스트용 클래스**

```java
public class Target implements TargetInterface {
	public void hello() {}
	public void hello(String a) {}
	public int minus(int a, int b) throws RuntimeException { return 0; }
	public int plus(int a, int b) { return 0; }
	public void method() {}
}

public class Bean {
	public void method() throws RuntimeException {}
}
```

execution() 지시자를 사용한 **포인트컷 표현식 문법**

```java
execution([접근제한자 패턴] 타입패턴 [타입패턴.]이름패턴 (타입패턴 | "..", ...)
[throws 예외 패턴])
```

ex) Target 클래스의 minus()

```java
public int springbook.learningtest.spring.pointcut.Target.minus(int,int) throws java.lang.RuntimeException
```

**포인트컷 표현식 테스트**

```java
@Test
public void methodSignaturePointcut() throws SecurityException, NoSuchMethodException {
	AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
	pointcut.setExpression("execution(public int " +
		"springbook.learningtest.spring.pointcut.Target.minus(int,int) " +
		"throws java.lang.RuntimeException)");

	// Target.minus()
	assertThat(pointcut.getClassFilter().matches(Target.class) && 
		pointcut.getMethodMatcher().matches(
			Target.class.getMethod("minus", int.class, int.class), null), is(true));

	// Target.plus()
	assertThat(pointcut.getClassFilter().matches(Target.class) && 
		pointcut.getMethodMatcher().matches(
			Target.class.getMethod("plus", int.class, int.class), null), is(false));

	...
}
```
