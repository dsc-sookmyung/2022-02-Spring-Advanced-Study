# 3장 템플릿

# 템플릿

## 초난감 DAO를 다시 보자.

- 예외상황에 대한 처리가 빠져 있다.

<br>

### 예외처리 기능 추가

- 예외상황에서도 리소스를 제대로 반환할 수 있도록 `try` / `catch` / `finally` 를 적용하자.

```java
public void deleteAll() throws SQLException {
	Connection c = null;
	PreparedStatement ps = null;

	try {
		c = dataSource.getConnection();
		ps = c.prepareStatement("delete from users");
		ps.executeUpdate();
	} catch (SQLException e) {
		throw e;
	} finally {                        // try에서 예외가 발생했을 때, 안 했을 때 모두 실행
		if (ps != null) {
			try {
				ps.close();
			} catch (SQLException e) {}
		}
		if (c != null) {
			try {
				c.close();
			} catch (SQLException e) {}
	}
```

<br>
<br>

### JDBC 조회 기능의 예외처리

```java
public int getCount() throws SQLException {
	Connection c = null;
	PreparedStatement ps = null;
	ResultSet rs = null;

	try {
		c = dataSource.getConnection();
		ps = c.prepareStatement("select count(*) from users");

		rs.psexecuteQuery();
		rs.next();
		return rs.getInt(1);
	} catch (SQLException e) {
		throw e;
	} finally {
		if (rs != null) {
			try {
				rs.close();
			} catch (SQLException e) {}
		}
		if (ps != null) {
			try {
				ps.close();
			} catch (SQLException e) {}
		}
		if (c != null) {
			try {
				c.close();
			} catch (SQLException e) {}
	}
```

⇒ 이제 실전에 적용해도 문제가 없는 잘 설계된 DAO가 되었다. 그러나 아쉬움은 존재!

<br>
<br>

## 변하는 것과 변하지 않는 것

### 위 코드의 문제점

- try/catch/finally 블록 2중 중첩
- 예외상황 테스트도 모든 상황에 적용하기 힘듬

<br>

### 디자인 패턴을 적용해 분리하고 재사용하자

예외 처리 코드에서 변하는 부분과 변하지 않는 부분을 분리해보자.

<br>

- 변하지 않는 부분
    - Connection, PreparedStatement 초기화
    - `dataSource.getConnection()`
    - catch 문, finally문
- 변하는 부분
    - 실제 전송되는 쿼리

<br>

> 메소드 추출

```java
public void deleteAll() throws SQLException {
	Connection c = null;
	PreparedStatement ps = null;

	try {
		c = dataSource.getConnection();
		
		ps = makeStatement(c);		            // 변하는 부분을 메소드로 추출

		ps.executeUpdate();
	} catch (SQLException e) {
		throw e;
	} finally {                 
		if (ps != null) {
			try {
				ps.close();
			} catch (SQLException e) {}
		}
		if (c != null) {
			try {
				c.close();
			} catch (SQLException e) {}
	}
}

private PreparedStatement makeStatement(Connection c) throws SQLException {
	PreparedStatement ps;
	ps = c.prepareStatement("delete from users");
	return ps;
}

```

⇒ 남은 메소드가 재사용이 필요한 부분이고, 분리된 메소드는 확장될 부분이다.

⇒ 반대로 구현됨

<br>
<br>

> 템플릿 메소드 패턴
- 상속을 통해 기능을 확장하는 패턴
- 변하지 않는 부분은 슈퍼클래스에, 변하는 부분은 추상 메소드로 정의한다.
- 서브클래스는 추상 메소드를 오버라이드해 새롭게 정의한다.

<br>

1. `makeStatement()` 메소드를 추상 메소드로 변경한다.
    
    ```java
    abstract protected PreparedStatement makeStatement(Connection c) throws SQLException;
    ```
    
2. 서브클래스를 구현한다.
    
    ```java
    public class UserDaoDeleteAll extends UserDao {
    	
    	protected PrepareStatement makeStatement(Connection c) throws SQLException {
    		PreparedStatement ps;
    		ps = c.prepareStatement("delete from users");
    		return ps;
    	}
    }
    ```
    
    ⇒ DAO 로직마다 상속으로 새 클래스를 만들어야 한다는 제한점이 있다.
    
