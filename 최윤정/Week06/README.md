# 📗 토비의 스프링 3.1 Vol.1 스프링의 이해와 원리
## 📝 6장 AOP
**AOP**는 IoC/DI, 서비스 추상화와 더불어 **스프링의 3대 기반기술의 하나**

>AOP를 바르게 이용하려면 OOP를 대체하려고 하는 것처럼 보이는 **AOP**라는 이름 뒤에 감춰진, 그 필연적인 등장배경과 스프링이 그것을 **도입한 이유**, 그 적용을 통해 **얻을 수 있는 장점**이 무엇인지에 대한 충분한 이해 필요

**스프링의 적용된 가장 인기 있는 AOP의 적용 대상 : 선언적 트랜잭션 기능**

#### ✨ 6장에서 다루는 것 : 서비스 추상화를 통해 많은 근본적인 문제를 해결했던 트랜잭션 경계설정 기능을 AOP를 이용해 더욱 세련되고 깔끔한 방식으로 바꾸기, 그 과정에서 스프링이 AOP를 도입해야 했던 이유

## 6.1 트랜잭션 코드의 분리
#### 지금까지의 UserService 
- 서비스 추상화 기법 적용해 트랜잭션 기술에 독립적

- 메일 발송 기술과 환경에도 종속적이지 않은 깔끔한 코드

#### 그런데, 못마땅한 부분 : 트랜잭션 경계설정을 위해 넣은 코드 
- 스프링이 제공하는 깔끔한 트랜잭션 인터페이스를 썼음에도 **비즈니스 로직이 주인이어야 할 메소드** 안에 이름도 길고 무시무시하게 생긴 트랜잭션 코드가 더 많은 자리 차지;

- 하지만, **트랜잭션의 경계는 분명 비즈니스 로직의 전후에 설정돼야 하는 것**이 분명하니 **UserService의 메소드에 두긴 해야 함.**

## 깔끔한 코드를 위해서는 어떻게 해야 할까? #리팩토링
### Step 1️⃣ 메소드 분리
#### 원래 코드 특징
- **비즈니스 로직 코드 사이에 두고 트랜잭션 시작과 종료 담당하는 코드가 앞뒤에 위치**

- 트랜잭션 경계설정 코드와 비즈니스 로직 코드 간에 서로 주고받는 정보 없음
- 이 메소드에서 시작된 트랜잭션 정보는 트랜잭션 동기화 방법 통해 DAO가 알아서 활용

#### 결론 
- 트랜잭션 경계설정 코드, 비즈니스 로직 코드는 **성격 다르고 서로 주고받는 것도 없고, 완벽하게 독립적인 코드**

- 다만 이 **비즈니스 로직을 담당하는 코드가 트랜잭션의 시작과 종료 작업 사이에서 수행돼야 한다**는 사항만 지키자.

#### Fix: 비즈니스 로직과 트랜잭션 경계설정의 분리
```java
public void upgradeLevels() throws Exception {
	TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
	try{
		upgradeLevelsInternal();
		this.transactionManager.commit(status);
	} catch(Exception e){
		this.transactionManager.rollback(status);
		throw e;
	}
}

private void upgradeLevelsInternal() {
	List<User> users = userDao.getAll();
	for(User user : users){
		if(canUpgradeLevel(user)){
			upgradeLevel(user);
		}
	}
}
```
> 바꾼 코드의 장점 : 이해 편함, 수정하기에 부담 x, 실수로 트랜잭션 코드 건드릴 염려 x

<br>

### Step 2️⃣ DI를 이용한 클래스의 분리
#### 그렇지만 여전히 트랜잭션을 담당하는 기술적인 코드가 UserService안에 존재 🤔
> 어차피 서로 직접적으로 정보 주고받는 게 없으면, **아예 트랜잭션 코드가 존재하지 않는 것처럼 사라지게 할 수 없을까?**
> 
> **적어도 UserService에서는 보이지 않게** 할 수 있지 않을까?
#### ✅ 간단하게 트랜잭션 코드를 클래스 밖으로 뽑아내면 된다.

> DI의 기본 아이디어는 실제 사용할 오브젝트 클래스 정체는 감춘 채 인터페이스를 통해 간접으로 접근하는 것. 
>
>그 덕분에 구현 클래스는 얼마든지 외부에서 변경 가능!

#### 목표
> UserService에는 순수하게 비즈니스 로직을 담고 있는 코드만 놔두고 트랜잭션 경계설정을 담당하는 코드를 외부로 빼내기

#### 현재 구조
> UserService 클래스와 그 사용 클라이언트(UserServiceTest)의 직접 연결을 통한 강한 결합

#### 첫 번째 아이디어 : UserService를 인터페이스로 만들고, 기존 코드는 UserService 인터페이스의 구현 클래스를 만들어넣도록 하자 => 결합이 약해지고, 유연한 확장 가능
> **보통 이렇게 인터페이스를 이용해 구현 클래스를 클라이언트에 노출하지 않고 런타임 시에 DI를 통해 적용하는 방법 쓰는 이유 :  구현 클래스를 바꿔가면서 사용하기 위해**
> 
>- 테스트 때는 필요에 따라 테스트 구현 클래스, 정식 운영 중에는 정규 구현 클래스를 DI 해주는 방법처럼 한 번에 한 가지 클래스 선택해 적용하도록 되어 있음.
>
>- 근데 꼭 그래야 한다는 제약 X

#### 두 번째 아이디어 : UserService를 구현한 또 다른 구현 클래스(UserServiceTx)를 만들자 => 여기서는 이 방법 사용 ✅
>- 이 클래스는 사용자 관리 로직 담고 있는 구현 클래스인 UserServiceImpl 대신하기 위해 만든 거 x
>
>- 단지 트랜잭션의 경계설정이라는 책임 맡고 있을 뿐
>- 스스로는 비즈니스 로직을 담고 있지 X -> 다른 비즈니스 로직을 담고 있는 UserService의 구현 클래스에 실제적인 로직 처리 작업은 위임 

<br>

### 요약 : 
- 기존 UserService 클래스 이름 UserServiceImpl로 변경

- 클라이언트가 사용할 로직 담은 핵심 메소드만 UserService 인터페이스로 만든 후 UserServiceImpl이 구현하도록 만들기

#### Feat: UserService 인터페이스 추가
```java
public interface UserService{
	void add(User user);
	void upgradeLevels();
}
```
#### Fix: UserService 구현 클래스에 트랜잭션 코드를 제거
```java
package springbook.user.service;
...
public class UserServiceImpl implements UserService{
	UserDao userDao;
	MailSender mailSender;
	
	public void upgradeLevels(){
		List<User> users = userDao.getAll();
		for(User user : users){
			if(canUpgradeLevel(user)){
				upgradeLevel(user);
			}
		}
	}
	...
}
```
#### Feat: 위임 기능을 가진 UserServiceTx 클래스 추가 
```java
package springbook.user.service;
...
public class UserServiceTx implements UserService{
	UserService userService;
	
	public void setUserService(UserService userService){
		this.userService = userService;
	}
	
	public void add(User user){
		userService.add(user);
	}
	
	public void upgradeLevels(){
		userService.upgradeLevels();
	}
}
```
> UserService를 구현한 다른 오브젝트를 DI 받음
>
> DI 받은 UserService 오브젝트에 모든 기능 위임
#### Feat: UserServiceTx에 트랜잭션 적용
```java
public class UserServiceTx implements UserService{
	UserService userService;
	PlatformTransactionManager transactionManager;
	
	public void setTransactionManager(PlatformTransactionManager transactionManager){
		this.transactionManager = transactionManager;
	}
	
	public void setUserService(UserService userService){
		this.userService = userService;
	}
	
	public void add(User user){
		userService.add(user);
	}
	
	public void upgradeLevels(){
		TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
		try{
			userService.upgradeLevels();
			this.transactionManager.commit(status);
		} catch(RuntimeException e){
			this.transactionManager.rollback(status);
			throw e;
		}
	}
}
```
> transactionManager 라는 이름의 빈으로 등록된 트랜잭션 매니저를 DI로 받아뒀다가 트랜잭션 안에서 동작하도록 만들어줘야 하는 메소드 호출의 전과 후에 필요한 트랜잭션 경계설정 API를 사용해주면 됨.
#### Fix : 설정파일에 트랜잭션 오브젝트 추가
```xml
<bean id="userService" class="springbook.user.service.UserServiceTx">
	<property name="transactionManager" ref="transactionManager" />
	<property name="userService" ref="userServiceImpl" />
</bean>

<bean id="userServiceImpl" class="springbook.user.service.UserServiceImpl">
	<property name="userDao" ref="userDao" />
	<property name="mailSender" ref="mailSender" />
</bean>
```
> userService라는 대표적인 빈 아이디는 UserServiceTx 클래스로 정의된 빈에게 부여
> 
> userService 빈은 UserServiceImpl 클래스로 정의되는, 아이디가 userServiceImpl인 빈을 DI하게 만듦

