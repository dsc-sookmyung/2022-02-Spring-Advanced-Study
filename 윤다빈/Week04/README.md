# 예외

<aside> 💡 스프링의 데이터 엑세스 기능에 담긴 예외처리와 관련 접근방법을 공부할 것

</aside>

## **예외 처리 핵심 원칙**

> 모든 예외는 적절히 복구되거나 작업을 중단한 후 운영자에게 통보되어야 함

**예외 처리에서 피해야 할 습관**

-   **예외**  **블랙홀**
    -   try-catch 문으로 예외를 잡아내지만, 아무 처리가 없거나 화면에 메세지 출력하는 것은 최악 👎🏻
    -   예상치 못한 문제를 일으킬 수 있으며 알아차리고 대응하기 매우 힘들기 때문. 차라리 예외를 잡지마
-   **무의미하고 무책임한 throws**
    -   API등에서 발생하는 예외를 하나씩 catch 하기도 귀찮으며 예외 이름을 적어서 선언하는게 귀찮아서 씀
    -   모든 예외를 모두 던져버리는 선언을 모든 메소드에 넣는 형태로 예외 블랙홀 보단 낫지만 나쁨 👎🏻
    -   메소드 선언에서 기계적으로 넣은 코드인지, 정말로 실행중 예외 상황의 발생 가능성이 있는지 구분 힘듦
    -   따라서 적절한 처리로 복구될 수 있는 예외들을 처리할 수 없게 됨

### **자바에서 throw로 발생시킬수 있는**  **예외 종류 및 특징**

-   Error (노신경)
    -   java.lang.Error 클래스의 서브 클래스로, 시스템에 비정상적 상황이 발생한 경우 사용
    -   보통 어플리케이션에서 잡아도 대응 방법이 없으므로 잡지 않음
-   **Exception과 체크 예외**
    -   java.lang.Exception 클래스와 서브클래스이며 다시 체크예외와 언체크예외로 구분
    -   전자는 RuntimeException을 상속하지 않았으며 후자는 상속했음, 일반적으로 예외 = 체크예외
    -   사용할 메소드가  **체크예외를 던지면 catch문으로 잡거나 throws를 정의해 메소드 밖으로 던져야함**
    -   체크 예외를 다루는게 가장 큰 이슈 → 예외처리 강제가 별로다라는 비판이 있었음
-   **Runtime Exception과 언체크/런타임예외**
    -   java.lang.RuntimeException 클래스를 상속한 예외로, 명시적인 예외처리가 필수가 아님
    -   주로 프로그램의 오류 발생시 일어나는것으로, 피할 수 있지만  **개발자의 부주의**로 발생

### **예외 처리 방법**

-   **예외 복구**
    -   예외 상황 파악 및 문제해결을 통해 정상으로 복구
    -   체크 예외들은 대부분 예외를 어떤식으로든 복구할 수 있는 경우에 사용함
    -   ex) 네트워크가 불안해서 서버 접속이 잘 안되는 경우 일정 시간 대기 후 정해진 횟수만큼 접속 재시도
-   **예외 처리 회피**
    -   예외 처리를 자신이 담당하지 않고, 자신을 호출한 쪽으로 예외를 던짐. 예외를 자신이 처리하지 X
    -   반드시 다른 오브젝트나 메소드가 예외를 대신 처리하도록 예외를 던져줘야 함
    -   콜백과 템플릿처럼  **긴밀하게 역할 분담이 된 관계**가 아니라면, 예외를 던지는건 무책임한 회피
-   **예외 전환**
    -   예외를 복구해서 정상적인 상태로 만들 수 없어, 발생한 예외를  **적절한 예외로 전환**해 메소드 밖으로 던짐
        
    -   내부에서 일어난 예외를 상황에 맞는 분명한 의미로 바꿔주기 위해 사용
        
        -   DAO에서 의미가 분명한 예외로 던지면, 서비스 계층 오브젝트에서 적절한 복구 작업 시도가능 👍🏻
        -   전환하는 예외에 원래 발생한 예외를 담아 중첩 예외로 만들기 권장 (새 예외 만들때 근본 예외 넣기)
    -   예외를 처리하기 쉽고 단순하게 만들기 위해 포장하기 위해 사용
        
        -   새로운 예외를 만들때 원인이 되는 에외를 넣는건 동일하지만, 의미를 명확히 하려고 바꾸는게 아님.
        -   대표적으로 EJBException - EJB 컴포넌트 코드에서 일어나는 대부분은 복구 불가 예외 ⇒ 포장
        -   **체크 예외를 계속 throws로 던지는 것은 무의미**하므로 빠르게 런타임 예외로 포장 👍🏻
        
        ```java
        try{
            OrderHome orderHome = EJBHomeFactory.getInstance().getOrderHome();
            Order order = orderHome.findByPrimaryKey(Integer id);
        }catch(NamingException ne){
            throw new EJBException(ne);
        }catch(SQLException se){
            throw new EJBException(se);
        }catch(RemoteException re){
            throw new EJBException(re);
        }
        
        // EJBException은 런타임 예외이므로 EJB는 시스템 익셉션으로 인식 후 트랜잭션을 자동 롤백함
        
        
        ```
        

