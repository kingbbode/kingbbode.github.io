스프링 시큐리티의 원리가되는 핵심 구조를 정리해보자. 시큐리티의 원리를 이용한 우회 방법을 공유하자.

스프링 스큐리티 Filter Chain
============================

스프링 시큐리티는 전적으로 서블릿필터를 기반으로 합니다.

![SERVLET](/images/2017/2017-02-21-SPRING-SECURITY-FILTER/sevlet.png)

어플리케이션의 모든 요청을 처리하는 스프링의 `DispatcherServlet`에 `Request`가 도달하기까지 거치게되고, `DispatcherServlet`으로부터 응답이 사용자에게 전달되기까지 거치게되는 `Filter`들을 서블릿필터라고 합니다. 흔히 알고있는 서블릿필터로는 `Static Resource`를 처리하는 필터와 `Character Encoding`가 있습니다.

-	[Spring Docs](http://docs.spring.io/spring-security/site/docs/4.2.2.BUILD-SNAPSHOT/reference/htmlsingle/#security-filter-chain)