<br>

### Step 3️⃣ 트랜잭션 분리에 따른 테스트 수정
#### ✅ 먼저 스프링의 테스트용 컨텍스트에서 가져올 빈들을 생각해보자.
- 기존 : UserService 클래스 타입의 빈을 `@Autowired`로 가져다가 사용

- 현재 : UserService는 이제 인터페이스로 바뀜 
- 인터페이스라고 해도 `@Autowired`로 가져오는 데는 아무 문제 없음.
- `@Autowired`는 기본적으로 타입을 이용해 빈을 찾지만, 만약 타입으로 하나의 빈을 결정할 수 없는 경우엔 필드 이름을 이용해 빈을 찾음.
- **UserServiceTest에서 다음과 같은 userService 변수를 설정**해두면, 아이디가 userService인 빈이 주입됨

	```java
	@Autowired UserService userService;
	```
<br>

#### ✅  그런데, UserServiceTest는 하나의 빈을 더 가져와야 함
- UserServiceImpl 클래스로 정의된 빈

- 목 오브젝트를 이용해 수동 DI를 적용하는 테스트라면 어떤 클래스의 오브젝트인지 분명하게 알 필요 있음. 
- **다음과 같이 UserServiceImpl 클래스 타입의 변수 선언하고 `@Autowired`를 지정해 해당 클래스로 만들어진 빈을 주입받도록 하자** (그래야 MockMailSender를 설정해주기 위한 수정자 메소드에 접근 가능)
	```java
	@Autowired UserServiceImpl userServiceImpl;
	```

<br>

**✅  `upgradeLevels()` 테스트 메소드에서 MailerSender의 목 오브젝트 설정해주는 건 이제 userService 인터페이스를 통해선 불가능 
=> 별도로 가져온 userServiceImpl 빈에 해주자.**
#### Fix: 목 오브젝트 설정이 필요한 테스트 코드 수정
```java
@Test
public void upgradeLevels() throws Exception {
	...
	MockMailSender mockMailSender = new MockMailSender();
	userServiceImpl.setMailSender(mockMailSender);
}
```

<br>

**✅  TestUserService가 트랜잭션 기능은 빠진 UserServiceImpl을 상속하도록 해야 함 
=> TestUserService 오브젝트를 UserServiceTx 오브젝트에 수동 DI 시킨 후에 트랜잭션 기능까지 포함된 UserServiceTx의 메소드를 호출하며 테스트 수행하도록 하자.**
#### Fix: 분리된 테스트 기능이 포함되도록 수정한 `upgradeAllOrNothing()` 
```java
@Test
public void upgradeAllOrNothing() throws Exception {
	TestUserService testUserService = new TestUserService(users.get(3).getId());
	testUserService.setUserDao(userDao);
	testUserService.setMailSender(mailSender);
	
	// 트랜잭션 기능을 분리한 UserServiceTx는 예외 발생용으로 수정할 필요가 없으니 그대로 사용
	UserServiceTx txUserService = new UserServiceTx();
	txUserService.setTransactionManager(transactionManager);
	txUserService.setUserService(testUserService);

	userDao.deleteAll();
	for(User user : users) userDao.add(user);
	
	try{
		txUserService.upgradeLevels(); // 트랜잭션 기능을 분리한 오브젝트를 통해 예외 발생용 TestUserService가 호출되게 해야 함
		fail("TestUserServiceException expected");
	}
	...
}
```
> 테스트 준비 코드가 길어지긴 했지만 원래 하나의 오브젝트만 만들던 것을 두 개로 나눴을 뿐이지 내용 달라지지 x


**트랜잭션 테스트용으로 특별히 정의한 TestUserService 클래스는 이제 UserServiceImpl 클래스를 상속하도록 바꾸자.**
```java
static class TestUserService extends UserServiceImpl{
```

<br>

### ✅ 트랜잭션 경계설정 코드 분리의 장점
1. 비즈니스 로직을 담당하고 있는 UserServiceImpl의 코드를 작성할 때는 트랜잭션과 같은 기술적인 내용에 전혀 신경쓰지 않아도 됨.
	- 언제든지 트랜잭션 도입 가능 

2. 비즈니스 로직에 대한 테스트를 손쉽게 만들어낼 수 있음

	- 이에 대해서는 다음 절에서 좀 더 자세히 살펴보자 👀

<br>

## 6.2 고립된 단위 테스트
#### 가장 편하고 좋은 테스트 방법 : 가능한 한 작은 단위로 쪼개서 테스트하는 것

#### 작은 단위의 테스트가 좋은 이유 :
- 테스트가 실패했을 때, 그 원인 찾기 쉬움

- 테스트 단위가 작아야 테스트의 의도나 내용이 분명해지고, 만들기도 쉬워짐
- 작은 단위의 테스트로 검증한 부분은 제외하고 접근 가능

#### 하지만, 작은 단위로 테스트하고 싶어도 그럴 수 없는 경우가 많음
- 예) 테스트 대상이 다른 오브젝트와 환경에 의존하고 있는 경우

<br>

### 복잡한 의존관계 속의 테스트
🧐 UserService의 경우를 생각해보자.
- UserService의 구현 클래스들이 동작하려면 세 가지 타입의 의존 오브젝트가 필요

	- UserDao 타입의 오브젝트를 통해 DB와 데이터를 주고받아야 하고, MailSender를 구현한 오브젝트를 이용해 메일을 발송해야 함

	- 트랜잭션 처리를 위해 PlatformTransactionManager와 커뮤니케이션이 필요함
	
다시 말해, **UserService는 UserDao, TransactionManager, MailSender라는 세 가지 의존관계를 갖고 있음**
- 그 세 가지 의존관계를 갖는 오브젝트들이 테스트가 진행되는 동안에 같이 실행됨.

- 더 큰 문제는 세 가지 의존 오브젝트도 자신의 코드만 실행하고 마는 게 아님.
- UserService를 테스트하는 것처럼 보이지만 **사실은 그 뒤에 존재하는 훨씬 더 많은 오브젝트와 환경, 서비스, 서버, 심지어 네트워크까지 함께 테스트**하는 셈이 됨.

✅ **따라서, 테스트의 대상이 환경이나, 외부 서버, 다른 클래스의 코드에 종속되고 영향을 받지 않도록 고립시킬 필요가 있다.**

<br>

## 테스트 대상 오브젝트 고립시키기
#### 테스트를 의존 대상으로부터 분리해서 고립시키는 방법 : 테스트를 위한 대역을 사용
5장에서
- MailSender에는 이미 DummyMailSender라는 테스트 스텁을 적용했음

- 테스트 대역이 테스트 검증에도 참여할 수 있도록, 특별히 만든 MockMailSender라는 목 오브젝트도 사용해봄

<br>

### 테스트를 위한 UserServiceImpl 고립
고립된 테스트가 가능하도록 **UserService를 재구성**하자.
- UserServiceImpl에 대한 테스트가 진행될 때 사전에 테스트를 위해 준비된 동작만 하도록 만든 **두 개의 목 오브젝트(MockUserDao, MockMailSender) 를 이용**하자.

- **UserDao**를 단지 테스트 대상의 코드가 정상적으로 수행되도록 도와주기만 하는 스텁이 아니라, 부가적인 검증 기능까지 가진 **목 오브젝트로 만든 이유** : 
	- 고립된 환경에서 동작하는 `upgradeLevels()`의 테스트 결과를 검증할 방법이 필요하기 때문

	- UserServiceImpl의 `upgradeLevels()` 메소드는 리턴 값이 없는 void형 => 메소드 실행하고 그 결과 받아서 검증하는 거 아예 불가능 => 코드 동작 바르게 됐는지 확인하려면 DB 직접 확인할 수밖에 없음 
	- **테스트 대상인 UserServiceImpl과 협력 오브젝트인 UserDao에게 어떤 요청을 했는지 확인하는 작업이 필요함.**
		- UserDao의 `update()` 메소드 호출 확인 = DB에 그 결과가 반영될 것이라고 결론 내릴 수 있음

		- 결론 : **UserDao**와 같은 역할하면서 UserServiceImpl과의 사이에서 주고받은 정보를 저장해뒀다가, 테스트의 검증에 사용할 수 있게 하는 **목 오브젝트 만들 필요 있음**

<br>

### ✅ 고립된 단위 테스트 활용
고립된 단위 테스트 방법을 UserServiceTest의 `upgradeLevels()` 테스트에 적용해보자.

#### 기존 `upgradeLevels()` 테스트의 구성
1. 테스트 실행 중 UserDao를 통해 가져올 테스트용 정보를 DB에 넣음. UserDao는 결국 DB를 이용해 정보 가져오기 때문에 최후의 의존 대상인 DB에 직접 정보 넣어줘야 함
	- **의존관계 따라 마지막에 등장하는 DB 준비**

2. 메일 발송 여부 확인 위해 MailSender 목 오브젝트를 DI 해줌
	- 테스트를 의존 오브젝트와 서버 등에서 고립시키도록 테스트만을 위한 **목 오브젝트**를 준비