### **예외 처리 전략**

-   **런타임 예외의 보편화**
    -   체크 예외에 대한 처리를 강제하는 것은 배려인 듯 싶지만 짜증나는 원인이기도 함
    -   자바의 환경이 서버로 이동 ⇒ API가 발생시키는 예외를 언체크로 정의하는게 일반화되는 중
    -   체크 예외의 활용도가 떨어지는 추세 ⇒ 런타임 예외로 전환후 던지는 게 나을때 많음
-   **add() 메소드의 예외 처리**
    -   어디에서든 예외를 잡아 처리가 가능하다면 체크예외가 아닌 런타임 예외로 만드는 것 권장
    -   다만 해당 예외를 던지는 메소드에 명시적으로 해당 예외를 던진다고 선언은 해야함
    -   런타임 예외로 만들 경우 장점이 많지만, 예외처리를 강제하지 않으므로 예외상황을 한번 고려하긴 할 것
-   **애플리케이션 예외**
    -   어플리케이션 자체 로직에 의해 의도적으로 발생시키고, 이를 잡아 조치 취하도록 요구하는 예외
    -   리턴 값을 결과 상태를 나타내는 정보로 활용 → 이를 확인하여 예외상황 체크 (체계가 없다면 불편)
    -   정상적 흐름의 코드는 두고, 예외상황에서는 비지니스적인 의미를 가진 예외를 던지게 함 → 처리는 강제

## SQLException

-   JdbcTemplate을 적용하면서 SQLException을 던지지 않음 →  **어디로 간 것인가?**
-   복구 불가능한 SQLException과 같은 예외는 빠르게 언체크/런타임 예외로 전환하는것을 권장했으므로
-   JdbcTemplate 템플릿 및 콜백 안의 모든 SQLException은 런타임 예외 DataAccessException로 포장
-   런타임 예외는 강제가 아니기 때문에 UserDao의 DAO메소드들에서 SQLException이 사라짐
-   JdbcContext나 JdbcTemplate이 사용하는 콜백 오브젝트는 SQLException을 템플릿으로 던짐
-   SQLException 처리는 콜백 오브젝트 아닌 템플릿이 하도록함

## 예외 전환

> **예외 전환의 목적**
> 
> 1.  런타임 예외로 포장해서 불필요한 catch/throws를 없애기 위해
> 2.  로우레벨의 예외를 좀 더 의미있고 추상화된 예외로 바꿔 던지기 위해

### **JDBC의 한계**

-   JDBC는 자바를 이용해 DB 접근법을 추상화된 API 형태로 정의 → DB종류 상관없이 동일방법 개발
-   하지만 DB 종류에 관계없는 유연한 코드 만들기는 어려우며 보장하지 X → (두가지 걸림돌 존재)
-   한계#1  **비표준 SQL**
    -   벤더마다 비표준 문법이 있기 때문 (ex. 오라클의 start with connect by 문법)
    -   비표준 SQL은 DAO 코드로 들어가게 되고, DAO는 특정 DB에 종속적인 코드가 됨
-   한계#2  **호환성 없는 SQLException의 DB 에러 정보**
    -   예외 발생시의 에러 코드의 종류와 원인이 벤더마다 다름
    -   e.getErrocode()로 가져오는 DB 에러코드가 각 DB마다 다름 😰
    -   SQLException은 예외 발생시 DB 상태를 담은 SQL상태정보를 부가제공 → getSQLState()
    -   JDBC 드라이버에서 SQLException을 담을 상태코드를 의존할 만큼 정확히 만들어주진 X

▶️ SQLException 만으로 DB에 독립적인 유연한 코드 작성은 어려움

### **DB 에러 코드 매핑을 통한 전환**

-   **DB별 에러 코드를 참고해서 발생한 예외 원인이 무엇인지 해석하는 기능 추가**로 한계#2 해결
-   DataAccessException은 SQLException을 대체하는 런타임 예외 뿐 아니라 여러 예외 클래스 정의
-   에러 코드 매핑정보 테이블 이용 → DB별 다른 에러코드 종류 확인
-   JdbcTemplate에서 던지는 모든 예외는 DataAccessException의 서브클래스 타입, 거의 신경 X 👌🏻

_어플리케이션 레벨의 체크 예외인 DuplicateUserUdException을 던지고 싶다면 (아래 같은 예외 전환 코드)_

```java
public void add() throws DuplicateUserIdException{
    try{
        //jdbctemplate을 이용해 user를 add하는 코드
    }catch(DuplicateKeyException e){
         //로그 등 작업
        throws new DuplicateUserIdException(e);//원인예외도 던져주기
    }
}


```

### **DAO 인터페이스와 DataAccessException 계층구조**

-   DataAccessException은 전환 외에도 다른 자바 데이터 엑세스 기술에서 발생하는 예외에도 적용
-   의미가 같은 예외라면 데이터 엑세스 기술 종류에 독립된 추상화 제공