<br>

> 전략 패턴
- `deleteAll()` 의 맥락(컨텍스트)
    - DB 커넥션 가져오기
    - PreparedStatement를 만들어줄 외부 기능 호출하기
    - 전달받은 PreparedStatement 실행하기
    - 예외 발생 시 메소드 밖으로 던지기
    - 모든 경우에 만들어진 PreparedStatement와 Connection을 적절히 닫아주기

1. StatementStrategy 인터페이스
    
    ```java
    public interface StatementStrategy {
    	PreparedStatement makePreparedStatement(Connection c) throws SQLException;
    }
    ```
    
2. 구현 클래스 생성
    
    ```java
    public class DeleteAllStatement implements StatementStrategy {
    	public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
    		PreparedStatement ps = c.prepareStatement("delete from users");
    		return ps;
    	}
    }
    ```
    
3. `deleteAll()` 에 전략 패턴 적용
    
    ```java
    public void deleteAll() throws SQLException {
    	Connection c = null;
    	PreparedStatement ps = null;
    
    	try {
    		c = dataSource.getConnection();
    		
    		StatementStrategy strategy = new DeleteAllStatement();
    		ps = strategy.makePreparedStatement(c);		
    
    		ps.executeUpdate();
    	} catch (SQLException e) {
    		throw e;
    	} finally {                 
    		if (ps != null) {
    			try {
    				ps.close();
    			} catch (SQLException e) {}
    		}
    		if (c != null) {
    			try {
    				c.close();
    			} catch (SQLException e) {}
    	}
    }
    
    ```
    

⇒ 코드 내에서 DeleteAllStatement를 사용하도록 고정되어 있어 OCP를 위배한다.

<br>

> DI 적용을 위한 클라이언트/컨텍스트 분리
- 클라이언트(`deleteAll()`)에 들어가야 할 코드
    
    ```java
    StatementStrategy strategy = new DeleteAllStatement();
    ```

<br>

1. try/catch/finally 컨텍스트 코드를 메소드로 분리
    
    ```java
    public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException {
    	Connection c = null;
    	PreparedStatement ps = null;
    
    	try {
    		c = dataSource.getConnection();
    		
    		ps = stnt.makePreparedStatement(c);		
    
    		ps.executeUpdate();
    	} catch (SQLException e) {
    		throw e;
    	} finally {                 
    		if (ps != null) {
    			try {
    				ps.close();
    			} catch (SQLException e) {}
    		}
    		if (c != null) {
    			try {
    				c.close();
    			} catch (SQLException e) {}
    	}
    }
    
    ```
    
2. 클라이언트 책임을 갖도록 `deleteAll()` 재구성
    
    ```java
    public void deleteAll() throws SQLException {
    	StatementStrategy st = new DeleteAllStatement();
    	jdbcContextWithStatementStrategy(st);
    }
    ```

<br>
<br>

## JDBC 전략 패턴 최적화

### `add()` 메소드에 적용하기

1. `AddStatement` 생성
    
    ```java
    public class AddStatement implements StatementStrategy {
    	User user;
    
    	public AddStatement(User user) {
    		this.user = user;
    	}	
    
    	public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
    		PreparedStatement ps = c.preparedStatement("insert into users(id, name, password) values (?,?,?)");
    		ps.setString(1, user.getId());
    		ps.setString(2, user.getName());
    		ps.setString(3, user.getPassword());
    		return ps;
    	}
    }
    ```
    
2. `add()` 메소드 수정
    
    ```java
    public void add(User user) throws SQLException {
    	StatementStrategy st = new AddStatement(user);
    	jdbcContextWithStatementStrategy(st);
    }
    ```
    

⇒ 두 군데에서 JDBC 컨텍스트를 공유해 사용할 수 있게 되었다.

<br>

## 전략과 클라이언트의 동거

- 아직 남아있는 문제점
    1. DAO 메소드마다 새로운 구현 클래스를 만들어야 한다.
    2. StatementStrategy에 전달할 정보가 있다면 생성자와 인스턴스 변수를 만들어야 한다.

<br>

### 로컬 클래스

