

# Test code
기존 ) 웹을 통한 테스트
- 모든 기능들 다 만들고 나서야 테스트 가능
- 어디서 에러 났는지 확인 어려움

### 단위테스트
테스트 가능하면 단위로 쪼개서 실행 ( 관심사 분리 적용 )  
단위 테스트 : 작은 단위의 코드에 대해 테스트 수행

## UserDaoTest
1. 장점
- 테스트 과정이 자동으로 진행
- 자주 반복 가능
2. 문제점
- 수동 확인 : 테스트 결과 확인은 사람의 몫
- 실행 작업의 번거로움
3. UserDaoTest 개선
- 테스트 검증 자동화 :  
  에러 발생 시 쉽게 확인 가능. 메소드 기능 일치 하지 않으면 에러 메시지 발생시키기. 둘다 성공하면 성공 메세지 발생


## JUnit을 이용한 테스트 코드 전환
- JUnit : 자바 테스팅 프레임 워크
- 프레임워크 : 클래스에 대한 주도권 받아 주도적으로 애플리케이션의 흐름 조정

기존 ) main() 메소드 사용 : main이 모든 주도권 가짐 -> 적합 x  
메소드 분리 후 JUnit 요구 조건 따르기
>JUnit의 요구 조건
>1. 메소드가 public으로 선언될 것
>2. 메소드에 @Test 어노테이션 붙이기

### if- else
테스트 결과 검증하는 if-else 문장 -> JUnit 제공 메소드 대체

assertThat(user2.getName(), is(user,getName()));- if = assertThat
- 첫번째 파라미터와 매쳐 비교 (is도 매쳐, equals()로 비교)
- 일치하면 다음으로 넘어감
- 메소드 실행 완료 시 테스트 성공 인식함

## Test code 작성
1. 테스트 결과의 일관성 : 외부환경에 상관없이 동일한 결과
- Unit : 하나의 클래스 안에 여러 테스트 메소드 가능
- @Test, public 접근자, return 값 void, 파라미터 x

2. 예외상황 테스트  
   EmptyResultDataAccessException 이용
- 상황 : get() 이용시 id에 해당정보가 없을 경우 -> 정보 전달
- 결과 : EmptyResultDataAccessException가 던져지면 테스트 성공, 정상적 작업 마치면 에러

>항상 네거티브 테스트를 먼저 만들것


## @Before
문제 : 스프링 애플리케이션컨텍스트 만드는 부분 + userDao 가져오는 부분 중복  
해결 : JUnit에서 제공하는 기능 사용
- 반복되는 준비작업 별도의 메소드에 넣음
- 테스트 메소드 실행하기 전에 먼저 실행시켜줌

before
: @Test 메소드 실행되기 전 먼저 실행되야 하는 메소드 정의  
각 테스트 메소드에 반복적으로 나타난 코드 빼서 새로 메소드로 만듬

>JUnit이 테스트 메소드 수행하는 과정
>1. @Test가 붙은 public, void형이며 파라미터가 없는 메소드 모두 찾음
>2. 테스트 클래스의 obj 만듬
>3. @Before     메소드 실행
>4. @Test 메소드 호출, 값 저장해둠
>5. @After 메소드 실행
>6. 나머지 테스트 메소드 2-5 반복
>7. 모든 결과 종합해서 돌려줌

- @Before나 @After를 테스트 메소드에서 직접 호출 x -> 오브젝트 주고받을 경우 인스턴스 변수로 선언해야됨
- 테스트 메소드 실행할 때마다 테스트 클래스의 오브젝트를 새로 만듬
    - 테스트 서로 영향 주지 x 보장
    - 인스턴스 변수 부담없이 사용 가능 -> 다음 테스트 메소드 실행 시 새로운 obj 만들어져서 *초기화* 될것
- 메소드 일부에서 나타나는 특징은  메소드 추출이 편함

### 픽스처
: 테스트 수행시 필요한 정보나 오브젝트  
- 일반적으로 여러 테스트에서 반복되어 나타남 -> @Before

ex) UserDaoTest의 UserDao
테스트 중 add() 메소드에 전달하는 user obj들이 픽스쳐
- 테스트 메소드에서 인스턴스 변수 선언
- @Before 메소드에서 obj 생성



