# 3장 템플릿

🔗 https://pickle-fireplant-fa1.notion.site/3-8e8a4a66a6034d73adee1814084b9020

<aside> 💡 스프링에 적용된 템플릿 기법과 적용과정을 알아볼 것 </aside>

템플릿이란 변동성 있는 코드들 사이, 일정 패턴으로 유지되는 부분만 독립시켜 활용하는 방법

## 예외상황 처리가 가능한 UserDao 코드

> JDBC 코드에서 꼭 지켜야할 원칙은 예외처리 - 예외 발생 여부와 관계없이 썼던 리소스를 반환해야 하므로

### JDBC 수정/조회 기능의 예외처리 코드

-   수정의 deleteAll() 에서 사용하는 리소스는 Connection, PreparedStatement
-   처리중 예외로 인해 close()가 실행되지 않아 풀로 리소스를 반환하지 못하는 경우를 대비 ⇒ try/catch/finally 구문
-   조회의 getCount() 메소드의 경우 리소스를 Connection, PreparedStatement, ResultSet을 사용 → 아래처럼 적용

```java
public void deleteAll() throws SQLException {
    Connection c = null;
    PreparedStatement ps = null;
  
    // try: 예외 발생 가능성 있는 코드들, catch: 예외발생시 처리할 작업들, finally: 예외 상관 없이 실행할 작업
    try {
        c = dataSource.getConnection();
        ***ps = c.prepareStatement("DELETE FROM users");  // 변하는 부분***
        ps.executeUpdate();
    } catch (SQLException e) {
        throw e;
    } finally {
  // **👆🏻 예외 발생 시점에 따라 c와 ps의 close() 메소드 호출이 달라지므로 여기서 처리 (null일 경우 close() 메소드 호출X)**
        if (ps != null)
            try {
                ps.close();
            } catch (SQLException e) { // ps.close()에서 문제 발생시 처리 }
        if (c != null) 
            try {
                c.close();             // Connection 반환
            } catch (SQLException e) { //c 리소스 반환 실패경우 처리 }
        // 현재 catch에서는 메소드 밖으로 SQLException을 던지는 일만 함
    }
}

```

> **⚠️ 문제점 - 모든 메소드마다 복잡한 try, catch, finally 구문이 반복되고, 수정이 어려움**

## 분리와 재사용을 위한 디자인 패턴

### # Way1 변하지 않는 부분의 메소드 추출

-   변하는 부분과 변하지 않는 부분이 분리되었으나, 변하지 않는 부분은 계속 반복되므로 개선된게 딱히 없음

### # Way2템플릿 메소드 패턴의 적용

-   변하지 않는 부분은 슈퍼클래스, 변하는 부분은 추상 메소드로 정의 후 서브클래스에서 오버라이드해 재정의
-   단점
    -   모든 DAO 메소드마다 상속으로 새로운 서브클래스를 만들어야함
    -   컴파일 시점에 클래스 간(슈퍼-서브) 관계가 결정되어 있어서 유연성이 없는 관계가 됨

### # Way3 전략 패턴의 적용

-   개방 폐쇠 원칙을 가장 잘 지키면서, Way2보다 유연하고 확장성이 뛰어남
-   변하는 부분을 별도의 클래스로 만들어 추상화된 인터페이스를 통해 소통하도록 구성

```java
// 인터페이스
public interface StatementStrategy {
    PreparedStatement makePreparedStatement(Connection c) throws SQLException;
}

// 인터페이스를 상속한 실제 전략 (바뀌는 부분)
public class DeleteAllStatement implements StatementStrategy {
    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        PreparedStatement ps = null;
        ps = c.prepareStatement("DELETE FROM users");
        return ps;
    }
}

```