- UserDao 클래스 안에 내부 클래스로 정의한다.
    - `UserDao` 클래스 밖에서는 사용되지 않기 때문
    - 클래스의 개수 ⬇️
    - 스태틱 클래스의 로컬 변수를 그대로 사용할 수 있음

<br>

```java
public void add(final User user) throws SQLException {
	public class AddStatement implements StatementStrategy {
		public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
			PreparedStatement ps = c.preparedStatement("insert into users(id, name, password) values (?,?,?)");
			ps.setString(1, user.getId());
			ps.setString(2, user.getName());
			ps.setString(3, user.getPassword());
			return ps;
		}
	}

	StatementStrategy st = new AddStatement();
	jdbcContextWithStatementStrategy(st);
}
```

<br>

### 익명 내부 클래스

- 이름을 갖지 않는 클래스로 구현한다.

```java
public void add(final User user) throws SQLException {
	StatementStrategy st = new StatementStrategy() {
		public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
			PreparedStatement ps = c.preparedStatement("insert into users(id, name, password) values (?,?,?)");
			ps.setString(1, user.getId());
			ps.setString(2, user.getName());
			ps.setString(3, user.getPassword());
			return ps;
		}
	}

	jdbcContextWithStatementStrategy(st);
}
```

<br>
<br>

## 컨텍스트와 DI

### JdbcContext의 분리

- `jdbcContextWithStatementStrategy()` 는 다른 DAO에서도 사용 가능하다.
- 이 메소드를 다른 클래스로 분리하자.

1. JdbcContext 클래스 생성
    
    ```java
    public class JdbcContext {
    	private DataSource dataSource;
    	
    	public void setDataSource(DataSource dataSource) {
    		this.dataSource = dataSource;
    	}
    
    	public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
    		Connection c = null;
    		PreparedStatement ps = null;
    	
    		try {
    			c = this.dataSource.getConnection();
    			ps = stnt.makePreparedStatement(c);		
    			ps.executeUpdate();
    		} catch (SQLException e) {
    			throw e;
    		} finally {                 
    			if (ps != null) {
    				try {
    					ps.close();
    				} catch (SQLException e) {}
    			}
    			if (c != null) {
    				try {
    					c.close();
    				} catch (SQLException e) {}
    		}
    ```
    
    - JdbcContext는 그 자체로 독립적인 JDBC 컨텍스트를 제공해주는 서비스 오브젝트이다.
    - 인터페이스를 사이에 두지 않고 DI를 적용하는 특별한 구조를 가진다.
<br>

2. UserDao가 분리된 클래스를 DI 받아 사용하도록 수정
    
    ```java
    public class UserDao {
    	private JdbcContext jdbcContext;
    	
    	public void setJdbcContext(JdbcContext jdbcContext) {
    		this.jdbcContext = jdbcContext;
    	}
    
    	public void add(final User user) throws SQLException {
    		this.jdbcContext.workWithStatementStrategy(
    			new StatementStrategy() { ... }
    		);
    	}
    
    	public void deleteAll() throws SQLException {
    		this.jdbcContext.workWithStatementStrategy(
    			new StatementStrategy() { ... }
    		);
    	}
    ```
<br>
<br>

### JdbcContext의 특별한 DI

> JdbcContext를 스프링 빈으로 등록한 DI
- JdbcContext를 스프링을 이용해 UserDao에 주입하는 것도 DI를 따른다고 볼 수 있다.

<br>

> 코드를 이용한 수동 DI
- UserDao가 JdbcContext에 대한 제어권을 갖고 DI까지 맡는다.

<br>

1. `setDataSource()` 메소드 수정
    
    ```java
    public class UserDao {
    	private JdbcContext jdbcContext;
    
    	public void setDataSource(DataSource dataSource) {
    		this.jdbcContext = new JdbcContext();          // IoC: 오브젝트 생성
    		this.jdbcContext.setDataSource(dataSource);    // DI: 의존성 주입
    		this.dataSource = dataSource;
    	}
    }
    ```
⇒ 긴밀한 관계를 갖는 DAO 클래스와 JdbcContext를 인터페이스 없이 DI에 적용하는 두 방법


<br>
<br>

## 템플릿과 콜백