3. 실제 테스트 대상인 userService의 메소드를 실행
4. 결과가 DB에 반영됐는지 확인하기 위해 UserDao를 이용해 DB에서 데이터를 가져와 결과 확인
	- **의존관계를 따라 결국 최종 결과가 반영된 DB의 내용을 확인하는 방법**
5. 목 오브젝트를 통해 UserService에 의한 메일 발송이 있었는지를 확인
	- 메일 서버까지 갈 필요 없이 **목 오브젝트**를 통해 `upgradeLevels()` 메소드가 실행되는 중에 메일 발송 요청이 나간 적이 있는지만 확인

실제 UserDao와 DB까지 직접 의존하고 있는 **첫 번째와 네 번째**의 테스트 방식도 **목 오브젝트**를 만들어서 적용해보자.

<br>

#### `upgradeLevels()` 메소드 실행되는 중에 UserDao와 어떤 정보 주고받는지 입출력 내역 확인하자.
- `List<User> users = userDao.getAll();` 레벨 업그레이드 후보가 될 사용자 목록 가져옴 => 스텁 필요

- `userDao.update(user);`  수정된 사용자 정보를 DB에 반영
	- `update()` 메소드의 사용 `upgradeLevels()` 의 핵심 로직인 '전체 사용자 중에서 업그레이드 대상자는 레벨을 변경해준다'에서 '변경'에 해당하는 부분 검증가능한 중요 기능 => 목 오브젝트 필요

#### 1️⃣ Feat: UserDao 오브젝트 추가
```java
static class MockUserDao implements UserDao{
	private List<User> users; // 레벨 업그레이드 후보 User 오브젝트 목록
	private List<User> updated = new ArrayList(); // 업그레이드 대상 오브젝트를 저장해둘 목록

	private MockUserDao(List<User> users){
		this.users = users;
	}

	public List<User> getUpdated(){
		return this.updated;
	}
	
	// 스텁 기능 제공
	public List<User> getAll() {
		return this.users;
	}
	
	// 목 오브젝트 기능 제공
	public void update(User user){
		updated.add(user);
	}
	
	// 테스트에 사용되지 않는 메소드들
	public void add(User user) { throw new UnsupportedOperationException(); }
	public void deleteAll() { throw new UnsupportedOperationException(); }
	public User get(String id) { throw new UnsupportedOperationException(); }
	public int getCount() { throw new UnsupportedOperationException(); }
}
```
> `getAll()`에 대해서는 **스텁**으로서, `update()`에 대해서는 **목 오브젝**트로서 동작하는 UserDao 타입의 테스트 대역. 
>
> 사용하지 않을 메소드들은 UnsupportedOperationException을 던지게 해서 지원하지 않는 기능이라는 예외가 발생하도록 만드는 게 좋음

#### 2️⃣ Fix: MockUserDao를 사용해서 만든 고립된 테스트
```java
@Test
public void upgradeLevels() throws Exception {
	// 고립된 테스트에서는 테스트 대상 오브젝트를 직접 생성하면 됨
	UserServiceImpl userServiceImpl = new UserServiceImpl(); 
	
	// 목 오브젝트로 만든 UserDao를 직접 DI 해줌
	MockUserDao mockUserDao = new MockUserDao(this.users);
	userServiceImpl.setUserDao(mockUserDao);
	
	MockMailSender mockMailSender = new MockMailSender();
	userServiceImpl.setMailSender(mockMailSender);
	
	userServiceImpl.upgradeLevels();

	List<User> updated = mockUserDao.getUpdated(); // MockUserDao로부터 업데이트 결과 가져옴
	
	// 업데이트 횟수와 정보 확인
	assertThat(updated.size(), is(2));
	checkUserAndLevel(updated.get(0), "joytouch", Level.SILVER);
	checkUserAndLevel(updated.get(1), "madnite1", Level.GOLD);
	
	List<String> request = mockMailSender.getRequests();
	assertThat(request.size(), is(2));
	assertThat(request.get(0), is(users.get(1).getEmail()));
	assertThat(request.get(1), is(users.get(3).getEmail()));
}

// id와 level을 확인하는 간단한 헬퍼 메소드
private void checkUserAndLevel(User updated, String expectedId, Level expectedLevel){
	assertThat(updated.getId(), is(expectedId));
	assertThat(updated.getLevel(), is(expectedLevel));
}
```
> `upgradeLevels()`의 테스트 수행시간은 이전보다 분명히 빨라짐. => JUnit으로 0.000초 나옴
>
>JUnit의 테스트 시간 측정 위한 최소 단위 : 0.001초
>
>어떻게 1밀리초도 되지 않는 짧은 시간에 나름 복잡한 비즈니스 로직 가진 테스트가 실행될 수 있었을까?
>- 두 개의 목 오브젝트 외에는 사용자 관리 로직을 검증하는 데 직접적으로 필요하지 않은 의존 오브젝트와 서비스를 모두 제거한 덕분.
>
>- 최대로 잡아서 0.001초 걸렸다고 쳐도 평균하면 DB를 사용한 여타 테스트와 비교해서 500배 차이남

#### 결론
> **고립된 테스트**를 만들려면 목 오브젝트 작성과 같은 약간의 수고가 더 필요할지 모르겠지만, **테스트 수행 성능이 크게 향상**됨.

<br>

## 단위 테스트와 통합 테스트
### ✅ 단위 테스트
> **테스트 대상 클래스를 목 오브젝트 등의 테스트 대역을 이용해 의존 오브젝트나 외부의 리소스를 사용하지 않도록 고립시켜서 테스트하는 것**
> 
> 하나의 단위에 초점을 맞춘 테스트

### ✅ 통합 테스트
> **두 개 이상의, 성격이나 계층이 다른 오브젝트가 연동하도록 만들어 테스트하거나, 또는 외부의 DB나 파일, 서비스 등의 리소스가 참여하는 테스트**
> 
> 두 개 이상의 단위가 결합해서 동작하면서 테스트가 수행되는 것
>
>스프링의 테스트 컨텍스트 프레임워크를 이용해서 컨텍스트에서 생성되고 DI된 오브젝트를 테스트하는 것도 통합 테스트

<br>

### 🧐 그렇다면, 단위 테스트와 통합 테스트 중 어떤 방법을 쓸지는 어떻게 결정?
#### [가이드라인]
- **항상 단위 테스트를 먼저 고려**

- 하나의 클래스나 성격과 목적 같은 긴밀한 클래스 몇 개를 모아 외부와의 의존관계를 모두 차단하고 필요에 따라 스텁이나 목 오브젝트 등의 테스트 대역을 이용하도록 테스트를 만듦. 
	- 단위 테스트 : 테스트 작성 간단, 실행 속도 빠름, 테스트 대상 외의 코드 or 환경으로부터 테스트 결과에 영향 받지 x => 가장 빠른 시간에 효과적인 테스트를 작성하기에 유리
- **외부 리소스**를 **사용**해야만 가능한 테스트는 **통합 테스트**로 만듦
- **단위 테스트로 만들기 어려운 코드 존재**
	- 대표적인 게 **DAO** 

	- DAO : 그 자체로 로직 담고 있기 보단, DB를 통해 로직 수행하는 **인터페이스 같은 역할**을 함
	- SQL을 JDBC를 통해 실행하는 코드만으로는 고립된 테스트를 작성하기가 힘듦
	- 작성한다고 해도 가치가 없는 경우가 대부분
	- DAO는 DB까지 연동하는 테스트로 만드는 편이 효과적.
	- **DB를 사용하는 테스트는 DB에 테스트 데이터를 준비하고, DB에 직접 확인을 하는 등의 부가적인 작업 필요**
- **DAO 테스트**는 DB라는 외부 리소스를 사용하기 때문에 **통합 테스트**로 분류됨.
	- 하지만 코드에서 보자면 하나의 기능 단위를 테스트하는 것이기도 함

	- DAO를 테스트를 통해 충분히 검증해두면, DAO를 이용하는 코드는 DAO의 역할이나 목 오브젝트로 대체해서 테스트 가능
	- 물론 각각의 단위 테스트가 성공했더라도, 여러 개의 단위를 연결해서 테스트하면 오류 발생할 수 있음.
	- 하지만 충분한 단위 테스트를 거친다면 통합 테스트에서 오류가 발생할 확률도 줄어들고 발생한다고 해도 쉽게 처리 가능
- **여러 개의 단위가 의존관계를 가지고 동작할 때를 위한 통합 테스트는 필요**. **다만, 단위 테스트를 충분히 거쳤다면 통합 테스트의 부담은 상대적으로 줄어듦.**
- **단위 테스트를 만들기가 너무 복잡하다고 판단되는 코드**는 **처음부터 통합 테스트를 고려**해봄. 이때도 통합 테스트에 참여하는 코드 중에서 가능한 한 많은 부분을 미리 단위 테스트로 검증해두는 게 유리
- **스프링 테스트 컨텍스트 프레임워크를 이용하는 테스트**는 **통합 테스트**
	- 가능하면 스프링의 지원 없이 직접 코드 레벨의 DI를 사용하면서 단위 테스트를 하는 게 좋겠지만, 스프링의 설정 자체도 테스트 대상이고, 스프링을 이용해 좀 더 추상적인 레벨에서 테스트해야 할 경우도 종종 존재 => 스프링 테스트 컨텍스트 프레임워크를 이용해 통합 테스트를 작성함

