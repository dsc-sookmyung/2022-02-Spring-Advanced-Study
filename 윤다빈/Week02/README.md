# ****2장****  테스트

<aside> 💡 테스트는 스프링이 강조하는 도구로 만들어진 코드에 대한 확신과 변화에 대한 유연성을 주는 기술

</aside>

➡️ 테스트란 무엇인지, 테스트의 가치와 장점, 테스트의 활용전략, 스프링과의 관계를 알아볼것, 대표프레임워크

## UserDaoTest 다시 보기 및 개선

### 테스트의 가치와 장점

-   테스트란? 코드가 예상하고 의도한 대로 동작하는지에 대한 확신을 주는 작업
-   코드의 구조와 설계, 적용된 기술이 변경되더라도 기능을 잘 수행함을 보장해줌
-   결과가 원하는 바가 아닐 경우에는 결함의 존재를 알수 있고 이를 제거해가는 디버깅을 할 수 있음

### UserDaoTest (단위 테스트)

```java
// 기존에 1장에서 main()에서 만든 테스트 코드
public class UserDaoTest {
    public static void main(String[] args) throws SQLException{
        ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
        
        UserDao dao = context.getBean("userDao", UserDao.class);
        User u = new User();
        User.setId("user");
        User.setName("윤다빈");
        User.setPassword("990608");
        
        dao.add(u);
        System.out.println(u.getId()+"등록 성공");

        User u2 = dao.get(u.getId());
        System.out.println(u2.getName());
        System.out.println(u2.getPassword());
        System.out.println(u2.getId()+"조회 성공");
    }
}
// 테스트 대상인 UserDao 객체를 직접 가져와서 메소드를 호출
// 테스트에서 쓸 입력값을 직접 코드에서 만들어서 넣어주며 그 테스트의 결과를 콘솔에 출력
// 각 단계의 작업이 에러 없이 끝나면 콘솔에 성공 메세지로 출력함
// DAO 라는 기능과 DB까지가 단위이고, DB 상태는 UserDaoTest가 관리하므로 단위테스트

```

**웹을 통한 DAO 테스트 방법의 문제점**

-   DAO만 테스트하는게 아니라 서비스, 컨트롤러, JSP 뷰등 모든 계층의 기능을 만들고 나서 테스트 가능
-   한 테스트에 여러 클래스와 코드가 참여하기 때문에 문제 발생시 발생위치를 찾아야하는 번거로움이 있음

**작은 단위 테스트**

-   관심사의 분리 원리 적용 : 테스트의 관심이 다르면 테스트할 대상을 분리하고 집중해서 접근할 것
-   **단위**  : 충분히  **하나의 관심에 집중해 효율적으로 테스트하는 범위**로 크기와 범위가 확정된 것은 아님
-   단위가 작을수록, 단위에 있지 않는 코드는 신경쓰지 않고 참여하지 않아도 테스트가 동작하면 👍🏻
-   **DB의 사용**  : 테스트가 DB의 상태를 관장하면 단위테스트, 아니라면 단위테스트 X (외부 리소스 의존 🙅🏻‍♀️)
-   **필요성**
    -   개발자의 코드가 의도대로 동작하는지 빨리 확인받기 위함
    -   확인 대상과 조건을 간단명료히 해서 수행과정을 단순화 할 수 있음
    -   문제의 원인을 빠르고 쉽게 찾아 수정하기 위해서 사용

**자동수행 테스트 코드**

-   테스트 자체가 사람의 수작업을 타는 방법보다는 코드로 만들어져 자동으로 수행되도록 해야됨
-   애플리케이션을 구성하는 클래스 안 보단 별도의 테스트 클래스를 만들어 테스트 코드를 넣을것
-   장점
    -   번거로운 작업 없이 빠르게 테스트를 실행하므로 자주 반복 가능
    -   운영 중인 프로그램의 코드를 수정하려고 할 때, 수정에 대한 기능 테스트를 빠르게 진행해 볼 수 있음
    -   기능이 동작하는 단순코드를 좋은 객체 지향적 코드로 점진적 발전시킬때, 기능보장&오류 원인 확인 도움

