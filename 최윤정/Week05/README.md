# 📗 토비의 스프링 3.1 Vol.1 스프링의 이해와 원리
## 📝 5장 서비스 추상화
#### ✨ 5장에서 다루는 것 : 지금까지 만든 DAO에 트랜잭션을 적용해보면서 스프링이 어떻게 성격이 비슷한 여러 종류의 기술을 추상화하고 이를 일관된 방법으로 사용할 수 있도록 지원하는지

## 5.1 사용자 레벨 관리 기능 추가
간단한 비즈니스 로직을 추가해보자.
- 지금까지 만들었던 `UserDao`를 **다수의 회원이 가입할 수 있는 인터넷 서비스의 사용자 관리 모듈에 적용**한다고 생각해보자.

	- **사용자 관리 기능** : 정보 넣고 검색 + 정기적으로 사용자의 활동내역 참고해 레벨 조정해주는 기능 필요

<br>

### ✅ 인터넷 서비스의 사용자 관리 기능에서 구현해야 할 비즈니스 로직
- 사용자의 레벨은 BASIC, SILVER, GOLD 세 가지 중 하나
- 사용자가 처음 가입하면 BASIC 레벨이 되며, 이후 활동에 따라서 한 단계씩 업그레이드될 수 있음
- 가입 후 50회 이상 로그인을 하면 BASIC에서 SILVER 레벨이 됨
- SILVER 레벨이면서 30번 이상 추천 받으면 GOLD 레벨이 됨
- 사용자 레벨의 변경 작업은 일정한 주기를 가지고 일괄적으로 진행됨. 변경 작업 전에는 조건을 충족하더라도 레벨의 변경이 일어나지 않음.

<br>

### ✅ Step 1. 필드 추가 
정수형 상수 값으로 정의한 사용자 레벨(`private static final int BASIC = 1;`)은 다른 종류의 정보를 넣는 실수해도 컴파일러가 체크못함 + 범위 벗어나는 값 넣을 위험 있음.

#### 자바 5 이상에서 제공하는 `enum` 이용하자
#### Feat: 사용자 레벨용 `enum` 추가
```java
package springbook.user.domain;
...
public enum Level{
	BASIC(1), SILVER(2), GOLD(3); // 세 개의 enum 오브젝트 정의
	
	private final int value;
	
	Level(int value){ // DB에 저장할 값을 넣어줄 생성자 
		this.value = value;
	}
	
	public int intValue(){ // 값을 가져오는 메소드
		return value;
	}
	
	public static Level valueOf(int value){ // 값으로부터 Level 타입 오브젝트를 가져오도록 만든 스태틱 메소드
		switch(value){
			case 1: return BAISC;
			case 2: return SILVER;
			case 3: return GOLD;
			default: throw new AssertionError("Unknown value: " + value);
		}
	}
}
```
> 내부에는 DB에 저장할 int 타입의 값을 갖고 있지만, 겉으로는 Level 타입의 오브젝트이기 때문에 안전하게 사용할 수 있음. 

#### Feat: User 필드 추가
```java
public class User {
	...
	Level level;
	int login;
	int recommend;
	
	public Level getLevel() {
		return level;
	}
	
	public void setLevel(Level level){
		this.level = level;
	}
	...
	// login, recommend getter/setter 생략
}
```

#### DB의 USER 테이블에도 필드 추가
|필드명|타입|설정|
|--|--|--|
|Level|tinyint|Not Null|
|Login|int|Not Null|
|Recommend|int|Not Null|

#### Fix: UserDaoTest 수정
```java
public class UserDaoTest {
	...
	@Before
	public void setUp(){
		this.user1 = new User("gyumee", "박성철", "springno1", Level.BASIC, 1, 0);
		this.user2 = new User("dobby", "윤도비", "springno2", Level.SILVER, 55, 10);
		this.user3 = new User("bumjin", "박범진", "springno3", Level.GOLD, 100, 40);
	}
}
```
#### Fix: 추가된 필드를 파라미터로 포함하는 생성자
```java
class User{
	...
	public User(String id, String name, String password, Level level, int login, int recommend){
		this.id = id;
		this.name = name;
		this.password = password;
		this.level = level;
		this.login = login;
		this.recommend = recommend;
	}
}
```
#### Fix: 새로운 필드를 포함하는 User 필드 값 검증 메소드 수정
```java
private void checkSameUser(User user1, User user2){
	assertThat(user1.getId(), is(user2.getId()));
	assertThat(user1.getName(), is(user2.getName()));
	assertThat(user1.getPassword(), is(user2.getPassword()));
	assertThat(user1.getLevel(), is(user2.getLevel()));
	assertThat(user1.getLogin(), is(user2.getLogin()));
	assertThat(user1.getRecommend(), is(user2.getRecommend()));
}
```
#### Fix: `checkSameUser()` 메소드를 사용하도록 수정한 `addAndGet()` 메소드
```java
@Test public void addAndGet(){
	...
	User userget1 = dao.get(user1.getId());
	checkSameUser(userget1, user1);
	
	User userget2 = dao.get(user2.getId());
	checkSameUser(userget2, user2);
}
```
#### Fix: 추가된 필드를 위한 UserDaoJdbc 수정 
```java
public class UserDaoJdbc implements UserDao{
	...
	private RowMapper<User> userMapper = new RowMapper<User>(){
		public User mapRow(ResultSet rs, int rowNum) throws SQLException {
			User user = new User();
			user.setId(rs.getString("id"));
			user.setName(rs.getString("name"));
			user.setPassword(rs.getString("password"));
			user.setLevel(Level.valueOf(rs.getInt("level")));
			user.setLogin(rs.getInt("login"));
			user.setRecommend(rs.getInt("recommend"));
			return user;
		}
	};

	public void add(User user){
		this.jdbcTemplate.update("insert into users(id, name, password, level, login, recommend)" + "values(?,?,?,?,?,?)", 
		user.getId(), user.getName(), user.getPassword(), user.getLevel().intValue(), user.getLogin(), user.getRecommend());
	}
}
```
> Level enum은 오브젝트이므로 DB에 저장될 수 있는 SQL 타입이 아님. 
>
>DB에 저장할 경우, Level enum -> int형으로 변환 : 미리 만들어둔 `intValue()` 메소드 사용
>
>조회할 경우, int -> Level enum으로 변환 : 미리 만들어둔 스태틱 메소드 `valueOf()` 이용

<br>

### ✅ Step 2. 사용자 수정 기능 추가
사용자 관리 비즈니스 로직에 따르면, 사용자 정보는 여러 번 수정될 수 있음.

수정할 정보가 담긴 User 오브젝트를 전달 -> id를 참고해서 사용자 찾아 -> 필드 정보를 UPDATE문을 이용해 모두 변경해주는 메소드를 하나 만들자.

#### Feat: 사용자 정보 수정 메소드 테스트 추가
```java
@Test
public void update() {
	dao.deleteAll();

	dao.add(user1);
	
	user1.setName("오민규");
	user1.setPassword("springno6");
	user1.setLevel(Level.GOLD);
	user1.setLogin(1000);
	user1.setRecommend(999);
	dao.update(user1);

	User user1update = dao.get(user1.getId());
	checkSameUser(user1, user1update);
}
```
> 1. 픽스처 오브젝트 하나 등록
> 
> 2. id를 제외한 필드의 내용 바꾼 뒤 update() 호출
> 3. 다시 id로 조회해서 가져온 User 오브젝트와 수정한 픽스처 오브젝트를 비교

