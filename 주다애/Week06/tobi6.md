## 6. AOP-1

> AOP는 스프링의 3대 기반기술 중 하나다.

### 1. 트랜잭션 코드의 분리

> 트랜잭션 경계설정 코드를 분리하면 더 깔끔한 구조를 만들 수 있다.

현재 upgradeLeveles() 메소드에는 try/catch문으로 트랜잭션 경계설정 코드와 비즈니스 로직 코드가 같이 구현되어있다. 
1. 비즈니스 로직을 **upgradeLevelsInternal**()이라는 메소드로 분리하자
2. DI적용을 위해 UserService 인터페이스를 도입하자
--> UserServiceImpl과 UserService로 구분하자
--> UserService 클래스에서 트랜잭션 경계설정 코드를 **제거하자**

UserServiceTx클래스를 통해 트랜잭션 경계설정을 담당시키자

    public class UserServiceTx implements UserService {
	    UserService userService;
	    PlatformTransactionManager txManager;
	    
	    public void setTransactoinManager(PlatformTransactionManager txManager){
		    this.txManager = txManager; // DI
	    }
	    public void setUserService(UserService userService){
		    this.userService = userService; // DI
	    }
	    public void add(User user){
		    userService.add(user);
		}
		public void updgradeLevels() {
			TransactionStatus status = this.txManager
					.getTransaction(new DefaultTransactionDefinition());
			try {
				userService.upgradeLevels();
				this.txManager.commit(status);
			}
			catch(RuntimeException e) {
				this.txManager.rollback(status);
				throw e;
			}
		}
    }

3. 테스트 수정이 필요하다.

upgradeAllOrNothig() 테스트 메소드에 UserServiceTx 호출하게 만든다.

이렇게 트랜잭션 경계설정 코드를 분리하면 좋은점이 무엇이 있을까?
1. 비즈니스 로직 담당 코드를 작성 시 트랜잭션 기술을 신경쓸 필요가 없다.
2. 비즈니스 로직에 대한 테스트를 손쉽게 만들 수 있다.

### 2. 고립된 단위 테스트

> 가능한 작은 단위로 쪼개서 테스트를 하는 것이 가장 편하고 좋은 테스트 방법이다.

작은 단위의 테스트가 좋은 이유는 테스트가 실패했을 때 그 원인을 찾기 쉽기 때문이다.
따라서 테스트의 대상이 환경이나, 외부 서버, 다른 클래스의 코드에 종속되고 영향을 받지 않도록 고립시키는 것이 좋다.

#### 테스트를 위한 UserServiceImpl 고립
UserServiceImpl이 두 개의 목 오브젝트에만 의존하도록 만들자.
(MockUserDao, MockMailSender)

    static class MockUserDao implements UserDao{
	    private List<User> users;
	    private List<User> updated = new ArrayList();
	    private MockUserDao(List<User> users) {
		    this.users = users;
	    }
	    public List<User> getUpdated() {
		    return this.updated;
	    }
	    public List<User> getAll() {
		    return this.users;
	    }
	    pubilc void update(User user) {
		    updated.add(user);
	    }
	    // 사용되지 않는 메소드도 구현이 필요함
    }
이 테스트의 장점은?
--> DB에 직접 가지 않아도 미리 준비된 테스트용 리스트를 메모리에 갖고 있다가 돌려주는 역할을 MockUserDao가 수행해준다.

이 테스트의 단점은?
--> 인터페이스를 구현하기 때문에 사용하지 않는 메소드도 모두 만들어 주어 야 한다.

#### 테스트 수행 성능의 향상
upgradeLevels() 메소드의 테스트 수행시간은 0.001초보다 빠른 시간 내에 수행되었다.
어떻게 이렇게 성능을 향상시킨건가?

--> 고립된 테스트를 통해 다른 의존 대상에 영향을 받지 않게 했기 때문이다. 목 오브젝트를 작성하는 수고는 고립 테스트의 성능 향상을 위해서는 필수적이라고 볼 수 있다.