- 템플릿/콜백 패턴
    - `UserDao` , `StatementStrategy` , `JdbcContext` 에 적용된 패턴
    - 복잡하지만 바뀌지 않는 일정한 패턴의 작업 흐름이 존재한다.
    - 그 중 일부분만 자주 바꿔 사용할 필요가 있을 때 적합하다.
    - 템플릿: 전략 패턴의 컨텍스트 (고정된 틀)
    - 콜백: 익명 내부 클래스 (호출되는 로직)

<br>

### 템플릿/콜백의 동작원리

> 특징
- 단일 메소드 인터페이스를 사용한다.
- 콜백 인터페이스 메소드에서 파라미터는 컨텍스트 정보를 전달받을 때 사용된다.
- DI 방식의 전략  패턴 구조이다.
- 콜백 오브젝트가 내부 클래스로서 클라이언트 메소드 내 정보를 직접 참조한다.
- 클라이언트와 콜백이 강하게 결합된다.

<br>

> 동작 과정
1. 클라이언트 역할은 콜백 오브젝트를 만들고, 콜백이 참조할 정보를 제공하는 것이다. 만들어진 콜백은 클라이언트가 템플릿의 메소드를 호출할 때 파라미터로 전달된다.
2. 템플릿은 정해진 작업 흐름을 따라 진행하다가 콜백 오브젝트 메소드를 호출한다. 콜백은 클라이언트 메소드 정보와 템플릿에서 제공한 참조정보로 작업을 수행하고 결과를 템플릿에 돌려준다.
3. 템플릿은 콜백이 돌려준 정보를 사용해 작업을 마저 수행한다.

<br>
<br>

### 콜백의 재활용

- 클라이언트 DAO의 메소드는 간결해지고 최소한의 데이터 액세스 로직만 갖고 있다.
- 익명 내부 클래스가 상대적으로 코드를 작성하고 읽기가 좀 불편하다.

⇒ 템플릿/콜백을 `UserDao` 에도 적용해보자!

<br>

기존 코드

```java
public void deleteAll() throws SQLException {
	this.jdbcContext.workWithStatementStrategy(
		new StatementStrategy() { 
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
				return c.prepareStatement("delete from users");
			}
		}
	);
}
```

<br>

- 변하지 않는 부분: 콜백 클래스 정의와 오브젝트 생성
- 변하는 부분 = SQL 문장인 “delete from users”

<br>

메소드 분리

```java
public void deleteAll() throws SQLException {
	executeSql("delete from users");
}

public void executeSql(final String query) throws SQLException {
	this.jdbcContext.workWithStatementStrategy(
		new StatementStrategy() { 
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
				return c.prepareStatement(query);
			}
		}
	);
}
```

<br>

- `executeSql()` 을 모든 DAO 클래스에서 사용할 수 있으면 좋겠다.
- `JdbcContext` 클래스로 옮기자.

<br>

```java
public class JdbcContext {
	...
	public void executeSql(final String query) throws SQLException {
		this.jdbcContext.workWithStatementStrategy(
			new StatementStrategy() { 
				public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
					return c.prepareStatement(query);
				}
			}
		);
	}
}
```

```java
public void deleteAll() throws SQLException {
	this.jdbcContext.executeSql("delete from users");
}
```

⇒ 익명 내부 클래스를 없애 깔끔하고 단순하게 만들었다.

<br>

- JdbcContext의 코드들은 하나의 목적을 위해 서로 긴밀하게 연관되어 동작하는 응집력 강한 코드들이기 때문에 한 군데 모여있는 것이 유리하다.
- 구체적인 구현과 기술은 최대한 감춰두고, 외부에는 꼭 필요한 기능만 제공하는 단순 메소드만 노출한다.

<br>
<br>

### 템플릿/콜백의 응용

- 스프링에서 적극적으로 활용하는 디자인패턴

<br>

템플릿/콜백 적용 방법

1. 중복되는 코드를 먼저 메소드로 분리한다.
2. 그중 일부를 필요에 따라 바꿔 사용해야 하면 인터페이스를 사이에 두고 전략 패턴과 DI를 적용한다.
3. 바뀌는 부분이 한 애플리케이션 안에서 동시에 여러 종류가 만들어질 수 있다면 템플릿/콜백 패턴을 고려해본다.

