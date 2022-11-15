# 서비스 추상화

🔗 링크 : https://pickle-fireplant-fa1.notion.site/5-4ca163fabbf24fd2bd71e24349ff2131

<aside> 🤓 DAO에 트랜잭션을 적용하면서 스프링의 기술 추상화와 사용방법등을 볼 것

</aside>

## 사용자 레벨 관리 기능 추가

> 기초 CRUD 작업만 가능한 UserDao에 정기적으로 사용자의 활동내역을 참고하여 레벨 올리는 기능 추가

**구현해야 할 비지니스 로직**

-   사용자 레벨은 BASIC, SILVER, GOLD이며 활동에 따라 한 단계씩 업그레이드
-   첫 가입시 BASIC 레벨, 가입후 50회 이상 로그인시 SILVER, SILVER에서 30회 이상 추천시 GOLD
-   사용자 레벨의 변경 작업은 일정 주기를 가지고 일괄적으로 진행되며, 작업 전에는 조건 충족해도 변경X

1.  필드 추가
    
    -   Level Enum - 내부로는 DB에 저장할 int 타입값이지만 겉으로는 오브젝트이므로 안전한 사용 가능
    -   User 필드 추가 - level 변수와 로그인 횟수, 추천수 추가
    -   UserDaoTest와 UserDaoJdbc 수정
2.  사용자 수정 기능 추가
    
    -   수정 정보가 담긴 User 오브젝트의 id로 사용자를 찾아 정보를 UPDATE로 변경하는 기능
    
    > **가장 실수가 많이 일어나는 곳 SQL 문장의 해결방안**
    > 
    > -   JdbcTemplate의 update()가 돌려주는 리턴값으로 확인
    > -   테스트를 보강하여 원하는 사용자만 정보 변경이 이뤄졌음을 확인
    
3.  UserService.upgradeLevels()
    
    -   사용자 관리 로직을 둘 위치
        -   UserDaoJdbc ❌ - DAO는 데이터를 어떻게 가져오고 조작할지를 다루는 곳. 비지니스 로직X
        -   UserService - 사용자 관리 비즈니스 로직 서비스를 제공할 클래스 추가
    -   UserService 추가
4.  UserService.add() - 처음 가입하는 사용자 BASIC는 비즈니스 로직을 담는 UserService가 담을것
    
5.  코드 개선
    
    > **😶 작성된 코드를 볼때 검토하며 할 질문들**
    > 
    > 1.  코드의 중복부분 유무
    > 2.  코드의 가독성이 좋은지 이해가 쉬운지
    > 3.  각 코드의 위치가 있어야 할 곳에 있는지
    > 4.  변경에 대해 유연하게 대응 가능한 코드인지
    
    -   upgradeLevels() - if/elseif/else 블록의 가독성이 나쁨
        -   성격이 다른 여러 가지 로직이 같이 있는것이 문제 → 단계에 따라 조건 비교 & 분리
    -   User Test
        -   User 도메인에도 메서드를 추가했으므로 적절하게 테스트 만들기
        -   레벨 업그레이드가 잘 되는가, 업그레이드 불가시 기대에 맞는 예외가 발생 하는가
    -   UserService Test
        -   private check메서드로 중복제거 & 하드코딩된 정수를 상수로 선언

## 트랜잭션 서비스 추상화

<aside> 🤔 업그레이드 진행중, 예외 발생시 이미 변경된 작업은 돌아가야 함 (전체 성공 또는 전체 실패)

</aside>

### 트랜잭션

-   더이상 나눌 수 없는 단위 작업 → 더이상 쪼개질 수 없는  **원자성**
-   논리적인 그룹으로 묶여있는 모든 작업 성공(Commit) 또는 모두 실패 Rollback) 중 하나의 상태를 보장
-   **복잡한 비지니스 로직에서 하나의 트랜잭션으로 처리할 경계를 설정하는게 중요**
-   하나의 SQL 명령은 DB가 트랜잭션을 보장, 여러 SQL을 쓴느 작업을 한 트랜잭션으로 취급해야 할때도 있음
-   글로벌 트랜잭션 방식은 DB 연결뿐 아니라 JMS 같은 메시징 작업도 하나의 트랜잭션으로 처리 가능

