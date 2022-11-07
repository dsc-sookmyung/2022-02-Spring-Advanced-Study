# 4장 예외

4.1 사라진 SQLException

JdbcTemplate으로 바꾼 후 deleteAll() 메소드 : thorws SQLException 선언 사라짐

왜 SQLException이 사라졌을까?

## 초난감 예외처리

### 예외 블랙홀

: 예외를 처리하지 않고 넘어가는 경우

- 예외 무시하고 계속 진행
    
     상황 : catch로 잡은 후 아무것도 하지 않고 넘어감
    
    문제 : 예외가 발생했는데 무시하고 계속 진행 → 최종적 오작동시 발견 힘듬
    
    ```java
    try{
    		...
    }
    catch(SQLException e){
    }
    ```
    
- 예외 잡고 화면에 로그 띄우기
    
    ```java
    }
    catch(SQLException e){
    	System.out.println(e);
    }
    ```
    
    상황 : 예외 발생시 화면에 출력해줌
    
    문제 : 로그와 메세지 금방 묻힘, 예외가 처리되지 않음
    
    콘솔, 로그에 예외 출력하는 것은 도움이 되지 않음. 처리를 해야됨
    

예외 처리시 지켜야 할 핵심 원칙

1. 모든 예외 적절하게 복구
2. 작업 중단시키고 운영자, 개발자에게 통보

### 무의미하고 무책임한 thorws

: 기계적으로 모든 메소드에 throw하는 방법

상황 : 모든 예외를 무조건 던져버리는 선언을 모든 메소드에 넣음

문제 : 정말 실행중 예외가 발생한건지 습관적으로 붙여놓은 건지 알 수 없

## 예외의 종류와 특징

예외 how 다룰까? 큰 이슈 : 체크예외 사용하고 다루는 방법

throw 통해 발생 시킬 수 있는 에러

### 1. Error

java.lang.Error의 서브클래스

- 에러 : 시스템에 비정상적 상황 발생 → 자바 VM에서 발생 시킴
- catch 블록으로는 대응할 수 없음
- 애플리케이션에서는 신경쓰지 않아도 됨

### 2. Exception과 체크 예외

`체크예외`

java.lang.Exception과 서브클래스

- 애플리케이션 코드의 작업중에 예외상황 발생
- Exception의 서브 클래스 , RuntimeException 클래스를 상속 X
- 예외 처리 코드 강제함
- 예외 상황에서 던져질 가능성 있는 것들 대부분 체크 예외

<aside>
👩🏻‍💻 반드시 예외를 처리하는 코드를 함께 작성

체크 예외 던짐 → catch로 잡기 or 다시 throws 정의, 메소드 밖으로 던지기

</aside>

### 3. RuntimeException과 언체크/런타임 예외

`언체크 예외=런타임예외`

java.lang.RuntimeException 클래스 상속 

- 개발자가 부주의해서 발생하는 경우 만드는 예외
- 명시적인 예외처리 강제하지 않음
- 프로그램 오류가 있을 때 발생
    - NullPointException : 오브젝트 할당하지 않은 래퍼런스 변수 사용
    - IllegalArgumentException : 허용되지 않는 값 사용해서 메소드 호출

<aside>
👩🏻‍💻 catch문 잡거나 throws로 선언하지 않아도 됨

</aside>

## 예외처리 방법

일반적인 예외처리 방법 알아보고 나서 효과적 예외처리 전략 알아보기

### 1. 예외 복구

: 예외 상황 파악하고 문제 해결한 후 정상 상태로 돌려놓는 것

- ex ) IOException 발생
    
    문제 : 요청한 파일 읽을 수 없음
    
    해결 : 사용자에게 상황 알려주고 다른 파일 이용하도록 안내
    
- 기본 작업 흐름 불가능 시 자연스럽게 다른 작업 흐름으로 유도
    - 에러 메세지를 그냥 사용자에게 던지는것 X
    - 애플리케이션에서는 정상적으로 설계된 흐름 따라야됨
    
- ex) SQLException 발생
    
    문제 : 원격 DB 서버 접속에 실패
    
    해결 : 일정 시간 대기 후 다시 접속 
    
    정해진 횟수만큼 재시도 해서 실패했으며 예외 복구 포기
    
- 체크 예외 : 예외를 어떤식으로든 복구할 가능성 있는 경우에 사용

### 2. 예외처리 회피

: 예외처리를 자신이 담당하지 않고 자신을 호출한 곳으로 던져버리는 것

예외를 자신이 처리하지 않고 회피하는 것

- throws 선언해서 알아서 예외 던져지게
    
    ```java
    public void add() throws SQLException{
      //JDBC API
    }
    ```
    
- catch 문으로 잡아 로그 남긴 후 다시 throws
    
    ```java
    public void add() throws SQLExceiption{
    	try{
    			//JDBC API
    	}
      catch(SQLException e){
    			//로그 출력
    			throw e;
    	}
    }
    ```
    

