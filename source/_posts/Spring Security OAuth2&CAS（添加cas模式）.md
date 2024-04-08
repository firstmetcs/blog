---
title: Spring Security OAuth2+CAS（添加cas模式）
excerpt: 本文简单介绍Spring Security OAuth2接入CAS并添加cas模式，现有的Spring Security OAuth2的认证对接cas认证中心。
date: 2024-04-08 12:50:00
index_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/spring-security.webp
banner_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/spring-security-oauth.png
tags:
- Spring Security OAuth2
- CAS
categories:
- Spring Security OAuth2
- CAS
---
{% note info %}
本文由 Fluid 用户授权转载，版权归原作者所有。

本文作者：纸上得来终觉浅，绝知此事要躬行
原文地址：https://blog.csdn.net/a294634473/article/details/109692209
{% endnote %}

# 前言
Spring Security OAuth2 自带有4中模式，有些特殊情况是需要自定义。（本文的前提是需要知道如何添加自定义的模式）
现在的需求是现有的Spring Security OAuth2的认证对接cas认证中心。

# maven
```xml
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-cas</artifactId>
			<version>5.0.6.RELEASE</version>
		</dependency>
```

# 添加CasTicketTokenGranter
这里的grant_type 就是这里定义的cas_ticket
```java
public class CasTicketTokenGranter extends AbstractTokenGranter {

    private static final String GRANT_TYPE = "cas_ticket";

    private final AuthenticationManager authenticationManager;

    public CasTicketTokenGranter(AuthenticationManager authenticationManager,
                                 AuthorizationServerTokenServices tokenServices, ClientDetailsService clientDetailsService, OAuth2RequestFactory requestFactory) {
        this(authenticationManager, tokenServices, clientDetailsService, requestFactory, GRANT_TYPE);
    }

    protected CasTicketTokenGranter(AuthenticationManager authenticationManager, AuthorizationServerTokenServices tokenServices,
                                    ClientDetailsService clientDetailsService, OAuth2RequestFactory requestFactory, String grantType) {
        super(tokenServices, clientDetailsService, requestFactory, grantType);
        this.authenticationManager = authenticationManager;
    }

    @Override
    public OAuth2AccessToken grant(String grantType, TokenRequest tokenRequest) {
        OAuth2AccessToken token = super.grant(grantType, tokenRequest);
        if (token != null) {
            DefaultOAuth2AccessToken norefresh = new DefaultOAuth2AccessToken(token);
            // The spec says that cas ticket should not be allowed to get a refresh token
            norefresh.setRefreshToken(null);
            token = norefresh;
        }
        return token;
    }

    @Override
    protected OAuth2Authentication getOAuth2Authentication(ClientDetails client, TokenRequest tokenRequest) {

        Map<String, String> parameters = new LinkedHashMap<String, String>(tokenRequest.getRequestParameters());
        String username = CasAuthenticationFilter.CAS_STATEFUL_IDENTIFIER;
        String password = parameters.get("ticket");

        if (password == null) {
            throw new InvalidRequestException("A cas ticket must be supplied.");
        }

        Authentication userAuth = new UsernamePasswordAuthenticationToken(username, password);
        ((AbstractAuthenticationToken) userAuth).setDetails(parameters);
        try {
            userAuth = authenticationManager.authenticate(userAuth);
        }
        catch (AccountStatusException ase) {
            //covers expired, locked, disabled cases (mentioned in section 5.2, draft 31)
            throw new InvalidGrantException(ase.getMessage());
        }
        catch (BadCredentialsException e) {
            // If the ticket is wrong the spec says we should send 400/invalid grant
            throw new InvalidGrantException(e.getMessage());
        }
        if (userAuth == null || !userAuth.isAuthenticated()) {
            throw new InvalidGrantException("Could not authenticate ticket: " + password);
        }

        OAuth2Request storedOAuth2Request = getRequestFactory().createOAuth2Request(client, tokenRequest);
        return new OAuth2Authentication(storedOAuth2Request, userAuth);
    }
}
```

