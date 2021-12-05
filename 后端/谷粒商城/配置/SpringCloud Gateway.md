## 使用

1. 配置网关的服务注册：`@EnableDiscoveryClient`
2. 配置配置中心的地址等
3. 

## 配置跨域

**跨域**：指的是浏览器不能执行其他网站的脚本，由浏览器的同源策略造成的

**同源策略**：协议、域名、端口都要相同，其中有一个不同都会产生跨域

跨域的预检请求：OPTIONS。对于[非简单请求](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)（PUT、DELETE等），都要发送预检请求

解决方案：

1. nginx代理前后端

2. 服务器配置允许跨域请求，即网关处配置

   ```java
   // 配置跨域的Bean
   @Configuration
   public class GulimailCorsConfiguration {
   
       @Bean
       public CorsWebFilter corsWebFilter(){
           UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
           CorsConfiguration configuration = new CorsConfiguration();
           configuration.addAllowedHeader("*");
           configuration.addAllowedOrigin("*");
           configuration.addAllowedMethod("*");
           // 允许cookie
           configuration.setAllowCredentials(true);
           source.registerCorsConfiguration("/**", configuration);
           return new CorsWebFilter(source);
       }
   }
   ```

   