# 📗 토비의 스프링 3.1 Vol.1 스프링의 이해와 원리
## 📝 3장 템플릿
확장에는 자유롭게 열려 있고, 변경에는 굳게 닫혀 있다는 객체지향 설계의 핵심 원칙인 '**개방 폐쇄 원칙(OCP)**'
>이 원칙은 코드에서 
>
>어떤 부분은 변경 통해 그 기능이 다양해지고, 확장하려는 성질이 있고, 
>
>어떤 부분은 고정되어 있고 변하지 않으려는 성질이 있음을 말해줌.
>
>변화의 특성이 다른 부분을 구분해주고, 각각 다른 목적과 이유에 의해 다른 시점에 독립적으로 변경될 수 있는 효율적인 구조를 만들어주는 것이 바로 이 '개방 폐쇄 원칙'

<br>

### ✅ 템플릿
>바뀌는 성질이 다른 코드 중에서
>
>변경이 거의 일어나지 않으며 **일정한 패턴으로 유지되는 특성을 가진 부분**을
>
>**자유롭게 변경되는 성질을 가진 부분으로부터 독립시켜**
>
>**효과적으로 활용할 수 있도록 하는 방법.**
#### ✨ 3장에서 다루는 것 : 스프링에 적용된 템플릿 기법, 이를 적용해 완성도 있는 DAO 코드 만드는 방법
<br>

## 3.1 다시 보는 초난감 DAO
### 💥 UserDao에 아직 남아있는 문제점 : 예외상황에 대한 처리
어떤 상황에서도 가져온 리소스를 반환하도록 `try/catch/finally` 구문 사용을 권장 중.
예외상황에서도 리소스를 제대로 반환할 수 있도록 `try/catch/finally`를 적용해보자.
#### Fix: 예외 발생 시에도 리소스를 반환하도록 수정한 deleteAll()
```java
public void deleteAll() throws SQLException {  
	Connection c = null;  
	PreparedStatement ps = null;  
	try{
		c = dataSource.getConnection();  
		ps = c.prepareStatement("delete from users");  
		ps.executeUpdate();  
  }catch (SQLException e) {  
        throw e;  
  }finally {  
        if(ps!=null){  
            try{  
                ps.close();  
  }catch (SQLException e){  
                  
            }  
        }  
        if(c!=null){  
            try{  
                c.close();  
  }catch (SQLException e){  
                  
            }  
        }  
    }  
}
```
> - `finally`는 `try` 블록을 수행한 후에 예외가 발생하든 정상적으로 처리되든 상관없이 반드시 실행되는 코드를 넣을 때 사용함.
> 
> - `close()`는 만들어진 순서의 반대로 하는 것이 원칙

<br>

조회를 위한 JDBC 코드는 `Connection, PreparedStatement` 외에도 `ResultSet`이 추가되어 조금 더 복잡해짐.

등록된 User의 수를 가져오는 `getCount()` 메소드에 예외처리 블록을 적용해보면 다음과 같은 코드가 만들어짐.
#### Fix: JDBC 예외처리를 적용한 getCount() 메소드
```java
public int getCount() throws SQLException{  
	Connection c = null;  
	PreparedStatement ps = null;  
	ResultSet rs = null;  
  
 try{
	 c = dataSource.getConnection();  
	 ps = c.prepareStatement("select count(*) from users");  
	 rs = ps.executeQuery();  
	 rs.next();  
	 return rs.getInt(1);  
  } catch (SQLException e) {  
        throw e;  
  } finally {  
        if(rs != null){  
            try{  
                rs.close();  
  } catch (SQLException e){  
            }  
        }  
        if(ps != null){  
            try{  
                ps.close();  
  } catch (SQLException e) {  
            }  
        }  
        if(c != null){  
            try{  
                c.close();  
  } catch (SQLException e) {  
            }  
        }  
    }  
}
```
<br> 
=> 서버환경에서도 안정적으로 수행 + DB 연결 기능을 자유롭게 확장할 수 있는 이상적인 DAO 완성

하지만, 뭔가 아쉬움이 남아있음😅

## 3.2 변하는 것과 변하지 않는 것
### 💥 JDBC try/catch/finally 코드의 문제점 : 복잡한 try/catch/finally 블록이 2중으로 중첩, 모든 메소드마다 반복
이런 코드를 효과적으로 다룰 수 있는 방법 : 

> 변하지 않는, 그러나 많은 곳에서 중복되는 코드와 로직에 따라 자꾸 확장되고 자주 변하는 코드를 잘 분리해내는 작업임. 다만, 이번 코드는 DAO와 DB연결 기능을 분리하는 것과는 성격이 다르기 때문에 해결 방법이 조금 다름.

### 분리와 재사용을 위한 디자인 패턴 적용
#### `deleteAll()` 메소드에서 변하는 부분
```java
ps = c.prepareStatement("delete from users");  
```
#### 그 이외의 코드는 변하지 않음.
=> 변하는 부분을 변하지 않는 나머지 코드에서 분리하자.
<br>

## 생각해볼 수 있는 방법: 

### 1️⃣ 첫 번째 방법,  변하는 부분을 메소드로 추출 

=> 별 이득 없음. 

보통 메소드 추출 리팩토링을 적용하는 경우, 

분리시킨 메소드를 다른 곳에서 재사용 가능해야 하는데, 

이건 반대로 분리시키고 남은 메소드가 재사용 필요한 부분이고, 

분리된 메소드는 DAO 로직마다 새롭게 만들어서 확장돼야 하는 부분이기 때문.

**[결론] : 뭔가 반대로 됨..😅**
<br>

