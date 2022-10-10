# 📗 토비의 스프링 3.1 Vol.1 스프링의 이해와 원리
## 📝 2장 테스트
🌱 스프링이 개발자에게 제공하는 가장 중요한 가치 : `객체지향`, `테스트`

>애플리케이션은 계속 변하고 복잡해져 간다. 
>
>그 변화에 대응하는 첫 번째 전략이 확장과 변화를 고려한 객체지향적 설계와 그것을 효과적으로 담아낼 수 있는 **IoC/DI** 같은 기술이라면, 
>
>두 번째 전략은 만들어진 코드를 확신할 수 있게 해주고, 변화에 유연하게 대처할 수 있는 자신감을 주는 **테스트** 기술이다.

#### ✨ 2장에서 다루는 것 : 테스트란 무엇이며, 그 가치와 장점, 활용 전략, 스프링과의 관계, 대표적인 테스트 프레임워크 소개 - 이를 이용한 학습 전략 

<br>

## 2.1 UserDaoTest 다시 보기
### ✅ 테스트
>- **내가 예상하고 의도했던 대로 코드가 정확히 동작하는지를 확인해서, 만든 코드를 확신할 수 있게 해주는 작업**
>
>- 테스트의 결과가 원하는 대로 나오지 않는 경우엔 코드나 설계에 결함이 있음을 알 수 있음.
>
>- 이를 통해 코드의 결함을 제거해가는 작업, 일명 디버깅을 거치게 되고, 결국 최종적으로 테스트가 성공하면 모든 결함이 제거됐다는 확신을 얻을 수 있음.

<br>

#### 💥 웹을 통한 DAO 테스트 방법의 문제점
>- DAO뿐만 아니라 서비스 클래스, 컨트롤러, JSP 뷰 등 **모든 레이어의 기능을 다 만들고 나서야 테스트가 가능하다**는 점이 가장 큰 문제
>
>- 테스트를 하는 중에 에러가 나거나 테스트가 실패했다면, **과연 어디에서 문제가 발생했는지를 찾아내야 하는** 수고도 필요 - 하나의 테스트를 수행하는데 참여하는 클래스와 코드가 너무 많기 때문

<br>

### 🧐 그렇다면, 테스트를 어떻게 만들면 이런 문제를 피할 수 있고, 효율적으로 테스트를 활용할 수 있을까?
> #### 🙋‍♀️ 작은 단위의 테스트
>테스트하고자 하는 대상 명확 -> 그 대상에만 집중해서 테스트하는 것이 바람직.
>
>한꺼번에 너무 많은 것들을 몰아서 테스트 -> 테스트 수행 과정 복잡 & 오류 발생 시 정확한 원인 찾기 힘듦
>
>따라서, **테스트는 가능하면 작은 단위로 쪼개서 집중할 수 있어야 한다.**

<br>

### ✅ 단위 테스트(unit test)
#### 작은 단위의 코드에 대해 테스트를 수행한 것
### 💬 "단위" 
>여기서 말하는 **단위**란 무엇인지, **그 크기와 범위가 어느 정도인지 딱 정해진 건 x**
>
>크게는 사용자 **관리 기능을 모두 통틀어서 하나의 단위**로 볼 수도 있고, 
>
>작게 보자면 UserDao 클래스의 `add()` **메소드 하나만 가지고 하나의 단위**라고 생각할 수도 있음.
>
>**충분히 하나의 관심에 집중해서 효율적으로 테스트할 만한 범위의 단위라고 보면 됨.**
>
> 단위는 작을수록 좋음.
> 
> 단위를 넘어서는 다른 코드들은 신경 쓰지 않고, 참여하지도 않고 테스트가 동작할 수 있으면 좋음.

<br>

#### 🧐 DB가 사용되면 단위 테스트가 아닐까?

> 그렇진 않음.
> 
> **사용할 DB의 상태를 테스트가 관장하고 있다면 이는 단위 테스트**라고 해도 됨.
> 
> 다만, DB의 상태가 매번 달라지고, 테스트를 위해 DB를 특정 상태로 만들어줄 수 없다면 그때는 단위 테스트로서 가치가 없어짐.
> 
> 그런 차원에서 통제할 수 없는 외부의 리소스에 의존하는 테스트는 단위 테스트가 아니라고 보기도 하는 것.

<br>

>때로는 웹 사용자 인터페이스부터 시작해 DB에 이르기까지 **애플리케이션 전 계층이 참여**하고, 또 단순 사용자 등록 작업 하나가 아니라, 초기 등록에서부터 시작해서 등록이 성공하면 로그인하고, 각종 기능을 모두 사용한 다음에 로그아웃까지 하는 **전과정을 하나로 묶어서 테스트할 필요가 있음.** 
>
> 각 단위 기능은 잘 동작하는데 **묶어놓으면 안 되는 경우가 종종 발생하기 때문.**
>
>#### ✅ **길고 많은 단위가 참여하는 테스트도 언젠간 필요함.**

<br>

### ✅ 단위 테스트를 하는 이유?

> 🙋‍♀️ **개발자가 설계하고 만든 코드가 원래 의도한 대로 동작하는지를 개발자 스스로 빨리 확인받기 위해서**
> 
> -  단위테스트 = 개발자 테스트 = 프로그래머 테스트

<br>

### ✅ 자동으로 수행되는 테스트 코드 
> **테스트** 자체가 사람의 수작업을 거치는 방법을 사용하기보다는 **코드로 만들어져서 자동으로 수행될 수 있어야 한다**는 건 매우 중요
> 
> 애플리케이션을 구성하는 클래스 안에 테스트 코드를 포함시키는 것보다는 **별도로 테스트용 클래스를 만들어서 테스트 코드를 넣는 편이 나음**
>
>#### 자동으로 수행되는 테스트의 장점 : 
>- 자주 반복 가능
>
> - 언제든 코드 수정하고 나서 테스트를 해볼 수 있음 
>
>	- 코드 수정 후 만들어둔 기능에 대한 테스트가 있다면, 수정 후 빠르게 전체 테스트를 수행해서 수정 때문에 다른 기능에 문제가 발생하지는 않는지 재빨리 확인하고, 성공한다면 마음에 확신 얻을 수 있음
>
>**결론 : 지속적인 개선과 점진적인 개발을 위해 테스트 필요**