# tokenGranters中添加自定义的cas模式
注意看最下面
```java
private List<TokenGranter> getDefaultTokenGranters() {
        AuthorizationServerTokenServices tokenServices = tokenServices();
        AuthorizationCodeServices authorizationCodeServices = authorizationCodeServices();
        OAuth2RequestFactory requestFactory = requestFactory();

        List<TokenGranter> tokenGranters = new ArrayList<TokenGranter>();
        // 添加授权码模式
        tokenGranters.add(new AuthorizationCodeTokenGranter(tokenServices, authorizationCodeServices, clientDetailsService, requestFactory));
        // 添加刷新令牌的模式
        tokenGranters.add(new RefreshTokenGranter(tokenServices, clientDetailsService, requestFactory));
        // 添加隐士授权模式
        tokenGranters.add(new ImplicitTokenGranter(tokenServices, clientDetailsService, requestFactory));
        // 添加客户端模式
        tokenGranters.add(new ClientCredentialsTokenGranter(tokenServices, clientDetailsService, requestFactory));
        if (authenticationManager != null) {
            // 添加密码模式
            tokenGranters.add(new ResourceOwnerPasswordTokenGranter(authenticationManager, tokenServices, clientDetailsService, requestFactory));
            // 添加自定义授权模式（手机号码）
            tokenGranters.add(new MobileCodeTokenGranter(authenticationManager, tokenServices, clientDetailsService, requestFactory));
            // 添加自定义授权模式（cas)
            tokenGranters.add(new CasTicketTokenGranter(authenticationManager, tokenServices, clientDetailsService, requestFactory));
        }
        return tokenGranters;
    }
```

# 添加CasAuthenticationProvider
注意cas携带的service地址要和 serviceProperties.setService()这里的地址一样
```java
	public static String casServerUrlPrefix="http://127.0.0.1:8081/cas";
	@Bean("pgtStorage")
    public ProxyGrantingTicketStorageImpl proxyGrantingTicketStorageImpl(){
        return new ProxyGrantingTicketStorageImpl();
    }

    @Bean("casAuthenticationProvider")
    public CasAuthenticationProvider casAuthenticationProvider(){
        CasAuthenticationProvider authenticationProvider=new CasAuthenticationProvider();
        authenticationProvider.setKey("casProvider") ;
        authenticationProvider.setAuthenticationUserDetailsService(OauthCasService);
        Cas20ProxyTicketValidator ticketValidator=new Cas20ProxyTicketValidator(casServerUrlPrefix);
        ticketValidator.setAcceptAnyProxy(true);//允许所有代理回调链接
        ticketValidator.setProxyGrantingTicketStorage(proxyGrantingTicketStorageImpl());
        authenticationProvider.setTicketValidator(ticketValidator);
        authenticationProvider.setServiceProperties(serviceProperties());
        return authenticationProvider;
    }

    @Bean
    public ServiceProperties serviceProperties(){
        ServiceProperties serviceProperties=new ServiceProperties();
        serviceProperties.setService("http://127.0.0.1:8881/base/oauth/toke");
        serviceProperties.setAuthenticateAllArtifacts(true);
        return serviceProperties;
    }
```

# AuthenticationManagerBuilder中添加Provider
```java
@Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(casAuthenticationProvider()).userDetailsService(OauthUserService).passwordEncoder(passwordEncoder()).and().authenticationProvider(mobileAuthenticationProvider());
    }
```

# OauthCasServiceImpl
```java
@Service("OauthCasService")
public class OauthCasServiceImpl implements AuthenticationUserDetailsService<CasAssertionAuthenticationToken> {

	@Autowired
	private com.datanew.service.UserService UserService;

	@Override
	public UserDetails loadUserDetails(CasAssertionAuthenticationToken token) throws UsernameNotFoundException {
		String name = token.getName();
		System.out.println("获得的用户名："+name);
		BaseOperator userinfo = UserService.getUserByUsername(name);
		if (userinfo==null){
			throw new UsernameNotFoundException(name+"不存在");
		}
		User user =  new User(name,userinfo.getPassword(), AuthorityUtils.createAuthorityList("ROLE_ADMIN"));
		return new OauthUser(userinfo,user);
	}
}

public interface OauthCasService extends UserDetailsService {

}
```

# cas登录流程
需要cas认证中心支持restfult模式
1. 客户端使用用户名密码获取TGT
2. 使用TGT获取ticket
3. 使用ticket获取access_token
4. 一般来说需要自己封装一个接口获取token并把token设置到cookie中并重定向到前端项目
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/04/08/17125514772252.jpg)