**UserDaoTest의 문제점**

-   **수동 확인 작업의 번거로움**  : 콘솔의 값을 가지고 사람이 확인해야함 ⇒ 테스트코드가 확인하지 ❌
-   **실행 작업의 번거로움**  : main() 메소드가 부담 ⇒ 더 편리하고 체계적으로 테스트 실행 및 결과 관리법 필요

**테스트 검증의 자동화**

-   테스트 실패 종류
    -   테스트 에러 : 테스트가 진행되는 동안 에러가 발생해서 실패 (콘솔에 다 나오므로 확인 쉬움)
    -   **테스트 실패**  : 테스트 진행중 에러는 없었지만 결과값과 기댓값이 다른 경우 (⭐️별도 확인 필요)
-   자동화된 테스트는 빠른 실행이 가능하며 스스로 테스트 수행과 기댓값과 결과값을 확인까지 해주는 코드

```java
if (!u.getName().equals(u2.getName())){
    System.out.println("테스트 실패 - name");
}
else if (!u.getPassword().equals(u2.getPassword())){
    System.out.println("테스트 실패 - pw");
}
else {
    System.out.println("조회 테스트 성공");
}
// 👉🏻 결과를 콘솔에서 직접 확인해서 비교X 태스트 성공 메세지 확인, 실패 경우 실패지점도 알 수 있음

```

**테스트의 효율적인 수행과 결과 관리**

-   어플의 규모와 테스트 갯수가 커질수록 테스트 수행 부담 테스트 ⬆️ ⇒  **테스트 지원 도구 및 작성 방법 필요**
-   테스트 지원도구의 기능 : 일정 패턴의 테스트 생성, 다수의 테스트를 쉽게 실행, 종합결과 확인, 실패지점 탐색
-   **JUnit 테스트로 전환**  : JUnit은 프레임워크이므로 main() 메소드와 객체를 만들어 실행하는 코드 필요 X
-   **테스트 메소드 전환**
    -   main()메소드의 테스트 코드를 일반 메소드로 옮기며 JUnit의 요구 조건에 따라야 함
    -   요구 조건 : 메소드가 public으로 선언될 것, 메소드에 @Test 어노테이션을 붙일 것
    -   메소드의 이름은 테스트의 의도를 알 수 있도록 붙여줄 것
-   검증 코드 전환
    -   위의 if/else 문장 대신 JUnit이 제공하는 assertThat 이란 스태틱 메소드 이용
    -   assertThat(u2.getName(), is(u.getName())) ; 첫 파라미터를 뒤의 매처라는 조건으로 비교

```java
import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.assertThat;

public class UserDaoTest {

    @Test
    public void  addAndGet() throws SQLException {
        ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
        UserDao dao = context.getBean("userDao", UserDao.class);
        
        User user = new User();
        User.setId("huhak");
        User.setName("휴하하학");
        User.setPassword("sofunny");

        dao.add(user);

        User user2 = dao.get(user.getId());

        assertThat(user2.getName(), is(user.getName()));
        assertThat(user2.getPassword(), is(user.getPassword()));
    }
}

```

-   JUnit 테스트 실행 : 자바 코드로 만들어진 프로그램이므로 시작시켜야함 ⇒ main() 메소드를 추가
    
    ```
     `public static void main(String[] args){`
          `JUnitCore.main(”springbook.user.dao.UserDaoTest”);`
     `}`
    
    ```
    

## 개발자를 위한 테스트 프레임워크 JUnit

-   스프링 테스트 모듈이 JUnit(main() 메소드와 System.out.println()으로 만들어져 단순하고 빠름)을 이용

### JUnit 테스트 실행방법

