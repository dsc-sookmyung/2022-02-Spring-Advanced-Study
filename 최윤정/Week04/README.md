# 📗 토비의 스프링 3.1 Vol.1 스프링의 이해와 원리
## 📝 4장 예외
#### ✨ 4장에서 다루는 것 : JdbcTemplate을 대표로 하는 스프링의 데이터 액세스 기능에 담겨있는 예외처리와 관련된 접근 방법, 이를 통해 예외를 처리하는 베스트 프랙티스

JdbcContext로 만들었던 코드를 스프링의 JdbcTemplate 적용하도록 바꾼 후 `throws SQLException` 이 사라졌다.

과연 어디로 간 것일까?
## 4.1 사라진 SQLException
## ✅ 만들어서는 안 되는 예외 처리 코드
### 1️⃣ 예외 블랙홀
#### - 예외를 잡고 아무것도 하지 않는 코드
```java
try{
	...
}
catch(SQLException e){
}
```
>#### 왜? 
>프로그램 실행 중에 어디선가 오류가 있어서 예외가 발생했는데 그것을 무시하고 계속 진행해버리기 때문.
>
>결국 발생한 예외로 인해 **어떤 기능이 비정상적으로 동작** or **메모리나 리소스가 소진** or **예상치 못한 다른 문제를 일으킬 것.**
>
>시스템 오류나 이상한 결과의 원인이 무엇인지 찾아내기가 매우 힘들어짐.

<br>

#### -  예외가 발생하면 화면에 출력해주도록 하는 코드
```java
}catch(SQLException e){
	System.out.println(e);
}
```
```java
}catch(SQLException e){
	e.printStackTrace();
}
```
>#### 왜?
>다른 로그나 메시지에 금방 묻혀버리면 놓치기 십상.
>
>운영서버에 올라가면 더욱 심각함.
>
>콘솔 로그를 누군가가 계속 모니터링하지 않는 한 이 예외 코드는 심각한 폭탄으로 남아 있을 것임.

<br>

### ✅ 예외를 처리할 때 반드시 지켜야 할 핵심 원칙 
> **모든 예외는 적절하게 복구되든지 아니면 작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보돼야 함.**

<br>

#### 그나마 나은 예외 처리
```java
}catch(SQLException e){
	e.printStackTrace();
	System.exit(1);
}
```
<br>

### 2️⃣ 무의미하고 무책임한 `throws` 
#### - 메소드에 `throws Exception` 을 붙여 선언하는 경우
```java
public void method1() throws Exception {
	...
}
```
>#### 왜? 
>
>의미 있는 정보를 얻을 수 x
>
>적절한 처리를 통해 복구될 수 있는 예외상황도 제대로 다룰 수 있는 기회 박탈당함.
>
>예외를 무시해버리는 첫 번째 문제보다는 낫다고 하지만 매우 안 좋은 예외처리 방법임.

<br>

## ✅ 예외의 종류와 특징
예외를 어떻게 다뤄야 할까?

가장 큰 이슈 : **체크 예외(checked exception)라고 불리는 명시적인 처리가 필요한 예외를 사용하고 다루는 방법**

자바에서 `throw`를 통해 발생시킬 수 있는 예외는 크게 3가지가 있음.

### 1️⃣ Error
`java.lang.Error` 클래스의 서브클래스들

에러는 시스템에 뭔가 비정상적인 상황이 발생했을 경우에 사용됨.

시스템 레벨에서 특별한 작업을 하는 게 아니라면, 애플리케이션에서는 이런 에러에 대한 처리는 신경 쓰지 않아도 됨.

### 2️⃣ Exception과 체크 예외
`java.lang.Exception` 클래스와 그 서브클래스로 정의되는 예외들은 에러와 달리, 개발자들이 만든 애플리케이션 코드의 작업 중에 예외상황이 발생했을 경우에 사용됨.

#### Exception 클래스
- **체크 예외**
	- Exception 클래스의 서브클래스이면서 `RuntimeException` 클래스를 **상속하지 않은 것**들

	- **일반적으로 예외**라고 하면, **체크 예외**라고 생각해도 됨.
	- 체크 예외가 발생할 수 있는 메소드를 사용할 경우, 반드시 예외를 처리하는 코드를 함께 작성해야 함. 사용할 메소드가 체크 예외를 던진다면, 이를 `catch`문으로 잡든지, 아니면 다시 `throws`를 정의해서 메소드 밖으로 던져야 함. 그렇지 않으면 컴파일 에러 발생.
	- `IOException` 이나 `SQLException` 을 비롯해 예외적인 상황에서 던져질 가능성이 있는 것들 대부분이 체크 예외로 만들어져 있음.

