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

