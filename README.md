# java-performance
자바 성능 튜닝 이야기를 읽으면서, 내용을 정리.<br>
블로그에 정리하려다, 깃허브에 정리하는 쪽으로 결정.

---

### Story 01 : 디자인 패턴 꼭 써야 한다.

1.디자인 패턴
  - 디자인 패턴 : 코드를 유연하게,  재사용성 및 유지보수가 용이하게 작성할 수 있도록 도와주는 여러가지 코드의 패턴들을 의미.

2.J2EE(JavaEE)
  - J2EE 패턴 : Java 2 Enterprise Edition 을 의미하며, 1999년에 썬 마이크로 시스템즈에서 정의한 기업용 자바 프로그램 개발을 위한 플랫폼(환경)이다.(EJB, JSP/Servlet, JDBC, JNDI, JTA, Java Mail 등)
  - J2EE 라고 불렸으나, 1.4 이후. 그러니까 J2EE 1.4 에서 다음버전으로 올라가면서 JavaEE 5로 개칭이 변경이 되었다.
  - 결론적으로, 자바를 이용하여 기업용 애플리케이션을 만드는데 필요한 스펙 또는 기술들의 집합이다.
  - tomcat / wildfly 등 벤더사들이 위 스펙을 구현할 수 있으며 호환성을 검증는 기구의 검증이 통과하면 JavaEE 호환제품으로 출시.
  - JavaEE는 MVC 구조가 기본으로 깔려있다.

3.MVC 패턴
  - Model / View / Controller 의 약자이며, 모델 / 뷰 / 컨트롤러 역할을 각각 나누어 애플리케이션을 3가지로 나누어 개발하는 패턴이다. 나누어서 개발을 하니, 유지보수와 확장성이 좋으며 디자이너 또는 퍼블리셔와 개발자간 협업시 역할도 명확히 구분이 된다는 장점이 있으나, 조그만한 프로젝트에서 개발 시, 구조 잡고 나누느라 개발시간이 더 소요가 될 수도 있다.
  - MVC 모델1 : 모델1은 요청 흐름을 제어하는 컨트롤러가 특별히 존재하지 않음. 바로 JSP에서 필요한 Java Bean을 호출하여 DB에서 정보 조회 / 갱신 / 수정 업무를 한 후, 바로 클라이언트에 응답을 해주는 방식. (그래서, 모델1은 컨트롤러가 없기 때문에 MVC라고 하기 어렵다)
  - MVC 모델2 : 모델2는 정확히 MVC 패턴을 따른다. 컨트롤러 역할을 하는 서블릿으로 요청을 하여, 모델에서 필요한 요청을 처리 후, 응답을 다시 서블릿으로 전달 후, 클라이언트에 응답을 해준다.

4.J2EE 디자인 패턴(너무 많아서 몇개만 한다.)
  - Front Controller : 클라이언트 요청 시, 제일 앞단에 다양한 요청을 제어하는 컨트롤러를 두어 단일 진입을 하게 한다고 해서, Front Controller 패턴이라고 불리운다. 컨트롤러로 향하는 모든 요청의 entry point 다.(spring 에서 Dispatcher Servlet이 이런 역할을 하지요. Dispatcher Servlet에서 요청을 가로채면 Handler Mapping을 통해(URL), 어떤 컨트롤러에게 요청을 위임하면 좋을지 찾음.)

  - Transfer Object : Value Object(VO) 라고도 불리는 데이터를 전송하기 위한 객체에 대한 패턴이다.
  ```java
  import java.io.Serializable;

  public class MemberTO implements Serializable {
      private String memberName;
      private int memberAge;

      public String getMemberName() {
          return memberName;
      }

      public void setMemberName(String memberName) {
          this.memberName = memberName;
      }

      public int getMemberAge() {
          return memberAge;
      }

      public void setMemberAge(int memberAge) {
          this.memberAge = memberAge;
      }

      @Override
      public String toString() {
          return "MemberTO{" +
                  "memberName='" + memberName + '\'' +
                  ", memberAge=" + memberAge +
                  '}';
      }
  }

  ```
  어디서, 많이 봤을법한 코드이다. 객체에 여러 타입의 값을 전달하는 일을 수행.
  getter / setter를 만들지, 필드에 접근제어자를 private으로 할지에 대한 정답은 없지만. 성능상으로 볼 때, getter / setter를 만들지 않는 것이 더 빠르다. Serializable 를 왜 구혀했을까? 객체를 직렬화를 하면, 서버 사이의 데이터 전송이 가능해 지기 때문이다. 이 패턴은 하나의 객체에 여러 타입의 값을 담아오기 위해 사용하는 패턴이라는 것을 명심하자.(값을 캡슐화 하는거임. 그리고 대부분, JavaBeans 규칙을 따라서 Java Bean이라고도 한다.)

  - Data Access Object : DAO패턴이라고 불리우며, 해당 패턴을 통해 저수준의 데이터 엑세스 로직과 비즈니스 로직을 분리한다. 일반적으로, DAO 인터페이스를 두고 해당 인터페이스를 구현하여, 구현클래스를 통해 DB와의 엑세스를 주고 받습니다.

