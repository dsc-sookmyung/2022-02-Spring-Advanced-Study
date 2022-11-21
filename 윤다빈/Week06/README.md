# 6장 AOP

🔗  [https://pickle-fireplant-fa1.notion.site/6-eb173cd8e16a46dbb50726e5ee8ce7aa](https://pickle-fireplant-fa1.notion.site/6-eb173cd8e16a46dbb50726e5ee8ce7aa)

<aside> 💡 서비스 추상화를 통해 여러 문제를 해결했던 트랜잭션 설정 기능을 AOP로 개선하자

</aside>

## **트랜잭션 코드의 분리**

비즈니스 로직과 트랜잭션 경계설정의 분리로 성격이 다른 코드를 독립적이게 만들 수 있음

```java
public void upgradeLevels() throws Exception {
    TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
        upgradeLevelsInternal();
        this.transactionManager.commit(status);
    } catch (Exception e) {
        this.transactionManager.rollback(status);
        throw e;
    }
}

private void upgradeLevelsInternal() { //비즈니스 로직 분리
    List<User> users = userDao.getAll();
    for(User user : users) {
        if(canUpgradeLevel(user)) {
            upgradeLevel(user);
        }
    }
}

```

****DI를 이용한 클래스의 분리****

-   리팩토링 했지만 트랜잭션 로직이 여전히 UserService 안에 있음
-   DI의 기본 아이디어인 실제로 쓸 오브젝트의 클래스는 감추고 인터페이스를 통해 간접적으로 접근하도록 함
-   UserService를 인터페이스로 만들고 기존 코드는 이 인터페이스의 구현 클래스에 넣어 유연하게 확장
-   트랜잭션은 DI를 이용해 트랜잭션 기능을 가진 오브젝트가 먼저 실행되도록 만듦

```java
public interface UserService {
    void add(User user);
    void upgradeLevels();
}

public class UserServiceImpl implements UserService {
    UserDao userDao;
    MailSender mailSender;
    
    public void upgradeLevels() {
        List<User> users = userDao.getAll();
        for(User user : users) {
            if(canUpgradeLevel(user)) {
                upgradeLevel(user);
            }
        }
    }
    ...
}

public class UserServiceTx implements UserService { // UserService 구현 오브젝트
    UserService userService;
    PlatformTransactionManager transactionManager;
    
    public void serTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }
    
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
    
    public void add(User user) {
        this.userService.add(user);
    }
    
    public void upgradeLevels() {
        TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
        try {
            userService.upgradeLevels();
            this.transactionManager.commit(status);
        } catch (RuntimeException e) {
            this.transactionManager.rollback(status);
            throw e;
        }
    }
}

```

-   UserService 인터페이스의 구현 클래스를 두개 사용한다
-   새로운 하나는 트랜잭션의 경계설정이라는 책임만 가지며 또 다른 구현 클래스에 실제 로직 처리 작업을 위임
-   UserServiceTx는 비지니스 로직이 전혀 없으며 이를 UserService 구현 오브젝트로 위임
-   가장 먼저 트랜잭션 담당 오브젝트가 쓰여 이에 관련된 작업을 하고, 이후 실제 비지니스 로직을 담은 오브젝트가 호출되어 관련된 작업을 수행하도록 만듦

> **트랜잭션 경계 설정 코드의 분리를 통한 장점**
> 
> 1.  비즈니스 로직을 담당하고 있는 클래스를 작성할 때 트랜잭션과 같은 기술적인 내용 신경X
> 2.  비즈니스 로직에 대한 테스트를 손쉽게 만들 수 있음

## **고립된 단위 테스트**

> **테스트의 대상이 외부 환경등에 영향받지 않도록 고립시켜야 함**

최선의 테스트 방법은 가능한 한 작은 단위로 쪼개서 테스트하는 것인데, 테스트 대상이 다른 오브젝트나 환경에 의존적이라면 작은 단위 테스트의 장점을 얻기 힘듦. 예로 UserServiceTest의 대상은 UserService 클래스이고, 이는 UserDao, TransactionManager와 의존관계를 갖고 있어 테스트 동안 같이 실행되므로 UserService에 문제 없어도 테스트 실패 가능성 존재

**테스트를 의존 대상으로 분리해서 고립시키는 방법**

-   테스트를 위한 대역 Test Double의 사용
-   트랜잭션 코드를 독립시켰으므로, UserServiced의 구현 클래스는 PlatformTransactionManager 의존X
-   UserDao : 테스트 대상의 정상 수행을 도우며, 부가적인 검증 기능까지 추가한 목 오브젝트로 만듦
-   목 오브젝트는 테스트 대상을 통해 쓰일때, 필요한 기능을 지원해주어야 함
-   목 오브젝트는 스텁과 같은 방식으로 테스트 대상을 통해 사용될 때 필요한 기능을 지원해줌
-   결과 : UserServiceImple에 대한 테스트를 두개의 목 오브젝트에만 의존하는 고립된 테스트 대상으로 만듦

```java
@Getter
**static** class MockUserDao implements UserDao { //= **UserServiceTest 에서만 쓸 것이라**

  private List<User> users;
  private List<User> updated = new ArrayList<>();

  private MockUserDao(List<User> users) {
    this.users = users;
  }

  @Override
  public List<User> getAll() { return this.users;    // 스텁 기능 제공 }

  @Override
  public void update(User user) { updated.add(user);    // 목 오브젝트 기능 제공 }    

  @Override
  public void add(User user) { throw new UnsupportedOperationException();}

  @Override
  public User get(String id) { throw new UnsupportedOperationException();}

  @Override
  public void deleteAll() { throw new UnsupportedOperationException();}

  @Override
  public int getCount() { throw new UnsupportedOperationException();}
}
// 인터페이스를 구현하면 사용하지 않는 메서드도 모두 만들어주어야 한다는 부담을 줄이기 위해
// throw new UnsupportedOperationException()으로 예외 명시

```

```java
@Test
  public void upgradeLevels() throws Exception {
    UserServiceImpl userServiceImpl = new UserServiceImpl();

    MockUserDao mockUserDao = new MockUserDao(this.users);
    UserLevelUpgradePolicy userLevelUpgradePolicy = new UserLevelUpgradePolicyImpl();
    userServiceImpl.setUserDao(mockUserDao);
    userServiceImpl.setUserLevelUpgradePolicy(userLevelUpgradePolicy);

    userServiceImpl.upgradeLevels();

    List<User> updated = mockUserDao.getUpdated();
    assertThat(updated.size(), is(2));
    checkUserAndLevel(updated.get(0), "bbb", Level.SILVER);
    checkUserAndLevel(updated.get(1), "ddd", Level.GOLD);
  }

  private void checkUserAndLevel(User updated, String expectedId, Level expectedLevel) {
    assertThat(updated.getId(), is(expectedId));
    assertThat(updated.getLevel(), is(expectedLevel));
  }

```

****단위 테스트와 통합 테스트****

-   단위 테스트 : 테스트 대역을 써서 의존 오브젝트나 외부의 리소스를 안쓰도록 고립시켜서 테스트하는 것
-   통합 테스트 : 두 개 이상의, 성격이나 계층이 다른 오브젝트가 연동하도록 만들어 테스트 혹은 외부의 DB나 파일, 서비스 등의 리소스가 참여하는 테스트

**목 프레임워크**

-   코드들이 대부분 의존 오브젝트를 필요하므로 단위 테스트를 만들땐 스텁이나 목 오브젝트 사용이 필수
-   단위테스트의 단점
    -   목 오브젝트를 만들어야 하므로 작성이 번거로움
    -   테스트 메서드별로 다른 검증이 필요하다면, 동일 의존 인터페이스를 구현한 목 클래스 여러번 선언
-   단위 테스트의 번거로움을 해결해주는 다양한 목 오브젝트 지원 프레임워크 존재 (**Mockito**)

**Mockito의 사용**

-   목 클래스를 따로 선언할 필요없이, 간단한 메서드 호출만으로 테스트용 목 오브젝트를 생성

```java
@Test
public void upgradeLevels() throws Exception {
  UserServiceImpl userServiceImpl = new UserServiceImpl();

  UserLevelUpgradePolicy userLevelUpgradePolicy = new UserLevelUpgradePolicyImpl();
  UserDao mockUserDao = **mock(UserDao.class);  // 메서드 호출로 테스트용 목 오브젝트 생성**
  when(mockUserDao.getAll()).thenReturn(this.users); // 기능 없으므로 스텁 기능 추가

  userServiceImpl.setUserDao(mockUserDao);
  userServiceImpl.setUserLevelUpgradePolicy(userLevelUpgradePolicy);

  userServiceImpl.upgradeLevels();

  verify(mockUserDao, times(2)).update(any((User.class))); // update 메서드가 2번 호출되었는지 확인
  verify(mockUserDao).update(users.get(1));
  assertThat(users.get(1).getLevel(), is(Level.SILVER));
  verify(mockUserDao).update(users.get(3));
  assertThat(users.get(3).getLevel(), is(Level.GOLD));
}

```

**Mockito 목 오브젝트 사용법**

1.  인터페이스를 사용해 목 오브젝트 생성

(2) 목 오브젝트의 리턴값 지정 (있을때만) 메서드 호출시 예외를 강제로 던지게 할 수도 있음

3.  테스트 대상 오브젝트에 DI 해서 목 오브젝트가 테스트 중에 쓰이도록 함

(4) 테스트 대상 오브젝트 사용 후 목 오브젝트의 특정 메서드가 호출됐는지, 몇번 어떤 값을 가졌는지등 검증

## ****다이내믹 프록시와 팩토리 빈****

### 프록시와 프록시 패턴, 데코레이터 패턴

-   부가기능 외의 나머지 모든 기능은 원래 핵심기능을 가진 클래스로 위임
-   핵심 기능은 부가기능을 가진 클래스의 존재 자체 모름 → 부가기능이 핵심기능을 사용하는 구조
-   클라이언트가 핵심기능을 가진 클래스를 직접 쓰면 부가기능이 적용될 기회가 없으므로 이러면 X
-   부가기능은 클라이언트가 자신을 거쳐 핵심기능을 쓰도록 자신이 핵심기능을 가진 클래스로 보이게함.
-   **프록시**  : 자신이 클라이언트가 쓰려는 실제 대상인 것처럼 위장해 클라이언트의 요청을 받아주는 대리인 역할
-   타깃 : 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트

****프록시 사용 목적****

> ****프록시 사용 목적****  - 둘다 프록시를 두지만, 목적에 따라 디자인 패턴에선 구분함

-   클라이언트가 타깃에 접근하는 방법을 제어하기 위함
-   타깃에 부가적인 기능을 부여해주기 위함

****데코레이터 패턴****

-   타깃에 부가적인 기능을 런타임 시  **다이내믹**하게 부여해주기 위해 프록시를 사용하는 패턴
-   코드상으로 어떤 방법과 순서로 프록시와 타깃이 연결되어 사용되는지 정해지지 않음
-   같은 인터페이스를 구현한 타겟과 여러 프록시 사용 가능 - 프록시가 하나로 결정된거 아님
-   위임 대상에도 인터페이스로 접근하므로, 다음 위임대상은 외부에서 런타임 시 주입받도록 만들어야함
-   인터페이스를 통한 데코레이터 정의와 런타임 시의 다이내믹한 구성 방법 → 스프링의 DI 이용시 편리
-   ex) 자바 IO 패키지의 InputStream과 OutputStream 구현 클래스

