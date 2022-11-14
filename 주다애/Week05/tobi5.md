## 5. 서비스 추상화

> 서비스 추상화는 로우레벨의 트랜잭션 기술과 API의 변환에 상관없이 일관된 API를 가진 추상화 계층을 도입한다.

### 1. 사용자 레벨 관리 기능 추가

> UserDao에 비즈니스 로직을 추가하여 기능을 더하고, 코드를 수정하는 과정을 거쳐야 한다.


#### 1. 필드 추가
숫자를 직접 추가하기 보단 **enum** 타입을 사용하자
Level 클래스와 User 필드 추가 작업 수행

    public class User {
	    Level level; // level 필드 추가
	    public Level getLevel(){
		    return level;
	    }
	    public void setLevel(Level level){
		    this.level = level;
		}
    }

테스트 코드와 UserDaoJdbc 수정

#### 2. 사용자 수정 기능 추가
사용자 관리 비즈니스 로직에 따라 사용자 정보를 수정하는 기능을 추가하자
수정할 정보가 담긴 User 오브젝트를 전달하면 id를 참고해서 사용자를 찾아서 update문을 사용해 정보를 수정하는 메소드로 기능 구현

    public interface UserDao{
	    ...
	    public void update(User user1);
    }

    public void update(User user) {
	    // update 쿼리 날림(jdbcTemplate 사용)
    }

