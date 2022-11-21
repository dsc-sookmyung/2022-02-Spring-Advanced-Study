# AOP

## 트랜잭션 코드의 분리

### 메소드 분리

- 트랜잭션 경계설정과 비즈니스 로직이 공존하는 메소드
- 비즈니스 로직 담당 코드를 메소드로 추출해 독립시켜보자.

### DI를 이용한 클래스의 분리

- UserService 인터페이스를 만든다.
- UserService의 구현 클래스인 UserServiceTx에 트랜잭션 코드를 분리한다.
- UserService의 구현 클래스인 UserServiceImpl에 비즈니스 로직을 작성한다.

```java
public interface UserService {
	void add(User user);
	void upgradeLevels();
}
```

```java
public class UserServiceImpl implements UserService {
	UserDao userDao;
	MailSender mailSender;

	public void upgradeLevels() {
		List<User> users = userDao.getAll();
		for (User user: users) {
			if (canUpgradeLevel(user)) {
				upgradeLevel(user);
			}
		}
	}
	...	
```

```java
@Setter
public class UserServiceTx implements UserService {
	UserService userService;
	PlatformTransactionManager transactionManager;

	public void add(User user) {
		userService.add(user);
	}

	public void upgradeLevels() {
		TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
		
		try {
			userService.upgradeLevels();
			this.transactionManager.commit(status);
		} catch (RuntimeException e) {
			this.transactionManager.rollback(status);
			throw e;
		}
	}
}
```

> 트랜잭션 경계설정 코드 분리의 장점
> 
1. 비즈니스 로직 담당 UserServiceImpl 코드 작성 시 기술적인 내용에 신경 쓰지 않아도 된다.
2. 비즈니스 로직에 대한 테스트를 손쉽게 만들어낼 수 있다.

## 고립된 단위 테스트

### 복잡한 의존관계 속의 테스트

- UserServiceTest
    - UserService의 의존 관계: UserDao, TransactionManager, MailSender
    - UserService가 아닌 의존 관계들까지 테스트하는 셈이다.

### 테스트 대상 오브젝트 고립시키기

- 테스트 대역을 사용하자.
- UserService의 의존 관계: MockUserDao, MockMailSender

- UserServiceTest의 upgradeLevels()에 적용하기

```java
@Test
public void upgradeLevels() throws Exception {
	UserServiceImpl userServiceImpl = new UserServiceImpl();

	MockUserDao mockUserDao = new MockUserDao(this.users);
	userService.Impl.setUserDao(mockUserDao);

	MockMailSender mockMailSender = new MockMailSender();
	userServiceImpl.setMailSender(mockMailSender);

	userService.upgradeLevels();

	List<User> updated = mockUserDao.getUpdated();
	assertThat(updated.size(), is(2));
	checkUserAndLevel(updated.get(0), "joytouch", Level.SILVER);
	checkUserAndLevel(updated.get(1), "madnite1", Level.GOLD);

	List<String> request = mockMailSender.getRequests();
	assertThat(request.size(), is(2));
	assertThat(request.get(0), is(users.get(1).getEmail()));
	assertThat(request.get(1), is(users.get(3).getEmail()));
}

private void checkUserAndLevel(User updated, String expectedId, Level expectedLevel) {
	assertThat(updated.getId(), is(expectedId);
	assertThat(updated.getLevel(), is(expectedLevel);
}
```

```java
static class MockUserDao implements UserDao {
	private List<User> users;
	private List<User> updated = new ArrayList();

	private MockUserDao(List<User> users) {
		this.users = users;
	}

	public List<User> getUpdated() {
		return this.updated;
	}

	public List<User> getAll() {
		return this.users;
	}

	public void update(User user) {
		updated.add(user);
	}

	// 테스트에 사용되지 않는 메소드
	public void add(User user) { throw new UnsupportedOperationException(); }
	public void deleteAll() { throw new UnsupportedOperationException(); }
	public User get(String id) { throw new UnsupportedOperationException(); }
	public int getCount() { throw new UnsupportedOperationException(); }
```

### 목 프레임워크