****프록시 패턴****

-   디자인 패턴에서 프록시 패턴 : 타깃의 기능 확장이나 추가 X, 타깃에 대한 접근 방법을 변경함
-   타깃의 기능 자체에는 관여하지 않고 접근 방법을 제어
-   프록시 사용 목적을 보고 구분하면, 어떤 목적으로 프록시가 쓰였는지, 어떤 패턴이 적용되었는지 알 수 있음

### ****다이내믹 프록시****

> 프록시를 만드는 번거로움을 해결하기 위해 자바의 java.lang.reflect 패키지 사용

****프록시의 구성과 프록시 작성의 문제점****

-   프록시의 기능
    -   타깃과 같은 메소드를 구현하고 있다가 메소드 호출시 타깃 오브젝트로 위임
    -   지정된 요청에 대해서는 부가기능을 수행
-   **프록시 만들기 번거로운 이유**
    -   타깃의 인터페이스를 구현하고 위임하는 코드의 작성이 번거로움 → 해결법 JDK의 다이나믹 프록시
    -   부가기능 코드의 중복 가능성이 높음

****리플렉션****

-   다이내믹 프록시는 리플렉션 기능을 이용해서 프록시 생성, 자바의 코드 자체를 추상화해서 접근하도록 만듦
-   자바의 모든 클래스는 그 클래스 자체의 구성정보를 담은 Class 타입의 오브젝트를 하나씩 가잠
-   클래스 오브젝트로 클래스 코드에 대한 메타정보를 가져오거나 오브젝트를 조작 가능

