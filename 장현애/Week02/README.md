# 2장 테스트

## 목차
1. [UserDaoTest 다시 보기](#userdaotest-다시-보기)
2. [UserDaoTest 개선](#userdaotest-개선)
3. [JUnit](#junit)
4. [스프링 테스트 적용](#스프링-테스트-적용)
5. [학습 테스트로 배우는 스프링](#학습-테스트로-배우는-스프링)

- 전체 내용 요약보다는 개인적으로 메모하고 싶은 부분만 정리했습니다.

<br>

---

<br>

## UserDaoTest 다시 보기

```java
public class UserDaoTest {
	public static void main(String[] args) throws SQLException {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
		
		UserDao dao = context.getBean("userDao", UserDao.class);
		
		User user = new User();
		user.setId("user");
		user.setName("백기선");
		user.setPassword("married");
		
		dao.add(user);

		System.out.println(user.getId() + " 등록 성공");
		
		User user2 = dao.get(user.getId());
		System.out.println(user2.getName());
		System.out.println(user2.getPassword());

		System.out.println(user2.getId() + " 조회 성공");
```

<br>

- 웹을 통한 DAO 테스트 방법의 문제점
    - DAO 뿐만 아니라 서비스, 컨트롤러, JSP 뷰 등 모든 레이어 기능을 다 만들어야 한다.
    - 문제가 발생하면 어느 레이어에서 생긴 오류인지 찾아야 한다.

<br>

- UserDaoTest는 …
    - 작은 단위의 테스트 = 단위 테스트
        - 테스트는 가능한 작은 단위로 쪼개 집중하도록 한다.
    - 자동수행 테스트 코드
    - 지속적 개선과 점진적인 개발을 위한 테스트

<br>

### UserDaoTest의 문제점

- 수동 확인 작업의 번거로움
    - 사람이 직접 값의 일치 여부를 확인해야 한다.
- 실행 작업의 번거로움
    - 수백 개의 기능을 테스트하기 위해 main() 메소드를 수백 번 실행할 수는 없다.

<br>

## UserDaoTest 개선

### 테스트 검증 자동화

```java
public class UserDaoTest {
	public static void main(String[] args) throws SQLException {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
		
		UserDao dao = context.getBean("userDao", UserDao.class);
		
		User user = new User();
		user.setId("user");
		user.setName("백기선");
		user.setPassword("married");
		
		dao.add(user);

		System.out.println(user.getId() + " 등록 성공");
		
		// 수정된 부분
		if (!user.getName().equals(user2.getName())) {
			System.out.println("테스트 실패 (name)");
		}
		else if (!user.getPassword().equals(user2.getPassword())) {
			System.out.println("테스트 실패 (password)");
		}
		else {
			System.out.println("조회 테스트 성공");
		}
	}
}
```

<br>

### 테스트의 효율적인 수행과 결과 관리

- main 메소드는..
    - main() 메소드에 직접 넣어 테스트가 직접 제어권을 갖고 있음
    - 즉, 프레임워크를 적용하기에 부적합

<br>

- JUnit 테스트로 전환
    
    ```java
    public class UserDaoTest {
    
    	@Test
    	public void addAndGet() throws SQLException {
    		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    	
    		UserDao dao = context.getBean("userDao", UserDao.class);
    		User user = new User();
    		user.setId("gyumee");
    		user.setName("박성철");
    		user.setPassword("springno1");
    		
    		dao.add(user);
    
    		User user2 = dao.get(user.getId());
    		
    		assertThat(user2.getName(), is(user.getName()));
    		assertThat(user2.getPassword(), is(user.getPassword()));
    		...
    	}
    }
    ```

<br>
    

- JUnit 테스트 실행
    
    ```java
    public static void main(String[] args) {
    	JUnitCore.main("springbook.user.dao.UserDaoTest");
    }
    ```
 
<br>
   
### 테스트 결과의 일관성

- 테스트 실행 전 매번 DB의 데이터를 삭제해줘야 한다는 불편함
- 테스트가 외부 상태에 따라 성공하기도, 실패하기도 한다.

⇒ 테스트가 끝나면 테스트가 등록한 정보를 삭제해, 이전 상태로 되돌리자!

<br>

- deleteAll
    - USER 테이블의 모든 레코드를 삭제
    
    ```java
    public void deleteAll() throws SQLException {
    	Connection c = dataSource.getConnection();
    
    	PreparedStatement ps = c.prepareStatement("dele from users");
    	ps.executeUpdate();
    	
    	ps.close();
    	c.close();
    }
    ```
    
<br>

- getCount
    - USER 테이블의 레코드 개수를 돌려준다.
    
    ```java
    public int getCount() throws SQLException {
    	Connection c = dataSource.getConnection();
    
    	PreparedStatement ps = c.prepareStatement("select count(*) from users");
    
    	ResultSet rs = ps.executeQuery();
    	rs.next();
    	int count = rs.getInt(1);
    
    	rs.close();
    	ps.close();
    	c.close();
    
    	return count;
    }
    ```
  
<br>

- 두 메소드 테스트 코드
    
    ```java
    public class UserDaoTest {
    
    	@Test
    	public void addAndGet() throws SQLException {
    		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    	
    		UserDao dao = context.getBean("userDao", UserDao.class);
    
    		// 수정된 코드
    		dao.deleteAll();
    		assertThat(dao.getCount(), is(0));
    
    		User user = new User();
    		user.setId("gyumee");
    		user.setName("박성철");
    		user.setPassword("springno1");
    		
    		dao.add(user);
    		// 수정된 코드
    		assertThat(dao.getCount(), is(1));
    
    		User user2 = dao.get(user.getId());
    		
    		assertThat(user2.getName(), is(user.getName()));
    		assertThat(user2.getPassword(), is(user.getPassword()));
    		...
    	}
    }
    ```
<br>    

## JUnit

### 포괄적인 테스트

- getCount()에 대한 더 꼼꼼한 테스트를 만들어보자.
    
    ```java
    @Test
    public void count() throws SQLException {
    	ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
    
    	UserDao dao = context.getBean("userDao", UserDao.class);
    	User user1 = new User("gyumee", "박성철", "springno1");
    	User user2 = new User("leegw700", "이철원", "springno1");
    	User user3 = new User("hyumee", "박범진", "springno1");
    	
    	dao.deleteAll();
    	assertThat(dao.getCount(), is(0));
    
    	dao.add(user1);
    	assertThat(dao.getCount(), is(1));
    
    	dao.add(user2);
    	assertThat(dao.getCount(), is(2));
    
    	dao.add(user3);
    	assertThat(dao.getCount(), is(3));
    }
    ```
   
 <br>
 
- addAndGet() 테스트를 보완해보자.
    
    ```java
    public class UserDaoTest {
    
    	@Test
    	public void addAndGet() throws SQLException {
    		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    	
    		UserDao dao = context.getBean("userDao", UserDao.class);
    
    		User user1 = new User("gyumee", "박성철", "springno1");
    		User user2 = new User("leegw700", "이철원", "springno1");
    
    		dao.deleteAll();
    		assertThat(dao.getCount(), is(0));
    		
    		dao.add(user1);
    		dao.add(user2);
    		assertThat(dao.getCount(), is(2));
    
    		User userget1 = dao.get(user1.getId());
    		assertThat(userget1.getName(), is(user1.getName()));
    		assertThat(userget1.getPassword(), is(user1.getPassword()));
    
    		User userget2 = dao.get(user2.getId());
    		assertThat(userget2.getName(), is(user2.getName()));
    		assertThat(userget2.getPassword(), is(user2.getPassword()));
    
    		...
    	}
    }
    ```

<br>    

- get() 예외조건 테스트
    - get() 메소드에 전달한 id 값에 해당하는 사용자 정보가 없다면?
        1. null 값을 리턴한다.
        2. 예외를 던진다. ⇒ 선택!

<br>
    
    ```java
    @Test(expected=EmptyResultDataAccessException.class)
    public void getUserFailure() throws SQLException {
    	ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    
    	UserDao dao = context.getBean("userDao", UserDao.class);
    
    	dao.deleteAll();
    	assertThat(dao.getCount(), is(0));
    	
    	dao.get("unknown_id");
    }
    ```
    
    - get() 메소드를 테스트 성공을 위해 수정한다.
    
    ```java
    public User get(String id) throws SQLException {
    	...
    	ResultSet rs = ps.executeQuery();
    
    	User user = null;
    	if (rs.next()) {
    		user = new User();
    		user.setId(rs.getString("id"));
    		user.setName(rs.getString("name"));
    		user.setPassword(rs.getString("password"));
    	}
    
    	rs.close();
    	ps.close();
    	c.close();
    
    	if (user == null) throw new EmptyResultDataAccessException(1);
    	return user;
    }
    ```

<br>

- DAO 메소드에 대한 포괄적인 테스트는 꼭 만들어놓자.
- 성공적인 테스트만 골라 만드는 실수를 저지르지 말자.
- 부정적인 케이스를 먼저 만드는 습관을 들이자.

<br>

### 테스트가 이끄는 개발

- 테스트 주도 개발
    - TDD, Test Driven Development
    - 테스트 코드를 먼저 만들고, 테스트를 성공하게 해주는 코드를 작성하는 방식
    - 테스트를 작성하고, 성공시키는 코드를 만드는 작업의 주기를 가능한 짧게 가져간다.
    
<br>

### 테스트 코드 개선

```java
public class UserDaoTest {
	private UserDao dao;

	@Before
	public void setUp() {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
		UserDao dao = context.getBean("userDao", UserDao.class);
	}
	...

	@Test
	public void addAndGet() throws SQLException {
		...
	}

	@Test
	public void count() throws SQLException {
		...
	}
	...
}
```

<br>

- JUnit이 테스트 클래스를 수행하는 방식
    1. @Test가 붙은 public void형에 파라미터가 없는 테스트 메소드를 모두 찾는다.
    2. 테스트 클래스의 오브젝트를 하나 만든다.
    3. @Before 메소드가 있으면 실행한다.
    4. @Test 메소드를 하나 호출하고 테스트 결과를 저장한다.
    5. @After 메소드가 있으면 실행한다.
    6. 나머지 테스트 메소드에 대해 2~5번을 반복한다.
    7. 모든 테스트 결과를 종합해 돌려준다.

<br>

- 픽스처
    - fixture, 테스트 수행에 필요한 정보나 오브젝트
    - 일반적으로 @Before 메소드를 이용해 생성
    
    ```java
    public class UserDaoTest {
    	private UserDao dao;
    	private User user1;
    	private User user2;
    	private User user3;
    
    	@Before
    	public void setUp() {
    		...
    		this.user1 = new User("gyumee", "박성철", "springno1");
    		this.user2 = new User("leegw700", "이길원", "springno1");
    		this.user3 = new User("bumjin", "박범진", "springno1");
    	}
    	...
    }
    ```

<br>

## 스프링 테스트 적용

- 매 테스트가 시작될 때마다 애플리케이션 컨텍스트를 생성하는 것은 비효율적이다.

<br>

### 테스트를 위한 애플리케이션 컨텍스트 관리

- 스프링 테스트 컨텍스트 프레임워크 적용
    
    ```java
    // 프레임워크의 확장 기능 지정
    @RunWith(SpringJUnit4ClassRunner.class)
    // 애플리케이션 컨텍스트의 위치 지정
    @ContextConfiguration(locations="/applicationContext.xml")
    public class UserDaoTest {
    	@Autowired
    	private ApplicationContext context;
    	...
    
    	@Before
    	public void setUp() {
    		this.dao = this.context.getBean("userDao", UserDao.class);
    		...
    	}
    	
    ```
    
    - 여러 개의 테스트 클래스가 같은 설정파일을 가진 애플리케이션 컨텍스트 사용 시, 공유할 수 있도록 도와준다.
    - @Autowired: 스프링 DI에 사용되는 특별한 애노테이션

<br>

### 테스트 코드에 의한 DI

```java
// 테스트 메소드에서 애플리케이션 컨텍스트 구성, 상태를 변경한다는 것을 알려줌
@DirtiesContext
public class UserDaoTest {
	@Autowired
	UserDao dao;

	@Before
	public void setUp() {
		...
		DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost:testdb", "spring", "book", true);
		dao.setDataSource(dataSource);
	}
}
```

<br>

- 애플리케이션 컨텍스트에서 가져온 빈의 의존관계를 강제로 변경한다.
    - 바람직하지 못한 코드

<br>

- `@DirtiesContext`
    - 해당 테스트 클래스에서만 애플리케이션 컨텍스트 공유를 허용하지 않는다.
    - 테스트 메소드 수행 후 매번 새로운 애플리케이션 컨텍스트를 만들어 사용한다.

<br>

- 그러나 위의 방법은 단점이 더 많다.
    - 테스트 전용 설정 xml 파일을 따로 만들어 사용하자.
    
    ```java
    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations="/text-applicationContext.xml")
    public class UserDaoTest {
    ```

<br>

- 스프링 컨테이너 없이 테스트를 만드는 것도 가능하다.
    
    ```java
    public class UserDaoTest {
    	UserDao dao;            // @Autowired X
    	...
    
    	@Before
    	public void setUp() {
    		...
    		dao = new UserDao();
    		// 관계 직접 설정
    		DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost/testdb", "spring", "book", true);
    		dao.setDataSource(dataSource);
    	}
    
    ```
    
    - 스프링은 애플리케이션 로직 코드에 아무런 영향을 주지 않고 적용할 수 있는 **비침투적 기술** 이기 때문에 순수 DI 테스트도 가능하다.

<br>

1. 스프링 없이 테스트하는 방법을 최우선으로 고려하자.
2. 복잡한 의존관계의 오브젝트를 테스트할 땐 스프링 설정을 이용한 DI 방식을 적용하자.
3. 예외적인 의존관계를 강제로 구성해야 할 때에는 수동 DI로 테스트한다.

<br>

## 학습 테스트로 배우는 스프링

- 학습 테스트
    
    자신이 만들지 않은 프레임워크, 라이브러리 등에 대해서 테스트를 작성하는 것
    
    - 자신이 사용할 API 기능을 테스트를 통해 익히기 위함

<br>

### 학습 테스트의 장점

- 다양한 조건에 따른 기능을 손쉽게 확인할 수 있다.
- 학습 테스트 코드를 개발 중에 참고할 수 있다.
- 프레임워크, 제품 업그레이드 시 호환성 검증을 도와준다.
- 테스트 작성 훈련이 된다.
- 새로운 기술을 공부하는 과정이 즐거워진다.

<br>

### 학습 테스트 예제

- JUnit에 대한 학습 테스트

```java
public class JUnitTest {
	static Set<JUnitTest> testObjects = new HashSet<JUnitTest>();
	
	@Test public void test1() {
		assertThat(testObjects, not(hasItem(this)));
		testObjects.add(this);
	}

	@Test public void test2() {
		assertThat(testObjects, not(hasItem(this)));
		testObjects.add(this);
	}

	@Test public void test3() {
		assertThat(testObjects, not(hasItem(this)));
		testObjects.add(this);
	}
}
```

<br>

### 버그 테스트

- 코드에 오류가 있을 때 그 오류를 가장 잘 드러내줄 수 있는 테스트
1. 테스트의 완성도를 높여준다.
2. 버그의 내용을 명확하게 분석하게 해준다.
3. 기술적 문제를 해결하는 데 도움이 된다.

<br>

- 동등분할
    - - 같은 결과를 내는 값의 범위를 구분해 각 대표 값으로 테스트하는 방법
- 경계값 분석
    - 에러는 동등분할 범위 경계에서 주로 많이 발생한다는 특징을 이용
    - 경계 근처에 있는 값을 이용해 테스트하는 방법