### 2️⃣ 두 번째 방법, 템플릿 메소드 패턴 이용해 분리
템플릿 메소드 패턴 : 
- 상속 통해 기능 확장해 사용하는 부분

- 변하지 않는 부분은 슈퍼클래스에 두고, 변하는 부분은 **추상 메소드**로 정의해둬서 서브클래스에서 오버라이드하여 새롭게 정의해 쓰도록 하는 것.
#### Feat: 추상 메소드 `makeStatement()` 
```java
abstract protected PreparedStatement makeStatement(Connection c) throws SQLException;
```
#### Feat: `makeStatement()`를 구현한 UserDao 서브 클래스(UserDao를 상속해서 만든 클래스)
```java
public class UserDaoDeleteAll extends UserDao{
	protected PreparedStatement makeStatement(Connection c) throws SQLException{
		PreparedStatement ps = c.prepareStatement("delete from users");
		return ps;
	}
}
```
=> UserDao 클래스 기능 확장하고 싶을 때마다 상속 통해 자유롭게 확장 가능

확장 때문에 기존의 상위 DAO 클래스에 불필요한 변화는 생기지 않도록 할 수 있음 -> 객체지향 설계의 핵심 원리인 개방 폐쇄 원칙(OCP) 그럭저럭 지키는 구조.

But, 템플릿 메소드 패턴으로의 접근은 제한 多

**💥 문제 1 : DAO 로직마다 상속통해 새로운 클래스 만들어야 함.**

예를 들어, UserDao의 JDBD 메소드가 4개일 경우, 4개의 서브클래스를 만들어서 사용해야 함.

**💥 문제 2 : 확장구조가 이미 클래스를 설계하는 시점에서 고정되어 버림.**

관계에 대한 유연성이 떨어져 버림. 

상속을 통해 확장을 꾀하는 템플릿 메소드 패턴의 단점이 고스란히 드러남

**[결론] : 장점보다 단점이 더 많아 보임😥**
<br>

### 3️⃣ 세 번째 방법, 전략 패턴의 적용
개방 폐쇄 원칙을 잘 지키는 구조이면서도 템플릿 메소드 패턴보다 유연하고 확장성 뛰어난 것 

**=  '전략 패턴'** 
### ✅ 전략 패턴
- 오브젝트를 아예 둘로 분리하고 클래스 레벨에서는 인터페이스를 통해서만 의존하도록 만드는 방식

- OCP(개방 폐쇄 원칙) 관점에서 보면 확장에 해당하는 변하는 부분을 별도의 클래스로 만들어 추상화된 인터페이스를 통해 위임하는 방식

**<전략 패턴의 구조>**
- 컨텍스트(Context)의 메소드에서 일정한 구조 가지고 동작하다가, 특정 확장 기능은 전략(Strategy) 인터페이스를 통해 외부의 독립된 전략 클래스에 위임

`deleteAll()` 메소드에서 변하지 않는 부분 = 컨텍스트의 메소드(`contextMethod()`)가 됨.

`deleteAll()`은 JDBC를 이용해 DB를 업데이트하는 작업이라는 변하지 않는 맥락(context)를 가짐.

**<deleteAll() 의 컨텍스트 정리>**
- DB 커넥션 가져오기
- **PreparedStatement 를 만들어줄 외부 기능 호출하기 ➡️ 전략 패턴에서 말하는 '전략'**
- 전달받은 preparedStatement 실행하기
- 예외가 발생하면 이를 다시 메소드 밖으로 던지기
- 모든 경우에 만들어진 PrepareStatement와 Connection을 적절히 닫아주기'

#### 👀 눈여겨볼 것 : 이 PreparedStatement 생성 전략을 호출할 때는 이 컨텍스트 내에서 만들어둔 DB 커넥션을 전달해야 한다. 

#### Feat: PreparedStatement를 만드는 전략의 인터페이스
```java
public interface StatementStrategy {
	PreparedStatement makePreparedStatement(Connection c) throws SQLException;
}
```
#### Feat: 실제 전략, 즉 바뀌는 부분인 PreparedStatement를 생성하는 클래스(전략 클래스)
```java
public class DeleteAllStatement implements StatementStrategy{  
    public PreparedStatement makePreparedStatement(Connection c) throws SQLException{  
	    PreparedStatement ps = c.prepareStatement("delete from users");  
		return ps;  
  }  
}
```
#### Fix: 전략 패턴을 따라 DeleteAllStatement가 적용된 `deleteAll()` 메소드
```java
public void deleteAll() throws SQLException {  
	...
	try{
		c = dataSource.getConnection();  
		StatementStrategy strategy = new DeleteAllStatement();
		ps = strategy.makePreparedStatement(c);

		ps.executeUpdate();
  }catch (SQLException e) {  
        ...
}
```
=> 하지만, 전략 패턴은 필요에 따라 컨텍스트는 그대로 유지되면서(OCP의 폐쇄 원칙) 전략을 바꿔 쓸 수 있다(OCP의 개방 원칙)는 것인데, 이렇게 컨텍스트 안에서 이미 구체적인 전략 클래스인 DeleteAllStatemen를 사용하도록 **고정되어 있다면 뭔가 이상**함.

컨텍스트가 StatementStrategy 인터페이스뿐 아니라 특정 구현 클래스인 DeleteAllStatement를 직접 알고 있다는 건, 전략 패턴에도 OCP에도 잘 들어맞는다고 볼 수 없기 때문.
<br>