#### Feat: UserDao 인터페이스에 `update()` 메소드 추가
```java
public interface UserDao {
	...
	public void update(User user1);
}
```
#### Feat: UserDaoJdbc에 사용자 정보 수정용 `update()` 메소드 추가
```java
public void update(User user){
	this.jdbcTemplate.update("update users set name = ?, password = ?, level = ?, login = ?, " + "recommend = ? where id = ?", 
		user.getName(), user.getPassword(), user.getLevel().intValue(), user.getLogin(), user.getRecommend(), user.getId());
}
```

### 🧐 위에 작성한 `update()` 테스트의 문제점과 해결방법
- **문제**
	- 현재 `update()` 테스트는 수정할 로우의 내용이 바뀐 것만 확인할 뿐, 수정하지 않아야 할 로우의 내용이 그대로 남아 있는지는 확인해주지 못함.

- **해결 방법** 
	1. JdbcTemplate의 `update()` 가 돌려주는 리턴값 확인

	2.  테스트를 보강해서 원하는 사용자 외의 정보는 변경되지 않았음을 직접 확인 => 여기서는 이 방법 적용해보자!

		- 사용자를 두 명 등록해놓고, 그중 하나만 수정한 뒤에 수정된 사용자와 수정하지 않은 사용자의 정보를 모두 확인하면 됨 

#### Fix: `update()` 테스트 보완
```java
@Test
public void update(){
	dao.deleteAll();

	dao.add(user1); // 수정할 사용자
	dao.add(user2); // 수정하지 않을 사용자
	
	user1.setName("오민규");
	user1.setPassword("springno6");
	user1.setLevel(Level.GOLD);
	user1.setLogin(1000);
	user1.setRecommend(999);
	
	dao.update(user1);

	User user1update = dao.get(user1.getId());
	checkSameUser(user1, user1update);
	User user2same = dao.get(user2.getId());
	checkSameUser(user2, user2same);
}
```
<br>

### ✅ Step 3. 본격적인 사용자 관리 비즈니스 로직 구현
데이터 액세스 기능은 UserDao의 `getAll()` 메소드로 사용자를 다 가져와서 사용자별로 레벨 업그레이드 작업 진행 + UserDao의 `update()` 호출해 DB에 결과 넣어주면 됨.

#### 사용자 관리 로직은 어디에 두는 것이 좋을까?
> 사용자 관리 비즈니스 로직을 담을 클래스 UserService를 하나 추가하자.
>
>UserService는 UserDao 인터페이스 타입으로 userDao 빈을 DI 받아 사용하게 만듦.
>- 👽 UserDao는 인터페이스 이름, userDao는 빈 오브젝트 이름임

#### Feat: UserService 클래스 추가
```java
package springbook.user.service;
...
public class UserService{
	UserDao userDao;
	
	public void setUserDao(UserDao userDao){
		this.userDao = userDao;
	}
}
```
#### Feat: 스프링 설정파이에 userService 빈 설정 추가
```xml
<bean id="userService" class="springbook.user.service.UserService">
	<property name="userDao" ref="userDao" />
</bean>
<bean id="userDao" class="springbook.dao.UserDaoJdbc">
	<property name="dataSource" ref="dataSource" />
</bean>
```
#### Feat: UserServiceTest 클래스 추가
```java
package springbook.user.service;
...
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserServiceTest{
	@Autowired // 테스트 대상인 UserService 빈을 제공받을 수 있도록 
	UserService userService;
}
```
#### Feat: userService 빈이 생성돼서 userService 변수에 주입되는지 확인하는 테스트 메소드 추가
```java
@Test
public void bean() {
	assertThat(this.userService, is(notNullValue()));
}
```
> 테스트가 성공하면 UserService 빈이 잘 등록됐음을 알 수 있음.
>
> 이후 userService 오브젝트를 추가하면 `bean()` 테스트는 별 의미가 없으니 삭제해도 좋음.
#### Feat: 사용자 레벨 업그레이드 메소드 추가
```java
public void upgradeLevels(){
	List<User> users = userDao.getAll();
	for(User user : users){
		Boolean changed = null; // 레벨의 변화가 있는지를 확인하는 플래그
		if (user.getLevel() == Level.BASIC && user.getLogin() >= 50){ // BASIC 레벨 업그레이드 작업
			user.setLevel(Level.SILVER);
			changed = true;
		}
		else if(user.getLevel() == Level.SILVER && user.getRecommend() >= 30){ // SILVER 레벨 업그레이드 작업
			user.setLevel(Level.GOLD);
			changed = true;
		}
		else if(user.getLevel() == Level.GOLD) {
			changed = false; // GOLD 레벨은 변경이 일어나지 않음
		}
		else {
			changed = false; // 일치하는 조건이 없으면 변경 없음.
		}

		if(changed) { // 레벨의 변경이 있는 경우만 udpate() 호출
			userDao.update(user);
		}
	}
}
```
#### 이를 테스트 하는 방법?
- 적어도 가능한 모든 조건을 하나씩은 확인해봐야 함.

- 사용자 레벨은 BASIC, SILVER, GOLD 세 가지, 변경이 일어나지 않는 GOLD를 제외한 나머지 두 가지는 업그레이드가 되는 경우와 아닌 경우 존재 => 최소한 5가지 살펴봐야 함.

**다섯 종류의 사용자 정보를 등록해두고 업그레이드를 진행한 후에 예상한 대로 결과가 나오는지 확인해보자.**
#### Feat: 리스트로 만든 테스트 픽스처 추가
```java
class UserServiceTest {
	...
	List<User> users; // 테스트 픽스처(테스트 실행을 위해 베이스라인으로서 사용되는 객체들의 고정된 상태)
	
	@Before
	public void setUp() {
		users = Arrays.asList(
			new User("bumjin", "박범진", "p1", Level.BASIC, 49, 0),
			new User("joytouch", "강명성", "p2", Level.BASIC, 50, 0),
			new User("erwins", "신승한", "p3", Level.SILVER, 60, 29),
			new User("madnite1", "이상호", "p4", Level.SILVER, 60, 30),
			new User("green", "오민규", "p5", Level.GOLD, 100, 100)
		);
	}
}
```
#### Feat: 사용자 레벨 업그레이드 메소드 테스트 추가
```java
@Test
public void upgradeLevels() {
	userDao.deleteAll();
	for(User user : users) userDao.add(user);
	
	userService.upgradeLevels();

	// 각 사용자별로 업그레이드 후의 예상 레벨 검증
	checkLevel(users.get(0), Level.BASIC);
	checkLevel(users.get(1), Level.SILVER);
	checkLevel(users.get(2), Level.SILVER);
	checkLevel(users.get(3), Level.GOLD);
	checkLevel(users.get(4), Level.GOLD);
}

private void checkLevel(User user, Level expectedLevel){
	User userUpdate = userDao.get(user.getId());
	assertThat(userUpdate.getLevel(), is(expectedLevel));
}
```
> 1. 먼저 준비한 다섯 가지 종류의 사용자 정보 저장 
>
> 2. upgradeLevels() 메소드 실행
> 
> 3. 업그레이드 작업 끝나면 사용자 정보를 하나씩 가져와 레벨의 변경 여부 확인

<br>

