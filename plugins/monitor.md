2022-01-14

### 主要使用的是spring boot admin 和 actuator

#### 1. 建立服务端
1. 一个springboot服务端项目，依赖只需选择spring-web
2. 引入依赖
```pom 
    <dependency>
        <groupId>de.codecentric</groupId>
        <artifactId>spring-boot-admin-starter-client</artifactId>
        <version></version>
    </dependency>
```
3. 在启动类似增加注解@EnableAdminServer
4. 在配置文件中配置端口号 server.port=8000
5. 项目启动后即可访问 `localhost:8000` 看到界面

#### 2. 常规项目加入监控
1. 一个自己的项目，引入依赖
```pom
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.1.6</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

admin版本与spring boot version有关， 2.1.6在springboot也是2.1.的时候可用，更多的没测

2. 在配置文件中添加服务端相关
```
# 监控服务端 端口号
spring.boot.admin.client.url=http://localhost:8000
# 跟actuator有关，也许可以不配吧
management.endpoints.web.exposure.include=*
```

3. 启动项目，即可在监控服务端页面看到内存信息