****다이나믹 프록시 적용과 확장****

> **프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어지는 오브젝트로 타깃의 인터페이스와 동일한 타입**

-   클라이언트는 다이내믹 프록시 오브젝트를 타깃 인터페이스를 통해 사용
-   프록시 팩토리에게 인터페이스 정보 제공시 해당 인터페이스를 구현한 클래스의 오브젝트 자동 생성
-   프록시로써 필요한 부가기능 제공 코드는 직접 생성 - 부가기능은 InvocationHandler 구현 오브젝트에 담음
-   리플랙션으로 메소드, 파라미터 정보를 모두 가지므로 타깃 오브젝트의 메소드 호출 가능

```java
public class UppercasteHandler implements InvocationHandler {
    Hello target;
    public UppercaseHandler (Hello target) {
        this.target = target;
    }
    public Object **invoke**(Object proxy, Method method, Object[] args) throws Throwable{
        String ret = (String) method.invoke(target, args);
        return ret.toUpperCase();
    }
}

```

```java
// 생성된 다이내믹 프록시 오브젝트는 Hello 인터페이스를 구현하고 있으므로 Hello 타입 캐스팅 OK
Hello proxiedHello = (Hello)Proxy.newProxyInstance(
	  getClass().getClassLoader(), // 동적으로 만들어지는 다이내믹 프록시 클래스의 로딩에 쓸 클래스 로더
    new Class[] { Hello.class }, // 구현할 인터페이스
    new UppercaseHandler(new HelloTarget())); // 부가기능과 위임 코드를 담은 InvocationHandler

```

