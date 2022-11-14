# 서비스 추상화

## 사용자 레벨 관리 기능 추가

- UserDao에 인터넷 서비스의 사용자를 관리할 수 있는 비즈니스 로직을 추가해보자.

<br>

### 필드 추가

> Level Enum : 사용자의 레벨을 관리하자.

**레벨 필드 추가 방법**

- DB에 `varchar` 타입으로 선언 후, “BASIC”, “SILVER”, “GOLD”라고 넣는다.
    
    ⇒ BAD
    
- 각 레벨을 코드화해서 숫자로 넣는다.
    
    ⇒ BAD (1,2,3 외의 다른 숫자가 들어가는 것을 컴파일러가 체크할 수 없다.)
    
- 자바 5 이상에서 제공하는 enum을 이용한다.
    
    ⇒ GOOD!

<br>

```java
public enum Level {
	BASIC(1), SILVER(2), GOLD(3);
	
	private final int value;

	Level(int value) {
		this.value = value;
	}
	
	public int intValue() {
		return value;
	}

	public static Level valueOf(int value) {
		switch(value) {
			case 1: return BASIC;
			case 2: return SILVER;
			case 3: return GOLD;
			default: throw new AssertionError("Unknown value: " + value);
		}
	}
}

```

<br>

> User 필드 추가


```java
public class User {
	...
	Level level;
	int login;
	int recommend;

	public Level getLevel() {
		return level;
	}

	public void setLevel(Level level) {
		this.level = level;
	}
	...
```

<br>

> UserDaoTest 수정
- 테스트에도 레벨 필드를 추가한다.

> UserDaoJdbc 수정
- 미리 준비한 테스트가 성공하도록 UserDaoJdbc 클래스에 레벨 필드를 추가한다.
- Level enum은 오브젝트 이므로 DB에 저장 가능한 정수형 값으로 변환해줘야 한다.
    
    ```java
    user.getLevel().intValue()
    ```

<br>

<br>

### 사용자 수정 기능 추가

- 비즈니스 로직에 따르면 사용자 정보는 여러 번 수정될 수 있다.

<br>

> 수정 기능 테스트 추가

```java
@Test
public void update() {
	dao.deleteAll();

	dao.add(user1);
	
	user1.setName("오민규");
	user1.setPassword("springno6");
	...
	dao.update(user1);

	User user1update = dao.get(user1.getId());
	checkSameUser(user1, user1update);
}
```

<br>

> UserDao와 UserDaoJdbc 수정
- UserDao 인터페이스에 `update()` 메소드 추가
- UserDaoJdbc의 `update()` 메소드 구현

⇒ 테스트 성공

<br>

> 수정 테스트 보완
- 수정할 로우의 내용이 바뀐 것만 확인할 뿐이지, 수정하면 안되는 내용의 로우가 그대로 남아 있는지는 확인하지 못한다.

<br>

**해결 방법**

1. JdbcTemplate의 `update()`가 돌려주는 리턴 값을 확인한다.
2. 테스트를 보강해 원하는 사용자 외의 정보는 변경되지 않았음을 직접 확인한다. ⇒ 채택!

```java
@Test
public void update() {
	dao.deleteAll();

	dao.add(user1);
	dao.add(user2);
	
	user1.setName("오민규");
	user1.setPassword("springno6");
	...
	dao.update(user1);

	User user1update = dao.get(user1.getId());
	checkSameUser(user1, user1update);
	User user2same = dao.get(user2.getId());
	checkSameUser(user2, user2same);
}
```

<br>

<br>

### UserService.upgradeLevels()

- 레벨 관리 기능
- UserService: 사용자 관리 비즈니스 로직

<br>

> UserService 클래스와 빈 등록

```java
public class UserService {
	UserDao userDao;

	public void setUserDao(UserDao userDao) {
		this.userDao = userDao;
	}
}
```

<br>

- 스프링 설정파일에 빈으로 등록한다.

<br>