```java
package springbook.user.dao;
// ...
public void deleteAll() throws SQLException {
    Connection c = null;
    PreparedStatement ps = null;

    try {
        c = dataSource.getConnection();
        **StatementStrategy stmt = new DeleteAllStatement(); // 메서드에 따라 바뀜**
        ps = stmt.makePreparedStatement(c);
        ps.executeUpdate();
    } catch (SQLException e) {
        throw e;
    } finally {
        if (ps != null)
            try {
                ps.close();
            } catch (SQLException e) {
            }
        if (c != null) try {
            c.close();
        } catch (SQLException e) {

        }
    }
}

```

**⚠️ 템플릿 메소드 패턴과 유사하게 UserDao(컨텍스트)가 무엇을 실행할 지 컴파일 시점에 알고 있음**

## # Way4 클라이언트와 컨텍스트 분리 - DI적용

![image.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c271cf68-c97f-481d-bc61-9d8da4f7ca2a/image.png)

```java
// Client
public void deleteAll() throws SQLException {
  StatementStrategy stmt = new DeleteAllStatement();// 택한 전략클래스의 오브젝트 생성
  jdbcContextWithStatementStrategy(stmt);           // 컨텍스트 호출 후 오브젝트 전달
}

// Context , StatementStrategy가 클라이언트가 컨텍스트 호출시 넘길 전략인자
// jdbc와 관련된 (리소스 요청, 반납 등의) 작업들
public void jdbcContextWithStatementStrategy(StatementStrategy stmt) throws SQLException{
    
    Connection c = null;
    PreparedStatement ps = null;

    try {
        c = dataSource.getConnection();
        **ps = stmt.makeStatement(c); // ps생성 시점에서 strategy를 호출해서 사용**
        ps.executeUpdate();
    } catch (SQLException e) {
        throw e;
    } finally {
        if (ps != null)
            try {
                ps.close();
            } catch (SQLException e) {
            }
        if (c != null) try {
            c.close();
        } catch (SQLException e) {
        }
    }
}

// Strategy (이전과 동일), 알맞은 preparedstatement 생성
public interface StatementStrategy {
    PreparedStatement makePreparedStatement(Connection c) throws SQLException;
}

public class DeleteAllStatement implements StatementStrategy { // 구현체
    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
        PreparedStatement ps = null;
        ps = c.prepareStatement("DELETE FROM users");
       
        return ps;
    }
}

```

-   컨텍스트가 쓸 전략을 클라이언트가 전달하므로 DI구조
-   PrepareStatement를 실행하는 JDBC 작업 흐름이 있는 컨텍스트를 DAO 메소드들이 공유 가능
-   DAO 메소드는 전략 패턴의 클라이언트로, 컨텍스트에 맞는 전략(ps 생성)을 제공
-   전략 클래스의 추가 정보가 있다면, 클라이언트 메소드 생성시 추가후, 전략 생성시 생성자로 제공함

## 전략과 클라이언트의 동거

> 위의 코드에서 문제점
> 
> -   DAO 메소드마다 새로운 전략 구현 클래스 생성
> -   DAO 메소드에서 추가 전달할 정보가 있는 경우, 구현체 생성시 번거로움 따로(생성자와 변수등 추가)

### # 해결방법1 로컬 클래스

-   전략은 해당 메소드안에서만 쓰니까 전략 구현 클래스를 매번 독립된 파일이 아닌 DAO의 내부 클래스로 정의
-   위에서 추가 정보가 있을때, 전달받기 위해 사용했던 생성자와 인스턴스 변수의 제거가 가능

### # 해결방법2 익명 내부 클래스

-   특정 메소드에서만 사용할 용도로 만들어진 클래스의 이름 제거를 위해 익명 내부 클래스를 사용
-   익명 내부 클래스 : 클래스의 선언 + 오브젝트 생성, 클래스 밖의 변수는 final 키워드가 있어야만 사용 가능

```java
// Client 
public void deleteAll() throws SQLException {
  // Context (선언은 분리되어 있음..)
  jdbcContextWithStatementStrategy(
    // Strategy - 선언 따로 없이 new 인터페이스이름() {구현 전략}
    new StatementStrategy() {
      @Override
      public PreparedStatement makeStatement(Connection c) throws SQLException      
      {return c.prepareStatement("DELETE FROM users");}
    }
  );
}

```