#### 단위 테스트와 통합 테스트
단위 테스트의 단위는? 정하기 나름이다!

**중요한 것은 하나의 단위에 초점을 맞추라는 것**

 - 단위 테스트 : 테스트 대상 클래스를 외부의 리소스를 사용하지 않도록 고립시키는 것
	😀 항상 단위 테스트 먼저 고려
 - 통합 테스트 : 두 개 이상의 다른 오브젝트카 연동하거나 외부 리소스가 참여하는 테스트 --> 두 개 이상 단위가 결합해서 동작하면서 테스트 하는 것
	1. 일단의 단위 테스트를 먼저 고려해야 한다.
	2. 외부 리소스가 필수적일 때는 통합 테스트
	3. DAO는 DB를 통해 로직을 수행하므로 통합 테스트로 분류된다.
	4. 의존 관계를 가지고 단위가 동작하면 통합 테스트를 진행시킨다.
	5. 스프링 테스트 컨텍스트 프레임워크 이용 테스트는 통합 테스트이다.

#### 목 프레임워크

> 단위 테스트를 위한 스텁이나 목 오브젝트는 필수적이다. 하지만 목 오브젝트 생성은 큰 부담이다.

스프링은 목 오브젝트 지원 프레임워크를 제공하는데 이것이 바로 **Mockito**이다.

#### Mockito 프레임워크

> Mockito 프레임워크는 목 클래스를 일일이 준비할 필요를 없애주고 mock() 메소드 호출을 통해 테스트용 목 오브젝트를 만들 수 있다.

    UserDao mockUserDao = mock(UserDao.class);
    when(mockUserDao.getAll()).thenReturn(this.users); // getAll()검증
    verify(mockUserDao, times(2)).update(any(User.class)); // update()검증

4단계의 사용법

1. 인터페이스를 이용해 목 오브젝트를 만든다.
2. 목 오브젝트가 리턴할 값이 있으면 지정해준다. 메소드가 호출되면 예외를 강제로 던지게 만들 수 있다.
3. 테스트 대상 오브젝트에 DI를 해서 목 오브젝트가 테스트 중에 사용되도록 만든다.
4. 테스트 대상 오브젝트를 사용한 후에 목 오브젝트의 특정 메소드가 호출됐는지, 어떤 값을 가지고 몇 번 호출됐는지를 검증한다.

Mockito를 적용한 테스트 코드

    @Test
    public void mockUpgradeLevels() throws Exception{
	    UserServiceImpl userServiceImpl = new UserServiceImpl();
	    // DI 및 목 오브젝트 생성
	    verify(mockUserDao, times(2)).update(any(User.class));
	    ...
    }

### 3. 다이나믹 프록시와 팩토리 빈

#### 프록시와 프록시 패턴, 데코레이터 패턴

