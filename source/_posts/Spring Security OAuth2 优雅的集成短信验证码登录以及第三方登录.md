---
title: Spring Security OAuth2 优雅的集成短信验证码登录以及第三方登录
excerpt: 基于不侵入Spring Security OAuth2的原有代码、对于不同的登录方式不扩展新的端点、可以对所有登录方式进行兼容，介绍如何开发一套集成登录认证组件。
date: 2024-06-21 15:30:00
index_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/spring-security-oauth.png
banner_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/spring-security-oauth.png
tags:
- Spring Security OAuth2
- 短信登录
categories:
- Spring Security OAuth2
---
{% note info %}
本文由 Fluid 用户授权转载，版权归原作者所有。

本文作者：李球
原文地址：https://segmentfault.com/a/1190000014371789
{% endnote %}
# 前言
基于SpringCloud做微服务架构分布式系统时，OAuth2.0作为认证的业内标准，Spring Security OAuth2也提供了全套的解决方案来支持在Spring Cloud/Spring Boot环境下使用OAuth2.0，提供了开箱即用的组件。但是在开发过程中我们会发现由于Spring Security OAuth2的组件特别全面，这样就导致了扩展很不方便或者说是不太容易直指定扩展的方案，例如：
1. 图片验证码登录
2. 短信验证码登录
3. 微信小程序登录
4. 第三方系统登录
5. CAS单点登录

在面对这些场景的时候，预计很多对Spring Security OAuth2不熟悉的人恐怕会无从下手。基于上述的场景要求，如何优雅的集成短信验证码登录及第三方登录，怎么样才算是优雅集成呢？有以下要求：
1. 不侵入Spring Security OAuth2的原有代码
2. 对于不同的登录方式不扩展新的端点，使用/oauth/token可以适配所有的登录方式
3. 可以对所有登录方式进行兼容，抽象一套模型只要简单的开发就可以集成登录

基于上述的设计要求，接下来将会在文章种详细介绍如何开发一套集成登录认证组件开满足上述要求。
> 阅读本篇文章您需要了解OAuth2.0认证体系、SpringBoot、SpringSecurity以及Spring Cloud等相关知识

# 思路
我们来看下Spring Security OAuth2的认证流程：
![39483741-5ad07588c1f78](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/06/21/394837415ad07588c1f78.webp)

这个流程当中，切入点不多，集成登录的思路如下：
1. 在进入流程之前先进行拦截，设置集成认证的类型，例如：短信验证码、图片验证码等信息。
2. 在拦截的通知进行预处理，预处理的场景有很多，比如验证短信验证码是否匹配、图片验证码是否匹配、是否是登录IP白名单等处理
3. 在UserDetailService.loadUserByUsername方法中，根据之前设置的集成认证类型去获取用户信息，例如：通过手机号码获取用户、通过微信小程序OPENID获取用户等等

接入这个流程之后，基本上就可以优雅集成第三方登录。

# 实现
介绍完思路之后，下面通过代码来展示如何实现：

## 第一步，定义拦截器拦截登录的请求
```java

/**
 * @author LIQIU
 * @date 2018-3-30
 **/
@Component
public class IntegrationAuthenticationFilter extends GenericFilterBean implements ApplicationContextAware {

    private static final String AUTH_TYPE_PARM_NAME = "auth_type";

    private static final String OAUTH_TOKEN_URL = "/oauth/token";

    private Collection<IntegrationAuthenticator> authenticators;

    private ApplicationContext applicationContext;

    private RequestMatcher requestMatcher;

    public IntegrationAuthenticationFilter(){
        this.requestMatcher = new OrRequestMatcher(
                new AntPathRequestMatcher(OAUTH_TOKEN_URL, "GET"),
                new AntPathRequestMatcher(OAUTH_TOKEN_URL, "POST")
        );
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;

        if(requestMatcher.matches(request)){
            //设置集成登录信息
            IntegrationAuthentication integrationAuthentication = new IntegrationAuthentication();
            integrationAuthentication.setAuthType(request.getParameter(AUTH_TYPE_PARM_NAME));
            integrationAuthentication.setAuthParameters(request.getParameterMap());
            IntegrationAuthenticationContext.set(integrationAuthentication);
            try{
                //预处理
                this.prepare(integrationAuthentication);

                filterChain.doFilter(request,response);

                //后置处理
                this.complete(integrationAuthentication);
            }finally {
                IntegrationAuthenticationContext.clear();
            }
        }else{
            filterChain.doFilter(request,response);
        }
    }

    /**
     * 进行预处理
     * @param integrationAuthentication
     */
    private void prepare(IntegrationAuthentication integrationAuthentication) {

        //延迟加载认证器
        if(this.authenticators == null){
            synchronized (this){
                Map<String,IntegrationAuthenticator> integrationAuthenticatorMap = applicationContext.getBeansOfType(IntegrationAuthenticator.class);
                if(integrationAuthenticatorMap != null){
                    this.authenticators = integrationAuthenticatorMap.values();
                }
            }
        }

        if(this.authenticators == null){
            this.authenticators = new ArrayList<>();
        }

        for (IntegrationAuthenticator authenticator: authenticators) {
            if(authenticator.support(integrationAuthentication)){
                authenticator.prepare(integrationAuthentication);
            }
        }
    }

    /**
     * 后置处理
     * @param integrationAuthentication
     */
    private void complete(IntegrationAuthentication integrationAuthentication){
        for (IntegrationAuthenticator authenticator: authenticators) {
            if(authenticator.support(integrationAuthentication)){
                authenticator.complete(integrationAuthentication);
            }
        }
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```
在这个类种主要完成2部分工作：1、根据参数获取当前的是认证类型，2、根据不同的认证类型调用不同的IntegrationAuthenticator.prepar进行预处理

