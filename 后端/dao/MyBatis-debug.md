# MyBatis-debug

**返回List**时，resultType应写为POJO类

```java
List<User> findAll();
```

```xml
<select id="findAll" resultType="com.sonkabin.bean.User">
    select * from user
</select>
```

