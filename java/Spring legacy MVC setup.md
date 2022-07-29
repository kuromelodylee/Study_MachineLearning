## 스프링 MVC

### 스프링을 왜사용하는가?

- 궁극적으로 각역할을 가지고 있는 객체들 (서버접속, 쿼리문, 서블릿등)을 한번 설정하면 쉽게 java 패키지에서 생성하지 않아도 사용할 수 있다.
- 그밖에
  - 어노테이션 설정으로 쉽게 객체를 생성할 수 있다.
- 

1. ### Maven 프로젝트 파일 기준 Spring  셋업 (동작원리의 이해)

   1. project 생성

      - dynamic web project (web.xml 추가)  - configure - convert to maven

   2. pom.xml 수정

      - 역할: maven으로 빌드된 프로젝트에서 maven dependency에 spring jar들을 설치해서 spring을 사용할수 있게한다.

      - 설치: spring-web 및 spring-webmvc를 dependency로추가 build 뒤에 추가 저장하면  maven Depedencies 라이브러리에 spring  관련 jar들이 생성된다.

        ```xml
        </build> 
            <dependencies>
        	 <!-- https://mvnrepository.com/artifact/org.springframework/spring-web -->
        	<dependency>
        	    <groupId>org.springframework</groupId>
        	    <artifactId>spring-web</artifactId>
        	    <version>5.3.18</version>
        	</dependency>
        	<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
        	<dependency>
        	    <groupId>org.springframework</groupId>
        	    <artifactId>spring-webmvc</artifactId>
        	    <version>5.3.18</version>
        	</dependency>
        	
          </dependencies>
        ```

        
   
   3. web.xml 수정
   
      - 역할: 프로젝트가 실행되면 web.xml내 요소들을 먼저 읽어 spring을 사용할 준비를 한다.

      - 설치: listener, context-param, servlet, mapping
        - listener: ContextLoaderListener) contextConfiguration을 바탕으로 스프링 객체 루트를 생성함
        
          ```xml
          <listener>
          	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
          </listener>
          ```
        
        - context-param: contextConfiguration) 스프링의 객체(bean 콩) 생성
        
          ```xml
          <context-param>
          		<param-name>contextConfigLocation</param-name>
          		<param-value>/WEB-INF/spring/root-context.xml</param-value>
          </context-param>
          ```
        
        - servlet: DispatcherServlet) 클라이언트 호출을 받았을때 분배해주는 역할
        
          ```xml
          <servlet>
          		<servlet-name>appServlet</servlet-name>
          		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          	</servlet>
          ```
        
        - mapping: servlet mapping) DispatcherServlet이 어떤 요청을 받았을때 분배를 작동하는지 정의
        
          ```xml
          <servlet-mapping>
          		<servlet-name>appServlet</servlet-name>
          		<url-pattern>*.do</url-pattern> <!-- 위에 디스패쳐서블릿이 받을수 있게 클라이언트로 들어오는 url을 지정한다. -->
          	</servlet-mapping>
          ```
        
          
   
   4. contextConfiguration (applicationContext.xml 등)
   
      - 역할: 스프링의 객체를 생성하고 AOP등을 설정해준다. (자세한 사항은 legacy 프로젝트에 설명)
   
        - 방법1: <bean> 테그사용
   
          ```
          <bean id="nickName" class="com.test01.NickName"></bean>
          <bean id="myNickName" class="com.test01.MyNickName">
          		<property name="nickName" ref="nickName"></property></bean>
          ```
   
        - 방법2: annotation 
   
          - Spring에서 annotation을 사용하려면 다음과 같은 설정이 필요하나 굳이 넣지 않아도됨
   
            ```
            <bean class="org.springframework.context.annotation.CommonAnnotationBeanPostProcessor"/>
            ```
   
          - <context:annotation-config>
   
             	@Autowired, @Required, @Resource을 자동으로 처리해주는 bean post processor
             
          - <context:component-scan>
          
             	@Component, @controller, @service, @repository등의 annotion을 처리
             	<context:component-scan base-package="com/test06"/>
          
          - <mvc:annotation-driven>
          
             	@RequestMapping, @vaild등의 spring mvc compoenet등을 자동 처리
             	<mvc:annotation-driven/>