### 트랜잭션의 경계설정

-   트랜잭션의 시작은 고정, 끝은 두가지로 모든 작업을 무효화하는 롤백과, 모든 작업을 확정하는 커밋
-   트랜잭션의 경계 : 어플리케이션 내에서 트랜잭션이 시작되고 끝나는 위치
-   **JDBC 트랜잭션의 경계설정**
    -   트랜잭션의 시작과 종료는 Connection 객체로 이뤄지므로 트랜잭션 시작하려면 자동 커밋→false 변경
    -   JDBC의 자동 커밋 옵션을 false로 변경하면 커밋이나 롤백 호출 전까지 하나의 트랜잭션으로 묶임
    -   로컬 트랜잭션 : 위처럼 하나의 DB 커넥션 안에서 만들어지는 트랜잭션
-   **비지니스 로직내 트랜잭션의 경계설정**
    -   데이터 액세스 코드를 DAO로 분리했을때 문제는 메소드 호출시마다 새 트랜잭션이 생성되어 롤백 불가
    -   트랜잭션의 경계설정 작업을 UserService로 가져올 것 - 트랜잭션 시작과 종료 담당 최소한의 코드
    -   매번 새 Connection 오브젝트를 생성시, 새 트랜잭션 생성 → 메소드 호출시 인자로 connection 객체
    -   UserService의 메소드 사이에서도 같은 Connection을 사용해야 하니 인자로 넘겨줄 것

> **문제점 발생**
> 
> 1.  JdbcTemplate의 깔끔한 리소스(DB 커넥션등..) 처리 활용 불가
> 2.  DAO의 메소드와 비지니스 로직을 담은 UserServic의 메소드에 Connection 인자 추가해야함
> 3.  Connection인자가 UserDao 인터페이스 메소드에 추가되면 UserDao는 데이터액세스기술에 독립X
> 4.  DAO 메소드에 Connection 인자를 받게 하면 테스트 코드에도 영향이 있음

### 트랜잭션 동기화

<aside> 🤔  **위와 같은 문제점**이 있는데  **스프링**은 위의 문제를  **해결**하면서 트랜잭션의 경계를 정해줄 수 있음

</aside>

![image.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5f6e842d-2ced-47bd-9a3e-15b8b6a51843/image.png)

![image.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/67281700-1afc-40da-9f60-8afb392f9ff1/image.png)

-   Connection 파라미터 제거
    -   독립적인 트랜잭션 동기화(Connection 오브젝트를 특별 저장소에 보관후 저장된거 가져다 이용) 이용
    -   트랜잭션 동기화 저장소는 작업 스레드마다 독립적으로 Connection 오브젝트를 저장하고 관리
-   트랜잭션 동기화 적용
    -   스프링은 멀티스레드 환경에서도 안전한 트랜잭션 동기화 기능을 지원하는 간단한 유틸리티 메소드 제공
    -   트랜잭션 동기화 관리 클래스는 TransactionSynchronizationManager 클래스 이용
    -   동기화 작업 초기화와, 로직 처리 후 트랜잭션 동기화를 마치도록 요청하면 됨
    -   DataSourceUtils의 getConnection()은 Connection 객체 생성 및 트랜잭션 저장소에 바인딩해줌
-   트랜잭션 테스트 보완
    -   dataSource 빈을 가져와서 주입하는 코드 추가할 것 (트랜잭션 동기화에 쓸 DataSource DI해야해서)
-   JdbcTemplate과 트랜잭션 동기화
    -   JdbcTemplate은 미리 생성된 DB 커넥션/트랜잭션 없으면 직접 만들며 있다면 있는것을 가져와서 사용
    -   따라서 트랜잭션 적용 여부에 맞춰 UserDao 코드를 수정할 필요가 없어짐

