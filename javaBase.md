# 跨域问题的解决
[江南一点雨](https://mp.weixin.qq.com/s/ASEJwiswLu1UCRE-e2twYQ)

## 同源策略

很多人对跨域有误解，以为是前端的事，跟后端没有关系，其实不是，跨域，牵扯到 浏览器的同源策略
同源策略是Netscape提出的著名的安全策略，是浏览器最核心也是最基本的安全功能，现在所有支持JavaScript的浏览器都会使用这个策略。所谓同源是指协议、域名以及端口要相同。同源策略是基于安全方面的考虑提出来的，这个策略本身没问题，但是实际开发中，由于各种原因经常有跨域问题的需求，传统的跨域方案是JSONP，JSONP虽然能解决跨域但是有很大的局限性，只支持GET请求，不支持其他类型。CORS（跨域源资源共享，Cross-origin resource sharing）是一个W3C标准，是一份浏览器技术的规范，提供了Web服务从不同网域传来沙盒脚本的方法，以避开浏览器的同源策略，是JSONP模式的现代版。
## SpringBoot实现CORS
### 跨域现象
```javascript
Access to XMLHttpRequest at 'http://localhost:8080/hello'
from origin 'http://localhost:8081' has been blocked by CORS policy :
No 'Access-Control-Allow-Origin' header is present on the requested resource.
```
### 某一接口接受某一域的请求
缺点：每个接口都要加
```java
@RestController
public class HelloController {

    @CrossOrigin(value = "http://localhost:8081")
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }


    @CrossOrigin(value = "http://localhost:8081")
    @PostMapping("/hello")
    public String hello2() {
        return "post hello";
    }

}
```
### 全局配置
```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:8081")
                .allowedMethods("*")
                .allowedHeaders("*");
    }
}
```
`/**`表示本应用的所有方法都会去处理跨域请求，`allowedMethods`表示允许通过的请求数，`allowedHeaders`则表示允许的请求头。经过这样的配置之后，就不必在每个方法上单独配置跨域了。
### 存在的问题
常见的CSRF(Cross-site request forgery)跨站请求伪造。跨站请求伪造也被称为one-click attacj或者session riding。通常缩写我CSRF或XSRF，是一种挟持用户在当前已登录的Web应用程序上执行非本意的操作的攻击方式。
假如一家银行用以运行转账操作的URL地址如下： http://icbc.com/aa?bb=cc，
那么，一个恶意攻击者可以在另一个网站上放置如下代码： <imgsrc="http://icbc.com/aa?bb=cc">，
如果用户访问了恶意站点，而她之前刚访问过银行不久，登录信息尚未过期，那么她就会遭受损失
基于此，浏览器在实际操作中，会对请求进行分类，分为简单请求，预先请求，带凭证的请求等，预先请求会首先发送一个options探测请求，和浏览器进行协商是否接受请求。默认情况下跨域请求是不需要凭证的，但是服务端可以配置要求客户端提供凭证，这样就可以有效避免csrf攻击。

### 公司使用的这个
```java
@Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurerAdapter() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**");
            }
        };
    }
    ```
WebMvcConfigurerAdapter已经过时啦，用WebMvcConfigurer