2. ### Spring legacy project 기준

   1. peoject 생성
   
      - Project - spring legacy project (template: spring MVC project) - source 입력 생성
   
   2. MVC 프로젝트 기본 사용
   
      - MySql
      - Mybatis
      - dbcp
      - spring-orm
   
   3. pom.xml 업데이트
   
      - property 수정
   
        - Java version: 11
        - springframework-version: 5.3.18
        - aspectj-version: 1.9.7
   
      - depandancy 추가
   
        ```xml
        		<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        		<dependency>
        		    <groupId>mysql</groupId>
        		    <artifactId>mysql-connector-java</artifactId>
        		    <version>8.0.29</version>
        		</dependency>
        		<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
        		<dependency>
        		    <groupId>org.mybatis</groupId>
        		    <artifactId>mybatis</artifactId>
        		    <version>3.5.7</version>
        		</dependency>
        		<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
        		<dependency>
        		    <groupId>org.mybatis</groupId>
        		    <artifactId>mybatis-spring</artifactId>
        		    <version>2.0.6</version>
        		</dependency>
        		<!-- https://mvnrepository.com/artifact/commons-dbcp/commons-dbcp -->
        		<dependency>
        		    <groupId>commons-dbcp</groupId>
        		    <artifactId>commons-dbcp</artifactId>
        		    <version>1.4</version>
        		</dependency>
        		<!-- https://mvnrepository.com/artifact/org.springframework/spring-orm -->
        		<dependency>
        		    <groupId>org.springframework</groupId>
        		    <artifactId>spring-orm</artifactId>
        		    <version>${org.springframework-version}</version>
        		</dependency>
        ```
   
   4. web.xml 업데이트
   
      - 경로: \src\main\webapp\WEB-INF\web.xml
      - 기본적용 사항
        - 기본적으로 context-param, listener, servlet, mapping은 설정되어 있다
        - 수정사항
          - <context-param> root-context.xml 필요시 파일 이름 혹은 경로 수정
          - <servlet> <param-value> 필요시 파일 이름 혹은 경로 수정
          - <servlet-mapping> <url-pattern> 호출받는 경로형태로 수정 (e.g. *.do (모든이름.do)) 
   
   		5. DB쿼리 준비 (선택)
   
        - 경로: /WEB-INF/spring/sql/test_db.txt
   
   		6. dto, dao, service, controller class file 생성
   
        - 스프링에서는 dao(@Repository), service(@Service), controller(@Controller) 어노테이션으로 객체를 생성한다. 즉, 스프링프레임워크에 내가 다오요, 서비스요, 컨트롤러요라고 선언하는 것
   
        - **dto**
   
          - DB의 컬럼 명과 동일하게 filed 생성 (나중에 jsp나 mapper에서 response 데이터를 dto형식으로 바로 받을수 있음(스프링에서만)
          - 기본, 필드 생성자 및 getter setter 생성
   
        - **dao**
   
          - interface dao하나 만들고 method 정의한다. 이때 NAMESPACE 변수도 하나 정의해준다.
   
            - NAMESPACE는 아래 dao implmention 클래스에서 sqlSession 실행할때 mybatis mapper에서 namespace로 호출값을 인식하기 때문이다.
   
              ```xml
              				-------------dao.java---------------------
              public interface BoardDao {
              	String NAMESPACE = "myboard."; 				**네임스페이스
              				-------------daoimpl.java------------------			
              @Repository
              public class BoardDaoImpl implements BoardDao{
              	private SqlSessionTemplate sqlSession;
              	@Override
              	public List<BoardDto> selectList() {
              		List<BoardDto> list = new ArrayList<BoardDto>();
              		list = sqlSession.selectList(NAMESPACE+"selectList"); 	**네임스페이스
              				-------------mapper.xml------------------
               <mapper namespace="myboard">  								**네임스페이스
                	<resultMap type="boardDto" id="boardMap">
              		중략
                		<result property="mydate" column="mydate"/> 
                	</resultMap>
                	<select id="selectList" resultType="boardDto">
                		select * 중략
                	</select>
              ```
   
          - implemention 클래스를 만들고 dao interface를 상속 받는다.
   
            - 클래스에 bean(객체) 생성을 위해  @Repository 어노테이션붙인다
   
            - SqlSessionTemplate 객체로 사용하기 위해 @Autowired로 인젝션한다.
   
              ```java
              @Repository
              public class BoardDaoImpl implements BoardDao{
              	@Autowired //sql세션객체로 사용
              	private SqlSessionTemplate sqlSession;
              ```
   
            - 주의: 호출하는 class에서 new연산자를 만들어 bean객체로 만들어진 클래스 method를 호출하는 경우 spring bean객체가 동작하지 않고 java 객체가 동작해서 클래스내 spring으로 annotation된 객체들을 사용할수 없다. (null로 나온다)
   
              ```java
               ------------------- serviceimpl --------------------
              서비스에서 new 연산자로 dao를 생성하고 dao의 selectList를 호출하면 
              @Service
              public class BoardServiceImpl implements BoardService{
              	주석//@Autowired
              	주석//private BoardDao dao;
              	BoardDao dao = new BoardDaoImpl();
              	@Override
              	public List<BoardDto> selectList() {
              		return dao.selectList();}
               ------------------ daoimpl -----------------------
              dao는 실행이되나 @Autowired된 sqlSession 값은 null을 가진다.
              "애초에 부를때 spring객체로 호출한게 아니고 java new 객체로 호출하였기 때문에 dao클래스 자체를 스프링 객체로 인식하지 못하고 역시 자바객체이기때문에 @Autowired도 동작을 안하는거라 추측됨"
              @Repository
              public class BoardDaoImpl implements BoardDao{
              	@Autowired
              	private SqlSessionTemplate sqlSession;
              	@Override
              	public List<BoardDto> selectList() {
              		System.out.println("new BoardDaoImpl(); 동작 ok");
              		System.out.println("sqlSession: " + sqlSession); <----------null
              ```
   
        - service
   
          - interface service하나 만들고 method 정의한다.
   
          - implemention 클래스를 더 만들고 interface를 상속 받는다.
   
          - 클래스에 bean(객체) 생성을 위해  @Service 어노테이션붙인다
   
          - dao를 호출해서 사용해야하기 때문에 dao 필드를 선언하고 @Autowired해서 객체를 생성한다.
   
            ```
            @Autowired
            private BoardDao dao;
            ```
   
        - controller
   
          - 클래스에 bean(객체) 생성을 위해  @Controller 어노테이션붙인다
   
          - 서비스를 호출해서 사용해야하기 때문에 service 필드를 선언하고 @Autowired해서 객체를 생성한다.
   
            ```
            @Autowired
            private BoardService service;
            ```
   
          -  디스패쳐서블릿에서 호출받은 명령 포멧은 requestmapping으로 읽어 온다.
   
            ```
            @RequestMapping("/list.do")
            public String list(Model model) {
            ```
   
   		7. board-mapper.xml
   
        - dao implement에서 sqlSession으로 호출하는 SqlSessionTemplate 객체 실행을 위한 mybatis xml 파일 (쿼리문 실행)
   
        - 경로: \src\main\resources\mybatis\board-mapper.xml
   
        - mapper 등록을 위해 상단에 문서 타입을 선언해준다 (https://mybatis.org/mybatis-3/ko/getting-started.html 매핑된 SQL 구문 살펴보기)
   
          ```xml
          <?xml version="1.0" encoding="UTF-8"?>
          mapper 등록
           <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
          ```
   
        - <mapper namespace="myboard">로  dao의 namespace를 등록해주고 아래와 같이 쿼리문 작성하면된다.
   
          - ***XML에서는 인자를 불러올때 #{ DB컬럼명 } 으로 한다.***
   
          ```
          <mapper namespace="myboard"> <!-- 매퍼파일 아래 영역의 이름 -->
          
            	<resultMap type="boardDto" id="boardMap">
            		<result property="myno" column="myno"/>
            		<result property="myname" column="myname"/>
            		<result property="mytitle" column="mytitle"/>
            		<result property="mycontent" column="mycontent"/>
            		<result property="mydate" column="mydate"/> <!-- 컬럼값 디비 쿼리 인덱스 -->
            	</resultMap>
            	
            	<select id="selectList" resultType="boardDto">
            		select *
            		from myboard
            		order by myno desc
            	</select>
            
            	<select id="selectOne" parameterType="int" resultMap="boardMap"> <!-- parameterType="int" 생략해도되나 넣어주면 좋음 -->
            		select *<!-- myno, myname, mytitle, mycontent, mydate -->
            		from myboard
            		where myno = #{myno}
            	</select>
            	
            	<select id="insert" parameterType="boardDto">
            		insert into myboard
            		values( null, #{myname }, #{mytitle }, #{mycontent }, now() )
            	</select>
          ```
   
          - resultmap: 쿼리실행후 리턴하려는 데이터의 타입을 새로정의 
   
            (db컬럼명과 dto의 변수명이 다를경우 /같으면 resulttype 자바클래스로 하면됨)
   
            - resultMap type: return하려는 객체 타입 (보통 dto클래스명)
            - resultMap id: resultMap을 호출했을때 쓰는 아이디
            - result property: resultMap type의 변수명
            - result column: DB의 컬럼명
   
          - select: SqlSessionTemplate 호출로 넘어온 매개변수로 쿼리문을 실행하게 함
   
            - select id: namespace와 함께 넘어온 아이디
   
            - select  parameterType: 넘어온 매개변수의 타입 (dao에서 호출시 myno)
   
              ```java
              dto = sqlSession.selectOne(NAMESPACE+"selectOne", myno);
              ```
   
            - resultType: 쿼리실행후 return시 기본 객체 데이터 타입으로 할때 
   
            - resultMap: 쿼리실행후 return시 resultmap에서 정의한 타입으로 하려고 할때