- ex   콜백 오브젝트 (JdbcContext, JdbcTemplate )
    - 작업하다 발생하는 SQLException 템플릿으로 던짐
    - 콜백 오브젝트의 메소드 모두 throws SQLException 붙어있음
    - 예외 회피, 템플릿 레벨에서 처리하도록

- ex  DAO가 던진다면
    - DAO 사용하는 서비스 계층, 웹컨트롤러에서 해결 x
    - 예외가 서버로 전달됨

> 정리
> 

예외회피는 의도가 분명해야됨

- 콜백/템프릿 : 긴밀한 관계에 있는 오브젝트에게 예외처리 책임 지게
- 자신을 사용하는 쪽에서 예외를 다루는게 최선이라는 확신있어야

### 3. 예외 전환

: 예외를 적절한 예외로 전환하여 메소드 밖으로 던지는 것

→ 예외를 정상적인 사태로 만들 수 없어서

**예외 전환의 목적**

명확한 의미를 만들기 위해

- 내부에서 발생한 예외를 그대로 던지는 것 →적절한 의미 부여하지 못하는 경우
    
    Ex. 사용자 등록시 같은 아이디 사용자가 있는 경우
    
    - 상황 : DB에러 발생 → SQLException 예외 생김
    - 문제 : DAO 메소드가 그대로 던지면  서비스 계층은 왜 예외 발생한지 모름
    - 해결 : DAO에서 예외 정보 해석후 DuplicateUserIdException 같은 예외로 바꿔서 던져줌
    
    <aside>
    👩🏻‍💻 의미가 분명한 예외로 던질것 !
    ⇒ 서비스 계층 obj에서 적절한 복구 작업 시도할 수 있음
    
    </aside>
    
    예외 전환 방법
    
      **중첩예외로 만들기**
    
    - 전환하는 예외에 원래 발생한 예외 담기
    - getCause() 메소드 : 처음 발생한 예외무엇인지 확인 가능
    - 새로운 예외 만들면서 생성자, initCause() 로 근본 원인이 되는 예외 넣기
    
    ```java
    //생성자로 근본 예외 넣기
    catch(SQLException e){
    	... throw DupilicateUserIdException(e);
    }
    ```
    
    ```java
    //initCause로 근본 예외 넣기
    catch(SQLException e){
    	... throw DuplicateUserIdException().initCause(e);
    }
    ```
    
- 예외 처리하기 쉽고 단순하게 만들기 위해
    
    **예외 포장하기**
    
    - 체크예외를 런타임 예외로 바꾸는 경우 사용
    - 의미를 명확하게 만들기 위해
    
    Ex .  EJBException : EJB 컴포넌트에서 예외 발생
    
    ```java
    try {
    OrderHome orderHome = EJBHomeFactory.getInstance ().getOrderHome () ;
    Order order = orderHome. findByPrimaryKey (Integer id);
    } catch (NamingException ne) {
    		throw new EJBException(ne);
    } catch (SQLException se) {
    		throw new EJBException(se);
    } catch (RemoteException re) {
    		throw new EJBException(re);}
    ```
    
    - 문제 : 발생하는 체크예외 대부분 복구가능 x, 의미있는 예외 x
    - 해결 : EJBException = 런타임 예외 로 포장해서 던지는 편이 나음
    - 효과 : 트랜잭션 자동 롤백, 클라이언트에서 일일이 예외 잡아 다시 던지지 x
        
        ⇒ 잡아도 복구할 방법 x
        
    
    Ex. 애플리케이션 로직상 예외 발생
    
    - API가 던지는 예외 아님
    - 애플리케이션 코드에서 의도적으로 던지는 예외
    - 체크 예외 사용 → 대응, 복구 작업

<aside>
👩🏻‍💻 결론 : 복구 불가능 예외

- **런타임 예외** 포장 던짐
- 예외 처리 서비스 안내 로그 남김
- 관리자 메일 통보
- 사용자 안내 메세지 보여주는 식으로 처리
</aside>

## 예외 처리 전략

예외를 효과적으로 사용하고 깔끔히 코드 정리

### 0. 런타임 예외 보편화

> 체크 예외 : 일반적 예외
> 
> 
> 런타임 예외 : 시스템 장애, 프로그램상 오류
> 

상황 : 자바 엔터프라이즈 서버 환경 

문제 : 서버 계층 예외 발생 시 작업 중단하고 예외사항 복구할 방법 없음

해결 : application 차원에서 예외 상황 미리 파악, 차단하는게 좋음

⇒ 대응 불가능한 체크 예외라면 런타임예외로 전환하는게 낫다! 

### 1. add() 메소드 예외처리

상황 : add() 메소드는 두가지 체크 예외 보냄 (DuplicatedUserIdException, SQLException)

