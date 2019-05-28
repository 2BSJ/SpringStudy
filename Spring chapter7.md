# Spring Chapter7-스프링 핵심 기술의 응용

작성자 : 

&#128204; 7장의 목차

    7.1 SQL과 DAO의 분리

    7.2 인터페이스의 분리와 자기참조 빈

    7.3 서비스 추상화 적용

    7.4 인터페이스 상속을 통한 안전한 기능확장

    7.5 DI를 이용해 다양한 구현 방법 적용하기

    7.6 스프링 3.1의 DI

    7.7 정리

&#10067; **개요**
  > 이전 장까지 스프링의 핵심 기술인 IoC/DI, 추상화, AOP에 대해 학습했다. 그리고 이번 장에서는 이러한 스프링의 제반 기술들을 응용하는 방법을 소개한다.
  누차 강조하지만, 스프링이 만들어진 이유, 스프링을 사용해야 하는 이유는 결국 OOP를 위함이다.
  따라서 모듈을 설계할 때는 반드시(라 불러도 좋을 만큼) 인터페이스를 기본으로 하는 DI를 적용하고, 특정 기술이나 특정 상황에 한정해 작성하지 않고 추상화를 적용한다.
  특히 이 장에서는 스프링에 기본적으로 내장되어 있는 여러가지 내장 기술들을 적용하는 방법을 제시한다.

#### 1. SQL을 코드에서 분리하기

* 먼저 SQL 구문을 코드에서 분리해 별도의 파일로 저장해 두는 상황을 가정해 보자.
  대충 생각하면 각 SQL 구문을 탭과 캐리지리턴으로 Key와 SQL로 구분하여 txt 파일로 저장한 다음 해당 텍스트를 스트림으로 읽어들인다음 배열이나 Map에 저장하는 방법이 있을 것이다. 실제로 ASP에서는 이러한 시도들이 있었고, SQL이 아니더라도 개발자마다 자신들의 개발 환경이나 기타 중요한 정보 등을 텍스트로 저장해 둔 다음 불러 들여 사용하면 된다.
  그런데 텍스트 파일은 관리와 식별이 용이하지 않다. 이런 경우에는 당연하게도 XML이 제격이다.
  XML을 불러 들이는 방법에는 여러가지가 있는데 책에서는 JAXB를 먼저 소개한다. (JAXB를 사용하여 XML을 불러 들이는 방법에 대해서는 책에 비교적 자세히 나와 있다.)
  여기서, 서버가 XML을 로드하는 시점이 언제인지를 생각해 볼 필요가 있다. 만약 각 메소드에서 SQL을 요청할 때 마다 XML을 로드한다면 자원의 낭비가 막심할 것이다. 코드 내에 if 명령을 이용해 내용이 있으면 불러 들이고, 없으면 로드하라는 식으로 기술할 수도 있지만 이런 경우 코드가 복잡해지고, 매번 XML을 로드하는 생성자가 호출됨에 따라 오류가 생길 여지가 높아진다. 따라서 XML의 로드는 별도로 처리될 필요가 있다. 그리고 이렇게 생성된 빈은 스프링이 초기화될 때 자동으로 로드하게 하면 된다.(앞선 6장에서 언급한 프록시 자동생성기와 같은 '후처리기'를 이용한 것이다.)
  스프링의 환경설정 문서에 다음과 같이 코드를 변경한다.
  ```java
  applicationContext.xml

  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
					http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
						http://www.springframework.org/schema/aop
						http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
						http://www.springframework.org/schema/context
						http://www.springframework.org/schema/context/spring-context-3.0.xsd
						http://www.springframework.org/schema/tx
						http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">
	<context:annotation-config /> // 이 태그를 통해 빈 후처리기가 동작하고, 이에 따라 새로운 애노테이션을 사용할 수 있게 된다.
    ...(중략)
  ```

#### 2. OXM 서비스 추상화

* 자바에는 JAXB 외에도 XML을 자바오브젝트와 연결하는 다양한 방법이 있다.
  또, XML 파일의 위치는 언제든 변경될 수 있기 때문에 이런 상황에 대비한 모듈의 설계가 필요하다.
  바로 추상화가 필요하다는 뜻이다.
  먼저 JAXB를 대체하는 다른 기술들을 손쉽게 적용할 수 있도록 만들기 위해서는 XML을 로드하는 부분을 따로 추출해 클래스로 만들고 다른 클래스에서 이를 사용할 때는 인터페이스를 통해 사용하도록 만든다. 그렇게 n개의 기술에 대해 n개의 클래스 또는 인터페이스로 분리하면 특정 기술에 종속되지 않는 유연한 코드를 작성할 수 있게 된다.
  장황하고 막연한 이야기처럼 보이지만, 이 역시 경험을 통해 직관으로 이해할 수 있는 내용이다.
  또 XML 파일의 위치를 자유롭게 지정하기 위해서는 Resource 추상화가 필요하다. 자바에서는 리소스에 대해 단일화된 접근 인터페이스를 제공하지 않지만, 스프링에서는 제한적이나마 이에 대한 인터페이스를 제공한다.
  적용 예시를 보자.
  ```java
  ...(중략)
  import javax.annotation.PostConstruct;
  import javax.xml.transform.Source;
  import javax.xml.transform.stream.StreamSource;
  import org.springframework.core.io.ClassPathResource;
  import org.springframework.core.io.Resource;    //스프링의 Resource 를 가져온다.
  import org.springframework.oxm.Unmarshaller;

  ...(중략)
  private class OxmSqlReader implements SqlReader {
	   private Unmarshaller unmarshaller;

	    private Resource sqlmap = new ClassPathResource("sqlmap.xml", UserDao.class);

	    public void setUnmarshaller(Unmarshaller unmarshaller) {
		       this.unmarshaller = unmarshaller;
	    }
	    public void setSqlmap(Resource sqlmap) {    // 자료 타입으로 Resource 를 쓰기만 하면 된다.
		           this.sqlmap = sqlmap;
	    }
	    public void read(SqlRegistry sqlRegistry) {
		         try {
		                 Source source = new StreamSource(sqlmap.getInputStream());
		                 Sqlmap sqlmap = (Sqlmap)this.unmarshaller.unmarshal(source);
		                 for(SqlType sql : sqlmap.getSql()) {
		                       sqlRegistry.registerSql(sql.getKey(), sql.getValue());
		                  }
		         } catch (IOException e) {
		              throw new IllegalArgumentException(this.sqlmap.getFilename() + "을 가져올  수 없습니다", e);
	               }
	                     }
                                      }
  ```

