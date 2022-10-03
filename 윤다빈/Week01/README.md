🔗 1주차 과제 정리 노션 : https://pickle-fireplant-fa1.notion.site/1-e4473c76d6104ad98151bc9407190a4d

# 📓 오브젝트와 의존관계

스프링보단 스프링이 관심을 갖는 오브젝트의 설계및 구현, 동작 원리에 집중할 것

## 1. DAO

<aside> 💡 DAO(Data Access Object): DB로 데이터를 조회하거나 조작하는 기능을 맡도록 한 오브젝트

</aside>

### User

-   사용자 정보 저장시  **자바빈**  규약을 따르는 오브젝트 이용하면 편리
    
    자바빈 ? 현재는 두가지 관례를 따라 만들어진 오브젝트를 가리키며 빈이라고도 부름
    
    -   디폴트 생성자 : 파라미터가 없는 디폴트 생성자를 가질 것
        
        ```
                                    (툴이나 프레임워크에서 리플렉션을 사용해 오브젝트를 만들기 때문)
        
        ```
        
    -   프로퍼티 : 자바빈이 노출하는 이름을 가진 속성을 프로퍼티라고 하며 setter, getter로 수정, 조회
        
-   자바빈 User 클래스 생성 → 클래스와 동일 프로퍼티를 갖는 DB 테이블 생성
    

### UserDao

-   사용자 정보를 DB에 넣고 관리할 수 있는 클래스로 사용자 정보의 등록, 수정, 삭제, 조회등의 기능이 있을것
-   일반적으로 JDBC를 이용하는 작업의 순서
    -   DB 연결을 위한 Connection 가져온뒤, SQL을 담은 Statement 생성 →만들어진 Statement 실행
    -   조회의 경우 SQL 쿼리의 실행 결과를 ResultSet으로 받아 정보를 저장할 객체로 옮김
    -   작업중 생성된 Connection, Statement, ResultSet 등은 작업 마친 후 닫아줌
    -   JDBC API가 만드는 예외를 잡아 직접 처리하거나, 메소드에 throws를 선언해 메소드 밖으로 예외 던짐

## 2. DAO의 분리

-   **필요성**  : 객체 지향의 세계는 모든것이 변화하므로 미래의 변화를 대비하는것이 중요하므로
-   분리와 확장을 고려한 설계 → 변경시 필요한 작업을 최소화하고, 수반하는 문제들이 없도록 함
-   관심사의 분리 : 관심이 같다면 동일 객체 안으로, 다르다면 가능한 떨어져서 영향 주지 않도록 분리

### 커넥션 만들기의 추출 (독립된 메소드로 분리)

**UserDao의 관심 사항**

1.  DB 연결을 위한 커넥션을 가져오는 방법
    
    -   중복 코드 분리 : 코드가 흩어져서 중복되어 나타나 있으므로 변경시 복잡해지니 수행 ⇒ 한가지 관심에 대한 변경이 일어나도 그 관심에 집중되는 코드 부분만 수정 가능
    -   리팩토링&테스트 : 코드 수정 뒤, 기능에 문제 없음을 보장하기 위해 사용
    
    ```java
    public void add(User user) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();
    }
    
    public void get(String id) throws ClassNotFoundException, SQLException {
        Connection c = getConnection();
    }
    
    **// 커넥션을 가져오는 독립적인 메소드를 만들어 중복 코드 삭제 => 메소드 추출 기법** 
    private Connection getConnection() throws ClassNotFoundException, SQLException {
        Class.forName("com.mysql.jdbc.Driver");
        connection c = DriverManager.getConnection(
            "jdbc:mysqlL://localhost/springbook", "spring", "book");
        return c;
    }
    
    ```
    
2.  사용 등록을 위해 DB에 보낼 SQL 문장을 담는 Statement 만든 후 실행
    
3.  작업 종료시, 사용 리소스(Statemnet, Connection 객체)를 닫아 시스템에 돌려주는 것
    

### DB 커넥션 만들기의 독립 (상하위 클래스로 분리)

-   변화에 유연한걸 넘어서 손쉬운 확장을 하도록 만들것
-   변화의 성격마다 분리해서 서로 영향 주지 않고 독립적으로 변경하기 위해 추상 메소드를 사용함
-   **상속을 통한 확장**  - 같은 클래스의 다른 메소드가 아닌 상속을 이용해 서브클래스로 분리(**추상메소드로**)
-   UserDao의 getConnection()를 추상메소드로 만들고, UserDao의 서브클래스에서 메소드 구현
-   템플릿 메소드 패턴과 팩토리 메소드 패턴
-   **문제**  : 상속을 이용하므로 부모-자식 클래스간 긴밀한 결합임 → 부모 수정시 자식 수정해야 할 수 있음 상속을 통해 만들어진 구현 코드가 매 클래스 마다 중복될 경우도 문제