## 第二步，将拦截器放入到拦截链条中
```java
/**
 * @author LIQIU
 * @date 2018-3-7
 **/
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private IntegrationUserDetailsService integrationUserDetailsService;

    @Autowired
    private WebResponseExceptionTranslator webResponseExceptionTranslator;

    @Autowired
    private IntegrationAuthenticationFilter integrationAuthenticationFilter;

    @Autowired
    private DatabaseCachableClientDetailsService redisClientDetailsService;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        // TODO persist clients details
        clients.withClientDetails(redisClientDetailsService);
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints
                .tokenStore(new RedisTokenStore(redisConnectionFactory))
//                .accessTokenConverter(jwtAccessTokenConverter())
                .authenticationManager(authenticationManager)
                .exceptionTranslator(webResponseExceptionTranslator)
                .reuseRefreshTokens(false)
                .userDetailsService(integrationUserDetailsService);
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.allowFormAuthenticationForClients()
                .tokenKeyAccess("isAuthenticated()")
                .checkTokenAccess("permitAll()")
                .addTokenEndpointAuthenticationFilter(integrationAuthenticationFilter);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        jwtAccessTokenConverter.setSigningKey("cola-cloud");
        return jwtAccessTokenConverter;
    }
}
```
通过调用security. .addTokenEndpointAuthenticationFilter(integrationAuthenticationFilter);方法，将拦截器放入到认证链条中。