-   이클립스와 같은 IDE 사용 : 테스트의 실행과 결과 확인이 매우 간단하고 직관적 (개인👍🏻)
-   메이븐 같은 빌드툴 사용 : 빌드툴에서 제공하는 플러그인등을 사용 결과는 HTML 혹은 텍스트 파일(단체👍🏻)

### 테스트 결과의 일관성

-   테스트는 항상 일관성 있는 결과가 보장되어야 함 - 외부환경, 실행순서등에 영향받지 않을 것
    
-   UserDaoTest의 개선점 : 테스트가 외부 상태에 따라 성공여부가 달라짐 (DB의 이전 데이터 삭제해야만 함)
    
-   UserDao에 deleteAll(USER 테이블의 모든 레코드 삭제), getCount(USER 테이블의 레코드 갯수) 추가
    
    ```java
    public void deleteAll() throws SQLException{
        Connection c = dataSource.getConnection();
        PreparedStatement ps = c.prepareStatement("delete from users");
        ps.executeUpdate();
        ps.close();
        c.close();
    }
    
    public int getCount() throws SQLException{
        Connection c = dataSource.getConnection();
        PreparedStatement ps = c.prepareStatement("select count(*) from users");
        ResultSet rs = ps.executeQuery();
        rs.next();
        int count = rs.getInt(1);
        
        rs.close();
        ps.close();
        c.close();
        rerurn count;
    }
    
    ```
    
-   테스트 후에 결과를 지워주는 것 보단  **테스트 수행 전 이전값을 지워주는 편**이 더 문제가 발생하지 않음
    

### 포괄적인 테스트

-   꼼꼼하지 않은 테스트로 인해 테스트가 성공하는 것도 위험한 일 → 테스트에서 한가지의 경우만 검증 ❌
    
-   JUnit은 하나의 클래스 안에 @Test가 붙은 public void에 파라미터가 없는 여러 메소드가 들어가는것 ⭕
    
-   단, JUnit은 특정 테스트 메소드의 실행 순서를 보장하지는 않음 ⇒ 모든 테스트는 순서 상관 없이 독립적일것
    
-   **예외조건의 테스트 ⇒ JUnit의 예외 테스트 기능**
    
    -   메소드에 전달된 값에 해당하는 사용자 정보가 없다면 : null등의 특정값 리턴 / 찾을 수 없다고 예외 던짐
    -   문제점 : 일반적으로 테스트중 예외가 발생하면 실행 중단되며 테스트가 실패 ⇒ 테스트 에러
    -   목표 : 테스트 진행중 특정 예외 던져지면 테스트 성공, 아니면 실패 - assertThat()으론 검증 불가
    
    ```java
    @Test(expected=EmptyResultDtatAccessException.class)
    public void getUserFailure() throws SQLException {
        //...
        // => 여기쯤에서 예외 발생, Test 어노테이션 안의 expected에 써둔 예외가 발생하면 테스트 성공
        ...//
    }
    
    ```
    
    ```java
    // UserDao 도 수정해야 함 get() 메소드에서 데이터 없을경우 에러 던지도록
    public User get(String id) throws SQLException {
        ResultSet rs = ps.executeQuery();
    
        User user = null //초기화
        if (rs.next()){
            user = new User();
            user.setId(rs.getString("id"));
            user.setName(rs.getString("name"));
            user.setPassword(rs.getString("pw"));
        }
        rs.close(); ps.close(); c.close();
        if (user==null) throws new EmptyResultDtatAccessException(1);
        return user
    }
    
    ```
    

<aside> 💡  **긍정적인 경우만 골라 성공할 테스트를 만들지 말고, 네거티브 테스트를 먼저 만들도록 노력할 것**

</aside>

### 테스트 주도 개발

**기능 설계를 위한 테스트**

-   테스트 코드를 기능 정의서라고 볼 수 있음 - 추가하고픈 기능을 테스트 코드로 표현해서 설계문서처럼 만듦
-   장점 : 해당 테스트 코드가 성공하게 되면, 코드 구현과 테스트의 과정 두 작업이 모두 끝남
-   ex) 존재X 아이디로 get() 메소드 수행시, 특정 예외 던져질 것 (아래는 getUserFailuer()에 나타난 기능)

