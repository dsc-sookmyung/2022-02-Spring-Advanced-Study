# 예외

## 사라진 SQLException

### 초난감 예외처리

- 초난감 예외처리 예시코드 1: 예외 블랙홀
    
    ```java
    try {
    	...
    }
    catch(SQLException e) {
    }
    ```
    
    ```java
    try {
    	...
    }
    catch(SQLException e) {
    	System.out.println(e);
    }
    ```
    
    ```java
    try {
    	...
    }
    catch(SQLException e) {
    	e.printStackTrace();
    }
    ```
    
    ⇒ 모든 예외는 적절하게 복구되든지 또는 작업을 중단해 분명히 통보해야 한다.

<br>
    
- 초난감 예외처리 예시코드 2: 무책임한 throws
    
    ```java
    public void method1() throws Exception {
    	method2();
    	...
    }
    
    public void method2() throws Exception {
    	method3();
    	...
    }
    
    public void method3() throws Exception {
    	...
    }
    ```


<br>

### 예외의 종류와 특징

자바에서 throw를 통해 발생시킬 수 있는 예외는 크게 3가지가 있다.

1. **Error**
    - `java.lang.Error` 클래스의 서브클래스들
    - 시스템에 비정상적인 상황이 발생했을 경우에 사용
    - 주로 자바 VM에서 발생시키고, 애플리케이션 코드에서 잡으면 X
    
2. **Exception과 체크 예외**
    - `Exception` 클래스 = 체크 예외 + 언체크 예외
        - 체크 예외: RuntimeException 클래스를 상속 X
        - 언체크 예외: RuntimeException 클래스를 상속 O
    - 일반적인 예외는 체크 예외라고 생각하면 된다.
    - 체크 예외는 반드시 catch 문으로 잡든지, throws를 정의해 밖으로 던져야 한다.
    
3. **RuntimeException과 언체크/런타임 예외**
    - 언체크 예외 = 런타임 예외
    - 프로그램에 오류가 있을 때 발생하도록 의도된 것
    - ex) `NullPointerException`, `IllegalArgumentException`
    - 미리 코드에서 조건을 체크하도록 주의 깊게 만들면 피할 수 있다.
    - 즉, 굳이 catch나 throws를 사용하지 않아도 된다.
    

<br>

<br>

### 예외처리 방법

> **예외를 처리하는 일반적인 방법**
1. 예외 복구
    - 예외 상황을 파악하고 문제를 해결해 정상 상태로 돌려놓는 것
    - 체크 예외는 어떤 식이로든 복구할 가능성이 있는 경우에 사용
    - ex) `IOException`
        - 해당 파일이 없거나 다른 문제가 있어 읽히지 않는다.
        - 사용자에게 상황을 알려주고 다른 파일로 안내해 예외를 피할 수 있다.
        - 그러나 에러 메세지가 사용자에게 그냥 던져져서는 안된다.
    - ex) `SQLException`
        - 네트워크가 불안해 원격 DB 서버 접속에 실패했다.
        - 일정 시간 대기했다가 다시 접속을 시도해 복구를 시도할 수 있다.
    - 예시 코드
        
        ```java
        int maxretry = MAX_RETRY;
        while (maxretry -- > 0) {
        	try {
        		...                          // 예외 발생 가능성이 있는 시도
        		return;                      // 작업 성공
        	}
        	catch(SomeException e) {
        		// 로그 출력. 정해진 시간만큼 대기
        	}
        	finally {
        		// 리소스 반납. 정리 작업
        	}
        }
        throw new RetryFailedException(); //최대 재시도 횟수를 넘기면 직접 예외 발생
        ```
        

<br>

2. 예외처리 회피
    - 예외처리를 직접 담당하지 않고 자신을 호출한 쪽으로 던져버린다.
    - `throws` 문 사용
        
        ```java
        public void add() throws SQLException {
        	// JDBC API
        }
        ```
        
    - catch 문으로 일단 예외를 잡아 로그를 남긴 후 다시 예외를 던짐
        
        ```java
        public void add() throws SQLException {
        	try {
        		// JDBC API
        	}
        	catch(SQLException e) {
        		// 로그 출력
        		// throw e;
        	}
        }
        ```
        
    
    3장에서 만든 콜백 오브젝트는 모두 `SQLException`을 템플릿으로 던지고 있다.
    
    - `SQLException`을 처리하는 일은 콜백 오브젝트 역할이 아니라고 보기 때문이다.
    - 그러나 예외 회피도 예외를 복구하는 것처럼 의도가 분명해야 한다.

<br>