- **언체크 예외**

	- Exception 클래스의 서브클래스이면서 `RuntimeException`을 **상속한** 클래스들

RuntimeException은 Exception의 서브클래스이므로 Exception의 일종이긴 하지만, 

자바는 이 RuntimeException과 그 서브클래스는 특별하게 다룸.

<br>

### 3️⃣ RuntimeException과 언체크/런타임 예외
`java.lang.RuntimeException` **클래스를 상속한 예외**들은 명시적인 예외처리를 강제하지 않기 때문에 **언체크 예외**라고 불림. 또는 대표 클래스 이름을 따서 **런타임 예외**라고도 함.

런타임 예외 = 주로 프로그램의 오류가 있을 때 발생하도록 의도된 것들.
- 오브젝트를 할당하지 않은 레퍼런스 변수를 사용하려고 시도했을 때 발생하는 NullPointerException
- 허용되지 않은 값을 사용해서 메소드를 호출할 때 발생하는 IllegalArgumentException 등

피할 수 있지만 개발자가 부주의해서 발생할 수 있는 경우에 발생하도록 만든 것이 '런타임 예외' 

따라서, 런타임 예외는 예상하지 못했던 예외상황에서 발생하는 게 아니기 때문에 굳이 catch나 throws 사용하지 않아도 되도록 만든 것.

최근에 새로 등장하는 자바 표준 스펙의 API들은 예상 가능한 예외상황을 다루는 예외를 체크 예외로 만들지 않는 경향이 있음.

<br>

#### [결론] 
- 체크예외는 반드시 에러 처리 해야 함(try/catch 또는 throw) 
- 언체크예외는 컴파일러가 에러 처리를 확인하지 않아서 예외처리를 강제하지 않음.

## 예외처리 방법
### 1️⃣ 예외 복구
- 예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것.

- 예외가 처리됐으면 비록 기능적으로는 사용자에게 예외상황으로 비쳐도 애플리케이션에서는 정상적으로 설계된 흐름을 따라 진행돼야 함.
- 정해진 횟수만큼 복구 재시도 -> 실패 -> 예외 복구는 포기해야함.
- 예외처리 코드를 강제하는 '체크 예외'는 예외를 어떤 식으로든 복구할 가능성이 있는 경우에 사용함.
#### 통제 불가능한 외부 요인으로 인해 예외 발생하면 `MAX_RETRY`만큼 재시도를 하는 예시
```java
int maxretry = MAX_RETRY;
while(maxretry -- > 0){
	try{
		... // 예외가 발생할 가능성이 있는 시도
		return; // 작업 성공
	}
	catch(SomeException e){
		// 로그 출력. 정해진 시간만큼 대기
	}
	finally{
		// 리소스 반납. 정리 작업
	}
}
throw new RetryFailedException(); // 최대 재시도 횟수를 넘기면 직접 예외 발생
```
> 사전에 미리 성공여부 확인 불가 + 재시도가 의미 있는 경우 위 코드 사용해보기!

<br>

### 2️⃣ 예외처리 회피
- 예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것

- `throws`문으로 선언해 예외가 발생하면 알아서 던져지게 하거나, `catch`문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던지는(rethrow) 것
- 예외를 자신이 처리하지 않고 회피하는 방법
- 빈 catch 블록으로 잡아서 예외가 발생하지 않은 것처럼 만드는 경우는, 회피한 것이 아님.
- 예외처리를 회피하려면 반드시 **다른 오브젝트나 메소드가 예외를 대신 처리할 수 있도록 던져줘야 함.**
- 예외를 회피하는 것은 예외를 복구하는 것처럼 **의도가 분명해야 함.**
- **콜백/템플릿처럼 긴밀한 관계에 있는 다른 오브젝트에게 예외처리 책임을 분명히 지게 하거나,** **자신을 사용하는 쪽에서 예외를 다루는 게 최선의 방법**이라는 **분명한 확신이 있어야 함.**
#### 예외처리 회피1
```java
public void add() throws SQLException {
	// JDBC API
}
```
#### 예외처리 회피2
```java
public void add() throws SQLException {
	try{
		// JDBC API
	}
	catch(SQLException e){
		// 로그 출력
		throw e;
	}
}
```