<aside> 💡  **TDD (Test Driven Development) : 테스트코드 선제작후, 테스트를 통과하는 코드를 작성하는 방식**

</aside>

-   장점
    -   테스트 제작을 잊지 않으며 코드에 대한 확신을 가지게 됨
    -   코드 완성과 테스트 진행 사이 간격이 짧아 빠르게 피드백이 가능 → 오류 빠르게 발견 가능
    -   테스트는 어플리케이션 코드보다 작성이 쉬우며 테스트마다 독립적이므로 비교적 빠른 작성 가능
-   테스트를 작성하고 이를 성공 시키는 코드를 만드는 작업 주기를 되도록 짧게 가져갈 것 (→ 단위테스트일것)
-   엔터프라이즈 애플리케이션의 테스트는 제작이 어려우나, 스프링은 테스트에 용이한 구조의 어플리케이션을 만들게 도움

### 테스트 코드 개선

-   JUnit 프레임워크는 테스트 메소드 실행마다 부가적으로 해주는 작업들이 존재 (@Before, @After)
    
-   테스트 실행마다 반복적 준비 작업을 별도 메소드에 넣고 이 메소드를 테스트 메소드 실행전/후 실행
    
-   단, @Before, @After 메소드를 테스트 메소드에서 직접 호출X ⇒  **주고 받을 정보는 인스턴스 변수 이용**
    
-   테스트 메소드 실행마다 테스트 클래스의 객체를 새로 만듦 → 테스트 후 객체는 버려짐 (독립성 보장을 위해)
    
-   테스트 메소드의 일부에서만 공통되는 코드라면 일반 메소드 추출기법을 써서 해당 메소드에서 직접 호출
    
-   JUnit이 하나의 테스트 클래스를 가져와서 수행하는 방식
    
    ```java
    import static org.junit Before;
    
    public class UserDaoTest {
    
        private UserDao dao;  // **픽스처 : 테스트 수행시 필요한 정보나 오브젝트** 
        private User u1       **//         => 중복 제거를 위해 @Before 메소드로 추출함**
        private User u2
    
        @Before
        public void setUp(){
            ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
            this.dao = context.getBean("userDao", UserDao.class);    
            
            this.u1 = new User("dabin", "다빈", "spring");
            this.u2 = new User("typhoon", "배고파", "springfun");
        }
    
        @Test
        public void  addAndGet() throws SQLException {}
        @Test
        public void  count() throws SQLException {}
        @Test(expected=EmptyResultDtatAccessException.class)
        public void getUserFailure() throws SQLException {}
    }
    
    ```
    
    1.  테스트 클래스에서 @Test 어노테이션이 있는 public void형의 파라미터 없는 테스트 메소드들 찾기
    2.  테스트 클래스의 객체를 하나 생성하고 @Before 메소드가 있는 메소드가 있다면 실행
    3.  @Test가 붙은 메소드를 하나 호출하고 테스트 결과를 저장후 @After가 붙은 메소드가 있으면 실행
    4.  나머지 테스트 메소드들에 대해서도 2-3 반복후 모든 결과를 종합해서 돌려줌

## 스프링 테스트 적용

어플리케이션 컨텍스트

빈 오브젝트 생성마다 많은 시간을 필요로 할 수도 있음

테스트 종료마다 어플리케이션 컨텍스트안의 빈이 할당한 자원을 깔끔하게 정리해야 함

### 테스트를 위한 어플리케이션 컨텍스트 관리

<aside> 💡  **스프링은 테스트 컨텍스트 프레임워크 지원 컨텍스트 프레임워크는 테스트에 필요한 어플리케이션 컨텍스트 만든 후 모든 테스트가 공유하도록 함**

</aside>

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest {

    @Autowired
    private Application context;

    @Before
    public void setUp(){
        this.dao = this.context.getBean("userDao", UserDao.class);
    }
}