> UserServiceTest 테스트 클래스
- userService 빈의 주입을 확인하는 테스트

```java
@Test
public void bean() {
	assertThat(this.userService, is(notNullValue()));
```

> upgradeLevels() 메소드

<br>

```java
public void upgradeLevels() {
	List<User> users = userDao.getAll();
	for(User user : users) {
		Boolean changed = false;
		if (user.getLevel() == Level.BASIC && user.getLogin() >= 50) {
			user.setLevel(Level.SILVER);
			changed = true;
		}
		else if (user.getLevel() == LEVEL.SILVER && user.getRecommend() >= 30) {
			user.setLevel(Level.GOLD);
			changed = true;
		}

		if (changed) {userDao.update(user);}
	}
}
```

> upgradeLevels() 테스트

<br>


```java
class UserServiceTest {
	...
	List<User> users;

	@Before
	public void setUp() {
		users = Arrays.asList(
			new User("asdf", "박범진", "p1", Level.BASIC, 49, 0),
			new User("asd2", "박범규", "p2", Level.BASIC, 50, 0),
			...
		);
	}

	@Test
	public void upgradeLevels() {
		userDao.deleteAll();
		for(User user : users) userDao.add(user);

		userService.upgradeLevels();

		checkLevel(user.get(0), Level.BASIC);
		checkLevel(user.get(1), Level.SILVER);
		...
	}

	private void checkLevel(User user, Level expectedLevel) {
		User userUpdate = userDao.get(user.getId());
		assertThat(userUpdate.getLevel(), is(expectedLevel));
	}
	...
```

<br>

<br>

### UserService.add()

- 처음 가입하는 사용자는 기본적으로 BASIC 레벨이어야 한다.

- add() 테스트

```java
@Test
public void add() {
	userDao.deleteAll();

	User userWithLevel = users.get(4);
	User userWIthoutLevel = users.get(0);
	userWithoutLevel.setLevel(null);

	userService.add(userWithLevel);
	userService.add(userWithoutLevel);

	User userWithLevelRead = userDao.get(userWithLevel.getId());
	User userWithoutLevelRead = userDao.get(userWithoutLevel.getId());

	assertThat(userWithLevelRead.getLevel(), is(userWithLevel.getLevel()));
	assertThat(userWithoutLevelRead.getLevel(), is(Level.BASIC));
```

<br>

- add() 구현

```java
public void add(User user) {
	if (user.getLevel() == null) user.setLevel(Level.BASIC);
	userDao.add(user);
}
```

<br>

<br>

### 코드 개선

- 코드에 중복된 부분은 없는가?
- 코드가 무엇을 하는 것인지 이해하기 불편하지 않은가?
- 코드가 자신이 있어야 할 자리에 있는가?
- 앞으로 변경이 일어난다면 어떤 것이 있을 수 있고, 그 변화에 쉽게 대응할 수 있게 작성되어 있는가?

<br>

> upgradeLevels() 코드의 문제점
1. for 루프 속의 if/ else if / else 블록들이 읽기 불편하다.
2. for 루프 속의 if 문은 레벨 개수에 따라 추가될 수 있다.
3. 레벨과 업그레이드 조건을 동시에 비교한느 부분도 문제가 될 수 있다.

<br>

> 리팩토링하기
1. 기본 작업 흐름만 남겨둔다.

```java
private void upgradeLevels() {
	List<User> users = userDao.getAll();
	for(User user : users) {
		if (canUpgradeLevel(user)) {
			upgradeLevel(user);
		}
	}
}
```

<br>

2. 필요한 메소드를 작성한다.

```java
private boolean canUpgradeLevel(User user) {
	Level currentLevel = user.getLevel();
	switch(currentLevel) {
		case BASIC: return (user.getLogin() >= 50);
		case SILVER: return ...;
		case GOLD: return false;
		default: throw new ILlegalArgumentException("Unkown Level: " + currentLevel);
	}
}
```

<br>

3. upgradeLevel() 메소드를 수정한다.
- Level enum이 업그레이드 순서를 담도록 수정

