# Spring boot에서 return시 redirect와 페이지 이동
## application properties 에서 아래와 같이 view pre and surfix 설정했을 경우.   
```xml
#view
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

### java의 경우, 컨트롤러에서
 - <mark>return 값으로 "/user/login"으로 하면</mark> 컨트롤러의 mapping된 url로 이동하는 것이 아닌, <mark>뷰의 user 폴더하단의 login jsp로 이동하는 것이다.<mark> 
 - <mark>return 값으로 <b>"redirect:/user/login"<b>으로 하면</mark> 컨트롤러의 mapping된 동일한 url로 실행이된다.

### jsp의 경우
 - onclick시에는 onclick="locaion.href='/user/login'"하면, 리다이렉트와 같은 기능을 한다.
 
하.....