<br>

## ✅ 목 프레임워크
단위 테스트 만들기 위해서 스텁 or 목 오브젝트 사용이 필수적
> 의존관계가 없는 단순한 클래스나 세부 로직을 검증하기 위해 메소드 단위로 테스트할 때가 아니라면, 대부분 의존 오브젝트를 필요로 하는 코드를 테스트하게 되기 때문

목 오브젝트를 편리하게 작성하도록 도와주는 다양한 목 오브젝트 지원 프레임워크가 있음.

### 🤗 Mockito 프레임워크
사용하기 편리, 코드 직관적이라 최근 인기 많아짐.
#### Mockito 같은 목 프레임워크 특징
- 목 클래스 일일이 준비할 필요 x 

- 간단한 메소드 호출만으로 다이내믹하게 특정 인터페이스를 구현한 테스트용 목 오브젝트 만들 수 있음

#### 1️⃣ UserDao 인터페이스를 구현한 테스트용 목 오브젝트 만드는 법
```java
UserDao mockUserDao = mock(UserDao.class);
```
> Mockito의 스태틱 메소드 `mock()` 한 번 호출해주면 만들어짐
>
>`mock()` 메소드는 org.mockito.Matchers 클래스에 정의된 스태틱 메소드

#### 2️⃣ `getAll()` 메소드가 불려올 때 사용자 목록을 리턴하도록 스텁 기능 추가하는 법
```java
when(mockUserDao.getAll()).thenReturn(this.users);
```
> `mockUserDao.getAll()`이 호출됐을 때(`when`), users 리스트를 리턴해주라(`thenReturn`)는 선언

#### 3️⃣ mockUserDao의 `update()` 메소드가 두 번 호출됐는지 검증하는 방법
```java
verify(mockUserDao, times(2)).update(any(User.class));
```
> User 타입의 오브젝트를 파라미터로 받으며 `update()` 메소드가 두 번 호출됐는지(`times(2)`) 확인하라(`verify`)는 것
>
>Mockito를 통해 만들어진 목 오브젝트는 메소드의 호출과 관련된 모든 내용을 자동으로 저장해두고, 이를 간단한 메소드로 검증할 수 있게 해줌.

<br>

### 📝 Mockito 목 오브젝트 사용법 정리
두 번째와 네 번째는 각각 필요한 경우에만 사용 가능
1. 인터페이스를 이용해 목 오브젝트를 만듦.

2. 목 오브젝트가 리턴할 값이 있으면 이를 지정해줌. 메소드가 호출되면 예외를 강제로 던지게 만들 수도 있음.
3. 테스트 대상 오브젝트에 DI 해서 목 오브젝트가 테스트 중에 사용되도록 만듦.
4. 테스트 대상 오브젝트를 사용한 후에 목 오브젝트의 특정 메소드가 호출됐는지, 어떤 값을 가지고 몇 번 호출됐는지를 검증함.

> 특별한 기능 가진 목 오브젝트를 만들어야 하는 경우가 아니라면, 거의 대부분의 단위 테스트에서 필요한 목 오브젝트는 Mockito를 사용하는 것으로 충분함

#### Fix: Mockito를 적용한 테스트 코드
```java
@Test
public void mockUpgradeLevels() throws Exception {
	UserServiceImpl userServiceImpl = new UserServiceImpl();
	
	UserDao mockUserDao = mock(UserDao.class); // 다이내믹한 목 오브젝트 생성
	when(mockUserDao.getAll()).thenReturn(this.users); // 메소드의 리턴 값 설정
	userServiceImpl.setUserDao(mockUserDao); // DI
	
	MailSender mockMailSender = mock(MailSender.class);
	userServiceImpl.setMailSender(mockMailSender);

	userServiceImpl.upgradeLevels();
	
	verify(mockUserDao, times(2)).update(any(User.class));
	verify(mockUserDao, times(2)).update(any(User.class));
	verify(mockUserDao).update(users.get(1));
	assertThat(users.get(1).getLevel(), is(Level.SILVER));
	verify(mockUserDao).update(users.get(3));
	assertThat(users.get(3).getLevel(), is(Level.GOLD));
	
	ArgumentCaptor<SimpleMailMessage> mailMessageArg = ArgumentCaptor.forClass(SimpleMailMessage.class);
	verify(mockMailSender, times(2)).send(mailMessageArg.capture());
	List<SimpleMailMessage> mailMessages = mailMessageArg.getAllValues();
	assertThat(mailMessages.get(0).getTo()[0], is(users.get(1).getEmail()));
	assertThat(mailMessages.get(1).getTo()[0], is(users.get(3).getEmail()));
}
```
> `times()` : 메소드 호출 횟수 검증
>
>`any()` : 파라미터의 내용은 무시하고 호출 횟수만 확인
>
>`ArgumentCaptor` : 실제 목 오브젝트에 전달된 파라미터를 가져와 내용을 검증. 
>
>- 파라미터를 직접 비교하기보단 **파라미터의 내부 정보 확인해야 하는 경우에 유용**

<br>

## 6.3 다이내믹 프록시와 팩토리 빈
분리된 부가기능을 담은 클래스는 부가기능 외에 나머지 모든 기능은 원래 핵심기능을 가진 클래스로 위임해줘야 한다.

💥 문제 : 클라이언트가 핵심기능 가진 클래스를 직접 사용 -> 부가기능이 적용될 기회가 없음

✅ 해결책 : 부가기능은 마치 자신이 핵심기능을 가진 클래스인 것처럼 꾸며서, 클라이언트가 자신을 거쳐서 핵심기능을 사용하도록 만들어야 함
- 그러기 위해서는 클라이언트는 인터페이스를 통해서만 핵심 기능을 사용하게 하고, 부가기능 자신도 같은 인터페이스를 구현한 뒤에 자신이 그 사이에 끼어들어야 함.

<br>

## ✅ 프록시(proxy)란?
- 마치 자신이 **클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 것**을 대리자, 대리인과 같은 역할을 한다고 해서 프록시(proxy)라고 부름

- 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트를 '**타깃(trarget)**', 또는 '**실체(real subject)**'라고 부름

### 프록시 특징
- 타깃과 같은 인터페이스를 구현

- 프록시가 타깃을 제어할 수 있는 위치에 있음


### 프록시는 사용 목적에 따라 두 가지로 구분
1. **타깃에 부가적인 기능을 부여**해주기 위해서  => **데코레이터 패턴**

2. **클라이언트가 타깃에 접근하는 방법을 제어**하기 위해서 => **프록시 패턴**

<br>

## ✅ 프록시 사용하는 방법 - 1) 데코레이터 패턴
#### **타깃에 부가적인 기능을 런타임 시 다이내믹하게 부여해주기 위해 프록시를 사용하는 패턴**
> 다이내믹하게 기능을 부가한다 = **컴파일 시점, 즉 코드상에서는 어떤 방법과 순서로 프록시와 타깃이 연결되어 사용되는지 정해져 있지 않다**는 뜻.

### 데코레이터라고 불리는 이유
- 마치 제품이나 케익 등을 여러 겹으로 포장하고 그 위에 장식을 붙이는 것처럼 실제 내용물은 동일하지만 부가적인 효과를 부여해줄 수 있기 때문

### 데코레이터 패턴 특징
- 프록시가 꼭 한 개로 제한되지 않고, 직접 타깃을 사용하도록 고정시킬 필요 X
	
	=> 같은 인터페이스를 구현한 타깃과 여러 개의 프록시를 사용할 수 있음 
- 여러 개인 만큼 순서 정해서 단계적으로 위임하는 구조로 만들면 됨
- 프록시로서 동작하는 각 데코레이터는 위임하는 대상에도 인터페이스로 접근 -> 자신이 최종 타깃으로 위임하는지, 다음 단계의 데코레이터 프록시로 위임하는지 알지 못함
	- 다음 위임 대상 : 인터페이스로 선언 + 생성자나 수정자 메소드 통해 위임 대상을 외부에서 런타임 시에 주입받을 수 있도록 만들어야 함

### 데코레이터 패턴 사용 예시
1️⃣ 자바 IO 패키지의 InputStream과 OutputStream 구현 클래스
```java
InputStream is = new BufferedInputStream(new FileInputStream("a.txt"));
```
> InputStream이라는 인터페이스를 구현한 타깃인 FileInputStream에 버퍼 읽기 기능을 제공해주는 BufferedInputStream이라는 데코레이터를 적용