### 3️⃣ 예외 전환(exception translation)
- 예외 회피와 비슷하게 예외를 복구해서 정상적인 상태로는 만들 수 없기 때문에 예외를 메소드 밖으로 던지는 것.

- 하지만, 예외 회피와 달리, 발생한 예외를 그대로 넘기는 게 아니라 적절한 예외로 전환해서 던진다는 특징이 있음.

- #### [예외 전환의 두 가지 방법]

	1. 좀 더 의미 있는 예외로 변경

		- API가 발생하는 기술적인 로우레벨을 상황에 적합한 의미를 가진 예외로 변경하는 것.
		- #### 예외 전환 기능을 가진 DAO 메소드
		```java
		public void add(User user) throws DuplicateUserIdException, SQLException {
			try{
			// JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
			// 그런 기능을 가진 다른 SQLException을 던지는 메소드를 호출하는 코드
			}
			catch(SQLException e){
			// ErrorCode가 MySQL의 "Duplicate Entry(1062)"이면 예외 전환
			if(e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
				throw DuplicateUserIdException();
			else 
				throw e; // 그 외의 경우는 SQLException 그대로
			}
		}
		```
		- 보통 전환하는 예외에 원래 발생한 예외를 담아서 '**중첩 예외(nested exception)**' 로 만드는 것이 좋음.

			- 중첩예외는 getCause() 메소드를 이용해 처음 발생한 예외가 무엇인지 확인 가능
			> 새로운 예외를 만들면서 **생성자**나 **`initCause()` 메소드**로 근본 원인이 되는 예외를 넣어주면 됨
			```java
			catch(SQLException e){
				...
				throw DuplicateUserIdException(e);
			```
			```java
			catch(SQLException e){
				...
				throw DuplicateUserIdException().initCause(e);
			```
	<br>
	
	2. 불필요한 catch/throws를 피하기 위해 런타임 예외로 포장(wrap)

		- 예외를 처리하기 쉽고 단순하게 만들기 위해 포장

		- 중첩 예외를 이용해 새로운 예외 만들고, 원인(cause)이 되는 예외를 내부에 담아서 던지는 방식은 같음. 
		- 하지만, 의미를 명확하게 하려고 다른 예외로 전환하는 것이 아니라, 주로 **예외처리를 강제하는 체크 예외를 언체크 예외인 런타임 예외로 바꾸는 경우에 사용**함.
		- 대표적으로 EJBException  : EJB 컴포넌트 코드에서 발생하는 대부분의 체크 예외는 비즈니스 로직으로 볼 때 의미 있는 예외이거나 복구 가능한 예외가 아님.
		```java
		try{
			OrderHome orderHome = EJBHomeFactory.getInstance().getOrderHome();
			Order order = orderHome.findByPrimaryKey(Integer id);
		} catch(NamingException ne) {
			throw new EJBException(ne);
		} catch(SQLException se) {
			throw new EJBException(se);
		} catch(RemoteException re) {
			throw new EJBException(re);
		}
		```
		> EJBException은 RuntimeException 클래스를 상속한 런타임 예외

<br>

✅ **어차피 복구가 불가능한 예외라면 가능한 한 빨리 런타임 예외로 포장해 던지게 해서 다른 계층의 메소드를 작성할 때 불필요한 `throws` 선언이 들어가지 않도록 하는 게 바람직하다.**

✅ **애플리케이션 로직을 담기 위한 예외는 체크 예외로 만든다.**

<br>

## 예외처리 전략
### ✅ 런타임 예외의 보편화
일반적으로 
>- 체크예외 : 일반적 예외 다룸
>
>- 언체크 예외 : 시스템 장애나 프로그램상의 오류에 사용함

💥 문제는 **체크 예외는 복구할 가능성이 조금이라도 있는**, 말 그대로 **예외적인 상황**이기 때문에 **자바는 이를 처리하는 catch 블록이나 throws 선언을 강제하고 있다**는 점이다.