3. 예외 전환
    - 발생한 예외를 적절한 예외로 전환해서 밖으로 던진다.
    - 예외 전환의 목적
        1. 내부 예외를 그대로 던지는 것이 적절한 의미를 부여해주지 못하는 경우
            - ex) 사용자 등록 시 중복된 아이디 값으로 에러가 나는 경우
                
                ```java
                public void add(User user) throws DuplicateUserIdException, SQLException {
                	try {
                		// JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
                		// 그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드
                	}
                	catch(SQLException e) {
                		// ErrorCode가 MySQL의 "Duplicate Entry(1062)"이면 예외 전환
                		if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
                			throw DuplicateUserIdException();
                		else
                			throw e;
                	}
                }
                ```
                
                - 보통 중첩 예외로 만들어 반환하는 것이 좋다.
                    
                    예시 1)
                    
                    ```java
                    catch(SQLException e) {
                    	...
                    	throw DuplicateUserIdException(e);
                    ```
                    
                    예시 2)
                    
                    ```java
                    catch(SQLException e) {
                    	...
                    	throw DuplicateUserIdException().initCause(e);
                    ```
                    
        2. 체크 예외를 언체크 예외인 런타임 예외로 바꾸는 경우에 사용


<br>

<br>

### 예외처리 전략

> **런타임 예외의 보편화**
- 대응이 불가능한 체크 예외는 빨리 런타임 예외로 전환해 던지는 게 낫다.
- 최근 등장하는 프레임워크는 언체크 예외를 더 일반화하고 있다.

> **add() 메소드의 예외처리**
- 사용자 아이디가 중복됐을 때 사용할 예외
    
    ```java
    public class DuplicateUserIdException extendsd RuntimeException {
    	public DuplicateUserIdException(Throwable cause) {
    		super(cause);
    	}
    }
    ```
    

- `add()` 메소드 수정
    - `SQLException`을 직접 밖으로 던지지 않고, 런타임 예외로 전환해 던진다.
    - 아이디 중복으로 예외가 발생한 경우는 `DuplicateUserIdException` 을 그대로 던진다.
        - `add()` 메소드를 사용하는 쪽에서 아이디 중복 예외 처리를 하고 싶은 경우 활용할 수 있음을 알려주도록 throws 선언에 포함시킨다.
    
    ```java
    public void add(User user) throws DuplicateUserIdException, SQLException {
    	try {
    		// JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
    		// 그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드
    	}
    	catch(SQLException e) {
    		// ErrorCode가 MySQL의 "Duplicate Entry(1062)"이면 예외 전환
    		if (e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
    			throw new DuplicateUserIdException(e);    // 예외 전환
    		else
    			throw RuntimeException(e);                // 예외 포장
    	}
    }
    ```
    
<br>

> **애플리케이션 예외**
- **런타임 예외 중심의 전략**은
    - 낙관적인 예외처리 기법
    - 직접 처리할 수 없는 예외가 대부분이더라도 놓치는 예외가 있을 수 있다.

- **애플리케이션 예외**는,
    - 애플리케이션 자체의 로직에 의해 의도적으로 발생시키고, 반드시 `catch` 해서 조치를 취하도록 요구하는 예외
    - 의도적으로 체크 예외로 만들어 자주 발생 가능한 예외 상황에 대한 로직을 구현하도록 강제한다.
    - ex) 예금을 인출 시 잔고 부족을 애플리케이션 예외로 만들어 처리하는 코드
        
        ```java
        try {
        	BigDecimal balance = account.withdraw(amount);
        	...
        	// 정상적인 처리 결과를 출력하도록 진행
        }
        catch(InsufficientBalanceException e) {            // 체크 예외
        	// InsufficientBalanceException에 담긴 인출 가능한 잔고금액 정보를 가져옴
        	BigDecimal availFunds = e.getAvailFunds();
        	...
        	// 잔고 부족 안내 메시지를 준비하고 이를 출력하도록 진행
        }
        ```

<br>

### SQLException은 어떻게 됐나?

DAO에 존재하는 `SQLException`에 대해 생각해보자.

- `SQLException` 은 복구 가능한 예외인가?
    - 99%는 코드 레벨에서 복구할 방법이 없다.
    - 개발자에게 가능한 빨리 전달하는 것 외에 할 수 있는게 없다.
    - 따라서 예외처리 전략을 적용해야 한다.

- 스프링 JdbcTemplate는 언체크/런타임 예외 전환이라는 전략을 따른다.
    - 모든 SQLException을 런타임 예외인 `DataAccessException`으로 포장해 던진다.
    - 즉, 꼭 필요한 경우에만 `DataAccessException` 을 잡아 처리하면 된다.
    
<br>

## 예외 전환

예외 전환의 목적

1. 런타임 예외로 포장해 필요하지 않은 `catch/throws` 를 줄인다.
2. 로우레벨의 예외를 좀 더 의미 있고 추상화된 예외로 바꾼다.

<br>

### JDBC의 한계

**JDBC**

- 자바를 이용해 DB에 접근하는 방법을 추상화된 API 형태로 정의해놓고, 각 DB가 JDBC 표준을 따라 만들어진 드라이버를 제공한다.
- 표준 인터페이스가 제공되기 때문에 개발자들은 DB 종류에 상관없이 일관된 방법으로 프로그램을 개발한다.
- 그러나 현실적으로는 DB를 자유롭게 바꿔 사용할 수 있는 DB 프로그램을 작성하기에 두 가지 걸림돌이 있다.