## 컨텍스트와 DI

**JDBC Context의 분리**

-   `jdbcContextWithStatementStrategy()`는 JDBC 의 일반 작업 흐름 → 다른 DAO에도 적용 가능
-   별도 클래스 분리 ⇒ JdbcContext 클래스 workWithStatementStrategy 메소드
-   UserDao 내부에 위치할때와 달리 따로 나왔으므로 JdbcContext에서  **dataSource**가 필요함
-   인터페이스 사용 없지만 객체의 생성과 관계설정의 제어권한을 외부로 위임(IoC)했으므로 DI를 따른다 봄
-   인터페이스를 사용하지 않는 이유는 UserDao와 JdbcContext가 긴밀한 관계로 결합되어 있기 때문
-   두가지 방법들 중 상황에 맞게 판단해서 사용할 것

![UserDao는 JdbcContext에 의존, JdbcContext는 구현 방법이 바뀌진 X ⇒ 사이에 인터페이스 없이 DI를 적용하는 구조](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4710e8a9-d6a5-4168-b66a-dfdaa5393a30/image.png)

UserDao는 JdbcContext에 의존, JdbcContext는 구현 방법이 바뀌진 X ⇒ 사이에 인터페이스 없이 DI를 적용하는 구조

```java
public class JdbcContext {
    private DataSource dataSource;

    //  DataSource 타입 빈을 DI받도록
    public void setDataSource(DataSource dataSource) {  
        this.dataSource = dataSource;
    }

    public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
        Connection c = null;
        PreparedStatement ps = null;

        //try catch finally...
    }
}

public class UserDao {
    // ...
    private JdbcContext jdbcContext;  // JdbcContext를 DI 받을 수 있다.
    public void setJdbcContext(JdbcContext jdbcContext) {
        this.jdbcContext = jdbcContext;             
    }

    public void add(final User user) throws SQLException {
        // 컨텍스트에서 전략 부분을 익명 클래스로 주입 
        this.jdbcContext.workWithStatementStrategy(    
            new StatementStrategy() {...}
        );
    }
}

```

**jdbcContext를 UserDao와 DI 구조로 만드는 이유**

1.  JdbcContext는 싱글톤이 되기에 충분한 조건을 갖추고 있음 → 공유자원으로 활용
2.  JdbcContext가 DI를 통해 다른 빈에 의존중임 : DataSource 객체를 주입받고 있음 DI를 위해서는 주입되는/주입하는 두 오브젝트가 모두 스프링 빈으로 등록될것 : 만족)
3.  두 오브젝트 사이의 실제 의존관계를 설정 파일에 명확히 표시 가능

하지만 DI의 근본적인 원칙에 부합하지 않는다. 즉, 구체적인 클래스 간 관계가 컴파일 단에 노출된다.

### # Way1 빈

-   스프링의 빈 설정은 클래스 레벨이 아닌 런타임시 만들어지는 객체 레벨의 의존관계로 정의
-   UserDao는 JdbcContext에 의존, JdbcContext는 DataSource에 의존하므로 밑의 XML 설정파일 수정

```xml
<bean id="userDao" class="springbook.user.dao.UserDao">
        <property name="jdbcContext" ref="jdbcContext" />
        <!-- dataSource를 안지우는 것은, 아직 다른 메소드들 리팩토링이 덜 끝나서 -->
        <property name="dataSource" ref="dataSource" />
</bean>
<!--추가된 JdbcContext 타입 빈-->
<bean id="jdbcContext" class="springbook.user.dao.JdbcContext">
        <property name="dataSource" ref="dataSource" />
</bean>
<bean id="dataSource" 
    class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
</bean>

```

### # Way2 수동 DI