## 3. DAO의 확장

-   객체는 변화하며, 관심사에 따라 분리한 객체들은 변화의 특징이 존재
-   변화의 성격이 다르다 = 변화의 이유, 시기, 주기가 다름

### 클래스의 분리와 인터페이스의 도입 (별도 클래스로 분리)

![image.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/73aa9e4e-68cb-47c5-847e-6c30e2ae1794/image.png)

-   클래스를 분리하면서도  **자유로운 확장**이 가능케하고,  **긴밀한 결합을 하지 않게**  하기 위해 인터페이스 씀
    
-   클래스끼리 긴밀한 연결을 하지 않도록 중간에 추상적인 느슨한 연결고리를 만들때 사용
    
-   인터페이스 : 자신을 구현한 클래스에 대한 구체적 정보는 모두 감춤, 기능만 정의하고 구현방법은 없음
    
-   인터페이스의 메소드 사용시 이를 통해 아는 기능만 쓰고 그것이 어떻게 구현되었는지는 알 필요가 없음
    
-   **문제**  : 초기 한번 어떤 클래스의 오브젝트를 쓸지 결정할때 클래스 이름을 넣어 오브젝트를 생성함
    
    -   원인 : UserDao와 그것을 쓰는 ConnectionMaker의 구현 클래스 사이 관계의 설정 코드가 존재 즉 UserDao 안에 분리되지 않은 다른 관심사항이 존재한다는 것.
    -   클라이언트 객체에 위의 관심사를 분리해서 넣어두기 좋음
    -   UserDao 객체 - 만들어진 ConnectionMaker의 객체 사이 관계 설정 (클래스간 관계설정 아님)
    -   클래스 사이 관계는 코드에 다른 클래스의 이름이 나타나야 형성되지만 오브젝트 사이 관계는 아님
    -   다형성의 특징 : 특정 클래스를 몰라도 그 클래스가 구현한 인터페이스를 썼다면 그 클래스의 오브젝트를 인터페이스 타입으로 받아서 쓸 수 있음
    
    ```java
    public class UserDaoTest {
    
        // 의존관계 주입하는 코드, 런타임 의존관계가 만들어짐
        // 아래에서 생성자를 통해 connectionMaker을 주입받는것
        public UserDao(ConnectionMaker connectionMaker){
            this.connectionMaker = connectionMaker  
        }
    
        public static void main(String[] args) throws ClassNotFoundException, SQLException {
            // 제 3의 객체가 UserDao 객체가 쓸 ConnectionMaker 객체 전달한것       
            ConnectionMaker connectionMaker = new DConnectionMaker();
            // |_ UserDao가 쓸 ConnectionMaker 구현 클래스를 결정 후 객체 생성
            UserDao dao = new UserDao(connectionMaker);
            // |_ UserDao 생성 후 두 오브젝트 사이 의존 관계 설정
    
        }
    }
    
    ```
    

### 원칙과 패턴

-   **객체 지향 개발 5대 원리 (SOLID)**
    
-   시**간이 지나도 유지보수와 확장이 용이한 프로그램을 만들기 위해 적용하는 원리 원칙.**
    
    1.  _**SRP (single responsibility Principle)**_  단일 책임 원칙
        
        : 작성된 클래스는 기능 하나만 가짐, 클래스가 제공하는 모든 서비스는  **책임**  하나를 수행하는데 집중.
        
        클래스의 이름은 수행하는 책임을 표현하는것이 좋음
        
    2.  _**OCP (Open Close Principle)**_  개방 폐쇠 원칙 객체지향의 장점 극대화
        
        : 소프트웨어의 구성요소(클래스, 컴포넌트, 모듈, 함수)는 확장에 열려있고, 변경에 닫혀있을 것.
        
        인터페이스는 가능한 변경X. 인터페이스 정의시, 여러 경우의 수에 대한 적절한 고려와 예측 필수
        
        인터페이스 설계시 적절한 추상화 레벨 선택 필수 (추상화 : 다른 모든 객체와 구분되는 본질적 특징)
        
    3.  _**LSP (the Liskov Substitution Principle)**_  리스코브 치환 원칙
        
        : 서브타입은 언제나 기반타입으로 교체될 수 있을 것. (서브 타입은 기반타입의 규약을 지킬것)
        
    4.  _**ISP (Interface Segregation Principle)**_  인터페이스 분리 원칙
        
        : 클래스는 자신이 수행하지 않을 인터페이스는 구현하지 않을 것.
        
        즉, 일반 인터페이스 1개 보단 여러개의 구체화 된 인터페이스가 나음
        
    5.  _**DIP (Dependency Inversion Principle)**_  의존성 역전 원칙
        
        : 의존 관계 역전 (= 구조적 디자인에서 발생하던 하위 레벨 모듈 변경이 상위 레벨 모듈 변경을 요구하는 위계 관계 끊는 것)
        
        실 사용 관계는 변하지 않으며, 추상을 매개로 메세지를 주고 받음으로써 관계를 최대한 느슨히 함
        