**DAO 인터페이스와 구현 분리**

-   DAO를 따로 만들어서 사용하는 이유는
    -   데이터 엑세스 로직을 담은 코드를  **성격이 다른 코드에서 분리**해놓기 위해
    -   분리된 DAO는 전략 패턴을 적용해 구현 방법을 변경해서 사용할 수 있게 만들기 위해
-   DAO는 인터페이스를 사용해 구체적인 클래스정보와 구현방법을 감추고 DI를 통해 제공되기를 권장
-   인터페이스의 메소드들은 throws 선언을 주는데 이때문에 JDBC 외의 기술로 DAO 구현 전환 불가
-   인터페이스로 메소드 구현은 추상화 했지만, 기술마다 던지는 예외가 달라 메소드 선언이 달라짐
-   **DAO에서 모든SQLException을 런타임 예외로 포장해서 던지면 인터페이스에서 throws 없애**도 👌🏻
-   데이터 액세스 예외를 모두 무시하는건 아니고, 몇몇은 시스템 레벨에서 의미있게 분류해야함
-   여기서도 데이터 액세스 기술이 달라지면 같은 상황에서도 다른 예외가 던져짐 → 클라이언트가 DAO에 의존

**데이터 엑세스 예외 추상화와 DataAccessException 계층구조**

-   스프링은 자바의 다양한 데이터 엑세스 기술 사용시 발생하는 예외들을 추상화해둠
-   DataAccessException 계층구조에 정리했음
-   JdbcTemplate과 같이 스프링 데이터 액세스 지원 기술로 DAO 생성하면, 기술에 독립적인 예외 던짐 🙆🏻‍♀️

### **기술에 독립적인 UserDao 만들기**

**인터페이스 적용**

-   사용자 처리 DAO 이름을 UserDao, JDBC로 구현한 클래스 이름을 UserDaoJdbc
-   UserDao 인터페이스에는 기존 클래스에서 DAO의 기능을 쓰려는 클라이언트들이 가질것만 빼냄
-   기존의 UserDao 클래스를 UserDaoJdbc로 변경하고 UserDao 인터페이스 구현하도록함

**테스트 보완**

-   UserDaoTest에서 add() 중복 호출을 통해 어떤 예외가 발생하는지 체크
-   이때 스프링의 DataAccessException나 서브클래스 예외가 던져져야함

**UserDaoTest.java 일부**

```java
@Test(expected=DataAccessException.class) // 예외 검증을 위한 어노테이션
public void duplicateKey() {
    dao.deleteAll();
    dao.add(user1);
    dao.add(user1); // 기본 키 중복 예외가 발생할 것!
} // 어떤 예외인지 보려면 테스트실패하도록 하면 됨 (expected 조건 빼서) 

```

**DataAccessException 활용 주의사항**

-   DuplicationKeyException은 JDBC를 이용하는 경우에만 발생 (다른 데이터 엑세스 기술 사용시 발생X)
-   JDBC는 SQLException에 담긴 DB 에러코드를 바로 해석하지만, JPA 등의 다른 기술들은 아님 🔽
-   재정의한 예외를 가져와 스프링이 DataAccessException으로 변환하는데이런 예외들은 세분화 X 때문
-   주의! DataAccessException은 기술 상관없이 어느정도는 추상화된 공통예외로 변환해주지만 한계 🈶

_SQLException을 직접 해석해서 DataAccessException으로 변환하는 코드_

```java
public class UserDaoTest{
    @Autowired UserDao dao;
    @Autowired DataSource dataSource;
}

```

-   에러코드 변환에 필요한 DB종류를 알아내야 하기 때문에 DataSource 변수추가 후 빈을 받음

```java
@Test
public void sqlExceptionTranslate() {
    dao.deleteAll();

    try {
        dao.add(user1);
        dao.add(user1); // 강제로 중복키 에러 발생
    }catch(DuplicateKeyException ex) {
        SQLException sqlEx = (SQLException)ex.getRootCause();//중첩예외 뽑기

        SQLExceptionTranslator set =         //스프링 예외 전환 API
            new SQLErrorCodeSQLExceptionTranslator(this.dataSource);
        assertThat(set.translate(null, null, sqlEx), 
                                       is(DuplicateKeyException.class));
    }
}

```

-   DataSource를 사용해 SQLException에서 DuplicateKeyException으로 전환하는 기능 테스트
-   DuplicateKeyException은 중첩된 예외로 중첩예외를 찾기 위해 getRootCause() 사용
-   SQLException 형변환 처리 후 sqlEx 변수에 담아줌
-   스프링 예외 전환 API를 사용하여 DuplicateKeyException이 만들어지는지 관찰
    -   주입받은 dataSource를 이용해 SQLErrorCodeSQLExceptionTranslator의 오브젝트 생성후
    -   SQLException을 파라미터로 넣어 translate()를 호출하면
    -   SQLExceptiond을 DataAccessException 타입 예외로 변환해줌
-   변환된 예외가 DataAccessException 타입의 예외가 맞는지 assertThat으로 확인