### ✅ DI 적용을 위한 클라이언트/컨텍스트 분리
DI란 전략 패턴의 장점을 일반적으로 활용할 수 있도록 만든 구조.
####  컨텍스트에 해당하는 JDBC try/catch/finally 코드를 클라이언트 코드인 StatementStrategy를 만드는 부분에서 독립시켜야 함.
#### 클라이언트에 들어가야 할 코드
```java
StatementStrategy strategy = new DeleteAllStatement(); 
```
#### 나머지 코드는 컨텍스트 코드이므로, 별도의 메소드로 독립시켜보자.
클라이언트는 DeleteAllStatement 오브젝트 같은 전략 클래스의 오브젝트를 컨텍스트 메소드로 전달해야 함. -> **전략 인터페이스인 StatementStrategy를 컨텍스트 메소드 파라미터로 지정**할 필요 있음.
#### Feat: 메소드로 분리한 try/catch/finally 컨텍스트 코드
```java
public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException{  
	Connection c = null;  
	PreparedStatement ps = null;  
	try{
		c = dataSource.getConnection();
		
		ps = stmt.makePreparedStatement(c);
		
		ps.executeUpdate();
	} catch(SQLException e) {
		throw e;
	} finally{
		if(ps!=null) { try { ps.close(); } catch(SQLException e}{} }
		if(c!=null) { try { c.close(); } catch(SQLException e}{} }
	}
}
```
#### Feat: 클라이언트 책임을 담당할 `deleteAll()` 메소드
```java
public void deleteAll() throws SQLException {
	StatementStrategy st = new DeleteAllStatement(); // 선정한 전략 클래스의 오브젝트 생성
	jdbcContextWithStatementStrategy(st); // 컨텍스트 호출. 전략 오브젝트 전달
}
```
> `deleteAll()` 은 전략 오브젝트를 만들고 컨텍스트를 호출하는 책임 지고 있음.
> 
> - 사용할 전략 클래스인 `DeleteAllStatement` 의 오브젝트를 생성하고, 컨텍스트로 분리한 `jdbcContextWithStatementStrategy()` 메소드를 호출

=> 구조로 볼 때 완벽한 전략 패턴의 모습.

비록 클라이언트와 컨텍스트 클래스를 분리하진 않았지만,

의존관계와 책임으로 볼 때 이상적인 클라이언트/컨텍스트 관계를 갖고 있음.

특히, 클라이언트가 컨텍스트가 사용할 전략을 정해서 전달한다는 면에서 'DI 구조'

이렇게 분리한 것에 크게 장점이 보이진 x

그러나 이 구조가 기반이 돼서 앞으로 진행할 UserDao 코드의 본격적인 개선 작업이 가능함.
<br>

### ✅ 마이크로 DI
p. 224
- [DI의 가장 중요한 개념]

	- 제3자의 도움을 통해 두 오브젝트 사이의 유연한 관계가 설정되도록 만든다는 것.
- [DI 적용 방법]

	- 일반적으로는 의존관계에 있는 두 개의 오브젝트와 이 관계를 다이내믹하게 설정해주는 오브젝트 팩토리(DI 컨테이너), 이를 사용하는 클라이언트라는 4개의 오브젝트 사이에서 일어남.

	- 클라이언트가 오브젝트 팩토리의 책임을 함께 지거나, 클라이언트와 전략(의존 오브젝트)이 결합될 수도 있음.
	- 심지어는 클라이언트와 DI 관계에 있는 두 개의 오브젝트가 모두 하나의 클래스 안에 담길 수도 있음.
- [마이크로 DI]
	- DI의 장점을 단순화해서 IoC 컨테이너의 도움 없이 코드 내에서 적용한 경우 
<br>

## 3.3 JDBC 전략 패턴의 최적화
아래 방법을 따라가 보면, 

try/catch/finally로 범벅된 코드를 만들다가 실수할 염려는 없어지고, 

DAO 코드는 간결해진다. 

**DAO 메소드** : 전략 패턴의 클라이언트. 
- 컨텍스트에 해당하는 `jdbcContextWithStatementStrategy()` 메소드에 적절한 전략, 즉 바뀌는 로직을 제공해주는 방법으로 사용 가능
	- 컨텍스트 : PreparedStatement 실행하는 JDBC의 작업 흐름

	- 전략 : PreparedStatement를 생성하는 것.

### 🤗 `add()` 메소드에도 적용해보자.
add() 메소드에서 변하는 부분인 PreparedStatement 만드는 코드를 AddStatement 클래스로 옮겨 담는다.
#### Feat: add() 메소드의 PreparedStatement 생성 로직을 분리한 클래스
```java
public class AddStatement implements StatementStrategy{
	public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
		PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?, ?, ?)");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());
	
		return ps;
	}
}
```
user라는 부가적인 정보가 필요할 경우, 클라이언트로부터 User 타입 오브젝트를 받을 수 있도록 AddStatement의 생성자를 통해 제공받게 만들자.
#### Feat: User 정보를 생성자로부터 제공받도록 만든 AddStatement
```java
	public class AddStatement implements StatementStrategy{
		User user;
	
		public AddStatement(User user){
			this.user = user;
		}
		public PreparedStatement makePreparedStatement(Connection c){
			...
			ps.setString(1, user.getId());
			ps.setString(2, user.getName());
			ps.setString(3, user.getPassword());
			...
		}
	}
```
#### Feat: user정보를 AddStatement에 전달해주는 add() 메소드
```java
	public void add(User user) throws SQLException{
		StatementStrategy st = new AddStatement(user);
		jdbcContextWithStatementStrategy(st);
	}
```
## 그런데, 현재까지의 구조에 두 가지 불만😥
#### 1️⃣ DAO 메소드마다 새로운 StatementStrategy 구현 클래스를 만들어야 함.
- 이렇게 되면 기존 UserDao보다 클래스 파일의 개수가 많이 늘어남.

