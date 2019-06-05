# 4장. 예외

## 4.1 사라진 SQLException

- try ~ catch 문을 사용할때, catch문으로 예외를 잡고 아무것도 처리해주지 않는 코드는 절대 만들어서는 안되는 코드이다.

  ##### <u>초난감 예외처리 코드(예제 3개)</u>

  ```java
  try{
      ...
  }
  1.	
      catch(SQLException e){
      }
  2. 	
      catch(SQLException e){
      	System.out.println(e);
      }
  3.	
      catch(SQLException e){
  		e.printStackTrace();
  	}
  
  // 에러를 확인하기 위해 처리해준다고 하지만 다른 로그나 메시지에 금방 묻혀 버리면 무의미하다. 운영서버에 올라가면 더욱심각하다 2번, 3번은 예외를 처리한 것이 아니다.
  ```

- 예외처리를 하기위해 메소드 옆에 **throws Exception을 붙이는 것**도 위에 코드 보다는 낫지만 **매우 안좋은 예외처리 기법**이다.  **두가지 방식은 용납하지 않아야한다.**





![img](https://t1.daumcdn.net/cfile/tistory/21476F3E577E91080E)

- 위의 사진을 보면 **RuntimeException을 포함**하거나 **그의 자식들**이 언체크예외이다.

  **언체크예외는 catch문으로 잡거나 throws로 선언하지 않아도 된다.**  물론 명시적으로 잡거나 throw로 선언해줘도 상관은없다.

  또한 **런타임 예외**에는 NullPointerException 이나 IllegalArgumentException등이 있는데, 이러한 예외는 조건은 개발자의 부주의로 발생할 수 있다.

- 그외의 예외는 모두 **체크예외인데, IOException이나 SQLException이 그에 속한다.** 이러한 체크예외가 발생할 수 있는 메소드를 사용할 경우 **반드시 예외처리를 해줘야한다.*

  

  #### 예외복구

- 첫 번째 예외처리 방법은 예외상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것이다. 예를 들어 사용자가 요청한 파일을 읽으려고 시도했는데 해당 파일이 없다거나 다른문제가있어서 읽히지가 않아서 IOException이 발생 했다고 생각해보자. 이때는 **사용자에게 상황을 알려주고 다른 파일을 이용하도록 안내해서 예외상황을 해결할 수 있다.**

- 원격 DB 서버에 접속하다 실패해서 SQLException이 발생하는 경우에 재시도를 해볼 수 있다. **네트워크 접속이 원활하지 않아서 예외가 발생했다면 일정 시간 대기했다가 다시 접속을 시도해보는 방법을 사용해서 예외상황으로 부터 복구를 시도할 수 있다.** 물론 **정해진 횟수 만큼 재시도 해서 실패했다면 예외복구는 포기**해야한다. 

  

  ##### <u>재시도를 통해 예외를 복구하는 코드</u>

  ```java
  int maxretry = MAX_RETRY;
  while(maxretry -- > 0){
      try{
          ...			// 예외가 발생할 가능성이 있는시도
          return;		// 작업 성공
      }
      catch(SomeException e){
          // 로그출력. 정해진 시간만큼대기
      }finally{
          // 리소스 반납. 정리작업
      }
      
  }
  throw new RetryFailedException(); // 최대 재시도 횟수를 넘기면 직접 예외 발생
  ```



#### 	  예외처리 회피

- 두번째 방법은 예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것이다. **throws 문으로 선언해서 예외가 발생하면 알아서 던져지게 하거나 catch 문으로 일단 예외를 잡은 후에 로그를 남기고 다시 예외를 던지는 것**이다.

  

  ##### 예외처리 회피 코드(예제 2개)

  

  ```java
  1.
  	public void add()throws SQLException{
      	// JDBC API
  	}
  
  2.
      public void add()throws SQLException{
      	// JDBC API
  	}
  	catch(SQLException e){
          // 로그 출력
          throw e;
      }
  }
  ```

- 예외를 회피 할때에는 자신을 사용하는 쪽에서 예외를 다루는 게 최선의 방법이라는 분명한 확신이 있어야한다.



#### 	  예외전환

- 마지막으로 예외를 처리하는 방법은 예외 전환을 하는것이다. **예외 회피와 비슷하게 예외를 복구해서 정상적인 상태로는 만들 수 없기 때문에 예외를 메소드 밖으로 던지는 것**이다. **하지만 예외 회피와는 달리, 발생한 예외를 그대로 넘기는 게 아니라 적절한 예외로 전환해서 던진다는 특징**이 있다.

- 예외 전환은 두가지 목적이있다. **첫째는 내부에서  발생한 예외를 그대로 던지는 것이 그 예외상황에 대한 적절한 의미를 부여해주지 못하는 경우**에, 의미를 분명하게 해줄 수 있는 예외로 바꿔주기 위해서다. **API가 발생하는 기술적인 로우레벨을 상화에 적합한 의미를 가진 예외로 변경하는 것**이다.

- 예를 들어 **새로운 사용자를 등록하려고 시도했을 때 아이디가 같은 사용자가 있어서 DB 에러가 발생하면 JDBC API는 SQLException을 발생**시킨다. 이 경우 **DAO가 메소드를 그대로 밖으로 던져버리면, 서비스 계층에서는 왜 SQL Exception이 발생했는지 쉽게 알 방법이없다.** 

  그래서 **DAO가 SQLException을 분석후DuplicateUserIdException 같은 예외로 바꿔서 던져주는 게 좋다**. 의미가 분명한 예외가 던져지면 서비스 계층 오브젝트에는 적절한 복구 작업을 시도할 수가 있다.



##### 		<u>예외 전환 기능을 가진 DAO 메소드</u>

```java
public void add(User user) throws DuplicateUserIdException, 			SQLException {
    try{
        // JDBC를 이용해 user 정보를 DB에 추가하는 코드 또는
        // 그런 기능을 가진 다른 SQL Exception을 던지는 메소드를 호출하는 코드 
    }
    catch(SQLException e){
        // ErrorCode가 MySQL의 "Duplicate Entry(1062)"이면 예외 전환
        if(e.getEorrorCode() == MysqlErrorNumbers.ER_DUP_ENTRY)
            throw DuplicateUserIdException();
        else
            throw e; // 그 외의 경우는 SQLException 그대로 던짐
    }
}
```



## 4.2 예외전환

- 예외를 다른 것으로 바꿔서 던지는 예외 전환의 목적은 두 가지라고 설명했다. **하나는 앞에서 적용해본 것처럼 런타임 예외로 포장해서 굳이 필요하지 않은 catch/throws를 줄여주는 것**이고, 다른 하나는 **로우레벨의 예외를 좀 더 의미있고 추상화된 예외로 바꿔서 던져 주는 것**이다.



#### 	JDBC의 한계

- **JDBC는 자바를 이용해 DB에 접근하는 방법을 추상화된 API 형태로 정희해놓고 각DB업체가 JDBC표준을 따라 만들어진 드라이버를 제공**하게 해준다. 내부 구현은 DB마다 다르겠지만 JDBC의 Connection, Statement, ResultSEt 등의 표준 인터페이스를 통해 그 기능을 제공해주기 때문에 자바 개발자들은 표준화된 JDBC의 API에만 익숙해지면 DB의 종류와 무관하게 일관된 방법으로 프로그램을 개발할 수 있다.

- 하지만 JDBC에도 한계가 있다. 첫번째로 JDBC 코드에서 사용하는 SQL 이다 SQL은 어느정도 표준화된 언어이고 몇가지 표준 규약이 있긴하지만 **각 데이터베이스 마다의 문법을 폭넓게 변화시킨 비표준 SQL 문장이 있기 때문에 자바 개발자들은 각 데이터베이스를 사용할때마다 달라진 쿼리문을 접해야 하기때문에 문제가된다**.

- 두번째는 바로 SQLException이다. DB를 사용하다가 발생할 수 있는 예외의 원인은 다양하다. SQL의 문법 오류도 있고, DB커넥션을 가져오지 못했을 수도 있으며, 테이블이나 필드가 존재하지 않거나, 키가 중복되거나 다양한 제약조건을 위배하는 시도를 한 경우, 데드락에 걸렸거나 락을 얻지 못했을 경우 등 수백여 가지에 이른다.

  이러한 상황들이 각 DB 마다 종류와 원인이 제각각이라는 것이 두번째 문제이다.



#### 	DB 에러 코드 매핑을 통한 전환

- 각 DB 마다 같은 종류의 에러가 나도  서로 다른 에러코드를 보내게 된다.  에러코드를 일관되게 전달 받을 수 있따면 효과적인 대응이 가능하다 .

  스프링은 DataAccessException이라는 SQLException을 대첼 수 있는 런타임 예외를 정의하고 있을 뿐 아니라  **DataAccessException의 서브 클래스로 세분화된 예외 클래스들을 정의하고 있다**. SQL 문법 때문에 발생하는 에러라면 DB 커넥션을 가져오지 못했을때, 데이터 제약조건을 위배했거나 일관성을 지키지 않는 작업을 수행했을 때는, 중복키 문제가 발생했을때 등이 있는데, **이러한 예외 상황을 수십 가지 예외로 분류하고 이를 추상화해 정의한 다양한 예외 클래스를 제공한다.**

- JDBCTemplate을 이용한다면 JDBC에서 발생하는 DB 관련 예외는 거의 신경 쓰지 않아도 된다. 그런데 중복키 에러가 발생했을때 애플리케이션에서 직접 정의한 예외를 발생 시키고 싶을 수 있다.  

- 애플리 케이션 레벨의 체크 예외인 **DuplicateUserIdException**을 던지게 하고 싶다면 스프링의 **DuplicateKeyException** 예외를 전환해주는 코드를 DAO 안에 넣으면된다.

  

  ##### <u>중복 키 예외의 전환</u>

  ```java
  public void add() throws DuplicateUserIdException {
     								//└ 어플리케이션 레벨의 체크 예외
  	try{
          // jdbcTemplate을 이용해 User를 add 하는코드
      }
      catch(DuplicateKeyException e){
          // 로그를 남기는 등의 필요한 작업
          throw new DuplicateUserIdException(e)
              //예외를 전환할 때는 원인이 되는 예외를 중첩하는 것이 좋다.
      }
  }
  ```



#### 	기술에 독립적인 UserDao 만들기

- 인터페이스와 구현 클래스의 이름을 정하는 방법은 여러 가지가 있다. **인터페이스를 구분하기 위해 인터페이스 이름앞에는 I라는 접두어를 붙이는 방법**, **인터페이스 이름은 가장 단순하게 하고 구현클래스는 각각의 특징을 따르는 이름을 붙이는 경우**도 있다. 여기서는 후자의 방법을 사용해보자. 

  사용자 처리 DAO의 이름을 UserDao라 하고, JDBC를 이용해 구현한 클래스의 이름을 UserDaoJdbc라고 하자.

  **UserDao 인터페이스에는 기존 UserDao 클래스 에서 DAO의 기능을 사용하려는 클라이언트들이 필요한 것만 추출**해내면된다.

  

  ##### <u>UserDao 인터페이스</u>

  ```java
  public interface UserDao{
  	void add(User user);
      User get(String id);
      List<User> getAll();
      void deleteAll();
      int getCount;
  }
  ```

  

- **public 접근자를 가진 메소드이긴 하지만 UserDao의 setDataSource() 메소드는 인터페이스에 추가하면 안 된다는 사실에 주의**하자. **setDataSource() 메소드는 UserDao의 구현 방법에 따라 변경될 수 있는 메소드**이고, **UserDao를 사용하는 클라이언트가 알고 있을 필요도 없다. 따라서 setDataSource()는 포함 시키지 않는다**.

  

  ```java
  // 기존의 UserDao 클래스는 다음과 같이 이름을 UserDaoJdbc로 변경하고 UserDao 인터페이스를 구현하도록 implements로 선언해줘야 한다.
  public class UserDaoJdbc implements UserDao{}
  ```

  

- 또 한 가지 변경 사항은 스프링 설정 파일의 userDao 빈 클래스 이름이다. 

  ```xml
  <bean id="userDao" class="springbook.dao.UserDaoJdbc">
      <property name="dataSource" ref="datasource"/>
  </bean>
  ```





## --내용 추가 예정--

#### 테스트보완

##### <u>UserDao 인터페이스와 구현의 분리</u>

#### DataAccessException 활용시 주의사항

