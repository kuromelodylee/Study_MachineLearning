# Spring Security
#### 본인은 초보라 용어에 대해서 잘몰르고, 클래스간 전송에 대해 이해한 내용을 바탕으로 작성해서 정확하지 않을 수 있습니다. 여러블로그들을 참고하여 직접 구현을 해본 내용으로 작성하였습니다. 
<br>

## 어떻게 일반적인 MVC패턴에서 스프링시큐리티를 적용할까?
<br>

## 1. 요약
### 일반적인 MVC 스프링부트 프로젝트에서 스프링 시큐리티를 적용하기 위해서는, 디펜던시와 컨피규레이션 생성, 서비스등을 추가해 주어야 합니다. 
### 아래부터, 로그인 기능을 적용하는 예제 입니다.

## 2. 환경
 - ### Spring boot 2.7.1 (maven)
 - ### Login url: /user/login

## 3. 일반적인 MVC login 패턴
 1. ### 클라이언트가 form submit 전송합니다.
 2. ### controller가 받아서 서비스로 넘깁니다.
 3. ### service가 mapper로 넘깁니다.
 4. ### mapper가 쿼리문을 실행합니다.
 5. ### return 타입에 맞는 데이터를 넘겨주고 컨트롤러에서 적절한 처리를 합니다

## 4. Spring Security login 패턴 (쉽게말해서...)
 1. ### 클라이언트가 form submit 전송합니다.
 2. ### 시큐리티의 "AuthenticationManager"가 "UserDetailsService"를 호출합니다.
 3. ### "UserDetailsService"는 데이터베이스의 인증정보를 저장하고 시큐리티로 리턴합니다.
 4. ### 시큐리티는 넘겨받은 인증정보와 클라이언트의 입력 정보를 확인합니다.
 5. ### 맞으면 성공