2️⃣ UserService 인터페이스를 구현한 타깃인 UserServiceImpl에 트랜잭션 부가기능을 제공해주는 UserServiceTx를 추가한 것
- 수정자 메소드를 이용해 데코레이터인 UserServiceTx에 위임할 타깃인 UserServiceImpl을 주입해줌

### 데코레이터 패턴 적용 방법 : 스프링의 DI 이용
- 데코레이터 빈의 프로퍼티로 같은 인터페이스를 구현한 다른 데코레이터 또는 타깃 빈을 설정하면 됨

### 데코레이터 패턴은 언제 사용할까?
- 타깃의 코드를 손대지 않고, 클라이언트가 호출하는 방법도 변경하지 않은 채로 새로운 기능을 추가할 때 유용함

<br>

## ✅  프록시 사용하는 방법 - 2) 프록시 패턴
### 일반적으로 사용하는 '프록시' 라는 용어 VS 디자인 패턴에서 말하는 '프록시 패턴'
- 전자 : 클라이언트와 사용 대상 사이에 대리 역할을 맡은 오브젝트

- 후자 : 프록시를 사용하는 방법 중 **타깃에 대한 접근 방법을 제어하려는 목적을 가진 경우**

### 프록시 패턴의 '프록시'
- 타깃의 기능을 확장하거나 추가하지 X

- 대신 클라이언트가 타깃에 접근하는 방식을 변경해줌

### 프록시 패턴은 언제 사용할까?
- **타깃 오브젝트에 대한 레퍼런스가 미리 필요한 경우**

	- 실제 타깃 오브젝트를 만드는 대신 프록시를 넘기고 프록시의 메소드를 통해 타깃을 사용하려고 시도하면 그 때 프록시가 타깃 오브젝트를 생성 + 요청 위임해주는 식
- **원격 오브젝트를 이용하는 경우**
	- 각종 리모팅 기술 이용해 다른 서버에 존재하는 오브젝트를 사용해야 할 경우, 원격 오브젝트에 대한 프록시 만들어두고 클라이언트는 마치 로컬에 존재하는 오브젝트를 쓰는 것처럼 프록시 사용 가능

	- 프록시는 클라이언트의 요청을 받으면 네트워크를 통해 원격의 오브젝트를 실행하고 결과를 받아 클라이언트에게 돌려줌
- **특별한 상황에서 타깃에 대한 접근권한을 제어하려고 할 때**

	- 전형적인 접근권한 제어용 프록시 
	
		ex) `Collections`의 `unmodifiableCollection()`을 통해 만들어지는 오브젝트

### 프록시와 데코레이터 차이점
- 프록시는 코드에서 자신이 만들거나 접근할 타깃 클래스 정보를 알고 있는 경우가 많음
- 물론 프록시 패턴이라고 하더라도 인터페이스를 통해 위임하도록 만들 수도 있음

<br>

이 책에서 앞으로는 **타깃과 동일한 인터페이스를 구현하고, 클라이언트와 타깃 사이에 존재하면서 기능의 부가 또는 접근 제어를 담당하는 오브젝트**를 모두 **프록시**라고 부를 것임.

하지만, 그때마다 **사용 목적**이 '**기능의 부가**'인지 '**접근 제어**'인지 구분해보면 각각 어떤 목적으로 프록시가 사용됐는지, 그에 따라 어떤 패턴이 적용됐는지 알 수 있을 것.
- 기능의 부가 = 데코레이터 패턴
- 접근 제어 = 프록시 패턴

<br>

## ✅ 다이내믹 프록시
**프록시**는 **기존 코드에 영향을 주지 않으면서 타깃의 기능을 확장하거나, 접근 방법을 제어할 수 있는 유용한 방법**

프록시를 일일이 모든 인터페이스 구현하고 클래스 새로 정의해서 만드는 일은 상당히 번거로움.

🤗 그래서, 자바에는 `java.lang.reflect` 패키지 안에 프록시를 손쉽게 만들 수 있도록 지원해주는 클래스들이 있음.
> **몇 가지 API를 이용해 프록시처럼 동작하는 오브젝트를 다이내믹하게 생성 가능**

<br>

### 🧐 프록시의 구성과 프록시 작성의 문제점
#### 프록시의 구성
- 타깃과 같은 메소드를 구현하고 있다가 메소드가 호출되면 타깃 오브젝트로 위임한다.
- 지정된 요청에 대해서는 부가기능을 수행한다.

#### 프록시의 기능을 구분해보기 위한 예시 코드 
```java
public class UserServiceTx implements UserService{
	UserService userService; // 타깃 오브젝트
	...
		
	// 메소드 위임과 구현
	public void add(User user){
		this.userService.add(user);
	}
	
	// 메소드 구현	
	public void upgradeLevels() {
		// 부가기능 수행
		TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
		try{
		
			userService.upgradeLevels(); // 위임
	
		// 부가기능 수행
			this.transactionManager.commit(status);
		} catch (RuntimeException e){
			this.transactionManager.rollback(status);
			throw e;
		}
	}
}
```

#### 프록시의 역할 두 가지
1. 위임

2. 부가작업

#### 프록시 만들기가 버거운 이유 두 가지
1. 타깃의 인터페이스를 구현하고 위임하는 코드를 작성하기가 번거로움
	=> ✅ **JDK의 다이내믹 프록시 사용해서 해결** 

2. 부가기능 코드가 중복될 가능성이 많음
	=> 중복 코드 분리해서 해결하면 될 것 같음

<br>

**다이내믹 프록시는 리플렉션 기능을 이용해서 프록시를 만들어준다.**
### ✅ 리플렉션 
- **자바의 코드 자체를 추상화해서 접근하도록 만든 것.**

- 자바의 모든 클래스는 그 클래스 자체의 구성정보를 담은 Class 타입의 오브젝트를 하나씩 갖고 있음.
	- '`클래스이름.class`' 또는 `오브젝트의 getClass()` 메소드 호출하면, 클래스 정보 담은 Class 타입의 오브젝트 가져올 수 있음
	- 클래스 오브젝트를 이용하면, 클래스 코드에 대한 메타정보를 가져오거나 오브젝트를 조작할 수 있음
	- 클래스 이름이 무엇인지 / 어떤 클래스를 상속하는지 / 어떤 인터페이스를 구현했는지 / 어떤 필드를 갖고 있는지 / 각각의 타입은 무엇인지 / 메소드를 어떤 것을 정의했고 메소드의 파라미터와 리턴타입은 무엇인지 등 
	
	- 오브젝트 필드의 값 읽고 수정 가능, 원하는 파라미터 값 이용해 메소드 호출 가능
	- 자세한 사항은 `java.lang.reflect` 패키지 자바문서 참조

<br>

### `Method` 인터페이스 이용해 메소드 호출
예를 들어, String의 `length()` 메소드라면, 
```java
Method lengthMethod = String.class.getMethod("length");
```
> 스트링이 가진 메소드 중에서 "length"라는 이름을 갖고 있고, 파라미터는 없는 메소드의 정보를 가져옴

<br>

### `Method` 인터페이스에 정의된 `invoke()` 메소드 이용해 메소드 실행
#### `invoke()` 메소드 
- 메소드를 실행시킬 대상 오브젝트(obj)와 파라미터 목록(args)을 받아서 메소드를 호출한 뒤에 그 결과를 Object 타입으로 돌려줌
```java
public Object invoke(Object obj, Object... args)
```
`length()` 메소드를 실행한다면, 다음과 같이 실행할 수 있음.
```java
int length = lengthMethod.invoke(name); // int length = name.length();
```

<br>

## 다이내믹 프록시를 이용한 프록시 만들어보자
### Step 1️⃣  프록시를 적용할 간단한 타깃 클래스와 인터페이스 정의
#### Feat: Hello 인터페이스
```java
interface Hello{
	String sayHello(String name);
	String sayHi(String name);
	String sayThankYou(String name);
}
```
#### Feat: 타깃 클래스
```java
public class HelloTarget implements Hello{
	public String sayHello(String name){
		return "Hello " + name;
	}
	public String sayHi(String name){
		return "Hi " + name;
	}
	public String sayThankYou(String name){
		return "Thank You " + name;
	}
	
}
```
<br>

### Step 2️⃣  Hello 인터페이스를 통해 HelloTarget 오브젝트를 사용하는 클라이언트 역할 테스트
#### Test: 클라이언트 역할의 테스트 
```java
@Test
public void simpleProxy() {
	Hello hello = new HelloTarget(); // 타깃은 인터페이스를 통해 접근하는 습관을 들이자.
	assertThat(hello.sayHello("Toby"), is("Hello Toby"));
	assertThat(hello.sayHi("Toby"), is("Hi Toby"));
	assertThat(hello.sayThankYou("Toby"), is("Thank You Toby"));
}
```
<br>

### Step 3️⃣  Hello 인터페이스를 구현한 프록시 클래스 만들기
프록시에는 데코레이터 패턴을 적용해 타깃인 HelloTarget 에 부가기능을 추가함.
- 추가할 기능 : 리턴하는 문자 모두 대문자로 바꿔주는 것