## 第三步，根据认证类型来处理用户信息
```java
@Service
public class IntegrationUserDetailsService implements UserDetailsService {

    @Autowired
    private UpmClient upmClient;

    private List<IntegrationAuthenticator> authenticators;

    @Autowired(required = false)
    public void setIntegrationAuthenticators(List<IntegrationAuthenticator> authenticators) {
        this.authenticators = authenticators;
    }

    @Override
    public User loadUserByUsername(String username) throws UsernameNotFoundException {
        IntegrationAuthentication integrationAuthentication = IntegrationAuthenticationContext.get();
        //判断是否是集成登录
        if (integrationAuthentication == null) {
            integrationAuthentication = new IntegrationAuthentication();
        }
        integrationAuthentication.setUsername(username);
        UserVO userVO = this.authenticate(integrationAuthentication);

        if(userVO == null){
            throw new UsernameNotFoundException("用户名或密码错误");
        }

        User user = new User();
        BeanUtils.copyProperties(userVO, user);
        this.setAuthorize(user);
        return user;

    }

    /**
     * 设置授权信息
     *
     * @param user
     */
    public void setAuthorize(User user) {
        Authorize authorize = this.upmClient.getAuthorize(user.getId());
        user.setRoles(authorize.getRoles());
        user.setResources(authorize.getResources());
    }

    private UserVO authenticate(IntegrationAuthentication integrationAuthentication) {
        if (this.authenticators != null) {
            for (IntegrationAuthenticator authenticator : authenticators) {
                if (authenticator.support(integrationAuthentication)) {
                    return authenticator.authenticate(integrationAuthentication);
                }
            }
        }
        return null;
    }
}
```
这里实现了一个IntegrationUserDetailsService ，在loadUserByUsername方法中会调用authenticate方法，在authenticate方法中会当前上下文种的认证类型调用不同的IntegrationAuthenticator 来获取用户信息，接下来来看下默认的用户名密码是如何处理的：
```java
@Component
@Primary
public class UsernamePasswordAuthenticator extends AbstractPreparableIntegrationAuthenticator {

    @Autowired
    private UcClient ucClient;

    @Override
    public UserVO authenticate(IntegrationAuthentication integrationAuthentication) {
        return ucClient.findUserByUsername(integrationAuthentication.getUsername());
    }

    @Override
    public void prepare(IntegrationAuthentication integrationAuthentication) {

    }

    @Override
    public boolean support(IntegrationAuthentication integrationAuthentication) {
        return StringUtils.isEmpty(integrationAuthentication.getAuthType());
    }
}
```
UsernamePasswordAuthenticator只会处理没有指定的认证类型即是默认的认证类型，这个类中主要是通过用户名获取密码。接下来来看下图片验证码登录如何处理的：
```java
/**
 * 集成验证码认证
 * @author LIQIU
 * @date 2018-3-31
 **/
@Component
public class VerificationCodeIntegrationAuthenticator extends UsernamePasswordAuthenticator {

    private final static String VERIFICATION_CODE_AUTH_TYPE = "vc";

    @Autowired
    private VccClient vccClient;

    @Override
    public void prepare(IntegrationAuthentication integrationAuthentication) {
        String vcToken = integrationAuthentication.getAuthParameter("vc_token");
        String vcCode = integrationAuthentication.getAuthParameter("vc_code");
        //验证验证码
        Result<Boolean> result = vccClient.validate(vcToken, vcCode, null);
        if (!result.getData()) {
            throw new OAuth2Exception("验证码错误");
        }
    }

    @Override
    public boolean support(IntegrationAuthentication integrationAuthentication) {
        return VERIFICATION_CODE_AUTH_TYPE.equals(integrationAuthentication.getAuthType());
    }
}
```
VerificationCodeIntegrationAuthenticator继承UsernamePasswordAuthenticator，因为其只是需要在prepare方法中验证验证码是否正确，获取用户还是用过用户名密码的方式获取。但是需要认证类型为"vc"才会处理
接下来来看下短信验证码登录是如何处理的：
```java
@Component
public class SmsIntegrationAuthenticator extends AbstractPreparableIntegrationAuthenticator implements  ApplicationEventPublisherAware {

    @Autowired
    private UcClient ucClient;

    @Autowired
    private VccClient vccClient;

    @Autowired
    private PasswordEncoder passwordEncoder;

    private ApplicationEventPublisher applicationEventPublisher;

    private final static String SMS_AUTH_TYPE = "sms";

    @Override
    public UserVO authenticate(IntegrationAuthentication integrationAuthentication) {

        //获取密码，实际值是验证码
        String password = integrationAuthentication.getAuthParameter("password");
        //获取用户名，实际值是手机号
        String username = integrationAuthentication.getUsername();
        //发布事件，可以监听事件进行自动注册用户
        this.applicationEventPublisher.publishEvent(new SmsAuthenticateBeforeEvent(integrationAuthentication));
        //通过手机号码查询用户
        UserVO userVo = this.ucClient.findUserByPhoneNumber(username);
        if (userVo != null) {
            //将密码设置为验证码
            userVo.setPassword(passwordEncoder.encode(password));
            //发布事件，可以监听事件进行消息通知
            this.applicationEventPublisher.publishEvent(new SmsAuthenticateSuccessEvent(integrationAuthentication));
        }
        return userVo;
    }

    @Override
    public void prepare(IntegrationAuthentication integrationAuthentication) {
        String smsToken = integrationAuthentication.getAuthParameter("sms_token");
        String smsCode = integrationAuthentication.getAuthParameter("password");
        String username = integrationAuthentication.getAuthParameter("username");
        Result<Boolean> result = vccClient.validate(smsToken, smsCode, username);
        if (!result.getData()) {
            throw new OAuth2Exception("验证码错误或已过期");
        }
    }

    @Override
    public boolean support(IntegrationAuthentication integrationAuthentication) {
        return SMS_AUTH_TYPE.equals(integrationAuthentication.getAuthType());
    }

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }
}
```
SmsIntegrationAuthenticator会对登录的短信验证码进行预处理，判断其是否非法，如果是非法的则直接中断登录。如果通过预处理则在获取用户信息的时候通过手机号去获取用户信息，并将密码重置，以通过后续的密码校验。

# 总结
在这个解决方案中，主要是使用责任链和适配器的设计模式来解决集成登录的问题，提高了可扩展性，并对spring的源码无污染。如果还要继承其他的登录，只需要实现自定义的IntegrationAuthenticator就可以。

项目地址：https://gitee.com/leecho/cola-cloud
大家有好的建议和想法可以一起沟通交流。