문제 : 전자 - 복구가능, 후자 - 복구 불가 대부분, 애플리케이션 밖으로 던져질 것

해결 : 런타임 예외로 포장해 던져서 다른 메소드들이 신경쓰지 않게 해주자! 

과정 :

1. DuplicatedUserIdException 만들기
    - add() 보다 더 앞단의 오브젝트에서 다룰 수 있음 → 런타임 예외로 만들기
    - add() 메소드에는 예외 던진다고 명시적 선언
    - 중첩 예외 만들 수 있게 생성자 추가
    
    ```java
    public class DuplicateUserIdException extends RuntimeException {
    		public DuplicateUserIdException (Throwable cause) {
    				super(cause);
    		}
    }
    ```
    

1. add() 메소드 수정하기
    - SQLException 런타임 예외로 전환해서 던지도록 하기

결론 : add() 메소드 이용하는 obj는 불필요한 throws 선언할 필요 x, 필요의 경우 아이디 중복 사항 처리 위한 예외 처리 이용 가능

### 2. 애플리케이션 예외

: 애플리케이션 자체 로직에 의해 의도적 예외 발생, 반드시 catch해서 조치 취하게 함

Ex. 요청한 금액을 은행 계좌에서 출금하는 기능 메소드

상황 : 현재 잔고 확인, 허용하는 범위 넘게 요청하면 작업 중단 시킴, 경고 날리기

- 설계 1 : 정상적 출금 처리와 잔고부족 발생 경우 리턴값 다르게
    - 리턴값으로 결과 상태 나타냄
    - 단점 : 개발자 사이의 소통 문제로 제대로 동작하지 않을 수도 있음 / 조건문 많은 지저분한 코드
- 설계 2 : 예외 상황에서만 예외 던지기
    - 잔고 부족시 InsufficientBalancedException 던짐
    - 코드 이해 편함 - try, catch에 따로 정리해둬서
    
    설계 2의 예외 - 체크 에외로 만듬 : 예외로직 구현 강제해줌
    

## SQLException의 행방

- SQLException 복구 가능 ? : 거의 no
- 개발자가 빨리 인식할 수 있도록 발생 예외 전달하는 것이 최선
- 예외 처리 전략 사용 위해 런타임 예외로 전달해야함

JdbcTemplate이 이전략 따름

- 콜백 안에 발생하는 모든 SQLException을 런타임 예외로 포장해 던져줌
- 꼭 필요한 경우에만 UserDao 에서 사용, 아님 무시
- ⇒ 그래서 UserDao에서 사라진 것

# 4.2 예외 전환

예외 전환 목적

1. 런타임 예외로 포장하여 필요하지 않은 catch/throws 줄임
2. 로우레벨 예외 더 의미있고 추상화된 예외로 바꿈

### JDBC의 한계

상황 : DB별로 다른 API 제공해야 할 경우

문제 : DB 변경시 DAO 코드 변경 해야함, 서로 다른 API 사용법 익혀야 됨

DB를 자유롭게 바꾸어 사용할 수 있는 프로그램 작성의 걸림돌

- 비표준 SQL
    
    : DB의 특별 기능이용하거나 최적화 된 SQL 만드는 등에 비표준 SQL 사용됨
    
    문제 : 비표준 SQL → DAO코드로 들어감 → DAO는 DB에 종속적인 코드가 됨
    
    다른 DB로 변경시 DAO에 담긴 SQL 코드 수정해야됨
    
    해결 : Dao를 DB별로 만들어 사용 / SQL을 외부에서 독립시켜 바꿔쓰게 함
    
- 호환성 없는 SQLException의 DB 에러정보
    
    : DB마다 SQL외에도 에러 종류, 원인 제각각
    
    문제 : SQLException의 에러코드가 DB에 종속됨. 서로 호환 안됨
    
    해결 : DB에 독립적인 예외로 전환되어야 함
    

### DB에러 코드 매핑을 통한 전환

문제 : SQL 상태코드가 정확하지 않음

해결 : DB별 에러코드 참고해서 발생한 예외 무엇인지 해석하는 기능 만들기

DB 종류와 상관없이 동일한 상황에서 일관된 예외 받도록

DataAccessException

- SQLException 대체할 만한 런타임 예외 정의
- 다양한 예외 클래스 제공

문제 : DB별 에러코드가 제각각

해결 : 에러코드 정보를 매핑파일에 담아둠

JdbcTemplate

- DB 에러코드를 DataAccessException 계층 구조 클래스 중 하나로 매핑해줌
- 예외는 모두 서브클래스 타입
- 디비가 달라져도 같은 종류의 에러면 동일 예외 받을 수 있음
- JDBC에서 발생하는 DB관련 외에는 거의 신경쓰지 않아도 됨

### DAO 인터페이스와 구현의 분리