-   UserDao 내부에서 직접 DI를 적용
-   JdbcContext를 싱글톤으로 만들어 공유하는것은 포기
-   DAO 마다 JdbcContext 클래스 하나를 보유하고 해당 DAO 내부에서 돌려쓰도록 함
-   UserDao가 JdbcContext에 대한 제어권(생성, 초기화(의존성 주입), 사용)을 가지도록 함
-   JdbcContext는 스프링의 빈이 아니니 DI컨테이너를 통해 DI불가 → UserDao가 DI도 해줌

![빈은 userDao, dataSource 두개, userDao 빈에 dataSource 빈을 주입받을것 ](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b120007-1441-40e1-8982-495679a964fb/image.png)

빈은 userDao, dataSource 두개, userDao 빈에 dataSource 빈을 주입받을것

```java
public class UserDao {
    JdbcContext jdbcContext;
    DataSource dataSource;

    // 수정자 메소드이면서 JdbcContext 생성 및 DI 진행
    public void setDataSource(DataSource dataSource) {
        this.jdbcContext = new JdbcContext();       // IoC
        this.jdbcContext.setDataSource(dataSource); // DI
        this.dataSource = dataSource;  // JdbcContext 적용하지 않은 메소드를 위함

    }

    // ...

}

```

**장점**

-   긴밀한 관계(인터페이스X)를 갖는 DAO클래스와 JdbcContext를 빈으로 분리하지 않아도 됨
-   DI는 나름대로 은밀하게 구색을 갖추었으므로 두 클래스 사이 관계가 노출되지 않음

## 템플릿과 콜백

> **템플릿/콜백 패턴 :**  익명 클래스를 통해 재사용할 부분만 따로 분리해 전략 패턴을 구성하는 방식

템플릿 : 목적을 위해 미리 만들어둔 틀. 고정 작업 흐름을 가진 코드를 재사용 (전략 패턴의 context) 콜백 : 다른 오브젝트의 메소드에 전달되는 오브젝트로 값 전달이 아닌 실행이 목적 (익명 내부 클래스 구조)

![image.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7cfcab69-ccca-46be-81b6-2d92b8917879/image.png)

### 템플릿과 콜백의 동작 원리

**일반적인 DI**  : 템플릿에 두고 쓸 인스턴스 변수를 setter로 담아 인스턴스 변수에 가지고 있음

**템플릿/콜백 패턴 (DI의 장점+전략패턴)**

1.  템플릿/콜백 방식에선 필요한 콜백 오브젝트를 매번 새롭게 전달받음. 메소드 레벨에서 일어나는 DI
2.  콜백 오브젝트는 자신을 생성한 클라이언트의 내부 클래스 → 클라이언트 내부의 필드에 직접 참조 OK

> 템플릿 콜백 패턴 동작
> 
> 1.  클라이언트가 템플릿에 콜백을 전달하고,
> 2.  템플릿은 자기 로직 실행 후 콜백을 실행한다.
> 3.  콜백 내부에서 리턴된 값은 템플릿으로 돌아가고,
> 4.  템플릿은 자기 로직 나머지를 수행한 후, 최종 결과를 클라이언트에 전달한다.

### 편리한 콜백의 재활용

**복잡하고 가독성 떨어지는 익명 내부 클래스 사용의 최소화**

-   분리를 통해 재사용할 코드 찾아내기
-   예로 deleteAll() 에선 SQL 문장만 변하므로 여기 부분만 빼내서 분리

**콜백과 템플릿 결합**

-   재사용 가능한 콜백을 담은 메소드를 템플릿 클래스 (DAO끼리 공유 가능) 안으로 이동시키기
-   JdbcContext 내부에 클라이언트, 템플릿, 콜백이 모두 공존하며 동작하는 구조 ⇒ 강한 응집력

### 사칙연산으로 이해하기

<aside> 📎 Calculator 클래스 예제를 발전시켜나가보자

</aside>

