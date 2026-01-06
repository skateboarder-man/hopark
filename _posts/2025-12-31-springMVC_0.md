---
layout: post
title: springMVC 패턴 spriong security_0(XML)
tags: [spring]
categories: spring
---

- Spring Security(JWT) 적용 이유

> React(front) + Spring MVC(backand) 방식으로 로그인 기능을 구현하기 위해 JWT 방식을 선택함.

- JWT를 선택한 이유

1. Stateless(무상태성)의 이점: 서버가 사용자의 상태를 저장하지 않으므로, 서버 부하가 줄어들고 수평적 확장(Scale-out)에 유리함.
2. REST API와의 궁합: React는 비동기 요청을 보내는 클라이언트임. 별도의 세션 관리 없이 헤더에 토큰을 실어 보내는 방식이 RESTful한 설계에 더 부합함.
3. 보안과 유연성: 세션 취약점(CSRF)에서 비교적 자유롭고, 한 번 발급된 토큰으로 다른 서비스(Microservices)에서도 인증이 가능하다는 점.

- JWT 흐름도 

> 1. 로그인 요청: 사용자가 React에서 ID/PW 전송.
> 2. 토큰 생성: Spring Security가 DB 확인 후, 유저 정보와 **권한(Role)**을 담은 Access Token 발급.
> 3. 토큰 저장: React는 받은 토큰을 localStorage나 Cookie에 저장.
> 4. 권한 요청: 이후 모든 요청 헤더에 Authorization: Bearer <JWT>를 포함.
> 5. 검증 및 인가: Spring Security 필터에서 토큰의 서명을 확인하고, 담겨있는 Role에 따라 요청을 허용하거나 거부.

- 구성

java 17 
DB: mysql

- 사용 라이브러리 (mvnrepository) https://mvnrepository.com/
```
    <!-- Spring Security-->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-web</artifactId>
            <version>5.8.13</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-config</artifactId>
            <version>5.8.13</version>
        </dependency>
            <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-core</artifactId>
            <version>5.8.13</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-taglibs</artifactId>
            <version>5.8.13</version>
        </dependency>
    <!-- jsonwebtoken -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.11.5</version> </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.11.5</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.11.5</version>
            <scope>runtime</scope>
        </dependency>
```

- context-security.xml 설정

```

<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
    xmlns:beans="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/security
                        http://www.springframework.org/schema/security/spring-security.xsd">
    <beans:bean id="passwordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
    <beans:bean id="jwtTokenProvider" class="co.spring.mvc.security.service.JwtTokenProvider"/>
    <beans:bean id="jwtAuthenticationEntryPoint" class="co.spring.mvc.security.service.JwtAuthenticationEntryPoint"/>
    <beans:bean id="refreshTokenService" class="co.spring.mvc.token.service.impl.RefreshTokenServiceImpl" />
    <beans:bean id="jwtAuthenticationFilter" class="co.spring.mvc.security.filter.JwtAuthenticationFilter">
        <beans:constructor-arg ref="jwtTokenProvider"/>
    </beans:bean>
    <beans:bean id="jwtAuthenticationSuccessHandler" class="co.spring.mvc.security.handler.JwtAuthenticationSuccessHandler">
        <beans:constructor-arg ref="jwtTokenProvider"/>
        <beans:constructor-arg ref="refreshTokenService"/>
    </beans:bean>
    <beans:bean id="jwtAuthenticationFailureHandler" class="co.spring.mvc.security.handler.JwtAuthenticationFailureHandler">
    </beans:bean>
    <beans:bean id="jsonAuthenticationFilter" class="co.spring.mvc.security.filter.JsonAuthenticationFilter">
        <beans:constructor-arg ref="authenticationManager"/>
        <beans:property name="authenticationSuccessHandler" ref="jwtAuthenticationSuccessHandler"/>
        <beans:property name="authenticationFailureHandler" ref="jwtAuthenticationFailureHandler"/>
    </beans:bean>
    <beans:bean id="corsConfigurationSource" class="org.springframework.web.cors.UrlBasedCorsConfigurationSource">
	    <beans:property name="corsConfigurations">
	        <beans:map>
	            <beans:entry key="/**"> 
	            	<beans:bean class="org.springframework.web.cors.CorsConfiguration">
	                    <beans:property name="allowedOrigins">
	                        <beans:list>
	                            <beans:value>http://localhost:3000</beans:value> 
	                        </beans:list>
	                    </beans:property>
	                    <beans:property name="allowedMethods">
	                        <beans:list>
	                            <beans:value>GET</beans:value>
	                            <beans:value>POST</beans:value>
	                            <beans:value>PUT</beans:value>
	                            <beans:value>DELETE</beans:value>
	                            <beans:value>OPTIONS</beans:value>
	                        </beans:list>
	                    </beans:property>
	                    <beans:property name="allowedHeaders">
	                        <beans:list>
	                            <beans:value>*</beans:value>
	                        </beans:list>
	                    </beans:property>
	                    <beans:property name="allowCredentials" value="true"/>
	                    <beans:property name="maxAge" value="3600"/>
	                </beans:bean>
	            </beans:entry>
	        </beans:map>
	    </beans:property>
	</beans:bean>
	<beans:bean id="corsFilter" class="org.springframework.web.filter.CorsFilter">
    	<beans:constructor-arg ref="corsConfigurationSource" />
	</beans:bean>
    <http use-expressions="true" create-session="stateless" entry-point-ref="jwtAuthenticationEntryPoint">
    	<custom-filter ref="corsFilter" before="FIRST" />
        <intercept-url pattern="/resources/**" access="permitAll" />
        <intercept-url pattern="/" access="permitAll" />
        <intercept-url pattern="/register/**" access="permitAll" />
        <intercept-url pattern="/css/**" access="permitAll" />
        <intercept-url pattern="/images/**" access="permitAll" />
        <intercept-url pattern="/common/**" access="permitAll" />
        <intercept-url pattern="/js/**" access="permitAll" />
        <intercept-url pattern="/public/**" access="permitAll" />
        <intercept-url pattern="/role/**" access="hasRole('ROLE_ADMIN')" />
        <intercept-url pattern="/**" access="isAuthenticated()" />
        <custom-filter ref="jsonAuthenticationFilter" position="FORM_LOGIN_FILTER" />
        <custom-filter ref="jwtAuthenticationFilter" before="BEARER_TOKEN_AUTH_FILTER" />
        <csrf disabled="true" />
        <headers>
            <frame-options disabled="true"/>
            <cache-control />
        </headers> 
    </http>
      <authentication-manager alias="authenticationManager">
    	<authentication-provider user-service-ref="userSecurityService">
	     	<password-encoder ref="passwordEncoder"/> 
        </authentication-provider>
	</authentication-manager>
</beans:beans>
```