<br>

> 테스트에 적용해보기

<br>

파일을 열어 모든 라인의 숫자를 더한 합을 돌려주는 테스트 코드

```java
public class CalcSumTest {
	@Test
	public void sumOfNumbers() throws IOException {
		Calculator calculator = new Calculator();
		int sum = calculator.calcSum(getClass().getResource("numbers.txt").getPath());
		assertThat(sum, is(10));
	}
}
```

<br>

`Calculator` 클래스 짜기

```java
public class Calculator {
	public Integer calcSum(String filepath) throws IOException {
		BufferedReader br = new BufferedReader(new FileReader(filepath));
		Integer sum = 0;
		String line = null;
		while ((line = br.readLine()) != null) {
			sum += Integer.valueOf(line);
		}
	
		br.close();
		return sum;
	}
}
```

⇒ 파일을 읽다가 예외가 발생하면 파일이 정상적으로 닫히지 않고 메소드를 빠져나간다.

<br>

`Calculator` 클래스에 예외처리 코드 추가

```java
public class Calculator {
	public Integer calcSum(String filepath) throws IOException {
		BufferedReader br = null;
		
	try {
			br = new BufferedReader(new FileReader(filepath));
			Integer sum = 0;
			String line = null;
			while ((line = br.readLine()) != null) {
				sum += Integer.valueOf(line);
			}
			return sum;
	} catch(IOException e) {
		System.out.println(e.getMessage());
		throw e;
	} finally {
		if (br != null) {		
			try { br.close(); }
			catch(IOException e) { System.out.println(e.getMessage()); }
		}
	}
}
```

<br>

`Calculator` 에 곱셈, 뺄셈 기능도 추가될 예정이다.

<br>

콜백 인터페이스 생성

```java
public interface BufferedReaderCallback {
	Integer doSomethinsWithReader(BufferedReader br) throws IOException;
}
```

<br>

템플릿 부분을 메소드로 분리

```java
public Integer fileReadTemplate(String filepath, bufferedReaderCallback callback) throws IOException {
	BufferedReader br = null;
	try {
		br = new BufferedReader(new FileReader(filepath));
		int ret = callback.doSomethingWithReader(br);
		return ret;
	} catch(IOException e) {
		System.out.println(e.getMessage());
		throw e;
	} finally {
		if (br != null) {		
			try { br.close(); }
			catch(IOException e) { System.out.println(e.getMessage()); }
		}
	}
```

<br>

`calcSum()` 메소드 수정

```java
public class Calculator {
	public Integer calcSum(String filepath) throws IOException {
		BufferedReaderCallback sumCallback = new BufferedReaderCallback() {
			public Integer doSomethingWithReader(BufferedReader br) throws IOException {
				Integer sum = 0;
				String line = null;
				while ((line = br.readLine()) != null) {
					sum += Integer.valueOf(line);
				}
				return sum;
			}
		};
		return fileReadTemplate(filepath, sumCallback);
	}
}
```

<br>

테스트마다 사용되는 오브젝트와 파일 이름이 공유된다.

<br>

`@Before` 픽스처 구현 및 새 테스트 추가

```java
public class CalcSumTest {
	Calculator calculator;
	String numFilepath;

	@Before public void setUp() {
		this.calculator = new Calculator();
		this.numFilepath = getClass().getResource("numbers.txt").getPath();

	@Test
	public void sumOfNumbers() throws IOException {
		assertThat(calculator.calcSum(this.numFilepath), is(10));
	}

	@Test
	public void sumOfNumbers() throws IOException {
		assertThat(calculator.calcMultiply(this.numFilepath), is(24));
	}
}
```

<br>

`calcMultiply()` 콜백 메소드 추가

```java
public Integer calcMultiply(String filepath) throws IOException {
	BufferedReaderCallback multiplyCallback = new BufferedReaderCallback() {
		public Integer doSomethingWithReader(BufferedReader br) throws IOException {
			Integer multiply = 1;
			String line = null;
			while ((line = br.readLine()) != null) {
				multiply *= Integer.valueOf(line);
			}
			return multiply;
		}
	};
	return fileReadTemplate(filepath, multiplyCallback);
}
```

