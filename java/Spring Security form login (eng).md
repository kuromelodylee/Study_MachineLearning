# Spring Security
## How to implement LOGIN function on normal mvc projects
### This document is just for my reference refering blogs. As it could contain wrong information, please avoid to scrap to be spread.
<br>

## 1. Abstract
### In order to implement on normal basic MVC pattern with Spring boot project, it needs to be set up with depandancy, configuration and some features on normal MVC projects.
### Following will be LOGIN examples for the implementation.

## 2. Environment
 - ### Spring boot 2.7.1 (maven)
 - ### Login url: /user/login

## 3. Basic MVC login pattern
 1. ### client requests with form submit
 2. ### controller class call serveice
 3. ### service call mapper
 4. ### mapper carries on query
 5. ### return matching data then controller handles result

## 4. Spring Security login pattern (Simplified)
 1. ### client requests with form submit
 2. ### Squrity "AuthenticationManager" call "UserDetailsService"
 3. ### "UserDetailsService" returns user access properties from data base
 4. ### comparing properties with client input data
 5. ### matching, return sucess

## 5. Implement Spring Sequrity Login
 1. ### Setting up dependency
    - <span style="background-color: yellow; color: black" > Step1) move to pom.xml insert following dependency</span>
        ```xml
        <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        ```
     - after setting dependancy, refresh or update maven required or terminate server then reopen it
     - if dependency works, default login page will be open autometically, after getting into main page.
 2. ### create configuration file
    - this file will help to customize setting for spring security.
    - <span style="background-color: yellow; color: black" > Step2) create the configuration under java package   
    e.g. src/main/java/com/com2/project/SecurityConfig.java
        ```java
        @Configuration
        public class SecurityConfig{
        
        //make static resource excepted from security frame work (later it should be modified for security safe)
        @Bean
        public WebSecurityCustomizer webSecCustomizer(){
            return web -> web.ignoring().antMatchers("/resources/**");
        }

        //setting up requests
        @Bean
        public SecurityFilterChain secFiltChain(HttpSecurity http) throws Exception{
            return http.csrf().disable()
                .headers()
                    .frameOptions().disable().and()
                //contorol all requests (permitall is to let all user can access)
                .authorizeRequests()
                    .antMatchers("/**").permitAll()
                    //.antMatchers("/**").hasRole("USER") //ONLY USER can access.
                    .anyRequest().authenticated().and()
                //contorol login
                .formLogin()
                    .loginPage("/user/login").permitAll() //control login request and page
                    .loginProcessingUrl("/seclogin") //set jsp form action (if not set, able not to assgin action (delete action property is ok))
                    .usernameParameter("userId") //set jsp name parameter (default username)
                    .passwordParameter("userPw") //set jsp pass word parameter (default password)
                    .defaultSuccessUrl("/") //request when sucess 
                    .failureUrl("/user/login?error=true") //request when failure
                    .and()
                .logout()
                    .logoutRequestMatcher(new AntPathRequestMatcher("/user/logout"))
                    .logoutSuccessUrl("/")
                    .invalidateHttpSession(true) //kill session after logout
                    .and()
                .build();
        }
        ```
     - <span style="background-color: yellow; color: black" > Step3) in order to make a connection with "UserDetailsService" interface, AuthenticationManager need to be created and added as a bean in configuration class.
        ```java
        @Bean
        public AuthenticationManager authMng(AuthenticationConfiguration authConfig) throws Exception{
            return authConfig.getAuthenticationManager();
        }

        ```

 3. ### Add implements on UserService class
     - <span style="background-color: yellow; color: black" > Step4) if userService class is implemented by service interface. add UserDetailsService to implement and override default function loadUserByUsername provided by UserDetailsService interface from Spring Security.
        ```java
        @Service
        public class UserServiceImpl implements UserService, UserDetailsService{

            public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            UserDto secUser = userMapper.secLogin(username);
            //assign error message when unable to find user information from database.
            if (secUser == null) {
                throw new UsernameNotFoundException("unable to find user ID");
            }
            //User object provided by Spring Sequrity is string, string, arraylist type. Therefore need to assign authority property by generic list type. 
            List<GrantedAuthority> auth = new ArrayList<>();
            if (secUser.getUserGrade().equals("BASIC")){
                auth.add(new SimpleGrantedAuthority("ROLE_USER"));
            }else{
                auth.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
            }
            User secReturnUser = new User(secUser.getUserId(), secUser.getUserPw(), auth);
            //console shows secReturnUser [Username=securityTest, Password=[PROTECTED], Enabled=true, AccountNonExpired=true, credentialsNonExpired=true, AccountNonLocked=true, Granted Authorities=[USER]]
            return secReturnUser;
            }
        }
        ```
     - <span style="background-color: yellow; color: black" > Step5) if there is not mapped with encryption property after loadUserByUsername called, error will appear. Therefore, required to create encryption object "in the confuigiration class file".   
     console -> java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"
       ```java
        @Bean
        public PasswordEncoder passwordEncoder() {
            return new BCryptPasswordEncoder();
        }
       ```
     - if pw is not hased in the database like the password saved before applying this framework without hashing, Spring Security occurs error.    
     console -> o.s.s.c.bcrypt.BCryptPasswordEncoder     : Encoded password does not look like BCrypt
 4. ### When finding invalid properties
     - option1) Simplified solution: If trying to enter wrong password or invalid ID, parameter will be transfered to client naming "param.error" and its value is as mentioned on config "true" with string type. in HTML, able to get value with ```${param.error}```. in javascript, able to get with ```"${param.error}"```
     - option2) Customizable solution: if you want to cutomize failure properties, please refer [clickMe(in Korean)](https://kimcoder.tistory.com/249?category=911141) referring add failure handler in configuration and handler class.
       ```java
        //add config property in Configuration
        .failureHandler(new failureclassname())
       ```
       ```java
       //create failureclassname class file and inplements AuthenticationFailureHandler and annote Service and override function
        @Service
        public class failureclassname implements AuthenticationFailureHandler{
            @Override
            functions.....
        } 
        ```
 5. Success handler(optional)
     - Spring Security basically provide filter function to prevent unauthorized requests but need to modify redirect location or add functions after success of login, it helps to modify its requests.
     - <span style="background-color: blue; color: white" > oprional Step1) find your service class and implements AuthenticationSuccessHandler then override its method
        ```java
        @Service
        public class UserServiceImpl implements UserService, UserDetailsService, AuthenticationSuccessHandler{
            @Override
            public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
                    Authentication authentication) throws IOException, ServletException {
                        
                    }
        ```
     - <span style="background-color: blue; color: white" > optional Step2) example by the cases wrting method  
     Case1: when user accesses login page with login request.  
     Case2: when user accesses login page directly by url.  
     Case3: when user called back from security filter
         - Controller for case1, case2 (case3 are not requeired for controller)

            ```java
            @GetMapping("/login")
                public String userLogin(HttpServletRequest request, Authentication authentication) {
                    //create Authentication object
                    AuthenticationTrustResolver trustResolver = new AuthenticationTrustResolverImpl();
                    //check Authentication for client.
                    if (trustResolver.isAnonymous(SecurityContextHolder.getContext().getAuthentication())) {
                        //get uri from Referer
                        //simply Referer may refer previous request information.
                        String uri = request.getHeader("Referer");
                        //null means user access with direct url.
                        if (uri == null) {
                            request.getSession().setAttribute("index", request.getHeader(""));
                        //contain login means user comming from login page (e.g. login failure). below else if is not comming from login page.
                        }else if (!uri.contains("/login")){
                            request.getSession().setAttribute("index", request.getHeader("Referer"));
                        }
                        //direct url has index key with "" value
                        //user from login uri has index key with mainpage uri
                        //user from login has not any key or value for index.

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
            //declare value for uri
            String uri = "/";
            //variable for case3: requestCache, savedRequest are stored previous request data before requesting invalid accesses.
            RequestCache requestCache = new HttpSessionRequestCache(); 
            SavedRequest savedRequest = requestCache.getRequest(request, response);


            //variable for Case1 or Case2, if case1, dataFromIndex has main url information, and if case 2, it has "".
            String dataFromIndex = (String) request.getSession().getAttribute(("index"));
            
            //delete attributes in session object to manage memory.
            if (dataFromIndex != null) {
                request.getSession().removeAttribute("index");
            }
            //in Case3, store saved information by security filter in uri
            if(savedRequest != null) {
                uri = savedRequest.getRedirectUrl();
            //in Case2, store "" in uri
            }else if (dataFromIndex != null && dataFromIndex.equals("")){
                uri = dataFromIndex;
            } 

            response.sendRedirect(uri);
            }
            ```
     - <span style="background-color: blue; color: white" > optional Step3) add configuration peoperty with successHandler
        ```java
        .formLogin()
                //.loginPage("/user/login").permitAll()
                //.defaultSuccessUrl("/")
                .successHandler(new "YOUR SERVICE CLASS NAME"())
        ``` 
     - others
         - To prevent to access login page after login success, refered below to verify login in contoller
            ```java
            AuthenticationTrustResolver trustResolver = new AuthenticationTrustResolverImpl();
            if (trustResolver.isAnonymous(SecurityContextHolder.getContext().getAuthentication())) {
                //for not logged in user
            }else{
                //user who has session
            }
            ```

 6. Getting properties in view
     - <span style="background-color: yellow; color: black" > Step6) in the jsp files for view, it is required to plant tag library. in order to use its tag library need to add dependency and property before. propery for the version is not mandatory but if not worked, better to try to add.
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
     - <span style="background-color: yellow; color: black" > Step7) add tag library on the top of the jsp.
        ```jsp
        <%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags"  %>

        <!DOCTYPE html>
        ```
     - <span style="background-color: yellow; color: black" > Step8) check the data to get from request using tab library on html sections.
        ```jsp
        <body>
            <sec:authorize access="isAnonymous()">
            <!-- sec:authorize means whether user has authorized token. using access property name if isAnonymous(), it means users who dont loggen in. therefor, it will show only to them(not login user)-->
            <p>Login</p>
            </sec:authorize>

            <sec:authorize access="isAuthenticated()" >
            <!-- this means users who loggen in. therefor, it will show only to them( login user)-->
            Logout
            </sec:authorize>

            <sec:authentication property="principal.username" var="username"/>
            <!-- with sec:authentication let to get data authenticated by property. principal has all data authenticated. declare variable on princial value then using ${variablename}, able to show on html-->
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