- 런타임 시에 다이내믹하게 DI 해준다는 점 빼고는 로직마다 상속을 사용하는 템플릿 메소드 패턴을 적용했을 때보다 그다지 나은 게 없음.
#### 2️⃣ DAO 메소드에서 StatementStrategy에 전달할 User와 같은 부가적 정보 있는 경우, 이를 위해 오브젝트를 전달받는 생성자와 이를 저장해둘 인스턴스 변수를 번거롭게 만들어야 함.
- 이 오브젝트가 사용되는 시점은 컨텍스트가 전략 오브젝트를 호출할 때이므로 잠시라도 어딘가에 다시 저장해둘 수밖에 없음.

## ✅ 이 두 가지 문제를 해결할 수 있는 방법
✅ 해결 방안 step1 : `UserDao` 클래스 안에 내부 클래스로 정의해버리자(**로컬 클래스**로 만들자)
- `add()` 메소드 내의 로컬 클래스로 이전한 `AddStatement` 클래스 - p. 227
	- 장점 : 
	
		- 메소드마다 추가해야 했던 클래스 파일 하나 줄일 수 있음. 
		
		- 내부 클래스의 특징을 이용해서 로컬 변수를 바로 가져다 사용 가능.

✅ 해결 방안 step2  : `AddStatement`를 익명 내부 클래스로 만들자 - p.230

✅ 해결 방안 step3  : 익명 내부 클래스의 오브젝트는 딱 한 번만 사용할 테니 굳이 변수에 담아두지 말고, `jdbcContextWithStatementStrategy()` 메소드의 파라미터에서 바로 생성하자
#### 최종 `add()` 메소드
```java
public void add(final User user) throws SQLException{
	jdbcContextWithStatementStrategy(new StatementStrategy() {
		public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
			PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
			ps.setString(1, user.getId()); // 로컬(내부) 클래스의 코드에서 외부의 메소드 로컬 변수에 직접 접근 가능
			ps.setString(2, user.getName());
			ps.setString(3, user.getPassword());
			
			return ps;
		}
	}
	);
}
```
✅ 해결 방안 step4 : 마찬가지로 DeleteAllStatement도 deleteAll()로 가져와서 익명 내부 클래스로 처리
#### 익명 내부 클래스를 적용한 `deleteAll()` 메소드
```java
public void deleteAll() throws SQLException{
	jdbcContextWithStatementStrategy(
		new StatementStrategy() {
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException{
				return c.prepareStatement("delete from users");
			}
		}
	);
}
```
<br>

### 📝 중첩 클래스의 종류
**[중첩 클래스(nested class)]**
- 다른 클래스 내부에 정의되는 클래스

**[중첩 클래스 종류]** 
- **스태틱 클래스** (독립적으로 오브젝트로 만들어질 수 있는 클래스)

- **내부 클래스** (자신이 정의된 클래스의 오브젝트 안에서만 만들어질 수 있는 클래스)
	- 범위에 따라 3가지로 구분됨.
		- **멤버 내부 클래스** (멤버 필드처럼 오브젝트 레벨에 정의되는 멤버 내부 클래스)

		- **로컬 클래스** (메소드 레벨에 정의되는 클래스)
			- 마치 로컬 변수 선언하듯이 선언하면 됨. 

			- 선언된 메소드 내에서만 사용 가능
			- 클래스가 내부 클래스이기 때문에 자신이 선언된 곳의 정보에 접근할 수 있음.
			- 다만, 내부 클래스에서 외부의 변수를 사용할 때는 외부 변수는 반드시 `final`로 선언해줘야 함.
		- **익명 내부 클래스** (이름을 갖지 않는 클래스)
			- 클래스 선언과 오브젝트 생성이 결합된 형태로 만들어지며, 상속할 클래스나 구현할 인터페이스를 생성자 대신 사용해서 다음과 같은 형태로 만들어 사용함. 

			- 클래스를 재사용할 필요 없고, 구현한 인터페이스 타입으로만 사용할 경우에 유용함.
			- `new 인터페이스이름( ) { 클래스 본문 };`
<br>  


## 3.4 컨텍스트와 DI
#### 전략 패턴의 구조로 보자면,
- 클라이언트 : UserDao의 메소드.

- 개별적인 전략 : 익명 내부 클래스로 만들어지는 것.
- 컨텍스트 : `jdbcContextWithStatementStrategy()` 메소드
- 컨텍스트 메소드 : UserDao 내의 PreparedStatement를 실행하는 기능을 가진 메소드에서 공유

JDBC의 일반적인 작업 흐름을 담고 있는 `jdbcContextWithStatementStrategy()` 메소드를 다른 DAO에서도 사용 가능 

-> `jdbcContextWithStatementStrategy()` 를 UserDao 클래스 밖으로 독립시켜서 모든 DAO가 사용할 수 있게 해보자.

>- 분리해서 만들 클래스의 이름 : JdbcContext
>
>- JdbcContext에 UserDao에 있던 컨텍스트 메소드를 `workWithStatementStrategy()`라는 이름으로 옮겨놓음.
> - DataSource 타입 빈을 DI 받을 수 있게 해줘야 함. 
>- UserDao가 분리된 JdbcContext를 DI 받아서 사용할 수 있게 만듦.
>- 소스코드 - p.232~233

🌱 스프링의 빈 설정은 클래스 레벨이 아니라 런타임 시에 만들어지는 오브젝트 레벨의 의존관계에 따라 정의됨.
>- JdbcContext 빈을 추가하도록 XML 설정파일 수정
>
>- 소스코드 - p.234

<br>

