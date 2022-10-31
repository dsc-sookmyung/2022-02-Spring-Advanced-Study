## 3. 템플릿

> 예외처리와 안전한 리소스 반환, 디자인 패턴과 템플릿/콜백 패턴을 통해 객체지향적인 코드를 작성하는 것이 중요하다.

### 1. DAO 예외처리

현재 UserDao 코드의 문제점은 **예외상황에 대한 처리**가 없다는 점
예외처리는 JDBC코드에서 반드시 지켜져야 한다. 예외가 발생했을 경우에 사용한 리소스를 반드시 반환하도록 코드를 작성해야 한다.
* 리소스는 보통 풀(pool)  방식으로 운영된다. 즉, 미리 정해진 풀 안에서 리소스를 할당하고 반환하여 풀에 넣는 방식으로 운영되므로 반드시 close() 메소드를 사용하여 사용한 리소스를 다시 풀으로 돌려주어야 한다.

#### try/catch/finally 구문을 사용하여 예외처리를 하자!

    public int getCount throws SQLException{
	    Connection c = null;
	    PreparedStatement ps = null;
	    ResultSet rs = null;
	    
	    try{
		    // 조회 작업 수행
	    }
	    catch(SQLException e){
		    throw e;
	    }
	    finally{
		    if(rs != null){
		    try{
			    rs.close();
			{
		    catch(SQLExeption e){
		    }
		  }
		  // ps와 c모두 동일하게 예외처리	
	    }
    }

### 2. 변하는 것과 변하지 않는 것

> 변하지 않는, 그러나 많은 곳에서 중복되는 코드와 로직에 따라 자꾸 확장되고 자주 변하는 코드를 잘 분리해내자

**분리와 재사용을 위한 디자인 패턴 적용**

    public void deletAll() throws SQLException{
	    try{
		    ...
		    ps = c.prepareStatement("delete from users"); // 변하는 부분
	    }
	    ...
    }

#### 1. 메소드 추출
#### ps = makeStatement(c)라는 메소드로 중복을 제거하자
문제는?
분리시키고 남은 메소드가 재사용이 필요하고, 분리된 메소드는 새롭게 확장되는 부분이다. 구현하고자 하는 의도와 반대 상황이 되었다.

#### 2. 템플릿 메소드 패턴의 적용
#### 상속을 통한 기능확장(탬플릿 메소드 패턴)을 사용해서 변하지 않는 부분은 슈퍼클래스, 변하는 부분은 추상 메소드로 정의하여 서브클래스에서 오버라이드 하자

    abstract protected PreparedStatement makeStatement(Connection c) throws SQLException;

문제는?
메소드 개수만큼의 서브클래스가 필요하다. 확장구조가 이미 클래스를 설계하는 시점에 고정되어 버린다.

#### 3. 전략 패턴의 적용
#### 오브젝트를 아예 둘로 분리하고 클래스 레벨에서는 인터페이스를 통해서만 의존하도록 하자. 변하는 부분을 별도의 클래스로 만들어서 추상화된 인터페이스를 통해 위임하는 방식을 사용하자
* 변하는 부분이 **전략**, 변하지 않는 부분이 **컨텍스트**가 된다.

인터페이스와 전략 클래스를 만들자

    public interface StatementStrategy{
	    PreparedStatement makePreparedStatement(Connection c) 
	    throws SQLException;
    }

    public class DeleteAllStatement implements StatementStrategy{}

* DeleteAllStatement가 확장된 StatementStrategy의 전략 클래스가 되었다.

문제는?
컨텍스트 안에 구체적인 전략 클래스가 고정되어있다. 이는 전략패턴에도 OCP에도 들어맞지 않는다.

#### DI 적용을 위한 클라이언트/컨텍스트 분리
#### 클라이언트에게 컨텍스트가 어떤 전략을 쓸지 결정하게 하자
![](https://velog.velcdn.com/images/yuseogi0218/post/95350e25-2321-4bd8-8ac0-a8d36051a03b/image.png)

* 전략 오브젝트 생성과 컨텍스트로의 전달의 책임을 분리하자
* 여기선 클라이언트 코드가 StatementStrategy가 된다.
* 클라이언트가 컨텍스트가 사용할 전략을 정해서 전달하므로 DI구조로 이해할 수 있다.

### 3. JDBC 전략 패턴의 최적화

> 컨텍스트(JDBC의 작업 흐름)과 전략(PreparedStatement)를 구분해서 전략패턴을 잘 사용하자

1. DAO 메소드마다 새로운 StatementStrategy 구현 클래스 생성 필요
2. 메소드에서 전달할 부가적 정보가 있는 경우에는 이를 위해 생성자와 인스턴스 변수를 추가하는 부담

#### 로컬 클래스
로컬클래스란 클래스 안에 내부 클래스로 정의하는 것
특정 메소드에서만 사용된다면 메소드 안에서 클래스를 정의해버리자  

    public void add(final User user) throws ~ { // 외부 변수는 final로 선언
	    class AddStatement implements StatementStrategy{
	    }
    }


#### 익명 내부 클래스
클래스 이름을 제거한 내부 클래스를 만들자

    StatementStrategy st = new StatementStrategy(){
	    public PreparedStatement makePreparedStatement(Connection c)
	    {...}
    }

### 4. 컨텍스트와 DI

> 클래스 분리와 DI를 구현하자

#### 클래스 분리
* JdbcContext라는 새로운 클래스를 만들어서 모든 DAO가 사용할 수 있게하자

다른 DAO도 사용할 수 있게 하자

    public class JdbcContext{
	    // datasource DI

		public void workWithStatementStrategy(StatementStrategy st){
		}
    }

* 이렇게 분리하면 UserDao에서 JdbcContext를 DI받아서 사용할 수 있다.
* 그런데 여기서 보면 UserDao는 인터페이스가 아니라 구체적인 클래스에 의존하고 있다. 
* 스프링 DI의 기본은 인터페이스를 사이에 두고 의존 클래스를 바꿔서 사용하는 것이다. 
* 따라서 이 경우는 인터페이스를 따로 두지 않고 DI를 적용하는 특별한 구조이다. 

#### 2가지의 DI
#### 1. 스프링 빈으로 DI
인터페이스를 사이에 두지 않으면 완전한 DI라고 볼 수 없다.
하지만 IoC관점에서보면 객체의 생성과 제어권한을 오브젝트에서 제거하고 외부에서 위임했다는 개념을 포괄한다.
1. JdbcContext는 싱글톤 빈이 된다.
* 싱글톤으로 등록하여 여러 오브젝트에서 공유하자
2. JdbcContext가 DI를 통해 다른 빈에 의존중이다.
* DataSource 오브젝트를 주입받는데, 이렇게 다른 빈을 주입받으려면 그 자신이 스프링 빈으로 등록되어야한다.

여기서 중요한건 **인터페이스의 사용 여부**이다.
인터페이스가 없다는 것은 두 개의 클래스가 강한 응집도를 가짐을 뜻한다.
이러한 경우에는 인터페이스를 두지 않아도 상관 없다.

#### 2. 코드를 이용하는 수동 DI
UserDao 내부에서 직접 DI를 적용하자
싱글톤은 포기해야 하지만 큰 문제는 없으며 JdbcContext의 제어권은 
UserDao가 가지게된다.
UserDao에게 JdbcContext의 DI까지 맡기자!

    public class UserDao{
	    ...
	    private JdbcContext jc;
	    // setter + 생성 + DI 작업 동시 실행
	    public void setDataSource(DataSource ds){
		    this.jc = new JdbcContext(); // 생성
		    this.jc.setDataSource(ds); // DI
		    this.ds = ds;
	    }
    }


#### 위의 두 방법도 좋지만, 특별한 이유가 없거나 설명할 수 없을 땐 인터페이스를 통해 DI를 구축하는 것을 원칙으로 하자!

### 5. 템플릿과 콜백

> 전략패턴의 컨텍스트는 템플릿이 되고, 익명 내부 클래스로 만들어지는 오브젝트를 콜백이 된다. 
> 템플릿은 고정된 작업 흐름을 가진 코드를 재사용. 콜백은 템플릿 안에서 호출되는 오브젝트를 뜻한다.

1. 보통 단일 메소드 인터페이스 사용
2. 콜백은 일반적으로 하나의 메소드를 가진 인터페이스를 구현한 익명 내부 클래스로 만들어진다. 
3. 콜백 인터페이스의 메소드는 보통 파라미터를 가진다.
4. 템플릿/콜백 방식은 매번 메소드단위로 사용할 오브젝트를 새롭게 전달받는다.
5. 클라이언트와 콜백이 강하게 결합한다.

#### 콜백의 분리와 재활용
중복될 가능성이 있는 자주 바뀌지 안흔 부분을 분리해보자
sql문을 받아오는 부분을 메소드로 분리하자!
* executeSql()이라는 메소드로 만들어서 query를 파라미터로 받자
* deleteAll()에서 원하는 sql문장을 넘겨주기만 하면 완성!
* 결국 deleteAll() 메소드는 콜백 함수에서 벗어나게 된다.

#### 템플릿/콜백의 응용
스프링은 수십 가지 템플릿/콜백 클래스와 API를 제공한다.
* 객체지향 언어를 사용하고 설계를 통해 코드를 작성하는 것이 기본 자세
* 템플릿/콜백을 사용할 때는 경계를 정하고 어떤 정보를 전달하는지 파악하는 것이 가장 중요하다.
* 콜백의 이름처럼 다시 불려지는 기능을 만들어서 보내고 템플릿과 콜백, 클라이언트 사이에 정보를 주고받는 일이 중요하다.

제네릭스를 이용한 콜백 인터페이스
* 콜백에서 템플릿에게 전달하는 결과의 타입을 하나로 지정하지 않고 다양한 결과를 보내고 싶다면 **제네릭스**를 도입하면 된다.
* 제네릭을 도입하면 다양한 오브젝트 타입을 지원하는 인터페이스/메소드를 정의할 수 있다.

인터페이스와 메소드를 제네릭을 사용해서 구현하자

    public interface LineCallback<T> {
	    T doSomething(String line, T value);
    }
    
    
    public <T> lineReadTemplate(String filepath, LineCallback<T> callback, T initVal) {
	    try{
		    T res = initVal;
		    while(...){
			    res = callback.doSomething(line, res);
		    }
	    }
    }


    public String concateneate(String filepath) {
	    LineCallback<String> cb = new LineCallback<String>(){
		    public String doSomething(String line, String value){
			    return value + line;
		    }};
	    }
	    return lineReadTemplate(filepath, cb, "");
    }


### 6. 스프링의 JdbcTemplate

> 스프링은 JDBC를 이용하는 DAO에서 사용할 수 있또록 다양한 템플릿과 콜백을 제공한다.
> 기본 템플릿은 JdbcTemplate이다.

    public class UserDao{
		private JdbcTemplate jdbcTemplate;
		public void setDataSource(DataSource ds){
			this.jdbcTemplate = new JdbcTemplate(ds);
			this.ds = ds;
		}
    }
    // JdbcTemplate 초기화 코드

#### 1. update()

    public void deleteAll(){
	    this.jdbcTemplate.updae("delete from users");
    }

#### 2. queryForInt()
Integer타입의 결과를 가져오는 sql문장을 전달해주면 queryForInt()메소드가 콜백을 내장하고 있어서 결과 반환이 가능하다.

    public int getCount(){
	    return this.jdbcTemplate.queyForInt("select count(*) from users");
    }

#### 3. queryForObject()
queryForObject()는 단순 값이 아닌 오브젝트를 생성하고 프로퍼티에 넣어줄 때 사용하며 예외처리도 알아서 해준다.

    public User get(String id){
	    return this.jdbcTemplate.queryForObject("select * from users where id =?", 
	    new Object[] {id}, 
	    new RowMapper<User>(){
		    // RowMapper 콜백 실행
		    public User mapRow() throws ~{
		    }
	    }
	    });
    }

#### 4. query()
List<User>을 리턴타입으로 하여 모든 사용자 정보를 돌려주자

    public List<User> getAll(){
	    return this.jdbcTemplate.query("select * from users
	    order by id", new RowMapper<User>() {
			    ...
	    }
	    });
    }