자바 엔터프라이즈 서버환경은 수많은 사용자가 동시에 요청을 보내고 각 요청이 독립적인 작업으로 취급됨. 

하나의 요청을 처리하는 중에 예외가 발생하면 해당 작업만 중단시키면 그만임.

독립형 애플리케이션과 달리 서버의 특정 계층에서 예외가 발생했을 때 작업을 일시 중지하고, 사용자와 바로 커뮤니케이션하면서 예외상황을 복구할 수 있는 방법이 없음. 

✅ 애플리케이션 차원에서 예외상황을 미리 파악하고, 예외가 발생하지 않도록 차단하는 게 좋음.
> 프로그램의 오류나 외부 환경으로 인해 예외가 발생하는 경우라면 빨리 해당 요청의 작업을 취소하고 서버 관리자나 개발자에게 통보해주는 편이 나음. 자바의 환경이 서버로 이동하면서 체크 예외의 활용도와 가치는 점점 떨어지고 있음. 대응이 불가능한 체크 예외라면, 빨리 런타임 예외로 전환해서 던지는 게 나음.

✅ SQLException은 대부분 복구 불가능한 예외이므로, 런타임 예외로 포장해야 함.

<br>

### Ex) `add()` 메소드의 예외처리 - 런타임 예외를 일반화해서 사용하는 방법
#### 아이디 중복 시 사용되는 예외(DuplicateUserIdException)
```java
public class DuplicateUserIdException extends RuntimeException {
	public DuplicateUserIdException(Throwable cause) {
		super(cause);
	}
}
```
>1. 사용자 아이디가 중복됐을 때 사용하는 `DuplicateUserIdException` 만듦.
>
>2. 신경 쓰지 않아도 되도록 `RuntimeException`을 상속한 런타임 예외로 만듦.
>3. 중첩 예외를 만들 수 있도록 생성자 추가

#### 예외처리 전략을 적용한 `add()`
```java
public void add() throws DuplicateUserIdException {
	try{
		// JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
		// 그런 기능이 있는 다른 SQLException을 던지는 메소드를 호출하는 코드
	}
	catch (SQLException e){
		if(e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
			throw new DuplicateUserIdException(e); // 예외 전환
		else
			throw new RuntimeException(e); // 예외 포장
	}
}
```
> `add()` 메소드를 사용하는 오브젝트는 `SQLException`을 처리하기 위해 불필요한 `throws` 선언을 할 필요는 없으면서, 필요한 경우 아이디 중복 상황을 처리하기 위해 `DuplicatedUserIdException`을 이용할 수 있음. 

<br>

### ✅ 애플리케이션 예외
- 시스템 또는 외부의 예외상황이 원인이 아니라 애플리케이션 자체의 로직에 의해 의도적으로 발생시키고, 반드시 `catch`해서 무엇인가 조치를 취하도록 요구하는 예외들을 '**애플리케이션 예외**'라고 함.

#### 사용자가 요청한 금액을 은행 계좌에서 출금하는 기능 있는 메소드 설계 방법 두 가지
1. 정상적인 출금처리를 했을 경우와 잔고 부족이 발생했을 경우에 각각 다른 종류의 리턴 값을 돌려주는 것.

	[단점]
	- 개발자들 사이의 의사소통 문제로 인해 제대로 작동하지 않을 수 있음.

	- 결과값 확인하는 조건문이 자주 등장

2. **정상적인 흐름을 따르는 코드는 그대로 두고, 잔고 부족과 같은 예외 상황에서는 비즈니스적인 의미를 띤 예외를 던지도록 만드는 것.**  -> 이 방법을 추천함!
	- 이때 사용하는 예외는 의도적으로 체크 예외로 만듦.

	[장점]
	- 예외 처리에 대한 처리는 catch 블록에 모아둘 수 있기 때문에 코드 이해하기 편함.

	- if문 남발하지 않아도 됨.

#### 애플리케이션 예외를 사용한 코드
```java
try{
	BigDecimal balance = account.withdraw(amount);
	...
	// 정상적인 처리 결과를 출력하도록 진행
}
catch(InsufficientBalanceException e){ // 체크 예외
	// InsufficientBalanceException에 담긴 인출 가능한 잔고금액 정보를 가져옴
	BigDecimal availFunds = e.getAvailFunds();
	...
	// 잔고 부족 안내 메시지를 준비하고 이를 출력하도록 진행
}
```
> 예금을 인출해서 처리하는 코드를 정상 흐름으로 만들어두고, 잔고 부족을 애플리케이션 예외로 만들어 처리하도록 만든 코드