<br>

### 😥 UserDaoTest의 문제점 
UserDaoTest 코드 p.147
1. **수동 확인 작업의 번거로움**
	>`add()` 에서 User 정보를 DB에 등록하고 이를 다시 `get()`을 이용해 가져왔을 때 입력한 값과 가져온 값이 일치하는지를 테스트 코드는 확인해주지 x
	>
	> 단지 콘솔에 값만 출력해줄 뿐.
	>
	> 결국 그 콘솔에 나온 값을 보고 등록과 조회가 성공적으로 되고 있는지를 확인하는 건 사람의 책임 => 완전히 **자동으로 테스트**되는 방법이라고 말할 수 x

2. **실행 작업의 번거로움**
	> 아무리 간단한 실행 가능한 `main()` 메소드라고 하더라도 매번 그것을 실행하는 것은 제법 번거로움 
	>
	>**=> `main()` 메소드를 이용하는 방법보다 좀 더 편리 & 체계적으로 테스트 실행하고 그 결과를 확인하는 방법 필요!**

<br>

## 2.2 UserDaoTest 개선
### ✅ 테스트 결과 검증 자동화
#### [이 테스트를 통해 확인하고 싶은 사항] 
- #### `add()`에 전달한 User 오브젝트에 담긴 사용자 정보와 `get()`을 통해 다시 DB에서 가져온 User 오브젝트의 정보가 서로 정확히 일치하는가

	> 정확히 일치한다면 :
	> - 모든 정보가 빠짐없이 DB에 등록됐고, 이를 다시 DB에서 정확히 가져왔다는 사실 알 수 있음

<br>

#### [나올 수 있는 테스트의 결과]
1. 성공
2. 실패

	> - 테스트 에러(테스트 진행되는 동안 에러 발생 -> 실패하는 경우)
	> 
	> 	- 콘솔에 에러 메시지와 긴 호출 스택 정보 출력으로 쉽게 확인 가능
	> 
	> - 테스트 실패(테스트 작업 중에 에러 발생 x 그런데, 그 결과가 기대한 것과 다르게 나오는 경우)
	> 	- 별도의 확인 작업 + 그 결과 있어야 알 수 있음

<br>

#### [UserDaoTest 수정하고자 하는 방향]
>- 테스트 코드 결과 직접 확인하고, 기대한 결과와 달라서 실패했을 경우 - "테스트 실패" 메시지 출력
>- 모든 확인 작업 통과했을 경우 - "테스트 성공" 메시지 출력

>#### - UserDaoTest 클래스 수정 전 
>``` java
>System.out.println(user2.getName());
>System.out.println(user2.getPassword());
>System.out.println(user2.getId()+"조회 성공");
>```
>

>#### - UserDaoTest 클래스 수정 후
>``` java
>if(!user.getName().equals(user2.getName())){
>	System.out.println("테스트 실패 (name)");
>}
>else if(!user.getPassword().equals(user2.getPassword())){
>	System.out.println("테스트 실패 (password)");
>}
>else{
>	System.out.println("조회 테스트 성공");
>}
>```

<br>

### ➕ 포괄적인 테스트(comprehensive test)
> 만들어진 코드의 기능을 모두 점검할 수 있는 테스트 

 <br>

 ### 😥 `main()` 메소드로 만든 테스트의 한계
> - 애플리케이션 규모가 커지고 테스트 개수가 많아지면 테스트 수행하는 일이 점점 부담이 됨
>
>	 => **JUnit 사용하자** 

<br>

### ✅ JUnit 이란?
> **프로그래머를 위한 자바 테스팅 프레임워크** 
> 
> **자바로 단위 테스트를 만들 때 유용하게 쓸 수 있음**

<br>

### 💡 프레임워크의 기본 동작원리 : 제어의 역전(Ioc)
> **프레임워크**는 개발자가 만든 클래스에 대한 제어 권한을 넘겨받아서 **주도적으로 애플리케이션의 흐름을 제어함.**
> 
> **개발자가 만든 클래스의 오브젝트를 생성 & 실행하는 일**은 **프레임워크에 의해 진행**됨
> 
> 프레임워크에서 동작하는 코드 : **`main()` 메소드도 필요 없고, 오브젝트를 만들어서 실행시키는 코드를 만들 필요도 없음**

<br>

### ✅ JUnit 프레임워크가 요구하는 조건 두 가지
1. **메소드**가 **`public`** 으로 선언되어야 함

2. **메소드**에 **`@Test`** 라는 애노테이션을 붙여줘야 함
>#### Fix: UserDaoTest 클래스 JUnit 프레임워크에서 동작하도록 테스트 메소드 재구성
>``` java
>import org.junit.Test;
>...
>public class UserDaoTest{
>	@Test // JUnit에게 테스트용 메소드임을 알려줌
>	public void addAndGet() throws SQLException { // JUnit 테스트 메소드 반드시 public 으로 선언
>		...
>	}
>}
>```

<br>

### ✅ JUnit이 제공하는 메소드 이용해 코드 변경
#### ✨ `assertThat()` 메소드 :
- 첫 번째 파라미터의 값을 뒤에 나오는 **매처**(matcher)라고 불리는 조건으로 비교해 일치하면 다음으로 넘어가고, 아니면 테스트가 실패하도록 만들어준다.

- `is()`는 매처의 일종으로 `equals()`로 비교해주는 기능을 가짐.

>``` java
>if(!user.getName().equals(user2.getName())) {...}
>```
>을 아래와 같이 변경할 수 있음.
>``` java
>import static org.hamcrest.CoreMatchers.is;
>import static org.junit.Assert.assertThat;
>...
>assertThat(user2.getName(), is(user.getName()));
>```

> #### 추가할 라이브러리 : 
>
>com.springsource.org.junit-4.7.0.jar
<br>

>JUnit은 예외가 발생하거나 assertThat()에서 실패하지 않고 테스트 메소드의 실행이 완료되면 테스트가 성공했다고 인식 
>
>=> **"테스트 성공"이라는 메시지를 굳이 출력할 필요 x**

<br>

