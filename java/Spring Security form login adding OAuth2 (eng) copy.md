# Spring Security Form Login adding OAuth2 (google login example) 
## How to add OAuth2 login on normal form login
### This document is just for my reference refering blogs. As it could contain wrong information, please avoid to scrap to be spread.
<br>

## 1. Abstract
### This document explains to add OAuth2 login on normal form login based on Spring Security5.
### Following will be Google login examples for the implementation.

## 2. Environment
 - ### Spring boot 2.7.1 (maven)
 - ### <mark> Spring Security 5 with form login</mark>   
    if not prepared with form login with spring security 5, please click [here](https://github.com/kuromelodylee/study/blob/master/java/Spring%20Security(eng).md) before proceed this.
 - ### Login url: /user/login

## 3. Simple guide
 1. ### add dependency
 2. ### add property
 3. ### add cofiguration
 4. ### add service
 5. ### DONE !

## 5. Implement Spring Sequrity Login
 1. ### Setting up dependency
    - <mark > Step1) move to pom.xml insert following dependency</mark>
        ```xml
        <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-oauth2-client</artifactId>
			<version>2.6.1</version>
		</dependency>
        ```
     - after setting dependancy, refresh or update maven required or terminate server then reopen it
     - if dependency works, default login page will be open autometically, after getting into main page.
 2. ### add configuration
    - <mark> Step2) create the configuration under java package </mark> 
    e.g. src/main/java/com/com2/project/SecurityConfig.java
        ```java
        @Configuration
        public class SecurityConfig{
       
        //setting up requests
        @Bean
        public SecurityFilterChain secFiltChain(HttpSecurity http) throws Exception{
            http
                .csrf().disable()
                //skip
                .logout()
                    .logoutRequestMatcher(new AntPathRequestMatcher("/user/logout"))
                    .logoutSuccessUrl("/")
                    .invalidateHttpSession(true) //kill session after logout
                    .and()
                .oauth2Login()
                    .loginPage("/user/login").permitAll() //direction where gateway gor OAuth2 login
                    .defaultSuccessUrl("/") 
                    .failureUrl("/user/login?error=true")
                    .userInfoEndpoint() //get user information after login success
                    .userService(userServiceImpl) //to deal with user information (similar with UserSevice in form login procedure) 
                ; 
            return http.build();
        }
        ```
    - if trying to add .oauth2Login() before .logout(), you may find error. 

 3. ### Add OAuth2User on service implement calss
     - extends DefaultOAuth2UserService and overide loadUser function.
        ```java
        @Override
        public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException{
        //must keep loadUser fucntion name
                    

        return null;
    }

    

        ```

### Reference
 - https://velog.io/@kyu9610/Spring-Security-6.-OAuth2-Google-Login
 - https://lotuus.tistory.com/79