#### Feat: 프록시 클래스
```java
public class HelloUppercase implements Hello{
	Hello hello; // 위임할 타깃 오브젝트. 다른 프록시를 추가할 수도 있으므로 여기서는 인터페이스로 접근함
	
	public HelloUppercase(Hello hello){
		this.hello = hello;
	}
		
	public String sayHello(String name){
		return hello.sayHello(name).toUpperCase(); // 위임과 부가기능 적용
	}
	
	public String sayHi(String name){
		return hello.sayHi(name).toUpperCase();
	}
	
	public String sayThankYou(String name){
		return hello.sayThankYou(name).toUpperCase();
	}
}
```

#### 💥 이 프록시는 프록시 적용의 일반적인 문제점 두 가지를 모두 갖고 있음.
1. 인터페이스의 모든 메소드를 구현해 위임하도록 코드를 만들어야 함.

2. 부가기능이 모든 메소드에 중복돼서 나타남.

<br>

### Step 4️⃣ 다이내믹 프록시 적용
**다이내믹 프록시** : 프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어지는 오브젝트
- **타깃의 인터페이스와 같은 타입으로 만들어짐**

- 클라이언트는 다이내믹 프록시 오브젝트를 타깃 인터페이스를 통해 사용 가능
	- 인터페이스 모두 구현해가면서 클래스 정의하지 않아도 됨 :)
- 다이내믹 프록시가 **인터페이스 구현 클래스의 오브젝트는 만들어주지만**, 프록시로서 필요한 **부가기능 제공 코드는 직접 작성**해야 함
	- 부가기능은 프록시 오브젝트와 독립적으로 `InvocationHandler`를 구현한 오브젝트에 담음

<br>

**`InvocationHandler` 인터페이스** : `invoke()` 메소드 한 개만 가진 간단한 인터페이스
```java
public Object invoke(Object proxy, Method method, Object[] args)
```
> `invoke()` 메소드는 리플렉션의 Method 인터페이스를 파라미터로 받고, 메소드를 호출할 때 전달되는 파라미터도 args로 받음
>
>다이내믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환해 InvocationHandler 구현 오브젝트의 `invoke()` 메소드로 넘기는 것
>
>타깃 인터페이스의 모든 메소드 요청이 하나의 메소드로 집중되기 때문에 중복되는 기능을 효과적으로 제공 가능!

<br>

#### 각 메소드 요청을 어떻게 처리할지 결정하는 방법 
- InvocationHandler 구현 오브젝트가 타깃 오브젝트 레퍼런스를 갖고 있다면 리플렉션을 이용해 간단히 위임 코드를 만들어낼 수 있음

<br>

#### InvocationHandler 를 통한 요청 처리 구조
- Hello 인터페이스 제공하면서 프록시 팩토리에게 다이내믹 프록시 만들어달라고 요청하면, 

	Hello 인터페이스의 모든 메소드를 구현한 오브젝트 생성해줌.

- InvocationHandler 인터페이스를 구현한 오브젝트를 제공해주면, 

	다이내믹 프록시가 받는 모든 요청을 InvocationHandler의 `invoke()` 메소드로 보내줌.
- Hello 인터페이스의 메소드가 아무리 많더라도 `invoke()` 메소드 하나로 처리 가능 👍

#### Feat: 다이내믹 프록시로부터 메소드 호출 정보 받아 처리하는 InvocationHandler 구현 클래스
```java
public class UppercaseHandler implements InvocationHandler {
	Hello target;
		
	// 다이내믹 프록시로부터 전달받은 요청 다시 타깃 오브젝트에 위임해야 해서 타깃 오브젝트 주입받음	
	public UppercaseHandler(Hello target){ 
		this.target = target;
	}
	
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		String ret = (String)method.invoke(target, args); // 타깃으로 위임 인터페이스의 메소드 호출에 모두 적용됨
		return ret.toUpperCase(); // 부가기능 제공
	}
}
```
> 리턴된 값은 다이내믹 프록시가 받아서 최종적으로 클라이언트에게 전달될 것

<br>

#### 다이내믹 프록시 생성은 Proxy 클래스의 `newProxyInstance()` 스태틱 팩토리 메소드 이용
- 첫 번째 파라미터 : 다이내믹 프록시가 정의되는 클래스 로더 지정

- 두 번째 파라미터 : 다이내믹 프록시가 구현해야 할 인터페이스
	- 다이내믹 프록시는 한 번에 하나 이상의 인터페이스 구현 가능 => 인터페이스의 배열 사용
- 마지막 파라미터 : 부가기능과 위임 관련 코드를 담고 있는 InvocationHandler 구현 오브젝트
#### Feat: InvocationHandler를 사용하고 Hello 인터페이스를 구현하는 프록시 생성
```java
	Hello proxiedHello = (Hello)Proxy.newProxyInstance(
			getClass().getClassLoader(), // 동적으로 생성되는 다이내믹 프록시 클래스의 로딩에 사용할 클래스 로더
			new Class[] { Hello.class }, // 구현할 인터페이스
			new UppercaseHandler(new HelloTarget())); // 부가기능과 위임코드를 담은 InvocationHandler
		)
```

<br>

### 다이내믹 프록시의 장점
직접 정의해 만든 프록시보다 좋은 점은?
- 다이내믹 프록시가 만들어질 때 추가된 메소드가 자동으로 포함되고, 부가기능은 `invoke()` 메소드에서 처리됨

- InvocationHandler 방식은 어떤 종류의 인터페이스를 구현한 타깃이든 상관없이 재사용 가능
- InvocationHandler는 단일 메소드 `invoke()` 에서 모든 요청을 처리하기 때문에 어떤 메소드에 어떤 기능 적용할지 선택할 수 있다. 
	- 호출하는 메소드 이름, 파라미터의 개수와 타입, 리턴 타입 등의 정보 가지고 부가적인 기능 적용할 메소드 선택 가능

	- 메소드의 이름도 조건으로 걸 수 있음.

#### Fix: 확장된 UppercaseHandler
```java
public class UppercaseHandler implements InvocationHandler{
	// 어떤 종류의 인터페이스를 구현한 타깃에도 적용 가능하도록 Object 타입으로 수정
	Object target;
	private UppercaseHandler(Object target){
		this.target = target;
	}
	
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		// 호출한 메소드의 리턴 타입이 String인 경우만 대문자 변경 기능 적용하도록 수정
		Object ret = method.invoke(target, args);
		if(ret instanceof String){
			return ((String)ret).toUpperCase();
		}
		else{
			return ret;
		}
	}
}
```

<br>

## 다이내믹 프록시 방식으로 UserServiceTx를 변경해보자
목표 : 트랜잭션 부가기능을 제공하는 다이내믹 프록시를 만들어 적용하는 것

#### 작업 순서 
1️⃣ 어떤 타깃에도 적용 가능한 트랜잭션 부가기능을 담은 핸들러(TransactionHandler)를 만든다.

2️⃣ TransactionHandler를 이용하는 다이내믹 프록시를 UserService에 적용하는 테스트를 만든다.

3️⃣ TransactionHandler와 다이내믹 프록시를 스프링의 DI를 통해 사용할 수 있게 만든다.
- 💥 문제 : DI의 대상이 되는 다이내믹 프록시 오브젝트는 일반적인 스프링의 빈으로는 등록할 방법 없음. 다이내믹 프록시는 Proxy 클래스의 `newProxyInstance()`라는 스태틱 팩토리 메소드를 통해서만 만들 수 있음.

- ✅ 해결 : **팩토리 빈**을 이용하자.

<br>

## 팩토리 빈(Factory Bean)
🌱 스프링은 클래스 정보를 가지고 디폴트 생성자를 통해 오브젝트를 만드는 방법 외에 빈을 만들 수 있는 여러 가지 방법을 제공하는데, 대표적으로 **팩토리 빈을 이용한 빈 생성 방법**이 있다.

### ✅ 팩토리 빈
- 스프링을 대신해서 오브젝트의 생성로직을 담당하도록 만들어진 특별한 빈

### 팩토리 빈 만드는 방법
- 가장 간단한 방법 : 스프링의 FactoryBean이라는 인터페이스를 구현

- FactoryBean 인터페이스는 아래와 같이 세 가지 메소드로 구성되어 있음.

```java
package org.springframework.beans.factory;

public interface FactoryBean<T> {
	T getObject() throws Exception; // 빈 오브젝트를 생성해서 돌려준다.
	Class<? extends T> getObjectType(); // 생성되는 오브젝트의 타입을 알려준다.
	boolean isSingleton(); // getObject()가 돌려주는 오브젝트가 항상 같은 싱글톤 오브젝트인지 알려준다.
}
```
- FactoryBean 인터페이스를 구현한 클래스를 스프링의 빈으로 등록하면 팩토리 빈으로 동작함