```java
public enum Level {
	GOLD(3, null), SILVER(2, GOLD), BASIC(1, SILVER);

	private final int value;
	private final Level next;

	Level(int value, Level next) {
		this.value = value;
		this.next = next;
	}
	
	public Level nextLevel() {
		return this.next;
	}
	
	...
}
```

<br>

- User 엔티티에 upgradeLevel() 작성

```java
public void upgradeLevel() {
	Level nextLevel = this.level.nextLevel();
	if (nextLevel == null) {
		throw new IllegalStateException("this.level + "은 업그레이드가 불가능합니다.");
	}
	else {
		this.level = nextLevel;
	}
}
```

<br>

- UserService의 upgradeLevel() 메소드 작성

```java
private void upgradeLevel(User user) {
	user.upgradeLevel();
	userDao.update(user);
}
```

<br>

> User 테스트
- User의 upgradeLevel() 테스트

```java
public class UserTest {
	User user;

	@Before
	public void setUp() {
		user = new User();
	}

	@Test()
	public void upgradeLevel() {
		Level[] levels = Level.values();
		for(Level level : levels) {
			if (level.nextLevel() == null) continue;
			user.setLevel(level);
			user.upgradeLevel();
			assertThat(user.getLevel(), is(level.nextLevel()));
		}
	}

	@Test(expected=IllegalStateException.class)
	public void cannotUpgradeLevel() {
		Level[] levels = Level.values();
		for(Level level : levels) {
			if (level.nextLevel() != null) continue;
			user.setLevel(level);
			user.upgradeLevel();
		}
	}
```

<br>

> UserTest 개선
- 레벨 체크 시 직접 SILVER, BASIC 등을 명시하기 보다는 upgrade 여부를 명시하도록 한다.

```java
class UserServiceTest {
	...

	@Test
	public void upgradeLevels() {
		userDao.deleteAll();
		for(User user : users) userDao.add(user);

		userService.upgradeLevels();

		checkLevelUpgraded(user.get(0), false);
		checkLevelUpgraded(user.get(1), true);
		...
	}

	private void checkLevelUpgraded(User user, boolean upgraded) {
		User userUpdate = userDao.get(user.getId());
		if (upgraded) {
			assertThat(userUpdate.getLevel(), is(user.getLevel().nextLevel()));
		}
		else {
			assertThat(userUpdate.getLevel(), is(user.getLevel()));
		}
	}

	...
```

<br>

또한 반복되는 숫자는 상수를 이용하도록 한다.

<br>

- 테스트에서도 UserService에 정의해둔 상수를 사용하고 싶다면, 다음처럼 정의한다.

```java
import static springbook.user.service.UserService.MIN_LOGCOUNT_FOR_SILVER;
```

<br>

연말 이벤트로 한시적으로 레벨 업그레이트 정책을 변경해야 할 필요가 있을 수 있다.

이 경우, UserService를 변경했다가, 이벤트가 끝나면 다시 돌려놓는 것은 위험한 방법이다.

⇒ 사용자 업그레이드 정책을 UserService에서 분리하는 방법을 고려해보자.

<br>

```java
public interface UserLevelUpgradePolicy {
	boolean canUpgradeLevel(User user);
	void upgradeLevel(User user);
}
```

<br>

- 평상시 정책을 구현한 클래스를 UserService에서 사용한다.
- 이벤트 때에만 새 업그레이드 정책을 담은 클래스를 DI한다.

<br>

<br>


## 트랜잭션 서비스 추상화

레벨 관리 작업 중에 문제가 발생해 중단된다면 그때까지 진행된 변경 작업도 모두 취소시키고 싶다.

<br>

### 모 아니면 도..

현재 모든 사용자의 업그레이드 작업을 진행하다가 예외가 발생해 작업이 중단된다면?

⇒ 테스트를 만들어 어떻게 실행되는지 알아보자!

<br>