**💡 새로운 기능마다 복붙해서 코드를 짜지 않기 위해 개선 ⇒ 템플릿 콜백 패턴 적용 (적용시 고려할 내용)**

-   템플릿에 담을 반복되는 작업 흐름
-   템플릿이 콜백에 전달할 내부 정보와 콜백에 템플릿에 돌려줄 내용
-   템플릿이 작업을 마친 뒤, 클라이언트에게 전달할것

1.  txt 파일을 열어 각 라인을 순차적으로 읽고 더하도록 하는 클래스
2.  (Client) BufferedReader를 열고 닫는 과정의 오류 처리를 위해 try/catch/finally 이용
3.  (Callback) **BufferedReaderCallback** 이라는 인터페이스를 만들고, 그 안에 `Integer doSomethingWithReader(BufferedReader br)` 메소드를 선언
4.  (Template) try/catch/finally 부분을 통째로 빼서 메소드로 분리 (위의 BufferedReaderCallback 사용)

```java
public Integer calcSum(String filepath) throws IOException {
    BufferedReaderCallback sumCallback =
        new BufferedReaderCallback() {
	        @Override
	        public Integer doSomethingWithBufferedReader(BufferedReader br) throws IOException {
            String line = null;
            Integer sum = 0;  // multiply의 경우 Integer multiply =1
            while ((line = br.readLine()) != null) {
                sum += Integer.valueOf(line); // multiply의 경우 multiply*=값
            }
            return sum;
        }
    };
    // fileReadTemplate의 자세한 코드는 생략한다.
    return fileReadTemplate(filepath, sumCallback);
}

```

➕ 곱셈을 구하는 메소드 추가시, sumCallback과 유사하게 곱하는 기능을 담은 콜백을 만들어 사용

🔽 곱셈과 덧셈 콜백을 비교하면 공통 패턴이 등장 ⇒ 라인별 작업을 정의한 콜백 인터페이스 정의

```java
public Integer calcSum(String filepath) throws IOException {
    LineCallback callback = new LineCallback<Integer>() {
        @Override
        public Integer doSomethingWithLine(String line, Integer value) {
            return value + Integer.valueOf(line);
        }
    };
    return lineReadTemplate(filepath, callback, 0); // 곱셈의 경우에는 0 대신 1
}

private Integer lineReadTemplate(String filepath, LineCallback callback, Integer initVal) throws IOException {
    BufferedReader br = null;
    try {
        br = new BufferedReader(new FileReader(filepath));
        T res = initVal;
        String line = null;
        while ((line = br.readLine()) != null) {
            res = callback.doSomethingWithLine(line, res); // 이 한 줄로 콜백이 압축됨
        }
        return res;
    } catch (IOException e) {
        System.out.println(e.getMessage());
        throw e;
    } finally {
        if (br != null) try {
            br.close();
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}

// ➕ 위의 경우 결과의 타입을 고정하지 않고 타입을 다양하기게 하기 위해 제네릭스 이용
private <T> T lineReadTemplate(String filepath, LineCallback<T> callback, T initVal) throws IOException {
}

```

## 스프링의 JdbcTemplate

-   스프링은 JDBC를 이용하는 DAO에서 사용할 수 있도록 다양한 템플릿, 콜백을 제공
-   JdbcTemplate : 스프링이 제공하는 JDBC 코드용 기본 템플릿
    -   update() : 인자로 SQL 문장 전달
        
        ```java
        public void deleteAll() {
            this.jdbcTemplate.update("delete from users");
        }
        
        ```
        
        ```java
        public void add(final User user) throws SQLException {
            this.jdbcTemplate.update("insert into user(id, name, password) values(?, ?, ?)"
                    , user.getId(), user.getName(), user.getPassword());
        }
        
        ```
        
    -   queryForInt()
        
    -   queryForObject()
        
    -   query()
        
-   템플릿은 한번에 여러개의 콜백 사용도 가능하며, 하나의 콜백을 여러번 호출도 가능함