-   다이내믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환후 invoke() 메소드로 넘김
-   타깃 인터페이스의 모든 메소드 요청이 하나의 메소드로 집중
-   다이나믹 프록시로 요청이 전달되면 리플렉션 API로 타깃 오브젝트의 메서드 호출
-   타깃 오브젝트는 생성자를 통해 미리 받아둠
-   다이나믹 프록시의 생성은 Proxy 클래스의 newProxyInstance() 메소드 사용

```java
// 다이나믹 프록시의 확장
public class UppercaseHandler implements InvocationHandler {
// 어떤 종류의 인터페이스를 구현한 타깃에도 적용할 수 있게 Object 타입으로 
    Object target;
    private UppercaseHandler(Object target) {
        this.target = target;
    }

    public Object invoke(Object proxy, Method method, Object[] args) thorws Throwable {
        Object ret = method.invoke(target, args);
        if (Ret instanceof String) {
            return ((String)ret).toUpperCase();
        } else {
            return ret;
        }
    }
}

```

### **다이내믹 프록시를 이용한 트랜잭션 부가기능**

### **다이내믹 프록시를 위한 팩토리 빈**

<aside> 🤔 TransactionHandler와 다이내믹 프록시를 스프링의 DI를 통해 쓰도록 만들것

</aside>

-   다이나믹 프록시는 Proxy 클래스의 newProxyInstance() 메소드로만 생성 가능
-   스프링의 빈은 클래스 이름과 프로퍼티로 정의되며 이 이름을 가지고 리플렉션을 써서 클래스 오브젝트 생성
-   클래스의 이름이 있다면 newInstance() 메소드로 새로운 오브젝트를 생성 가능
-   다이내믹 프록시 오브젝트는 윗 방식으로 프록시 오브젝트 생성되지 않음 → 사전에 스프링의 빈에 정의X

**팩토리 빈**