## TDD
1. 테스트 주도 개발 (TDD)
- 만들고자 하는 기능의 내용 o
- 만들어진 코드 점증할 테스트 코드
- 테스트 성공하게 해주는 코드 작성

2. TDD 특징
- 테스트 먼저 만들고 테스트가 성공하도록 하는 코드만 작성
- 코드 작성하면 바로 테스트 실행 가능
- 자연스러운 단위테스트 가능

3. TDD 장점  
   코드 만들어 테스트를 실행하는 간격이 짧음- 너무 많은 코드 작성 x
- 애플리케이션 코드보다 상대적으로 작성하기 쉬움, 독립적
- 작성시간 짧음


## 스프링 테스트 : 애플리케이션 컨텍스트
애플리케이션 컨텍스트
: - 빈이 많고 복잡할 경우 컨텍스트 생성에 오랜시간이 걸림
: - 만들어질 때 모든 싱글톤 빈 오브젝트를 초기화
: - 어떤 빈은 obj 생성시 자체적 초기화 -> 시간 오래

=> 스프링이 직접 제공하는 애플리케션 컨텍스트 텍스트 지원기능 사용

    @RunWith(SpringJUnit4ClassRunner.Class)
    @ContextConfiguration(locations="/applicationContext.xml")
    public class UserDaoTest{
	    @Autowired
	    private ApplicationContext context;
	    ---
	    @Before
	    public void setup(){
		    this.dao = this.context.getBean("userDao",UserDao.class);
		    ---
		}
- context 초기화 하는 코드 없어도 JUnit확장기능이 알아서 해줌

@RunWith
: JUnit 프레임워크 테스트 실행 방법 확장
SpringJUnit4ClassRunner : 컨텍스트 프레임워크 확장 클래스 지정 -> 테스트가 사용할 애플리케이션 컨텍스트 생성 & 관리


@ContextConfiguration
:  자동으로 생성될 애플리케이션 컨텍스트의 설정파일 위치 저장


결론 : 하나의 애플리케이션 컨텍스트가 만들어져 모든 테스트 메소드에서 적용됨
하나의 테스트 클래스 뿐만 아니라 여러 테스트 클래스에서 공유 가능

#### @Autowired
-  @Autowired  붙은 인스턴스 변수 있으면 변수 타입과 일치하는 컨텍스트 내 빈 찾음
- 타입 일치하면 빈 찾으면 인스턴스 변수에 주입
- **타입에 의한 자동와이어링** : 별도의 DI 설정 없이 필드 타입정보 이용해 자동으로 빈 가져옴

- getBean() 대신 @Autowired 이용 -> UserDao 빈 직접 DI 받을 수 있음
- 가능한 인터페이스 사용 -> 느슨하게 연결

## DI와 테스트 코드

인터페이스를 두고 DI를 적용해야되는 이유
- 개발시 절대로 바뀌지 않는 것은 없음
-  인터페이스를 두고 DI 적용시 다른 차원의 서비스도입 가능
-  테스트 효율적 적용 가능

### 1. 수동 설정
상황 : UserDao가 DataSource 오브젝트를 테스트코드에서 변경해야됨
방법 : 테스트코드에 의한 DI -> 테스트 중 DAO가 사용할 DataSource obj를 바꿔줌

    @DirtiesContext
    public class UserDaoTest{
	    @Autowired
	    UserDao dao;
	    
	    @Before
	    public void setUp(){
	    ---
		    DataSource dataSource = new SingleConnectionDataSource
			    ( "jdbc:mysql://localhost/testdb", "spring","book",true);
		    dao.setDataSource(dataSource);
		 }

코드 :
xml 수정하지 않고도 테스트 코드를 통해 obj 관계 재구성 가능
xml 파일의 설정정보를 따라 구성한 오브젝트를 가져와 관계 강제로 변경

@Before
- 테스트에서 사용할 DataSource 오브젝트 생성
- setDataSource() 메소드 통해 DI해줌

@DirtiesContext
- 테스트에서 애플리케이션 컨텍스트 상태 변경 알림
- 애플리케이션 컨텍스트의 공유 허용 x
- 매번 새로운 애플리케이션 컨텍스트 만듬 -> 뒤의 테스트 영향 주지 x

### 2. 별도의 DI 설정

문제 : 매번 애플리케이션 컨텍스트 만들기 부담, 단점 많음
해결 : 빈으로 정의된 테스트 전용 설정파일 따로 만들어 사용

- 기존 설정파일 복사해서 테스트용 설정 파일 만들기 ( applicationContext.xml , test-applicationContext.xml)
- datasource 관련된 property만 변경


###  3.컨테이너 없이 테스트 만들기

테스트코드에서 직접 오브젝트 만들고 DI 하여 사용가능
=>수동 DI
@DirtiesContext
public class UserDaoTest{
@Autowired
UserDao dao;

	    @Before
	    public void setUp(){
	    ---
		    dao = new UserDao();
		    DataSource dataSource = new SingleConnectionDataSource
			    ( "jdbc:mysql://localhost/testdb", "spring","book",true);
		    dao.setDataSource(dataSource);
		 }


장점 : 애플리케이션 컨텍스트 사용 x  코드 단순, 시간 절약
단점 : 매번 새로운 UserDao 오브젝트 만들어짐

## 학습 테스트
일반적으로 자신이 만들고 있는 코드에 대한 테스트만 작성함

### 학습 테스트
자신이 만들지 않은 프레임워크나 개발팀에서 만들어 제공한 lib 등에 대한 테스트

목적
- 자신이 사용할 API나 프레임워크 기능을 테스트로 보며 사용법 익히기
- 테스트 만드려는 기술 얼마나 이해했는지 사용방법 알고있는지 검증

장점
-   다양한 조건에 따른 기능 쉽게 확인 가능
- 개발 중 참고 가능
- 호환성 검증 도와줌
- 테스트 코드 작성시 좋은 훈련 

스프링 자체에 대한 테스트 코드 좋은 훈련 가능

### Unit 테스트 오브젝트 테스트
JUnit으로 만드는 JUnit에 대한 테스트
검증 : 정말 매번 새로운 오브젝트가 생성되는가 ?

    public class JUnitTest{
	    static JunitTest testObject;
	    
	    @Test public void test1(){
		    asserThat(this, is(not(sameInstance(testObject))));
		    testObject=this;
		    }
		@Test public void test2(){
		    asserThat(this, is(not(sameInstance(testObject))));
		    testObject=this;
		    }
		@Test public void test3(){
		    asserThat(this, is(not(sameInstance(testObject))));
		    testObject=this;
		    }

-   not() : 뒤에 나오는 결과 부정하는 매처
-  is(not()) : 같지 않아야 성공
- sameInstance() : 동일성 비교 매처

스태틱 변수 testObject에 저장한 오브젝트와 새로운 테스트 오브젝트 만듬 확인
문제 : 바로 전 테스트 코드랑만 비교 가능 -> 1,3 코드 비교 불가능

    public class JUnitTest{
	    static Set<JUnitTest> testObjects = new HashSet<JUnitTest>();
	    
	    @Test public void test1(){
		    assertThat(testObjects, not(hasItem(this)));
		    testObjects.add(this);
		   }
		@Test public void test2(){
		    assertThat(testObjects, not(hasItem(this)));
		    testObjects.add(this);
		   }
		@Test public void test3(){
		    assertThat(testObjects, not(hasItem(this)));
		    testObjects.add(this);
		   }
		 }

- 스태틱 변수로 테스트 오브젝트 저장할 수 있는 컬렉션 만듬
- 테스트마다 현재 테스트 오브젝트가 컬렉션에 등록됐나 확인
- 없으면 자기자신 추가

@hasItem() : 컬렉션의 원소인지 검사하는 매쳐

### 스프링 테스트 컨텍스트 테스트
검증 : 애플리케이션 컨텍스트는 한개만 생성됨 -> 모두에서 공유되는가?
junit.xml => 새로운 설정파일 생성

애플리케이션 컨텍스트가 context 변수에 주입됐는지 확인
-> contextObject 가 null인지 확인

## 버그 테스트
오류가 있을 때 오류를 잘 드러낼 줄 수 있는 테스트
- 일단 실패하도록 만들 것

장점
- 테스트 완성도 높여줌
- 버그의 내용 명확히 분석 가능
- 기술적 문제를 해결하는데 도움이 됨
   

		





	