## 4. IoC (제어의 역전)과 스프링의 IoC

### 오브젝트 팩토리와 활용

-   분리할 기능 담당의 클래스에서 객체의 생성 방법을 고른뒤 오브젝트를 리턴하고 이 오브젝트가 팩토리임
    
-   팩토리 사용 목적 : 오브젝트를 생성하는 쪽과 오브젝트를 사용하는 쪽의 책임과 역할을 분리하기 위해 씀
    
-   **오브젝트 팩토리 소개를 위해 UserDaoTest를 분리하는 코드**
    
    ```java
    // 기존에 UserDaoTest에 있던 UserDao, CoonnectionMaker 관련 생성 작업 수행
    public class DaoFactory{
        public UserDao userDao(){
            ConnectionMaker connectionMaker = new DConnectionMaker();
            UserDao userDao = new UserDao(connectionMaker);
            return userDao;
        }
    }
    
    // 활용 VER. DaoFactory애 다른 DAO 생성 기능이 들어간 상태. connectionMaker의 중복을 피해야할것
    public class DaoFacotry{
        pubic UserDao userDao(){
             return new UserDao(ConnectionMaker());
        }
        pubic AccountDao accountDao(){
             return new AccountDao(ConnectionMaker());
        }
        pubic MessageDao messageDao(){
             return new MessageDao(ConnectionMaker());
        }
        pubic ConnectionMaker connectionMaker(){
            return new DConnectionMaker();
        }
    }
    
    // 미리 만들어진 UserDao 오브젝트 가져와서 사용
    public class userDaoTest{
        public static void main(String[] args) throws classNotFounException, SQLException{
            UserDao dao = new DaoFactory().userDao();
        }
    }
    
    ```
    
    -   분리된 오브젝트들의 역할과 관계
        -   팩토리는 어플리케이션의 오브젝트 구성 및 그들의 관계를 정의하는 설계도 역할을 함
        -   핵심 기술인 UserDao 변경 필요 없음 DaoFactory의 코드 변경만으로도 수정가능
        -   제어권 UserDao → DaoFactory 하면서 UserDao는 팩토리에 의해 수동적으로 만들어지며며며자신이 쓸 오브젝트도 직접 만드는게 아니라 DaoFactory가 공급하는걸 사용해야함 (**제어의 역전**)
-   의의 : 애플리케이션 컴포넌트 역할의 오브젝트와 애플리케이션 구조를 결정하는  **오브젝트의 분리**
    
    ![image.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3c997293-53cc-45b0-856e-aebf37c573e3/image.png)
    

### 제어권의 이전을 통한 제어관계 역전

-   일반 프로그램 흐름 : 프로그램 시작에서 쓸 오브젝트 결정 및 생성후 안의 메소드 호출 및 다음 결정, 호출
-   모든 오브젝트가 능동적으로 자신이 쓸 클래스를 정하고, 언제 그 오브젝트를 생성할지 스스로 관리
-   제어의 역전 : 객체가 자신이 쓸 객체를 스스로 고르고 만들지 않음. 다른 대상에 제어권한 위임하기 때문
-   제어 권한을 상위 템플릿 메소드에 넘기고 자신은 필요할때 호출되서 쓰여지는 것
-   자기 자신은 메소드가 언제 어떻게 사용될 지 알 수 없음. 슈퍼클래스에서 필요할때 사용하는 것
-   라이브러리와 프레임워크의 차이
    -   라이브러리쓰는 어플 코드는 흐름 직접 제어, 동작중 필요 기능이 있을때 능동적으로 라이브러리 씀
    -   프레임워크가 흐름을 주도하는 중 개발자의 어플 코드를 가져와서 사용하는 방식 (역전 개념 사용)
