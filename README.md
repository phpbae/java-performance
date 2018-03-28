# java-performance
자바 성능 튜닝 이야기를 읽으면서, 내용을 정리.<br>
블로그에 정리하려다, 깃허브에 정리하는 쪽으로 결정.


Story 01 : 디자인 패턴 꼭 써야 한다.

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