## JdbcContext의 특별한 DI
UserDao는 인터페이스를 거치지 않고 코드에서 바로 JdbcContext 클래스를 사용 중.

UserDao와 JdbcContext는 클래스 레벨에서 의존관계가 결정됨. 

비록, 런타임 시에 DI 방식으로 외부에서 오브젝트를 주입해주는 방식을 사용하긴 했지만, 의존 오브젝트의 구현 클래스를 변경할 순 없음.

### ✅ 스프링 빈으로 DI
**인터페이스 사용하지 않고 DI 적용해도 괜찮음.**

>의존관계 주입(DI)이라는 개념을 충실히 따르자면,
>
>인터페이스를 사이에 둬서 클래스 레벨에서는 의존관계가 고정되지 않게 하고, 
>
>런타임 시에 의존할 오브젝트와의 관계를 다이내믹하게 주입해주는 것이 맞음.
>
>따라서 **인터페이스를 사용하지 않았다면 엄밀히 말해서 온전한 DI 라고 볼 순 X**
>
>### BUT, **스프링의 DI는 넓게 보자면 객체의 생성과 관계설정에 대한 제어권한을 오브젝트에서 제거하고 외부로 위임했다는 IoC라는 개념을 포괄함.**
>
>그런 의미에서, JdbcContext를 스프링을 이용해 UserDao 객체에서 사용하게 주입했다는 건 **DI의 기본을 따르고 있다고 볼 수 있음.**

### 인터페이스를 사용해서 클래스를 자유롭게 변경할 수 있게 하진 않았지만, JdbcContext를 UserDao와 DI 구조로 만들어야 할 이유는..?
1) JdbcContext가 스프링 컨테이너의 싱글톤 레지스트리에서 관리되는 싱글톤 빈이 되기 때문.

2) JdbcContext가 DI를 통해 다른 빈에 의존하고 있기 때문.
- DI를 위해서는 주입되는 오브젝트와 주입받는 오브젝트 양쪽 모두 스프링 빈으로 등록돼야 함.

- 스프링이 생성하고 관리하는 IoC 대상이어야 DI에 참여할 수 있기 때문.
- 다른 빈을 DI 받기 위해서라도 JdbcContext는 스프링 빈으로 등록돼야 함.

#### 실제로 스프링엔 드물지만, 이렇게 인터페이스를 사용하지 않는 클래스를 직접 의존하는 DI가 등장하는 경우 있음. 
### 왜 인터페이스를 사용하지 않았을까?
>- 인터페이스가 없다는 건 UserDao와 JdbcContext가 매우 긴밀한 관계를 가지고 강하게 결합되어 있다는 의미.
>
>- 비록 클래스는 구분되어 있지만 이 둘은 강한 응집도 갖고 있음.
>- 이런 경우는 굳이 인터페이스를 두지 말고 강력한 결합을 가진 관계를 허용하면서 위에서 말한 두 가지 이유인, 싱글톤으로 만드는 것과 JdbcContext에 대한 DI 필요성을 위해 스프링의 빈으로 등록해서 UserDao에 DI 되도록 만들어도 좋음.
>- 단, 이런 클래스를 바로 사용하는 코드 구성을 DI에 적용하는 것은 가장 마지막 단계에서 고려해볼 사항임.

<br>

### ✅ 코드를 이용하는 수동 DI
JdbcContext를 스프링의 빈으로 등록해서 UserDao에 DI 하는 대신 사용할 수 있는 방법 : **'UserDao 내부에서 직접 DI를 적용하는 방법'** 

> 싱글톤으로 만들려는 것은 포기해야 함.
> 
> 오브젝트가 수백개 만들어진다고 해도 메모리에 주는 부담 거의 없음.
> 
> 자주 만들어졌다가 제거되는 게 아니기 때문에 GC에 대한 부담도 없음.
> 
> JdbcContext의 제어권은 UserDao가 갖는 것이 적당함. 
> 
> JdbcContext는 DataSource 타입 빈을 다이내믹하게 주입받아서 사용해야 함.
> 
> 그렇지 않으면, DataSource 구현 클래스를 자유롭게 바꿔가면서 적용 불가.
> 
> 하지만, JdbcContext 자신은 스프링의 빈이 아니니 DI 컨테이너를 통해 DI 받을 수 없음.

이런 경우에 사용하는 방법 : 

✅ **JdbcContext에 대한 제어권을 갖고 생성과 관리를 담당하는 UserDao에게 DI까지 맡기는 것.**
> 오브젝트를 생성하고 그 의존 오브젝트를 수정자 메소드로 주입해주는 것이 바로 DI의 동작원리
> UserDao가 임시로 DI 컨테이너처럼 동작하게 만들면 됨.
#### jdbcContext 빈을 제거한 설정파일
```xml
<beans>
	<bean id="userDao" class="springbook.user.dao.UserDao">
		<property name="dataSource" ref="dataSource" />
	</bean>
	<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
	...
	</bean>
</beans>
```
#### JdbcContext 생성과 DI 작업을 수행하는 `setDataSource()` 메소드
```java
public class UserDao{
	...
	private JdbcContext jdbcContext;
	public void setDataSource(DataSource dataSource){
		this.jdbcContext = new JdbcContext();
		this.jdbcContext.setDataSource(dataSource);
		this.dataSource = dataSource;
	}
}
```
장점 : 굳이 인터페이스를 두지 않아도 될 만큼 긴밀한 관계를 갖는 DAO 클래스와 JdbcContext를 어색하게 따로 분리하지 않고 내부에서 직접 만들어 사용하면서도 다른 오브젝트에 대한 DI를 적용할 수 있다는 점 -> 스프링에서도 종종 사용되는 기법.

<br>