-   IoC 적용 장점 : 깔끔한 설계와, 유연성, 확장성이 증가

### 오브젝트 팩토리를 사용한 스프링 IoC

-   **빈 (Bean)**  : 스프링이 제어권을 가지고 직접 만들고 관계를 설정하는  **오브젝트 (=IoC 적용됨**)
    
-   **스프링 빈**  : 스프링 컨테이너가 생성, 관계설정, 사용등을 제어해주는 오브젝트들
    
-   **빈 팩토리**  : 스프링에서 빈의 생성과 관계설정등의 제어 담당하는 오브젝트 (IoC 담당 핵심 컨테이너)
    
-   DaoFactory를 사용하는  **어플리케이션 컨텍스트**
    
    -   DaoFactory를 스프링의 빈 팩토리가 쓸 수 있는 설정정보로 만들것
    -   @Configuration : 스프링이 빈 팩토리를 위해 쓸 오브젝트의 설정 담당 클래스라고 인식시킴
    -   @Bean : 오브젝트를 만드는 IoC용 메소드임을 나타냄
    -   getBean(빈의 이름,) : ApplicationContext가 관리하는 오브젝트를 요청하는 메소드
    
    ```java
    @Configuration
    public class DaoFactory{
        @Bean
        public UserDao ***userDao***(){
            return new UserDao(connectionMaker());
        }
    
        @Bean
        public ConnectionMaker ***connectionMaker***(){
            return new DConnectionMaker();
        }
    }
    
    // DaoFactory를 설정정보로 쓰는 어플리케이션 컨텍스트 생성 (ApplicationContext 타입 오브젝트임)
    puvlic class UserDaoTest{
        public static void main(String[] args) throws ClassNotFoundException, SQLException {
            ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
            UserDao dao = context.getBean("***userDao***", UserDao.class);
        }
    }
    
    ```
    

### 어플리케이션 컨텍스트의 작동방식

![image.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/30168913-ff62-4f02-99a2-8f907093671e/image.png)

-   어플리케이션 컨텍스트 : 빈팩토리를 확장한 IoC 컨테이너. (어플리케이션 전체 구성요소 제어)
    
-   별도의 정보를 참고해 빈을 만들고 관계설정하는 작업 전체를 제어 (빈팩토리와 용어 혼용 사용)
    
-   앞의 팩토리에서 어플리케이션의 로직부분과 설계도 역할 하는 부분으로 구분했는데 설계도부분임
    
-   **어플리케이션 컨텍스트 = IoC 컨테이너 = 스프링 컨테이너 = 빈 팩토리의 일종 = (대표예가) 스프링**
    
-   오브젝트 팩토리를 이용한 방식 VS 스프링의 어플리케이션 컨텍스트 사용 방식
    
    -   오브젝트 팩토리와 달리 어플리케이션에서 제어의 역전을 써 관리하는 모든 오브젝트 생성 및 관리
    -   어플리케이션 컨텍스트는 직접 오브젝트 생성, 관계 설정 코드가 없고 별도의  _생성정보_를 통해 얻음
    -   @Configurationㅇ 붙은 DaoFactory : 어플리케이션 컨텍스트가 쓰는  _생성정보_
-   사용 장점
    
    -   클라이언트는 구체적 팩토리 클래스 몰라도 됨 (필요할때마다 오브젝트 팩토리 생성 필요 X)
    -   종합 IoC 서비스 제공
    -   다양하게 빈 검색 (빈의 이름이나 타입만으로 혹은 특별 어노테이션이 붙어있는 빈 검색 가능)

### 스프링 IoC 용어 정리

-   설정정보/설정메타 정보 : 어플리케이션 컨텍스트가 IoC 적용을 위해 쓰는 메타 정보 (blueprint)
-   IoC 컨테이너 : 컨테이너 자체가 IoC 개념을 담으며 어플리케이션 컨텍스트 = (스프링) 컨테이너 어플리케이션 컨텍스트 그자체로 ApplicaationConext 인터페이스를 구현한 오브젝트 의미
-   스프링 (프레임워크) : IoC 컨테이너, 어플리케이션 컨텍스트를 포함, 스프링이 제공하는 모든 기능 의미

## 5. 싱글톤 레지스트리와 오브젝트 스코프

-   DaoFactory의 직접적 사용 VS  **@Configuration**  으로 어플리케이션 컨텍스 사용
    
    : userDao() 여러번 호출시 전자는 매번 새 오브젝트가 생성,
    
    ```
     후자의 경우 getBean() 으로 userDao 라는 이름의 **오브젝트 여러번 가져올 경우 동일 오브젝트 가져옴**
    
    ```
    