<br>

> 첫 번째 문제점. **비표준 SQL**
- 대부분의 DB는 표준인 SQL을 따르지 않는 비표준 문법과 기능을 제공한다.

<br>

**해결방법**

- 호환 가능한 표준 SQL만 사용한다. → 현실성 X
- DAO 별로 DAO를 만들거나 SQL을 외부에 독립시켜서 DB에 따라 변경한다.

<br>

> 두 번째 문제점. **호환성 없는 SQLException의 DB 에러 정보**
- DB를 사용하다가 발생하는 `SQLException` 의 원인은 다양하다.
- DB마다 발생하는 에러의 종류와 원인도 모두 제각각이라, JDBC는 모두 `SQLException` 하나로 퉁쳐버린다.

<br>

### DB 에러 코드 매핑을 통한 전환

- DB별 에러 코드를 참고해서 발생한 예외의 원인이 무엇인지 해석하는 기능을 만든다.
- 스프링은 `DataAccessException` 의 서브 클래스로 데이터 액세스 작업 중에 발생하는 예외 상황을 수십 가지의 예외클래스로 제공한다.

⇒ 스프링은 DB별 에러 코드를 분류해 스프링이 정의한 예외 클래스와 매핑해놓은 에러 코드 매핑정보 테이블을 만들어 사용한다.

```jsx
<bean id="Oracle" class="org.springframework.jdbc.support.SQLErrorCodes">
	<property name="badSqlGrammarCodes">
		<value>900, 903, 904, 917, 936, 942, 17006</value>
	</property>
	...
</bean>
```
<br>

이렇게 JdbcTemplate를 이용하면 JDBC에서 발생하는 DB 관련 예외는 거의 신경 쓰지 않아도 된다.

<br>

<br>

### DAO 인터페이스와 DataAccessException 계층구조

스프링이 왜 `DataAccessException` 계층구조를 이용해 예외를 정의하고 사용하는지 생각해보자.

<br>

> **DAO 인터페이스와 구현의 분리**

**DAO를 굳이 따로 만들어 사용하는 이유**

- 데이터 액세스 로직을 담은 코드를 성격이 다른 코드에서 분리하기 위해
- 전략 패턴을 적용해 구현 방법을 변경해서 사용하기 위해

그러나 메소드 선언에 나타나는 예외정보는 문제를 야기한다.

```jsx
public interface UserDao {
	public void add(User user) throws SQLException;
	...
}
```

이 메소드는 JDBC가 아닌 JPA, Hibernate, JDO ORM에서는 사용할 수 없다. 던지는 예외 클래스가 달라지기 때문이다.

<br>

<br>

**해결 방법**

1. `Exception` 으로 대체한다. ⇒ 간단하나 무책임하다.
2. JDBC API의 `SQLException` 을 런타임 예외로 포장해 던진다.

그러나, 이것만으로는 불충분하다.

<br>

> **데이터 액세스 예외 추상화와 DataAccessException 계층구조**

그래서 스프링은 `DataAccessException` 계층구조를 지원한다. `DataAccessException` 클래스들은 자바의 주요 데이터 액세스 기술에서 발생 가능한 대부분의 예외를 추상화한다.

→ `JdbcTemplate`과 같이 스프링의 데이터 액세스 지원 기술로 DAO를 만들면 기술에 독립적인 일관성 있는 예외를 던질 수 있다.

<br>

### 기술에 독립적인 UserDao 만들기

> 인터페이스 적용

```java
public interface UserDao {
	void add(User user);
	User get(String id);
	List<User> getAll();
	void deleteAll();
	int getCount();
}
```

<br>

**기존의 UserDao 이름 변경**

```java
public class UserDaoJdbc implements UserDao {
	...
}
```

<br>

> 테스트 보완

```java
@Test(expected=DataAccessException.class)
public void dupliciateKey() {
	dao.deleteAll();
	
	dao.add(user1);
	dao.add(user1);
}
```

<br>

> DataAccessException 활용 시 주의사항

학습 테스트를 만들어 SQLException을 직접 해석해 DataAccessException으로 변환하는 코드 사용법을 살펴보자.

```java
public class UserDaoTest{
	@Autowired UserDao dao;
	@Autowired DataSource dataSource;

	@Test
	public void sqlExceptionTranslate() {
		dao.deleteAll();

		try {
			dao.add(user1);
			dao.add(user2);
		}
		catch(DuplicateKeyException ex) {
			SQLException sqlEx = (SQLException)ex.getRootCause();
			SQLExceptionTranslator set = new SQLErrorCodeSQLExceptionTranslator(this.dataSource);
	
			assertThat(set.translate(null, null, sqlEx), is (DuplicateKeyException.class));
	}
```