### ✅ Step 4. 처음 가입한 사용자는 기본적으로 BASIC 레벨이어야 함
먼저 테스트부터 만들어보자.

#### 테스트 케이스는 두 종류를 만들면 됨.
1️⃣ 레벨이 미리 정해진 경우

2️⃣ 레벨이 비어있는 경우 
> 각각 add() 메소드 호출 후 결과 확인하도록 만들자.

<br>

#### User 오브젝트의 레벨이 변경됐는지 확인하기 위해 사용할 수 있는 두 가지 방법
1️⃣ UserService의 `add()` 메소드를 호출할 때 파라미터로 넘긴 User 오브젝트에 level 필드를 확인해보기

2️⃣ UserDao의 `get()` 메소드를 이용해 DB에 저장된 User 정보를 가져와 확인하기 -> 이게 가장 확실한 방법
>  두 가지 다 해도 좋고, 후자만 해도 괜찮음!

<br>

#### 테스트부터 만든 뒤, 메소드 코드 만들어보자.
#### Feat: `add()` 메소드의 테스트 추가
```java
@Test
public void add(){
	userDao.deleteAll();
	
	User userWithLevel = users.get(4); // GOLD 레벨
	User userWithoutLevel = users.get(0); 
	userWithoutLevel.setLevel(null);

	userService.add(userWithLevel);
	userService.add(userWithoutLevel);

	User userWithLevelRead = userDao.get(userWithLevel.getId());
	User userWithoutLevelRead = userDao.get(userWithoutLevel.getId());

	assertThat(userWithLevelRead.getLevel(), is(userWithLevel.getLevel()));
	assertThat(userWithoutLevelRead.getLevel(), is(Level.BASIC));
}
```
> 1. 이미 레벨이 GOLD로 설정된 사용자와 레벨이 null인 사용자 오브젝트 두 개 준비
> 
> 2. 준비한 두 개의 사용자 오브젝트를 UserService의 `add()` 메소드를 통해 초기화 -> DB에 저장하도록 
> 3. 확인을 위해 DB에서 사용자 정보 읽어오기
> 4. 레벨이 이미 설정됐던 것은 그대로 유지되어 있어야 하고, 레벨이 없던 것은 디폴트인 BASIC으로 설정됐는지 확인

위와 같은 테스트가 성공하도록 코드를 만들어보자.

#### Feat: 사용자 신규 등록 로직을 담은 `add()` 메소드
```java
public void add(User user){
	if(user.getLevel() == null) user.setLevel(Level.BASIC);
	userDao.add(user);
}
```

<br>

### ✅ Step 5. 코드 개선 
#### 작성된 코드를 살펴볼 때, 다음과 같은 질문을 해볼 필요가 있다.
- 코드에 **중복**된 부분은 없는가?

- 코드가 무엇을 하는 것인지 **이해하기 불편**하지 않은가?
- 코드가 **자신이 있어야 할 자리**에 있는가?
- 앞으로 **변경**이 일어난다면 어떤 것이 있을 수 있고, 그 **변화에 쉽게 대응**할 수 있게 작성되어 있는가?

#### 💥 `upgradeLevels()` 메소드 코드의 문제점
1. for 루프 속 if/elseif/else 블록 읽기 불편

2. 현재 페벨과 업그레이드 조건을 동시에 비교하는 부분
	- 성격이 다른 두 가지 경우가 모두 한 곳에서 처리되는 것이 뭔가 이상

#### 💡 해결방법 : 조건을 두 단계에 걸쳐서 비교한다
1. 레벨 확인하기

2. 각 레벨별로 다시 조건 판단하는 조건식 넣기

<br>

### ✅ `upgradeLevels()` 리팩토링
레벨을 업그레이드하는 작업의 기본 흐름만 만들자.

구체적인 구현에서 외부에 노출할 인터페이스를 분리하는 것과 마찬가지 작업이라 생각하면 됨.
#### Refactor: 기본 작업 흐름만 남겨둔 `upgradeLevels()`
```java
public void upgradeLevels() {
	List<User> users = userDao.getAll();
	for(User user : users){
		if(canUpgradeLevel(user)){
			upgradeLevel(user);
		}
	}
}
```
> 모든 사용자 정보 가져와 한 명씩 업그레이드가 가능한지 확인 -> 가능하면 업그레이드
#### Refactor: 업그레이드 가능 확인 메소드 추가
```java
private boolean canUpgradeLevel(User user){
	Level currentLevel = user.getLevel();
	switch(currentLevel){
		case BASIC: return (user.getLogin() >= 50);
		case SILVER: return (user.getRecommend() >= 30);
		case GOLD: return false;
		default: throw new IllegalArgumentException("Unknown Level: " + currentLevel);
	}
}
```
> 주어진 user에 대해 업그레이드가 가능하면 true, 가능하지 않으면 false 리턴
> 
> 상태에 따라서 업그레이드 조건만 비교하면 되므로, 역할과 책임이 명료해짐
#### 💥 Refactor: 레벨 업그레이드 작업 메소드 추가 (아래 읽어보면 알겠지만 별로인 방법임)
```java
private void upgradeLevel(User user){
	if(user.getLevel() == Level.BASIC) user.setLevel(Level.SILVER);
	else if(user.getLevel() == Level.SILVER) user.setLevel(Level.GOLD);
	userDao.update(user);
}
```
> 사용자 오브젝트의 레벨정보를 다음 단계로 변경, 변경된 오브젝트를 DB에 업데이트
> 
> ### 그런데, 
> #### 💥 문제점 : 다음 단계가 무엇인가 하는 로직과 그 때 사용자 오브젝트의 level 필드를 변경해준다는 로직이 함께 있고, 예외상황 처리 x
> #### 💡 해결방법 : 더 분리하자. 레벨의 순서와 다음 단계 레벨이 무엇인지를 결정하는 일을 Level에게 맡기자.
#### 💡 Fix: 업그레이드 순서를 담고 있도록 Level 수정
```java
public enum Level{

	// enum 선언에 DB에 저장할 값, 다음 단계 레벨 정보도 추가함
	GOLD(3, null), SILVER(2, GOLD), BASIC(1, SILVER); 
	
	private final int value;
	private final Level next;
	
	Level(int value, Level next){ 
		this.value = value;
		this.next = next;
	}
	
	public int intValue(){ 
		return value;
	}
	
	public Level nextLevel(){ 
		return this.next;
	}
	
	public static Level valueOf(int value){ 
		switch(value){
			case 1: return BAISC;
			case 2: return SILVER;
			case 3: return GOLD;
			default: throw new AssertionError("Unknown value: " + value);
		}
	}
}
```
#### 💡 Fix: 사용자 정보가 바뀌는 부분을 User에 추가
```java
public void upgradeLevel(){
	Level nextLevel = this.level.nextLevel();
	if(nextLevel == null){
		throw new IllegalStateException(this.level + "은 업그레이드가 불가능합니다");
	}
	else{
		this.level = nextLevel;
	}
}
```
#### 💡 Fix: 간결해진 `upgradeLevel()`
```java
private void upgradeLevel(User user){
	user.upgradeLevel();
	userDao.update(user);
}
```

<br>

#### 지금까지 개선한 코드
- 각 오브젝트와 메소드가 각각 자기 몫의 책임 맡아 일을 하는 구조