### ✅ JUnit 테스트 실행
> 스프링 컨테이너와 마찬가지로 **JUnit 프레임워크도 자바 코드로 만들어진 프로그램**
> 
> **어디선가 한 번은 JUnit 프레임워크를 시작시켜줘야 함.**
> 
> 어디에든 `main()` 메소드를 하나 추가하고, 그 안에 `JUnitCore` 클래스의 `main` 메소드를 호출해주는 간단한 코드를 넣어주면 됨.  
>
>>메소드 파라미터에는 `@Test` 테스트 메소드를 가진 클래스의 이름을 넣어줌.

>#### Fix: `main()` 메소드 JUnit 이용해 테스트를 실행해주도록 수정
>```java
>import org.junit.runner.JUnitCore;
>...
>public static void main(String[] args){
>	JUnitCore.main("springbook.user.dao.UserDaoTest");
>}
>```
<br>


## 2.3 개발자를 위한 테스팅 프레임워크 JUnit
>가장 좋은 JUnit 테스트 실행 방법 = 자바 IDE에 내장된 JUnit 테스트 지원 도구를 사용하는 것

<br>

### 🔎 이클립스(Eclipse)에서 JUnit Test 사용법
>#### JUnit Test 실행 방법 : 
>1. @Test가 들어있는 테스트 클래스를 선택
>
>2. 이클립스 run 메뉴의 Run As 항목 중 JUnit Test 선택
>
>테스트가 실행되면 **JUnit 테스트 정보를 표시해주는 뷰(View)** 가 나타나서 테스트 진행 상황을 보여줌.

>####  ⌨️ 선택해둔 클래스나 패키지, 프로젝트의 테스트 바로 실행하는 단축키 
>>
>>Alt + Shift + X와 T를 순서대로 누르기

>#### 테스트가 실패해서 코드 수정한 뒤, 다시 테스트를 실행하는 방법 : 
>
>> JUnit 테스트 뷰의 녹색 Rerun Test 버튼 클릭 

>#### 한 번에 여러 테스트 클래스를 동시에 실행하는 법 
> 1. 소스 트리에서 특정 패키지 선택
> 
> 2. 컨텍스트 메뉴의 [Run As] -> [JUnit Test] 실행

<br>

### [여러 개발자가 만든 코드를 모두 통합해서 테스트를 수행해야 할 때]
- 서버에서 **모든 코드를 가져와 통합하고 빌드한 뒤에 테스트를 수행**하는 것이 좋음.

- 이때는 **빌드 스크립트를 이용해 JUnit 테스트를 실행**하고 그 결과를 메일 등으로 통보받는 방법을 사용하면 됨.

<br>

### ✅ 테스트 결과의 일관성
#### 💥 현재 UserDaoTest 테스트의 문제점 :
이전 테스트 때문에 DB에 등록된 중복 데이터가 있을 수 있다는 점

####  ✅ 해결책 :
`addAndGet()` 테스트를 마치고 나면 테스트가 등록한 사용자 정보를 삭제 -> 테스트를 수행하기 이전 상태로 만들어주자

=> `deleteAll()`의 `getCount()` 추가하자

> #### Feat: UserDao 클래스에 `deleteAll()`, `getCount()` 메소드 추가
```java
public void deleteAll() throws SQLException {
	Connection c = dataSource.getConnection();

	PreparedStatement ps = c.prepareStatement("delete from users");
	ps.executeUpdate();

	ps.close();
	c.close();
}

public int getCount() throws SQLException {
	Connection c = dataSource.getConnection();

	PreparedStatement ps = c.prepareStatement("select count(*) from users");

	ResultSet rs = ps.executeQuery();
	rs.next();
	int count = rs.getInt(1);

	rs.close();
	ps.close();
	c.close();

	return count;
}
```

> #### Feat: UserDaoTest 클래스 `addAndGet()` 테스트에 `deleteAll()`, `getCount()` 메소드 추가
```java
@Test
public void addAndGet() throws SQLException {
	...
	dao.deleteAll(); // 추가
	assertThat(dao.getCount(), is(0)); // 추가 

	User user = new User();
	user.setId("gyumee");
	user.setName("박성철");
	user.setPassword("springno1");

	dao.add(user);
	assertThat(dao.getCount(), is(1)); // 추가

	User user2 = dao.get(user.getId());

	assertThat(user2.getName(), is(user.getName()));
	assertThat(user2.getPassword(), is(user.getPassword()));
}
```

- #### 결론 : 단위 테스트는 항상 일관성 있는 결과가 보장돼야 함
	>DB에 남아있는 데이터와 같은 외부 환경에 영향을 받지 말아야 하는 것은 물론이고, 
	>
	>테스트를 실행하는 순서를 바꿔도 동일한 결과가 보장되도록 만들어야 함.

<br>

###  테스트 메소드는 한 번에 한 가지 검증 목적에만 충실한 것이 좋음
### ✅ `getCount()`를 위한 새로운 테스트 메소드를 만들어보자.

>JUnit은 하나의 클래스 안에 여러 개의 테스트 메소드가 들어가는 것을 허용함.
>> @Test가 붙어 있고, public 접근자가 있으며 리턴 값이 void형이고 파라미터가 없다는 조건을 지키기만 하면 됨.
> 
> #### 📝 테스트 시나리오 
>
>(준비단계) User 클래스에 한 번에 모든 정보를 넣을 수 있도록 초기화 가능한 생성자 추가
> 
>  1. USER 테이블의 데이터를 모두 지우고 `getCount()`로 레코드 개수가 0임을 확인
>  
>  2. 3개의 사용자 정보를 하나씩 추가하면서 매번 `getCount()`의 결과가 하나씩 증가하는지를 확인

>#### Feat: `getCount()`를 위한 새로운 테스트 메소드 `count()` 생성
>```java
>@Test
>public void count() throws SQLException {
>	ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
>
>	UserDao dao = context.getBean("userDao", UserDao.class);
>	User user1 = new User("gyumee", "박성철", "springno1");
>	User user2 = new User("dobby", "윤도비", "springno2");
>	User user3 = new User("bumjin", "박범진", "springno3");
>	
>	dao.deleteAll();
>	assertThat(dao.getCount(), is(0));
>
>	dao.add(user1);
>	assertThat(dao.getCount(), is(1));
>
>	dao.add(user2);
>	assertThat(dao.getCount(), is(2));
>	
>	dao.add(user3);
>	assertThat(dao.getCount(), is(3));
>}
>```