**▶️ 트랜잭션의 처리 방식 ⇒ JDBC에 종속된 환경에서의 트랜잭션 처리 로직**

### 트랜잭션 서비스 추상화

<aside> 🤓 스프링에선 JDBC외 기술들에 대해서도 공통적으로 쓸수 있는 트랜잭션 처리 서비스 추상형태로 제공

</aside>

-   자바에서 제공하는 글로벌 트랜잭션 지원을 위한 API인 JTA
    
    : 로컬 트랜잭션은 하나의 DB Connection에 종속되므로
    
    하나의 트랜잭션 안에서 여러 DB를 만지는 작업은 글로벌 트랜잭션(분산 트랜잭션) 방식을 사용해야 함
    
-   JTA를 이용하면 여러 DB나 메세징 서버에 대한 작업을 하나의 트랜잭션으로 통합하는 분산 트랜잭션 가능
    

**트랜잭션 API의 의존관계 문제와 해결책**

> DAO 패턴을 써서 구현 데이터 액세스 기술을 유연하게 바꿔쓸수 있게 짰던 것이 UserService의 트랜잭션 경계 설정을 할 필요가 생기면서 다시 데이터 액세스 기술에 종속적이게 되버림

![https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2Fb17b6904-4451-4db4-a068-b31ca2e107da%2F95E93271-D85D-4D9D-BBE3-26CC6273BC7E.png](https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2Fb17b6904-4451-4db4-a068-b31ca2e107da%2F95E93271-D85D-4D9D-BBE3-26CC6273BC7E.png)

<aside> 🤔 UserService의 코드가 특정 트랜잭션 방법에 독립적일 수 있게 하는 방법?

</aside>

-   트랜잭션의 경계설정을 담당하는 코드는 일정한 패턴을 갖는 유사한 구조이므로 추상화를 고려해봄
-   JDBC, JPA, JTA 등 각 DB 접근 기술들은 트랜잭션 개념을 가지고 있으니 경계설정법에 공통점이 있을 것

**스프링의 트랜잭션 서비스 추상화**

-   스프링은 트랜잭션 기술의 공통점을 담은 트랜잭션 추상화 기술을 제공중
-   어플리케이션에서 각 기술의 트랜잭션 API를 직접 쓰지 않고도 일관되게 트랜잭션 경계 설정이 가능
-   PlatformTransactionManager : 스프링이 제공하는 트랜잭션 경계설정을 위한 추상 인터페이스
-   JDBC의 로컬 트랜잭션 이용시, DataSouceTransactionManager을 씀
-   스프링의 트랜잭션 추상화 기술은 트랜잭션 동기화를 사용하므로, 트랜잭션 동기화 저장소에 저장

![https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2F455569ff-fe9c-4608-8543-d41cf4d96933%2F90560972-F222-4B34-891D-E27440A186BB.png](https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2F455569ff-fe9c-4608-8543-d41cf4d96933%2F90560972-F222-4B34-891D-E27440A186BB.png)

**트랜잭션 기술 설정의 분리**

-   트랜잭션 추상화 API를 적용한 UserService 코드를 글로벌 트랜잭션으로 변경하려면
-   PlatformTransactionManager의 구현 클래스를 기술에 맞게 사용할 것.
-   하지만, UserService 내에서 어떤 트랜잭션 매니저 구현 클래스를 사용할지 스스로 알는 것은 DI 원칙 위배
-   자신이 사용할 구체적인 클래스를 컨테이너를 통해 외부에서 제공받게 하는 스프링 DI의 방식으로 변경
-   스프링이 제공하는 모든 PlatformTransactionManager의 구현 클래스는 싱글톤으로 사용 가능

> **스프링 빈 등록시 고려사항**

싱글톤으로 만들어져 여러 스레드에서 동시 사용이 괜찮은가? 상태가 있으며 멀티 스레드 환경에서 안전하지 않은 클래스를 빈으로 그냥 등록하면 매우 위험하기 때문

