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
 ### sign up google cloud platform and set up OAuth2 login api first following [here kor](https://lotuus.tistory.com/79)
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

 2. ### add application properties
    - <mark>Step2) add google oauth2.client.registration information on application properies</mark>
        ```html
        spring.security.oauth2.client.registration.google.client-id = "your client id"
        spring.security.oauth2.client.registration.google.client-secret = "your client key"
        spring.security.oauth2.client.registration.google.scope = profile, email
        ```
 3. ### add configuration
    - <mark> Step3) add configuration for .oauth2Login() </mark> 
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

 4. ### Add OAuth2User on service implement calss
     - <mark>Step4) extends DefaultOAuth2UserService and overide loadUser function.</mark>
        ```java
        @Service
        public class UserServiceImpl extends DefaultOAuth2UserService implements UserService, UserDetailsService, AuthenticationSuccessHandler {

        //skip for form login

            @Override
            public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException{
            //must keep loadUser fucntion name
                        

            return null;
            }
        }
        ```
 5. ### composing return for loaduser
     - when loadUser activates you can find the login information form google id such as login key, google name, email and so on. Moverover, when you return DefaultOAuth2User class information, Spring security creates session and delivers token to views. 
     - <mark>Step5) So, what we need to do is to decide to save user information on data base or to let login data on memory. </mark>   
     below is the example to save informaion to database and return to loadUser
         - 1st step: extract the email (or whatelse related with your database column) and check weather exist already.
         - 2nd step: if not existing, store user information then call stored information
         - 3rd step: transfer the object type to DefaultOAuth2User the return


        ```java
        @Override
        public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException{
            //1st Step
            OAuth2User oauth2User = super.loadUser(userRequest); //call user information serialized
            String userEmail = oauth2User.getAttribute("email"); //get email form OAuth2User
            UserSocialDto dtoRes = chkUserSocialData(userEmail); //call function for checking user information in database by email (details skip in this doc)

            //2nd Step
            if (dtoRes == null) { // no information for user
                UserSocialDto dto = transferToDto(userRequest, oauth2User); //refer FUNCTION #1
                int res = regUserSocial(dto); //call function for save user information in database (details skip in this doc)
                if (res == 1){ //success to insert to database
                    dtoRes = chkUserSocialData(dto.getSocialEmail());            
                }else{ // fail to insert to database
                    throw new OAuth2AuthenticationException("404: system error when social login.");
                }
            }
            //3rd Step (refer below FUNCTION #2)
            OAuth2User res = transOAuth2User(dtoRes);

        return res;
        }

        //FUNCTION #1
        private UserSocialDto transferToDto(OAuth2UserRequest userRequest, OAuth2User oauth2User){
        
            UserSocialDto userSocialDto = new UserSocialDto();
            userSocialDto.setSocialProvider(userRequest.getClientRegistration().getRegistrationId());
            userSocialDto.setSocialId(oauth2User.getAttribute("sub"));
            userSocialDto.setSocialEmail(oauth2User.getAttribute("email"));
            userSocialDto.setSocialName(oauth2User.getAttribute("name"));
        
        return userSocialDto;
        }

        //FUNCTION #2
        private OAuth2User transOAuth2User(UserSocialDto dto){

            Map<String, Object> userDetails = new HashMap<>();
            userDetails.put("provider", dto.getSocialProvider());
            userDetails.put("sub", dto.getSocialId());
            userDetails.put("email", dto.getSocialEmail());
            userDetails.put("username", dto.getSocialName());

            Collection<GrantedAuthority> auth = new ArrayList<>();
            auth.add(new SimpleGrantedAuthority("ROLE_USER"));

            OAuth2User transedOA2User = new DefaultOAuth2User(auth, userDetails, "username");

        return transedOA2User;
        }
        
        ```






### Reference
 - https://velog.io/@kyu9610/Spring-Security-6.-OAuth2-Google-Login
 - https://lotuus.tistory.com/79
