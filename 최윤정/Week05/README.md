# π ν λΉμ μ€νλ§ 3.1 Vol.1 μ€νλ§μ μ΄ν΄μ μλ¦¬
## π 5μ₯ μλΉμ€ μΆμν
#### β¨ 5μ₯μμ λ€λ£¨λ κ² : μ§κΈκΉμ§ λ§λ  DAOμ νΈλμ­μμ μ μ©ν΄λ³΄λ©΄μ μ€νλ§μ΄ μ΄λ»κ² μ±κ²©μ΄ λΉμ·ν μ¬λ¬ μ’λ₯μ κΈ°μ μ μΆμννκ³  μ΄λ₯Ό μΌκ΄λ λ°©λ²μΌλ‘ μ¬μ©ν  μ μλλ‘ μ§μνλμ§

## 5.1 μ¬μ©μ λ λ²¨ κ΄λ¦¬ κΈ°λ₯ μΆκ°
κ°λ¨ν λΉμ¦λμ€ λ‘μ§μ μΆκ°ν΄λ³΄μ.
- μ§κΈκΉμ§ λ§λ€μλ `UserDao`λ₯Ό **λ€μμ νμμ΄ κ°μν  μ μλ μΈν°λ· μλΉμ€μ μ¬μ©μ κ΄λ¦¬ λͺ¨λμ μ μ©**νλ€κ³  μκ°ν΄λ³΄μ.

	- **μ¬μ©μ κ΄λ¦¬ κΈ°λ₯** : μ λ³΄ λ£κ³  κ²μ + μ κΈ°μ μΌλ‘ μ¬μ©μμ νλλ΄μ­ μ°Έκ³ ν΄ λ λ²¨ μ‘°μ ν΄μ£Όλ κΈ°λ₯ νμ

<br>

### β μΈν°λ· μλΉμ€μ μ¬μ©μ κ΄λ¦¬ κΈ°λ₯μμ κ΅¬νν΄μΌ ν  λΉμ¦λμ€ λ‘μ§
- μ¬μ©μμ λ λ²¨μ BASIC, SILVER, GOLD μΈ κ°μ§ μ€ νλ
- μ¬μ©μκ° μ²μ κ°μνλ©΄ BASIC λ λ²¨μ΄ λλ©°, μ΄ν νλμ λ°λΌμ ν λ¨κ³μ© μκ·Έλ μ΄λλ  μ μμ
- κ°μ ν 50ν μ΄μ λ‘κ·ΈμΈμ νλ©΄ BASICμμ SILVER λ λ²¨μ΄ λ¨
- SILVER λ λ²¨μ΄λ©΄μ 30λ² μ΄μ μΆμ² λ°μΌλ©΄ GOLD λ λ²¨μ΄ λ¨
- μ¬μ©μ λ λ²¨μ λ³κ²½ μμμ μΌμ ν μ£ΌκΈ°λ₯Ό κ°μ§κ³  μΌκ΄μ μΌλ‘ μ§νλ¨. λ³κ²½ μμ μ μλ μ‘°κ±΄μ μΆ©μ‘±νλλΌλ λ λ²¨μ λ³κ²½μ΄ μΌμ΄λμ§ μμ.

<br>

### β Step 1. νλ μΆκ° 
μ μν μμ κ°μΌλ‘ μ μν μ¬μ©μ λ λ²¨(`private static final int BASIC = 1;`)μ λ€λ₯Έ μ’λ₯μ μ λ³΄λ₯Ό λ£λ μ€μν΄λ μ»΄νμΌλ¬κ° μ²΄ν¬λͺ»ν¨ + λ²μ λ²μ΄λλ κ° λ£μ μν μμ.

#### μλ° 5 μ΄μμμ μ κ³΅νλ `enum` μ΄μ©νμ
#### Feat: μ¬μ©μ λ λ²¨μ© `enum` μΆκ°
```java
package springbook.user.domain;
...
public enum Level{
	BASIC(1), SILVER(2), GOLD(3); // μΈ κ°μ enum μ€λΈμ νΈ μ μ
	
	private final int value;
	
	Level(int value){ // DBμ μ μ₯ν  κ°μ λ£μ΄μ€ μμ±μ 
		this.value = value;
	}
	
	public int intValue(){ // κ°μ κ°μ Έμ€λ λ©μλ
		return value;
	}
	
	public static Level valueOf(int value){ // κ°μΌλ‘λΆν° Level νμ μ€λΈμ νΈλ₯Ό κ°μ Έμ€λλ‘ λ§λ  μ€νν± λ©μλ
		switch(value){
			case 1: return BAISC;
			case 2: return SILVER;
			case 3: return GOLD;
			default: throw new AssertionError("Unknown value: " + value);
		}
	}
}
```
> λ΄λΆμλ DBμ μ μ₯ν  int νμμ κ°μ κ°κ³  μμ§λ§, κ²μΌλ‘λ Level νμμ μ€λΈμ νΈμ΄κΈ° λλ¬Έμ μμ νκ² μ¬μ©ν  μ μμ. 