```

-   @RunWith : JUnit 프레임워크의 테스트 실행 방법 확장시 사용
-   SpringJUnit4ClassRunner.class 로 설정시, 테스트시 JUnit이 테스트가 쓸 어플리케이션 컨텍스트 관리
-   ContextConfiguration(locations) : 테스트 컨텍스트가 자동으로 만들 어플리케이션 컨텍스트의 위치 결정
-   @Autowired private Application context : 테스트 오브젝트 생성후 테스트 컨텍스트에 의해 자동 주입
-   context 변수에는 어플리케이션 컨텍스트가 들어 있음

😉 테스트 클래스 내 테스트 메소드끼리 어플리케이션 컨텍스트 공유, context 변수엔 어플리케이션 컨텍스트 듦

-   테스트 실행전 딱 한번만 어플리케이션 컨텍스트 생성 후, 테스트 객체 만들어질때마다 테스트 특정 필드에 DI

**테스트 클래스의 컨텍스트 공유**

-   테스트 클래스가 같은 설정 파일을 사용한다면, 테스트 클래스끼리도 어플리케이션 컨텍스트 공유 가능
-   @Autowired : 스프링의 의존성 주입에 이용되는 어노테이션 ⇒ 어떤 빈이든 다 가져올 수 있음
    -   스프링 어플리케이션 컨텍스트는 초기화시 자신도 빈으로 등록하므로 DI가 가능
    -   해당 어노테이션이 붙은 인스턴스 변수가 있다면, 컨텍스트 내 동일 변수타입인 빈을 찾음
    -   타입이 일치하는 빈이 있을 경우, 인스턴스 변수에 주입 (메소드 필요 없음)

### DI와 테스트

**인터페이스를 사용하고 DI를 적용해야 하는 이유**

-   언젠가 변경이 일어날 수 있기 때문. 이 변화에 간단한 처리로 시간과 비용 부담을 줄여줌 (클래스대신 인터페이스, new로 생성대신 DI 주입으로 변경하는건 아주 쉽기 때문)
-   인터페이스를 두면 다른 차원의 서비스 기능을 도입 및 삭제를 간단한 조작으로 가능케 함
-   작은 단위의 테스트(효율적인 테스트)가 독립적으로 만들어지고 실행되는데 중요 역할

테스트에 DI를 이용하는 방법 3가지

1️⃣ **테스트 코드에서 수동으로 DI**

-   UserDao가 쓸 DataSource 오브젝트를 테스트 코드에서 변경 가능 ⇒ 테스트 코드 내에서 직접 DI
-   테스트중 DB 삭제 위험 있으므로 테스트중 DAO가 쓸 DataSource 오브젝트 변경
-   장점보다 단점이 큼 : 코드가 많아져 번거로우며, 어플리케이션 컨텍스트도 매번 새로 만들어야 함

```java
// 테스트를 위한 수동 DI를 적용한 UserDaoTest
@DirtiesContext 
// 테스트 메소드에서 애플리케이션 컨텍스트의 구성이나 상태를 변경함을 테스트 컨텍스트 프레임워크에 알림
public class UserDaoTest{
    @Autowired
	  UserDao dao;
    @Before
	  public void setUp(){
		    ... // 테스트에서 UserDao가 쓸 DataSource 오브젝트를 직접 생성
		    DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost/testdb", "spring", "book", true);
		    dao.setDataSource(dataSource); // 코드에 의한 수동 DI
	}
}

