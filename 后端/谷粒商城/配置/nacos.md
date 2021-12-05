## Nacos

启动Nacos-Server

```shell
# 单击启动
startup.cmd -m standalone
```



## Nacos服务注册与发现

步骤：

1. 引入依赖

   ```xml
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

   

2. 配置应用名和nacos服务器地址

   在application.yml文件中

   ```yaml
   spring:
     cloud:
       nacos:
         discovery:
           server-addr: 127.0.0.1:8848
     application:
       name: gulimail-coupon
   ```

   

### 远程调用

1. 在SpringBoot入口类上配置`@EnableFeignClients(basePackages = "com.sjm.gulimail.member.fegin")`

2. 在对应的包中创建接口，配置与远程调用服务的同一个方法（会使用HTTP）

   ```java
   // 声明式的远程调用，指定服务名
   @FeignClient("gulimail-coupon")
   public interface CouponFeginService {
   	// 与远程调用服务的@RequestMapping一致
       @RequestMapping("/coupon/coupon/coupons")
       R coupons();
   }
   
   // gulimail-coupon服务中
   @RestController
   @RequestMapping("coupon/coupon")
   public class CouponController {
       @RequestMapping("/coupons")
       public R coupons(){
           CouponEntity couponEntity = new CouponEntity();
           couponEntity.setCouponName("满1000减5");
           return R.ok().put("coupons", Arrays.asList(couponEntity));
       }
   }
   ```

   

3. 注入Service，然后就像调用本地方法一样进行远程调用



## Nacos配置中心

1. 引入依赖

   ```xml
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
   </dependency>
   ```

   

2. 在application.yml文件中配置

   ```yaml
   spring:
     cloud:
       nacos:
         config:
           server-addr: 127.0.0.1:8848
   ```

   

3. 在Nacos-Server中配置**Data ID**，默认规则为 **应用名.properties**

   如：gulimail-coupon.properties

4. 给 应用名.properties 添加任何配置

5. 动态获取配置

   `@RefreshScope`：动态刷新配置

   `@Value("{配置项名}")`：获取到配置

   ```java
   @RefreshScope
   @RestController
   @RequestMapping("coupon/coupon")
   public class CouponController {
       @Value("${coupon.user.name}")
       String name;
       @Value("${coupon.user.age}")
       int age;
   
       @RequestMapping("/get")
       public R test(){
           return R.ok().put("name", name).put("age", age);
       }
   }
   ```



### 其他细节

1. 命名空间：每个微服务之间互相隔离的配置，每个微服务创建自己的命名空间，只会加载命名空间下的所有配置。默认命名空间为public

   ```properties
   spring.cloud.nacos.config.namespace=UUID码
   ```

   

2. 配置集：所有配置的集合

3. 配置集ID（Data ID）

4. 配置分组：默认分组为DEFAULT_GROUP。可以选择，每个微服务创建自己的命名空间，使用配置分组区分环境：dev、test、prod

### 利用配置中心统一管理配置文件

1. 先将aplication.yml文件中的部分内容搬迁到bootstrap.properties文件中，然后配置配置集ID

   ```properties
   spring.application.name=gulimail-coupon
   spring.cloud.nacos.config.server-addr=127.0.0.1:8848
   server.port=7000
   # 指定命名空间
   spring.cloud.nacos.config.namespace=d7fe410c-0e1d-47dc-8198-19bbe6fa8ff6
   
   spring.cloud.nacos.config.extension-configs[0].data-id=datasource.yml
   spring.cloud.nacos.config.extension-configs[0].refresh=true
   spring.cloud.nacos.config.extension-configs[0].group=dev
   
   spring.cloud.nacos.config.extension-configs[1].data-id=mybatis.yml
   spring.cloud.nacos.config.extension-configs[1].refresh=true
   spring.cloud.nacos.config.extension-configs[1].group=dev
   
   spring.cloud.nacos.config.extension-configs[2].data-id=other.yml
   spring.cloud.nacos.config.extension-configs[2].refresh=true
   spring.cloud.nacos.config.extension-configs[2].group=dev
   ```

   

2. application.yml文件中的其余配置可以迁移到配置中心里

3. 开发环境可以用application.yml来配置，生产环境最好用配置中心。