- UserService, User, Level이 내부 정보 다루는 자신의 책임에 충실한 기능 갖고 있으며 필요가 생기면 이런 작업 수행해달라고 서로 요청하는 구조
- 코드 이해하기 쉽고 변화 대응하기 편함

>객체지향적인 코드는 다른 오브젝트의 데이터를 가져와서 작업하는 대신 데이터를 갖고 있는 다른 오브젝트에게 작업을 해달라고 요청.
>
>**오브젝트에게 데이터를 요구하지 말고 작업을 요청하라는 것**이 **객체지향 프로그래밍의 가장 기본이 되는 원리**이기도 함.

<br>

### ✅ 지금까지 리팩토링한 코드에 맞게 다른 코드들도 수정하자.
#### Feat: User에 추가한 `upgradeLevel()` 메소드 테스트 추가
```java
public class UserTest{
	User user;
	
	@Before
	public void setUp() {
		user = new User();
	}
	
	@Test()
	public void upgradeLevel() {
		Level[] levels = Level.values();
		for(Level level : levels){
			if(level.nextLevel() == null) continue;
			user.setLevel(level);
			user.upgradeLevel();
			assertThat(user.getLevel(), is(level.nextLevel()));
		}
	}

	@Test(expected=IllegalStateException.class)
	public void cannotUpgradeLevel(){
		Level[] levels = Level.values();
		for(Level level : levels){
			if(level.nextLevel() != null) continue;
			user.setLevel(level);
			user.upgradeLevel();
		}
	}
}
```
 #### Fix: UserServiceTest에 `upgradeLevels()` 테스트 개선 
```java
@Test
public void upgradeLevels() {
	userDao.deleteAll();
	for(User user : users) userDao.add(user);
	
	userService.upgradeLevels();

	// 각 사용자별로 업그레이드 후의 예상 레벨 검증
	checkLevelUpgraded(users.get(0), false);
	checkLevelUpgraded(users.get(1), true);
	checkLevelUpgraded(users.get(2), false);
	checkLevelUpgraded(users.get(3), true);
	checkLevelUpgraded(users.get(4), false);
}

private void checkLevelUpgraded(User user, Boolean upgraded){
	User userUpdate = userDao.get(user.getId());
	if(upgraded){
		assertThat(userUpdate.getLevel(), is(user.getLevel().nextLevel())); // 업데이트가 일어났는지 확인
	}
	else{
		assertThat(userUpdate.getLevel(), is(user.getLevel())); // 업데이트가 일어나지 않았는지 확인	
	}
}
```
#### Fix: UserService 에 상수 도입
```java
public static final int MIN_LOGCOUNT_FOR_SILVER = 50;
public static final int MIN_RECOMMEND_FOR_GOLD = 30;
private boolean canUpgradeLevel(User user){
	Level currentLevel = user.getLevel();
	switch(currentLevel){
		case BASIC: return (user.getLogin() >= MIN_LOGCOUNT_FOR_SILVER);
		case SILVER: return (user.getRecommend() >= MIN_RECOMMEND_FOR_GOLD);
		case GOLD: return false;
		default: throw new IllegalArgumentException("Unknown Level: " + currentLevel);
	}
}
```
#### Fix: 상수 사용하도록 테스트도 수정
```java
import static springbok.user.service.UserService.MIN_LOGCOUNT_FOR_SILVER;
import static springbok.user.service.UserService.MIN_RECOMMEND_FOR_GOLD;

class UserServiceTest {
	...
	@Before
	public void setUp() {
		users = Arrays.asList(
			new User("bumjin", "박범진", "p1", Level.BASIC, MIN_LOGCOUNT_FOR_SILVER-1, 0),
			new User("joytouch", "강명성", "p2", Level.BASIC, MIN_LOGCOUNT_FOR_SILVER, 0),
			new User("erwins", "신승한", "p3", Level.SILVER, 60, MIN_RECOMMEND_FOR_GOLD-1),
			new User("madnite1", "이상호", "p4", Level.SILVER, 60, MIN_RECOMMEND_FOR_GOLD),
			new User("green", "오민규", "p5", Level.GOLD, 100, Integer.MAX_VALUE)
		);
	}
}
```

🤗 연말 이벤트나 새로운 서비스 홍보 기간 중 레벨 업그레이드 정책을 다르게 적용할 필요가 있을 경우, 사용자 업그레이드 정책을 담은 인터페이스를 만들어두고 UserService라는 DI로 제공받은 정책 구현 클래스를 이 인터페이스를 통해 사용하자.

#### Feat: 업그레이드 정책 인터페이스
```java
public interface UserLevelUpgradePolicy{
	boolean canUpgradeLevel(User user);
	void upgradeLevel(User user);
}
```

<br>

## 5.2 트랜잭션 서비스 추상화
"정기 사용자 레벨 관리 작업을 수행하는 도중 네트워크가 끊기거나 서버에 장애가 생겨 작업을 완료할 수 없다면, 그때까지 변경된 사용자의 레벨은 그대로 둘까? 아님 모두 초기 상태로 되돌려 놓아야 할까?"
> 그때까지 진행된 변경 작업도 모두 취소시키기로 했음.

### 그렇다면 지금까지 만든 사용자 레벨 업그레이드 코드는 어떻게 동작할까?
#### 👀 모든 사용자에 대해 업그레이드 작업을 진행하다가 중간에 예외 발생해서 작업 중단된다면 어떻게 될까?

이미 변경된 사용자의 레벨은 작업 이전 상태로 돌아갈까?

아니면 바뀐 채로 남아 있을까?

<br>

### ✅ 테스트용으로 UserService 대역 만들어 예외 강제로 만들고 확인해보자.

#### Step 1. 테스트용 UserService 확장 클래스인 TestUserService 클래스 만들기
- 간단히 UserService를 상속 후 테스트에 필요한 기능 추가하도록 일부 메소드를 오버라이딩

- 이 때, UserService의 `upgradeLevel()` 메소드 접근권한을 `protected`로 수정해 상속을 통해 오버라이딩이 가능하도록 함

- 오버라이드된 `upgradeLevel()` 메소드는 UserService 메소드 기능 그대로 수행 하지만 미리 지정된 id를 가진 사용자가 발견되면 강제로 예외 던지도록 구현

#### Step 2. 다른 예외가 발생했을 경우와 구분하기 위해 테스트 목적을 띤 예외를 아래와 같이 정의
```java
static class TestUserServiceException extends RuntimeException{
}
```
#### Step 3. 예외 발생 시 작업 취소 여부 테스트 코드 추가
테스트의 목적 : 
- 사용자 레벨 업그레이드를 시도하다가 중간에 예외 발생할 경우, 그 전에 업그레이드했던 사용자도 다시 원래 상태로 돌아갔는지 확인
- 소스 코드 p.351 

#### 위의 테스트는 실패한다. 그 원인은?
> ✅ 트랜잭션 문제 때문.
>
>모든 사용자의 레벨을 업그레이드하는 작업인 `upgradeLevels()` 메소드가 하나의 트랜잭션 안에서 동작하지 않았기 때문

<br>

## ✅ 트랜잭션
#### 더 이상 나눌 수 없는 단위 작업
**작업을 쪼개서 작은 단위로 만들 수 없다**는 것은 **트랜잭션의 핵심 속성**인 **원자성**을 의미함.
> 위의 예시로 설명하자면, 모든 사용자에 대한 레벨 업그레이드 작업은 새로 추가된 기술 요구사항대로 전체가 다 성공하든지 아니면 전체가 다 실패하든지 해야 함. 
> 
더 이상은 쪼개질 수 없는 물질이라는 의미로 이름을 붙인 원자와 마찬가지로, 이 작업도 더 이상 쪼개서 이뤄질 수 없는 원자와 같은 성질을 띔.