```

⏬ 테스트에서 쓸 DataSource 클래스가 빈으로 정의된 테스트 전용 설정파일을 따로 만들어보자.

2️⃣ ** 테스트를 위한 별도 DI 설정**

-   설정파일을 두가지 생성해 하나는 서버 운영용의 DataSource를 빈으로 등록
-   다른 하나에는 테스트에 맞게 준비된 DB를 쓸 가벼운 DataSource를 빈으로 등록
-   항상 테스트에는 테스트 전용 설정파일만 사용

😊어플리케이션 컨텍스트 한개를 모든 테스트에서 공유할 수도 있게 되었음

```java
// 기존 applicationContext.xml 복사후 **test-applicationContext.xml** 생성 
// 다른 빈의 설정은 두고 dataSouce 빈의 설정만 테스트용으로 변경
<bean id="dataSource"
    class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
	  <property name="url" value="jdbc:mysql://localhost/testdb"/>
	  <property name="username" value="spring"/>
	  <property name="password" value="book"/>
</bean

```

```java
// 테스트용 설정 파일 적용 ContextConfiguration을 이용
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(location="/test-applicationContext.xml")
public class UserDaoTest {}

```

**3️⃣ 스프링 컨테이너 없는 DI 테스트**

-   스프링 컨테이너를 써서 IoC 방식의 생성 및 DI 되는 대신, 테스트 코드에서 직접 오브젝트 생성 후 DI
-   UserDao코드가 DAO로서 DB에 정보 등록 및 조회를 잘 하는지 확인하는 UserDaoTest (그외는 관심밖)

```java
public class UserDaoTest{
    UserDao dao; 

	  @Before
	  public void setUp(){
		    // 오브젝트의 생성, 관계설정 등을 모두 직접 해줌
		    dao = new UserDao();
		    DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost/testdb", "spring", "book", true);
		    dao.setDataSource(dataSource);
	}
}

```

-   @RunWith, @Autowired(어플리케이션 컨텍스트에서 가져와줌)가 없음
-   @Before 메소드에서 직접 UserDao 오브젝트 생성 후 테스트용 DataSource 오브젝트 생성 후 직접 DI

😊 어플리케이션 컨텍스트가 만들어질 필요가 없으니 코드가 단순해지고, 테스트 시간도 절약됨

▶️ DI는 객체지향 프로그래밍 스타일이므로  **DI를 위해 컨테이너가 반드시 필요한 것은 x**

<aside> 💡  **DI 컨테이너나 프레임워크는 DI를 적용을 편리하게 돕는 도구, 없어도 다른 방법으로 DI 가능**

</aside>

**3가지 방법중 어떤 것을 선택할까?**

😊 세 가지 모두 장단점 존재, 상황에 따라 달리 사용

-   **우선 고려 : 스프링 컨테이너 없이 테스트 (방법3)**
-   **여러 오브젝트와 복잡한 의존 관계를 가진 오브젝트 테스트할 경우**  : 테스트 전용 설정파일 따로 생성 (방법2)
-   **예외적인 의존 관계를 강제로 구성해서 테스트하는 경우**  : (방법1)

### 학습 테스트로 배우는 스프링

<aside> 💡  **학습테스트는 직접 만들지 않은 프레임워크나 제공받은 라이브러리등에 대해 테스트하는것**

</aside>

**학습테스트의 목적**

: 직접 쓸 API나 프레임워크의 기능을 테스트로 보면서 사용법을 익힐 수 있음 (이해도나 사용법을 아는지 검증)

**학습 테스트의 장점**

-   다양한 조건에 따른 기능을 손쉽게 확인할 수 있음
-   학습 테스트 코드를 개발 중에 참고 할 수 있음
-   프레임워크나 제품을 업그레이드할 때 호환성 검증을 도움
-   테스트 작성에 대한 좋은 훈련이 됨
-   새로운 기술을 공부하는 과정이 즐거워짐

👉🏻 스프링 학습 테스트를 만들 때 가장 좋은 참고 소스는 스프링 자신의 테스트 코드. 스프링 배포판 압축 풀면 👀

**학습 테스트 예제**

1️⃣ **JUnit으로 만드는 JUnit 자신에 대한 테스트**

-   새로운 테스트 클래스 생성 후 적당한 이름의 테스트 메소드 3개 추가
-   스태틱 변수로 테스트 오브젝트를 저장할 컬렉션 생성
-   테스트마다 현재 테스트 오브젝트가 컬렉션에 있는지 확인 후 없으면 자신을 추가 (반복)

```java
import static org.junit.matchers.JUnitMatchers.hasItem;