## 인터페이스 없이 DAO와 밀접한 관계 갖는 클래스를 DI에 적용하는 방법 두 가지 정리
1️⃣ **스프링 빈으로 DI** - 인터페이스를 사용하지 않는 클래스와의 의존관계이지만 스프링의 DI를 이용하기 위해 빈으로 등록해서 사용하는 방법 

- 장점 : 오브젝트 사이의 실제 의존관계가 설정파일에 명확하게 드러남

- 단점 : DI 근본적인 원칙에 부합하지 않는 구체적인 클래스와의 관계가 설정에 직접 노출됨.

2️⃣ **코드를 이용하는 수동 DI** - DAO의 코드를 이용해 수동으로 DI 하는 방법
- 장점 : 내부에서 만들어지고 사용되면서 그 관계를 외부에는 드러내지 않음 -> 필요에 따라 내부에서 은밀히 DI를 수행하고 그 전략을 외부에는 감출 수 있음.

- 단점 : 여러 오브젝트가 사용하더라도 싱글톤으로 만들 수 없고, DI 작업을 위한 부가적인 코드가 필요함.

어떤 방법이 더 낫다고 말할 수 x

상황에 따라 적절하다고 판단되는 방법 선택해서 사용하면 됨.

다만, 왜 그렇게 선택했는지 분명한 이유, 근거 설명할 자신 없다면 차라리 인터페이스 만들어서 평범한 DI 구조로 만드는 게 나을 수 있음.

<br>  

## 3.5 템플릿과 콜백
###  ✅ 템플릿/콜백 패턴
- 전략 패턴의 기본 구조에 익명 내부 클래스를 활용한 방식

- 복잡하지만 바뀌지 않는 일정한 패턴을 갖는 작업 흐름이 존재하고, 그 중 일부분만 자주 바꿔서 사용해야 하는 경우에 적합한 구조
- 전략 패턴의 컨텍스트 = **템플릿**
	 - 프로그래밍에서는 고정된 틀 안에 바꿀 수 있는 부분을 넣어서 사용하는 경우에 템플릿이라고 부름.

	 - 템플릿 메소드 패턴은 고정된 틀의 로직을 가진 템플릿 메소드를 슈퍼클래스에 두고, 바뀌는 부분을 서브 클래스의 메소드에 두는 구조로 이뤄짐.
	 - JSP도 일종의 템플릿 파일 (HTML이라는 고정된 부분에 EL과 스크립릿이라는 변하는 부분을 넣은 것이라서!)

- 익명 내부 클래스로 만들어지는 오브젝트 = **콜백**
	- 콜백은 실행되는 것을 목적으로 다른 오브젝트의 메소드에 전달되는 오브젝트를 말함.

	- 파라미터로 전달되지만 값을 참조하기 위한 것이 아닌 특정 로직을 담은 메소드를 실행시키기 위해 사용함.
	- 자바에선 메소드 자체를 파라미터로 전달할 방법은 없기 때문에 메소드가 담긴 오브젝트를 전달해야 함. =>  펑셔널 오브젝트(functional object) 라고도 함.

<br>

##  템플릿/콜백의 동작원리
템플릿 : 고정된 작업 흐름을 가진 코드를 재사용한다는 의미에서 붙인 이름

콜백 : 템플릿 안에서 호출되는 것을 목적으로 만들어진 오브젝트

### ✅ 전략 VS 콜백
- 전략 패턴의 전략 : 
	- 여러 개의 메소드를 가진 일반적인 인터페이스를 사용할 수 있음

- 템플릿/콜백 패턴의 콜백 : 
	- 보통 하나의 메소드를 가진 인터페이스를 구현한 익명 내부 클래스로 만들어짐 (템플릿의 작업 흐름 중 특정 기능을 위해 한 번 호출되는 경우가 일반적이기 때문)
	- **하나의 템플릿에서 여러 가지 종류의 전략 사용해야 한다면, 하나 이상의 콜백 오브젝트를 사용할 수도 있음.**
	- 콜백 인터페이스의 메소드에는 보통 파라미터 있음. (템플릿의 작업 흐름 중에 만들어지는 커텍스트 정보 받을 때 사용됨)

<br>

### ✅ 템플릿/콜백 패턴의 일반적인 작업 흐름 - 자세한 그림 p.242
1) Callback 생성

2) Callback 전달 / Template 호출
3) Workflow 시작
4) 참조정보 생성
5) Callback 호출 / 참조정보 전달
6) Client final 변수 참조
7) 작업 수행
8) Callback 작업 결과
9) Workflow 진행
10) Workflow 마무리
11) Template 작업 결과
>클라이언트의 역할 : 템플릿 안에서 실행될 로직 담은 콜백 오브젝트 만들고, 콜백이 참조할 정보 제공. 
>
>만들어진 콜백은 클라이언트가 템플릿의 메소드를 호출할 때 파라미터로 전달됨.
>
>템플릿은 정해진 작업 흐름 따라 작업 진행하다가 내부에서 생성한 참조정보를 가지고 콜백 오브젝트의 메소드 호출.
>
>콜백은 클라이언트 메소드에 있는 정보와 템플릿이 제공한 참조정보를 이용해 작업 수행하고 그 결과를 다시 템플릿에 돌려줌.
>
>템플릿은 콜백이 돌려준 정보를 사용해 작업 마저 수행함. 
>
>경우에 따라 최종 결과를 클라이언트에 다시 돌려주기도 함.

<br>

