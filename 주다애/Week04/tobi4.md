## 4. 예외

> 코드 작성시, 적절한 상황에서 적절한 예외를 처리해주는 것은 아주 중요하다.

### 1. 사라진 SQLException

> 예외가 발생했을 때 catch블록으로 잡기만하고 아무 조치를 취하지 않는 것은 예외처리를 하지 않는 것보다 훨씬 나쁜 일이다.
> 예외는 처리되어야 하지 단순 출력 되는 것이 아니다.
> 모든 예외는 복구 or 통보 되어야 한다.

    public void method1() throws Exception{
	    method2();
    }

위의 코드가 문제인 이유는?
무책임하게 throws Exception을 날리는 것은 적절한 처리를 통해 복구 가능한 예외상황도 제대로 다룰 수 없게 해버린다.

#### **예외의 종류**에는
1. Error
* java.lang.Error 클래스의 서브클래스로 시스템에 비정상적 상황이 발생했을 때 사용된다.
* 애플리케이션 레벨에서는 딱히 신경 쓸 필요가 없다.
2.  Exception과 **체크예외**
* java.lang.Exception 클래스의 서브클래스로 코드의 작업 중 예외가 발생했을 때 사용된다.
* Exception은 체크예외와 언체크예외로 구분된다.
* 일반적인 예외는 체크예외로, 반드시 예외를 처리하는 코드를 함께 작성해주어야 한다.(catch or throws)
3. RuntimeException과 **언체크**/**런타임 예외**
* java.lang.RuntimeException 클래스의 서브클래스로 오류가 있을 때 발생되도록 의도된 예외상황이다.
* 코드에서 미리 조건을 체크하도록 만들면 피할 수 있다.
* catch문으로 잡거나 throws로 선언하지 않아도 된다.

#### **예외처리** 방법에는
1. 예외복구

> 예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것

* 사용자에게 상황을 알려주고 다른 작업 흐름으로 안내해서 예외상황을 해결할 수 있다.
* 체크예외는 이렇게 예외를 어떤 식으로든 복구할 가능성이 있는 경우에 사용한다.
2. 예외처리 회피

> 예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것

    public void add() throws SQLException{
	    // 처리
    }

* 콜백 오브젝트에는 모두 throw SQLException이 붙어서 템플릿에게 예외를 던져버린다.
* 예외를 던지는 쪽과 받는 쪽이 콜백/템플릿 처럼 긴밀한 관계일 때 사용해라

3. 예외 전환

> 예외를 적절한 예외로 전환해서 메소드 밖으로 던지는 것

첫번째는 내부에서 예외를 던질 때 의미를 분명하게 해줄 수 있는 예외로 바꾸려고 사용한다.

    public void add(User user) throws DuplicateUserIdException, SQLException{
	    try{
	    }
	    catch(SQLException e){
		    if(e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY){
			    throw DuplicateUserIdException();
		    }
	    }
    }

--> 전환하는 예외(DuplicateUserIdException)에 원래 예외(SQLException)을 담다서 중첩 예외로 선언한다.

두번째는 체크예외를 런타임 예외로 바꾸는 것이다.
--> 일반적으로 체크 예외를 계속 throws로 넘기는 것은 무의미하다. 메소드 선언만 지저분해지고 장점이 없다!
--> 어차피 복구 불가한 예외면 빨리 런타임 예외로 포장해 던지게 하는 것이 좋다.

#### 예외처리 전략

> 체크 예외 대신 언체크 예외로 정의하는 것이 일반화되고 있다.

* 체크 예외를 강제하는 것은 예외를 제대로 다루고 싶지 않을 만큼 개발자에게 짜증나는 일이다.
* 대응 불가한 체크 예외는 그냥 런타임 예외로 전환해서 던져라!

#### add() 메소드의 예외처리
현재는 두 가지의 체크 예외를 던지게 되어있는데, 런타임 예외로 바꿔서 던져주자. (런타임 예외도 throws로 던져줄 수 있다.)

    public class DuplicateUserIdException extends RuntimeException{
	    public DuplicateUserIdException(Throwabld causer){
		    super(caue);
	    }
    }

DuplicateUserIdException 클래스를 만든다. 중첩 예외를 만들어야 하므로 생성자 코드를 항상 넣어주어야 한다.

    public void add() throws DuplicateUserIdException {
	    try{
	    }
	    catch(SQLException e){
		    if(e.getErrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
			    throw new DuplicateUserIdException(e); // 예외전환
	    }
    }
* SQLException을 던져줄 필요가 없으며, DuplicateUserIdException를 런타임 예외로 전환하여 던져준다.
* 런타임 예외는 신경 써서 처리해주는 것이 중요하다.
* 런타임 예외는 낙관적인 예외라고 할 수 있다. 

#### 애플리케이션 예외

> 애플리케이션 자체의 로직에 의해 의도적으로 발생시키고, 반드시 catch해서 무엇인가 조치를 취하도록 요구하는 예외

1. 정상적 상황과 예외 상황에서 각각 다른 리턴 값을 돌려준다. 
* 예외 상황에서 0이나 -1을 리턴해준다.
* 하지만! 리턴 값을 명확하게 코드화하고 관리하지 않으면 혼란이 온다. 
* 예외에 대한 리턴 값의 표준이 없다는 것을 꼭 알아야한다.
* 또한 if/else문의 중첩으로 코드가 지저분해진다.

2. 정상적 코드는 그대로 두고, 예외 상황을 비즈니스적 의미를 띤 예외를 던지도록 만든다.
* 정상적 흐름을 따르지만 예외가 발생할 수 있는 코드는 try안에 넣어두고, 예외를 catch에서 처리해줄 수 있다.
* 이때 의도적으로 **체크 예외**로 만든다. 그래서 잊지 않고 **예외상황에 대한 로직**을 꼭 구현해주어야 한다. (throws Exception을 해주는게 무책임하긴 해도 런타임 예외로 만드는 것 보단 안전하다.)

#### SQLException은 어떻게 됐나?
1. 일단, SQLException은 복구가 가능한가?
---> 99%로는 코드 레벨에선 **복구 불가하다.**
* 시스템 예외는 당연히 애플리케이션 레벨에서 복구 불가하다.
2. 따라서 예외처리 전략을 적용해야한다. 가능한 빨리 언체크/런타임 예외로 전환을 해주자
3. **스프링의 JdbcTemplate은 위의 예외처리 전략을 따르고 있다.**
---> 모든 SQLException을 DataAccessException으로 포장해서 던져준다.
---> 즉, UserDao메소드에서 꼭 필요한 경우에만 런타임 예외인 DataAccessException을 잡아서 처리해주면 된다.
4. 스프링의 API메소드에 정의되어 있는 예외는 대부분 런타임 예외이다.
 즉, 발생 가능 예외가 있다고 하더라도 처리하도록 강제하지 않는다.


### 2. 예외 전환

> 예외 전환은 런타임 예외로 포장하여 의미없는 catch/throws를 줄이거나 로우레벨의 예외를 좀 더 의미있고 추상화된 예외로 바꿔서 던지기 위해 사용한다.

**DataAccessException은 일단 런타임 예외로 SQLException을 포장한다.**

#### JDBC의 한계

> 먼저, JDBC는 자바를 이용해 디비에 접근하는 방법을 추상화된 API로 정의하고 표준 인터페이스를 통해 기능을 제공해 객체지향 프로그래밍의 장점을 살려준다.

그러나, DB를 자유롭게 변경해서 사용할 수 있는 유연한 코드를 보장해주지는 못한다는 한계가 있다.

1. 비표준 SQL
대부분의 DB는 표준을 따르지않는 문법과 기능을 제공한다.
* 비표준 SQL은 결국 dao코드에 들어가고, 해당 dao는 특정 DB에 종속적 코드가 되어버린다.
2. SQLException하나에 DB마다 나타나는 다양한 예외의 종류와 원인을 담아버린다.
JDBC API는 이 SQLException 한 가지만 던지도록 되어있어서 다양한 DB에 대한 대응이 편하지 않다.
* getSQLState()메소드로 상태정보를 가져올 수 있지만, JDBC 드라이버에서 상태 코드를 정확하게 만들어주지 않는다.

#### DB 에러 코드 매핑을 통한 전환

> DB가 바뀌더라도 dao를 수정하지 않게 하기 위해 SQLException의 비표준 에러 코드와 상태정보를 해결해야 한다.

1. DB별 에러 코드를 참고해서 발생한 예외이 원인이 무엇인지 해석하는 기능을 만들자
스프링은 DataAccessException라는 런타임 예외를 정의하고 있다.
그리고 그의 서브클래스들도 정의하고 있다.

2. 에러코드가 제각각인것은 에러 코드 매핑정보 테이블을 만들어서 해결하자

코드는 다음과 같이 바뀐다.

    public void add() throws DuplicateKeyException{
    }

JbdcTemplate를 이용하면 JDBC에서 발생하는 DB관련 예외는 거의 신경 쓰지 않아도 된다.

하지만, SQLException의 서브클래스이므로 여전히 체크 예외 + 예외 세분화 기준이 SQL 상태정보를 이용한다는 문제점이 있다.

#### DAO 인터페이스와 DataAccessException 계층구조

> 자바에는 JDBC 이외에도 다양한 데이터 엑세스 표준 기술이 존재하는데, DataAccessException은 의미가 같은 예외라면 데이터 엑세스 기술의 종류와 상관없이 일관된 예외가 발생하도록 만들어준다.
> 즉, 데이터 엑세스 기술에 독립적인 추상화된 예외를 제공한다.

**DAO 인터페이스와 구현의 분리**
왜 dao를 따로 빼서 코드를 만드는가?
데이터 엑세스 로직을 담은 코드를 성격이 다른 코드와 분리하려고! (전략패턴과 의존성 주입 사용)

하지만 메소드 선언에 나오는 예외정보는 문제가 될 수 있다.

    public interface UserDao{
	    public void add(User user);
    }
하지만, 예외를 던지기 때문에 메소드 선언은 사용할 수 없다.
* public void add(User user) throws SQLException으로 던져주어야한다.
하지만 데이터 엑세스 기술의 차이로 인해서 SQLException로 선언한 인터페이스는 다른 디비에서 사용 불가하다.
* public void add(User user) throws Exception;으로 던져주어야 하는데 이는 무책임한 예외선언이다.
* 또한 데이터 엑세스 기술이 달라지면 같은 상황에서도 다른 종류의 예외가 던져진다는 문제가 있다.
* 결국 클라이언트는 dao의 기술에 의존적이게 된다.

#### 데이터 엑세스 예외 추상화와 DataAccessException 계층구조

> 스프링의 DataAccessException은 일부 기술에서만 공통적으로 발생하는 예외를 포함해서 데이터 엑세스 기술에서 발생 가능한 대부분의 예외를 계층구조로 분류해놓았다.


#### 기술에 독립적인 UserDao 만들기
#### 1. 인터페이스 적용
UserDao클래스를 인터페이스와 구현으로 분리하자.

    public interface UserDao{
	    void add(User user);
	    User get(String id);
	    ...
    }

    public class UserDaoJdbc implements UserDao{
	    // interface의 메소드 구현
    }

#### 2. 테스트 보완
테스트 코드의 인스턴스 변수는 그냥 인터페잇  타입으로 해주어도 된다.
dao의 기능에 집중하면 인터페이스를, 특정 기술을 사용한 dao의 구현에 관심이 있으면 클래스를 인스턴스 변수로 선언해주면 된다.
![예외 처리 전략과 기술에 독립적인 Dao 만들기](https://velog.velcdn.com/images/jeong_hun_hui/post/2624920b-7a95-4c31-b954-d15b64634e24/image.png)

테스트 코드를 예를 들면

    @Test(expected=DataAccessException.class)
    public vlid duplicateKey(){
	    dao.deleteAll();
	    dao.add(user1);
	    dao.add(user1);
    }
이렇게 되면 DataAccessException의 예외 중 하나가 던져지면서 테스트가 성공한다.
어떤 예외가 발생했는지 구체적으로 알고 싶으면 테스트를 실패시키자.(expected=DataAccessException.class 이 부분을 빼고 테스트를 하자)

#### DataAccessException 활용 시 주의사항

> DataAccessException이 기술에 상관없이 어느 정도 추상화된 공통 예외로 변환해주긴 하지만 근본적 한계가 존재해서 완벽하다고 할 수 없다.

DuplicateKeyException은 아직까지 JDBC를 이용하는 경우에만 발생한다.
SQLException을 직접 해석해서 DataAccessException으로 변환하는 방법도 있다. 

### 3. 정리
1. 예외를 잡아서 처리를 안하거나 throws를 남발하는 것은 매우 좋지 않다.
2. 예외는 복구하거나 적절한 예외로 전환해야한다.
3. 복구 불가한 예외는 빠르게 런타임 예외로 변환해라
4. 애플리케이션레벨의 예외는 체크 예외로 만들어라
5. SQLException의 에러 코드는 DB에 종속되기 때문에 DB에 독립적인 예외로 전환해야 한다.
6. 스프링의 DataAccessException은 DB에 독립적으로 적용 가능한 추상화된 런타임 예외 계층을 제공한다.
