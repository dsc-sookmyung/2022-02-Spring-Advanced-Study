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

**타깃 클래스의 메소드에 대해 포인트컷 선정 여부를 검사하는 헬퍼 메소드**

```java
public void targetClassPointcutMatches(String expr, boolean... expected) throws Exception {
	pointcutMatches(expr, expected[0], Target.class, "hello");
	pointcutMatches(expr, expected[1], Target.class, "hello", String.class);
	pointcutMatches(expr, expected[2], Target.class, "plus", int.class, int.class);
	pointcutMatches(expr, expected[3], Target.class, "minus", int.class, int.class);
	pointcutMatches(expr, expected[4], Target.class, "method");
	pointcutMatches(expr, expected[5], Bean.class, "method");
}
```

**포인트컷 표현식 테스트**

```java
@Test
public void pointcut() rhwos Exception {
	targetClassPointcutMatches("execution(* *(..))", true, true, true, true, true, true);
	// 나머지는 생략 - 표 6-1의 내용과 동일하다.
}
```

`execution()` 외에도 다양한 포인트컷 표현식

- `bean()` : 빈의 이름으로 비교
- `@annotation()` : 어노테이션이 적용된 메소드 선정

포인트컷 표현식을 사용해 빈을 설정해보자.

```java
<bean id="transactionPointcut"
	class="org.springframework.aop.aspectj.AspectJExpressionPointcut">
	<property name="expression" value="execution(* *..*ServiceImpl.upgrade*(..))" />
</bean>
```

`TestUserServiceImpl` → `TestUserService` 로 이름을 바꿔보자.

포인트컷 표현식의 클래스 이름에 적용되는 패턴 = 타입 패턴

`*..*ServiceImpl` 타입 패턴: `TestUserService`는 `UserServiceImpl` 타입이므로 이 조건을 충족한다.

### AOP란 무엇인가?

`UserService` 에 트랜잭션을 적용해온 과정

1. 트랜잭션 경계설정 코드를 비즈니스 로직에 담았다.

→ 특정 트랜잭션 기술에 종속되는 코드

1. 서비스 추상화 기법을 적용한다.

→ 트랜잭션 적용이라는 추상적인 작업 내용은 유지하고, 구체적인 구현 방법을 자유롭게 바꿀 수 있다.

→ 여전히 트랜잭션을 적용하고 있다는 사실이 비즈니스 로직에 드러나 있다.

1. DI를 이용해 데코레이터 패턴을 적용한다.

→ 비즈니스 로직 코드에 전혀 영향을 주지 않고 트랜잭션이라는 부가기능을 부여할 수 있다.

→ 모든 메소드마다 트랜잭션 기능을 부여하는 코드를 넣어 프록시 클래스를 만드는 작업이 힘들어졌다.

1. 프록시 클래스 없이도 프록시 오브젝트를 런타임 시에 만들어주는 JDK 다이내믹 프록시 기술을 적용한다.

→ 프록시 클래스 코드 작성 부담이 줄어든다.

→ 부가 기능 부여 코드의 중복 문제를 해결한다.

→ 동일한 기능의 프록시를 여러 오브젝트에 적용할 때 오브젝트 단위의 중복이 일어난다.

1. 스프링의 프록시 팩토리 빈을 이용해 다이내믹 프록시 생성 방법에 DI를 도입한다.

→ 부가기능을 담은 어드바이스와 부가기능 선정 알고리즘을 담은 포인트컷이 프록시에서 분리되어 공유해 사용할 수 있었다.

→ 트랜잭션 적용 대상 빈마다 일일이 프록시 팩토리 빈을 설정해줘야 한다.

1. 스프링 컨테이너의 빈 생성 후처리 기법을 활용해 컨테이너 초기화 시점에서 자동으로 프록시를 만들어주는 방법을 도입한다.

→ 패턴을 이용해 대상을 자동 선정하는 확장된 포인트컷을 사용한다.

**Aspect**

애플리케이션의 핵심기능을 담고 있지는 않지만, 애플리케이션을 구성하는 중요한 한 가지 요소이고, 핵심기능에 부가되어 의미를 갖는 특별한 모듈

**AOP(Aspect Oriented Programming)**

애플리케이션의 핵심 기능에서 부가 기능을 분리해 애스펙트라는 독특한 모듈로 만들어 설계하고 개발하는 방법

= 관점 지향 프로그래밍

### AOP 적용 기술

- 프록시 방식의 AOP
- AOP 프레임워크(AspectJ)

**AspectJ**

클래스가 JVM에 로딩되는 시점을 가로채서 바이트코드를 조작하는 복잡한 방법을 사용한다.

**이러한 복잡한 방법을 사용하는 이유**

1. 컨테이너가 사용되지 않는 환경에서도 AOP를 적용할 수 있어서
2. 프록시보다 훨씬 강력하고 유연한 AOP가 가능해서

보통은 스프링 AOP로도 충분하나, 그 수준을 넘어서는 기능이 필요하면 그 때 AspectJ를 사용한다.