>👽 주의해야할 점 : JUnit은 특정한 테스트 메소드의 실행 순서를 보장해주지 않음
>- 테스트 결과가 테스트 실행 순서에 영향을 받지 받는다면 테스트를 잘못 만든 것.
>- ex) `addAndGet()` 메소드에서 등록한 사용자 정보를 `count()` 테스트에서 활용하는 식으로 테스트를 만들면 안 됨.

<br>

### ✅ `addAndGet()` 테스트 보완
>- id를 조건으로 해서 사용자를 검색하는 기능을 가진 `get()`에 대한 테스트는 조금 부족함.
>- `get()`이 파라미터로 주어진 id에 해당하는 사용자를 가져온 것인지, 그냥 아무거나 가져온 것인지 테스트에서 검증 못함
> => User를 하나 더 추가해서 두 개의 User를 `add()`하고, 각 User의 id를 파라미터로 전달해서 `get()`을 실행하도록 만들어보자
>
>#### Fix: `get()` 테스트 기능을 보완한 `addAndGet()` 테스트 
```java
@Test
public void addAndGet() throws SQLException {
	...

	UserDao dao = context.getBean("userDao", UserDao.class);
	User user1 = new User("gyumee", "박성철", "springno1");
	User user2 = new User("dobby", "윤도비", "springno2");
	
	dao.deleteAll();
	assertThat(dao.getCount(), is(0));

	dao.add(user1);
	dao.add(user2);
	assertThat(dao.getCount(), is(2));

	User userget1 = dao.get(user1.getId());
	assertThat(userget1.getName(), is(user1.getName()));
	assertThat(userget1.getPassword(), is(user1.getPassword()));

	User userget2 = dao.get(user2.getId());
	assertThat(userget2.getName(), is(user2.getName()));
	assertThat(userget2.getPassword(), is(user2.getPassword()));
}
```

<br>

### ✅ `get()` 예외조건에 대한 테스트

> #### 🧐 get() 메소드에 전달된 id값에 해당하는 사용자 정보가 없을 경우 어떤 결과가 나오면 좋을까?
> 
> **[두 가지 방법]**
>
> 1. null과 같은 특별한 값을 리턴하는 것
> 2. id에 해당하는 정보를 찾을 수 없다고 예외를 던지는 것 -> 스프링의 EmptyResultDataAccessException 예외를 이용
>
> <br>
> 
> #### JUnit 예외조건 테스트 특별한 방법
> 
> - 테스트 메소드 하나 더 추가
> 
> - 테스트 방법 
> 
> 	1. 모든 데이터 지우기
> 	2. 존재하지 않는 id로 `get()` 호출 
> 		- EmptyResultDataAccessException 던져지면 성공
> 		- 아니면 실패
>

>#### Feat: `get()` 메소드의 예외상황에 대한 테스트 추가
>```java
>@Test(expected=EmptyResultDataAccessException.class) // 테스트 중 발생할 것으로 기대하는 예외 클래스 지정
>public void getUserFailure() throws SQLException {
>	ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
>	
>	UserDao dao = context.getBean("userDao", UserDao.class);
>	dao.deleteAll();
>	assertThat(dao.getCount(), is(0));
>
>	dao.get("unknown_id"); // 이 메소드 실행 중에 예외가 발생해야 함. 예외 발생하지 않으면 테스트 실패
>}
>```

>#### Fix: 데이터를 찾지 못하면 예외를 발생시키도록 `get()` 메소드 수정
>```java
>public User get(String id) throws SQLException {
>	...
>	ResultSet rs = ps.executeQuery();
>	
>	User user = null;
>	if(rs.next()){
>		user = new User();
>		user.setId(rs.getString("id"));
>		user.setName(rs.getString("name"));
>		user.setPassword(rs.getString("password"));
>	}
>	rs.close();
>	ps.close();
>	c.close();
>
>	if(user == null) throw new EmptyResultDataAccessException(1);
>
>	return user;
>}
>```

>### 📝 테스트를 작성할 때 부정적인 케이스를 먼저 만드는 습관을 들이는 게 좋음

<br>

## 테스트가 이끄는 개발
### ✅ 기능 설계를 위한 테스트
#### 🧐 우리가 했던 작업을 돌이켜보자.
>**기능** : 존재하지 않는 id로 `get()` 메소드를 실행하면 특정한 예외가 던져져야 한다
>
>> 추가하고 싶은 기능을 코드로 표현하려고 하면 테스트를 만들 수 있다.
 
<br>

#### `getUserFailure()` 테스트 코드에 나타난 기능
|  | 단계  | 내용  | 코드 |
|--|--|--|--|
|조건  |어떤 조건을 가지고	  |가져올 사용자 정보가 존재하지 않는 경우에  |`dao.deleteAll();`<br> `assertThat(dao.getCount(), is(0));`  |
|행위  |무엇을 할 때	  |존재하지 않는 id로 get()을 실행하면  |`get("unknown_id");`  |
|결과  |어떤 결과가 나온다	  |특별한 예외가 던져진다  |`@Test(expected=EmptyResultDataAccessException.class)`  |

> 비교해보면, 이 **테스트 코드**는 마치 잘 작성된 하나의 **기능정의서**처럼 보임
> 
> 그래서 보통 **기능설계, 구현, 테스트라는 일반적인 개발 흐름의 기능설계에 해당하는 부분을 이 테스트 코드가 일부분 담당하고
> 있다고 볼 수도 있음**
> 
> 이런 식으로 **추가하고 싶은 기능**을 일반 언어가 아니라 **테스트 코드**로 표현 -> 마치 **코드로 된
> 설계문서**처럼 만들어놓은 것이라고 생각해보자. 
> 
> 그러고 나서 실제 기능 가진 애플리케이션 코드를 만들고 나면, 바로 이 테스트를 실행해서 설계한 대로 코드가 동작하는지를
> 빠르게 검증 가능
> 
> 결국, 테스트가 성공한다면, 그 순간 코드 **구현과 테스트라는 두 가지 작업이 동시에 끝남**

<br>