#### 3. UserService.upgradeLevels()
사용자 관리 로직은 어디에 넣는 것이 좋나? 
--> dao는 데이터를 어떻게 관리하고 가져올지에 관한 역할만 하도록 하자
--> **UserService** 클래스를 추가하여 이곳에 비즈니스 로직을 담자
* UserService는 userDao빈은 DI받아서 사용하며, UserDao의 구현 클래스가 변경되더라도 영향을 받으면 안된다.
![토비의 스프링 [5장 서비스 추상화 - 5.2장] 스터디](https://images.velog.io/images/devsigner9920/post/eca48521-b42f-452b-a075-01dc47cc75a7/258E371C-2076-44C7-8888-92B2E9D3681E.png)

UserService 클래스와 빈 등록의 코드를 살펴보면 아래와 같다.

    public class UserService{
	    UserDao userDao;
	    public void setUserDao(UserDao userDao){
		    this.userDao = userDao;
	    }
    }
테스트 클래스에도 @Autowired를 통해 userService를 주입해주어야한다.

    public void upgradeLevels(){
	    List<User> users = userDao.getAll();
	    for(User user : users){
		    // 레벨 업그레이드 구문 작성(if else if else if else)
		    if(changed) userDao.update(user);
	    }
    }
--> changed 플래그를 확인해서 레벨이 변경되었을 경우에만 UserDao의 update()를 호출한다.

테스트를 진행할 때 좋은 방법은 테스트에 사용할 데이터를 경계가 되는 값의 전후로 선택하는 것이 좋다.(경계가 50이면 49, 51 사용하는 것이 좋음)

#### 4. UserService.add()
처음 가입하는 사용자의 디폴트 레벨은 BASIC이어야 한다는 조건을 구현해야한다.
--> 이 로직은 어디에 담을까?
1. User 클래스의 level필드를 직접 초기화하자
--> 단지 초기값일 뿐인데 이를 위해 클래스 직접 초기화는 좋지 않다.
2. UserService에 로직을 담자
--> UserSerivce 안에도 **add() 메소드**를 만들자! userService.add(user)를 호출하면 디폴트 레벨이 BASIC이 되도록 코딩을하자

테스트 먼저 진행해서 결과 확인

    public void add(User user){
	    if(user.getLevel() == null) user.setLevel(Level.BASIC);
	    userDao.add(user);
    }


#### 5. 코드개선
1. 코드에 중복된 경우는 없는가?
2. 코드를 이해하기 어렵진 않은가?
3. 코드가 적절한 자리에 있는가?
4. 변경이 일어난다면 어떤 것이고, 변화에 쉽게 대응가능한가?

#### upgradeLevels() 메소드의 문제점
1. if/else if문의 중첩으로 읽기 힘들다.
2. 성격이 다른 로직이 한 군데에 몰려있다.
한가지 if 문에 여러가지의 로직이 섞여있다. 
레벨 파악, 업그레이드 조건, 다음 단계 레벨 파악을 하는 로직이 하나의 if 문에 들어가있어서 코드가 길어지고 파악이 힘들다.
3. 이러한 if 조건 블록들이 레벨 개수만큼 반복된다.
4. 결국, 성격이 다른 두 가지 경우(현재 레벨과 업그레이드 조건 동시 비교)가 모두 한 곳에서 처리되는 것은 이상하다.

#### upgradeLevels() 리팩토링

    public void upgradeLevels(){
	    List<User> users = userDao.getAll();
	    for(User user : users){
		    if(canUpgradeLevel(user)){
			    upgradeLevel(user);
		    }
	    }
    }
먼저 추상적인 레벨에서 로직을 작성하면 위와같다.
그리고 구체적 메소드를 따로 빼서 처리한다.

    private boolean canUpgradeLevel(User user){
	    Level currentLevel = user.getLevel();
	    switch(currentLevel){
		    case BASIC: return (user.getLogin() >= 50);
		    ...
	    }
    }
* switch 구문으로 레벨을 구분한다.

다음으로 업그레이드 조건 만족 시 구체적으로 할 것을 정하는 upgradeLevel() 메소드를 만든다. 

    private void upgradeLevel(User user){
	    if (user.getLevel() == Level.BASIC) user.setLevel(Level.SILVER);
	    else if ... 
	    userDao.update(user); // DB에 업그레이드 반영까지 처리
    }
--> 이 코드가 마음에 드는가?
아니다!!
먼저, 로직이 두 개가 함께 있고, 예외처리가 엇다. 또한 레벨이 늘어나면 if문이 늘어나서 코드가 지저분해질 것이다.
--> **분리하자**
1. 레벨의 순서와 다음 단계 레벨 결정은 Level 클래스에게 맡기자
2. 사용자 정보는 User 클래스가 관리하게 하자

그렇게 되면 upgradeLevel()은 더 명확해진다.


    public enum Level{
	    ...
	    private final Level next; // 다음 레벨 정보
	    
	    public Level nextLevel(){
		    return this.next;
	    }
	    public static Level valueOf(int value){
		    switch(level){
			    case 1: return BASIC;
			    ...
			    default: throw new AssertionError("unknown value:" + value);
		    }
	    }
    }
    
    public class User {
	    ...
	    public void upgradeLevel(){
		    Level nextLevel = this.level.nextLevel();
		    if(nextLevel == null) // 예외처리도 스스로 가짐
		    else this.level = nextLevel;
	    }
	    // 
    }

	// 리팩토링된 UserService의 upgradeLevel 메소드
    private void upgradeLevel(User user){
	    user.upgradeLevel();
	    userDao.update(user);
	}
--> 각 오브젝트가 해야 할 책임이 잘 분리가 되었다.

> **객체지향적** 코드는 다른 오브젝트의 데이터를 가져와서 작업하는 대신, 데이터를 갖고있는 다른 오브젝트에게 작업을 요청한다.

UserService가 User에게 레벨 업그레이드를 요청하고 User은 Level에게서 다음 레벨을 가지고 온다.

#### UserService 테스트 리팩토링
항상 테스트를 거쳐야 한다는 것을 잊으면 안된다.

    @Test
    public void upgradeLevels() {
	    userDao.deleteAll();
	    for(User user : users) userDao.add(user);
	    userService.upgradeLevels();
	    
	    checkLevelUpgraded(users.get(0), false);
	    ...
    }
    private void checkLevelUpgraded(User user, boolean upgraded){
	    // 업그레이드 되었는지 확인
    }

테스트와 애플리케이션 코드의 중복된 숫자도 제거해야하나?
--> 당연하다. 한 가지 변경 이유가 발생했을 때 여러 군데를 고치게 만든다면 중복이다. 또한 상수 값 중복은 바람직하지 못하다.

    private static final int MIN_LOGCOUNT_FOR_SILVER = 50;
	
위와 같이 구체적 값을 정수형 상수로 변경한다.
숫자의 의도 파악도 쉽고 중복 제거를 했기에 상수 값만 변경해주면 된다.

### 2. 트랜잭션 서비스 추상화

> 트랜잭션은 DBMS의 작업 단위로, 원자성을 만족하며 작동되어야 한다.

#### 1. 모 아니면 도
만약 서비스 중간에 작업이 중단된다면 어떻게 될까?
테스트를 통해서 살펴보자
테스트용으로 특별히 만든 UserService의 대역을 사용하자

    static class TestUserService extends UserService{
	    private String id;
	    private TestUserService(String id){
		    this.id = id;
	    }
	    protected void upgradeLevel(User user){
		    if(user.getId().equals(this.id)) throw new TestUserServiceException();
		    super.upgradeLevel(user);
	    }
    }

과연 테스트가 성공할까?
테스트는 실패한다.

#### 트랜잭션
위의 테스트가 왜 실패하는가에 대한 대답은 바로 트랜잭션 문제이다.
트랜잭션이랑 더 이상 나눌 수 없는 단위 작업니다.
**전체가 다 성공하든지 아니면 전체가 다 실패해야한다.**
즉, 중간에 예외가 발생하면 아예 작업이 시작되지 않았던 것처럼 작업을 처음상태로 돌려놓아야한다.

#### 2. 트랜잭션의 경계 설정
먼저, DB는 그 자체로 완벽한 트랜잭션을 지원한다.
즉, 하나의 sql 명령을 처리하는 경우는 DB가 트랜잭션을 보장해준다고 믿을 수 있다. 
그런데 여러 개의 sql이 사용되는 작업을 하나의 트랜잭션으로 취급해주어야 할 때가 있다. (은행 계좌이체 시스템과 같은 경우)
1. 트랜잭션 롤백 : 첫 번째 sql은 성공했지만 두 번째 sql이 실해하면 앞에서 처리한 작업도 취소해야한다.
2. 트랜잭션 커밋 : 모든 sql이 성공적으로 완료되면 DB에 커밋되었다고 알려주어야 한다.

#### 트랜잭션 경계

> 트랜잭션 경계란 애플리케이션 내에서 트랜잭션이 시작하고 끝나는 위치를 뜻한다.
![5장 서비스 추상화](https://velog.velcdn.com/images%2Fpu1etproof%2Fpost%2Fd968c84a-836b-4046-b333-a5f3e7ad23d1%2FKakaoTalk_Photo_2021-12-20-02-14-38.jpeg)
* jdbc의 트랜잭션은 하나의 Connection을 가져와 사용하다가 닫는 사이에 일어난다. 트랜잭션의 시작과 종료는 Connection 오브젝트를 통해 이루어지기 때문이다.
* 이렇게 트랜잭션의 시작을 선언하고 종료하는 작업을 **트랜잭션 경계설정** 이라고 한다.
* 하나의 DB 커넥션 안에서 만들어지는 트랜잭션을 **로컬 트랜잭션** 이라고 한다.

#### UserDao와 UserService의 트랜잭션 문제
지금까지 만든 코드 어디에도 트랜잭션 경계설정 코드가 존재하지 않는다!
* JdbcTemplate의 메소드를 사용하는 UserDao는 각 메소드마다 독립적인 트랜잭션으로 실행되어야만 한다.
![토비의 스프링 정리 프로젝트 #5.2 트랜잭션 서비스 추상화](https://velog.velcdn.com/images%2Fjakeseo_me%2Fpost%2F37f1bb46-3bf6-4c49-bd9a-4f15f3dc7e75%2Fimage.png)

우리는 DAO에 데이터 엑세스 코드를 분리해놓았기 때문에 위와 같이 DAO 메소드 호출 시 하나의 새로운 트랜잭션이 만들어지는 구조를 갖는다.
--> 결국 DAO를 사용하면 여러 가지 작업을 하나의 트랜잭션으로 묶는 일이 불가능해진다.
* 어떤 일련의 작업이 하나의 트랜잭션으로 묶이려면 그 작업이 진행되는 동안 DB 커넥션도 하나만 사용되어야 한다.
* 하지만 현재는 UserService에서 DB 커넥션 못 다룬다..!

#### 비즈니스 로직 내의 트랜잭션 경계설정
트랜잭션 경계설정 작업을 UserService쪽으로 가져와야 한다.(upgradeLevels() 메소드의 시작과 함께 트랜잭션이 시작하고 메소드를 빠져나올 때 트랜잭션이 끝나기 때문)

    public void upgradeLevels() throws Exception {
      // (1) DB Connection 생성
        // (2) 트랜잭션 시작
        try {
		    // (3) DAO 메소드 호출
		    // (4) 트랜잭션 커밋
		}
		catch(Exception e) {
		    // (5) 트랜잭션 롤백
		    throw e;
		}
		finally {
		    // (6) DB Connection 종료
		}
	}

* UserDao의 update() 메소드는 반드시 upgradeLevels() 메소드에서 만든 Connection을 사용해야 한다. 그래야 같은 트랜잭션 내에서 동작!

즉, UserDao의 메소드에 파라미터로 Connetcion c가 추가되어야 한다.
흐름을 보면 UserSerivce에서 upgradeLevel() 호출 -> upgradeLevel)에서 userDao의 update(c, user)호출 -> 업데이트 작동

위 방법의 **문제점은** 무엇일까?
1. 더이상 JdbcTemplate이 사용 불가하다.
2. DAO와 UserService 메소드에 Connection 파라미터가 추가되어야 한다.
3. UserDao는 Connection 파라미터를 가진 순간 데이터 엑세스 기술에 독립적일 수 없다.
4. 테스트 코드도 Connection 오브젝트를 일일이 만들어야 한다.

#### 3. 트랜잭션 동기화
그러면 우리는 트랜잭션 기능을 포기해야 하는가?
**아니다. 스프링은 이 딜레마를 해결해준다.**

#### 3-1. Connection 파라미터 제거

> 트랜잭션 동기화란 Connection 오브젝트를 특별한 저장소에 보관해두고 호출되는 DAO 메소드에서는 저장된 Connection을 가져다가 사용하는 것이다.

JdbcTemplate이 트랜잭션 동기화 방식을 사용하고 트랜잭션이 종료되면 같이 동기화를 종효한다.
![](https://velog.velcdn.com/images%2Fjakeseo_me%2Fpost%2F0971840d-5764-421d-8b67-609b739588c6%2Fimage.png)

트랜잭션 동기화 저장소는 스레드마다 독립적인 Connection오브젝트를 저장하고 관리하므로 멀티스레드 환경에서 충돌 걱정이 없다.

#### 3-2. 트랜잭션 동기화 적용

> 스프링은 트랜잭션 동기화 클래스로 TransactionSynchronizationManager 클래스를 제공한다.

스프링은 JdbcTemplate과 더불어 트랜잭션 동기화 기능을 지원하는 간단한 유틸리티 메소드를 제공한다.

#### 3-3. 트랜잭션 테스트 보완

    @Autowired DataSource dataSource;
    ...
    @Test
    public void upgradeAllOrNothing() throws Exception{
	   ...
	   testUserService.setDataSource(this.dataSource);
    }

#### 3-4. JdbcTemplate과 트랜잭션 동기화
JdbcTemplate의 동작방식을 보면 스스로 Connection을 생성해서 사용한다는 것을 알 수 있다.
트랜잭션 동기화를 해주면 DAO에서 사용하는 JdbcTemplate은 자동으로 트랜잭션 안에서 동작한다.
하지만! 스프링에서는 지금부터가 트랜잭션 적용에 대한 본격적인 고민의 시작이다.

#### 4. 트랜잭션 서비스 추상화

> 추상화란 하위 시스템의 공통점을 뽑아내서 분리시키는 것이다.

만약, 여러 개의 DB에 데이터를 넣는 요구사항이 추가된다면?
하나의 트랜잭션 안에서 여러 개의 DB에 데이터를 넣어야한다.
즉, 한 개 이상의 DB로의 작업을 하나의 트랜잭션으로 만들어야하는데 이는 로컬 트랜잭션에서는 불가하다.
따라서, 별도의 트랜잭션 관리자를 통해 트랜잭션을 관리하는 **글로벌 트랜잭션** 방식을 사용하자

JTA라는 API를 사용하자
![](https://velog.velcdn.com/images%2Fpu1etproof%2Fpost%2F8b43ea63-918a-4e12-89ba-478b0a158dcc%2FKakaoTalk_Photo_2021-12-20-15-42-28.jpeg)

문제는 UserService 코드를 수정해야 한다는 것이다. 
자신의 로직이 바뀌지 않았는데도 기술 환경이 바뀌어서 수정되어야 하는 문제가 발생한다.
UserService의 코드가 특정 트랜잭션 방법에 의존적이지 않고 독립적이게 만드려면 어떻게 해야하는가?
--> 이 때 우리는 **추상화**를 생각해볼 수 있다!

#### 스프링의 트랜잭션 서비스 추상화

> 스프링은 트랜잭션 기술의 공통점을 담은 트랜잭션 추상화 기술을 제공한다. 공통적인 특성을 모아서 추상화 해놓은 것이다.

    public void upgradeLevels(){
	    PlatformTransactionManager tm = new DataSourceTransactionManager(dataSource)
	    TransactionStatus  status = tm.getTransaction(new DefaultTransactionDefinition()); // status에 트랜잭션 저장
	    try {
		     // 작업 진행
		     // commit
	    }
	    catch() {
		    // rollback
	    }
    }
PlatformTransactionManager은 스프링이 제공하는 트랜잭션 경계설정을 위한 추상 인터페이스이다. 

#### 트랜잭션 기술 설정의 분리

    PlatformTransactionManager txManager = new JTATransactionManager();

위의 코드의 문제는?
UserService가 어떤 트랜잭션 매니저 구현 클래스를 사용할지 알게 되기 때문에 DI 원칙에 위배된다.
--> 스프링 DI의 방식으로 바꾸자(PlatformTransactionManager의 구현 클래스는 싱글톤으로 사용이 가능하니 안심하자)

    public class UserService{
	    private PlatformTransactionManager tm;
	    public void setTransactionManager(PlatformTransactionManager tm){
		    this.tm = tm;
	    }
	    // PlatformTransactionManager DI 과정
	    // + 설정파일 등록
    }


### 3. 서비스 추상화와 단일 책임 원칙

> 객체지향적 프로그래밍을 위해서는 단일 책임 원칙을 지켜야하는데, 단일 책임 원칙이란 하나의 모듈은 한 가지 책임을 가져야 한다는 의미이다.

![](https://velog.velcdn.com/images%2Fpu1etproof%2Fpost%2F7e3846e7-65f7-4a31-bb95-8c6cf2013ba5%2FKakaoTalk_Photo_2021-12-20-17-59-09.jpeg)

* 결합도가 낮은 분리는 애플리케이션 코드를 로우레벨의 기술 서비스와 환경에서 독립시켜준다.

#### 단일 책임 원칙
단일 책임 원칙은 하나의 모듈은 한 가지 책임을 가져야 한다는 의미로, 하나의 모듈이 바뀌는 이유는 한 가지여야 한다고 설명한다.

* 두 가지 책임은 코드가 수정되는 이유가 두 가지라는 뜻과 같다. 이는 단일 책임 원칙을 위배한다.

단일 책임 원칙의 장점은
1. 변경시 수정 대상이 명확해진다.
2. 코드 수정이 방대하게 커지는 것을 방지한다.
3. DI를 사용하면 모듈 간의 결합도를 낮추고 응집도를 높이게 된다.

**결국 DI를 왜 사용해야하는지 알고, 프로젝트에 도입하다 보면, 객제지향적 프로그래밍을 할 수 있게 된다.**

### 4. 메일 서비스 추상화

> JavaMail을 사용해서 메일 서비스를 추가하려면 지켜야 할 것들이 있다.

#### 1. JavaMail을 이용한 메일 발송 기능
레벨이 업그레이드되면 사용자에게 메일을 발송 해달라는 요구사항이 추가되었다.
자바에서 메일을 발송할 때는 JavaMail이라는 표준 기술을 사용하면 된다.
SMTP 프로토콜을 지원하는 메일 전송 서버가 준비되어 있다면 정상적으로 안내 메일이 발송된다.

#### 2. JavaMail이 포함된 코드의 테스트
만약 메일 서버가 준비되어 있지 않다면 어떻게 되는가?
메일 발송은 매우 부하가 큰 작업이다.
또한 JavaMail은 안정적인 모듈이므로 테스트를 위해 굳이 구동시킬 필요가 없다.

#### 3. 테스트를 위한 서비스 추상화
테스트의 문제점은?
--> JavaMail의 핵심 API에는 인터페이스로 만들어져서 구현을 바꿀 수 있는 것이 없다.
그러면?
--> 서비스 추상화를 적용하면 된다.
++  스프링의 DI를 적용하자
 
 우리가 원하는 것은?
 --> JavaMail을 사용하지 않고 메일 발송 기능이 포함된 코드를 테스트 하는 것 테스트용 메일 전송 클래스를 만들어보자

    public class DummyMailSender implements MailSender{
	    public void send(SimpleMailMessage mailMessage) throws MailException{
	    }
	    ...
    }
    // 딱히 하는 일이 없다. 대상 메일이 메일 서버로 발송될 일은 없다.


#### 4. 테스트 대역

> 테스트 대역이란 테스트 대상이 되는 오브젝트의 기능에만 충실하게 수행하면서 빠르게, 자주 테스트를 실행할 수 있도록 사용하는 오브젝트들을 통칭해서 말하는 것이다.

* DummyMailSender 클래스는 하는 일은 없지만 가치가 매우 크다.
테스트 대상이 되는 코드를 수정하지 않고, 메일 발송 작업 때문에 UserService자체에 대한 테스트에 지장을 주지 않기 위해 도입되었다.
UserService가 반드시 이용해야 하는 의존 오브젝트의 역할을 한다.

의존 오브젝트란 하나의 오브젝트가 사용하는 DI이다. 협력 오브젝트라고 부르기도 한다. 

#### 테스트 대역의 종류와 특징
DummyMailSender 는 가장 단순하고 심플한 테스트 스텁의 예라고 할 수 있다.
테스트 스텁은 테스트 대역의 한 종류이다.

테스트 스텁을 이용하면 간접적인 입력 값을 지정해줄 수 있다. 또한 간접적인 출력 값을 받게 할 수 있다.

테스트 오브젝트가 간접적으로 의존 오브젝트에 넘기는 값과 그 행위 자체에 대해 검증하고 싶다면 어떻게 해야 하는가?
--> **목 오브젝트**를 사용해야 한다.
목 오브젝트?
테스트 대상 오브젝트와 의존 오브젝트 사이에서 일어나는 일을 검증할 수 있도록 특별히 설계된 오브젝트를 뜻함

#### 목 오브젝트를 이용한 테스트

    stati class MockMailSender implements MailSender{
	    private List<String> requests = new ArrayList<String>;
	    public List<String> getRequests() {
		    return requests;
		}
		public void send(SimpleMailMessage mailMessage) throws MailException{
			requests.add(mailMessage.getTo()[0]);
		}
    }

* 테스트가 수행될 수 있도록 의존 오브젝트에 간접적으로 입력 값을 제공해주는 스텁 오브젝트와 간접적인 출력 값까지 확인이 가능한 목 오브젝트를 통해 테스트를 진행하는 것은 매우 효율적인 방법이다.

### 5. 정리
1. 비즈니스 로직은 내부적으로 책임과 역할에 따라 메소드로 분리해야 한다.
2. 트랜잭션은 단위 작업을 보장한다.
3. 트랜잭션 경계설정이란 트랜잭션의 시작과 종료를 지정하는 일이다.
4. 객체지향적 프로그래밍을 위해서 단일 책임 원칙을 지키며 프로그래밍을 해야한다.
5. 서비스 추상화는 로우레벨의 트랜잭션 기술과 API의 변화에 상관없이 일관된 API를 가진 추상화 계층을 도입한다.
6. 서비스 추상화는 테스트를 편하게 만들어준다는 것만으로도 가치가 충분하다.
7. 테스트 대역이란 테스트 대상이 사용하는 의존 오브젝트를 대체할 수 있도록 만든 오브젝트로, 그 안에 목 오브젝트가 있다.