### AOP 용어

**타깃**

 부가기능을 부여할 대상

**어드바이스**

 타깃에게 제공할 부가기능을 담은 모듈

**조인 포인트**

 어드바이스가 적용될 수 있는 위치

**포인트컷**

 어드바이스를 적용할 조인 포인트를 선별하는 작업 또는 그 기능을 정의한 모듈

**프록시**

 클라이언트와 타깃 사이에 투명하게 존재하면서 부가기능을 제공하는 오브젝트

**어드바이저**

 포인트컷과 어드바이스를 하나씩 갖고 있는 오브젝트

**애스펙트**

 AOP의 기본 모듈로 한 개 이상의 포인트 컷과 어드바이스 조합으로 만들어진 싱글톤 형태의 오브젝트

### AOP 네임스페이스

스프링의 프록시 방식 AOP를 적용하려면 최소 4개의 빈을 등록해야 한다.

- 자동 프록시 생성기
- 어드바이스
- 포인트컷
- 어드바이저

스프링은 AOP를 위해 기계적으로 적용하는 빈들을 간편하게 등록할 수 있도록 도와준다.

aop 네임스페이스를 적용해 AOP 관련 빈 설정 변경하기

```java
<aop:config>
	<aop:pointcut id="transactionPointcut"
								expression="execution(* *..*ServiceImpl.upgrade*(..))" />
	<aop:advisor advice-ref="transactionAdvice" pointcut-ref="transactionPointcut" />
</aop:config>
```

---

## 트랜잭션 속성

### 트랜잭션 정의

`TransactionDefinition` 인터페이스의 4가지 속성

1. 트랜잭션 전파
    
     트랜잭션 경계에서 이미 진행 중인 트랜잭션이 있을 때 또는 없을 때 어떻게 동작할 것인가를 결정하는 방식
    
    - PROPAGATION_REQUIRED
    - PROPAGATION_REQUIRES_NEW
    - PROPAGATION_NOT_SUPPORTED
2. 격리수준
    - 기본값: ISOLATION_DEFAULT
3. 제한시간
    - 기본값: 제한시간 없음
4. 읽기전용

TransactionDefinition을 DI 받아 사용하면 TransactionAdvice를 사용하는 모든 트랜잭션의 속성이 한꺼번에 바뀐다.

⇒ 원하는 메소드만 선택해 독자적인 정의를 적용할 수 있는 방법은?

### 트랜잭션 인터셉터와 트랜잭션 속성

`TransactionInterceptor`

트랜잭션 정의를 메소드 이름 패턴으로 다르게 지정할 수 있는 방법을 추가로 제공하는 어드바이스이다.

- Properties 타입의 transactionAttributes 프로퍼티
    - 트랜잭션 속성을 정의한 프로퍼티
    - rollbackOn() 속성으로 예외처리를 가능하게 만든다.
    - 메소드 패턴과 트랜잭션 속성을 키와 값으로 갖는 컬렉션
    - 트랜잭션 속성: `PROPAGATION_NAME`, `ISOLATION_NAME`, `readOnly`, `timeout_NNNN`, `-Exception1`, `+Exception2`

이러한 어드바이스 빈과 속성 정보도 tx 스키마의 전용 태그로 정의할 수 있다.

### 포인트컷과 트랜잭션 속성의 적용 전략

포인트컷 표현식과 트랜잭션 속성 정의 시 따르면 좋은 전략

1. 트랜잭션 포인트컷 표현식은 타입 패턴이나 빈 이름을 사용한다.
2. 공통된 메소드 이름 규칙을 통해 최소한의 트랜잭션 어드바이스와 속성을 정의한다.
    
    ```python
    <tx:advice id="transactionAdvice">
    	<tx:attributes>
    		<tx:method name="get*" read-only="true" />
    		<tx:method name="*" />
    	</tx:attributes>
    </tx:advice>
    		
    ```
    
3. 프록시 방식 AOP는 같은 타깃 오브젝트 내 메소드를 호출할 때 적용되지 않는다.
    - 해결방법
        1. 스프링 API를 이용해 프록시 오브젝트 레퍼런스를 가져온 다음 같은 오브젝트 메소드 호출도 프록시를 이용하도록 강제한다.
        2. AspectJ 등 타깃의 바이트코드를 직접 조작하는 방식의 AOP 기술을 적용한다.
        

### 트랜잭션 속성 적용

트랜잭션 속성과 전략을 UserService에 적용해보자.

**트랜잭션 경계설정의 일원화**

```java
public interface UserService {
	void add(User user);
	User get(String id);
	List<User> getAll();
	void deleteAll();
	void update(User user);
	
	void upgradeLevels();
}
```

```java
public class UserServiceImpl implements UserService {
	UserDao userDao;
	...

	public void deleteAll() { userDao.deleteAll(); }
	public User get(String id) { return userDao.get(id); }
	public List<User> getAll() { return userDao.getAll(); }
	public void update(User user) {userDao.update(user); }
	...
```

