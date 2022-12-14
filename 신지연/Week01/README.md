
### 관심사의 분리, 리팩토링

관심이 같은것 끼리는 하나의 객체, 또는 친한 객체안으로 모이게 하고 관심이 다른것은 떨어져서 서로 영향을 주지않도록 분리하는것, 관심사를 구분하고 분리해줘야 같은 관심에 효과적으로 집중할 수 있고 변경시에도 자기의 관심사에 집중하면 된다는 이점이 있다.

 

1. 중복된 코드 메소드로 추출하기

 

2. 상속을 통해 서브클래스로 분리하여 확장하기

    - 템플릿 메소드 패턴

        슈퍼클래스에 기본적인 로직의  흐름을 만들고 그 기능의 일부를 서브클래스에서 오버라이딩하여 필요에 맞게 구현해 사용하도록 하는 것

    - 팩토리 메소드 패턴

        서브클래스에서 구체적인 오브젝트 생성 방법을 결정하게 하는 것

 

3. 아예 독립적인 클래스로 분리

이 때 두 개의 클래스가 긴밀하게 연결되지 않도록 느슨한 연결고리를 만들어준다. 인터페이스는 자신을 구현한 클래스에 대한 구체적인 정보를 감추고 인터페이스로 추상화해놓은 통로를 통해서만 접근하기 떄문에 자신을 구현한 클래스는 알 필요가 없고 그 클래스가 변경되어도 그냥 인터페이스를 통해 기능을 사용하면 된다.



### 개방 폐쇄 원칙
클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다. 예로 인터페이스를 통해 제공되는 확장 포인트는 확장을 위해 개방되어 있고 인터페이스를 이용하는 클래스는 불필요한 변화가 일어나지 않도록 폐쇄되어 있다.

 

### 낮은 결합도, 높은 응집도
높은 응집도란 하나의 모듈, 클래스가 하나의 관심사에만 집중되어 있다는 것이다. 자신의 기능에 충실하도록 독립되어 있기 때문에 변경을 할때 관련된 자신의 관심사만 확인해 주면 된다. 낮은 결합도란 관심사가 다른 오브젝트나 모듈간에는 관계를 유지하는데 최소한의 간접적인 형태로 제공하고 최대한 서로 모르도록 만들어 주는 것이다. 결합도가 낮아지면 하나의 오브젝트에서 변경이 일어날때 다른 오브젝트에서도 변경이 일어나는 정도를 낮추는 것이므로 확장하기 편리하다.

 

### 전략패턴
자신의 기능 맥락에서 필요에 따라 변경이 필요한 부분을 인터페이스를 통해 외부로 분리시키고 이를 구현한 클래스를 필요에 따라 바꿔서 사용할 수 있게 하는 디자인 패턴이다.

 

### 제어의 역전/IoC
1. 오브젝트 팩토리

컴포넌트의 구조와 관계를 정의한 설계도 같은 역할을 한다. 오브젝트를 생성해주고 오브젝트 사이의 관계를 맺어주는 기능을 한다.

 

2. 제어의 역전

사용하는 쪽에서 모든 오브젝트가 능동적으로 자신의 사용할 클래스를 결정하고, 언제 어떻게 그 오브젝트를 만들지 제어하는것이 아니라 모든 제어 권한을 자신이 아닌 다른 대상에게 위임하여 수동적으로 제어를 받는것이다. 

 

3. 스프링 IoC

스프링 빈은 스프링 컨테이너가 생성과 관계설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트이다. 이러한 빈의 제어를 담당하는 Ioc오브젝트를 빈 팩토리, 애플리케이션 컨텍스트라고 한다. 애플리케이션컨텍스트에는 설정정보를 가져와 Ioc를 적용해서 관리할 모든 오브젝트에 대한 빈의 생성과, 관계설정 등의 제어작업을 한다.

 

❓애플리케이션 컨텍스트를 사용했을떄 얻는 장점