트랜잭션 경계설정 코드를 비즈니스 로직에서 분리하면 기능은 분리되었지만 코드에는 그대로 남아있다.
부가기능은 핵심기능과 분리해주어야 한다.
![토비의 스프링 | D-log](https://leejaedoo.github.io/assets/img/%ED%95%B5%EC%8B%AC%EA%B8%B0%EB%8A%A5_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EC%9D%98%EC%A0%81%EC%9A%A9.jpeg)

> **프록시란** 마치 자신이 클라이언트가 사용하려고 하는 실제 대상인 것처럼 위장해서 클라이언트의 요청을 받아주는 대리자이다.
> **타겟**/**실체는** 프록시를 통해 최종적으로 요청을 위임받아 처리하는 실제 오브젝트이다.

![프록시와 타깃](https://leejaedoo.github.io/assets/img/%ED%94%84%EB%A1%9D%EC%8B%9C%EC%99%80_%ED%83%80%EA%B9%83.jpeg)

프록시의 특징 2가지는
1. 타겟과 같은 인터페이스를 구현한다.
2. 프록시가 타겟을 제어하는 위치에 있다.

프록시의 사용 목적 2가지는
3. 클라이언트가 타겟에 접근하는 방법을 제어하는 것
4. 타겟에 부가적인 기능을 부여하는 것이다.

#### 데코레이터 패턴
> 데코레이터 패턴은 타겟에 부가적인 기능을 런타임 시 다이나믹하게 부여해주기 위해 프록시를 사용하는 패턴이다.

다이나믹하게 기능을 부가한다는 의미는 컴파일시점, 즉 코드상에서는 어떤 방법과 순서로 프록시와 타깃이 연결되어 사용되는지 정해져 있지 않다는 뜻이다.

![데코레이터 패턴 적용 예](https://leejaedoo.github.io/assets/img/%EB%8D%B0%EC%BD%94%EB%A0%88%EC%9D%B4%ED%84%B0_%ED%8C%A8%ED%84%B4.jpeg)

* 데코레이터 패턴은 타깃의 코드를 손대지 않고 클라이언트가 호출하는 방법도 변경하지 않은 채로 새 기능을 추가할 때 유용하다.

#### 프록시 패턴

> 프록시 패턴은 타깃에 대한 접근 방법을 제어하려는 목적을 가진 패턴으로 타깃의 기능 변경은 하지 않는다.

프록시를 클라이언트에게 넘겨주면 타깃 오브젝트를 생성하는 것을 최대한 늦춘다는 장점이 있다.

![프록시 패턴과 데코레이터 패턴의 혼용](https://leejaedoo.github.io/assets/img/proxy_decorator.jpeg)

#### 다이나믹 프록시

지금까지의 프록시의 단점은 2가지로 볼 수 있다.
1. 타깃의 인터페이스를 구현하고 위임하는 코드를 작성하기가 번거롭다.
2. 부가기능 코드가 중복될 가능성이 많다.

--> 이러한 문제를 해결하는 것이 바로 JDK 다이내믹 프록시이다.

#### 리플렉션
다이내믹 프록시는 리플렉션 기능을 이용해서 프록시를 만들어준다.

리플렉션이란 자바의 코드 자체를 추상화해서 접근하도록 만든 것이다.

    Method lenghMethod = String.class.getMethod("length");
    assertThat((Integer)lenghMethod.invoke(name), is(6));

#### 프록시 클래스

    interface Hello{
	    String sayHello(String name);
	    String sayHi(String name);
	    String sayThankYou(String name);
    }

    public class HelloTarget implements Hello{
	    public String sayHello(String name){
		    return "Hello" + name;
	    }
	    ...
    }
    
    public class HelloUppercase implements Hello{
	    Hello hello;
	    
	    public HelloUppercase(Hello hello){
		    this.hello = hello;
	    }
	    public String sayHello(String name){
		    return hello.sayHello(name).toUpperCase();
	    }
	    // 위와 같은 메소드 반복 --> 결국 중복이 생긴다.
    }

#### 다이내믹 프록시 적용
다이내믹 프록시는 프록시 팩토리에게 인터페이스 정보만 넘겨주면 해당 인터페이스를 구현한 클래스를 자동으로 만들어준다.
![다이내믹 프록시의 동작방식](https://leejaedoo.github.io/assets/img/dynamic_proxy.jpeg)

* InvocatoinHandler 인터페이스의 Invoke() 메소드는 인터페이스의 메소드들을 하나로 처리해준다. --> 중복 제거의 역할

순서를 정리하면
다이나믹 프록시가 InvocationHandler에게 요청을 하고, invoke() 메소드가 호출이 된다. 그리고 리플렉션 API를 통해 타깃의 메소드가 호출되고 부가작업이 수행된 다음 다이나믹 프록시가 클라이언트에게 전달한다.

    public class UppercaseHandler implements InvocationHandler{
	    Hello target;
	    public UppercaseHandler(Hello target){
		    this.target = target;   
	    }
	    @Override
	    public Object invoke(Object  o,  Method  method,  Object[]  objects)  throws  Throwable{
		    String ret = (String)method.invoke(target, objects);
		    return ret.toUpperCase(); // 부가기능
	    }
    }
    // 다이내믹 프록시
    Hello  proxiedHello  =  (Hello)  Proxy.newProxyInstance{
	    getClass().getClassLoader(),  
	    new  Class[]{Hello.class},  
	    new  UppercaseHandler(new  HelloTarget())
    };
    
 InvocationHandler의 장점은 타깃이 종류에 구애받지 않는다는 것이다.

즉, 아래와 같이 Object 타입으로 DI가 가능하다.

    Object  target;  
    public  UppercaseHandler(Object  target)  {
      this.target  =  target; 
    }

#### 다이내믹 프록시를 이용한 트랜잭션 부가기능
TransactionHandler가 InvocationHandler를 구현하도록 트랜잭션 부가기능 제공 다이내믹 프록시를 따로 만들어서  UserServiceTx를 다이내믹 프록시 방법으로 전환하자.

#### 다이내믹 프록시를 위한 팩토리 빈
지금까지의 과정에서 문제는? 바로  DI의 대상이 되는 다이내믹 프록시는 일반적인 스프링의 빈으로는 등록하지 못한다는 점이다.

**팩토리 빈**이란 스프링을 대신해서 오브젝트의 생성로직을 담당하도록 만들어진 특별한 빈이다. 여기서는 FactoryBean이라는 인터페이스를 구현해서 팩토리 빈을 생성한다.

    public interface FactoryBean{
	    T  getObject()  throws  Exception;  
	    Class<?>  getObjectType();
	    boolean  isSingleton();
    }

#### 다이내믹 프록시를 만들어주는 팩토리 빈
팩토리 빈을 사용하면 다이내믹 프록시 오브젝트를 스프링의 빈으로 만들어줄 수 있다.
![팩토리 빈을 이용한 트랜잭션 다이내믹 프록시의 적용](https://leejaedoo.github.io/assets/img/factorybean_dynamic_proxy.jpeg)

팩토리 빈이 만드는 다이내믹 프록시는 구현 인터페이스나 타깃의 종류에 제한이 없다. 따라서 얼마든지 다이내믹 프록시의 재사용이 가능하다.

    public class TxProxyFactoryBean implements FactoryBean<Object>{
    } // TransactionHandler를 사용하는 다이내믹 프록시를 생성하는 팩토리 빈 클래스

#### 프록시 팩토리 빈 방식의 장점과 한계
장점
1. 프록시 팩토리 빈은 재사용이 가능하다.
2. 타깃 인터페이스를 구현하는 클래스를 일일이 만드는 번거로움을 제거할 수 있다.

단점
3. 여러 개의 클래스에 공통적인 부가기능을 제공하는 일은 불가하다.
4. 하나의 타깃에 여러 개의 부가기능을 적용하는 것도 문제가 된다.
5. TransactionHandler 오브젝트가 프록시 팩토리 빈 개수만큼 만들어진다.

### 4. 스프링의 프록시 팩토리 빈

> 스프링이 제공하는 ProxyFactoryBean은 기존 Proxy 오브젝트를 생성해주는 기술을 추상화한 FactoryBean이다.

ProxyFactoryBean이 생성하는 프록시에서 사용할 부가기능은 MethodInterceptor 인터페이스를 구현해서 만든다.
MethodInterceptor의 invoke() 메소드는 타깃 오브젝트에 대한 정보까지 함께 제공받기때문에 독립적으로 MethodInterceptor 를 만들 수 있다.
**MethodInterceptor 오브젝트는 타깃이 다른 여러 프록시에서 함께 사용이 가능하고, 싱글톤 빈으로도 등록이 가능하다**

#### 어드바이스:  타깃이 필요없는 순수한 부가기능
**어드바이스란** MethodInterceptor 처럼 타깃 오브젝트에 적용하는 부가기능을 담은 오브젝트로 타깃 오브젝트에 종속되지 않는 순수한 부가기능을 담은 오브젝트이다.

#### 포인트컷: 부가기능 적용 대상 메소드 선정 방법
**포인트컷이란** 메소드 선정 알고리즘을 담은 오브젝트이다.

기존 JDK 다이내믹 프록시를 이용하면 InvocationHandler 인터페이스가 메소드 선정 알고리즘과 부가기능 타깃에 의존을 하게된다. 
즉, 한번 빈으로 구성된 InvocationHandler  오브젝트는 특정 타깃을 위한 프록시에서 제한되어 여러 프록시에서 공유가 불가하다.

만약 스프링의 ProxyFactoryBean을 사용하면 어떻게 될까?
![ProxyFactoryBean_방식](https://leejaedoo.github.io/assets/img/ProxyFactoryBean_%EB%B0%A9%EC%8B%9D.jpeg)

위와 같이 포인트컷과 어드바이스가 분리된 모습을 볼 수 있다.
1. 프록시는 클라이언트로부터 요청을 받는다.
2. 그다음 먼저 포인트컷에게 부가기능을 부여할 메소드인지 확인을 요청한다.
3. 포인트컷은 PointCut 인터페이스를 구현해서 만들어지고, 위의 요청이 확인되면 MethodInterceptor 타입의 어드바이스를 호출한다.
4. 어드바이스는 직접 타깃을 호출하지않고 타깃 메소드의 호출이 필요하면 MethodInterceptor 타입 콜백 오브젝트의 proceed() 메소드를 호출한다.(어드바이스는 공유되므로 상태를 가지면 안된다.)

* 프록시로부터 어드바이스와 포인트컷을 독립시키고 DI를 사용하게 한 것은 전형적인 전략 패턴 구조다.
* 덕분에 프록시가 공유해서 사용이 가능하고 프록시와 팩토리 빈의 변경이 없이도 자유롭게 기능이 확장되어서 OCP를 준수하게 된다.

#### 어드바이저: 포인트컷+어드바이스
어드바이저란 어드바이스와 포인트컷을 묶은 오브젝트를 의미한다.

왜 어드바이스와 포인트컷을 묶어야 하는가?
* ProxyFactoryBean에 여러 개의 어드바이스와 포인트컷이 추가되면 조합이 만들어지기 때문이다. 
* addAdvisor() 메소드를 호출해서 어드바이스와 포인트컷을 Advisor 타입으로 묶어서 추가해야 한다.


#### ProxyFactoryBean 적용

    public class TransactionAdvisor implements MethodInterceptor{
	    PlatformTransactionManager  transactionManager;  
	    // transactoinManager DI해주기
	    public Object invoke(MethodInvocation invocation) throw Trowble{
		    // 콜백 호출해서 타깃의 메소드 실행
		    // 예외가 발생하면 예외 던짐(MethodInvocation은 예외를 
		    포장하지 않고 그대로 던진다.
	    }
    }

#### 어드바이스와 포인트컷의 재사용
ProxyFactoryBean 은 스프링의 DI와 템플릿/콜백 패턴, 서비스 추상화 등의 기법이 모두 적용된 것이다.

**어드바이스**를 하나 만들어서 **싱글톤 빈**으로 등록만 해주면, DI를 통해 필요한 서비스에서 적용이 가능하며 **포인트컷이 추가**된다면 어드바이스와 함께 조합하여 **어드바이저**로 만들어주면 된다.
![ProxyFactoryBean_Advice_Pointcut](https://leejaedoo.github.io/assets/img/ProxyFactoryBean_Advice_Pointcut.png)