### 싱글톤 레지스트리로서의 어플리케이션 컨텍스트

-   오브젝트 팩토리와 동작 방식이 유사 또한  **싱글톤을 저장 및 관리하는 싱글톤 레지스트리**

✅ 스프링은 기본적으로 내부에서 생성하는 빈 오브젝트를 모두 싱글톤으로 만듦

-   싱글톤으로 빈을 생성하는 이유 : 스프링은 대규모 엔터프라이즈 서버 환경에서 사용 ⇒ 요청마다 새 오브젝트를 만들어 로직 담당 ☹️
    
-   서버의 싱글톤(권장) : 서블릿은 클래스당 오브젝트 하나만 만들고, 요청 담당 스레드에서 이것을 공유
    
-   싱글톤 패턴의 한계 🙁 - 자바의 싱글톤 패턴 구현 방식은 추천하지 않음
    
    -   생성자를 private으로 제한하여 싱글톤 클래스 자신만 오브젝트를 만듦 ⇒ 상속 불가 (다형성 X)
    -   싱글톤은 테스트가 어렵거나 불가 : 테스트시 테스트용 오브젝트로 대체 불가
    -   서버환경에서 클래스 로더 구성에 따라 싱글톤이 보장되지 않을 수 있음
    -   싱글톤을 쓰는 클라인언트가 정해진 게 아니므로 전역상태로 쓰기 쉬워짐 이는 권장되지 않음
-   싱글톤 레지스트리 : 스프링이 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공
    
-   스프링 컨테이너는 싱글톤 관리 컨테이너
    
-   장점 : 평범한 자바 클래스도 제어권을 컨테이너에 넘겨 싱글톤으로 활용해줌
    
-   싱글톤 패턴과 달리 스프링이 권장하는 객체지향적 설계, 디자인 패턴을 적용하는데 어려움이 없음
    

<aside> 💡 스프링 = loC(DI) 컨테이너 = 싱글톤 레지스트리 (싱글톤 패턴 대신해 싱글톤을 만들고 관리)

</aside>

### 싱글톤과 오브젝트의 상태

-   상태 관리의 중요성 : 멀티 스레드 환경의 경우 여러 스레드 동시 접근 가능
-   싱글톤은 기본적으로 상태 유지 방식으로 만들 X,  **무상태 방식**으로 만듦 (상태정부 내부에 없음)
-   **방법**  : 메소드 파라미터나, 메소드 안의 로컬변수, 리턴값 사용 (매번 새로운 공간이 만들어지므로)

```java
public class UserDao {
    private ConnectionMaker connectionMaker; // 초기 설정후 바뀌지 않는 읽기전용 인스턴스변수
    // 매번 새로운 값으로 바뀌는 정보를 담은 인스턴스 변수 🚨 여기는 문제
    private Connection c;
    private User user;

    public User get(String id) throws ~~Exception {
        this.c = connectionMaker.makeConnection();
        ...
        this.user = new User();
        this.user.setId(rs.getString("id"));
        this.user.setName(rs.getString("name"));
        this.user.setPassword(rs.getString("password"));
        ...
        return this.user;
    }
}

==> 자신이 쓰는 싱글톤 객체 저장하는 용도이거나 읽기 전용의 정보는 인스턴스 변수 가능 
==> 그 외의 경우는 멀티 스레드 환경위에서 큰 문제 발생

```

### 스프링 빈의 스코프

-   빈의 스코프 : 빈이 생성, 존재, 적용되는 범위로 스프링 빈의 기본 스코프는 싱글톤
-   싱글톤 스코프 : 컨테이너 내 한개 오브젝트만 생성되며, 스프링 컨테이너가 존재하는 한 유지
-   싱글톤 외의 스코프 : 프로토타입 스코프, 요청 스코프, 세션 스코프등

## 6. 의존관계 주입 (DI)

<aside> 💡  **DI : IoC의 정의가 매우 폭넓기 때문에 스프링이 제공하는 IoC 방식이라는 새로운 용어 생성**

</aside>

### 런타임 의존관계 설명

-   의존관계 : A→B 형태로 의존관계일 경우, B의 변화가 A에 영향을 미치는 것 (방향성이 있어야함)
    