---

### Story 02 : 내가 만든 프로그램의 속도를 알고 싶다.

1.프로파일링 툴?
- 프로파일링툴이란, 분석 도구라고 생각하면 된다.(profiler : 심리 분석관 / profile : 개요, 윤곽)

2.APM?
- APM 툴을 의미한다.(Application Performance Monitoring) 애플리케이션 성능 분석 도구라고 생각하면 된다.
3.프로파일링 툴 vs APM 뭐가 다른가요?
- 프로파일링툴은 개발환경
- APM은 운영환경
- 프로파일링툴은 개발환경에서, 세부응답속도 및 객체 / 클래스 등의 소스레벨 분석을 위해 사용한다.
- APM은 운영환경에서 실시간 모니터링 및 문제점 진단과 서버에 사용자 수 / 리소스에 대한 모니터링을 위해 사용한다.

3.이러한 좋은 툴이 있는건 알겠는데? 간단한 방법 없을까?
- Java System 클래스를 이용하면 된다.
- System.currentTimeMillis() : 현재의 시간을 ms로 리턴(1/1000 초)
- 위 메서드를 활용하여, 시작시간과 종료시간을 구하고, 응답시간 = 종료시간 - 시작시간을 하면 된다.
- JDK 5 버전에서 nanoTime() 메서드가 추가되어 이를 이용해도된다.(ns로 리턴)

---

### Story 03 : 왜 자꾸 String을 쓰지 말라는거야.

```Java
        //자바 메모리 사용량 확인을 위한 예제 코드
        //totalMemory : 현재 Allocate 된 메모리 사용량을 리턴(할당된, 전체 메모리)
        //freeMemory : 현재 Allocate 된 메모리 중, 사용 가능한 메모리 양을 리턴(전체에서 사용되어진걸 제외한 메모리 용량)

        System.gc(); //JVM 메모리 초기화.(절대 명시적으로 사용하면 안된다.)

        long startTime = System.currentTimeMillis();

        String testString = "";
        for(int i = 1; i<=20000; i++){
            testString += "이 문자열을 200번 계속 붙이자!";
        }

        long endTime = System.currentTimeMillis();

        long afterMemory = Runtime.getRuntime().totalMemory() - Runtime.getRuntime().freeMemory();
        System.out.println("사용 메모리(MB) : " + afterMemory / (1024.0*1024.0));
        System.out.println("응답시간 : " + (endTime - startTime) / 1000.0);
```
결과 <br>
사용 메모리(MB) : 247.31979370117188 <br>
응답시간 : 7.405 <br>

계속 돌려보면 딱 저 결과로 나오지는 않지만.. 뭔가 상당히 느리고 낭비가 된다는 생각이 든다.

```Java
System.gc(); //JVM 메모리 초기화.(절대 명시적으로 사용하면 안된다.)

        long startTime = System.currentTimeMillis();

        StringBuilder testString = new StringBuilder();
        for(int i = 1; i<=20000; i++){
            testString.append("이 문자열을 200번 계속 붙이자!");
        }

        long endTime = System.currentTimeMillis();

        long afterMemory = Runtime.getRuntime().totalMemory() - Runtime.getRuntime().freeMemory();
        System.out.println("사용 메모리(MB) : " + afterMemory / (1024.0*1024.0));
        System.out.println("응답시간 : " + (endTime - startTime) / 1000.0);

```
위 코드를 StringBulider로 변경해서 실행을 해봤다. 놀랍다. <br>
결과 <br>
사용 메모리(MB) : 2.9069900512695312 <br>
응답시간 : 0.004 <br>