### ✅ 테스트 주도 개발(TDD, Test Driven Development)
> **만들고자 하는 기능의 내용을 담고 있으면서 만들어진 코드를 검증도 해줄 수 있도록 테스트 코드를 먼저 만들고, 테스트를 성공하게 해주는 코드를 작성하는 방식의 개발 방법**
>
> 테스트를 코드보다 먼저 작성한다고 해서 **테스트 우선 개발(Test First Development)** 라고도 함
>
>#### **TDD 기본 원칙** : 
>
>>"실패한 테스트를 성공시키기 위한 목적이 아닌 코드는 만들지 않는다"
>
>#### TDD 장점 :
>
>- TDD는 아예 테스트를 먼저 만들고, 그 테스트가 성공하도록 하는 코드만 만드는 식으로 진행하기 때문에 테스트를 빼먹지 않고 꼼꼼하게 만들어낼 수 있음.
>
>- 코드를 만들어 테스트를 실행하는 그 사이의 간격이 매우 짧음.
>
>- 코드에 대한 피드백을 매우 빠르게 받을 수 있게 됨.
>
>- 매번 테스트가 성공하는 것을 보면서 작성한 코드에 대한 확신 가질 수 있음
>
>
>TDD에서는 **테스트 작성하고 이를 성공시키는 코드를 만드는 작업 주기를 가능한 한 짧게** 가져가도록 권장함. 
>
>TDD를 하면 **자연스럽게 단위 테스트를 만들 수 있음.**
>>빠르게 자동으로 실행할 수 있는 단위 테스트가 아니고서는 이런 식의 개발은 거의 불가능하기 때문

<br>

## ✅ 테스트 코드 개선
현재까지 만든 테스트 메소드 3개 `addAndGet()`, `count()`, `getUserFailure()` 를 리팩토링해보자.

### ✨ 1. UserDaoTest에서 스프링의 애플리케이션 컨텍스트 만드는 부분과 컨텍스트에서 UserDao 가져오는 부분 중복 => `@Before` 애노테이션 사용
#### 중복되는 부분 코드
>```java
>ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
>
>User dao = context.getBean("userDao", UserDao.class);
>```
>
>- 메소드 추출 리팩토링 방법 말고 **JUnit이 제공하는 기능 활용**해보자!
>
>	>JUnit 프레임워크는 테스트 메소드를 실행할 때 부가적으로 해주는 작업 몇 가지 존재.
>	>- 그중에서 **테스트를 실행할 때마다 반복되는 준비 작업을 별도의 메소드에 넣게 해주고, 이를 매번 테스트 메소드를 실행하기 전에 먼저 실행시켜주는 기능** = **`@Before`**
>
><br>
>
>#### Fix: 중복 코드를 제거한 UserDaoTest
>```java
>import org.junit.Before;
>...
>public class UserDaoTest{
>	private UserDao dao; // setUp() 메소드에서 만드는 오브젝트를 테스트 메소드에서 사용할 수 있도록 인스턴스 변수로 선언
>	
>	@Before // JUnit이 제공하는 애노테이션. @Test 메소드가 실행되기 전에 먼저 실행돼야 하는 메소드를 정의
>	public void setUp(){ // 각 테스트 메소드에 반복적으로 나타났던 코드 제거하고 별도의 메소드로 추출
>		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
>		this.dao = context.getBean("userDao", UserDao.class);
>	}
>}
>```

<br>

### JUnit이 하나의 테스트 클래스를 가져와 테스트를 수행하는 방식

> 1. 테스트 클래스에서 `@Test`가 붙은 public이고 void형이며 파라미터가 없는 테스트 메소드를 모두 찾음
> 
> 2. 테스트 클래스의 오브젝트를 하나 만듦
> 3. `@Before`가 붙은 메소드가 있으면 실행
> 4. `@Test`가 붙은 메소드를 하나 호출하고 테스트 결과를 저장해둠
> 5. `@After`가 붙은 메소드가 있으면 실행
> 6. 나머지 테스트 메소드에 대해 2~5번을 반복
> 7. 모든 테스트의 결과를 종합해서 돌려줌
> 
> <br>
> 
> @Before나 @After 메소드를 테스트 메소드에서 직접 호출하지 x => 서로 주고받을 정보나 오브젝트가 있다면 인스턴스 변수를 이용해야 함.
> 
> #### ✅ 한 가지 꼭 기억해야 할 사항 : 
> - **각 테스트 메소드를 실행할 때마다 테스트 클래스의 오브젝트를 새로 만듦**
>	- ex) UserDaoTest에서는 스프링 컨테이너에서 가져온 UserDao 오브젝트를 인스턴스 변수 dao에 저장해뒀다가, 각 테스트 메소드에서 사용하게 만듦
>
> - 한 번 만들어진 테스트 클래스의 오브젝트는 하나의 테스트 메소드를 사용하고 나면 버려짐
> 
> - 테스트 클래스가 `@Test` 테스트 메소드를 두 개 갖고 있다면, 테스트가 실행되는 중에 JUnit은 이 클래스의 오브젝트를 두 번 만들 것.

<br>

### 🧐 왜 테스트 메소드를 실행할 때마다 새로운 오브젝트를 만드는 걸까?
- JUnit 개발자는 **각 테스트가 서로 영향을 주지 않고 독립적으로 실행됨을 확실히 보장해주기 위해서** 매번 새로운 오브젝트를 만들게 했다. 
- 테스트 메소드의 일부에서만 공통적으로 사용되는 코드가 있다면?
	-  `@Before`보다는 **일반적인 메소드 추출 방법** 쓰는 게 나음.
	- 아니면 아예 공통적 특징 지닌 테스트 메소드 모아서 별도의 테스트 클래스로 만드는 방법도 생각해볼 수 있음.

<br>

### ✅ 픽스처(fixture)
> **테스트를 수행하는 데 필요한 정보나 오브젝트**
>
>일반적으로 픽스처는 여러 테스트에서 반복적으로 사용되기 때문에 @Before 메소드를 이용해 생성해두면 편리
>
>지금까지의 코드에서 픽스처 예시 : 
>- UserDaoTest에서는 dao
>- add() 메소드에 전달하는 User 오브젝트들
>=> 👀 중복된 코드 존재

<br>