⇒ User 관련 데이터 조작은 UserService라는 트랜잭션 경계를 통해 진행할 경우 모두 트랜잭션을 적용할 수 있게 되었다.

**서비스 빈에 적용되는 포인트컷 표현식 등록**

```java
<aop:config>
	<aop:advisor advice-ref="transactionAdvice" pointcut="bean(*Service)" />
</aop:config>
```

**트랜잭션 속성을 가진 트랜잭션 어드바이스 등록**

```java
<bean id="transactionAdvice"
	class="org.springframework.transaction.interceptor.TransactionInterceptor">
	<property name="transactionManager" ref="transactionManager" />
	<property name="transactionAttributes">
		<props>
			<prop key="get*">PROPAGATION_REQUIRED, readOnly</prop>
			<prop key="*">PROPAGATION_REQUIRED</prop>
		</props>
	</property>
</bean>
```

**트랜잭션 속성 테스트**

```java
static class TestUserService extends UserServiceImpl {
	...

	public List<User> getAll() {
		for (User user : super.getAll()) {
			super.update(user); // 읽기 전용 메소드에 쓰기 작업 추가
		}
		return null;
	}
}
```

```java
@Test(expected=TransientDataAccessResourceException.class)
public void readOnlyTransactionAttribute() {
	testUserService.getAll();
}
```

---

## 애노테이션 트랜잭션 속성과 포인트컷

### 트랜잭션 애노테이션

`@Transactional`

- 메소드, 클래스, 인터페이스에 사용
- 트랜잭션 속성 정의 + 포인트컷의 자동등록 역할

`@Transactional` 의 대체 정책

타깃 메소드 → 타깃 클래스 → 선언 메소드 → 선언 타입의 순서에 따라 어노테이션 적용을 차례로 확인하고, 가장 먼저 발견되는 속성정보를 사용한다.

---

## 트랜잭션 지원 테스트

### 선언적 트랜잭션과 트랜잭션 전파 속성

**선언적 트랜잭션**

AOP를 이용해 코드 외부에서 트랜잭션 기능을 부여해주고 속성을 지정할 수 있게 하는 방법

**프로그램에 의한 트랜잭션**

 개별 데이터 기술의 트랜잭션 API를 사용해 직접 코드 안에서 사용하는 방법

### 트랜잭션 동기화와 테스트

**트랜잭션 추상화 기술의 핵심**

1. 트랜잭션 매니저
    
    구체적인 트랜잭션 기술 종류와 상관없이 일관된 트랜잭션 제어
    
2. 트랜잭션 동기화 기술
    
    트랜잭션 전파에 중요한 역할 수행
    

테스트에서는 트랜잭션 매니저 빈을 가져와 직접 사용할 수 있다.

```java
public class UserServiceTest {
	@Autowired
	PlatformTransactionManager transactionManager;
	...

	@Test
	public void transactionSync() {
		userService.deleteAll();

		userService.add(users.get(0));
		userService.add(users.get(1));
}		
```

⇒ 총 3개의 트랜잭션이 만들어진다.

**트랜잭션 매니저를 이용한 테스트용 트랜잭션 제어**

3개의 트랜잭션을 하나로 통합할 수는 없을까?

```java
@Test
public void transactionSync() {
	DefaultTransactionDefinition txDefinition = new DefaultTransactionDefinition();
	TransactionStatus txStatus = transactionManager.getTransaction(txDefinition);

	userService.deleteAll();
	userService.add(users.get(0));
	userService.add(users.get(1));

	transactionManager.commit(txStatus);
}
```

**롤백 테스트**

테스트 내 모든 DB 작업을 한 트랜잭션 안에서 동작하게 하고 테스트가 끝나면 무조건 롤백

```java
@Test
public void transactionSync() {
	DefaultTransactionDefinition txDefinition = new DefaultTransactionDefinition();
	TransactionStatus txStatus = transactionManager.getTransaction(txDefinition);

	try {
		userService.deleteAll();
		userService.add(users.get(0));
		userService.add(users.get(1));
	}
	finally {
		transactionManager.rollback(txStatus);
	}	
}
```

### 테스트를 위한 트랜잭션 어노테이션

`@Transactional`

- 테스트 메소드의 트랜잭션 경계를 자동 설정
- 자동으로 롤백 테스트가 된다.

`@Rollback`

- 테스트 결과를 DB에 반영하고 싶을 때 사용

`@TransactionConfiguration(defaultRollback=false)`

- 메소드보다 큰 클래스 단위로 롤백에 대한 공통 속성을 지정

`@NotTransactional`

- 해당 메소드에서만 트랜잭션이 시작하지 않는다.
- == `Transactional(propagation=Propagation.NEVER)`

**효과적으로 DB를 테스트하는 법**

- 단위 테스트와 통합 테스트는 아예 클래스를 구분해 따로 둔다.