- context-security.xml 설정 설명

> 1. 주요 보안 빈(Bean) 설정
> - BCryptPasswordEncoder: 비밀번호를 안전하게 해싱(암호화)하여 저장하고 비교할 때 사용.
> - JwtTokenProvider: JWT 토큰의 생성, 파싱, 유효성 검증을 담당하는 핵심 클래스.
> - JwtAuthenticationEntryPoint: 인증되지 않은 사용자가 보호된 리소스에 접근했을 때 에러(401 Unauthorized)를 처리.
> - RefreshTokenService: 토큰 만료 시 재발급을 위한 Refresh 토큰 로직을 담당.
> - jsonAuthenticationFilter : 일반적인 폼 로그인이 아닌, JSON 데이터로 들어오는 로그인 요청(ID/PW)을 처리, 성공 시 jwtAuthenticationSuccessHandler를 통해 JWT를 발급, 실패시 jwtAuthenticationFailureHandler 통해 계정 잠금 및 메시지 출력.

> 2. HTTP 보안 설정 (http)
> - 비상태성 유지 (create-session="stateless"): 서버에서 세션을 생성하지 않겠다는 설정, JWT를 사용하므로 서버 메모리에 사용자 상태를 저장할 필요가 없음.
> - CORS 설정 (corsFilter): http://localhost:3000 (주로 React나 Vue 같은 프런트엔드)에서 오는 요청을 허용하며, 모든 HTTP 메서드(GET, POST 등)를 허용.
> - URL별 접근 권한: 로그인 없이 누구나 접근 가능 (permitAll), 관리자(ROLE_ADMIN)만 접근 가능, 그 외 모든 요청 반드시 인증된(로그인한) 사용자만 접근 가능.
> -  CSRF 비활성화 (csrf disabled="true"): JWT를 사용하는 Stateless 환경에서는 CSRF 공격 위험이 적고, API 서버 특성상 비활성화하는 경우가 많음.

> 2. 인증 관리 (authentication-manager)
> - userSecurityService : 데이터베이스에서 사용자 정보를 조회하는 서비스.
> - passwordEncoder: 사용자가 입력한 비밀번호와 DB의 암호화된 비밀번호를 비교할 때 사용.

이번 포스팅에서는 XML 설정을 통한 보안 구조를 살펴보았습니다. 이어지는 다음 포스팅에서는 실제 동작을 담당하는 주요 Bean 설정 클래스와 필터, 그리고 인증 관리 클래스를 직접 구현하는 방법에 대해 자세히 알아보겠다.