### ✨ 2. UserDaoTest에서 add() 메소드에 전달하는 User오브젝트 중복 => @Before 메소드로 추출
>#### Fix: User 픽스처를 적용한 UserDaoTest 클래스
```java
public class UserDaoTest{ 
	private UserDao dao;
	private User user1; // 인스턴스 변수 선언
	private User user2;
	private User user3;

	@Before
	public void setUp() { // 오브젝트 생성은 @Before 메소드에서 진행
		...
		this.user1 = new User("gyumee", "박성철", "springno1");
		this.user2 = new User("dobby", "윤도비", "springno2");
		this.user3 = new User("bumjin", "박범진", "springno3");
	}
}
```

<br>

## 2.4 스프링 테스트 적용
현재까지의 테스트 코드 어느 정도 정리를 마친 것 같지만..
### ✅ 한 가지 찜찜한 부분 : 애플리케이션 컨텍스트 생성 방식

> #### 문제1 :
> - `@Before` 메소드가 테스트 메소드 개수만큼 반복 => 애플리케이션 컨텍스트도 3번 만들어짐
> - 빈이 많아지고 복잡해지면 **애플리케이션 컨텍스트 생성에 적지 않은 시간 걸릴 수 있음**
> 	
> 	- 애플리케이션 컨텍스트가 만들어질 때는 모두 싱글톤 빈 오브젝트를 초기화하기 때문 
> 
> #### 문제2:
> - 애플리케이션 컨텍스트 **초기화 시 어떤 빈은 독자적으로 많은 리소스를 할당하거나 독립적인 스레드를 띄우기도 함.**
> 
> 	- 이 경우 테스트를 마칠 때마다 애플리케이션 컨텍스트 내의 빈이 할당한 리소스 등을 깔끔하게 정리해주지 않으면 다음 테스트에서 새로운 애플리케이션 컨텍스트가 만들어지면서 문제 일으킬 수 있음.

<br>

테스트는 가능한 한 독립적으로 매번 새로운 오브젝트를 만들어서 사용하는 것이 원칙.

하지만, 애플리케이션 컨텍스트처럼 생성에 많은 시간과 자원이 소모되는 경우에는 테스트 전체가 공유하는 오브젝트를 만들기도 함.

이때도 테스트는 일관성 있는 실행 결과를 보장해야 하고, 테스트의 실행 순서가 결과에 영향을 미치지 않아야 함. 

다행히도 애플리케이션 컨텍스트는 초기화되고 나면 내부의 상태가 바뀌는 일은 거의 x 

빈은 싱글톤으로 만들었기 때문에 상태 갖지 x

<br>

#### ✅ 따라서 애플리케이션 컨텍스트는 1번만 만들고 여러 테스트가 공유해서 사용해도 된다.

> 그런데 문제는 **JUnit이 매번 테스트 클래스의 오브젝트를 새로 만듦**(여러 테스트가 함께 참조할 애플리케이션 컨텍스트를
> 오브젝트 레벨에 저장해두면 곤란)
> 
> => **해결방법 : 스프링이 직접 제공하는 애플리케이션 컨텍스트 테스트 지원 기능 사용하자**

<br>

## ✅ 테스트를 위한 애플리케이션 컨텍스트 관리

 ### 🌱 스프링은 JUnit을 이용하는 테스트 컨텍스트 프레임워크를 제공함.
>#### 기능 1)
>
>- 간단한 **애노테이션 설정**만으로 테스트에서 필요로 하는 애플리케이션 컨텍스트를 만들어 모든 테스트가 공유하게 할 수 있음.
>#### 기능 2)
>
>- 여러 개의 테스트 클래스가 있는데 모두 같은 설정파일을 가진 애플리케이션 컨텍스트 사용 => 스프링은 **테스트 클래스 사이에서도 애플리케이션 컨텍스트를 공유하게 해줌**
>
><br>
>
>#### Fix: 스프링 테스트 컨텍스트를 적용한 UserDaoTest 클래스
>`@Before` 메소드에서 아래 코드 삭제**
>```java
>ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
>```
>
>아래와 같이 애노테이션 추가
>```java
>@RunWith(SpringJUnit4ClassRunner.class) // 스프링의 테스트 컨텍스트 프레임워크의 JUnit 확장기능 지정
>@ContextConfiguration(location="/applicationContext.xml") // 테스트 컨텍스트가 자동으로 만들어줄 애플리케이션 컨텍스트의 위치 지정
>public class UserDaoTest {
>	@Autowired
>	private ApplicationContext context; // 테스트 오브젝트가 만들어지고 나면 스프링 테스트 컨텍스트에 의해 자동으로 값이 주입됨
>	...
>	@Before
>	public void setUp() {
>		this.dao = this.context.getBean("userDao", UserDao.class);
>		...
>	}
>}
>```
>
> #### ✅ `@RunWith` : JUnit 프레임워크의 테스트 실행 방법을 확장할 때 사용
> - SpringJUnit4ClassRunner라는 JUnit용 테스트 컨텍스트 프레임워크 확장 클래스 지정해주면 JUnit이 테스트를 진행하는 중에 테스트가 사용할 애플리케이션 컨텍스트를 만들고 관리하는 작업 진행해줌
> 
> #### ✅ `@ContextConfiguration` : 자동으로 만들어줄 애플리케이션 컨텍스트의 설정파일 위치 지정
> 
> #### ✅ `@Autowired` : 스프링의 DI에 사용되는 특별한 애노테이션 
> - @Autowired가 붙은 인스턴스 변수가 있으면, 테스트 컨텍스트 프레임워크는 변수 타입과 일치하는 컨텍스트 내의 빈 찾음
>
>- 타입 일치하는 빈이 있으면 인스턴스 변수가 주입해줌
>
>- 일반적으로는 주입을 위해서 생성자나 수정자 메소드 같은 메소드가 필요하지만, 이 경우엔 메소드가 없어도 주입 가능
>
>- **별도의 DI 설정 없이 필드의 타입정보를 이용해 빈 자동으로 가져올 수 있음** -> 이런 방법을 **타입에 의한 자동와이어링**이라고 함
>
>- 단, @Autowired는 같은 타입의 빈이 두 개 이상 있는 경우에는 타입만으로는 어떤 빈을 가져올지 결정할 수 없음


> #### 추가할 라이브러리 : 
>
>com.springframework.test-3.0.7.RELEASE.jar

<br>

### ✅ 스프링 애플리케이션 컨텍스트는 초기화 시 자기 자신도 빈으로 등록 -> 굳이 `getBean()` 사용하지 않아도 됨. 아예 UserDao 빈을 직접 DI 받을 수 있음

