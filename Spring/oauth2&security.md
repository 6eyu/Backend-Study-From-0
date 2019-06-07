## 入门Spring Boot Security 和 Oauth2：

Authentication 和 Authorization 服务由**3**部分构成。
1. Class **SecurityConfig** (extends **WebSecurityConfigurerAdapter**) 
    <br>依赖于Spring Security
    <br>作用：**认证** Authentication
    - 过滤URL请求 
    - 获取User信息&验证信息

2. Class **ResourceServerConfig** (extends **ResourceServerConfigurerAdapter**)
    <br>依赖于Oauth2
    <br>作用：
    - 过滤URL请求 **(#### ?与securityconfig 的区别 有待进一步研究 ####)**
    - 设置访问ROLE

3. Class **AuthorizationServerConfig** (extends **AuthorizationServerConfigurerAdapter**)
    <br>依赖于Oauth2
    <br>作用：**授权** Authorization
    - 检测终端(client)信息: APP, WebApp等
    - 发放&储存令牌(token): JWT等

代码：
- SecurityConfig.java
```
//这两个annotation一定要标注上
@Configuration 
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private JdbcUserDetails jdbcUserDetails; //从数据库获取User信息

    //需指定AuthenticationManager
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        AuthenticationManager manager = super.authenticationManagerBean();
        return manager;
    }

    //指定认证供应信息。
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(jdbcUserDetails)
//                .passwordEncoder(new BCryptPasswordEncoder())// 指定编码方式, 因下面passwordEncoder()已经指定，这里不需要再重复
                ;
    }

    //指定密码模式
    @Bean
    PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    //过滤URL请求
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/oauth2test/**", "/client","/oauth/**").permitAll() //允许URL通过
                .antMatchers("/oauth2private/**").authenticated() //指定URL需认证
                .and()
                .httpBasic()
                .and()
                .csrf().disable();
    }
}

```

- ResourceServerConfig.java
```
//需要annotation
@EnableResourceServer
@Configuration
public class ResourceConfig extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {

        http.authorizeRequests().antMatchers("/private/**").hasRole("USER"); //设置权限

    }

}

```

- AuthorizationServerConfig.java
```
//需要annotation
@Configuration
@EnableAuthorizationServer
public class AuthenticationConfig extends AuthorizationServerConfigurerAdapter {
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private DataSource dataSource;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {

        clients.jdbc(dataSource).build(); //链接数据库 获得 client信息
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.tokenKeyAccess("permitAll()")
                .checkTokenAccess("permitAll()");//设置permitAll后可以用过postman调通URI: "/oauth/check_token"
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenStore(new InMemoryTokenStore()) //token 存于内存中，
                .authenticationManager(authenticationManager)
                .allowedTokenEndpointRequestMethods(HttpMethod.GET, HttpMethod.POST);
    }
}
```

### PasswordEncoder方式： 
Oauth2 默认为 DelegatingPasswordEncoder() 可以解析 有前缀的 Hash 值 <br>如：{BCrypt}

有 2 种方式

> 1. 服务内全局设置: 

```
    @Bean
    PasswordEncoder passwordEncoder(){
        return PasswordEncoderFactories.createDelegatingPasswordEncoder(); //可修改指定编码器 如 new BCryptPasswordEncoder()
    }
```
通过该设置 <SecurityConfig.java> 中的 User password 加密 和 <AuthorizationServerConfig.java> 中 client secret 加密 都可以通过@Autowired调用. <br>**注意**只能设置一次

> 2. 单独设置解码器
    <br>**注**：如果使用此方法， <AuthorizationServerConfig.java> 中 Clients secret 还是需要用方法(1) Bean 声明 Encoder, 
    <br>因为无法在client中使用passwordEncoder()方法注入指定解码.  ***也许可以，但是我没有实验成功：）***
```
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(jdbcUserDetails)
            .passwordEncoder(new BCryptPasswordEncoder()) // 指定编码方式
            ;
    }
```
如果自定义了 AuthenticationManagerProvider (如**extends** AbstractUserDetailsAuthenticationProvider). 请注意在constructor方法中指定需要的passwordEncoder方式. 或者不加入制定Encoder而通过外部调用set方法加入 ***应该可以行 :)***
```
public XxxAuthenticationProvider() {
    this.setPasswordEncoder(new BCryptPasswordEncoder());
}
```

## * TODO salt value 存储

![](http://latex.codecogs.com/gif.latex?\\frac{1}{1+sin(x)})