## 5. Spring Sequrity Login 삽입
 1. ### 디펜던시 설정
    - <mark> Step1) pom.xml 설정으로가서 아래의 스프링부트 디펜던시를 넣어줍니다.</mark>
        ```xml
        <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        ```
     - 디펜던시를 추가하고 페이지 접근을 하려면, 디펜던시가 제대로 적용됬다면, 스프링 시큐리티가 제공하는 로그인 화면이 바로 보일겁니다. 만약 변한게 없다고 생각되면, 메이븐을 업데이트 하던가, 새로고침 해주어야 할 겁니다. 아님 서버를껏다 다시켜보시던가, 잠깐 쉬고오셔도 될듯합니다. 저도 한 1분정도 후에 적용되는 것 같았습니다. 
 2. ### configuration file 생성
    - 이파일은 스프링 시큐리티의 설정을 커스터마이징해줄 수 있습니다.
    - <mark> Step2) 자바 패키지 아래 설정 클라스 파일을 아래와 같이 하나 만들어주세요. </mark>   
    e.g. src/main/java/com/com2/project/SecurityConfig.java
        ```java
        @Configuration //설정파일이라고 정의해 줍니다. 시큐리티 5부터 설정파일은 아답타를 상속받지 않고 매서드마다 빈(객체)을 생성해야 합니다.
        public class SecurityConfig{
        
        //WebSecurityCustomizer를 만들어 시큐리티에서 제외할 경로를 지정해 봅니다. 일단 개발 편의를 위해서 리소스 파일은 제외 하였습니다. (나중에는 보안을 위해서 리소스파일도 시큐리티를 적용해야합니다.)
        @Bean
        public WebSecurityCustomizer webSecCustomizer(){
            return web -> web.ignoring().antMatchers("/resources/**");
        }

        //SecurityFilterChain을 만들어 요청사항들을 정의해 봅니다.
        @Bean
        public SecurityFilterChain secFiltChain(HttpSecurity http) throws Exception{
            return http.csrf().disable()
                .headers()
                    .frameOptions().disable().and()
                //모든 페이지의 요청을 컨트롤 합니다. (permitAll()로하면 인증을 받던 안받던 모두가 접근 가능합니다.)
                .authorizeRequests()
                    .antMatchers("/**").permitAll()
                    //.antMatchers("/**").hasRole("USER") //이것은 USER권한만 접근가능하다는 뜻입니다.
                    .anyRequest().authenticated().and()
                //로그인에 관한 설정합니다.
                .formLogin()
                    .loginPage("/user/login").permitAll() //로그인 페이지 url을 정의하고, 접근가능한 사용자를 정합니다. (여기서는 모두)
                    .loginProcessingUrl("/seclogin") //jsp에서 서브밋이 될때 action 값을 넣어줍니다. (jsp에서 액션값을 안넣어줘도 자동으로 스프링시큐리티랑 연결이됩니다. 이경우 loginProcessingUrl("/seclogin") 아예 빼버려도 됩니다)
                    .usernameParameter("userId") //jsp에서 로그인 정보로 보내는 id값의 파라미터 이름을 설정해 줍니다. (기본설정은 username이라서 안적어주시면 jsp에 input name값을 username 이라고 적어주면됩니다.)
                    .passwordParameter("userPw") //jsp에서 로그인 정보로 보내는 pw값의 파라미터 이름을 설정해 줍니다. (기본설정은 password이라서 안적어주시면 jsp에 input name값을 password 이라고 적어주면됩니다.)
                    .defaultSuccessUrl("/") // 성공을 하면 넘어가는 url 입니다. (뒤에 나오겠지만 success헨들러를 적용하면 이것은 적용되지 않습니다.)
                    .failureUrl("/user/login?error=true") //로그인이 실패했을때 전송하는 url입니다.
                    .and()
                .logout()
                    .logoutRequestMatcher(new AntPathRequestMatcher("/user/logout"))
                    .logoutSuccessUrl("/")
                    .invalidateHttpSession(true)
                    .and()
                .build();
        }
        ```
     - <mark> Step3) "UserDetailsService" interface와 연결을위해서는, AuthenticationManager 컨피규레이션 파일에 AuthenticationManager 객체가 생성이 되어야 합니다. 아래에서 "UserDetailsService" 에대해 설명 하겠습니다.
        ```java
        @Bean
        public AuthenticationManager authMng(AuthenticationConfiguration authConfig) throws Exception{
            return authConfig.getAuthenticationManager();
        }

        ```

 3. ### UserService의 상속
     - <mark>Step4) 위에서 언급한 AuthenticationManager는 UserService클래스를 동작시킵니다. </mark>
     - mvc 패턴을 작성해 두었다면, 기본의 있던 서비스에서 UserService를 상속받습니다. 인터페이스를 상속받고 매써드를 오버라이딩 해줍니다. 그러면 public UserDetails loadUserByUsername(String username) 이라는 매써드가 오버라이딩 됩니다. 상속중 UserDetailsService는 여기서 해당사항이 아니여서 아래에서 설명하겠습니다.
     - UserService는 클라이언트의 id 정보를 받아와서 id를 가지고 데이터베이스의 비밀번호와 권한을 조회 및 저장을 하여 스프링 시큐리티로 리턴해주는 역할까지만 합니다.
        ```java
        @Service
        public class UserServiceImpl implements UserService, UserDetailsService{

            @Override
            public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            //username을 가지고 데이터베이스에 저장된 유저의 정보를 불러 옵니다.
            UserDto secUser = userMapper.secLogin(username);
            //유저정보가 없으면 (데이터가 없다면) UsernameNotFoundException을 실행해 줍니다. (이건작동하는지 확인 안해봄.)
            if (secUser == null) {
                throw new UsernameNotFoundException("unable to find user ID");
            }
            //UserDetails는 스프링 시큐리티에서 제공하는 User라는 타입의 객체를 리턴으로 받을 수 있습니다. 이 User라는 객체는 (string, string, arraylist(특히, GrantedAuthority)) 타입으로 되어 있습니다. 따라서, 아래와 같이 User에 아이디, 비밀번호 권한(GrantedAuthority로 전환)을 저장해 줍니다.
            List<GrantedAuthority> auth = new ArrayList<>();
            if (secUser.getUserGrade().equals("BASIC")){
                //권한을 저장할때 ROLE_USER 형태로 저장을 하면 컨피큐레이션에서 .antMatchers("/cBoard").hasRole("USER") 이렇게 유저만 접근하게 할 수 있습니다.
                auth.add(new SimpleGrantedAuthority("ROLE_USER"));
            }else{
                auth.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
            }
            User secReturnUser = new User(secUser.getUserId(), secUser.getUserPw(), auth);
            //secReturnUser를 콘솔로 확인해 보면 이렇게 보입니다. secReturnUser [Username=securityTest, Password=[PROTECTED], Enabled=true, AccountNonExpired=true, credentialsNonExpired=true, AccountNonLocked=true, Granted Authorities=[USER]]
            return secReturnUser;
            }
        }
        ```
     - <mark> Step5) 암호화 속성이 되지 않아 있으면 loadUserByUsername이 호출시, error가 생깁니다. 따라서 컨피규레이션 파일에서 암호화 객체를 생성해 주어야 합니다. </mark>   
     콘솔 -> java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"
       ```java
        @Bean
        public PasswordEncoder passwordEncoder() {
            return new BCryptPasswordEncoder();
        }
       ```
     - 그리고, 만약 DB에 비밀번호가 헤싱되어서 저장이 되어있지 않으면 에러가 뜹니다.   
     콘솔 -> o.s.s.c.bcrypt.BCryptPasswordEncoder : Encoded password does not look like BCrypt   

 4. ### 유효하지 않은 데이터를 받은 경우 (로그인실패) 
     - 옵션 1) 간단한 solution: 만약 잘못된 아이디나, 비밀번호가 다르다던가 모든 경우에 "param.error"라는 파라미터로 view로 전송이됩니다. 저는 이미 컨피큐레이션파일에서 .failureUrl("/user/login?error=true")라고 설정해 주었습니다. 기본적으로 /user/login까지만 실패시 경로를 지정 해주면 param으로 ${param}으로 받아올 수 있고, 그때 값은 {error=} 이렇게 아마 받아올겁니다.   

        HTML에서는 값을 ```${param.error}``` 이렇게. javascript에서는, able to ```"${param.error}"``` 이렇게 받아올 수 있습니다.
     - 옵션 2) 커스터마이징 솔루션: 실패시 커스터마이징하고 싶다면, 여기를 클릭해 보세요 [clickMe(in Korean)](https://kimcoder.tistory.com/249?category=911141) 이것을 참고해서 failure handler를 in configuration과 handler class를 상속해주면 됩니다.
       ```java
        //컨피규레이션에 로그인form 아래 넣어줍니다.
        .failureHandler(new failureclassname())
       ```
       ```java
       //사용하고자 하는 서비스 클래스에서 AuthenticationFailureHandler 를 상속받아 매쏘드를 오버라이드 합니다. 
        @Service
        public class failureclassname implements AuthenticationFailureHandler{
            @Override
            functions.....
        } 
        ```
 5. Success handler(선택사항)
     - 이것은 스프링스큐리티에서 인증정보가 일치하였을때 추가로 작업해줄수 있는 것을 설정해 주는 것 입니다. 이것을 설정하면 컨피규레이션에서 defaultSuccessUrl("/")이 작동하지 않으며, 여기서 설정해 주는 url로 넘어가게 됩니다. URL로 넘어가기전에 다양한 기능을 추가 할 수 있습니다.
     - <mark > 선택사항 Step1) 사용할 서비스 클래스를 찾아 AuthenticationSuccessHandler를 상속받아 매쏘드를 오버라이드 해주세요. </mark>
        ```java
        @Service
        public class UserServiceImpl implements UserService, UserDetailsService, AuthenticationSuccessHandler{
            @Override
            public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
                    Authentication authentication) throws IOException, ServletException {
                        
                    }
        ```
     - <mark> 선택사항 Step2) 아래와 같은 경우를 고려해 메쏘드를 추가해 봅니다 (블로그를 거의 참조하였습니다) </mark>   
     Case1: 사용자가 페이지의 로그인 버튼을 통해 로그인을 시도한 경우   
     Case2: 사용자가 주소창에 url을 직접 입력해 접근한 경우   
     Case3: 사용자가 접근권한이 없는 페이지에 접근했다가 security filter에 의해 빠꾸당한 경우
         - 아래컨트롤러는 case1, case2에 대해서만 우선 설정합니다.
         - 일단, login 페이지에 접근할때, 인증을 받은 사용자인지 확인해 봅니다 (로그인을 했는가?)

            ```java
            @GetMapping("/login")
                public String userLogin(HttpServletRequest request, Authentication authentication) {
                    //인증을 받은 사용자인지 먼저 확인
                        //Authentication object를 생성해줍니다.
                    AuthenticationTrustResolver trustResolver = new AuthenticationTrustResolverImpl();
                        //SecurityContextHolder를 통해 인증정보를 가져오고 isAnonymous인지 확인합니다.
                    if (trustResolver.isAnonymous(SecurityContextHolder.getContext().getAuthentication())) {
                        //Referer로부터 uri를 가져옵니다.
                        //간단하게 Referer 쉽게말해서 직전에 요청한 정보를 가지고 있습니다.. 헤더정보는 url을 포함합니다.
                        String uri = request.getHeader("Referer");
                        //null이란 의미는 직접 url을 치고들어왔다는 뜻입니다.
                        if (uri == null) {
                            request.getSession().setAttribute("index", request.getHeader(""));
                        //contain login은 사용자가 login page에서 왔다는 뜻입니다. 예를 들어 로그인이 실패되서 새로고침되었던지. 아래는 else if는 login page로부터 안왔다는 뜻입니다.
                        }else if (!uri.contains("/login")){
                            request.getSession().setAttribute("index", request.getHeader("Referer"));
                        }
                        //직접 치고온경우 url index key 는 "" 값을 가집니다.
                        //메인페이지의 로그인 요청으로 경우 index key 값은 mainpage url을 가집니다.
                        //login페이지로 부터 새로고침된 사용자는 index라는 키자체가 없다.

                        return "/user/login";
                    }else {
                        return "redirect:/";
                    }
                }
            ```
         - Service

            ``` java
            @Override
            public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
            Authentication authentication) throws IOException, ServletException {
            //로그인 성공시 넘어갈 기본 uri을 우선 정의해 줍니다.
            String uri = "/";
            //case3을 구현하기 위한 변수 설정: requestCache, savedRequest 접근 권한이 없는 페이지를 요청하기전에 직전에 요청받은 데이터를 저장해 줍니다. 만약 마이페이지에서 커뮤니티페이지로 갔다가 어드민을 접근하려할때 어드민에서 빠꾸당하면, 마이페이지에서 커뮤니티페이지로 요청할때의 데이터가 저장되어 있습니다.
            RequestCache requestCache = new HttpSessionRequestCache(); 
            SavedRequest savedRequest = requestCache.getRequest(request, response);


            // Case1 or Case2에 대한 변수, case1인 경우 dataFromIndex는 main url 을 가지고, case 2의경우, ""를 가진다..
            String dataFromIndex = (String) request.getSession().getAttribute(("index"));
            
            //request에 저장되어있는 Attribute index를 지운다. 메모리 관리를 위해
            if (dataFromIndex != null) {
                request.getSession().removeAttribute("index");
            }
            //Case3, security filter에 의해 저장된 uri
            if(savedRequest != null) {
                uri = savedRequest.getRedirectUrl();
            //Case2, uri ""에 저장한다.
            }else if (dataFromIndex != null && dataFromIndex.equals("")){
                uri = dataFromIndex;
            } 

            response.sendRedirect(uri);
            }
            ```
     - <mark > optional Step3) 컨피규레이션에 successHandler 추가</mark>
        ```java
        .formLogin()
                //.loginPage("/user/login").permitAll()
                //.defaultSuccessUrl("/")
                .successHandler(new "YOUR SERVICE CLASS NAME"())
        ``` 
     - others
         - 로그인 성공하고 로그인페이지에 접근했을때, 로그인페이지로 접근하지 못하도록 아래를 컨트롤러에 추가한다.
            ```java
            AuthenticationTrustResolver trustResolver = new AuthenticationTrustResolverImpl();
            if (trustResolver.isAnonymous(SecurityContextHolder.getContext().getAuthentication())) {
                //로그인안한 사용자처리
            }else{
                //로그인한 사용자 처리
            }
            ```

 6. Getting properties in view
     - <mark> Step6) 뷰의 jsp파일에서 데이터를 가져오기 위해서는 시큐리티에서 제공하는 테그라이브러리를 심어주어야 합니다. 이 테그라이브러리를 사용하려면 pom.xml에 디펜던시와 프로퍼티를 추가합니다. 
     </mark>   

        ```xml
        <properties>
            <spring-security.version>5.7.2</spring-security.version>
        </properties>
        <dependency>	
                <groupId>org.springframework.security</groupId>
                <artifactId>spring-security-taglibs</artifactId>
                <version>${spring-security.version}</version>
        </dependency>
        ```

     - <mark> Step7) jsp에 테그라이브러리 작성. </mark>   
        ```jsp
        <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags"  %>

        <!DOCTYPE html>
        ```
     - <span style="background-color: yellow; color: black" > Step8) html 에서 tab library를 통해 넘어오는 데이터 확인해보기.
        ```jsp
        <body>
            <sec:authorize access="isAnonymous()">
            <!-- sec:authorize 는 authorized token을 가지고 있는지 확인한다. access를 통해 property name 이 isAnonymous()라는 것은, 로그인을 하지않은 사용자를 의미한다. 따라서, 이것은 로그인하지 않은 사용자에게만 보여진다. -->
            <p>Login</p>
            </sec:authorize>

            <sec:authorize access="isAuthenticated()" >
            <!-- 이것은 로그인한 사용자를 의미한다. -->
            Logout
            </sec:authorize>

            <sec:authentication property="principal.username" var="username"/>
            <!-- sec:authentication을 가지고 property를 통해서 데이터를 확인할 수 있다. principal 인증된 모든 데이터정보를 가지고 있다. variable 선언을 통해서 princial value를 ${variablename}사용해 페이지에서 활용할 수 있다-->
            <p>${username}</p>

            <sec:authentication property="principal" var="pcp"/>
            <p>${pcp}</p>
            </sec:authorize>
        </body>
        ```



### Reference
 - https://velog.io/@csh0034/Spring-Security-Config-Refactoring
 - https://wikidocs.net/162255
 - https://nahwasa.com/entry/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-Spring-Security-%EA%B8%B0%EB%B3%B8-%EC%84%B8%ED%8C%85-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0
 - https://lemontia.tistory.com/601
 - https://kimcoder.tistory.com/250?category=911141
 - https://codevang.tistory.com/269
 - https://okky.kr/article/416253
 - https://www.baeldung.com/spring-security-taglibs