`Calculator` 클래스의 메소드가 유사한 형태를 보여주고 있다.

<br>

`Calculator` 에서 변하는 부분을 콜백 인터페이스로 분리

```java
public interface LineCallback {
	Integer doSomethingWithLine(String line, Integer value);
}
```

<br>

`LineCallback` 을 사용하는 템플릿 (변하지 않는 부분 분리)

```java
public Integer lineReadTemplate(String filepath, LineCallback callback, int initVal) throws IOException {
	BufferedReader br = null;
	try {
		br = new BufferedReader(new FileReader(filepath));
		Integer res = initVal;
		String line = null;
		while((line = br.readLine()) != null) {
			res = callback.doSomethingWithLine(line, res);
		}
		return res;
	}
	catch(IOException e) { ... }
	finally { ... }
}
```

<br>

`Calculator` 클래스가 템플릿을 사용하도록 수정

```java
public Integer calcSum(String filePath) throws IOException {
	LineCallback sumCallback = new LineCallback() {
		public Integer doSomethingWithLine(String line, Integer value) {
			return value + Integer.valueOf(line);
		}};
	return lineReadTemplate(filepath, sumCallback, 0);
}
```

<br>

제네릭스를 활용해 Integer 타입 뿐만 아니라 다양한 타입을 읽을 수 있도록 한다.

```java
public interface LineCallback<T> {
	T doSomethingWithLine(String line, T value);
}
```

```java
public <T> T lineReadTemplate(String filepath, LineCallback<T> callback, T initVal) throws IOException {
	BufferedReader br = null;
	try {
		br = new BufferedReader(new FileReader(filepath));
		T res = initVal;
		String line = null;
		while((line = br.readLiine()) != null) {
			res = callback.doSomethingWithLine(line, res);
		}
		return res;
	}
	catch(IOException e) {...}
	finally {...}
}
```

⇒ 범용적인 템플릿/콜백 패턴을 완성했다.

<br>
<br>

## 스프링의 JdbcTemplate

- 스프링은 거의 모든 종류의 JDBC 코드에 사용가능한 템플릿과 콜백을 제공한다.

```java
public class UserDao {
	private JdbcTemplate jdbcTempalte;

	public void setDataSource(DataSource dataSource) {
		this.jdbcTemplate = new JdbcTemplate(dataSource);
		this.dataSource = dataSource;
	}
```

<br>

### `update()`

`deleteAll()` 에 적용하자.

```java
public void deletAll() {
	this.jdbcTemplate.update(
		new PreparedStatementCreator() {
			public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
				return con.prepareStatement("delete from users");
			}
		}
	);
}
```

```java
public void deleteAll() {
	this.jdbcTemplate.update("delete from users");
}
```

<br>

### `queryForInt()`

`getCount()` 을 콜백/템플릿으로 만들어보자.

```java
public int getCount() {
	return this.jdbcTemplate.query(new PreparedStatementCreator() {
		public PreparedStatement createPreparedStatement(Connection cno) throws SQLExceptio n{
			return con.prepareStatement("select count(*) from users");
		]
	}, new ResultSetExtractor<Integer>() {
		public Integer extractData(ResultSet rs) throws SQLEcxeption, DataAccessException {
			rs.next();
			return rs.getInt(1);
		}
	});
}
```

⇒ 두개의 콜백을 사용하고 있다.

<br>

```java
public int getCount() {
	return this.jdbcTemplate.queryForInt("select count(*) from users");
}
```

<br>

### `queryForObject()`

`get()` 메소드에 JdbcTemplate를 적용해보자.

```java
public User get(String id) {
	return this.jdbcTemplate.queryForObject("select * from users where id = ?",
		new Object[] {id},
		new RowMapper<User>() {
			public User mapRow(ResultSet rs, int rowNum) throws SQLException {
				User user = new User();
				user.setId(rs.getString("id"));
				user.setName(rs.getString("name"));
				return user;
			}
		}
	});
}
```

<br>

- 조회 결과가 없는 예외 상황을 어떻게 처리할까?
    - 특별히 처리할 필요가 없다.
    - `queryForObject()` 는 SQL을 실행한 결과의 row 개수가 하나가 아니면 예외를 던진다.

<br>

### `query()`