> 테스트용 UserService 대역
- 예외를 강제 발생시키도록 애플리케이션 코드를 수정한다. → 테스트용 UserService 사용

```java
static class TestUserService extends UsrService {
	private String id;             // 예외를 발생시킬 User 오브젝트의 id 지정

	private TestUserService(String id) {
		this.id = id;
	}

	protected void upgradeLevel(User user) {
		if (user.getId().equals(this.id)) throw new TestUserServiceException();
		super.upgradeLevel(user);
	}
}
```

<br>

> 강제 예외 발생 테스트

```java
@Test
public void upgradeAllOrNothing() {
	UserService testUserService = new TestUserService(users.get(3).getId());
	testUserService.setUserDao(this.userDao); // userDao를 수동 DI
	userDao.deleteAll();
	for(User user : users) userDao.add(user);

	try {
		testUserService.upgradeLevels();
		fail("TestUserServiceException expected");
	}
	catch(TestUserServiceException e) {}

	checkLevelUpgrade(users.get(1), false);
}
```

테스트 실패 ⇒ 두 번째 사용자의 레벨 업데이트가 그대로 유지되고 있다.

<br>

> 테스트 실패의 원인
- 모든 사용자의 레벨을 업그레이드하는 작업인 `upgradeLevels()` 메소드가 한 트랜잭션 안에서 동작하지 않았기 때문이다.

<br>

<br>

### 트랜잭션 경계 설정

- 여러 개의 SQL이 사용되는 작업을 한 트랜잭션으로 취급해야 하는 경우
    - 계좌이체
    - 여러 사용자에 대한 레벨 수정 작업
- 트랜잭션 롤백
    - 문제 발생시 앞에서 처리한 SQL 작업도 취소
- 트랜잭션 커밋
    - 모든 SQL 수행 작업이 성공적으로 마무리됐다고 DB에 알려 작업을 확정
- 트랜잭션의 경계 설정
    - `setAutoCommit(false)`로 트랜잭션의 시작을 선언하고 `commit()` 또는 `rollback()` 으로 트랜잭션을 종료하는 작업

<br>

> 비즈니스 로직 내의 트랜잭션 경계 설정

결국 트랜잭션의 경계 설정 작업을 UserService로 가져와야 한다. 

트랜잭션의 시작과 종료를 담당하는 최소한의 코드만 가져오게 만들면 된다.

<br>

- Connection을 공유하도록 수정한 UserService 메소드

```java
class UserService {
	public void upgradeLevels() throws Exception {
		Connection c = ...;
		...
		try {
			...
			upgradeLevel(c, user);
			...
		}
		...
	}

	protected v oid upgradeLevel(Connection c, User user) {
		user.upgradeLevel();
		userDao.update(c, user);
	}
}

interface UserDao {
	public update(Connection c, User user);
	...
}
```

<br>

> 이 때의 문제점
1. JdbcTemplate을 더 이상 활용할 수 없다. → JDBC API를 직접 사용하는 초기 방식 사용
2. UserService 메소드에 Connection 파라미터가 추가된다.
3. Connection 파라미터로 인해 더 이상 데이터 액세스 기술에 독립적일 수 없다.
4. Connection 파라미터 추가가 테스트 코드에도 영향을 미친다.

<br>

<br>

### 트랜잭션 동기화

> Connection 파라미터 제거
- 트랜잭션 동기화
    - UserService가 만든 Connection 오브젝트를 특별한 저장소에 보관해둔다.
    - 이후에 호출되는 DAO 메소드에서는 저장된 Connection을 가져다가 사용하게 한다.
    - 작업 스레드마다 독립적으로 Connection 오브젝트를 저장하고 관리된다.
    
    ⇒ 서버의 멀티스레드 환경에서도 충돌 날 염려는 없다.
    
<br>

> 트랜잭션 동기화 적용
- 스프링은 트랜잭션 동기화 기능을 지원하는 간단한 유틸리티 메소드를 제공한다.

<br>

- UserService 수정