>#### Fix: UserDao를 직접 DI 받도록 UserDaoTest 클래스 수정

```java
...
public class UserDaoTest{
	@Autowired
	UserDao dao; // UserDao 타입 빈을 직접 DI 받음
}
```

<br>

### ✅ 인터페이스를 두고 DI를 적용해야 하는 이유 

> 1. 소프트웨어 개발에서 절대로 바뀌지 않는 것은 없기 때문
> 2. 클래스의 구현 방식은 바뀌지 않는다고 하더라도 인터페이스를 두고 DI를 적용하게 해두면 다른 차원의 서비스 기능을 도입할 수 있기 때문
> 3. 효율적인 테스트를 손쉽게 만들기 위해서

<br>

## ✅ 테스트에 DI를 이용하는 방법 3가지
### ✅ 방법 1. 테스트 코드에서 빈 오브젝트에 수동으로 DI

> 테스트 코드 내에서 직접 DI를 해도 된다.
> => 즉, UserDao가 사용할 DataSource 오브젝트를 테스트 코드에서 변경 가능.
> 
> 테스트하다가 DB 사용자 정보 모두 삭제될 수도 있으니... 
> 
> **테스트 코드에 의한 DI를 이용해서 테스트 중에 DAO가 사용할 DataSource 오브젝트를 바꿔주자!**
> 
>#### Fix: 테스트를 위한 수동 DI를 적용한 UserDaoTest
```java
...
@DirtiesContext // 테스트 메소드에서 애플리케이션 컨텍스트의 구성이나 상태를 변경한다는 것을 테스트 컨텍스트 프레임워크에 알려줌
public class UserDaoTest{
	@Autowired
	UserDao dao;

	

@Before
	public void setUp(){
		...
		// 테스트에서 UserDao가 사용할 DataSource 오브젝트를 직접 생성
		DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost/testdb", "spring", "book", true);
		dao.setDataSource(dataSource); // 코드에 의한 수동 DI
	}
}
```

#### 그런데 ... 테스트 코드에서 빈 오브젝트에 수동으로 DI 하는 방법은 **장점보다 단점이 많음.** 😥 

> (자세한 사항은 192p 참고)
>
>코드가 많아져 번거롭기도 하고, 애플리케이션 컨텍스트도 매번 새로 만들어야 하는 부담 존재
>

그래서 아예 테스트에서 사용될 DataSource 클래스가 빈으로 정의된 테스트 전용 설정파일을 따로 만들어두는 방법 이용하자.

<br>

### ✅ 방법 2. 테스트를 위한 별도의 DI 설정

> 두 가지 종류의 설정파일을 만들어서 
> 
>하나에는 서버에서 운영용으로 사용할 DataSource를 빈으로 등록해주고, 
>
>다른 하나에는 테스트에 적합하게 준비된 DB를 사용하는 가벼운 DataSource가 빈으로 등록되게 만들자. 
> 
> 그리고 테스트에서는 항상 테스트 전용 설정파일만 사용하게 해주자.

#### 기존의 applicationContext.xml 복사해서 test-applicationContext.xml 만들고, 다른 빈 설정 그대로 두고 dataSource 빈의 설정을 테스트용으로 바꾸기 
>#### Feat: test-applicationContext.xml 추가
```xml
<bean id="dataSource"
	class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
	<property name="driverClass" value="com.mysql.jdbc.Driver"/>
	<property name="url" value="jdbc:mysql://localhost/testdb"/>
	<property name="username" value="spring"/>
	<property name="password" value="book"/>
</bean>
```