따라서, 중간에 예외가 발생해서 작업을 완료할 수 없다면 아예 작업이 시작되지 않은 것처럼 초기 상태로 돌려놔야 함.

<br>

### ✅ 트랜잭션 경계설정
DB는 그 자체로 완벽한 트랜잭션 지원.

하나의 SQL 명령을 처리하는 경우는 DB가 트랜잭션을 보장해준다고 믿을 수 있음.

하지만, **여러 개의 SQL이 사용되는 작업을 하나의 트랜잭션으로 취급해야 하는 경우**가 있음.
> 예) 계좌이체, 사용자에 대한 레벨 수정 작업 등

#### 예를 들어, 두 개의 SQL이 필요한데, 첫 번째 SQL 성공적 실행하고 두 번째 SQL이 성공하지 전에 장애가 생겨 작업이 중단되는 경우 

- 이때 두가지 작업이 하나의 트랜잭션이 되려면, 앞에서 처리한 첫 번째 SQL 작업도 취소해야 한다. 

	- 이런 취소 작업을 '**트랜잭션 롤백(transaction rollback)**'이라고 함.

- 반대로, 여러 개의 SQL을 하나의 트랜잭션으로 처리하는 경우, 

	- 모든 SQL 수행 작업이 다 성공적으로 마무리됐다고 DB에 알려줘서 작업을 확정시켜야 함. = **트랜잭션 커밋(transaction commit)**

<br>

### ✅ JDBC 트랜잭션의 트랜잭션 경계설정
모든 트랜잭션은 시작하는 지점과 끝나는 지점이 있음.

시작하는 방법은 한 가지, 끝나는 방법은 두 가지(롤백 또는 커밋)
- 롤백 : 모든 작업을 무효화
- 커밋 : 모든 작업을 다 확정

#### ✅ 트랜잭션의 경계란?
> 애플리케이션 내에서 트랜잭션이 시작되고 끝나는 위치

복잡한 로직 흐름 사이에서 정확하게 트랜잭션 경계 설정하는 일은 매우 중요한 작업!

#### JDBC를 이용해 트랜잭션 적용하는 가장 간단한 예제
- 소스 코드 p.354

- JDBC의 트랜잭션은 하나의 Connection을 가져와 사용하다가 닫는 사이에서 일어남
- 트랜잭션의 시작과 종료는 Connection 오브젝트를 통해 이뤄지기 때문
- JDBC에서 트랜잭션을 시작하려면 자동커밋 옵션을 false로 만들어주면 됨
- 트랜잭션이 한 번 시작되면 `commit()` 또는 `rollback()` 메소드가 호출될 때까지의 작업이 하나의 트랜잭션으로 묶임.
- `commit()` 또는 `rollback()`이 호출되면 그에 따라 작업 결과가 DB에 반영되거나 취소되고 트랜잭션이 종료됨.
- **일반적으로 작업 중 예외 발생하면 트랜잭션을 rollback함**
	- 이유? 예외 발생했다는 건, 트랜잭션을 구성하는 데이터 액세스 작업을 마무리할 수 없는 상황이거나 DB에 결과를 반영하면 안 되는 이유가 생겼기 때문

#### ✅ 결론
- `setAutoCommit(false)`로 트랜잭션의 시작을 선언하고 `commit()` 또는 `rollback()`으로 트랜잭션을 종료하는 작업 = **트랜잭션의 경계설정(transaction demarcation)**

- **트랜잭션의 경계는 하나의 Connection이 만들어지고 닫히는 범위 안에 존재**
- 이렇게 하나의 DB 커넥션 안에서 만들어지는 트랜잭션 = **로컬 트랜잭션(local transaction)**

<br>

### 🤔 왜 지금까지 코드에는 트랜잭션 경계설정 코드가 존재하지 않았을까?
- JdbcTemplate은 하나의 템플릿 메소드 안에서 DataSource의 `getConnection()` 메소드 호출해 Connection 오브젝트 가져오고, 작업 마치면 Connection을 확실하게 닫아주고 템플릿 메소드를 빠져나옴.
=> 템플릿 메소드 호출 한 번에 한 개의 DB 커넥션이 만들어지고 닫히는 일까지 일어남.

- **일반적으로 트랜잭션은 커넥션보다도 존재 범위가 짧음.**
- 따라서, **템플릿 메소드가 호출될 때마다 트랜잭션이 새로 만들어지고 메소드를 빠져나오기 전에 종료**됨.
- 결국, JdbcTemplate의 메소드를 사용하는 **UserDao는 각 메소드마다 하나씩의 독립적인 트랜잭션으로 실행**될 수밖에 없음.

<br>

### 🧐 그렇다면, `upgradeLevels()`와 같이 여러 번 DB에 업데이트를 해야 하는 작업을 하나의 트랜잭션으로 만들려면 어떻게 해야 할까?
- 어떤 일련의 작업이 하나의 트랜잭션으로 묶이려면 그 작업이 진행되는 동안 DB 커넥션도 하나만 사용돼야 함.

### 트랜잭션의 경계설정 작업을 UserService쪽으로 가져오자.
> UserDao가 가진 SQL이나 JDBC API를 이용한 데이터 액세스 코드는 최대한 그대로 남겨둔 채로, UserService에는 트랜잭션 시작과 종료를 담당하는 최소한의 코드만 가져오게 만들자.

#### `upgradeLevels()` 의 트랜잭션 경계설정 구조
```java
public void upgradeLevels() throws Exception {
	(1) DB Connection 생성
	(2) 트랜잭션 시작
	try{
		(3) DAO 메소드 호출
		(4) 트랜잭션 커밋
	} 
	catch(Exception e){
		(5) 트랜잭션 롤백
		throw e;
	}
	finally{
		(6) DB Connection 종료
	}
}
```
> 👽 여기서 생성된 Connection 오브젝트를 갖고 데이터 액세스 작업 진행하는 코드는 UserDao의 `update()` 메소드 안에 있어야 함.
> 
>- 그래야만 같은 트랜잭션 안에서 동작하기 때문!
>
#### UserService에서 만든 Connection 오브젝트를 UserDao에서 사용하기 위해 할 일(소스코드 p.358 ~ p.359)
1. DAO 메소드를 호출할 때마다 Connection 오브젝트를 파라미터로 전달하기

2. UserService의 메소드 사이에도 같은 Connection 오브젝트를 사용하도록 파라미터로 전달하기

<br>

### 💥 UserService와 UserDao를 이런 식으로 수정하면 트랜잭션 문제는 해결가능,  그러나 여러 가지 문제점 발생
1. DB 커넥션을 비롯한 리소스의 깔끔한 처리를 가능하게 했던 JdbcTemplate 더 이상 활용 불가

