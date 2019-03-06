# apache-Shiro

## 关于CredentialsMatcher密码匹配

1. 置一个CredentialsMatcher

```java
@Bean
public HashedCredentialsMatcher hashedCredentialsMatcher(){
    HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
    hashedCredentialsMatcher.setHashAlgorithmName("MD5");
    hashedCredentialsMatcher.setHashIterations(1024);
    return hashedCredentialsMatcher;
}
```

2. 设置realm的CredentialsMatcher

```java
@Bean
public EmployeeRealm employeeRealm(){
    EmployeeRealm employeeRealm = new EmployeeRealm();
    employeeRealm.setCredentialsMatcher(hashedCredentialsMatcher());
    return employeeRealm;
}
```

3. 在realm中准备SimpleAuthenticationInfo返回

```java
//pwd为加密过的密码，getName()：返回realm的名
SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(empId,pwd,salt,getName());
```

## Spring用注解的方式配置Shiro

```java
@Configuration
public class ShiroConfig {

    /**
     * 配置SecurityManager
     */
    @Bean
    public SecurityManager securityManager(){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        /*
        由于userRealm()方法上添加了@Bean注解，Spring会拦截所有对它的调用，
        确保直接返回该方法创建的bean，而不是进行实际的调用
         */
        securityManager.setRealm(employeeRealm());
        return securityManager;
    }

    @Bean
    public EmployeeRealm employeeRealm(){
        EmployeeRealm employeeRealm = new EmployeeRealm();
        employeeRealm.setCredentialsMatcher(hashedCredentialsMatcher());
        return employeeRealm;
    }

    /*
    配置验证器
    */
    @Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher(){
        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
        hashedCredentialsMatcher.setHashAlgorithmName("MD5");
        hashedCredentialsMatcher.setHashIterations(1024);
        return hashedCredentialsMatcher;
    }

    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor(){
        return new LifecycleBeanPostProcessor();
    }

    @Bean(name = "shiroFilter")
    public ShiroFilterFactoryBean shiroFilterFactoryBean(){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager());

        Map<String, Filter> filters = new LinkedHashMap<>();
        //可添加filter，但是官网中有这么一句话
        /*
        因为filter类的bean在chain definitions通过它的beanName能自动被获得，所有'filters' 没有必要了，
        但是你能给它们起别名
         */
        shiroFilterFactoryBean.setFilters(filters);
        shiroFilterFactoryBean.setLoginUrl("/login.html");

        //使用LinkedHashMap，因为使用了FIRST MATCH WINS
        Map<String,String> filterChainDefinitions = new LinkedHashMap<>();
        filterChainDefinitions.put("/","anon");
        // ......
        filterChainDefinitions.put("/logout", "logout");
        //  /**表示任何请求/以及其子路径的都会触发filter chain
        filterChainDefinitions.put("/**", "authc");
        //要放入ShiroFilterFactoryBean才行
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitions);
        return shiroFilterFactoryBean;
    }

    /*
    DefaultAdvisorAutoProxyCreator和 AuthorizationAttributeSourceAdvisor
    启动Shiro注解（如@RequiresRoles, @RequiresPermissions），借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证
     */
    /*
    配置好像有点问题，并不能启动注解
    */
    @Bean
    @DependsOn("lifecycleBeanPostProcessor")//是一个String数组
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator(){
        DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
        return defaultAdvisorAutoProxyCreator;
    }
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(){
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager());
        return authorizationAttributeSourceAdvisor;
    }

    //Shiro支持Spring RPC调用
}

```

## Shiro对SpringBoot有了更方便的自动配置

[shiro-spring-boot-web-starter](https://shiro.apache.org/spring-boot.htm)

1. 配置starter

   ```xml
   <dependency>
       <groupId>org.apache.shiro</groupId>
       <artifactId>shiro-spring-boot-web-starter</artifactId>
       <version>1.4.1-SNAPSHOT</version>
   </dependency>
   ```

2. 提供一个Realm的实现

   ```java
   @Bean
   public Realm realm() {
     ...
   }
   ```

3. 定义`ShiroFilterChainDefinition` 

   ```java
   @Bean
   public ShiroFilterChainDefinition shiroFilterChainDefinition() {
       DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();
   
   	chainDefinition.addPathDefinition("/","anon");
       chainDefinition.addPathDefinition("/**", "authc");
       return chainDefinition;
   }
   ```

4. 注解已由启动器配置，可直接使用