DAO 를 분리해서 사용기술과 구현코드 전략 클라이언트에게 숨길 수 있음

문제 : Jdbc가 아닌 기술로 구현하면 사용 할 수 없음

         SQLException 던지도록 선언한 인터페이스 메소드 사용 불가능

해결 : 

1. throws Exceptions 선언 모든 예외 다 받게 
    
    너무 무책임
    
2. JDBc를 이용한 DAO에서  모든 SQLException을 런타임 예외로 포장

문제 → 모든 예외를 무시 할 순 없음 : 중복키 에러

### 데이터 액세스 추상화와 DataAccessException 계층구조

해결 : 엑세스 기술 사용할때 발생하는 예외들을 추상화해서 DataAccessException 계층구조 안에 정리해둠

- 에러코드 DB별로 매핑해서 해당하는 서브클래스 중 하나로 전환 후 던짐
    - 발생가능한 대부분의 예외를 계층구조로 분류해둠
    
    Ex) 엔티티/ 오브젝트 단위로 정보 업데이트 하는 경우 낙관적 락킹 발생 가능
    
    - 같은 정보를 두명 이상 사용자가 순차적으로 업데이트 할때 먼저 업데이트 한것 덮지 않게 막아주는 기능
    - 사용자에게 안내 메세지 보여줘야됨
    - 다른 기술 엑세스들도 서브클래스를 이용해서 예외 전환 가능
    
- 템플릿 메소드나 DAO에서 활용가능한 예외도 있음
    
    Ex .queryForObject() 
    
    - 한개의 로우만 돌려주는 쿼리에 사용
    - 하나 이상 로우 가져올 경우 JDBC에선 에러 나지 않음
    - 해결할 서브클래스 있음

## 기술에 독립적인 DAO 만들기

### 인터페이스 적용

UserDao 클래스를 인터페이스와 구현으로 분리

- UserDao 인터페이스 : DAO의 기능만 넣음
    - setDataSource() 포함 x
- UserDaoJdbc : 구현만 남게
    - implement UserDao 해줌
- 설정파일 빈클래스 이름을 변경해줌

### 테스트 보완

문제 : user 인스턴스 변수 선언도 jdbc 변경?

해결 : @Autowired - 스프링 컨텍스트 내에서 정의된 빈 중 인스턴스 변수에 주입 가능한 타입의 빈 찾아줌

- 구현 기술 상관없이 DAO 기능 체크 : UserDao 인터페이스만 받기
- 구현 내용 관심 갖고 테스트  : 테스트에서 @Autowired로 DI 받을 때 특정 타입 이용

⇒ UserDao 테스트는 DAO의 기능 검증이 목적, 전자가 적합

- 중복키 가진 정보 등록했을 때 테스트
    - test를 클라이언트로 생각
    - 어떤 예외 발생하는지 보기
    - 아이디가 같은 사용자 두번 add() 등록 → 예외 발생할 것 기대
    - 어떤 예외 발생했는지 다시 테스트
- 중복키 어떤 예외 발생했는지 test
    - expected=DataAccessException.class 빼고 테스트 실행
    - 테스트 실패 but 에러메세지 통해 예외 클래스 던져짐
    - 서브클래스의 한종류임을 알 수 있게 됨 → 성공

### DataAccessException 활용시 주의사항

문제 : DuplicatedKeyException이 아직 JDBC를 이용하는 경우에만 발생

DB에러 코드와 달리 예외들이 세분화 되어있지 않음

DataAccessException이 추상화된 공통 예외로 변환 o → 근본적 한계 있음

DataAccessException 잡아 처리하는 코드 말들 경우 미리 학습테스트 만들어서 전환되는 에외 종류 확인해 볼것

- Ex. 학습테스트 만들어 SQLException → DataAccessException으로 변환
    
    SQLException 직접 전환해보기
    
    SQLErrorCodeSQLExceptionTranslator 사용
    
    - 에러 코드 변환에 필요한 DB종류 알아내기 위해 현재 연결된 DataSource를 필요로 함
    - DataSource 변수 추가 → DataSource 타입의 빈 받기
    
    ```java
    public class UserDaoTest{
    	@Autowired UserDao dao;
    	@Autowired Datasource dataSource;
    }
    ```
    
    학습테스트
    
    ```java
    @Test
    public void sqlExceptionTranslate(){
    	dao.deleteAll();
    
    	try{
    		dao.add(user1);
    		dao.add(user2);
    	}
    	catch (DuplicateKeyException ex){
    	  SQLException sqlEx = (SQLException)ex.getRootCause();
    		SQLExceptionTranslator set = 
    			new SQLErrorCodeeSQLExceptionTranslaotr(this.dataSource);
    
    			asserThat(set.translate(null,null,sqlEx), is(DuplicateKeyException.class);
    	}
    }
    ```