```java
private DataSource dataSource;

public void setDataSource(DataSource dataSource) {
	this.dataSource = dataSource;
}

public void upgradeLevels() throws Exception {
	// 트랜잭션 동기화 관리자를 이용해 동기화 작업 초기화
	TransactionSynchronizationManager.initSynchronization();
	Connection c = DataSourceUtils.getConnection(dataSource);
	c.setAutoCommit(false);

	try {
		List<User> users = userDao.getAll();
		for (User user : users) {
			if (canUpgradeLevel(user)) {
				upgradeLevel(user);
			}
		}
		c.commit();
	} catch (Exception e) {
		c.rollback();
		throw e;
	} finally {
		// 스프링 유틸리티 메소드를 이용해 DB 커넥션 안전 종료
		DataSourceUtils.releaseConnection(c, dataSource);
		// 동기화 작업 종료 및 정리
		TransactionSynchronizationManager.unbindResource(this.dataSource);
		TrnasactionSynchronizationManager.clearSynchronization();
	}
}
```

<br>

> 트랜잭션 테스트 보완
- TestUserService에 dataSource를 DI한다. ⇒ 테스트 성공!
- 마지막으로, 스프링 설정 파일에서 userService 빈의 dataSource 프로퍼티를 추가한다.

<br>

### 트랜잭션 서비스 추상화

> 기술과 환경에 종속되는 트랜잭션 경계설정 코드

한 트랜잭션 안에서 여러 DB에 데이터를 넣는 작업이 필요하다.

- JDBC의 Connection을 이용한 트랜잭션 방식 → 로컬 트랜잭션, 불가능
- 별도의 트랜잭션 관리자를 통해 트랜잭션을 관리 → 글로벌 트랜잭션, 가능!
- 자바에서는 글로벌 트랜잭션 매니저를 지원하기 위한 API로 JTA를 제공한다.
    - JTA는 여러 DB나 메시징 서버에 대한 작업을 한 트랜잭션으로 통합해준다.

<br>

- JTA를 이용한 트랜잭션 코드 구조

```java
// JNDI를 이용해 서버의 UserTransaction 오브젝트를 가져온다.
InnitialContext ctx = new InitialContext();
UserTransaction tx = (UserTransaction) ctx.lookup(USER_TX_JNDI_NAME);

tx.begin();
Connection c = dataSource.getConnection();
try {
	// 데이터 액세스 코드
	tx.commit();
} catch (Exception e) {
	tx.rollback();
	throw e;
} finally {
	c.close();
}
```

- 로컬 트랜잭션으로 충분한 고객은 JDBC를 적용한 클래스 제공
- 글로벌 트랜잭션을 필요로 하는 곳은 JTA를 적용한 클래스 제공

⇒ 너무 귀찮다.

<br>

> 트랜잭션 API의 의존관계 문제와 해결책
- JDBC에 종속적인 Connection을 이용한 트랜잭션 코드가 UsreService에 들어가게 되었다.
- 트랜잭션 경계설정 코드는 모두 유사한 구조를 가지므로 추상화하자.

<br>

> 스프링의 트랜잭션 서비스 추상화
- 스프링이 제공하는 트랜잭션 추상화 방법을 UserService에 적용해보자.

```java
public void upgradeLevels() {
	// JDBC 트랜잭션 추상 오브젝트 생성
	PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSoure);
	TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

	try {
		List<User> users = userDao.getAll();
		for (User user : usres) {
			if (canUpgradeLevel(user)) {
				upgradeLevel(user);
			}
		}
		transactionManager.commit(status);
	} catch (RuntimeException e) {
		transactionManager.rollback(status);
		throw e;
	}
}
```

<br>

> 트랜잭션 기술 설정의 분리

어떤 트랜잭션 매니저 구현 클래스를 사용할지 UserService가 알고 있는 것은 DI 원칙에 위배됨.

DataSourceTransactionManager를 빈으로 등록하고 UserService에 DI하자.

<br>