![https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2Fb8ccc66a-129e-46de-bb0a-8b6020d5cad6%2F52A33A93-7105-4744-ABA8-97FA1F20CFCE.png](https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2Fb8ccc66a-129e-46de-bb0a-8b6020d5cad6%2F52A33A93-7105-4744-ABA8-97FA1F20CFCE.png)

## 서비스 추상화와 단일 책임 원칙

<aside> 🤓  **기술과 서비스에 대한 추상화 기법 사용시 특정 기술환경에 종속되지 않는 포터블한 코드를 짤 수 있음**

</aside>

### 수직, 수평 계층구조와 의존관계

-   수평적 분리 : 애플리케이션 로직의 관심, 책임, 성격이 다르기 때문에 분리하는 것
-   수직적 분리 “ 애플리케이션의 비지니스 로직과 그 하위에서 동작하는 기술을 분리하는 것 (✔️트랜잭션)

![                   그림은 Application - Service Abstract - Tech Service 수직적, 수펑적 의존 관계를 잘 나타냄](https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2Fd3f96736-e46f-4631-af7a-14c51654965f%2Fimage.png)

```
               그림은 Application - Service Abstract - Tech Service 수직적, 수펑적 의존 관계를 잘 나타냄

```

UserDao, UserService 모두 어플리케이션의 로직을 담았으며 UserService는 순수하게 사용자 관리 업무

**비즈니스 로직**을 UserDao는 데이터를 어떻게 가져오고 등록할 것인가에 대한 **데이터 액세스 로직**을 담고 있음

UserDao <- -> UserService 는 인터페이스와 DI를 통해 연결됨으로써 낮은 결합도를 가짐

UserDao는 DB 연결을 생성하는 방법에 대해 독립적이므로 결합도가 낮음.

UserService와 트랜잭션 기술과의 관계 또한 인터페이스를 통한 추상화 계층을 사이에 두므로 결합도가 낮음

각각의 모듈들은 서로 최대한 영향을 주지 않고 자유롭게 확장될 수 있는 구조 (스프링의 DI가 큰 역할을 함 👍🏻)

**단일 책임 원칙**

-   단일 책임 원칙 : 하나의 모듈은 한 가지 책임을 가짐 즉 하나의 모듈이 바뀌는 이유는 하나여야 함
-   ex) UserService에 JDBC Connection 메소드를 직접 사용하는 트랜잭션 코드가 들어 있을 때, UserService는 어떻게 사용자 레벨을 관리할지, 트랜잭션을 관리할지 책임 2개였으므로 위배 하지만, 트랜잭션 서비스의 추상화하면서 PlatformTransactionManager인터페이스가 트랜잭션을 관리하도록 위임하면서 UserService는 비지니스적인 요소만 신경쓸 수 있게 됨
-   장점 : 변경이 필요할 때, 수정 대상이 명확해짐
-   단일 책임 원칙에 위배되는 코드 유형
    -   첫째는 같은 책임을 가지는 코드가 여러 모듈에 나누어져있을 경우. 이때 한 가지 이유로 변경이 발생하면 여러 모듈의 코드를 같이 수정해야 한다.
    -   두 번째는 하나의 모듈에 여러 책임을 가지는 코드가 섞여 있을 경우. 이때 한 가지 이유로 변경이 발생하면 모듈 내 코드를 수정하며 다른 역할의 코드에 영향을 줄 수 있음
-   단일 책임 원칙을 잘 지킨다면 ****여러 모듈들이 각자의 책임에 충실함으로 코드 이해 및 변경에 용이
-   **단일 책임원칙을 돕는 DI :**  DI를 통해 다른 성격의 모듈은 외부에서 제어하고 주입받아 사용

## 메일 서비스 추상화

**JavaMail을 이용한 메일 발송 기능**

-   레벨이 업그레이드 되는 고객에겐 안내 메일을 발송기능 추가
-   User에 Email 필드를 추가하고, UserService의 upgradeLevel()에 메일 발송 기능을 추가

### JavaMail 메일 발송