-   스프링은 빈을 만들 수 있는 여러 방법 제공 → 대표적으로 팩토리 빈을 이용한 빈 생성 방법이 있음
-   팩토리 빈 : 스프링을 대신해 오브젝트의 생성로직을 담당하도록 만들어진 특별한 빈
-   팩토리 생성 방법중 가장 간단한 방법은 스프링의 FactoryBean이라는 인터페이스를 구현하는 것
-   스프링은 private 생성자를 가진 클래스도 빈으로 등록하면 리플렉션을 써서 오브젝트를 만들어줌

**다이나믹 프록시를 만들어주는 팩토리 빈**

![https://jongmin92.github.io/images/post/2018-04-15/dynamic-proxy-with-factorybean.png](https://jongmin92.github.io/images/post/2018-04-15/dynamic-proxy-with-factorybean.png)

-   다이내믹 프록시 오브젝트는 일반적인 방법으로는 스프링의 빈을 등록 불가
-   팩토리 빈을 사용하면 다이내믹 프록시 오브젝트를 스프링의 빈으로 만들어줄 있음
-   팩토리 빈의 getObject() 메소드에 다이내믹 프록시 오브젝트를 만들어주는 코드를 넣으면 됨-

### ****프록시 팩토리 빈 방식의 장점과 한계****

-   장점
    -   타깃 인터페이스를 구현하는 클래스를 일일이 만드는 번거로움 사라짐 → 부가기능 코드의 중복 문제도 X
    -   한번 부가기능을 가진 프록시를 생성하는 팩토리 빈을 만들면 타깃 타입에 무관하게 재사용 가능
    -   다이나믹 프록시에 팩토리 빈을 이용한 DI까지 더하면 번거로운 다이나믹 프록시 생성 코드 제거 가능
    -   DI 설정 만으로도 다양한 타깃 오브젝트에 적용 가능
-   한계
-   한번에 여러 클래스에 공통적인 부가 기능을 제공하는 것은 불가능
-   하나의 타깃에 여러 부가기능을 적용하려고 할 때도 프록시 팩토리 빈 설정이 반복될 것

<aside> 🤔  ****TransactionHandler의 중복을 없애고 모든 타깃에 적용 가능한 싱글톤 빈으로 만들어서 적용할래****

</aside>

## ****스프링의 프록시 팩토리 빈****

**ProxyFactoryBean**

-   스프링은 일관된 방법으로 프록시를 만들 수 있게 도와주는 추상 레이어 제공
    
-   스프링은 **프록시 오브젝트를 생성해주는 기술을 추상화한 팩토리 빈**을 제공
    
-   스프링의 **`ProxyFactoryBean`** : 프록시를 생성 후 빈 오브젝트로 등록하게 해주는 팩토리 빈
    
    ```
      ⬆️ 프록시를 생성하는 작업만 담당하고, 프록시를 통해 제공해줄 부가 기능은 별도의 빈에 둘 수 있음 
    
    ```
    
-   MethodInterceptor 인터페이스
    
    -   ProxyFactoryBean이 만든 프록시에서 쓸 부가기능을 담음
    -   invoke() 메서드 : ProxyFactoryBean으로부터 타겟 오브젝트의 정보도 함께 받음 → 오브젝트 상관 X
    -   타겟이 다른 여러 프록시에서 함께 쓸 수 있으며 싱글톤 빈으로 등록될 수 있음
-   작은 단위의 템플릿/콜백 구조를 응용해서 적용했음
    
    -   템플릿 역할을 하는 MethodInvocation을 싱글톤으로 두고 공유 가능
    -   다수의 MethodInterceptor을 추가 가능

```java
// public class DynamicProxyTest 안의 내용
@Test
public void proxyFactoryBean() {
    ProxyFactoryBean pfBean =new ProxyFactoryBean();
    pfBean.setTarget(new HelloTarget());
    pfBean.addAdvice(new UppercaseAdvice());

    Hello proxiedHello = (Hello) pfBean.getObject();

    assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
    assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
    assertThat(proxiedHello.sayThankYou("Toby"), is("THANK YOU TOBY"));
}

static class UppercaseAdvice implements MethodInterceptor {

    @Override
    public Objectinvoke(MethodInvocation methodInvocation)throws Throwable {
        String ret = (String) methodInvocation.proceed();
        return ret.toUpperCase();
    }
}

```

-   MethodInterceptor로는 메서드 정보와 함께 타겟 오브젝트가 담긴 MethodInvocation 오브젝트 전달
-   MethodInterceptor는 부가 기능을 제공하는데만 집중 가능
-   MethodInvocation는 일종 콜백 오브젝트, proceed() 호출시 타겟 오브젝트의 메서드를 내부적으로 실행

어드바이스`Advice`는 `MethodInterceptor` 처럼 타겟 오브젝트에 적용하는 부가 기능을 담은 오브젝트를 말한다. 어드바이스는 타겟 오브젝트에 종속되지 않는 순수한 부가 기능만을 담은 오브젝트이다.

`ProxyFactoryBean` 은 기본적으로 JDK가 제공하는 다이나믹 프록시를 만들어준다. 경우에 따라서는 CGLib라는 오픈소스 바이트코드 생성 프레임워크를 이용해 프록시를 만들기도 한다.

이전에 JDK 다이나믹 프록시를 이용한 방식에서는 `pattern` 을 통해 부가 기능을 적용시킬 메서드를 선정했었다.

> **어드바이스**

> **타깃 오브젝트에 적용하는 부가기능을 담은 오브젝트**
> 
> -   위의 MethodInterceptor을 어드바이스라고 부름
> -   타깃 오브젝트에 종속되지 않는 순수한 부가 기능을 담은 오브젝트 (타깃 필요 X)

> **포인트컷**

> **부가기능 적용 대상 메소드 선정 알고리즘을 담은 오브젝트**
> 
> -   MethodInterceptor 오브젝트를 여러 프록시가 공유할 수 있어 스프링의 싱글톤 빈으로 등록 O
> -   트랜잭션 적용 메소드 패턴은 프록시마다 다를 수 있어 MethodUnterceptor에 특정 패턴 넣으면X
> -   ProxyFactoryBean을 사용해 포인트컷을 써서 메소드 선정 가능

-   어드바이스와 포인트컷은 모두 프록시에 DI로 주입돼서 사용되며, 스프링의 싱글톤 빈으로 등록 가능
-   포인트컷은 Pointcut 인터페이스를 구현해 만들며, 어드바이스는 직접 타겟 호출하지 않음
-   프록시는 클라이언트로부터 요청을 받으면 먼저 포인트컷에게 부가기능을 부여할 메소드인지 확인 요청
-   프록시는 포인트컷으로부터 확인받으면, MethodInterceptor 타입의 어드바이스를 호출
-   어드바이스는 일종의 템플릿, 타깃을 호출하는 기능을 갖고 있는 MethodInvocation 오브젝트가 콜백이 됨
-   템플릿처럼 어드바이스도 독립적인 싱글톤 빈으로 등록하고 DI를 주입해서 여러 프록시가 사용할 수 있음
-   **프록시로부터 어드바이스와 포인트컷을 독립시키고 DI를 사용하게 한 것은 전형적인 전략 패턴 구조다.**

```java
@Test
public void pointcutAdvisor() {
    ProxyFactoryBean pfBean =new ProxyFactoryBean();
    pfBean.setTarget(new HelloTarget());
    
    // 메소드 이름을 비교해서 대상을 선정하는 알고리즘을 제공하는 포인트컷 생성
    NameMatchMethodPointcut pointcut =new NameMatchMethodPointcut();
    pointcut.setMappedName("sayH*"); //sayH로 시작하는 모든 메소드를 선택하도록

    // 포인트컷과 어드바이스를 advisor로 묶어서 한 번에 추가
    pfBean.addAdvisor(new DefaultBeanFactoryPointcutAdvisor(pointcut,new UppercaseAdvice()));

    Hello proxiedHello = (Hello) pfBean.getObject();

    assertThat(proxiedHello.sayHello("Toby"), is("HELLO TOBY"));
    assertThat(proxiedHello.sayHi("Toby"), is("HI TOBY"));
    assertThat(proxiedHello.sayThankYou("Toby"), is("Thank You Toby"));    
  //⬆️ 포인트컷의 선정조건에 맞지 않음
}

```

<aside> 🤔  **어드바이저 = 포인트컷(메서드 선정 알고리즘) + 어드바이스(부가 기능) (따로 등록X)**

</aside>