- UserService 클래스 코드 수정

```java
public class UserService {
	...
	private PlatformTransactionManager transactionManager;

	public void setTransactionManager(PlatformTransactionManager transasctionManager) {
		this.transactionManager = transactionManager;
	}

	public void upgradeLevels() {
		TransactionStatus status = this.transactionManager.get...
```

- 스프링 설정파일에 트랜잭션 매니저 빈 등록

<br>

<br>

## 서비스 추상화와 단일 책임 원칙

이제 스프링의 트랜잭션 서비스 추상화 기법으로 다양한 트랜잭션 기술을 일관된 방식으로 제어할 수 있게 되었다.

> 수직, 수평 계층구조와 의존관계
- UserService, UserDao : 애플리케이션 계층
    - UserService: 비즈니스 로직
    - UserDao: 데이터 액세스 로직
- TransactionManager, DataSource: 서비스 추상화 계층
    - DataSource DI → UserDao : DB 연결을 생성하는 방법에 대해 독립적
    - PlatformTransactionManager DI → UserService : 구체적인 트랜잭션 기술에 독립적
- JDBC, JTA, JPA, … : 기술 서비스 계층

<br>

> 단일 책임 원칙

UserService에 Connection을 직접 사용하는 트랜잭션 코드가 있었다면,

- 사용자 레벨 관리
- 트랜잭션 관리

라는 두 가지 책임을 갖게 된다.

<br>

> 단일 책임 원칙의 장점
- 어떤 변경이 필요할 때 수정 대상이 명확해진다.

<br>

**SRP를 지키는 방법**

- 적절하게 책임과 관심이 다른 코드를 분리한다.
- 서로 영향을 주지 않도록 다양한 추상화 기법을 도입한다.
- 애플리케이션 로직과 기술/환경을 분리한다… 등등

<br>

## 메일 서비스 추상화

> 테스트 대역의 종류와 특징
- 테스트 대역
    
    테스트 환경을 만들어주기 위해, 테스트 대상이 되는 오브젝트의 기능에만 충실하게 수행하면서 빠르게, 자주 테스트를 실행할 수 있도록 사용하는 오브젝트들
    
<br>

- 대표적인 예: 테스트 스텁
    - 테스트 대상 오브젝트의 의존객체로서 존재하면서 테스트 동안 코드가 정상적으로 수행할 수 있도록 돕는 것
    - ex) DummyMailSender

<br>

- 목 오브젝트
    - 테스트 대상 오브젝트가 간접적으로 의존 오브젝트에 넘기는 값 자체도 검증하고 싶을 때
    - 테스트 대상의 간접적인 출력 결과를 검증하고, 테스트 대상 오브젝트와 의존 오브젝트 사이에서 일어나는 일을 검증할 수 있도록 설계됨

<br>

> 목 오브젝트를 이용한 테스트

목 오브젝트로 만든 메일 전송 확인용 클래스

```java
static class MockMailSender implements MailSender {
	private List<String> requests = new ArrayList<String>();

	public List<String> getRequests() {
		return requests;
	}

	public void send(SimpleMailMessage mailMessage) throws MailException {
		requests.add(mailMessage.getTo()[0]);
	}

	public void send(SimpleMailMessage[] mailMessage) throws MailException {}
}
```

<br>

테스트 코드 수정

```java
@Test
@DirtiesContext // 컨텍스트의 DI 변경
public void upgradeLevels() throws Exception {
	userDao.deleteAll();
	for (User user: users) userDao.add(user);

	MockMailSender mockMailSender = new MockMailSender();
	userService.setMailSender(mockMailSender);

	userService.upgradeLevels();

	checkLevelUpgraded(user.get(0), false);
	checkLevelUpgraded(user.get(1), true);
	...

	List<String> request = mockMailSender.getRequests();
	assertThat(request.size(), is(2));
	assertThat(request.get(0), is(users.get(1).getEmail()));
	assertThat(request.get(1), is(users.get(3).getEmail()));	
}
```