#### 3. 내장 데이터베이스 사용하기

* XML 파일을 불러 들이는 방법 외에도, 내장 데이터베이스를 사용하는 방법도 있다. SQL 구문 내용의 업데이트와 이에 대한 트랜잭  션 등의 기능을 기본적으로 지원한다는 점에서 내장 DB의 사용은 권장될 만 하다.
먼저 SQL, 또는 기타 주요 정보를 저장해 놓을 테이블의 생성 스크립트를 .sql 파일로 만들어 서버의 특정 위치에 저장한다.
또 해당 테이블에 들어갈 초기 정보 역시 insert 구문으로 저장한다.
JUnit 테스트 시에는 스프링이 제공하는 내장DB를 호출해 사용한다. 사용 후 내장DB를 닫아줘야 하는 경우에는 .close() 구문을 이용해 DB를 닫는다.

```java
applicationContext.xml


	<bean id="sqlRegistry" class="springbook.user.sqlservice.updatable.EmbeddedDbSqlRegistry">
		<property name="dataSource" ref="embeddedDatabase" />
	</bean>
	<jdbc:embedded-database id="embeddedDatabase" type="HSQL">
		<jdbc:script location="classpath:springbook/user/sqlservice/updatable/sqlRegistrySchema.sql"/>
	</jdbc:embedded-database>
```

```java
EmbeddedDbSqlRegistry.java


 public class EmbeddedDbSqlRegistry implements UpdatableSqlRegistry {
	SimpleJdbcTemplate jdbc;
	TransactionTemplate transactionTemplate;

	public void setDataSource(DataSource dataSource) {
		jdbc = new SimpleJdbcTemplate(dataSource);
		transactionTemplate = new TransactionTemplate(
								new DataSourceTransactionManager(dataSource));
		transactionTemplate.setIsolationLevel(TransactionTemplate.ISOLATION_READ_COMMITTED);
	}
	public void registerSql(String key, String sql) {
		jdbc.update("insert into sqlmap(key_, sql_) values(?,?)", key, sql);
	}
	public String findSql(String key) throws SqlNotFoundException {
		try {
			return jdbc.queryForObject("select sql_ from sqlmap where key_ = ?", String.class, key);
		}
		catch(EmptyResultDataAccessException e) {
			throw new SqlNotFoundException(key + "에 해당하는 SQL을 찾을 수 없습니다", e);
		}
	}
}

...(중략)
```

## 정리

1. 스프링의 DI 기술이나 서비스 추상화 기술을 응용하여 Dao에 들어있는 SQL을 외부의 리소스에서 가져오게끔 변경하면 편리하다.
2. XML로 만든 SQL 외부 리소스는 스프링의 OXM 추상화 기능을 활용하면 좋다.
3. SQL을 XML로 저장하는 대신, 스프링에서 지원하는 내장 DB를 이용해 SQL을 저장할 수도 있다.

## &#127908; 느낀점
* 7.6장에 XML의 빈 설정을 Java로 옮기는 부분이 개인적으로 가장 인상깊었다. 요새는 왠만해서는 더이상 XML에 빈을 일일히 정의하지 않는다. 스프링 부트가 나오면서 더더욱 안쓰게 된 것 같다. XML로 일일히 등록 했던 빈들을 Java Configuration 파일로 옮기고, 클래스들에 직접 빈설정(`@Component`, `@Repository`, `@Service`)을 붙임으로서 Java 빈 설정 까지도 거의 없애 나간다. 스프링 3.0에 `@MVC`를 처음 접했을 때 서비스 클래스에는 `@Service`, Data Access Object에는 `@Repository`, 기타 빈으로 등록해야 하는 클래스는 `@Component`로 되어있었는데, 새삼 그 때 처음의 기억이 떠올랐다. 점점 더 기술은 단순화 추상화 되면서 더 편리해 지는 것 같지만 또 한편으로는 새로운 기술들과 알아야 하는 것들이 계속 나오니 참 아이러니하다. 아무튼, XML 빈 설정 파일도 제거했다. 그리고는 조금 더 실제 서비스 개발 할 때 직면할 수 있는 부분들에 대해 도움이 되는 설명들이 나온다. `@Import`, `@Profile`, `@ActiveProfiles`, `@PropertySource`, `@Value` 등 단편적으로 필요할 때마다 구글링해 봤던 것들을 정리할 수 있다.