2. DAO의 메소드와 비즈니스 로직 담고 있는 UserService의 메소드에 Connection 파라미터가 추가돼야 함 -> 메소드가 지저분해짐.
3. Connection 파라미터가 UserDao 인터페이스 메소드에 추가되면 UserDao는 더 이상 데이터 액세스 기술에 독립적일 수 x -> 기껏 인터페이스 사용해 DAO 분리하고 DI 적용했던 수고 물거품 됨
4. 테스트 코드에도 영향 미침 -> 테스트 코드에서 직접 Connection 오브젝트를 일일이 만들어 DAO 메소드를 호출하도록 모두 변경해야 함;

<br>

## ✅ 트랜잭션 동기화
#### 🌱 스프링이 제공하는 트랜잭션 동기화 기법을 활용하자.

### ✅ 트랜잭션 동기화(transaction synchronization) 란?
- **UserService에서 트랜잭션을 시작하기 위해 만든 Connection 오브젝트를 특별한 저장소에 보관해두고, 이후에 호출되는 DAO의 메소드에서는 저장된 Connection을 가져다가 사용하게 하는 것**

- 정확히는 DAO가 사용하는 JdbcTemplate이 트랜잭션 동기화 방식을 이용하도록 하는 것
- 트랜잭션이 모두 종료되면, 그때는 동기화를 마치면 됨.
- 트랜잭션의 경계설정이 필요한 `upgradeLevels()` 에서만 Connection을 다루게 하고, 여기서 생성된 Connection과 트랜잭션을 DAO의 JdbcTemplate이 사용할 수 있도록 별도의 저장소에 동기화하는 방법을 적용하기만 하면 됨.

<br>

### ✅ 트랜잭션 동기화 저장소 
> 작업 스레드마다 독립적으로 Connection 오브젝트를 저장하고 관리하기 때문에 다중 사용자를 처리하는 서버의 멀티스레드 환경에서도 충돌이 날 염려는 없음

<br>

### ✅ 트랜잭션 동기화 적용
문제는 멀티스레드 환경에서도 안전한 트랜잭션 동기화 방법을 구현하는 일이 기술적으로 간단하지 않다는 점.

다행히도 스프링은 JdbcTemplate과 더불어 이런 트랜잭션 동기화 기능을 지원하는 간단한 유틸리티 메소드를 제공 중🤗
#### Fix: 트랜잭션 동기화 방식을 적용한 UserService
```java
private DataSource dataSource;

public void setDataSource(DataSource dataSource){
	this.dataSource = dataSource;
}

public void upgradeLevels() throws Exception {
	TransactionSynchronizationManager.initSynchronization(); // 트랜잭션 동기화 관리자를 이용해 동기화 작업을 초기화함
	Connection c = DataSourceUtils.getConnection(dataSource);
	c.setAutoCommit(false);

	try{
		List<User> users = userDao.getAll();
		for (User user : users) {
			if (canUpgradeLevel(user)){
				upgradeLevel(user);
			}
		}
		c.commit();
	} catch(Exception e){
		c.rollback();
		throw e;
	} finally {
		DataSourceUtils.releaseConnection(c, dataSource);
		
		// 동기화 작업 종료 및 정리
		TransactionSynchronizationManager.unbindResource(this.dataSource);
		TransactionSynchronizationManager.clearSynchronization();
	}
}
```
#### Fix: 동기화가 적용된 UserService에 따라 수정된 테스트 
```java
@Autowired DataSource dataSource;
...
@Test
public void upgradeAllOrNothing() throws Exception {
	UserService testUserService = new TestUserService(users.get(3).getId());
	testUserService.setUserDao(this.userDao);
	testUserService.setDataSource(this.dataSource);
	...
}
```
#### Fix: dataSource 프로퍼티를 추가한 userService 빈 설정
```xml
<bean id="userService" class="springbook.user.service.UserService">
	<property name="userDao" ref="userDao" />
	<property name="dataSource" ref="dataSource" />
</bean>
```

<br>

## 트랜잭션 서비스 추상화
### 기술과 환경에 종속되는 트랜잭션 경계설정 코드
#### 🤷‍♀️ 만약, 이 사용자 관리 모듈을 구매해서 사용하기로 한 G사에서 새로운 요구를 했을 경우?
- G사는 여러 개의 DB 사용 중

- **하나의 트랜잭션 안에서 여러 개의 DB에 데이터를 넣는 작업**은 각 DB와 독립적으로 만들어지는 Connection을 통해서가 아니라, **별도의 트랜잭션 관리자를 통해 트랜잭션을 관리**하는 **글로벌 트랜잭션(global transaction)** 방식을 사용해야 함.

###  JTA(Java Transaction API)
>JDBC 외에 이런 글로벌 트랜잭션을 지원하는 트랜잭션 매니저를 지원하기 위한 API
>
>자바에서 제공 중
>
> JTA를 이용해 트랜잭션 매니저를 활용하면 여러 개의 DB나 메시징 서버에 대한 작업을 하나의 트랜잭션으로 통합하는 분산 트랜잭션 또는 글로벌 트랜잭션이 가능해짐.
>
>✅ **하나 이상의 DB가 참여하는 트랜잭션을 만들려면 JTA를 사용해야 한다.**

#### [G사 요청 결론] 
- 서버가 제공하는 트랜잭션 매니저와 트랜잭션 서비스를 사용할테니 JDBC API가 아닌 **JTA**를 사용해 트랜잭션을 관리하게 해달라.

#### 🤦‍♀️ 이 와중에 하이버네이트를 이용해 UserDao를 직접 구현했다고 Y사에서 연락이 왔다면?
- 💥 문제 : UserService의 코드가 특정 트랜잭션 방법(JTA, 하이버네이트 등)에 의존적이지 않고, 독립적일 수 있게 만들어야 함
- ✅ 해결 : 스프링이 제공하는 **트랜잭션 추상화**를 이용하자.

<br>

### ✅ 스프링의 트랜잭션 추상화
- 스프링이 제공하는 트랜잭션 경계설정을 위한 추상 인터페이스 : `PlatformTransactionManager` 

- JDBC의 로컬 트랜잭션을 이용한다면 `PlatformTransactionManager`를 구현한 `DataSourceTransactionManager`를 사용하면 됨
- JTA를 이용한 글로벌 트랜잭션으로 변경하려면 `PlatformTransactionManager`를 구현 클래스를 `JTATransactionManager`를 사용하면 됨
- 하이버네이트는 `PlatformTransactionManager`를 구현 클래스를 `HibernateTransactionManager`를 사용하면 됨
- JPA를 적용했다면 `PlatformTransactionManager`를 구현 클래스를 `JPATransactionManager`를 사용하면 됨

#### Fix: 트랜잭션 매니저를 빈으로 분리시킨 UserService
```java
public class UserService{
	...
	private PlatformTransactionManager transactionManager;
	public void setTransactionManager(PlatformTransactionManager transactionManager){
		this.transactionManager = transactionManager;
	}
	
	public void upgradeLevels() {
		TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());

		try{
			List<User> users = userDao.getAll();
			for (User user : users) {
				if (canUpgradeLevel(user)){
					upgradeLevel(user);
				}
			}
			this.transactionManager.commit(status);
		} catch(RuntimeException e){
			this.transactionManager.rollback(status);
			throw e;
		}
	}
	...
}
```
#### Fix: 스프링 설정파일에 UserService에 DI될 transactionManager 빈 등록 (소스 코드 p.374)
#### Fix: 트랜잭션 매니저를 수동 DI하도록 테스트 수정 (소스 코드 p.374)
#### [결론]
- 자바에서 사용되는 트랜잭션 API의 종류와 방법은 다양하고, 환경과 서버에 따라서 트랜잭션 방법이 변경되면 경계설정 코드도 함께 변경돼야 함.