- `getAll()` 메소드를 JdbcTemplate으로 만들어보자.

<br>

1. 테스트 만들기
    
    ```java
    @Test
    public void getAll() {
    	dao.deleteAll();
    
    	dao.add(user1);
    	List<User> users1 = dao.getAll();
    	assertThat(users1.size(), is(1));
    	checkSameUser(user1, users1.get(0));
    
    	dao.add(user2);
    	List<User> users2 = dao.getAll();
    	assertThat(users2.size(), is(2));
    	checkSameUser(user1, users2.get(0));
    	checkSameUser(user2, users2.get(1));
    
    	dao.add(user3);
    	List<User> users3 = dao.getAll();
    	assertThat(users3.size(), is(3));
    	checkSameUser(user1, users3.get(0));
    	checkSameUser(user2, users3.get(1));
    	checkSameUser(user3, users3.get(2));
    }
    
    private void checkSameUser(User user1, User user2) {
    	assertThat(user1.getId(), is(user2.getId()));
    	assertThat(user1.getName(), is(user2.getName()));
    }
    ```
    
<br>

2. `query` 를 이용해 `getAll()` 구현
    - `queryForObject()` : 쿼리 결과가 로우 하나 일 때 사용
    - `query()` : 쿼리 결과가 로우 여러 개일 때 사용
    
    ```java
    public List<User> getAll() {
    	return this.jdbcTemplate.query("select * from users order by id",
    		new RowMapper<User>() {
    			public User mapRow(ResultSet rs, int rowNum) throws SQLException {
    				User user = new User();
    				user.setId(rs.getString("id"));
    				user.setName(rs.getString("name"));
    				return user;
    			}
    		});
    }
    ```

<br>

3. 테스트 보완
    - 현재 긍정적인 결과만 테스트하고 있다.
    - 예외상황에 대한 테스트(네거티브 테스트)를 추가하자.
    - 데이터가 없는 경우에 대한 테스트
        
        ```java
        public void getAll() {
        	dao.deleteAll();
        	
        	List<User> users0 = dao.getAll();
        	assertThat(users0.size(), is(0));
        	...
        ```
 
<br>
<br>

## 재사용 가능한 콜백의 분리

> DI를 위한 코드 정리
- 필요 없어진 DataSource 인스턴스 변수는 제거한다.
- JdbcTemplate을 이용하므로 DataSource의 필요가 사라졌다.

```java
// UserDao의 DI 코드

public void setDataSource(DataSource dataSource) {
	this.jdbcTemplate = new JdbcTemplate(dataSource);
}
```

<br>

> 중복 제거
- `get()` 과 `getAll()` 에서 사용하는 RowMapper의 내용이 똑같다.
- RowMapper는 ResultSet 로우 하나를 User 오브젝트 하나로 변환하는 기능을 수행한다.
- User용 RowMapper 콜백을 메소드에서 분리해 재사용하자.

```java
public class UserDao {
	private RowMapper<User> userMapper = 
		new RowMapper<User>() {
			public User mapRow(ResultSet rs, int rowNum) throws SQLException {
				User user = new User();
				user.setId(rs.getString("id"));
				user.setName(rs.getString("name"));
			}
	};

	...

	public User get(String id) {
		return this.jdbcTemplate.queryForObject("select * from users where id = ?", 
				new Object[] {id}, this.userMapper);
	}

	public User getAll() {
		return this.jdbcTemplate.queryForObject("select * from users order by id", 
				this.userMapper);
	}
```

<br>

> 최종
- UserDao는 User 정보를 DB에 넣거나 조작하는 방법에 대한 핵심로직만 갖고 있다.
- JDBC API를 사용하는 방식, DB 연결 등의 책임과 관심은 모두 JdbcTemplate에게 있다.
- userMapper를 UserDao 빈의 DI용 프로퍼티로 만든다면?
    - User 테이블 정보나 매핑 방식이 바뀌어도 UserDao 코드 수정 없이 변경할 수 있다.
- SQL 문장을 UserDao 코드가 아니라 외부 리소스에 담고 읽어와 사용한다면?
    - 필드 이름 변경 또는 쿼리 최적화 시에 Dao코드에 손을 댈 필요가 없다.