#### Feat: User νλ μΆκ°
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
	// login, recommend getter/setter μλ΅
}
```

#### DBμ USER νμ΄λΈμλ νλ μΆκ°
|νλλͺ|νμ|μ€μ |
|--|--|--|
|Level|tinyint|Not Null|
|Login|int|Not Null|
|Recommend|int|Not Null|

#### Fix: UserDaoTest μμ 
```java
public class UserDaoTest {
	...
	@Before
	public void setUp(){
		this.user1 = new User("gyumee", "λ°μ±μ² ", "springno1", Level.BASIC, 1, 0);
		this.user2 = new User("dobby", "μ€λλΉ", "springno2", Level.SILVER, 55, 10);
		this.user3 = new User("bumjin", "λ°λ²μ§", "springno3", Level.GOLD, 100, 40);
	}
}
```
#### Fix: μΆκ°λ νλλ₯Ό νλΌλ―Έν°λ‘ ν¬ν¨νλ μμ±μ
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
#### Fix: μλ‘μ΄ νλλ₯Ό ν¬ν¨νλ User νλ κ° κ²μ¦ λ©μλ μμ 
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
#### Fix: `checkSameUser()` λ©μλλ₯Ό μ¬μ©νλλ‘ μμ ν `addAndGet()` λ©μλ
```java
@Test public void addAndGet(){
	...
	User userget1 = dao.get(user1.getId());
	checkSameUser(userget1, user1);
	
	User userget2 = dao.get(user2.getId());
	checkSameUser(userget2, user2);
}
```
#### Fix: μΆκ°λ νλλ₯Ό μν UserDaoJdbc μμ  
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
> Level enumμ μ€λΈμ νΈμ΄λ―λ‘ DBμ μ μ₯λ  μ μλ SQL νμμ΄ μλ. 
>
>DBμ μ μ₯ν  κ²½μ°, Level enum -> intνμΌλ‘ λ³ν : λ―Έλ¦¬ λ§λ€μ΄λ `intValue()` λ©μλ μ¬μ©
>
>μ‘°νν  κ²½μ°, int -> Level enumμΌλ‘ λ³ν : λ―Έλ¦¬ λ§λ€μ΄λ μ€νν± λ©μλ `valueOf()` μ΄μ©

<br>

### β Step 2. μ¬μ©μ μμ  κΈ°λ₯ μΆκ°
μ¬μ©μ κ΄λ¦¬ λΉμ¦λμ€ λ‘μ§μ λ°λ₯΄λ©΄, μ¬μ©μ μ λ³΄λ μ¬λ¬ λ² μμ λ  μ μμ.

μμ ν  μ λ³΄κ° λ΄κΈ΄ User μ€λΈμ νΈλ₯Ό μ λ¬ -> idλ₯Ό μ°Έκ³ ν΄μ μ¬μ©μ μ°Ύμ -> νλ μ λ³΄λ₯Ό UPDATEλ¬Έμ μ΄μ©ν΄ λͺ¨λ λ³κ²½ν΄μ£Όλ λ©μλλ₯Ό νλ λ§λ€μ.

#### Feat: μ¬μ©μ μ λ³΄ μμ  λ©μλ νμ€νΈ μΆκ°
```java
@Test
public void update() {
	dao.deleteAll();

	dao.add(user1);
	
	user1.setName("μ€λ―Όκ·");
	user1.setPassword("springno6");
	user1.setLevel(Level.GOLD);
	user1.setLogin(1000);
	user1.setRecommend(999);
	dao.update(user1);

	User user1update = dao.get(user1.getId());
	checkSameUser(user1, user1update);
}
```
> 1. ν½μ€μ² μ€λΈμ νΈ νλ λ±λ‘
> 
> 2. idλ₯Ό μ μΈν νλμ λ΄μ© λ°κΎΌ λ€ update() νΈμΆ
> 3. λ€μ idλ‘ μ‘°νν΄μ κ°μ Έμ¨ User μ€λΈμ νΈμ μμ ν ν½μ€μ² μ€λΈμ νΈλ₯Ό λΉκ΅

#### Feat: UserDao μΈν°νμ΄μ€μ `update()` λ©μλ μΆκ°
```java
public interface UserDao {
	...
	public void update(User user1);
}
```
#### Feat: UserDaoJdbcμ μ¬μ©μ μ λ³΄ μμ μ© `update()` λ©μλ μΆκ°
```java
public void update(User user){
	this.jdbcTemplate.update("update users set name = ?, password = ?, level = ?, login = ?, " + "recommend = ? where id = ?", 
		user.getName(), user.getPassword(), user.getLevel().intValue(), user.getLogin(), user.getRecommend(), user.getId());
}
```

### π§ μμ μμ±ν `update()` νμ€νΈμ λ¬Έμ μ κ³Ό ν΄κ²°λ°©λ²
- **λ¬Έμ **
	- νμ¬ `update()` νμ€νΈλ μμ ν  λ‘μ°μ λ΄μ©μ΄ λ°λ κ²λ§ νμΈν  λΏ, μμ νμ§ μμμΌ ν  λ‘μ°μ λ΄μ©μ΄ κ·Έλλ‘ λ¨μ μλμ§λ νμΈν΄μ£Όμ§ λͺ»ν¨.

- **ν΄κ²° λ°©λ²** 
	1. JdbcTemplateμ `update()` κ° λλ €μ£Όλ λ¦¬ν΄κ° νμΈ

	2.  νμ€νΈλ₯Ό λ³΄κ°ν΄μ μνλ μ¬μ©μ μΈμ μ λ³΄λ λ³κ²½λμ§ μμμμ μ§μ  νμΈ => μ¬κΈ°μλ μ΄ λ°©λ² μ μ©ν΄λ³΄μ!

		- μ¬μ©μλ₯Ό λ λͺ λ±λ‘ν΄λκ³ , κ·Έμ€ νλλ§ μμ ν λ€μ μμ λ μ¬μ©μμ μμ νμ§ μμ μ¬μ©μμ μ λ³΄λ₯Ό λͺ¨λ νμΈνλ©΄ λ¨ 

#### Fix: `update()` νμ€νΈ λ³΄μ
```java
@Test
public void update(){
	dao.deleteAll();

	dao.add(user1); // μμ ν  μ¬μ©μ
	dao.add(user2); // μμ νμ§ μμ μ¬μ©μ
	
	user1.setName("μ€λ―Όκ·");
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

### β Step 3. λ³Έκ²©μ μΈ μ¬μ©μ κ΄λ¦¬ λΉμ¦λμ€ λ‘μ§ κ΅¬ν
λ°μ΄ν° μ‘μΈμ€ κΈ°λ₯μ UserDaoμ `getAll()` λ©μλλ‘ μ¬μ©μλ₯Ό λ€ κ°μ Έμμ μ¬μ©μλ³λ‘ λ λ²¨ μκ·Έλ μ΄λ μμ μ§ν + UserDaoμ `update()` νΈμΆν΄ DBμ κ²°κ³Ό λ£μ΄μ£Όλ©΄ λ¨.

#### μ¬μ©μ κ΄λ¦¬ λ‘μ§μ μ΄λμ λλ κ²μ΄ μ’μκΉ?
> μ¬μ©μ κ΄λ¦¬ λΉμ¦λμ€ λ‘μ§μ λ΄μ ν΄λμ€ UserServiceλ₯Ό νλ μΆκ°νμ.
>
>UserServiceλ UserDao μΈν°νμ΄μ€ νμμΌλ‘ userDao λΉμ DI λ°μ μ¬μ©νκ² λ§λ¦.
>- π½ UserDaoλ μΈν°νμ΄μ€ μ΄λ¦, userDaoλ λΉ μ€λΈμ νΈ μ΄λ¦μ

#### Feat: UserService ν΄λμ€ μΆκ°
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
#### Feat: μ€νλ§ μ€μ νμ΄μ userService λΉ μ€μ  μΆκ°
```xml
<bean id="userService" class="springbook.user.service.UserService">
	<property name="userDao" ref="userDao" />
</bean>
<bean id="userDao" class="springbook.dao.UserDaoJdbc">
	<property name="dataSource" ref="dataSource" />
</bean>
```
#### Feat: UserServiceTest ν΄λμ€ μΆκ°
```java
package springbook.user.service;
...
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
public class UserServiceTest{
	@Autowired // νμ€νΈ λμμΈ UserService λΉμ μ κ³΅λ°μ μ μλλ‘ 
	UserService userService;
}
```
#### Feat: userService λΉμ΄ μμ±λΌμ userService λ³μμ μ£Όμλλμ§ νμΈνλ νμ€νΈ λ©μλ μΆκ°
```java
@Test
public void bean() {
	assertThat(this.userService, is(notNullValue()));
}
```
> νμ€νΈκ° μ±κ³΅νλ©΄ UserService λΉμ΄ μ λ±λ‘λμμ μ μ μμ.
>
> μ΄ν userService μ€λΈμ νΈλ₯Ό μΆκ°νλ©΄ `bean()` νμ€νΈλ λ³ μλ―Έκ° μμΌλ μ­μ ν΄λ μ’μ.
#### Feat: μ¬μ©μ λ λ²¨ μκ·Έλ μ΄λ λ©μλ μΆκ°
```java
public void upgradeLevels(){
	List<User> users = userDao.getAll();
	for(User user : users){
		Boolean changed = null; // λ λ²¨μ λ³νκ° μλμ§λ₯Ό νμΈνλ νλκ·Έ
		if (user.getLevel() == Level.BASIC && user.getLogin() >= 50){ // BASIC λ λ²¨ μκ·Έλ μ΄λ μμ
			user.setLevel(Level.SILVER);
			changed = true;
		}
		else if(user.getLevel() == Level.SILVER && user.getRecommend() >= 30){ // SILVER λ λ²¨ μκ·Έλ μ΄λ μμ
			user.setLevel(Level.GOLD);
			changed = true;
		}
		else if(user.getLevel() == Level.GOLD) {
			changed = false; // GOLD λ λ²¨μ λ³κ²½μ΄ μΌμ΄λμ§ μμ
		}
		else {
			changed = false; // μΌμΉνλ μ‘°κ±΄μ΄ μμΌλ©΄ λ³κ²½ μμ.
		}

		if(changed) { // λ λ²¨μ λ³κ²½μ΄ μλ κ²½μ°λ§ udpate() νΈμΆ
			userDao.update(user);
		}
	}
}
```
#### μ΄λ₯Ό νμ€νΈ νλ λ°©λ²?
- μ μ΄λ κ°λ₯ν λͺ¨λ  μ‘°κ±΄μ νλμ©μ νμΈν΄λ΄μΌ ν¨.

- μ¬μ©μ λ λ²¨μ BASIC, SILVER, GOLD μΈ κ°μ§, λ³κ²½μ΄ μΌμ΄λμ§ μλ GOLDλ₯Ό μ μΈν λλ¨Έμ§ λ κ°μ§λ μκ·Έλ μ΄λκ° λλ κ²½μ°μ μλ κ²½μ° μ‘΄μ¬ => μ΅μν 5κ°μ§ μ΄ν΄λ΄μΌ ν¨.

**λ€μ― μ’λ₯μ μ¬μ©μ μ λ³΄λ₯Ό λ±λ‘ν΄λκ³  μκ·Έλ μ΄λλ₯Ό μ§νν νμ μμν λλ‘ κ²°κ³Όκ° λμ€λμ§ νμΈν΄λ³΄μ.**
#### Feat: λ¦¬μ€νΈλ‘ λ§λ  νμ€νΈ ν½μ€μ² μΆκ°
```java
class UserServiceTest {
	...
	List<User> users; // νμ€νΈ ν½μ€μ²(νμ€νΈ μ€νμ μν΄ λ² μ΄μ€λΌμΈμΌλ‘μ μ¬μ©λλ κ°μ²΄λ€μ κ³ μ λ μν)
	
	@Before
	public void setUp() {
		users = Arrays.asList(
			new User("bumjin", "λ°λ²μ§", "p1", Level.BASIC, 49, 0),
			new User("joytouch", "κ°λͺμ±", "p2", Level.BASIC, 50, 0),
			new User("erwins", "μ μΉν", "p3", Level.SILVER, 60, 29),
			new User("madnite1", "μ΄μνΈ", "p4", Level.SILVER, 60, 30),
			new User("green", "μ€λ―Όκ·", "p5", Level.GOLD, 100, 100)
		);
	}
}
```
#### Feat: μ¬μ©μ λ λ²¨ μκ·Έλ μ΄λ λ©μλ νμ€νΈ μΆκ°
```java
@Test
public void upgradeLevels() {
	userDao.deleteAll();
	for(User user : users) userDao.add(user);
	
	userService.upgradeLevels();

	// κ° μ¬μ©μλ³λ‘ μκ·Έλ μ΄λ νμ μμ λ λ²¨ κ²μ¦
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
> 1. λ¨Όμ  μ€λΉν λ€μ― κ°μ§ μ’λ₯μ μ¬μ©μ μ λ³΄ μ μ₯ 
>
> 2. upgradeLevels() λ©μλ μ€ν
> 
> 3. μκ·Έλ μ΄λ μμ λλλ©΄ μ¬μ©μ μ λ³΄λ₯Ό νλμ© κ°μ Έμ λ λ²¨μ λ³κ²½ μ¬λΆ νμΈ

<br>

### β Step 4. μ²μ κ°μν μ¬μ©μλ κΈ°λ³Έμ μΌλ‘ BASIC λ λ²¨μ΄μ΄μΌ ν¨
λ¨Όμ  νμ€νΈλΆν° λ§λ€μ΄λ³΄μ.

#### νμ€νΈ μΌμ΄μ€λ λ μ’λ₯λ₯Ό λ§λ€λ©΄ λ¨.
1οΈβ£ λ λ²¨μ΄ λ―Έλ¦¬ μ ν΄μ§ κ²½μ°

2οΈβ£ λ λ²¨μ΄ λΉμ΄μλ κ²½μ° 
> κ°κ° add() λ©μλ νΈμΆ ν κ²°κ³Ό νμΈνλλ‘ λ§λ€μ.

<br>

#### User μ€λΈμ νΈμ λ λ²¨μ΄ λ³κ²½λλμ§ νμΈνκΈ° μν΄ μ¬μ©ν  μ μλ λ κ°μ§ λ°©λ²
1οΈβ£ UserServiceμ `add()` λ©μλλ₯Ό νΈμΆν  λ νλΌλ―Έν°λ‘ λκΈ΄ User μ€λΈμ νΈμ level νλλ₯Ό νμΈν΄λ³΄κΈ°

2οΈβ£ UserDaoμ `get()` λ©μλλ₯Ό μ΄μ©ν΄ DBμ μ μ₯λ User μ λ³΄λ₯Ό κ°μ Έμ νμΈνκΈ° -> μ΄κ² κ°μ₯ νμ€ν λ°©λ²
>  λ κ°μ§ λ€ ν΄λ μ’κ³ , νμλ§ ν΄λ κ΄μ°?μ!

<br>

#### νμ€νΈλΆν° λ§λ  λ€, λ©μλ μ½λ λ§λ€μ΄λ³΄μ.
#### Feat: `add()` λ©μλμ νμ€νΈ μΆκ°
```java
@Test
public void add(){
	userDao.deleteAll();
	
	User userWithLevel = users.get(4); // GOLD λ λ²¨
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
> 1. μ΄λ―Έ λ λ²¨μ΄ GOLDλ‘ μ€μ λ μ¬μ©μμ λ λ²¨μ΄ nullμΈ μ¬μ©μ μ€λΈμ νΈ λ κ° μ€λΉ
> 
> 2. μ€λΉν λ κ°μ μ¬μ©μ μ€λΈμ νΈλ₯Ό UserServiceμ `add()` λ©μλλ₯Ό ν΅ν΄ μ΄κΈ°ν -> DBμ μ μ₯νλλ‘ 
> 3. νμΈμ μν΄ DBμμ μ¬μ©μ μ λ³΄ μ½μ΄μ€κΈ°
> 4. λ λ²¨μ΄ μ΄λ―Έ μ€μ λλ κ²μ κ·Έλλ‘ μ μ§λμ΄ μμ΄μΌ νκ³ , λ λ²¨μ΄ μλ κ²μ λν΄νΈμΈ BASICμΌλ‘ μ€μ λλμ§ νμΈ

μμ κ°μ νμ€νΈκ° μ±κ³΅νλλ‘ μ½λλ₯Ό λ§λ€μ΄λ³΄μ.

#### Feat: μ¬μ©μ μ κ· λ±λ‘ λ‘μ§μ λ΄μ `add()` λ©μλ
```java
public void add(User user){
	if(user.getLevel() == null) user.setLevel(Level.BASIC);
	userDao.add(user);
}
```

<br>

### β Step 5. μ½λ κ°μ  
#### μμ±λ μ½λλ₯Ό μ΄ν΄λ³Ό λ, λ€μκ³Ό κ°μ μ§λ¬Έμ ν΄λ³Ό νμκ° μλ€.
- μ½λμ **μ€λ³΅**λ λΆλΆμ μλκ°?

- μ½λκ° λ¬΄μμ νλ κ²μΈμ§ **μ΄ν΄νκΈ° λΆνΈ**νμ§ μμκ°?
- μ½λκ° **μμ μ΄ μμ΄μΌ ν  μλ¦¬**μ μλκ°?
- μμΌλ‘ **λ³κ²½**μ΄ μΌμ΄λλ€λ©΄ μ΄λ€ κ²μ΄ μμ μ μκ³ , κ·Έ **λ³νμ μ½κ² λμ**ν  μ μκ² μμ±λμ΄ μλκ°?

#### π₯ `upgradeLevels()` λ©μλ μ½λμ λ¬Έμ μ 
1. for λ£¨ν μ if/elseif/else λΈλ‘ μ½κΈ° λΆνΈ

2. νμ¬ νλ²¨κ³Ό μκ·Έλ μ΄λ μ‘°κ±΄μ λμμ λΉκ΅νλ λΆλΆ
	- μ±κ²©μ΄ λ€λ₯Έ λ κ°μ§ κ²½μ°κ° λͺ¨λ ν κ³³μμ μ²λ¦¬λλ κ²μ΄ λ­κ° μ΄μ

#### π‘ ν΄κ²°λ°©λ² : μ‘°κ±΄μ λ λ¨κ³μ κ±Έμ³μ λΉκ΅νλ€
1. λ λ²¨ νμΈνκΈ°

2. κ° λ λ²¨λ³λ‘ λ€μ μ‘°κ±΄ νλ¨νλ μ‘°κ±΄μ λ£κΈ°

<br>

### β `upgradeLevels()` λ¦¬ν©ν λ§
λ λ²¨μ μκ·Έλ μ΄λνλ μμμ κΈ°λ³Έ νλ¦λ§ λ§λ€μ.

κ΅¬μ²΄μ μΈ κ΅¬νμμ μΈλΆμ λΈμΆν  μΈν°νμ΄μ€λ₯Ό λΆλ¦¬νλ κ²κ³Ό λ§μ°¬κ°μ§ μμμ΄λΌ μκ°νλ©΄ λ¨.
#### Refactor: κΈ°λ³Έ μμ νλ¦λ§ λ¨κ²¨λ `upgradeLevels()`
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
> λͺ¨λ  μ¬μ©μ μ λ³΄ κ°μ Έμ ν λͺμ© μκ·Έλ μ΄λκ° κ°λ₯νμ§ νμΈ -> κ°λ₯νλ©΄ μκ·Έλ μ΄λ
#### Refactor: μκ·Έλ μ΄λ κ°λ₯ νμΈ λ©μλ μΆκ°
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
> μ£Όμ΄μ§ userμ λν΄ μκ·Έλ μ΄λκ° κ°λ₯νλ©΄ true, κ°λ₯νμ§ μμΌλ©΄ false λ¦¬ν΄
> 
> μνμ λ°λΌμ μκ·Έλ μ΄λ μ‘°κ±΄λ§ λΉκ΅νλ©΄ λλ―λ‘, μ­ν κ³Ό μ±μμ΄ λͺλ£ν΄μ§
#### π₯ Refactor: λ λ²¨ μκ·Έλ μ΄λ μμ λ©μλ μΆκ° (μλ μ½μ΄λ³΄λ©΄ μκ² μ§λ§ λ³λ‘μΈ λ°©λ²μ)
```java
private void upgradeLevel(User user){
	if(user.getLevel() == Level.BASIC) user.setLevel(Level.SILVER);
	else if(user.getLevel() == Level.SILVER) user.setLevel(Level.GOLD);
	userDao.update(user);
}
```
> μ¬μ©μ μ€λΈμ νΈμ λ λ²¨μ λ³΄λ₯Ό λ€μ λ¨κ³λ‘ λ³κ²½, λ³κ²½λ μ€λΈμ νΈλ₯Ό DBμ μλ°μ΄νΈ
> 
> ### κ·Έλ°λ°, 
> #### π₯ λ¬Έμ μ  : λ€μ λ¨κ³κ° λ¬΄μμΈκ° νλ λ‘μ§κ³Ό κ·Έ λ μ¬μ©μ μ€λΈμ νΈμ level νλλ₯Ό λ³κ²½ν΄μ€λ€λ λ‘μ§μ΄ ν¨κ» μκ³ , μμΈμν© μ²λ¦¬ x
> #### π‘ ν΄κ²°λ°©λ² : λ λΆλ¦¬νμ. λ λ²¨μ μμμ λ€μ λ¨κ³ λ λ²¨μ΄ λ¬΄μμΈμ§λ₯Ό κ²°μ νλ μΌμ Levelμκ² λ§‘κΈ°μ.
#### π‘ Fix: μκ·Έλ μ΄λ μμλ₯Ό λ΄κ³  μλλ‘ Level μμ 
```java
public enum Level{

	// enum μ μΈμ DBμ μ μ₯ν  κ°, λ€μ λ¨κ³ λ λ²¨ μ λ³΄λ μΆκ°ν¨
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
#### π‘ Fix: μ¬μ©μ μ λ³΄κ° λ°λλ λΆλΆμ Userμ μΆκ°
```java
public void upgradeLevel(){
	Level nextLevel = this.level.nextLevel();
	if(nextLevel == null){
		throw new IllegalStateException(this.level + "μ μκ·Έλ μ΄λκ° λΆκ°λ₯ν©λλ€");
	}
	else{
		this.level = nextLevel;
	}
}
```
#### π‘ Fix: κ°κ²°ν΄μ§ `upgradeLevel()`
```java
private void upgradeLevel(User user){
	user.upgradeLevel();
	userDao.update(user);
}
```

<br>

#### μ§κΈκΉμ§ κ°μ ν μ½λ
- κ° μ€λΈμ νΈμ λ©μλκ° κ°κ° μκΈ° λͺ«μ μ±μ λ§‘μ μΌμ νλ κ΅¬μ‘°

- UserService, User, Levelμ΄ λ΄λΆ μ λ³΄ λ€λ£¨λ μμ μ μ±μμ μΆ©μ€ν κΈ°λ₯ κ°κ³  μμΌλ©° νμκ° μκΈ°λ©΄ μ΄λ° μμ μνν΄λ¬λΌκ³  μλ‘ μμ²­νλ κ΅¬μ‘°
- μ½λ μ΄ν΄νκΈ° μ½κ³  λ³ν λμνκΈ° νΈν¨

>κ°μ²΄μ§ν₯μ μΈ μ½λλ λ€λ₯Έ μ€λΈμ νΈμ λ°μ΄ν°λ₯Ό κ°μ Έμμ μμνλ λμ  λ°μ΄ν°λ₯Ό κ°κ³  μλ λ€λ₯Έ μ€λΈμ νΈμκ² μμμ ν΄λ¬λΌκ³  μμ²­.
>
>**μ€λΈμ νΈμκ² λ°μ΄ν°λ₯Ό μκ΅¬νμ§ λ§κ³  μμμ μμ²­νλΌλ κ²**μ΄ **κ°μ²΄μ§ν₯ νλ‘κ·Έλλ°μ κ°μ₯ κΈ°λ³Έμ΄ λλ μλ¦¬**μ΄κΈ°λ ν¨.

<br>

### β μ§κΈκΉμ§ λ¦¬ν©ν λ§ν μ½λμ λ§κ² λ€λ₯Έ μ½λλ€λ μμ νμ.
#### Feat: Userμ μΆκ°ν `upgradeLevel()` λ©μλ νμ€νΈ μΆκ°
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
 #### Fix: UserServiceTestμ `upgradeLevels()` νμ€νΈ κ°μ  
```java
@Test
public void upgradeLevels() {
	userDao.deleteAll();
	for(User user : users) userDao.add(user);
	
	userService.upgradeLevels();

	// κ° μ¬μ©μλ³λ‘ μκ·Έλ μ΄λ νμ μμ λ λ²¨ κ²μ¦
	checkLevelUpgraded(users.get(0), false);
	checkLevelUpgraded(users.get(1), true);
	checkLevelUpgraded(users.get(2), false);
	checkLevelUpgraded(users.get(3), true);
	checkLevelUpgraded(users.get(4), false);
}

private void checkLevelUpgraded(User user, Boolean upgraded){
	User userUpdate = userDao.get(user.getId());
	if(upgraded){
		assertThat(userUpdate.getLevel(), is(user.getLevel().nextLevel())); // μλ°μ΄νΈκ° μΌμ΄λ¬λμ§ νμΈ
	}
	else{
		assertThat(userUpdate.getLevel(), is(user.getLevel())); // μλ°μ΄νΈκ° μΌμ΄λμ§ μμλμ§ νμΈ	
	}
}
```
#### Fix: UserService μ μμ λμ
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
#### Fix: μμ μ¬μ©νλλ‘ νμ€νΈλ μμ 
```java
import static springbok.user.service.UserService.MIN_LOGCOUNT_FOR_SILVER;
import static springbok.user.service.UserService.MIN_RECOMMEND_FOR_GOLD;

class UserServiceTest {
	...
	@Before
	public void setUp() {
		users = Arrays.asList(
			new User("bumjin", "λ°λ²μ§", "p1", Level.BASIC, MIN_LOGCOUNT_FOR_SILVER-1, 0),
			new User("joytouch", "κ°λͺμ±", "p2", Level.BASIC, MIN_LOGCOUNT_FOR_SILVER, 0),
			new User("erwins", "μ μΉν", "p3", Level.SILVER, 60, MIN_RECOMMEND_FOR_GOLD-1),
			new User("madnite1", "μ΄μνΈ", "p4", Level.SILVER, 60, MIN_RECOMMEND_FOR_GOLD),
			new User("green", "μ€λ―Όκ·", "p5", Level.GOLD, 100, Integer.MAX_VALUE)
		);
	}
}
```

π€ μ°λ§ μ΄λ²€νΈλ μλ‘μ΄ μλΉμ€ νλ³΄ κΈ°κ° μ€ λ λ²¨ μκ·Έλ μ΄λ μ μ±μ λ€λ₯΄κ² μ μ©ν  νμκ° μμ κ²½μ°, μ¬μ©μ μκ·Έλ μ΄λ μ μ±μ λ΄μ μΈν°νμ΄μ€λ₯Ό λ§λ€μ΄λκ³  UserServiceλΌλ DIλ‘ μ κ³΅λ°μ μ μ± κ΅¬ν ν΄λμ€λ₯Ό μ΄ μΈν°νμ΄μ€λ₯Ό ν΅ν΄ μ¬μ©νμ.

#### Feat: μκ·Έλ μ΄λ μ μ± μΈν°νμ΄μ€
```java
public interface UserLevelUpgradePolicy{
	boolean canUpgradeLevel(User user);
	void upgradeLevel(User user);
}
```

<br>

## 5.2 νΈλμ­μ μλΉμ€ μΆμν
"μ κΈ° μ¬μ©μ λ λ²¨ κ΄λ¦¬ μμμ μννλ λμ€ λ€νΈμν¬κ° λκΈ°κ±°λ μλ²μ μ₯μ κ° μκ²¨ μμμ μλ£ν  μ μλ€λ©΄, κ·ΈλκΉμ§ λ³κ²½λ μ¬μ©μμ λ λ²¨μ κ·Έλλ‘ λκΉ? μλ λͺ¨λ μ΄κΈ° μνλ‘ λλλ € λμμΌ ν κΉ?"
> κ·ΈλκΉμ§ μ§νλ λ³κ²½ μμλ λͺ¨λ μ·¨μμν€κΈ°λ‘ νμ.

### κ·Έλ λ€λ©΄ μ§κΈκΉμ§ λ§λ  μ¬μ©μ λ λ²¨ μκ·Έλ μ΄λ μ½λλ μ΄λ»κ² λμν κΉ?
#### π λͺ¨λ  μ¬μ©μμ λν΄ μκ·Έλ μ΄λ μμμ μ§ννλ€κ° μ€κ°μ μμΈ λ°μν΄μ μμ μ€λ¨λλ€λ©΄ μ΄λ»κ² λ κΉ?

μ΄λ―Έ λ³κ²½λ μ¬μ©μμ λ λ²¨μ μμ μ΄μ  μνλ‘ λμκ°κΉ?

μλλ©΄ λ°λ μ±λ‘ λ¨μ μμκΉ?

<br>

### β νμ€νΈμ©μΌλ‘ UserService λμ­ λ§λ€μ΄ μμΈ κ°μ λ‘ λ§λ€κ³  νμΈν΄λ³΄μ.

#### Step 1. νμ€νΈμ© UserService νμ₯ ν΄λμ€μΈ TestUserService ν΄λμ€ λ§λ€κΈ°
- κ°λ¨ν UserServiceλ₯Ό μμ ν νμ€νΈμ νμν κΈ°λ₯ μΆκ°νλλ‘ μΌλΆ λ©μλλ₯Ό μ€λ²λΌμ΄λ©

- μ΄ λ, UserServiceμ `upgradeLevel()` λ©μλ μ κ·ΌκΆνμ `protected`λ‘ μμ ν΄ μμμ ν΅ν΄ μ€λ²λΌμ΄λ©μ΄ κ°λ₯νλλ‘ ν¨

- μ€λ²λΌμ΄λλ `upgradeLevel()` λ©μλλ UserService λ©μλ κΈ°λ₯ κ·Έλλ‘ μν νμ§λ§ λ―Έλ¦¬ μ§μ λ idλ₯Ό κ°μ§ μ¬μ©μκ° λ°κ²¬λλ©΄ κ°μ λ‘ μμΈ λμ§λλ‘ κ΅¬ν

#### Step 2. λ€λ₯Έ μμΈκ° λ°μνμ κ²½μ°μ κ΅¬λΆνκΈ° μν΄ νμ€νΈ λͺ©μ μ λ€ μμΈλ₯Ό μλμ κ°μ΄ μ μ
```java
static class TestUserServiceException extends RuntimeException{
}
```
#### Step 3. μμΈ λ°μ μ μμ μ·¨μ μ¬λΆ νμ€νΈ μ½λ μΆκ°
νμ€νΈμ λͺ©μ  : 
- μ¬μ©μ λ λ²¨ μκ·Έλ μ΄λλ₯Ό μλνλ€κ° μ€κ°μ μμΈ λ°μν  κ²½μ°, κ·Έ μ μ μκ·Έλ μ΄λνλ μ¬μ©μλ λ€μ μλ μνλ‘ λμκ°λμ§ νμΈ
- μμ€ μ½λ p.351 

#### μμ νμ€νΈλ μ€ν¨νλ€. κ·Έ μμΈμ?
> β νΈλμ­μ λ¬Έμ  λλ¬Έ.
>
>λͺ¨λ  μ¬μ©μμ λ λ²¨μ μκ·Έλ μ΄λνλ μμμΈ `upgradeLevels()` λ©μλκ° νλμ νΈλμ­μ μμμ λμνμ§ μμκΈ° λλ¬Έ

<br>

## β νΈλμ­μ
#### λ μ΄μ λλ μ μλ λ¨μ μμ
**μμμ μͺΌκ°μ μμ λ¨μλ‘ λ§λ€ μ μλ€**λ κ²μ **νΈλμ­μμ ν΅μ¬ μμ±**μΈ **μμμ±**μ μλ―Έν¨.
> μμ μμλ‘ μ€λͺνμλ©΄, λͺ¨λ  μ¬μ©μμ λν λ λ²¨ μκ·Έλ μ΄λ μμμ μλ‘ μΆκ°λ κΈ°μ  μκ΅¬μ¬ν­λλ‘ μ μ²΄κ° λ€ μ±κ³΅νλ μ§ μλλ©΄ μ μ²΄κ° λ€ μ€ν¨νλ μ§ ν΄μΌ ν¨. 
> 
λ μ΄μμ μͺΌκ°μ§ μ μλ λ¬Όμ§μ΄λΌλ μλ―Έλ‘ μ΄λ¦μ λΆμΈ μμμ λ§μ°¬κ°μ§λ‘, μ΄ μμλ λ μ΄μ μͺΌκ°μ μ΄λ€μ§ μ μλ μμμ κ°μ μ±μ§μ λ.

λ°λΌμ, μ€κ°μ μμΈκ° λ°μν΄μ μμμ μλ£ν  μ μλ€λ©΄ μμ μμμ΄ μμλμ§ μμ κ²μ²λΌ μ΄κΈ° μνλ‘ λλ €λμΌ ν¨.

<br>

### β νΈλμ­μ κ²½κ³μ€μ 
DBλ κ·Έ μμ²΄λ‘ μλ²½ν νΈλμ­μ μ§μ.

νλμ SQL λͺλ Ήμ μ²λ¦¬νλ κ²½μ°λ DBκ° νΈλμ­μμ λ³΄μ₯ν΄μ€λ€κ³  λ―Ώμ μ μμ.

νμ§λ§, **μ¬λ¬ κ°μ SQLμ΄ μ¬μ©λλ μμμ νλμ νΈλμ­μμΌλ‘ μ·¨κΈν΄μΌ νλ κ²½μ°**κ° μμ.
> μ) κ³μ’μ΄μ²΄, μ¬μ©μμ λν λ λ²¨ μμ  μμ λ±

#### μλ₯Ό λ€μ΄, λ κ°μ SQLμ΄ νμνλ°, μ²« λ²μ§Έ SQL μ±κ³΅μ  μ€ννκ³  λ λ²μ§Έ SQLμ΄ μ±κ³΅νμ§ μ μ μ₯μ κ° μκ²¨ μμμ΄ μ€λ¨λλ κ²½μ° 

- μ΄λ λκ°μ§ μμμ΄ νλμ νΈλμ­μμ΄ λλ €λ©΄, μμμ μ²λ¦¬ν μ²« λ²μ§Έ SQL μμλ μ·¨μν΄μΌ νλ€. 

	- μ΄λ° μ·¨μ μμμ '**νΈλμ­μ λ‘€λ°±(transaction rollback)**'μ΄λΌκ³  ν¨.

- λ°λλ‘, μ¬λ¬ κ°μ SQLμ νλμ νΈλμ­μμΌλ‘ μ²λ¦¬νλ κ²½μ°, 

	- λͺ¨λ  SQL μν μμμ΄ λ€ μ±κ³΅μ μΌλ‘ λ§λ¬΄λ¦¬λλ€κ³  DBμ μλ €μ€μ μμμ νμ μμΌμΌ ν¨. = **νΈλμ­μ μ»€λ°(transaction commit)**

<br>

### β JDBC νΈλμ­μμ νΈλμ­μ κ²½κ³μ€μ 
λͺ¨λ  νΈλμ­μμ μμνλ μ§μ κ³Ό λλλ μ§μ μ΄ μμ.

μμνλ λ°©λ²μ ν κ°μ§, λλλ λ°©λ²μ λ κ°μ§(λ‘€λ°± λλ μ»€λ°)
- λ‘€λ°± : λͺ¨λ  μμμ λ¬΄ν¨ν
- μ»€λ° : λͺ¨λ  μμμ λ€ νμ 

#### β νΈλμ­μμ κ²½κ³λ?
> μ νλ¦¬μΌμ΄μ λ΄μμ νΈλμ­μμ΄ μμλκ³  λλλ μμΉ

λ³΅μ‘ν λ‘μ§ νλ¦ μ¬μ΄μμ μ ννκ² νΈλμ­μ κ²½κ³ μ€μ νλ μΌμ λ§€μ° μ€μν μμ!

#### JDBCλ₯Ό μ΄μ©ν΄ νΈλμ­μ μ μ©νλ κ°μ₯ κ°λ¨ν μμ 
- μμ€ μ½λ p.354

- JDBCμ νΈλμ­μμ νλμ Connectionμ κ°μ Έμ μ¬μ©νλ€κ° λ«λ μ¬μ΄μμ μΌμ΄λ¨
- νΈλμ­μμ μμκ³Ό μ’λ£λ Connection μ€λΈμ νΈλ₯Ό ν΅ν΄ μ΄λ€μ§κΈ° λλ¬Έ
- JDBCμμ νΈλμ­μμ μμνλ €λ©΄ μλμ»€λ° μ΅μμ falseλ‘ λ§λ€μ΄μ£Όλ©΄ λ¨
- νΈλμ­μμ΄ ν λ² μμλλ©΄ `commit()` λλ `rollback()` λ©μλκ° νΈμΆλ  λκΉμ§μ μμμ΄ νλμ νΈλμ­μμΌλ‘ λ¬Άμ.
- `commit()` λλ `rollback()`μ΄ νΈμΆλλ©΄ κ·Έμ λ°λΌ μμ κ²°κ³Όκ° DBμ λ°μλκ±°λ μ·¨μλκ³  νΈλμ­μμ΄ μ’λ£λ¨.
- **μΌλ°μ μΌλ‘ μμ μ€ μμΈ λ°μνλ©΄ νΈλμ­μμ rollbackν¨**
	- μ΄μ ? μμΈ λ°μνλ€λ κ±΄, νΈλμ­μμ κ΅¬μ±νλ λ°μ΄ν° μ‘μΈμ€ μμμ λ§λ¬΄λ¦¬ν  μ μλ μν©μ΄κ±°λ DBμ κ²°κ³Όλ₯Ό λ°μνλ©΄ μ λλ μ΄μ κ° μκ²ΌκΈ° λλ¬Έ

#### β κ²°λ‘ 
- `setAutoCommit(false)`λ‘ νΈλμ­μμ μμμ μ μΈνκ³  `commit()` λλ `rollback()`μΌλ‘ νΈλμ­μμ μ’λ£νλ μμ = **νΈλμ­μμ κ²½κ³μ€μ (transaction demarcation)**

- **νΈλμ­μμ κ²½κ³λ νλμ Connectionμ΄ λ§λ€μ΄μ§κ³  λ«νλ λ²μ μμ μ‘΄μ¬**
- μ΄λ κ² νλμ DB μ»€λ₯μ μμμ λ§λ€μ΄μ§λ νΈλμ­μ = **λ‘μ»¬ νΈλμ­μ(local transaction)**

<br>

### π€ μ μ§κΈκΉμ§ μ½λμλ νΈλμ­μ κ²½κ³μ€μ  μ½λκ° μ‘΄μ¬νμ§ μμμκΉ?
- JdbcTemplateμ νλμ ννλ¦Ώ λ©μλ μμμ DataSourceμ `getConnection()` λ©μλ νΈμΆν΄ Connection μ€λΈμ νΈ κ°μ Έμ€κ³ , μμ λ§μΉλ©΄ Connectionμ νμ€νκ² λ«μμ£Όκ³  ννλ¦Ώ λ©μλλ₯Ό λΉ μ Έλμ΄.
=> ννλ¦Ώ λ©μλ νΈμΆ ν λ²μ ν κ°μ DB μ»€λ₯μμ΄ λ§λ€μ΄μ§κ³  λ«νλ μΌκΉμ§ μΌμ΄λ¨.

- **μΌλ°μ μΌλ‘ νΈλμ­μμ μ»€λ₯μλ³΄λ€λ μ‘΄μ¬ λ²μκ° μ§§μ.**
- λ°λΌμ, **ννλ¦Ώ λ©μλκ° νΈμΆλ  λλ§λ€ νΈλμ­μμ΄ μλ‘ λ§λ€μ΄μ§κ³  λ©μλλ₯Ό λΉ μ Έλμ€κΈ° μ μ μ’λ£**λ¨.
- κ²°κ΅­, JdbcTemplateμ λ©μλλ₯Ό μ¬μ©νλ **UserDaoλ κ° λ©μλλ§λ€ νλμ©μ λλ¦½μ μΈ νΈλμ­μμΌλ‘ μ€ν**λ  μλ°μ μμ.

<br>

### π§ κ·Έλ λ€λ©΄, `upgradeLevels()`μ κ°μ΄ μ¬λ¬ λ² DBμ μλ°μ΄νΈλ₯Ό ν΄μΌ νλ μμμ νλμ νΈλμ­μμΌλ‘ λ§λ€λ €λ©΄ μ΄λ»κ² ν΄μΌ ν κΉ?
- μ΄λ€ μΌλ ¨μ μμμ΄ νλμ νΈλμ­μμΌλ‘ λ¬Άμ΄λ €λ©΄ κ·Έ μμμ΄ μ§νλλ λμ DB μ»€λ₯μλ νλλ§ μ¬μ©λΌμΌ ν¨.

### νΈλμ­μμ κ²½κ³μ€μ  μμμ UserServiceμͺ½μΌλ‘ κ°μ Έμ€μ.
> UserDaoκ° κ°μ§ SQLμ΄λ JDBC APIλ₯Ό μ΄μ©ν λ°μ΄ν° μ‘μΈμ€ μ½λλ μ΅λν κ·Έλλ‘ λ¨κ²¨λ μ±λ‘, UserServiceμλ νΈλμ­μ μμκ³Ό μ’λ£λ₯Ό λ΄λΉνλ μ΅μνμ μ½λλ§ κ°μ Έμ€κ² λ§λ€μ.

#### `upgradeLevels()` μ νΈλμ­μ κ²½κ³μ€μ  κ΅¬μ‘°
```java
public void upgradeLevels() throws Exception {
	(1) DB Connection μμ±
	(2) νΈλμ­μ μμ
	try{
		(3) DAO λ©μλ νΈμΆ
		(4) νΈλμ­μ μ»€λ°
	} 
	catch(Exception e){
		(5) νΈλμ­μ λ‘€λ°±
		throw e;
	}
	finally{
		(6) DB Connection μ’λ£
	}
}
```
> π½ μ¬κΈ°μ μμ±λ Connection μ€λΈμ νΈλ₯Ό κ°κ³  λ°μ΄ν° μ‘μΈμ€ μμ μ§ννλ μ½λλ UserDaoμ `update()` λ©μλ μμ μμ΄μΌ ν¨.
> 
>- κ·ΈλμΌλ§ κ°μ νΈλμ­μ μμμ λμνκΈ° λλ¬Έ!
>
#### UserServiceμμ λ§λ  Connection μ€λΈμ νΈλ₯Ό UserDaoμμ μ¬μ©νκΈ° μν΄ ν  μΌ(μμ€μ½λ p.358 ~ p.359)
1. DAO λ©μλλ₯Ό νΈμΆν  λλ§λ€ Connection μ€λΈμ νΈλ₯Ό νλΌλ―Έν°λ‘ μ λ¬νκΈ°

2. UserServiceμ λ©μλ μ¬μ΄μλ κ°μ Connection μ€λΈμ νΈλ₯Ό μ¬μ©νλλ‘ νλΌλ―Έν°λ‘ μ λ¬νκΈ°

<br>

### π₯ UserServiceμ UserDaoλ₯Ό μ΄λ° μμΌλ‘ μμ νλ©΄ νΈλμ­μ λ¬Έμ λ ν΄κ²°κ°λ₯,  κ·Έλ¬λ μ¬λ¬ κ°μ§ λ¬Έμ μ  λ°μ
1. DB μ»€λ₯μμ λΉλ‘―ν λ¦¬μμ€μ κΉλν μ²λ¦¬λ₯Ό κ°λ₯νκ² νλ JdbcTemplate λ μ΄μ νμ© λΆκ°

2. DAOμ λ©μλμ λΉμ¦λμ€ λ‘μ§ λ΄κ³  μλ UserServiceμ λ©μλμ Connection νλΌλ―Έν°κ° μΆκ°λΌμΌ ν¨ -> λ©μλκ° μ§μ λΆν΄μ§.
3. Connection νλΌλ―Έν°κ° UserDao μΈν°νμ΄μ€ λ©μλμ μΆκ°λλ©΄ UserDaoλ λ μ΄μ λ°μ΄ν° μ‘μΈμ€ κΈ°μ μ λλ¦½μ μΌ μ x -> κΈ°κ» μΈν°νμ΄μ€ μ¬μ©ν΄ DAO λΆλ¦¬νκ³  DI μ μ©νλ μκ³  λ¬Όκ±°ν λ¨
4. νμ€νΈ μ½λμλ μν₯ λ―ΈμΉ¨ -> νμ€νΈ μ½λμμ μ§μ  Connection μ€λΈμ νΈλ₯Ό μΌμΌμ΄ λ§λ€μ΄ DAO λ©μλλ₯Ό νΈμΆνλλ‘ λͺ¨λ λ³κ²½ν΄μΌ ν¨;

<br>

## β νΈλμ­μ λκΈ°ν
#### π± μ€νλ§μ΄ μ κ³΅νλ νΈλμ­μ λκΈ°ν κΈ°λ²μ νμ©νμ.

### β νΈλμ­μ λκΈ°ν(transaction synchronization) λ?
- **UserServiceμμ νΈλμ­μμ μμνκΈ° μν΄ λ§λ  Connection μ€λΈμ νΈλ₯Ό νΉλ³ν μ μ₯μμ λ³΄κ΄ν΄λκ³ , μ΄νμ νΈμΆλλ DAOμ λ©μλμμλ μ μ₯λ Connectionμ κ°μ Έλ€κ° μ¬μ©νκ² νλ κ²**

- μ ννλ DAOκ° μ¬μ©νλ JdbcTemplateμ΄ νΈλμ­μ λκΈ°ν λ°©μμ μ΄μ©νλλ‘ νλ κ²
- νΈλμ­μμ΄ λͺ¨λ μ’λ£λλ©΄, κ·Έλλ λκΈ°νλ₯Ό λ§μΉλ©΄ λ¨.
- νΈλμ­μμ κ²½κ³μ€μ μ΄ νμν `upgradeLevels()` μμλ§ Connectionμ λ€λ£¨κ² νκ³ , μ¬κΈ°μ μμ±λ Connectionκ³Ό νΈλμ­μμ DAOμ JdbcTemplateμ΄ μ¬μ©ν  μ μλλ‘ λ³λμ μ μ₯μμ λκΈ°ννλ λ°©λ²μ μ μ©νκΈ°λ§ νλ©΄ λ¨.

<br>

### β νΈλμ­μ λκΈ°ν μ μ₯μ 
> μμ μ€λ λλ§λ€ λλ¦½μ μΌλ‘ Connection μ€λΈμ νΈλ₯Ό μ μ₯νκ³  κ΄λ¦¬νκΈ° λλ¬Έμ λ€μ€ μ¬μ©μλ₯Ό μ²λ¦¬νλ μλ²μ λ©ν°μ€λ λ νκ²½μμλ μΆ©λμ΄ λ  μΌλ €λ μμ

<br>

### β νΈλμ­μ λκΈ°ν μ μ©
λ¬Έμ λ λ©ν°μ€λ λ νκ²½μμλ μμ ν νΈλμ­μ λκΈ°ν λ°©λ²μ κ΅¬ννλ μΌμ΄ κΈ°μ μ μΌλ‘ κ°λ¨νμ§ μλ€λ μ .

λ€ννλ μ€νλ§μ JdbcTemplateκ³Ό λλΆμ΄ μ΄λ° νΈλμ­μ λκΈ°ν κΈ°λ₯μ μ§μνλ κ°λ¨ν μ νΈλ¦¬ν° λ©μλλ₯Ό μ κ³΅ μ€π€
#### Fix: νΈλμ­μ λκΈ°ν λ°©μμ μ μ©ν UserService
```java
private DataSource dataSource;

public void setDataSource(DataSource dataSource){
	this.dataSource = dataSource;
}

public void upgradeLevels() throws Exception {
	TransactionSynchronizationManager.initSynchronization(); // νΈλμ­μ λκΈ°ν κ΄λ¦¬μλ₯Ό μ΄μ©ν΄ λκΈ°ν μμμ μ΄κΈ°νν¨
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
		
		// λκΈ°ν μμ μ’λ£ λ° μ λ¦¬
		TransactionSynchronizationManager.unbindResource(this.dataSource);
		TransactionSynchronizationManager.clearSynchronization();
	}
}
```
#### Fix: λκΈ°νκ° μ μ©λ UserServiceμ λ°λΌ μμ λ νμ€νΈ 
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
#### Fix: dataSource νλ‘νΌν°λ₯Ό μΆκ°ν userService λΉ μ€μ 
```xml
<bean id="userService" class="springbook.user.service.UserService">
	<property name="userDao" ref="userDao" />
	<property name="dataSource" ref="dataSource" />
</bean>
```

<br>

## νΈλμ­μ μλΉμ€ μΆμν
### κΈ°μ κ³Ό νκ²½μ μ’μλλ νΈλμ­μ κ²½κ³μ€μ  μ½λ
#### π€·ββοΈ λ§μ½, μ΄ μ¬μ©μ κ΄λ¦¬ λͺ¨λμ κ΅¬λ§€ν΄μ μ¬μ©νκΈ°λ‘ ν Gμ¬μμ μλ‘μ΄ μκ΅¬λ₯Ό νμ κ²½μ°?
- Gμ¬λ μ¬λ¬ κ°μ DB μ¬μ© μ€

- **νλμ νΈλμ­μ μμμ μ¬λ¬ κ°μ DBμ λ°μ΄ν°λ₯Ό λ£λ μμ**μ κ° DBμ λλ¦½μ μΌλ‘ λ§λ€μ΄μ§λ Connectionμ ν΅ν΄μκ° μλλΌ, **λ³λμ νΈλμ­μ κ΄λ¦¬μλ₯Ό ν΅ν΄ νΈλμ­μμ κ΄λ¦¬**νλ **κΈλ‘λ² νΈλμ­μ(global transaction)** λ°©μμ μ¬μ©ν΄μΌ ν¨.

###  JTA(Java Transaction API)
>JDBC μΈμ μ΄λ° κΈλ‘λ² νΈλμ­μμ μ§μνλ νΈλμ­μ λ§€λμ λ₯Ό μ§μνκΈ° μν API
>
>μλ°μμ μ κ³΅ μ€
>
> JTAλ₯Ό μ΄μ©ν΄ νΈλμ­μ λ§€λμ λ₯Ό νμ©νλ©΄ μ¬λ¬ κ°μ DBλ λ©μμ§ μλ²μ λν μμμ νλμ νΈλμ­μμΌλ‘ ν΅ν©νλ λΆμ° νΈλμ­μ λλ κΈλ‘λ² νΈλμ­μμ΄ κ°λ₯ν΄μ§.
>
>β **νλ μ΄μμ DBκ° μ°Έμ¬νλ νΈλμ­μμ λ§λ€λ €λ©΄ JTAλ₯Ό μ¬μ©ν΄μΌ νλ€.**

#### [Gμ¬ μμ²­ κ²°λ‘ ] 
- μλ²κ° μ κ³΅νλ νΈλμ­μ λ§€λμ μ νΈλμ­μ μλΉμ€λ₯Ό μ¬μ©ν νλ JDBC APIκ° μλ **JTA**λ₯Ό μ¬μ©ν΄ νΈλμ­μμ κ΄λ¦¬νκ² ν΄λ¬λΌ.

#### π€¦ββοΈ μ΄ μμ€μ νμ΄λ²λ€μ΄νΈλ₯Ό μ΄μ©ν΄ UserDaoλ₯Ό μ§μ  κ΅¬ννλ€κ³  Yμ¬μμ μ°λ½μ΄ μλ€λ©΄?
- π₯ λ¬Έμ  : UserServiceμ μ½λκ° νΉμ  νΈλμ­μ λ°©λ²(JTA, νμ΄λ²λ€μ΄νΈ λ±)μ μμ‘΄μ μ΄μ§ μκ³ , λλ¦½μ μΌ μ μκ² λ§λ€μ΄μΌ ν¨
- β ν΄κ²° : μ€νλ§μ΄ μ κ³΅νλ **νΈλμ­μ μΆμν**λ₯Ό μ΄μ©νμ.

<br>

### β μ€νλ§μ νΈλμ­μ μΆμν
- μ€νλ§μ΄ μ κ³΅νλ νΈλμ­μ κ²½κ³μ€μ μ μν μΆμ μΈν°νμ΄μ€ : `PlatformTransactionManager` 

- JDBCμ λ‘μ»¬ νΈλμ­μμ μ΄μ©νλ€λ©΄ `PlatformTransactionManager`λ₯Ό κ΅¬νν `DataSourceTransactionManager`λ₯Ό μ¬μ©νλ©΄ λ¨
- JTAλ₯Ό μ΄μ©ν κΈλ‘λ² νΈλμ­μμΌλ‘ λ³κ²½νλ €λ©΄ `PlatformTransactionManager`λ₯Ό κ΅¬ν ν΄λμ€λ₯Ό `JTATransactionManager`λ₯Ό μ¬μ©νλ©΄ λ¨
- νμ΄λ²λ€μ΄νΈλ `PlatformTransactionManager`λ₯Ό κ΅¬ν ν΄λμ€λ₯Ό `HibernateTransactionManager`λ₯Ό μ¬μ©νλ©΄ λ¨
- JPAλ₯Ό μ μ©νλ€λ©΄ `PlatformTransactionManager`λ₯Ό κ΅¬ν ν΄λμ€λ₯Ό `JPATransactionManager`λ₯Ό μ¬μ©νλ©΄ λ¨

#### Fix: νΈλμ­μ λ§€λμ λ₯Ό λΉμΌλ‘ λΆλ¦¬μν¨ UserService
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
#### Fix: μ€νλ§ μ€μ νμΌμ UserServiceμ DIλ  transactionManager λΉ λ±λ‘ (μμ€ μ½λ p.374)
#### Fix: νΈλμ­μ λ§€λμ λ₯Ό μλ DIνλλ‘ νμ€νΈ μμ  (μμ€ μ½λ p.374)
#### [κ²°λ‘ ]
- μλ°μμ μ¬μ©λλ νΈλμ­μ APIμ μ’λ₯μ λ°©λ²μ λ€μνκ³ , νκ²½κ³Ό μλ²μ λ°λΌμ νΈλμ­μ λ°©λ²μ΄ λ³κ²½λλ©΄ κ²½κ³μ€μ  μ½λλ ν¨κ» λ³κ²½λΌμΌ ν¨.

<br>

## 5.3 μλΉμ€ μΆμνμ λ¨μΌ μ±μ μμΉ
**κΈ°μ κ³Ό μλΉμ€μ λν μΆμν κΈ°λ²** = **κ°μ κ³μΈ΅**μμ **μνμ  λΆλ¦¬**

**νΈλμ­μμ μΆμν κΈ°λ²** = **λ€λ₯Έ κ³μΈ΅** μ½λ **μμ§μ  λΆλ¦¬**

μ νλ¦¬μΌμ΄μ λ‘μ§μ μ’λ₯μ λ°λ₯Έ μνμ μΈ κ΅¬λΆμ΄λ , λ‘μ§κ³Ό κΈ°μ μ΄λΌλ μμ§μ μΈ κ΅¬λΆμ΄λ  λͺ¨λ κ²°ν©λκ° λ?μΌλ©°, μλ‘ μν₯μ μ£Όμ§ μκ³  μμ λ‘­κ² νμ₯λ  μ μλ κ΅¬μ‘°λ₯Ό λ§λ€ μ μλ λ°λ '**μ€νλ§μ DI**'κ° μ€μν μ­ν  νκ³  μμ.
>#### DIμ κ°μΉ
>
>- κ΄μ¬, μ±μ, μ±κ²©μ΄ λ€λ₯Έ μ½λλ₯Ό κΉλνκ² λΆλ¦¬

β μλΉμ€ μΆμνλ λ‘μ°λ λ²¨μ νΈλμ­μ κΈ°μ κ³Ό APIμ λ³νμ μκ΄μμ΄ μΌκ΄λ APIλ₯Ό κ°μ§ μΆμν κ³μΈ΅μ λμν¨.

<br>

### β λ¨μΌ μ±μ μμΉ(Single Responsibility Principle)
κ°μ²΄μ§ν₯ μ€κ³μ μμΉ μ€ νλ

>**νλμ λͺ¨λμ ν κ°μ§ μ±μμ κ°μ ΈμΌ νλ€.**

#### [λ¨μΌ μ±μ μμΉμ μ₯μ ]
- μ΄λ€ λ³κ²½μ΄ νμν  λ μμ  λμμ΄ λͺνν΄μ§

	- κΈ°μ μ΄ λ°λλ©΄ κΈ°μ  κ³μΈ΅κ³Όμ μ°λμ λ΄λΉνλ κΈ°μ  μΆμν κ³μΈ΅μ μ€μ λ§ λ°κΏμ£Όλ©΄ λ¨

	- λ°μ΄ν°λ₯Ό κ°μ Έμ€λ νμ΄λΈμ μ΄λ¦μ΄ λ°λλ©΄ λ°μ΄ν° μ‘μΈμ€ λ‘μ§μ λ΄κ³  μλ UserDaoλ₯Ό λ³κ²½νλ©΄ λ¨

<br>

### κ°μ²΄μ§ν₯ μ€κ³μ νλ‘κ·Έλλ°μ μμΉμ μλ‘ κΈ΄λ°νκ² κ΄λ ¨μ΄ μλ€.

λ¨μΌ μ±μ μμΉμ μ μ§ν€λ μ½λλ₯Ό λ§λ€λ €λ©΄, μΈν°νμ΄μ€ λμνκ³  μ΄λ₯Ό DIλ‘ μ°κ²°ν΄μΌ νλ©°, κ·Έ κ²°κ³Όλ‘ λ¨μΌ μ±μ μμΉλΏ μλλΌ κ°λ°© νμ μμΉλ μ μ§ν€κ³ , λͺ¨λ κ°μ κ²°ν©λκ° λ?μμ μλ‘μ λ³κ²½μ΄ μν₯μ μ£Όμ§ μκ³ , κ°μ μ΄μ λ‘ λ³κ²½μ΄ λ¨μΌ μ±μμ μ§μ€λλ μμ§λ λμ μ½λκ° λμ΄.

λν, μ΄λ° κ³Όμ μμ μ λ΅ ν¨ν΄, μ΄λν° ν¨ν΄, λΈλ¦¬μ§ ν¨ν΄, λ―Έλμμ΄ν° ν¨ν΄ λ± λ§μ λμμΈ ν¨ν΄μ΄ μμ°μ€λ½κ² μ μ©λ¨.

κ°μ²΄μ§ν₯ μ€κ³ μμΉ μ μ§μΌμ λ§λ  μ½λλ νμ€νΈνκΈ°λ νΈν¨.

μ€νλ§μ΄ μ§μνλ DIμ μ±κΈν€ λ μ§μ€νΈλ¦¬ λλΆμ λμ± νΈλ¦¬νκ² μλνλ νμ€νΈ λ§λ€ μ μμ.

> μ€νλ§μ μμ‘΄κ΄κ³ μ£Όμ κΈ°μ μΈ DIλ λͺ¨λ  μ€νλ§ κΈ°μ μ κΈ°λ°μ΄ λλ ν΅μ¬ μμ§μ΄μ μλ¦¬μ΄λ©°, μ€νλ§μ΄ μ§μ§νκ³  μ§μνλ μ’μ μ€κ³μ μ½λλ₯Ό λ§λλ λͺ¨λ  κ³Όμ μμ μ¬μ©λλ κ°μ₯ μ€μν λκ΅¬.

<br>

## 5.4 λ©μΌ μλΉμ€ μΆμν
#### κ³ κ°μΌλ‘λΆν° μ¬μ©μ λ λ²¨ κ΄λ¦¬μ κ΄ν μλ‘μ΄ μμ²­μ¬ν­ :
> λ λ²¨μ΄ μκ·Έλ μ΄λλλ μ¬μ©μμκ²λ μλ΄ λ©μΌμ λ°μ‘ν΄λ¬λΌλ κ²

#### μλ΄ λ©μΌ λ°μ‘νκΈ° μν΄ ν΄μΌ ν  μΌ λ κ°μ§

1. μ¬μ©μμ μ΄λ©μΌ μ λ³΄ κ΄λ¦¬ν΄μΌ ν¨ -> Userμ email νλ μΆκ°

2. μκ·Έλ μ΄λ μμμ λ΄μ UserServiceμ `upgradeLevel()` λ©μλμ λ©μΌ λ°μ‘ κΈ°λ₯ μΆκ°

<br>

## JavaMailμ μ΄μ©ν λ©μΌ λ°μ‘ κΈ°λ₯
### Step 1. μ¬μ©μ μ λ³΄μ μ΄λ©μΌ μΆκ°
1. DBμ User νμ΄λΈμ email νλ μΆκ°

2. User ν΄λμ€μ email νλ‘νΌν° μΆκ°
3. UserDaoμ userMapperμ insert(), update()μ email νλ μ²λ¦¬ μ½λ μΆκ°
4. User μμ±μκ° email λ°μ μ μκ² νκ³ , νμ€νΈ λ°μ΄ν° λ§κ² μ€λΉν λ€ λ±λ‘, μμ , μ‘°νμμ email κ°μ΄ μ μ²λ¦¬λλμ§ νμΈνλλ‘ UserDaoTest μμ 

<br>

### Step 2. JavaMail λ©μΌ λ°μ‘
μλ°μμ λ©μΌμ λ°μν  λλ νμ€ κΈ°μ μΈ JavaMail μ¬μ©νλ©΄ λ¨
> javax.mail ν¨ν€μ§μμ μ κ³΅νλ μλ°μ μ΄λ©μΌ ν΄λμ€ μ¬μ©

1οΈβ£ **μκ·Έλ μ΄λ μμμ λ΄μ λ©μλμΈ upgradeLevel()μμ λ©μΌ λ°μ‘ λ©μλ νΈμΆνκΈ°**
>#### Fix: λ λ²¨ μκ·Έλ μ΄λ μμ λ©μλ μμ 
```java
protected void upgradeLevel(User user){
	user.upgradeLevel();
	userDao.update(user);
	sendUpgradeEMail(user);
}
```

2οΈβ£ **JavaMail API μ¬μ©νλ λ©μλ μΆκ°νκΈ°**
>#### Feat: JavaMailμ μ΄μ©ν λ©μΌ λ°μ‘ λ©μλ μΆκ°
```java
private void sendUpgradeEMail(User user){
	Properties props = new Properties();
	props.put("mail.smtp.host", "mail.ksug.org");
	Session s = Session.getInstance(props, null);
	
	MimeMessage message = new MimeMessage(s);
	try{
		message.setFrom(new InternetAddress("useradmin@ksug.org"));
		message.addRecipient(Message.RecipientType.TO, new InternetAddress(user.getEmail()));
		message.setSubject("Upgrade μλ΄");
		message.setText("μ¬μ©μλμ λ±κΈμ΄ " + user.getLevel().name() + "λ‘ μκ·Έλ μ΄λλμμ΅λλ€.");
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
### π₯ κ·Έλ°λ°, λ©μΌ μλ²κ° μ€λΉλμ΄ μμ§ μλ€λ©΄ μ΄λ»κ² ν κΉ?
#### ν΄κ²°λ°©λ² 1) νμ€νΈ λ©μΌ μλ² μ¬μ©
λ©μΌ νμ€νΈλ₯Ό νλ€κ³  λ§€λ² λ§€μΌ μμ  μ¬λΆκΉμ§ μΌμΌμ΄ νμΈν  νμλ μκ³ , νμ€νΈ κ°λ₯ν λ©μΌ μλ²κΉμ§λ§ μ μ μ‘λλμ§ νμΈνλ©΄ λ¨.

κ·Έλ¦¬κ³  νμ€νΈμ© λ©μΌ μλ²λ λ©μΌ μ μ‘ μμ²­μ λ°μ§λ§ μ€μ λ‘ λ©μΌμ΄ λ°μ‘λμ§ μλλ‘ μ€μ 

#### ν΄κ²°λ°©λ² 2) νμ€νΈμ© JavaMail μ¬μ© (μ΄ λ°©λ²μ μ ννμ β)
JavaMailμ μλ°μ νμ€ κΈ°μ , μ΄λ―Έ κ²μ¦λ μμ μ  λͺ¨λ => κ΅³μ΄ νμ€νΈν  λλ§λ€ JavaMailμ μ§μ  κ΅¬λμν¬ νμ μμ

λν, JavaMailμ΄ λμνλ©΄ μΈλΆ λ©μΌ μλ²μ λ€νΈμν¬λ‘ μ°λνκ³  μ μ‘νλ λΆνκ° ν° μμμ΄ μΌμ΄λκΈ° λλ¬Έμ μ΄λ₯Ό μλ΅ν  μ μλ€λ©΄ λ μ’κΈ° λλ¬Έ

#### π₯ κ·Έλ°λ°, JavaMailμ μ΄μ©ν νμ€νΈμ ν κ°μ§ λ¬Έμ μ  : JavaMailμ APIλ μ΄ λ°©λ²μ μ μ© λΆκ° 
> JavaMail ν΅μ¬ APIμλ DataSourceμ²λΌ μΈν°νμ΄μ€λ‘ λ§λ€μ΄μ Έμ κ΅¬νμ λ°κΏ μ μλ κ² μμ
> 
> λ μμΈν μ¬ν­μ p.384 μ°Έκ³ 

#### π‘ ν΄κ²°λ°©λ² : μλΉμ€ μΆμνλ₯Ό μ μ©νμ.
> μ€νλ§μ JavaMailμ μ¬μ©ν΄ λ§λ  μ½λλ μμ½κ² νμ€νΈνκΈ° νλ€λ€λ λ¬Έμ λ₯Ό ν΄κ²°νκΈ° μν΄ JavaMailμ λν μΆμν κΈ°λ₯μ μ κ³΅νκ³  μμ

<br>

### Step 3. λ©μΌ λ°μ‘ κΈ°λ₯ μΆμν (μμ€ μ½λ p.385 ~ p.387 )
μΆκ°ν΄μΌ νλ λΌμ΄λΈλ¬λ¦¬
```java
com.springsource.javax.activation-1.1.0.jar
com.springsource.javax.mail-1.4.0.jar
org.springframework.context.support-3.0.7.RELEASE.jar
```
#### Feat: μ€νλ§μ MailSenderλ₯Ό μ΄μ©ν λ©μΌ λ°μ‘ λ©μλ μΆκ°
1. JavaMailμ μ¬μ©ν΄ λ©μΌ λ°μ‘ κΈ°λ₯μ μ κ³΅νλ `JavaMailSenderImpl` ν΄λμ€μ μ€λΈμ νΈ μμ±

2. MailMessage μΈν°νμ΄μ€μ κ΅¬ν ν΄λμ€ μ€λΈμ νΈλ₯Ό λ§λ€μ΄ λ©μΌ λ΄μ© μμ±
#### Fix: λ©μΌ μ μ‘ κΈ°λ₯μ κ°μ§ μ€λΈμ νΈλ₯Ό DI λ°λλ‘ UserService μμ 
- MailSender μΈν°νμ΄μ€ νμμ λ³μ λ§λ€κ³ , μμ μ λ©μλ μΆκ°ν΄ DI κ°λ₯νλλ‘ λ§λ¦
#### Feat: μ€νλ§ μ€μ νμΌ μμ λ©μΌ λ°μ‘ μ€λΈμ νΈμ λΉ λ±λ‘
- JavaMailSenderImpl ν΄λμ€λ‘ λΉμ λ§λ€κ³  UserServiceμ DI ν΄μ€

<br>

### Step 4. νμ€νΈμ© λ©μΌ λ°μ‘ μ€λΈμ νΈ (μμ€ μ½λ p.388)
1. μ€νλ§μ΄ μ κ³΅ν λ©μΌ μ μ‘ κΈ°λ₯μ λν μΈν°νμ΄μ€(MailSender)λ₯Ό κ΅¬νν΄ νμ€νΈμ© λ©μΌ μ μ‘ ν΄λμ€(DummyMailSender) λ§λ€κΈ°.

2. νμ€νΈ μ€μ νμΌμ mailSender λΉ ν΄λμ€λ₯Ό JavaMailμ μ¬μ©νλ JavaMailSenderImpl λμ  DummyMailSenderλ‘ λ³κ²½
3. νμ€νΈμ© UserSErviceλ₯Ό μν λ©μΌ μ μ‘ μ€λΈμ νΈμ μλ DI

<br>

### π₯ νμ¬κΉμ§ μ½λμ λΆμ‘±ν μ  : λ©μΌ λ°μ‘ μμμ νΈλμ­μ κ°λ λΉ μ Έ μμ
> λ λ²¨ μκ·Έλ μ΄λ μμ μ€κ°μ μμΈ λ°μν΄ DBμ λ°μλλ λ λ²¨ μκ·Έλ μ΄λ λͺ¨λ λ‘€λ°±λλ€κ³  νμ.
>
>νμ§λ§ λ©μΌμ μ¬μ©μλ³λ‘ μκ·Έλ μ΄λ μ²λ¦¬ν  λ μ΄λ―Έ λ°μ‘ν΄λ²λ¦Ό. κ·Έκ²μ μ΄λ»κ² μ·¨μν  κ²μΈκ°?

<br>

### π‘ λ°λΌμ, λ©μΌ λ°μ‘ κΈ°λ₯μλ νΈλμ­μ κΈ°λ₯μ μ μ©νμ.
#### λ°©λ² 1 : λ©μΌμ μκ·Έλ μ΄λν  μ¬μ©μλ₯Ό λ°κ²¬ν  λλ§λ€ λ°μ‘νμ§ μκ³  λ°μ‘ λμμ λ³λμ λͺ©λ‘μ μ μ₯ν΄λ 
- λ¨μ  : λ©μΌ μ μ₯μ© λ¦¬μ€νΈ λ±μ νλΌλ―Έν°λ‘ κ³μ κ°κ³  λ€λμΌ ν¨

#### λ°©λ² 2 : MailSenderλ₯Ό νμ₯ν΄ λ©μΌ μ μ‘μ νΈλμ­μ κ°λ μ μ©
- μ₯μ  : μλ‘ λ€λ₯Έ μ’λ₯μ μμμ λΆλ¦¬ν΄ μ²λ¦¬ κ°λ₯

<br>

#### β μ΄ μμ λ₯Ό μ€λͺνλ©΄μ νμκ° λ§νκ³ μ νλ μ΄μΌκΈ°
- μλΉμ€ μΆμνλ νμ€νΈνκΈ° μ΄λ €μ΄ JavaMail κ°μ κΈ°μ μλ μ μ© κ°λ₯.

- νμ€νΈλ₯Ό νΈλ¦¬νκ² μμ±νλλ‘ λμμ£Όλ κ²λ§μΌλ‘λ μλΉμ€ μΆμνλ κ°μΉκ° μμ.

<br>

## β νμ€νΈ λμ­(test double)
>**νμ€νΈ λμμ΄ μ¬μ©νλ μμ‘΄ μ€λΈμ νΈλ₯Ό λμ²΄ν  μ μλλ‘ λ§λ  μ€λΈμ νΈ**
>
>νμ€νΈμ©μΌλ‘ μ¬μ©λλ νΉλ³ν μ€λΈμ νΈ
>
>- νμ€νΈ λμ μ€λΈμ νΈκ° μννκ² λμν  μ μλλ‘ λμ°λ©΄μ νμ€νΈλ₯Ό μν΄ κ°μ μ μΈ μ λ³΄λ₯Ό μ κ³΅ν΄μ£ΌκΈ°λ ν¨.

#### νμ€νΈ λμ­μ μ¬μ©νλ μ΄μ  :
- νμ€νΈ λμμ΄ λλ μ€λΈμ νΈκ° λ λ€λ₯Έ μ€λΈμ νΈμ μμ‘΄νλ κ²½μ°λ ννκΈ° λλ¬Έ -> μ¬λ¬ κ°μ§ λ¬Έμ  λ°μ

<br>

### β λνμ  νμ€νΈ λμ­ 1. νμ€νΈ μ€ν(test stub)
> **νμ€νΈ λμ μ€λΈμ νΈμ μμ‘΄κ°μ²΄λ‘μ μ‘΄μ¬νλ©΄μ νμ€νΈ λμμ μ½λκ° μ μμ μΌλ‘ μνν  μ μλλ‘ λλ κ²**
>- μΌλ°μ μΌλ‘ λ©μλλ₯Ό ν΅ν΄ μ λ¬νλ νλΌλ―Έν°μ λ¬λ¦¬, νμ€νΈ μ½λ λ΄λΆμμ κ°μ μ μΌλ‘ μ¬μ©λ¨.
>
>- λ°λΌμ, DI λ±μ ν΅ν΄ λ―Έλ¦¬ μμ‘΄ μ€λΈμ νΈλ₯Ό νμ€νΈ μ€νμΌλ‘ λ³κ²½ν΄μΌ ν¨.
>- DummyMailSenderλ κ°μ₯ λ¨μνκ³  μ¬νν νμ€νΈ μ€νμ μ
>- κ°μ μ μΈ μλ ₯ κ°μ μ§μ  κ°λ₯, κ°μ μ μΈ μΆλ ₯ κ°μ λ°κ² ν  μλ μμ

<br>

### β λνμ  νμ€νΈ λμ­ 2. λͺ© μ€λΈμ νΈ(mock object)
> **νμ€νΈ λμμ κ°μ μ μΈ μΆλ ₯ κ²°κ³Όλ₯Ό κ²μ¦νκ³ , νμ€νΈ λμ μ€λΈμ νΈμ μμ‘΄ μ€λΈμ νΈ μ¬μ΄μμ μΌμ΄λ μΌμ κ²μ¦ν  μ μλλ‘ μ€κ³λ κ².**
> - λͺ© μ€λΈμ νΈλ μ€νμ²λΌ νμ€νΈ μ€λΈμ νΈκ° μ μμ μΌλ‘ μ€νλλλ‘ λμμ£Όλ©΄μ, νμ€νΈ μ€λΈμ νΈμ μμ μ μ¬μ΄μμ μΌμ΄λλ μ»€λ?€λμΌμ΄μ λ΄μ©μ μ μ₯ν΄λλ€κ° νμ€νΈ κ²°κ³Όλ₯Ό κ²μ¦νλ λ° νμ©ν  μ μκ² ν΄μ€.

<br>

### π§ μ€ν VS λͺ© μ€λΈμ νΈ
>λͺ© μ€λΈμ νΈλ κ°μ  νμ€νΈ κ΅¬κ°μ μΆλ ₯ νμΈμ΄ μλ€.

<br>

### β λͺ© μ€λΈμ νΈλ₯Ό μ΄μ©ν νμ€νΈ
DummyMailSender λμ  μλ‘μ΄ MailSenderλ₯Ό λμ²΄ν  ν΄λμ€ MockMailSender λ§λ€κΈ°

#### Feat: λͺ© μ€λΈμ νΈλ‘ λ§λ  λ©μΌ μ μ‘ νμΈμ© ν΄λμ€
```java
static class MockMailSender implements MailSender{
	private List<String> requests = new ArrayList<String>();

	public List<String> getRequests(){
		return requests;
	}
	
	public void send(SimpleMailMessage mailMessage) throws MailException {
		requests.add(mailMessage.getTo()[0]); // μ μ‘ μμ²­ λ°μ μ΄λ©μΌ μ£Όμ μ μ₯. κ°λ¨νκ² μ²« λ²μ§Έ μμ μ λ©μΌ μ£Όμλ§ μ μ₯ν¨
	}
		
	public void send(SimpleMailMessage[] mailMessage) throws MailException {
	}
}
```
> λ¬Όλ‘  λ©μΌ λ°μ‘ κΈ°λ₯μ μμ.
> 
> UserServiceμ μ½λκ° μ μμ μΌλ‘ μνλλλ‘ λλ μ­ν μ΄ μ°μ μ.
> 
> λμ  νμ€νΈ λμμ΄ λκ²¨μ£Όλ μΆλ ₯ κ°μ λ³΄κ΄ν΄λλ κΈ°λ₯ μΆκ°

#### Fix: λͺ© μ€λΈμ νΈλ₯Ό ν΅ν΄ λ©μΌ λ°μ‘ μ¬λΆ κ²μ¦νλλ‘ μμ ν νμ€νΈ
```java
@Test
@DirtiesContext // μ»¨νμ€νΈμ DI μ€μ μ λ³κ²½νλ νμ€νΈλΌλ κ²μ μλ €μ€
public void upgradeLevels() throws Exception {
	userDao.deleteAll();
	for(User user : users) userDao.add(user);

	// λ©μΌ λ°μ‘ κ²°κ³Όλ₯Ό νμ€νΈν  μ μλλ‘ λͺ© μ€λΈμ νΈλ₯Ό λ§λ€μ΄ userServiceμ μμ‘΄ μ€λΈμ νΈλ‘ μ£Όμν΄μ€
	MockMailSender mockMailSender = new MockMailSender();
	userService.setMailSender(mockMailSender);
	
	// μκ·Έλ μ΄λ νμ€νΈ, λ©μΌ λ°μ‘μ΄ μΌμ΄λλ©΄ MockMailSender μ€λΈμ νΈμ λ¦¬μ€νΈμ κ·Έ κ²°κ³Όκ° μ μ₯λ¨
	userService.upgradeLevels();

	checkLevelUpgraded(users.get(0), false);
	checkLevelUpgraded(users.get(1), true);
	checkLevelUpgraded(users.get(2), false);
	checkLevelUpgraded(users.get(3), true);
	checkLevelUpgraded(users.get(4), false);

	// λͺ© μ€λΈμ νΈμ μ μ₯λ λ©μΌ μμ μ λͺ©λ‘μ κ°μ Έμ μκ·Έλ μ΄λ λμκ³Ό μΌμΉνλμ§ νμΈ
	List<String> request = mockMailSender.getRequests();
	assertThat(request.size(), is(2));
	assertThat(request.get(0), is(users.get(1).getEmail()));
	assertThat(request.get(1), is(users.get(3).getEmail()));
}
```

νμ€νΈκ° μνλ  μ μλλ‘ 

- **μμ‘΄ μ€λΈμ νΈμ κ°μ μ μΌλ‘ μλ ₯ κ°μ μ κ³΅**ν΄μ£Όλ **μ€ν μ€λΈμ νΈ**

- **κ°μ μ μΈ μΆλ ₯ κ°κΉμ§ νμΈμ΄ κ°λ₯ν λͺ© μ€λΈμ νΈ**

μ΄ λ κ°μ§λ **νμ€νΈ λμ­μ κ°μ₯ λνμ μΈ λ°©λ²**μ΄λ©° ν¨κ³Όμ μΈ νμ€νΈ μ½λλ₯Ό μμ±νλ λ° λΉ μ§ μ μλ μ€μν λκ΅¬λ€.