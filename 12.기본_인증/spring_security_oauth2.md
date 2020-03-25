# Spring Security OAuth2

기본적으로 `OAuth2` 방식에는 4가지 방식이 있습니다.

- Authorization Code Grant
- Implicit Grant
- Resource Owner Password Credentials Grant
- Client Credentials Grant

## Authorization Code Grant Flow

그중 첫번째인 **Authorization Code Grant** 방식이 제일 많이 사용됩니다.

![](https://user-images.githubusercontent.com/30451129/70374365-b0988200-1934-11ea-8c4d-e901c10af800.png)

- (A) `Resource Owner(사용자)`는 `User-Agent(브라우저)`를 통해 `Client(Application)`에게 domain.com/oauth2/authorization/github 경로로 요청을 보냅니다.
- (A) `Client`는 위 경로로 들어온 요청에 대해 OAuth2 인증 방식 요청임을 확인하고 `Authorization Server(권한 서버, OAuth Provider 서버 : Github OAuth App Server)`에게 접근할 수 있는 경로를 Location 해더에 담아 응답합니다. 이때 Client는 Location 정보로 authorization-endpoint ([https://github.com/login/oauth/authorize](https://github.com/login/oauth/authorize))와 쿼리 스트링으로 Client Identifier (client-id), Redirection-URI(domain.com/login/oauth2/code/github) 등(scope, state...)을 담아줍니다.
- (A) 302 리다이렉션 응답을 받은 `User-Agent`는 Location 경로에 의해 `Authorization Server`에게 요청을 보내고 이 권한 서버는 위에서 `Client`가 담아놓은 client-id를 확인하여 해당 oauth app의 권한 승인 페이지로 이동시켜줍니다. (oauth2 provider에 로그인 되어 있지 않다면 로그인을 먼저 하라는 페이지로 이동시킵니다.)
- (B) `User-Agent`에 승인페이지가 띄우고 `Resource Owner`는 권한을 부여하거나 거절합니다. 권한 승인(또는 거절) 정보를 `Authorization Server`에 보냅니다.
- (C) 만약 `Resource Owner`가 권한 승인했다면 `Authorization Server`는 token(code)을 발행합니다. 그리고 `User-Agent`를 이전에 전달 받은 Redirect-URI 경로(domain.com/login/oauth2/code/github)로 리다이렉트 시킵니다. 이때 리다이렉트 경로에 발급한 토큰과 더불어 이전에 `Client`가 전달한 여러 상태 값을 같이 담아줍니다.
- (D) `User-Agent`는 `Client`로 리다이렉트될 것이고 `Client`는 다시 `Authorization Server`에게 Token-URI([https://github.com/login/oauth/access_token](https://github.com/login/oauth/access_token))경로에 Post 요청으로 Access Token을 달라는 요청을 보냅니다. 이때 Access Token을 발급하기위한 인증을 위해 이전에 받은 token(code)과 더불어 `Authorization Server`에 보냈었던 상태값(client-id, client-secret, redirect-uri)을 같이 보냅니다.
- (E) `Authorization Server`는 `Client`가 보낸 값을 가지고 타당한지를 확인하고 유효한 정보가 확인됐을 경우 Access Token을 발행하여 (선택적으로 Refresh Token도 같이 발행한다.) `Client`로 응답합니다.

    *(C)에서 발행한 token과 (E)에서 발행한 access token은 다른 것입니다.*

- 추가로 `Client`가 Access Token을 발급 받으면 해당 토큰을 이용하여 `Resource Server`의 user-Info-endpoint([https://api.github.com/user](https://api.github.com/user)/{user-id}) 경로로 사용자 정보를 받아옵니다.

## Spring Security + OAuth2

### OAuth 설정

사실상 Spring Security에서 OAuth2에서 필요한 설정을 대부분 해주고 있습니다.
```java
    package org.springframework.security.config.oauth2.client;
    
    public enum CommonOAuth2Provider {
    	...
    
    	GITHUB {
    
    		@Override
    		public Builder getBuilder(String registrationId) {
    			ClientRegistration.Builder builder = getBuilder(registrationId,
    					ClientAuthenticationMethod.BASIC, DEFAULT_REDIRECT_URL);
    			builder.scope("read:user");
    			builder.authorizationUri("https://github.com/login/oauth/authorize");
    			builder.tokenUri("https://github.com/login/oauth/access_token");
    			builder.userInfoUri("https://api.github.com/user");
    			builder.userNameAttributeName("id");
    			builder.clientName("GitHub");
    			return builder;
    		}
    	},
    
    	...
    }
```
이미 Spring Security가 위 `CommonOAuth2Provider` 처럼 OAuth 연동에 필요한 (github oauth의)기본 설정 정보를 다 만들어놔서 바로 사용할 수 있도록 제공하고 있습니다. 

**Authorization Code Grant Flow**에서 이야기한 `Authorization Server`로 접근하기 위한 authorization-endpoint, Access Token을 발급받기 위한 uri, 인가를 받고 사용자 정보를 받기위한 userInfo-endpoint 등 `CommonOAuth2Provider.GITHUB` 에서 정의된 값으로 자동 설정된 것입니다.

추가로 처음 사용자(`Resource Owner`)가 oauth 인증을 하기위한 시도의 시발점(A)으로 접근하는 경로 [domain.com/oauth2/authorization/github](http://domain.com/oauth2/authorization/github) 또한 Spring Security가 기본으로 제공하는 설정 값입니다.

그래서 우리는 아래처럼 두가지(client-id / client-secret)만 설정해주면 됩니다.

    # application.yml
    
    spring:
      security:
        oauth2:
          client:
            registration:
              github:
                client-id: <클라이언트 ID>
                client-secret: <클라이언트 SECRET>

### OAuth 적용
```java
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
        @Override
        protected void configure(final HttpSecurity http) throws Exception {
            http.authorizeRequests()
                    .antMatchers("/").permitAll()
                    .anyRequest().authenticated()
                .and()
                    .oauth2Login();            // 기본 oauth login 적용
        }
    }
```
`.oauth2Login()` 을 적용함으로써 Spring Security 에 새로운 필터가 두개가 추가됩니다.

이 필터가 OAuth2 적용을 위한 설정과 통신을 담당하게 됩니다.

    class org.springframework.security.oauth2.client.web.OAuth2AuthorizationRequestRedirectFilter
    class org.springframework.security.oauth2.client.web.OAuth2LoginAuthenticationFilter

위 **Authorization Code Grant Flow** 에서 (A), (B), (C) 까지 `User-Agent(브라우저)`와 `Authorization Server`간의 통신이었고 (D), (E)가 `Client`와 `Authorization / Resource Server`간의 통신입니다.

*브라우저를 통한 통신은 그렇다 쳐도 서버간(`Client` - `Authorization Server`) 통신은 어떻게 이뤄질까?*
```java
    package org.springframework.security.oauth2.client.endpoint;
    
    // Access Token 발급 과정
    public class DefaultAuthorizationCodeTokenResponseClient {
    	...
    
    	@Override
    	public OAuth2AccessTokenResponse getTokenResponse(OAuth2AuthorizationCodeGrantRequest authorizationCodeGrantRequest) {
    		...
    		// post request 생성
    		// public T convert(S s) {
    		//	...
    		//	// 기존에 Spring Security가 설정한 token uri 값을 불러와 uri 경로로 설정
    		//	URI uri = UriComponentsBuilder.fromUriString(clientRegistration.getProviderDetails().getTokenUri())
    		//		.build()
    		//		.toUri();
    		//	return new RequestEntity<>(formParameters, headers, HttpMethod.POST, uri);
    		// }
    		RequestEntity<?> request = this.requestEntityConverter.convert(authorizationCodeGrantRequest);
    
    		ResponseEntity<OAuth2AccessTokenResponse> response;
    		try {
    			// RestOperation을 이용한 통신을 진행
    			response = this.restOperations.exchange(request, OAuth2AccessTokenResponse.class);
    		} catch (RestClientException ex) {
    			OAuth2Error oauth2Error = new OAuth2Error(INVALID_TOKEN_RESPONSE_ERROR_CODE,
    					"An error occurred while attempting to retrieve the OAuth 2.0 Access Token Response: " + ex.getMessage(), null);
    			throw new OAuth2AuthorizationException(oauth2Error, ex);
    		}
    
    		OAuth2AccessTokenResponse tokenResponse = response.getBody();
    
    		...
    	}
    
    	...
    }
```
위 코드에 나와있다시피 `this.requestEntityConverter.convert(authorizationCodeGrantRequest);` 를 통해 Http Request를 Access Token을 발급받기 위한 요청 규약에 맞게 생성하고 `this.restOperations.exchange(request, OAuth2AccessTokenResponse.class);` 로 통신을 하게 됩니다.

*`SecurityConfig` Spring Security 설정에서 `.oauth2Login()`만으로 설정해놨기 때문에 Default로 설정된 클래스`DefaultAuthorizationCodeTokenResponseClient`에 의해 oauth 인증 - 인가가 처리됩니다.*
```java
    package org.springframework.security.oauth2.client.userinfo;
    
    public class DefaultOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    	...
    	@Override
    	public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
    		...
    		// convert() 로 request 요청 생성
    		RequestEntity<?> request = this.requestEntityConverter.convert(userRequest);
    
    		ResponseEntity<Map<String, Object>> response;
    
    		try {
    			// Authorization Server와 통신
    			response = this.restOperations.exchange(request, PARAMETERIZED_RESPONSE_TYPE);
    		} catch (OAuth2AuthorizationException ex) {
    			...
    		}
    
    		Map<String, Object> userAttributes = response.getBody();
    		Set<GrantedAuthority> authorities = new LinkedHashSet<>();
    		authorities.add(new OAuth2UserAuthority(userAttributes));
    		OAuth2AccessToken token = userRequest.getAccessToken();
    		for (String authority : token.getScopes()) {
    			authorities.add(new SimpleGrantedAuthority("SCOPE_" + authority));
    		}
    
    		return new DefaultOAuth2User(authorities, userAttributes, userNameAttributeName);
    	}
    
    	...
    }
```
Access Token을 받아오는 방식과 비슷하게 `DefaultOAuth2UserService`에서 user-info-endpoint 경로로 사용자 정보를 받아옵니다.
```java
    package org.springframework.security.oauth2.client.userinfo;
    
    public class OAuth2UserRequestEntityConverter {
    	...
    
    	@Override
    	public RequestEntity<?> convert(OAuth2UserRequest userRequest) {
    		ClientRegistration clientRegistration = userRequest.getClientRegistration();
    
    		...
    
    		HttpHeaders headers = new HttpHeaders();
    		headers.setAccept(Collections.singletonList(MediaType.APPLICATION_JSON));
    
    		// 기존에 Spring Security가 설정한 user-info-endpoint의 값을 불러와 uri 경로 설정
    		URI uri = UriComponentsBuilder.fromUriString(clientRegistration.getProviderDetails().getUserInfoEndpoint().getUri())
    				.build()
    				.toUri();
    
    		...
    
    		return request;
    	}
    	...
    }
```
`Client` - `Authorization Server` 통신 담당 클래스    
*위에서 설명한 Access Token 발급 로직, 사용자 정보 조회 로직을 호출하는 클래스다.*
```java
    package org.springframework.security.oauth2.client.authentication;
    
    public class OAuth2LoginAuthenticationProvider implements AuthenticationProvider {
    	...
    	
    	@Override
    	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    		
    		...
                OAuth2AccessTokenResponse accessTokenResponse;
		try {
			OAuth2AuthorizationExchangeValidator.validate(
					authorizationCodeAuthentication.getAuthorizationExchange());
                        
                        // access token 발급
			accessTokenResponse = this.accessTokenResponseClient.getTokenResponse(
					new OAuth2AuthorizationCodeGrantRequest(
							authorizationCodeAuthentication.getClientRegistration(),
							authorizationCodeAuthentication.getAuthorizationExchange()));

		} catch (OAuth2AuthorizationException ex) {
			OAuth2Error oauth2Error = ex.getError();
			throw new OAuth2AuthenticationException(oauth2Error, oauth2Error.toString());
		}

                // 위에서 받아온 accessToken 추출
    		OAuth2AccessToken accessToken = accessTokenResponse.getAccessToken();
    		Map<String, Object> additionalParameters = accessTokenResponse.getAdditionalParameters();
    
    		// 인가된 사용자 정보 가져옴 with accessToken
    		OAuth2User oauth2User = this.userService.loadUser(new OAuth2UserRequest(
    				authorizationCodeAuthentication.getClientRegistration(), accessToken, additionalParameters));
    
    		...
    
    		// 위에서 가져온 정보 저장
    		OAuth2LoginAuthenticationToken authenticationResult = new OAuth2LoginAuthenticationToken(
    			authorizationCodeAuthentication.getClientRegistration(),
    			authorizationCodeAuthentication.getAuthorizationExchange(),
    			oauth2User,
    			mappedAuthorities,
    			accessToken,
    			accessTokenResponse.getRefreshToken());
    		authenticationResult.setDetails(authorizationCodeAuthentication.getDetails());
    
    		return authenticationResult;
    	}
    
    	...
    }
```
위에서 저장된`OAuth2LoginAuthenticationToken authenticationResult` 는 추후 여러 권한 값과 함께 `OAuth2AuthenticationToken` 클래스를 생성하고 SecurityContextHolder에 저장됩니다.

SecurityContextHolder에 등록된 여러 컨텍스트들은 oauth 인증 - 인가 흐름에 맞는 생명주기로 관리됩니다.
```java
    package com.gaejangmo.apiserver;
    
    @ResController
    public class Controller {
    
      @GetMapping("/access_token")
      public String index(OAuth2AuthenticationToken authenticationToken) {

    	// SecurityContextHolder에 저장된 사용자 정보 사용
        log.info("authenticationToken {}", authenticationToken);
    
    	return null;
      }
    }
```
그래서 위와 같이 우리는 컨트롤러에서 oauth 인가된 사용자 정보를 관리하는 `OAuth2AuthenticationToken` 객체를 바로 사용할 수 있습니다.

### reference
* https://tools.ietf.org/html/rfc6749