<br>

### SQLException은 어떻게 됐나?
지금까지 다룬 예외처리 내용은 JdbcTemplate을 적용하는 중에 `throws SQLException` 선언이 왜 사라졌는가를 설명하는 데 필요한 것이었음.

스프링의 예외처리 전략과 원칙을 알고 있어야 하기 때문.

#### DAO에 존재하는 `SQLException`에 대해 생각해보자.
- `SQLException`은 과연 복구가 가능한 예외인가? 

	- 99%는 코드 레벨에서 복구할 방법 없음

	- 시스템의 예외라면 당연히 애플리케이션 레벨에서 복구할 방법 없음.
	- 즉, **대부분의 `SQLException`은 복구가 불가능**함.
	- 따라서, 예외처리 전략 적용 : 필요도 없는 기계적인 `throws` 선언이 등장하도록 방치하지 말고 가능한 빨리 **언체크/런타임 예외로 전환해줘야 함.**
	- 스프링의 JdbcTemplate은 바로 이 예외처리 전략을 따르고 있음. 
	- JdbcTemplate 템플릿과 콜백 안에서 발생하는 모든 SQLException을 런타임 예외인 DataAccessException으로 포장해서 던져줌. -> JdbcTemplate을 사용하는 UserDao 메소드에서는 꼭 필요한 경우에만 런타임 예외인 DataAccessException을 잡아서 처리하면 되고 그 외의 경우엔 무시해도 되기 때문에 DAO 메소드에서 SQLException이 모두 사라진 것임.

**스프링의 API 메소드에 정의되어 있는 대부분의 예외**는 **런타임 예외**.

발생 가능한 예외가 있다고 하더라도 이를 **처리하도록 강제하지 x**

<br>

## 4.2 예외 전환
### ✅ 예외 전환의 두 가지 목적
1. 로우레벨의 예외를 좀 더 의미있고 추상화된 예외로 바꿔서 던져주는 것.

2. 런타임 예외로 포장해서 굳이 필요하지 않은 `catch/throws`를 줄여주는 것.

### ✅ JDBC의 한계
- DB 종류에 상관없이 사용할 수 있는 데이터 액세스 코드를 작성하는 일은 쉽지 않음.

- 표준화된 JDBC API가 DB 프로그램 개발 방법을 학습하는 부담은 확실히 줄여주지만 DB를 자유롭게 변경해서 사용할 수 있는 유연한 코드를 보장해주지는 못함.

<br>

### 현실적으로 DB를 자유롭게 바꾸어 사용할 수 있는 DB 프로그램을 작성하는 데 있는 두 가지 걸림돌
#### 💥 첫 번째 문제 : 비표준 SQL
- DAO를 DB별로 만들어 사용하거나 SQL을 외부에서 독립시켜서 바꿔 쓸 수 있게 해야 함 -> 7장에서 시도
#### 💥 두 번째 문제 : 호환성 없는 `SQLException`의 DB 에러정보
- 호환성 없는 에러 코드와 표준을 잘 따르지 않는 상태 코드를 가진 SQLException만으로 DB에 독립적인 유연한 코드를 작성하는 건 불가능에 가까움.

<br>

### ✅ DB 에러 코드 매핑을 통한 전환
DB 종류가 바뀌더라도 DAO를 수정하지 않으려면 위 두 가지 문제 해결해야 함.

SQL과 관련된 부분은 뒤에서 다루도록 하고, 

여기서는 **두 번째 문제인 `SQLException`의 비표준 에러 코드와 SQL 상태정보에 대한 해결책**에 대해 알아보자.

#### ✅ 해결 방법
DB별 에러 코드를 참고해 발생한 예외의 원인이 무엇인지 해석해주는 기능을 만들기
> ex. 키 값이 중복돼서 중복 오류 발생하는 경우, 
>
>MySQL : 1062, 오라클 : 1, DB2 : -803 이라는 에러 코드 받음.
>
>이런 에러 코드 값을 확인할 수 있다면, 키 중복 때문에 발생하는 `SQLException`을 `DuplicateKeyException`이라는 의미가 분명히 드러나는 예외로 전환할 수 있음.
>
>DB 종류에 상관없이 동일한 상황에서 일관된 예외를 전달받을 수 있다면 효과적인 대응 가능