#### 5. 네거티브 테스트
1. 예외상황에 대한 테스트는 항상 빼먹기 쉽다.
2. 미리 예외상황에 대한 일관성 있는 기준을 정해두고 이를 테스트로 만들어 검증해야 한다.
3. 네거티브 테스트를 먼저하는 습관을 들이자

#### 재사용 가능한 콜백의 분리
#### 1. DI를 위한 코드 정리
DataSource 인스턴스 변수는 더이상 필요가 없으므로 제거하자

    public void setDataSource(DataSource dataSource){
	    this.jdbcTemplate = new JdbcTemplate(dataSource);

		// this.dataSource = dataSource 제거(더이상 저장X)
    }

#### 2. 중복 제거
코드에서 중복이 되는 부분은?
**RowMapper 콜백 메소드**
나중에 더 많은 조회기능이 추가될 수 있기에 미리 중복제거를 해야한다!

RowMapper 콜백 메소드를 하나만 만들어서 멀티스레드에서 공유하자

    public class UserDao{
	    private RowMaper<User> userMapper = 
		    new RowMapper<User>() {
			    public User mapRow(...){...}
		    }
    };
    
userMapper라는 인스턴스 변수에 콜백 오브젝트를 초기화하도록 했다.

#### 3. 템플릿/콜백 패턴과 UserDao
최종적으로 완성된 UserDao 클래스는 탬플릿/콜백 패턴과 DI를 이용하여 예외처리+리소스관리까지 구현한다.

    public class UserDao {
	    public void setDataSource(DataSource dataSource){
	    this.jdbcTemplate = new JdbcTemplate(dataSource);
    }
	    private JbdcTemplate jdbcTemplate;
	    
	    private RowMaper<User> userMapper = 
		    new RowMapper<User>() {
			    public User mapRow(...){...}
		    }
		   
	    public void deleteAll() {
		    this.jdbcTemplate.update("delete from users");
	    } 
	    public List<User> getAll() {
		    return this.jdbcTemplate.query("select *
		    from users order by id", this.userMapper);
	    }
    }


1. 해당 코드는 **응집도**가 높다. 즉, 서로가 연관되어 있음이 강하다.
2. 하지만 책임과 관심은 모두 JdbcTemplate에 있기에 책임이 다른 코드와는 **결합도**가 낮다.

### 7. 정리
1. 예외처리와 리소스반환을 보장해야 한다.
2. 전략 패턴을 사용하여 컨텍스트와 전략을 구분하자
3. 익명 내부 클래스를 사용해서 전략 오브젝트를 구현하면 이점이 많다.
4. 템플릿/콜백 패턴을 사용하여 전략 패턴과 전략 DI를 수행하자
5. 스프링은 다양한 템플릿과 콜백을 지원한다.
6. 템플릿/콜백 패턴은 스프링이 얼마나 객체지향적 설계와 프로그래밍에 가치를 두는지 알려주는 패턴이다.