- 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없기 떄문에 오브젝트 팩토리가 많아져도 따로 관리할 필요 없이 일관된 방식으로 원하는 오브젝트를 가져올 수 있다.

- 오브젝트 생성, 관계설정 뿐만 아니라 오브젝트 생성 방식, 시점, 오브젝트 후처리 등 오브젝트를 효과적으로 사용할 수 있는부가적인 기능을 제공한다.

- 빈의 이름을 통해 찾는것 뿐만 아니라 타입, 애노테이션 설정 등으로 빈을 찾을 수 있다.

 

### 싱글톤 레지스트리
스프링은 주로 서버환경에서 사용되며 여러 시스템으로 부터 요청을 받아 여러 오브젝트들이 계층적으로 관여하여 요청을 처리한다. 이러한 오브젝트들을 요청마다 새롭게 만들면 서버가 감당하기 힘들기 때문에 싱글톤으로 만들어 여러 스레드에서 하나의 오브젝트를 공유하도록 하는것을 지지한다. 그러나 기존 자바의 싱글톤 패턴 구현 방식은 여러 다음과 같은 단점 이있다.

 

- private생성자를 갖고 있어 상속할 수 없다.

- 싱글톤을 만들어 지는 방식이 제한적이라 테스트에서 사용할 목을 만들기 어렵다.

- 서버환경에서는 싱글톤이 하나만 만들어지는것을 보장하지 못한다.

- 싱글톤의 static메소드를 이용하면 싱글톤에 쉽게 접근할 수 있어 전역상태가 되기 쉽다.

 

따라서 스프링은 싱글톤 형태의 오브젝트를 관리하는 싱글톤 레지스트리를 제공한다. 평범한 자바클래스를 싱글톤으로 활용하게 해준다. 싱글톤은 여러 스레드가 동시에 접근하여 사용하기 때문에 내부에 상태를 갖고 있지 않은 무상태방식으로 만들어야 하며 이때 파라미터나 로컬 변수, 리턴값들을 이용해 각 스레드마다 정보를 관리할 수 있다.

 

### 의존관계 주입/DI
A가 B한테 의존한다는 것은 B의 변화가 A에 영향을 끼진다는 것이다. 인터페이스를 사용함으로써 직접적으로 구현클래스와 관계를 맺는것이 아니라 인터페이스에 대해서만 관계를 맺으면서 구현클래스와의 관계를 느슨하게 만들었다. 설계시점에는 어떤 클래스와 관계를 맺는지 알 수 없고 런타임 시에 제 3자가 실제 사용할 오브젝트와 의존관계를 설정하여 의존관계를 주입해준다. 제 3자는이렇게 의존관계를 담당하고 IoC방식으로 오브젝트의 생성, 제공등의 작업을 수행한다. 이를 DI컨테이너라고 한다.

 

❓의존관계 주입 방법

1. 생성자를 통한 주입

  한번에 여러 개의 파라미터를 받을 수 있다. 그러나 무조건 한 번에 필요한 모든 파라미터를 다 받아야 하므로 파라미터의 개수가 늘어날 경우 실수할 수도 있다.

2. 수정자 메소드를 이용한 주입

  관례적으로 set--() 라는 이름을 사용하고 한번에 하나의 파라미터만 가질 수 있다. 파라미터로 받은 오브젝트를 인스턴트 변수에 저장한다.

3. 일반 메소드를 이요한 주입

  여러 파라미터를 가진 여러 개의 초기화 메소드를 만들 수 있어 실수할 확률이 줄어든다.

 

### XML을 이용한 설정
애플리케이션 컨텍스트는 XML 설정정보를 활용할 수 있다. 자바코드로 만들면 반복된 틀에 박힌 구조가 반복되고 수정 시 다시 컴파일해줘야 하는 불편함이 있다. XML은 단순 텍스트 파일로 다루기 쉽고 별도의 빌드 작업도 없으며 오브젝트의 관계정보가 바뀔때도 빠르게 변경이 가능하다. 
