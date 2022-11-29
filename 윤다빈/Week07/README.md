# 6장 AOP (2)

🔗 [](https://www.notion.so/7-2714d717d2264813bac6424f88ed1d35)[https://pickle-fireplant-fa1.notion.site/7-2714d717d2264813bac6424f88ed1d35](https://pickle-fireplant-fa1.notion.site/7-2714d717d2264813bac6424f88ed1d35)

## 스프링 AOP

### 자동 프록시 생성

과제: 부가기능의 적용이 필요한 타깃 오브젝트마다 유사내용의 ProxyFactoryBean 빈 설정정보를 추가해줘야함

**중복 문제의 접근 방법**

-   JDBC API를 사용하는 DAO 코드의 해결법 → 템플릿 콜백 패턴
-   프록시 클래스 코드 : 다이내믹 프록시라는 런타임 코드 자동생성 기법 사용
    -   JDK의 다이내믹 프록시는 특정 인터페이스를 구현한 오브젝트에 대해 프록시 역할을 하는 클래스를 내부적으로 생성
    -   변하는 로직과 변하지 않는 기계적인 코드 분리
-   ProxyFactoryBean 설정 문제를 설정 자동 등록 기법으로 해결 가능한가 🔽

**빈 후처리기를 이용한 자동 프록시 생성기**

![https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.5.1.png](https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.5.1.png)

-   BeanPostProcessor 인터페이스를 구현해 생성하는 빈 후처리
-   **빈 후처리**  : 스프링 빈 오브젝트로 만들어진 뒤, 빈 오브젝트의 재가공을 할 수 있도록 함
    -   스프링은 빈 후처리기가 빈으로 등록되어 있으면 빈 오브젝트가 생성될때마다 빈 후처리기에 보내 후처리
    -   프록시 생성된 경우, 원래 컨테이너가 전달해준 빈 오브젝트 대신 프록시 오브젝트를 컨테이너에 반납
    -   컨테이너는 결과적으로 빈 후처리가 반납한 오브젝트를 빈으로 등록후 사용
-   DefaultAdvisorAutoProxyCreator :
    -   스프링이 제공하는 빈 후처리기의 일종
    -   빈으로 등록된 모든 어드바이저 내 포인트컷을 사용해 전달받은 빈이 프록시 적용 대상인지 체크
    -   프록시 적용 대상일 경우, 내장된 프록시 생성기에 현재 빈에 대한 프록시 만들게한 뒤 어드바이저에 연결

**확장된 포인트컷**

-   포인트컷은 클래스 필터와 메소드 매처 둘을 돌려주는 메소드를 가짐
    -   ClassFilter : 프록시에 적용할 클래스인지 확인
    -   MethodMatcher : 어드바이스를 적용할 메소드인지 확인
-   NameMatchMethodPointcut : 기존 사용, 메소드 선별 기능만 가진 특별한 포인트컷
-   ProxyFactoryBean에서 포인트컷 사용시 이미 타깃지정 → 포인트컷은 메소드 선별 수행 OK
-   포인트컷 선정 기능을 모두 적용시
    -   먼저 프록시를 적용할 클래스인지 판단
    -   적용 대상 클래스인 경우, 어드바이스를 적용할 메소드인지 확인
-   DefaultAdvisorAutoProxyCreator : 모든 빈에 대해 프록시 자동 적용 대상을 선별해야 하는 빈 후처리기
-   클래스와 메소드 선정 알고리즘이 모두 있는 포인트컷 즉, 포인트컷과 어드바이스가 결합된 어드바이저 등록

### DefaultAdvisorAutoProxyCreator의 적용

**클래스 필터를 적용한 포인트컷 작성**

-   NameMatchMethodPointcut 상속 - 메소드 이름만 비교하는 포인트컷
-   프로퍼티로 주어진 이름 패턴으로 클래스 이름을 비교하는 ClassFilter를 추가할 것

**어드바이저를 이용하는 자동 프록시 생성기 등록**

-   DefaultAdvisorAutoProxyCreator (자동 프록시 생성용 빈 후처리기)
    -   등록된 빈 중 어드바이저 인터페이스를 구현한 것을 모두 탐색
    -   생성되는 모든 빈에 대해 어드바이저의 포인트컷을 적용하며 프록시 적용 대상을 선정
-   기존 포인트컷의 설정 삭제 후 새로 만든 클래스 필터 지원 포인트컷을 빈으로 등록
-   어드바이저를 명시적으로 DI 하는 빈은 없지만 어드바이저를 이용하는 자동 프록시 생성기에 의해 자동 수집
-   프록시 대상 선정 과정에 참여하며 자동 생성된 프록시에 동적으로 DI되 작동하는 어드바이저가 됨

**자동생성 프록시 확인**

-   DefaultAdvisorAutoProxyCreator에 의해 빈이 프록시로 바꿔치기된 경우,
-   getBean() 메소드로 가져온 오브젝트는 JDK의 Proxy 타입
-   JDK 다이내믹 프록시 방식으로 만들어지는 프록시는 Proxy 클래스의 서브클래스이므로

### 포인트컷 표현식을 이용한 포인트컷

<aside> 🤓 스프링은 간단하고 효과적이인 포인트컷 표현식을 제공

</aside>

**포인트컷 표현식**

-   AspectJExpressionPointcut 클래스를 사용 → 포인트컷 표현식을 지원하는 포인트컷 적용
-   AspectJExpressionPointcut : 클래스와 메소드의 선정 알고리즘을 포인트컷 표현식으로 한 번에 지정
-   AspectJ : 스프링에서 쓰는 포인트컷 표현식

**포인트컷 표현식 문법**

-   AspectJ 포인트컷 표현식 : 포인트컷 지시자를 써서 작성
-   execution() 지시자를 쓴 포인트컷 표현식의 문법구조
    -   execution 지시자 - 메소드의 풀 시그니처를 문자열로 비교하는 개념이라고 생각
    -   execution([접근제한자 패턴] 타입패턴 [타입패턴.]이름패턴 (타입패턴 | “..”, …) [throws 예외 패턴])
    -   [] 괄호는 옵션이라 생략 가능, |는 OR 조건, 접근제한자 패턴은 생략 가능
    -   타입 패턴 : 리턴 값의 타입을 나타내는 패턴 포인트컷의 표현식에서  **필수항목**  (*는 모든 타입 선택)
    -   [타입패턴.] : 패키지와 타입 이름을 포함한 클래스의 타입 패턴으로 생략가능 (생략=모든 타입 허용
    -   이름 패턴 : 메소드 이름 패턴.  **필수항목**  (*는 모든 메소드 선택)
    -   (타입패턴 | “..”, …) : 메소드 파라미터의 타입 패턴
    -   [throws 예외 패턴] : 예외 이름에 대한 타입 패턴으로 생략가능.

**포인트컷 표현식을 이용하는 포인트컷 적용**

-   bean() : 스프링에서 사용될 때 빈의 이름으로 비교
    -   bean(*Service) ⇒ 아이디가 Service로 끝나는 모든 빈을 선택
-   @annotation : 특정 애노테이션이 타입, 메소드, 파라미터에 적용되어 있는 것을 보고 메소드를 선정
-   장점 : 짧은 문자열에 로직이 담기므로 클래스나 코드 추가 필요없음 → 코드 및 설정 단순화
-   단점 : 문자열로 된 표현식 → 런타임 시점까지 문법의 검증/기능 확인 불가

**타입 패턴과 클래스 이름 패턴**

-   포인트컷 표현식의 클래스 이름에 적용되는 패턴은 타입 패턴 (클래스 이름 패턴 NO)
    -   Targetlnterface 인터페이스를 표현식에 썼을때,
        -   Target 클래스의 오브젝트가 포인트컷에 의해 선정
        -   Target은 Targetlnterface를 구현했으므로 Target 클래스의 오브젝트는 Targetlnterface타입
-   포인트컷 표현식의 타입 패턴 항목에 인터페이스 이름 명시 → 해당 인터페이스를 구현한 빈 전부 선정

### AOP란 무엇인가?

**트랜잭션 서비스 추상화**

-   트랜잭션 적용이라는 추상적 작업 내용은 유지하고 구체적인 구현 방법을 바꿀 수 있게 추상화
-   비즈니스 로직 코드는 트랜잭션을 어떻게 처리해야 한다는 구체적인 방법과 서버환경에 독립적

**프록시와 데코레이터 패턴**

-   트랜잭션 처리코드는 일종의 데코레이터에 담기며 클라이언트와 비즈니스 로직을 담은 타깃 클래스 사이 존재
-   클라이언트가 대리자인 프록시 역할을 하는 트랜잭션 데코레이터를 거쳐 타깃에 접근
-   비즈니스 로직 코드는 트랜잭션과 같은 성격이 다른 코드로부터 자유로워짐

**다이내믹 프록시**

-   비즈니스 로직 인터페이스의 모든 메소드마다 트랜잭션 기능을 부여하는 코드를 넣어 짐이 됨
-   프록시 클래스 없이도 프록시 오브젝트를 런타임 시에 만들어주는 JDK 다이내믹 프록시 기술 이용

**프록시 팩토리 빈**

![https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.4.png](https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.4.png)

![https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.4.1.png](https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.4.1.png)

-   스프링의 프록시 팩토리 빈으로 다이내믹 프록시 생성 방법에 DI를 도입
-   템플릿/콜백 패턴을 활용하는 스프링의 프록시 팩토리 빈 덕분에 어드바이스와 포인트컷은 프록시에서 분리

**자동 프록시 생성 방법과 포인트컷**

-   트랜잭션 적용 대상이 되는 빈마다 일일이 프록시 팩토리 빈을 설정해줘야 한다는 부담 해결 🔽
-   스프링 컨테이너의 빈 생성 후처리 기법을 활용해 컨테이너 초기화 시점에서 자동으로 프록시 생성
-   부가기능의 위치에 대한 정보를 포인트컷이라는 독립적인 정보로 완벽히 분리

**부가기능의 모듈화**

-   관심사가 같은 코드를 분리해 모아두는것은 기본
-   트랜잭션 같은 부가기능은 핵심기능과 같은 방식으로는 모듈화 어려움
-   트랜잭션 부가기능은 트랜잭션 기능을 추가할 다른 대상이 있어야 유의미하므로 그 코드안에 있거나 연결될것
-   부가 기능의 모듈화 ← 데코레이터 패턴, 다이내믹 프록시, 오브젝트 생성후처리, 자동 프록시 생성, 포인트컷

**AOP : 애스펙트 지향 프로그래밍**

![https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.5.4.png](https://gunju-ko.github.io//assets/img/posts/toby-spring/aop/AOP-3.5.4.png)

-   애스펙트 : 자체적으로 애플리케이션의 핵심기능을 담진 않지만이를 구성하는 중요한 한 가지 요소
-   🔼 어드바이스(부가될 기능 정의) + 포인트컷(어드바이스의 적용 위치 결정)
-   왼쪽 : 애스펙트로 부가기능을 분리하기 전 상태
    -   핵심기능은 모듈화되어있지만 부가기능이 핵심기능의 모듈에 침투해서 설계 및 코드가 모두 지저분
-   오른쪽 : 핵심기능 코드 사이에 침투한 부가기능을 독립적 모듈인 애스펙트로 구분
    -   설계와 개발에서 다른 특성을 가진 애스펙트들을 독립적인 관점으로 작성
-   AOP : 애스팩트 지향 프로그래밍 애플리케이션의 핵심적인 기능에서 부가적인 기능을 분리 → 애스팩트 모듈
-   AOP는 OOP를 돕는 보조적인 기술이지 OOP의 완전 대체 개념은 아님

### AOP 적용 기술

**프록시를 이용한 AOP**

-   프록시로 만들어서 DI로 연결된 빈 사이 적용해 부가기능을 제공해주도록
-   프록시 방식을 썼으므로 메소드 호출 과정에 참여해 부가기능을 제공
-   부가기능 모듈을 여러 타깃 오브젝트의 메소드에 동적으로 적용하기 위해 가장 중요한게 프록시
-   따라서 스프링 AOP는 프록시 방식의 AOP라고도 함

**바이트코드 생성과 조작을 통한 AOP**

-   AspectJ는 스프링처럼 다이내믹 프록시 방식 쓰지 않음
-   프록시처럼 간접적인게 아니라, 타깃 오브젝트를 직접 고쳐서 부가 기능 넣는 방법 사용
-   컴파일된 타깃의 클래스 파일 자체를 수정 / 클래스가 JVM에 로딩되는 시점의 바이트코드를 조작
-   장점
    -   자동 프록시 생성 방식 없이도 AOP를 적용 가능 → 이보다 더 강력하고 유연한 AOP 가능
    -   프록시를 이용한 AOP는 부가기능을 부여할 대상은 클라이언트가 호출할 때 사용하는 메소드로 제한
    -   바이트 코드 조작은 AOP를 적용하면 다양한 작업에 부가 기능을 부여 가능

## 트랜잭션 속성

<aside> 🤓 트랜잭션 매니저에게 트랜잭션 가져오게 하는것과 커밋과 롤백중 하나 호출로 트랜잭션 경계 설정됨

</aside>

```java
@Overridepublic Object invoke(MethodInvocation invocation) throws Throwable {
    // 트랜잭션 시작TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
    try {
        Object result = invocation.proceed();
        this.transactionManager.commit(status);
        return result;
    } catch (Exception e) {
        this.transactionManager.rollback(status);
        throw e;
    }
}

```

### 트랜잭션 정의

-   TransactionDefinition 인터페이스 - 트랜잭션의 동작방식에 영향을 줄 수 있는 네 가지 속성 정의함
-   **트랜잭션 전파**
    -   트랜잭션의 경계에서 이미 진행 중인 트랜잭션이 있을 때나 없을 때 어떻게 동작할지를 결정하는 방식
        -   PROPAGATION_REQUIRED : 진행 중인 트랜잭션이 없으면 새로 시작하고, 있다면 거기 참여
        -   PROPAGATION_REQUIRES_NEW : 항상 새로운 트랜잭션을 시작
        -   PROPAGATION_NOT_SUPPORTED : 트랜잭션 없이 동작하도록 만듦. 진행중인 트랜잭션 무시
        -   getTransaction( ) 메소드는 항상 트랜잭션을 새로 시작하는게 아님
-   **격리수준**
    -   모든 DB 트랜잭션은 격리 수준을 필수로 가져야 함
    -   가능한 한 많은 트랜잭션을 동시에 진행시키면서 문제가 발생하지 않게 하는 제어가 필요
-   **제한시간**
    -   트랜잭션을 수행하는 제한시간을 설정
    -   기본 설정은 제한시간이 없음
    -   PROPAGATION_REQUIRED, PROPAGATION_REQUIRES_NEW와 같이 쓰일것
-   **읽기전용**
    -   읽기 전용이면 트랜잭션 내에서 데이터를 조작하는 시도 방지
    -   데이터 액세스 기술에 따라서 성능이 향상될 가능성 존재

### 트랜잭션 인터셉터와 트랜잭션 속성

**TransactionInterceptor**

-   스프링 제공 클래스, 편리하게 트랜잭션 경계설정 어드바이스로 사용하도록 만들어짐.
-   PlatformTransactionManager와 Properties 타입의 두 가지 속성
    -   Properties : 트랜잭션 속성을 정의
    -   TransactionAttribute 인터페이스로 정의되며 속성은 위의 네 기본 항목 + rollbackOn( ) 메소드
    -   rollbackOn() : 예외 발생시 롤백의 수행 여부 결정
-   스프링이 제공하는 TransactionInterceptor에는 기본적으로 두 가지 종류의 예외 처리 방식
    -   런타임 예외 : 트랜잭션은 롤백
    -   체크 예외 : 예외상황이 아닌 일종의 비즈니스 로직에 따른, 의미가 있는 리턴 방식의 한 가지 ⇒ 커밋
-   TransactionAttribute는 rollbackOn( ) 속성으로 기본원칙과 다른 예외처리 수행

**메소드 이름 패턴을 이용한 트랜잭션 속성 지정**

-   TransactionInterceptor의 Properties 타입 속성은 메소드 패턴,트랜잭션 속성을 키-값으로 갖는 컬렉션
-   트랜잭션 속성을 문자열로도 정의 가능
    -   PROPAGATION_NAME, ISOLATION_NAME, readOnly, timeout_NNNN, -/+Exception1/2
    -   트랜잭션 전파 항목만 필수 나머지는 생략 가능, 순서 상관 없음
    -   Exception1 : 체크 예외 중 롤백 대상으로 추가할 것 한 개 이상 등록 가능
    -   +Exception1 : 런타임 예외지만 롤백시키지 않을 예외들로 한 개 이상 등록 가능
-   메소드 이름이 여러 패턴과 일치하는 경우, 가장 정확히 일치하는 것 적용
-   TransactionInterceptor로 트랜잭션 어드바이스 정의 → 메소드 이름 패턴에 따라 다른 트랜잭션 속성 적용

**tx 네임스페이스를 이용한 설정 방법**

-   TransactionInterceptor 타입의 어드바이스 빈과 TransactionAttribute 타입의 속성 정보도 정의 가능
-   트랜잭션 속성이 개별 어트리뷰트로 지정되므로 설정 내용을 읽기가 더 쉽고 작성에 용이함

### 포인트 컷과 트랜잭션 속성의 적용 전략

**트랜잭션 포인트컷 표현식은 타입 패턴이나 빈 이름을 이용한다**

-   트랜잭션용 포인트컷 표현식에는 메소드나 파라미터, 예외에 대한 패턴을 정의하지 말 것
-   트랜잭션의 경계로 삼을 클래스들이 선정됐다면, 패키지를 통으로 선택하거나 일정한 패턴을 찾아서 표현식
-   가능하면 클래스보다는 인터페이스 타입을 기준으로 타입 패턴 적용할 것
-   스프링의 빈 이름을 이용하는 bean() 표현식 사용도 권장

**공통된 메소드 이름 규칙을 통해 최소한의 트랜잭션 어드바이스와 속성을 정의한다**

-   트랜잭션 적용 대상 클래스의 메소드는 일정한 명명 규칙을 따를 것
-   기준인 몇 가지 트랜잭션 속성을 정의후 그에 따라 적절한 메소드 명명 규칙을 만들면,
-   하나의 어드바이스로 애플리케이션의 모든 서비스 빈에 트랜잭션 속성을 지정가능

**프록시 방식 AOP는 같은 타킷 오브젝트 내의 메소드를 호출할 때는 적용되지 않는다**

-   프록시 방식의 AOP에선 프록시를 통한 부가기능의 적용은 클라이언트로부터 호출이 일어날 때만 가능
-   자신의 메소드를 호출할 땐 프록시를 통한 부가기능의 적용은 일어나지 않음
-   타깃 안에서의 호출에는 프록시가 적용되지 않는 문제 해결법
    -   스프링 API로 프록시 오브젝트에 대한 레퍼런스를 가져온 뒤에 같은 오브젝트의 메소드 호출 (강제적)
    -   AspectJ와 같은 타깃의 바이트코드를 직접 조작하는 방식의 AOP 기술 사용

**트랜잭션 속성 적용**

-   트랜잭션 경계설정의 일원화
    -   일반적으로 특정 계층의 경계를 트랜잭션 경계와 일치시킬 것
    -   비즈니스 로직을 담고 있는 서비스 계층 오브젝트의 메소드가 가장 적절한 대상
    -   가능하면 다른 모듈의 DAO에 접근할 때는 서비스 계층을 거치게 할 것
    -   서비스 계층에서 다른 모듈의 DAO를 직접 쓸때 신중할 것 (다른 모듈의 서비스 계층을 통해 접근 권장)

## 어노테이션 트랜잭션 속성과 포인트 컷

### 트랜잭션 애노테이션

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited@Documentedpublic @interface Transactional {

    @AliasFor("transactionManager")
	String value() default "";
	@AliasFor("value")
	String transactionManager() default "";
	Propagation propagation() default Propagation.REQUIRED;
	Isolation isolation() default Isolation.DEFAULT;
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;
	boolean readOnly() default false;
	Class<? extends Throwable>[] rollbackFor() default {};
	String[] rollbackForClassName() default {};
	Class<? extends Throwable>[] noRollbackFor() default {};
	String[] noRollbackForClassName() default {};

}

```

-   **@Transactional**
    -   트랜잭션 속성의 모든 항목을 엘리먼트로 지정 가능
    -   @Inherited : 상속을 통해서도 애노테이션 정보를 얻을 수 있음
    -   트랜잭션 속성을 정의하며, 동시에 포인트컷의 자동등록에도 씀

**트랜잭션 속성을 이용하는 포인트컷**

![https://gunju-ko.github.io//assets/img/posts/toby-spring/transaction/트랜잭션-2.png](https://gunju-ko.github.io//assets/img/posts/toby-spring/transaction/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-2.png)

-   🔼 Transactional 애노태이션을 사용했을 때 어드바이저의 동작 방식
-   @Transactional 방식 사용시, 포인트컷과 트랜잭션 속성을 어노테이션 하나로 지정 가능
-   메소드 단위로 트랜잭션 속성을 세분화해서 지정 가능하므로 세밀한 트랜잭션 속성 제어 가능

**대체 정책**

```java
[1]
public interface Service { 
	[2]
	void method1(); 
	[3]
	void method2();
}

[4]
public class Servicelmpl implements Service {
	[5]
	public void method1() (
	[6]
	public void method2() {
}

```

-   메소드의 속성을 확인할 때 타깃 메소드, 타깃 클래스, 선언 메소드, 선언 타입(클래스, 인터메이스)순으로
-   @Transactional이 적용됐는지 차례로 확인하고, 가장 먼저 발견되는 속성 정보 사용
-   @Transactional을 사용하면, 대체 정책을 잘 활용해서 최소한의 어노테이션으로도 세밀한 제어 가능
-   [5], [6] : 스프링은 트랜잭션 기능이 부여될 위치인 타깃 오브젝트의 메소드부터 시작해서 @Transactional 애노테이션이 존재하는지 확인한다.
-   [5], [6]번이 @Transactional이 위치할 수 있는 첫후보
-   [4] : 메소드에서 @Transactional을 발견하지 못하면, 다음은 타깃 클래스를 확인
-   [2, 3] : 스프링은 메소드가 선언된 인터페이스로 넘어감, 인터페이스에서도 먼저 메소드확인
-   [1] : 인터페이스 타입 [1]의 위치에 어노테이션 있는지 확인
-   @Transactional도 타깃 클래스보다는 인터페이스에 두는것 권장.
-   인터페이스를 사용하는 프록시 방식의 AOP가 아닌 방식은 타깃 클래스에 @Transactional을 두는것 권장

## 트랜잭션 지원 테스트

### 선언적 트랜잭션과 트랜잭션 전파 속성

![https://gunju-ko.github.io//assets/img/posts/toby-spring/transaction/트랜잭션-3.png](https://gunju-ko.github.io//assets/img/posts/toby-spring/transaction/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-3.png)

-   add() 메소드에 REQUIRED 방식의 트랜잭션 전파 속성을 지정했을 때 트랜잭션의 경계를 보여줌
-   add() 메소드도 트랜잭션 경계를 스스로 설정할 수 있지만,다른 메소드에서 만들어진 트랜잭션의 경계에 포함
-   트랜잭션 부여 방법
    -   선언적 트랜잭션 : AOP로 코드 외부에서 트랜잭션의 기능을 부여하고, 속성을 지정
    -   프로그램에 의한 트랜잭션 : TransactionTemplate이나 개별 데이터 기술의 트랜잭션 API를 사용

### 트랜잭션 동기화와 테스트

**트랜잭션 매니저와 트랜잭션 동기화**

-   트랜잭션 추상화 기술
    -   트랜잭션 매니저
        -   PlatformTransactionManager 인터페이스를 구현
        -   구체적인 트랜잭션 기술과 무관하게 일관된 트랜잭션 제어 가능
    -   트랜잭션 동기화
        -   시작된 트랜잭션 정보를 저장소에 보관해뒀다가 DAO에서 공유
        -   트랜잭션 전파에 중요한 역할 - 진행중인 트랜잭션 확인하고 전파 속성에 따라 참여하도록 만듦

**트랜잭션 매니저를 이용한 테스트용 트랜잭션 제어**

-   테스트 메소드에서 만들어지는 세 개의 트랜잭션을 하나로 통합하는 방법
    -   모두 트랜잭션 전파 속성이 REQUIRED → 메소드들 호출전 트랜잭션이 시작되게 할 것.
    -   트랜잭션의 전파는 트랜잭션 매니저를 통해 트랜잭션 동기화 방식이 적용되기 때문에 가능하다 함
    -   즉 테스트에서 트랜잭션 매니저로 트랜잭션을 시작시키고 이를 동기화 할 것

**트랜잭션 동기화 검증**

-   트랜잭션 속성 중에서 읽기전용과 제한시간 등은 처음 트랜잭션이 시작시에만 적용, 나중엔 무시
-   TransientDataAccessResourceException : 읽기 전용 트랜잭션에서 쓰기 진행시 발생하는 예외
-   스프링의 트랜잭션 추상화가 제공하는 트랜잭션 동기화 기술과 트랜잭션 전파 속성 → 테스트도 트랙잭션 묶O
-   JdbcTemplate과 같이 스프링이 제공하는 데이터 액세스 추상화를 적용한 DAO에도 동일한 영향
-   JdbcTemplate은 트랜잭션이 있으면 거기 참여, 없으면 트랜잭션 없이 자동커밋 모드로 JDBC 작업을 수행

**롤백 테스트**

-   테스트 내의 모든 DB 작업을 한 트랜잭션 안에서 동작하도록 하고 끝나면 모두 롤백 → DB에 영향 안줌
-   앞에서 실행된 테스트에서 DB 데이터를 바꾸면 이후 테스트에 영향 줄 수 있으므로 조작한 데이터 모두 롤백

### 테스트를 위한 트랜잭션 애노테이션

**@Transactional**

-   테스트에도 @Transactional을 적용 가능 → 테스트 메소드에 트랜잭션 경계가 자동 설정됨
-   테스트에서 사용하는 @Transactional은 AOP를 위한 것은 아님.

**@Rollback**

-   테스트용 트랜잭션은 테스트가 끝나면 자동 롤백 (기본적으로 트랜잭션을 강제 롤백하도록 설정)
-   강제 롤백이 싫다면 @Rollback 어노테이션 사용 → @Rollback(false)라고 지정

**@TransactionConfiguration**

-   @Rollback 애노테이션은 메소드 레벨에만 적용 가능
-   테스트 클래스의 모든 테스트 메소드에 트랜잭션을 적용하면서 롤백하지 않으려면 클래스 레벨에 부여할 것
-   @TransactionConfiguration으로 롤백에 대한 공통 속성 지정

**Propagation.NEVER**

-   트랜잭션을 시작하지 않은 채로 테스트 진행

### 효과적인 DB 테스트

-   DB가 사용되는 통합 테스트를 별도의 클래스로 만들어둔다면 기본적으로 클래스 레벨에 @Transactional
-   DB가 사용되는 통합 테스트는 가능한 한 롤백 테스트 → 독립적이고 자동화 된 테스트
