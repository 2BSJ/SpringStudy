# Spring Chapter2 테스트
-----
### 개요
* 스프링이 개발자에게 제공하는 가장 중요한 가치를 답한다면 객체지향과 테스트라고 대답할 것이다.
스프링의 핵심인 IoC와 DI는 오브젝트의 설계와 생성, 관계, 사용에 관한 기술이다.
스프링은 IoC/DI를 이용해 객체지향 프로그래밍 언어의 근본과 가치를 개발자가 손쉽게 적용하고 사용할 수 있게 도와주는 기술이다.
---
### 2.1.1 테스트의 유용성
* 테스트란 결국 내가 예상하고 의도했던 대로 코드가 정확히 동작하는지를 확인해서,
만든 코드를 확신할 수 있게 해주는 작업이다. 또한 테스트의 결과가 원하는 대로 나오지 않는 경우에는 코드나 설계에 결함이 있음을 알 수 있다. 이를 통해 코드의 결함을 제거해가는 작업, 일명 디버깅을 거치게 되고, 결국 최종적으로 테스트가 성공하면 모든 결함이 제거됐다는 확신을 얻을 수 있다.

##### 웹을 통한 DAO 테스트 방법의 문제점
* 웹 화면을 통해 값을 입력하고, 기능을 수행하고, 결과를 확인하는 방법은 가장 흔히 쓰이는 방법이지만, DAO에 대한 테스트로서는 단점이 너무 많다. DAO뿐만 아니라 서비스 클래스, 컨트롤러, JSP 뷰 등 모든 레이어의 기능을 다 만들고 나서야 테스트가 가능하다는 점이 가장 큰 문제다. 예를 들어  폼을 띄우고 값을 입력하고 등록 버튼을 눌렀을때 에러메세지가 떳을때 간단히 원인을 찾기가 굉장히 힘들다.
하나의 테스트에 여러 클래스와 코드가 많이 참여하고 있기 떄문이다.
다음은 효율적으로 테스트를 활용할 수 있는 방법에 대해 설명한다.

##### 작은 단위의 테스트
*  테스트하고자 하는 대상이 명확하다면 그 대상에만 집중해서 테스트하는 것이 바람직하다.
따라서 테스트는 가능하면 작은 단위로 쪼개서 집중해서 할 수 있어야 한다.
UserDaoTest는 한 가지 관심에 집중할 수 있게 작은 단위로 만들어진 테스트다.
  ```java
  public class UserDaoTest {

  public static void main(String[] args) throws SQLException {

      ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml")
      UserDao dao = context.getBean("userDao",UserDao.class);

      User user = new User();
      user.setId("user");
      user.setName("백기선");
      user.setPassword("married");

      dao.add(user);

      System.out.println(user.getId() + " 등록 성공");

      User user2 = userDao.get(user.getId());
      System.out.println(user2.getName());
      System.out.println(user2.getPassword());

      System.out.println(user2.getId() + " 조회 성공");
  }

}
  ```
  > 자바에서 가장 손쉽게 실행 가능한 main 메서드를 이용한다.
    테스트할 대상인 UserDao의 오브젝트를 가져와 메서드를 호출한다.
    테스트에 사용할 입력 값을 직접 코드에서 만들어 넣어준다.
    테스트의 결과를 콘솔에 출력해준다.
    각 단계의 작업이 에러 없이 끝나면 콘솔에 성공 메시지로 출력해준다.

  위의 코드는 mvc클래스나 서비스 오브젝트 등이 필요없고 서버에 배포할 필요는 물론 없다.
  간단히 얘기해서 UserDao 기능을 돌려보려고 만든 JSP나 서블릿에서 에러가 발생해서 그것을 찾으려고 시간을 낭비할 필요가 없다는 소리이다.
  이렇게 작은 단위의 코드에 대해 테스트를 수행한 것을`단위 테스트`라고 한다.
  일반적으로 단위는 작을수록 좋다.

##### 자동수행 테스트 코드
* 앞의 코드는 매번 화면에 폼을 띄어놓고 데이터를 입력해야하는 번거로움이 있다.
하지만 UserDaoTest는 자바 클래스의 메인 메소드를 실행하는 가장 간단한 방법만으로 테스트의 전 과정이 자동으로 진행된다. User 오브젝트를 만들어 적절한 값을 넣고, 이미 DB 연결 준비까지 다 되어 있는 UserDao 오브젝트를 스프링 컨테이너에서 가져와서 add 메서드를 호출하고, 그 키 값으로 get을 호출하는 것까지 자동으로 진행된다. 번거롭게 매번 입력할 필요도 없고, 매번 서버를 띄우고 브라우저를 열어야 하는 불편함도 없다.
IDE에서 단축키를 사용하면 메인메서드를 시작하는 데 0.5초면 충분하다.
하루에 100번을 돌려도 2분도 안걸린다.
이 처럼 테스트는 `자동으로 수행되도록` 만드는 것이 중요하다.
---
###2.1.3 UserDaoTest의 문제점