<br>

## 5.3 서비스 추상화와 단일 책임 원칙
**기술과 서비스에 대한 추상화 기법** = **같은 계층**에서 **수평적 분리**

**트랜잭션의 추상화 기법** = **다른 계층** 코드 **수직적 분리**

애플리케이션 로직의 종류에 따른 수평적인 구분이든, 로직과 기술이라는 수직적인 구분이든 모두 결합도가 낮으며, 서로 영향을 주지 않고 자유롭게 확장될 수 있는 구조를 만들 수 있는 데는 '**스프링의 DI**'가 중요한 역할 하고 있음.
>#### DI의 가치
>
>- 관심, 책임, 성격이 다른 코드를 깔끔하게 분리

✅ 서비스 추상화는 로우레벨의 트랜잭션 기술과 API의 변화에 상관없이 일관된 API를 가진 추상화 계층을 도입함.

<br>

### ✅ 단일 책임 원칙(Single Responsibility Principle)
객체지향 설계의 원칙 중 하나

>**하나의 모듈은 한 가지 책임을 가져야 한다.**

#### [단일 책임 원칙의 장점]
- 어떤 변경이 필요할 때 수정 대상이 명확해짐

	- 기술이 바뀌면 기술 계층과의 연동을 담당하는 기술 추상화 계층의 설정만 바꿔주면 됨

	- 데이터를 가져오는 테이블의 이름이 바뀌면 데이터 액세스 로직을 담고 있는 UserDao를 변경하면 됨

<br>

### 객체지향 설계와 프로그래밍의 원칙은 서로 긴밀하게 관련이 있다.

단일 책임 원칙을 잘 지키는 코드를 만들려면, 인터페이스 도입하고 이를 DI로 연결해야 하며, 그 결과로 단일 책임 원칙뿐 아니라 개방 폐쇄 원칙도 잘 지키고, 모듈 간에 결합도가 낮아서 서로의 변경이 영향을 주지 않고, 같은 이유로 변경이 단일 책임에 집중되는 응집도 높은 코드가 나옴.

또한, 이런 과정에서 전략 패턴, 어댑터 패턴, 브리지 패턴, 미디에이터 패턴 등 많은 디자인 패턴이 자연스럽게 적용됨.

객체지향 설계 원칙 잘 지켜서 만든 코드는 테스트하기도 편함.

스프링이 지원하는 DI와 싱글톤 레지스트리 덕분에 더욱 편리하게 자동화된 테스트 만들 수 있음.

> 스프링의 의존관계 주입 기술인 DI는 모든 스프링 기술의 기반이 되는 핵심 엔진이자 원리이며, 스프링이 지지하고 지원하는 좋은 설계와 코드를 만드는 모든 과정에서 사용되는 가장 중요한 도구.

<br>

## 5.4 메일 서비스 추상화
#### 고객으로부터 사용자 레벨 관리에 관한 새로운 요청사항 :
> 레벨이 업그레이드되는 사용자에게는 안내 메일을 발송해달라는 것

#### 안내 메일 발송하기 위해 해야 할 일 두 가지

1. 사용자의 이메일 정보 관리해야 함 -> User에 email 필드 추가

2. 업그레이드 작업을 담은 UserService의 `upgradeLevel()` 메소드에 메일 발송 기능 추가

<br>

## JavaMail을 이용한 메일 발송 기능
### Step 1. 사용자 정보에 이메일 추가
1. DB의 User 테이블에 email 필드 추가

2. User 클래스에 email 프로퍼티 추가
3. UserDao의 userMapper와 insert(), update()에 email 필드 처리 코드 추가
4. User 생성자가 email 받을 수 있게 하고, 테스트 데이터 맞게 준비한 뒤 등록, 수정, 조회에서 email 값이 잘 처리되는지 확인하도록 UserDaoTest 수정

<br>

### Step 2. JavaMail 메일 발송
자바에서 메일을 발생할 때는 표준 기술인 JavaMail 사용하면 됨
> javax.mail 패키지에서 제공하는 자바의 이메일 클래스 사용

1️⃣ **업그레이드 작업을 담은 메소드인 upgradeLevel()에서 메일 발송 메소드 호출하기**
>#### Fix: 레벨 업그레이드 작업 메소드 수정
```java
protected void upgradeLevel(User user){
	user.upgradeLevel();
	userDao.update(user);
	sendUpgradeEMail(user);
}
```

2️⃣ **JavaMail API 사용하는 메소드 추가하기**
>#### Feat: JavaMail을 이용한 메일 발송 메소드 추가
```java
private void sendUpgradeEMail(User user){
	Properties props = new Properties();
	props.put("mail.smtp.host", "mail.ksug.org");
	Session s = Session.getInstance(props, null);
	
	MimeMessage message = new MimeMessage(s);
	try{
		message.setFrom(new InternetAddress("useradmin@ksug.org"));
		message.addRecipient(Message.RecipientType.TO, new InternetAddress(user.getEmail()));
		message.setSubject("Upgrade 안내");
		message.setText("사용자님의 등급이 " + user.getLevel().name() + "로 업그레이드되었습니다.");
		Transport.send(message);
	} catch(AddressException e){
		throw new RuntimeException(e);
	} catch(MessagingException e){
		throw new RuntimeException(e);
	} catch(UnsupportedEncodingException e){
		throw new RuntimeException(e);
	}
}
```
### 💥 그런데, 메일 서버가 준비되어 있지 않다면 어떻게 할까?
#### 해결방법 1) 테스트 메일 서버 사용
메일 테스트를 한다고 매번 매일 수신 여부까지 일일이 확인할 필요는 없고, 테스트 가능한 메일 서버까지만 잘 전송되는지 확인하면 됨.

그리고 테스트용 메일 서버는 메일 전송 요청은 받지만 실제로 메일이 발송되지 않도록 설정

#### 해결방법 2) 테스트용 JavaMail 사용 (이 방법을 선택하자 ✅)
JavaMail은 자바의 표준 기술, 이미 검증된 안정적 모듈 => 굳이 테스트할 때마다 JavaMail을 직접 구동시킬 필요 없음

또한, JavaMail이 동작하면 외부 메일 서버와 네트워크로 연동하고 전송하는 부하가 큰 작업이 일어나기 때문에 이를 생략할 수 있다면 더 좋기 때문

#### 💥 그런데, JavaMail을 이용한 테스트의 한 가지 문제점 : JavaMail의 API는 이 방법을 적용 불가 
> JavaMail 핵심 API에는 DataSource처럼 인터페이스로 만들어져서 구현을 바꿀 수 있는 게 없음
> 
> 더 자세한 사항은 p.384 참고

#### 💡 해결방법 : 서비스 추상화를 적용하자.
> 스프링은 JavaMail을 사용해 만든 코드는 손쉽게 테스트하기 힘들다는 문제를 해결하기 위해 JavaMail에 대한 추상화 기능을 제공하고 있음

<br>

### Step 3. 메일 발송 기능 추상화 (소스 코드 p.385 ~ p.387 )
추가해야 하는 라이브러리
```java
com.springsource.javax.activation-1.1.0.jar
com.springsource.javax.mail-1.4.0.jar
org.springframework.context.support-3.0.7.RELEASE.jar
```
#### Feat: 스프링의 MailSender를 이용한 메일 발송 메소드 추가
1. JavaMail을 사용해 메일 발송 기능을 제공하는 `JavaMailSenderImpl` 클래스의 오브젝트 생성