-   UserDao의 의존 관계
    
    ![image.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/20469e90-597c-46bf-ba51-cbcace3cafac/image.png)
    
    -   UserDao → ConnectionMaker : UserDao가 ConnectionMaker 인터페이스에 영향을 받음
    -   인터페이스에 대해서만 의존관계를 가지면 DConnectionMaker과의 관계X ⇒ 느슨한 관계 😁
    -   위는 모델의 UML에서 말하는 의존관계이고, 이와 달리 런타임시 오브젝트 사이 생성되는 관계O
    -   런타임(오브젝트) 의존관계 : 설계 시점의 의존관계가 구현된것
-   의존관계 주입의 조건
    
    -   인터페이스에만 의존하므로 런타임 시점의 의존관계가 클래스 모델 혹은 코드에서 볼 수 없음
    -   런타임 시점 의존관계는 컨테이너나 팩토리등  **제3의 존재**가 결정
    -   의존관계는 사용할 오브젝트에 대한 참조를 외부에서 주입해서 만들어짐
-   **제 3의 존재 (DI컨테이너)**  : 분리해서 만들어진 관계설정 책임을 가진 코드가 있는 오브젝트
    

### 의존관계 검색 및 주입 (주입을 권장)

-   스프링이 제공하는 IoC 방법중 하나로 의존관계 주입과 달리 스스로 검색해 의존관계를 맺음
-   getBean() 이라는 메소드가 의존관계 검색에 이용
-   의존관게 검색만 가능할 때 : 맨 처음 기동 시점에서 오브젝트를 가져올 경우 사용. DI주입은 불가
-   의존관계 검색 : 검색하는 객체는 스프링 빈일 필요 없음 (new UserDao()로 만들고 써도 ㄱㅊ)
-   의존관계 주입 : UserDao와 ConnectionMaker 모두 컨테이너가 만드는 빈 객체여야함
-   🚨 **DI를 원하는 객체는 우선 컨테이너가 관리하는 빈이 되어야 의존관계 주입이 가능**
-   🚨 의존관계 주입의 경우  **인터페이스 타입의 파라미터**로 이뤄져야 함 (특정 클래스로 고정 안돼)

### 의존관계 주입 응용

-   DI 장점 : 다른 의존관계의 대상이 바뀌더라도 영향X
-   기능 구현 교환 (예. connectionMaker 사용시 로컬 DB 사용하다 운영서버에서 DB 연결하도록 변경)

```java
@Bean
public ConnectionMaker connectionMaker(){
    return new LocalDBConenctionMaker();
}

@Bean 
public ConnectionMaker connectionMaker(){
    return new ProductionDBMaker();
}

@@@ 위에서 아래로만 바꾸 면됨 즉, DI 설정정보인 DaoFactory만 다르게 만들면 코드 수정 없이 해결 

```

-   DAO의 DB 연결 횟수 파악 기능 구현
    
    -   DAO의 수정 아니라 관심사항을 분리해 DAO와 DB연결을 하는 객체 사이 연결횟수 세는 객체 추가
        
    -   CountingConnectionMaker 클래스는 직접 DB 연결을 하진 않지만 연결횟수 카운터를 증가시킴
        
    -   즉 UserDao → CountingConnectionMaker →  **DConnectionMaker**  의 의존관계 가짐
        
        ![image.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d8677aa2-a454-4ddb-90f7-9bd75df2e526/image.png)
        
    
    ```java
    public class CountingConnectionMaker implements ConnectionMaker {
        int counter = 0;
        private ConnectionMaker realConnectionMaker;
    
        public CountingConnectionMaker(**ConnectionMaker realConnectionMaker**){
            this.realConnectionMaker = realConnectionMaker;
        }
    
        public Connection makeConnection() throws ~~Exception {
            this.counter++;
            return realConnectionMaker.makeConnection();
        }
    
        public int getCounter() { return this.counter; }
    }
    
    ```
    
    ```java
    // 의존관계주입 설정용 클래스
    @Configuration
    public class CountingDaoFactory {
    
        @Bean
        public UserDao userDao(){
            return new UserDao(ConnectionMaker));
        }
    
        @Bean
        public ConnectionMaker connectionMaker() {
            return new CountingConnectionMaker(realConnectionMaker())
        }
    
        @Bean
        public ConnectionMaker RealConnectionMaker(){ //여기서 만든걸 connectionMaker에서 DI함
            return new DConnectionMaker(); //실제 DB연결
        }
    } 
    
    ```
    
    ```java
    // 커넥션 카운팅을 위한 코드로 DAO를 의존관계 검색으로 가져와서 실행하고 확인
    public class UserDaoConnectionCountingTest{
        public static void main(Stirng[] args) throws ~~Exception {
            AnnotationConfigApplicationContext context = 
                new AnnotationConfigApplicationContext(CountingDaoFactory.class);
            UserDao dao = context.getBean("userDao", UserDao.class);
        }
    
        // DAO 사용 코드 ~~
        CountingConnectionMaker ccm = context.getBean("connectionMaker", CountingConnectionMaker.class);
        System.out.println("connection counter : "+ ccm.getCounter());
    }
    
    ```
    
    -   DB 연결을 세는 작업이 모두 끝나면 CountingDaoFactory→DaoFactory 변경 혹은 ConnectionMaker() 메소드 수정하면 런타임 의존관계가 이전으로 돌아감