- Mockito 프레임워크
    
    ```java
    @Test public void mockUpgradeLevels() throws Exception {
    	UserServiceImpl userServiceImpl = new UserServiceImpl();
    
    	UserDao mockUserDao = mock(UserDao.class);
    	when(mockUserDao.getAll()).thenReturn(this.users);
    	userServiceImpl.setUserDao(mockUserDao);
    
    	MailSender mockMailSender = mock(MailSender.class);
    	userServiceImpl.setMailSender(mockMailSender);
    
    	userServiceImpl.upgradeLevels();
    
    	
    	verify(mockUserDao, times(2)).update(any(User.class));
    	verify(mockUserDao, times(2)).update(any(User.class));
    	verify(mockUserDao).update(users.get(1));
    	assertThat(users.get(1).getLevel(), is(Level.SILVER));
    	verify(mockUserDao).update(users.get(3));
    	assertThat(users.get(3).getLevel(), is(Level.GOLD));
    
    	ArgumentCaptor<SimpleMailMessage> mailMessageArg = ArgumentCaptor.forClass(SimpleMailMessage.class);
    	List<SimpleMailMessage> mailMessages = mailMessageArg.getAllValues();
    	assertThat(mailMessages.get(0).getTo()[0], is(users.get(1).getEmail()));
    	assertThat(mailMessages.get(1).getTo()[0], is(users.get(3).getEmail()));
    ```
    

## 다이내믹 프록시와 팩토리 빈

### 프록시와 프록시 패턴, 데코레이터 패턴

- 프록시
    
    자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 대리자, 대리인처럼 클라이언트의 요청을 받아 주는 것
    
- 프록시 사용 목적
    1. 클라이언트에 타깃에 접근하는 방법을 제어하기 위해서
    2. 타깃에 부가적인 기능을 부여하기 위해서
- 실체, 타깃
    
    프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트
    

> 데코레이터 패턴
> 

타깃에 부가적인 기능을 런타임 시 다이내믹하게 부여해주기 위해 프록시를 사용하는 패턴

> 프록시 패턴
> 
- 프록시는 클라이언트가 타깃에 접근하는 방식을 변경한다.
- 실제 타깃 오브젝트를 만드는 대신 프록시를 넘겨 오브젝트 생성 시점을 최대한 늦춘다.
- 특별한 상황에서 타깃에 대한 접근권한을 제어하기 위해 사용할 수도 있다.
- 프록시를 쉽게 만들 수 있도록 지원하는 클래스: `java.lang.reflect`

> 프록시 작성의 문제점
> 
- 인터페이스 메소드를 구현하고 위임하는 코드 작성이 번거롭다.

⇒ JDK 다이내믹 프록시가 유용

리플렉션 학습 테스트

```java
public class ReflectionTest {
	@Test
	public void invokeMethod() throws Exception {
		String name = "Spring";

		assertThat(name.length(), is(6));

		Method lengthMethod = String.class.getMethod("length");
		assertThat((Integer)lengthMethod.invoke(name), is(6));
		...
	}
}
```

다이내믹 프록시를 이용해 프록시를 만들어보자.

타깃 클래스, 인터페이스

```java
interface Hello {
	String sayHello(String name);
	String sayHi(String name);
	String sayThankYou(String name);
}
```

```java
public class HelloTarget implements Hello {
	public String sayHello(String name) {
		return "Hello " + name;
	}

	public String sayHi(String name) {
		return "Hi " + name;
	}

	public String sayThankYou(String name) {
		return "Thank you " + name;
	}
}
```

```java
@Test
public void simpleProxy() {
	Hello hello = new HelloTarget();
	assertThat(hello.sayHello("Toby"), is("Hello Toby"));
	...
}
```

프록시 클래스

```java
public class HelloUppercase implements Hello {
	Hello hello;

	public HelloUppercase(Hello hello) {
		this.hello = hello;
	}

	public String sayHello(String name) {
		return hello.sayHello(name).toUpperCase();
	}

	public String sayHi(String name) {
			return hello.sayHi(name).toUpperCase();
	}

	...
}
```

프록시 클래스의 문제점

1. 인터페이스의 모든 메소드를 구현해 위임해야 한다.
2. 리턴 값을 대문자로 바꾸는 기능이 모든 메소드에 중복된다.

다이나믹 프록시를 사용해보자.

InvocationHandler: 모든 요청을 타깃에 위임하면서 리턴 값을 대문자로 바꾸는 부가기능을 가짐

```java
@AllArgsConstructor
public class UppercaseHandler implements InvocationHandler {
	Object target;

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		Object ret = method.invoke(target, args);
		if (ret instanceof String && method.getName().startsWith("say")) {
			return ((String)ret).toUpperCase();
		}
		else {
			return ret;
		}
	}
}
```

프록시 생성

```java
Hello proxiedHello = (Hello)Proxy.newProxyInstance(
	getClass().getClassLoader(),
	new Class[] { Hello.class },
	new UppercaseHandler(new HelloTarget()));
```

