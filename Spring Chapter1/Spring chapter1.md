# Spring_190520

> ## Chapter1_오브젝트와 의존관계

#### `JavaBean`

(기존) 비주얼 툴에서 조작 가능한 컴포넌트

(현재) 디폴트 생성자와 프로퍼티를 갖는 오브젝트(=일반적인 클래스)

#### `JDBC 구동 순서`

1. DB 연결을 위한 Connection 객체 호출

2. preparedStatement 변수 생성

3. SQL 담은 후 쿼리 실행

4. 조회의 경우, ResultSet으로 받아 DTO 객체에 각 정보 저장(excuteQuery)

5. 삽입/수정/삭제의 경우, excuteUpdate로 구문 실행

6. 사용한 Connection, pstmt, rs 리소스는 close() (일괄종료)

   ```java
   //전역변수
   static Connection conn = null;
   static PreparedStatement pstmt = null;
   static ResultSet rs = null;
   static String sql = "";
   
   //일괄 종료
   public static void close() {
       try {
           if(rs!=null&&!rs.isClosed()) {
               rs.close();
           }
       } catch (SQLException e) {
           e.printStackTrace();
       }try {
           if(pstmt!=null&&!pstmt.isClosed()) {
               pstmt.close();
           }
       } catch (SQLException e) {
           e.printStackTrace();
       }try {
           if(conn!=null&&!conn.isClosed()) {
               conn.close();
           }
       } catch (SQLException e) {
           e.printStackTrace();
       }
   }
   ```

   

7. 예외처리

#### `리팩토링 & 메소드 추출`

(리팩토링)  기능에 영향을 주지 않으면서 코드의 가독성, 유연성이 향상하는 작업

(메소드 추출) 공통 기능인 메소드를 따로 작성하여 중복을 최소화 하는 작업

#### `템플릿 메소드 패턴`

- 부모 클래스의 추상 및 오버라이딩 가능한 protected 메소드를 자식이 재구현

#### `개방 폐쇄 원칙`

- 클래스나 모듈을 확장에는 열려 있고, 변경에는 닫혀 있어야 한다는 원칙

- UserDAO의 경우,

  DB 연결에 있어서는 확장성을 / 기능 유지에 있어서는 변경에 닫혀 있으므로 적합

#### `객체 지향 SOLID 원칙`

- 단일 책임 원칙
- 개방 폐쇄 원칙
- 리스코프 치환 원칙
- 인터페이스 분리 원칙
- 의존관계 역전 원칙

[SOLID 원칙 상세 설명](http://www.nextree.co.kr/p6960/)

#### `전략 패턴`

- 자신의 기능 맥락에서, 

  필요에 따라 변경이 요구되는 알고리즘을 인터페이스로 외부 분리시키고,

  이를 구현한 구체적인 알고리즘 클래스를 필요에 따라 바꿔 사용하는 전략

- ex) 액션 인터페이스

#### `팩토리`

- 객체의 생성 방법을 결정하고, 그렇게 생성된 객체 반환
- ex) UserActionFactory

1. 서블릿에서 수행하고자 하는 작업과 관련된 명령 추출
2. 추출된 명령어를 파라미터로 가지는 팩토리 객체 생성
3. action 객체 반환
4. 반환된 action 객체 정보에 맞는 작업 수행

![1558353266941](C:\Users\leejiyeon\AppData\Roaming\Typora\typora-user-images\1558353266941.png)

#### `빈 팩토리 & 애플리케이션 컨텍스트`

- 빈의 생성, 관계 설정과 같은 제어 담당하는 IoC 오브젝트
- 빈 팩토리가 확장된 개념 = 애플리케이션 컨텍스트
- 애플리케이션 컨텍스트 = IoC 컨테이너

#### `애플리케이션 컨텍스트의 장점`

1. 클라이언트가 구체적인 팩토리 클래스 알 필요 X
2. 애플리케이션 컨텍스트는 종합 IoC 서비스 제공
3. 빈을 검색하는 다양한 방법 제공

#### `서버 애플리케이션 & 싱글톤`

- 매번 객체 생성 시 엄청난 부하가 예상되므로,

  각 서블릿을 통해 접속하는 여러 클라이언트가 스레드를 통해 하나의 오브젝트 공유