🚨 DL(의존관계 검색)을 이용하면 어떤 빈이든 가져올 수 있음 (getBean 사용)

### 메소드를 통한 의존관계 주입

-   생성자를 통한 의존관계 주입이 아닌 메소드를 이용한 DI가 더 보편적
-   수정자 메소드 이용한 주입
    -   Setter 메소드 : 인자로 받은 값을 내부 인스턴스 변수에 저장 후 내부 메소드에서 쓰므로 DI에 이용
    -   단 수정자 메소드는 set으로 시작하며 한번에 인자를 하나만 가질 수 있음
    -   DaoFactory 같은 코드 대신 XML을 쓸 경우 매우 편리
    -   메소드의 이름을 의미있고 단순하게 짓는것이 중요 Set(인터페이스이름) 라 짓는것이 관례
-   일반 메소드를 이용한 주입
    -   한번에 여러개 인자를 받는 경우 유리하지만 인자갯수가 늘어날수록 실수 확률이 올라감

### ➕Java에서 의존관계 주입하는 방법

-   **생성자 주입 방식 (스프링 추천 방식)**  _아래의 내용._
    
-   필드 주입 방식
    
-   수정자 주입 방식 (setter)
    

객체들이 의존하는 Coffee 는 인터페이스고 twosome과 starbucks를 구현체로 가지고 있음

```java
@Component
public class Starbucks implements Coffee{

	...
}

@Component
public class Twosome implements Coffee{

	...
}

@Component
public class Person {

    private Coffee coffee;

    @Autowired
    public Person(Coffee coffee){
        this.coffee = coffee;
    }

    ...
    
}

👉🏻 spring bean : Person, Starbucks, Twosome
👉🏻 의존관계 주입 : DI 컨테이너가, @Autowired가 붙은 곳에 주입 할 **또 다른 spring bean** 주입

```

-   **DI 컨테이너**는 애플리케이션이 시작될 때, **@Component** 어노테이션이 붙은 Class들을 **스프링 빈**  등록
-   스프링 빈으로 등록된 클래스들이 필요한 곳(**= @Autowired**이 붙은 곳)에 의존관계를 주입.
-   즉, Starbucks, Twosome중 어느 것이든 @Autowired가 붙은 곳에 Coffee 타입으로 주입 될 것.
-   뭘 주입할지 몰라 에러 나지만 해결 방법 존재 : @Primary, @Qualifier, 필드명/파라미터명 의존관계 주입

😇 **스프링빈은 DI 컨테이너가 관리하는(=등록된) 자바 객체**

## 7. XML을 이용한 설정

-   DaoFactory로 의존관계 주입 정보를 제공하는게 번거로워지면서 의존관계 설정정보를 XML을 통해 제작
-   XML 장점 : 텍스트파일로 다루기 쉬우며 이해하기도 쉬움, 별도의 빌드작업 필요 없음

### XML 설정

-   **<beans> (@Configuration) 안에 여러 <bean> (@Bean) 의 정의**가 가능
    
-   XML 설정은 @Configuration, @Bean 이 붙은 자바 클래스로 만든 설정과 동일한 내용
    
-   빈 이름 (@Bean의 메소드명), 빈 클래스 (빈 객체를 어떤 클래스를 통해 만들지 정의)
    
-   빈의 의존객체 : 빈의 생성자 혹은 수정자 메소드로 의존 객체를 넣어줌
    
-   XML에서 <bean>을 써도 빈의 이름, 빈의 클래스, 빈의 의존객체 즉 빈의 DI정보 정의 가능
    
-   **connectionMaker()의 전환**
    
    -   의존하는 다른 객체는 없으니 빈의 이름과 빈의 클래스만 정의하면 됨
    -   클래스 속성에 지정하는건 자바 메소드에서 객체를 만들때 쓰는 클래스 이름을 써야함
    -   또한 class 속성에 넣는 클래스명은 패키지까지 모두 포함해야함
    
    ```java
    @Bean                                       <bean
    public connectionMaker connectionMaker(){    id = "connectionMaker"
        return new DConnectionMaker();           class="springbook...DConnectionMaker"
    }                                           />
    
    
    ```
    