<br>

✅ **스프링은 `DataAccessException`을 통해 DB에 독립적으로 적용 가능한 추상화된 런타임 예외 계층을 제공함.**
- 문제상황 : DB마다 에러 코드 제각각

	- 스프링은 DB별 에러 코드 분류해 스프링이 정의한 예외 클래스와 매핑해놓은 에러 코드 매핑정보 테이블을 만들어두고 이용함.
- JdbcTemplate은 `SQLException`을 단지 런타임 예외인 `DataAccessException`으로 포장하는 것이 아니라 DB의 에러 코드를 `DataAccessException` 계층구조의 클래스 중 하나로 매핑해줌. 
- 전환되는 JdbcTemplate에서 던지는 예외는 모두 `DataAccessException`의 서브클래스 타입임.
- DB의 종류와 상관없이 중복 키로 인해 발생하는 에러는 `DataAccessException`의 서브클래스인 `DuplicateKeyException`으로 매핑돼서 던져짐.
#### JdbcTemplate이 제공하는 예외 전환 기능을 이용하는 `add()` 메소드
```java
public void add() throws DuplicateKeyException{
	// JdbcTemplate을 이용해 User를 add 하는 코드
}
```
> DuplicateKeyException은 DB를 변경하더라도 동일한 예외가 던져지는 것이 보장됨. 
> 
> JdbcTemplate 안에서 DB별로 미리 준비된 에러 코드와 비교해서 적절한 예외를 던져주기 때문

✅ JdbcTemplate을 이용한다면, JDBC에서 발생하는 DB 관련 예외는 거의 신경 쓰지 않아도 됨.

<br>

## DAO 인터페이스와 DataAccessException 계층 구조
#### `DataAccessException` 용도 : 
- JDBC의 `SQLException`을 전환
- JDBC 외의 자바 데이터 액세스 기술(JDO, JPA, ORM, iBatis 등)에서 발생하는 예외에 적용

	- 의미가 같은 예외라면 데이터 액세스 기술 종류 상관없이 일관된 예외 발생하도록 만듦.

	- 즉, 데이터 액세스 기술에 독립적인 추상화된 예외 제공

🧐 스프링이 왜 이렇게 `DataAccessException` 계층 구조를 이용해 기술에 독립적인 예외를 정의하고 사용하게 할까?

<br>

### ✅ DAO 인터페이스와 구현의 분리
#### DAO를 굳이 따로 만들어서 사용하는 이유 
- 데이터 액세스 로직을 담은 코드를 성격이 다른 코드에서 분리해놓기 위해서

- 분리된 DAO는 전략 패턴을 적용해 구현 방법을 변경해서 사용할 수 있게 만들기 위해서

<br>

### ✅ 낙관적인 락킹(optimistic locking) 
- JDO, JPA, 하이버네이트처럼 오브젝트/엔티티 단위로 정보를 업데이트 하는 경우 발생

- 같은 정보를 두 명 이상의 사용자가 동시에 조회하고 순차적으로 업데이트를 할 때, 뒤늦게 업데이트한 것이 먼저 업데이트한 것을 덮어쓰지 않도록 막아주는 데 쓸 수 있는 편리한 기능.

스프링은 자바의 다양한 데이터 액세스 기술을 사용할 때 발생하는 예외들을 추상화해서 `DataAccessException` 계층 구조 안에 정리해놓음.

#### Jdbc를 이용한 낙관적인 락킹 예외 클래스의 적용 - 그림 p.307
- `DataAccessException` 
	- `OptimisticLockingFailureException`
		- `ObjectOptimisticLockingFailureException`
			- `HibernateOptimisticLockingFailureException`
			- `JdoOptimisticLockingFailureException`
		- **`JdbcOptimisticLockingFailureException`**

