## 목차
1. [초난감 DAO](#초난감-dao)
2. [DAO의 분리](#dao의-분리)
3. [DAO의 확장](#dao의-확장)
4. [원칙과 패턴](#원칙과-패턴)
5. [제어의 역전(IoC)](제어의-역전ioc)
6. [싱글톤 레지스트리와 오브젝트 스코프](#싱글톤-레지스트리와-오브젝트-스코프)
7. [XML을 이용한 설정](#xml을-이용한-설정)

- 1장 분량이 많아 전체 내용 요약보다는 개인적으로 메모하고 싶은 부분만 정리했습니다.
- 중간 중간 추가 공부한 부분은 개인 블로그 게시글 링크를 달았습니다.
<br>

---

<br>

## 초난감 DAO

### DAO (Data Access Object)

DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트

<br>

### 자바빈 (JavaBean, 빈)

다음 두 가지 관례를 따라 만들어진 오브젝트를 가리킨다.

- 디폴트 생성자
    - 툴이나 프레임워크에서 리플렉션을 이용해 오브젝트를 생성하기 때문에 필요하다.
- 프로퍼티
    - 수정자 메소드(setter)와 접근자 메소드(getter)를 이용해 수정/조회할 수 있다.

<br>

### Ex) UserDAO

사용자 정보를 DB에 넣고 관리할 수 있는 DAO 클래스를 만들어보자.

- UserDAO 코드

```java
public class UserDao {
	public void add(User user) throws ... {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection("jdbc:mysql://localhost/springbook", "spring", "book");
		
		PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());
	
		ps.executeUpdate();
	
		ps.close();
		c.close();
	}

	public User get(String id) throws ... {
		...
	}
}
```

<br>

### main()을 사용한 테스트 코드

⇒ 테스트는 통과했으나 UserDao 코는 사실 문제가 많다. Why?

<br>
<br>

## DAO의 분리

### 관심사의 분리

- 객체 지향의 세계에서는 모든 것이 변한다.
- 그래서 `분리`와 `확장`을 고려한 설계가 필요하다.

- `분리` ?
  - 변화는 대체로 한 가지 관심에 대해 일어나지만 그에 따른 작업은 한 곳에 집중되지 않는 경우가 많다.  
  - 한 가지 관심이 한 군데에 집중되어야 한다.

<br>
    
- **관심사의 분리** 
  <br>
  관심이 같은 것끼리는 한 객체 안으로 또는 친한 객체로 모이게 하고, 관심이 다른 것은 가능한 한 따로 떨어져서 서로 영향을 주지 않도록 분리한다.

<br>
<br>

### 커넥션 만들기의 추출
add() 메소드 하나에서 적어도 세 가지 관심사항을 발견할 수 있다.

1. DB와 연결을 위한 커넥션을 가져오는 것
2. DB에 보낼 SQL 문장을 담을 Statement를 만들고 실행하는 것
3. 작업이 끝나면 사용한 리소스를 닫아주는 것

<br>
<br>

### 중복 코드의 메소드 추출
중복된 DB 연결 코드를 getConnection()이라는 독립적인 메소드로 만들자.

```java
public void add(User user) throws ... {
	Connection c = getConnection();
	...
}

private Connection getConnection() throws ... {
		Class.forName("com.mysql.jdbc.Driver");
		return DriverManager.getConnection("jdbc:mysql://localhost/springbook", "spring", "book");
}
```

<br>
<br>


### 변경사항에 대한 검증: 리팩토링과 테스트

- 코드를 수정한 후에 기능에 문제가 없다는 게 보장되지 않는다. ⇒ 앞서 만든 main() 메소드 테스트를 다시 실행해본다.

<br>

- 방금 사용한 리팩토링 기법은 **[메소드 추출 기법](https://aeliketodo.tistory.com/34)**
    - 공통 기능을 담당하는 메소드로 중복 코드를 뽑아낸다.

<br>
<br>


### DB 커넥션 만들기의 독립

- UserDao를 사용하는 업체들이 각기 다른 종류의 DB를 사용하고 각 DB마다 연결 방법이 다르다고 한다.
- UserDao 소스코드를 업체들에게 제공하지 않고도 업체 스스로 원하는 DB 커넥션 생성 방식을 적용할 수 있도록 개발하고 싶다.

<br>
<br>

### 상속을 통한 확장

- getConnection 메소드를 추상 메소드로 만든다.
- 각 고객들은 UserDao 클래스를 상속하는 서브클래스를 만들어 getConnection 메소드를 구현한다.

<br>

```java
public class UserDao {
	public abstract Connection getConnection() throws ... ;

	public void add(User user) throws ... {	
		PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());
	
		ps.executeUpdate();
	
		ps.close();
		c.close();
	}

	public User get(String id) throws ... {
		...
	}
}
```

⇒ 이제 UserDao는 1.변경도 용이하고 2.손쉽게 확장된다.

<br>

- [템플릿 메소드 패턴](https://aeliketodo.tistory.com/99)
    - 슈퍼클래스에 기본적인 로직의 흐름을 만든다.
    - 그 기능의 일부를 추상 메소드나 protected 메소드 등으로 만든 뒤 서브클래스에서 필요에 맞게 구현해 사용한다.

- [팩토리 메소드 패턴](https://aeliketodo.tistory.com/92)
    - 서브클래스에서 구체적인 오브젝트 생성 방법을 결정하도록 한다.

<br>

- 그러나 상속은 **많은 한계점**이 있다.
    - 이미 UserDao가 다른 목적으로 상속을 사용하고 있다면? → 자바는 다중 상속을 지원하지 않는다.
    - 상속을 통한 상하위 클래스 관계는 생각보다 밀접하다. → 슈퍼 클래스 변경 시 서브 클래스도 수정해야 한다.
    - DB 커넥션 생성 코드를 다른 DAO 클래스에 적용할 수 없다. → 중복 코드가 발생한다.

<br>
<br>

## DAO의 확장

### 클래스의 분리

관심사가 다르고 변화의 성격, 주기가 다른 두 가지 코드를 아예 독립적인 클래스로 분리해보자.

```java
public class UserDao {
	private SimpleConnectionMaker simpleConnectionMaker;

	public UserDao() {
		simpleConnectionMaker = new SimpleConnectionMaker();
	}

	public void add(User user) throws ... {
		Connection c = simpleConnectionMaker.makeNewConnection();
	}

	...
}
```

```java
public class SimpleConnectionMaker {
	public Connection makeNewConnection() throws ... {
		Class.forName("com.mysql.jdbc.Driver");
		return DriverManager.getConnection("jdbc:mysql://localhost/springbook", "spring", "book");
	}
}
```

<br>

- UserDao 코드가 SimpleConnectionMaker라는 특정 클래스에 종속되었다.
⇒ 고객에게 UserDao 클래스만 공급하고 상속을 통해 기능을 확장해 사용했던게 불가능해졌다.

<br>
<br>

### 인터페이스의 도입

- 클래스를 분리하면서도 두 클래스가 서로 긴밀하게 연결되어 있지 않도록 추상화한다. == 인터페이스

- ConnectionMaker를 **인터페이스**로 정의하면,
    - UserDao는 자신이 사용할 클래스가 어떤 것인지 몰라도 된다.
    - UserDao는 인터페이스의 메소드만 관심을 가질 뿐, 메소드 구현 방법은 몰라도 된다.

<br>

```java
public interface SimpleConnectionMaker {
	public Connection makeNewConnection() throws ... ;
}

public class DConnectionMaker implements Connection Maker {
	public Connection makeNewConnection() throws ... {
		// D 사의 Connection 생성 코드
	}
}
```

```java
public class UserDao {
	private ConnectionMaker connectionMaker;

	public UserDao() {
		connectionMaker = new DConnectionMaker(); // 클래스 이름이 나타남
	}

	...
}
```

⇒ 아직 UserDao의 생성자 메소드를 직접 수정하지 않고는 ConnectionMaker 구현 클래스를 자유롭게 확장할 수 없다.

<br>
<br>

### 관계설정 책임의 분리

- UserDao의 클라이언트가 직접 UserDao에서 사용할 ConnectionMaker 구현 클래스를 결정하도록 코드를 수정해보자.
- UserDao의 생성자 파라미터를 인터페이스로 선언해, 해당 인터페이스의 구현 클래스라면 모두 사용할 수 있도록 한다.

 ⇒ 즉 UserDao와 ConnectionMaker는 자신이 맡은 역할만 담당하고, 의존 관계 주입 역할을 분리해 클라이언트에게 떠넘긴다.

<br>

```java
public class UserDao {
	private ConnectionMaker connectionMaker;

	public UserDao(ConnectionMaker connectionMaker) {
		this.connectionMaker = connectionMaker; // 클래스 이름이 나타나지 않음
	}

	...
}
```

<br>

- main() 메소드

```java
// 클라이언트가 사용할 ConnectionMaker 구현 클래스 생성
ConnectionMaker connectionMaker = new DConnectionMaker();

// UserDao와 ConnectionMaker 두 오브젝트 사이의 의존관계 설정
UserDao dao = new UserDao(connectionMaker);
```
⇒ UserDao는 자신의 관심사인 SQL 생성과 실행에만 집중

<br>

<결론>
- 인터페이스 도입 > 상속
- 다른 DAO 클래스에서도 의존관계 주입을 통해 ConnectionMaker 기능을 사용할 수 있게 되었다.

<br>
<br>

## 원칙과 패턴

### 개방 폐쇄 원칙

- OCP, Open-Closed Principle
- **클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다.**

<br>

ex) UserDao 클래스
- DB 연결 방법이라는 기능 확장에 열려있다.
- UserDao의 핵심 기능을 구현한 add, get 코드는 변화에 영향을 받지 않기에 닫혀 있다.

<br>

- [SOLID 원칙(객체지향 설계 원칙)](https://aeliketodo.tistory.com/74)
    - SRP(The Single Responsibility Principle)
    - OCP(The Open Closed Principle)
    - LSP(The Liskov Substitution Principle)
    - ISP(The Interface Segregation Principle)
    - DIP(The Dependency Inversion Principle)

<br>
<br>

### 높은 응집도와 낮은 결합도

- 소프트웨어 개발의 고전적인 원리
- 응집도가 높다?
    - 한 모듈, 클래스가 하나의 책임/관심사에만 집중되어 있다.
    - 변화가 일어날 때 해당 모듈에서 변하는 부분이 크다.
- 결합도가 낮다?
    - 책임과 관심사가 다른 오브젝트/모듈과는 느슨하게 연결된 형태를 유지한다.
    - 느슨한 연결 = 관계 유지에 꼭 필요한 최소한의 방법만 간접적인 형태로 제공하는 것
    - 변화 대응 속도가 높아지고, 구성이 깔끔해진다. 확장에 편리하다.

<br>
<br>

### [전략 패턴](https://aeliketodo.tistory.com/86)

- Strategy Pattern
- 자신의 기능 컨텍스트에서 필요에 따라 변경이 필요한 코드를 인터페이스를 통해 통째로 외부로 분리시킨다.
- 분리한 알고리즘 인터페이스의 구현 클래스를 필요에 따라 바꿔 사용할 수 있게 한다.
- ex) UserDao는 전략 패턴의 컨텍스트에 해당한다.

<br>
<br>

## 제어의 역전(IoC)
- Inversion of Control

<br>
<br>

### 오브젝트 팩토리
- 문제 상황: main() 메소드는 1.테스트 뿐만 아니라 2.의존관계를 설정하는 두 가지의 책임을 맡고 있다.

<br>
<br>

### 팩토리
- 해결 방법: 의존관계를 설정하는 역할의 새로운 클래스를 만들자!
- 팩토리(factory): 객체 생성 방법을 결정하고 그 방법대로 만들어진 오브젝트를 돌려준다.

```java
public class DaoFactory {
	public UserDao userDao() {
		ConnectionMaker connectionMaker = new DConnectionMaker();
		UserDao userDao = new UserDao(connectionMaker);
		return userDao;
	}
}
```

<br>

- main 메소드

```java
UserDao dao = new DaoFactory().userDao();
```

<br>
<br>

### 설계도로서의 팩토리
- UserDao, ConnectionMaker: 애플리케이션의 각 핵심 로직을 담당
- DaoFactory: 오브젝트 구성과 관계를 정의 ( == 설계도)

<br>

 ⇒ ConnectionMaker 구현 클래스 변경이 필요하면 DaoFactory에서 한 줄만 수정하면 된다.
 ⇒ 핵심 로직을 담당하는 오브젝트와 구조를 결정하는 오브젝트를 분리해냈다.

<br>
<br>

### 제어권 이전을 통한 제어관계 역전
- **제어의 역전**
    - 프로그램 제어 흐름 구조가 뒤바뀌는 것
    - 오브젝트는 자신이 사용할 오브젝트를 스스로 선택하지 않는다.
    - 모든 제어 권한을 자신이 아닌 다른 대상에게 위임한다.

<br>

- 라이브러리 ↔ 프레임워크
  - 라이브러리
    - 애플리케이션 흐름을 직접 제어한다.
    - 필요한 기능이 있을 때 능동적으로 라이브러리를 사용한다.
  - 프레임워크
    - 거꾸로 애플리케이션 코드가 프레임워크에 의해 사용된다.
    - 분명한 제어의 역전 개념이 적용되어 있다.
    
<br>
<br>

### 오브젝트 팩토리를 이용한 스프링 IoC

<애플리케이션 컨텍스트와 설정정보>

- 빈(Bean)
    - 스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트
    - 스프링 컨테이너가 빈을 제어하는 IoC 개념 적용
    
- 빈 팩토리(Bean Factory) = 애플리케이션 컨텍스트(application context)
    - 빈 생성, 관계 설정 등 제어를 담당하는 IoC 오브젝트
    - 별도의 설정 정보를 담고 있는 무언가를 가져와 제어 작업 실행

<br>

<DaoFactory를 사용하는 애플리케이션 컨텍스트> 

```java
@Configuration
public class DaoFactory {
	@Bean
	public ConnectionMaker connectionMaker() {
		return new DConnectionMaker();
	}

	@Bean
	public UserDao userDao() {
		return new UserDao(connectionMaker);
	}

	@Bean
	public AccountDao accountDao() {
		return new UserDao(connectionMaker);
	}
}
```

<br>

- main() 코드

```java

ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
UserDao dao = context.getBean("userDao", UserDao.class);
```

<br>


### 애플리케이션 컨텍스트의 동작방식

- 애플리케이션 컨텍스트의 또다른 이름
    - IoC 컨테이너
    - 스프링 컨테이너
    - 빈 팩토리

<br>

- 우리가 만든 DaoFactory ↔ 스프링의 Application Context
    - 우리가 만든 DaoFactory
        - DAO 오브젝트 생성, 관계설정만 담당
    - 스프링의 Application Context
        - IoC를 적용해 모든 빈 오브젝트의 생성, 관계설정 담당
        - DaoFactory와 달리 별도 설정정보(@Configuration이 붙은 IoC 설정정보 클래스) 사용


<br>

- 애플리케이션 컨텍스트 장점
    1. 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다.
    2. 종합적인 IoC 서비스를 제공한다.
        - 오브젝트 생성, 다른 오브젝트와의 관계설정
        - 오브젝트의 생성 방식, 시점, 전략을 다르게 설정
        - 자동 생성, 후처리, 정보 조합, 설정 방식 다변화, 인터셉팅 등
    3. 다양한 빈 검색 방법을 제공한다.

<br>
<br>

### 스프링 IoC 용어 정리

- 빈 팩토리
    - 스프링의 IoC를 담당하는 핵심 컨테이너
    - 보통 빈 팩토리를 바로 사용하지 않고 이를 확장한 애플리케이션 컨텍스트 이용
- 애플리케이션 컨텍스트
    - 빈 팩토리를 확장한 IoC 컨테이너
    - 빈 등록, 관리 기능 + 스프링의 부가 서비스 추가 제공
- 설정정보/설정 메타정보
    - IoC를 적용하기 위해 애플리케이션 컨텍스트가 사용하는 메타정보
    - 영어로 ‘configuration’
- 컨테이너(IoC 컨테이너)
    - 애플리케이션 컨텍스트나 빈 팩토리의 다른 이름
- 스프링 프레임워크
    - 위의 것들을 포함한 스프링의 모든 제공 기능

<br>
<br>

## 싱글톤 레지스트리와 오브젝트 스코프

- 오브젝트의 동일성 ↔ 동등성
    - 동일성 비교
        - 완전히 같은 동일한 오브젝트
    - 동등성 비교
        - 동일한 정보를 담고 있는 오브젝트
    
    ⇒ 자바의 equals()는 동일성을 비교한다!
 
<br>


- userDao()를 여러 번 호출하면 동일한 오브젝트가 돌아오는가?
  - DaoFactory ⇒ 매번 다른 오브젝트가 돌아온다.
  - Application Context ⇒ 동일한 오브젝트가 돌아온다.
 
<br>
<br>

### 싱글톤 레지스트리로서의 애플리케이션 컨텍스트

- 애플리케이션 컨텍스트는 싱글톤을 저장하고 관리하는 **싱글톤 레지스트리**이기도 하다.
- 별다른 설정이 없으면 빈 오브젝트를 모두 싱글톤으로 만든다.

<br>

### 서버 애플리케이션과 싱글톤
- 클라이언트에서 요청이 올 때마다 담당 오브젝트를 생성해 사용하면 한 시간이 수백만 개의 오브젝트가 생성된다.
- 그래서 엔터프라이즈 분야는 이 문제를 해결하기 위해 서비스 오브젝트라는 개념을 사용한다.
- 예를 들어 서비스 오브젝트인 서블릿은 대부분 멀티스레드 환경에서 싱글톤으로 동작한다.

<br>

- **싱글톤 패턴**
    - 애플리케이션 내에서 어떤 클래스를 하나만 존재하도록 강제하는 패턴

<br>
<br>

### 싱글톤 패턴의 한계
- 싱글톤 구현 방법
    
    ```java
    public class UserDao {
    	private static UserDao instance;
    
    	public static synchronized UserDao getInstance() {
    		if (instance == null) instance = new UserDao(???);
    		return instance;
    	}
    	...
    }
    ```
    
<br>

- 문제점
    - private 생성자를 갖고 있어 상속할 수 없다.
    - 만들어지는 방식이 제한적이라 테스트하기가 힘들다.
    - 서버환경에서 싱글톤이 하나만 만들어지는 것을 보장할 수 없다.
    - 싱글톤 사용은 전역 상태를 만들 수 있어 바람직하지 못하다.

<br>
<br>

### 싱글톤 레지스트리
- 스프링은 위의 자바 구현 방식을 사용하지 않고 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공한다.
- 싱글톤 레지스트리는 고전적인 싱글톤 패턴과 달리 아무 제약이 없다.

<br>
<br>

### 싱글톤과 오브젝트의 상태

- 멀티스레드 환경에서 여러 스레드가 싱글톤에 동시에 접근할 수 있다.
- 그래서 싱글톤은 상태 정보를 내부에 갖고 있지 않는 **무상태 방식**이어야 한다.
- 각 요청에 대한 정보, DB나 서버 리소스로부터 생성한 정보는 파라미터, 로컬 변수, 리턴 값등을 이용한다.

<br>
<br>

### 스프링 빈의 스코프

- 빈의 스코프
    - 빈이 생성되고, 존재하고, 적용되는 범위
    - 종류
      - 싱글톤 스코프: 기본 스코프
      - 프로토타입 스코프: 컨테이너에서 빈을 요청할 때마다 매번 새로운 오브젝트 생성
      - 요청 스코프: 새 HTTP 요청마다 생성
      - 세션 스코프: 웹 세션과 유사

<br>
<br>

### 런타임 의존관계 설정

- 의존한다?
    - 의존대상인 B가 변하면 그것이 A에 영향을 미친다.
    - ex) A가 B에 정의된 메소드를 호출해서 사용하는 경우

<br>

- UserDao의 의존관계
  UserDao 클래스는 ConnectionMaker 인터페이스에게만 직접 의존한다.
 ⇒ 인터페이스에만 의존관계를 만들어두면 구현 클래스와의 관계가 느슨해지면서 변화에 영향을 덜 받는 상태가 된다.

<br>

- 의존 오브젝트
    - 프로그램이 시작되고 런타임 시에 의존관계를 맺는 실제 사용 대상의 오브젝트

<br>

즉, 의존관계 주입은 다음 3가지 조건을 충족하는 작업이다.

1. 클래스 모델, 코드에는 런타임 시점의 의존관계가 드러나지 않는다.  ⇒ 인터페이스에만 의존한다.
2. 런타임 시점 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.
3. 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 재공해줌으로써 만들어진다.

<br>
<br>

### 의존관계 검색과 주입

- 의존관계 검색(Dependency Lookup)
  <br>
  스스로 의존관계를 검색해 맺는 방법
    

```java
public UserDao() {
	AnnotationConfigApplicationContext context = 
			new AnnotationConfigApplicationContext(DaoFactory.class);
	this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
}
```

<br>

- DI ↔ DL
    - DL의 단점
        - DL은 코드 안에 오브젝트 팩토리 클래스, 스프링 API가 함께 나타남
    - DL의 장점
        - 테스트 코드처럼 오브젝트를 가져와야 하는 경우가 발생
        - DI와 달리 DL의 검색하는 오브젝트는 자신이 스프링 빈일 필요가 없다.

<br>
<br>


### 의존관계 주입 응용

> 기능 구현의 교환
> 
- 개발용 로컬 DB에서 배포를 위해 서버 DB로 변경해야 하는 경우
    - DI를 사용하지 않으면
        - 코드를 전면 수정해야 함
    - DI를 사용하면
        - ConnectionMaker 구현 클래스를 하나더 만들면 된다.
        - DaoFactory 코드는 connectionMaker 구현 클래스 설정 코드 1줄만 변경

<br>

> 부가기능 추가
> 
- DB 연결 횟수를 카운팅해보자.
    - DI를 사용하지 않으면
        - 모든 DB 연결 코드 부분에 카운터 증가 코드를 넣는다.
    - DI를 사용하면
        - 연결 횟수 카운팅 기능이 있는 구현 클래스를 또 만들면 된다.

<br>
<br>

## XML을 이용한 설정

### XML 설정

|  | 자바 코드 설정정보 | XML 설정정보 |
| --- | --- | --- |
| 빈 설정파일 | @Configuration | <beans> |
| 빈의 이름 | @Bean methodName() | <bean id=”methodName” |
| 빈의 클래스 | return new BeanClass(); | class=”a.b.c… BeanClass”> |

<br>
  
### userDao() 전환

- <property>: 의존 오브젝트와의 관계 정의
    - name: 프로퍼티 이름
    - ref: 수정자 메소드를 통해 주입할 오브젝트 빈 이름

```java
<bean id="userDao" class="springbook.dao.UserDao">
	<property name="connectionMaker" ref="connectionMaker" />
</bean>
```

<br>
  
### XML의 의존관계 주입 정보

```java
<beans>
	<bean id="connectionMaker" class="springbook.user.dao.DConnectionMaker" />
	<bean id="userDao" class="springbook.dao.UserDao">
		<property name="connectionMaker" ref="connectionMaker" />
	</bean>
</beans>
```

- XML 문서 구조 정의 방법
    1. DTD
    2. 스키마(schema)
    
    ⇒ 스키마 사용이 더 바람직
    

<br>
  
### XML을 이용하는 애플리케이션 컨텍스트

```java
ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
```

<br>
  
### DataSource 인터페이스 변환

> 인터페이스 적용
> 
- 자바에 이미 존재하는 DB 커넥션 용도의 DataSource 인터페이스를 사용한다.

```java
public interface DataSource extends CommonDataSource, Wrapper {
	Connection getConnection() throws SQLException;
	...
}
```

<br>
  
> 자바 코드 설정 방식
> 

```java
@Bean
public DataSource dataSource() {
	SimpleDriverDataSource dataSource = new SimpleDriverDataSource();

	dataSource.setDriverClass(com.mysql.jdbc.Driver.class);
	dataSource.setUrl("jdbc:mysql://localhost:springbook");
	dataSource.setUsername("spring");
	dataSource.setPassword("book");
	
	return dataSource;
}

@Bean
public UserDao userDao() {
	UserDao userDao = new UserDao();
	userDao.setDataSource(dataSource());
	return UserDao;
}
```

<br>

> XML 설정 방식
> 

```java
<bean id="dataSource"
	class="org.springframework.jdbc.datasource.SimpleDriverDataSource" />
```

<br>
  
### 프로퍼티 값 주입

```java
<property name"driverClass" value="com.mysql.jdbc.Driver" />
<property name="url" value="jdbc:mysql://localhost/springbook" />
<property name="username" value="spring" />
<property name="password" value="book" />
```

- `com.mysql.jdbc.Driver` 은 스프링이 setter 파라미터 타입을 참고해 클래스 타입으로 자동 변환