### 다이내믹 프록시를 이용한 트랜잭션 부가 기능

트랜잭션 InvocationHandler

```java
@Setter
public class TransactionHandler implements InvocationHandler {
	private Object target;
	private PlatformTransactionManager transactionManager;
	private String pattern;

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		if (method.getName().startsWith(pattern)) {
			return invokeInTransaction(method, args);
		} else {
			return method.invoke(target, args);
		}
	}

	private Object invokeInTransaction(Method method, Object[] args) throws Throwable {
		TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
		try {
			Object ret = method.invoke(target, args);
			this.transactionManager.commit(status);
			return ret;
		} catch (InvocationTargetException e) {
			this.transactionManager.rollback(status);
			throw e.getTargetException();
		}
	}
}
```

```java
@Test
public void upgradeAllOrNothing() throws Exception {
	...
	TransactionHandler txHandler = new TransactionHandler();
	txHandler.setTarget(testUserService);
	txHandler.setTransactionManager(trnasactionManager);
	txHandler.setPattern("upgradeLevels");
	UserService txUserService = (UserService)Proxy.newProxyInstance(
		getClass().getClassLoader(), new Class[] { UserService.class }, txHandler);
	...
}
```

### 다이내믹 프록시를 위한 팩토리 빈

다이내믹 프록시를 스프링 빈으로 등록하는 방법

1. 팩토리 빈: 스프링의 FactoryBean 인터페이스를 구현한다.

트랜잭션 팩토리 빈

```java
@Setter
public class TxProxyFactoryBean implements FactoryBean<Object> {
	Object target;
	PlatformTransactionManager transactionManager;
	String pattern;
	Class<?> serviceInterface;

	public Object getObject() throws Exception {
		TransactionHandler txHandler = new TransactionHandler();
		txHandler.setTarget(target);
		txHandler.setTransactionManager(trnasactionManager);
		txHandler.setPattern(pattern);
		return Proxy.newProxyInstance(
			getClass().getClassLoader(), new Class[] { serviceInterface }, txHandler);
	}

	public Class<?> getObjectType() {
		return serviceInterface;
	}

	public boolean isSingleton() {
		return false;
	}
}
```

```java
@Autowired ApplicationContext context;

@Test
@DirtiesContext
public void upgradeAllOrNothing() throws Exception {
	TestUserService testUserService = new TestUserService(users.get(3).getId());
	testUserService.setUserDao(userDao);
	testUserService.setMailSender(mailSender);

	TxProxyFactoryBean txProxyFactoryBean = context.getBean("$userService", TxProxyFactoryBean.class);
	txProxyFactoryBean.setTarget(testUserService);
	UserService txUserService = (UserService) txProxyFactoryBean.getObject();

	...
}
```

1. 

### 프록시 팩토리 빈 방식의 장점과 한계

장점

1. 프록시 팩토리 빈 재사용
2. 타깃 인터페이스의 구현 클래스를 일일이 만들 필요가 없다.
3. 부가기능 코드의 중복 문제도 해결한다.
4. …

한계

1. 한 번에 여러 개의 클래스에 공통적인 부가기능을 제공하는 것은 불가능하다.
2. 한 타깃에 여러 개의 부가기능을 적용하기 힘들다.
3. TransactionHandlre 오브젝트가 프록시 팩토리 빈 개수만큼 만들어진다.

## 스프링의 프록시 팩토리 빈

### ProxyFactoryBean

스프링에서 프록시를 생성해 빈 오브젝트로 등록하게 해주는 팩토리 빈

⇒ MethodInterceptor 인터페이스를 구현해 InvocationHandler 역할을 수행하도록 한다.

- Advice: 타깃이 필요없는 순수한 부가기능의 오브젝트
- Pointcut: 부가기능 적용 대상 메소드를 선정하는 오브젝트
- Advisor = Advice + Pointcut

### ProxyFactoryBean 적용

스프링이 제공하는 ProxyFactoryBean을 이용하도록 수정해보자.

TransactionAdvice

```java
public class TransactionAdvice implements MethodInterceptor {
	PlatformTransactionManager transactionManager;

	public void setTransactionManager(PlatformTransactionManager transactionManager) {
		this.transactionManager = transactionManager;
	}

	public Object invoke(MethodInvocation invocation) throws Throwable {
		TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
		try {
			Object ret = invocation.proceed();
			this.transactionManager.commit(status);
			return ret;
		} catch (RuntimeException e) {
			this.transactionManager.rollback(status);
			throw e;
		}
	}
}
```