1.StringBuffer / StringBuilder 클래스
- JDK 5 기준으로, StringBuilder 클래스 추가.
- 버퍼와 빌더는 무엇이 다를까? StringBuffer는 ThreadSafe 설계되어 있어서, 여러 스레드가 하나의 StringBuffer 객체를 처리해도 전혀 문제가 되지 않으면, StringBuilder는 단일스레드에서의 안전성만을 보장한다.
- append() / insert() 의 차이는? append()는 기존 값의 끝 자리에 넘어온 값을 덧붙이는 작업을 해주며, insert()는 지정된 위치 이후에 넘어온 값을 덧붙이는 작업을 수행해준다. insert()는 지정한 위치까지 값이 할당되어 있지않으면 exception이 발생하니 주의하자.

```Java
StringBuilder testString = new StringBuilder();
        testString.append("전설의죽음" + 123123);
```
만약 append()를 이렇게 쓰지 않길 바란다. 효과가 없다..

2.왜? String은 느리고, StringBuffer / StringBuilder 는 빠른가요?
```Java
String addString = "abcd";
        String a = "";

        for(int i = 1; i<=3; i++){
            a += addString;
            System.out.println(a.hashCode());
        }

        System.out.println("-------------------");

        StringBuffer buffer = new StringBuffer();
        for(int i = 1; i<=3 ; i++){
            buffer.append("abcd");
            System.out.println(buffer.hashCode());
        }

```
결과 <br>
2987074 <br>
1259550596 <br>
-1743689274 <br>
------------------- <br>
1452126962 <br>
1452126962 <br>
1452126962 <br>

hashCode()를 이용하면, 주소값을 출력했다. String 클래스같은 경우 더해지면 새로운 주소와 더해진 결과값을 갖는 a 객체가 생성된다. 이런 작업이 반복 수행되면서, 메모리를 많이 사용하게 되고 응답속도에도 영향을 미친다.(이 과정에서 쓰레기 객체는 GC가 수행이 되는데. 이녀석은 호출하면 할 수록 CPU 사용도 많이하고 느림..ㅠㅠ)

반면에, StringBuffer / StringBuilder 는 객체를 새로 생성하지 않고, 기존 객체에서 크기를 증가시키면서 값을 증가시킨다.
그러면, String 클래스는 왜 새로객체 만듬? 이녀석 자체가 불변클래스라서 그렇다. 그래서 새로운 녀석을 만들어 버리는거다.(원래 녀석은 절대 변하지 않음.)

- String / StringBuffer / StringBuilder 사용 시, 각각의 특징을 파악하여 사용해야함.(String은 짧은 문자열 처리 또는 변하질 않을 문자열에 사용. StringBuffer는 스레드에 안전한 프로그램이 필요할 때 또는 안전한지 모를 경우 사용. StringBuilder는 스레드 안전성 여부와 관계 없는 프로그램을 개발 할 때 사용.)

---

### Story 04 : 어디에 담아야 하는지..

- Java에서 데이터를 담아둘 때, collection framework(List, Map 등) 와 배열을 많이 이용한다.
- 왜? 의미있는 데이터는 함께 묶어 관리하는게 이득이기 때문. 데이터가 많아지면 하나하나 변수선언하면서 할 것도 아니지 않는가.

1.배열(array)
- 배열은 데이터의 집합이다. 같은 데이터를 한번에 또는 하나의 이름으로 관리하기 위한 자료구조이다.

```Java
//배열 선언
        int a[] = new int[3];
        int[] b = new int[5];

        int c[] = {1,2,3,4,5};
        int[] d;
        d = new int[3];
        d[0] = 11;
```

- 배열은 정의와 동시에, offset을 변경할 수 없다.
- 배열의 index는 값에 대한, 유일무이한 식별자이다.
- 배열도 객체이기 때문에, Heap영역에 할당이 되며, 메모리는 연속적으로 잡힘.

2.collection framework
- 컬렉션 프레임워크는 자바에서 데이터의 집합을 관리하기 쉽게 제공해주는 표준 클래스들이다.