>#### Fix: UserDaoTest에 테스트용 설정 파일 적용
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(location="/test-applicationContext.xml") 
public class UserDaoTest {```
```

<br>

### ✅ 방법 3. 스프링 컨테이너 없는 DI 테스트
>UserDaoTest는 UserDao코드가 DAO로서 DB에 정보를 잘 등록하고 잘 가져오는지만 확인하면 됨.
>
>스프링 컨테이너에서 UserDao가 동작함을 확인하는 일은 UserDaoTest의 관심사 x
>
>1장에서 DaoFactory를 만들어 의존관계 설정 책임을 분리하기 직전에 테스트 코드에서 DI 작업을 직접 했던 것이 기억나는가?
>
>그 방법을 사용해서 UserDao가 동작하게 만들고 테스트할 수 있음.
>
> #### Fix: 스크링 컨테이너 없이 테스트 코드의 수동 DI만을 이용하도록 UserDaoTest 클래스 수정
```java
public class UserDaoTest{
	UserDao dao; // @Autowired가 없음

	@Before
	public void setUp(){
		...
		// 오브젝트의 생성, 관계설정 등을 모두 직접 해줌
		dao = new UserDao();
		DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost/testdb", "spring", "book", true);
		dao.setDataSource(dataSource);
	}
}
```
>#### 코드 설명 : 
>
> - @RunWith 사용해 스프링 테스트 컨텍스트 프레임워크 적용하지 x
> - @Autowired 사용해서 애플리케이션 컨텍스트에서 UserDao를 가져오지도 x
> - 대신 @Before 메소드에서 직접 UserDao의 오브젝트를 생성하고, 테스트용 DataSource 오브젝트를 만들어 직접 DI 해줌

<br>

DI는 객체지향 프로그래밍 스타일이다.

따라서, **DI를 위해 컨테이너가 반드시 필요한 것은 x**

**DI 컨테이너나 프레임워크는 DI를 편하게 적용하도록 도움을 줄 뿐, 컨테이너가 DI를 가능하게 해주는 것은 아님.**

<br>

## 🧐 DI를 테스트에 이용하는 3가지 방법 중 어떤 것을 선택해야 할까?
세 가지 모두 장단점 있고 상황에 따라 유용하게 사용 가능.

#### 가장 우선적으로 고려해야 하는 방법
>**방법 3. 스프링 컨테이너 없이 DI 테스트**
>
>- 테스트 수행 속도 가장 빠르고 테스트 자체가 간결

<br>

#### 여러 오브젝트와 복잡한 의존관계 갖고 있는 오브젝트를 테스트할 경우
> **방법 2. 스프링 설정을 이용한 DI 방식의 테스트**를 이용하면 편리.
>
>- 테스트에서 애플리케이션 컨텍스트를 사용하는 경우에는 테스트 전용 설정파일을 따로 만들어 사용하는 편이 좋음.

<br>

#### 예외적인 의존관계를 강제로 구성해서 테스트해야 할 경우
> **방법 1. 컨텍스트에서 DI 받은 오브젝트에 다시 테스트 코드로 수동 DI 해서 테스트** 하는 방식을 사용하면 됨
>
>- 테스트 메소드나 클래스에 @DirtiesContext 애노테이션 붙이는 것을 잊지 말자.
>

<br>

## 2.5 학습 테스트로 배우는 스프링
### ✅ 학습 테스트(learning test)
> #### 자신이 만들지 않은 프레임워크나 다른 개발팀에서 만들어서 제공한 라이브러리 등에 대한 테스트
>
> #### [학습 테스트의 목적]
> - 자신이 사용할 API나 프레임워크의 기능을 테스트 코드를 작성해보면서 빠르고 정확하게 사용법을 익히는 것
> 
> - 사용 방법 알고 있는지를 검증
>
><br>
>
>#### [학습 테스트의 장점]
>- 다양한 조건에 따른 기능을 손쉽게 확인 가능
>
>- 학습 테스트 코드를 개발 중에 참고 가능
>
>- 프레임워크나 제품을 업그레이드할 때 호환성 검증 도와줌
>
>- 테스트 작성에 대한 좋은 훈련이 됨
>
>- 새로운 기술을 공부하는 과정이 즐거워짐
>
><br>
>
>#### [스프링 학습 테스트 만들 때 참고할 수 있는 가장 좋은 소스]
>- 스프링 자신에 대한 테스트 코드(스프링 배포판의 압축 풀어봤을 때 있는 테스트 코드)

<br>

## ✅ 학습 테스트 예제
### ✅ 예제 1. JUnit으로 만드는 JUnit 자신에 대한 테스트 

> #### [테스트 방법]
> 1. 새로운 테스트 클래스 만들고 적당한 이름으로 세 개의 테스트 메소드 추가
> 
> 2. 스태틱 변수로 테스트 오브젝트를 저장할 수 있는 컬렉션 만듦
> 
> 3. 테스트마다 현재 테스트 오브젝트가 컬렉션에 이미 등록되어 있는지를 확인하고, 없으면 자기 자신을 추가 - 이 과정을 반복
> 
> => 테스트가 어떤 순서로 실행되는지에 상관없이 오브젝트 중복 여부를 확인 가능

```java
import static org.junit.matchers.JUnitMatchers.hasItem;
...
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

<br>

### ✅ 예제 2. 스프링 테스트 컨텍스트 프레임워크에 대한 학습 테스트

> JUnit과 반대로 스프링의 테스트용 애플리케이션 컨텍스트는 테스트 개수에 상관없이 한 개만 만들어짐.
> 
> 이렇게 만들어진 컨텍스트는 모든 테스트에서 공유됨
> 
> #### [테스트 방법]
> 1. 새로운 설정 파일 junit.xml 생성
> 
> 2. 앞에서 만들었던 JUnitTest 클래스에 @RunWith과 @ContextConfiguration 애노테이션 추가, 방금 만든 설정파일 사용하는 테스트 컨텍스트 적용
> 
> 3. @Autowired 로 주입된 context 변수가 같은 오브젝트인지 확인하는 코드 추가

>#### Feat: JUnit 테스트를 위한 빈 설정파일 junit.xml 추가
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```
>#### Fix: 스프링 테스트 컨텍스트에 대한 학습 테스트를 위해 JUnitTest 수정
```java
...
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

### ✅ 위에 코드에서 볼 수 있는 다양한 검증 방법 
각 테스트 메소드마다 다른 방법을 사용했지만, 모두 같은 내용 검증하는 코드임.

1. **`assertThat()`과 `is()`를 적절히 사용**
	- 매처와 비교할 대상인 첫 번째 파라미터에 Boolean 타입의 결과가 나오는 조건문 넣고, 그 결과를 `is()` 매처를 써서 true와 비교
	- `is()`는 타입만 일치하면 어떤 값이든 검증 가능
2. **조건문 받아서 그 결과가 true인지 false인지 확인하도록 만들어진 `assertTrue()` 사용**

3. **조건문 넣어서 그 결과를 true와 비교하는 대신 매처의 조합을 이용하는 방법 사용** 

	- `either()`는 뒤에 이어서 나오는 `or()`와 함께 두 개의 매처의 결과를 OR 조건으로 비교해줌
	- 두 가지 매처 중 하나만 true로 나와도 성공
	- `nullValue()` : 오브젝트가 null인지 확인

<br>

### ✅ 버그 테스트(bug test)
 > **코드에 오류가 있을 때 그 오류를 가장 잘 드러내줄 수 있는 테스트**
 > 
 > 일단 실패하도록 만들어야 함
 > 
 > 버그가 원인이 되서 테스트가 실패하는 코드를 만들자.
>
> 그리고 나서 버그 테스트가 성공할 수 있도록 애플리케이션 코드를 수정 
>
>테스트가 성공하면 버그는 해결된 것.
><br>
>
> #### [버그 테스트의 필요성과 장점] 
> 
> - 테스트의 완성도를 높여줌
> 
> - 버그의 내용을 명확하게 분석하게 해줌
> 
> - 기술적인 문제를 해결하는 데 도움이 됨

<br>

### ➕ 테스트 기법 
#### ✅ 동등분할(equivalence partitioning)
> **같은 결과를 내는 값의 범위를 구분해서 각 대표 값으로 테스트하는 방법**
>
>어떤 작업의 결과의 종류가 true, false 또는 예외발생 세 가지라면 각 결과를 내는 입력 값이나 상황의 조합을 만들어 모든 경우에 대한 테스트를 해보는 것이 좋음

#### ✅경계값 분석(boundary value analysis)
> **에러는 동등분할 범위의 경계에서 주로 많이 발생한다는 특징을 이용해 경계의 근처에 있는 값을 이용해 테스트하는 방법**
>
>보통 숫자의 입력 값인 경우 0이나 그 주변 값 또는 정수의 최대값, 최소값 등으로 테스트해보면 도움 될 때 많음