### 👀 팩토리 빈 동작원리 확인
**1️⃣ 스프링에서 빈 오브젝트로 만들어 사용하고 싶은 클래스 정의하자**
#### Feat: Message 클래스 추가
```java
public class Message{
	String text;
	
	// 생성자가 private으로 선언 -> 외부에서 생성자 통해 오브젝트 만들 수 x
	private Message(String text) {
		this.text = text;
	}
		
	public String getText() {
		return text;
	}
	
	// 생성자 대신 사용할 수 있는 스태틱 팩토리 메소드를 제공
	public static Message newMessage(String text) {
		return new Message(text); 
	}
}
```
**2️⃣ 위에서 정의한 클래스의 오브젝트를 생성해주는 팩토리 빈 클래스를 만들자**
#### Feat: Message의 팩토리 빈 클래스 추가
```java
public class MessageFactoryBean implements FactoryBean<Message>{
	String text;
	
	// 오브젝트 생성 시 필요한 정보를 팩토리 빈의 프로퍼티로 설정해 대신 DI 받을 수 있게 함
	public void setText(String text){ 
		this.text = text;
	}
	
	// 실제 빈으로 사용될 오브젝트 직접 생성
	public Message getObject() throws Exception {
		return Message.newMessage(this.text);
	}
	
	public Class<? extends Message> getObjectType() {
		return Message.class;
	}
	
	// getObject() 메소드가 돌려주는 오브젝트가 싱글톤인지 알려줌
	public boolean isSingleton() {
		return false; // 이 팩토리 빈은 매번 요청할 때마다 새로운 오브젝트 만드니까 false로 설정
	}
}

```
- 팩토리 빈은 전형적인 팩토리 메소드를 가진 오브젝트
- 스프링은 FactoryBean 인터페이스를 구현한 클래스가 빈의 클래스로 지정되면, 팩토리 빈 클래스의 오브젝트의 `getObject()` 메소드를 이용해 오브젝트를 가져오고, 이를 빈 오브젝트로 사용함.
- 빈의 클래스로 등록된 팩토리 빈은 빈 오브젝트를 생성하는 과정에서만 사용될 뿐임.

 **3️⃣ 팩토리 빈을 설정해주자** 
 - id와 class 애트리뷰트를 사용해 빈의 아이디와 클래스를 지정하면 됨
 - 여타 빈 설정과 다른 점 : 

	 - **빈 오브젝트의 타입**이 class 애트리뷰트에 정의된 MessageFactoryBean이 아니라 **Message 타입**이라는 것
	 - Message 빈의 타입은 MessageFactoryBean의 `getObjectType()` 메소드가 돌려주는 타입으로 결정됨.
	 - `getObject()` 메소드가 생성해주는 오브젝트가 message 빈의 오브젝트가 됨
#### Feat: 팩토리 빈 설정 추가
```xml
<bean id="message"
	class="springbook.learningtest.spring.factorybean.MessageFactoryBean">
	<property name="text" value="Factory Bean" />
</bean>
```

 **4️⃣ 학습 테스트를 만들어서 확인해보자** 
#### Test: 팩토리 빈 테스트
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration // 설정파일 이름을 지정하지 않으면 클래스이름 + "-context.xml"이 디폴트로 사용됨
public class FactoryBeanTest {
	@Autowired
	ApplicationContext context;
	
	@Test
	public void getMessageFromFactoryBean(){
		Object message = context.getBean("message");
		assertThat(message, is(Message.class)); // 타입 확인
		assertThat(((Message)message).getText(), is("Factory Bean")); // 설정과 기능 확인
	}
	...
}
```
드물지만 팩토리 빈이 만들어주는 빈 오브젝트가 아니라 팩토리 빈 자체를 가져오고 싶을 경우 : 
- 빈 이름 앞에  '&'를 붙여주면 팩토리 빈 자체를 돌려준다.

	 ```java
	  Object factory = context.getBean("&message");
	 ```

#### ✅ 다이내믹 프록시 오브젝트를 스프링의 빈으로 만들기 위해 '팩토리 빈'을 사용한다.
> 팩토리 빈의 `getObject()` 메소드에 다이내믹 프록시 오브젝트를 만들어주는 코드 넣으면 됨.

<br>

### (다이내믹 프록시를 생성해주는) 팩토리 빈 방식의 장점
- 한번 부가기능을 가진 프록시를 생성하는 팩토리 빈을 만들어두면 **타깃의 타입에 상관없이 재사용 가능**

- **데코레이터 패턴이 적용된 프록시** 사용 시 발생하는 **문제점 두 가지 해결** 가능
	- **타깃 인터페이스를 구현하는 클래스**를 **일일이 만드는 번거로움을 제거**할 수 있음

	- 하나의 핸들러 메소드를 구현하는 것만으로도 수많은 메소드에 부가기능 부여해줄 수 있어 **부가기능 코드의 중복 문제 사라짐**
- 다이내믹 프록시에 팩토리 빈을 이용한 DI까지 더해주면, **번거로운 다이내믹 프록시 생성 코드도 제거**할 수 있음 
- **DI 설정**만으로 **다양한 타깃 오브젝트에 적용 가능**

### (다이내믹 프록시를 생성해주는) 팩토리 빈 방식의 한계
- 하나의 클래스 안에 존재하는 여러 개의 메소드에 부가기능을 한 번에 제공하는 건 가능하지만, **한 번에 여러 개의 클래스에 공통적인 부가기능을 제공하는 건 불가능**

- **하나의 타깃에 여러 개의 부가기능을 적용**하려고 할 때 **설정파일이 급격히 복잡**해짐
- **핸들러 오브젝트가 프록시 팩토리 빈 개수만큼 만들어짐**

=> ✅ 이런 문제에 대한 해결책 : **스프링의 ProxyFactoryBean**

<br>

## 6.4 스프링의 프록시 팩토리 빈(ProxyFactoryBean)
스프링은 서비스 추상화를 프록시 기술에도 동일하게 적용하여,

 **프록시 오브젝트를 생성해주는 기술을 추상화한 팩토리 빈**을 제공해준다.

### ✅ 스프링의 ProxyFactoryBean 
- **프록시를 생성해서 빈 오브젝트로 등록하게 해주는 팩토리 빈**

- 순수하게 프록시를 생성하는 작업만을 담당하고 프록시를 통해 제공해줄 부가기능은 별도의 빈에 둘 수 있음
- ProxyFactoryBean 이 생성하는 프록시에서 사용할 부가기능은 `MethodInterceptor` 인터페이스를 구현해서 만듦
- `MethodInterceptor` VS `InvocationHandler`
	- `InvocationHandler`의 `invoke()` 메소드는 타깃 오브젝트에 대한 정보 제공 x  -> 타깃은 InvocationHandler를 구현한 클래스가 직접 알고 있어야 함.

	- `MethodInterceptor`는 타깃 오브젝트에 상관없이 독립적으로 만들어질 수 있음 -> 타깃이 다른 여러 프록시에서 함께 사용 가능 + 싱글톤 빈으로 등록 가능

 #### Test: 스프링 ProxyFactoryBean을 이용한 다이내믹 프록시 테스트
```java
package springbook.learningtest.jdk.proxy;
...
public class DynamicProxyTest{
	@Test
	public void simpleProxy() {
		// JDK 다이내믹 프록시 생성
		Hello proxiedHello = (Hello)Proxy.newProxyInstance(
			getClass().getClassLoader(),
			new Class[] { Hello.class },
			new UppercaseHandler(new HelloTarget()));
	...
	}
	
	@Test
	public void proxyFactoryBean(){
		ProxyFactoryBean pfBean = new ProxyFactoryBean();
		pfBean.setTarget(new HelloTarget()); // 타깃 설정
		pfBean.addAdvice(new UppercaseAdvice()); // 부가기능을 담은 어드바이스 추가

		Hello proxiedHello = (Hello) pfBean.getObject(); // FactoryBean이므로 getObject()로 생성된 프록시를 가져옴
		assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
		assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
		assertThat(proxiedHello.sayThankYou("Toby"), is("THANK YOU TOBY"));
	}
	
	static class UppercaseAdvice implements MethodInterceptor {
		public Object invoke(MethodInvocation invocation) throws Throwable {
			String ret = (String)invocation.proceed(); // 리플렉션의 Method와 달리 메소드 실행 시 타깃 오브젝트를 전달할 필요x
			return ret.toUpperCase(); // 부가기능 적용
		}
	}
	
	// 타깃과 프록시가 구현할 인터페이스
	static interface Hello {
		String sayHello(String name);
		String sayHi(String name);
		String sayThankYou(String name);
	}
		