##### 수동 확인 작업의 번거로움
* UserDaoTest는 테스트를 수행하는 과정과 입력 데이터의 준비를 모두 자동으로 진행 하도록 만들어졌지만 여전히 사람의 눈으로 확인하는 과정이 필요하다.
수행은 자동으로 진행되지만 테스트의 결과를 확인하는 일은 사람의 책임이므로 완전히 자동화된 방법이 아니다.

##### 실행 작업의 번거로움
---
### UserDaoTest 개선

##### 2.2.1 테스트 검증의 자동화
* 첫번째 문제점인 테스트 결과의 검증 부분을 코드로 만들어야 한다.
**이 테스트를 통해 확인하고 싶은 사항** 은 add에 전달한 User오브젝트에 담긴 사용자 정보와 get을 통해 다시 db에서 가져온 User 오브젝트의 정보가 서로 정확히 일치하는가이다. 정확히 일치한다면 모든 정보가 빠짐없이 DB에 등록됐고, 이를 다시 DB에서 정확히 가져왔다는 사실을 알 수 있다.

* 리스트 2-2 수정 전 테스트 코드
  ```java
  System.out.println(user2.getName());
  System.out.println(user2.getPassword());

  System.out.println(user2.getId() + " 조회 성공");
  ```

* 리스트 2-3 수정 후 테스트 코드
  ```java
  if(!user.getName().equals(user2.getName()))
    System.out.println(user2.getName());
  else if(!user.getPassword().equals(user2.getPassword()))
    System.out.println(user2.getPassword());
  else
    System.out.println(user2.getId() + " 조회 성공");
  ```

### 2.2.2 테스트의 효율적인 수행과 결과 관리
* main 메서드로 만든 테스트는 필요한 기능은 모두 갖춘 셈이지만 편리하게 결과를 확인하려면 단순한 main 메서드로는 한계가 있다.
일정한 패턴을 가진 테스트를 만들 수 있고, 많은 테스트를 간단히 실행시킬 수 있으며, 테스트 결과를 종합해서 볼 수 있고, 테스트가 실패한 곳을 빠르게 찾을 수 있는 기능을 갖춘 테스트 지원 도구와 그에 맞는 테스트 작성 방법이 필요하다.
메인 메서드를 이요한 테스트 작성 방법만으로는 애플리케이션 규모가 커지고 테스트 개수가 많아지면 테스트를 수행하는 일이 점점 부담이된다.
자바에는 실용적인 테스트를 위한 도구가 여러가지 존재한다.
그중에서도 프로그래머를 위한 자바 테스틍 프레임워크라고 불리는`JUnit`은 자바 개발자라면 한 번쯤 들어봤건 ㅏ사용해봤을 유명한 테스트 지원 도구다.
`JUnit`은 이름 그대로 자바로 단위 테스트를 만들 때 유용하게 쓸 수 있다.

##### JUnit 테스트로 전환
* 프레임워크의 기본 동작원리인 `제어의 역전(IOC)`인 것처럼 프레임워크에서 동작하는 코드는 메인 메서드도 필요 없고 오브젝트를 만들어서 실행시키는 코드를 만들 필요도 없다.

##### 테스트 메소드 전환
* 기존에 만들었던 메인메서드 테스트는 그런 면에서 프레임워크에 적용하기엔 적합하지 않다.
테스트가 main() 메소드로 만들어졌다는 건 제어권을 직접 갖는다는 의미이기 떄문이다.
새로 만들 테스트 메소드는 JUnit 프레임워크가 요구하는 조건 두가지를 따라야 한다.
**첫째로 메서드가 public으로 선언돼야 하는 것이다**
**둘쨰로 메소드에 @Test라는 어노테이션을 붙여주는 것이다**

  ```java
    public class UserDaoTest{
      @Test
      public void addAndGet() throws SQLException{
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        ......
      }
    }
  ```
* main()대신에 일반 메소드로 만들고 적절한 이름을 붙여준다. public 액세스 권한을 주는 것을 잊으면 안된다.

##### 검증 코드 전환

  ```java
    if(!user.getName().equals(user2.getName())){....}
    ->assertThat(user2.getName(), is(user.getName()));
    //위의 assertThat이라는 스태틱 메서드를 이용해 바꿀 수 있다.
  ```
  * assertThat 메서드는 첫 번째 파라미터의 값을 뒤에 나오는 매처라고 불리는 조건으로 비교해서 일치하면 넘어가고 아니면 테스트가 실패하도록 만들어준다.
  is()는 매처의 일종으로 equals()로 비교해주는 기능을 가졌다.
  JUnit은 예외가 발생허가나 assertThat에서 실패하지 않으면 테스트가 성공했다고 인식한다.
  JUnit 프레임워크에서 실행될 수 있게 수정한 UserDaoTest는 **159P 참조**
    > 추가할 라이브러리 com.springsource.org.junit-4.7.0.jar

# Continue~~~~~~~~~~~~~~~