-   **UserDao()의 전환**
    
    -   이 경우 세가지 요소가 모두 들어있으며 의존관계는 수정자 메소드를 통해 주입하는데 여기 주의
    -   의존관계 정의는 <property> 태그를 사용하며 name과 ref 라는 두가지 속성을 지님
    -   name(속성의 이름, 수정자 메소드를 알 수 있음), ref(수정자 메소드로 주입해줄 객체의 빈 이름)
    -   `UserDao.setConnectionMaker(connectionMaker())`  에서 setConnectionMaker는 connectionMaker 속성을 통해 의존관계 정보를 주입한다는 의미
    
    ```java
    <bean id = "userDao" class="springbook.dao.UserDao">
         <property name="connectionMaker" ref="connectionMaker"/>
    </bean>
    
    ```
    

### XML의 의존관계 주입 정보

-   name 속성 : DI에 쓸 수정자 메소드의 프로퍼티명 (주입할 빈 객체의 인터페이스명을 주로 따름)
-   ref 속성 : 주입할 객체를 정의한 빈의 ID (보통 인터페이스 이름을 사용하지만 다른 이름도 괜찮)
-   보통 name 속성과 ref 값이 같은 경우 많음

```java
// ConnectionMaker 빈을 myConnectionMaker라고 변경했을 경우
<beans>
    // 같은 인터페이스 타입의 빈을 여러개 정의한 경우
    <bean id="myConnectionMaker" class="springbook.user.dao.DConnectionMaker"/>
    <bean id="localConnectionMaker" class ="springbook.user.dao.LocalDBConnectionMaker"/>
    <bean id="userDao" class="springbook.user.dao.UserDao">
        <property name= "connectionMaker" ref="myConnectionMaker"/>
    </bean>
</beans>

```

### XML을 쓰는 어플리케이션 컨텍스트

-   어플리케이션 컨텍스트도 팩토리 대신 XML을 써서 설정정보를 지정하도록
-   GenericXmlApplicationContext 이용하여 IoC/DI 작업 수행
-   관례적으로 파일명은 applicaationContext.xml임
-   위의 xml 파일 생성뒤 생성자에 해당 파일을 넣고 new GenericXmlApplication(“~~”)

### DataSource 인터페이스 변환

-   DB 연결을 가져오는 객체의 기능을 추상화해서 만들어둔 자바의 인터페이스
    
-   DataSsource를 구현해 클래스를 만들일은 거의 없고 이미 구현된 클래스를 가져와서 활용함
    
-   DataSource를 사용하는 UserDao
    
    -   DataSource의 구현 클래스가 필요하므로 SimpleDriverDataSource를 사용할 것
    
    ```java
    import javax.sql.DataSource;
    
    public class UserDao {
        private DataSource dataSource;
    
        public void setDataSource(DataSource dataSource){
           this.dataSource = dataSource;
        }
    
        public void add(User user) throws SQLException{
            Connection c. dataSource.getConnection();
            ...
        }
        ...
    }
    
    ```
    
    -   자바코드 설정 방식
    
    ```java
    @Bean
    public DataSource dataSource() {
        SimpleDriverDataSouce dataSource = new SimpleDriverDataSource();
        
        dataSouce. setDriverClass, setUrl, serUsername, serPassword 등 설정
        DB 연결정보를 set메소드로 넣어 데이터베이스 연결 방식 변경
    }
    
    @Bean
    public UserDao userDao(){
        UserDao userDao = new UserDao();
        userDao.setDataSource(dataSouce());
        return userDao;
    }
    
    ```
    
    -   XML 설정 방식
    
    ```java
    connectionMaker인 빈을 없애고 dataSource라는 이름의 빈을 등록
    <bean id="dataSource"
        class="org.springframework.jdbc.datasource.SimpleDriverDataSource"/>
    
    ```
    
    -   다른빈에 의존할 경우, <property name=“” ref=“의존할 빈 이름”> 으로 설정해주면 됨
    -   스프링의 빈으로 등록될 클래스에 수정자 메소드가 정의되어있다면 ref 대신 value=“” 사용
    -   Value = “com.mysql.jdbc.Driver”의 경우 성공하는 이유
    -   스프링이 프로퍼티 값을 set메소드의 이자 타입을 참고해 알아서 변환해주기 때문