### ✅ 템플릿/콜백 방식의 특징
- 매번 메소드 단위로 사용할 오브젝트를 새롭게 전달받음.
- 콜백 오브젝트가 내부 클래스로서 자신을 생성한 클라이언트 메소드 내의 정보를 직접 참조
- 클라이언트와 콜백이 강하게 결합됨.
- 전략 패턴과 DI의 장점을 익명 내부 클래스 사용 전략과 결합한 독특한 활용법이라고 이해할 수 있음.
- 템플릿과 클라이언트가 메소드 단위임.

<br>

### ✅ 템플릿/콜백 방식의 장단점
- 장점 : 클라이언트인 DAO의 메소드는 간결해지고, 최소한의 데이터 액세스 로직만 갖고 있게 됨.

- 단점 : DAO 메소드에서 매번 익명 내부 클래스를 사용하기 때문에 상대적으로 코드 작성하고 읽기기 조금 불편함.

<br>

##  편리한 콜백의 재활용
### ✅ 콜백의 분리와 재활용
🧐 복잡한 익명 내부 클래스의 사용을 최소화할 수 있는 방법
- 중복될 가능성이 있는 자주 바뀌지 않는 부분을 분리하자.
- SQL 문장만 바뀌니까 파라미터로 받아서 바꿀 수 있게 하고, 메소드 내용 전체를 분리해 별도의 메소드로 만들어보자.
- 바뀌지 않는 모든 부분을 빼내서 executeSQL() 메소드로 만들자.
```java
public void deleteAll() throws SQLException {
	executeSql("delete from users");
}

private void executeSql(final String query) throws SQLException {
	this.jdbcContext.workWithStatementStrategy(
		new StatementStrategy() {
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
				return c.prepareStatement(query);
			}
		}
	)
}
```
- 재활용 가능한 콜백을 담은 메소드가 만들어짐.

<br>

### ✅ 콜백과 템플릿의 결합
재사용 가능한 콜백을 담고 있는 메소드(`executeSql()`) 라면 DAO 가 공유할 수 있는 템플릿 클래스 안으로 옮겨도 됨.

- JdbcContext 클래스 안으로 `executeSql()` 메소드를 옮기자.

- `deleteAll()` 메소드에서도 jdbcContext를 통해 `executeSql()` 메소드 호출하도록 `this.jdbcContext.executeSql("delete from users");` 로 수정

=> 이제 모든 DAO 메소드에서 executeSql() 메소드 사용 가능.

결국, JdbcContext 안에 클라이언트와 템플릿, 콜백이 모두 함께 공존하면서 동작하는 구조가 됨.

일반적으로는 성격 다른 코드들은 가능한 한 분리하는 편이 낫지만, 이 경우는 반대임.

하나의 목적을 위해 서로 긴밀하게 연관되어 동작하는 응집력 강한 코드 => 한 군데 모여 있는 게 유리

<br>

## 스프링에서 제공하는 템플릿/콜백 클래스와 API
고정된 작업 흐름을 갖고 있으면서 여기저기서 자주 반복되는 코드가 있다면, 중복되는 코드를 분리할 방법을 생각해보는 습관을 기르자.

#### 중복된 코드?

- 먼저 메소드로 분리해봄.

- 그중 일부 작업을 필요에 따라 바꾸어 사용해야 한다면 인터페이스를 사이에 두고 분리해 전략 패턴 적용하고 DI로 의존관계를 관리하도록 만들기.
- 바뀌는 부분이 한 애플리케이션 안에서 동시에 여러 종류가 만들어질 수 있다면 이번엔 템플릿/콜백 패턴을 적용하는 것을 고려해볼 수 있음.

<br>

#### 가장 전형적으로 템플릿/콜백 패턴을 적용해야 하는 경우는? 
- try/catch/finally 블록을 사용하는 코드

<br>

#### ✅ 템플릿/콜백 패턴 적용할 때 생각해야 하는 순서
1) 템플릿에 담을 반복되는 작업 흐름은 어떤 것인지 살펴보기

2) 템플릿이 콜백에게 전달해줄 내부의 정보는 무엇이고, 콜백이 템플릿에게 돌려줄 내용은 무엇인지 생각해보자

3) 템플릿이 작업을 마친 뒤 클라이언트에게 전달해줘야할 것도 있는지 살펴보자.

4) 즉, **템플릿과 콜백의 경계를 정하고, 템플릿이 콜백에게 콜백이 템플릿에게 각각 전달하는 내용이 무엇인지 파악해** 그에 따라 콜백의 인터페이스를 정의하자.

#### 💡 템플릿/콜백 패턴 적용 예시 p.248 ~ p.256

<br>

### 제네릭스를 이용한 콜백 인터페이스
제네릭스 : 자바 언어에 타입 파라미터라는 개념을 도입한 것
- 다양한 오브젝트 타입을 지원하는 인터페이스나 메소드를 정의할 수 있음.
<br>

#### <스프링의 템플릿/콜백 패턴이 적용된 곳에서 종종 사용되는 기법>
- 리턴 값을 갖는 템플릿 
- 템플릿 내에서 여러 번 호출되는 콜백 오브젝트
- 제네릭스 타입 갖는 메소드나 콜백 인터페이스 

<br>

## 3.6 스프링의 JdbcTemplate
스프링이 제공하는 템플릿/콜백 기술을 살펴보자.
- 스프링은 JDBC를 이용하는 DAO에서 사용할 수 있도록 준비된 다양한 템플릿과 콜백 제공

	- 거의 모든 종류의 JDBC 코드에 사용 가능한 템플릿과 콜백을 제공할 뿐만 아니라, 자주 사용되는 패턴을 가진 콜백은 다시 템플릿에 결합시켜서 간단한 메소드 호출만으로 사용이 가능하도록 만들어져 있기 때문에 템플릿/콜백 방식의 기술을 사용하고 있는지 모르고도 쓸 수 있을 정도로 편리.