-   자바에서 메일을 발송할 때는 표준 기술인 JavaMail을 사용
    
-   upgradeLevel 메소드에 수정사항 추가
    
    ![https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2F0039063c-deba-4d5b-b254-ddec893a53a5%2Fimage.png](https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2F0039063c-deba-4d5b-b254-ddec893a53a5%2Fimage.png)
    
    ![https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2F455b78ff-1119-43bb-953a-f2fcc9c097c7%2Fimage.png](https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2F455b78ff-1119-43bb-953a-f2fcc9c097c7%2Fimage.png)
    

### JavaMail이 포함된 코드의 테스트

-   JavaMail을 테스트 할 때 고려사항
    -   메일 발송은 부하가 많이 가는 작업인데, 테스트마다 실제 메일이 발송되는 것이 옳은가?
    -   실제로 메일을 고객들에게 발송한다면 혼란을 겪을텐데 테스트 메일 주소를 두어야 할까?
    -   메일이 실제로 잘 동작하는지 어떻게 테스트 해야할까?
-   JavaMail은 이미 꽤 검증된 API이므로 JavaMail API가 요청을 올바르게 통신한다는 보장만 있으면 됨
-   테스트용 메일 서버 따로 두고, 메일 서버는 SMTP 프로토콜 요청이 JavaMail로 부터 잘 도착하는지만 확인

### 테스트를 위한 서비스 추상화

-   JavaMail은 구조상 확장이나 지원이 불가능하도록 만들어진 API
-   스프링이 제공하는 메일 서비스 추상화 인터페이스로 테스트 진행 & JavaMail로 구현된 소스 리팩토링

### 테스트와 서비스 추상화

> **서비스 추상화**  트랜잭션과 같이 기능은 유사하나 사용 방법이 다른 로우레벨의 다양한 기술에 대해 추상 인터페이스와 일관성 있는 접근 방법을 제공해주는 것.

-   JavaMail의 경우 테스트가 어려운 API
-   따라서 실제 메일을 전송하는 구현 클래스를 더미 전송 구현 클래스로 바꿔치기하여 테스트를 진행

![https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2Ff9886155-9813-40d3-82ae-687f9ce1e361%2Fimage.png](https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2Ff9886155-9813-40d3-82ae-687f9ce1e361%2Fimage.png)

### 테스트 대역

**의존 오브젝트의 변경을 통한 테스트 방법**

![UserDao/UserService 테스트시 운영/테스트를 구분하는 property나 자바 configure를 써서 위처럼 구성 가능](https://velog.velcdn.com/images%2Fdevsigner9920%2Fpost%2F5e4b5f5f-c474-4c1e-bb57-ca1d488790ec%2Fimage.png)

UserDao/UserService 테스트시 운영/테스트를 구분하는 property나 자바 configure를 써서 위처럼 구성 가능

**테스트 대역의 종류와 특징**

-   **Test Double**  : 테스트 환경을 만들어주기 위해, 테스트 대상이 되는 오브젝트의 기능에만 충실히 수행하며,  
    빠르고 자주 테스트를 실행하도록 쓰는 오브젝트들의 총칭
-   Test Stub : 테스트 대상 오브젝트의 의존객체로서 테스트 동안 코드가 정상 수행하도록 돕는 것
    -   단순한 호출만 하는 경우 뿐 아니라, 실행 후 결과를 반환받아 처리하는 경우도 있음
    -   메소드를 호출하면 강제예외를 발생시켜 예외 상황에서의 테스트 대상 오브젝트 처리방법 고려에 사용

**Mock Object 활용**

-   스텁처럼 테스트 오브젝트가 정상적으로 실행되게 도움
-   테스트 오브젝트와 자신 사이 일어나는 커뮤니케이션 저장해뒀다가 테스트 결과 검증시 활용하도록 도움
-   목 오브젝트를 사용한 테스트의 장점 : 단순 요청, 호출 후 반환되 ㄴ값에 대한 검증등 테스트 진행에 용이

> Written with [StackEdit](https://stackedit.io/).