	static class HelloTarget implements Hello {
		public String sayHello(String name) { return "Hello " + name; }
		public String sayHi(String name) { return "Hi " + name; }
		public String sayThankYou(String name) { return "Thank You " + name; }
	}
}
```
#### 추가할 라이브러리
```java
com.springsource.org.aopalliance-1.0.0.jar
org.springframework.aop-3.0.7.RELEASE.jar
```

<br>

## JDK 다이내믹 프록시 사용했던 코드 -> ProxyFactoryBean 적용한 코드, 어떻게 바뀌었을까?
1️⃣ **타깃 오브젝트가 등장하지 x**
- MethodInterceptor는 메소드 정보와 함께 타깃 오브젝트가 담긴 MethodInvocation 오브젝트가 전달됨
- MethodInvocation : 일종의 콜백 오브젝트로, `proceed()` 메소드를 실행하면 타깃 오브젝트의 메소드를 내부적으로 실행해줌
- MethodInvocation 구현 클래스는 일조의 공유 가능한 템플릿처럼 동작 => MethodInvocation을 싱글톤으로 두고 공유 가능

2️⃣ ProxyFactoryBean에 이 MethodInterceptor를 설정해줄 때, 일반적인 DI 경우처럼 수정자 메소드를 사용하는 대신 **`addAdvice()`라는 메소드를 사용** 
- 여러 개의 MethodInterceptor 추가 가능 = ProxyFactoryBean 하나만으로 여러 개의 부가기능을 제공해주는 프록시를 만들 수 있음 ( 팩토리 빈 방식의 단점 해결 ✅ )

3️⃣ **프록시가 구현해야 하는 인터페이스를 제공해주는 부분이 없음.**
> 인터페이스를 굳이 알려주지 않아도 ProxyFactoryBean에 있는 인터페이스 자동검출 기능 사용 -> 타깃 오브젝트가 구현하고 있는 인터페이스 정보 알아내고, 알아낸 인터페이스를 모두 구현하는 프록시를 만들어줌.

<br>

## 🧐 방식 비교
### 1) 기존 JDK 다이내믹 프록시 이용한 방식 
- InvocationHandler에 부가기능 + 메소드 선정 알고리즘 모두 있고, InvocationHandler가 다이내믹 프록시로부터 받은 요청을 타깃 오브젝트에 위임함.

- 💥 **문제** : **부가기능 가진 InvocationHandler가 타깃과 메소드 선정 알고리즘 코드에 의존**하고 있다는 점
	- 만약, 타깃이 다르고 메소드 선정 방식이 다르면 InvocationHandler 오브젝트를 여러 프록시가 공유 불가

	- 확장에는 유연하게 열려있지 x, 관련 없는 코드 변경 필요할 수 있는, OCP 원칙을 깔끔하지 잘 지키지 못하는 어정쩡한 구조

<br>

### 2) 스프링의 ProxyFactoryBean 방식 
- 두 가지 확장 기능인 `부가기능(Advice)`과 `메소드 선정 알고리즘(Pointcut)`을 활용하는 유연한 구조 제공
- **어드바이스와 포인트컷**은 모두 **프록시에 DI로 주입돼서 사용**됨
- 두 가지 모두 여러 프록시에서 공유가 가능하도록 만들어지기 때문에 **스프링의 싱글톤 빈으로 등록 가능**

### ✅ 어드바이스(Advice)
- `MethodInterceptor`처럼 **타깃 오브젝트에 적용하는** 부가기능을 담은 오브젝트 

- **타깃 오브젝트에 종속되지 않는 순수한 부가기능을 담은 오브젝트**

### ✅ 포인트컷(Pointcut)
- **메소드 선정 알고리즘**을 담은 **오브젝트**

#### 2-1) 진행 순서
1️⃣ 프록시는 클라이언트로부터 요청 받으면 먼저 포인트컷에게 부가기능을 부여할 메소드인지 확인해달라고 요청함. 
- 포인트컷은 Pointcut 인터페이스를 구현해 만들면 됨

2️⃣ 프록시는 포인트컷으로부터 부가기능을 적용할 대상 메소드인지 확인받으면, MethodInterceptor 타입의 어드바이스를 호출함
- 어드바이스는 JDK의 다이내믹 프록시의 InvocationHandler와 달리 직접 타깃 호출하지 x => 타깃에 직접 의존하지 않도록 일종의 템플릭 구조로 설계되어 있음

3️⃣ 어드바이스가 부가기능을 부여하는 중에 타깃 메소드의 호출이 필요하면, 프록시로부터 전달받은 MethodInvocation 타입 콜백 오브젝트의 `proceed()` 메소드를 호출해주기만 하면 됨

#### 2-2) 정리 
- **어드바이스**가 일종의 **템플릿**이 되고, 타깃을 호출하는 기능을 갖고 있는 **MethodInvocation** 오브젝트가 **콜백**이 됨.
- 템플릿은 한 번 만들면 재사용 가능, 여러 빈이 공유해서 사용 가능 

	- 어드바이스도 독립적인 싱글톤 빈으로 등록 +  DI 주입 => **여러 프록시가 사용하도록 만들 수 있음**

- **프록시로부터 어드바이스와 포인트컷을 독립시키고 DI를 사용**하게 한 것 : 전형적인 **전략 패턴 구조**
	- OCP(개방 폐쇄 원칙)를 충실히 지키는 구조

#### Test: 포인트컷까지 적용한 ProxyFactoryBean 
```java
@Test
public void pointcutAdvisor() {
	ProxyFactoryBean pfBean = new ProxyFactoryBean();
	pfBean.setTarget(new HelloTarget()); 
	
	NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut(); // 메소드 이름을 비교해 대상 선정하는 알고리즘을 제공하는 포인트컷 생성
	pointcut.setMappedName("sayH*"); // 이름 비교조건 설정
	
	// 포인트컷과 어드바이스를 Advisor로 묶어서 한 번에 추가
	pfBean.addAdvisor(new DefaultPointcutAdvisor(pointcut, new UppercaseAdvice()));
	
	Hello proxiedHello = (Hello) pfBean.getObject();
	
	assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
	assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
	assertThat(proxiedHello.sayThankYou("Toby"), is("Thank You TOBY")); // 메소드 이름이 포인트컷의 선정조건에 맞지 않으므로, 부가기능(대문자변환)이 적용 x
}
```
 > 어드바이스와 포인트컷을 함께 등록할 때 Advisor 타입으로 묶어서 `addAdvisor()` 메소드를 호출하는 이유?
 > - ProxyFactoryBean에는 여러 개의 어드바이스와 포인트컷이 추가될 수 있기 때문

<br>

### ✅ 어드바이저(Advisor)
- 어드바이스와 포인트컷을 묶은 오브젝트

- **어드바이저 = 포인트컷(메소드 선정 알고리즘) + 어드바이스(부가기능)**

<br>

## ProxyFactoryBean 적용
#### Feat: 트랜잭션 어드바이스
```java
package springbook.user.service;
...
public class TransactionAdvice implements MethodInterceptor { // 스프링의 어드바이스 인터페이스 구현
	PlatformTransactionManager transactionManager;
	
	public void setTransactionManager(PlatformTransactionManager transactionManager) {
		this.transactionManager = transactionManager;
	}
	
	// 타깃을 호출하는 기능을 가진 콜백 오브젝트를 프록시로부터 받음(어드바이스는 특정 타깃에 의존하지 않고 재사용 가능)
	public Object invoke(MethodInvocation invocation) throws Throwable {
		TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
	
		try{
			Object ret = invocation.proceed(); // 콜백 호출해 타깃의 메소드 실행
			this.transactionManager.commit(status);
			return ret;
		} catch(RuntimeException e){ // 예외가 포장되지 않고 타깃에서 보낸 그대로 전달됨
			this.transactionManager.rollback(status);
			throw e;
		}
	}
}
```

#### Feat: 트랜잭션 어드바이스 빈 설정
```xml
<bean id="transactionAdvice" class="springbook.user.service.TransactionAdvice">
	<property name="transactionManager" ref="transactionManager" />
</bean>
```
#### Feat: 포인트컷 빈 설정
```xml
<bean id="transactionPointcut" class="org.springframework.aop.support.NameMatchMethodPointcut">
	<property name="mappedName" value="upgrade*" />
</bean>
```
#### Feat: 어드바이저 빈 설정
```xml
<bean id="transactionAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
	<property name="advice" ref="transactionAdvice" />
	<property name="pointcut" ref="transactionPointcut" />
</bean>
```
#### Feat: ProxyFactoryBean 설정
```xml
<bean id="userService" class="org.springframework.aop.framework.ProxyFactoryBean">
	<property name="target" ref="userServiceImpl" />
	<property name="interceptorNames">
		<list>
			<value>transactionAdvisor</value>
		</list>
	</property>
</bean>
```

<br>

### 어드바이스와 포인트컷의 재사용
- ProxyFactoryBean은 스프링의 DI와 템플릿/콜백 패턴, 서비스 추상화 등의 기법이 모두 적용된 것.

- 독립적이며, 여러 프록시가 공유할 수 있는 어드바이스와 포인트컷으로 확장 기능 분리 가능
- 새로운 비즈니스 로직 담은 서비스 클래스 만들어져도 이미 만들어둔 TransactionAdvice를 그대로 재사용 가능
- 메소드 선정 위한 포인트컷이 필요하면 이름 패턴만 지정해 ProxyFactoryBean에 등록해주면 됨