### ✅ 스프링이 제공하는 JDBC 코드용 기본 템플릿 : JdbcTemplate
- 앞에서 만든 JdbcContext와 유사하지만 훨씬 강력 + 편리한 기능 제공.
- JdbcContext는 버리고 스프링의 JdbcTemplate으로 변경하자.
#### - 템플릿을 사용할 준비
```java
public class UserDao {
	...
	private JdbcTemplate jdbcTemplate;
	
	public void setDataSource(DataSource dataSource){
		this.jdbcTemplate = new JdbcTemplate(dataSource);

		this.dataSource = dataSource;
	}
}
```
#### 💡 소스코드 p.260 ~ p.274

1. `deleteAll()` 에서 적용했던 콜백은 StatementStrategy 인터페이스의 `makePreparedStatement()` 메소드 

	=> **JdbcTemplate의 콜백인 PreparedStatementCreator 인터페이스의 `createPreparedStatement()` 메소드로 변경**
	- PreparedStatementCreator 타입의 콜백을 받아서 사용하는 JdbcTemplate 의 템플릿 메소드는 `update()` 메소드임.

2. add() 메소드에서 만드는 콜백은 PreparedStatement를 만드는 것과 파라미터를 바인딩하는 두 가지 작업을 수행함.

	=> **PreparedStatement를 만들 때 사용하는 SQL은 동일하고 바인딩할 파라미터는 순서대로 넣어줌.**

3. getCount() 메소드

	=> PreparedStatementCreator 콜백과 ResultSetExtractor 콜백을 파라미터로 받는 `query()` 메소드로 변경
	-  ResultSetExtractor 콜백 : 템플릿이 제공하는 ResultSet을 이용해 원하는 값을 추출해서 템플릿에 전달하면, 템플릿은 나머지 작업을 수행한 뒤에 그 값을 `query()` 메소드의 리턴값으로 돌려줌. 
	- ResultSetExtractor는 제네릭스 타입 파라미터를 가짐.
	
	=> **이런 기능을 가진 콜백 내장한 `queryForInt()` 메소드로 변경하자.** 

4. `get()` 메소드

	=> RowMapper 콜백 사용하고 `queryForObject()` 메소드로 변경하자.
		- RowMapper가 ResultSetExtractor과 다른 점 : ResultSet의 로우 하나를 매핑하기 위해 사용되기 때문에 여러 번 호출될 수 있음. 
		
5. 현재 등록되어 있는 모든 사용자 정보를 가져오는 `getAll()` 메소드 

	=> JdbcTemplate의 `query()` 메소드 사용해서 만들자.
	- `queryForObject()` VS `query()` 
	- 전자는 쿼리의 결과가 로우 하나일 때 사용하고, 후자는 여러 개의 로우가 결과로 나오는 일반적인 경우에 쓸 수 있음.
	
6. 테스트 보완

	=> 네거티브 테스트라고 불리는 예외상황에 대한 테스트 체크하기.
	- 네거티브 테스트 : `get()`이라면 Id가 없을 때는 어떻게 되는지, `getAll()`이라면 결과가 하나도 없는 경우에는 어떻게 되는지를 검증하는 것.

	✅ 예외적인 조건에 대해 먼저 테스트를 만드는 습관을 들여보자.

7. DI를 위한 코드 정리

	=> UserDao에서 필요 없어진 DataSource 인스턴스 변수 제거

8. 중복 제거

	=> `get()`과 `getAll()`에서 사용한 RowMapper 의 내용 중복
	
	=> User용 RowMapper 콜백을 메소드에서 분리해 중복을 없애고 재사용되게 만들자.
	- userMapper라는 이름으로 인스턴스 변수 만들고, 사용할 매핑용 콜백 오브젝트를 초기화하도록 만듦.

#### JdbcTemplate을 적용한 UserDao 클래스
```java
public class UserDao{
	public void setDataSource(DataSource dataSource){
		this.jdbcTemplate = new JdbcTemplate(dataSource);
	}
	
	private JdbcTemplate jdbcTemplate;

	private RowMapper<User> userMapper = new RowMapper<User>(){
		public User mapRow(ResultSet rs, int rowNum) throws SQLException {
			User user = new User();
			user.setId(rs.getString("id"));
			user.setName(rs.getString("name"));
			user.setPassword(rs.getString("password"));
			return user;
		}
	};

	public void add(final User user){
		this.jdbcTemplate.update("insert into users(id, name, password) values(?,?,?)", user.getId(), user.getName(), user.getPassword());
	}
	
	public User get(String id){
		return this.jdbcTemplate.queryForObject("select * from users where id = ?", new Object[] {id}, this.userMapper);
	}
	
	public void deleteAll(){
		this.jdbcTemplate.update("delete from users");
	}

	public int getCount(){
		return this.jdbcTemplate.queryForInt("select count(*) from users");
	}

	public List<User> getAll(){
		return this.jdbcTemplate.query("select * from users order by id", this.userMapper);
	}
}
```
#### 여기서 UserDao를 더 개선할 수 있을까?
1. userMapper를 아예 UserDao 빈의 DI용 프로퍼티로 만들어버리는 것은 어떨까?
2. DAO 메소드에서 사용하는 SQL문장을 UserDao 코드가 아니라 외부 리소스에 담고 이를 읽어와 사용하게 하는 것은 어떨까?

일단, 다른 스프링의 기술을 먼저 살펴보고 손을 대보기로 하자.

클래스 이름이 Template으로 끝나거나 인터페이스 이름이 Callback으로 끝난다면 템플릿/콜백 패턴이 적용된 것이라고 보면 된다.