public class JUnitTest{
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

2️⃣  **스프링 테스트 컨텍스트 프레임워크에 대한 학습 테스트**

-   새로운 설정 파일 junit.xml 생성
-   JUnitTest 클래스에 @RunWith, @ContextConfiguration 추가 새 설정 파일 쓰는 테스트 컨텍스트 적용
-   @Autowired 로 주입된 context 변수가 같은 오브젝트인지 확인하는 코드 추가

```java
// JUnit 테스트를 위한 빈 설정파일 junit.xml 추가
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="<http://springframework.org/schema/beans>"
    xmlns:xsi="<http://www.w3.org/2001/XMLSchema-instance>"
	  xsi:schemaLocation="<http://www.springframework.org/schema/beans>
		    <http://www.springframework.org/schema/beans/spring-beans.xsd>">
</beans>

// 스프링 테스트 컨텍스트에 대한 학습 테스트를 위해 JUnitTest 수정
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration
public class JUnitTest{

    @Autowired
	  ApplicationContext context;

	  static Set<JUnitTest> testObjects = new HashSet<JUnitTest>();
	  static ApplicationContext contextObject = null;

	  @Test public void test1() {
		    assertThat(testObjects, not(hasItem(this)));
		    testObjects.add(this);
        assertThat(contextObject == null || contextObject == this.context, is(true));
		    contextObject = this.context;
	  }

	 @Test public void test2() {
	 	   assertThat(testObjects, not(hasItem(this)));
		   testObjects.add(this);
       assertTrue(contextObject == null || contextObject == this.context);
		   contextObject = this.context;
	 }

	@Test public void test3() {
		   assertThat(testObjects, not(hasItem(this)));
		   testObjects.add(this);
       assertThat(contextObject, either(is(nullValue())).or(is(this.context)));
		   contextObject = this.context;
	}
}

```

**검증 로직을 코드로 만드는 방법**

-   **assertThat()**과 **is()**를 적절히 사용
    -   is()는 타입만 일치하면 어떤 값이든 검증 가능
-   조건문 받아서 그 결과가 true인지 false인지 확인하도록 만들어진 **assertTrue()** 사용 (더 간결)
-   조건문 넣어서 그 결과를 true와 비교하는 대신 매처의 조합을 이용
    -   **either()**는 뒤에 이어나오는 **or()**와 함께 두 개의 매처의 결과를 OR로 비교 (둘중 하나만 T 성공)
    -   **nullValue()** : 오브젝트가 null인지 확인

**버그 테스트(bug test)**

-   코드에 오류가 있을때, 그 오류를 가장 잘 보여주는 테스트로 실패하도록 만들어야 함
-   버그가 원인이 되어 테스트가 실패 → 버그 테스트가 성공하도록 어플리케이션 코드 수정 (성공시 해결완료)
-   필요성 및 장점
    -   테스트의 완성도를 높여줌 (불충분했던 테스트 보완, 비슷한 문제의 추적 용이)
    -   버그의 내용을 명확하게 분석하게 해줌
    -   기술적인 문제를 해결하는 데 도움이 됨

> **동등분할**  같은 결과를 내는 값의 범위마다 각 대표 값으로 테스트하는 방법 어떤 작업의 결과의 종류가 true, false 또는 예외발생 세 가지라면 각 결과를 내는 조합을 만들어 모든 경우에 대한 테스트할 것

**경계값 분석**  에러는 동등분할 범위의 경계에서 주로 많이 발생한다는 특징을 이용 경계의 근처에 있는 값을 이용해 테스트하는 방법 보통 숫자의 입력 값인 경우 0이나 그 주변 값 또는 정수의 최대값, 최소값 등으로 테스트할 것