2. MailMessage 인터페이스의 구현 클래스 오브젝트를 만들어 메일 내용 작성
#### Fix: 메일 전송 기능을 가진 오브젝트를 DI 받도록 UserService 수정
- MailSender 인터페이스 타입의 변수 만들고, 수정자 메소드 추가해 DI 가능하도록 만듦
#### Feat: 스프링 설정파일 안에 메일 발송 오브젝트의 빈 등록
- JavaMailSenderImpl 클래스로 빈을 만들고 UserService에 DI 해줌

<br>

### Step 4. 테스트용 메일 발송 오브젝트 (소스 코드 p.388)
1. 스프링이 제공한 메일 전송 기능에 대한 인터페이스(MailSender)를 구현해 테스트용 메일 전송 클래스(DummyMailSender) 만들기.

2. 테스트 설정파일의 mailSender 빈 클래스를 JavaMail을 사용하는 JavaMailSenderImpl 대신 DummyMailSender로 변경
3. 테스트용 UserSErvice를 위한 메일 전송 오브젝트의 수동 DI

<br>

### 💥 현재까지 코드의 부족한 점 : 메일 발송 작업에 트랜잭션 개념 빠져 있음
> 레벨 업그레이드 작업 중간에 예외 발생해 DB에 반영됐던 레벨 업그레이드 모두 롤백됐다고 하자.
>
>하지만 메일은 사용자별로 업그레이드 처리할 때 이미 발송해버림. 그것을 어떻게 취소할 것인가?

<br>

### 💡 따라서, 메일 발송 기능에도 트랜잭션 기능을 적용하자.
#### 방법 1 : 메일을 업그레이드할 사용자를 발견할 때마다 발송하지 않고 발송 대상을 별도의 목록에 저장해둠
- 단점 : 메일 저장용 리스트 등을 파라미터로 계속 갖고 다녀야 함

#### 방법 2 : MailSender를 확장해 메일 전송에 트랜잭션 개념 적용
- 장점 : 서로 다른 종류의 작업을 분리해 처리 가능

<br>

#### ✅ 이 예제를 설명하면서 필자가 말하고자 하는 이야기
- 서비스 추상화는 테스트하기 어려운 JavaMail 같은 기술에도 적용 가능.

- 테스트를 편리하게 작성하도록 도와주는 것만으로도 서비스 추상화는 가치가 있음.

<br>

## ✅ 테스트 대역(test double)
>**테스트 대상이 사용하는 의존 오브젝트를 대체할 수 있도록 만든 오브젝트**
>
>테스트용으로 사용되는 특별한 오브젝트
>
>- 테스트 대상 오브젝트가 원활하게 동작할 수 있도록 도우면서 테스트를 위해 간접적인 정보를 제공해주기도 함.

#### 테스트 대역을 사용하는 이유 :
- 테스트 대상이 되는 오브젝트가 또 다른 오브젝트에 의존하는 경우는 흔하기 때문 -> 여러 가지 문제 발생

<br>

### ✅ 대표적 테스트 대역 1. 테스트 스텁(test stub)
> **테스트 대상 오브젝트의 의존객체로서 존재하면서 테스트 동안에 코드가 정상적으로 수행할 수 있도록 돕는 것**
>- 일반적으로 메소드를 통해 전달하는 파라미터와 달리, 테스트 코드 내부에서 간접적으로 사용됨.
>
>- 따라서, DI 등을 통해 미리 의존 오브젝트를 테스트 스텁으로 변경해야 함.
>- DummyMailSender는 가장 단순하고 심플한 테스트 스텁의 예
>- 간접적인 입력 값을 지정 가능, 간접적인 출력 값을 받게 할 수도 있음

<br>

### ✅ 대표적 테스트 대역 2. 목 오브젝트(mock object)
> **테스트 대상의 간접적인 출력 결과를 검증하고, 테스트 대상 오브젝트와 의존 오브젝트 사이에서 일어난 일을 검증할 수 있도록 설계된 것.**
> - 목 오브젝트는 스텁처럼 테스트 오브젝트가 정상적으로 실행되도록 도와주면서, 테스트 오브젝트와 자신의 사이에서 일어나는 커뮤니케이션 내용을 저장해뒀다가 테스트 결과를 검증하는 데 활용할 수 있게 해줌.

<br>

### 🧐 스텁 VS 목 오브젝트
>목 오브젝트는 간접 테스트 구간의 출력 확인이 있다.

<br>

### ✅ 목 오브젝트를 이용한 테스트
DummyMailSender 대신 새로운 MailSender를 대체할 클래스 MockMailSender 만들기

#### Feat: 목 오브젝트로 만든 메일 전송 확인용 클래스
```java
static class MockMailSender implements MailSender{
	private List<String> requests = new ArrayList<String>();

	public List<String> getRequests(){
		return requests;
	}
	
	public void send(SimpleMailMessage mailMessage) throws MailException {
		requests.add(mailMessage.getTo()[0]); // 전송 요청 받은 이메일 주소 저장. 간단하게 첫 번째 수신자 메일 주소만 저장함
	}
		
	public void send(SimpleMailMessage[] mailMessage) throws MailException {
	}
}
```
> 물론 메일 발송 기능은 없음.
> 
> UserService의 코드가 정상적으로 수행되도록 돕는 역할이 우선임.
> 
> 대신 테스트 대상이 넘겨주는 출력 값을 보관해두는 기능 추가

#### Fix: 목 오브젝트를 통해 메일 발송 여부 검증하도록 수정한 테스트
```java
@Test
@DirtiesContext // 컨텍스트의 DI 설정을 변경하는 테스트라는 것을 알려줌
public void upgradeLevels() throws Exception {
	userDao.deleteAll();
	for(User user : users) userDao.add(user);

	// 메일 발송 결과를 테스트할 수 있도록 목 오브젝트를 만들어 userService의 의존 오브젝트로 주입해줌
	MockMailSender mockMailSender = new MockMailSender();
	userService.setMailSender(mockMailSender);
	
	// 업그레이드 테스트, 메일 발송이 일어나면 MockMailSender 오브젝트의 리스트에 그 결과가 저장됨
	userService.upgradeLevels();

	checkLevelUpgraded(users.get(0), false);
	checkLevelUpgraded(users.get(1), true);
	checkLevelUpgraded(users.get(2), false);
	checkLevelUpgraded(users.get(3), true);
	checkLevelUpgraded(users.get(4), false);

	// 목 오브젝트에 저장된 메일 수신자 목록을 가져와 업그레이드 대상과 일치하는지 확인
	List<String> request = mockMailSender.getRequests();
	assertThat(request.size(), is(2));
	assertThat(request.get(0), is(users.get(1).getEmail()));
	assertThat(request.get(1), is(users.get(3).getEmail()));
}
```

테스트가 수행될 수 있도록 

- **의존 오브젝트에 간접적으로 입력 값을 제공**해주는 **스텁 오브젝트**

- **간접적인 출력 값까지 확인이 가능한 목 오브젝트**

이 두 가지는 **테스트 대역의 가장 대표적인 방법**이며 효과적인 테스트 코드를 작성하는 데 빠질 수 없는 중요한 도구다.