> 기술에 상관없이 낙관적인 락킹이 발생했을 때 일관된 방식으로 예외처리를 해주려면 `OptimisticLockingFailureException`을 잡도록 만들면 됨.
>
> 더 자세하게 `DataAccessException` 계층구조 보고 싶다면, p.308 그림 확인
>
>스프링의 데이터 액세스 전략이나 DataAccessException의 예외 사용법은 11장에서 더 자세히 살펴볼 예정

<br>

#### ✅ 인터페이스 사용, 런타임 예외 전환과 함께 DataAccessException 예외 추상화를 적용하면 데이터 액세스 기술과 구현 방법에 독립적인 이상적인 DAO를 만들 수 있음.
<br>

## ✅ 기술에 독립적인 UserDao 만들기
UserDao : 사용자 처리 DAO
UserDaoJdbc : JDBC를 이용해 구현한 클래스 이름
#### Feat: UserDao 인터페이스
```java
public interface UserDao{
	void add(User user);
	User get(String id);
	List<User> getAll();
	void deleteAll();
	int getCount();
}
```
> public 접근자를 가진 메소드이긴 하지만 UserDao의 setDataSource() 메소드는 인터페이스에 추가하면 안 됨(UserDao 구현방법에 따라 변경될 수 있는 메소드이고, 클라이언트가 알고 있을 필요가 없기 때문)


#### Fix: 기존의 UserDao 클래스 이름 UserDaoJdbc로 변경, UserDao 인터페이스를 구현하도록 선언
```java
public class UserDaoJdbc implements UserDao{
```

#### Fix: 스프링 설정파일의 userDao 빈 클래스 이름 변경
```java
<bean id="userDao" class="springbook.dao.UserDaoJdbc">
	<property name="dataSource" ref="dataSource" />
</bean>
```

#### 테스트 코드에서는 UserDao를 UserDaoJdbc로 변경할 필요 없음 
```java
public class UserDaoTest{
	@Autowired // 스프링의 컨텍스트 내에서 정의된 빈 중에서 인스턴스 변수에 주입 가능한 타입의 빈을 찾아줌
	private UserDao dao; 
}
```

#### Feat: `DataAccessException`에 대한 테스트
```java
@Test(expected=DataAccessException.class)
public void duplicateKey(){
	dao.deleteAll();
	
	dao.add(user1);
	dao.add(user1); // 강제로 같은 사용자를 두 번 등록 -> 여기서 예외 발생
}
```
> `expected=DataAccessException.class` 부분 빼고 테스트 실행하면, 에러 메시지 통해 어떤 예외 클래스가 던져졌는지 확인 가능

`DataAccessException`이 기술에 상관없이 어느 정도 추상화된 공통 예외로 변환해주긴 하지만 근본적인 한계 때문에 완벽하다고 기대할 순 x

<br>

### ✅ 스프링은 `SQLException`을 `DataAccessException`으로 전환하는 다양한 방법 제공
- 가장 보편적 + 효과적 방법 : DB 에러 코드 이용하는 방법

- `SQLException`을 코드에서 직접 전환하고 싶다면, `SQLExceptionTranslator` 인터페이스를 구현한 클래스 중에서 `SQLErrorCodeSQLExceptionTranslator`를 사용하면 됨.

#### Fix: `DataSource` 빈을 주입받도록 만든 UserDaoTest
```java
public class UserDaoTest{
	@Autowired UserDao dao;
	@Autowired DataSource dataSource;
}
```

#### Feat: `SQLException` 전환 기능의 학습 테스트
```java
@Test
public void sqlExceptionTranslate(){
	dao.deleteAll();

	try{
		dao.add(user1);
		dao.add(user1);
	}
	catch(DuplicateKeyException ex){
		SQLException sqlEx = (SQLException)ex.getRootCause();
		// 코드를 이용한 SQLException의 전환
		SQLExceptionTranslator set = new SQLErrorCodeSQLExceptionTranslator(this.dataSource);
		
		// 에러 메시지 만들 때 사용하는 정보이므로 null로 넣어도 됨.
		assertThat(set.translate(null, null, sqlEx), is(DuplicateKeyException.class));
	}
}
```
> `assertThat()`의 **`is()`** 메소드에 클래스를 넣으면 오브젝트의 `equals()` 비교 대신 **주어진 클래스의 인스턴스인지 검사**해줌.
> 
> 이 테스트가 성공한다면 코드에 의한 예외 전환이 잘 됐음을 확인할 수 있음.
